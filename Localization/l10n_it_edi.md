# Odoo Module: l10n_it_edi

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Italy - E-invoicing',
    'version': '0.3',
    'depends': ['l10n_it'],
    'author': 'Odoo',
    'description': """
E-invoice implementation
    """,
    'category': 'Localization',
    'website': 'http://www.odoo.com/',
    'data': [
        'security/ir.model.access.csv',
        'data/invoice_it_template.xml',
        'views/l10n_it_view.xml',
        ],
    'demo': [
        'data/account_invoice_demo.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\account_invoice_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- add VAT, codice fiscal and tax system for main company -->
        <record id="base.main_company" model="res.company">
            <field name="vat">IT00410128888</field>
            <field name="l10n_it_codice_fiscale">0123456789987654</field>
            <field name="l10n_it_tax_system">RF01</field>
            <field name="zip">12345</field>
        </record>

        <record id="base.res_partner_1" model="res.partner">
            <field name="vat">IT12345670124</field>
        </record>
        <record id="base.res_partner_2" model="res.partner">
            <field name="vat">IT12345670231</field>
        </record>
        <record id="base.res_partner_12" model="res.partner">
            <field name="vat">IT12345670348</field>
        </record>

    </data>
</odoo>

```

## File: data\invoice_it_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="account_invoice_line_it_FatturaPA">
            <t t-set="taxes" t-value="line.tax_ids.compute_all(line.price_unit)"/>
            <DettaglioLinee>
                <NumeroLinea t-esc="line_counter"/>
                <CodiceArticolo t-if="line.product_id.barcode">
                    <!--2.2.1.3-->
                    <CodiceTipo>EAN</CodiceTipo>
                    <CodiceValore t-esc="line.product_id.barcode"/>
                </CodiceArticolo>
                 <CodiceArticolo t-if="line.product_id.default_code">
                    <CodiceTipo>INTERNAL</CodiceTipo>
                    <CodiceValore t-esc="line.product_id.default_code"/>
                </CodiceArticolo>
                <Descrizione>
                    <t t-esc="line.name[:1000]"/>
                    <t t-if="not line.name" t-esc="'NO NAME'"/>
                </Descrizione>
                <Quantita t-esc="format_numbers(line.quantity)"/>
                <UnitaMisura t-if="line.product_uom_id.category_id.measure_type != 'unit'" t-esc="line.product_uom_id.name"/>
                <PrezzoUnitario t-esc="format_monetary(taxes['total_excluded'], currency)"/>
                <ScontoMaggiorazione t-if="line.discount != 0">
                    <!-- [2.2.1.10] -->
                    <Tipo t-esc="discount_type(line.discount)"/>
                    <Percentuale t-esc="format_numbers(abs(line.discount))"/>
                </ScontoMaggiorazione>
                <PrezzoTotale t-esc="format_monetary(line.price_subtotal, currency)"/>
                <!-- without tax, must include any discounts and any extra charge-->
                <AliquotaIVA t-if="line.tax_ids.amount_type == 'percent'" t-esc="format_numbers(line.tax_ids.amount)"/>
                <AliquotaIVA t-if="line.tax_ids.amount_type != 'percent'" t-esc="'0.00'"/>
                <Natura t-if="line.tax_ids.l10n_it_has_exoneration" t-esc="line.tax_ids.l10n_it_kind_exoneration"/>
            </DettaglioLinee>
        </template>

        <template id="account_invoice_it_FatturaPA_sede">
            <Sede>
                <Indirizzo><t t-if="partner.street" t-esc="partner.street"/> <t t-if="partner.street2" t-esc="partner.street2"/></Indirizzo>
                <CAP><t t-if="partner.country_id.code != 'IT'" t-esc="'00000'"/><t t-else="" t-esc="partner.zip"/></CAP>
                <Comune t-esc="partner.city"/>
                <Provincia t-if="partner.country_id.code == 'IT'" t-esc="partner.state_id.code"/>
                <Nazione t-esc="partner.country_id.code"/>
            </Sede>
        </template>

        <template id="account_invoice_it_FatturaPA_export">
            <t t-set="currency" t-value="record.currency_id or record.company_currency_id"/>
            <t t-set="bank" t-value="record.invoice_partner_bank_id"/>
                <p:FatturaElettronica  t-att-versione="formato_trasmissione" xmlns:ds="http://www.w3.org/2000/09/xmldsig#" xmlns:p="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.2" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://ivaservizi.agenziaentrate.gov.it/docs/xsd/fatture/v1.2 http://www.fatturapa.gov.it/export/fatturazione/sdi/fatturapa/v1.2/Schema_del_file_xml_FatturaPA_versione_1.2.xsd">
                <FatturaElettronicaHeader>
                    <DatiTrasmissione>
                        <IdTrasmittente>
                            <IdPaese t-esc="get_vat_country(record.company_id.vat)"/>
                            <IdCodice t-esc="record.company_id.l10n_it_codice_fiscale"/>
                        </IdTrasmittente>
                        <ProgressivoInvio t-esc="record.name.replace('/','')[-10:]"/>
                        <FormatoTrasmissione t-esc="formato_trasmissione"/>
                        <CodiceDestinatario t-if="record.commercial_partner_id.l10n_it_pa_index and record.commercial_partner_id.country_id.code == 'IT'" t-esc="record.commercial_partner_id.l10n_it_pa_index.upper()"/>
                        <CodiceDestinatario t-if="not record.commercial_partner_id.l10n_it_pa_index and record.commercial_partner_id.country_id.code == 'IT'" t-esc="'0000000'"/>
                        <CodiceDestinatario t-if="record.commercial_partner_id.country_id.code != 'IT'" t-esc="'XXXXXXX'"/>
                        <ContattiTrasmittente>
                            <Telefono t-if="format_phone(record.company_id.partner_id.phone)" t-esc="format_phone(record.company_id.partner_id.phone)"/>
                            <Telefono t-if="not format_phone(record.company_id.partner_id.phone) and format_phone(record.company_id.partner_id.mobile)" t-esc="format_phone(record.company_id.partner_id.mobile)"/>
                            <Email t-if="record.company_id.email" t-esc="record.company_id.email"/>
                        </ContattiTrasmittente>
                        <PECDestinatario t-if="record.commercial_partner_id.l10n_it_pec_email" t-esc="record.commercial_partner_id.l10n_it_pec_email"/>
                    </DatiTrasmissione>
                    <CedentePrestatore>
                        <DatiAnagrafici>
                            <IdFiscaleIVA>
                                <IdPaese t-esc="get_vat_country(record.company_id.vat)"/>
                                <IdCodice t-esc="get_vat_number(record.company_id.vat)"/>
                            </IdFiscaleIVA>
                            <CodiceFiscale t-if="record.company_id.l10n_it_codice_fiscale" t-esc="record.company_id.l10n_it_codice_fiscale"/>
                            <Anagrafica>
                                <Denominazione t-esc="record.company_id.partner_id.display_name"/>
                            </Anagrafica>
                            <RegimeFiscale t-esc="record.company_id.l10n_it_tax_system"/>
                        </DatiAnagrafici>
                        <t t-call="l10n_it_edi.account_invoice_it_FatturaPA_sede">
                            <t t-set="partner" t-value="record.company_id.partner_id"/>
                        </t>
                        <IscrizioneREA t-if="record.company_id.l10n_it_has_eco_index">
                            <!--1.2.4-->
                            <Ufficio t-esc="record.company_id.l10n_it_eco_index_office.code"/>
                            <NumeroREA t-esc="record.company_id.l10n_it_eco_index_number"/>
                            <CapitaleSociale t-if="record.company_id.l10n_it_eco_index_share_capital != 0" t-esc="format_numbers_two(record.company_id.l10n_it_eco_index_share_capital)"/>
                            <SocioUnico t-if="record.company_id.l10n_it_eco_index_sole_shareholder != 'NO'" t-esc="record.company_id.l10n_it_eco_index_sole_shareholder"/>
                            <StatoLiquidazione t-esc="record.company_id.l10n_it_eco_index_liquidation_state"/>
                        </IscrizioneREA>
                    </CedentePrestatore>
                    <RappresentanteFiscale t-if="record.company_id.l10n_it_has_tax_representative">
                        <!--1.3-->
                        <DatiAnagrafici>
                            <IdFiscaleIVA>
                                <IdPaese t-esc="get_vat_country(record.company_id.l10n_it_tax_representative_partner_id.vat)"/>
                                <IdCodice t-esc="get_vat_number(record.company_id.l10n_it_tax_representative_partner_id.vat)"/>
                            </IdFiscaleIVA>
                            <CodiceFiscale t-if="record.company_id.l10n_it_tax_representative_partner_id.l10n_it_codice_fiscale" t-esc="record.company_id.l10n_it_tax_representative_partner_id.l10n_it_codice_fiscale"/>
                            <Anagrafica>
                                <Denominazione t-if="record.company_id.l10n_it_tax_representative_partner_id.is_company" t-esc="record.company_id.l10n_it_tax_representative_partner_id.display_name"/>
                                <Nome t-if="not record.company_id.l10n_it_tax_representative_partner_id.is_company" t-esc="' '.join(record.company_id.l10n_it_tax_representative_partner_id.name.split()[:1])"/>
                                <Cognome t-if="not record.company_id.l10n_it_tax_representative_partner_id.is_company" t-esc="' '.join(record.company_id.l10n_it_tax_representative_partner_id.name.split()[1:])"/>
                            </Anagrafica>
                        </DatiAnagrafici>
                    </RappresentanteFiscale>
                    <CessionarioCommittente>
                        <DatiAnagrafici>
                            <IdFiscaleIVA t-if="record.commercial_partner_id.vat and in_eu(record.commercial_partner_id)">
                                <IdPaese t-esc="get_vat_country(record.commercial_partner_id.vat)"/>
                                <IdCodice t-esc="get_vat_number(record.commercial_partner_id.vat)"/>
                            </IdFiscaleIVA>
                            <IdFiscaleIVA t-if="record.commercial_partner_id.vat and not in_eu(record.commercial_partner_id)">
                                <IdPaese t-esc="record.commercial_partner_id.country_id.code"/>
                                <IdCodice t-esc="'OO99999999999'"/>
                            </IdFiscaleIVA>
                            <IdFiscaleIVA t-if="not record.commercial_partner_id.vat and record.commercial_partner_id.country_id.code != 'IT'">
                                <IdCodice t-esc="'0000000'"/>
                            </IdFiscaleIVA>
                            <CodiceFiscale t-if="not record.commercial_partner_id.vat" t-esc="record.commercial_partner_id.l10n_it_codice_fiscale"/>
                            <CodiceFiscale t-if="not record.commercial_partner_id.vat and not record.commercial_partner_id.l10n_it_codice_fiscale" t-esc="99999999999"/>
                            <Anagrafica>
                                <Denominazione t-if="record.commercial_partner_id.is_company" t-esc="record.commercial_partner_id.display_name"/>
                                <Nome t-if="not record.commercial_partner_id.is_company" t-esc="' '.join(record.commercial_partner_id.name.split()[:1])"/>
                                <Cognome t-if="not record.commercial_partner_id.is_company" t-esc="' '.join(record.commercial_partner_id.name.split()[1:])"/>
                            </Anagrafica>
                        </DatiAnagrafici>
                        <t t-call="l10n_it_edi.account_invoice_it_FatturaPA_sede">
                            <t t-set="partner" t-value="record.commercial_partner_id"/>
                        </t>
                    </CessionarioCommittente>
                </FatturaElettronicaHeader>
                <FatturaElettronicaBody>
                    <DatiGenerali>
                        <DatiGeneraliDocumento>
                            <!--2.1.1-->
                            <TipoDocumento t-esc="document_type"/>
                            <Divisa t-esc="currency.name"/>
                            <Data t-esc="format_date(record.invoice_date)"/>
                            <Numero t-esc="record.name[-20:]"/>
                            <DatiBollo t-if="record.l10n_it_stamp_duty">
                                <!--2.1.1.6-->
                                <BolloVirtuale>SI</BolloVirtuale>
                                <ImportoBollo t-esc="format_numbers(record.l10n_it_stamp_duty)"/>
                            </DatiBollo>
                        </DatiGeneraliDocumento>
                        <DatiOrdineAcquisto t-if="record.ref">
                            <IdDocumento t-esc="record.ref" />
                        </DatiOrdineAcquisto>
                        <DatiDDT t-if="record.l10n_it_ddt_id">
                            <!--2.1.8-->
                            <NumeroDDT t-esc="record.l10n_it_ddt_id.name"/>
                            <DataDDT t-esc="format_date(record.l10n_it_ddt_id.date)"/>
                        </DatiDDT>
                    </DatiGenerali>
                    <DatiBeniServizi>
                        <!-- Invoice lines. -->
                        <t t-set="line_counter" t-value="0"/>
                        <t t-foreach="record.invoice_line_ids.filtered(lambda l: not l.display_type)" t-as="line">
                            <t t-set="line_counter" t-value="line_counter + 1"/>
                            <t t-call="l10n_it_edi.account_invoice_line_it_FatturaPA"/>
                        </t>
                        <t t-foreach="record.line_ids.filtered(lambda line: line.tax_line_id)" t-as="tax_line">
                            <DatiRiepilogo>
                                <!--2.2.2-->
                                <AliquotaIVA t-esc="format_numbers(tax_line.tax_line_id.amount)"/>
                                <Natura t-if="tax_line.tax_line_id.l10n_it_has_exoneration" t-esc="tax_line.tax_line_id.l10n_it_kind_exoneration"/>
                                <ImponibileImporto t-esc="format_monetary(tax_line.tax_base_amount, currency)"/>
                                <Imposta t-esc="format_monetary(tax_line.price_unit, currency)"/>
                                <EsigibilitaIVA t-if="not tax_line.tax_line_id.l10n_it_has_exoneration or tax_line.tax_line_id.l10n_it_kind_exoneration=='N6'" t-esc="tax_line.tax_line_id.l10n_it_vat_due_date"/>
                                <RiferimentoNormativo t-if="tax_line.tax_line_id.l10n_it_has_exoneration" t-esc="tax_line.tax_line_id.l10n_it_law_reference"/>
                            </DatiRiepilogo>
                        </t>
                        <!-- 0% tax lines -->
                        <t t-foreach="tax_map" t-as="tax">
                            <DatiRiepilogo>
                                <AliquotaIVA t-esc="format_numbers(tax.amount)"/>
                                <Natura t-if="tax.l10n_it_has_exoneration" t-esc="tax.l10n_it_kind_exoneration"/>
                                <ImponibileImporto t-esc="format_monetary(tax_map[tax], currency)"/>
                                <Imposta t-esc="format_monetary(0.00, currency)"/>
                                <EsigibilitaIVA t-if="not tax.l10n_it_has_exoneration or tax.l10n_it_kind_exoneration=='N6'" t-esc="tax.l10n_it_vat_due_date"/>
                                <RiferimentoNormativo t-if="tax.l10n_it_has_exoneration" t-esc="tax.l10n_it_law_reference"/>
                            </DatiRiepilogo>
                        </t>
                    </DatiBeniServizi>
                    <t t-set="company_bank_account" t-value="record.invoice_partner_bank_id"/>
                    <DatiPagamento t-if="company_bank_account and record.type != 'out_refund'">
                        <t t-set="payments" t-value="record.line_ids.filtered(lambda line: line.account_id.user_type_id.type in ('receivable', 'payable'))"/>
                        <CondizioniPagamento><t t-if="len(payments) == 1">TP02</t><t t-else="">TP01</t></CondizioniPagamento>
                        <t t-foreach="payments" t-as="payment">
                            <DettaglioPagamento>
                                <ModalitaPagamento>MP05</ModalitaPagamento>
                                <DataScadenzaPagamento t-esc="format_date(payment.date_maturity)"/>
                                <ImportoPagamento t-esc="format_monetary(abs(payment.price_total), currency)"/>
                                <IstitutoFinanziario t-if="company_bank_account.bank_id" t-esc="company_bank_account.bank_id.name[:80]"/>
                                <IBAN t-if="company_bank_account.acc_type == 'iban'" t-esc="company_bank_account.sanitized_acc_number"/>
                                <BIC t-if="company_bank_account.acc_type == 'bank' and company_bank_account.bank_id.bic" t-esc="company_bank_account.bank_id.bic"/>
                                <CodicePagamento t-esc="record.invoice_payment_ref[:60]"/>
                            </DettaglioPagamento>
                        </t>
                    </DatiPagamento>
                    <Allegati t-if="pdf">
                        <NomeAttachment t-esc="pdf_name"/>
                        <FormatoAttachment>PDF</FormatoAttachment>
                        <Attachment t-esc="pdf"/>
                    </Allegati>
                </FatturaElettronicaBody>
            </p:FatturaElettronica>
        </template>
    </data>
</odoo>

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
from odoo.tools import float_repr
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
    ], default='to_send', copy=False)

    l10n_it_stamp_duty = fields.Float(default=0, string="Dati Bollo", readonly=True, states={'draft': [('readonly', False)]})

    l10n_it_ddt_id = fields.Many2one('l10n_it.ddt', string='DDT', readonly=True, states={'draft': [('readonly', False)]}, copy=False)

    l10n_it_einvoice_name = fields.Char(readonly=True, copy=False)

    l10n_it_einvoice_id = fields.Many2one('ir.attachment', string="Electronic invoice", copy=False)

    def post(self):
        # OVERRIDE
        super(AccountMove, self).post()

        # Retrieve invoices to generate the xml.
        invoices_to_export = self.filtered(lambda move:
                move.company_id.country_id == self.env.ref('base.it') and
                move.is_sale_document() and
                move.l10n_it_send_state not in ['sent', 'delivered', 'delivered_accepted'])

        invoices_to_export.write({'l10n_it_send_state': 'other'})
        invoices_to_send = self.env['account.move']
        invoices_other = self.env['account.move']
        for invoice in invoices_to_export:
            invoice._check_before_xml_exporting()
            invoice.invoice_generate_xml()
            if len(invoice.commercial_partner_id.l10n_it_pa_index or '') == 6:
                invoice.message_post(
                    body=(_("Invoices for PA are not managed by Odoo, you can download the document and send it on your own."))
                )
                invoices_other += invoice
            else:
                invoices_to_send += invoice

        invoices_other.write({'l10n_it_send_state': 'other'})
        invoices_to_send.write({'l10n_it_send_state': 'to_send'})

        for invoice in invoices_to_send:
            invoice.send_pec_mail()

    def _check_before_xml_exporting(self):
        seller = self.company_id
        buyer = self.commercial_partner_id

        # <1.1.1.1>
        if not seller.country_id:
            raise UserError(_("%s must have a country") % (seller.display_name))

        # <1.1.1.2>
        if not seller.vat:
            raise UserError(_("%s must have a VAT number") % (seller.display_name))
        elif len(seller.vat) > 30:
            raise UserError(_("The maximum length for VAT number is 30. %s have a VAT number too long: %s.") % (seller.display_name, seller.vat))

        # <1.2.1.2>
        if not seller.l10n_it_codice_fiscale:
            raise UserError(_("%s must have a codice fiscale number") % (seller.display_name))

        # <1.2.1.8>
        if not seller.l10n_it_tax_system:
            raise UserError(_("The seller's company must have a tax system."))

        # <1.2.2>
        if not seller.street and not seller.street2:
            raise UserError(_("%s must have a street.") % (seller.display_name))
        if not seller.zip:
            raise UserError(_("%s must have a post code.") % (seller.display_name))
        if len(seller.zip) != 5 and seller.country_id.code == 'IT':
            raise UserError(_("%s must have a post code of length 5.") % (seller.display_name))
        if not seller.city:
            raise UserError(_("%s must have a city.") % (seller.display_name))
        if not seller.country_id:
            raise UserError(_("%s must have a country.") % (seller.display_name))

        # <1.4.1>
        if not buyer.vat and not buyer.l10n_it_codice_fiscale and buyer.country_id.code == 'IT':
            raise UserError(_("The buyer, %s, or his company must have either a VAT number either a tax code (Codice Fiscale).") % (buyer.display_name))

        # <1.4.2>
        if not buyer.street and not buyer.street2:
            raise UserError(_("%s must have a street.") % (buyer.display_name))
        if not buyer.zip:
            raise UserError(_("%s must have a post code.") % (buyer.display_name))
        if len(buyer.zip) != 5 and buyer.country_id.code == 'IT':
            raise UserError(_("%s must have a post code of length 5.") % (buyer.display_name))
        if not buyer.city:
            raise UserError(_("%s must have a city.") % (buyer.display_name))
        if not buyer.country_id:
            raise UserError(_("%s must have a country.") % (buyer.display_name))

        # <2.2.1>
        for invoice_line in self.invoice_line_ids:
            if len(invoice_line.tax_ids) != 1:
                raise UserError(_("You must select one and only one tax by line."))

        for tax_line in self.line_ids.filtered(lambda line: line.tax_line_id):
            if not tax_line.tax_line_id.l10n_it_has_exoneration and tax_line.tax_line_id.amount == 0:
                raise ValidationError(_("%s has an amount of 0.0, you must indicate the kind of exoneration." % tax_line.name))

    def invoice_generate_xml(self):
        for invoice in self:
            if invoice.l10n_it_einvoice_id and invoice.l10n_it_send_state not in ['invalid', 'to_send']:
                raise UserError(_("You can't regenerate an E-Invoice when the first one is sent and there are no errors"))
            if invoice.l10n_it_einvoice_id:
                invoice.l10n_it_einvoice_id.unlink()

            a = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"
            n = invoice.id
            progressive_number = ""
            while n:
                (n,m) = divmod(n,len(a))
                progressive_number = a[m] + progressive_number

            report_name = '%(country_code)s%(codice)s_%(progressive_number)s.xml' % {
                'country_code': invoice.company_id.country_id.code,
                'codice': invoice.company_id.l10n_it_codice_fiscale,
                'progressive_number': progressive_number.zfill(5),
                }
            invoice.l10n_it_einvoice_name = report_name

            data = b"<?xml version='1.0' encoding='UTF-8'?>" + invoice._export_as_xml()
            description = _('Italian invoice: %s') % invoice.type
            invoice.l10n_it_einvoice_id = self.env['ir.attachment'].create({
                'name': report_name,
                'res_id': invoice.id,
                'res_model': invoice._name,
                'datas': base64.encodebytes(data),
                'description': description,
                'type': 'binary',
                })

            invoice.message_post(
                body=(_("E-Invoice is generated on %s by %s") % (fields.Datetime.now(), self.env.user.display_name))
            )

    def _prepare_fatturapa_export_values(self):
        ''' Create the xml file content.
        :return: The XML content as str.
        '''
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

        def in_eu(partner):
            europe = self.env.ref('base.europe', raise_if_not_found=False)
            country = partner.country_id
            if not europe or not country or country in europe.country_ids:
                return True
            return False

        formato_trasmissione = "FPR12"
        if len(self.commercial_partner_id.l10n_it_pa_index or '1') == 6:
            formato_trasmissione = "FPA12"

        if self.type == 'out_invoice':
            document_type = 'TD01'
        elif self.type == 'out_refund':
            document_type = 'TD04'
        else:
            document_type = 'TD0X'

        pdf = self.env.ref('account.account_invoices').render_qweb_pdf(self.id)[0]
        pdf = base64.b64encode(pdf)
        pdf_name = re.sub(r'\W+', '', self.name) + '.pdf'

        # tax map for 0% taxes which have no tax_line_id
        tax_map = dict()
        for line in self.line_ids:
            for tax in line.tax_ids:
                if tax.amount == 0.0:
                    tax_map[tax] = tax_map.get(tax, 0.0) + line.price_subtotal

        # Create file content.
        template_values = {
            'record': self,
            'format_date': format_date,
            'format_monetary': format_monetary,
            'format_numbers': format_numbers,
            'format_numbers_two': format_numbers_two,
            'format_phone': format_phone,
            'discount_type': discount_type,
            'get_vat_number': get_vat_number,
            'get_vat_country': get_vat_country,
            'in_eu': in_eu,
            'abs': abs,
            'formato_trasmissione': formato_trasmissione,
            'document_type': document_type,
            'pdf': pdf,
            'pdf_name': pdf_name,
            'tax_map': tax_map,
        }
        return template_values

    def _export_as_xml(self):
        template_values = self._prepare_fatturapa_export_values()
        content = self.env.ref('l10n_it_edi.account_invoice_it_FatturaPA_export').render(template_values)
        return content

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
            'subject': _('Sending file: %s') % (self.l10n_it_einvoice_id.name),
            'body': _('Sending file: %s to ES: %s') % (self.l10n_it_einvoice_id.name, self.env.company.l10n_it_address_recipient_fatturapa),
            'author_id': self.env.user.partner_id.id,
            'email_from': self.env.company.l10n_it_address_send_fatturapa,
            'reply_to': self.env.company.l10n_it_address_send_fatturapa,
            'mail_server_id': self.env.company.l10n_it_mail_pec_server_id.id,
            'attachment_ids': [(6, 0, self.l10n_it_einvoice_id.ids)],
        })

        mail_fattura = self.env['mail.mail'].with_context(wo_bounce_return_path=True).create({
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

    def _import_xml_invoice(self, tree):
        ''' Extract invoice values from the E-Invoice xml tree passed as parameter.

        :param content: The tree of the xml file.
        :return: A dictionary containing account.invoice values to create/update it.
        '''

        invoices = self.env['account.move']
        multi = False

        # possible to have multiple invoices in the case of an invoice batch, the batch itself is repeated for every invoice of the batch
        for body_tree in tree.xpath('//FatturaElettronicaBody'):
            if multi:
                # make sure all the iterations create a new invoice record (except the first which could have already created one)
                self = self.env['account.move']
            multi = True

            elements = tree.xpath('//DatiGeneraliDocumento/TipoDocumento')
            if elements and elements[0].text and elements[0].text == 'TD01':
                self_ctx = self.with_context(default_type='in_invoice')
            elif elements and elements[0].text and elements[0].text == 'TD04':
                self_ctx = self.with_context(default_type='in_refund')
            else:
                _logger.info(_('Document type not managed: %s.') % (elements[0].text))

            # type must be present in the context to get the right behavior of the _default_journal method (account.move).
            # journal_id must be present in the context to get the right behavior of the _default_account method (account.move.line).

            elements = tree.xpath('//CessionarioCommittente//IdCodice')
            company = elements and self.env['res.company'].search([('vat', 'ilike', elements[0].text)], limit=1)
            if not company:
                elements = tree.xpath('//CessionarioCommittente//CodiceFiscale')
                company = elements and self.env['res.company'].search([('l10n_it_codice_fiscale', 'ilike', elements[0].text)], limit=1)

            if company:
                self_ctx = self_ctx.with_context(company_id=company.id)
            else:
                company = self.env.company
                if elements:
                    _logger.info(_('Company not found with codice fiscale: %s. The company\'s user is set by default.') % elements[0].text)
                else:
                    _logger.info(_('Company not found. The company\'s user is set by default.'))

            if not self.env.is_superuser():
                if self.env.company != company:
                    raise UserError(_("You can only import invoice concern your current company: %s") % self.env.company.display_name)

            # Refund type.
            # TD01 == invoice
            # TD02 == advance/down payment on invoice
            # TD03 == advance/down payment on fee
            # TD04 == credit note
            # TD05 == debit note
            # TD06 == fee
            elements = tree.xpath('//DatiGeneraliDocumento/TipoDocumento')
            if elements and elements[0].text and elements[0].text == 'TD01':
                type = 'in_invoice'
            elif elements and elements[0].text and elements[0].text == 'TD04':
                type = 'in_refund'
            # self could be a single record (editing) or be empty (new).
            with Form(self.with_context(default_type=type)) as invoice_form:
                message_to_log = []

                # Partner (first step to avoid warning 'Warning! You must first select a partner.'). <1.2>
                elements = tree.xpath('//CedentePrestatore//IdCodice')
                partner = elements and self.env['res.partner'].search(['&', ('vat', 'ilike', elements[0].text), '|', ('company_id', '=', company.id), ('company_id', '=', False)], limit=1)
                if not partner:
                    elements = tree.xpath('//CedentePrestatore//CodiceFiscale')
                    partner = elements and self.env['res.partner'].search(['&', ('l10n_it_codice_fiscale', '=', elements[0].text), '|', ('company_id', '=', company.id), ('company_id', '=', False)], limit=1)
                if not partner:
                    elements = tree.xpath('//DatiTrasmissione//Email')
                    partner = elements and self.env['res.partner'].search(['&', '|', ('email', '=', elements[0].text), ('l10n_it_pec_email', '=', elements[0].text), '|', ('company_id', '=', company.id), ('company_id', '=', False)], limit=1)
                if partner:
                    invoice_form.partner_id = partner
                else:
                    message_to_log.append("%s<br/>%s" % (
                        _("Vendor not found, useful informations from XML file:"),
                        self._compose_info_message(
                            tree, './/CedentePrestatore')))

                # Numbering attributed by the transmitter. <1.1.2>
                elements = tree.xpath('//ProgressivoInvio')
                if elements:
                    invoice_form.invoice_payment_ref = elements[0].text

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
                    date_str = elements[0].text
                    date_obj = datetime.strptime(date_str, DEFAULT_FACTUR_ITALIAN_DATE_FORMAT)
                    invoice_form.invoice_date = date_obj.strftime(DEFAULT_FACTUR_ITALIAN_DATE_FORMAT)

                #  Dati Bollo. <2.1.1.6>
                elements = body_tree.xpath('.//DatiGeneraliDocumento/DatiBollo/ImportoBollo')
                if elements:
                    invoice_form.l10n_it_stamp_duty = float(elements[0].text)

                # List of all amount discount (will be add after all article to avoid to have a negative sum)
                discount_list = []
                percentage_global_discount = 1.0

                # Global discount. <2.1.1.8>
                discount_elements = body_tree.xpath('.//DatiGeneraliDocumento/ScontoMaggiorazione')
                total_discount_amount = 0.0
                if discount_elements:
                    for discount_element in discount_elements:
                        discount_line = discount_element.xpath('.//Tipo')
                        discount_sign = -1
                        if discount_line and discount_line[0].text == 'SC':
                            discount_sign = 1
                        discount_percentage = discount_element.xpath('.//Percentuale')
                        if discount_percentage and discount_percentage[0].text:
                            percentage_global_discount *= 1 - float(discount_percentage[0].text)/100 * discount_sign

                        discount_amount_text = discount_element.xpath('.//Importo')
                        if discount_amount_text and discount_amount_text[0].text:
                            discount_amount = float(discount_amount_text[0].text) * discount_sign * -1
                            discount = {}
                            discount["seq"] = 0

                            if discount_amount < 0:
                                discount["name"] = _('GLOBAL DISCOUNT')
                            else:
                                discount["name"] = _('GLOBAL EXTRA CHARGE')
                            discount["amount"] = discount_amount
                            discount["tax"] = []
                            discount_list.append(discount)

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
                            self._compose_info_message(element, '.')))

                #  Dati DDT. <2.1.8>
                elements = body_tree.xpath('.//DatiGenerali/DatiDDT')
                if elements:
                    message_to_log.append("%s<br/>%s" % (
                        _("Transport informations from XML file:"),
                        self._compose_info_message(body_tree, './/DatiGenerali/DatiDDT')))

                # Due date. <2.4.2.5>
                elements = body_tree.xpath('.//DatiPagamento/DettaglioPagamento/DataScadenzaPagamento')
                if elements:
                    date_str = elements[0].text
                    date_obj = datetime.strptime(date_str, DEFAULT_FACTUR_ITALIAN_DATE_FORMAT)
                    invoice_form.invoice_date_due = fields.Date.to_string(date_obj)

                # Total amount. <2.4.2.6>
                elements = body_tree.xpath('.//ImportoPagamento')
                amount_total_import = 0
                for element in elements:
                    amount_total_import += float(element.text)
                if amount_total_import:
                    message_to_log.append(_("Total amount from the XML File: %s") % (
                        amount_total_import))

                # Bank account. <2.4.2.13>
                if invoice_form.type not in ('out_invoice', 'in_refund'):
                    elements = body_tree.xpath('.//DatiPagamento/DettaglioPagamento/IBAN')
                    if elements:
                        if invoice_form.partner_id and invoice_form.partner_id.commercial_partner_id:
                            bank = self.env['res.partner.bank'].search([
                                ('acc_number', '=', elements[0].text),
                                ('partner_id.id', '=', invoice_form.partner_id.commercial_partner_id.id)
                                ])
                        else:
                            bank = self.env['res.partner.bank'].search([('acc_number', '=', elements[0].text)])
                        if bank:
                            invoice_form.invoice_partner_bank_id = bank
                        else:
                            message_to_log.append("%s<br/>%s" % (
                                _("Bank account not found, useful informations from XML file:"),
                                self._compose_multi_info_message(
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
                                self._compose_info_message(body_tree, './/DatiPagamento')))

                # Invoice lines. <2.2.1>
                elements = body_tree.xpath('.//DettaglioLinee')
                if elements:
                    for element in elements:
                        with invoice_form.invoice_line_ids.new() as invoice_line_form:

                            # Sequence.
                            line_elements = element.xpath('.//NumeroLinea')
                            if line_elements:
                                invoice_line_form.sequence = int(line_elements[0].text) * 2

                            # Product.
                            line_elements = element.xpath('.//Descrizione')
                            if line_elements:
                                invoice_line_form.name = " ".join(line_elements[0].text.split())

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
                                        product_supplier = self.env['product.supplierinfo'].search([('name', '=', partner.id), ('product_code', '=', code.text)])
                                        if product_supplier and product_supplier.product_id:
                                            invoice_line_form.product_id = product_supplier.product_id
                                            break
                                if not invoice_line_form.product_id:
                                    for element_code in elements_code:
                                        code = element_code.xpath('.//CodiceValore')[0]
                                        product = self.env['product.product'].search([('default_code', '=', code.text)])
                                        if product:
                                            invoice_line_form.product_id = product
                                            break

                            # Price Unit.
                            line_elements = element.xpath('.//PrezzoUnitario')
                            if line_elements:
                                invoice_line_form.price_unit = float(line_elements[0].text)

                            # Quantity.
                            line_elements = element.xpath('.//Quantita')
                            if line_elements:
                                invoice_line_form.quantity = float(line_elements[0].text)
                            else:
                                invoice_line_form.quantity = 1

                            # Taxes
                            tax_element = element.xpath('.//AliquotaIVA')
                            natura_element = element.xpath('.//Natura')
                            invoice_line_form.tax_ids.clear()
                            if tax_element and tax_element[0].text:
                                percentage = float(tax_element[0].text)
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

                            # Discount in cascade mode.
                            # if 3 discounts : -10% -50€ -20%
                            # the result must be : (((price -10%)-50€) -20%)
                            # Generic form : (((price -P1%)-A1€) -P2%)
                            # It will be split in two parts: fix amount and pourcent amount
                            # example: (((((price - A1€) -P2%) -A3€) -A4€) -P5€)
                            # pourcent: 1-(1-P2)*(1-P5)
                            # fix amount: A1*(1-P2)*(1-P5)+A3*(1-P5)+A4*(1-P5) (we must take account of all
                            # percentage present after the fix amount)
                            line_elements = element.xpath('.//ScontoMaggiorazione')
                            total_discount_amount = 0.0
                            total_discount_percentage = percentage_global_discount
                            if line_elements:
                                for line_element in line_elements:
                                    discount_line = line_element.xpath('.//Tipo')
                                    discount_sign = -1
                                    if discount_line and discount_line[0].text == 'SC':
                                        discount_sign = 1
                                    discount_percentage = line_element.xpath('.//Percentuale')
                                    if discount_percentage and discount_percentage[0].text:
                                        pourcentage_actual = 1 - float(discount_percentage[0].text)/100 * discount_sign
                                        total_discount_percentage *= pourcentage_actual
                                        total_discount_amount *= pourcentage_actual

                                    discount_amount = line_element.xpath('.//Importo')
                                    if discount_amount and discount_amount[0].text:
                                        total_discount_amount += float(discount_amount[0].text) * discount_sign * -1

                                # Save amount discount.
                                if total_discount_amount != 0:
                                    discount = {}
                                    discount["seq"] = invoice_line_form.sequence + 1

                                    if total_discount_amount < 0:
                                        discount["name"] = _('DISCOUNT: ') + invoice_line_form.name
                                    else:
                                        discount["name"] = _('EXTRA CHARGE: ') + invoice_line_form.name
                                    discount["amount"] = total_discount_amount
                                    discount["tax"] = []
                                    for tax in invoice_line_form.tax_ids:
                                        discount["tax"].append(tax)
                                    discount_list.append(discount)
                            invoice_line_form.discount = (1 - total_discount_percentage) * 100

                # Apply amount discount.
                for discount in discount_list:
                    with invoice_form.invoice_line_ids.new() as invoice_line_form_discount:
                        invoice_line_form_discount.tax_ids.clear()
                        invoice_line_form_discount.sequence = discount["seq"]
                        invoice_line_form_discount.name = discount["name"]
                        invoice_line_form_discount.price_unit = discount["amount"]

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

    @api.model
    def _get_xml_decoders(self):
        # Override
        ubl_decoders = [('Italy', self._detect_italy_edi, self._import_xml_invoice)]
        return super(AccountMove, self)._get_xml_decoders() + ubl_decoders

    @api.model
    def _detect_italy_edi(self, tree, file_name):
        # Quick check the file name looks like the one of an Italian EDI XML.
        flag = re.search("([A-Z]{2}[A-Za-z0-9]{2,28}_[A-Za-z0-9]{0,5}.(xml.p7m|xml))", file_name)
        error = None

        return {'flag': flag, 'error': error}

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

    invoice_id = fields.One2many('account.move', 'l10n_it_ddt_id', string='Invoice Reference', ondelete='cascade')
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
import dateutil
import pytz
import base64
try:
    from xmlrpc import client as xmlrpclib
except ImportError:
    import xmlrpclib


from lxml import etree
from datetime import datetime

from odoo import api, fields, models, tools, _
from odoo.exceptions import ValidationError, UserError


_logger = logging.getLogger(__name__)

class FetchmailServer(models.Model):
    _name = 'fetchmail.server'
    _inherit = 'fetchmail.server'

    l10n_it_is_pec = fields.Boolean('PEC server', help="If PEC Server, only mail from '...@pec.fatturapa.it' will be processed.")
    l10n_it_last_uid = fields.Integer(string='Last message UID', default=1)

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

                result, data = imap_server.uid('search', None, '(FROM "@pec.fatturapa.it")', '(UID %s:*)' % (server.l10n_it_last_uid))
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
                    msg_txt = email.message_from_bytes(message)

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
        from_address = tools.decode_smtp_header(msg_txt.get('from'))
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
                elif re.search("([A-Z]{2}[A-Za-z0-9]{2,28}_[A-Za-z0-9]{0,5}.(xml.p7m|xml))", attachment.fname):
                    # we have a new E-invoice
                    self._create_invoice_from_mail(attachment.content, attachment.fname, from_address)
            else:
                if split_underscore[1] == 'AT':
                    # Attestazione di avvenuta trasmissione della fattura con impossibilità di recapito
                    self._message_AT_invoice(attachment)
                else:
                    _logger.info('New E-invoice in zip file: %s', attachment.fname)
                    self._create_invoice_from_mail_with_zip(attachment, from_address)

    def _create_invoice_from_mail(self, att_content, att_name, from_address):
        if self.env['account.move'].search([('l10n_it_einvoice_name', '=', att_name)], limit=1):
            # invoice already exist
            _logger.info('E-invoice already exist: %s', att_name)
            return

        invoice_attachment = self.env['ir.attachment'].create({
                'name': att_name,
                'datas': base64.encodebytes(att_content),
                'type': 'binary',
                })

        try:
            tree = etree.fromstring(att_content)
        except Exception:
            raise UserError(_('The xml file is badly formatted : {}').format(att_name))

        invoice = self.env['account.move']._import_xml_invoice(tree)
        invoice.l10n_it_send_state = "new"
        invoice.source_email = from_address
        self._cr.commit()

        _logger.info('New E-invoice: %s', att_name)


    def _create_invoice_from_mail_with_zip(self, attachment_zip, from_address):
        with zipfile.ZipFile(io.BytesIO(attachment_zip.content)) as z:
            for att_name in z.namelist():
                if self.env['account.move'].search([('l10n_it_einvoice_name', '=', att_name)], limit=1):
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

                    related_invoice = self.env['account.move'].search([
                        ('l10n_it_einvoice_name', '=', filename)])
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
            tree = etree.fromstring(attachment.content)
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
            related_invoice = self.env['account.move'].search([
                ('l10n_it_einvoice_name', '=', filename),
                ('l10n_it_send_state', '=', 'sent')])
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
            related_invoice = self.env['account.move'].search([
                ('l10n_it_einvoice_name', '=', filename),
                ('l10n_it_send_state', '=', 'sent')])
            if not related_invoice:
                _logger.info('Error: invoice not found for receipt file: %s', attachment.fname)
                return
            related_invoice.l10n_it_send_state = 'invalid'
            error = self._return_error_xml(tree)
            related_invoice.message_post(
                body=(_("Errors in the E-Invoice :<br/>%s") % (error))
            )
            activity_vals = {
                'activity_type_id': self.env.ref('mail.mail_activity_data_todo').id,
                'invoice_user_id': related_invoice.invoice_user_id.id if related_invoice.invoice_user_id else self.env.user.id
            }
            related_invoice.activity_schedule(summary='Rejection notice', **activity_vals)

        elif receipt_type == 'MC':
            # Failed delivery notice
            # This is the receipt sent by the ES to the transmitting subject if the file is not
            # delivered to the addressee.
            related_invoice = self.env['account.move'].search([
                ('l10n_it_einvoice_name', '=', filename),
                ('l10n_it_send_state', '=', 'sent')])
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
                file to the Administration in question again. More informations:<br/>%s") % (info))
            )

        elif receipt_type == 'NE':
            # Outcome notice
            # This is the receipt sent by the ES to the invoice sender to communicate the result
            # (acceptance or refusal of the invoice) of the checks carried out on the document by
            # the addressee.
            related_invoice = self.env['account.move'].search([
                ('l10n_it_einvoice_name', '=', filename),
                ('l10n_it_send_state', '=', 'delivered')])
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
                activity_vals = {
                    'activity_type_id': self.env.ref('mail.mail_activity_data_todo').id,
                    'invoice_user_id': related_invoice.invoice_user_id.id if related_invoice.invoice_user_id else self.env.user.id
                }
                related_invoice.activity_schedule(summary='Outcome notice: Refused', **activity_vals)

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
            related_invoice = self.env['account.move'].search([
                ('l10n_it_einvoice_name', '=', filename), ('l10n_it_send_state', '=', 'delivered')])
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
                    'l10n_it_eco_index_share_capital',
                    'l10n_it_eco_index_sole_shareholder',
                    'l10n_it_eco_index_liquidation_state')
    def _check_eco_admin_index(self):
        for record in self:
            if not record.l10n_it_has_eco_index:
                continue
            if not record.l10n_it_eco_index_office\
               or not record.l10n_it_eco_index_number\
               or not record.l10n_it_eco_index_share_capital\
               or not record.l10n_it_eco_index_sole_shareholder\
               or not record.l10n_it_eco_index_liquidation_state:
                raise ValidationError(_("All fields about the Economic and Administrative Index must be completed."))

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

from odoo import api, fields, models
from odoo.exceptions import ValidationError

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
            "CHECK(l10n_it_codice_fiscale IS NULL OR LENGTH(l10n_it_codice_fiscale) >= 11)",
            "Codice fiscale must have between 11 and 16 characters."),

        ('l10n_it_pa_index',
            "CHECK(l10n_it_pa_index IS NULL OR LENGTH(l10n_it_pa_index) >= 6)",
            "PA index must have between 6 and 7 characters."),
    ]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner
from . import res_company
from . import account_invoice
from . import ir_mail_server
from . import ddt

```

## File: security\ir.model.access.csv

```csv
"id","name","model_id:id","group_id:id","perm_read","perm_write","perm_create","perm_unlink"
"access_it_ddt_manager","it_ddt manager","model_l10n_it_ddt","account.group_account_invoice",1,1,1,1
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
                    <field name="l10n_it_law_reference" attrs="{'invisible': [('l10n_it_has_exoneration', '=', False)]}"/>
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
                <field name="l10n_it_mail_pec_server_id"/>
                <field name="l10n_it_address_send_fatturapa"/>
                <field name="l10n_it_address_recipient_fatturapa"/>
            </xpath>
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_it_codice_fiscale"/>
                <field name="l10n_it_tax_system"/>
            </xpath>
            <xpath expr="//page" position="after">
                <page string="Electronic Invoicing">
                    <group>
                        <separator string="Economic and Administrative Index" colspan="4"/>
                        <div colspan="4">
                            The seller/provider is a company listed on the register of companies and as
                            such must also indicate the registration data on all documents (art. 2250, Italian
                            Civil Code)
                        </div>
                        <group>
                            <field name="l10n_it_has_eco_index" string="Company listed on the register of companies"/>
                        </group>
                        <group attrs="{'invisible': [('l10n_it_has_eco_index', '=', False)]}">
                            <field name="l10n_it_eco_index_office"/>
                            <field name="l10n_it_eco_index_number"/>
                            <field name="l10n_it_eco_index_share_capital"/>
                            <field name="l10n_it_eco_index_sole_shareholder"/>
                            <field name="l10n_it_eco_index_liquidation_state"/>
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
                        </group>
                        <group attrs="{'invisible': [('l10n_it_has_tax_representative', '=', False)]}">
                            <field name="l10n_it_tax_representative_partner_id"/>
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
                <i class="text-success fa fa-plus-circle" t-if="record.l10n_it_send_state.raw_value == 'new'"/>
                <i class="text-warning fa fa-paper-plane-o" t-if="record.l10n_it_send_state.raw_value == 'sent'"/>
                <i class="text-danger fa fa-exclamation-triangle" t-if="record.l10n_it_send_state.raw_value == 'invalid'"/>
                <i class="text-success fa fa-check" t-if="['delivered', 'delivered_accepted', 'delivered_refused', 'delivered_expired', 'failed_delivery'].indexOf(record.l10n_it_send_state.raw_value) >= 0" />
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
            <xpath expr="//button[@name='preview_invoice']" position="after">
                <button name="post" type="object" icon="fa-envelope-o" class="btn btn-primary" string="Resend"
                        attrs="{'invisible': ['|', ('type', 'not in', ('out_invoice', 'out_refund')), ('l10n_it_send_state', '!=', 'invalid')]}"/>
            </xpath>
            <xpath expr="//field[@name='type']" position="before">
                <div class="alert alert-success" role="alert"
                     attrs="{'invisible': ['|', ('type', 'not in', ('out_invoice', 'out_refund')), ('l10n_it_send_state', 'not in', ['delivered', 'delivered_accepted', 'delivered_refused', 'delivered_expired', 'failed_delivery'])]}">
                    <i class="fa fa-check" aria-label="Delivered" title="Delivered"></i> <field name="l10n_it_send_state" readonly="1"/>
                </div>
                <div class="alert alert-warning" role="alert"
                     attrs="{'invisible': ['|', ('type', 'not in', ('out_invoice', 'out_refund')), ('l10n_it_send_state', '!=', 'sent')]}">
                    <i class="fa fa-paper-plane-o"/> E-Invoice sent, waiting for a response
                </div>
                <div class="alert alert-danger" role="alert"
                     attrs="{'invisible': ['|', ('type', 'not in', ('out_invoice', 'out_refund')), ('l10n_it_send_state', '!=', 'invalid')]}">
                    <i class="fa fa-exclamation-triangle"/> E-Invoice check failed. You can modify the invoice, and resend it.
                </div>
            </xpath>
            <xpath expr="//page[@name='other_info']" position="after">
                <page string="Electronic Invoicing" attrs="{'invisible': [('type', 'not in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund'))]}">
                    <group>
                        <group>
                            <field name="l10n_it_stamp_duty"/>
                            <field name="l10n_it_ddt_id"
                                   attrs="{'invisible': [('type', 'not in', ('out_invoice', 'out_refund'))]}"/>
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

