# Odoo Module: l10n_es_edi_sii

Category: Accounting/Localizations/EDI

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

# Thanks to AEOdoo and the Spanish community
# Specially among others Ignacio Ibeas, Pedro Baeza and Landoo

{
    'name': "Spain - SII EDI Suministro de Libros",
    'countries': ['es'],
    'version': '1.0',
    'category': 'Accounting/Localizations/EDI',
    'description': """
This module sends the taxes information (mostly VAT) of the
vendor bills and customer invoices to the SII.  It is called
Procedimiento G417 - IVA. Llevanza de libros registro.  It is
required for every company with a turnover of +6M€ and others can
already make use of it.  The invoices are automatically
sent after validation.

How the information is sent to the SII depends on the
configuration that is put in the taxes.  The taxes
that were in the chart template (l10n_es) are automatically
configured to have the right type.  It is possible however
that extra taxes need to be created for certain exempt/no sujeta reasons.

You need to configure your certificate and the tax agency.
    """,
    'depends': [
        'l10n_es',
        'account_edi',
    ],
    'data': [
        'data/account_edi_data.xml',

        'security/ir.model.access.csv',
        'security/l10n_es_edi_certificate.xml',

        'views/l10n_es_edi_certificate_views.xml',
        'views/res_config_settings_views.xml',
        'views/account_move_views.xml',
    ],
    'demo': ['demo/demo_certificate.xml'],
    'external_dependencies': {
        'python': ['pyOpenSSL'],
    },
    'license': 'LGPL-3',
}

```

## File: data\account_edi_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="edi_es_sii" model="account.edi.format">
            <field name="name">SII IVA Llevanza de libros registro (ES)</field>
            <field name="code">es_sii</field>
        </record>

    </data>
</odoo>

```

## File: data\neutralize.sql

```sql
-- disable_l10n_es_edi_integration
UPDATE res_company
   SET l10n_es_edi_test_env = true;

```

## File: models\account_edi_format.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from collections import defaultdict
from urllib3.util.ssl_ import create_urllib3_context
from urllib3.contrib.pyopenssl import inject_into_urllib3
from OpenSSL.crypto import load_certificate, load_privatekey, FILETYPE_PEM

from odoo import fields, models, _
from odoo.tools import html_escape, zeep

import math
import json
import requests


# Custom patches to perform the WSDL requests.
# Avoid failure on servers where the DH key is too small
EUSKADI_CIPHERS = "DEFAULT:!DH"


class PatchedHTTPAdapter(requests.adapters.HTTPAdapter):
    """ An adapter to block DH ciphers which may not work for the tax agencies called"""

    def init_poolmanager(self, *args, **kwargs):
        # OVERRIDE
        inject_into_urllib3()
        kwargs['ssl_context'] = create_urllib3_context(ciphers=EUSKADI_CIPHERS)
        return super().init_poolmanager(*args, **kwargs)

    def cert_verify(self, conn, url, verify, cert):
        # OVERRIDE
        # The last parameter is only used by the super method to check if the file exists.
        # In our case, cert is an odoo record 'l10n_es_edi.certificate' so not a path to a file.
        # By putting 'None' as last parameter, we ensure the check about TLS configuration is
        # still made without checking temporary files exist.
        super().cert_verify(conn, url, verify, None)
        conn.cert_file = cert
        conn.key_file = None

    def get_connection(self, url, proxies=None):
        # OVERRIDE
        # Patch the OpenSSLContext to decode the certificate in-memory.
        conn = super().get_connection(url, proxies=proxies)
        context = conn.conn_kw['ssl_context']

        def patched_load_cert_chain(l10n_es_odoo_certificate, keyfile=None, password=None):
            cert_file, key_file, _certificate = l10n_es_odoo_certificate.sudo()._decode_certificate()
            cert_obj = load_certificate(FILETYPE_PEM, cert_file)
            pkey_obj = load_privatekey(FILETYPE_PEM, key_file)

            context._ctx.use_certificate(cert_obj)
            context._ctx.use_privatekey(pkey_obj)

        context.load_cert_chain = patched_load_cert_chain

        return conn


class AccountEdiFormat(models.Model):
    _inherit = 'account.edi.format'

    # -------------------------------------------------------------------------
    # ES EDI
    # -------------------------------------------------------------------------

    def _l10n_es_edi_get_invoices_tax_details_info(self, invoice, filter_invl_to_apply=None):

        def grouping_key_generator(base_line, tax_values):
            tax = tax_values['tax_repartition_line'].tax_id
            return {
                'applied_tax_amount': tax.amount,
                'l10n_es_type': tax.l10n_es_type,
                'l10n_es_exempt_reason': tax.l10n_es_exempt_reason if tax.l10n_es_type == 'exento' else False,
                'l10n_es_bien_inversion': tax.l10n_es_bien_inversion,
            }

        def filter_to_apply(base_line, tax_values):
            # For intra-community, we do not take into account the negative repartition line
            return (tax_values['tax_repartition_line'].factor_percent > 0.0
                    and tax_values['tax_repartition_line'].tax_id.amount != -100.0
                    and tax_values['tax_repartition_line'].tax_id.l10n_es_type != 'ignore')

        def full_filter_invl_to_apply(invoice_line):
            if all(t == 'ignore' for t in invoice_line.tax_ids.flatten_taxes_hierarchy().mapped('l10n_es_type')):
                return False
            return filter_invl_to_apply(invoice_line) if filter_invl_to_apply else True

        tax_details = invoice._prepare_edi_tax_details(
            grouping_key_generator=grouping_key_generator,
            filter_invl_to_apply=full_filter_invl_to_apply,
            filter_to_apply=filter_to_apply,
        )
        sign = -1 if invoice.move_type in ('out_refund', 'in_refund') else 1

        tax_details_info = defaultdict(dict)

        # Detect for which is the main tax for 'recargo'. Since only a single combination tax + recargo is allowed
        # on the same invoice, this can be deduced globally.

        recargo_tax_details = {} # Mapping between main tax and recargo tax details
        invoice_lines = invoice.invoice_line_ids.filtered(lambda x: x.display_type not in ('line_note', 'line_section'))
        if filter_invl_to_apply:
            invoice_lines = invoice_lines.filtered(filter_invl_to_apply)
        for line in invoice_lines:
            taxes = line.tax_ids.flatten_taxes_hierarchy()
            recargo_tax = [t for t in taxes if t.l10n_es_type == 'recargo']
            if recargo_tax and taxes:
                recargo_main_tax = taxes.filtered(lambda x: x.l10n_es_type in ('sujeto', 'sujeto_isp'))[:1]
                if not recargo_tax_details.get(recargo_main_tax):
                    recargo_tax_details[recargo_main_tax] = [
                        x for x in tax_details['tax_details'].values()
                        if x['group_tax_details'][0]['tax_repartition_line'].tax_id == recargo_tax[0]
                    ][0]

        tax_amount_deductible = 0.0
        tax_amount_retention = 0.0
        base_amount_not_subject = 0.0
        base_amount_not_subject_loc = 0.0
        has_not_subject = False
        has_not_subject_loc = False
        tax_subject_info_list = []
        tax_subject_isp_info_list = []
        for tax_values in tax_details['tax_details'].values():
            if invoice.is_sale_document():
                # Customer invoices

                if tax_values['l10n_es_type'] in ('sujeto', 'sujeto_isp'):
                    tax_amount_deductible += tax_values['tax_amount']

                    base_amount = sign * tax_values['base_amount']
                    tax_info = {
                        'TipoImpositivo': tax_values['applied_tax_amount'],
                        'BaseImponible': round(base_amount, 2),
                        'CuotaRepercutida': round(math.copysign(tax_values['tax_amount'], base_amount), 2),
                    }

                    recargo = recargo_tax_details.get(tax_values['group_tax_details'][0]['tax_repartition_line'].tax_id)
                    if recargo:
                        tax_info['CuotaRecargoEquivalencia'] = round(sign * recargo['tax_amount'], 2)
                        tax_info['TipoRecargoEquivalencia'] = recargo['applied_tax_amount']

                    if tax_values['l10n_es_type'] == 'sujeto':
                        tax_subject_info_list.append(tax_info)
                    else:
                        tax_subject_isp_info_list.append(tax_info)

                elif tax_values['l10n_es_type'] == 'exento':
                    tax_details_info['Sujeta'].setdefault('Exenta', {'DetalleExenta': []})
                    tax_details_info['Sujeta']['Exenta']['DetalleExenta'].append({
                        'BaseImponible': round(sign * tax_values['base_amount'], 2),
                        'CausaExencion': tax_values['l10n_es_exempt_reason'],
                    })
                elif tax_values['l10n_es_type'] == 'retencion':
                    tax_amount_retention += tax_values['tax_amount']
                elif tax_values['l10n_es_type'] == 'no_sujeto':
                    has_not_subject = True
                    base_amount_not_subject += tax_values['base_amount']
                elif tax_values['l10n_es_type'] == 'no_sujeto_loc':
                    has_not_subject_loc = True
                    base_amount_not_subject_loc += tax_values['base_amount']
                elif tax_values['l10n_es_type'] == 'ignore':
                    continue

            else:
                # Vendor bills
                if tax_values['l10n_es_type'] in ('sujeto', 'sujeto_isp', 'no_sujeto', 'no_sujeto_loc', 'dua'):
                    tax_amount_deductible += tax_values['tax_amount']
                elif tax_values['l10n_es_type'] == 'retencion':
                    tax_amount_retention += tax_values['tax_amount']
                elif tax_values['l10n_es_type'] == 'no_sujeto':
                    has_not_subject = True
                    base_amount_not_subject += tax_values['base_amount']
                elif tax_values['l10n_es_type'] == 'no_sujeto_loc':
                    has_not_subject_loc = True
                    base_amount_not_subject_loc += tax_values['base_amount']
                elif tax_values['l10n_es_type'] == 'ignore':
                    continue

                if tax_values['l10n_es_type'] not in ['retencion', 'recargo']: # = in sujeto/sujeto_isp/no_deducible
                    base_amount = sign * tax_values['base_amount']
                    tax_details_info.setdefault('DetalleIVA', [])
                    tax_info = {
                        'BaseImponible': round(base_amount, 2),
                    }
                    if tax_values['l10n_es_type'] == 'sujeto_agricultura':
                        tax_info.update({
                            'PorcentCompensacionREAGYP': tax_values['applied_tax_amount'],
                            'ImporteCompensacionREAGYP': round(math.copysign(tax_values['tax_amount'], base_amount), 2),
                        })
                    elif tax_values['applied_tax_amount'] > 0.0:
                        tax_info.update({
                            'TipoImpositivo': tax_values['applied_tax_amount'],
                            'CuotaSoportada': round(math.copysign(tax_values['tax_amount'], base_amount), 2),
                        })
                    if tax_values['l10n_es_bien_inversion']:
                        tax_info['BienInversion'] = 'S'
                    recargo = recargo_tax_details.get(tax_values['group_tax_details'][0]['tax_repartition_line'].tax_id)
                    if recargo:
                        tax_info['CuotaRecargoEquivalencia'] = round(sign * recargo['tax_amount'], 2)
                        tax_info['TipoRecargoEquivalencia'] = recargo['applied_tax_amount']
                    tax_details_info['DetalleIVA'].append(tax_info)

        if tax_subject_isp_info_list and not tax_subject_info_list:  # Only for sale_invoices
            tax_details_info['Sujeta']['NoExenta'] = {'TipoNoExenta': 'S2'}
        elif not tax_subject_isp_info_list and tax_subject_info_list:
            tax_details_info['Sujeta']['NoExenta'] = {'TipoNoExenta': 'S1'}
        elif tax_subject_isp_info_list and tax_subject_info_list:
            tax_details_info['Sujeta']['NoExenta'] = {'TipoNoExenta': 'S3'}

        if tax_subject_info_list:
            tax_details_info['Sujeta']['NoExenta'].setdefault('DesgloseIVA', {})
            tax_details_info['Sujeta']['NoExenta']['DesgloseIVA'].setdefault('DetalleIVA', [])
            tax_details_info['Sujeta']['NoExenta']['DesgloseIVA']['DetalleIVA'] += tax_subject_info_list
        if tax_subject_isp_info_list:
            tax_details_info['Sujeta']['NoExenta'].setdefault('DesgloseIVA', {})
            tax_details_info['Sujeta']['NoExenta']['DesgloseIVA'].setdefault('DetalleIVA', [])
            tax_details_info['Sujeta']['NoExenta']['DesgloseIVA']['DetalleIVA'] += tax_subject_isp_info_list

        if has_not_subject and invoice.is_sale_document():
            tax_details_info['NoSujeta']['ImportePorArticulos7_14_Otros'] = round(sign * base_amount_not_subject, 2)
        if has_not_subject_loc and invoice.is_sale_document():
            tax_details_info['NoSujeta']['ImporteTAIReglasLocalizacion'] = round(sign * base_amount_not_subject_loc, 2)
        if not tax_details_info and invoice.is_sale_document():
            if any(t['l10n_es_type'] == 'no_sujeto' for t in tax_details['tax_details'].values()):
                tax_details_info['NoSujeta']['ImportePorArticulos7_14_Otros'] = 0
            if any(t['l10n_es_type'] == 'no_sujeto_loc' for t in tax_details['tax_details'].values()):
                tax_details_info['NoSujeta']['ImporteTAIReglasLocalizacion'] = 0

        return {
            'tax_details_info': tax_details_info,
            'tax_details': tax_details,
            'tax_amount_deductible': tax_amount_deductible,
            'tax_amount_retention': tax_amount_retention,
            'base_amount_not_subject': base_amount_not_subject,
            'S1_list': tax_subject_info_list, #TBAI has separate sections for S1 and S2
            'S2_list': tax_subject_isp_info_list, #TBAI has separate sections for S1 and S2
        }

    def _l10n_es_edi_get_partner_info(self, partner):
        eu_country_codes = set(self.env.ref('base.europe').country_ids.mapped('code'))

        partner_info = {}
        IDOtro_ID = partner.vat or 'NO_DISPONIBLE'

        if (not partner.country_id or partner.country_id.code == 'ES') and partner.vat:
            # ES partner with VAT.
            partner_info['NIF'] = partner.vat[2:] if partner.vat.startswith('ES') else partner.vat
            if self.env.context.get('error_1117'):
                partner_info['IDOtro'] = {'IDType': '07', 'ID': IDOtro_ID}

        elif partner.country_id.code in eu_country_codes and partner.vat:
            # European partner.
            partner_info['IDOtro'] = {'IDType': '02', 'ID': IDOtro_ID}
        else:
            partner_info['IDOtro'] = {'ID': IDOtro_ID}
            if partner.vat:
                partner_info['IDOtro']['IDType'] = '04'
            else:
                partner_info['IDOtro']['IDType'] = '06'
            if partner.country_id:
                partner_info['IDOtro']['CodigoPais'] = partner.country_id.code
        return partner_info

    def _l10n_es_edi_get_invoices_info(self, invoices):
        eu_country_codes = set(self.env.ref('base.europe').country_ids.mapped('code'))

        info_list = []
        for invoice in invoices:
            com_partner = invoice.commercial_partner_id
            is_simplified = invoice.l10n_es_is_simplified

            info = {
                'PeriodoLiquidacion': {
                    'Ejercicio': str(invoice.date.year),
                    'Periodo': str(invoice.date.month).zfill(2),
                },
                'IDFactura': {
                    'FechaExpedicionFacturaEmisor': invoice.invoice_date.strftime('%d-%m-%Y'),
                },
            }

            if invoice.is_sale_document():
                invoice_node = info['FacturaExpedida'] = {}
            else:
                invoice_node = info['FacturaRecibida'] = {}

            # === Partner ===

            partner_info = self._l10n_es_edi_get_partner_info(com_partner)

            # === Invoice ===

            if invoice.delivery_date and invoice.delivery_date != invoice.invoice_date:
                invoice_node['FechaOperacion'] = invoice.delivery_date.strftime('%d-%m-%Y')
            invoice_node['DescripcionOperacion'] = invoice.invoice_origin[:500] if invoice.invoice_origin else 'manual'
            reagyp = invoice.invoice_line_ids.tax_ids.filtered(lambda t: t.l10n_es_type == 'sujeto_agricultura')
            if invoice.is_sale_document():
                nif = invoice.company_id.vat[2:] if invoice.company_id.vat.startswith('ES') else invoice.company_id.vat
                info['IDFactura']['IDEmisorFactura'] = {'NIF': nif}
                info['IDFactura']['NumSerieFacturaEmisor'] = invoice.name[:60]
                if not is_simplified:
                    invoice_node['Contraparte'] = {
                        **partner_info,
                        'NombreRazon': com_partner.name[:120],
                    }
                export_exempts = invoice.invoice_line_ids.tax_ids.filtered(lambda t: t.l10n_es_exempt_reason == 'E2')
                # If an invoice line contains an OSS tax, the invoice is considered as an OSS operation
                is_oss = self._has_oss_taxes(invoice)

                if is_oss:
                    invoice_node['ClaveRegimenEspecialOTrascendencia'] = '17'
                elif export_exempts:
                    invoice_node['ClaveRegimenEspecialOTrascendencia'] = '02'
                else:
                    invoice_node['ClaveRegimenEspecialOTrascendencia'] = '01'
            else:
                if invoice._l10n_es_is_dua():
                    partner_info = self._l10n_es_edi_get_partner_info(invoice.company_id.partner_id)
                info['IDFactura']['IDEmisorFactura'] = partner_info
                # In case of cancel
                info["IDFactura"]["IDEmisorFactura"].update(
                    {"NombreRazon": com_partner.name[0:120]}
                )
                info["IDFactura"]["NumSerieFacturaEmisor"] = (invoice.ref or "")[:60]
                if not is_simplified:
                    invoice_node['Contraparte'] = {
                        **partner_info,
                        'NombreRazon': com_partner.name[:120],
                    }

                if invoice.l10n_es_registration_date:
                    invoice_node['FechaRegContable'] = invoice.l10n_es_registration_date.strftime('%d-%m-%Y')
                else:
                    invoice_node['FechaRegContable'] = fields.Date.context_today(self).strftime('%d-%m-%Y')

                mod_303_10 = self.env.ref('l10n_es.mod_303_casilla_10_balance')._get_matching_tags()
                mod_303_11 = self.env.ref('l10n_es.mod_303_casilla_11_balance')._get_matching_tags()
                tax_tags = invoice.invoice_line_ids.tax_ids.repartition_line_ids.tag_ids
                intracom = bool(tax_tags & (mod_303_10 + mod_303_11))
                if intracom:
                    invoice_node['ClaveRegimenEspecialOTrascendencia'] = '09'
                elif reagyp:
                    invoice_node['ClaveRegimenEspecialOTrascendencia'] = '02'
                else:
                    invoice_node['ClaveRegimenEspecialOTrascendencia'] = '01'

            if invoice.move_type == 'out_invoice':
                invoice_node['TipoFactura'] = 'F2' if is_simplified else 'F1'
            elif invoice.move_type == 'out_refund':
                invoice_node['TipoFactura'] = 'R5' if is_simplified else 'R1'
                invoice_node['TipoRectificativa'] = 'I'
            elif invoice.move_type == 'in_invoice':
                if reagyp:
                    invoice_node['TipoFactura'] = 'F6'
                elif invoice._l10n_es_is_dua():
                    invoice_node['TipoFactura'] = 'F5'
                else:
                    invoice_node['TipoFactura'] = 'F1'
            elif invoice.move_type == 'in_refund':
                invoice_node['TipoFactura'] = 'R4'
                invoice_node['TipoRectificativa'] = 'I'

            # === Taxes ===

            sign = -1 if invoice.move_type in ('out_refund', 'in_refund') else 1

            if invoice.is_sale_document():
                # Customer invoices

                if (com_partner.country_id.code in ('ES', False) and not (com_partner.vat or '').startswith("ESN")) or is_simplified:
                    tax_details_info_vals = self._l10n_es_edi_get_invoices_tax_details_info(invoice)
                    invoice_node['TipoDesglose'] = {'DesgloseFactura': tax_details_info_vals['tax_details_info']}

                    invoice_node['ImporteTotal'] = round(sign * (
                        tax_details_info_vals['tax_details']['base_amount']
                        + tax_details_info_vals['tax_details']['tax_amount']
                        - tax_details_info_vals['tax_amount_retention']
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

                    if tax_details_info_service_vals['tax_details_info']:
                        invoice_node.setdefault('TipoDesglose', {})
                        invoice_node['TipoDesglose'].setdefault('DesgloseTipoOperacion', {})
                        invoice_node['TipoDesglose']['DesgloseTipoOperacion']['PrestacionServicios'] = tax_details_info_service_vals['tax_details_info']
                    if tax_details_info_consu_vals['tax_details_info']:
                        invoice_node.setdefault('TipoDesglose', {})
                        invoice_node['TipoDesglose'].setdefault('DesgloseTipoOperacion', {})
                        invoice_node['TipoDesglose']['DesgloseTipoOperacion']['Entrega'] = tax_details_info_consu_vals['tax_details_info']

                    invoice_node['ImporteTotal'] = round(sign * (
                        tax_details_info_service_vals['tax_details']['base_amount']
                        + tax_details_info_service_vals['tax_details']['tax_amount']
                        - tax_details_info_service_vals['tax_amount_retention']
                        + tax_details_info_consu_vals['tax_details']['base_amount']
                        + tax_details_info_consu_vals['tax_details']['tax_amount']
                        - tax_details_info_consu_vals['tax_amount_retention']
                    ), 2)

            else:
                # Vendor bills

                tax_details_info_isp_vals = self._l10n_es_edi_get_invoices_tax_details_info(
                    invoice,
                    filter_invl_to_apply=lambda x: any(t for t in x.tax_ids if t.l10n_es_type == 'sujeto_isp'),
                )
                tax_details_info_other_vals = self._l10n_es_edi_get_invoices_tax_details_info(
                    invoice,
                    filter_invl_to_apply=lambda x: not any(t for t in x.tax_ids if t.l10n_es_type == 'sujeto_isp'),
                )

                invoice_node['DesgloseFactura'] = {}
                if tax_details_info_isp_vals['tax_details_info']:
                    invoice_node['DesgloseFactura']['InversionSujetoPasivo'] = tax_details_info_isp_vals['tax_details_info']
                if tax_details_info_other_vals['tax_details_info']:
                    invoice_node['DesgloseFactura']['DesgloseIVA'] = tax_details_info_other_vals['tax_details_info']

                if invoice._l10n_es_is_dua() or any(t.l10n_es_type == 'ignore' for t in invoice.invoice_line_ids.tax_ids):
                    invoice_node['ImporteTotal'] = round(sign * (
                            tax_details_info_isp_vals['tax_details']['base_amount']
                            + tax_details_info_isp_vals['tax_details']['tax_amount']
                            + tax_details_info_other_vals['tax_details']['base_amount']
                            + tax_details_info_other_vals['tax_details']['tax_amount']
                    ), 2)
                else: # Intra-community -100 repartition line needs to be taken into account
                    invoice_node['ImporteTotal'] = round(-invoice.amount_total_signed
                                                         - sign * tax_details_info_isp_vals['tax_amount_retention']
                                                         - sign * tax_details_info_other_vals['tax_amount_retention'], 2)

                invoice_node['CuotaDeducible'] = round(sign * (
                    tax_details_info_isp_vals['tax_amount_deductible']
                    + tax_details_info_other_vals['tax_amount_deductible']
                ), 2)

            info_list.append(info)
        return info_list

    def _l10n_es_edi_web_service_aeat_vals(self, invoices):
        if invoices[0].is_sale_document():
            return {
                'url': 'https://www2.agenciatributaria.gob.es/static_files/common/internet/dep/aplicaciones/es/aeat/ssii_1_1/fact/ws/SuministroFactEmitidas.wsdl',
                'test_url': 'https://prewww1.aeat.es/wlpl/SSII-FACT/ws/fe/SiiFactFEV1SOAP',
            }
        else:
            return {
                'url': 'https://www2.agenciatributaria.gob.es/static_files/common/internet/dep/aplicaciones/es/aeat/ssii_1_1/fact/ws/SuministroFactRecibidas.wsdl',
                'test_url': 'https://prewww1.aeat.es/wlpl/SSII-FACT/ws/fr/SiiFactFRV1SOAP',
            }

    def _l10n_es_edi_web_service_bizkaia_vals(self, invoices):
        if invoices[0].is_sale_document():
            return {
                'url': 'https://www.bizkaia.eus/ogasuna/sii/documentos/SuministroFactEmitidas.wsdl',
                'test_url': 'https://pruapps.bizkaia.eus/SSII-FACT/ws/fe/SiiFactFEV1SOAP',
            }
        else:
            return {
                'url': 'https://www.bizkaia.eus/ogasuna/sii/documentos/SuministroFactRecibidas.wsdl',
                'test_url': 'https://pruapps.bizkaia.eus/SSII-FACT/ws/fr/SiiFactFRV1SOAP',
            }

    def _l10n_es_edi_web_service_gipuzkoa_vals(self, invoices):
        if invoices[0].is_sale_document():
            return {
                'url': 'https://egoitza.gipuzkoa.eus/ogasuna/sii/ficheros/v1.1/SuministroFactEmitidas.wsdl',
                'test_url': 'https://sii-prep.egoitza.gipuzkoa.eus/JBS/HACI/SSII-FACT/ws/fe/SiiFactFEV1SOAP',
            }
        else:
            return {
                'url': 'https://egoitza.gipuzkoa.eus/ogasuna/sii/ficheros/v1.1/SuministroFactRecibidas.wsdl',
                'test_url': 'https://sii-prep.egoitza.gipuzkoa.eus/JBS/HACI/SSII-FACT/ws/fr/SiiFactFRV1SOAP',
            }

    def _l10n_es_edi_call_web_service_sign(self, invoices, info_list):
        return self._l10n_es_edi_call_web_service_sign_common(invoices, info_list)

    def _l10n_es_edi_call_web_service_sign_common(self, invoices, info_list, cancel=False):
        company = invoices.company_id

        # All are sharing the same value.
        csv_number = invoices.mapped('l10n_es_edi_csv')[0]

        # Set registration date
        invoices.filtered(lambda inv: not inv.l10n_es_registration_date).write({
            'l10n_es_registration_date': fields.Date.context_today(self),
        })

        # === Call the web service ===

        # Get connection data.
        l10n_es_edi_tax_agency = company.mapped('l10n_es_edi_tax_agency')[0]
        connection_vals = getattr(self, f'_l10n_es_edi_web_service_{l10n_es_edi_tax_agency}_vals')(invoices)

        header = {
            'IDVersionSii': '1.1',
            'Titular': {
                'NombreRazon': company.name[:120],
                'NIF': company.vat[2:] if company.vat.startswith('ES') else company.vat,
            },
            'TipoComunicacion': 'A1' if csv_number else 'A0',
        }

        session = requests.Session()
        session.cert = company.l10n_es_edi_certificate_id
        session.mount('https://', PatchedHTTPAdapter())

        client = zeep.Client(connection_vals['url'], operation_timeout=60, timeout=60, session=session)

        if invoices[0].is_sale_document():
            service_name = 'SuministroFactEmitidas'
        else:
            service_name = 'SuministroFactRecibidas'
        if company.l10n_es_edi_test_env and not connection_vals.get('test_url'):
            service_name += 'Pruebas'

        # Establish the connection.
        serv = client.bind('siiService', service_name)
        if company.l10n_es_edi_test_env and connection_vals.get('test_url'):
            serv._binding_options['address'] = connection_vals['test_url']

        error_msg = None
        try:
            if cancel:
                if invoices[0].is_sale_document():
                    res = serv.AnulacionLRFacturasEmitidas(header, info_list)
                else:
                    res = serv.AnulacionLRFacturasRecibidas(header, info_list)
            else:
                if invoices[0].is_sale_document():
                    res = serv.SuministroLRFacturasEmitidas(header, info_list)
                else:
                    res = serv.SuministroLRFacturasRecibidas(header, info_list)
        except requests.exceptions.SSLError as error:
            error_msg = _("The SSL certificate could not be validated.")
        except (zeep.exceptions.Error, requests.exceptions.ConnectionError) as error:
            error_msg = _("Networking error:\n%s", error)
        except Exception as error:
            error_msg = str(error)

        if error_msg:
            return {inv: {
                'error': error_msg,
                'blocking_level': 'warning',
            } for inv in invoices}

        # Process response.

        if not res or not res.RespuestaLinea:
            return {inv: {
                'error': _("The web service is not responding"),
                'blocking_level': 'warning',
            } for inv in invoices}

        resp_state = res["EstadoEnvio"]
        l10n_es_edi_csv = res['CSV']

        if resp_state == 'Correcto':
            invoices.write({'l10n_es_edi_csv': l10n_es_edi_csv})
            return {inv: {'success': True} for inv in invoices}

        results = {}
        for respl in res.RespuestaLinea:
            invoice_number = respl.IDFactura.NumSerieFacturaEmisor

            # Retrieve the corresponding invoice.
            # Note: ref can be the same for different partners but there is no enough information on the response
            # to match the partner.

            # Note: Invoices are batched per move_type.
            if invoices[0].is_sale_document():
                inv = invoices.filtered(lambda x: x.name[:60] == invoice_number)
            else:
                # 'ref' can be the same for different partners.
                candidates = invoices.filtered(lambda x: x.ref[:60] == invoice_number)
                if len(candidates) > 1:
                    respl_partner_info = respl.IDFactura.IDEmisorFactura
                    inv = None
                    for candidate in candidates:
                        partner = candidate.commercial_partner_id
                        if candidate._l10n_es_is_dua():
                            partner = candidate.company_id.partner_id
                        partner_info = self._l10n_es_edi_get_partner_info(partner)
                        if partner_info.get('NIF') and partner_info['NIF'] == respl_partner_info.NIF:
                            inv = candidate
                            break
                        if (
                            partner_info.get('IDOtro')
                            and respl_partner_info['IDOtro']
                            and all(respl_partner_info['IDOtro'][k] == v for k, v in partner_info['IDOtro'].items())
                        ):
                            inv = candidate
                            break

                    if not inv:
                        # This case shouldn't happen and means there is something wrong in this code. However, we can't
                        # raise anything since the document has already been approved by the government. The result
                        # will only be a badly logged message into the chatter so, not a big deal.
                        inv = candidates[0]
                else:
                    inv = candidates

            resp_line_state = respl.EstadoRegistro
            respl_dict = dict(respl)
            if resp_line_state in ('Correcto', 'AceptadoConErrores'):
                inv.l10n_es_edi_csv = l10n_es_edi_csv
                results[inv] = {'success': True}
                if resp_line_state == 'AceptadoConErrores':
                    inv.message_post(body=_("This was accepted with errors: ") + html_escape(respl.DescripcionErrorRegistro))
            elif (
                (respl_dict.get('RegistroDuplicado') and respl.RegistroDuplicado.EstadoRegistro == 'Correcta')
                or
                (cancel and respl_dict.get('CodigoErrorRegistro') == 3001)
            ):
                results[inv] = {'success': True}
                inv.message_post(body=_("We saw that this invoice was sent correctly before, but we did not treat "
                                        "the response.  Make sure it is not because of a wrong configuration."))

            elif respl.CodigoErrorRegistro == 1117 and not self.env.context.get('error_1117'):
                return self.with_context(error_1117=True)._l10n_es_edi_sii_post_invoices(invoices)


            else:
                results[inv] = {
                    'error': _("[%s] %s", respl.CodigoErrorRegistro, respl.DescripcionErrorRegistro),
                    'blocking_level': 'error',
                }

        return results

    def _has_oss_taxes(self, invoice):
        if self.env['ir.module.module'].sudo().search([('name', '=', 'l10n_eu_oss'), ('state', '=', 'installed')]):
            oss_tag = self.env.ref('l10n_eu_oss.tag_oss')
            lines = invoice.invoice_line_ids.filtered(lambda line: line.display_type not in ('line_section', 'line_note'))
            tax_tags = lines.mapped('tax_ids.invoice_repartition_line_ids.tag_ids')
            return oss_tag in tax_tags
        return False

    # -------------------------------------------------------------------------
    # EDI OVERRIDDEN METHODS
    # -------------------------------------------------------------------------

    def _l10n_es_edi_sii_xml_invoice_content(self, invoice):
        return json.dumps(self._l10n_es_edi_get_invoices_info(invoice)).encode()

    def _get_move_applicability(self, move):
        # EXTENDS account_edi
        self.ensure_one()
        if self.code != 'es_sii':
            return super()._get_move_applicability(move)

        if move.l10n_es_edi_is_required:
            return {
                'post': self._l10n_es_edi_sii_post_invoices,
                'post_batching': lambda invoice: (invoice.move_type, invoice.l10n_es_edi_csv),
                'edi_content': self._l10n_es_edi_sii_xml_invoice_content,
                'cancel': self._l10n_es_edi_sii_cancel_invoices,
            }

    def _needs_web_services(self):
        # OVERRIDE
        return self.code == 'es_sii' or super()._needs_web_services()

    def _check_move_configuration(self, move):
        # OVERRIDE
        res = super()._check_move_configuration(move)
        if self.code != 'es_sii':
            return res

        if not move.company_id.vat:
            res.append(_("VAT number is missing on company %s", move.company_id.display_name))
        for line in move.invoice_line_ids.filtered(lambda line: line.display_type not in ('line_note', 'line_section')):
            taxes = line.tax_ids.flatten_taxes_hierarchy()
            recargo_count = taxes.mapped('l10n_es_type').count('recargo')
            retention_count = taxes.mapped('l10n_es_type').count('retencion')
            sujeto_count = taxes.mapped('l10n_es_type').count('sujeto')
            no_sujeto_count = taxes.mapped('l10n_es_type').count('no_sujeto')
            no_sujeto_loc_count = taxes.mapped('l10n_es_type').count('no_sujeto_loc')
            if retention_count > 1:
                res.append(_("Line %s should only have one retention tax.", line.display_name))
            if recargo_count > 1:
                res.append(_("Line %s should only have one recargo tax.", line.display_name))
            if sujeto_count > 1:
                res.append(_("Line %s should only have one sujeto tax.", line.display_name))
            if no_sujeto_count > 1:
                res.append(_("Line %s should only have one no sujeto tax.", line.display_name))
            if no_sujeto_loc_count > 1:
                res.append(_("Line %s should only have one no sujeto (localizations) tax.", line.display_name))
            if sujeto_count + no_sujeto_loc_count + no_sujeto_count > 1:
                res.append(_("Line %s should only have one main tax.", line.display_name))
        if move.move_type in ('in_invoice', 'in_refund'):
            if not move.ref:
                res.append(_("You should put a vendor reference on this vendor bill. "))
        return res

    def _is_compatible_with_journal(self, journal):
        # OVERRIDE
        if self.code != 'es_sii':
            return super()._is_compatible_with_journal(journal)

        return journal.country_code == 'ES'

    def _l10n_es_edi_sii_send(self, invoices, cancel=False):
        # Ensure a certificate is available.
        certificate = invoices.company_id.l10n_es_edi_certificate_id
        if not certificate:
            return {inv: {
                'error': _("Please configure the certificate for SII."),
                'blocking_level': 'error',
            } for inv in invoices}

        # Ensure a tax agency is available.
        l10n_es_edi_tax_agency = invoices.company_id.mapped('l10n_es_edi_tax_agency')[0]
        if not l10n_es_edi_tax_agency:
            return {inv: {
                'error': _("Please specify a tax agency on your company for SII."),
                'blocking_level': 'error',
            } for inv in invoices}

        # Generate the JSON.
        info_list = self._l10n_es_edi_get_invoices_info(invoices)

        # Call the web service.
        if not cancel: #retrocompatibility and mocks in tests
            res = self._l10n_es_edi_call_web_service_sign(invoices, info_list)
        else:
            res = self._l10n_es_edi_call_web_service_sign_common(invoices, info_list, cancel=True)

        for inv in invoices:
            if res.get(inv, {}).get('success'):
                attachment = self.env['ir.attachment'].create({
                    'type': 'binary',
                    'name': 'jsondump.json',
                    'raw': json.dumps(info_list),
                    'mimetype': 'application/json',
                    'res_model': inv._name,
                    'res_id': inv.id,
                })
                res[inv]['attachment'] = attachment
                if cancel:
                    inv.l10n_es_edi_csv = False
        return res

    def _l10n_es_edi_sii_post_invoices(self, invoices):
        return self._l10n_es_edi_sii_send(invoices)

    def _l10n_es_edi_sii_cancel_invoices(self, invoices):
        return self._l10n_es_edi_sii_send(invoices, cancel=True)

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_es_edi_is_required = fields.Boolean(
        string="Is the Spanish EDI needed",
        compute='_compute_l10n_es_edi_is_required'
    )
    l10n_es_edi_csv = fields.Char(string="CSV return code", copy=False, tracking=True)
    # Technical field to keep the date the invoice was sent the first time as
    # the date the invoice was registered into the system.
    l10n_es_registration_date = fields.Date(
        string="Registration Date", copy=False,
    )

    # -------------------------------------------------------------------------
    # COMPUTE METHODS
    # -------------------------------------------------------------------------

    @api.depends('move_type', 'company_id', 'invoice_line_ids.tax_ids')
    def _compute_l10n_es_edi_is_required(self):
        for move in self:
            has_tax = True
            # Check it is not an importation invoice (which will be report through the DUA invoice)
            if move.is_purchase_document():
                taxes = move.invoice_line_ids.tax_ids
                has_tax = any(t.l10n_es_type and t.l10n_es_type != 'ignore' for t in taxes)
            move.l10n_es_edi_is_required = move.is_invoice() \
                                           and move.country_code == 'ES' \
                                           and move.company_id.l10n_es_edi_tax_agency \
                                           and has_tax

    def _l10n_es_is_dua(self):
        self.ensure_one()
        return any(t.l10n_es_type == 'dua' for t in self.invoice_line_ids.tax_ids.flatten_taxes_hierarchy())

    def _check_edi_documents_for_reset_to_draft(self):
        docs = self.edi_document_ids.filtered(lambda d: d.edi_format_id._needs_web_services())
        if len(docs) == 1 and docs.edi_format_id.code == 'es_sii' and docs.state != 'to_cancel':
            return True
        return super()._check_edi_documents_for_reset_to_draft()

    def _edi_allow_button_draft(self):
        docs = self.edi_document_ids.filtered(lambda d: d.edi_format_id._needs_web_services())
        if len(docs) == 1 and docs.edi_format_id.code == 'es_sii' and docs.state != 'to_cancel':
            return True
        return super()._edi_allow_button_draft()

```

## File: models\account_move_send.py

```python
from odoo import models


class AccountMoveSend(models.TransientModel):
    _inherit = 'account.move.send'

    def _get_mail_attachment_from_doc(self, doc):
        if doc.name == 'jsondump.json' and doc.edi_format_id.code == 'es_sii':
            return self.env['ir.attachment']
        return super()._get_mail_attachment_from_doc(doc)

```

## File: models\l10n_es_edi_certificate.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from base64 import b64decode
from pytz import timezone
from datetime import datetime
from cryptography.hazmat.primitives.serialization import Encoding, NoEncryption, PrivateFormat


from odoo import _, api, fields, models, tools
from odoo.exceptions import ValidationError
from odoo.addons.account.tools.certificate import load_key_and_certificates


class Certificate(models.Model):
    _name = 'l10n_es_edi.certificate'
    _description = 'Personal Digital Certificate'
    _order = 'date_start desc, id desc'
    _rec_name = 'date_start'

    content = fields.Binary(string="File", required=True, help="PFX Certificate")
    password = fields.Char(help="Passphrase for the PFX certificate", groups="base.group_system")
    date_start = fields.Datetime(readonly=True, help="The date on which the certificate starts to be valid")
    date_end = fields.Datetime(readonly=True, help="The date on which the certificate expires")
    company_id = fields.Many2one(comodel_name='res.company', required=True, default=lambda self: self.env.company)

    # -------------------------------------------------------------------------
    # HELPERS
    # -------------------------------------------------------------------------

    @api.model
    def _get_es_current_datetime(self):
        """Get the current datetime with the Peruvian timezone. """
        return datetime.now(timezone('Europe/Madrid'))

    @tools.ormcache('self.content', 'self.password')
    def _decode_certificate(self):
        """Return the content (DER encoded) and the certificate decrypted based in the point 3.1 from the RS 097-2012
        http://www.vauxoo.com/r/manualdeautorizacion#page=21
        """
        self.ensure_one()

        if not self.password:
            return None, None, None

        private_key, certificate = load_key_and_certificates(
            b64decode(self.content),
            self.password.encode(),
        )

        pem_certificate = certificate.public_bytes(Encoding.PEM)
        pem_private_key = private_key.private_bytes(
            Encoding.PEM,
            format=PrivateFormat.TraditionalOpenSSL,
            encryption_algorithm=NoEncryption(),
        )
        return pem_certificate, pem_private_key, certificate

    # -------------------------------------------------------------------------
    # LOW-LEVEL METHODS
    # -------------------------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        certificates = super().create(vals_list)

        spain_tz = timezone('Europe/Madrid')
        spain_dt = self._get_es_current_datetime()
        for certificate in certificates:
            try:
                _pem_certificate, _pem_private_key, certif = certificate._decode_certificate()
                cert_date_start = spain_tz.localize(certif.not_valid_before)
                cert_date_end = spain_tz.localize(certif.not_valid_after)
            except Exception:
                raise ValidationError(_(
                    "There has been a problem with the certificate, some usual problems can be:\n"
                    "- The password given or the certificate are not valid.\n"
                    "- The certificate content is invalid."
                ))
            # Assign extracted values from the certificate
            certificate.write({
                'date_start': fields.Datetime.to_string(cert_date_start),
                'date_end': fields.Datetime.to_string(cert_date_end),
            })
            if spain_dt > cert_date_end:
                raise ValidationError(_("The certificate is expired since %s", certificate.date_end))
        return certificates

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_es_edi_certificate_id = fields.Many2one(
        string="Certificate (ES)",
        store=True,
        readonly=False,
        comodel_name='l10n_es_edi.certificate',
        compute="_compute_l10n_es_edi_certificate",
    )
    l10n_es_edi_certificate_ids = fields.One2many(
        comodel_name='l10n_es_edi.certificate',
        inverse_name='company_id',
    )
    l10n_es_edi_tax_agency = fields.Selection(
        string="Tax Agency for SII",
        selection=[
            ('aeat', "Agencia Tributaria española"),
            ('gipuzkoa', "Hacienda Foral de Gipuzkoa"),
            ('bizkaia', "Hacienda Foral de Bizkaia"),
        ],
        default=False,
    )
    l10n_es_edi_test_env = fields.Boolean(
        string="Test Mode",
        help="Use the test environment",
        default=True,
    )

    @api.depends('country_id', 'l10n_es_edi_certificate_ids')
    def _compute_l10n_es_edi_certificate(self):
        for company in self:
            if company.country_code == 'ES':
                company.l10n_es_edi_certificate_id = self.env['l10n_es_edi.certificate'].search(
                    [('company_id', '=', company.id)],
                    order='date_end desc',
                    limit=1,
                )
            else:
                company.l10n_es_edi_certificate_id = False

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_es_edi_certificate_ids = fields.One2many(related='company_id.l10n_es_edi_certificate_ids', readonly=False)
    l10n_es_edi_tax_agency = fields.Selection(related='company_id.l10n_es_edi_tax_agency', readonly=False)
    l10n_es_edi_test_env = fields.Boolean(related='company_id.l10n_es_edi_test_env', readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_edi_format
from . import account_move_send
from . import account_move
from . import l10n_es_edi_certificate
from . import res_company
from . import res_config_settings

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_es_edi_certificate,access_l10n_es_edi_certificate,model_l10n_es_edi_certificate,base.group_system,1,1,1,1

```

## File: security\l10n_es_edi_certificate.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!-- Allow access to certificates of current company/companies -->
    <record id="l10n_ec_digital_certificate" model="ir.rule">
        <field name="name">Spanish Digital Certificate</field>
        <field name="model_id" ref="l10n_es_edi_sii.model_l10n_es_edi_certificate"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>
</odoo>

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_move_form_inherit_l10n_es_edi" model="ir.ui.view">
            <field name="name">account.move.form.inherit.l10n_es_edi</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@id='other_tab_group']//group[@name='accounting_info_group']" position="inside">
                    <field name="l10n_es_registration_date" readonly="state == 'posted'" invisible="country_code != 'ES'"/>
                    <field name="l10n_es_edi_csv" invisible="1"/>
                </xpath>
                <field name="ref" position="attributes">
                    <attribute name="readonly">l10n_es_edi_csv</attribute>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\l10n_es_edi_certificate_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>

        <record id="l10n_es_edi_certificate_form" model="ir.ui.view">
            <field name="name">l10n_es_edi.certificate.form</field>
            <field name="model">l10n_es_edi.certificate</field>
            <field name="arch" type="xml">
                <form>
                    <sheet>
                        <group>
                            <field name="content"/>
                            <field name="password" password="True"/>
                            <label for="date_start" string="Validity"/>
                            <div>
                                <field name="date_start"/> -
                                <field name="date_end"/>
                            </div>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="l10n_es_edi_certificate_tree" model="ir.ui.view">
            <field name="name">l10n_es_edi.certificate.tree</field>
            <field name="model">l10n_es_edi.certificate</field>
            <field name="arch" type="xml">
                <tree>
                    <field name="date_start"/>
                    <field name="date_end"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id="l10n_es_edi_certificate_action" model="ir.actions.act_window">
            <field name="name">Certificates for EDI invoices on Spain</field>
            <field name="res_model">l10n_es_edi.certificate</field>
            <field name="view_mode">tree,form</field>
            <field name="help" type="html">
                <p class="oe_view_nocontent_create">Create the first certificate</p>
            </field>
        </record>

        <menuitem id="menu_l10n_es_edi_root"
                  name="Spain"
                  sequence="110"
                  groups="account.group_account_manager"
                  parent="account.menu_finance_configuration">
            <menuitem id="menu_l10n_es_edi_certificates"
                      name="Certificates (ES)"
                      action="l10n_es_edi_certificate_action"
                      sequence="100"
                      groups="account.group_account_manager"/>
        </menuitem>

    </data>
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
            <xpath expr="//app[@name='account']/block" position="after">
                <block title="Spain Localization" name="spain_localization" invisible="country_code != 'ES'">
                    <!-- Invisible fields -->
                    <field name="l10n_es_edi_certificate_ids" invisible="1"/>
                    <setting string="Registro de Libros connection SII" company_dependent="1">
                        <div class="content-group">
                            <div class="mt16">
                                <label for="l10n_es_edi_tax_agency" class="o_light_label"/>
                                <field name="l10n_es_edi_tax_agency"/>
                                <div class="text-muted" invisible="l10n_es_edi_tax_agency">
                                    No tax agency selected: SII not activated.
                                </div>
                                <div class="text-muted" invisible="not l10n_es_edi_tax_agency">
                                    Tax agency selected: invoices will be sent by SII for journals where it is activated.
                                </div>
                                <br/>
                                <div class="o_row">
                                    <label for="l10n_es_edi_test_env" class="o_light_label"/>
                                    <field name="l10n_es_edi_test_env"/>
                                </div>
                                <div class="text-muted" invisible="not l10n_es_edi_test_env">
                                    Test mode: EDI data is sent to separate test servers and is not considered official.
                                </div>
                                <div class="text-muted" invisible="l10n_es_edi_test_env">
                                    Production mode: EDI data is sent to the official agency servers.
                                </div>
                                <br/>
                                <div>
                                    <button name="%(l10n_es_edi_certificate_action)d" type="action" class="oe_link">Manage certificates (SII/TicketBAI)</button>
                                </div>
                            </div>
                        </div>
                    </setting>
                </block>
            </xpath>

        </field>
    </record>
</odoo>

```

