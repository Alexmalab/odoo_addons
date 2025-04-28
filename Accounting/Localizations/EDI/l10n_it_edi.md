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
    'version': '0.3',
    'depends': [
        'l10n_it',
        'fetchmail',
        'account_edi'
    ],
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
                    <t t-foreach="record.invoice_line_ids.filtered(lambda l: not l.display_type)" t-as="line">
                        <t t-call="l10n_it_edi.account_invoice_line_it_simplified_FatturaPA"/>
                    </t>
                    <Allegati t-if="pdf">
                        <NomeAttachment t-esc="format_alphanumeric(pdf_name[:60])"/>
                        <FormatoAttachment>PDF</FormatoAttachment>
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
                        <CodiceTipo>EAN</CodiceTipo>
                        <CodiceValore t-esc="format_alphanumeric(line.product_id.barcode)[:35]"/>
                    </CodiceArticolo>
                    <CodiceArticolo t-elif="line.product_id.default_code">
                        <CodiceTipo>INTERNAL</CodiceTipo>
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
                    <AliquotaIVA t-if="line.tax_ids.amount_type == 'percent'" t-esc="format_numbers(line.tax_ids.amount)"/>
                    <AliquotaIVA t-elif="line.tax_ids.amount_type != 'percent'" t-esc="'0.00'"/>
                    <Natura t-if="line.tax_ids.l10n_it_has_exoneration" t-esc="line.tax_ids.l10n_it_kind_exoneration"/>
                    <AltriDatiGestionali t-if="conversion_rate">
                        <TipoDato>Currency</TipoDato>
                        <RiferimentoTesto t-esc="format_alphanumeric(record.currency_id.name)"/>
                        <RiferimentoNumero t-esc="'%.06f' % line.price_subtotal"/>
                    </AltriDatiGestionali>
                    <AltriDatiGestionali t-if="conversion_rate">
                        <TipoDato>Exch.Rate</TipoDato>
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
                <IdFiscaleIVA t-if="seller.vat and in_eu(seller)">
                    <IdPaese t-esc="get_vat_country(seller.vat)"/>
                    <IdCodice t-esc="get_vat_number(seller.vat)"/>
                </IdFiscaleIVA>
                <IdFiscaleIVA t-if="seller.vat and not in_eu(seller)">
                    <IdPaese t-esc="seller.country_id.code"/>
                    <IdCodice t-esc="'OO99999999999'"/>
                </IdFiscaleIVA>
                <IdFiscaleIVA t-if="not seller.vat and seller.country_id.code != 'IT'">
                    <IdPaese t-esc="seller.country_id.code"/>
                    <IdCodice t-esc="'0000000'"/>
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
                <IdFiscaleIVA t-if="buyer.vat and in_eu(buyer)">
                    <IdPaese t-esc="get_vat_country(buyer.vat)"/>
                    <IdCodice t-esc="get_vat_number(buyer.vat)"/>
                </IdFiscaleIVA>
                <IdFiscaleIVA t-if="buyer.vat and not in_eu(buyer)">
                    <IdPaese t-esc="buyer.country_id.code"/>
                    <IdCodice t-esc="'OO99999999999'"/>
                </IdFiscaleIVA>
                <IdFiscaleIVA t-if="not buyer.vat and buyer.country_id.code != 'IT'">
                    <IdPaese t-esc="buyer.country_id.code"/>
                    <IdCodice t-esc="'0000000'"/>
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
                    <BolloVirtuale>SI</BolloVirtuale>
                    <ImportoBollo t-esc="format_numbers(record.l10n_it_stamp_duty)"/>
                </DatiBollo>
                <ImportoTotaleDocumento t-esc="format_monetary(document_total, currency)"/>
            </DatiGeneraliDocumento>
            <DatiOrdineAcquisto t-if="record.ref">
                <IdDocumento t-esc="format_alphanumeric(record.ref[:20])"/>
            </DatiOrdineAcquisto>
            <DatiDDT t-if="record.l10n_it_ddt_id">
                <NumeroDDT t-esc="format_alphanumeric(record.l10n_it_ddt_id.name[-20:])"/>
                <DataDDT t-esc="format_date(record.l10n_it_ddt_id.date)"/>
            </DatiDDT>
        </DatiGenerali>
        <DatiBeniServizi>
            <t t-foreach="invoice_lines" t-as="line_dict">
                <t t-set="line" t-value="line_dict['line']"/>
                <t t-call="l10n_it_edi.account_invoice_line_it_FatturaPA"/>
            </t>
            <t t-foreach="tax_details['tax_details']" t-as="tax_name">
                <t t-set="tax_dict" t-value="tax_details['tax_details'][tax_name]"/>
                <t t-set="tax" t-value="tax_dict['tax']"/>
                <t t-set="has_exoneration" t-value="tax.l10n_it_has_exoneration"/>
                <t t-set="kind_exoneration" t-value="tax.l10n_it_kind_exoneration"/>
                <DatiRiepilogo>
                    <AliquotaIVA t-esc="format_numbers(tax.amount)"/>
                    <Natura t-if="has_exoneration" t-esc="kind_exoneration"/>
                    <Arrotondamento t-if="(tax_dict.get('rounding') and currency.name != 'EUR') or (tax_dict.get('rounding_euros') and currency.name == 'EUR')" t-esc="format_numbers(tax_dict['rounding'] if currency.name != 'EUR' else tax_dict['rounding_euros'])"/>
                    <ImponibileImporto t-esc="format_monetary(tax_dict['base_amount'] if currency.name == 'EUR' else tax_dict['base_amount_currency'], currency)"/>
                    <Imposta t-esc="format_monetary(tax_dict['tax_amount'] if currency.name == 'EUR' else tax_dict['base_amount_currency'], currency)"/>
                    <EsigibilitaIVA t-if="not has_exoneration or kind_exoneration == 'N6'" t-esc="tax.l10n_it_vat_due_date"/>
                    <RiferimentoNormativo t-if="tax.l10n_it_law_reference" t-esc="format_alphanumeric(tax.l10n_it_law_reference[:100])"/>
                </DatiRiepilogo>
            </t>
        </DatiBeniServizi>
        <DatiPagamento t-if="partner_bank and record.move_type != 'out_refund'">
            <t t-set="payments" t-value="record.line_ids.filtered(lambda line: line.account_id.user_type_id.type in ('receivable', 'payable'))"/>
            <CondizioniPagamento><t t-if="len(payments) == 1">TP02</t><t t-else="">TP01</t></CondizioniPagamento>
            <t t-foreach="payments" t-as="payment">
                <DettaglioPagamento>
                    <ModalitaPagamento>MP05</ModalitaPagamento>
                    <DataScadenzaPagamento t-esc="format_date(payment.date_maturity)"/>
                    <ImportoPagamento t-esc="format_monetary(abs(payment.price_total), currency)"/>
                    <IstitutoFinanziario t-if="partner_bank.bank_id" t-esc="format_alphanumeric(partner_bank.bank_id.name[:80])"/>
                    <IBAN t-if="partner_bank.acc_type == 'iban'" t-esc="partner_bank.sanitized_acc_number"/>
                    <BIC t-elif="partner_bank.acc_type == 'bank' and partner_bank.bank_id.bic" t-esc="partner_bank.bank_id.bic"/>
                    <CodicePagamento t-if="record.payment_reference" t-esc="format_alphanumeric(record.payment_reference[:60])"/>
                </DettaglioPagamento>
            </t>
        </DatiPagamento>
        <Allegati t-if="pdf">
            <NomeAttachment t-esc="format_alphanumeric(pdf_name[:60])"/>
            <FormatoAttachment>PDF</FormatoAttachment>
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

    </data>
</odoo>


```

## File: models\account_chart_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountChartTemplate(models.Model):

    _inherit = 'account.chart.template'

    def _load(self, sale_tax_rate, purchase_tax_rate, company):
        res = super()._load(sale_tax_rate, purchase_tax_rate, company)
        if self == self.env.ref('l10n_it.l10n_it_chart_template_generic', raise_if_not_found=False):
            tax = self.env.ref(f'l10n_it.{company.id}_00eu', raise_if_not_found=False)
            if tax:
                tax.write({
                    'l10n_it_has_exoneration': True,
                    'l10n_it_kind_exoneration': 'N3.2',
                    'l10n_it_law_reference': 'Art. 41, DL n. 331/93',
                })
        return res

```

## File: models\account_edi_format.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields, _
from odoo.tests.common import Form
from odoo.exceptions import UserError
from odoo.addons.l10n_it_edi.tools.remove_signature import remove_signature
from odoo.osv.expression import OR, AND

from lxml import etree
from datetime import datetime
import re
import logging


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
        report_lines_xmlids = invoice.line_ids.tax_tag_ids.tax_report_line_ids.mapped(lambda x: x.get_external_id().get(x.id, ''))
        return (invoice.is_purchase_document()
                and any([x.startswith("l10n_it.tax_report_line_vj") for x in report_lines_xmlids]))

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

        # <2.2.1>
        for invoice_line in invoice.invoice_line_ids:
            if not invoice_line.display_type and len(invoice_line.tax_ids) != 1:
                errors.append(_("In line %s, you must select one and only one tax by line.", invoice_line.name))

        for tax_line in invoice.line_ids.filtered(lambda line: line.tax_line_id):
            if not tax_line.tax_line_id.l10n_it_kind_exoneration and tax_line.tax_line_id.amount == 0:
                errors.append(_("%s has an amount of 0.0, you must indicate the kind of exoneration.", tax_line.name))

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
        for line in invoice.invoice_line_ids.filtered(lambda l: not l.display_type):
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

        return errors

    def _l10n_it_goods_in_italy(self, invoice):
        """
            There is a specific TipoDocumento (Document Type TD19) and tax grid (VJ3) for goods
            that are phisically in Italy but are in a VAT deposit, meaning that the goods
            have not passed customs.
        """
        report_lines_xmlids = invoice.line_ids.tax_tag_ids.tax_report_line_ids.mapped(lambda x: x.get_external_id().get(x.id, ''))
        return any([x == "l10n_it.tax_report_line_vj3" for x in report_lines_xmlids])

    def _l10n_it_document_type_mapping(self):
        return {
            'TD01': dict(move_types=['out_invoice'], import_type='in_invoice', downpayment=False),
            'TD02': dict(move_types=['out_invoice'], import_type='in_invoice', downpayment=True),
            'TD04': dict(move_types=['out_refund'], import_type='in_refund'),
            'TD07': dict(move_types=['out_invoice'], import_type='in_invoice', simplified=True),
            'TD08': dict(move_types=['out_refund'], import_type='in_refund', simplified=True),
            'TD09': dict(move_types=['out_invoice'], import_type='in_invoice', simplified=True),
            'TD17': dict(move_types=['in_invoice', 'in_refund'], import_type='in_invoice', self_invoice=True, services_or_goods="service"),
            'TD18': dict(move_types=['in_invoice', 'in_refund'], import_type='in_invoice', self_invoice=True, services_or_goods="consu", partner_in_eu=True),
            'TD19': dict(move_types=['in_invoice', 'in_refund'], import_type='in_invoice', self_invoice=True, services_or_goods="consu", goods_in_italy=True),
        }

    def _l10n_it_get_document_type(self, invoice):
        is_simplified = self._l10n_it_edi_is_simplified(invoice)
        is_self_invoice = self._l10n_it_edi_is_self_invoice(invoice)
        services_or_goods = self._l10n_it_edi_services_or_goods(invoice)
        goods_in_italy = services_or_goods == 'consu' and self._l10n_it_goods_in_italy(invoice)
        partner_in_eu = self._l10n_it_edi_partner_in_eu(invoice.commercial_partner_id)
        for code, infos in self._l10n_it_document_type_mapping().items():
            info_services_or_goods = infos.get('services_or_goods', "both")
            info_partner_in_eu = infos.get('partner_in_eu', False)
            if all([
                invoice.move_type in infos.get('move_types', False),
                # Only check downpayment if the key is specified in the document_type_mapping entry
                # If it's not specified, the get() will return None and the condition will be True
                infos.get('downpayment') in (None, invoice._is_downpayment()),
                is_self_invoice == infos.get('self_invoice', False),
                is_simplified == infos.get('simplified', False),
                info_services_or_goods in ("both", services_or_goods),
                info_partner_in_eu in (False, partner_in_eu),
                goods_in_italy == infos.get('goods_in_italy', False),
            ]):
                return code

        return None

    def _l10n_it_is_simplified_document_type(self, document_type):
        return self._l10n_it_document_type_mapping().get(document_type, {}).get('simplified', False)

    # -------------------------------------------------------------------------
    # Export
    # -------------------------------------------------------------------------

    def _is_embedding_to_invoice_pdf_needed(self):
        # OVERRIDE
        self.ensure_one()
        return True if self.code == 'fattura_pa' else super()._is_embedding_to_invoice_pdf_needed()

    def _is_compatible_with_journal(self, journal):
        # OVERRIDE
        self.ensure_one()
        if self.code != 'fattura_pa':
            return super()._is_compatible_with_journal(journal)
        return journal.type in ('sale', 'purchase') and journal.country_code == 'IT'

    def _l10n_it_edi_is_required_for_invoice(self, invoice):
        """ Is the edi required for this invoice based on the method (here: PEC mail)
            Deprecated: in future release PEC mail will be removed.
            TO OVERRIDE
        """
        return ((invoice.is_sale_document() or self._l10n_it_get_document_type(invoice))
                and invoice.country_code == 'IT'
                and invoice.l10n_it_send_state not in ('sent', 'delivered', 'delivered_accepted'))

    def _is_required_for_invoice(self, invoice):
        # OVERRIDE
        self.ensure_one()
        if self.code != 'fattura_pa':
            return super()._is_required_for_invoice(invoice)

        return self._l10n_it_edi_is_required_for_invoice(invoice)

    def _post_fattura_pa(self, invoices):
        # TO OVERRIDE
        invoice = invoices  # no batching ensure that we only have one invoice
        invoice.l10n_it_send_state = 'other'
        invoice._check_before_xml_exporting()
        if invoice.l10n_it_einvoice_id and invoice.l10n_it_send_state not in ['invalid', 'to_send']:
            return {'error': _("You can't regenerate an E-Invoice when the first one is sent and there are no errors")}
        if invoice.l10n_it_einvoice_id:
            invoice.l10n_it_einvoice_id.unlink()
        res = invoice.invoice_generate_xml()
        if invoice._is_commercial_partner_pa():
            invoice.message_post(
                body=(_("Invoices for PA are not managed by Odoo, you can download the document and send it on your own."))
            )
        else:
            invoice.l10n_it_send_state = 'to_send'
        return {invoice: res}

    def _post_invoice_edi(self, invoices, test_mode=False):
        # OVERRIDE
        self.ensure_one()
        edi_result = super()._post_invoice_edi(invoices, test_mode=test_mode)
        if self.code != 'fattura_pa':
            return edi_result

        return self._post_fattura_pa(invoices)

    # -------------------------------------------------------------------------
    # Import
    # -------------------------------------------------------------------------

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
        decoded_content = remove_signature(content)
        if not decoded_content:
            return None

        try:
            # Some malformed XML are accepted by FatturaPA, this expends compatibility
            parser = etree.XMLParser(recover=True)
            xml_tree = etree.fromstring(decoded_content, parser)
        except Exception as e:
            _logger.exception("Error when converting the xml content to etree: %s", e)
            return None
        if not len(xml_tree):
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

    def _import_fattura_pa(self, tree, invoice):
        """ Decodes a fattura_pa invoice into an invoice.

        :param tree:    the fattura_pa tree to decode.
        :param invoice: the invoice to update or an empty recordset.
        :returns:       the invoice where the fattura_pa data was imported.
        """
        invoices = self.env['account.move']
        first_run = True

        # possible to have multiple invoices in the case of an invoice batch, the batch itself is repeated for every invoice of the batch
        for body_tree in tree.xpath('//FatturaElettronicaBody'):
            if not first_run or not invoice:
                # make sure all the iterations create a new invoice record (except the first which could have already created one)
                invoice = self.env['account.move']
            first_run = False

            # Type must be present in the context to get the right behavior of the _default_journal method (account.move).
            # journal_id must be present in the context to get the right behavior of the _default_account method (account.move.line).
            elements = tree.xpath('//CessionarioCommittente//IdCodice')
            company = elements and self.env['res.company'].search([('vat', 'ilike', elements[0].text)], limit=1)
            if not company:
                elements = tree.xpath('//CessionarioCommittente//CodiceFiscale')
                company = elements and self.env['res.company'].search([('l10n_it_codice_fiscale', 'ilike', elements[0].text)], limit=1)
                if not company:
                    # Only invoices with a correct VAT or Codice Fiscale can be imported
                    _logger.warning('No company found with VAT or Codice Fiscale like %r.', elements[0].text)
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

            simplified = self._l10n_it_is_simplified_document_type(document_type)

            # Setup the context for the Invoice Form
            invoice_ctx = invoice.with_company(company) \
                                 .with_context(default_move_type=move_type)

            # move could be a single record (editing) or be empty (new).
            with Form(invoice_ctx) as invoice_form:
                message_to_log = []

                # Partner (first step to avoid warning 'Warning! You must first select a partner.'). <1.2>
                elements = tree.xpath('//CedentePrestatore//IdCodice')
                partner = elements and self.env['res.partner'].search(['&', ('vat', 'ilike', elements[0].text), '|', ('company_id', '=', company.id), ('company_id', '=', False)], limit=1)
                if not partner:
                    elements = tree.xpath('//CedentePrestatore//CodiceFiscale')
                    if elements:
                        codice = elements[0].text
                        domains = [[('l10n_it_codice_fiscale', '=', codice)]]
                        if re.match(r'^[0-9]{11}$', codice):
                            domains.append([('l10n_it_codice_fiscale', '=', 'IT' + codice)])
                        elif re.match(r'^IT[0-9]{11}$', codice):
                            domains.append([('l10n_it_codice_fiscale', '=', self.env['res.partner']._l10n_it_normalize_codice_fiscale(codice))])
                        partner = elements and self.env['res.partner'].search(
                            AND([OR(domains), OR([[('company_id', '=', company.id)], [('company_id', '=', False)]])]), limit=1)
                if not partner:
                    elements = tree.xpath('//DatiTrasmissione//Email')
                    partner = elements and self.env['res.partner'].search(['&', '|', ('email', '=', elements[0].text), ('l10n_it_pec_email', '=', elements[0].text), '|', ('company_id', '=', company.id), ('company_id', '=', False)], limit=1)
                if partner:
                    invoice_form.partner_id = partner
                else:
                    message_to_log.append("%s<br/>%s" % (
                        _("Vendor not found, useful informations from XML file:"),
                        invoice._compose_info_message(
                            tree, './/CedentePrestatore')))

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
                    invoice_form.narration = '%s%s\n' % (invoice_form.narration or '', element.text)

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
                if not simplified:
                    elements = body_tree.xpath('.//DettaglioLinee')
                else:
                    elements = body_tree.xpath('.//DatiBeniServizi')

                if elements:
                    for element in elements:
                        with invoice_form.invoice_line_ids.new() as invoice_line_form:

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
                                        product_supplier = self.env['product.supplierinfo'].search([('name', '=', partner.id), ('product_code', '=', code.text)], limit=2)
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
                            if not simplified:
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
                            invoice_line_form.tax_ids.clear()
                            if percentage is not None:
                                if natura_element and natura_element[0].text:
                                    l10n_it_kind_exoneration = natura_element[0].text
                                    tax = self.env['account.tax'].search([
                                        ('company_id', '=', invoice_form.company_id.id),
                                        ('amount_type', '=', 'percent'),
                                        ('type_tax_use', '=', 'purchase'),
                                        ('amount', '=', percentage),
                                        ('l10n_it_kind_exoneration', '=', l10n_it_kind_exoneration),
                                    ], limit=1)
                                else:
                                    tax = self.env['account.tax'].search([
                                        ('company_id', '=', invoice_form.company_id.id),
                                        ('amount_type', '=', 'percent'),
                                        ('type_tax_use', '=', 'purchase'),
                                        ('amount', '=', percentage),
                                    ], limit=1)
                                    l10n_it_kind_exoneration = ''

                                if tax:
                                    invoice_line_form.tax_ids.add(tax)
                                else:
                                    if l10n_it_kind_exoneration:
                                        message_to_log.append(_("Tax not found with percentage: %s and exoneration %s for the article: %s") % (
                                            percentage,
                                            l10n_it_kind_exoneration,
                                            invoice_line_form.name))
                                    else:
                                        message_to_log.append(_("Tax not found with percentage: %s for the article: %s") % (
                                            percentage,
                                            invoice_line_form.name))

                            # Price Unit.
                            if not simplified:
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


                # Global discount summarized in 1 amount
                discount_elements = body_tree.xpath('.//DatiGeneraliDocumento/ScontoMaggiorazione')
                if discount_elements:
                    taxable_amount = invoice_form.amount_untaxed
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

                    with invoice_form.invoice_line_ids.new() as invoice_line_global_discount:
                        invoice_line_global_discount.tax_ids.clear()
                        invoice_line_global_discount.sequence = sequence
                        invoice_line_global_discount.name = 'SCONTO' if general_discount < 0 else 'MAGGIORAZIONE'
                        invoice_line_global_discount.price_unit = general_discount

            new_invoice = invoice_form.save()
            new_invoice.l10n_it_send_state = "other"

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

                    # default_res_id is had to context to avoid facturx to import his content
                    # no_new_invoice to prevent from looping on the message_post that would create a new invoice without it
                    new_invoice.with_context(no_new_invoice=True, default_res_id=new_invoice.id).message_post(
                        body=(_("Attachment from XML")),
                        attachment_ids=[attachment_64.id]
                    )

            for message in message_to_log:
                new_invoice.message_post(body=message)

            invoices += new_invoice

        return invoices

```

## File: models\account_invoice.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import zipfile
import io
import logging
import re

from datetime import date, datetime
from lxml import etree

from odoo import api, fields, models, _
from odoo.tools import float_repr, float_compare
from odoo.exceptions import UserError, ValidationError
from odoo.addons.base.models.ir_mail_server import MailDeliveryException
from odoo.tests.common import Form


_logger = logging.getLogger(__name__)

DEFAULT_FACTUR_ITALIAN_DATE_FORMAT = '%Y-%m-%d'


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_it_send_state = fields.Selection([
        ('new', 'New'),
        ('other', 'Other'),
        ('to_send', 'Not yet send'),
        ('sent', 'Sent, waiting for response'),
        ('invalid', 'Sent, but invalid'),
        ('delivered', 'This invoice is delivered'),
        ('delivered_accepted', 'This invoice is delivered and accepted by destinatory'),
        ('delivered_refused', 'This invoice is delivered and refused by destinatory'),
        ('delivered_expired', 'This invoice is delivered and expired (expiry of the maximum term for communication of acceptance/refusal)'),
        ('failed_delivery', 'Delivery impossible, ES certify that it has received the invoice and that the file \
                        could not be delivered to the addressee') # ok we must do nothing
    ], default='to_send', copy=False, string="FatturaPA Send State")

    l10n_it_stamp_duty = fields.Float(default=0, string="Dati Bollo", readonly=True, states={'draft': [('readonly', False)]})

    l10n_it_ddt_id = fields.Many2one('l10n_it.ddt', string='DDT', readonly=True, states={'draft': [('readonly', False)]}, copy=False)

    l10n_it_einvoice_name = fields.Char(compute='_compute_l10n_it_einvoice')

    l10n_it_einvoice_id = fields.Many2one('ir.attachment', string="Electronic invoice", compute='_compute_l10n_it_einvoice')

    @api.depends('edi_document_ids', 'edi_document_ids.attachment_id')
    def _compute_l10n_it_einvoice(self):
        fattura_pa = self.env.ref('l10n_it_edi.edi_fatturaPA')
        for invoice in self:
            einvoice = invoice.edi_document_ids.filtered(lambda d: d.edi_format_id == fattura_pa)
            invoice.l10n_it_einvoice_id = einvoice.attachment_id
            invoice.l10n_it_einvoice_name = einvoice.attachment_id.name

    def _check_before_xml_exporting(self):
        # DEPRECATED use AccountEdiFormat._l10n_it_edi_check_invoice_configuration instead
        errors = self.env['account.edi.format']._l10n_it_edi_check_invoice_configuration(self)
        if errors:
            raise UserError(self.env['account.edi.format']._format_error_message(_("Invalid configuration:"), errors))

    def invoice_generate_xml(self):
        self.ensure_one()
        report_name = self.env['account.edi.format']._l10n_it_edi_generate_electronic_invoice_filename(self)

        data = b"<?xml version='1.0' encoding='UTF-8'?>" + self._export_as_xml()
        description = _('Italian invoice: %s', self.move_type)
        attachment = self.env['ir.attachment'].create({
            'name': report_name,
            'res_id': self.id,
            'res_model': self._name,
            'datas': base64.encodebytes(data),
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
        return len(self.commercial_partner_id.l10n_it_pa_index or '')  == 6

    def _l10n_it_edi_prepare_fatturapa_line_details(self, reverse_charge_refund=False, is_downpayment=False, convert_to_euros=True):
        """ Returns a list of dictionaries passed to the template for the invoice lines (DettaglioLinee)
        """
        invoice_lines = []
        lines = self.invoice_line_ids.filtered(lambda l: not l.display_type)

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
            if not is_downpayment:
                if line.price_subtotal < 0:
                    moves = line._get_downpayment_lines().move_id
                    if moves:
                        description += ', '.join([move.name for move in moves])

            line_dict = {
                'line': line,
                'line_number': num + 1,
                'description': description or 'NO NAME',
                'unit_price': price_unit,
                'subtotal_price': price_subtotal,
            }
            invoice_lines.append(line_dict)
        return invoice_lines

    def _l10n_it_edi_prepare_fatturapa_tax_details(self, tax_details, reverse_charge_refund=False):
        """ Returns an adapted dictionary passed to the template for the tax lines (DatiRiepilogo)
        """
        for _tax_name, tax_dict in tax_details['tax_details'].items():
            # The assumption is that the company currency is EUR.
            base_amount = tax_dict['base_amount']
            base_amount_currency = tax_dict['base_amount_currency']
            tax_amount = tax_dict['tax_amount']
            tax_amount_currency = tax_dict['tax_amount_currency']
            tax_rate = tax_dict['tax'].amount
            expected_base_amount_currency = tax_amount_currency * 100 / tax_rate if tax_rate else False
            expected_base_amount = tax_amount * 100 / tax_rate if tax_rate else False
            # Constraints within the edi make local rounding on price included taxes a problem.
            # To solve this there is a <Arrotondamento> or 'rounding' field, such that:
            #   taxable base = sum(taxable base for each unit) + Arrotondamento
            if tax_dict['tax'].price_include and tax_dict['tax'].amount_type == 'percent':
                if expected_base_amount_currency and float_compare(base_amount_currency, expected_base_amount_currency, 2):
                    tax_dict['rounding'] = base_amount_currency - (tax_amount_currency * 100 / tax_rate)
                    tax_dict['base_amount_currency'] = base_amount_currency - tax_dict['rounding']
                if expected_base_amount and float_compare(base_amount, expected_base_amount, 2):
                    tax_dict['rounding_euros'] = base_amount - (tax_amount * 100 / tax_rate)
                    tax_dict['base_amount'] = base_amount - tax_dict['rounding_euros']

            if not reverse_charge_refund:
                balance_multiplicator = -1 if self.is_inbound() else 1
                if tax_dict['base_amount'] != 0:  # We shouldn't change 0 into -0
                    tax_dict['base_amount'] *= balance_multiplicator
                if tax_dict['base_amount_currency'] != 0:
                    tax_dict['base_amount_currency'] *= balance_multiplicator
                if tax_dict['tax_amount'] != 0:
                    tax_dict['tax_amount'] *= balance_multiplicator
                if tax_dict['tax_amount_currency'] != 0:
                    tax_dict['tax_amount_currency'] *= balance_multiplicator
        return tax_details

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

        formato_trasmissione = "FPA12" if self._is_commercial_partner_pa() else "FPR12"

        # Flags
        in_eu = self.env['account.edi.format']._l10n_it_edi_partner_in_eu
        is_self_invoice = self.env['account.edi.format']._l10n_it_edi_is_self_invoice(self)
        document_type = self.env['account.edi.format']._l10n_it_get_document_type(self)
        if self.env['account.edi.format']._l10n_it_is_simplified_document_type(document_type):
            formato_trasmissione = "FSM10"

        document_type = self.env['account.edi.format']._l10n_it_get_document_type(self)
        # Represent if the document is a reverse charge refund in a single variable
        reverse_charge = document_type in ['TD17', 'TD18', 'TD19']
        is_downpayment = document_type in ['TD02']
        reverse_charge_refund = self.move_type == 'in_refund' and reverse_charge
        convert_to_euros = self.currency_id.name != 'EUR'

        pdf = self.env.ref('account.account_invoices')._render_qweb_pdf(self.id)[0]
        pdf = base64.b64encode(pdf)
        pdf_name = re.sub(r'\W+', '', self.name) + '.pdf'

        # tax map for 0% taxes which have no tax_line_id
        tax_map = dict()
        for line in self.line_ids:
            for tax in line.tax_ids:
                if tax.amount == 0.0:
                    tax_map[tax] = tax_map.get(tax, 0.0) + line.price_subtotal

        tax_details = self._prepare_edi_tax_details(
            filter_to_apply=lambda l: l['tax_repartition_line_id'].factor_percent >= 0
        )

        company = self.company_id
        partner = self.commercial_partner_id
        buyer = partner if not is_self_invoice else company
        seller = company if not is_self_invoice else partner
        codice_destinatario = (
            (is_self_invoice and company.partner_id.l10n_it_pa_index)
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

        # Reference line for finding the conversion rate used in the document
        conversion_line = self.invoice_line_ids.sorted(lambda l: abs(l.balance), reverse=True)[0] if self.invoice_line_ids else None
        conversion_rate = float_repr(
            abs(conversion_line.balance / conversion_line.amount_currency), precision_digits=5,
        ) if convert_to_euros and conversion_line else None

        invoice_lines = self._l10n_it_edi_prepare_fatturapa_line_details(reverse_charge_refund, is_downpayment, convert_to_euros)
        tax_details = self._l10n_it_edi_prepare_fatturapa_tax_details(tax_details, reverse_charge_refund)

        # Create file content.
        template_values = {
            'record': self,
            'company': company,
            'sender': company,
            'sender_partner': company.partner_id,
            'partner': partner,
            'buyer': buyer,
            'buyer_partner': partner if not is_self_invoice else company.partner_id,
            'buyer_is_company': is_self_invoice or partner.is_company,
            'seller': seller,
            'seller_partner': company.partner_id if not is_self_invoice else partner,
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
            'tax_map': tax_map,
            'tax_details': tax_details,
            'abs': abs,
            'normalize_codice_fiscale': partner._l10n_it_normalize_codice_fiscale,
            'get_vat_number': get_vat_number,
            'get_vat_country': get_vat_country,
            'in_eu': in_eu,
            'rc_refund': reverse_charge_refund,
            'invoice_lines': invoice_lines,
            'conversion_rate': conversion_rate,
        }
        return template_values

    def _export_as_xml(self):
        '''DEPRECATED : this will be moved to AccountEdiFormat in a future version.
        Create the xml file content.
        :return: The XML content as str.
        '''
        template_values = self._prepare_fatturapa_export_values()
        if not self.env['account.edi.format']._l10n_it_is_simplified_document_type(template_values['document_type']):
            content = self.env.ref('l10n_it_edi.account_invoice_it_FatturaPA_export')._render(template_values)
        else:
            content = self.env.ref('l10n_it_edi.account_invoice_it_simplified_FatturaPA_export')._render(template_values)
            self.message_post(body=_("A simplified invoice was created instead of an ordinary one. This is because the invoice \
                                    is a domestic invoice with a total amount of less than or equal to 400€ and the customer's address is incomplete."))
        return content

    def _post(self, soft=True):
        # OVERRIDE
        posted = super()._post(soft=soft)

        for move in posted.filtered(lambda m: m.l10n_it_send_state == 'to_send' and m.move_type in ['out_invoice', 'out_refund'] and m.company_id.country_id.code == 'IT'):
            move.send_pec_mail()

        return posted

    def send_pec_mail(self):
        self.ensure_one()
        allowed_state = ['to_send', 'invalid']

        if (
            not self.company_id.l10n_it_mail_pec_server_id
            or not self.company_id.l10n_it_mail_pec_server_id.active
            or not self.company_id.l10n_it_address_send_fatturapa
        ):
            self.message_post(
                body=(_("Error when sending mail with E-Invoice: Your company must have a mail PEC server and must indicate the mail PEC that will send electronic invoice."))
                )
            self.l10n_it_send_state = 'invalid'
            return

        if self.l10n_it_send_state not in allowed_state:
            raise UserError(_("%s isn't in a right state. It must be in a 'Not yet send' or 'Invalid' state.") % (self.display_name))

        message = self.env['mail.message'].create({
            'subject': _('Sending file: %s') % (self.l10n_it_einvoice_name),
            'body': _('Sending file: %s to ES: %s') % (self.l10n_it_einvoice_name, self.env.company.l10n_it_address_recipient_fatturapa),
            'author_id': self.env.user.partner_id.id,
            'email_from': self.env.company.l10n_it_address_send_fatturapa,
            'reply_to': self.env.company.l10n_it_address_send_fatturapa,
            'mail_server_id': self.env.company.l10n_it_mail_pec_server_id.id,
            'attachment_ids': [(6, 0, self.l10n_it_einvoice_id.ids)],
        })

        mail_fattura = self.env['mail.mail'].sudo().with_context(wo_bounce_return_path=True).create({
            'mail_message_id': message.id,
            'email_to': self.env.company.l10n_it_address_recipient_fatturapa,
        })
        try:
            mail_fattura.send(raise_exception=True)
            self.message_post(
                body=(_("Mail sent on %s by %s") % (fields.Datetime.now(), self.env.user.display_name))
                )
            self.l10n_it_send_state = 'sent'
        except MailDeliveryException as error:
            self.message_post(
                body=(_("Error when sending mail with E-Invoice: %s") % (error.args[0]))
                )
            self.l10n_it_send_state = 'invalid'

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
            ("N7", "[N7] IVA assolta in altro stato UE (vendite a distanza ex art. 40 c. 3 e 4 e art. 41 c. 1 lett. b,  DL 331/93; prestazione di servizi di telecomunicazioni, tele-radiodiffusione ed elettronici ex art. 7-sexies lett. f, g, art. 74-sexies DPR 633/72)")],
        string="Exoneration",
        help="Exoneration type",
        default="N1")
    l10n_it_law_reference = fields.Char(string="Law Reference", size=100)

    @api.constrains('l10n_it_has_exoneration',
                    'l10n_it_kind_exoneration',
                    'l10n_it_law_reference',
                    'amount',
                    'l10n_it_vat_due_date')
    def _check_exoneration_with_no_tax(self):
        for tax in self:
            if tax.l10n_it_has_exoneration:
                if not tax.l10n_it_kind_exoneration or not tax.l10n_it_law_reference or tax.amount != 0:
                    raise ValidationError(_("If the tax has exoneration, you must enter a kind of exoneration, a law reference and the amount of the tax must be 0.0."))
                if tax.l10n_it_kind_exoneration == 'N6' and tax.l10n_it_vat_due_date == 'S':
                    raise UserError(_("'Scissione dei pagamenti' is not compatible with exoneration of kind 'N6'"))

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

## File: models\ir_mail_server.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import zipfile
import io
import re
import logging
import email
import email.policy
import dateutil
import pytz

from lxml import etree
from datetime import datetime
from xmlrpc import client as xmlrpclib

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError, UserError
from odoo.addons.l10n_it_edi.tools.remove_signature import remove_signature


_logger = logging.getLogger(__name__)

class FetchmailServer(models.Model):
    _name = 'fetchmail.server'
    _inherit = 'fetchmail.server'

    l10n_it_is_pec = fields.Boolean('PEC server', help="If PEC Server, only mail from '...@pec.fatturapa.it' will be processed.")
    l10n_it_last_uid = fields.Integer(string='Last message UID', default=1)

    def _search_edi_invoice(self, att_name, send_state=False):
        """ Search sent l10n_it_edi fatturaPA invoices """

        conditions = [
            ('move_id', "!=", False),
            ('edi_format_id.code', '=', 'fattura_pa'),
            ('attachment_id.name', '=', att_name),
        ] 
        if send_state:
            conditions.append(('move_id.l10n_it_send_state', '=', send_state))

        return self.env['account.edi.document'].search(conditions, limit=1).move_id

    @api.constrains('l10n_it_is_pec', 'server_type')
    def _check_pec(self):
        for record in self:
            if record.l10n_it_is_pec and record.server_type != 'imap':
                raise ValidationError(_("PEC mail server must be of type IMAP."))

    def fetch_mail(self):
        """ WARNING: meant for cron usage only - will commit() after each email! """

        MailThread = self.env['mail.thread']
        for server in self.filtered(lambda s: s.l10n_it_is_pec):
            _logger.info('start checking for new emails on %s PEC server %s', server.server_type, server.name)

            count, failed = 0, 0
            imap_server = None
            try:
                imap_server = server.connect()
                imap_server.select()

                # Only download new emails
                email_filter = ['(UID %s:*)' % (server.l10n_it_last_uid)]

                # The l10n_it_edi.fatturapa_bypass_incoming_address_filter prevents the sender address check on incoming email.
                bypass_incoming_address_filter = self.env['ir.config_parameter'].get_param('l10n_it_edi.bypass_incoming_address_filter', False)
                if not bypass_incoming_address_filter:
                    email_filter.append('(FROM "@pec.fatturapa.it")')

                data = imap_server.uid('search', None, *email_filter)[1]

                new_max_uid = server.l10n_it_last_uid
                for uid in data[0].split():
                    if int(uid) <= server.l10n_it_last_uid:
                        # We get always minimum 1 message.  If no new message, we receive the newest already managed.
                        continue

                    result, data = imap_server.uid('fetch', uid, '(RFC822)')

                    if not data[0]:
                        continue
                    message = data[0][1]

                    # To leave the mail in the state in which they were.
                    if "Seen" not in data[1].decode("utf-8"):
                        imap_server.uid('STORE', uid, '+FLAGS', '(\\Seen)')
                    else:
                        imap_server.uid('STORE', uid, '-FLAGS', '(\\Seen)')

                    # See details in message_process() in mail_thread.py
                    if isinstance(message, xmlrpclib.Binary):
                        message = bytes(message.data)
                    if isinstance(message, str):
                        message = message.encode('utf-8')
                    msg_txt = email.message_from_bytes(message, policy=email.policy.SMTP)

                    try:
                        self._attachment_invoice(msg_txt)
                        new_max_uid = max(new_max_uid, int(uid))
                    except Exception:
                        _logger.info('Failed to process mail from %s server %s.', server.server_type, server.name, exc_info=True)
                        failed += 1
                    self._cr.commit()
                    count += 1
                server.write({'l10n_it_last_uid': new_max_uid})
                _logger.info("Fetched %d email(s) on %s server %s; %d succeeded, %d failed.", count, server.server_type, server.name, (count - failed), failed)
            except Exception:
                _logger.info("General failure when trying to fetch mail from %s server %s.", server.server_type, server.name, exc_info=True)
            finally:
                if imap_server:
                    imap_server.close()
                    imap_server.logout()
                server.write({'date': fields.Datetime.now()})
        return super(FetchmailServer, self.filtered(lambda s: not s.l10n_it_is_pec)).fetch_mail()

    def _attachment_invoice(self, msg_txt):
        parsed_values = self.env['mail.thread']._message_parse_extract_payload(msg_txt)
        body, attachments = parsed_values['body'], parsed_values['attachments']
        from_address = msg_txt.get('from')
        for attachment in attachments:
            split_attachment = attachment.fname.rpartition('.')
            if len(split_attachment) < 3:
                _logger.info('E-invoice filename not compliant: %s', attachment.fname)
                continue
            attachment_name = split_attachment[0]
            attachment_ext = split_attachment[2]
            split_underscore = attachment_name.rsplit('_', 2)
            if len(split_underscore) < 2:
                _logger.info('E-invoice filename not compliant: %s', attachment.fname)
                continue

            if attachment_ext != 'zip':
                if split_underscore[1] in ['RC', 'NS', 'MC', 'MT', 'EC', 'SE', 'NE', 'DT']:
                    # we have a receipt
                    self._message_receipt_invoice(split_underscore[1], attachment)
                else:
                    att_filename = attachment.fname
                    match = re.search("[A-Z]{2}[A-Za-z0-9]{2,28}_[A-Za-z0-9]{0,5}.((?i:xml.p7m|xml))", att_filename)
                    # If match, we have an invoice.
                    if match:
                        # If it's signed, the content has a bytes type and we just remove the signature's envelope
                        if match.groups()[0].lower() == 'xml.p7m':
                            att_content_data = remove_signature(attachment.content)
                            # If the envelope cannot be removed, the remove_signature returns None, so we skip
                            if not att_content_data:
                                _logger.warning("E-invoice couldn't be read: %s", att_filename)
                                continue
                            att_filename = att_filename.replace('.xml.p7m', '.xml')
                        else:
                            # Otherwise, it should be an utf-8 encoded XML string
                            att_content_data = attachment.content.encode()
                    self._create_invoice_from_mail(att_content_data, att_filename, from_address)
            else:
                if split_underscore[1] == 'AT':
                    # Attestazione di avvenuta trasmissione della fattura con impossibilità di recapito
                    self._message_AT_invoice(attachment)
                else:
                    _logger.info('New E-invoice in zip file: %s', attachment.fname)
                    self._create_invoice_from_mail_with_zip(attachment, from_address)

    def _create_invoice_from_mail(self, att_content_data, att_name, from_address):
        """ Creates an invoice from the content of an email present in ir.attachments

        :param att_content_data:   The 'utf-8' encoded bytes string representing the content of the attachment.
        :param att_name:           The attachment's file name.
        :param from_address:       The sender address of the email.
        """

        invoices = self.env['account.move']

        # Check if we already imported the email as an attachment
        existing = self.env['ir.attachment'].search([('name', '=', att_name), ('res_model', '=', 'account.move')])
        if existing:
            _logger.info('E-invoice already exist: %s', att_name)
            return invoices

        # Create the new attachment for the file
        attachment = self.env['ir.attachment'].create({
            'name': att_name,
            'raw': att_content_data,
            'res_model': 'account.move',
            'type': 'binary'})

        # Decode the file.
        try:
            tree = etree.fromstring(att_content_data)
        except Exception:
            _logger.info('The xml file is badly formatted: %s', att_name)
            return invoices

        invoices = self.env.ref('l10n_it_edi.edi_fatturaPA')._create_invoice_from_xml_tree(att_name, tree)
        if not invoices:
            _logger.info('E-invoice not found in file: %s', att_name)
            return invoices
        invoices.l10n_it_send_state = "new"
        invoices.invoice_source_email = from_address
        for invoice in invoices:
            invoice.with_context(no_new_invoice=True, default_res_id=invoice.id) \
                    .message_post(body=(_("Original E-invoice XML file")), attachment_ids=[attachment.id])

        self._cr.commit()

        _logger.info('New E-invoices (%s), ids: %s', att_name, [x.id for x in invoices])
        return invoices

    def _create_invoice_from_mail_with_zip(self, attachment_zip, from_address):
        with zipfile.ZipFile(io.BytesIO(attachment_zip.content)) as z:
            for att_name in z.namelist():
                existing = self.env['ir.attachment'].search([('name', '=', att_name), ('res_model', '=', 'account.move')])
                if existing:
                    # invoice already exist
                    _logger.info('E-invoice in zip file (%s) already exist: %s', attachment_zip.fname, att_name)
                    continue
                att_content = z.open(att_name).read()

                self._create_invoice_from_mail(att_content, att_name, from_address)

    def _message_AT_invoice(self, attachment_zip):
        with zipfile.ZipFile(io.BytesIO(attachment_zip.content)) as z:
            for attachment_name in z.namelist():
                split_name_attachment = attachment_name.rpartition('.')
                if len(split_name_attachment) < 3:
                    continue
                split_underscore = split_name_attachment[0].rsplit('_', 2)
                if len(split_underscore) < 2:
                    continue
                if split_underscore[1] == 'AT':
                    attachment = z.open(attachment_name).read()
                    _logger.info('New AT receipt for: %s', split_underscore[0])
                    try:
                        tree = etree.fromstring(attachment)
                    except:
                        _logger.info('Error in decoding new receipt file: %s', attachment_name)
                        return

                    elements = tree.xpath('//NomeFile')
                    if elements and elements[0].text:
                        filename = elements[0].text
                    else:
                        return

                    related_invoice = self._search_edi_invoice(filename)
                    if not related_invoice:
                        _logger.info('Error: invoice not found for receipt file: %s', filename)
                        return

                    related_invoice.l10n_it_send_state = 'failed_delivery'
                    info = self._return_multi_line_xml(tree, ['//IdentificativoSdI', '//DataOraRicezione', '//MessageId', '//PecMessageId', '//Note'])
                    related_invoice.message_post(
                        body=(_("ES certify that it has received the invoice and that the file \
                        could not be delivered to the addressee. <br/>%s") % (info))
                    )

    def _message_receipt_invoice(self, receipt_type, attachment):

        try:
            tree = etree.fromstring(attachment.content.encode())
        except:
            _logger.info('Error in decoding new receipt file: %s', attachment.fname)
            return {}

        elements = tree.xpath('//NomeFile')
        if elements and elements[0].text:
            filename = elements[0].text
        else:
            return {}

        if receipt_type == 'RC':
            # Delivery receipt
            # This is the receipt sent by the ES to the transmitting subject to communicate
            # delivery of the file to the addressee
            related_invoice = self._search_edi_invoice(filename, 'sent')
            if not related_invoice:
                _logger.info('Error: invoice not found for receipt file: %s', attachment.fname)
                return
            related_invoice.l10n_it_send_state = 'delivered'
            info = self._return_multi_line_xml(tree, ['//IdentificativoSdI', '//DataOraRicezione', '//DataOraConsegna', '//Note'])
            related_invoice.message_post(
                body=(_("E-Invoice is delivery to the destinatory:<br/>%s") % (info))
            )

        elif receipt_type == 'NS':
            # Rejection notice
            # This is the receipt sent by the ES to the transmitting subject if one or more of
            # the checks carried out by the ES on the file received do not have a successful result.
            related_invoice = self._search_edi_invoice(filename, 'sent')
            if not related_invoice:
                _logger.info('Error: invoice not found for receipt file: %s', attachment.fname)
                return
            related_invoice.l10n_it_send_state = 'invalid'
            error = self._return_error_xml(tree)
            related_invoice.message_post(
                body=(_("Errors in the E-Invoice :<br/>%s") % (error))
            )
            related_invoice.activity_schedule(
                'mail.mail_activity_data_todo',
                summary='Rejection notice',
                user_id=related_invoice.invoice_user_id.id if related_invoice.invoice_user_id else self.env.user.id)

        elif receipt_type == 'MC':
            # Failed delivery notice
            # This is the receipt sent by the ES to the transmitting subject if the file is not
            # delivered to the addressee.
            related_invoice = self._search_edi_invoice(filename, 'sent')
            if not related_invoice:
                _logger.info('Error: invoice not found for receipt file: %s', attachment.fname)
                return
            info = self._return_multi_line_xml(tree, [
                '//IdentificativoSdI',
                '//DataOraRicezione',
                '//Descrizione',
                '//MessageId',
                '//Note'])
            related_invoice.message_post(
                body=(_("The E-invoice is not delivered to the addressee. The Exchange System is\
                unable to deliver the file to the Public Administration. The Exchange System will\
                contact the PA to report the problem and request that they provide a solution. \
                During the following 15 days, the Exchange System will try to forward the FatturaPA\
                file to the Administration in question again. More information:<br/>%s") % (info))
            )

        elif receipt_type == 'NE':
            # Outcome notice
            # This is the receipt sent by the ES to the invoice sender to communicate the result
            # (acceptance or refusal of the invoice) of the checks carried out on the document by
            # the addressee.
            related_invoice = self._search_edi_invoice(filename, 'delivered')
            if not related_invoice:
                _logger.info('Error: invoice not found for receipt file: %s', attachment.fname)
                return
            elements = tree.xpath('//Esito')
            if elements and elements[0].text:
                if elements[0].text == 'EC01':
                    related_invoice.l10n_it_send_state = 'delivered_accepted'
                elif elements[0].text == 'EC02':
                    related_invoice.l10n_it_send_state = 'delivered_refused'

            info = self._return_multi_line_xml(tree,
                                               ['//Esito',
                                                '//Descrizione',
                                                '//IdentificativoSdI',
                                                '//DataOraRicezione',
                                                '//DataOraConsegna',
                                                '//Note'
                                               ])
            related_invoice.message_post(
                body=(_("Outcome notice: %s<br/>%s") % (related_invoice.l10n_it_send_state, info))
            )
            if related_invoice.l10n_it_send_state == 'delivered_refused':
                related_invoice.activity_schedule(
                    'mail.mail_activity_todo',
                    user_id=related_invoice.invoice_user_id.id if related_invoice.invoice_user_id else self.env.user.id,
                    summary='Outcome notice: Refused')

        # elif receipt_type == 'MT':
            # Metadata file
            # This is the file sent by the ES to the addressee together with the invoice file,
            # containing the main reference data of the file useful for processing, including
            # the IdentificativoSDI.
            # Useless for Odoo

        elif receipt_type == 'DT':
            # Deadline passed notice
            # This is the receipt sent by the ES to both the invoice sender and the invoice
            # addressee to communicate the expiry of the maximum term for communication of
            # acceptance/refusal.
            related_invoice = self._search_edi_invoice(filename, 'delivered')
            if not related_invoice:
                _logger.info('Error: invoice not found for receipt file: %s', attachment.fname)
                return
            related_invoice.l10n_it_send_state = 'delivered_expired'
            info = self._return_multi_line_xml(tree, [
                '//Descrizione',
                '//IdentificativoSdI',
                '//Note'])
            related_invoice.message_post(
                body=(_("Expiration of the maximum term for communication of acceptance/refusal:\
                 %s<br/>%s") % (filename, info))
            )

    def _return_multi_line_xml(self, tree, element_tags):
        output_str = "<ul>"

        for element_tag in element_tags:
            elements = tree.xpath(element_tag)
            if not elements:
                continue
            for element in elements:
                if element.text:
                    text = " ".join(element.text.split())
                    output_str += "<li>%s: %s</li>" % (element.tag, text)
        return output_str + "</ul>"

    def _return_error_xml(self, tree):
        output_str = "<ul>"

        elements = tree.xpath('//Errore')
        if not elements:
            return
        for element in elements:
            descrizione = " ".join(element[1].text.split())
            if descrizione:
                output_str += "<li>Errore %s: %s</li>" % (element[0].text, descrizione)
        return output_str + "</ul>"

class IrMailServer(models.Model):
    _name = "ir.mail_server"
    _inherit = "ir.mail_server"

    def _get_test_email_addresses(self):
        self.ensure_one()

        company = self.env["res.company"].search([("l10n_it_mail_pec_server_id", "=", self.id)], limit=1)
        if not company:
            # it's not a PEC server
            return super()._get_test_email_addresses()
        email_from = self.smtp_user
        if not email_from:
            raise UserError(_('Please configure Username for this Server PEC'))
        email_to = company.l10n_it_address_recipient_fatturapa
        if not email_to:
            raise UserError(_('Please configure Government PEC-mail	in company settings'))
        return email_from, email_to

    def build_email(self, email_from, email_to, subject, body, email_cc=None, email_bcc=None, reply_to=False,
                attachments=None, message_id=None, references=None, object_id=False, subtype='plain', headers=None,
                body_alternative=None, subtype_alternative='plain'):

        if self.env.context.get('wo_bounce_return_path') and headers:
            headers['Return-Path'] = email_from
        return super(IrMailServer, self).build_email(email_from, email_to, subject, body, email_cc=email_cc, email_bcc=email_bcc, reply_to=reply_to,
                attachments=attachments, message_id=message_id, references=references, object_id=object_id, subtype=subtype, headers=headers,
                body_alternative=body_alternative, subtype_alternative=subtype_alternative)

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

    # PEC server
    l10n_it_mail_pec_server_id = fields.Many2one('ir.mail_server', string="Server PEC",
        help="Configure your PEC-mail server to send electronic invoices.")
    l10n_it_address_recipient_fatturapa = fields.Char(string="Government PEC-mail",
        help="Enter Government PEC-mail address. Ex: sdi01@pec.fatturapa.it")
    l10n_it_address_send_fatturapa = fields.Char(string="Company PEC-mail",
        help="Enter your company PEC-mail address. Ex: yourcompany@pec.mail.it")


    # Economic and Administrative Index
    l10n_it_has_eco_index = fields.Boolean(default=False,
        help="The seller/provider is a company listed on the register of companies and as\
        such must also indicate the registration data on all documents (art. 2250, Italian\
        Civil Code)")
    l10n_it_eco_index_office = fields.Many2one('res.country.state', domain="[('country_id','=','IT')]",
        string="Province of the register-of-companies office")
    l10n_it_eco_index_number = fields.Char(string="Number in register of companies", size=20,
        help="This field must contain the number under which the\
        seller/provider is listed on the register of companies.")
    l10n_it_eco_index_share_capital = fields.Float(default=0.0, string="Share capital actually paid up",
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
    l10n_it_has_tax_representative = fields.Boolean(default=False,
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

```

## File: models\res_partner.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from stdnum.it import codicefiscale, iva

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError

import re


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    l10n_it_pec_email = fields.Char(string="PEC e-mail")
    l10n_it_codice_fiscale = fields.Char(string="Codice Fiscale", size=16)
    l10n_it_pa_index = fields.Char(string="PA index",
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
            "PA index must have between 6 and 7 characters."),
    ]

    @api.model
    def _l10n_it_normalize_codice_fiscale(self, codice):
        if codice and re.match(r'^IT[0-9]{11}$', codice):
            return codice[2:13]
        return codice

    @api.onchange('vat', 'country_id')
    def _l10n_it_onchange_vat(self):
        if not self.l10n_it_codice_fiscale and self.vat and (self.country_id.code == "IT" or self.vat.startswith("IT")):
            self.l10n_it_codice_fiscale = self._l10n_it_normalize_codice_fiscale(self.vat)
        elif self.country_id.code not in [False, "IT"]:
            self.l10n_it_codice_fiscale = ""

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
from . import account_invoice
from . import account_edi_format
from . import account_chart_template
from . import ir_mail_server
from . import ddt

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_it_ddt_manager","it_ddt manager","model_l10n_it_ddt","account.group_account_invoice",1,1,1,1
```

## File: tools\remove_signature.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import warnings

_logger = logging.getLogger(__name__)

try:
    from OpenSSL import crypto as ssl_crypto
    import OpenSSL._util as ssl_util
except ImportError:
    ssl_crypto = None
    _logger.warning("Cannot import library 'OpenSSL' for PKCS#7 envelope extraction.")


def remove_signature(content):
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
        try:
            loaded_data = ssl_crypto.load_pkcs7_data(ssl_crypto.FILETYPE_ASN1, content)
        except ssl_crypto.Error:
            _logger.warning("Error reading the content, PKCS#7 signature missing or invalid. Content will be tentatively used as it is.")
            return content

    # Verify the signature
    if verify(loaded_data._pkcs7, null, null, null, out_buffer, flags) != 1:
        ssl_crypto._raise_current_error()

    # Get the content as a byte-string
    decoded_content = ssl_crypto._bio_to_string(out_buffer)
    return decoded_content

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
    <record id="fetchmail_server_form_l10n_it" model="ir.ui.view">
        <field name="name">fetchmail.server.form.l10n.it</field>
        <field name="model">fetchmail.server</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="fetchmail.view_email_server_form"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//field[@name='date']" position="after">
                <field name="l10n_it_is_pec"/>
            </xpath>
        </data>
        </field>
    </record>

    <record id="account_tax_form_l10n_it" model="ir.ui.view">
        <field name="name">account.tax.form.l10n.it</field>
        <field name="model">account.tax</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//page" position="inside">
                <group>
                    <field name="l10n_it_vat_due_date"/>
                    <field name="l10n_it_has_exoneration" readonly="False"/>
                    <field name="l10n_it_kind_exoneration" attrs="{'invisible': [('l10n_it_has_exoneration', '=', False)]}"/>
                    <field name="l10n_it_law_reference"/>
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
            <xpath expr="//div[hasclass('o_address_format')]" position="after">
                <field name="l10n_it_mail_pec_server_id" attrs="{'invisible': [('country_code', '!=', 'IT')]}"/>
                <field name="l10n_it_address_send_fatturapa" attrs="{'invisible': [('country_code', '!=', 'IT')]}"/>
                <field name="l10n_it_address_recipient_fatturapa" attrs="{'invisible': [('country_code', '!=', 'IT')]}"/>
            </xpath>
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

    <record id="invoice_supplier_tree_l10n_it" model="ir.ui.view">
        <field name="name">account.invoice.supplier.tree.l10n.it</field>
        <field name="model">account.move</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="account.view_move_tree"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//field[@name='currency_id']" position="after">
                <field name="l10n_it_send_state" invisible="1" widget="label_selection" options="{'classes': {'to_send': 'default', 'invalid': 'danger', 'sent': 'warning',
                                            'delivered': 'success', 'delivered_accepted': 'success', 'delivered_refused': 'success', 'delivered_expired': 'success', 'failed_delivery': 'success'}}"/>

                <button icon="fa-paper-plane-o" class="btn-outline-warning disabled" aria-label="Sent" title="Sent" attrs="{'invisible': [('l10n_it_send_state', '!=', 'sent')]}"/>
                <button icon="fa-exclamation-triangle" class="btn-outline-danger disabled" aria-label="Error" title="Error" attrs="{'invisible': [('l10n_it_send_state', '!=', 'invalid')]}"/>
                <button icon="fa-check" class="btn-outline-success disabled" aria-label="Delivered" title="Delivered" attrs="{'invisible': [('l10n_it_send_state', 'not in', ['delivered', 'delivered_accepted', 'delivered_refused', 'delivered_expired', 'failed_delivery'])]}"/>
            </xpath>
        </data>
        </field>
    </record>

    <record id="invoice_kanban_l10n_it" model="ir.ui.view">
        <field name="name">account.invoice.kanban.l10n.it</field>
        <field name="model">account.move</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="account.view_account_move_kanban"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//field[@name='currency_id']" position="after">
                <field name="l10n_it_send_state"/>
            </xpath>

            <xpath expr="//div[hasclass('o_kanban_record_headings')]" position="inside">
                <i class="text-success fa fa-plus-circle" aria-label="New" t-if="record.l10n_it_send_state.raw_value == 'new'"/>
                <i class="text-warning fa fa-paper-plane-o" aria-label="Sent, waiting for response" t-if="record.l10n_it_send_state.raw_value == 'sent'"/>
                <i class="text-danger fa fa-exclamation-triangle" aria-label="Sent, but invalid" t-if="record.l10n_it_send_state.raw_value == 'invalid'"/>
                <i class="text-success fa fa-check" aria-label="Delivered Invoice" t-if="['delivered', 'delivered_accepted', 'delivered_refused', 'delivered_expired', 'failed_delivery'].indexOf(record.l10n_it_send_state.raw_value) >= 0" />
            </xpath>
        </data>
        </field>
    </record>

    <record id="account_invoice_form_l10n_it" model="ir.ui.view">
        <field name="name">account.move.form.l10n.it</field>
        <field name="model">account.move</field>
        <field name="priority">20</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
        <data>
            <xpath expr="//field[@name='move_type']" position="before">
                <div class="alert alert-success" role="alert"
                     attrs="{'invisible': ['|', ('move_type', 'not in', ('out_invoice', 'out_refund')), ('l10n_it_send_state', 'not in', ['delivered', 'delivered_accepted', 'delivered_refused', 'delivered_expired', 'failed_delivery'])]}">
                    <i class="fa fa-check" aria-label="Delivered" title="Delivered"></i> <field name="l10n_it_send_state" readonly="1"/>
                </div>
                <div class="alert alert-warning" role="alert"
                     attrs="{'invisible': ['|', ('move_type', 'not in', ('out_invoice', 'out_refund')), ('l10n_it_send_state', '!=', 'sent')]}">
                    <i class="fa fa-paper-plane-o"/> E-Invoice sent, waiting for a response
                </div>
                <div class="alert alert-danger" role="alert"
                     attrs="{'invisible': ['|', ('move_type', 'not in', ('out_invoice', 'out_refund')), ('l10n_it_send_state', '!=', 'invalid')]}">
                    <i class="fa fa-exclamation-triangle"/> E-Invoice check failed. You can modify the invoice, and resend it.
                </div>
            </xpath>
            <xpath expr="//page[@name='other_info']" position="after">
                <page string="Electronic Invoicing"
                    name="electronic_invoicing"
                    attrs="{'invisible': ['|', ('move_type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('country_code', '!=', 'IT')]}">
                    <group>
                        <group>
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

    <record id="view_account_invoice_filter_l10n_it" model="ir.ui.view">
        <field name="name">account.invoice.select.l10n.it</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_invoice_filter"/>
        <field name="arch" type="xml">
            <xpath expr="//filter[@name='late']" position="after">
                <separator/>
                <filter name="error" string="E-invoice error" domain="[('l10n_it_send_state', '=', 'invalid')]"/>
                <filter name="sent" string="E-invoice sent" domain="[('l10n_it_send_state', '=', 'sent')]"/>
                <filter name="no_sent" string="E-invoice to send" domain="[('l10n_it_send_state', 'in', ['to_send',False])]"/>
                <filter name="delivered" string="E-invoice delivered" domain="[('l10n_it_send_state', 'in', ['delivered', 'delivered_accepted', 'delivered_refused', 'delivered_expired', 'failed_delivery'])]"/>
            </xpath>
            <xpath expr="//filter[@name='status']" position="after">
                <filter name="send_status" string="Send status" context="{'group_by':'l10n_it_send_state'}"/>
            </xpath>
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

