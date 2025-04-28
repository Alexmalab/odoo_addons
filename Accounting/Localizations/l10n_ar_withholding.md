# Odoo Module: l10n_ar_withholding

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizards
from . import demo

import logging

_logger = logging.getLogger(__name__)


def _l10n_ar_withholding_post_init(env):
    """ Existing companies that have the Argentinean Chart of Accounts set """
    template_codes = ['ar_ri', 'ar_ex', 'ar_base']
    ar_companies = env['res.company'].search([('chart_template', 'in', template_codes), ('parent_id', '=', False)])
    for company in ar_companies:
        template_code = company.chart_template
        ChartTemplate = env['account.chart.template'].with_company(company)
        data = {
            model: ChartTemplate._parse_csv(template_code, model, module='l10n_ar_withholding')
            for model in [
                'account.account',
                'account.tax.group',
                'account.tax',
            ]
        }
        ChartTemplate._deref_account_tags(template_code, data['account.tax'])
        ChartTemplate._pre_reload_data(company, {}, data)
        ChartTemplate._load_data(data)
        company.l10n_ar_tax_base_account_id = ChartTemplate.ref('base_tax_account')

        if env.ref('base.module_l10n_ar_withholding').demo:
            env['account.chart.template']._post_load_demo_data(company)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Argentina - Payment Withholdings',
    'version': "1.0",
    'description': """Allows to register withholdings during the payment of an invoice.""",
    'author': 'ADHOC SA',
    'countries': ['ar'],
    'category': 'Accounting/Localizations',
    'depends': [
        'l10n_ar',
        'l10n_latam_check',
    ],
    'data': [
        'views/account_tax_views.xml',
        'views/account_payment_view.xml',
        'views/report_payment_receipt_templates.xml',
        'views/res_config_settings.xml',
        'wizards/account_payment_register_views.xml',
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'post_init_hook': '_l10n_ar_withholding_post_init',
    'license': 'LGPL-3',
}

```

## File: data\template\account.account-ar_base.csv

```csv
"id","code","account_type","name","reconcile","name@es"
"base_tax_account","6.0.0.00.020","asset_current","Tax Base Account","True","Base imponible"

```

## File: data\template\account.tax-ar_ex.csv

```csv
"id","name","description","active","sequence","amount_type","amount","tax_group_id","type_tax_use","l10n_ar_withholding_payment_type","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","name@es","description@es"
"ri_tax_withholding_suss_incurred","WTH SUSS I","SUSS Withholding incurred","True",10,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret SUSS S","Retención SUSS Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_suss_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_suss_sufrida","",""
"ri_tax_withholding_ganancias_incurred","Earnings WTH I","Earnings withholding incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret Ganancias S","Retención Ganancias Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_ganancias_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_ganancias_sufrida","",""
"ri_tax_withholding_ganancias_applied","Earnings WTH A","Earnings withholding applied","True",4,"percent",15,"tax_group_withholding_vat","none","supplier","","","","Ret Ganancias A","Retención Ganancias Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ri_tax_withholding_iibb_caba_incurred","IIBB WTH CABA I","IIBB withholding CABA incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB CABA S","Retención IIBB CABA Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_caba_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_caba_sufrida","",""
"ri_tax_withholding_iibb_ba_incurred","IIBB WTH ARBA I","IIBB withholding ARBA incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB ARBA S","Retención IIBB ARBA Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_ba_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_ba_sufrida","",""
"ri_tax_withholding_iibb_ca_incurred","IIBB WTH Catamarca I","IIBB withholding Catamarca incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Catamarca S","Retención IIBB Catamarca Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_ca_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_ca_sufrida","",""
"ri_tax_withholding_iibb_co_incurred","IIBB WTH Córdoba I","IIBB withholding Córdoba incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Córdoba S","Retención IIBB Córdoba Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_co_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_co_sufrida","",""
"ri_tax_withholding_iibb_rr_incurred","IIBB WTH Corrientes I","IIBB withholding Corrientes incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Corrientes S","Retención IIBB Corrientes Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_rr_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_rr_sufrida","",""
"ri_tax_withholding_iibb_er_incurred","IIBB WTH Entre Ríos I","IIBB withholding Entre Ríos incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Entre Ríos S","Retención IIBB Entre Ríos Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_er_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_er_sufrida","",""
"ri_tax_withholding_iibb_ju_incurred","IIBB WTH Jujuy I","IIBB withholding Jujuy incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Jujuy S","Retención IIBB Jujuy Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_ju_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_ju_sufrida","",""
"ri_tax_withholding_iibb_za_incurred","IIBB WTH Mendoza I","IIBB withholding Mendoza incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Mendoza S","Retención IIBB Mendoza Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_za_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_za_sufrida","",""
"ri_tax_withholding_iibb_lr_incurred","IIBB WTH La Rioja I","IIBB withholding La Rioja incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB La Rioja S","Retención IIBB La Rioja Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_lr_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_lr_sufrida","",""
"ri_tax_withholding_iibb_sa_incurred","IIBB WTH Salta I","IIBB withholding Salta incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Salta S","Retención IIBB Salta Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_sa_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_sa_sufrida","",""
"ri_tax_withholding_iibb_nn_incurred","IIBB WTH San Juan I","IIBB withholding San Juan incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB San Juan S","Retención IIBB San Juan Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_nn_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_nn_sufrida","",""
"ri_tax_withholding_iibb_sl_incurred","IIBB WTH San Luis I","IIBB withholding San Luis incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB San Luis S","Retención IIBB San Luis Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_sl_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_sl_sufrida","",""
"ri_tax_withholding_iibb_sf_incurred","IIBB WTH Santa Fe I","IIBB withholding Santa Fe incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Santa Fe S","Retención IIBB Santa Fe Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_sf_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_sf_sufrida","",""
"ri_tax_withholding_iibb_se_incurred","IIBB WTH Santiago del Estero I","IIBB withholding Santiago del Estero incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Santiago del Estero S","Retención IIBB Santiago del Estero Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_se_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_se_sufrida","",""
"ri_tax_withholding_iibb_tn_incurred","IIBB WTH Tucumán I","IIBB withholding Tucumán incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Tucumán S","Retención IIBB Tucumán Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_tn_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_tn_sufrida","",""
"ri_tax_withholding_iibb_ha_incurred","IIBB WTH Chaco I","IIBB withholding Chaco incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Chaco S","Retención IIBB Chaco Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_ha_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_ha_sufrida","",""
"ri_tax_withholding_iibb_ct_incurred","IIBB WTH Chubut I","IIBB withholding Chubut incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Chubut S","Retención IIBB Chubut Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_ct_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_ct_sufrida","",""
"ri_tax_withholding_iibb_fo_incurred","IIBB WTH Formosa I","IIBB withholding Formosa incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Formosa S","Retención IIBB Formosa Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_fo_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_fo_sufrida","",""
"ri_tax_withholding_iibb_mi_incurred","IIBB WTH Misiones I","IIBB withholding Misiones incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Misiones S","Retención IIBB Misiones Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_mi_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_mi_sufrida","",""
"ri_tax_withholding_iibb_ne_incurred","IIBB WTH Neuquén I","IIBB withholding Neuquén incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Neuquén S","Retención IIBB Neuquén Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_ne_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_ne_sufrida","",""
"ri_tax_withholding_iibb_lp_incurred","IIBB WTH La Pampa I","IIBB withholding La Pampa incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB La Pampa S","Retención IIBB La Pampa Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_lp_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_lp_sufrida","",""
"ri_tax_withholding_iibb_rn_incurred","IIBB WTH Río Negro I","IIBB withholding Río Negro incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Río Negro S","Retención IIBB Río Negro Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_rn_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_rn_sufrida","",""
"ri_tax_withholding_iibb_az_incurred","IIBB WTH Santa Cruz I","IIBB withholding Santa Cruz incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Santa Cruz S","Retención IIBB Santa Cruz Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_az_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_az_sufrida","",""
"ri_tax_withholding_iibb_tf_incurred","IIBB WTH Tierra del Fuego I","IIBB withholding Tierra del Fuego incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","","","","Ret IIBB Tierra del Fuego S","Retención IIBB Tierra del Fuego Sufrida"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","base_retencion_iibb_tf_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","base_retencion_iibb_tf_sufrida","",""
"ri_tax_withholding_iibb_caba_applied","IIBB WTH CABA A","IIBB withholding CABA applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB CABA A","Retención IIBB CABA Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_caba_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_caba_aplicada","",""
"ri_tax_withholding_iibb_ba_applied","IIBB WTH ARBA A","IIBB withholding ARBA applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB ARBA A","Retención IIBB ARBA Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_ba_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_ba_aplicada","",""
"ri_tax_withholding_iibb_ca_applied","IIBB WTH Catamarca A","IIBB withholding Catamarca applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Catamarca A","Retención IIBB Catamarca Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_ca_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_ca_aplicada","",""
"ri_tax_withholding_iibb_co_applied","IIBB WTH Córdoba A","IIBB withholding Córdoba applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Córdoba A","Retención IIBB Córdoba Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_co_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_co_aplicada","",""
"ri_tax_withholding_iibb_rr_applied","IIBB WTH Corrientes A","IIBB withholding Corrientes applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Corrientes A","Retención IIBB Corrientes Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_rr_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_rr_aplicada","",""
"ri_tax_withholding_iibb_er_applied","IIBB WTH Entre Ríos A","IIBB withholding Entre Ríos applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Entre Ríos A","Retención IIBB Entre Ríos Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_er_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_er_aplicada","",""
"ri_tax_withholding_iibb_ju_applied","IIBB WTH Jujuy A","IIBB withholding Jujuy applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Jujuy A","Retención IIBB Jujuy Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_ju_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_ju_aplicada","",""
"ri_tax_withholding_iibb_za_applied","IIBB WTH Mendoza A","IIBB withholding Mendoza applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Mendoza A","Retención IIBB Mendoza Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_za_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_za_aplicada","",""
"ri_tax_withholding_iibb_lr_applied","IIBB WTH La Rioja A","IIBB withholding La Rioja applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB La Rioja A","Retención IIBB La Rioja Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_lr_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_lr_aplicada","",""
"ri_tax_withholding_iibb_sa_applied","IIBB WTH Salta A","IIBB withholding Salta applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Salta A","Retención IIBB Salta Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_sa_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_sa_aplicada","",""
"ri_tax_withholding_iibb_nn_applied","IIBB WTH San Juan A","IIBB withholding San Juan applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB San Juan A","Retención IIBB San Juan Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_nn_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_nn_aplicada","",""
"ri_tax_withholding_iibb_sl_applied","IIBB WTH San Luis A","IIBB withholding San Luis applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB San Luis A","Retención IIBB San Luis Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_sl_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_sl_aplicada","",""
"ri_tax_withholding_iibb_sf_applied","IIBB WTH Santa Fe A","IIBB withholding Santa Fe applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Santa Fe A","Retención IIBB Santa Fe Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_sf_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_sf_aplicada","",""
"ri_tax_withholding_iibb_se_applied","IIBB WTH Santiago del Estero A","IIBB withholding Santiago del Estero applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Santiago del Estero A","Retención IIBB Santiago del Estero Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_se_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_se_aplicada","",""
"ri_tax_withholding_iibb_tn_applied","IIBB WTH Tucumán A","IIBB withholding Tucumán applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Tucumán A","Retención IIBB Tucumán Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_tn_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_tn_aplicada","",""
"ri_tax_withholding_iibb_ha_applied","IIBB WTH Chaco A","IIBB withholding Chaco applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Chaco A","Retención IIBB Chaco Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_ha_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_ha_aplicada","",""
"ri_tax_withholding_iibb_ct_applied","IIBB WTH Chubut A","IIBB withholding Chubut applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Chubut A","Retención IIBB Chubut Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_ct_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_ct_aplicada","",""
"ri_tax_withholding_iibb_fo_applied","IIBB WTH Formosa A","IIBB withholding Formosa applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Formosa A","Retención IIBB Formosa Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_fo_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_fo_aplicada","",""
"ri_tax_withholding_iibb_mi_applied","IIBB WTH Misiones A","IIBB withholding Misiones applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Misiones A","Retención IIBB Misiones Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_mi_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_mi_aplicada","",""
"ri_tax_withholding_iibb_ne_applied","IIBB WTH Neuquén A","IIBB withholding Neuquén applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Neuquén A","Retención IIBB Neuquén Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_ne_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_ne_aplicada","",""
"ri_tax_withholding_iibb_lp_applied","IIBB WTH La Pampa A","IIBB withholding La Pampa applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB La Pampa A","Retención IIBB La Pampa Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_lp_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_lp_aplicada","",""
"ri_tax_withholding_iibb_rn_applied","IIBB WTH Río Negro A","IIBB withholding Río Negro applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Río Negro A","Retención IIBB Río Negro Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_rn_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_rn_aplicada","",""
"ri_tax_withholding_iibb_az_applied","IIBB WTH Santa Cruz A","IIBB withholding Santa Cruz applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Santa Cruz A","Retención IIBB Santa Cruz Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_az_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_az_aplicada","",""
"ri_tax_withholding_iibb_tf_applied","IIBB WTH Tierra del Fuego A","IIBB withholding Tierra del Fuego applied","True",4,"percent",0,"tax_group_withholding_vat","none","supplier","","","","Ret IIBB Tierra del Fuego A","Retención IIBB Tierra del Fuego Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iibb_tf_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iibb_tf_aplicada","",""

```

## File: data\template\account.tax-ar_ri.csv

```csv
"id","name","description","active","sequence","amount_type","amount","tax_group_id","type_tax_use","l10n_ar_withholding_payment_type","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","name@es","description@es"
"ri_tax_withholding_vat_incurred","VAT WTH I","Withholding VAT incurred","True",4,"percent",0,"tax_group_withholding_vat","none","customer","base","invoice","","Ret IVA S","Retención IVA Sufrida"
"","","","","","","","","","","tax","invoice","ri_retencion_iva_sufrida","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iva_sufrida","",""
"ri_tax_withholding_vat_applied","VAT WTH A","IVA withholding applied","True",4,"percent","4.1","tax_group_withholding_vat","none","supplier","","","","Ret IVA A","Retención IVA Aplicada"
"","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","tax","invoice","ri_retencion_iva_aplicada","",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","ri_retencion_iva_aplicada","",""

```

## File: data\template\account.tax.group-ar_ex.csv

```csv
"id","name","country_id","l10n_ar_tribute_afip_code","name@es"
"tax_group_withholding_vat","VAT Withholding","base.ar","01","Retenciones"

```

## File: models\account_chart_template.py

```python
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):

    _inherit = 'account.chart.template'

    # ar base
    @template('ar_base', 'account.account')
    def _get_ar_base_withholding_account_account(self):
        return self._parse_csv('ar_base', 'account.account', module='l10n_ar_withholding')

    # ri chart
    @template('ar_ri', 'account.tax.group')
    def _get_ar_ri_withholding_account_tax_group(self):
        return self._parse_csv('ar_ri', 'account.tax.group', module='l10n_ar_withholding')

    @template('ar_ri', 'account.tax')
    def _get_ar_ri_withholding_account_tax(self):
        additional = self._parse_csv('ar_ri', 'account.tax', module='l10n_ar_withholding')
        self._deref_account_tags('ar_ri', additional)
        return additional

    # ex chart
    @template('ar_ex', 'account.tax.group')
    def _get_ar_ex_withholding_account_tax_group(self):
        return self._parse_csv('ar_ex', 'account.tax.group', module='l10n_ar_withholding')

    @template('ar_ex', 'account.tax')
    def _get_ar_ex_withholding_account_tax(self):
        additional = self._parse_csv('ar_ex', 'account.tax', module='l10n_ar_withholding')
        self._deref_account_tags('ar_ex', additional)
        return additional

    @template('ar_base', 'res.company')
    def _get_ar_base_res_company(self):
        res = super()._get_ar_base_res_company()
        res[self.env.company.id].update({'l10n_ar_tax_base_account_id': 'base_tax_account'})
        return res

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api


class AccountMove(models.Model):

    _inherit = 'account.move'

    l10n_ar_withholding_ids = fields.One2many(
        'account.move.line', 'move_id', string='Withholdings',
        compute='_compute_l10n_ar_withholding_ids',
        readonly=True
    )

    @api.depends('line_ids')
    def _compute_l10n_ar_withholding_ids(self):
        for move in self:
            move.l10n_ar_withholding_ids = move.line_ids.filtered(lambda l: l.tax_line_id.l10n_ar_withholding_payment_type)

```

## File: models\account_payment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountPayment(models.Model):

    _inherit = 'account.payment'

    def _synchronize_to_moves(self, changed_fields):
        ''' If we change a payment with withholdings, delete all withholding lines as the synchronization mechanism is not
        implemented yet
        '''
        if self._context.get('skip_account_move_synchronization'):
            return

        if not any(field_name in changed_fields for field_name in self._get_trigger_fields_to_synchronize()):
            return

        for pay in self.with_context(
                skip_account_move_synchronization=True, skip_invoice_sync=True, dynamic_unlink=True):
            pay.line_ids.filtered(lambda x: x.account_id == pay.company_id.l10n_ar_tax_base_account_id or x.tax_line_id.l10n_ar_withholding_payment_type).unlink()
        res = super()._synchronize_to_moves(changed_fields)
        return res

```

## File: models\account_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models


class AccountTax(models.Model):

    _inherit = 'account.tax'

    l10n_ar_withholding_payment_type = fields.Selection(
        [('supplier', 'Supplier'), ('customer', 'Customer')], 'Argentinean Withholding type',
        compute="_compute_l10n_ar_withholding_payment_type", store=True, readonly=False)

    l10n_ar_withholding_sequence_id = fields.Many2one(
        'ir.sequence', 'Withholding Number Sequence', copy=False, check_company=True,
        help='If no sequence provided then it will be required for you to enter withholding number when registering one.')

    @api.depends('type_tax_use', 'country_code')
    def _compute_l10n_ar_withholding_payment_type(self):
        self.filtered(lambda x: not x.l10n_ar_withholding_payment_type or x.type_tax_use != 'none' or x.country_code != 'AR').l10n_ar_withholding_payment_type = False

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResCompany(models.Model):

    _inherit = 'res.company'

    l10n_ar_tax_base_account_id = fields.Many2one(
        comodel_name='account.account',
        domain=[('deprecated', '=', False)],
        string="Tax Base Account",
        help="Account that will be set on lines created to represent the tax base amounts.")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_ar_tax_base_account_id = fields.Many2one(
        comodel_name='account.account',
        related='company_id.l10n_ar_tax_base_account_id',
        readonly=False,
        domain=[('deprecated', '=', False)],
        string="Tax Base Account",
        help="Account that will be set on lines created to represent the tax base amounts.")

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_tax
from . import account_move
from . import account_payment
from . import account_chart_template
from . import res_company
from . import res_config_settings

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_ar_payment_register_withholding,access_l10n_ar_payment_register_withholding,model_l10n_ar_payment_register_withholding,account.group_account_invoice,1,1,1,1

```

## File: views\account_payment_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_account_payment_form" model="ir.ui.view">
        <field name="name">account.payment.form.inherited</field>
        <field name="model">account.payment</field>
        <field name="inherit_id" ref="account.view_account_payment_form" />
        <field name="arch" type="xml">
            <group position="inside">
                <group name="group4" colspan="2" invisible="country_code != 'AR'">
                    <field name="l10n_ar_withholding_ids" nolabel="1" colspan="2"
                        readonly="True" invisible="is_internal_transfer">
                        <tree>
                            <field name="move_name" column_invisible="True"/>
                            <field name="tax_line_id" string="Tax"/>
                            <field name="name" string="Withholding Number" required="1"/>
                            <field name="amount_currency" readonly="0" required="1" string="Amount" sum="Total"/>
                        </tree>
                    </field>
                </group>
            </group>
        </field>
    </record>
</odoo>

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_tax_form" model="ir.ui.view">
        <field name="name">account.tax.form</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <field name="type_tax_use" position="after">
                <field name="l10n_ar_withholding_payment_type" invisible="type_tax_use != 'none' or country_code != 'AR'"/>
                <field name="l10n_ar_withholding_sequence_id" context="{'default_name': name}" invisible="l10n_ar_withholding_payment_type != 'supplier'"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\report_payment_receipt_templates.xml

```xml
<odoo>
    <template inherit_id="account.report_payment_receipt_document" id="report_payment_receipt_document">
        <xpath expr="//table" position="before">
            <t t-if="o.l10n_ar_withholding_ids">
                <table id="l10n_ar_withholding"  class="table table-sm">
                        <thead>
                            <tr>
                                <th><span>Tax</span></th>
                                <th><span>Withholding number</span></th>
                                <th><span>Base</span></th>
                                <th><span>Amount</span></th>
                            </tr>
                        </thead>
                        <tbody>
                            <t t-foreach="o.l10n_ar_withholding_ids" t-as="line">
                                <t t-set="withholding_base" t-value="o.line_ids.filtered(lambda x: line.tax_line_id.id in x.tax_ids.ids)"/>
                                <tr>
                                    <td>
                                        <span t-field='line.tax_line_id.name'/>
                                    </td>
                                    <td>
                                        <span t-field='line.name'/>
                                    </td>
                                    <td class="text-end">
                                        <span t-out="abs(withholding_base.amount_currency)" t-options="{'widget': 'monetary', 'display_currency': o.currency_id}"/>
                                    </td>
                                    <td class="text-end">
                                        <span t-out="abs(line.amount_currency)" t-options="{'widget': 'monetary', 'display_currency': o.currency_id}"/>
                                    </td>
                                </tr>
                            </t>
                        </tbody>
                </table>
            </t>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="res_config_settings_view_form">
        <field name="name">res.config.settings.view.form</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="l10n_ar.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//block[@id='argentina_localization']" position="inside">
                <setting id="withholding_tax" string="Withholding"  company_dependent="1" title="Account that will be set on lines created to represent the tax base amounts.">
                    <div class="content-group" invisible="country_code != 'AR'">
                        <div class="row mt16">
                            <label for="l10n_ar_tax_base_account_id" class="col-lg-3 o_light_label"/>
                            <field name="l10n_ar_tax_base_account_id"/>
                        </div>
                    </div>
                </setting>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizards\account_payment_register.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import models, fields, api, Command, _
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class AccountPaymentRegister(models.TransientModel):
    _inherit = 'account.payment.register'

    l10n_ar_withholding_ids = fields.One2many('l10n_ar.payment.register.withholding', 'payment_register_id', string="Withholdings")
    l10n_ar_net_amount = fields.Monetary(compute='_compute_l10n_ar_net_amount', readonly=True, help="Net amount after withholdings")
    l10n_ar_adjustment_warning = fields.Boolean(compute="_compute_l10n_ar_adjustment_warning")

    @api.depends('l10n_latam_check_id', 'amount', 'l10n_ar_net_amount')
    def _compute_l10n_ar_adjustment_warning(self):
        for rec in self:
            if rec.l10n_latam_check_id and rec.l10n_ar_net_amount != rec.l10n_latam_check_id.amount:
                rec.l10n_ar_adjustment_warning = True
            else:
                rec.l10n_ar_adjustment_warning = False

    @api.depends('l10n_ar_withholding_ids.amount', 'amount')
    def _compute_l10n_ar_net_amount(self):
        for rec in self:
            rec.l10n_ar_net_amount = rec.amount - sum(rec.l10n_ar_withholding_ids.mapped('amount'))

    def _create_payment_vals_from_wizard(self, batch_result):
        payment_vals = super()._create_payment_vals_from_wizard(batch_result)
        payment_vals['amount'] = self.l10n_ar_net_amount
        conversion_rate = self._get_conversion_rate()
        sign = 1
        if self.partner_type == 'supplier':
            sign = -1
        for line in self.l10n_ar_withholding_ids:
            if not line.name:
                if line.tax_id.l10n_ar_withholding_sequence_id:
                    line.name = line.tax_id.l10n_ar_withholding_sequence_id.next_by_id()
                else:
                    raise UserError(_('Please enter withholding number for tax %s') % line.tax_id.name)
            dummy, account_id, tax_repartition_line_id = line._tax_compute_all_helper()
            balance = self.company_currency_id.round(line.amount * conversion_rate)
            payment_vals['write_off_line_vals'].append({
                    'currency_id': self.currency_id.id,
                    'name': line.name,
                    'account_id': account_id,
                    'amount_currency': sign * line.amount,
                    'balance': sign * balance,
                    'tax_base_amount': sign * line.base_amount,
                    'tax_repartition_line_id': tax_repartition_line_id,
            })

        for base_amount in list(set(self.l10n_ar_withholding_ids.mapped('base_amount'))):
            withholding_lines = self.l10n_ar_withholding_ids.filtered(lambda x: x.base_amount == base_amount)
            nice_base_label = ','.join(withholding_lines.mapped('name'))
            account_id = self.company_id.l10n_ar_tax_base_account_id.id
            base_amount = sign * base_amount
            cc_base_amount = self.company_currency_id.round(base_amount * conversion_rate)
            payment_vals['write_off_line_vals'].append({
                'currency_id': self.currency_id.id,
                'name': _('Base Ret: ') + nice_base_label,
                'tax_ids': [Command.set(withholding_lines.mapped('tax_id').ids)],
                'account_id': account_id,
                'balance': cc_base_amount,
                'amount_currency': base_amount,
            })
            payment_vals['write_off_line_vals'].append({
                'currency_id': self.currency_id.id,  # Counterpart 0 operation
                'name': _('Base Ret Cont: ') + nice_base_label,
                'account_id': account_id,
                'balance': -cc_base_amount,
                'amount_currency': -base_amount,
            })

        return payment_vals

    def _get_conversion_rate(self):
        self.ensure_one()
        if self.currency_id != self.company_id.currency_id:
            return self.env['res.currency']._get_conversion_rate(
                self.currency_id,
                self.company_id.currency_id,
                self.company_id,
                self.payment_date,
            )
        return 1.0

```

## File: wizards\account_payment_register_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_account_payment_register_form" model="ir.ui.view">
        <field name="name">account.payment.register.form</field>
        <field name="model">account.payment.register</field>
        <field name="inherit_id" ref="l10n_latam_check.view_account_payment_register_form"/>
        <field name="arch" type="xml">
            <group>
                <group name="withholdings" colspan="2" invisible="country_code != 'AR'">
                    <field name="l10n_ar_withholding_ids" invisible="not can_edit_wizard or (can_group_payments and not group_payment)">
                        <tree editable="bottom">
                            <field name="withholding_sequence_id" column_invisible="True"/>
                            <field name="company_id" column_invisible="True"/>
                            <field name="currency_id" column_invisible="True"/>
                            <field name="tax_id" options="{'no_open': True, 'no_create': True}"/>
                            <field name="name" readonly="withholding_sequence_id"/>
                            <field name="base_amount"/>
                            <field name="amount"/>

                        </tree>
                    </field>
                    <group colspan="2" invisible="not can_edit_wizard or (can_group_payments and not group_payment)">
                        <label for="l10n_ar_net_amount" string="Net Amount" invisible="l10n_latam_check_id"/>
                        <label for="l10n_ar_net_amount" string="Check amount" invisible="not l10n_latam_check_id"/>
                        <field name="l10n_ar_net_amount" nolabel="1" />
                        <field name="l10n_ar_adjustment_warning" invisible="True"/>
                        <p colspan="2" invisible="not l10n_ar_adjustment_warning" class="alert alert-warning" role="alert">
                            Adjust total amount or withholdings amount so that the check amount is the correct one.
                        </p>
                    </group>
                    <div colspan="2" class="o_row" invisible="(can_edit_wizard and not can_group_payments) or (can_group_payments and group_payment)">
                    <p class="alert alert-warning" role="alert">You can't register withholdings when paying invoices of different partners or same partner without grouping</p>
                    </div>
                </group>
            </group>
        </field>
    </record>
</odoo>

```

## File: wizards\l10n_ar_payment_register_withholding.py

```python
# pylint: disable=protected-access
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import models, fields, api

_logger = logging.getLogger(__name__)


class l10nArPaymentRegisterWithholding(models.TransientModel):
    _name = 'l10n_ar.payment.register.withholding'
    _description = 'Payment register withholding lines'
    _check_company_auto = True

    payment_register_id = fields.Many2one('account.payment.register', required=True, ondelete='cascade')
    company_id = fields.Many2one(related='payment_register_id.company_id')
    currency_id = fields.Many2one(related='payment_register_id.currency_id')
    name = fields.Char(string='Number')
    tax_id = fields.Many2one(
        'account.tax', check_company=True, required=True,
        domain="[('l10n_ar_withholding_payment_type', '=', parent.partner_type)]")
    withholding_sequence_id = fields.Many2one(related='tax_id.l10n_ar_withholding_sequence_id')
    base_amount = fields.Monetary(required=True)
    amount = fields.Monetary(required=True, compute='_compute_amount', store=True, readonly=False)

    def _tax_compute_all_helper(self):
        self.ensure_one()
        # Computes the withholding tax amount provided a base and a tax
        # It is equivalent to: amount = self.base * self.tax_id.amount / 100
        taxes_res = self.tax_id.compute_all(
            self.base_amount,
            currency=self.payment_register_id.currency_id,
            quantity=1.0,
            product=False,
            partner=False,
            is_refund=False,
        )
        tax_amount = taxes_res['taxes'][0]['amount']
        tax_account_id = taxes_res['taxes'][0]['account_id']
        tax_repartition_line_id = taxes_res['taxes'][0]['tax_repartition_line_id']
        return tax_amount, tax_account_id, tax_repartition_line_id

    @api.depends('base_amount')
    def _compute_amount(self):
        for line in self:
            if not line.tax_id:
                line.amount = 0.0
            else:
                line.amount, dummy, dummy = line._tax_compute_all_helper()

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_register
from . import l10n_ar_payment_register_withholding

```

