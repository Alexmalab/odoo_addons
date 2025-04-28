# Odoo Module: l10n_ar

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Argentina - Accounting',
    'version': "3.5",
    'description': """
Functional
----------

This module add accounting features for the Argentinian localization, which represent the minimal configuration needed for a company  to operate in Argentina and under the AFIP (Administración Federal de Ingresos Públicos) regulations and guidelines.

Follow the next configuration steps for Production:

1. Go to your company and configure your VAT number and AFIP Responsibility Type
2. Go to Accounting / Settings and set the Chart of Account that you will like to use.
3. Create your Sale journals taking into account AFIP POS info.

Demo data for testing:

* 3 companies were created, one for each AFIP responsibility type with the respective Chart of Account installed. Choose the company that fix you in order to make tests:

  * (AR) Responsable Inscripto
  * (AR) Exento
  * (AR) Monotributo

* Journal sales configured to Pre printed and Expo invoices in all companies
* Invoices and other documents examples already validated in “(AR) Responsable Inscripto” company
* Partners example for the different responsibility types:

  * ADHOC (IVA Responsable Inscripto)
  * Consejo Municipal Rosario (IVA Sujeto Exento)
  * Gritti (Monotributo)
  * Cerro Castor. IVA Liberado in Zona Franca
  * Expresso (Cliente del Exterior)
  * Odoo (Proveedor del Exterior)

Highlights:

* Chart of account will not be automatically installed, each CoA Template depends on the AFIP Responsibility of the company, you will need to install the CoA for your needs.
* No sales journals will be generated when installing a CoA, you will need to configure your journals manually.
* The Document type will be properly pre selected when creating an invoice depending on the fiscal responsibility of the issuer and receiver of the document and the related journal.
* A CBU account type has been added and also CBU Validation


Technical
---------

This module adds both models and fields that will be eventually used for the electronic invoice module. Here is a summary of the main features:

Master Data:

* Chart of Account: one for each AFIP responsibility that is related to a legal entity:

  * Responsable Inscripto (RI)
  * Exento (EX)
  * Monotributo (Mono)

* Argentinian Taxes and Account Tax Groups (VAT taxes with the existing aliquots and other types)
* AFIP Responsibility Types
* Fiscal Positions (in order to map taxes)
* Legal Documents Types in Argentina
* Identification Types valid in Argentina.
* Country AFIP codes and Country VAT codes for legal entities, natural persons and others
* Currency AFIP codes
* Unit of measures AFIP codes
* Partners: Consumidor Final and AFIP
""",
    'author': 'ADHOC SA',
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'l10n_latam_invoice_document',
        'l10n_latam_base',
    ],
    'data': [
        'security/ir.model.access.csv',
        'data/l10n_latam_identification_type_data.xml',
        'data/l10n_ar_afip_responsibility_type_data.xml',
        'data/account_chart_template_data.xml',
        'data/account.group.template.csv',
        'data/account.account.template.csv',
        'data/account_chart_template_data2.xml',
        'data/account_tax_group.xml',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_template.xml',
        'data/uom_uom_data.xml',
        'data/l10n_latam.document.type.csv',
        'data/l10n_latam.document.type.xml',
        'data/res_partner_data.xml',
        'data/res.currency.csv',
        'data/res.country.csv',
        'views/l10n_ar.xml',
        'views/account_move_view.xml',
        'views/res_partner_view.xml',
        'views/res_company_view.xml',
        'views/res_country_view.xml',
        'views/afip_menuitem.xml',
        'views/l10n_ar_afip_responsibility_type_view.xml',
        'views/res_currency_view.xml',
        'views/account_fiscal_position_view.xml',
        'views/uom_uom_view.xml',
        'views/account_journal_view.xml',
        'views/l10n_latam_document_type_view.xml',
        'views/report_invoice.xml',
        'report/invoice_report_view.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'demo': [
        # we create demo data on different companies (not main_company) to
        # allow different setups and also to allow multi-localization demo data
        'demo/exento_demo.xml',
        'demo/mono_demo.xml',
        'demo/respinsc_demo.xml',
        'demo/res_partner_demo.xml',
        'demo/product_product_demo.xml',
        'demo/account_customer_invoice_demo.xml',
        'demo/account_customer_refund_demo.xml',
        'demo/account_supplier_invoice_demo.xml',
        'demo/account_supplier_refund_demo.xml',
        # restore
        'demo/res_users_demo.xml',
    ],
    'installable': True,
    'auto_install': False,
    'application': False,
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,chart_template_id/id,code,user_type_id/id,name,reconcile
l10n_ar.base_cheques_de_terceros,l10n_ar.l10nar_base_chart_template,1.1.1.02.010,account.data_account_type_liquidity,Cheques de Terceros ,False
l10n_ar.base_cheques_de_terceros_rechazados,l10n_ar.l10nar_base_chart_template,1.1.1.02.020,account.data_account_type_current_assets,Cheques de Terceros Rechazados,False
l10n_ar.base_fondo_comun_de_inversion,l10n_ar.l10nar_base_chart_template,1.1.2.01.010,account.data_account_type_current_assets,Fondo común de inversión,False
l10n_ar.base_plazo_fijo,l10n_ar.l10nar_base_chart_template,1.1.2.01.020,account.data_account_type_current_assets,Plazo fijo,False
l10n_ar.base_deudores_por_ventas,l10n_ar.l10nar_base_chart_template,1.1.3.01.010,account.data_account_type_receivable,Deudores por ventas,True
l10n_ar.base_deudores_por_ventas_pos,l10n_ar.l10nar_base_chart_template,1.1.3.01.020,account.data_account_type_receivable,Deudores por ventas (PoS),True
l10n_ar.base_ret_percepcion_tasa_municipal,l10n_ar.l10nar_base_chart_template,1.1.4.01.010,account.data_account_type_current_assets,Ret/Percepción Tasa Municipal,False
l10n_ar.base_saldo_a_favor_tasa_municipal,l10n_ar.l10nar_base_chart_template,1.1.4.01.020,account.data_account_type_current_assets,Saldo a favor Tasa Municipal,False
l10n_ar.base_saldo_favor_iibb_caba,l10n_ar.l10nar_base_chart_template,1.1.4.02.010,account.data_account_type_current_assets,Saldo a favor IIBB CABA,False
l10n_ar.base_retencion_iibb_caba_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.020,account.data_account_type_current_assets,Retención IIBB CABA sufrida,False
l10n_ar.base_percepcion_iibb_caba_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.030,account.data_account_type_current_assets,Percepción IIBB CABA sufrida,False
l10n_ar.base_saldo_favor_iibb_ba,l10n_ar.l10nar_base_chart_template,1.1.4.02.040,account.data_account_type_current_assets,Saldo a favor IIBB Buenos Aires,False
l10n_ar.base_retencion_iibb_ba_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.050,account.data_account_type_current_assets,Retención IIBB Buenos Aires sufrida,False
l10n_ar.base_percepcion_iibb_ba_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.060,account.data_account_type_current_assets,Percepción IIBB Buenos Aires sufrida,False
l10n_ar.base_saldo_favor_iibb_ca,l10n_ar.l10nar_base_chart_template,1.1.4.02.070,account.data_account_type_current_assets,Saldo a favor IIBB Catamarca,False
l10n_ar.base_retencion_iibb_ca_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.080,account.data_account_type_current_assets,Retención IIBB Catamarca sufrida,False
l10n_ar.base_percepcion_iibb_ca_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.090,account.data_account_type_current_assets,Percepción IIBB Catamarca sufrida,False
l10n_ar.base_saldo_favor_iibb_co,l10n_ar.l10nar_base_chart_template,1.1.4.02.100,account.data_account_type_current_assets,Saldo a favor IIBB Córdoba,False
l10n_ar.base_retencion_iibb_co_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.110,account.data_account_type_current_assets,Retención IIBB Córdoba sufrida,False
l10n_ar.base_percepcion_iibb_co_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.120,account.data_account_type_current_assets,Percepción IIBB Córdoba sufrida,False
l10n_ar.base_saldo_favor_iibb_rr,l10n_ar.l10nar_base_chart_template,1.1.4.02.130,account.data_account_type_current_assets,Saldo a favor IIBB Corrientes,False
l10n_ar.base_retencion_iibb_rr_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.140,account.data_account_type_current_assets,Retención IIBB Corrientes sufrida,False
l10n_ar.base_percepcion_iibb_rr_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.150,account.data_account_type_current_assets,Percepción IIBB Corrientes sufrida,False
l10n_ar.base_saldo_favor_iibb_er,l10n_ar.l10nar_base_chart_template,1.1.4.02.160,account.data_account_type_current_assets,Saldo a favor IIBB Entre Ríos,False
l10n_ar.base_retencion_iibb_er_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.170,account.data_account_type_current_assets,Retención IIBB Entre Ríos sufrida,False
l10n_ar.base_percepcion_iibb_er_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.180,account.data_account_type_current_assets,Percepción IIBB Entre Ríos sufrida,False
l10n_ar.base_saldo_favor_iibb_ju,l10n_ar.l10nar_base_chart_template,1.1.4.02.190,account.data_account_type_current_assets,Saldo a favor IIBB Jujuy,False
l10n_ar.base_retencion_iibb_ju_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.200,account.data_account_type_current_assets,Retención IIBB Jujuy sufrida,False
l10n_ar.base_percepcion_iibb_ju_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.210,account.data_account_type_current_assets,Percepción IIBB Jujuy sufrida,False
l10n_ar.base_saldo_favor_iibb_za,l10n_ar.l10nar_base_chart_template,1.1.4.02.220,account.data_account_type_current_assets,Saldo a favor IIBB Mendoza,False
l10n_ar.base_retencion_iibb_za_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.230,account.data_account_type_current_assets,Retención IIBB Mendoza sufrida,False
l10n_ar.base_percepcion_iibb_za_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.240,account.data_account_type_current_assets,Percepción IIBB Mendoza sufrida,False
l10n_ar.base_saldo_favor_iibb_lr,l10n_ar.l10nar_base_chart_template,1.1.4.02.250,account.data_account_type_current_assets,Saldo a favor IIBB La Rioja,False
l10n_ar.base_retencion_iibb_lr_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.260,account.data_account_type_current_assets,Retención IIBB La Rioja sufrida,False
l10n_ar.base_percepcion_iibb_lr_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.270,account.data_account_type_current_assets,Percepción IIBB La Rioja sufrida,False
l10n_ar.base_saldo_favor_iibb_sa,l10n_ar.l10nar_base_chart_template,1.1.4.02.280,account.data_account_type_current_assets,Saldo a favor IIBB Salta,False
l10n_ar.base_retencion_iibb_sa_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.290,account.data_account_type_current_assets,Retención IIBB Salta sufrida,False
l10n_ar.base_percepcion_iibb_sa_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.300,account.data_account_type_current_assets,Percepción IIBB Salta sufrida,False
l10n_ar.base_saldo_favor_iibb_nn,l10n_ar.l10nar_base_chart_template,1.1.4.02.310,account.data_account_type_current_assets,Saldo a favor IIBB San Juan,False
l10n_ar.base_retencion_iibb_nn_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.320,account.data_account_type_current_assets,Retención IIBB San Juan sufrida,False
l10n_ar.base_percepcion_iibb_nn_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.330,account.data_account_type_current_assets,Percepción IIBB San Juan sufrida,False
l10n_ar.base_saldo_favor_iibb_sl,l10n_ar.l10nar_base_chart_template,1.1.4.02.340,account.data_account_type_current_assets,Saldo a favor IIBB San Luis,False
l10n_ar.base_retencion_iibb_sl_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.350,account.data_account_type_current_assets,Retención IIBB San Luis sufrida,False
l10n_ar.base_percepcion_iibb_sl_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.360,account.data_account_type_current_assets,Percepción IIBB San Luis sufrida,False
l10n_ar.base_saldo_favor_iibb_sf,l10n_ar.l10nar_base_chart_template,1.1.4.02.370,account.data_account_type_current_assets,Saldo a favor IIBB Santa Fe,False
l10n_ar.base_retencion_iibb_sf_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.380,account.data_account_type_current_assets,Retención IIBB Santa Fe sufrida,False
l10n_ar.base_percepcion_iibb_sf_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.390,account.data_account_type_current_assets,Percepción IIBB Santa Fe sufrida,False
l10n_ar.base_saldo_favor_iibb_se,l10n_ar.l10nar_base_chart_template,1.1.4.02.400,account.data_account_type_current_assets,Saldo a favor IIBB Santiago del Estero,False
l10n_ar.base_retencion_iibb_se_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.410,account.data_account_type_current_assets,Retención IIBB Santiago del Estero sufrida,False
l10n_ar.base_percepcion_iibb_se_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.420,account.data_account_type_current_assets,Percepción IIBB Santiago del Estero sufrida,False
l10n_ar.base_saldo_favor_iibb_tn,l10n_ar.l10nar_base_chart_template,1.1.4.02.430,account.data_account_type_current_assets,Saldo a favor IIBB Tucumán,False
l10n_ar.base_retencion_iibb_tn_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.440,account.data_account_type_current_assets,Retención IIBB Tucumán sufrida,False
l10n_ar.base_percepcion_iibb_tn_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.450,account.data_account_type_current_assets,Percepción IIBB Tucumán sufrida,False
l10n_ar.base_saldo_favor_iibb_ha,l10n_ar.l10nar_base_chart_template,1.1.4.02.460,account.data_account_type_current_assets,Saldo a favor IIBB Chaco,False
l10n_ar.base_retencion_iibb_ha_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.470,account.data_account_type_current_assets,Retención IIBB Chaco sufrida,False
l10n_ar.base_percepcion_iibb_ha_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.480,account.data_account_type_current_assets,Percepción IIBB Chaco sufrida,False
l10n_ar.base_saldo_favor_iibb_ct,l10n_ar.l10nar_base_chart_template,1.1.4.02.490,account.data_account_type_current_assets,Saldo a favor IIBB Chubut,False
l10n_ar.base_retencion_iibb_ct_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.500,account.data_account_type_current_assets,Retención IIBB Chubut sufrida,False
l10n_ar.base_percepcion_iibb_ct_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.510,account.data_account_type_current_assets,Percepción IIBB Chubut sufrida,False
l10n_ar.base_saldo_favor_iibb_fo,l10n_ar.l10nar_base_chart_template,1.1.4.02.520,account.data_account_type_current_assets,Saldo a favor IIBB Formosa,False
l10n_ar.base_retencion_iibb_fo_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.530,account.data_account_type_current_assets,Retención IIBB Formosa sufrida,False
l10n_ar.base_percepcion_iibb_fo_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.540,account.data_account_type_current_assets,Percepción IIBB Formosa sufrida,False
l10n_ar.base_saldo_favor_iibb_mi,l10n_ar.l10nar_base_chart_template,1.1.4.02.550,account.data_account_type_current_assets,Saldo a favor IIBB Misiones,False
l10n_ar.base_retencion_iibb_mi_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.560,account.data_account_type_current_assets,Retención IIBB Misiones sufrida,False
l10n_ar.base_percepcion_iibb_mi_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.570,account.data_account_type_current_assets,Percepción IIBB Misiones sufrida,False
l10n_ar.base_saldo_favor_iibb_ne,l10n_ar.l10nar_base_chart_template,1.1.4.02.580,account.data_account_type_current_assets,Saldo a favor IIBB Neuquén,False
l10n_ar.base_retencion_iibb_ne_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.590,account.data_account_type_current_assets,Retención IIBB Neuquén sufrida,False
l10n_ar.base_percepcion_iibb_ne_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.600,account.data_account_type_current_assets,Percepción IIBB Neuquén sufrida,False
l10n_ar.base_saldo_favor_iibb_lp,l10n_ar.l10nar_base_chart_template,1.1.4.02.610,account.data_account_type_current_assets,Saldo a favor IIBB La Pampa,False
l10n_ar.base_retencion_iibb_lp_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.620,account.data_account_type_current_assets,Retención IIBB La Pampa sufrida,False
l10n_ar.base_percepcion_iibb_lp_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.630,account.data_account_type_current_assets,Percepción IIBB La Pampa sufrida,False
l10n_ar.base_saldo_favor_iibb_rn,l10n_ar.l10nar_base_chart_template,1.1.4.02.640,account.data_account_type_current_assets,Saldo a favor IIBB Río Negro,False
l10n_ar.base_retencion_iibb_rn_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.650,account.data_account_type_current_assets,Retención IIBB Río Negro sufrida,False
l10n_ar.base_percepcion_iibb_rn_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.660,account.data_account_type_current_assets,Percepción IIBB Río Negro sufrida,False
l10n_ar.base_saldo_favor_iibb_az,l10n_ar.l10nar_base_chart_template,1.1.4.02.670,account.data_account_type_current_assets,Saldo a favor IIBB Santa Cruz,False
l10n_ar.base_retencion_iibb_az_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.680,account.data_account_type_current_assets,Retención IIBB Santa Cruz sufrida,False
l10n_ar.base_percepcion_iibb_az_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.690,account.data_account_type_current_assets,Percepción IIBB Santa Cruz sufrida,False
l10n_ar.base_saldo_favor_iibb_tf,l10n_ar.l10nar_base_chart_template,1.1.4.02.700,account.data_account_type_current_assets,Saldo a favor IIBB Tierra del Fuego,False
l10n_ar.base_retencion_iibb_tf_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.710,account.data_account_type_current_assets,Retención IIBB Tierra del Fuego sufrida,False
l10n_ar.base_percepcion_iibb_tf_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.02.720,account.data_account_type_current_assets,Percepción IIBB Tierra del Fuego sufrida,False
l10n_ar.base_sircreb,l10n_ar.l10nar_base_chart_template,1.1.4.02.730,account.data_account_type_current_assets,SIRCREB,False
l10n_ar.base_saldo_a_favor_suss,l10n_ar.l10nar_base_chart_template,1.1.4.03.010,account.data_account_type_current_assets,Saldo a favor SUSS,False
l10n_ar.base_retencion_suss_sufrida,l10n_ar.l10nar_base_chart_template,1.1.4.03.020,account.data_account_type_current_assets,Retención SUSS Sufrida,False
l10n_ar.ri_iva_credito_fiscal,l10n_ar.l10nar_ri_chart_template,1.1.4.04.010,account.data_account_type_current_assets,IVA crédito fiscal,False
l10n_ar.ri_percepcion_iva_sufrida,l10n_ar.l10nar_ri_chart_template,1.1.4.04.020,account.data_account_type_current_assets,Percepción IVA Sufrida,False
l10n_ar.ri_retencion_iva_sufrida,l10n_ar.l10nar_ri_chart_template,1.1.4.04.030,account.data_account_type_current_assets,Retención IVA Sufrida,False
l10n_ar.ri_iva_saldo_tecnico_favor,l10n_ar.l10nar_ri_chart_template,1.1.4.04.040,account.data_account_type_current_assets,IVA Saldo técnico,False
l10n_ar.ri_iva_saldo_libre_disponibilidad,l10n_ar.l10nar_ri_chart_template,1.1.4.04.050,account.data_account_type_current_assets,IVA Saldo Libre Disponibilidad,False
l10n_ar.base_anticipo_ganancias,l10n_ar.l10nar_ex_chart_template,1.1.4.05.010,account.data_account_type_current_assets,Anticipo ganancias,False
l10n_ar.base_percepcion_ganancias_sufrida,l10n_ar.l10nar_ex_chart_template,1.1.4.05.020,account.data_account_type_current_assets,Percepciones de Ganancias Sufridas,False
l10n_ar.base_retencion_ganancias_sufrida,l10n_ar.l10nar_ex_chart_template,1.1.4.05.030,account.data_account_type_current_assets,Retenciones de Ganancias Sufridas,False
l10n_ar.base_saldo_favor_ganancias,l10n_ar.l10nar_ex_chart_template,1.1.4.05.040,account.data_account_type_current_assets,Saldo a favor Ganancias,False
l10n_ar.base_imp_debitos_computables,l10n_ar.l10nar_base_chart_template,1.1.4.05.050,account.data_account_type_current_assets,Impuesto a los Débitos Computables,False
l10n_ar.base_otros_creditos,l10n_ar.l10nar_base_chart_template,1.1.5.01.010,account.data_account_type_receivable,Deudores Varios,True
l10n_ar.base_materia_prima,l10n_ar.l10nar_base_chart_template,1.1.6.01.010,account.data_account_type_current_assets,Materia Prima,False
l10n_ar.base_productos_en_proceso,l10n_ar.l10nar_base_chart_template,1.1.6.01.020,account.data_account_type_current_assets,Productos en proceso,False
l10n_ar.base_productos_terminados,l10n_ar.l10nar_base_chart_template,1.1.6.01.030,account.data_account_type_current_assets,Productos terminados,False
l10n_ar.base_mercaderia_reventa,l10n_ar.l10nar_base_chart_template,1.1.6.01.040,account.data_account_type_current_assets,Mercaderia de reventa,False
l10n_ar.base_anticipo_proveedores,l10n_ar.l10nar_base_chart_template,1.1.6.01.050,account.data_account_type_current_assets,Anticipo a Proveedores,False
l10n_ar.base_instalaciones,l10n_ar.l10nar_base_chart_template,1.2.1.01.010,account.data_account_type_fixed_assets,Instalaciones,False
l10n_ar.base_amortizacion_acumulada_instalaciones,l10n_ar.l10nar_base_chart_template,1.2.1.01.020,account.data_account_type_fixed_assets,Amortización acumulada instalaciones,False
l10n_ar.base_maq_y_equipos,l10n_ar.l10nar_base_chart_template,1.2.1.02.010,account.data_account_type_fixed_assets,Maquinarias y equipos,False
l10n_ar.base_amortizacion_acumulada_maq_y_equipos,l10n_ar.l10nar_base_chart_template,1.2.1.02.020,account.data_account_type_fixed_assets,Amortización acumulada maquinarias y equipos,False
l10n_ar.base_muebles_y_utiles,l10n_ar.l10nar_base_chart_template,1.2.1.03.010,account.data_account_type_fixed_assets,Muebles y útiles,False
l10n_ar.base_amortizacion_acumulada_muebles_utiles,l10n_ar.l10nar_base_chart_template,1.2.1.03.020,account.data_account_type_fixed_assets,Amortización acumulada muebles y útiles,False
l10n_ar.base_rodados,l10n_ar.l10nar_base_chart_template,1.2.1.04.010,account.data_account_type_fixed_assets,Rodados,False
l10n_ar.base_amortizacion_acumulada_rodados,l10n_ar.l10nar_base_chart_template,1.2.1.04.020,account.data_account_type_fixed_assets,Amortización acumulada rodados,False
l10n_ar.base_derechos_de_marca,l10n_ar.l10nar_base_chart_template,1.2.2.01.010,account.data_account_type_fixed_assets,Derechos de marca,False
l10n_ar.base_amortizacion_acumulada_derechos_de_marca,l10n_ar.l10nar_base_chart_template,1.2.2.01.020,account.data_account_type_fixed_assets,Amortización acumulada Derechos de marca,False
l10n_ar.base_proveedores,l10n_ar.l10nar_base_chart_template,2.1.1.01.010,account.data_account_type_payable,Proveedores,True
l10n_ar.base_cheques_diferidos,l10n_ar.l10nar_base_chart_template,2.1.1.01.020,account.data_account_type_current_liabilities,Cheques diferidos a pagar,True
l10n_ar.base_cheques_rechazados,l10n_ar.l10nar_base_chart_template,2.1.1.01.030,account.data_account_type_current_liabilities,Cheques Rechazados,False
l10n_ar.base_anticipos_de_clientes,l10n_ar.l10nar_base_chart_template,2.1.1.01.040,account.data_account_type_current_liabilities,Anticipos de clientes,False
l10n_ar.base_alquileres_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.1.01.050,account.data_account_type_payable,Alquileres a pagar,True
l10n_ar.base_intereses_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.2.01.010,account.data_account_type_current_liabilities,Intereses a pagar,False
l10n_ar.base_giro_en_descubierto,l10n_ar.l10nar_base_chart_template,2.1.2.01.020,account.data_account_type_current_liabilities,Giro en descubierto Banco xxxxx,False
l10n_ar.base_otras_deudas_documentadas,l10n_ar.l10nar_base_chart_template,2.1.2.01.030,account.data_account_type_current_liabilities,Otras deudas documentadas,False
l10n_ar.base_prestamo_banco_x,l10n_ar.l10nar_base_chart_template,2.1.2.01.040,account.data_account_type_non_current_liabilities,Prestamo Banco xxxxx,False
l10n_ar.base_acreedores_varios,l10n_ar.l10nar_base_chart_template,2.1.2.01.050,account.data_account_type_payable,Acreedores varios,True
l10n_ar.base_tasa_municipal_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.3.01.010,account.data_account_type_payable,Tasa Municipal a pagar,True
l10n_ar.base_plan_tasa_municipal_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.3.01.020,account.data_account_type_payable,Plan Tasa Municipal a pagar,True
l10n_ar.base_iibb_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.3.02.010,account.data_account_type_payable,IIBB a pagar,True
l10n_ar.ri_retencion_sicore_a_pagar,l10n_ar.l10nar_ex_chart_template,2.1.3.02.020,account.data_account_type_payable,SICORE a pagar,True
l10n_ar.ri_retencion_iibb_caba_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.030,account.data_account_type_current_liabilities,Retención IIBB CABA aplicada,False
l10n_ar.ri_percepcion_iibb_caba_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.040,account.data_account_type_current_liabilities,Percepción IIBB CABA aplicada,False
l10n_ar.ri_retencion_iibb_ba_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.050,account.data_account_type_current_liabilities,Retención IIBB ARBA aplicada,False
l10n_ar.ri_percepcion_iibb_ba_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.060,account.data_account_type_current_liabilities,Percepción IIBB ARBA aplicada,False
l10n_ar.ri_retencion_iibb_ca_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.070,account.data_account_type_current_liabilities,Retención IIBB Catamarca aplicada,False
l10n_ar.ri_percepcion_iibb_ca_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.080,account.data_account_type_current_liabilities,Percepción IIBB Catamarca aplicada,False
l10n_ar.ri_retencion_iibb_co_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.090,account.data_account_type_current_liabilities,Retención IIBB Córdoba aplicada,False
l10n_ar.ri_percepcion_iibb_co_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.100,account.data_account_type_current_liabilities,Percepción IIBB Córdoba aplicada,False
l10n_ar.ri_retencion_iibb_rr_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.110,account.data_account_type_current_liabilities,Retención IIBB Corrientes aplicada,False
l10n_ar.ri_percepcion_iibb_rr_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.120,account.data_account_type_current_liabilities,Percepción IIBB Corrientes aplicada,False
l10n_ar.ri_retencion_iibb_er_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.130,account.data_account_type_current_liabilities,Retención IIBB Entre Río aplicada,False
l10n_ar.ri_percepcion_iibb_er_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.140,account.data_account_type_current_liabilities,Percepción IIBB Entre Río aplicada,False
l10n_ar.ri_retencion_iibb_ju_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.150,account.data_account_type_current_liabilities,Retención IIBB Jujuy aplicada,False
l10n_ar.ri_percepcion_iibb_ju_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.160,account.data_account_type_current_liabilities,Percepción IIBB Jujuy aplicada,False
l10n_ar.ri_retencion_iibb_za_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.170,account.data_account_type_current_liabilities,Retención IIBB Mendoza aplicada,False
l10n_ar.ri_percepcion_iibb_za_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.180,account.data_account_type_current_liabilities,Percepción IIBB Mendoza aplicada,False
l10n_ar.ri_retencion_iibb_lr_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.190,account.data_account_type_current_liabilities,Retención IIBB La Rioja aplicada,False
l10n_ar.ri_percepcion_iibb_lr_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.200,account.data_account_type_current_liabilities,Percepción IIBB La Rioja aplicada,False
l10n_ar.ri_retencion_iibb_sa_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.210,account.data_account_type_current_liabilities,Retención IIBB Salta aplicada,False
l10n_ar.ri_percepcion_iibb_sa_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.220,account.data_account_type_current_liabilities,Percepción IIBB Salta aplicada,False
l10n_ar.ri_retencion_iibb_nn_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.230,account.data_account_type_current_liabilities,Retención IIBB San Juan aplicada,False
l10n_ar.ri_percepcion_iibb_nn_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.240,account.data_account_type_current_liabilities,Percepción IIBB San Juan aplicada,False
l10n_ar.ri_retencion_iibb_sl_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.250,account.data_account_type_current_liabilities,Retención IIBB San Luis aplicada,False
l10n_ar.ri_percepcion_iibb_sl_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.260,account.data_account_type_current_liabilities,Percepción IIBB San Luis aplicada,False
l10n_ar.ri_retencion_iibb_sf_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.270,account.data_account_type_current_liabilities,Retención IIBB Santa Fe aplicada,False
l10n_ar.ri_percepcion_iibb_sf_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.280,account.data_account_type_current_liabilities,Percepción IIBB Santa Fe aplicada,False
l10n_ar.ri_retencion_iibb_se_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.290,account.data_account_type_current_liabilities,Retención IIBB Santiago del Estero aplicada,False
l10n_ar.ri_percepcion_iibb_se_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.300,account.data_account_type_current_liabilities,Percepción IIBB Santiago del Estero aplicada,False
l10n_ar.ri_retencion_iibb_tn_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.310,account.data_account_type_current_liabilities,Retención IIBB Tucumán aplicada,False
l10n_ar.ri_percepcion_iibb_tn_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.320,account.data_account_type_current_liabilities,Percepción IIBB Tucumán aplicada,False
l10n_ar.ri_retencion_iibb_ha_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.330,account.data_account_type_current_liabilities,Retención IIBB Chaco aplicada,False
l10n_ar.ri_percepcion_iibb_ha_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.340,account.data_account_type_current_liabilities,Percepción IIBB Chaco aplicada,False
l10n_ar.ri_retencion_iibb_ct_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.350,account.data_account_type_current_liabilities,Retención IIBB Chubut aplicada,False
l10n_ar.ri_percepcion_iibb_ct_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.360,account.data_account_type_current_liabilities,Percepción IIBB Chubut aplicada,False
l10n_ar.ri_retencion_iibb_fo_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.370,account.data_account_type_current_liabilities,Retención IIBB Formosa aplicada,False
l10n_ar.ri_percepcion_iibb_fo_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.380,account.data_account_type_current_liabilities,Percepción IIBB Formosa aplicada,False
l10n_ar.ri_retencion_iibb_mi_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.390,account.data_account_type_current_liabilities,Retención IIBB Misiones aplicada,False
l10n_ar.ri_percepcion_iibb_mi_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.400,account.data_account_type_current_liabilities,Percepción IIBB Misiones aplicada,False
l10n_ar.ri_retencion_iibb_ne_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.410,account.data_account_type_current_liabilities,Retención IIBB Neuquén aplicada,False
l10n_ar.ri_percepcion_iibb_ne_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.420,account.data_account_type_current_liabilities,Percepción IIBB Neuquén aplicada,False
l10n_ar.ri_retencion_iibb_lp_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.430,account.data_account_type_current_liabilities,Retención IIBB La Pampa aplicada,False
l10n_ar.ri_percepcion_iibb_lp_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.440,account.data_account_type_current_liabilities,Percepción IIBB La Pampa aplicada,False
l10n_ar.ri_retencion_iibb_rn_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.450,account.data_account_type_current_liabilities,Retención IIBB Río Negro aplicada,False
l10n_ar.ri_percepcion_iibb_rn_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.460,account.data_account_type_current_liabilities,Percepción IIBB Río Negro aplicada,False
l10n_ar.ri_retencion_iibb_az_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.470,account.data_account_type_current_liabilities,Retención IIBB Santa Cruz aplicada,False
l10n_ar.ri_percepcion_iibb_az_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.480,account.data_account_type_current_liabilities,Percepción IIBB Santa Cruz aplicada,False
l10n_ar.ri_retencion_iibb_tf_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.490,account.data_account_type_current_liabilities,Retención IIBB Tierra del Fuego aplicada,False
l10n_ar.ri_percepcion_iibb_tf_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.02.500,account.data_account_type_current_liabilities,Percepción IIBB Tierra del Fuego aplicada,False
l10n_ar.ri_retencion_iibb_a_pagar,l10n_ar.l10nar_ex_chart_template,2.1.3.02.510,account.data_account_type_payable,Retención/Percepción IIBB a pagar,True
l10n_ar.base_plan_de_iibb_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.3.02.520,account.data_account_type_payable,Plan de IIBB a pagar,True
l10n_ar.ri_iva_debito_fiscal,l10n_ar.l10nar_ri_chart_template,2.1.3.03.010,account.data_account_type_current_liabilities,IVA débito fiscal,False
l10n_ar.ri_iva_saldo_a_pagar,l10n_ar.l10nar_ri_chart_template,2.1.3.03.020,account.data_account_type_payable,IVA saldo a pagar,True
l10n_ar.ri_retencion_iva_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.03.030,account.data_account_type_current_liabilities,Retención IVA aplicada,False
l10n_ar.ri_percepcion_iva_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.03.040,account.data_account_type_current_liabilities,Percepción IVA aplicada,False
l10n_ar.base_plan_iva_a_pagar,l10n_ar.l10nar_ri_chart_template,2.1.3.03.050,account.data_account_type_current_liabilities,Plan IVA a pagar,False
l10n_ar.base_impuesto_ganancias_a_pagar,l10n_ar.l10nar_ex_chart_template,2.1.3.04.010,account.data_account_type_payable,Impuesto a las ganancias a pagar,True
l10n_ar.ri_retencion_ganancias_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.04.020,account.data_account_type_current_liabilities,Retención ganancias aplicada,False
l10n_ar.ri_percepcion_ganancias_aplicada,l10n_ar.l10nar_ex_chart_template,2.1.3.04.030,account.data_account_type_current_liabilities,Percepción ganancias aplicada,False
l10n_ar.base_provision_imp_a_las_ganancias,l10n_ar.l10nar_ex_chart_template,2.1.3.04.040,account.data_account_type_current_liabilities,Provisión Imp a las Ganancias,True
l10n_ar.base_anticipos_imp_a_las_ganancias_a_pagar,l10n_ar.l10nar_ex_chart_template,2.1.3.04.050,account.data_account_type_current_liabilities,Anticipos Imp a las Ganancias a Pagar,True
l10n_ar.base_planes_a_pagar_afip,l10n_ar.l10nar_ex_chart_template,2.1.3.04.060,account.data_account_type_non_current_liabilities,Plan Ganancias a pagar,False
l10n_ar.base_sueldos_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.4.01.010,account.data_account_type_payable,Sueldos a pagar,True
l10n_ar.base_suss_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.4.01.020,account.data_account_type_payable,Leyes Sociales a pagar,True
l10n_ar.base_seguro_de_vida_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.4.01.030,account.data_account_type_payable,Seguro de Vida a Pagar,True
l10n_ar.base_art_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.4.01.040,account.data_account_type_payable,ART a Pagar,True
l10n_ar.base_aportes_sindicales_a_pagar,l10n_ar.l10nar_base_chart_template,2.1.4.01.050,account.data_account_type_payable,Sindicato a pagar,True
l10n_ar.base_embargos_a_depositar,l10n_ar.l10nar_base_chart_template,2.1.4.01.060,account.data_account_type_payable,Embargos a depositar,True
l10n_ar.base_cta_particular_socio_x,l10n_ar.l10nar_base_chart_template,2.1.5.01.010,account.data_account_type_non_current_liabilities,Cta particular socio x,False
l10n_ar.base_proveedores_largo_plazo,l10n_ar.l10nar_base_chart_template,2.2.1.01.010,account.data_account_type_non_current_liabilities,Deudas Proveedores a largo plazo,False
l10n_ar.base_prevision_sac_a_pagar,l10n_ar.l10nar_base_chart_template,2.2.2.01.010,account.data_account_type_current_liabilities,Previsión SAC a Pagar,False
l10n_ar.base_prevision_para_despidos,l10n_ar.l10nar_base_chart_template,2.2.2.01.020,account.data_account_type_current_liabilities,Previsión para Despidos,False
l10n_ar.base_prevision_gastos,l10n_ar.l10nar_base_chart_template,2.2.2.01.030,account.data_account_type_current_liabilities,Prevision gastos,False
l10n_ar.base_capital,l10n_ar.l10nar_base_chart_template,3.1.1.01.010,account.data_account_type_equity,Capital suscripto,False
l10n_ar.base_ajustes_de_capital,l10n_ar.l10nar_base_chart_template,3.1.1.01.020,account.data_account_type_equity,Ajustes de capital,False
l10n_ar.base_aportes_no_capitalizados,l10n_ar.l10nar_base_chart_template,3.1.1.01.030,account.data_account_type_equity,Aportes no capitalizados,False
l10n_ar.base_reserva_legal,l10n_ar.l10nar_base_chart_template,3.2.1.01.010,account.data_account_type_equity,Reserva legal,False
l10n_ar.base_reserva_estatuitaria,l10n_ar.l10nar_base_chart_template,3.2.1.01.020,account.data_account_type_equity,Reserva estatuitaria,False
l10n_ar.base_futuras_inversiones,l10n_ar.l10nar_base_chart_template,3.2.1.01.030,account.data_account_type_equity,Reservas futuras inversiones,False
l10n_ar.base_reservas_facultativas,l10n_ar.l10nar_base_chart_template,3.2.1.01.040,account.data_account_type_equity,Reservas facultativas,False
l10n_ar.base_resultado_del_ejercicio,l10n_ar.l10nar_base_chart_template,3.3.1.01.010,account.data_unaffected_earnings,Resultado del ejercicio,False
l10n_ar.base_resultado_acumulados,l10n_ar.l10nar_base_chart_template,3.3.1.01.020,account.data_account_type_equity,Resultados de ejercicios anteriores,False
l10n_ar.base_ajuste_resultados,l10n_ar.l10nar_base_chart_template,3.3.1.01.030,account.data_account_type_equity,Ajuste resultados ejercicios anteriores,False
l10n_ar.base_venta_de_mercaderia,l10n_ar.l10nar_base_chart_template,4.1.1.01.010,account.data_account_type_revenue,Venta de mercadería,False
l10n_ar.base_venta_de_servicios,l10n_ar.l10nar_base_chart_template,4.1.1.01.020,account.data_account_type_revenue,Venta de servicios,False
l10n_ar.base_resultado_intereses_ganados,l10n_ar.l10nar_base_chart_template,4.2.1.01.010,account.data_account_type_other_income,Intereses ganados,False
l10n_ar.base_diferencias_de_cambio,l10n_ar.l10nar_base_chart_template,4.2.1.01.020,account.data_account_type_other_income,Diferencias de cambio,False
l10n_ar.base_ajuste_por_redondeo,l10n_ar.l10nar_base_chart_template,4.2.1.01.030,account.data_account_type_other_income,Ajuste por redondeo,False
l10n_ar.base_resultado_venta_bienes_de_uso,l10n_ar.l10nar_base_chart_template,4.3.1.01.010,account.data_account_type_other_income,Resultado venta bienes de uso,False
l10n_ar.base_recupero_de_gastos,l10n_ar.l10nar_base_chart_template,4.3.1.01.020,account.data_account_type_other_income,Recupero de gastos,False
l10n_ar.base_aportes_no_reembolsables,l10n_ar.l10nar_base_chart_template,4.3.1.01.030,account.data_account_type_other_income,Aportes no reeombolsables (subsidios),False
l10n_ar.base_cmv,l10n_ar.l10nar_base_chart_template,5.1.1.01.010,account.data_account_type_expenses,Costo de Mercadería Vendida,False
l10n_ar.base_descuentos_obtenidos,l10n_ar.l10nar_base_chart_template,5.1.1.01.020,account.data_account_type_expenses,Descuentos Obtenidos,False
l10n_ar.base_compra_mercaderia,l10n_ar.l10nar_base_chart_template,5.1.1.01.030,account.data_account_type_expenses,Compra de mercadería,False
l10n_ar.base_haberes_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.010,account.data_account_type_expenses,Sueldos y SAC Producción,False
l10n_ar.base_cargas_sociales_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.020,account.data_account_type_expenses,Cargas Sociales Producción,False
l10n_ar.base_gastos_varios_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.030,account.data_account_type_expenses,Gastos Varios Producción,False
l10n_ar.base_alquileres_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.040,account.data_account_type_expenses,Alquileres Producción,False
l10n_ar.base_servicio_de_luz_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.050,account.data_account_type_expenses,Servicio Eléctrico Producción,False
l10n_ar.base_servicio_de_agua_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.060,account.data_account_type_expenses,Servicio de Agua Producción,False
l10n_ar.base_servicio_de_gas_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.070,account.data_account_type_expenses,Servicio de Gas Producción,False
l10n_ar.base_impuesto_inmobiliario_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.080,account.data_account_type_expenses,Impuesto Inmobiliario Producción,False
l10n_ar.base_mantenimiento_y_reparaciones_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.090,account.data_account_type_expenses,Mantenimiento y Reparaciones Producción,False
l10n_ar.base_higiene_y_seguridad,l10n_ar.l10nar_base_chart_template,5.1.2.01.100,account.data_account_type_expenses,Higiene y Seguridad,False
l10n_ar.base_honorarios_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.110,account.data_account_type_expenses,Honorarios Producción,False
l10n_ar.base_mantenimiento_y_limpieza,l10n_ar.l10nar_base_chart_template,5.1.2.01.120,account.data_account_type_expenses,Mantenimiento y limpieza ,False
l10n_ar.base_seguros_produccion,l10n_ar.l10nar_base_chart_template,5.1.2.01.130,account.data_account_type_expenses,Seguros Producción,False
l10n_ar.base_haberes_comerciales,l10n_ar.l10nar_base_chart_template,5.2.1.01.010,account.data_account_type_expenses,Sueldos y SAC Comercial,False
l10n_ar.base_cargas_sociales_comerciales,l10n_ar.l10nar_base_chart_template,5.2.1.01.020,account.data_account_type_expenses,Cargas Sociales Comercial,False
l10n_ar.base_gastos_varios_comerciales,l10n_ar.l10nar_base_chart_template,5.2.1.01.030,account.data_account_type_expenses,Gastos Varios Comercial,False
l10n_ar.base_alquileres_comerciales,l10n_ar.l10nar_base_chart_template,5.2.1.01.040,account.data_account_type_expenses,Alquileres Comercial,False
l10n_ar.base_movilidad_y_viaticos,l10n_ar.l10nar_base_chart_template,5.2.1.01.050,account.data_account_type_expenses,Movilidad y viáticos ,False
l10n_ar.base_publicidad,l10n_ar.l10nar_base_chart_template,5.2.1.01.060,account.data_account_type_expenses,Publicidad,False
l10n_ar.base_comisiones,l10n_ar.l10nar_base_chart_template,5.2.1.01.070,account.data_account_type_expenses,Comisiones Pagadas,False
l10n_ar.base_servicios_de_luz_ecomercial,l10n_ar.l10nar_base_chart_template,5.2.1.01.080,account.data_account_type_expenses,Servicio Eléctrico Comercial,False
l10n_ar.base_servicio_de_agua_comercial,l10n_ar.l10nar_base_chart_template,5.2.1.01.090,account.data_account_type_expenses,Servicio de Agua Comercial,False
l10n_ar.base_servicio_de_gas_comercial,l10n_ar.l10nar_base_chart_template,5.2.1.01.100,account.data_account_type_expenses,Servicio de Gas Comercial,False
l10n_ar.base_honorarios_comercial,l10n_ar.l10nar_base_chart_template,5.2.1.01.110,account.data_account_type_expenses,Honorarios Comercial,False
l10n_ar.base_patentes_comercial,l10n_ar.l10nar_base_chart_template,5.2.1.01.120,account.data_account_type_expenses,Patentes Comercial,False
l10n_ar.base_seguros_comercial,l10n_ar.l10nar_base_chart_template,5.2.1.01.130,account.data_account_type_expenses,Seguros Comercial,False
l10n_ar.base_haberes_administrativos,l10n_ar.l10nar_base_chart_template,5.3.1.01.010,account.data_account_type_expenses,Sueldos y SAC Administrativos,False
l10n_ar.base_cargas_sociales_administrativos,l10n_ar.l10nar_base_chart_template,5.3.1.01.020,account.data_account_type_expenses,Cargas Sociales Administrativos,False
l10n_ar.base_gastos_varios_administrativos,l10n_ar.l10nar_base_chart_template,5.3.1.01.030,account.data_account_type_expenses,Gastos varios Administrativos,False
l10n_ar.base_alquileres_administrativos,l10n_ar.l10nar_base_chart_template,5.3.1.01.040,account.data_account_type_expenses,Alquileres Administrativos,False
l10n_ar.base_servicio_de_luz_administrativos,l10n_ar.l10nar_base_chart_template,5.3.1.01.050,account.data_account_type_expenses,Servicio Eléctrico Administrativos,False
l10n_ar.base_servicio_de_agua_administrativos,l10n_ar.l10nar_base_chart_template,5.3.1.01.060,account.data_account_type_expenses,Servicio de Agua Administrativos,False
l10n_ar.base_servicio_de_gas_administrativos,l10n_ar.l10nar_base_chart_template,5.3.1.01.070,account.data_account_type_expenses,Servicio de Gas Administrativos,False
l10n_ar.base_servicio_de_internet,l10n_ar.l10nar_base_chart_template,5.3.1.01.080,account.data_account_type_expenses,Servicio de Internet,False
l10n_ar.base_sistema_y_software,l10n_ar.l10nar_base_chart_template,5.3.1.01.090,account.data_account_type_expenses,Sistema y Software,False
l10n_ar.base_cadeteria_y_franqueo,l10n_ar.l10nar_base_chart_template,5.3.1.01.100,account.data_account_type_expenses,Cadeteria y franqueo,False
l10n_ar.base_honorarios_administracion,l10n_ar.l10nar_base_chart_template,5.3.1.01.110,account.data_account_type_expenses,Honorarios Administración,False
l10n_ar.base_articulos_de_libreria,l10n_ar.l10nar_base_chart_template,5.3.1.01.120,account.data_account_type_expenses,Artículos de librería,False
l10n_ar.base_seguros_administracion,l10n_ar.l10nar_base_chart_template,5.3.1.01.130,account.data_account_type_expenses,Seguros Administración,False
l10n_ar.base_sellados_y_certificaciones,l10n_ar.l10nar_base_chart_template,5.3.1.01.140,account.data_account_type_expenses,Sellados y Certificaciones,False
l10n_ar.base_tasa_municipal,l10n_ar.l10nar_base_chart_template,5.4.1.01.010,account.data_account_type_expenses,Tasa Municipal,False
l10n_ar.base_impuestos_iibb_caba,l10n_ar.l10nar_base_chart_template,5.4.2.01.010,account.data_account_type_expenses,IIBB CABA,False
l10n_ar.base_impuestos_iibb_ba,l10n_ar.l10nar_base_chart_template,5.4.2.01.020,account.data_account_type_expenses,IIBB ARBA,False
l10n_ar.base_impuestos_iibb_ca,l10n_ar.l10nar_base_chart_template,5.4.2.01.030,account.data_account_type_expenses,IIBB Catamarca,False
l10n_ar.base_impuestos_iibb_co,l10n_ar.l10nar_base_chart_template,5.4.2.01.040,account.data_account_type_expenses,IIBB Córdoba,False
l10n_ar.base_impuestos_iibb_rr,l10n_ar.l10nar_base_chart_template,5.4.2.01.050,account.data_account_type_expenses,IIBB Corrientes,False
l10n_ar.base_impuestos_iibb_er,l10n_ar.l10nar_base_chart_template,5.4.2.01.060,account.data_account_type_expenses,IIBB Entre Ríos,False
l10n_ar.base_impuestos_iibb_ju,l10n_ar.l10nar_base_chart_template,5.4.2.01.070,account.data_account_type_expenses,IIBB Jujuy,False
l10n_ar.base_impuestos_iibb_za,l10n_ar.l10nar_base_chart_template,5.4.2.01.080,account.data_account_type_expenses,IIBB Mendoza,False
l10n_ar.base_impuestos_iibb_lr,l10n_ar.l10nar_base_chart_template,5.4.2.01.090,account.data_account_type_expenses,IIBB La Rioja,False
l10n_ar.base_impuestos_iibb_sa,l10n_ar.l10nar_base_chart_template,5.4.2.01.100,account.data_account_type_expenses,IIBB Salta,False
l10n_ar.base_impuestos_iibb_nn,l10n_ar.l10nar_base_chart_template,5.4.2.01.110,account.data_account_type_expenses,IIBB San Juan,False
l10n_ar.base_impuestos_iibb_sl,l10n_ar.l10nar_base_chart_template,5.4.2.01.120,account.data_account_type_expenses,IIBB San Luis,False
l10n_ar.base_impuestos_iibb_sf,l10n_ar.l10nar_base_chart_template,5.4.2.01.130,account.data_account_type_expenses,IIBB Santa Fe,False
l10n_ar.base_impuestos_iibb_se,l10n_ar.l10nar_base_chart_template,5.4.2.01.140,account.data_account_type_expenses,IIBB Santiago del Estero,False
l10n_ar.base_impuestos_iibb_tn,l10n_ar.l10nar_base_chart_template,5.4.2.01.150,account.data_account_type_expenses,IIBB Tucumán,False
l10n_ar.base_impuestos_iibb_ha,l10n_ar.l10nar_base_chart_template,5.4.2.01.160,account.data_account_type_expenses,IIBB Chaco,False
l10n_ar.base_impuestos_iibb_ct,l10n_ar.l10nar_base_chart_template,5.4.2.01.170,account.data_account_type_expenses,IIBB Chubut,False
l10n_ar.base_impuestos_iibb_fo,l10n_ar.l10nar_base_chart_template,5.4.2.01.180,account.data_account_type_expenses,IIBB Formosa,False
l10n_ar.base_impuestos_iibb_mi,l10n_ar.l10nar_base_chart_template,5.4.2.01.190,account.data_account_type_expenses,IIBB Misiones,False
l10n_ar.base_impuestos_iibb_ne,l10n_ar.l10nar_base_chart_template,5.4.2.01.200,account.data_account_type_expenses,IIBB Neuquén,False
l10n_ar.base_impuestos_iibb_lp,l10n_ar.l10nar_base_chart_template,5.4.2.01.210,account.data_account_type_expenses,IIBB La Pampa,False
l10n_ar.base_impuestos_iibb_rn,l10n_ar.l10nar_base_chart_template,5.4.2.01.220,account.data_account_type_expenses,IIBB Río Negro,False
l10n_ar.base_impuestos_iibb_az,l10n_ar.l10nar_base_chart_template,5.4.2.01.230,account.data_account_type_expenses,IIBB Santa Cruz,False
l10n_ar.base_impuestos_iibb_tf,l10n_ar.l10nar_base_chart_template,5.4.2.01.240,account.data_account_type_expenses,IIBB Tierra del Fuego,False
l10n_ar.base_impuestos_debitos_y_creditos,l10n_ar.l10nar_base_chart_template,5.4.3.01.010,account.data_account_type_expenses,Impuestos a los débitos y créditos bancarios,False
l10n_ar.base_impuestos_a_las_ganancias,l10n_ar.l10nar_ex_chart_template,5.5.1.01.010,account.data_account_type_expenses,Impuestos a las ganancias,False
l10n_ar.base_resultado_intereses_y_recargos,l10n_ar.l10nar_base_chart_template,5.6.1.01.020,account.data_account_type_expenses,Intereses por préstamos,False
l10n_ar.base_intereses_por_descubierto,l10n_ar.l10nar_base_chart_template,5.6.1.01.030,account.data_account_type_expenses,Intereses por descubierto,False
l10n_ar.base_intereses_por_venta_de_valores,l10n_ar.l10nar_base_chart_template,5.6.1.01.040,account.data_account_type_expenses,Intereses por venta de valores,False
l10n_ar.base_intereses_fiscales,l10n_ar.l10nar_base_chart_template,5.6.1.01.050,account.data_account_type_expenses,Intereses fiscales,False
l10n_ar.base_gastos_bancarios,l10n_ar.l10nar_base_chart_template,5.6.1.01.060,account.data_account_type_expenses,Gastos Bancarios,False
l10n_ar.base_r_e_c_p_a_m,l10n_ar.l10nar_base_chart_template,5.6.1.01.070,account.data_account_type_expenses,R.E.C.P.A.M.,False
l10n_ar.base_amortizacion_instalaciones,l10n_ar.l10nar_base_chart_template,5.7.1.01.010,account.data_account_type_depreciation,Amortización instalaciones,False
l10n_ar.base_amortizacion_maq_y_equipos,l10n_ar.l10nar_base_chart_template,5.7.1.01.020,account.data_account_type_depreciation,Amortización maquinarias y equipos,False
l10n_ar.base_amortizacion_muebles_utiles,l10n_ar.l10nar_base_chart_template,5.7.1.01.030,account.data_account_type_depreciation,Amortización muebles y útiles,False
l10n_ar.base_amortizacion_rodados,l10n_ar.l10nar_base_chart_template,5.7.1.01.040,account.data_account_type_depreciation,Amortización rodados,False
l10n_ar.base_amortizacion_derechos_de_marca,l10n_ar.l10nar_base_chart_template,5.7.1.01.050,account.data_account_type_depreciation,Amortización Derechos de marca,False
l10n_ar.base_contrapartida_auxiliar,l10n_ar.l10nar_base_chart_template,6.0.0.00.010,account.data_account_type_current_assets,Contrapartida Auxiliar,False

```

## File: data\account.group.template.csv

```csv
id,code_prefix_start,code_prefix_end,name,chart_template_id/id
account_group_activo,1,,Activo,l10nar_base_chart_template
account_group_activo_corriente,1.1,,Activo Corriente,l10nar_base_chart_template
account_group_cajas_y_bancos,1.1.1,,Cajas y Bancos,l10nar_base_chart_template
account_group_caja,1.1.1.01,,Caja,l10nar_base_chart_template
account_group_bancos,1.1.1.02,,Bancos,l10nar_base_chart_template
account_group_inversiones,1.1.2,,Inversiones,l10nar_base_chart_template
account_group_creditos_por_ventas,1.1.3,,Créditos por ventas,l10nar_base_chart_template
account_group_creditos_fiscales,1.1.4,,Créditos fiscales,l10nar_base_chart_template
account_group_creditos_fiscales_municipales,1.1.4.01,,Créditos Fiscales Municipales,l10nar_base_chart_template
account_group_creditos_fiscales_iibb,1.1.4.02,,Créditos Fiscales IIBB,l10nar_base_chart_template
account_group_creditos_fiscales_suss,1.1.4.03,,Créditos Fiscales SUSS,l10nar_base_chart_template
account_group_creditos_fiscales_iva,1.1.4.04,,Créditos Fiscales IVA,l10nar_base_chart_template
account_group_creditos_fiscales_ganancias,1.1.4.05,,Créditos Fiscales Ganancias,l10nar_base_chart_template
account_group_otros_creditos,1.1.5,,Otros créditos,l10nar_base_chart_template
account_group_bienes_de_cambio,1.1.6,,Bienes de Cambio,l10nar_base_chart_template
account_group_activo_no_corriente,1.2,,Activo no corriente,l10nar_base_chart_template
account_group_activos_fijos,1.2.1,,Activos Fijos,l10nar_base_chart_template
account_group_instalaciones,1.2.1.01,,Instalaciones,l10nar_base_chart_template
account_group_maquinarias_y_equipos,1.2.1.02,,Maquinarias y Equipos,l10nar_base_chart_template
account_group_muebles_y_útiles,1.2.1.03,,Muebles y Útiles,l10nar_base_chart_template
account_group_rodados,1.2.1.04,,Rodados,l10nar_base_chart_template
account_group_activos_intangibles,1.2.2,,Activos Intangibles,l10nar_base_chart_template
account_group_derechos_de_marcas,1.2.2.01,,Derechos de Marcas,l10nar_base_chart_template
account_group_pasivo,2,,Pasivo,l10nar_base_chart_template
account_group_pasivo_corriente,2.1,,Pasivo Corriente,l10nar_base_chart_template
account_group_deudas_comerciales,2.1.1,,Deudas Comerciales,l10nar_base_chart_template
account_group_deudas_financieras,2.1.2,,Deudas Financieras,l10nar_base_chart_template
account_group_deudas_fiscales,2.1.3,,Deudas Fiscales,l10nar_base_chart_template
account_group_deudas_tasas_municipales,2.1.3.01,,Deudas Tasas Municipales,l10nar_base_chart_template
account_group_deudas_iibb,2.1.3.02,,Deudas IIBB,l10nar_base_chart_template
account_group_deudas_iva,2.1.3.03,,Deudas IVA,l10nar_base_chart_template
account_group_deudas_imp_ganancias,2.1.3.04,,Deudas Imp. Ganancias,l10nar_base_chart_template
account_group_remueraciones_y_cargas_sociales,2.1.4,,Remuneraciones y Cargas Sociales,l10nar_base_chart_template
account_group_cuentas_particulares,2.1.5,,Cuentas Particulares,l10nar_base_chart_template
account_group_pasivo_no_corriente,2.2,,Pasivo no Corriente,l10nar_base_chart_template
account_group_deudas_comerciales_a_largo_plazo,2.2.1,,Deudas Comerciales a Largo Plazo,l10nar_base_chart_template
account_group_previsiones,2.2.2,,Previsiones,l10nar_base_chart_template
account_group_patrimonio_neto,3,,Patrimonio Neto,l10nar_base_chart_template
account_group_capital_social,3.1,,Capital Social,l10nar_base_chart_template
account_group_reservas,3.2,,Reservas,l10nar_base_chart_template
account_group_resultados,3.3,,Resultados,l10nar_base_chart_template
account_group_ingresos,4,,Ingresos,l10nar_base_chart_template
account_group_ingresos_por_ventas,4.1,,Ingresos por ventas,l10nar_base_chart_template
account_group_ingresos_por_resultados_financieros,4.2,,Ingresos por resultados financieros,l10nar_base_chart_template
account_group_ingresos_extraordinarios,4.3,,Ingresos Extraordinarios,l10nar_base_chart_template
account_group_egresos,5,,Egresos,l10nar_base_chart_template
account_group_gastos_operativos,5.1,,Gastos Operativos,l10nar_base_chart_template
account_group_costo_de_mercadería_vendida,5.1.1,,Costo de Mercadería Vendida,l10nar_base_chart_template
account_group_gastos_de_producción,5.1.2,,Gastos de Producción,l10nar_base_chart_template
account_group_gastos_comerciales,5.2,,Gastos Comerciales,l10nar_base_chart_template
account_group_gastos_administrativos,5.3,,Gastos Administrativos,l10nar_base_chart_template
account_group_impuestos,5.4,,Impuestos,l10nar_base_chart_template
account_group_tasas_municipales,5.4.1,,Tasas Municipales,l10nar_base_chart_template
account_group_iibb,5.4.2,,IIBB,l10nar_base_chart_template
account_group_otros_impuestos,5.4.3,,Otros Impuestos,l10nar_base_chart_template
account_group_imp_a_las_ganancias,5.5,,Imp a las Ganancias,l10nar_base_chart_template
account_group_gastos_financieros,5.6,,Gastos Financieros,l10nar_base_chart_template
account_group_cuentas_puentes,6,,CUENTAS PUENTES,l10nar_base_chart_template

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_ar.l10nar_ri_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>

    <record model="account.chart.template" id="l10nar_base_chart_template">
        <field name="name">Plan Contable Genérico Argentina Monotributista / Base</field>
        <field name="currency_id" ref="base.ARS"/>
        <field name="bank_account_code_prefix">1.1.1.02.</field>
        <field name="cash_account_code_prefix">1.1.1.01.</field>
        <field name="code_digits">12</field>
        <field name="transfer_account_code_prefix">6.0.00.00.</field>
    </record>

    <record id="l10nar_ex_chart_template" model="account.chart.template">
        <field name="name">Plan Contable Genérico Argentina para Exentos</field>
        <field name="parent_id" ref="l10nar_base_chart_template"/>
        <field name="currency_id" ref="base.ARS"/>
        <field name="bank_account_code_prefix">1.1.1.02.</field>
        <field name="cash_account_code_prefix">1.1.1.01.</field>
        <field name="code_digits">12</field>
        <field name="transfer_account_code_prefix">6.0.00.00.</field>
    </record>

    <record id="l10nar_ri_chart_template" model="account.chart.template">
        <field name="name">Plan Contable Genérico Argentina para Responsables Inscriptos</field>
        <field name="parent_id" ref="l10nar_ex_chart_template"/>
        <field name="currency_id" ref="base.ARS"/>
        <field name="bank_account_code_prefix">1.1.1.02.</field>
        <field name="cash_account_code_prefix">1.1.1.01.</field>
        <field name="code_digits">12</field>
        <field name="transfer_account_code_prefix">6.0.00.00.</field>
    </record>

</odoo>

```

## File: data\account_chart_template_data2.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <menuitem id="account_reports_ar_statements_menu" name="Argentinian Statements" parent="account.menu_finance_reports" sequence="3" groups="account.group_account_readonly"/>

    <record id="l10nar_base_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="base_deudores_por_ventas"/>
        <field name="property_account_payable_id" ref="base_proveedores"/>
        <field name="property_account_expense_categ_id" ref="base_compra_mercaderia"/>
        <field name="property_account_income_categ_id" ref="base_venta_de_mercaderia"/>
        <field name="expense_currency_exchange_account_id" ref="base_diferencias_de_cambio"/>
        <field name="income_currency_exchange_account_id" ref="base_diferencias_de_cambio"/>
        <field name="default_pos_receivable_account_id" ref="base_deudores_por_ventas_pos"/>
    </record>

    <record id="l10nar_ex_chart_template" model="account.chart.template">
        <field name="default_pos_receivable_account_id" ref="base_deudores_por_ventas_pos"/>
        <field name="expense_currency_exchange_account_id" ref="base_diferencias_de_cambio"/>
        <field name="income_currency_exchange_account_id" ref="base_diferencias_de_cambio"/>
    </record>

    <record id="l10nar_ri_chart_template" model="account.chart.template">
        <field name="default_pos_receivable_account_id" ref="base_deudores_por_ventas_pos"/>
        <field name="expense_currency_exchange_account_id" ref="base_diferencias_de_cambio"/>
        <field name="income_currency_exchange_account_id" ref="base_diferencias_de_cambio"/>
    </record>

</odoo>

```

## File: data\account_fiscal_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <!-- Exempt Operations -->
    <record model='account.fiscal.position.template' id='fiscal_position_template_exempt_operations'>
        <field name='name'>Compras / Ventas al exterior</field>
        <field name="auto_apply" eval="True"/>
        <field name="l10n_ar_afip_responsibility_type_ids" eval="[(6, False, [ref('l10n_ar.res_EXT')])]"/>
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
    </record>
    <record id="fiscal_position_template_exempt_operations_tax_ventas_0" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exempt_operations"/>
        <field name="tax_src_id" ref="ri_tax_vat_0_ventas"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_ventas"/>
    </record>
    <record id="fiscal_position_template_exempt_operations_tax_ventas_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exempt_operations"/>
        <field name="tax_src_id" ref="ri_tax_vat_10_ventas"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_ventas"/>
    </record>
    <record id="fiscal_position_template_exempt_operations_tax_ventas_21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exempt_operations"/>
        <field name="tax_src_id" ref="ri_tax_vat_21_ventas"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_ventas"/>
    </record>
    <record id="fiscal_position_template_exempt_operations_tax_ventas_27" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exempt_operations"/>
        <field name="tax_src_id" ref="ri_tax_vat_27_ventas"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_ventas"/>
    </record>
    <record id="fiscal_position_template_exempt_operations_tax_compras_0" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exempt_operations"/>
        <field name="tax_src_id" ref="ri_tax_vat_0_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>
    <record id="fiscal_position_template_exempt_operations_tax_compras_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exempt_operations"/>
        <field name="tax_src_id" ref="ri_tax_vat_10_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>
    <record id="fiscal_position_template_exempt_operations_tax_compras_21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exempt_operations"/>
        <field name="tax_src_id" ref="ri_tax_vat_21_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>
    <record id="fiscal_position_template_exempt_operations_tax_compras_27" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exempt_operations"/>
        <field name="tax_src_id" ref="ri_tax_vat_27_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>
    <record id="fiscal_position_template_exempt_operations_exento" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exempt_operations"/>
        <field name="tax_src_id" ref="ri_tax_vat_exento_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>
    <record id="fiscal_position_template_exempt_operations_no_gravado" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exempt_operations"/>
        <field name="tax_src_id" ref="ri_tax_vat_no_gravado_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>

    <!-- Free Zone -->
    <record model='account.fiscal.position.template' id='fiscal_position_template_free_zone'>
        <field name='name'>Compras / Ventas Zona Franca</field>
        <field name="auto_apply" eval="True"/>
        <field name="l10n_ar_afip_responsibility_type_ids" eval="[(6, False, [ref('l10n_ar.res_IVA_LIB')])]"/>
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
    </record>
    <record id="fiscal_position_template_free_zone_tax_ventas_0" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_free_zone"/>
        <field name="tax_src_id" ref="ri_tax_vat_0_ventas"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_ventas"/>
    </record>
    <record id="fiscal_position_template_free_zone_tax_ventas_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_free_zone"/>
        <field name="tax_src_id" ref="ri_tax_vat_10_ventas"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_ventas"/>
    </record>
    <record id="fiscal_position_template_free_zone_tax_ventas_21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_free_zone"/>
        <field name="tax_src_id" ref="ri_tax_vat_21_ventas"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_ventas"/>
    </record>
    <record id="fiscal_position_template_free_zone_tax_ventas_27" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_free_zone"/>
        <field name="tax_src_id" ref="ri_tax_vat_27_ventas"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_ventas"/>
    </record>
    <record id="fiscal_position_template_free_zone_tax_compras_0" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_free_zone"/>
        <field name="tax_src_id" ref="ri_tax_vat_0_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_compras"/>
    </record>
    <record id="fiscal_position_template_free_zone_tax_compras_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_free_zone"/>
        <field name="tax_src_id" ref="ri_tax_vat_10_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_compras"/>
    </record>
    <record id="fiscal_position_template_free_zone_tax_compras_21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_free_zone"/>
        <field name="tax_src_id" ref="ri_tax_vat_21_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_compras"/>
    </record>
    <record id="fiscal_position_template_free_zone_tax_compras_27" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_free_zone"/>
        <field name="tax_src_id" ref="ri_tax_vat_27_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_exento_compras"/>
    </record>

    <!-- VAT dont correspond -->
    <record model='account.fiscal.position.template' id='fiscal_position_template_iva_no_corresponde'>
        <field name='name'>Compras IVA no corresponde</field>
        <field name="auto_apply" eval="True"/>
        <field name="l10n_ar_afip_responsibility_type_ids" eval="[(6, False, [ref('l10n_ar.res_IVAE'), ref('l10n_ar.res_RM')])]"/>
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
    </record>
    <record id="fiscal_position_template_iva_no_corresponde_tax_0" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_iva_no_corresponde"/>
        <field name="tax_src_id" ref="ri_tax_vat_0_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>
    <record id="fiscal_position_template_iva_no_corresponde_tax_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_iva_no_corresponde"/>
        <field name="tax_src_id" ref="ri_tax_vat_10_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>
    <record id="fiscal_position_template_iva_no_corresponde_tax_21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_iva_no_corresponde"/>
        <field name="tax_src_id" ref="ri_tax_vat_21_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>
    <record id="fiscal_position_template_iva_no_corresponde_tax_27" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_iva_no_corresponde"/>
        <field name="tax_src_id" ref="ri_tax_vat_27_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>
    <record id="fiscal_position_template_iva_no_corresponde_exento" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_iva_no_corresponde"/>
        <field name="tax_src_id" ref="ri_tax_vat_exento_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>
    <record id="fiscal_position_template_iva_no_corresponde_no_gravado" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_iva_no_corresponde"/>
        <field name="tax_src_id" ref="ri_tax_vat_no_gravado_compras"/>
        <field name="tax_dest_id" ref="ri_tax_vat_no_corresponde_compras"/>
    </record>

</odoo>

```

## File: data\account_tax_group.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- VAT Taxes -->

    <record id="tax_group_iva_21" model="account.tax.group">
        <field name="name">VAT 21%</field>
        <field name="l10n_ar_vat_afip_code">5</field>
    </record>

    <record id="tax_group_iva_27" model="account.tax.group">
        <field name="name">VAT 27%</field>
        <field name="l10n_ar_vat_afip_code">6</field>
    </record>

    <record id="tax_group_iva_105" model="account.tax.group">
        <field name="name">VAT 10.5%</field>
        <field name="l10n_ar_vat_afip_code">4</field>
    </record>

    <record id="tax_group_iva_025" model="account.tax.group">
        <field name="name">VAT 2,5%</field>
        <field name="l10n_ar_vat_afip_code">9</field>
    </record>

    <record model="account.tax.group" id="tax_group_iva_no_corresponde">
        <field name="name">VAT Not Applicable</field>
        <field name="l10n_ar_vat_afip_code">0</field>
    </record>

    <record model="account.tax.group" id="tax_group_iva_no_gravado">
        <field name="name">VAT Untaxed</field>
        <field name="l10n_ar_vat_afip_code">1</field>
    </record>

    <record model="account.tax.group" id="tax_group_iva_exento">
        <field name="name">VAT Exempt</field>
        <field name="l10n_ar_vat_afip_code">2</field>
    </record>

    <record model="account.tax.group" id="tax_group_iva_0">
        <field name="name">VAT 0%</field>
        <field name="l10n_ar_vat_afip_code">3</field>
    </record>

    <record model="account.tax.group" id="tax_group_iva_5">
        <field name="name">VAT 5%</field>
        <field name="l10n_ar_vat_afip_code">8</field>
    </record>

    <!-- Others Taxes -->

    <record model="account.tax.group" id="account.tax_group_taxes">
        <field name="name">Otros Impuestos</field>
        <field name="sequence">20</field>
        <field name="l10n_ar_tribute_afip_code">99</field>
    </record>

    <record model="account.tax.group" id="tax_impuestos_internos">
        <field name="name">Internal Taxes</field>
        <field name="sequence">15</field>
        <field name="l10n_ar_tribute_afip_code">04</field>
    </record>

    <!-- Perceptions and Withholding -->

    <record model="account.tax.group" id="tax_group_percepcion_iva">
        <field name="name">VAT Perception</field>
        <field name="l10n_ar_tribute_afip_code">06</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_caba">
        <field name="name">Perc IIBB CABA</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_ba">
        <field name="name">Perc IIBB ARBA</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_ca">
        <field name="name">Perc IIBB Catamarca</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_co">
        <field name="name">Perc IIBB Córdoba</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_rr">
        <field name="name">Perc IIBB Corrientes</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_er">
        <field name="name">Perc IIBB Entre Ríos</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_ju">
        <field name="name">Perc IIBB Jujuy</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_za">
        <field name="name">Perc IIBB Mendoza</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_lr">
        <field name="name">Perc IIBB La Rioja</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_sa">
        <field name="name">Perc IIBB Salta</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_nn">
        <field name="name">Perc IIBB San Juan</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_sl">
        <field name="name">Perc IIBB San Luis</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_sf">
        <field name="name">Perc IIBB Santa Fe</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_se">
        <field name="name">Perc IIBB Santiago del Estero</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_tn">
        <field name="name">Perc IIBB Tucumán</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_ha">
        <field name="name">Perc IIBB Chaco</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_ct">
        <field name="name">Perc IIBB Chubut</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_fo">
        <field name="name">Perc IIBB Formosa</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_mi">
        <field name="name">Perc IIBB Misiones</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_ne">
        <field name="name">Perc IIBB Neuquén</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_lp">
        <field name="name">Perc IIBB La Pampa</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_rn">
        <field name="name">Perc IIBB Río Negro</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_az">
        <field name="name">Perc IIBB Santa Cruz</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb_tf">
        <field name="name">Perc IIBB Tierra del Fuego</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_iibb">
        <field name="name">IIBB Perceptions</field>
        <field name="sequence">25</field>
        <field name="l10n_ar_tribute_afip_code">07</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_municipal">
        <field name="name">Municipal Taxes Perceptions</field>
        <field name="l10n_ar_tribute_afip_code">08</field>
    </record>

    <record model="account.tax.group" id="tax_group_percepcion_ganancias">
        <field name="name">Profit Perceptions</field>
        <field name="l10n_ar_tribute_afip_code">09</field>
    </record>

    <record model="account.tax.group" id="tax_group_otras_percepciones">
        <field name="name">Other Perceptions</field>
        <field name="l10n_ar_tribute_afip_code">09</field>
    </record>

</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="ri_tax_percepcion_iva_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IVA Aplicada</field>
        <field name="description">Perc IVA A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_percepcion_iva_aplicada'),
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
                'account_id': ref('ri_percepcion_iva_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iva"/>
    </record>

    <record id="ri_tax_percepcion_ganancias_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción Ganancias Aplicada</field>
        <field name="description">Perc Ganancias A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_percepcion_ganancias_aplicada'),
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
                'account_id': ref('ri_percepcion_ganancias_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_ganancias"/>
    </record>

    <record id="ri_tax_percepcion_ganancias_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción Ganancias Sufrida</field>
        <field name="description">Perc Ganancias S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_ganancias_sufrida'),
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
                'account_id': ref('base_percepcion_ganancias_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_ganancias"/>
    </record>

    <record id="ri_tax_percepcion_iibb_caba_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB CABA Sufrida</field>
        <field name="description">Perc IIBB CABA S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_caba_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_caba_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_caba"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ba_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB ARBA Sufrida</field>
        <field name="description">Perc IIBB ARBA S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_ba_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_ba_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ba"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ca_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Catamarca Sufrida</field>
        <field name="description">Perc IIBB Catamarca S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_ca_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_ca_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ca"/>
    </record>

    <record id="ri_tax_percepcion_iibb_co_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Córdoba Sufrida</field>
        <field name="description">Perc IIBB Córdoba S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_co_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_co_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_co"/>
    </record>

    <record id="ri_tax_percepcion_iibb_rr_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Corrientes Sufrida</field>
        <field name="description">Perc IIBB Corrientes S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_rr_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_rr_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_rr"/>
    </record>

    <record id="ri_tax_percepcion_iibb_er_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Entre Ríos Sufrida</field>
        <field name="description">Perc IIBB Entre Ríos S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_er_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_er_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_er"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ju_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Jujuy Sufrida</field>
        <field name="description">Perc IIBB Jujuy S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_ju_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_ju_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ju"/>
    </record>

    <record id="ri_tax_percepcion_iibb_za_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Mendoza Sufrida</field>
        <field name="description">Perc IIBB Mendoza S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_za_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_za_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_za"/>
    </record>

    <record id="ri_tax_percepcion_iibb_lr_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB La Rioja Sufrida</field>
        <field name="description">Perc IIBB La Rioja S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_lr_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_lr_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_lr"/>
    </record>

    <record id="ri_tax_percepcion_iibb_sa_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Salta Sufrida</field>
        <field name="description">Perc IIBB Salta S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_sa_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_sa_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_sa"/>
    </record>

    <record id="ri_tax_percepcion_iibb_nn_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB San Juan Sufrida</field>
        <field name="description">Perc IIBB San Juan S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_nn_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_nn_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_nn"/>
    </record>

    <record id="ri_tax_percepcion_iibb_sl_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB San Luis Sufrida</field>
        <field name="description">Perc IIBB San Luis S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_sl_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_sl_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_sl"/>
    </record>

    <record id="ri_tax_percepcion_iibb_sf_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Santa Fe Sufrida</field>
        <field name="description">Perc IIBB Santa Fe S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_sf_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_sf_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_sf"/>
    </record>

    <record id="ri_tax_percepcion_iibb_se_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Santiago del Estero Sufrida</field>
        <field name="description">Perc IIBB Santiago del Estero S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_se_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_se_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_se"/>
    </record>

    <record id="ri_tax_percepcion_iibb_tn_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Tucumán Sufrida</field>
        <field name="description">Perc IIBB Tucumán S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_tn_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_tn_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_tn"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ha_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Chaco Sufrida</field>
        <field name="description">Perc IIBB Chaco S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_ha_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_ha_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ha"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ct_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Chubut Sufrida</field>
        <field name="description">Perc IIBB Chubut S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_ct_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_ct_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ct"/>
    </record>

    <record id="ri_tax_percepcion_iibb_fo_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Formosa Sufrida</field>
        <field name="description">Perc IIBB Formosa S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_fo_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_fo_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_fo"/>
    </record>

    <record id="ri_tax_percepcion_iibb_mi_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Misiones Sufrida</field>
        <field name="description">Perc IIBB Misiones S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_mi_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_mi_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_mi"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ne_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Neuquén Sufrida</field>
        <field name="description">Perc IIBB Neuquén S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_ne_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_ne_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ne"/>
    </record>

    <record id="ri_tax_percepcion_iibb_lp_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB La Pampa Sufrida</field>
        <field name="description">Perc IIBB La Pampa S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_lp_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_lp_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_lp"/>
    </record>

    <record id="ri_tax_percepcion_iibb_rn_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Río Negro Sufrida</field>
        <field name="description">Perc IIBB Río Negro S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_rn_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_rn_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_rn"/>
    </record>

    <record id="ri_tax_percepcion_iibb_az_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Santa Cruz Sufrida</field>
        <field name="description">Perc IIBB Santa Cruz S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_az_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_az_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_az"/>
    </record>

    <record id="ri_tax_percepcion_iibb_tf_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_base_chart_template"/>
        <field name="name">Percepción IIBB Tierra del Fuego Sufrida</field>
        <field name="description">Perc IIBB Tierra del Fuego S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('base_percepcion_iibb_tf_sufrida'),
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
                'account_id': ref('base_percepcion_iibb_tf_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_tf"/>
    </record>

    <record id="ri_tax_percepcion_iibb_caba_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB CABA Aplicada</field>
        <field name="description">Perc IIBB CABA A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_caba_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_caba_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_caba"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ba_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB ARBA Aplicada</field>
        <field name="description">Perc IIBB ARBA A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ba_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ba_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ba"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ca_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Catamarca Aplicada</field>
        <field name="description">Perc IIBB Catamarca A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ca_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ca_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ca"/>
    </record>

    <record id="ri_tax_percepcion_iibb_co_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Córdoba Aplicada</field>
        <field name="description">Perc IIBB Córdoba A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_co_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_co_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_co"/>
    </record>

    <record id="ri_tax_percepcion_iibb_rr_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Corrientes Aplicada</field>
        <field name="description">Perc IIBB Corrientes A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_rr_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_rr_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_rr"/>
    </record>

    <record id="ri_tax_percepcion_iibb_er_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Entre Ríos Aplicada</field>
        <field name="description">Perc IIBB Entre Ríos A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_er_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_er_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_er"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ju_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Jujuy Aplicada</field>
        <field name="description">Perc IIBB Jujuy A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ju_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ju_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ju"/>
    </record>

    <record id="ri_tax_percepcion_iibb_za_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Mendoza Aplicada</field>
        <field name="description">Perc IIBB Mendoza A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_za_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_za_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_za"/>
    </record>

    <record id="ri_tax_percepcion_iibb_lr_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB La Rioja Aplicada</field>
        <field name="description">Perc IIBB La Rioja A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_lr_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_lr_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_lr"/>
    </record>

    <record id="ri_tax_percepcion_iibb_sa_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Salta Aplicada</field>
        <field name="description">Perc IIBB Salta A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_sa_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_sa_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_sa"/>
    </record>

    <record id="ri_tax_percepcion_iibb_nn_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB San Juan Aplicada</field>
        <field name="description">Perc IIBB San Juan A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_nn_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_nn_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_nn"/>
    </record>

    <record id="ri_tax_percepcion_iibb_sl_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB San Luis Aplicada</field>
        <field name="description">Perc IIBB San Luis A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_sl_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_sl_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_sl"/>
    </record>

    <record id="ri_tax_percepcion_iibb_sf_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Santa Fe Aplicada</field>
        <field name="description">Perc IIBB Santa Fe A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_sf_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_sf_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_sf"/>
    </record>

    <record id="ri_tax_percepcion_iibb_se_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Santiago del Estero Aplicada</field>
        <field name="description">Perc IIBB Santiago del Estero A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_se_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_se_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_se"/>
    </record>

    <record id="ri_tax_percepcion_iibb_tn_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Tucumán Aplicada</field>
        <field name="description">Perc IIBB Tucumán A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_tn_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_tn_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_tn"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ha_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Chaco Aplicada</field>
        <field name="description">Perc IIBB Chaco A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ha_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ha_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ha"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ct_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Chubut Aplicada</field>
        <field name="description">Perc IIBB Chubut A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ct_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ct_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ct"/>
    </record>

    <record id="ri_tax_percepcion_iibb_fo_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Formosa Aplicada</field>
        <field name="description">Perc IIBB Formosa A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_fo_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_fo_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_fo"/>
    </record>

    <record id="ri_tax_percepcion_iibb_mi_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Misiones Aplicada</field>
        <field name="description">Perc IIBB Misiones A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_mi_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_mi_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_mi"/>
    </record>

    <record id="ri_tax_percepcion_iibb_ne_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Neuquén Aplicada</field>
        <field name="description">Perc IIBB Neuquén A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ne_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_ne_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_ne"/>
    </record>

    <record id="ri_tax_percepcion_iibb_lp_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB La Pampa Aplicada</field>
        <field name="description">Perc IIBB La Pampa A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_lp_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_lp_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_lp"/>
    </record>

    <record id="ri_tax_percepcion_iibb_rn_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Río Negro Aplicada</field>
        <field name="description">Perc IIBB Río Negro A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_rn_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_rn_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_rn"/>
    </record>

    <record id="ri_tax_percepcion_iibb_az_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Santa Cruz Aplicada</field>
        <field name="description">Perc IIBB Santa Cruz A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_az_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_az_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_az"/>
    </record>

    <record id="ri_tax_percepcion_iibb_tf_aplicada" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ex_chart_template"/>
        <field name="name">Percepción IIBB Tierra del Fuego Aplicada</field>
        <field name="description">Perc IIBB Tierra del Fuego A</field>
        <field name="sequence">4</field>
        <field name="active" eval="False"/>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ar.ri_percepcion_iibb_tf_aplicada'),
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
                'account_id': ref('l10n_ar.ri_percepcion_iibb_tf_aplicada'),
            }),
        ]"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iibb_tf"/>
    </record>

    <!-- Only for Responsable Inscription account.chart.template -->

    <!-- Remove this record in V14 -->
    <record id="ri_tax_vat_no_corresponde_ventas" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="description">IVA No Corresponde</field>
        <field name="name">IVA No Corresponde</field>
        <field name="active" eval="False"/>
        <field name="sequence">2</field>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_debito_fiscal'),
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
                'account_id': ref('ri_iva_credito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_no_corresponde"/>
        <field name="type_tax_use">sale</field>
    </record>

    <record id="ri_tax_vat_no_corresponde_compras" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="description">IVA No Corresponde</field>
        <field name="name">IVA No Corresponde</field>
        <field name="sequence">2</field>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_credito_fiscal'),
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
                'account_id': ref('ri_iva_debito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_no_corresponde"/>
        <field name="type_tax_use">purchase</field>
    </record>

    <record id="ri_tax_vat_no_gravado_ventas" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="description">IVA No Gravado</field>
        <field name="name">IVA No Gravado</field>
        <field name="sequence">2</field>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_debito_fiscal'),
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
                'account_id': ref('ri_iva_credito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_no_gravado"/>
        <field name="type_tax_use">sale</field>
    </record>

    <record id="ri_tax_vat_no_gravado_compras" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="description">IVA No Gravado</field>
        <field name="name">IVA No Gravado</field>
        <field name="sequence">2</field>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_credito_fiscal'),
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
                'account_id': ref('ri_iva_debito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_no_gravado"/>
        <field name="type_tax_use">purchase</field>
    </record>

    <record id="ri_tax_vat_exento_ventas" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="description">IVA Exento</field>
        <field name="name">IVA Exento</field>
        <field name="sequence">2</field>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_debito_fiscal'),
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
                'account_id': ref('ri_iva_credito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_exento"/>
        <field name="type_tax_use">sale</field>
    </record>

    <record id="ri_tax_vat_exento_compras" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="description">IVA Exento</field>
        <field name="name">IVA Exento</field>
        <field name="sequence">2</field>
        <field name="amount_type">fixed</field>
        <field eval="0.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_credito_fiscal'),
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
                'account_id': ref('ri_iva_debito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_exento"/>
        <field name="type_tax_use">purchase</field>
    </record>

    <record id="ri_tax_vat_0_ventas" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="description">IVA 0%</field>
        <field name="name">IVA 0%</field>
        <field name="sequence">2</field>
        <field eval="0.0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_debito_fiscal'),
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
                'account_id': ref('ri_iva_credito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_0"/>
        <field name="type_tax_use">sale</field>
    </record>

    <record id="ri_tax_vat_0_compras" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="description">IVA 0%</field>
        <field name="name">IVA 0%</field>
        <field name="sequence">2</field>
        <field eval="0.0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_credito_fiscal'),
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
                'account_id': ref('ri_iva_debito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_0"/>
        <field name="type_tax_use">purchase</field>
    </record>

    <record id="ri_tax_vat_10_ventas" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="description">IVA 10.5%</field>
        <field name="name">IVA 10.5%</field>
        <field name="sequence">2</field>
        <field eval="10.5" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_debito_fiscal'),
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
                'account_id': ref('ri_iva_credito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_105"/>
        <field name="type_tax_use">sale</field>
    </record>

    <record id="ri_tax_vat_10_compras" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="description">IVA 10.5%</field>
        <field name="name">IVA 10.5%</field>
        <field name="sequence">2</field>
        <field eval="10.5" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_credito_fiscal'),
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
                'account_id': ref('ri_iva_debito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_105"/>
        <field name="type_tax_use">purchase</field>
    </record>

    <record id="ri_tax_vat_21_ventas" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="name">IVA 21%</field>
        <field name="description">IVA 21%</field>
        <field name="sequence">1</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_debito_fiscal'),
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
                'account_id': ref('ri_iva_credito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_21"/>
        <field name="type_tax_use">sale</field>
    </record>

    <record id="ri_tax_vat_21_compras" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="name">IVA 21%</field>
        <field name="description">IVA 21%</field>
        <field name="sequence">1</field>
        <field eval="21" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_credito_fiscal'),
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
                'account_id': ref('ri_iva_debito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_21"/>
        <field name="type_tax_use">purchase</field>
    </record>

    <record id="ri_tax_vat_27_ventas" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="name">IVA 27%</field>
        <field name="description">IVA 27%</field>
        <field name="sequence">3</field>
        <field eval="27" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_debito_fiscal'),
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
                'account_id': ref('ri_iva_credito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_27"/>
        <field name="type_tax_use">sale</field>
    </record>

    <record id="ri_tax_vat_27_compras" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="name">IVA 27%</field>
        <field name="description">IVA 27%</field>
        <field name="sequence">3</field>
        <field eval="27" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_credito_fiscal'),
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
                'account_id': ref('ri_iva_debito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_27"/>
        <field name="type_tax_use">purchase</field>
    </record>

    <record id="ri_tax_vat_25_ventas" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="name">IVA 2,5%</field>
        <field name="description">IVA 2,5%</field>
        <field name="sequence">9</field>
        <field name="active" eval="False"/>
        <field eval="2.5" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_debito_fiscal'),
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
                'account_id': ref('ri_iva_credito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_025"/>
        <field name="type_tax_use">sale</field>
    </record>

    <record id="ri_tax_vat_25_compras" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="name">IVA 2,5%</field>
        <field name="description">IVA 2,5%</field>
        <field name="sequence">9</field>
        <field name="active" eval="False"/>
        <field eval="2.5" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_credito_fiscal'),
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
                'account_id': ref('ri_iva_debito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_025"/>
        <field name="type_tax_use">purchase</field>
    </record>

    <record id="ri_tax_vat_5_ventas" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="name">IVA 5%</field>
        <field name="description">IVA 5%</field>
        <field name="sequence">10</field>
        <field name="active" eval="False"/>
        <field eval="5" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_debito_fiscal'),
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
                'account_id': ref('ri_iva_credito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_5"/>
        <field name="type_tax_use">sale</field>
    </record>

    <record id="ri_tax_vat_5_compras" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="name">IVA 5%</field>
        <field name="description">IVA 5%</field>
        <field name="sequence">10</field>
        <field name="active" eval="False"/>
        <field eval="5" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_iva_credito_fiscal'),
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
                'account_id': ref('ri_iva_debito_fiscal'),
            }),
        ]"/>
        <field name="tax_group_id" ref="l10n_ar.tax_group_iva_5"/>
        <field name="type_tax_use">purchase</field>
    </record>

    <record id="ri_tax_percepcion_iva_sufrida" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="name">Percepción IVA Sufrida</field>
        <field name="description">Perc IVA S</field>
        <field name="sequence">4</field>
        <field name="amount_type">fixed</field>
        <field eval="1.0" name="amount"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_percepcion_iva_sufrida'),
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
                'account_id': ref('ri_percepcion_iva_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iva"/>
    </record>

    <record id="ri_tax_ganancias_iva_adicional" model="account.tax.template">
        <field name="chart_template_id" ref="l10nar_ri_chart_template"/>
        <field name="name">IVA Adicional 20%</field>
        <field name="description">IVA Adicional 20%</field>
        <field name="sequence">4</field>
        <field eval="20.0" name="amount"/>
        <field name="amount_type">percent</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ri_percepcion_iva_sufrida'),
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
                'account_id': ref('ri_percepcion_iva_sufrida'),
            }),
        ]"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="l10n_ar.tax_group_percepcion_iva"/>
    </record>
</odoo>

```

## File: data\l10n_ar_afip_responsibility_type_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVARI'>
        <field name='code'>1</field>
        <field name='name'>IVA Responsable Inscripto</field>
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVARNI'>
        <field name='code'>2</field>
        <field name='name'>IVA Responsable no Inscripto</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVANR'>
        <field name='code'>3</field>
        <field name='name'>IVA no Responsable</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVAE'>
        <field name='code'>4</field>
        <field name='name'>IVA Sujeto Exento</field>
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_CF'>
        <field name='code'>5</field>
        <field name='name'>Consumidor Final</field>
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_RM'>
        <field name='code'>6</field>
        <field name='name'>Responsable Monotributo</field>
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_NOCATEG'>
        <field name='code'>7</field>
        <field name='name'>Sujeto no Categorizado</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_EXT'>
        <field name='code'>9</field>
        <field name='name'>Cliente / Proveedor del Exterior</field>
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVA_LIB'>
        <field name='code'>10</field>
        <field name='name'>IVA Liberado – Ley Nº 19.640</field>
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVARI_AP'>
        <field name='code'>11</field>
        <field name='name'>IVA Responsable Inscripto – Agente de Percepción</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_EVENTUAL'>
        <field name='code'>12</field>
        <field name='name'>Pequeño Contribuyente Eventual</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_MON_SOCIAL'>
        <field name='code'>13</field>
        <field name='name'>Monotributista Social</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_EVENTUAL_SOCIAL'>
        <field name='code'>14</field>
        <field name='name'>Pequeño Contribuyente Eventual Social</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVA_NO_ALC'>
        <field name='code'>99</field>
        <field name='name'>IVA No Alcanzado</field>
    </record>
</odoo>

```

## File: data\l10n_latam.document.type.csv

```csv
id,sequence,code,name,l10n_ar_letter,report_name,internal_type,doc_code_prefix,country_id/id,purchase_aliquots
dc_a_f,10,1,FACTURAS A,A,FACTURA,invoice,FA-A,base.ar,not_zero
dc_a_nd,20,2,NOTAS DE DEBITO A,A,NOTA DE DEBITO,debit_note,ND-A,base.ar,not_zero
dc_a_nc,30,3,NOTAS DE CREDITO A,A,NOTA DE CREDITO,credit_note,NC-A,base.ar,not_zero
dc_a_r,40,4,RECIBOS A,A,RECIBO,invoice,RE-A,base.ar,not_zero
dc_a_nvc,50,5,NOTAS DE VENTA AL CONTADO A,A,,invoice,NVC-A,base.ar,not_zero
dc_b_f,60,6,FACTURAS B,B,FACTURA,invoice,FA-B,base.ar,zero
dc_b_nd,70,7,NOTAS DE DEBITO B,B,NOTA DE DEBITO,debit_note,ND-B,base.ar,zero
dc_b_nc,80,8,NOTAS DE CREDITO B,B,NOTA DE CREDITO,credit_note,NC-B,base.ar,zero
dc_b_r,90,9,RECIBOS B,B,RECIBO,invoice,RE-B,base.ar,zero
dc_b_nvc,100,10,NOTAS DE VENTA AL CONTADO B,B,,invoice,NVC-B,base.ar,zero
dc_c_f,110,11,FACTURAS C,C,FACTURA,invoice,FA-C,base.ar,zero
dc_c_nd,120,12,NOTAS DE DEBITO C,C,NOTA DE DEBITO,debit_note,ND-C,base.ar,zero
dc_c_nc,130,13,NOTAS DE CREDITO C,C,NOTA DE CREDITO,credit_note,NC-C,base.ar,zero
dc_aduana,140,14,DOCUMENTO ADUANERO,,,,,base.ar,
dc_c_r,150,15,RECIBOS C,C,RECIBO,invoice,RE-C,base.ar,zero
dc_c_nvc,160,16,NOTAS DE VENTA AL CONTADO C,C,,invoice,NVC-C,base.ar,zero
dc_e_f,170,19,FACTURAS DE EXPORTACION,E,FACTURA,invoice,FA-E,base.ar,not_zero
dc_e_nd,180,20,NOTAS DE DEBITO POR OPERACIONES CON EL EXTERIOR,E,NOTA DE DEBITO,debit_note,ND-E,base.ar,not_zero
dc_e_nc,190,21,NOTAS DE CREDITO POR OPERACIONES CON EL EXTERIOR,E,NOTA DE CREDITO,credit_note,NC-E,base.ar,not_zero
dc_e_fs,200,22,FACTURAS - PERMISO EXPORTACION SIMPLIFICADO - DTO. 855/97,E,,,,base.ar,not_zero
dc_usados,210,30,COMPROBANTES DE COMPRA DE BIENES USADOS,,,invoice,CBU,base.ar,not_zero
dc_mandato,220,31,MANDATO - CONSIGNACION,,,,,base.ar,
dc_reciclado,230,32,COMPROBANTES PARA RECICLAR MATERIALES,,,invoice,CRM,base.ar,not_zero
dc_a_rg1415,240,34,"COMPROBANTES A DEL APARTADO A, INC. F), R.G. Nº 1415",A,,invoice,CA-A,base.ar,not_zero
dc_b_rg1415,250,35,"COMPROBANTES B DEL ANEXO I, APARTADO A, INC. F), R.G. Nº 1415",B,,invoice,CA-B,base.ar,zero
dc_c_rg1415,260,36,"COMPROBANTES C DEL ANEXO I, APARTADO A, INC. F), R.G. Nº 1415",C,,invoice,CA-C,base.ar,zero
dc_nd_rg1415,270,37,NOTAS DE DEBITO O DOCUMENTO EQUIVALENTE QUE CUMPLAN CON LA R.G. Nº 1415,,,debit_note,ND1415,base.ar,not_zero
dc_nc_rg1415,280,38,NOTAS DE CREDITO O DOCUMENTO EQUIVALENTE QUE CUMPLAN CON LA R.G. Nº 1415,,,credit_note,NC1415,base.ar,not_zero
dc_a_o_rg1415,290,39,OTROS COMPROBANTES A QUE CUMPLEN CON LA R.G. Nº 1415,A,,invoice,OC-A,base.ar,not_zero
dc_b_o_rg1415,300,40,OTROS COMPROBANTES B QUE CUMPLAN CON LA R.G. Nº 1415,B,,invoice,OC-B,base.ar,zero
dc_c_o_rg1415,310,41,OTROS COMPROBANTES C QUE CUMPLAN CON LA R.G. Nº 1415,C,,invoice,OC-C,base.ar,zero
dc_a_rf,320,50,RECIBO FACTURA A REGIMEN DE FACTURA DE CREDITO,A,,,,base.ar,not_zero
dc_m_f,330,51,FACTURAS M,M,FACTURA,invoice,FA-M,base.ar,not_zero
dc_m_nd,340,52,NOTAS DE DEBITO M,M,NOTA DE DEBITO,debit_note,ND-M,base.ar,not_zero
dc_m_nc,350,53,NOTAS DE CREDITO M,M,NOTA DE CREDITO,credit_note,NC-M,base.ar,not_zero
dc_m_r,360,54,RECIBOS M,M,RECIBO,invoice,RE-M,base.ar,not_zero
dc_m_nvc,370,55,NOTAS DE VENTA AL CONTADO M,M,,invoice,NVC-M,base.ar,not_zero
dc_m_rg1415,380,56,"COMPROBANTES M DEL ANEXO I, APARTADO A, INC. F), R.G. Nº 1415",M,,invoice,CA-M,base.ar,not_zero
dc_m_o_rg1415,390,57,OTROS COMPROBANTES M QUE CUMPLAN CON LA R.G. Nº 1415,M,,invoice,OC-M,base.ar,not_zero
dc_m_cvl,400,58,CUENTAS DE VENTA Y LIQUIDO PRODUCTO M,M,,invoice,LP-M,base.ar,not_zero
dc_m_l,410,59,LIQUIDACIONES M,M,LIQUIDACION,invoice,LI-M,base.ar,not_zero
dc_a_cvl,420,60,CUENTAS DE VENTA Y LIQUIDO PRODUCTO A,A,CTA VTA LIQUIDO PRODUCTO,invoice,LP-A,base.ar,not_zero
dc_b_cvl,430,61,CUENTAS DE VENTA Y LIQUIDO PRODUCTO B,B,CTA VTA LIQUIDO PRODUCTO,invoice,LP-B,base.ar,zero
dc_a_l,440,63,LIQUIDACIONES A,A,LIQUIDACION,invoice,LI-A,base.ar,not_zero
dc_b_l,450,64,LIQUIDACIONES B,B,LIQUIDACION,invoice,LI-B,base.ar,zero
dc_nc,460,65,"NOTAS DE CREDITO DE COMPROBANTES CON COD. 34, 39, 58, 59, 60, 63, 96, 97,",,,,,base.ar,
dc_desp_imp,470,66,DESPACHO DE IMPORTACION,,,invoice,DI,base.ar,not_zero
dc_imp_serv,480,67,IMPORTACION DE SERVICIOS,,,,,base.ar,
dc_c_l,490,68,LIQUIDACION C,C,LIQUIDACION,invoice,LI-C,base.ar,not_zero
dc_rfc,500,70,RECIBOS FACTURA DE CREDITO,,,,,base.ar,not_zero
dc_cfcp,510,71,CREDITO FISCAL POR CONTRIBUCIONES PATRONALES,,,,,base.ar,
dc_f1116,520,73,FORMULARIO 1116 RT,,,,,base.ar,
dc_cptag,530,74,CARTA DE PORTE PARA EL TRANSPORTE AUTOMOTOR PARA GRANOS,,,,,base.ar,
dc_cptfg,540,75,CARTA DE PORTE PARA EL TRANSPORTE FERROVIARIO PARA GRANOS,,,,,base.ar,
dc_zeta,550,80,INFORME DIARIO DE CIERRE (ZETA) - CONTROLADORES FISCALES,,,,,base.ar,
dc_a_t,560,81,TIQUE FACTURA A,A,TIQUE FACTURA,invoice,TF-A,base.ar,not_zero
dc_b_t,570,82,TIQUE - FACTURA B,B,TIQUE FACTURA,invoice,TF-B,base.ar,zero
dc_t,580,83,TIQUE,,TIQUE,invoice,TI-X,base.ar,zero
dc_sp_c,590,84,COMPROBANTE FACTURA DE SERVICIOS PUBLICOS INTERESES FINANCIEROS,,,,,base.ar,
dc_sp_nc,600,85,NOTA DE CREDITO SERVICIOS PUBLICOS NOTA DE CREDITO CONTROLADORES FISCALES,,,,,base.ar,
dc_sp_nd,610,86,NOTA DE DEBITO SERVICIOS PUBLICOS,,,,,base.ar,
dc_oc_se,620,87,OTROS COMPROBANTES - SERVICIOS DEL EXTERIOR,,,,,base.ar,
dc_oc_c,630,88,REMITO ELECTRONICO,,,,,base.ar,
dc_oc_nd,640,89,RESUMEN DE DATOS,,,,,base.ar,
dc_oc_nc,650,90,OTROS COMPROBANTES - DOCUMENTOS EXCEPTUADOS - NOTAS DE CREDITO,,,credit_note,OC,base.ar,not_zero
dc_r_r,660,91,REMITOS R,R,,,,base.ar,
dc_ac_inc_df,670,92,AJUSTES CONTABLES QUE INCREMENTAN EL DEBITO FISCAL,,,,,base.ar,
dc_ac_dis_df,680,93,AJUSTES CONTABLES QUE DISMINUYEN EL DEBITO FISCAL,,,,,base.ar,
dc_ac_inc_cf,690,94,AJUSTES CONTABLES QUE INCREMENTAN EL CREDITO FISCAL,,,,,base.ar,
dc_ac_dis_cf,700,95,AJUSTES CONTABLES QUE DISMINUYEN EL CREDITO FISCAL,,,,,base.ar,
dc_f1116b,710,96,FORMULARIO 1116 B,,,,,base.ar,
dc_f1116c,720,97,FORMULARIO 1116 C,,,,,base.ar,
dc_oc_ncrg3419,730,99,OTROS COMPROBANTES QUE NO CUMPLEN O ESTAN EXCEPTUADOS DE LA R.G. Nº 1415 Y SUS MODIF,,,invoice,OC-X,base.ar,not_zero
dc_aa_dj_pos,740,101,AJUSTE ANUAL PROVENIENTE DE LA D J DEL IVA POSITIVO,,,,,base.ar,
dc_aa_dj_neg,750,102,AJUSTE ANUAL PROVENIENTE DE LA D J DEL IVA NEGATIVO,,,,,base.ar,
dc_na,760,103,NOTA DE ASIGNACION,,,,,base.ar,
dc_nca,770,104,NOTA DE CREDITO DE ASIGNACION,,,,,base.ar,
dc_remito_x,790,94,REMITOS X,X,REMITO,,RM-X,base.ar,
dc_liq_s_a,800,17,LIQUIDACION DE SERVICIOS PUBLICOS CLASE A,A,,invoice,LS-A,base.ar,not_zero
dc_liq_s_b,810,18,LIQUIDACION DE SERVICIOS PUBLICOS CLASE B,B,,invoice,LS-B,base.ar,zero
dc_com_a_m,820,23,"COMPROBANTES ""A"" DE COMPRA PRIMARIA PARA EL SECTOR PESQUERO MARITIMO",,,,,base.ar,
dc_con_a_m,830,24,"COMPROBANTES ""A"" DE CONSIGNACION PRIMARIA PARA EL SECTOR PESQUERO MARITIMO",,,,,base.ar,
dc_com_b_m,840,25,"COMPROBANTES ""B"" DE COMPRA PRIMARIA PARA EL SECTOR PESQUERO MARITIMO",,,,,base.ar,
dc_con_b_m,850,26,"COMPROBANTES ""B"" DE CONSIGNACION PRIMARIA PARA EL SECTOR PESQUERO MARITIMO",,,,,base.ar,
dc_liq_uci_a,860,27,LIQUIDACION UNICA COMERCIAL IMPOSITIVA CLASE A,A,,invoice,LU-A,base.ar,not_zero
dc_liq_uci_b,870,28,LIQUIDACION UNICA COMERCIAL IMPOSITIVA CLASE B,B,,invoice,LU-B,base.ar,zero
dc_liq_uci_c,880,29,LIQUIDACION UNICA COMERCIAL IMPOSITIVA CLASE C,C,,invoice,LU-C,base.ar,zero
dc_liq_prim_gr,890,33,LIQUIDACION PRIMARIA DE GRANOS,,,invoice,LPG,base.ar,not_zero
dc_nc_liq_uci_a,900,43,NOTA DE CREDITO LIQUIDACION UNICA COMERCIAL IMPOSITIVA CLASE B,B,,credit_note,NCLU-B,base.ar,zero
dc_nc_liq_uci_b,910,44,NOTA DE CREDITO LIQUIDACION UNICA COMERCIAL IMPOSITIVA CLASE C,C,,credit_note,NCLU-C,base.ar,zero
dc_nd_liq_uci_a,920,45,NOTA DE DEBITO LIQUIDACION UNICA COMERCIAL IMPOSITIVA CLASE A,A,,debit_note,NDLU-A,base.ar,not_zero
dc_nd_liq_uci_b,930,46,NOTA DE DEBITO LIQUIDACION UNICA COMERCIAL IMPOSITIVA CLASE B,B,,debit_note,NDLU-B,base.ar,zero
dc_nd_liq_uci_c,940,47,NOTA DE DEBITO LIQUIDACION UNICA COMERCIAL IMPOSITIVA CLASE C,C,,debit_note,NDLU-C,base.ar,zero
dc_nc_liq_uci_c,950,48,NOTA DE CREDITO LIQUIDACION UNICA COMERCIAL IMPOSITIVA CLASE A,A,,credit_note,NCLU-A,base.ar,not_zero
dc_bs_no_reg,960,49,COMPROBANTES DE COMPRA DE BIENES NO REGISTRABLES A CONSUMIDORES FINALES,,,invoice,BNR,base.ar,not_zero
dc_t_nc,970,110,TIQUE NOTA DE CREDITO,,TIQUE NOTA DE CREDITO,credit_note,TC-X,base.ar,not_zero
dc_t_c,980,111,TIQUE FACTURA C,C,TIQUE FACTURA,invoice,TF-C,base.ar,zero
dc_t_nc_a,990,112,TIQUE NOTA DE CREDITO A,A,TIQUE NOTA DE CREDITO,credit_note,TN-A,base.ar,not_zero
dc_t_nc_b,1000,113,TIQUE NOTA DE CREDITO B,B,TIQUE NOTA DE CREDITO,credit_note,TN-B,base.ar,zero
dc_t_nc_c,1010,114,TIQUE NOTA DE CREDITO C,C,TIQUE NOTA DE CREDITO,credit_note,TN-C,base.ar,zero
dc_t_nd_a,1020,115,TIQUE NOTA DE DEBITO A,A,TIQUE NOTA DE DEBITO,debit_note,TD-A,base.ar,not_zero
dc_t_nd_b,1030,116,TIQUE NOTA DE DEBITO B,B,TIQUE NOTA DE DEBITO,debit_note,TD-B,base.ar,zero
dc_t_nd_c,1040,117,TIQUE NOTA DE DEBITO C,C,TIQUE NOTA DE DEBITO,debit_note,TD-C,base.ar,zero
dc_t_m,1050,118,TIQUE FACTURA M,M,FACTURA,invoice,TF-M,base.ar,not_zero
dc_t_nc_m,1060,119,TIQUE NOTA DE CREDITO M,M,NOTA DE CREDITO,credit_note,TC-M,base.ar,not_zero
dc_t_nd_m,1070,120,TIQUE NOTA DE DEBITO M,M,NOTA DE DEBITO,debit_note,TD-M,base.ar,not_zero
dc_liq_sec_gr,1080,331,LIQUIDACION SECUNDARIA DE GRANOS,,,invoice,LSG,base.ar,not_zero
dc_cert_ele_gr,1090,332,CERTIFICACION ELECTRONICA (GRANOS),,,,,base.ar,
dc_fce_a_f,1100,201,FACTURA DE CREDITO ELECTRONICA MiPyMEs (FCE) A,A,FACTURA DE CREDITO ELECTRONICA,invoice,FCE-A,base.ar,not_zero
dc_fce_a_nd,1110,202,NOTA DE DEBITO ELECTRONICA MiPyMEs (FCE) A,A,NOTA DE DEBITO ELECTRONICA,debit_note,NDE-A,base.ar,not_zero
dc_fce_a_nc,1120,203,NOTA DE CREDITO ELECTRONICA MiPyMEs (FCE) A,A,NOTA DE CREDITO ELECTRONICA,credit_note,NCE-A,base.ar,not_zero
dc_fce_b_f,1130,206,FACTURA DE CREDITO ELECTRONICA MiPyMEs (FCE) B,B,FACTURA DE CREDITO ELECTRONICA,invoice,FCE-B,base.ar,zero
dc_fce_b_nd,1140,207,NOTA DE DEBITO ELECTRONICA MiPyMEs (FCE) B,B,NOTA DE DEBITO ELECTRONICA,debit_note,NDE-B,base.ar,zero
dc_fce_b_nc,1150,208,NOTA DE CREDITO ELECTRONICA MiPyMEs (FCE) B,B,NOTA DE CREDITO ELECTRONICA,credit_note,NCE-B,base.ar,zero
dc_fce_c_f,1160,211,FACTURA DE CREDITO ELECTRONICA MiPyMEs (FCE) C,C,FACTURA DE CREDITO ELECTRONICA,invoice,FCE-C,base.ar,zero
dc_fce_c_nd,1170,212,NOTA DE DEBITO ELECTRONICA MiPyMEs (FCE) C,C,NOTA DE DEBITO ELECTRONICA,debit_note,NDE-C,base.ar,zero
dc_fce_c_nc,1180,213,NOTA DE CREDITO ELECTRONICA MiPyMEs (FCE) C,C,NOTA DE CREDITO ELECTRONICA,credit_note,NCE-D,base.ar,zero
fa_exterior,195,,FACTURAS Y COMPROBANTES DEL EXTERIOR,I,,invoice,FA-I,base.ar,zero
nc_exterior,196,,NOTAS DE CREDITO Y REEMBOLSOS DEL EXTERIOR,I,,credit_note,NC-I,base.ar,zero
dc_liq_cpst_a,1190,150,LIQUIDACIÓN DE COMPRA PRIMARIA PARA EL SECTOR TABACALERO A,A,,invoice,LCT-A,base.ar,not_zero
dc_liq_cpst_b,1200,151,LIQUIDACIÓN DE COMPRA PRIMARIA PARA EL SECTOR TABACALERO B,B,,invoice,LCT-B,base.ar,zero
dc_cvl_sa_a,1210,157,CUENTA DE VENTA Y LÍQUIDO PRODUCTO A – SECTOR AVÍCOLA,A,,invoice,CVA-A,base.ar,not_zero
dc_cvl_sa_b,1220,158,CUENTA DE VENTA Y LÍQUIDO PRODUCTO B – SECTOR AVÍCOLA,B,,invoice,CVA-B,base.ar,zero
dc_liq_c_sa_a,1230,159,LIQUIDACIÓN DE COMPRA A – SECTOR AVÍCOLA,A,,invoice,LCA-A,base.ar,not_zero
dc_liq_c_sa_b,1240,160,LIQUIDACIÓN DE COMPRA B – SECTOR AVÍCOLA,B,,invoice,LCA-B,base.ar,zero
dc_liq_cd_sa_a,1250,161,LIQUIDACIÓN DE COMPRA DIRECTA A - SECTOR AVÍCOLA,A,,invoice,LCDA-A,base.ar,not_zero
dc_liq_cd_sa_b,1260,162,LIQUIDACIÓN DE COMPRA DIRECTA B - SECTOR AVÍCOLA,B,,invoice,LCDA-B,base.ar,zero
dc_liq_cd_sa_c,1270,163,LIQUIDACIÓN DE COMPRA DIRECTA C - SECTOR AVÍCOLA,C,,invoice,LCDA-C,base.ar,zero
dc_liq_vd_sa_a,1280,164,LIQUIDACIÓN DE VENTA DIRECTA A - SECTOR AVÍCOLA,A,,invoice,LVDA-A,base.ar,not_zero
dc_liq_vd_sa_b,1290,165,LIQUIDACIÓN DE VENTA DIRECTA B - SECTOR AVÍCOLA,B,,invoice,LVDA-B,base.ar,zero
dc_liq_ccpc_a,1300,166,LIQUIDACIÓN DE CONTRATACIÓN DE CRIANZA POLLOS PARRILLEROS A,A,,invoice,LCCP-A,base.ar,not_zero
dc_liq_ccpc_b,1310,167,LIQUIDACIÓN DE CONTRATACIÓN DE CRIANZA POLLOS PARRILLEROS B,B,,invoice,LCCP-B,base.ar,zero
dc_liq_ccpc_c,1320,168,LIQUIDACIÓN DE CONTRATACIÓN DE CRIANZA POLLOS PARRILLEROS C,C,,invoice,LCCP-C,base.ar,zero
dc_liq_cpc_a,1330,169,LIQUIDACIÓN DE CRIANZA POLLOS PARRILLEROS A,A,,invoice,LCPP-A,base.ar,not_zero
dc_liq_cpc_b,1340,170,LIQUIDACIÓN DE CRIANZA POLLOS PARRILLEROS B,B,,invoice,LCPP-B,base.ar,zero
dc_liq_cca_a,1350,171,LIQUIDACIÓN DE COMPRA DE CAÑA DE AZÚCAR A,A,,invoice,LCCA-A,base.ar,not_zero
dc_liq_cca_b,1360,172,LIQUIDACIÓN DE COMPRA DE CAÑA DE AZÚCAR B,B,,invoice,LCCA-B,base.ar,zero
dc_cvl_sp_a,1370,180,CUENTA DE VENTA Y LÍQUIDO PRODUCTO A - SECTOR PECUARIO,A,,invoice,CVP-A,base.ar,not_zero
dc_cvl_sp_b,1380,182,CUENTA DE VENTA Y LÍQUIDO PRODUCTO B - SECTOR PECUARIO,B,,invoice,CVP-B,base.ar,zero
dc_liq_c_sp_a,1390,183,LIQUIDACIÓN DE COMPRA A - SECTOR PECUARIO,A,,invoice,LCP-A,base.ar,not_zero
dc_liq_c_sp_b,1400,185,LIQUIDACIÓN DE COMPRA B - SECTOR PECUARIO,B,,invoice,LCP-B,base.ar,zero
dc_liq_cd_sp_a,1410,186,LIQUIDACIÓN DE COMPRA DIRECTA A - SECTOR PECUARIO,A,,invoice,LCDP-A,base.ar,not_zero
dc_liq_cd_sp_b,1420,188,LIQUIDACIÓN DE COMPRA DIRECTA B - SECTOR PECUARIO,B,,invoice,LCDP-B,base.ar,zero
dc_liq_cd_sp_c,1430,189,LIQUIDACIÓN DE COMPRA DIRECTA C - SECTOR PECUARIO,C,,invoice,LCDP-C,base.ar,zero
dc_liq_vd_sp_a,1440,190,LIQUIDACIÓN DE VENTA DIRECTA A - SECTOR PECUARIO,A,,invoice,LVDP-A,base.ar,not_zero
dc_liq_vd_sp_b,1450,191,LIQUIDACIÓN DE VENTA DIRECTA B - SECTOR PECUARIO,B,,invoice,LVDP-B,base.ar,zero

```

## File: data\l10n_latam.document.type.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <function model="l10n_latam.document.type" name="write">
            <value model="l10n_latam.document.type" eval="obj().search([('country_id.code', '=', 'AR'), ('code', 'in', ['5','10','14','16','22','30','31','32','34','35','36','37','38','50','55','56','57','58','59','60','61','65','67','68','70','71','73','74','75','80','84','85','86','87','88','89','90','91','92','93','94','95','96','97','101','102','103','104','94','23','24','25','26','33','331','332','150','151','157','158','159','160','161','162','163','164','165','166','167','168','169','170','171','172','180','182','183','185','186','188','189','190','191'])]).ids"/>
            <value eval="{'active': False}"/>
        </function>
    </data>
</odoo>

```

## File: data\l10n_latam_identification_type_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
<data>
    <record model='l10n_latam.identification.type' id='l10n_latam_base.it_fid'>
        <field name='l10n_ar_afip_code'>91</field>
    </record>
    <record model='l10n_latam.identification.type' id='l10n_latam_base.it_pass'>
        <field name='l10n_ar_afip_code'>94</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_cuit'>
        <field name='name'>CUIT</field>
        <field name='description'>Código Único de Identificación Tributaria</field>
        <field name='country_id' ref='base.ar'/>
        <field name='is_vat' eval='True'/>
        <field name='l10n_ar_afip_code'>80</field>
        <field name='sequence'>10</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_dni'>
        <field name='name'>DNI</field>
        <field name='description'>Documento Nacional de Identidad</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>96</field>
        <field name='sequence'>20</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CUIL'>
        <field name='name'>CUIL</field>
        <field name='description'>Código Único de Identificación Laboral</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>86</field>
        <field name='sequence'>30</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_Sigd'>
        <field name='name'>Sigd</field>
        <field name='description'>Sin identificar/venta global diaria</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>99</field>
        <field name='sequence'>110</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CPF'>
        <field name='name'>CPF</field>
        <field name='description'>CI Policía Federal</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>0</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CBA'>
        <field name='name'>CBA</field>
        <field name='description'>CI Buenos Aires</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>1</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CCat'>
        <field name='name'>CCat</field>
        <field name='description'>CI Catamarca</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>2</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CCor'>
        <field name='name'>CCor</field>
        <field name='description'>CI Córdoba</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>3</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CCorr'>
        <field name='name'>CCorr</field>
        <field name='description'>CI Corrientes</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>4</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIER'>
        <field name='name'>CIER</field>
        <field name='description'>CI Entre Ríos</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>5</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIJ'>
        <field name='name'>CIJ</field>
        <field name='description'>CI Jujuy</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>6</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIMen'>
        <field name='name'>CIMen</field>
        <field name='description'>CI Mendoza</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>7</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CILR'>
        <field name='name'>CILR</field>
        <field name='description'>CI La Rioja</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>8</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIS'>
        <field name='name'>CIS</field>
        <field name='description'>CI Salta</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>9</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CISJ'>
        <field name='name'>CISJ</field>
        <field name='description'>CI San Juan</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>10</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CISL'>
        <field name='name'>CISL</field>
        <field name='description'>CI San Luis</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>11</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CISF'>
        <field name='name'>CISF</field>
        <field name='description'>CI Santa Fe</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>12</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CISdE'>
        <field name='name'>CISdE</field>
        <field name='description'>CI Santiago del Estero</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>13</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIT'>
        <field name='name'>CIT</field>
        <field name='description'>CI Tucumán</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>14</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CICha'>
        <field name='name'>CICha</field>
        <field name='description'>CI Chaco</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>16</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIChu'>
        <field name='name'>CIChu</field>
        <field name='description'>CI Chubut</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>17</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIF'>
        <field name='name'>CIF</field>
        <field name='description'>CI Formosa</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>18</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIMis'>
        <field name='name'>CIMis</field>
        <field name='description'>CI Misiones</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>19</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIN'>
        <field name='name'>CIN</field>
        <field name='description'>CI Neuquén</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>20</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CILP'>
        <field name='name'>CILP</field>
        <field name='description'>CI La Pampa</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>21</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIRN'>
        <field name='name'>CIRN</field>
        <field name='description'>CI Río Negro</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>22</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CISC'>
        <field name='name'>CISC</field>
        <field name='description'>CI Santa Cruz</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>23</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CITdF'>
        <field name='name'>CITdF</field>
        <field name='description'>CI Tierra del Fuego</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>24</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CDI'>
        <field name='name'>CDI</field>
        <field name='description'>CDI</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>87</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_LE'>
        <field name='name'>LE</field>
        <field name='description'>LE</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>89</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_LC'>
        <field name='name'>LC</field>
        <field name='description'>LC</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>90</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_ET'>
        <field name='name'>ET</field>
        <field name='description'>en trámite</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>92</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_AN'>
        <field name='name'>AN</field>
        <field name='description'>Acta nacimiento</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>93</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIBAR'>
        <field name='name'>CIBAR</field>
        <field name='description'>CI Bs. As. RNP</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>95</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CdM'>
        <field name='name'>CdM</field>
        <field name='description'>Certificado de Migración</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>30</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_UpApP'>
        <field name='name'>UpApP</field>
        <field name='description'>Usado por Anses para Padrón</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>88</field>
    </record>
</data>
<data noupdate="True">
    <record model='l10n_latam.identification.type' id='it_CBA'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CCat'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CCor'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CCorr'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIER'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIJ'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIMen'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CILR'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIS'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CISJ'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CISL'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CISF'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CISdE'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIT'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CICha'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIChu'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIF'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIMis'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIN'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CILP'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIRN'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CISC'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CITdF'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CDI'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_LE'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_LC'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_ET'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_AN'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CIBAR'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CdM'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_UpApP'>
        <field name='active' eval='False'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_Sigd'>
        <field name='active' eval='True'/>
    </record>
    <record model='l10n_latam.identification.type' id='it_CPF'>
        <field name='active' eval='False'/>
    </record>
</data>
</odoo>

```

## File: data\res.country.csv

```csv
id,l10n_ar_afip_code,l10n_ar_natural_vat,l10n_ar_legal_entity_vat,l10n_ar_other_vat
base.af,301,50000003015,55000003017,51600003015
base.al,401,50000004011,55000004013,51600004011
base.dz,102,50000001020,55000001022,51600001020
base.as,695,50000006952,55000006954,51600006952
base.de,438,50000004380,55000004382,51600004380
base.ad,404,50000004046,55000004048,51600004046
base.ao,149,50000001497,55000001499,51600001497
base.ai,652,50000006529,55000006520,51600006529
base.aq,265,,,
base.ag,237,50000002256,55000002258,51600002256
base.sa,302,50000003023,55000003025,51600003023
base.ar,200,,,
base.am,349,50000006022,55000006024,51600006022
base.aw,653,50000006537,55000006539,51600006537
base.au,501,50000004992,55000004994,51600004992
base.at,405,50000004054,55000004056,51600004054
base.az,350,50000003902,55000003904,51600003902
base.bs,239,50000002906,55000002908,51600002906
base.bh,303,50000003031,55000003033,51600003031
base.bd,345,50000003457,55000003459,51600003457
base.bb,201,50000002019,55000002010,51600002019
base.by,439,50000004399,55000004390,51600004399
base.be,406,50000004062,55000004064,51600004062
base.bz,236,50000002361,55000002363,51600002361
base.bj,112,50000001624,55000001626,51600001624
base.bm,663,50000006634,55000006636,51600006634
base.bt,305,50000002825,55000002827,51600002825
base.mm,304,50000002841,55000002843,51600002841
base.bo,202,50000000040,55000000042,51600000040
base.bq,241,50000006596,55000006598,51600006596
base.ba,446,50000004461,55000004463,51600004461
base.bw,103,50000001039,55000001030,51600001039
base.br,203,50000000059,55000000050,51600000059
base.bn,346,50000003910,55000003912,51600003910
base.bg,407,50000004070,55000004072,51600004070
base.bf,101,50000001012,55000001014,51600001012
base.bi,104,50000001047,55000001049,51600001047
base.kh,306,50000003066,55000003068,51600003066
base.cm,105,50000001055,55000001057,51600001055
base.ca,204,50000002043,55000002045,51600002043
base.cv,150,50000001500,55000001502,51600001500
base.ky,671,50000006715,55000006717,51600006715
base.cf,107,50000001071,55000001073,51600001071
base.td,111,50000001535,55000001537,51600001535
base.cl,208,50000000032,55000000034,51600000032
base.cn,310,50000003104,55000003106,51600003104
base.cy,311,50000003112,55000003114,51600003112
base.co,205,50000002051,55000002053,51600002051
base.km,155,50000001896,55000001898,51600001896
base.cg,108,,,
base.kp,308,50000003082,55000003084,51600003082
base.kr,309,50000003090,55000003092,51600003090
base.ci,110,50000001101,55000001103,51600001101
base.cr,206,50000001586,55000001588,51600001586
base.hr,447,50000006030,55000006032,51600006030
base.cu,207,50000002396,55000002398,51600002396
base.dk,409,50000004097,55000004099,51600004097
base.dm,233,50000002337,55000002339,51600002337
base.ec,210,50000002426,55000002428,51600002426
base.eg,113,50000001136,55000001138,51600001136
base.sv,211,50000002116,55000002118,51600002116
base.ae,331,50000003317,55000003319,51600003317
base.er,160,50000001853,55000001855,51600001853
base.sk,448,50000006065,55000006067,51600006065
base.si,449,50000004496,55000004498,51600004496
base.es,410,50000004100,55000004102,51600004100
base.ps,314,50000003570,55000003572,51600003570
base.us,212,50000002124,55000002126,51600002124
base.ee,440,50000004402,55000004404,51600004402
base.et,161,50000001144,55000001146,51600001144
base.ru,444,50000006014,55000006016,51600006014
base.fj,512,50000005123,55000005125,51600005123
base.ph,312,50000003120,55000003122,51600003120
base.fi,411,50000004119,55000004110,51600004119
base.fr,412,50000004127,55000004129,51600004127
base.ga,115,50000001152,55000001154,51600001152
base.gm,116,50000001160,55000001162,51600001160
base.ge,351,50000003880,55000003882,51600003880
base.gh,117,50000001179,55000001170,51600001179
base.gi,665,50000006650,55000006652,51600006650
base.gd,240,50000002882,55000002884,51600002884
base.gr,413,50000004135,55000004137,51600004135
base.gl,666,50000006669,55000006660,51600006669
base.gu,667,50000006677,55000006679,51600006677
base.gt,213,50000002132,55000002134,51600002132
base.gy,214,50000002140,55000002142,51600002140
base.gg,670,50000006707,55000006709,51600006707
base.gn,118,50000001187,55000001189,51600001187
base.gw,156,50000001845,55000001847,51600001845
base.gq,119,50000001195,55000001197,51600001195
base.ht,215,50000002159,55000002150,51600002159
base.nl,423,50000004232,55000004234,51600004232
base.hn,216,50000002167,55000002169,51600002167
base.hk,341,50000006685,55000006687,51600006685
base.hu,414,50000004143,55000004145,51600004143
base.in,315,50000003155,55000003157,51600003155
base.id,316,50000003163,55000003165,51600003163
base.iq,317,50000003171,55000003173,51600003171
base.ir,318,50000002930,55000002932,51600002930
base.ie,415,50000004151,55000004153,51600004151
base.im,676,50000006766,55000006768,51600006766
base.cx,672,50000006723,55000006725,51600006723
base.is,416,50000003813,55000003815,51600003813
base.nf,677,50000006774,55000006776,51600006774
base.cc,673,50000006731,55000006733,51600006731
base.ck,654,50000006545,55000006547,51600006545
base.fk,254,,,
base.mp,521,50000005212,55000005214,51600005212
base.mh,520,50000005204,55000005206,51600005204
base.pn,690,50000006901,55000006903,51600006901
base.sb,518,50000005182,55000005184,51600005182
base.tc,678,50000006782,55000006784,51600006782
base.vg,682,50000006820,55000006822,51600006820
base.vi,683,50000006839,55000006830,51600006839
base.il,319,50000002876,55000002878,51600002876
base.it,417,50000003546,55000003548,51600003546
base.jm,217,50000002175,55000002177,51600002175
base.jp,320,50000003201,55000003203,51600003201
base.je,670,50000006707,55000006709,51600006707
base.jo,321,50000003007,55000003009,51600003007
base.kz,352,50000003929,55000003920,51600003929
base.ke,120,50000001209,55000001200,51600001209
base.kg,353,50000003937,55000003939,51600003937
base.ki,514,50000005166,55000005168,51600005166
base.kw,323,50000003236,55000003238,51600003236
base.la,324,50000003244,55000003246,51600003244
base.ls,121,50000001217,55000001219,51600001217
base.lv,441,50000004410,55000004412,51600004410
base.lb,325,50000003252,55000003254,51600003252
base.lr,122,50000001225,55000001227,51600001225
base.ly,123,50000001233,55000001235,51600001233
base.li,418,50000004186,55000004188,51600004186
base.lt,442,50000004429,55000004420,51600004429
base.lu,419,50000004194,55000004196,51600004194
base.mo,344,50000003449,55000003440,51600003449
base.mk,450,50000004909,55000004900,51600004909
base.mg,124,50000001241,55000001243,51600001241
base.my,326,50000003260,55000003262,51600003260
base.mw,125,50000001543,55000001545,51600001543
base.mv,327,50000003279,55000003270,51600003279
base.ml,126,50000001632,55000001634,51600001632
base.mt,420,50000004364,55000004366,51600004364
base.ma,127,50000001276,55000001278,51600001276
base.mu,128,50000001284,55000001286,51600001284
base.mr,129,50000001292,55000001294,51600001292
base.mx,218,50000002183,55000002185,51600002183
base.fm,515,50000005905,55000005907,51600005905
base.md,443,50000004437,55000004439,51600004437
base.mc,421,50000004216,55000004218,51600004216
base.mn,329,50000003295,55000003297,51600003295
base.me,453,,,
base.ms,686,50000006863,55000006865,51600006863
base.mz,151,50000001519,55000001510,51600001519
base.na,158,50000001837,55000001839,51600001837
base.nr,503,50000005034,55000005036,51600005034
base.np,330,50000003309,55000003300,51600003309
base.ni,219,50000002191,55000002193,51600002191
base.ne,130,50000001306,55000001308,51600001306
base.ng,131,50000001314,55000001316,51600001314
base.nu,687,50000006871,55000006873,51600006871
base.no,422,50000004224,55000004226,51600004224
base.nz,504,50000005042,55000005044,51600005042
base.om,328,50000003287,55000003289,51600003287
base.pk,332,50000003325,55000003327,51600003325
base.pw,516,50000005913,55000005915,51600005913
base.pa,220,50000002205,55000002207,51600002205
base.pg,513,50000005131,55000005133,51600005131
base.py,221,50000000024,55000000026,51600000024
base.pe,222,50000002221,55000002223,51600002221
base.pf,656,50000006561,55000006563,51600006561
base.pl,424,50000004240,55000004242,51600004240
base.pt,425,50000004259,55000004250,51600004259
base.pr,223,50000002213,55000002215,51600002213
base.qa,322,50000002981,55000002983,51600002981
base.uk,426,50000004267,55000004269,51600004267
base.cz,451,50000006057,55000006059,51600006057
base.cd,109,50000001527,55000001529,55000001529
base.do,209,50000002094,55000002096,51600002094
base.rw,133,50000001330,55000001332,51600001330
base.ro,427,50000004275,55000004277,51600004275
base.ws,506,50000005069,55000005069,51600005069
base.kn,238,50000002892,55000002894,51600002892
base.sm,428,50000004283,55000004285,51600004283
base.pm,680,50000006804,55000006806,51600006804
base.sh,662,50000006979,55000006970,51600006979
base.lc,234,50000002345,55000002347,51600002345
base.va,431,50000004313,55000004315,51600004313
base.st,157,50000001829,55000001820,51600001829
base.vc,235,50000002353,55000002355,51600002353
base.sn,134,50000001349,55000001340,51600001349
base.rs,454,,,
base.sc,152,50000001810,55000001812,51600001810
base.sl,135,50000001357,55000001359,51600001357
base.sg,333,50000003333,55000003335,51600003333
base.sy,334,50000003341,55000003343,51600003341
base.so,136,50000001365,55000001367,51600001365
base.lk,307,50000003074,55000003076,51600003074
base.sz,137,50000001373,55000001375,51600001373
base.za,159,50000001713,55000001715,51600001713
base.sd,138,50000001381,55000001383,51600001381
base.se,429,50000004291,55000004293,51600004291
base.ch,430,50000004305,55000004307,51600004305
base.sr,232,50000002329,55000002320,51600002329
base.sj,696,50000006960,55000006962,51600006960
base.th,335,50000002914,55000002916,51600002914
base.tw,313,50000003139,55000003130,51600003139
base.tj,354,50000003899,55000003890,51600003899
base.tz,139,50000001551,55000001553,51600001551
base.tl,358,,,
base.tg,140,50000001403,55000001405,51600001403
base.tk,699,50000006995,55000006997,51600006995
base.to,519,50000005190,55000005192,51600005190
base.tt,224,50000002434,55000002436,51600002434
base.tn,141,50000001411,55000001413,51600001411
base.tm,355,50000003554,55000003556,51600003554
base.tr,336,50000003503,55000003505,51600003503
base.tv,517,50000005174,55000005176,51600005174
base.ua,445,50000006049,55000006040,51600006049
base.ug,142,50000001705,55000001707,51600001705
base.uy,225,50000000016,55000000018,51600000016
base.uz,356,50000003562,55000003564,51600003562
base.vu,505,50000005050,55000005052,51600005050
base.ve,226,50000002264,55000002266,51600002264
base.vn,337,50000003376,55000003378,51600003376
base.ye,348,50000003392,55000003394,51600003392
base.dj,153,50000001861,55000001863,51600001861
base.zm,144,50000001446,55000001448,51600001446
base.zw,132,50000001322,55000001324,51600001322

```

## File: data\res.currency.csv

```csv
id,l10n_ar_afip_code
base.ARS,PES
base.EUR,060
base.USD,DOL
base.ANG,028
base.AUD,026
base.BOB,031
base.BRL,012
base.CAD,018
base.CHF,009
base.CLP,033
base.CNY,064
base.COP,032
base.CZK,024
base.DKK,014
base.DOP,042
base.ECS,036
base.EGP,046
base.GBP,021
base.GTQ,055
base.HKD,051
base.HNL,063
base.HUF,056
base.ILS,030
base.INR,062
base.ITL,004
base.JMD,053
base.JPY,019
base.KWD,059
base.MAD,045
base.MXN,010
base.NIO,044
base.NOK,015
base.PAB,043
base.PEN,035
base.PLN,061
base.PYG,029
base.RON,040
base.SAR,047
base.SEK,016
base.SGD,052
base.THB,057
base.TWD,054
base.UYU,011
base.VEF,023
base.ZAR,034

```

## File: data\res_partner_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="True">

    <record model='res.partner' id='par_cfa'>
        <field name='name'>Consumidor Final Anónimo</field>
        <field name='l10n_latam_identification_type_id' ref='l10n_ar.it_Sigd'/>
        <field name='l10n_ar_afip_responsibility_type_id' ref="res_CF"/>
    </record>

    <record model='res.partner' id='par_iibb_pagar'>
        <field name='name'>IIBB a pagar</field>
    </record>

    <record id="partner_afip" model="res.partner">
        <field name="name">AFIP</field>
        <field name="is_company" eval="True"/>
        <field name='l10n_latam_identification_type_id' ref="l10n_ar.it_cuit"/>
        <field name='vat'>33693450239</field>
        <field name='l10n_ar_afip_responsibility_type_id' ref="res_IVA_NO_ALC"/>
        <field name='l10n_ar_special_purchase_document_type_ids' eval='[(4, ref("dc_desp_imp"), False), (4, ref("dc_imp_serv"), False)]'/>
    </record>

</odoo>

```

## File: data\uom_uom_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="True">

    <record model='uom.uom' id='uom.product_uom_kgm'>
        <field name='l10n_ar_afip_code'>01</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_meter'>
        <field name='l10n_ar_afip_code'>02</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_litre'>
        <field name='l10n_ar_afip_code'>05</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_unit'>
        <field name='l10n_ar_afip_code'>07</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_dozen'>
        <field name='l10n_ar_afip_code'>09</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_cm'>
        <field name='l10n_ar_afip_code'>20</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_day'>
        <field name='l10n_ar_afip_code'>98</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_floz'>
        <field name='l10n_ar_afip_code'>98</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_foot'>
        <field name='l10n_ar_afip_code'>98</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_gram'>
        <field name='l10n_ar_afip_code'>14</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_gal'>
        <field name='l10n_ar_afip_code'>98</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_hour'>
        <field name='l10n_ar_afip_code'>98</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_inch'>
        <field name='l10n_ar_afip_code'>98</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_km'>
        <field name='l10n_ar_afip_code'>17</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_lb'>
        <field name='l10n_ar_afip_code'>98</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_mile'>
        <field name='l10n_ar_afip_code'>98</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_oz'>
        <field name='l10n_ar_afip_code'>98</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_qt'>
        <field name='l10n_ar_afip_code'>98</field>
    </record>
    <record model='uom.uom' id='uom.product_uom_ton'>
        <field name='l10n_ar_afip_code'>29</field>
    </record>
</odoo>

```

## File: models\account_chart_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, _
from odoo.exceptions import UserError
from odoo.http import request


class AccountChartTemplate(models.Model):

    _inherit = 'account.chart.template'

    def _get_fp_vals(self, company, position):
        res = super()._get_fp_vals(company, position)
        if company.country_id.code == "AR":
            res['l10n_ar_afip_responsibility_type_ids'] = [
                (6, False, position.l10n_ar_afip_responsibility_type_ids.ids)]
        return res

    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        """ If Argentinian chart, we modify the defaults values of sales journal to be a preprinted journal
        """
        res = super()._prepare_all_journals(acc_template_ref, company, journals_dict=journals_dict)
        if company.country_id.code == "AR":
            for vals in res:
                if vals['type'] == 'sale':
                    vals.update({
                        "name": "Ventas Preimpreso",
                        "code": "0001",
                        "l10n_ar_afip_pos_number": 1,
                        "l10n_ar_afip_pos_partner_id": company.partner_id.id,
                        "l10n_ar_afip_pos_system": 'II_IM',
                        "l10n_ar_share_sequences": True,
                        "refund_sequence": False
                    })
        return res

    @api.model
    def _get_ar_responsibility_match(self, chart_template_id):
        """ return responsibility type that match with the given chart_template_id
        """
        match = {
            self.env.ref('l10n_ar.l10nar_base_chart_template').id: self.env.ref('l10n_ar.res_RM'),
            self.env.ref('l10n_ar.l10nar_ex_chart_template').id: self.env.ref('l10n_ar.res_IVAE'),
            self.env.ref('l10n_ar.l10nar_ri_chart_template').id: self.env.ref('l10n_ar.res_IVARI'),
        }
        return match.get(chart_template_id)

    def _load(self, sale_tax_rate, purchase_tax_rate, company):
        """ Set companies AFIP Responsibility and Country if AR CoA is installed, also set tax calculation rounding
        method required in order to properly validate match AFIP invoices.

        Also, raise a warning if the user is trying to install a CoA that does not match with the defined AFIP
        Responsibility defined in the company
        """
        self.ensure_one()
        coa_responsibility = self._get_ar_responsibility_match(self.id)
        if coa_responsibility:
            company_responsibility = company.l10n_ar_afip_responsibility_type_id
            company.write({
                'l10n_ar_afip_responsibility_type_id': coa_responsibility.id,
                'country_id': self.env['res.country'].search([('code', '=', 'AR')]).id,
                'tax_calculation_rounding_method': 'round_globally',
            })
            # set CUIT identification type (which is the argentinian vat) in the created company partner instead of
            # the default VAT type.
            company.partner_id.l10n_latam_identification_type_id = self.env.ref('l10n_ar.it_cuit')

        res = super()._load(sale_tax_rate, purchase_tax_rate, company)

        # If Responsable Monotributista remove the default purchase tax
        if self == self.env.ref('l10n_ar.l10nar_base_chart_template') or \
           self == self.env.ref('l10n_ar.l10nar_ex_chart_template'):
            company.account_purchase_tax_id = self.env['account.tax']

        return res

```

## File: models\account_fiscal_position.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api, _


class AccountFiscalPosition(models.Model):

    _inherit = 'account.fiscal.position'

    l10n_ar_afip_responsibility_type_ids = fields.Many2many(
        'l10n_ar.afip.responsibility.type', 'l10n_ar_afip_reponsibility_type_fiscal_pos_rel',
        string='AFIP Responsibility Types', help='List of AFIP responsibilities where this fiscal position '
        'should be auto-detected')

    @api.model
    def get_fiscal_position(self, partner_id, delivery_id=None):
        """ Take into account the partner afip responsibility in order to auto-detect the fiscal position """
        company = self.env.company
        if company.country_id.code == "AR":
            PartnerObj = self.env['res.partner']
            partner = PartnerObj.browse(partner_id)

            # if no delivery use invoicing
            if delivery_id:
                delivery = PartnerObj.browse(delivery_id)
            else:
                delivery = partner

            # partner manually set fiscal position always win
            if delivery.property_account_position_id or partner.property_account_position_id:
                return delivery.property_account_position_id or partner.property_account_position_id
            domain = [
                ('auto_apply', '=', True),
                ('l10n_ar_afip_responsibility_type_ids', '=', self.env['res.partner'].browse(
                    partner_id).l10n_ar_afip_responsibility_type_id.id),
                ('company_id', '=', company.id),
            ]
            return self.sudo().search(domain, limit=1)
        return super().get_fiscal_position(partner_id, delivery_id=delivery_id)

    @api.onchange('l10n_ar_afip_responsibility_type_ids', 'country_group_id', 'country_id', 'zip_from', 'zip_to')
    def _onchange_afip_responsibility(self):
        if self.company_id.country_id.code == "AR":
            if self.l10n_ar_afip_responsibility_type_ids and any(['country_group_id', 'country_id', 'zip_from', 'zip_to']):
                return {'warning': {
                    'title': _("Warning"),
                    'message': _('If use AFIP Responsibility then the country / zip codes will be not take into account'),
                }}

```

## File: models\account_fiscal_position_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountFiscalPositionTemplate(models.Model):

    _inherit = 'account.fiscal.position.template'

    l10n_ar_afip_responsibility_type_ids = fields.Many2many(
        'l10n_ar.afip.responsibility.type', 'l10n_ar_afip_reponsibility_type_fiscal_pos_temp_rel',
        string='AFIP Responsibility Types')

```

## File: models\account_journal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError, RedirectWarning


class AccountJournal(models.Model):

    _inherit = "account.journal"

    l10n_ar_afip_pos_system = fields.Selection(
        selection='_get_l10n_ar_afip_pos_types_selection', string='AFIP POS System')
    l10n_ar_afip_pos_number = fields.Integer(
        'AFIP POS Number', help='This is the point of sale number assigned by AFIP in order to generate invoices')
    company_partner = fields.Many2one('res.partner', related='company_id.partner_id')
    l10n_ar_afip_pos_partner_id = fields.Many2one(
        'res.partner', 'AFIP POS Address', help='This is the address used for invoice reports of this POS',
        domain="['|', ('id', '=', company_partner), '&', ('id', 'child_of', company_partner), ('type', '!=', 'contact')]"
    )
    l10n_ar_share_sequences = fields.Boolean(
        'Unified Book', help='Use same sequence for documents with the same letter')

    def _get_l10n_ar_afip_pos_types_selection(self):
        """ Return the list of values of the selection field. """
        return [
            ('II_IM', _('Pre-printed Invoice')),
            ('RLI_RLM', _('Online Invoice')),
            ('BFERCEL', _('Electronic Fiscal Bond - Online Invoice')),
            ('FEERCELP', _('Export Voucher - Billing Plus')),
            ('FEERCEL', _('Export Voucher - Online Invoice')),
            ('CPERCEL', _('Product Coding - Online Voucher')),
        ]

    def _get_journal_letter(self, counterpart_partner=False):
        """ Regarding the AFIP responsibility of the company and the type of journal (sale/purchase), get the allowed
        letters. Optionally, receive the counterpart partner (customer/supplier) and get the allowed letters to work
        with him. This method is used to populate document types on journals and also to filter document types on
        specific invoices to/from customer/supplier
        """
        self.ensure_one()
        letters_data = {
            'issued': {
                '1': ['A', 'B', 'E', 'M'],
                '3': [],
                '4': ['C'],
                '5': [],
                '6': ['C', 'E'],
                '9': ['I'],
                '10': [],
                '13': ['C', 'E'],
                '99': []
            },
            'received': {
                '1': ['A', 'B', 'C', 'E', 'M', 'I'],
                '3': ['B', 'C', 'I'],
                '4': ['B', 'C', 'I'],
                '5': ['B', 'C', 'I'],
                '6': ['A', 'B', 'C', 'I'],
                '9': ['E'],
                '10': ['E'],
                '13': ['A', 'B', 'C', 'I'],
                '99': ['B', 'C', 'I']
            },
        }
        if not self.company_id.l10n_ar_afip_responsibility_type_id:
            action = self.env.ref('base.action_res_company_form')
            msg = _('Can not create chart of account until you configure your company AFIP Responsibility and VAT.')
            raise RedirectWarning(msg, action.id, _('Go to Companies'))

        letters = letters_data['issued' if self.type == 'sale' else 'received'][
            self.company_id.l10n_ar_afip_responsibility_type_id.code]
        if counterpart_partner:
            counterpart_letters = letters_data['issued' if self.type == 'purchase' else 'received'].get(
                counterpart_partner.l10n_ar_afip_responsibility_type_id.code, [])
            letters = list(set(letters) & set(counterpart_letters))
        return letters

    def _get_journal_codes(self):
        self.ensure_one()
        usual_codes = ['1', '2', '3', '6', '7', '8', '11', '12', '13']
        mipyme_codes = ['201', '202', '203', '206', '207', '208', '211', '212', '213']
        invoice_m_code = ['51', '52', '53']
        receipt_m_code = ['54']
        receipt_codes = ['4', '9', '15']
        expo_codes = ['19', '20', '21']
        if self.type != 'sale':
            return []
        elif self.l10n_ar_afip_pos_system == 'II_IM':
            # pre-printed invoice
            return usual_codes + receipt_codes + expo_codes + invoice_m_code + receipt_m_code
        elif self.l10n_ar_afip_pos_system in ['RAW_MAW', 'RLI_RLM']:
            # electronic/online invoice
            return usual_codes + receipt_codes + invoice_m_code + receipt_m_code + mipyme_codes
        elif self.l10n_ar_afip_pos_system in ['CPERCEL', 'CPEWS']:
            # invoice with detail
            return usual_codes + invoice_m_code
        elif self.l10n_ar_afip_pos_system in ['BFERCEL', 'BFEWS']:
            # Bonds invoice
            return usual_codes + mipyme_codes
        elif self.l10n_ar_afip_pos_system in ['FEERCEL', 'FEEWS', 'FEERCELP']:
            return expo_codes

    @api.constrains('type', 'l10n_ar_afip_pos_system', 'l10n_ar_afip_pos_number', 'l10n_ar_share_sequences',
                    'l10n_latam_use_documents')
    def _check_afip_configurations(self):
        """ Do not let the user update the journal if it already contains confirmed invoices """
        journals = self.filtered(lambda x: x.company_id.country_id.code == "AR" and x.type in ['sale', 'purchase'])
        invoices = self.env['account.move'].search([('journal_id', 'in', journals.ids), ('posted_before', '=', True)], limit=1)
        if invoices:
            raise ValidationError(
                _("You can not change the journal's configuration if it already has validated invoices") + ' ('
                + ', '.join(invoices.mapped('journal_id').mapped('name')) + ')')

    @api.constrains('l10n_ar_afip_pos_number')
    def _check_afip_pos_number(self):
        to_review = self.filtered(
            lambda x: x.type == 'sale' and x.l10n_latam_use_documents and
            x.company_id.country_id.code == "AR")

        if to_review.filtered(lambda x: x.l10n_ar_afip_pos_number == 0):
            raise ValidationError(_('Please define an AFIP POS number'))

        if to_review.filtered(lambda x: x.l10n_ar_afip_pos_number > 99999):
            raise ValidationError(_('Please define a valid AFIP POS number (5 digits max)'))

    @api.onchange('l10n_ar_afip_pos_system')
    def _onchange_l10n_ar_afip_pos_system(self):
        """ On 'Pre-printed Invoice' the usual is to share sequences. On other types, do not share """
        self.l10n_ar_share_sequences = bool(self.l10n_ar_afip_pos_system == 'II_IM')

    @api.onchange('l10n_ar_afip_pos_number', 'type')
    def _onchange_set_short_name(self):
        """ Will define the AFIP POS Address field domain taking into account the company configured in the journal
        The short code of the journal only admit 5 characters, so depending on the size of the pos_number (also max 5)
        we add or not a prefix to identify sales journal.
        """
        if self.type == 'sale' and self.l10n_ar_afip_pos_number:
            self.code = "%05i" % self.l10n_ar_afip_pos_number

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api, _
from odoo.exceptions import UserError, RedirectWarning, ValidationError
from dateutil.relativedelta import relativedelta
from lxml import etree
import logging
_logger = logging.getLogger(__name__)


class AccountMove(models.Model):

    _inherit = 'account.move'

    @api.model
    def _l10n_ar_get_document_number_parts(self, document_number, document_type_code):
        # import shipments
        if document_type_code in ['66', '67']:
            pos = invoice_number = '0'
        else:
            pos, invoice_number = document_number.split('-')
        return {'invoice_number': int(invoice_number), 'point_of_sale': int(pos)}

    l10n_ar_afip_responsibility_type_id = fields.Many2one(
        'l10n_ar.afip.responsibility.type', string='AFIP Responsibility Type', help='Defined by AFIP to'
        ' identify the type of responsibilities that a person or a legal entity could have and that impacts in the'
        ' type of operations and requirements they need.')

    l10n_ar_currency_rate = fields.Float(copy=False, digits=(16, 6), readonly=True, string="Currency Rate")

    # Mostly used on reports
    l10n_ar_afip_concept = fields.Selection(
        compute='_compute_l10n_ar_afip_concept', selection='_get_afip_invoice_concepts', string="AFIP Concept",
        help="A concept is suggested regarding the type of the products on the invoice but it is allowed to force a"
        " different type if required.")
    l10n_ar_afip_service_start = fields.Date(string='AFIP Service Start Date', readonly=True, states={'draft': [('readonly', False)]})
    l10n_ar_afip_service_end = fields.Date(string='AFIP Service End Date', readonly=True, states={'draft': [('readonly', False)]})

    @api.constrains('move_type', 'journal_id')
    def _check_moves_use_documents(self):
        """ Do not let to create not invoices entries in journals that use documents """
        not_invoices = self.filtered(lambda x: x.company_id.country_id.code == "AR" and x.journal_id.type in ['sale', 'purchase'] and x.l10n_latam_use_documents and not x.is_invoice())
        if not_invoices:
            raise ValidationError(_("The selected Journal can't be used in this transaction, please select one that doesn't use documents as these are just for Invoices."))

    @api.constrains('move_type', 'l10n_latam_document_type_id')
    def _check_invoice_type_document_type(self):
        """ LATAM module define that we are not able to use debit_note or invoice document types in an invoice refunds,
        However for Argentinian Document Type's 99 (internal type = invoice) we are able to used in a refund invoices.

        In this method we exclude the argentinian documents that can be used as invoice and refund from the generic
        constraint """
        docs_used_for_inv_and_ref = self.filtered(
            lambda x: x.country_code == 'AR' and
            x.l10n_latam_document_type_id.code in self._get_l10n_ar_codes_used_for_inv_and_ref() and
            x.move_type in ['out_refund', 'in_refund'])

        super(AccountMove, self - docs_used_for_inv_and_ref)._check_invoice_type_document_type()

    def _get_afip_invoice_concepts(self):
        """ Return the list of values of the selection field. """
        return [('1', 'Products / Definitive export of goods'), ('2', 'Services'), ('3', 'Products and Services'),
                ('4', '4-Other (export)')]

    @api.depends('invoice_line_ids', 'invoice_line_ids.product_id', 'invoice_line_ids.product_id.type', 'journal_id')
    def _compute_l10n_ar_afip_concept(self):
        recs_afip = self.filtered(lambda x: x.company_id.country_id.code == "AR" and x.l10n_latam_use_documents)
        for rec in recs_afip:
            rec.l10n_ar_afip_concept = rec._get_concept()
        remaining = self - recs_afip
        remaining.l10n_ar_afip_concept = ''

    def _get_concept(self):
        """ Method to get the concept of the invoice considering the type of the products on the invoice """
        self.ensure_one()
        invoice_lines = self.invoice_line_ids.filtered(lambda x: not x.display_type)
        product_types = set([x.product_id.type for x in invoice_lines if x.product_id])
        consumable = set(['consu', 'product'])
        service = set(['service'])
        mixed = set(['consu', 'service', 'product'])
        # on expo invoice you can mix services and products
        expo_invoice = self.l10n_latam_document_type_id.code in ['19', '20', '21']

        # Default value "product"
        afip_concept = '1'
        if product_types == service:
            afip_concept = '2'
        elif product_types - consumable and product_types - service and not expo_invoice:
            afip_concept = '3'
        return afip_concept

    @api.model
    def _get_l10n_ar_codes_used_for_inv_and_ref(self):
        """ List of document types that can be used as an invoice and refund. This list can be increased once needed
        and demonstrated. As far as we've checked document types of wsfev1 don't allow negative amounts so, for example
        document 60 and 61 could not be used as refunds. """
        return ['99', '186', '188', '189']

    def _get_l10n_latam_documents_domain(self):
        self.ensure_one()
        domain = super()._get_l10n_latam_documents_domain()
        if self.journal_id.company_id.country_id.code == "AR":
            letters = self.journal_id._get_journal_letter(counterpart_partner=self.partner_id.commercial_partner_id)
            domain += ['|', ('l10n_ar_letter', '=', False), ('l10n_ar_letter', 'in', letters)]
            codes = self.journal_id._get_journal_codes()
            if codes:
                domain.append(('code', 'in', codes))
            if self.move_type == 'in_refund':
                domain = ['|', ('code', 'in', self._get_l10n_ar_codes_used_for_inv_and_ref())] + domain
        return domain

    def _check_argentinian_invoice_taxes(self):

        # check vat on companies thats has it (Responsable inscripto)
        for inv in self.filtered(lambda x: x.company_id.l10n_ar_company_requires_vat):
            purchase_aliquots = 'not_zero'
            # we require a single vat on each invoice line except from some purchase documents
            if inv.move_type in ['in_invoice', 'in_refund'] and inv.l10n_latam_document_type_id.purchase_aliquots == 'zero':
                purchase_aliquots = 'zero'
            for line in inv.mapped('invoice_line_ids').filtered(lambda x: x.display_type not in ('line_section', 'line_note')):
                vat_taxes = line.tax_ids.filtered(lambda x: x.tax_group_id.l10n_ar_vat_afip_code)
                if len(vat_taxes) != 1:
                    raise UserError(_('There should be a single tax from the "VAT" tax group per line, add it to "%s". If you already have it, please check the tax configuration, in advanced options, in the corresponding field "Tax Group".') % line.name)

                elif purchase_aliquots == 'zero' and vat_taxes.tax_group_id.l10n_ar_vat_afip_code != '0':
                    raise UserError(_('On invoice id "%s" you must use VAT Not Applicable on every line.')  % inv.id)
                elif purchase_aliquots == 'not_zero' and vat_taxes.tax_group_id.l10n_ar_vat_afip_code == '0':
                    raise UserError(_('On invoice id "%s" you must use VAT taxes different than VAT Not Applicable.')  % inv.id)

    def _set_afip_service_dates(self):
        for rec in self.filtered(lambda m: m.invoice_date and m.l10n_ar_afip_concept in ['2', '3', '4']):
            if not rec.l10n_ar_afip_service_start:
                rec.l10n_ar_afip_service_start = rec.invoice_date + relativedelta(day=1)
            if not rec.l10n_ar_afip_service_end:
                rec.l10n_ar_afip_service_end = rec.invoice_date + relativedelta(day=1, days=-1, months=+1)

    def _set_afip_responsibility(self):
        """ We save the information about the receptor responsability at the time we validate the invoice, this is
        necessary because the user can change the responsability after that any time """
        for rec in self:
            rec.l10n_ar_afip_responsibility_type_id = rec.commercial_partner_id.l10n_ar_afip_responsibility_type_id.id

    def _set_afip_rate(self):
        """ We set the l10n_ar_currency_rate value with the accounting date. This should be done
        after invoice has been posted in order to have the proper accounting date"""
        for rec in self:
            if rec.company_id.currency_id == rec.currency_id:
                rec.l10n_ar_currency_rate = 1.0
            elif not rec.l10n_ar_currency_rate:
                rec.l10n_ar_currency_rate = rec.currency_id._convert(
                    1.0, rec.company_id.currency_id, rec.company_id, rec.date, round=False)

    @api.onchange('partner_id')
    def _onchange_afip_responsibility(self):
        if self.company_id.country_id.code == 'AR' and self.l10n_latam_use_documents and self.partner_id \
           and not self.partner_id.l10n_ar_afip_responsibility_type_id:
            return {'warning': {
                'title': _('Missing Partner Configuration'),
                'message': _('Please configure the AFIP Responsibility for "%s" in order to continue') % (
                    self.partner_id.name)}}

    @api.onchange('partner_id')
    def _onchange_partner_journal(self):
        """ This method is used when the invoice is created from the sale or subscription """
        expo_journals = ['FEERCEL', 'FEEWS', 'FEERCELP']
        for rec in self.filtered(lambda x: x.company_id.country_id.code == "AR" and x.journal_id.type == 'sale'
                                 and x.l10n_latam_use_documents and x.partner_id.l10n_ar_afip_responsibility_type_id):
            res_code = rec.partner_id.l10n_ar_afip_responsibility_type_id.code
            domain = [('company_id', '=', rec.company_id.id), ('l10n_latam_use_documents', '=', True), ('type', '=', 'sale')]
            journal = self.env['account.journal']
            msg = False
            if res_code in ['9', '10'] and rec.journal_id.l10n_ar_afip_pos_system not in expo_journals:
                # if partner is foregin and journal is not of expo, we try to change to expo journal
                journal = journal.search(domain + [('l10n_ar_afip_pos_system', 'in', expo_journals)], limit=1)
                msg = _('You are trying to create an invoice for foreign partner but you don\'t have an exportation journal')
            elif res_code not in ['9', '10'] and rec.journal_id.l10n_ar_afip_pos_system in expo_journals:
                # if partner is NOT foregin and journal is for expo, we try to change to local journal
                journal = journal.search(domain + [('l10n_ar_afip_pos_system', 'not in', expo_journals)], limit=1)
                msg = _('You are trying to create an invoice for domestic partner but you don\'t have a domestic market journal')
            if journal:
                rec.journal_id = journal.id
            elif msg:
                # Throw an error to user in order to proper configure the journal for the type of operation
                action = self.env.ref('account.action_account_journal_form')
                raise RedirectWarning(msg, action.id, _('Go to Journals'))

    def _post(self, soft=True):
        ar_invoices = self.filtered(lambda x: x.company_id.country_id.code == "AR" and x.l10n_latam_use_documents)
        # We make validations here and not with a constraint because we want validation before sending electronic
        # data on l10n_ar_edi
        ar_invoices._check_argentinian_invoice_taxes()
        posted = super()._post(soft=soft)

        posted_ar_invoices = posted & ar_invoices
        posted_ar_invoices._set_afip_responsibility()
        posted_ar_invoices._set_afip_rate()
        posted_ar_invoices._set_afip_service_dates()
        return posted

    def _reverse_moves(self, default_values_list=None, cancel=False):
        if not default_values_list:
            default_values_list = [{} for move in self]
        for move, default_values in zip(self, default_values_list):
            default_values.update({
                'l10n_ar_afip_service_start': move.l10n_ar_afip_service_start,
                'l10n_ar_afip_service_end': move.l10n_ar_afip_service_end,
            })
        return super()._reverse_moves(default_values_list=default_values_list, cancel=cancel)

    @api.onchange('l10n_latam_document_type_id', 'l10n_latam_document_number')
    def _inverse_l10n_latam_document_number(self):
        super()._inverse_l10n_latam_document_number()

        to_review = self.filtered(
            lambda x: x.journal_id.type == 'sale' and x.l10n_latam_document_type_id and x.l10n_latam_document_number and
            (x.l10n_latam_manual_document_number or not x.highest_name)
            and x.l10n_latam_document_type_id.country_id.code == 'AR')
        for rec in to_review:
            number = rec.l10n_latam_document_type_id._format_document_number(rec.l10n_latam_document_number)
            current_pos = int(number.split("-")[0])
            if current_pos != rec.journal_id.l10n_ar_afip_pos_number:
                invoices = self.search([('journal_id', '=', rec.journal_id.id), ('posted_before', '=', True)], limit=1)
                # If there is no posted before invoices the user can change the POS number (x.l10n_latam_document_number)
                if (not invoices):
                    rec.journal_id.l10n_ar_afip_pos_number = current_pos
                    rec.journal_id._onchange_set_short_name()
                # If not, avoid that the user change the POS number
                else:
                    raise UserError(_('The document number can not be changed for this journal, you can only modify'
                                      ' the POS number if there is not posted (or posted before) invoices'))

    def _get_formatted_sequence(self, number=0):
        return "%s %05d-%08d" % (self.l10n_latam_document_type_id.doc_code_prefix,
                                 self.journal_id.l10n_ar_afip_pos_number, number)

    def _get_starting_sequence(self):
        """ If use documents then will create a new starting sequence using the document type code prefix and the
        journal document number with a 8 padding number """
        if self.journal_id.l10n_latam_use_documents and self.company_id.country_id.code == "AR":
            if self.l10n_latam_document_type_id:
                return self._get_formatted_sequence()
        return super()._get_starting_sequence()

    def _get_last_sequence(self, relaxed=False, lock=True):
        """ If use share sequences we need to recompute the sequence to add the proper document code prefix """
        res = super()._get_last_sequence(relaxed=relaxed, lock=lock)
        if res and self.journal_id.l10n_ar_share_sequences and self.l10n_latam_document_type_id.doc_code_prefix not in res:
            res = self._get_formatted_sequence(number=self._l10n_ar_get_document_number_parts(
                res.split()[-1], self.l10n_latam_document_type_id.code)['invoice_number'])
        return res

    def _get_last_sequence_domain(self, relaxed=False):
        where_string, param = super(AccountMove, self)._get_last_sequence_domain(relaxed)
        if self.company_id.country_id.code == "AR" and self.l10n_latam_use_documents:
            if not self.journal_id.l10n_ar_share_sequences:
                where_string += " AND l10n_latam_document_type_id = %(l10n_latam_document_type_id)s"
                param['l10n_latam_document_type_id'] = self.l10n_latam_document_type_id.id or 0
            elif self.journal_id.l10n_ar_share_sequences:
                where_string += " AND l10n_latam_document_type_id in %(l10n_latam_document_type_ids)s"
                param['l10n_latam_document_type_ids'] = tuple(self.l10n_latam_document_type_id.search(
                    [('l10n_ar_letter', '=', self.l10n_latam_document_type_id.l10n_ar_letter)]).ids)
        return where_string, param

    def _l10n_ar_get_amounts(self, company_currency=False):
        """ Method used to prepare data to present amounts and taxes related amounts when creating an
        electronic invoice for argentinian and the txt files for digital VAT books. Only take into account the argentinian taxes """
        self.ensure_one()
        amount_field = company_currency and 'balance' or 'price_subtotal'
        # if we use balance we need to correct sign (on price_subtotal is positive for refunds and invoices)
        sign = -1 if (company_currency and self.is_inbound()) else 1

        # if we are on a document that works invoice and refund and it's a refund, we need to export it as negative
        sign = -sign if self.move_type in ('out_refund', 'in_refund') and\
            self.l10n_latam_document_type_id.code in self._get_l10n_ar_codes_used_for_inv_and_ref() else sign

        tax_lines = self.line_ids.filtered('tax_line_id')
        vat_taxes = tax_lines.filtered(lambda r: r.tax_line_id.tax_group_id.l10n_ar_vat_afip_code)

        vat_taxable = self.env['account.move.line']
        for line in self.invoice_line_ids:
            if any(tax.tax_group_id.l10n_ar_vat_afip_code and tax.tax_group_id.l10n_ar_vat_afip_code not in ['0', '1', '2'] for tax in line.tax_ids):
                vat_taxable |= line

        profits_tax_group = self.env.ref('l10n_ar.tax_group_percepcion_ganancias')
        return {'vat_amount': sign * sum(vat_taxes.mapped(amount_field)),
                # For invoices of letter C should not pass VAT
                'vat_taxable_amount': sign * sum(vat_taxable.mapped(amount_field)) if self.l10n_latam_document_type_id.l10n_ar_letter != 'C' else self.amount_untaxed,
                'vat_exempt_base_amount': sign * sum(self.invoice_line_ids.filtered(lambda x: x.tax_ids.filtered(lambda y: y.tax_group_id.l10n_ar_vat_afip_code == '2')).mapped(amount_field)),
                'vat_untaxed_base_amount': sign * sum(self.invoice_line_ids.filtered(lambda x: x.tax_ids.filtered(lambda y: y.tax_group_id.l10n_ar_vat_afip_code == '1')).mapped(amount_field)),
                # used on FE
                'not_vat_taxes_amount': sign * sum((tax_lines - vat_taxes).mapped(amount_field)),
                # used on BFE + TXT
                'iibb_perc_amount': sign * sum(tax_lines.filtered(lambda r: r.tax_line_id.tax_group_id.l10n_ar_tribute_afip_code == '07').mapped(amount_field)),
                'mun_perc_amount': sign * sum(tax_lines.filtered(lambda r: r.tax_line_id.tax_group_id.l10n_ar_tribute_afip_code == '08').mapped(amount_field)),
                'intern_tax_amount': sign * sum(tax_lines.filtered(lambda r: r.tax_line_id.tax_group_id.l10n_ar_tribute_afip_code == '04').mapped(amount_field)),
                'other_taxes_amount': sign * sum(tax_lines.filtered(lambda r: r.tax_line_id.tax_group_id.l10n_ar_tribute_afip_code == '99').mapped(amount_field)),
                'profits_perc_amount': sign * sum(tax_lines.filtered(lambda r: r.tax_line_id.tax_group_id == profits_tax_group).mapped(amount_field)),
                'vat_perc_amount': sign * sum(tax_lines.filtered(lambda r: r.tax_line_id.tax_group_id.l10n_ar_tribute_afip_code == '06').mapped(amount_field)),
                'other_perc_amount': sign * sum(tax_lines.filtered(lambda r: r.tax_line_id.tax_group_id.l10n_ar_tribute_afip_code == '09' and r.tax_line_id.tax_group_id != profits_tax_group).mapped(amount_field)),
                }

    def _get_vat(self, company_currency=False):
        """ Applies on wsfe web service and in the VAT digital books """
        amount_field = company_currency and 'balance' or 'price_subtotal'
        # if we use balance we need to correct sign (on price_subtotal is positive for refunds and invoices)
        sign = -1 if (company_currency and self.is_inbound()) else 1

        # if we are on a document that works invoice and refund and it's a refund, we need to export it as negative
        sign = -sign if self.move_type in ('out_refund', 'in_refund') and\
            self.l10n_latam_document_type_id.code in self._get_l10n_ar_codes_used_for_inv_and_ref() else sign

        res = []
        vat_taxable = self.env['account.move.line']
        # get all invoice lines that are vat taxable
        for line in self.line_ids:
            if any(tax.tax_group_id.l10n_ar_vat_afip_code and tax.tax_group_id.l10n_ar_vat_afip_code not in ['0', '1', '2'] for tax in line.tax_line_id) and line[amount_field]:
                vat_taxable |= line
        for tax_group in vat_taxable.mapped('tax_group_id'):
            base_imp = sum(self.invoice_line_ids.filtered(lambda x: x.tax_ids.filtered(lambda y: y.tax_group_id.l10n_ar_vat_afip_code == tax_group.l10n_ar_vat_afip_code)).mapped(amount_field))
            imp = sum(vat_taxable.filtered(lambda x: x.tax_group_id.l10n_ar_vat_afip_code == tax_group.l10n_ar_vat_afip_code).mapped(amount_field))
            res += [{'Id': tax_group.l10n_ar_vat_afip_code,
                     'BaseImp': sign * base_imp,
                     'Importe': sign * imp}]

        # Report vat 0%
        vat_base_0 = sign * sum(self.invoice_line_ids.filtered(lambda x: x.tax_ids.filtered(lambda y: y.tax_group_id.l10n_ar_vat_afip_code == '3')).mapped(amount_field))
        if vat_base_0:
            res += [{'Id': '3', 'BaseImp': vat_base_0, 'Importe': 0.0}]

        return res if res else []

    def _get_name_invoice_report(self):
        self.ensure_one()
        if self.l10n_latam_use_documents and self.company_id.country_id.code == 'AR':
            return 'l10n_ar.report_invoice_document'
        return super()._get_name_invoice_report()

```

## File: models\account_tax_group.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api


class AccountTaxGroup(models.Model):

    _inherit = 'account.tax.group'

    # values from http://www.afip.gob.ar/fe/documentos/otros_Tributos.xlsx
    l10n_ar_tribute_afip_code = fields.Selection([
        ('01', '01 - National Taxes'),
        ('02', '02 - Provincial Taxes'),
        ('03', '03 - Municipal Taxes'),
        ('04', '04 - Internal Taxes'),
        ('06', '06 - VAT perception'),
        ('07', '07 - IIBB perception'),
        ('08', '08 - Municipal Taxes Perceptions'),
        ('09', '09 - Other Perceptions'),
        ('99', '99 - Others'),
    ], string='Tribute AFIP Code', index=True, readonly=True)
    # values from http://www.afip.gob.ar/fe/documentos/OperacionCondicionIVA.xls
    l10n_ar_vat_afip_code = fields.Selection([
        ('0', 'Not Applicable'),
        ('1', 'Untaxed'),
        ('2', 'Exempt'),
        ('3', '0%'),
        ('4', '10.5%'),
        ('5', '21%'),
        ('6', '27%'),
        ('8', '5%'),
        ('9', '2,5%'),
    ], string='VAT AFIP Code', index=True, readonly=True)

```

## File: models\l10n_ar_afip_responsibility_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class L10nArAfipResponsibilityType(models.Model):

    _name = 'l10n_ar.afip.responsibility.type'
    _description = 'AFIP Responsibility Type'
    _order = 'sequence'

    name = fields.Char(required=True, index=True)
    sequence = fields.Integer()
    code = fields.Char(required=True, index=True)
    active = fields.Boolean(default=True)

    _sql_constraints = [('name', 'unique(name)', 'Name must be unique!'),
                        ('code', 'unique(code)', 'Code must be unique!')]

```

## File: models\l10n_latam_document_type.py

```python
from odoo import models, api, fields, _
from odoo.exceptions import UserError


class L10nLatamDocumentType(models.Model):

    _inherit = 'l10n_latam.document.type'

    l10n_ar_letter = fields.Selection(
        selection='_get_l10n_ar_letters',
        string='Letters',
        help='Letters defined by the AFIP that can be used to identify the'
        ' documents presented to the government and that depends on the'
        ' operation type, the responsibility of both the issuer and the'
        ' receptor of the document')
    purchase_aliquots = fields.Selection(
        [('not_zero', 'Not Zero'), ('zero', 'Zero')], help='Raise an error if a vendor bill is miss encoded. "Not Zero"'
        ' means the VAT taxes are required for the invoices related to this document type, and those with "Zero" means'
        ' that only "VAT Not Applicable" tax is allowed.')

    def _get_l10n_ar_letters(self):
        """ Return the list of values of the selection field. """
        return [
            ('A', 'A'),
            ('B', 'B'),
            ('C', 'C'),
            ('E', 'E'),
            ('M', 'M'),
            ('T', 'T'),
            ('R', 'R'),
            ('X', 'X'),
            ('I', 'I'),  # used for mapping of imports
        ]
    def _filter_taxes_included(self, taxes):
        """ In argentina we include taxes depending on document letter """
        self.ensure_one()
        if self.country_id.code == "AR" and self.l10n_ar_letter in ['B', 'C', 'X', 'R']:
            return taxes.filtered(lambda x: x.tax_group_id.l10n_ar_vat_afip_code)
        return super()._filter_taxes_included(taxes)

    def _format_document_number(self, document_number):
        """ Make validation of Import Dispatch Number
          * making validations on the document_number. If it is wrong it should raise an exception
          * format the document_number against a pattern and return it
        """
        self.ensure_one()
        if self.country_id.code != "AR":
            return super()._format_document_number(document_number)

        if not document_number:
            return False

        msg = "'%s' " + _("is not a valid value for") + " '%s'.<br/>%s"

        if not self.code:
            return document_number

        # Import Dispatch Number Validator
        if self.code in ['66', '67']:
            if len(document_number) != 16:
                raise UserError(msg % (document_number, self.name, _('The number of import Dispatch must be 16 characters')))
            return document_number

        # Invoice Number Validator (For Eg: 123-123)
        failed = False
        args = document_number.split('-')
        if len(args) != 2:
            failed = True
        else:
            pos, number = args
            if len(pos) > 5 or not pos.isdigit():
                failed = True
            elif len(number) > 8 or not number.isdigit():
                failed = True
            document_number = '{:>05s}-{:>08s}'.format(pos, number)
        if failed:
            raise UserError(msg % (document_number, self.name, _(
                'The document number must be entered with a dash (-) and a maximum of 5 characters for the first part'
                'and 8 for the second. The following are examples of valid numbers:\n* 1-1\n* 0001-00000001'
                '\n* 00001-00000001')))

        return document_number

```

## File: models\l10n_latam_identification_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class L10nLatamIdentificationType(models.Model):

    _inherit = "l10n_latam.identification.type"

    l10n_ar_afip_code = fields.Char("AFIP Code")

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api, _
from odoo.exceptions import ValidationError

class ResCompany(models.Model):

    _inherit = "res.company"

    l10n_ar_gross_income_number = fields.Char(
        related='partner_id.l10n_ar_gross_income_number', string='Gross Income Number', readonly=False,
        help="This field is required in order to print the invoice report properly")
    l10n_ar_gross_income_type = fields.Selection(
        related='partner_id.l10n_ar_gross_income_type', string='Gross Income', readonly=False,
        help="This field is required in order to print the invoice report properly")
    l10n_ar_afip_responsibility_type_id = fields.Many2one(
        domain="[('code', 'in', [1, 4, 6])]", related='partner_id.l10n_ar_afip_responsibility_type_id', readonly=False)
    l10n_ar_company_requires_vat = fields.Boolean(compute='_compute_l10n_ar_company_requires_vat', string='Company Requires Vat?')
    l10n_ar_afip_start_date = fields.Date('Activities Start')

    @api.onchange('country_id')
    def onchange_country(self):
        """ Argentinian companies use round_globally as tax_calculation_rounding_method """
        for rec in self.filtered(lambda x: x.country_id.code == "AR"):
            rec.tax_calculation_rounding_method = 'round_globally'

    @api.depends('l10n_ar_afip_responsibility_type_id')
    def _compute_l10n_ar_company_requires_vat(self):
        recs_requires_vat = self.filtered(lambda x: x.l10n_ar_afip_responsibility_type_id.code == '1')
        recs_requires_vat.l10n_ar_company_requires_vat = True
        remaining = self - recs_requires_vat
        remaining.l10n_ar_company_requires_vat = False

    def _localization_use_documents(self):
        """ Argentinian localization use documents """
        self.ensure_one()
        return True if self.country_id.code == "AR" else super()._localization_use_documents()

    @api.constrains('l10n_ar_afip_responsibility_type_id')
    def _check_accounting_info(self):
        """ Do not let to change the AFIP Responsibility of the company if there is already installed a chart of
        account and if there has accounting entries """
        if self.env['account.chart.template'].existing_accounting(self):
            raise ValidationError(_(
                'Could not change the AFIP Responsibility of this company because there are already accounting entries.'))

```

## File: models\res_country.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCountry(models.Model):

    _inherit = 'res.country'

    l10n_ar_afip_code = fields.Char('AFIP Code', size=3, help='This code will be used on electronic invoice')
    l10n_ar_natural_vat = fields.Char(
        'Natural Person VAT', size=11, help="Generic VAT number defined by AFIP in order to recognize partners from"
        " this country that are natural persons")
    l10n_ar_legal_entity_vat = fields.Char(
        'Legal Entity VAT', size=11, help="Generic VAT number defined by AFIP in order to recognize partners from this"
        " country that are legal entity")
    l10n_ar_other_vat = fields.Char(
        'Other VAT', size=11, help="Generic VAT number defined by AFIP in order to recognize partners from this"
        " country that are not natural persons or legal entities")

```

## File: models\res_currency.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResCurrency(models.Model):

    _inherit = "res.currency"

    l10n_ar_afip_code = fields.Char('AFIP Code', size=4, help='This code will be used on electronic invoice')

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api, _
from odoo.exceptions import UserError, ValidationError
import stdnum.ar
import re
import logging

_logger = logging.getLogger(__name__)


class ResPartner(models.Model):

    _inherit = 'res.partner'

    l10n_ar_vat = fields.Char(
        compute='_compute_l10n_ar_vat', string="VAT", help='Computed field that returns VAT or nothing if this one'
        ' is not set for the partner')
    l10n_ar_formatted_vat = fields.Char(
        compute='_compute_l10n_ar_formatted_vat', string="Formatted VAT", help='Computed field that will convert the'
        ' given VAT number to the format {person_category:2}-{number:10}-{validation_number:1}')

    l10n_ar_gross_income_number = fields.Char('Gross Income Number')
    l10n_ar_gross_income_type = fields.Selection(
        [('multilateral', 'Multilateral'), ('local', 'Local'), ('exempt', 'Exempt')],
        'Gross Income Type', help='Type of gross income: exempt, local, multilateral')
    l10n_ar_afip_responsibility_type_id = fields.Many2one(
        'l10n_ar.afip.responsibility.type', string='AFIP Responsibility Type', index=True, help='Defined by AFIP to'
        ' identify the type of responsibilities that a person or a legal entity could have and that impacts in the'
        ' type of operations and requirements they need.')
    l10n_ar_special_purchase_document_type_ids = fields.Many2many(
        'l10n_latam.document.type', 'res_partner_document_type_rel', 'partner_id', 'document_type_id',
        string='Other Purchase Documents', help='Set here if this partner can issue other documents further than'
        ' invoices, credit notes and debit notes')

    @api.depends('l10n_ar_vat')
    def _compute_l10n_ar_formatted_vat(self):
        """ This will add some dash to the CUIT number (VAT AR) in order to show in his natural format:
        {person_category}-{number}-{validation_number} """
        recs_ar_vat = self.filtered('l10n_ar_vat')
        for rec in recs_ar_vat:
            try:
                rec.l10n_ar_formatted_vat = stdnum.ar.cuit.format(rec.l10n_ar_vat)
            except Exception as error:
                rec.l10n_ar_formatted_vat = rec.l10n_ar_vat
                _logger.runbot("Argentinian VAT was not formatted: %s", repr(error))
        remaining = self - recs_ar_vat
        remaining.l10n_ar_formatted_vat = False

    @api.depends('vat', 'l10n_latam_identification_type_id')
    def _compute_l10n_ar_vat(self):
        """ We add this computed field that returns cuit (VAT AR) or nothing if this one is not set for the partner.
        This Validation can be also done by calling ensure_vat() method that returns the cuit (VAT AR) or error if this
        one is not found """
        recs_ar_vat = self.filtered(lambda x: x.l10n_latam_identification_type_id.l10n_ar_afip_code == '80' and x.vat)
        for rec in recs_ar_vat:
            rec.l10n_ar_vat = stdnum.ar.cuit.compact(rec.vat)
        remaining = self - recs_ar_vat
        remaining.l10n_ar_vat = False

    @api.constrains('vat', 'l10n_latam_identification_type_id')
    def check_vat(self):
        """ Since we validate more documents than the vat for Argentinian partners (CUIT - VAT AR, CUIL, DNI) we
        extend this method in order to process it. """
        # NOTE by the moment we include the CUIT (VAT AR) validation also here because we extend the messages
        # errors to be more friendly to the user. In a future when Odoo improve the base_vat message errors
        # we can change this method and use the base_vat.check_vat_ar method.s
        l10n_ar_partners = self.filtered(lambda x: x.l10n_latam_identification_type_id.l10n_ar_afip_code)
        l10n_ar_partners.l10n_ar_identification_validation()
        return super(ResPartner, self - l10n_ar_partners).check_vat()

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['l10n_ar_afip_responsibility_type_id']

    def ensure_vat(self):
        """ This method is a helper that returns the VAT number is this one is defined if not raise an UserError.

        VAT is not mandatory field but for some Argentinian operations the VAT is required, for eg  validate an
        electronic invoice, build a report, etc.

        This method can be used to validate is the VAT is proper defined in the partner """
        self.ensure_one()
        if not self.l10n_ar_vat:
            raise UserError(_('No VAT configured for partner [%i] %s') % (self.id, self.name))
        return self.l10n_ar_vat

    def _get_validation_module(self):
        self.ensure_one()
        if self.l10n_latam_identification_type_id.l10n_ar_afip_code in ['80', '86']:
            return stdnum.ar.cuit
        elif self.l10n_latam_identification_type_id.l10n_ar_afip_code == '96':
            return stdnum.ar.dni

    def l10n_ar_identification_validation(self):
        for rec in self.filtered('vat'):
            try:
                module = rec._get_validation_module()
            except Exception as error:
                module = False
                _logger.runbot("Argentinian document was not validated: %s", repr(error))

            if not module:
                continue
            try:
                module.validate(rec.vat)
            except module.InvalidChecksum:
                raise ValidationError(_('The validation digit is not valid for "%s"', rec.l10n_latam_identification_type_id.name))
            except module.InvalidLength:
                raise ValidationError(_('Invalid length for "%s"', rec.l10n_latam_identification_type_id.name))
            except module.InvalidFormat:
                raise ValidationError(_('Only numbers allowed for "%s"', rec.l10n_latam_identification_type_id.name))
            except Exception as error:
                raise ValidationError(repr(error))

    def _get_id_number_sanitize(self):
        """ Sanitize the identification number. Return the digits/integer value of the identification number
        If not vat number defined return 0 """
        self.ensure_one()
        if not self.vat:
            return 0
        if self.l10n_latam_identification_type_id.l10n_ar_afip_code in ['80', '86']:
            # Compact is the number clean up, remove all separators leave only digits
            res = int(stdnum.ar.cuit.compact(self.vat))
        else:
            id_number = re.sub('[^0-9]', '', self.vat)
            res = int(id_number)
        return res

```

## File: models\res_partner_bank.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, api, _
from odoo.exceptions import ValidationError
import logging
_logger = logging.getLogger(__name__)


try:
    from stdnum.ar.cbu import validate as validate_cbu
except ImportError:
    import stdnum
    _logger.warning("stdnum.ar.cbu is avalaible from stdnum >= 1.6. The one installed is %s" % stdnum.__version__)

    def validate_cbu(number):
        def _check_digit(number):
            """Calculate the check digit."""
            weights = (3, 1, 7, 9)
            check = sum(int(n) * weights[i % 4] for i, n in enumerate(reversed(number)))
            return str((10 - check) % 10)
        number = stdnum.util.clean(number, ' -').strip()
        if len(number) != 22:
            raise ValidationError('Invalid Length')
        if not number.isdigit():
            raise ValidationError('Invalid Format')
        if _check_digit(number[:7]) != number[7]:
            raise ValidationError('Invalid Checksum')
        if _check_digit(number[8:-1]) != number[-1]:
            raise ValidationError('Invalid Checksum')
        return number


class ResPartnerBank(models.Model):

    _inherit = 'res.partner.bank'

    @api.model
    def _get_supported_account_types(self):
        """ Add new account type named cbu used in Argentina """
        res = super()._get_supported_account_types()
        res.append(('cbu', _('CBU')))
        return res

    @api.model
    def retrieve_acc_type(self, acc_number):
        try:
            validate_cbu(acc_number)
        except Exception:
            return super().retrieve_acc_type(acc_number)
        return 'cbu'

```

## File: models\uom_uom.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class Uom(models.Model):

    _inherit = 'uom.uom'

    l10n_ar_afip_code = fields.Char('AFIP Code', help='This code will be used on electronic invoice')

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import l10n_latam_identification_type
from . import l10n_ar_afip_responsibility_type
from . import account_journal
from . import account_tax_group
from . import account_fiscal_position
from . import account_fiscal_position_template
from . import l10n_latam_document_type
from . import res_partner
from . import res_country
from . import res_currency
from . import res_company
from . import res_partner_bank
from . import uom_uom
from . import account_chart_template
from . import account_move

```

## File: report\invoice_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class AccountInvoiceReport(models.Model):

    _inherit = 'account.invoice.report'

    l10n_ar_state_id = fields.Many2one('res.country.state', 'State', readonly=True)
    date = fields.Date(readonly=True, string="Accounting Date")

    _depends = {
        'account.move': ['partner_id', 'date'],
        'res.partner': ['state_id'],
    }

    def _select(self):
        return super()._select() + ", contact_partner.state_id as l10n_ar_state_id, move.date"

    def _from(self):
        return super()._from() + " LEFT JOIN res_partner contact_partner ON contact_partner.id = move.partner_id"

```

## File: report\invoice_report_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record model="ir.ui.view" id="view_account_invoice_report_search_inherit">
        <field name="name">account.invoice.report.search</field>
        <field name="model">account.invoice.report</field>
        <field name="inherit_id" ref="account.view_account_invoice_report_search" />
        <field name="arch" type="xml">
            <search>
                <field name="l10n_ar_state_id"/>
                <filter name="with_document" string="With Document" domain="[('l10n_latam_document_type_id', '!=', False)]"/>
                <filter name="filter_accounting_date_this_year" invisible="1" string="Accounting Date: This Year" domain="[('date', '&lt;', (context_today() + relativedelta(years=1, month=1, day=1)).strftime('%Y-%m-%d')), ('date', '&gt;=', (context_today() + relativedelta(month=1, day=1)).strftime('%Y-%m-%d'))]"/>
            </search>
            <filter name="user" position="after">
                <filter string="State" name="groupby_l10n_ar_state_id" context="{'group_by': 'l10n_ar_state_id'}"/>
                <filter string="Account" name="groupby_account_id" context="{'group_by':'account_id'}" groups="account.group_account_readonly" />
            </filter>
        </field>
    </record>

    <record id="action_iibb_sales_by_state_and_account_pivot" model="ir.actions.act_window">
        <field name="name">IIBB - Sales by jurisdiction</field>
        <field name="res_model">account.invoice.report</field>
        <field name="view_mode">pivot</field>
        <field name="context">{'search_default_current': 1, 'search_default_customer': 1, 'search_default_with_document': 1, 'search_default_company': 1, 'search_default_groupby_l10n_ar_state_id': 2, 'search_default_groupby_account_id': 3, 'search_default_filter_accounting_date_this_year': 1}</field>
    </record>

    <menuitem
        id="menu_iibb_sales_by_state_and_account"
        action="action_iibb_sales_by_state_and_account_pivot"
        parent="l10n_ar.account_reports_ar_statements_menu"
        sequence="30"/>

    <record id="action_iibb_purchases_by_state_and_account_pivot" model="ir.actions.act_window">
        <field name="name">IIBB - Purchases by jurisdiction</field>
        <field name="res_model">account.invoice.report</field>
        <field name="view_mode">pivot</field>
        <field name="context">{'search_default_current': 1, 'search_default_supplier': 1, 'search_default_with_document': 1, 'search_default_company': 1, 'search_default_groupby_l10n_ar_state_id': 2, 'search_default_groupby_account_id': 3, 'search_default_filter_accounting_date_this_year': 1}</field>
    </record>

    <menuitem
        id="menu_iibb_purchases_by_state_and_account"
        action="action_iibb_purchases_by_state_and_account_pivot"
        parent="l10n_ar.account_reports_ar_statements_menu"
        sequence="40"/>

</odoo>

```

## File: report\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import invoice_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_ar_afip_responsibility_type_all,l10n_ar.afip.responsibility.type.all,model_l10n_ar_afip_responsibility_type,,1,0,0,0

```

## File: static\src\js\tours\account.js

```javascript
odoo.define('l10n_ar.account_tour', function(require) {
"use strict";

    let tour = require('web_tour.tour');
    let account_tour = tour.tours.account_tour;
    // Remove the step suggesting to change the name as it is done another way (document number)
    account_tour.steps = _.filter(account_tour.steps, step => step.trigger != "input[name=name]");

    // Configure the AFIP Responsibility
    let partner_step_idx = _.findIndex(account_tour.steps, step => step.trigger == 'div[name=partner_id] input');
    account_tour.steps.splice(partner_step_idx + 2, 0, {
        trigger: "div[name=l10n_ar_afip_responsibility_type_id] input",
        extra_trigger: "[name=move_type][raw-value=out_invoice]",
        position: "bottom",
        content: "Set the AFIP Responsability",
        run: "text IVA",
    })
    account_tour.steps.splice(partner_step_idx + 3, 0, {
        trigger: ".ui-menu-item > a:contains('IVA').ui-state-active",
        auto: true,
        in_modal: false,
    })

})

```

## File: views\account_fiscal_position_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_position_form" model="ir.ui.view">
        <field name="name">account.fiscal.position.form</field>
        <field name="model">account.fiscal.position</field>
        <field name="inherit_id" ref="account.view_account_position_form"/>
        <field name="arch" type="xml">
            <field name="auto_apply" position="after">
                <field name="l10n_ar_afip_responsibility_type_ids" options="{'no_open': True, 'no_create': True}" attrs="{'invisible': [('auto_apply', '!=', True)]}" groups="base.group_no_one" widget="many2many_tags"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\account_journal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_journal_form" model="ir.ui.view">
        <field name="model">account.journal</field>
        <field name="name">account.journal.form</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_account_journal_form"/>
        <field name="arch" type="xml">
            <field name="l10n_latam_use_documents" position="after">
                <field name="company_partner" invisible="1"/>
                <field name="l10n_ar_afip_pos_system" attrs="{'invisible':['|', '|', ('country_code', '!=', 'AR'), ('l10n_latam_use_documents', '=', False), ('type', '!=', 'sale')], 'required':[('country_code', '=', 'AR'), ('l10n_latam_use_documents', '=', True), ('type', '=', 'sale')]}"/>
                <field name="l10n_ar_afip_pos_number" attrs="{'invisible':['|', '|', ('country_code', '!=', 'AR'), ('l10n_latam_use_documents', '=', False), ('type', '!=', 'sale')], 'required':[('country_code', '=', 'AR'), ('l10n_latam_use_documents', '=', True), ('type', '=', 'sale')]}"/>
                <field name="l10n_ar_afip_pos_partner_id" attrs="{'invisible':['|', '|', ('country_code', '!=', 'AR'), ('l10n_latam_use_documents', '=', False), ('type', '!=', 'sale')], 'required':[('country_code', '=', 'AR'), ('l10n_latam_use_documents', '=', True), ('type', '=', 'sale')]}"/>
                <field name="l10n_ar_share_sequences" attrs="{'invisible':[('l10n_ar_afip_pos_system', '!=', 'II_IM')]}"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\account_move_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_move_filter" model="ir.ui.view">
        <field name="name">account.move.filter</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_account_move_filter"/>
        <field name="arch" type="xml">
            <field name="partner_id" position="after">
                <field name="l10n_ar_afip_responsibility_type_id"/>
            </field>
            <group>
                <filter string="AFIP Responsibility Type" domain="[]" name="l10n_ar_afip_responsibility_type_id_filter" context="{'group_by':'l10n_ar_afip_responsibility_type_id'}"/>
            </group>
        </field>
    </record>


    <record id="view_move_form" model="ir.ui.view">
        <field name="name">account.move.form</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
			<group id="other_tab_group" position="inside">
                <group name="afip_group" string="AFIP">
                    <field name='l10n_ar_afip_concept' attrs="{'invisible': ['|', ('country_code', '!=', 'AR'), ('l10n_latam_use_documents', '=', False)]}"/>
                    <label for="l10n_ar_afip_service_start" attrs="{'invisible':[('l10n_ar_afip_concept','not in',['2', '3', '4'])]}" string="Service Date"/>
                    <div attrs="{'invisible':[('l10n_ar_afip_concept','not in',['2', '3', '4'])]}">
                        <field name="l10n_ar_afip_service_start" class="oe_inline"/> to <field name="l10n_ar_afip_service_end" class="oe_inline"/>
                    </div>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: views\afip_menuitem.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <menuitem id="menu_afip_config" name="AFIP" parent="account.menu_finance_configuration" sequence="25"/>

</odoo>

```

## File: views\l10n_ar.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="account assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/l10n_ar/static/src/js/tours/account.js"/>
        </xpath>
    </template>
</odoo>

```

## File: views\l10n_ar_afip_responsibility_type_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_afip_responsibility_type_form" model="ir.ui.view">
        <field name="name">afip.responsibility.type.form</field>
        <field name="model">l10n_ar.afip.responsibility.type</field>
        <field name="arch" type="xml">
            <form string="AFIP Responsibility Type">
                <group>
                    <field name="name"/>
                    <field name='code'/>
                    <field name='active'/>
                </group>
            </form>
        </field>
    </record>

    <record id="view_afip_responsibility_type_tree" model="ir.ui.view">
        <field name="name">afip.responsibility.type.tree</field>
        <field name="model">l10n_ar.afip.responsibility.type</field>
        <field name="arch" type="xml">
            <tree string="AFIP Responsibility Type" decoration-muted="(not active)">
                <field name="name"/>
                <field name="code"/>
                <field name='active'/>
            </tree>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_afip_responsibility_type">
        <field name="name">AFIP Responsibility Types</field>
        <field name="res_model">l10n_ar.afip.responsibility.type</field>
    </record>

    <menuitem name="Responsibility Types" action="action_afip_responsibility_type" id="menu_afip_responsibility_type" sequence="10" parent="menu_afip_config"/>

</odoo>

```

## File: views\l10n_latam_document_type_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_document_type_form" model="ir.ui.view">
        <field name="name">l10n_latam.document.type.form</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_document_type_form"/>
        <field name="arch" type="xml">
            <field name='doc_code_prefix' position="after">
                <field name='l10n_ar_letter'/>
                <field name='purchase_aliquots'/>
            </field>
        </field>
    </record>

    <record id="view_document_type_tree" model="ir.ui.view">
        <field name="name">l10n_latam.document.type.tree</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_document_type_tree"/>
        <field name="arch" type="xml">
            <field name='doc_code_prefix' position="after">
                <field name='l10n_ar_letter'/>
            </field>
        </field>
    </record>

    <record id="view_document_type_filter" model="ir.ui.view">
        <field name="name">l10n_latam.document.type.filter</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_document_type_filter"/>
        <field name="arch" type="xml">
            <field name='code' position="after">
                <field name='l10n_ar_letter'/>
                <filter string="Argentinian Documents" name="localization" domain="[('country_id.code', '=', 'AR')]"/>
            </field>
            <group>
                <filter string="Document Letter" name="l10n_ar_letter" context="{'group_by':'l10n_ar_letter'}"/>
            </group>
        </field>
    </record>


    <record model="ir.actions.act_window" id="action_document_type_argentina">
        <field name="name">Document Types</field>
        <field name="res_model">l10n_latam.document.type</field>
        <field name="context">{'search_default_localization': 1}</field>
    </record>

    <menuitem action="action_document_type_argentina" id="menu_document_type_argentina" sequence="5" parent="menu_afip_config"/>

</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- this header can be used on any Argentinean report, to be useful some variables should be passed -->
    <template id="custom_header">
        <div>
            <div class="row">
                <div name="left-upper-side" class="col-5" t-if="not pre_printed_report">
                    <img t-if="o.company_id.logo" t-att-src="image_data_uri(o.company_id.logo)" style="max-height: 45px;" alt="Logo"/>
                </div>
                <div name="center-upper" class="col-2 text-center" t-att-style="'color: %s;' % o.company_id.primary_color">
                    <span style="display: inline-block; text-align: center; line-height: 8px;">
                        <h1 style="line-height: 25px;">
                            <strong><span t-esc="not pre_printed_report and document_letter or '&#160;'"/></strong>
                        </h1>
                        <span style="font-size: x-small;" t-esc="not pre_printed_report and document_legend or '&#160;'"/>
                    </span>
                </div>
                <div name="right-upper-side" class="col-5 text-right" style="padding-left: 0px;" t-if="not pre_printed_report">

                    <!-- (6) Titulo de Documento -->
                    <h4 t-att-style="'color: %s;' % o.company_id.primary_color"><strong>
                        <span t-esc="report_name"/>
                    </strong></h4>

                </div>
            </div>
            <div class="row">
                <div class="col-6" style="padding-right: 0px;">
                    <t t-if="not pre_printed_report">
                        <!-- (1) Nombre de Fantasia -->
                        <!-- (2) Apellido y Nombre o Razon Social -->
                        <span t-field="o.company_id.partner_id.name"/>

                        <!-- (3) Domicilio Comercial (Domicilio Fiscal is the same) -->
                        <br/>
                        <div></div>
                        <!-- we dont use the address widget as it adds a new line on the phone and we want to reduce at maximum lines qty -->
                        <t t-esc="' - '.join([item for item in [
                            ', '.join([item for item in [header_address.street, header_address.street2] if item]),
                            header_address.city,
                            header_address.state_id and header_address.state_id.name,
                            header_address.zip,
                            header_address.country_id and header_address.country_id.name] if item])"/><span t-if="header_address.phone"> - </span><span t-if="header_address.phone" style="white-space: nowrap;" t-esc="'Tel: ' + header_address.phone"/>
                        <br/>
                        <span t-att-style="'color: %s;' % o.company_id.primary_color" t-esc="' - '.join([item for item in [(header_address.website or '').replace('https://', '').replace('http://', ''), header_address.email] if item])"/>
                    </t>
                </div>
                <div class="col-6 text-right" style="padding-left: 0px;">

                    <t t-if="not pre_printed_report">
                        <!-- (7) Numero punto venta - (8) numero de documento -->
                        <span t-att-style="'color: %s;' % o.company_id.secondary_color">Nro: </span><span t-esc="report_number"/>
                    </t>
                    <br/>

                    <!-- (9) Fecha -->
                    <span t-att-style="'color: %s;' % o.company_id.secondary_color">Date: </span><span t-esc="report_date" t-options='{"widget": "date"}'/>

                    <t t-if="not pre_printed_report">
                        <!-- (5) Condicion de IVA / Responsabilidad -->
                        <!-- (10) CUIT -->
                        <br/>
                        <span t-field="o.company_id.l10n_ar_afip_responsibility_type_id"/><span t-att-style="'color: %s;' % o.company_id.secondary_color"> - CUIT: </span><span t-field="o.company_id.partner_id.l10n_ar_formatted_vat"/>

                        <!-- (11) IIBB: -->
                        <!-- (12) Inicio de actividades -->
                        <br/><span t-att-style="'color: %s;' % o.company_id.secondary_color">IIBB: </span><span t-esc="o.company_id.l10n_ar_gross_income_type == 'exempt' and 'Exento' or o.company_id.l10n_ar_gross_income_number"/><span t-att-style="'color: %s;' % o.company_id.secondary_color"> - Activities Start: </span><span t-field="o.company_id.l10n_ar_afip_start_date"/>
                    </t>

                </div>
            </div>
        </div>
    </template>

    <template id="report_invoice_document" inherit_id="account.report_invoice_document" primary="True">
      <!-- custom header and footer -->
        <t t-set="o" position="after">
            <t t-set="custom_header" t-value="'l10n_ar.custom_header'"/>
            <t t-set="report_date" t-value="o.invoice_date"/>
            <t t-set="report_number" t-value="o.l10n_latam_document_number"/>
            <t t-set="pre_printed_report" t-value="report_type == 'pdf' and o.journal_id.l10n_ar_afip_pos_system == 'II_IM'"/>
            <t t-set="document_letter" t-value="o.l10n_latam_document_type_id.l10n_ar_letter"/>
            <t t-set="document_legend" t-value="o.l10n_latam_document_type_id.code and 'Cod. %02d' % int(o.l10n_latam_document_type_id.code) or ''"/>
            <t t-set="report_name" t-value="o.l10n_latam_document_type_id.report_name"/>
            <t t-set="header_address" t-value="o.journal_id.l10n_ar_afip_pos_partner_id"/>

            <t t-set="custom_footer">
                <div class="row">
                    <div name="footer_left_column" class="col-8 text-left">
                    </div>
                    <div name="footer_right_column" class="col-4 text-right">
                        <div name="pager" t-if="report_type == 'pdf'">
                            Page: <span class="page"/> / <span class="topage"/>
                        </div>
                    </div>
                </div>
            </t>
            <t t-set="fiscal_bond" t-value="o.journal_id.l10n_ar_afip_pos_system in ['BFERCEL', 'BFEWS']"/>
        </t>

        <!-- remove default partner address -->
        <t t-set="address" position="replace"/>

        <!-- remove default document title -->
        <h2 position="replace"/>

        <!-- NCM column for fiscal bond -->
        <th name="th_description" position="after">
            <th t-if="fiscal_bond" name="th_ncm_code" class="text-left"><span>NCM</span></th>
        </th>
        <td name="account_invoice_line_name" position="after">
            <td t-if="fiscal_bond" name="ncm_code"><span t-field="line.product_id.l10n_ar_ncm_code"/></td>
        </td>

        <!-- use latam prices (to include/exclude VAT) -->
        <xpath expr="//span[@t-field='line.price_unit']" position="attributes">
            <attribute name="t-field">line.l10n_latam_price_unit</attribute>
        </xpath>
        <xpath expr="//span[@id='line_tax_ids']" position="attributes">
            <attribute name="t-esc">', '.join(map(lambda x: (x.description or x.name), line.l10n_latam_tax_ids))</attribute>
        </xpath>
        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" position="attributes">
            <attribute name="t-value">current_subtotal + line.l10n_latam_price_subtotal</attribute>
        </t>
        <!-- if b2c we still wants the latam subtotal -->
        <t t-set="current_subtotal" t-value="current_subtotal + line.price_total" position="attributes">
            <attribute name="t-value">current_subtotal + line.l10n_latam_price_subtotal</attribute>
        </t>
        <!-- label amount for subtotal column on b2b and b2c -->
        <xpath expr="//th[@name='th_subtotal']/span[@groups='account.group_show_line_subtotals_tax_included']" position="replace">
            <span groups="account.group_show_line_subtotals_tax_included">Amount</span>
        </xpath>
        <span t-field="line.price_subtotal" position="attributes">
            <attribute name="t-field">line.l10n_latam_price_subtotal</attribute>
        </span>
        <!-- if b2c we still wants the latam subtotal -->
        <span t-field="line.price_total" position="attributes">
            <attribute name="t-field">line.l10n_latam_price_subtotal</attribute>
        </span>
        <span t-field="o.amount_untaxed" position="attributes">
            <attribute name="t-field">o.l10n_latam_amount_untaxed</attribute>
        </span>

        <!-- use column vat instead of taxes and only if vat discriminated -->
        <xpath expr="//th[@name='th_taxes']/span" position="replace">
            <span>% VAT</span>
        </xpath>

        <xpath expr="//th[@name='th_taxes']" position="attributes">
            <attribute name="t-if">o.amount_by_group</attribute>
        </xpath>

        <!-- use column vat instead of taxes and only list vat taxes-->
        <xpath expr="//span[@id='line_tax_ids']/.." position="attributes">
            <attribute name="t-if">o.amount_by_group</attribute>
        </xpath>
        <span id="line_tax_ids" position="attributes">
            <attribute name="t-esc">', '.join(map(lambda x: (x.description or x.name), line.l10n_latam_tax_ids.filtered(lambda x: x.tax_group_id.l10n_ar_vat_afip_code)))</attribute>
        </span>

        <!-- remove payment reference that is not used in Argentina -->
        <xpath expr="//span[@t-field='o.payment_reference']/../.." position="replace"/>

        <!-- replace information section and usage argentinean style -->
        <div id="informations" position="replace">
            <div id="informations" class="row mt8 mb8">
                <div class="col-6">

                    <!-- IDENTIFICACION (ADQUIRIENTE-LOCATARIO-PRESTARIO) -->

                    <!-- (14) Apellido uy Nombre: Denominicacion o Razon Soclial -->
                    <strong>Customer: </strong><span t-field="o.partner_id.commercial_partner_id.name"/>

                    <!-- (15) Domicilio Comercial -->
                    <br/>
                    <span t-field="o.partner_id" t-options="{'widget': 'contact', 'fields': ['address'], 'no_marker': true, 'no_tag_br': True}"/>

                    <!-- (16) Responsabilidad AFIP -->
                    <strong>VAT Cond: </strong><span t-field="o.partner_id.l10n_ar_afip_responsibility_type_id"/>

                    <!-- (17) CUIT -->
                    <t t-if="o.partner_id.vat and o.partner_id.l10n_latam_identification_type_id and o.partner_id.l10n_latam_identification_type_id.l10n_ar_afip_code != '99'">
                        <br/><strong><t t-esc="o.partner_id.l10n_latam_identification_type_id.name or o.company_id.country_id.vat_label" id="inv_tax_id_label"/>:</strong> <span t-esc="o.partner_id.l10n_ar_formatted_vat if o.partner_id.l10n_ar_vat else o.partner_id.vat"/>
                    </t>

                </div>
                <div class="col-6">

                    <t t-if="o.invoice_date_due">
                        <strong>Due Date: </strong>
                        <span t-field="o.invoice_date_due"/>
                    </t>

                    <t t-if="o.invoice_payment_term_id" name="payment_term">
                        <br/><strong>Payment Terms: </strong><span t-field="o.invoice_payment_term_id.name"/>
                    </t>

                    <t t-if="o.invoice_origin">
                        <br/><strong>Source:</strong>
                        <span t-field="o.invoice_origin"/>
                    </t>

                    <t t-if="o.ref">
                        <br/><strong>Reference:</strong>
                        <span t-field="o.ref"/>
                    </t>

                    <!-- (18) REMITOS -->
                    <!-- We do not have remitos implement yet. print here the remito number when we have it -->

                    <t t-if="o.invoice_incoterm_id">
                        <br/><strong>Incoterm:</strong><span t-field="o.invoice_incoterm_id.name"/>
                    </t>

                </div>

            </div>
        </div>

        <!--  we remove the ml auto and also give more space to avoid multiple lines on tax detail -->
        <xpath expr="//div[@id='total']/div" position="attributes">
            <attribute name="t-attf-class">#{'col-6' if report_type != 'html' else 'col-sm-7 col-md-6'}</attribute>
        </xpath>

        <xpath expr="//div[@id='total']/div" position="before">
            <div t-attf-class="#{'col-6' if report_type != 'html' else 'col-sm-7 col-md-6'}">

                <t t-if="o.l10n_ar_afip_concept in ['2', '3', '4'] and o.l10n_ar_afip_service_start and o.l10n_ar_afip_service_end">
                    <strong>Invoiced period: </strong><span t-field="o.l10n_ar_afip_service_start"/> to <span t-field="o.l10n_ar_afip_service_end"/>
                </t>
                <t t-if="o.currency_id != o.company_id.currency_id">
                    <br/><strong>Currency: </strong><span t-esc="'%s - %s' % (o.currency_id.name, o.currency_id.currency_unit_label)"/>
                    <br/><strong>Exchange rate: </strong> <span t-field="o.l10n_ar_currency_rate"/>
                </t>
                <!-- Show CBU for FACTURA DE CREDITO ELECTRONICA MiPyMEs and NOTA DE DEBITO ELECTRONICA MiPyMEs -->
                <t t-if="o.l10n_latam_document_type_id.code in ['201', '206', '211', '202', '207', '212'] and o.partner_bank_id">
                    <br/><strong>CBU for payment: </strong><span t-esc="o.partner_bank_id.acc_number or '' if o.partner_bank_id.acc_type == 'cbu' else ''"/>
                </t>

            </div>
        </xpath>

        <!-- Show total amount in letters for MiPyMEs document types according to the law
         http://biblioteca.afip.gob.ar/dcp/LEY_C_027440_2018_05_09 article 5.f -->
        <xpath expr="//div[@id='total']/div/table" position="after">
            <t t-if="o.l10n_latam_document_type_id.code in ['201', '202', '203', '206', '207', '208', '211', '212', '213']">
                <strong>Son: </strong><span t-esc="o.currency_id.with_context(lang='es_AR').amount_to_text(o.amount_total)"/>
            </t>
        </xpath>

        <!-- RG 5003: Add legend for 'A' documents that have a Monotribuista receptor -->
        <p name="comment" position="after">
            <p t-if="o.partner_id.l10n_ar_afip_responsibility_type_id.code in ['6', '13'] and o.l10n_latam_document_type_id.l10n_ar_letter == 'A'" >
                El crédito fiscal discriminado en el presente comprobante, sólo podrá ser computado a efectos del Régimen de Sostenimiento e Inclusión Fiscal para Pequeños Contribuyentes de la Ley Nº 27.618
            </p>
        </p>

    </template>

    <!-- FIXME: Temp fix to allow fetching invoice_documemt in Studio Reports with localisation -->
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-if="o._get_name_invoice_report() == 'l10n_ar.report_invoice_document'"
                t-call="l10n_ar.report_invoice_document" t-lang="lang"/>
        </xpath>
    </template>

    <template id="report_invoice_with_payments" inherit_id="account.report_invoice_with_payments">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-if="o._get_name_invoice_report() == 'l10n_ar.report_invoice_document'"
                t-call="l10n_ar.report_invoice_document" t-lang="lang"/>
        </xpath>
    </template>

</odoo>

```

## File: views\res_company_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="view_company_form">
        <field name="name">res.company.form.inherit</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="model">res.company</field>
        <field name="arch" type="xml">
            <field name="vat" position="after">
                <field name="l10n_ar_afip_responsibility_type_id" options="{'no_open': True, 'no_create': True}" attrs="{'invisible': [('country_code', '!=', 'AR')]}"/>
                <label for="l10n_ar_gross_income_number" string="Gross Income" attrs="{'invisible': [('country_code', '!=', 'AR')]}"/>
                <div attrs="{'invisible': [('country_code', '!=', 'AR')]}" name="gross_income">
                    <field name="l10n_ar_gross_income_type" class="oe_inline"/>
                    <field name="l10n_ar_gross_income_number" placeholder="Number..." class="oe_inline" attrs="{'invisible': [('l10n_ar_gross_income_type', 'in', [False, 'exempt'])], 'required': [('l10n_ar_gross_income_type', 'not in', [False, 'exempt'])]}"/>
                </div>
                <field name="l10n_ar_afip_start_date" attrs="{'invisible': [('country_code', '!=', 'AR')]}"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\res_country_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_res_country_form" model="ir.ui.view">
        <field name="name">res.country.form</field>
        <field name="model">res.country</field>
        <field name="inherit_id" ref="base.view_country_form"/>
        <field name="arch" type="xml">
            <field name="code" position="after">
                <field name="l10n_ar_afip_code" groups="base.group_no_one"/>
                <field name="l10n_ar_natural_vat"/>
                <field name="l10n_ar_legal_entity_vat"/>
                <field name="l10n_ar_other_vat"/>
            </field>
        </field>
    </record>

    <record id="view_res_country_tree" model="ir.ui.view">
        <field name="name">res.country.tree</field>
        <field name="model">res.country</field>
        <field name="inherit_id" ref="base.view_country_tree"/>
        <field name="arch" type="xml">
            <field name="code" position="after">
                <field name="l10n_ar_afip_code" groups="base.group_no_one"/>
                <field name="l10n_ar_natural_vat"/>
                <field name="l10n_ar_legal_entity_vat"/>
                <field name="l10n_ar_other_vat"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\res_currency_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="view_currency_form">
        <field name="name">res.currency.form</field>
        <field name="inherit_id" ref="base.view_currency_form"/>
        <field name="model">res.currency</field>
        <field name="arch" type="xml">
            <field name="name" position="after">
                <field name="l10n_ar_afip_code"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="base_view_partner_form" model="ir.ui.view">
        <field name="name">res.partner.form</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <field name="vat" position="after">
                <field name="l10n_ar_afip_responsibility_type_id" string="AFIP Responsibility" options="{'no_open': True, 'no_create': True}" attrs="{'readonly': [('parent_id','!=',False)]}"/>
            </field>
        </field>
    </record>

    <record id="view_partner_property_form" model="ir.ui.view">
        <field name="name">res.partner.form</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">

            <xpath expr="//page[@name='accounting']/group" position="inside">
                <group string="Accounting Documents" name="accounting_documents">
                    <field name="l10n_ar_special_purchase_document_type_ids" widget="many2many_tags"/>
                </group>
            </xpath>

            <field name="property_account_position_id" position="after">
                <label for="l10n_ar_gross_income_number" string="Gross Income"/>
                <div name="gross_income">
                    <field name="l10n_ar_gross_income_type" class="oe_inline"/>
                    <field name="l10n_ar_gross_income_number" placeholder="Number..." class="oe_inline" attrs="{'invisible': [('l10n_ar_gross_income_type', 'not in', ['multilateral', 'local'])], 'required': [('l10n_ar_gross_income_type', 'in', ['multilateral', 'local'])]}"/>
                </div>
            </field>

        </field>
    </record>

    <record id="view_res_partner_filter" model="ir.ui.view">
        <field name="name">view.res.partner.filter.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_res_partner_filter"/>
        <field name="arch" type="xml">
            <field name="category_id" position="after">
                <field name="l10n_ar_afip_responsibility_type_id"/>
            </field>
            <filter name="salesperson" position="before">
                <filter string="AFIP Responsibility Type" name="l10n_ar_afip_responsibility_type_id_filter" context="{'group_by': 'l10n_ar_afip_responsibility_type_id'}"/>
            </filter>
        </field>
    </record>

</odoo>

```

## File: views\uom_uom_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="product_uom_tree_view" model="ir.ui.view">
        <field name="name">uom.uom.tree</field>
        <field name="model">uom.uom</field>
        <field name="inherit_id" ref="uom.product_uom_tree_view"/>
        <field name="arch" type="xml">
            <tree>
                <field name="l10n_ar_afip_code"/>
            </tree>
        </field>
    </record>

    <record id="product_uom_form_view" model="ir.ui.view">
        <field name="name">uom.uom.form</field>
        <field name="model">uom.uom</field>
        <field name="inherit_id" ref="uom.product_uom_form_view"/>
        <field name="arch" type="xml">
            <field name="rounding" position="after">
                <field name="l10n_ar_afip_code"/>
            </field>
        </field>
    </record>

</odoo>

```

