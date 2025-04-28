# Odoo Module: l10n_it_edi

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import tools

from odoo import api, SUPERUSER_ID


def _l10n_it_edi_update_export_tax(env):
    chart_template = env.ref('l10n_it.l10n_it_chart_template_generic', raise_if_not_found=False)
    if chart_template:
        for company in env['res.company'].search([('chart_template_id', '=', chart_template.id)]):
            tax = env.ref(f'l10n_it.{company.id}_00eu', raise_if_not_found=False)
            if tax:
                tax.write({
                    'l10n_it_has_exoneration': True,
                    'l10n_it_kind_exoneration': 'N3.2',
                    'l10n_it_law_reference': 'Art. 41, DL n. 331/93',
                })


def _l10n_it_edi_post_init(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    _l10n_it_edi_update_export_tax(env)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Italy - E-invoicing',
    'icon': '/l10n_it/static/description/icon.png',
    'version': '0.3',
    'depends': [
        'l10n_it',
        # Although account_edi is a dependency of account_edi_proxy_client,
        # it is here because it's in the auto-install
        'account_edi',
        'account_edi_proxy_client',
    ],
    'auto_install': ['l10n_it', 'account_edi'],
    'author': 'Odoo',
    'description': """
E-invoice implementation
    """,
    'category': 'Accounting/Localizations/EDI',
    'website': 'http://www.odoo.com/',
    'data': [
        'security/ir.model.access.csv',
        'data/account_edi_data.xml',
        'data/invoice_it_template.xml',
        'data/invoice_it_simplified_template.xml',
        'data/ir_cron.xml',
        'views/res_config_settings_views.xml',
        'views/l10n_it_view.xml',
        ],
    'demo': [
        'data/account_invoice_demo.xml',
    ],
    'post_init_hook': '_l10n_it_edi_post_init',
    'license': 'LGPL-3',
}

```

## File: data\account_edi_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="edi_fatturaPA" model="account.edi.format">
            <field name="name">Fattura PA (IT)</field>
            <field name="code">fattura_pa</field>
        </record>

    </data>
</odoo>
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

        <record id="demo_l10n_it_edi_partner_pa_fiscal_position" model="ir.property">
            <field name="name">property_account_position_id</field>
            <field name="fields_id" search="[('model', '=', 'res.partner'), ('name', '=', 'property_account_position_id')]"/>
            <field name="res_id" model="res.partner" eval="'res.partner,' + str(obj().env.ref('l10n_it_edi.demo_l10n_it_edi_partner_pa').id)"/>
            <field name="value" eval="'account.fiscal.position,' 
                + str(ref('l10n_it.' + str(ref('l10n_it.demo_company_it')) + '_split_payment_fiscal_position'))"/>
            <field name="company_id" ref="l10n_it.demo_company_it"/>
        </record>

        <record id="demo_l10n_it_edi_proxy_user" model="account_edi_proxy_client.user">
            <field name="id_client">demo_id_client</field>
            <field name="company_id" ref="l10n_it.demo_company_it"/>
            <field name="edi_format_id" ref="l10n_it_edi.edi_fatturaPA"/>
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
                    <t t-esc="format_alphanumeric(line.name[:1000])"/>
                    <t t-if="not line.name" t-esc="'NO NAME'"/>
                </Descrizione>
                <Importo t-esc="format_monetary(line.price_total, currency)"/>
                <DatiIVA>
                    <Imposta t-esc="format_monetary(line.price_total - line.price_subtotal, currency)"/>
                </DatiIVA>
                <Natura t-if="line.tax_ids.l10n_it_has_exoneration" t-esc="line.tax_ids.l10n_it_kind_exoneration"/>
            </DatiBeniServizi>
        </template>

        <template id="account_invoice_it_simplified_FatturaPA_export">
            <t t-set="currency" t-value="record.currency_id or record.company_currency_id"/>
            <t t-set="bank" t-value="record.partner_bank_id"/>
            <p:FatturaElettronicaSemplificata  t-att-versione="formato_trasmissione" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:p="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.0 http://www.fatturapa.gov.it/export/fatturazione/sdi/fatturapa/v1.0/Schema_del_file_xml_FatturaPA_versione_1.0.xsd">
            <FatturaElettronicaHeader>
                    <DatiTrasmissione>
                        <IdTrasmittente>
                            <IdPaese t-esc="get_vat_country(record.company_id.vat)"/>
                            <IdCodice t-esc="normalize_codice_fiscale(record.company_id.l10n_it_codice_fiscale) or get_vat_number(record.company_id.vat)"/>
                        </IdTrasmittente>
                        <ProgressivoInvio t-esc="format_alphanumeric(record.name.replace('/','')[-10:])"/>
                        <FormatoTrasmissione t-esc="formato_trasmissione"/>
                        <CodiceDestinatario t-if="record.commercial_partner_id.l10n_it_pa_index" t-esc="record.commercial_partner_id.l10n_it_pa_index.upper()"/>
                        <CodiceDestinatario t-if="not record.commercial_partner_id.l10n_it_pa_index" t-esc="'0000000'"/>
                        <PECDestinatario t-if="record.commercial_partner_id.l10n_it_pec_email" t-esc="record.commercial_partner_id.l10n_it_pec_email"/>
                    </DatiTrasmissione>
                    <CedentePrestatore>
                        <IdFiscaleIVA>
                            <IdPaese t-esc="get_vat_country(record.company_id.vat)"/>
                            <IdCodice t-esc="get_vat_number(record.company_id.vat)"/>
                        </IdFiscaleIVA>
                        <CodiceFiscale t-if="record.company_id.l10n_it_codice_fiscale" t-esc="normalize_codice_fiscale(record.company_id.l10n_it_codice_fiscale)"/>
                        <Denominazione t-esc="format_alphanumeric(record.company_id.partner_id.display_name[:80])"/>
                        <t t-call="l10n_it_edi.account_invoice_it_FatturaPA_sede">
                            <t t-set="partner" t-value="record.company_id.partner_id"/>
                        </t>
                        <RappresentanteFiscale t-if="record.company_id.l10n_it_has_tax_representative">
                            <IdFiscaleIVA>
                                <IdPaese t-esc="get_vat_country(record.company_id.l10n_it_tax_representative_partner_id.vat)"/>
                                <IdCodice t-esc="get_vat_number(record.company_id.l10n_it_tax_representative_partner_id.vat)"/>
                            </IdFiscaleIVA>
                            <Anagrafica>
                                <Denominazione t-if="record.commercial_partner_id.is_company" t-esc="format_alphanumeric(record.commercial_partner_id.display_name[:80])"/>
                                <Nome t-if="not record.commercial_partner_id.is_company" t-esc="format_alphanumeric(' '.join(record.commercial_partner_id.name.split()[:1])[:60])"/>
                                <Cognome t-if="not record.commercial_partner_id.is_company" t-esc="format_alphanumeric(' '.join(record.commercial_partner_id.name.split()[1:])[:60])"/>
                            </Anagrafica>
                        </RappresentanteFiscale>
                        <IscrizioneREA t-if="record.company_id.l10n_it_has_eco_index">
                            <Ufficio t-esc="record.company_id.l10n_it_eco_index_office.code"/>
                            <NumeroREA t-esc="format_alphanumeric(record.company_id.l10n_it_eco_index_number)"/>
                            <CapitaleSociale t-if="record.company_id.l10n_it_eco_index_share_capital != 0" t-esc="format_numbers_two(record.company_id.l10n_it_eco_index_share_capital)"/>
                            <SocioUnico t-if="record.company_id.l10n_it_eco_index_sole_shareholder != 'NO'" t-esc="record.company_id.l10n_it_eco_index_sole_shareholder"/>
                            <StatoLiquidazione t-esc="record.company_id.l10n_it_eco_index_liquidation_state"/>
                        </IscrizioneREA>
                        <RegimeFiscale t-esc="record.company_id.l10n_it_tax_system"/>
                    </CedentePrestatore>
                    <CessionarioCommittente>
                        <IdentificativiFiscali>
                            <IdFiscaleIVA t-if="record.commercial_partner_id.vat and in_eu(record.commercial_partner_id)">
                                <IdPaese t-esc="get_vat_country(record.commercial_partner_id.vat)"/>
                                <IdCodice t-esc="get_vat_number(record.commercial_partner_id.vat)"/>
                            </IdFiscaleIVA>
                            <CodiceFiscale t-if="record.commercial_partner_id.l10n_it_codice_fiscale" t-esc="normalize_codice_fiscale(record.commercial_partner_id.l10n_it_codice_fiscale)"/>
                        </IdentificativiFiscali>
                    </CessionarioCommittente>
                </FatturaElettronicaHeader>
                <FatturaElettronicaBody>
                    <DatiGenerali>
                        <DatiGeneraliDocumento>
                            <!--2.1.1-->
                            <TipoDocumento t-esc="document_type"/>
                            <Divisa t-esc="currency.name"/>
                            <Data t-esc="format_date(record.invoice_date)"/>
                            <Numero t-esc="format_alphanumeric(record.name[-20:])"/>
                        </DatiGeneraliDocumento>
                        <DatiFatturaRettificata t-if="record.move_type == 'out_refund' and record.reversed_entry_id">
                            <NumeroFR t-esc="format_alphanumeric(record.reversed_entry_id.name[-20:])"/>
                            <DataFR t-esc="format_date(record.reversed_entry_id.invoice_date)"/>
                            <ElementiRettificati t-esc="format_alphanumeric(record.ref[:1000])"/>
                        </DatiFatturaRettificata>
                    </DatiGenerali>
                    <!-- Invoice lines. -->
                    <t t-foreach="record.invoice_line_ids.filtered(lambda l: l.display_type not in ('line_note', 'line_section'))" t-as="line">
                        <t t-call="l10n_it_edi.account_invoice_line_it_simplified_FatturaPA"/>
                    </t>
                    <Allegati t-if="pdf">
                        <NomeAttachment t-esc="format_alphanumeric(pdf_name[:60])"/>
                        <FormatoAttachment t-translation="off">PDF</FormatoAttachment>
                        <Attachment t-esc="pdf"/>
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
                    <NumeroLinea t-esc="line_dict['line_number']"/>
                    <CodiceArticolo t-if="line.product_id.barcode">
                        <CodiceTipo t-translation="off">EAN</CodiceTipo>
                        <CodiceValore t-esc="format_alphanumeric(line.product_id.barcode)[:35]"/>
                    </CodiceArticolo>
                    <CodiceArticolo t-elif="line.product_id.default_code">
                        <CodiceTipo t-translation="off">INTERNAL</CodiceTipo>
                        <CodiceValore t-esc="format_alphanumeric(line.product_id.default_code)[:35]"/>
                    </CodiceArticolo>
                    <Descrizione t-esc="format_alphanumeric(line_dict['description'])[:1000]"/>
                    <Quantita t-esc="format_numbers(abs(line.quantity))"/>
                    <UnitaMisura t-if="line.product_uom_id and line.product_uom_id.category_id != env.ref('uom.product_uom_categ_unit')" t-esc="format_alphanumeric(line.product_uom_id.name)[:10]"/>
                    <PrezzoUnitario t-esc="'%.06f' % (line_dict['unit_price'])"/>
                    <ScontoMaggiorazione t-if="line.discount != 0">
                        <Tipo t-esc="discount_type(line.discount)"/>
                        <Percentuale t-esc="format_numbers(abs(line.discount))"/>
                    </ScontoMaggiorazione>
                    <PrezzoTotale t-esc="format_monetary(line_dict['subtotal_price'], currency)"/>
                    <AliquotaIVA t-if="vat_tax.amount_type == 'percent'" t-esc="format_numbers(vat_tax.amount)"/>
                    <AliquotaIVA t-elif="vat_tax.amount_type != 'percent'" t-esc="'0.00'"/>
                    <Natura t-if="vat_tax.l10n_it_has_exoneration" t-esc="vat_tax.l10n_it_kind_exoneration"/>
                    <AltriDatiGestionali t-if="conversion_rate">
                        <TipoDato t-translation="off">DIVISA</TipoDato>
                        <RiferimentoTesto t-esc="format_alphanumeric(record.currency_id.name)"/>
                        <RiferimentoNumero t-esc="'%.06f' % line.price_subtotal"/>
                    </AltriDatiGestionali>
                    <AltriDatiGestionali t-if="conversion_rate">
                        <TipoDato t-translation="off">CAMBIO</TipoDato>
                        <RiferimentoNumero t-esc="conversion_rate"/>
                        <RiferimentoData t-esc="format_date(record.invoice_date)"/>
                    </AltriDatiGestionali>
                </DettaglioLinee>
</template>

<template id="account_invoice_it_FatturaPA_export">
<p:FatturaElettronica t-att-versione="formato_trasmissione" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:p="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.2" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.2 http://www.fatturapa.gov.it/export/fatturazione/sdi/fatturapa/v1.2/Schema_del_file_xml_FatturaPA_versione_1.2.xsd">
    <FatturaElettronicaHeader>
        <DatiTrasmissione>
            <IdTrasmittente>
                <IdPaese t-esc="get_vat_country(sender.vat)"/>
                <IdCodice t-esc="normalize_codice_fiscale(sender.l10n_it_codice_fiscale) or get_vat_number(sender.vat)"/>
            </IdTrasmittente>
            <ProgressivoInvio t-esc="format_alphanumeric(record.name.replace('/','')[-10:])"/>
            <FormatoTrasmissione t-esc="formato_trasmissione"/>
            <CodiceDestinatario t-esc="codice_destinatario"/>
            <ContattiTrasmittente>
                <Telefono t-if="sender_partner.phone" t-esc="format_alphanumeric(format_phone(sender_partner.phone))"/>
                <Telefono t-elif="sender_partner.mobile" t-esc="format_alphanumeric(format_phone(sender_partner.mobile))"/>
                <Email t-if="sender_partner.email" t-esc="sender_partner.email[:256]"/>
            </ContattiTrasmittente>
            <PECDestinatario t-if="not is_self_invoice and partner.l10n_it_pec_email" t-esc="partner.l10n_it_pec_email[:256]"/>
        </DatiTrasmissione>
        <CedentePrestatore>
            <DatiAnagrafici>
                <IdFiscaleIVA>
                    <IdPaese t-out="seller_info['country_code']"/>
                    <IdCodice t-out="seller_info['vat']"/>
                </IdFiscaleIVA>
                <CodiceFiscale t-if="seller.l10n_it_codice_fiscale" t-esc="normalize_codice_fiscale(seller.l10n_it_codice_fiscale)"/>
                <Anagrafica>
                    <Denominazione t-esc="format_alphanumeric(seller_partner.display_name[:80])"/>
                </Anagrafica>
                <RegimeFiscale t-esc="regime_fiscale"/>
            </DatiAnagrafici>
            <t t-call="l10n_it_edi.account_invoice_it_FatturaPA_sede">
                <t t-set="partner" t-value="seller_partner"/>
            </t>
            <IscrizioneREA t-if="not is_self_invoice and company.l10n_it_has_eco_index">
                <Ufficio t-esc="company.l10n_it_eco_index_office.code"/>
                <NumeroREA t-esc="format_alphanumeric(company.l10n_it_eco_index_number)"/>
                <CapitaleSociale t-if="company.l10n_it_eco_index_share_capital != 0" t-esc="format_numbers_two(company.l10n_it_eco_index_share_capital)"/>
                <SocioUnico t-if="company.l10n_it_eco_index_sole_shareholder != 'NO'" t-esc="company.l10n_it_eco_index_sole_shareholder"/>
                <StatoLiquidazione t-esc="company.l10n_it_eco_index_liquidation_state"/>
            </IscrizioneREA>
        </CedentePrestatore>
        <RappresentanteFiscale t-if="not is_self_invoice and representative">
            <DatiAnagrafici>
                <IdFiscaleIVA>
                    <IdPaese t-esc="get_vat_country(representative.vat)"/>
                    <IdCodice t-esc="get_vat_number(representative.vat)"/>
                </IdFiscaleIVA>
                <CodiceFiscale t-if="representative.l10n_it_codice_fiscale" t-esc="normalize_codice_fiscale(representative.l10n_it_codice_fiscale)"/>
                <Anagrafica>
                <t t-if="representative.is_company">
                    <Denominazione t-esc="format_alphanumeric(representative.display_name[:80])"/>
                </t>
                <t t-else="">
                    <Nome t-esc="format_alphanumeric(' '.join(representative.name.split()[:1])[:60])"/>
                    <Cognome t-esc="format_alphanumeric(' '.join(representative.name.split()[1:])[:60])"/>
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
                <CodiceFiscale t-if="buyer.l10n_it_codice_fiscale" t-esc="normalize_codice_fiscale(buyer.l10n_it_codice_fiscale)"/>
                <Anagrafica>
                    <t t-if="buyer_is_company">
                        <Denominazione t-esc="format_alphanumeric(buyer.display_name[:80])"/>
                    </t>
                    <t t-else="">
                        <Nome t-esc="format_alphanumeric(' '.join(buyer.name.split()[:1])[:60])"/>
                        <Cognome t-esc="format_alphanumeric(' '.join(buyer.name.split()[1:])[:60])"/>
                    </t>
                </Anagrafica>
            </DatiAnagrafici>
            <t t-call="l10n_it_edi.account_invoice_it_FatturaPA_sede">
                <t t-set="partner" t-value="buyer_partner"/>
            </t>
        </CessionarioCommittente>
    </FatturaElettronicaHeader>
    <FatturaElettronicaBody>
        <DatiGenerali>
            <DatiGeneraliDocumento>
                <TipoDocumento t-esc="document_type"/>
                <Divisa t-esc="currency.name"/>
                <Data t-esc="format_date(record.invoice_date)"/>
                <Numero t-esc="format_alphanumeric(record.name[-20:])"/>
                <DatiBollo t-if="record.l10n_it_stamp_duty">
                    <BolloVirtuale t-translation="off">SI</BolloVirtuale>
                    <ImportoBollo t-esc="format_numbers(record.l10n_it_stamp_duty)"/>
                </DatiBollo>
                <ImportoTotaleDocumento t-esc="format_monetary(document_total, currency)"/>
            </DatiGeneraliDocumento>
            <DatiOrdineAcquisto t-if="origin_document_type == 'purchase_order'">
                <t t-call="l10n_it_edi.account_invoice_FatturaPA_origin_document"/>
            </DatiOrdineAcquisto>
            <DatiOrdineAcquisto t-elif="record.ref and not record.reversed_entry_id">
                <IdDocumento t-esc="format_alphanumeric(record.ref[:20])"/>
            </DatiOrdineAcquisto>
            <DatiContratto t-if="origin_document_type == 'contract'">
                <t t-call="l10n_it_edi.account_invoice_FatturaPA_origin_document"/>
            </DatiContratto>
            <DatiConvenzione t-if="origin_document_type == 'agreement'">
                <t t-call="l10n_it_edi.account_invoice_FatturaPA_origin_document"/>
            </DatiConvenzione>
            <DatiFattureCollegate t-if="record.reversed_entry_id">
                <IdDocumento t-out="format_alphanumeric(record.reversed_entry_id.name[-20:])"/>
                <Data t-out="format_date(record.reversed_entry_id.invoice_date)"/>
            </DatiFattureCollegate>
            <DatiFattureCollegate t-foreach="downpayment_moves" t-as="downpayment_move">
                <IdDocumento t-out="format_alphanumeric(downpayment_move.name[-20:])"/>
                <Data t-out="format_date(downpayment_move.invoice_date)"/>
            </DatiFattureCollegate>
            <DatiDDT t-if="record.l10n_it_ddt_id">
                <NumeroDDT t-esc="format_alphanumeric(record.l10n_it_ddt_id.name[-20:])"/>
                <DataDDT t-esc="format_date(record.l10n_it_ddt_id.date)"/>
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
                <t t-set="has_exoneration" t-value="tax.l10n_it_has_exoneration"/>
                <t t-set="kind_exoneration" t-value="tax.l10n_it_kind_exoneration"/>
                <DatiRiepilogo>
                    <AliquotaIVA t-esc="format_numbers(tax.amount)"/>
                    <Natura t-if="has_exoneration" t-esc="kind_exoneration"/>
                    <Arrotondamento t-if="tax_line.get('rounding')" t-esc="format_numbers(-tax_line['rounding'])"/>
                    <t t-if="rc_refund">
                        <ImponibileImporto t-esc="format_monetary(balance_multiplicator * tax_line['base_amount'], currency)"/>
                        <Imposta t-esc="format_monetary(balance_multiplicator * tax_line['tax_amount'], currency)"/>
                    </t>
                    <t t-else="">
                        <ImponibileImporto t-esc="format_monetary(tax_line['base_amount'], currency)"/>
                        <Imposta t-esc="format_monetary(tax_line['tax_amount'], currency)"/>
                    </t>
                    <EsigibilitaIVA t-esc="tax_line['exigibility_code']"/>
                    <RiferimentoNormativo t-if="tax.l10n_it_law_reference" t-esc="format_alphanumeric(tax.l10n_it_law_reference[:100])"/>
                </DatiRiepilogo>
            </t>
        </DatiBeniServizi>
        <DatiPagamento t-if="partner_bank and record.move_type != 'out_refund'">
            <t t-set="payments" t-value="record.line_ids.filtered(lambda line: line.account_id.account_type in ('asset_receivable', 'liability_payable'))"/>
            <CondizioniPagamento t-translation="off"><t t-if="len(payments) == 1">TP02</t><t t-else="">TP01</t></CondizioniPagamento>
            <t t-foreach="payments" t-as="payment">
                <DettaglioPagamento>
                    <ModalitaPagamento t-translation="off">MP05</ModalitaPagamento>
                    <DataScadenzaPagamento t-esc="format_date(payment.date_maturity)"/>
                    <ImportoPagamento t-esc="format_monetary(abs(payment.amount_currency), currency)"/>
                    <IstitutoFinanziario t-if="partner_bank.bank_id" t-esc="format_alphanumeric(partner_bank.bank_id.name[:80])"/>
                    <IBAN t-if="partner_bank.acc_type == 'iban'" t-esc="partner_bank.sanitized_acc_number"/>
                    <BIC t-elif="partner_bank.acc_type == 'bank' and partner_bank.bank_id.bic" t-esc="partner_bank.bank_id.bic"/>
                    <CodicePagamento t-if="record.payment_reference" t-esc="format_alphanumeric(record.payment_reference[:60])"/>
                </DettaglioPagamento>
            </t>
        </DatiPagamento>
        <Allegati t-if="pdf">
            <NomeAttachment t-esc="format_alphanumeric(pdf_name[:60])"/>
            <FormatoAttachment t-translation="off">PDF</FormatoAttachment>
            <Attachment t-esc="pdf"/>
        </Allegati>
    </FatturaElettronicaBody>
</p:FatturaElettronica>
</template>

<template id="account_invoice_it_FatturaPA_sede">
            <Sede>
                <Indirizzo><t t-if="partner.street or partner.street2" t-esc="format_alphanumeric((partner.street or '') + ' ' + (partner.street2 or ''))[:60]"/></Indirizzo>
                <CAP><t t-if="partner.country_id.code != 'IT'" t-esc="'00000'"/><t t-elif="partner.zip" t-esc="partner.zip"/></CAP>
                <Comune t-esc="format_alphanumeric(partner.city[:60])"/>
                <Provincia t-if="partner.country_id.code == 'IT' and partner.state_id" t-esc="partner.state_id.code[:2]"/>
                <Nazione t-esc="partner.country_id.code"/>
            </Sede>
</template>

<template id="account_invoice_FatturaPA_origin_document">
                <IdDocumento t-if="origin_document_name" t-esc="format_alphanumeric(origin_document_name[:20])"/>
                <Data t-if="origin_document_date" t-esc="format_date(origin_document_date)"/>
                <CodiceCUP t-if="cup" t-esc="format_alphanumeric(cup)[-15:]"/>
                <CodiceCIG t-if="cig" t-esc="format_alphanumeric(cig)[-15:]"/>
</template>

    </data>
</odoo>

```

## File: data\ir_cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_receive_fattura_pa_invoice" model="ir.cron">
        <field name="name">FatturaPA: Receive invoices from the exchange system</field>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="model_id" ref="account_edi.model_account_edi_format"/>
        <field name="code">model._cron_receive_fattura_pa()</field>
        <field name="doall" eval="False"/>
        <field name="state">code</field>
    </record>
</odoo>

```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def _load(self, company):
        """
            Override normal default taxes, which are the ones with lowest sequence.
        """
        result = super()._load(company)
        template = company.chart_template_id
        if template == self.env.ref('l10n_it.l10n_it_chart_template_generic'):
            company.account_sale_tax_id = self.env.ref(f'l10n_it.{company.id}_22v')
            company.account_purchase_tax_id = self.env.ref(f'l10n_it.{company.id}_22am')
            tax = self.env.ref(f'l10n_it.{company.id}_00eu', raise_if_not_found=False)
            if tax:
                tax.write({
                    'l10n_it_has_exoneration': True,
                    'l10n_it_kind_exoneration': 'N3.2',
                    'l10n_it_law_reference': 'Art. 41, DL n. 331/93',
                })
        return result

```

## File: models\account_edi_format.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import Command, api, models, fields, _, _lt
from odoo.exceptions import UserError
from odoo.addons.account_edi_proxy_client.models.account_edi_proxy_user import AccountEdiProxyError
from odoo.addons.l10n_it_edi.tools.remove_signature import remove_signature
from odoo.osv.expression import OR, AND

from lxml import etree
from datetime import datetime
import re
import logging
import base64


_logger = logging.getLogger(__name__)

DEFAULT_FACTUR_ITALIAN_DATE_FORMAT = '%Y-%m-%d'


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    # -------------------------------------------------------------------------
    # Helpers
    # -------------------------------------------------------------------------

    @api.model
    def _l10n_it_edi_generate_electronic_invoice_filename(self, invoice):
        '''Returns a name conform to the Fattura pa Specifications:
           See ES documentation 2.2
        '''
        a = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
        # Each company should have its own filename sequence. If it does not exist, create it
        n = self.env['ir.sequence'].with_company(invoice.company_id).next_by_code('l10n_it_edi.fattura_filename')
        if not n:
            # The offset is used to avoid conflicts with existing filenames
            offset = 62 ** 4
            sequence = self.env['ir.sequence'].sudo().create({
                'name': 'FatturaPA Filename Sequence',
                'code': 'l10n_it_edi.fattura_filename',
                'company_id': invoice.company_id.id,
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
            'country_code': invoice.company_id.country_id.code,
            'codice': self.env['res.partner']._l10n_it_normalize_codice_fiscale(invoice.company_id.l10n_it_codice_fiscale),
            'progressive_number': progressive_number.zfill(5),
        }

    def _l10n_it_edi_check_invoice_configuration(self, invoice):
        errors = self._l10n_it_edi_check_ordinary_invoice_configuration(invoice)

        if not errors:
            errors = self._l10n_it_edi_check_simplified_invoice_configuration(invoice)

        return errors

    def _l10n_it_edi_is_self_invoice(self, invoice):
        """
            Italian EDI requires Vendor bills coming from EU countries to be sent as self-invoices.
            We recognize these cases based on the taxes that target the VJ tax grids, which imply
            the use of VAT External Reverse Charge.
        """
        if not invoice.is_purchase_document():
            return False

        invoice_lines_tags = invoice.line_ids.tax_tag_ids
        it_tax_report_vj_lines = self.env['account.report.line'].search([
            ('report_id.country_id.code', '=', 'IT'),
            ('code', 'like', 'VJ%'),
        ])
        vj_lines_tags = it_tax_report_vj_lines.expression_ids._get_matching_tags()
        return bool(invoice_lines_tags & vj_lines_tags)

    def _l10n_it_edi_check_ordinary_invoice_configuration(self, invoice):
        errors = []
        seller = invoice.company_id
        buyer = invoice.commercial_partner_id
        is_self_invoice = self._l10n_it_edi_is_self_invoice(invoice)
        if is_self_invoice:
            seller, buyer = buyer, seller

        # <1.1.1.1>
        if not seller.country_id:
            errors.append(_("%s must have a country", seller.display_name))

        # <1.1.1.2>
        if not invoice.company_id.vat:
            errors.append(_("%s must have a VAT number", seller.display_name))
        if seller.vat and len(seller.vat) > 30:
            errors.append(_("The maximum length for VAT number is 30. %s have a VAT number too long: %s.", seller.display_name, seller.vat))

        # <1.2.1.2>
        if not is_self_invoice and not seller.l10n_it_codice_fiscale:
            errors.append(_("%s must have a codice fiscale number", seller.display_name))

        # <1.2.1.8>
        if not is_self_invoice and not seller.l10n_it_tax_system:
            errors.append(_("The seller's company must have a tax system."))

        # <1.2.2>
        if not seller.street and not seller.street2:
            errors.append(_("%s must have a street.", seller.display_name))
        if not seller.zip:
            errors.append(_("%s must have a post code.", seller.display_name))
        elif len(seller.zip) != 5 and seller.country_id.code == 'IT':
            errors.append(_("%s must have a post code of length 5.", seller.display_name))
        if not seller.city:
            errors.append(_("%s must have a city.", seller.display_name))
        if not seller.country_id:
            errors.append(_("%s must have a country.", seller.display_name))

        if not is_self_invoice and seller.l10n_it_has_tax_representative and not seller.l10n_it_tax_representative_partner_id.vat:
            errors.append(_("Tax representative partner %s of %s must have a tax number.", seller.l10n_it_tax_representative_partner_id.display_name, seller.display_name))

        # <1.4.1>
        if not buyer.vat and not buyer.l10n_it_codice_fiscale and buyer.country_id.code == 'IT':
            errors.append(_("The buyer, %s, or his company must have a VAT number and/or a tax code (Codice Fiscale).", buyer.display_name))

        if is_self_invoice and self._l10n_it_edi_services_or_goods(invoice) == 'both':
            errors.append(_("Cannot apply Reverse Charge to a bill which contains both services and goods."))

        if is_self_invoice and not buyer.partner_id.l10n_it_pa_index:
            errors.append(_("Vendor bills sent as self-invoices to the SdI require a valid PA Index (Codice Destinatario) on the company's contact."))

        for tax_line in invoice.line_ids.filtered(lambda line: line.tax_line_id):
            if not tax_line.tax_line_id.l10n_it_kind_exoneration and tax_line.tax_line_id.amount == 0:
                errors.append(_("%s has an amount of 0.0, you must indicate the kind of exoneration.", tax_line.name))

        errors += self._l10n_it_edi_check_taxes_configuration(invoice)

        return errors

    def _l10n_it_edi_check_taxes_configuration(self, invoice):
        """
            Can be overridden by submodules like l10n_it_edi_withholding, which also allows for withholding and pension_fund taxes.
        """
        errors = []
        for invoice_line in invoice.invoice_line_ids.filtered(lambda x: x.display_type == 'product'):
            all_taxes = invoice_line.tax_ids.flatten_taxes_hierarchy()
            vat_taxes = all_taxes.filtered(lambda t: t.amount_type == 'percent' and t.amount >= 0)
            if len(vat_taxes) != 1:
                errors.append(_("In line %s, you must select one and only one VAT tax.", invoice_line.name))
        return errors

    def _l10n_it_edi_is_simplified(self, invoice):
        """
            Simplified Invoices are a way for the invoice issuer to create an invoice with limited data.
            Example: a consultant goes to the restaurant and wants the invoice instead of the receipt,
            to be able to deduct the expense from his Taxes. The Italian State allows the restaurant
            to issue a Simplified Invoice with the VAT number only, to speed up times, instead of
            requiring the address and other informations about the buyer.
            Only invoices under the threshold of 400 Euroes are allowed, to avoid this tool
            be abused for bigger transactions, that would enable less transparency to tax institutions.
        """
        buyer = invoice.commercial_partner_id
        return all([
            self.env.ref('l10n_it_edi.account_invoice_it_simplified_FatturaPA_export', raise_if_not_found=False),
            not self._l10n_it_edi_is_self_invoice(invoice),
            self._l10n_it_edi_check_buyer_invoice_configuration(invoice),
            not buyer.country_id or buyer.country_id.code == 'IT',
            buyer.l10n_it_codice_fiscale or (buyer.vat and (buyer.vat[:2].upper() == 'IT' or buyer.vat[:2].isdecimal())),
            invoice.amount_total <= 400,
        ])

    def _l10n_it_edi_check_simplified_invoice_configuration(self, invoice):
        return [] if self._l10n_it_edi_is_simplified(invoice) else self._l10n_it_edi_check_buyer_invoice_configuration(invoice)

    def _l10n_it_edi_partner_in_eu(self, partner):
        europe = self.env.ref('base.europe', raise_if_not_found=False)
        country = partner.country_id
        return not europe or not country or country in europe.country_ids

    def _l10n_it_edi_services_or_goods(self, invoice):
        """
            Services and goods have different tax grids when VAT is Reverse Charged, and they can't
            be mixed in the same invoice, because the TipoDocumento depends on which which kind
            of product is bought and it's unambiguous.
        """
        scopes = []
        for line in invoice.invoice_line_ids.filtered(lambda l: l.display_type not in ('line_note', 'line_section')):
            tax_ids_with_tax_scope = line.tax_ids.filtered(lambda x: x.tax_scope)
            if tax_ids_with_tax_scope:
                scopes += tax_ids_with_tax_scope.mapped('tax_scope')
            else:
                scopes.append(line.product_id and line.product_id.type or 'consu')

        if set(scopes) == set(['consu', 'service']):
            return "both"
        return scopes and scopes.pop()

    def _l10n_it_edi_check_buyer_invoice_configuration(self, invoice):
        errors = []
        buyer = invoice.commercial_partner_id

        # <1.4.2>
        if not buyer.street and not buyer.street2:
            errors.append(_("%s must have a street.", buyer.display_name))
        if not buyer.country_id:
            errors.append(_("%s must have a country.", buyer.display_name))
        if not buyer.zip:
            errors.append(_("%s must have a post code.", buyer.display_name))
        elif len(buyer.zip) != 5 and buyer.country_id.code == 'IT':
            errors.append(_("%s must have a post code of length 5.", buyer.display_name))
        if not buyer.city:
            errors.append(_("%s must have a city.", buyer.display_name))

        for tax_line in invoice.line_ids.filtered(lambda line: line.tax_line_id):
            if not tax_line.tax_line_id.l10n_it_kind_exoneration and tax_line.tax_line_id.amount == 0:
                errors.append(_("%s has an amount of 0.0, you must indicate the kind of exoneration.", tax_line.name))

        return errors

    def _l10n_it_goods_in_italy(self, invoice):
        """
            There is a specific TipoDocumento (Document Type TD19) and tax grid (VJ3) for goods
            that are phisically in Italy but are in a VAT deposit, meaning that the goods
            have not passed customs.
        """
        invoice_lines_tags = invoice.line_ids.tax_tag_ids
        it_tax_report_vj3_lines = self.env['account.report.line'].search([
            ('report_id.country_id.code', '=', 'IT'),
            ('code', '=', 'VJ3'),
        ])
        vj3_lines_tags = it_tax_report_vj3_lines.expression_ids._get_matching_tags()
        return bool(invoice_lines_tags & vj3_lines_tags)

    def _l10n_it_document_type_mapping(self):
        """ Returns a dictionary with the required features for every TDxx FatturaPA document type """
        return {
            'TD01': dict(move_types=['out_invoice'], import_type='in_invoice', self_invoice=False, simplified=False, downpayment=False),
            'TD02': dict(move_types=['out_invoice'], import_type='in_invoice', self_invoice=False, simplified=False, downpayment=True),
            'TD04': dict(move_types=['out_refund'], import_type='in_refund', self_invoice=False, simplified=False),
            'TD07': dict(move_types=['out_invoice'], import_type='in_invoice', self_invoice=False, simplified=True),
            'TD08': dict(move_types=['out_refund'], import_type='in_refund', self_invoice=False, simplified=True),
            'TD09': dict(move_types=['out_invoice'], import_type='in_invoice', self_invoice=False, simplified=True),
            'TD28': dict(move_types=['in_invoice', 'in_refund'], import_type='in_invoice', simplified=False, self_invoice=True, partner_country_code="SM"),
            'TD17': dict(move_types=['in_invoice', 'in_refund'], import_type='in_invoice', simplified=False, self_invoice=True, services_or_goods="service"),
            'TD18': dict(move_types=['in_invoice', 'in_refund'], import_type='in_invoice', simplified=False, self_invoice=True, services_or_goods="consu", goods_in_italy=False, partner_in_eu=True),
            'TD19': dict(move_types=['in_invoice', 'in_refund'], import_type='in_invoice', simplified=False, self_invoice=True, services_or_goods="consu", goods_in_italy=True),
        }

    @api.model
    def _l10n_it_buyer_seller_info(self):
        return {
            'buyer': {
                'role': 'buyer',
                'section_xpath': './/CessionarioCommittente',
                'vat_xpath': '//CessionarioCommittente//IdCodice',
                'codice_fiscale_xpath': '//CessionarioCommittente//CodiceFiscale',
                'type_tax_use_domain': [('type_tax_use', '=', 'purchase')],
            },
            'seller': {
                'role': 'seller',
                'section_xpath': './/CedentePrestatore',
                'vat_xpath': '//CedentePrestatore//IdCodice',
                'codice_fiscale_xpath': '//CedentePrestatore//CodiceFiscale',
                'type_tax_use_domain': [('type_tax_use', '=', 'sale')],
            },
        }

    def _l10n_it_get_invoice_features_for_document_type_selection(self, invoice):
        """ Returns a dictionary of features to be compared with the TDxx FatturaPA
            document type requirements. """
        services_or_goods = self._l10n_it_edi_services_or_goods(invoice)
        return {
            'move_types': invoice.move_type,
            'partner_in_eu': self._l10n_it_edi_partner_in_eu(invoice.commercial_partner_id),
            'partner_country_code': invoice.commercial_partner_id.country_id.code,
            'simplified': self._l10n_it_edi_is_simplified(invoice),
            'self_invoice': self._l10n_it_edi_is_self_invoice(invoice),
            'downpayment': invoice._is_downpayment(),
            'services_or_goods': services_or_goods,
            'goods_in_italy': services_or_goods == 'consu' and self._l10n_it_goods_in_italy(invoice),
        }

    def _l10n_it_get_document_type(self, invoice):
        """ Compare the features of the invoice to the requirements of each TDxx FatturaPA
            document type until you find a valid one. """
        invoice_features = self._l10n_it_get_invoice_features_for_document_type_selection(invoice)
        for code, document_type_features in self._l10n_it_document_type_mapping().items():
            comparisons = []
            for key, invoice_feature in invoice_features.items():
                if key not in document_type_features:
                    continue
                document_type_feature = document_type_features.get(key)
                if isinstance(document_type_feature, (tuple, list)):
                    comparisons.append(invoice_feature in document_type_feature)
                else:
                    comparisons.append(invoice_feature == document_type_feature)
            if all(comparisons):
                return code
        return False

    def _l10n_it_is_simplified_document_type(self, document_type):
        return self._l10n_it_document_type_mapping().get(document_type, {}).get('simplified', False)

    # -------------------------------------------------------------------------
    # Import
    # -------------------------------------------------------------------------

    def _cron_receive_fattura_pa(self):
        ''' Check the proxy for incoming invoices for all companies.
        '''
        if self.env['account_edi_proxy_client.user']._get_demo_state() == 'demo':
            return

        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        for proxy_user in self.env['account_edi_proxy_client.user'].search([('edi_format_code', '=', 'fattura_pa')]):
            fattura_pa._receive_fattura_pa(proxy_user)

    def _receive_fattura_pa(self, proxy_user):
        ''' Check the proxy for incoming invoices for a specified proxy user.
        '''
        try:
            res = proxy_user._make_request(
                proxy_user._get_server_url() + '/api/l10n_it_edi/1/in/RicezioneInvoice',
                params={'recipient_codice_fiscale': proxy_user.company_id.l10n_it_codice_fiscale})
        except AccountEdiProxyError as e:
            res = {}
            _logger.warning('Error while receiving file from SdiCoop: %s', e)

        retrigger = False
        proxy_acks = []
        for id_transaction, fattura in res.items():

            # The server has a maximum number of documents it can send at a time
            # If that maximum is reached, then we search for more
            # by re-triggering the download cron, avoiding the timeout.
            current_num, max_num = fattura.get('current_num', 0), fattura.get('max_num', 0)
            retrigger = retrigger or current_num == max_num > 0

            if self._save_incoming_attachment_fattura_pa(proxy_user, id_transaction, fattura['filename'], fattura['file'], fattura['key']):
                proxy_acks.append(id_transaction)

        if proxy_acks:
            try:
                proxy_user._make_request(
                    proxy_user._get_server_url() + '/api/l10n_it_edi/1/ack',
                    params={'transaction_ids': proxy_acks})
            except AccountEdiProxyError as e:
                _logger.warning('Error while receiving file from SdiCoop: %s', e)

        if retrigger:
            _logger.info('Retriggering "Receive invoices from the exchange system"...')
            self.env.ref('l10n_it_edi.ir_cron_receive_fattura_pa_invoice')._trigger()

    def _save_incoming_attachment_fattura_pa(self, proxy_user, id_transaction, filename, content, key):
        ''' Save an incoming file from the SdI as an attachment.

            :param proxy_user:     the user that saves the attachment.
            :param id_transaction: id of the SdI transaction for communication with the IAP proxy.
            :param filename:       name of the file to be saved.
            :param content:        encrypted content of the file to be saved.
            :param key:            key to decrypt the file.
            :returns:              True if everything went well, or the file already exists.
                                   False if the file cannot be parsed as an XML.
        '''

        company = proxy_user.company_id
        # Name should be unique per company, the invoice already exists
        Attachment = self.env['ir.attachment'].sudo().with_company(company)
        if Attachment.search_count([
            ('name', '=', filename),
            ('res_model', '=', 'account.move'),
            ('company_id', '=', proxy_user.company_id.id),
        ], limit=1):
            # name should be unique, the invoice already exists
            _logger.info('E-invoice already exists: %s', filename)
            return True

        raw_content = proxy_user._decrypt_data(content, key)
        invoice = self.env['account.move'].with_company(company).create({'move_type': 'in_invoice'})
        attachment = Attachment.create({
            'name': filename,
            'raw': raw_content,
            'type': 'binary',
            'res_model': 'account.move',
            'res_id': invoice.id
        })

        # In case something fails after, we still have the attachment
        # So that we don't delete the attachment when deleting the invoice
        self.env.cr.commit()

        # Detach the attachment and unlink the stub invoice.
        attachment.res_id = False
        attachment.res_model = False
        invoice.unlink()

        # Import the invoice from the attachment and reattach.
        invoice = self.with_company(company)._create_document_from_attachment(attachment)
        attachment.write({'res_model': 'account.move', 'res_id': invoice.id})
        self.env.cr.commit()

        return True

    def _check_filename_is_fattura_pa(self, filename):
        return re.search("[A-Z]{2}[A-Za-z0-9]{2,28}_[A-Za-z0-9]{0,5}.((?i:xml.p7m|xml))", filename)

    def _is_fattura_pa(self, filename, tree=None):
        return self.code == 'fattura_pa' and self._check_filename_is_fattura_pa(filename)

    def _create_invoice_from_xml_tree(self, filename, tree, journal=None):
        self.ensure_one()
        if self._is_fattura_pa(filename, tree):
            return self._import_fattura_pa(tree, self.env['account.move'])
        return super()._create_invoice_from_xml_tree(filename, tree, journal=journal)

    def _update_invoice_from_xml_tree(self, filename, tree, invoice):
        self.ensure_one()
        if self._is_fattura_pa(filename, tree):
            if len(tree.xpath('//FatturaElettronicaBody')) > 1:
                invoice.message_post(body='The attachment contains multiple invoices, this invoice was not updated from it.',
                                     message_type='comment',
                                     subtype_xmlid='mail.mt_note',
                                     author_id=self.env.ref('base.partner_root').id)
            else:
                return self._import_fattura_pa(tree, invoice)
        return super()._update_invoice_from_xml_tree(filename, tree, invoice)

    def _decode_p7m_to_xml(self, filename, content):
        def parse_xml(parser, filename, content):
            try:
                return etree.fromstring(content, parser)
            except (etree.ParseError, ValueError) as e:
                _logger.info("XML parsing of %s failed: %s", filename, e)

        parser = etree.XMLParser(recover=True, resolve_entities=False)
        xml_tree = parse_xml(parser, filename, content)
        if xml_tree is None:
            # The file may have a Cades signature, trying to remove it
            xml_tree = parse_xml(parser, filename, remove_signature(content))
            if xml_tree is None:
                _logger.info("Italian EDI invoice file %s cannot be decoded.", filename)
                return None

        return xml_tree

    def _create_invoice_from_binary(self, filename, content, extension):
        self.ensure_one()
        if extension.lower() == '.xml.p7m' and self._is_fattura_pa(filename):
            decoded_content = self._decode_p7m_to_xml(filename, content)
            if decoded_content is not None:
                return self._import_fattura_pa(decoded_content, self.env['account.move'])
        return super()._create_invoice_from_binary(filename, content, extension)

    def _update_invoice_from_binary(self, filename, content, extension, invoice):
        self.ensure_one()
        if extension.lower() == '.xml.p7m' and self._is_fattura_pa(filename):
            decoded_content = self._decode_p7m_to_xml(filename, content)
            if decoded_content is not None:
                return self._import_fattura_pa(decoded_content, invoice)
        return super()._update_invoice_from_binary(filename, content, extension, invoice)

    def _l10n_it_get_partner_invoice(self, tree, company, partner_info=None):
        if partner_info is None:
            partner_info = self._l10n_it_buyer_seller_info()['seller']
        elements = tree.xpath(partner_info['vat_xpath'])
        partner = elements and self.env['res.partner'].search(
            ['&', ('vat', 'ilike', elements[0].text), '|', ('company_id', '=', company.id), ('company_id', '=', False)],
            limit=1)
        if not partner:
            elements = tree.xpath(partner_info['codice_fiscale_xpath'])
            if elements:
                codice = elements[0].text
                domains = [[('l10n_it_codice_fiscale', '=', codice)]]
                if re.match(r'^[0-9]{11}$', codice):
                    domains.append([('l10n_it_codice_fiscale', '=', 'IT' + codice)])
                elif re.match(r'^IT[0-9]{11}$', codice):
                    domains.append([('l10n_it_codice_fiscale', '=',
                                     self.env['res.partner']._l10n_it_normalize_codice_fiscale(codice))])
                partner = elements and self.env['res.partner'].search(
                    AND([OR(domains), OR([[('company_id', '=', company.id)], [('company_id', '=', False)]])]), limit=1)
        if not partner and partner_info['role'] == 'seller':
            elements = tree.xpath('//DatiTrasmissione//Email')
            partner = elements and self.env['res.partner'].search(
                ['&', '|', ('email', '=', elements[0].text), ('l10n_it_pec_email', '=', elements[0].text), '|',
                 ('company_id', '=', company.id), ('company_id', '=', False)], limit=1)

        return partner

    def _convert_date_from_xml(self, xsdate_str):
        """ Dates in FatturaPA are ISO 8601 date format, pattern '[-]CCYY-MM-DD[Z|(+|-)hh:mm]' """
        xsdate_str = xsdate_str.strip()
        xsdate_pattern = r"^-?(?P<date>-?\d{4}-\d{2}-\d{2})(?P<tz>[zZ]|[+-]\d{2}:\d{2})?$"
        try:
            match = re.match(xsdate_pattern, xsdate_str)
            converted_date = datetime.strptime(match.group("date"), DEFAULT_FACTUR_ITALIAN_DATE_FORMAT).date()
        except Exception:
            converted_date = False
        return converted_date

    def _l10n_it_edi_search_tax_for_import(self, company, percentage, extra_domain=None):
        """ Returns the VAT, Withholding or Pension Fund tax that suits the conditions given
            and matches the percentage found in the XML for the company. """
        conditions = [
            ('company_id', '=', company.id),
            ('amount', '=', percentage),
            ('amount_type', '=', 'percent'),
        ] + (extra_domain or [])

        # As we're importing vendor bills, we're excluding Reverse Charge Taxes
        # which have a [100.0, 100.0, -100.0] repartition lines factor_percent distribution.
        # We only allow for taxes that have all positive repartition lines factor_percent distribution.
        taxes = self.env['account.tax'].search(conditions).filtered(
            lambda tax: all([rep_line.factor_percent >= 0 for rep_line in tax.invoice_repartition_line_ids]))

        return taxes[0] if taxes else taxes

    def _l10n_it_edi_get_extra_info(self, company, document_type, body_tree, incoming=True):
        """ This function is meant to collect other information that has to be inserted on the invoice lines by submodules.
            :return extra_info, messages_to_log"""
        return {
            'simplified': self._l10n_it_is_simplified_document_type(document_type),
            'type_tax_use_domain': [('type_tax_use', '=', 'purchase' if incoming else 'sale')],
        }, []

    def _import_fattura_pa(self, tree, invoice):
        """ Decodes a fattura_pa invoice into an invoice.

        :param tree:    the fattura_pa tree to decode.
        :param invoice: the invoice to update or an empty recordset.
        :returns:       the invoice where the fattura_pa data was imported.
        """
        invoices = self.env['account.move']
        first_run = True

        buyer_seller_info = self._l10n_it_buyer_seller_info()

        # possible to have multiple invoices in the case of an invoice batch, the batch itself is repeated for every invoice of the batch
        for body_tree in tree.xpath('//FatturaElettronicaBody'):
            if not first_run or not invoice:
                # make sure all the iterations create a new invoice record (except the first which could have already created one)
                invoice = self.env['account.move']
            first_run = False

            # There are 2 cases:
            # - cron:
            #     * Move direction (incoming / outgoing) flexible (no 'default_move_type')
            #     * All companies are possible (no 'allowed_company_ids')
            #     * I.e. used for import from tax agency
            # - "Upload" button (invoices / bills view)
            #     * Fixed move direction; the button sets the 'default_move_type'
            #     * Companies are restricted to 'allowed_company_ids'.

            default_move_type = self.env.context.get('default_move_type')
            if default_move_type is None:
                incoming_possibilities = [True, False]
            elif default_move_type in invoice.get_purchase_types(include_receipts=True):
                incoming_possibilities = [True]
            elif default_move_type in invoice.get_sale_types(include_receipts=True):
                incoming_possibilities = [False]
            else:
                _logger.warning("Cannot handle default_move_type '%s'.", default_move_type)
                continue

            allowed_company_ids = self.env.context.get('allowed_company_ids')
            allowed_company_domain = [('id', 'in', allowed_company_ids)] if allowed_company_ids else []

            for incoming in incoming_possibilities:
                company_role, partner_role = ('buyer', 'seller') if incoming else ('seller', 'buyer')
                company_info = buyer_seller_info[company_role]
                elements = tree.xpath(company_info['vat_xpath'])
                company = elements and self.env['res.company'].search([
                    ('vat', 'ilike', elements[0].text),
                    *allowed_company_domain
                ], limit=1)
                if not company:
                    elements = tree.xpath(company_info['codice_fiscale_xpath'])
                    company = elements and self.env['res.company'].search([
                        ('l10n_it_codice_fiscale', 'ilike', elements[0].text),
                        *allowed_company_domain
                    ], limit=1)
                if company:
                    break
            else:
                _logger.warning('Could not determine company (by looking at the VAT and codice fiscale in the buyer and/or seller section).')
                continue

            # Refund type.
            # TD01 == invoice
            # TD02 == advance/down payment on invoice
            # TD03 == advance/down payment on fee
            # TD04 == credit note
            # TD05 == debit note
            # TD06 == fee
            # TD07 == simplified invoice
            # TD08 == simplified credit note
            # TD09 == simplified debit note
            # For unsupported document types, just assume in_invoice, and log that the type is unsupported
            elements = tree.xpath('//DatiGeneraliDocumento/TipoDocumento')
            document_type = elements[0].text if elements else ''
            move_type = self._l10n_it_document_type_mapping().get(document_type, {}).get('import_type', False)
            if not move_type:
                move_type = "in_invoice"
                _logger.info('Document type not managed: %s. Invoice type is set by default.', document_type)
            if not incoming and move_type.startswith('in_'):
                move_type = 'out' + move_type[2:]

            invoice_ctx = invoice.with_company(company) \
                                 .with_context(
                                    default_move_type=move_type,
                                    account_predictive_bills_predict_product=False,
                                    account_predictive_bills_predict_taxes=False
                                )

            # move could be a single record (editing) or be empty (new).
            with invoice_ctx._get_edi_creation() as invoice_form:

                # Collect extra info from the XML that may be used by submodules to further put information on the invoice lines
                extra_info, message_to_log = self._l10n_it_edi_get_extra_info(company, document_type, body_tree, incoming=incoming)

                partner_info = buyer_seller_info[partner_role]
                partner = self._l10n_it_get_partner_invoice(tree, company, partner_info)
                if partner:
                    invoice_form.partner_id = partner
                else:
                    message_to_log.append("%s<br/>%s" % (
                        _("Partner not found, useful informations from XML file:"),
                        invoice._compose_info_message(
                            tree, partner_info['section_xpath'])))

                # Numbering attributed by the transmitter. <1.1.2>
                elements = tree.xpath('//ProgressivoInvio')
                if elements:
                    invoice_form.payment_reference = elements[0].text

                elements = body_tree.xpath('.//DatiGeneraliDocumento//Numero')
                if elements:
                    invoice_form.ref = elements[0].text

                # Currency. <2.1.1.2>
                elements = body_tree.xpath('.//DatiGeneraliDocumento/Divisa')
                if elements:
                    currency_str = elements[0].text
                    currency = self.env.ref('base.%s' % currency_str.upper(), raise_if_not_found=False)
                    if currency != self.env.company.currency_id and currency.active:
                        invoice_form.currency_id = currency

                # Date. <2.1.1.3>
                elements = body_tree.xpath('.//DatiGeneraliDocumento/Data')
                if elements:
                    document_date = self._convert_date_from_xml(elements[0].text)
                    if document_date:
                        invoice_form.invoice_date = document_date
                    else:
                        message_to_log.append("%s<br/>%s" % (
                            _("Document date invalid in XML file:"),
                            invoice._compose_info_message(elements[0], '.')
                        ))

                #  Dati Bollo. <2.1.1.6>
                elements = body_tree.xpath('.//DatiGeneraliDocumento/DatiBollo/ImportoBollo')
                if elements:
                    invoice_form.l10n_it_stamp_duty = float(elements[0].text)


                # Comment. <2.1.1.11>
                elements = body_tree.xpath('.//DatiGeneraliDocumento//Causale')
                for element in elements:
                    invoice_form.narration = '%s%s<br/>' % (invoice_form.narration or '', element.text)

                # Informations relative to the purchase order, the contract, the agreement,
                # the reception phase or invoices previously transmitted
                # <2.1.2> - <2.1.6>
                for document_type in ['DatiOrdineAcquisto', 'DatiContratto', 'DatiConvenzione', 'DatiRicezione', 'DatiFattureCollegate']:
                    elements = body_tree.xpath('.//DatiGenerali/' + document_type)
                    if elements:
                        for element in elements:
                            message_to_log.append("%s %s<br/>%s" % (document_type, _("from XML file:"),
                            invoice._compose_info_message(element, '.')))

                #  Dati DDT. <2.1.8>
                elements = body_tree.xpath('.//DatiGenerali/DatiDDT')
                if elements:
                    message_to_log.append("%s<br/>%s" % (
                        _("Transport informations from XML file:"),
                        invoice._compose_info_message(body_tree, './/DatiGenerali/DatiDDT')))

                # Due date. <2.4.2.5>
                elements = body_tree.xpath('.//DatiPagamento/DettaglioPagamento/DataScadenzaPagamento')
                if elements:
                    date_str = elements[0].text.strip()
                    if date_str:
                        due_date = self._convert_date_from_xml(date_str)
                        if due_date:
                            invoice_form.invoice_date_due = fields.Date.to_string(due_date)
                        else:
                            message_to_log.append("%s<br/>%s" % (
                                _("Payment due date invalid in XML file:"),
                                invoice._compose_info_message(elements[0], '.')
                            ))

                # Information related to the purchase order <2.1.2>
                po_refs = []
                elements = body_tree.xpath('//DatiGenerali/DatiOrdineAcquisto/IdDocumento')
                if elements:
                    po_refs = [element.text.strip() for element in elements]
                    invoice_form.invoice_origin = ", ".join(po_refs)

                # Total amount. <2.4.2.6>
                elements = body_tree.xpath('.//ImportoPagamento')
                amount_total_import = 0
                for element in elements:
                    amount_total_import += float(element.text)
                if amount_total_import:
                    message_to_log.append(_("Total amount from the XML File: %s") % (
                        amount_total_import))

                # Bank account. <2.4.2.13>
                if invoice_form.move_type not in ('out_invoice', 'in_refund'):
                    elements = body_tree.xpath('.//DatiPagamento/DettaglioPagamento/IBAN')
                    if elements:
                        if invoice_form.partner_id and invoice_form.partner_id.commercial_partner_id:
                            bank = self.env['res.partner.bank'].search([
                                ('acc_number', '=', elements[0].text),
                                ('partner_id', '=', invoice_form.partner_id.commercial_partner_id.id),
                                ('company_id', 'in', [invoice_form.company_id.id, False])
                            ], order='company_id', limit=1)
                        else:
                            bank = self.env['res.partner.bank'].search([
                                ('acc_number', '=', elements[0].text), ('company_id', 'in', [invoice_form.company_id.id, False])
                            ], order='company_id', limit=1)
                        if bank:
                            invoice_form.partner_bank_id = bank
                        else:
                            message_to_log.append("%s<br/>%s" % (
                                _("Bank account not found, useful informations from XML file:"),
                                invoice._compose_multi_info_message(
                                    body_tree, ['.//DatiPagamento//Beneficiario',
                                        './/DatiPagamento//IstitutoFinanziario',
                                        './/DatiPagamento//IBAN',
                                        './/DatiPagamento//ABI',
                                        './/DatiPagamento//CAB',
                                        './/DatiPagamento//BIC',
                                        './/DatiPagamento//ModalitaPagamento'])))
                else:
                    elements = body_tree.xpath('.//DatiPagamento/DettaglioPagamento')
                    if elements:
                        message_to_log.append("%s<br/>%s" % (
                            _("Bank account not found, useful informations from XML file:"),
                            invoice._compose_info_message(body_tree, './/DatiPagamento')))

                # Invoice lines. <2.2.1>
                if not extra_info['simplified']:
                    elements = body_tree.xpath('.//DettaglioLinee')
                else:
                    elements = body_tree.xpath('.//DatiBeniServizi')

                for element in (elements or []):
                    invoice_line_form = invoice_form.invoice_line_ids.create({
                        'move_id': invoice_form.id,
                        'tax_ids': [fields.Command.clear()],
                    })
                    if invoice_line_form:
                        message_to_log += self._import_fattura_pa_line(element, invoice_line_form, extra_info)

                # Global discount summarized in 1 amount
                discount_elements = body_tree.xpath('.//DatiGeneraliDocumento/ScontoMaggiorazione')
                if discount_elements:
                    taxable_amount = float(invoice_form.tax_totals['amount_untaxed'])
                    discounted_amount = taxable_amount
                    for discount_element in discount_elements:
                        discount_type = discount_element.xpath('.//Tipo')
                        discount_sign = 1
                        if discount_type and discount_type[0].text == 'MG':
                            discount_sign = -1
                        discount_amount = discount_element.xpath('.//Importo')
                        if discount_amount:
                            discounted_amount -= discount_sign * float(discount_amount[0].text)
                            continue
                        discount_percentage = discount_element.xpath('.//Percentuale')
                        if discount_percentage:
                            discounted_amount *= 1 - discount_sign * float(discount_percentage[0].text) / 100

                    general_discount = discounted_amount - taxable_amount
                    sequence = len(elements) + 1

                    invoice_form.invoice_line_ids = [Command.create({
                        'sequence': sequence,
                        'name': 'SCONTO' if general_discount < 0 else 'MAGGIORAZIONE',
                        'price_unit': general_discount,
                        'tax_ids': [],  # without this, a tax is automatically added to the line
                    })]

            new_invoice = invoice_form

            elements = body_tree.xpath('.//Allegati')
            if elements:
                for element in elements:
                    name_attachment = element.xpath('.//NomeAttachment')[0].text
                    attachment_64 = str.encode(element.xpath('.//Attachment')[0].text)
                    attachment_64 = self.env['ir.attachment'].create({
                        'name': name_attachment,
                        'datas': attachment_64,
                        'type': 'binary',
                        'res_model': 'account.move',
                        'res_id': new_invoice.id,
                    })

                    # no_new_invoice to prevent from looping on the message_post that would create a new invoice without it
                    new_invoice.with_context(no_new_invoice=True).message_post(
                        body=(_("Attachment from XML")),
                        attachment_ids=[attachment_64.id]
                    )

            for message in message_to_log:
                new_invoice.message_post(body=message)

            invoices += new_invoice

        return invoices

    def _import_fattura_pa_line(self, element, invoice_line_form, extra_info=None):
        extra_info = extra_info or {}
        company = invoice_line_form.company_id
        partner = invoice_line_form.partner_id
        message_to_log = []

        # Sequence.
        line_elements = element.xpath('.//NumeroLinea')
        if line_elements:
            invoice_line_form.sequence = int(line_elements[0].text)

        # Product.
        elements_code = element.xpath('.//CodiceArticolo')
        if elements_code:
            for element_code in elements_code:
                type_code = element_code.xpath('.//CodiceTipo')[0]
                code = element_code.xpath('.//CodiceValore')[0]
                if type_code.text == 'EAN':
                    product = self.env['product.product'].search([('barcode', '=', code.text)])
                    if product:
                        invoice_line_form.product_id = product
                        break
                if partner:
                    product_supplier = self.env['product.supplierinfo'].search([('partner_id', '=', partner.id), ('product_code', '=', code.text)], limit=2)
                    if product_supplier and len(product_supplier) == 1 and product_supplier.product_id:
                        invoice_line_form.product_id = product_supplier.product_id
                        break
            if not invoice_line_form.product_id:
                for element_code in elements_code:
                    code = element_code.xpath('.//CodiceValore')[0]
                    product = self.env['product.product'].search([('default_code', '=', code.text)], limit=2)
                    if product and len(product) == 1:
                        invoice_line_form.product_id = product
                        break

        # Label.
        line_elements = element.xpath('.//Descrizione')
        if line_elements:
            invoice_line_form.name = " ".join(line_elements[0].text.split())

        # Quantity.
        line_elements = element.xpath('.//Quantita')
        if line_elements:
            invoice_line_form.quantity = float(line_elements[0].text)
        else:
            invoice_line_form.quantity = 1

        # Taxes
        percentage = None
        price_subtotal = 0
        if not extra_info['simplified']:
            tax_element = element.xpath('.//AliquotaIVA')
            if tax_element and tax_element[0].text:
                percentage = float(tax_element[0].text)
        else:
            amount_element = element.xpath('.//Importo')
            if amount_element and amount_element[0].text:
                amount = float(amount_element[0].text)
                tax_element = element.xpath('.//Aliquota')
                if tax_element and tax_element[0].text:
                    percentage = float(tax_element[0].text)
                    price_subtotal = amount / (1 + percentage / 100)
                else:
                    tax_element = element.xpath('.//Imposta')
                    if tax_element and tax_element[0].text:
                        tax_amount = float(tax_element[0].text)
                        price_subtotal = amount - tax_amount
                        percentage = round(tax_amount / price_subtotal * 100)

        natura_element = element.xpath('.//Natura')
        invoice_line_form.tax_ids = ()
        if percentage is not None:
            type_tax_use_domain = extra_info.get('type_tax_use_domain', [('type_tax_use', '=', 'purchase')])
            l10n_it_kind_exoneration = bool(natura_element) and natura_element[0].text
            conditions = (
                l10n_it_kind_exoneration and [('l10n_it_kind_exoneration', '=', l10n_it_kind_exoneration)]
                or [('l10n_it_has_exoneration', '=', False)]
            ) + type_tax_use_domain
            tax = self._l10n_it_edi_search_tax_for_import(company, percentage, conditions)
            if tax:
                invoice_line_form.tax_ids += tax
            else:
                message_to_log.append("%s<br/>%s" % (
                    _("Tax not found for line with description '%s'", invoice_line_form.name),
                    self.env['account.move']._compose_info_message(element, '.'),
                ))

        # Price Unit.
        if not extra_info['simplified']:
            line_elements = element.xpath('.//PrezzoUnitario')
            if line_elements:
                invoice_line_form.price_unit = float(line_elements[0].text)
        else:
            invoice_line_form.price_unit = price_subtotal

        # Discounts
        discount_elements = element.xpath('.//ScontoMaggiorazione')
        if discount_elements:
            discount_element = discount_elements[0]
            discount_percentage = discount_element.xpath('.//Percentuale')
            # Special case of only 1 percentage discount
            if discount_percentage and len(discount_elements) == 1:
                discount_type = discount_element.xpath('.//Tipo')
                discount_sign = 1
                if discount_type and discount_type[0].text == 'MG':
                    discount_sign = -1
                invoice_line_form.discount = discount_sign * float(discount_percentage[0].text)
            # Discounts in cascade summarized in 1 percentage
            else:
                total = float(element.xpath('.//PrezzoTotale')[0].text)
                discount = 100 - (100 * total) / (invoice_line_form.quantity * invoice_line_form.price_unit)
                invoice_line_form.discount = discount

        return message_to_log

    # -------------------------------------------------------------------------
    # Export
    # -------------------------------------------------------------------------

    def _prepare_invoice_report(self, pdf_writer, edi_document):
        self.ensure_one()
        if self.code != 'fattura_pa':
            return super()._prepare_invoice_report(pdf_writer, edi_document)
        attachment = edi_document.sudo().attachment_id
        if attachment:
            pdf_writer.embed_odoo_attachment(attachment)

    def _is_compatible_with_journal(self, journal):
        # OVERRIDE
        self.ensure_one()
        if self.code != 'fattura_pa':
            return super()._is_compatible_with_journal(journal)
        return journal.type in ('sale', 'purchase') and journal.country_code == 'IT'

    def _get_move_applicability(self, move):
        # OVERRIDE
        self.ensure_one()
        if self.code != 'fattura_pa':
            return super()._get_move_applicability(move)

        is_it_purchase_document = self._l10n_it_edi_is_self_invoice(move) and move.is_purchase_document()
        if move.country_code == 'IT' and (move.is_sale_document() or is_it_purchase_document):
            return {
                'post': self._post_fattura_pa,
                'post_batching': lambda move: (move.move_type, bool(move.l10n_it_edi_transaction)),
                'batching_limit': 20,
            }

    def _l10n_it_edi_export_invoice_as_xml(self, invoice):
        ''' Create the xml file content.
        :return: The XML content as str.
        '''
        template_values = invoice._prepare_fatturapa_export_values()
        if not self._l10n_it_is_simplified_document_type(template_values['document_type']):
            content = self.env['ir.qweb']._render('l10n_it_edi.account_invoice_it_FatturaPA_export', template_values)
        else:
            content = self.env['ir.qweb']._render('l10n_it_edi.account_invoice_it_simplified_FatturaPA_export', template_values)
            invoice.message_post(body=_(
                "A simplified invoice was created instead of an ordinary one. This is because the invoice \
                is a domestic invoice with a total amount of less than or equal to 400€ and the customer's address is incomplete."
            ))
        return content

    def _check_move_configuration(self, move):
        # OVERRIDE
        res = super()._check_move_configuration(move)
        if self.code != 'fattura_pa':
            return res

        res.extend(self._l10n_it_edi_check_invoice_configuration(move))

        if not self._get_proxy_user(move.company_id):
            res.append(_("You must accept the terms and conditions in the settings to use FatturaPA."))

        return res

    def _needs_web_services(self):
        self.ensure_one()
        return self.code == 'fattura_pa' or super()._needs_web_services()

    def _l10n_it_post_invoices_step_1(self, invoices):
        ''' Send the invoices to the proxy.
        '''
        to_return = {}

        to_send = {}
        for invoice in invoices:
            xml = "<?xml version='1.0' encoding='UTF-8'?>" + str(self._l10n_it_edi_export_invoice_as_xml(invoice))
            filename = self._l10n_it_edi_generate_electronic_invoice_filename(invoice)
            attachment = self.env['ir.attachment'].create({
                'name': filename,
                'res_id': invoice.id,
                'res_model': invoice._name,
                'raw': xml.encode(),
                'description': _('Italian invoice: %s', invoice.move_type),
                'type': 'binary',
            })
            invoice.l10n_it_edi_attachment_id = attachment

            if invoice._is_commercial_partner_pa():
                invoice.message_post(
                    body=(_("Invoices for PA are not managed by Odoo, you can download the document and send it on your own."))
                )
                to_return[invoice] = {'attachment': attachment, 'success': True}
            else:
                to_send[filename] = {
                    'invoice': invoice,
                    'data': {'filename': filename, 'xml': base64.b64encode(xml.encode()).decode()}}

        company = invoices.company_id
        proxy_user = self._get_proxy_user(company)
        if not proxy_user:  # proxy user should exist, because there is a check in _check_move_configuration
            return {invoice: {
                'error': _("You must accept the terms and conditions in the settings to use FatturaPA."),
                'blocking_level': 'error'} for invoice in invoices}

        responses = {}
        if proxy_user._get_demo_state() == 'demo':
            responses = {i['data']['filename']: {'id_transaction': 'demo'} for i in to_send.values()}
        else:
            try:
                responses = self._l10n_it_edi_upload([i['data'] for i in to_send.values()], proxy_user)
            except AccountEdiProxyError as e:
                return {invoice: {'error': e.message, 'blocking_level': 'error'} for invoice in invoices}

        for filename, response in responses.items():
            invoice = to_send[filename]['invoice']
            to_return[invoice] = response
            if 'id_transaction' in response:
                invoice.l10n_it_edi_transaction = response['id_transaction']
                to_return[invoice].update({
                    'error': _('The invoice was sent to FatturaPA, but we are still awaiting a response. Click the link above to check for an update.'),
                    'blocking_level': 'info'})
        return to_return

    def _l10n_it_post_invoices_step_2(self, invoices):
        ''' Check if the sent invoices have been processed by FatturaPA.
        '''
        to_check = {i.l10n_it_edi_transaction: i for i in invoices}
        to_return = {}
        company = invoices.company_id

        proxy_user = self._get_proxy_user(company)
        if not proxy_user:  # proxy user should exist, because there is a check in _check_move_configuration
            return {invoice: {
                'error': _("You must accept the terms and conditions in the settings to use FatturaPA."),
                'blocking_level': 'error'} for invoice in invoices}

        if proxy_user._get_demo_state() == 'demo':
            # simulate success and bypass ack
            return {invoice: {'attachment': invoice.l10n_it_edi_attachment_id} for invoice in invoices}
        else:
            try:
                responses = proxy_user._make_request(proxy_user._get_server_url() + '/api/l10n_it_edi/1/in/TrasmissioneFatture',
                                                    params={'ids_transaction': list(to_check.keys())})
            except AccountEdiProxyError as e:
                return {invoice: {'error': e.message, 'blocking_level': 'error'} for invoice in invoices}

        proxy_acks = []
        for id_transaction, response in responses.items():
            invoice = to_check[id_transaction]
            if 'error' in response:
                to_return[invoice] = response
                continue

            state = response['state']
            if state == 'awaiting_outcome':
                to_return[invoice] = {
                    'error': _('The invoice was sent to FatturaPA, but we are still awaiting a response. Click the link above to check for an update.'),
                    'blocking_level': 'info'}

            elif state == 'not_found':
                # Invoice does not exist on proxy. Either it does not belong to this proxy_user or it was not created correctly when
                # it was sent to the proxy.
                to_return[invoice] = {'error': _('You are not allowed to check the status of this invoice.'), 'blocking_level': 'error'}

            elif state == 'ricevutaConsegna':
                if invoice._is_commercial_partner_pa():
                    to_return[invoice] = {'error': _('The invoice has been succesfully transmitted. The addressee has 15 days to accept or reject it.')}
                else:
                    to_return[invoice] = {'attachment': invoice.l10n_it_edi_attachment_id, 'success': True}
                proxy_acks.append(id_transaction)

            elif state == 'notificaMancataConsegna':
                if invoice._is_commercial_partner_pa():
                    to_return[invoice] = {'error': _(
                        'The invoice has been issued, but the delivery to the Public Administration'
                        ' has failed. The Exchange System will contact them to report the problem'
                        ' and request that they provide a solution.'
                        ' During the following 10 days, the Exchange System will try to forward the'
                        ' FatturaPA file to the Public Administration in question again.'
                        ' Should this also fail, the System will notify Odoo of the failed delivery,'
                        ' and you will be required to send the invoice to the Administration'
                        ' through another channel, outside of the Exchange System.')}
                else:
                    to_return[invoice] = {'success': True, 'attachment': invoice.l10n_it_edi_attachment_id}
                    invoice._message_log(body=_(
                        'The invoice has been issued, but the delivery to the Addressee has'
                        ' failed. You will be required to send a courtesy copy of the invoice'
                        ' to your customer through another channel, outside of the Exchange'
                        ' System, and promptly notify him that the original is deposited'
                        ' in his personal area on the portal "Invoices and Fees" of the'
                        ' Revenue Agency.'))
                proxy_acks.append(id_transaction)

            elif state == 'NotificaDecorrenzaTermini':
                # This condition is part of the Public Administration flow
                invoice._message_log(body=_(
                    'The invoice has been correctly issued. The Public Administration recipient'
                    ' had 15 days to either accept or refused this document, but they did not reply,'
                    ' so from now on we consider it accepted.'))
                to_return[invoice] = {'attachment': invoice.l10n_it_edi_attachment_id, 'success': True}
                proxy_acks.append(id_transaction)

            # In the transaction states above, we don't need to read the attachment.
            # In the following cases instead we need to read the information inside
            # about the notification itself, i.e. the error message in case of rejection.
            else:
                attachment_file = response.get('file')
                if not attachment_file: # It means there is no status update, so we can skip it
                    document = invoice.edi_document_ids.filtered(lambda d: d.edi_format_id.code == 'fattura_pa')
                    to_return[invoice] = {'error': document.error, 'blocking_level': document.blocking_level}
                    continue

                xml = proxy_user._decrypt_data(attachment_file, response['key'])
                response_tree = etree.fromstring(xml)

                if state == 'notificaScarto':
                    elements = response_tree.xpath('//Errore')
                    error_codes = [element.find('Codice').text for element in elements]
                    errors = [element.find('Descrizione').text for element in elements]
                    # Duplicated invoice
                    if '00404' in error_codes:
                        idx = error_codes.index('00404')
                        invoice.message_post(body=_(
                            'This invoice number had already been submitted to the SdI, so it is'
                            ' set as Sent. Please verify that the system is correctly configured,'
                            ' because the correct flow does not need to send the same invoice'
                            ' twice for any reason.\n'
                            ' Original message from the SDI: %s', errors[idx]))
                        to_return[invoice] = {'attachment': invoice.l10n_it_edi_attachment_id, 'success': True}
                    else:
                        # Add helpful text if duplicated filename error
                        if '00002' in error_codes:
                            idx = error_codes.index('00002')
                            errors[idx] = _(
                                'The filename is duplicated. Try again (or adjust the FatturaPA Filename sequence).'
                                ' Original message from the SDI: %s', [errors[idx]]
                            )
                        to_return[invoice] = {'error': self._format_error_message(_('The invoice has been refused by the Exchange System'), errors), 'blocking_level': 'error'}
                        invoice.l10n_it_edi_transaction = False
                    proxy_acks.append(id_transaction)

                elif state == 'notificaEsito':
                    outcome = response_tree.find('Esito').text
                    if outcome == 'EC01':
                        to_return[invoice] = {'attachment': invoice.l10n_it_edi_attachment_id, 'success': True}
                    else:  # ECO2
                        to_return[invoice] = {'error': _('The invoice was refused by the addressee.'), 'blocking_level': 'error'}
                    proxy_acks.append(id_transaction)

        if proxy_acks:
            try:
                proxy_user._make_request(proxy_user._get_server_url() + '/api/l10n_it_edi/1/ack',
                                        params={'transaction_ids': proxy_acks})
            except AccountEdiProxyError as e:
                # Will be ignored and acked again next time.
                _logger.error('Error while acking file to SdiCoop: %s', e)

        return to_return

    def _post_fattura_pa(self, invoice):
        if not invoice[0].l10n_it_edi_transaction:
            return self._l10n_it_post_invoices_step_1(invoice)
        else:
            return self._l10n_it_post_invoices_step_2(invoice)

    def _post_invoice_edi(self, invoices):
        # OVERRIDE
        self.ensure_one()
        edi_result = super()._post_invoice_edi(invoices)
        if self.code != 'fattura_pa':
            return edi_result

        return self._post_fattura_pa(invoices)

    # -------------------------------------------------------------------------
    # Proxy methods
    # -------------------------------------------------------------------------

    def _get_proxy_identification(self, company):
        if self.code != 'fattura_pa':
            return super()._get_proxy_identification()

        if not company.l10n_it_codice_fiscale:
            raise UserError(_('Please fill your codice fiscale to be able to receive invoices from FatturaPA'))

        return self.env['res.partner']._l10n_it_normalize_codice_fiscale(company.l10n_it_codice_fiscale)

    def _l10n_it_edi_upload(self, files, proxy_user):
        '''Upload files to fatturapa.

        :param files:    A list of dictionary {filename, base64_xml}.
        :returns:        A dictionary.
        * message:       Message from fatturapa.
        * transactionId: The fatturapa ID of this request.
        * error:         An eventual error.
        * error_level:   Info, warning, error.
        '''
        ERRORS = {
            'EI01': {'error': _lt('Attached file is empty'), 'blocking_level': 'error'},
            'EI02': {'error': _lt('Service momentarily unavailable'), 'blocking_level': 'warning'},
            'EI03': {'error': _lt('Unauthorized user'), 'blocking_level': 'error'},
        }

        if not files:
            return {}

        result = proxy_user._make_request(proxy_user._get_server_url() + '/api/l10n_it_edi/1/out/SdiRiceviFile', params={'files': files})

        # Translate the errors.
        for filename in result.keys():
            if 'error' in result[filename]:
                result[filename] = ERRORS.get(result[filename]['error'], {'error': result[filename]['error'], 'blocking_level': 'error'})

        return result

```

## File: models\account_invoice.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
from functools import reduce
import logging
import re

from datetime import datetime

from odoo import api, fields, models, _
from odoo.tools import float_repr, float_compare
from odoo.exceptions import UserError, ValidationError


_logger = logging.getLogger(__name__)

DEFAULT_FACTUR_ITALIAN_DATE_FORMAT = '%Y-%m-%d'


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_it_edi_transaction = fields.Char(copy=False, string="FatturaPA Transaction")
    l10n_it_edi_attachment_id = fields.Many2one('ir.attachment', copy=False, string="FatturaPA Attachment", ondelete="restrict")

    l10n_it_stamp_duty = fields.Float(string="Dati Bollo", readonly=True, states={'draft': [('readonly', False)]})

    l10n_it_ddt_id = fields.Many2one('l10n_it.ddt', string='DDT', readonly=True, states={'draft': [('readonly', False)]}, copy=False)

    l10n_it_einvoice_name = fields.Char(compute='_compute_l10n_it_einvoice')

    l10n_it_einvoice_id = fields.Many2one('ir.attachment', string="Electronic invoice", compute='_compute_l10n_it_einvoice')

    def _get_l10n_it_amount_split_payment(self):
        self.ensure_one()
        if not self.is_sale_document(False):
            return 0.0
        sign = -1 if self.move_type == "out_invoice" else 1
        return sum(sign * line.balance for line in self.line_ids.filtered(lambda l: l.tax_line_id and l.tax_line_id._l10n_it_is_split_payment()))

    @api.depends('edi_document_ids', 'edi_document_ids.attachment_id')
    def _compute_l10n_it_einvoice(self):
        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        for invoice in self:
            einvoice = invoice.edi_document_ids.filtered(lambda d: d.edi_format_id == fattura_pa).sudo()
            invoice.l10n_it_einvoice_id = einvoice.attachment_id
            invoice.l10n_it_einvoice_name = einvoice.attachment_id.name

    @api.depends('l10n_it_edi_transaction')
    def _compute_show_reset_to_draft_button(self):
        super(AccountMove, self)._compute_show_reset_to_draft_button()
        for move in self.filtered(lambda m: m.l10n_it_edi_transaction):
            move.show_reset_to_draft_button = False

    def invoice_generate_xml(self):
        self.ensure_one()
        report_name = self.env['account.edi.format']._l10n_it_edi_generate_electronic_invoice_filename(self)

        data = "<?xml version='1.0' encoding='UTF-8'?>" + str(self._l10n_it_edi_export_invoice_as_xml())
        description = _('Italian invoice: %s', self.move_type)
        attachment = self.env['ir.attachment'].create({
            'name': report_name,
            'res_id': self.id,
            'res_model': self._name,
            'raw': data.encode(),
            'description': description,
            'type': 'binary',
            })

        self.message_post(
            body=(_("E-Invoice is generated on %s by %s") % (fields.Datetime.now(), self.env.user.display_name))
        )
        return {'attachment': attachment}

    def _is_commercial_partner_pa(self):
        """
            Returns True if the destination of the FatturaPA belongs to the Public Administration.
        """
        return len(self.commercial_partner_id.l10n_it_pa_index or '') == 6

    def _l10n_it_edi_prepare_fatturapa_line_details(self, reverse_charge_refund=False, is_downpayment=False, convert_to_euros=True):
        """ Returns a list of dictionaries passed to the template for the invoice lines (DettaglioLinee)
        """
        invoice_lines = []
        lines = self.invoice_line_ids.filtered(lambda l: not l.display_type in ('line_note', 'line_section'))
        for num, line in enumerate(lines):
            sign = -1 if line.move_id.is_inbound() else 1
            price_subtotal = (line.balance * sign) if convert_to_euros else line.price_subtotal
            # The price_subtotal should be inverted when the line is a reverse charge refund.
            if reverse_charge_refund:
                price_subtotal = -price_subtotal

            # Unit price
            price_unit = 0
            if line.quantity and line.discount != 100.0:
                price_unit = price_subtotal / ((1 - (line.discount or 0.0) / 100.0) * abs(line.quantity))
            else:
                price_unit = line.price_unit

            description = line.name

            # Down payment lines:
            # If there was a down paid amount that has been deducted from this move,
            # we need to put a reference to the down payment invoice in the DatiFattureCollegate tag
            downpayment_moves = self.env['account.move']
            if not is_downpayment and line.price_subtotal < 0:
                downpayment_moves = line._get_downpayment_lines().mapped("move_id")
                if downpayment_moves:
                    downpayment_moves_description = ', '.join([m.name for m in downpayment_moves])
                    sep = ', ' if description else ''
                    description = f"{description}{sep}{downpayment_moves_description}"

            vat_tax = line.tax_ids.flatten_taxes_hierarchy().filtered(lambda t: t._l10n_it_filter_kind('vat') and t.amount >= 0)
            invoice_lines.append({
                'line': line,
                'line_number': num + 1,
                'description': description or 'NO NAME',
                'unit_price': price_unit,
                'subtotal_price': price_subtotal,
                'vat_tax': vat_tax,
                'downpayment_moves': downpayment_moves,
            })
        return invoice_lines

    def _l10n_it_edi_prepare_fatturapa_tax_details(self, tax_details, reverse_charge_refund=False):
        """ Returns a list of dictionaries passed to the template for the invoice lines (DatiRiepilogo)
        """
        tax_lines = []
        for _tax_name, tax_dict in tax_details['tax_details'].items():
            # The assumption is that the company currency is EUR.
            tax = tax_dict['tax']
            base_amount = tax_dict['base_amount']
            tax_amount = tax_dict['tax_amount']
            tax_rate = tax.amount
            tax_exigibility_code = (
                'S' if tax._l10n_it_is_split_payment()
                else 'D' if tax.tax_exigibility == 'on_payment'
                else 'I' if tax.tax_exigibility == 'on_invoice'
                else False
            )
            expected_base_amount = tax_amount * 100 / tax_rate if tax_rate else False
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

    def _l10n_it_edi_filter_fatturapa_tax_details(self, line, tax_values):
        """Filters tax details to only include the positive amounted lines regarding VAT taxes."""
        repartition_line = tax_values['tax_repartition_line']
        return (repartition_line.factor_percent >= 0 and repartition_line.tax_id.amount >= 0)

    def _prepare_fatturapa_export_values(self):
        self.ensure_one()

        def format_date(dt):
            # Format the date in the italian standard.
            dt = dt or datetime.now()
            return dt.strftime(DEFAULT_FACTUR_ITALIAN_DATE_FORMAT)

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

        def discount_type(discount):
            return 'SC' if discount > 0 else 'MG'

        def format_phone(number):
            if not number:
                return False
            number = number.replace(' ', '').replace('/', '').replace('.', '')
            if len(number) > 4 and len(number) < 13:
                return number
            return False

        def get_vat_number(vat):
            if vat[:2].isdecimal():
                return vat.replace(' ', '')
            return vat[2:].replace(' ', '')

        def get_vat_country(vat):
            if vat[:2].isdecimal():
                return 'IT'
            return vat[:2].upper()

        def format_alphanumeric(text_to_convert):
            return text_to_convert.encode('latin-1', 'replace').decode('latin-1') if text_to_convert else False

        def get_vat_values(partner):
            """ Generate the VAT and country code needed by l10n_it_edi XML export.

                VAT number:
                If there is a VAT number and the partner is not in EU and San Marino, then the exported value is 'OO99999999999'
                If there is a VAT number and the partner is in EU or San Marino, then remove the country prefix
                If there is no VAT and the partner is not in Italy, then the exported value is '0000000'
                If there is no VAT and the partner is in Italy, the VAT is not set and Codice Fiscale will be relevant in the XML.
                If there is no VAT and no Codice Fiscale, the invoice is not even exported, so this case is not handled.

                Country:
                First, take the country configured on the partner.
                If there's a codice fiscale and no country, the country is 'IT'.
            """
            europe = self.env.ref('base.europe', raise_if_not_found=False)
            in_eu = europe and partner.country_id and partner.country_id in europe.country_ids
            is_sm = partner.country_code == 'SM'

            normalized_vat = partner.vat
            normalized_country = partner.country_code
            has_vat = partner.vat and not partner.vat in ['/', 'NA']
            if has_vat:
                normalized_vat = partner.vat.replace(' ', '')
                if in_eu:
                    # If the partner is from the EU, the country-code prefix of the VAT must be taken away
                    if not normalized_vat[:2].isdecimal():
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
            if not normalized_country and partner.l10n_it_codice_fiscale:
                normalized_country = 'IT'
            # If customer has not VAT
            elif not has_vat and partner.country_id and partner.country_id.code != 'IT':
                normalized_vat = '0000000'

            return {
                'vat': normalized_vat,
                'country_code': normalized_country,
            }

        formato_trasmissione = "FPA12" if self._is_commercial_partner_pa() else "FPR12"

        # Flags
        in_eu = self.env['account.edi.format']._l10n_it_edi_partner_in_eu
        is_self_invoice = self.env['account.edi.format']._l10n_it_edi_is_self_invoice(self)
        document_type = self.env['account.edi.format']._l10n_it_get_document_type(self)
        if self.env['account.edi.format']._l10n_it_is_simplified_document_type(document_type):
            formato_trasmissione = "FSM10"

        # Represent if the document is a reverse charge refund in a single variable
        reverse_charge = document_type in ['TD17', 'TD18', 'TD19']
        is_downpayment = document_type in ['TD02']
        reverse_charge_refund = self.move_type == 'in_refund' and reverse_charge
        convert_to_euros = self.currency_id.name != 'EUR'

        # b64encode returns a bytestring, the template tries to turn it to string,
        # but only gets the repr(pdf) --> "b'<base64_data>'"
        pdf = self.env['ir.actions.report']._render_qweb_pdf("account.account_invoices", self.id)[0]
        pdf = base64.b64encode(pdf).decode()
        pdf_name = re.sub(r'\W+', '', self.name) + '.pdf'

        tax_details = self._prepare_edi_tax_details(filter_to_apply=self._l10n_it_edi_filter_fatturapa_tax_details)

        company = self.company_id
        partner = self.commercial_partner_id
        buyer = partner if not is_self_invoice else company
        seller = company if not is_self_invoice else partner
        codice_destinatario = (
            (is_self_invoice and company.partner_id.l10n_it_pa_index)
            # San Marino is externally integrated with the SdI.
            # The country as a whole has a single fixed Destination Code (i.e. "2R4GTO8").
            # https://www.agenziaentrate.gov.it/portale/documents/20143/3788702/Modifiche+ProvvedimentonSanMarino+0248717-2021.pdf/429b5571-17b9-0cce-7f62-f79cf53086d7
            or (partner.country_code == 'SM' and '2R4GTO8')
            or partner.l10n_it_pa_index
            or (partner.country_id.code == 'IT' and '0000000')
            or 'XXXXXXX')

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
        conversion_line = self.invoice_line_ids.sorted(lambda l: abs(l.balance), reverse=True)[0] if self.invoice_line_ids else None
        conversion_rate = float_repr(
            abs(conversion_line.balance / conversion_line.amount_currency), precision_digits=5,
        ) if convert_to_euros and conversion_line and conversion_line.amount_currency else None

        invoice_lines = self._l10n_it_edi_prepare_fatturapa_line_details(reverse_charge_refund, is_downpayment, convert_to_euros)
        tax_lines = self._l10n_it_edi_prepare_fatturapa_tax_details(tax_details, reverse_charge_refund)

        # Reduce downpayment views to a single recordset
        downpayment_moves = [l.get('downpayment_moves', self.env['account.move']) for l in invoice_lines]
        downpayment_moves = self.browse(move.id for moves in downpayment_moves for move in moves)

        # Create file content.
        template_values = {
            'record': self,
            'balance_multiplicator': -1 if self.is_inbound() else 1,
            'company': company,
            'sender': company,
            'sender_partner': company.partner_id,
            'partner': partner,
            'buyer': buyer,
            'buyer_partner': partner if not is_self_invoice else company.partner_id,
            'buyer_is_company': is_self_invoice or partner.is_company,
            'seller': seller,
            'seller_partner': company.partner_id if not is_self_invoice else partner,
            'origin_document_type': False, # see module l10n_it_edi_origin_document, will be merged in master
            'currency': self.currency_id or self.company_currency_id if not convert_to_euros else self.env.ref('base.EUR'),
            'document_total': document_total,
            'representative': company.l10n_it_tax_representative_partner_id,
            'codice_destinatario': codice_destinatario,
            'regime_fiscale': company.l10n_it_tax_system if not is_self_invoice else 'RF18',
            'is_self_invoice': is_self_invoice,
            'partner_bank': self.partner_bank_id,
            'format_date': format_date,
            'format_monetary': format_monetary,
            'format_numbers': format_numbers,
            'format_numbers_two': format_numbers_two,
            'format_phone': format_phone,
            'format_alphanumeric': format_alphanumeric,
            'discount_type': discount_type,
            'formato_trasmissione': formato_trasmissione,
            'document_type': document_type,
            'pdf': pdf,
            'pdf_name': pdf_name,
            'tax_details': tax_details,
            'downpayment_moves': downpayment_moves,
            'abs': abs,
            'normalize_codice_fiscale': partner._l10n_it_normalize_codice_fiscale,
            'get_vat_number': get_vat_number,
            'get_vat_country': get_vat_country,
            'in_eu': in_eu,
            'rc_refund': reverse_charge_refund,
            'invoice_lines': invoice_lines,
            'tax_lines': tax_lines,
            'conversion_rate': conversion_rate,
            'buyer_info': get_vat_values(buyer),
            'seller_info': get_vat_values(seller),
        }
        return template_values

    def _post(self, soft=True):
        # OVERRIDE
        posted = super()._post(soft=soft)
        return posted

    def _compose_info_message(self, tree, element_tags):
        output_str = ""
        elements = tree.xpath(element_tags)
        for element in elements:
            output_str += "<ul>"
            for line in element.iter():
                if line.text:
                    text = " ".join(line.text.split())
                    if text:
                        output_str += "<li>%s: %s</li>" % (line.tag, text)
            output_str += "</ul>"
        return output_str

    def _compose_multi_info_message(self, tree, element_tags):
        output_str = "<ul>"

        for element_tag in element_tags:
            elements = tree.xpath(element_tag)
            if not elements:
                continue
            for element in elements:
                text = " ".join(element.text.split())
                if text:
                    output_str += "<li>%s: %s</li>" % (element.tag, text)
        return output_str + "</ul>"

class AccountTax(models.Model):
    _name = "account.tax"
    _inherit = "account.tax"

    l10n_it_vat_due_date = fields.Selection([
        ("I", "[I] IVA ad esigibilità immediata"),
        ("D", "[D] IVA ad esigibilità differita"),
        ("S", "[S] Scissione dei pagamenti")], default="I", string="VAT due date")

    l10n_it_has_exoneration = fields.Boolean(string="Has exoneration of tax (Italy)", help="Tax has a tax exoneration.")
    l10n_it_kind_exoneration = fields.Selection(selection=[
            ("N1", "[N1] Escluse ex art. 15"),
            ("N2", "[N2] Non soggette"),
            ("N2.1", "[N2.1] Non soggette ad IVA ai sensi degli artt. Da 7 a 7-septies del DPR 633/72"),
            ("N2.2", "[N2.2] Non soggette – altri casi"),
            ("N3", "[N3] Non imponibili"),
            ("N3.1", "[N3.1] Non imponibili – esportazioni"),
            ("N3.2", "[N3.2] Non imponibili – cessioni intracomunitarie"),
            ("N3.3", "[N3.3] Non imponibili – cessioni verso San Marino"),
            ("N3.4", "[N3.4] Non imponibili – operazioni assimilate alle cessioni all’esportazione"),
            ("N3.5", "[N3.5] Non imponibili – a seguito di dichiarazioni d’intento"),
            ("N3.6", "[N3.6] Non imponibili – altre operazioni che non concorrono alla formazione del plafond"),
            ("N4", "[N4] Esenti"),
            ("N5", "[N5] Regime del margine / IVA non esposta in fattura"),
            ("N6", "[N6] Inversione contabile (per le operazioni in reverse charge ovvero nei casi di autofatturazione per acquisti extra UE di servizi ovvero per importazioni di beni nei soli casi previsti)"),
            ("N6.1", "[N6.1] Inversione contabile – cessione di rottami e altri materiali di recupero"),
            ("N6.2", "[N6.2] Inversione contabile – cessione di oro e argento puro"),
            ("N6.3", "[N6.3] Inversione contabile – subappalto nel settore edile"),
            ("N6.4", "[N6.4] Inversione contabile – cessione di fabbricati"),
            ("N6.5", "[N6.5] Inversione contabile – cessione di telefoni cellulari"),
            ("N6.6", "[N6.6] Inversione contabile – cessione di prodotti elettronici"),
            ("N6.7", "[N6.7] Inversione contabile – prestazioni comparto edile esettori connessi"),
            ("N6.8", "[N6.8] Inversione contabile – operazioni settore energetico"),
            ("N6.9", "[N6.9] Inversione contabile – altri casi"),
            ("N7", "[N7] IVA assolta in altro stato UE (prestazione di servizi di telecomunicazioni, tele-radiodiffusione ed elettronici ex art. 7-octies, comma 1 lett. a, b, art. 74-sexies DPR 633/72)")],
        string="Exoneration",
        help="Exoneration type",
        default="N1")
    l10n_it_law_reference = fields.Char(string="Law Reference", size=100)

    @api.constrains('l10n_it_has_exoneration',
                    'l10n_it_kind_exoneration',
                    'l10n_it_law_reference',
                    'amount',
                    'invoice_repartition_line_ids',
                    'refund_repartition_line_ids')
    def _check_exoneration_with_no_tax(self):
        for tax in self:
            if tax.l10n_it_has_exoneration:
                if not tax.l10n_it_kind_exoneration or not tax.l10n_it_law_reference or tax.amount != 0:
                    raise ValidationError(_("If the tax has exoneration, you must enter a kind of exoneration, a law reference and the amount of the tax must be 0.0."))
                if tax.l10n_it_kind_exoneration == 'N6' and tax._l10n_it_is_split_payment():
                    raise UserError(_("Split Payment is not compatible with exoneration of kind 'N6'"))

    def _l10n_it_filter_kind(self, kind):
        """ This can be overridden by l10n_it_edi_withholding for different kind of taxes (withholding, pension_fund)."""
        return self if kind == 'vat' else self.env['account.tax']

    def _l10n_it_is_split_payment(self):
        """ Split payment means that the Public Administration buyer will pay VAT
            to the tax agency instead of the vendor
        """
        self.ensure_one()

        tax_tags = self.get_tax_tags(is_refund=False, repartition_type='base') | self.get_tax_tags(is_refund=False, repartition_type='tax')
        if not tax_tags:
            return False

        it_tax_report_ve38_lines = self.env['account.report.line'].search([
            ('report_id.country_id.code', '=', 'IT'),
            ('code', '=', 'VE38'),
        ])
        if not it_tax_report_ve38_lines:
            return False

        ve38_lines_tags = it_tax_report_ve38_lines.expression_ids._get_matching_tags()
        return bool(tax_tags & ve38_lines_tags)

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

    def name_get(self):
        res = []
        for ddt in self:
            res.append((ddt.id, ("%s (%s)") % (ddt.name, ddt.date)))
        return res

```

## File: models\mail_template.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class MailTemplate(models.Model):
    _inherit = "mail.template"

    def _get_edi_attachments(self, document):
        """
        Will return the information about the attachment of the edi document for adding the attachment in the mail.
        Can be overridden where e.g. a zip-file needs to be sent with the individual files instead of the entire zip
        :param document: an edi document
        :return: list with a tuple with the name and base64 content of the attachment
        """
        if document.edi_format_id.code == 'fattura_pa':
            return {}
        return super()._get_edi_attachments(document)

```

## File: models\res_company.py

```python
# -*- coding:utf-8 -*-
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

    @api.onchange("l10n_it_has_tax_representative")
    def _onchange_l10n_it_has_tax_represeentative(self):
        for company in self:
            if not company.l10n_it_has_tax_representative:
                company.l10n_it_tax_representative_partner_id = False

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields, _
from odoo.exceptions import UserError

class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    is_edi_proxy_active = fields.Boolean(compute='_compute_is_edi_proxy_active')
    l10n_it_edi_proxy_current_state = fields.Char(compute='_compute_l10n_it_edi_proxy_current_state')
    l10n_it_edi_sdicoop_register = fields.Boolean(compute='_compute_l10n_it_edi_sdicoop_register', inverse='_set_l10n_it_edi_sdicoop_register_demo_mode')
    l10n_it_edi_sdicoop_demo_mode = fields.Selection(
        [('demo', 'Demo'),
         ('test', 'Test (experimental)'),
         ('prod', 'Official')],
        compute='_compute_l10n_it_edi_sdicoop_demo_mode',
        inverse='_set_l10n_it_edi_sdicoop_register_demo_mode',
        readonly=False)

    def _create_proxy_user(self, company_id):
        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        edi_identification = fattura_pa._get_proxy_identification(company_id)
        self.env['account_edi_proxy_client.user']._register_proxy_user(company_id, fattura_pa, edi_identification)

    @api.depends('company_id.account_edi_proxy_client_ids', 'company_id.account_edi_proxy_client_ids.active')
    def _compute_l10n_it_edi_sdicoop_demo_mode(self):
        for config in self:
            config.l10n_it_edi_sdicoop_demo_mode = self.env['account_edi_proxy_client.user']._get_demo_state()

    def _set_l10n_it_edi_sdicoop_demo_mode(self):
        for config in self:
            self.env['ir.config_parameter'].set_param('account_edi_proxy_client.demo', config.l10n_it_edi_sdicoop_demo_mode)

    @api.depends('company_id.account_edi_proxy_client_ids', 'company_id.account_edi_proxy_client_ids.active')
    def _compute_is_edi_proxy_active(self):
        for config in self:
            config.is_edi_proxy_active = config.company_id.account_edi_proxy_client_ids

    @api.depends('company_id.account_edi_proxy_client_ids', 'company_id.account_edi_proxy_client_ids.active')
    def _compute_l10n_it_edi_proxy_current_state(self):
        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        for config in self:
            proxy_user = config.company_id.account_edi_proxy_client_ids.search([
                ('company_id', '=', config.company_id.id),
                ('edi_format_id', '=', fattura_pa.id),
            ], limit=1)

            config.l10n_it_edi_proxy_current_state = 'inactive' if not proxy_user else 'demo' if proxy_user.id_client[:4] == 'demo' else 'active'

    @api.depends('company_id')
    def _compute_l10n_it_edi_sdicoop_register(self):
        """Needed because it expects a compute"""
        self.l10n_it_edi_sdicoop_register = False

    def button_create_proxy_user(self):
        # For now, only fattura_pa uses the proxy.
        # To use it for more, we have to either make the activation of the proxy on a format basis
        # or create a user per format here (but also when installing new formats)
        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        edi_identification = fattura_pa._get_proxy_identification(self.company_id)
        if not edi_identification:
            return

        self.env['account_edi_proxy_client.user']._register_proxy_user(self.company_id, fattura_pa, edi_identification)

    def _set_l10n_it_edi_sdicoop_register_demo_mode(self):

        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        for config in self:

            proxy_user = self.env['account_edi_proxy_client.user'].search([
                ('company_id', '=', config.company_id.id),
                ('edi_format_id', '=', fattura_pa.id)
            ], limit=1)

            real_proxy_users = self.env['account_edi_proxy_client.user'].sudo().search([
                ('id_client', 'not like', 'demo'),
            ])

            # Update the config as per the selected radio button
            previous_demo_state = proxy_user._get_demo_state()
            self.env['ir.config_parameter'].set_param('account_edi_proxy_client.demo', config.l10n_it_edi_sdicoop_demo_mode)

            # If the user is trying to change from a state in which they have a registered official or testing proxy client
            # to another state, we should stop them
            if real_proxy_users and previous_demo_state != config.l10n_it_edi_sdicoop_demo_mode:
                raise UserError(_("The company has already registered with the service as 'Test' or 'Official', it cannot change."))

            if config.l10n_it_edi_sdicoop_register:
                # There should only be one user at a time, if there are no users, register one
                if not proxy_user:
                    self._create_proxy_user(config.company_id)
                    return

                # If there is a demo user, and we are transitioning from demo to test or production, we should
                # delete all demo users and then create the new user.
                elif proxy_user.id_client[:4] == 'demo' and config.l10n_it_edi_sdicoop_demo_mode != 'demo':
                    self.env['account_edi_proxy_client.user'].search([('id_client', '=like', 'demo%')]).sudo().unlink()
                    self._create_proxy_user(config.company_id)

```

## File: models\res_partner.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from stdnum.it import codicefiscale, iva

from odoo import api, fields, models, _
from odoo.exceptions import UserError

import re


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    l10n_it_pec_email = fields.Char(string="PEC e-mail")
    l10n_it_codice_fiscale = fields.Char(string="Codice Fiscale", size=16)
    l10n_it_pa_index = fields.Char(string="Destination Code",
        size=7,
        help="Must contain the 6-character (or 7) code, present in the PA\
              Index in the information relative to the electronic invoicing service,\
              associated with the office which, within the addressee administration, deals\
              with receiving (and processing) the invoice.")

    _sql_constraints = [
        ('l10n_it_codice_fiscale',
            "CHECK(l10n_it_codice_fiscale IS NULL OR l10n_it_codice_fiscale = '' OR LENGTH(l10n_it_codice_fiscale) >= 11)",
            "Codice fiscale must have between 11 and 16 characters."),

        ('l10n_it_pa_index',
            "CHECK(l10n_it_pa_index IS NULL OR l10n_it_pa_index = '' OR LENGTH(l10n_it_pa_index) >= 6)",
            "Destination Code must have between 6 and 7 characters."),
    ]

    @api.model
    def _l10n_it_normalize_codice_fiscale(self, codice):
        if codice and re.match(r'^IT[0-9]{11}$', codice):
            return codice[2:13]
        return codice

    @api.onchange('vat', 'country_id')
    def _l10n_it_onchange_vat(self):
        if self.vat and (
            self.country_code == "IT"
            if self.country_code
            else self.vat.startswith("IT")
        ):
            self.l10n_it_codice_fiscale = self._l10n_it_normalize_codice_fiscale(self.vat)
        else:
            self.l10n_it_codice_fiscale = False

    @api.constrains('l10n_it_codice_fiscale')
    def validate_codice_fiscale(self):
        for record in self:
            if record.l10n_it_codice_fiscale and (not codicefiscale.is_valid(record.l10n_it_codice_fiscale) and not iva.is_valid(record.l10n_it_codice_fiscale)):
                raise UserError(_("Invalid Codice Fiscale '%s': should be like 'MRTMTT91D08F205J' for physical person and '12345670546' or 'IT12345670546' for businesses.", record.l10n_it_codice_fiscale))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner
from . import res_company
from . import res_config_settings
from . import account_chart_template
from . import account_invoice
from . import account_edi_format
from . import ddt
from . import mail_template

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
    <record id="account_tax_form_l10n_it" model="ir.ui.view">
        <field name="name">account.tax.form.l10n.it</field>
        <field name="model">account.tax</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//page[@name='advanced_options']" position="inside">
                <group attrs="{'invisible': [('country_code', '!=', 'IT')]}">
                    <group>
                        <field name="l10n_it_vat_due_date" invisible="1"/>
                        <field name="l10n_it_has_exoneration"/>
                        <field name="l10n_it_kind_exoneration" attrs="{'invisible': [('l10n_it_has_exoneration', '=', False)]}"/>
                        <field name="l10n_it_law_reference"/>
                    </group>
                </group>
            </xpath>
        </data>
        </field>
    </record>

    <record id="res_partner_form_l10n_it" model="ir.ui.view">
        <field name="name">res.partner.form.l10n.it</field>
        <field name="model">res.partner</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//field[@name='category_id']" position="after">
                <field name="l10n_it_pec_email" attrs="{'invisible': [('parent_id', '!=', False)]}"/>
                <field name="l10n_it_codice_fiscale" attrs="{'invisible': [('parent_id', '!=', False)]}"/>
                <field name="l10n_it_pa_index" attrs="{'invisible': [('parent_id', '!=', False)]}"/>
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
                <field name="l10n_it_codice_fiscale" attrs="{'invisible': [('country_code', '!=', 'IT')]}"/>
                <field name="l10n_it_tax_system" attrs="{'invisible': [('country_code', '!=', 'IT')]}"/>
            </xpath>
            <xpath expr="//page" position="after">
                <page string="Electronic Invoicing" name="electronic_invoicing" attrs="{'invisible': [('country_code', '!=', 'IT')]}">
                    <group>
                        <separator string="Economic and Administrative Index" colspan="4"/>
                        <div colspan="4">
                            The seller/provider is a company listed on the register of companies and as
                            such must also indicate the registration data on all documents (art. 2250, Italian
                            Civil Code)
                        </div>
                        <group>
                            <field name="l10n_it_has_eco_index" string="Company listed on the register of companies"/>
                            <field name="l10n_it_eco_index_office" attrs="{'invisible': [('l10n_it_has_eco_index', '=', False)]}"/>
                            <field name="l10n_it_eco_index_number" attrs="{'invisible': [('l10n_it_has_eco_index', '=', False)]}"/>
                            <field name="l10n_it_eco_index_share_capital" attrs="{'invisible': [('l10n_it_has_eco_index', '=', False)]}"/>
                            <field name="l10n_it_eco_index_sole_shareholder" attrs="{'invisible': [('l10n_it_has_eco_index', '=', False)]}"/>
                            <field name="l10n_it_eco_index_liquidation_state" attrs="{'invisible': [('l10n_it_has_eco_index', '=', False)]}"/>
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
                            <field name="l10n_it_tax_representative_partner_id" attrs="{'invisible': [('l10n_it_has_tax_representative', '=', False)]}"/>
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
            </field>
        </field>
    </record>

    <record id="view_account_invoice_filter" model="ir.ui.view">
        <field name="name">account.invoice.select.inherit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account_edi.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//search/field[@name='journal_id']" position="after">
                <field name="l10n_it_edi_transaction" groups="base.group_no_one"/>
                <field name="l10n_it_edi_attachment_id" groups="base.group_no_one"/>
            </xpath>
        </field>
    </record>

    <record id="view_in_bill_tree_inherit" model="ir.ui.view">
        <field name="name">account.move.tree.inherit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_in_invoice_bill_tree" />
        <field name="arch" type="xml">
            <field name="state" position="before">
            </field>
        </field>
    </record>

    <record id="view_out_invoice_tree_inherit" model="ir.ui.view">
        <field name="name">account.move.tree.inherit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account_edi.view_out_invoice_tree_inherit" />
        <field name="arch" type="xml">
            <field name="state" position="before">
                <field name="l10n_it_edi_transaction" optional="hide" invisible="1"/>
                <field name="l10n_it_edi_attachment_id" optional="hide" invisible="1"/>
            </field>
        </field>
    </record>

    <record id="view_account_invoice_filter" model="ir.ui.view">
        <field name="name">account.invoice.select.inherit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account_edi.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//search/field[@name='journal_id']" position="after">
                <field name="l10n_it_edi_transaction" groups="base.group_no_one" invisible="1"/>
                <field name="l10n_it_edi_attachment_id" groups="base.group_no_one" invisible="1"/>
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
            <xpath expr="//page[@name='other_info']" position="after">
                <page string="Electronic Invoicing"
                    name="electronic_invoicing"
                    attrs="{'invisible': ['|', ('move_type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('country_code', '!=', 'IT')]}">
                    <group>
                        <group>
                            <field name="l10n_it_edi_transaction" groups="base.group_no_one" readonly="1"/>
                            <field name="l10n_it_stamp_duty"/>
                            <field name="l10n_it_ddt_id"
                                   attrs="{'invisible': [('move_type', 'not in', ('out_invoice', 'out_refund'))]}"/>
                        </group>
                    </group>
                </page>
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

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.proxy.user</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='account_vendor_bills']" position="after">
                <div attrs="{'invisible':[('country_code', '!=', 'IT')]}">
                    <h2>Electronic Document Invoicing</h2>
                    <div class="row mt16 o_settings_container" id='account_edi'>
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_right_pane">
                                <div class="group-content">
                                    <field name="l10n_it_edi_proxy_current_state" invisible="1"/>
                                    <span class="o_form_label">
                                        Fattura Elettronica mode
                                    </span>
                                    <div class="text-muted">
                                        In demo mode Odoo will just simulate the sending of invoices to the government.<br/>
                                        In test mode (experimental) Odoo will send the invoices to a non-production service.
                                        Saving this change will direct all companies on this database to this use this configuration.
                                        Once registered for testing or official, the mode cannot be changed.
                                    </div>
                                    <field name="l10n_it_edi_sdicoop_demo_mode"
                                           widget="radio"
                                           options="{'horizontal': true}"/>
                                </div>
                                <div class="mt8 content-group" attrs="{'invisible': ['|',('l10n_it_edi_proxy_current_state','=','active'), '&amp;', ('l10n_it_edi_proxy_current_state','=','demo'), ('l10n_it_edi_sdicoop_demo_mode','=','demo')]}">
                                    <span class="o_form_label">Allow Odoo to process invoices</span>
                                    <div class="text-muted">
                                        By checking this box, I accept that Odoo may process my invoices.
                                    </div>
                                    <div class="content-group">
                                        <field name="l10n_it_edi_sdicoop_register"/>
                                    </div>

                                </div>
                                <div class="text-success mt8" attrs="{'invisible': [('l10n_it_edi_proxy_current_state','in', ['inactive', 'demo'])]}">
                                    An Official or Test service has been registered.
                                </div>
                                <div class="text-success mt8" attrs="{'invisible': ['|',('l10n_it_edi_proxy_current_state','!=', 'demo'), ('l10n_it_edi_sdicoop_demo_mode', '!=', 'demo')]}">
                                    A Demo service is in use.
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

