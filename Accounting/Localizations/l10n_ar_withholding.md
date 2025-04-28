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


def _l10n_ar_wth_post_init(env):
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
        'views/res_partner_view.xml',
        'views/l10n_ar_earnings_scale_view.xml',
        'wizards/account_payment_register_views.xml',
        'security/ir.model.access.csv',
        'security/security.xml',
        'data/earnings_table_data.xml',
    ],
    'installable': True,
    'post_init_hook': '_l10n_ar_wth_post_init',
    'license': 'LGPL-3',
}

```

## File: data\earnings_table_data.xml

```xml
<odoo noupdate="1">
<!--
    Earnings regimenes according to scale:
-->

    <record model="l10n_ar.earnings.scale" id="normal_scale">
        <field name="name">Normal Scale</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_1">
        <field name="scale_id" ref="normal_scale"/>
        <field name="to_amount">8000</field>
        <field name="fixed_amount">0</field>
        <field name="percentage">5</field>
        <field name="excess_amount">0</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_2">
        <field name="scale_id" ref="normal_scale"/>
        <field name="to_amount">16000</field>
        <field name="fixed_amount">400</field>
        <field name="percentage">9</field>
        <field name="excess_amount">8000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_3">
        <field name="scale_id" ref="normal_scale"/>
        <field name="to_amount">24000</field>
        <field name="fixed_amount">1120</field>
        <field name="percentage">12</field>
        <field name="excess_amount">16000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_4">
        <field name="scale_id" ref="normal_scale"/>
        <field name="to_amount">32000</field>
        <field name="fixed_amount">2080</field>
        <field name="percentage">15</field>
        <field name="excess_amount">24000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_5">
        <field name="scale_id" ref="normal_scale"/>
        <field name="to_amount">48000</field>
        <field name="fixed_amount">3280</field>
        <field name="percentage">19</field>
        <field name="excess_amount">32000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_6">
        <field name="scale_id" ref="normal_scale"/>
        <field name="to_amount">64000</field>
        <field name="fixed_amount">6320</field>
        <field name="percentage">23</field>
        <field name="excess_amount">48000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_7">
        <field name="scale_id" ref="normal_scale"/>
        <field name="to_amount">96000</field>
        <field name="fixed_amount">10000</field>
        <field name="percentage">27</field>
        <field name="excess_amount">64000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_8">
        <field name="scale_id" ref="normal_scale"/>
        <field name="to_amount">999999999</field>
        <field name="fixed_amount">18640</field>
        <field name="percentage">31</field>
        <field name="excess_amount">96000</field>
    </record>

<!--
    Earnings regimenes according to scale: scale 119
-->


    <record model="l10n_ar.earnings.scale" id="scale_119">
        <field name="name">Scale 119</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_9">
        <field name="scale_id" ref="scale_119"/>
        <field name="to_amount">71000</field>
        <field name="fixed_amount">0</field>
        <field name="percentage">5</field>
        <field name="excess_amount">0</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_10">
        <field name="scale_id" ref="scale_119"/>
        <field name="to_amount">142000</field>
        <field name="fixed_amount">3550</field>
        <field name="percentage">9</field>
        <field name="excess_amount">71000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_11">
        <field name="scale_id" ref="scale_119"/>
        <field name="to_amount">213000</field>
        <field name="fixed_amount">9940</field>
        <field name="percentage">12</field>
        <field name="excess_amount">142000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_12">
        <field name="scale_id" ref="scale_119"/>
        <field name="to_amount">284000</field>
        <field name="fixed_amount">18460</field>
        <field name="percentage">15</field>
        <field name="excess_amount">213000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_13">
        <field name="scale_id" ref="scale_119"/>
        <field name="to_amount">426000</field>
        <field name="fixed_amount">29110</field>
        <field name="percentage">19</field>
        <field name="excess_amount">284000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_14">
        <field name="scale_id" ref="scale_119"/>
        <field name="to_amount">568000</field>
        <field name="fixed_amount">56090</field>
        <field name="percentage">23</field>
        <field name="excess_amount">426000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_15">
        <field name="scale_id" ref="scale_119"/>
        <field name="to_amount">852000</field>
        <field name="fixed_amount">88750</field>
        <field name="percentage">27</field>
        <field name="excess_amount">568000</field>
    </record>

    <record model="l10n_ar.earnings.scale.line" id="scale_16">
        <field name="scale_id" ref="scale_119"/>
        <field name="to_amount">999999999</field>
        <field name="fixed_amount">165430</field>
        <field name="percentage">31</field>
        <field name="excess_amount">852000</field>
    </record>

</odoo>

```

## File: data\template\account.account-ar_base.csv

```csv
"id","code","account_type","name","reconcile","name@es"
"base_tax_account","6.0.0.00.020","asset_current","Tax Base Account","True","Base imponible"

```

## File: data\template\account.tax-ar_base.csv

```csv
"id","name","description","invoice_label","sequence","amount_type","amount","type_tax_use","tax_group_id","active","l10n_ar_withholding_payment_type","l10n_ar_tax_type","l10n_ar_state_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","name@es","description@es","invoice_label@es"
"base_tax_withholding_iibb_caba_incurred","IIBB WTH CABA","IIBB withholding Ciudad Autónoma de Buenos Aires","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_c","","","","Ret IIBB CABA","Retención IIBB CABA",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_caba_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_caba_sufrida","","",""
"base_tax_withholding_iibb_ba_incurred","IIBB WTH BA","IIBB withholding Buenos Aires","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_b","","","","Ret IIBB BA","Retención IIBB ARBA",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_ba_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_ba_sufrida","","",""
"base_tax_withholding_iibb_c_incurred","IIBB WTH C","IIBB withholding Catamarca","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_k","","","","Ret IIBB C","Retención IIBB Catamarca",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_ca_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_ca_sufrida","","",""
"base_tax_withholding_iibb_cba_incurred","IIBB WTH CBA","IIBB withholding Córdoba","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_x","","","","Ret IIBB CBA","Retención IIBB Córdoba",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_co_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_co_sufrida","","",""
"base_tax_withholding_iibb_cts_incurred","IIBB WTH CTS","IIBB withholding Corrientes","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_w","","","","Ret IIBB CTS","Retención IIBB Corrientes",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_rr_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_rr_sufrida","","",""
"base_tax_withholding_iibb_er_incurred","IIBB WTH ER","IIBB withholding Entre Ríos","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_e","","","","Ret IIBB ER","Retención IIBB Entre Ríos",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_er_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_er_sufrida","","",""
"base_tax_withholding_iibb_j_incurred","IIBB WTH J","IIBB withholding Jujuy","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_y","","","","Ret IIBB J","Retención IIBB Jujuy",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_ju_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_ju_sufrida","","",""
"base_tax_withholding_iibb_mza_incurred","IIBB WTH MZA","IIBB withholding Mendoza","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_m","","","","Ret IIBB MZA","Retención IIBB Mendoza",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_za_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_za_sufrida","","",""
"base_tax_withholding_iibb_lr_incurred","IIBB WTH LR","IIBB withholding La Rioja","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_f","","","","Ret IIBB LR","Retención IIBB La Rioja",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_lr_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_lr_sufrida","","",""
"base_tax_withholding_iibb_s_incurred","IIBB WTH S","IIBB withholding Salta","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_a","","","","Ret IIBB S","Retención IIBB Salta",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_sa_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_sa_sufrida","","",""
"base_tax_withholding_iibb_sj_incurred","IIBB WTH SJ","IIBB withholding San Juan","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_j","","","","Ret IIBB SJ","Retención IIBB San Juan",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_nn_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_nn_sufrida","","",""
"base_tax_withholding_iibb_sl_incurred","IIBB WTH SL","IIBB withholding San Luis","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_d","","","","Ret IIBB SL","Retención IIBB San Luis",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_sl_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_sl_sufrida","","",""
"base_tax_withholding_iibb_sf_incurred","IIBB WTH SF","IIBB withholding Santa Fe","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_s","","","","Ret IIBB SF","Retención IIBB Santa Fe",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_sf_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_sf_sufrida","","",""
"base_tax_withholding_iibb_se_incurred","IIBB WTH SE","IIBB withholding Santiago del Estero","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_g","","","","Ret IIBB SE","Retención IIBB Santiago del Estero",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_sf_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_se_sufrida","","",""
"base_tax_withholding_iibb_t_incurred","IIBB WTH T","IIBB withholding Tucumán","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_t","","","","Ret IIBB T","Retención IIBB Tucumán",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_tn_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_tn_sufrida","","",""
"base_tax_withholding_iibb_cho_incurred","IIBB WTH CHO","IIBB withholding Chaco","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_h","","","","Ret IIBB CHO","Retención IIBB Chaco",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_ha_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_ha_sufrida","","",""
"base_tax_withholding_iibb_cht_incurred","IIBB WTH CHT","IIBB withholding Chubut","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_u","","","","Ret IIBB CHT","Retención IIBB Chubut",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_ct_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_ct_sufrida","","",""
"base_tax_withholding_iibb_f_incurred","IIBB WTH F","IIBB withholding Formosa","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_p","","","","Ret IIBB F","Retención IIBB Formosa",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_fo_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_fo_sufrida","","",""
"base_tax_withholding_iibb_ms_incurred","IIBB WTH MS","IIBB withholding Misiones","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_n","","","","Ret IIBB MS","Retención IIBB Misiones",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_mi_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_mi_sufrida","","",""
"base_tax_withholding_iibb_n_incurred","IIBB WTH N","IIBB withholding Neuquén","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_q","","","","Ret IIBB N","Retención IIBB Neuquén",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_ne_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_ne_sufrida","","",""
"base_tax_withholding_iibb_lp_incurred","IIBB WTH LP","IIBB withholding La Pampa","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_l","","","","Ret IIBB LP","Retención IIBB La Pampa",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_lp_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_lp_sufrida","","",""
"base_tax_withholding_iibb_rn_incurred","IIBB WTH RN","IIBB withholding Río Negro","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_r","","","","Ret IIBB RN","Retención IIBB Río Negro",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_rn_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_rn_sufrida","","",""
"base_tax_withholding_iibb_sc_incurred","IIBB WTH SC","IIBB withholding Santa Cruz","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_z","","","","Ret IIBB SC","Retención IIBB Santa Cruz",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_az_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_az_sufrida","","",""
"base_tax_withholding_iibb_tais_incurred","IIBB WTH TAIS","IIBB withholding Tierra del Fuego","","4","percent","1","none","tax_group_withholding","True","customer","iibb_untaxed","base.state_ar_v","","","","Ret IIBB TAIS","Retención IIBB Tierra del Fuego",""
"","","","","","","","","","","","","","base","invoice","","","",""
"","","","","","","","","","","","","","tax","invoice","base_retencion_iibb_tf_sufrida","","",""
"","","","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","","","tax","refund","base_retencion_iibb_tf_sufrida","","",""

```

## File: data\template\account.tax-ar_ex.csv

```csv
"id","name","description","active","sequence","amount_type","amount","tax_group_id","type_tax_use","l10n_ar_withholding_payment_type","l10n_ar_scale_id","l10n_ar_tax_type","l10n_ar_code","l10n_ar_state_id","l10n_ar_non_taxable_amount","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","name@es","description@es"
"ex_tax_retencion_suss_sufrida","WTH SUSS","SUSS Withholding","True","10","percent","0","tax_group_withholding","none","customer","","","","","","","base","invoice","","Ret SUSS","Retención SUSS"
"","","","","","","","","","","","","","","","","tax","invoice","base_retencion_suss_sufrida","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","base_retencion_suss_sufrida","",""
"ex_tax_retencion_profits_incurred","Profits WTH","Profits withholding","True","4","percent","0","tax_group_withholding","none","customer","","","","","","","base","invoice","","Ret Ganancias","Retención Ganancias"
"","","","","","","","","","","","","","","","","tax","invoice","base_retencion_ganancias_sufrida","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","base_retencion_ganancias_sufrida","",""
"ex_tax_withholding_iibb_caba_applied","IIBB WTH CABA 0%","IIBB withholding Ciudad Autónoma Bs As","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_c","","","","","","Ret IIBB CABA 0%","Retención IIBB Ciudad Autónoma Bs As"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_caba_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_caba_aplicada","",""
"ex_tax_withholding_iibb_ba_applied","IIBB WTH BA 0%","IIBB withholding Buenos Aires","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_b","","","","","","Ret IIBB BA 0%","Retención IIBB Buenos Aires"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_ba_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_ba_aplicada","",""
"ex_tax_withholding_iibb_c_applied","IIBB WTH C 0%","IIBB withholding Catamarca","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_k","","","","","","Ret IIBB C 0%","Retención IIBB Catamarca"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_ca_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_ca_aplicada","",""
"ex_tax_withholding_iibb_cba_applied","IIBB WTH CBA 0%","IIBB withholding Córdoba","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_total","","base.state_ar_x","","","","","","Ret IIBB CBA 0%","Retención IIBB Córdoba"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_co_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_co_aplicada","",""
"ex_tax_withholding_iibb_cts_applied","IIBB WTH CTS 0%","IIBB withholding Corrientes","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_w","","","","","","Ret IIBB CTS 0%","Retención IIBB Corrientes"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_rr_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_rr_aplicada","",""
"ex_tax_withholding_iibb_er_applied","IIBB WTH ER 0%","IIBB withholding Entre Ríos","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_e","","","","","","Ret IIBB ER 0%","Retención IIBB Entre Ríos"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_er_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_er_aplicada","",""
"ex_tax_withholding_iibb_j_applied","IIBB WTH J 0%","IIBB withholding Jujuy","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_y","","","","","","Ret IIBB J 0%","Retención IIBB Jujuy"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_ju_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_ju_aplicada","",""
"ex_tax_withholding_iibb_mza_applied","IIBB WTH MZA 0%","IIBB withholding Mendoza","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_m","","","","","","Ret IIBB MZA 0%","Retención IIBB Mendoza"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_za_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_za_aplicada","",""
"ex_tax_withholding_iibb_lr_applied","IIBB WTH LR 0%","IIBB withholding La Rioja","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_f","","","","","","Ret IIBB LR 0%","Retención IIBB La Rioja"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_lr_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_lr_aplicada","",""
"ex_tax_withholding_iibb_s_applied","IIBB WTH S 0%","IIBB withholding Salta","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_a","","","","","","Ret IIBB S 0%","Retención IIBB Salta"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_sa_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_sa_aplicada","",""
"ex_tax_withholding_iibb_sj_applied","IIBB WTH SJ 0%","IIBB withholding San Juan","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_j","","","","","","Ret IIBB SJ 0%","Retención IIBB San Juan"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_nn_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_nn_aplicada","",""
"ex_tax_withholding_iibb_sl_applied","IIBB WTH SL 0%","IIBB withholding San Luis","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_d","","","","","","Ret IIBB SL 0%","Retención IIBB San Luis"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_sl_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_sl_aplicada","",""
"ex_tax_withholding_iibb_sf_applied","IIBB WTH SF 0%","IIBB withholding Santa Fe","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_s","","","","","","Ret IIBB SF 0%","Retención IIBB Santa Fe"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_sf_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_sf_aplicada","",""
"ex_tax_withholding_iibb_se_applied","IIBB WTH SE 0%","IIBB withholding Santiago del Estero","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_g","","","","","","Ret IIBB SE 0%","Retención IIBB Santiago del Estero"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_se_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_se_aplicada","",""
"ex_tax_withholding_iibb_t_applied","IIBB WTH T 0%","IIBB withholding Tucumán","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_t","","","","","","Ret IIBB T 0%","Retención IIBB Tucumán"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_tn_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_tn_aplicada","",""
"ex_tax_withholding_iibb_cho_applied","IIBB WTH CHO 0%","IIBB withholding Chaco","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_h","","","","","","Ret IIBB CHO 0%","Retención IIBB Chaco"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_ha_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_ha_aplicada","",""
"ex_tax_withholding_iibb_cht_applied","IIBB WTH CHT 0%","IIBB withholding Chubut","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_u","","","","","","Ret IIBB CHT 0%","Retención IIBB Chubut"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_ct_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_ct_aplicada","",""
"ex_tax_withholding_iibb_f_applied","IIBB WTH F 0%","IIBB withholding Formosa","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_p","","","","","","Ret IIBB F 0%","Retención IIBB Formosa"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_fo_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_fo_aplicada","",""
"ex_tax_withholding_iibb_ms_applied","IIBB WTH MS 0%","IIBB withholding Misiones","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_n","","","","","","Ret IIBB MS 0%","Retención IIBB Misiones"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_mi_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_mi_aplicada","",""
"ex_tax_withholding_iibb_n_applied","IIBB WTH N 0%","IIBB withholding Neuquén","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_q","","","","","","Ret IIBB N 0%","Retención IIBB Neuquén"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_ne_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_ne_aplicada","",""
"ex_tax_withholding_iibb_lp_applied","IIBB WTH LP 0%","IIBB withholding La Pampa","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_l","","","","","","Ret IIBB LP 0%","Retención IIBB La Pampa"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_lp_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_lp_aplicada","",""
"ex_tax_withholding_iibb_rn_applied","IIBB WTH RN 0%","IIBB withholding Río Negro","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_r","","","","","","Ret IIBB RN 0%","Retención IIBB Río Negro"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_rn_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_rn_aplicada","",""
"ex_tax_withholding_iibb_sc_applied","IIBB WTH SC 0%","IIBB withholding Santa Cruz","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_z","","","","","","Ret IIBB SC 0%","Retención IIBB Santa Cruz"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_az_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_az_aplicada","",""
"ex_tax_withholding_iibb_tais_applied","IIBB WTH TAIS 0%","IIBB withholding Tierra del Fuego","True","4","percent","0","tax_group_withholding","none","supplier","","iibb_untaxed","","base.state_ar_v","","","","","","Ret IIBB TAIS 0%","Retención IIBB Tierra del Fuego"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iibb_tf_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iibb_tf_aplicada","",""
"ex_tax_withholding_vat_applied","VAT WTH","VAT withholding","True","4","percent","0","tax_group_withholding","none","supplier","","","","","","","","","","Ret IVA","Retención IVA"
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iva_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iva_aplicada","",""
"ex_tax_withholding_profits_regimen_19_insc","Profits WTH regimen 19 inscripto","Intereses por operaciones realizadas en entidades financieras. Ley N° 21.526, y sus modificaciones o agentes de bolsa o mercado abierto.","True","4","percent","3","tax_group_withholding","none","supplier","","earnings","19","","0","","","","","Retenciones de ganancias regimen 19 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_21_insc","Profits WTH regimen 21 inscripto","Intereses originados en operaciones no comprendidas en el punto 1.","True","4","percent","6","tax_group_withholding","none","supplier","","earnings","21","","7870","","","","","Retenciones de ganancias regimen 21 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_30_insc","Profits WTH regimen 30 inscripto","Alquileres o arrendamientos de bienes muebles.","True","4","percent","6","tax_group_withholding","none","supplier","","earnings","30","","11200","","","","","Retenciones de ganancias regimen 30 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_31_insc","Profits WTH regimen 31 inscripto","Bienes Inmuebles Urbanos, incluidos los efectuados bajo la modalidad de leasing - incluye suburbanos-","True","4","percent","6","tax_group_withholding","none","supplier","","earnings","31","","11200","","","","","Retenciones de ganancias regimen 31 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_32_insc","Profits WTH regimen 32 inscripto","Bienes Inmuebles Rurales, incluidos los efectuados bajo la modalidad de leasing - incluye subrurales-","True","4","percent","6","tax_group_withholding","none","supplier","","earnings","32","","11200","","","","","Retenciones de ganancias regimen 32 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_35_insc","Profits WTH regimen 35 inscripto","Regalías","True","4","percent","6","tax_group_withholding","none","supplier","","earnings","35","","7870","","","","","Retenciones de ganancias regimen 35 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_43_insc","Profits WTH regimen 43 inscripto","Interés accionario, excedentes y retornos distribuidos entre asociados, cooperativas, -excepto consumo-.","True","4","percent","6","tax_group_withholding","none","supplier","","earnings","43","","7870","","","","","Retenciones de ganancias regimen 43 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_51_insc","Profits WTH regimen 51 inscripto","Obligaciones de no hacer, o por abandono o no ejercicio de una actividad.","True","4","percent","6","tax_group_withholding","none","supplier","","earnings","51","","7870","","","","","Retenciones de ganancias regimen 51 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_78_insc","Profits WTH regimen 78 inscripto","Enajenación de bienes muebles y bienes de cambio.","True","4","percent","2","tax_group_withholding","none","supplier","","earnings","78","","224000","","","","","Retenciones de ganancias regimen 78 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_86_insc","Profits WTH regimen 86 inscripto","Transferencia temporaria o definitiva de derechos de llave, marcas, patentes de invención, regalías, concesiones y similares.","True","4","percent","2","tax_group_withholding","none","supplier","","earnings","86","","224000","","","","","Retenciones de ganancias regimen 86 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_110_insc","Profits WTH regimen 110 inscripto","Explotación de derechos de autor (Ley N° 11.723).","True","4","percent","-1","tax_group_withholding","none","supplier","l10n_ar_withholding.normal_scale","earnings_scale","110","","10000","","","","","Retenciones de ganancias regimen 110 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_94_insc","Profits WTH regimen 94 inscripto","Locaciones de obra y/o servicios no ejecutados en relación de dependencia no mencionados expresamente en otros incisos.","True","4","percent","2","tax_group_withholding","none","supplier","","earnings","94","","67170","","","","","Retenciones de ganancias regimen 94 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_25_insc","Profits WTH regimen 25 inscripto","Comisiones u otras retribuciones derivadas de la actividad de comisionista, rematador, consignatario y demás auxiliares de comercio a que se refiere el inciso c) del artículo 49 de la Ley de Impuesto a las Ganancias, texto ordenado en 1997 y sus modificaciones.","True","4","percent","-1","tax_group_withholding","none","supplier","l10n_ar_withholding.normal_scale","earnings_scale","25","","16830","","","","","Retenciones de ganancias regimen 25 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_116I_insc","Profits WTH regimen 116I inscripto","Honorarios de director de sociedades anónimas, síndico fiduciario, consejero de sociedades cooperativas integrante de consejos de vigilancia y socios administradores de las sociedades de responsabilidad limitada, en comandita simple y en comandita por acciones.","True","4","percent","-1","tax_group_withholding","none","supplier","l10n_ar_withholding.normal_scale","earnings_scale","116 I","","67170","","","","","Retenciones de ganancias regimen 116I inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_116II_insc","Profits WTH regimen 116II inscripto","Albacea, mandatario, gestor de negocio.","True","4","percent","-1","tax_group_withholding","none","supplier","l10n_ar_withholding.normal_scale","earnings_scale","116 II","","16830","","","","","Retenciones de ganancias regimen 116II inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_119_insc","Profits WTH regimen 119 inscripto","Profesionales liberales, oficios","True","4","percent","-1","tax_group_withholding","none","supplier","l10n_ar_withholding.scale_119","earnings_scale","119","","160000","","","","","Retenciones de ganancias regimen 119 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_124_insc","Profits WTH regimen 124 inscripto","Corredor, viajante de comercio y despachante de aduana.","True","4","percent","-1","tax_group_withholding","none","supplier","l10n_ar_withholding.normal_scale","earnings_scale","124","","16830","","","","","Retenciones de ganancias regimen 124 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_95_insc","Profits WTH regimen 95 inscripto","Operaciones de transporte de carga nacional e internacional.","True","4","percent","0.25","tax_group_withholding","none","supplier","","earnings","95","","67170","","","","","Retenciones de ganancias regimen 95 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_53_insc","Profits WTH regimen 53 inscripto","Operaciones realizadas por intermedio de mercados de cereales a término que se resuelvan en el curso del término (arbitrajes) y de mercados de futuros y opciones.","True","4","percent","0.5","tax_group_withholding","none","supplier","","earnings","53","","0","","","","","Retenciones de ganancias regimen 53 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_55_insc","Profits WTH regimen 55 inscripto","Distribución de películas . Transmisión de programación. Televisión vía satelital.","True","4","percent","0.5","tax_group_withholding","none","supplier","","earnings","55","","0","","","","","Retenciones de ganancias regimen 55 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_60_insc","Profits WTH regimen 60 inscripto","Dividendos y utilidades asimilables. Art. 90.3 de la ley. Alícuota 7%.","True","4","percent","7","tax_group_withholding","none","supplier","","earnings","60","","0","","","","","Retenciones de ganancias regimen 60 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_61_insc","Profits WTH regimen 61 inscripto","Dividendos y utilidades asimilables. Art. 90.3 de la ley. Alícuota 13%.","True","4","percent","13","tax_group_withholding","none","supplier","","earnings","61","","0","","","","","Retenciones de ganancias regimen 61 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_111_insc","Profits WTH regimen 111 inscripto","Cualquier otra cesión o locación de derechos, excepto las que correspondan a operaciones realizadas por intermedio de mercados de cereales a término que se resuelvan en el curso del término (arbitrajes) y de mercados de futuros y opciones.","True","4","percent","0.5","tax_group_withholding","none","supplier","","earnings","111","","0","","","","","Retenciones de ganancias regimen 111 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_112_insc","Profits WTH regimen 112 inscripto","Beneficios provenientes del cumplimiento de los requisitos de los planes de seguro de retiro privados administrados por entidades sujetas al control de la Superintendencia de Seguros de la Nación, establecidos por el inciso d) del artículo 45 y el inciso d) del artículo 79 de la Ley del Impuesto a las Ganancias, texto ordenado en 1997 y sus modificaciones -excepto cuando se encuentren alcanzados por el régimen de retención establecido por la Resolución General Nº 1261, sus modificatorias y complementarias-.","True","4","percent","3","tax_group_withholding","none","supplier","","earnings","112","","16830","","","","","Retenciones de ganancias regimen 112 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_113_insc","Profits WTH regimen 113 inscripto","Rescates -totales o parciales- por desistimiento  de los planes de seguro de retiro a que se refiere el inciso o), excepto que sea de aplicación lo normado en el artículo 101 de la Ley del Impuesto a las Ganancias, texto ordenado en 1997 y sus modificaciones.","True","4","percent","3","tax_group_withholding","none","supplier","","earnings","113","","16830","","","","","Retenciones de ganancias regimen 113 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_779_insc","Profits WTH regimen 779 inscripto","Subsidios abonados por los estados Nacional, provinciales, municipales o el Gobierno de la Ciudad Autónoma de Buenos Aires, en concepto de enajenación de bienes muebles y bienes de cambio, en la medida que una ley general o especial no establezca la exención de los mismos en el impuesto a las ganancias.","True","4","percent","2","tax_group_withholding","none","supplier","","earnings","779","","76140","","","","","Retenciones de ganancias regimen 779 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_780_insc","Profits WTH regimen 780 inscripto","Subsidios abonados por los estados Nacional, provinciales, municipales o el Gobierno de la Ciudad Autónoma de Buenos Aires, en concepto de locaciones de obra y/o servicios, no ejecutados en relación de dependencia, en la medida que una ley general o especial no establezca la exención de los mismos en el impuesto a las ganancias.","True","4","percent","2","tax_group_withholding","none","supplier","","earnings","780","","31460","","","","","Retenciones de ganancias regimen 780 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_99_insc","Profits WTH regimen 99 inscripto","Factura M - Ganancias - Emisión de comprobantes con discriminación del gravamen.","True","4","percent","6","tax_group_withholding","none","supplier","","earnings","99","","0","","","","","Retenciones de ganancias regimen 99 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""
"ex_tax_withholding_profits_regimen_965_insc","Profits WTH regimen 965 inscripto","Factura 'A' con leyenda 'OPERACIÓN SUJETA A RETENCIÓN' - Ganancias - Emisión de comprobantes con discriminación del gravamen.","True","4","percent","3","tax_group_withholding","none","supplier","","earnings","965","","0","","","","","Retenciones de ganancias regimen 965 inscripto",""
"","","","","","","","","","","","","","","","","base","invoice","","",""
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_ganancias_aplicada","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_ganancias_aplicada","",""

```

## File: data\template\account.tax-ar_ri.csv

```csv
"id","name","type_tax_use","amount_type","description","active","sequence","tax_group_id","l10n_ar_withholding_payment_type","l10n_ar_scale_id","l10n_ar_tax_type","l10n_ar_code","l10n_ar_state_id","amount","l10n_ar_non_taxable_amount","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","name@es","description@es"
"ri_tax_withholding_vat_incurred","VAT WTH","none","percent","Withholding VAT","True","4","tax_group_withholding","customer","","","","","0","","","base","invoice","","Ret IVA","Retención IVA"
"","","","","","","","","","","","","","","","","tax","invoice","ri_retencion_iva_sufrida","",""
"","","","","","","","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","","","","","","","tax","refund","ri_retencion_iva_sufrida","",""

```

## File: data\template\account.tax.group-ar_base.csv

```csv
"id","name","country_id","name@es"
"tax_group_withholding","Withholding","base.ar","Retenciones"

```

## File: data\template\account.tax.group-ar_ex.csv

```csv
"id","name","country_id","l10n_ar_tribute_afip_code","name@es"
"tax_group_withholding","VAT Withholding","base.ar","01","Retenciones"

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
from odoo import models, fields


class AccountPayment(models.Model):

    _inherit = 'account.payment'

    l10n_ar_withholding_ids = fields.One2many(related='move_id.l10n_ar_withholding_ids')

    def _synchronize_to_moves(self, changed_fields):
        ''' If we change a payment with withholdings, delete all withholding lines as the synchronization mechanism is not
        implemented yet
        '''
        if not any(field_name in changed_fields for field_name in self._get_trigger_fields_to_synchronize()):
            return
        for pay in self:
            pay.move_id.line_ids.filtered(
                lambda x:
                x.account_id == pay.company_id.l10n_ar_tax_base_account_id or
                x.tax_line_id.l10n_ar_withholding_payment_type
            ).unlink()
        res = super()._synchronize_to_moves(changed_fields)
        return res

```

## File: models\account_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models


class AccountTax(models.Model):

    _inherit = 'account.tax'

    l10n_ar_type_tax_use = fields.Selection(
        selection=[
            ('sale', 'Sales'),
            ('purchase', 'Purchases'),
            ('none', 'Other'),
            ('supplier', 'Vendor Payment Withholding'),
            ('customer', 'Customer Payment Withholding')
        ],
        compute='_compute_l10n_ar_type_tax_use', inverse='_inverse_l10n_ar_type_tax_use',
        string="Argentina Tax Type"
    )
    l10n_ar_withholding_payment_type = fields.Selection(
        selection=[('supplier', 'Vendor Payment'), ('customer', 'Customer Payment')],
        string="Argentina Withholding Payment Type",
        help="Withholding tax for supplier or customer payments.")
    l10n_ar_tax_type = fields.Selection(
        string='WTH Tax',
        selection=[
            ('earnings', 'Earnings'),
            ('earnings_scale', 'Earnings Scale'),
            ('iibb_untaxed', 'IIBB Untaxed'),
            ('iibb_total', 'IIBB Total Amount'),
        ]
    )
    l10n_ar_withholding_sequence_id = fields.Many2one(
        'ir.sequence',
        string='WTH Sequence',
        copy=False, check_company=True,
        help='If no sequence provided then it will be required for you to enter withholding number when registering one.')
    l10n_ar_code = fields.Char('AFIP Code')
    l10n_ar_non_taxable_amount = fields.Float(
        string='Non Taxable Amount',
        digits='Account',
        help="Until this base amount, the tax is not applied."
    )
    l10n_ar_minimum_threshold = fields.Float(
        string="Minimum Treshold",
        help="If the calculated withholding tax amount is lower than minimum withholding threshold then it is 0.0.")
    l10n_ar_state_id = fields.Many2one(
        'res.country.state', string="Jurisdiction", ondelete='restrict', domain="[('country_id', '=?', country_id)]")
    l10n_ar_scale_id = fields.Many2one(
        comodel_name='l10n_ar.earnings.scale',
        string="Scale", help="Earnings table scale if tax type is 'Earnings Scale'."
    )

    @api.depends('type_tax_use', 'l10n_ar_withholding_payment_type')
    def _compute_l10n_ar_type_tax_use(self):
        for tax in self:
            if tax.type_tax_use in ('sale', 'purchase'):
                tax.l10n_ar_type_tax_use = tax.type_tax_use
            elif tax.l10n_ar_withholding_payment_type in ('supplier', 'customer'):
                tax.l10n_ar_type_tax_use = tax.l10n_ar_withholding_payment_type
            else:
                tax.l10n_ar_type_tax_use = 'none'

    @api.onchange('l10n_ar_type_tax_use')
    def _inverse_l10n_ar_type_tax_use(self):
        for tax in self:
            if tax.l10n_ar_type_tax_use in ('sale', 'purchase'):
                tax.type_tax_use = tax.l10n_ar_type_tax_use
                tax.l10n_ar_tax_type = False
                tax.l10n_ar_state_id = False
                tax.l10n_ar_withholding_payment_type = False
            else:
                if tax.l10n_ar_type_tax_use in ('supplier', 'customer'):
                    tax.l10n_ar_withholding_payment_type = tax.l10n_ar_type_tax_use
                else:
                    tax.l10n_ar_withholding_payment_type = False
                    tax.l10n_ar_tax_type = False
                tax.type_tax_use = 'none'

```

## File: models\l10n_ar_earnings_scale.py

```python
from odoo import models, fields, api


class L10nArEarningsScale(models.Model):
    _name = 'l10n_ar.earnings.scale'
    _description = 'l10n_ar.earnings.scale'

    name = fields.Char(required=True, translate=True)
    line_ids = fields.One2many('l10n_ar.earnings.scale.line', 'scale_id')


class L10nArEarningsScaleLine(models.Model):
    _name = 'l10n_ar.earnings.scale.line'
    _description = 'l10n_ar.earnings.scale.line'
    _order = 'to_amount'

    scale_id = fields.Many2one(
        'l10n_ar.earnings.scale', required=True, ondelete='cascade',
        help="Calculation of the withholding amount: From the taxable amount (tax base + tax bases applied this month to same tax and partner - non-taxable minimum) subtract the immediately previous amount of the column 'S/ Exceeding $' to detect which row to work with and apply the percentage of said row to the result of the subtraction. Then add to this amount the amount of the '$' column."
    )
    currency_id = fields.Many2one(
        'res.currency', default=lambda self: self.env.ref('base.ARS'), store=False
    )
    from_amount = fields.Monetary(
        string='From $',
        currency_field='currency_id',
        compute="_compute_from_amount"
    )
    to_amount = fields.Monetary(
        string='To $',
        currency_field='currency_id',
        help="The taxable amount (tax base + tax bases applied this month to same tax and partner - non-taxable minimum) must be between the amount in the 'S/ Exced' column. of $' and the amount of this column."
    )
    fixed_amount = fields.Monetary(
        string='$',
        currency_field='currency_id',
        help="To obtain the withholding amount first from the taxable amount (tax base + tax bases applied this month to same tax and partner - non-taxable minimum) subtract the immediately previous amount of 'S/ Exced. of $' column to detect which row to work with and apply the percentage of said row to the result of the subtraction. Then add the amount of this column to the result of applying the percentage."
    )
    percentage = fields.Monetary(
        string='Add %',
        currency_field='currency_id',
        help="Percentage to apply to the result of the subtraction between the taxable amount (tax base + tax basis of the previous month - non-taxable minimum) and the immediately previous amount of 'S/ Exced. from $' column."
    )
    excess_amount = fields.Monetary(
        string='S/ Exceeding $',
        currency_field='currency_id',
        help="From the taxable amount (tax base + tax bases applied this month to same tax and partner - non-taxable minimum) subtract the immediately previous amount of this column to detect which row to work with and apply the percentage of said row to the result of the subtraction."
    )

    @api.depends('to_amount', 'scale_id.line_ids')
    def _compute_from_amount(self):
        for line in self:
            line.from_amount = line.scale_id.line_ids.sorted(reverse=True).filtered(lambda l: l.to_amount < line.to_amount)[:1].to_amount

```

## File: models\l10n_ar_partner_tax.py

```python
from odoo import models, fields, api, _
from odoo.exceptions import ValidationError
import logging
# from dateutil.relativedelta import relativedelta
_logger = logging.getLogger(__name__)


class L10nArPartnerTax(models.Model):
    _name = "l10n_ar.partner.tax"
    _description = "Argentinean Partner Taxes"
    _order = "to_date desc, from_date desc, tax_id"
    _check_company_auto = True
    _check_company_domain = models.check_company_domain_parent_of

    partner_id = fields.Many2one(
        'res.partner',
        required=True,
        ondelete='cascade',
        check_company=True,
    )
    tax_id = fields.Many2one(
        'account.tax',
        required=True,
    )
    company_id = fields.Many2one(
        related='tax_id.company_id', store=True,
    )
    from_date = fields.Date(
        string="From Date"
    )
    to_date = fields.Date(
        string="To Date"
    )
    ref = fields.Char(
        string="ref"
    )

    @api.constrains('from_date', 'to_date')
    def check_partner_tax_dates(self):
        if self.filtered(lambda x: x.from_date and x.to_date and x.from_date >= x.to_date):
            raise ValidationError(_('"From date" must be lower than "To date" on Withholding (AR) taxes.'))

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

## File: models\res_partner.py

```python
from odoo import models, fields


class ResPartner(models.Model):
    _inherit = "res.partner"

    l10n_ar_partner_tax_ids = fields.One2many(
        'l10n_ar.partner.tax',
        'partner_id',
        'Argentinean Withholding Taxes',
    )

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
from . import res_partner
from . import l10n_ar_partner_tax
from . import l10n_ar_earnings_scale

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_ar_payment_register_withholding,access_l10n_ar_payment_register_withholding,model_l10n_ar_payment_register_withholding,account.group_account_invoice,1,1,1,1
access_l10n_ar_partner_tax_all,access_l10n_ar.partner.tax_all,model_l10n_ar_partner_tax,base.group_user,1,0,0,0
access_l10n_ar_partner_tax_manager,access_l10n_ar.partner.tax_manager,model_l10n_ar_partner_tax,account.group_account_manager,1,1,1,1
access_l10n_ar_earnings_scale_manager,access_l10n_ar_earnings_scale_manager,model_l10n_ar_earnings_scale,account.group_account_manager,1,1,1,1
access_l10n_ar_earnings_scale_all,access_l10n_ar_earnings_scale_all,model_l10n_ar_earnings_scale,base.group_user,1,0,0,0
access_l10n_ar_earnings_scale_line_manager,access_l10n_ar_earnings_scale_line_manager,model_l10n_ar_earnings_scale_line,account.group_account_manager,1,1,1,1
access_l10n_ar_earnings_scale_line_all,access_l10n_ar_earnings_scale_line_all,model_l10n_ar_earnings_scale_line,base.group_user,1,0,0,0

```

## File: security\security.xml

```xml
<odoo noupdate="1">

    <record id="l10n_ar_partner_tax_comp_rule" model="ir.rule">
        <field name="name">Argentinean Partner Taxes Company Rule</field>
        <field name="model_id" ref="model_l10n_ar_partner_tax"/>
        <field name="domain_force">[('company_id', 'parent_of', company_ids)]</field>
    </record>

</odoo>

```

## File: views\account_payment_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_account_payment_form" model="ir.ui.view">
        <field name="name">account.payment.form.inherited</field>
        <field name="model">account.payment</field>
        <field name="inherit_id" ref="l10n_latam_check.view_account_payment_form_inherited" />
        <field name="arch" type="xml">
            <page name="latam_checks_page" position="after">
                <page name="withholdings_page" string="Withholdings" invisible="country_code != 'AR'">
                    <field name="l10n_ar_withholding_ids" nolabel="1" colspan="2" readonly="True">
                        <list>
                            <field name="move_name" column_invisible="True"/>
                            <field name="tax_line_id" string="Tax"/>
                            <field name="name" string="Withholding Number" required="1"/>
                            <field name="tax_base_amount"/>
                            <field name="amount_currency" readonly="0" required="1" string="Amount" sum="Total"/>
                        </list>
                    </field>
                </page>
            </page>
        </field>
    </record>
</odoo>

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_tax_form-l10n_ar" model="ir.ui.view">
        <field name="name">account.tax.form.l10n_ar.inherit</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <field name="type_tax_use" position="attributes">
                <attribute name="invisible" separator="or" add="country_code == 'AR'"/>
            </field>
            <field name="amount_type" position="attributes">
                <attribute name="invisible" separator="or" add="l10n_ar_tax_type == 'earnings_scale'"/>
            </field>
            <label for="amount" position="attributes">
                <attribute name="invisible" separator="or" add="l10n_ar_tax_type == 'earnings_scale'"/>
            </label>
            <xpath expr="//field[@name='amount']/.." position="attributes">
                <attribute name="invisible" separator="or" add="l10n_ar_tax_type == 'earnings_scale'"/>
            </xpath>
            <field name="name" position="after">
                <field name="l10n_ar_tax_type" invisible="l10n_ar_type_tax_use not in ('customer', 'supplier')"/>
                <field name="l10n_ar_scale_id" invisible="l10n_ar_tax_type != 'earnings_scale'" required="l10n_ar_tax_type == 'earnings_scale'" options="{'no_create': True}"/>
            </field>
            <field name="type_tax_use" position="after">
                <field name="l10n_ar_type_tax_use" invisible="country_code != 'AR'" string="Tax Type" required="country_code == 'AR'"/>
            </field>
            <xpath expr="//field[@name='amount']/.." position="after">
                <field name="l10n_ar_withholding_payment_type" invisible="1"/><!-- Because of onchange-->
                <field name="l10n_ar_state_id" invisible="l10n_ar_tax_type not in ['iibb_untaxed', 'iibb_total']" required="l10n_ar_tax_type in ['iibb_untaxed', 'iibb_total']" options="{'no_create': True}"/>
                <field name="l10n_ar_non_taxable_amount" invisible="country_code != 'AR' or l10n_ar_type_tax_use != 'supplier'"/>
                <field name="l10n_ar_minimum_threshold" invisible="country_code != 'AR' or l10n_ar_type_tax_use != 'supplier'"/>
                <field name="l10n_ar_withholding_sequence_id" context="{'default_name': name}" invisible="l10n_ar_withholding_payment_type != 'supplier'"/>
                <field name="l10n_ar_code" invisible="l10n_ar_tax_type not in ['earnings', 'earnings_scale'] or l10n_ar_type_tax_use != 'supplier'" required="l10n_ar_tax_type in ['earnings', 'earnings_scale'] and l10n_ar_type_tax_use == 'supplier'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\l10n_ar_earnings_scale_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_afip_earnings_table_scale_tree" model="ir.ui.view">
        <field name="name">l10n_ar.earnings.scale.tree</field>
        <field name="model">l10n_ar.earnings.scale</field>
        <field name="arch" type="xml">
            <list>
                <field name="name"/>
            </list>
        </field>
    </record>

    <record id="view_afip_earnings_table_scale_form" model="ir.ui.view">
        <field name="name">l10n_ar.earnings.scale.form</field>
        <field name="model">l10n_ar.earnings.scale</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                    <field name="name"/>
                    <field name="line_ids">
                        <list editable="bottom">
                            <field name="from_amount"/>
                            <field name="to_amount"/>
                            <field name="fixed_amount"/>
                            <field name="percentage"/>
                            <field name="excess_amount"/>
                        </list>
                    </field>
                    </group>
                    <b><u>Formula Earnings with Scale</u></b><br/>
                    <div>
                        <ul>
                            <li>Base amount = Tax base + tax bases applied this month to the same tax and partner - non-taxable amount</li>
                            <li>Withholding amount = (Base amount - immediately previous amount of the column 'S/ Exceeding $') * row percentage / 100 + row fixed amount ('$' column)</li>
                            <li>If the base amount is lower than 0.0 then the withholding is 0.0</li>
                            <li>If the withholding amount is lower than the minimum threshold on the tax then the final withholding amount is 0.0</li>
                        </ul>
                    </div>
                    <b><u>AFIP Sources</u></b><br/>
                    <span>Calculator of the Withholding Amount: </span>
                    <a href="https://servicioscf.afip.gob.ar/calc-rg830/" target="_blank">AFIP Calculator</a><br/>
                    <span>Link to AFIP aditional info: </span>
                    <a href="https://servicioscf.afip.gob.ar/publico/abc/ABCpaso2.aspx?id_nivel1=3269&amp;id_nivel2=3304&amp;p=R%C3%A9g.%20General%20de%20Retenci%C3%B3n%20del%20Impuesto%20a%20las%20Ganancias%20-%20RG%20830" target="_blank">AFIP Additional info</a>
                </sheet>
            </form>
        </field>
    </record>

    <record model="ir.actions.act_window" id="act_afip_earnings_table_scale">
        <field name="name">AFIP tax</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">l10n_ar.earnings.scale</field>
        <field name="view_mode">list,form</field>
        <field name="view_id" ref="view_afip_earnings_table_scale_tree"/>
    </record>

    <menuitem name="Earnings Scale" action="act_afip_earnings_table_scale" id="menu_action_afip_earnings_table_scale_line" sequence="95" parent="l10n_ar.menu_afip_config"/>

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
                                <tr>
                                    <td>
                                        <span t-field='line.tax_line_id.name'/>
                                    </td>
                                    <td>
                                        <span t-field='line.name'/>
                                    </td>
                                    <td class="text-end">
                                        <span t-out="abs(line.tax_base_amount)" t-options="{'widget': 'monetary', 'display_currency': o.currency_id}"/>
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

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="view_partner_form">
        <field name="name">res.partner.form.inherit</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="model">res.partner</field>
        <field name="arch" type="xml">
            <group id="invoice_send_settings" position="after">
                <group string="Purchase Withholding" invisible="'AR' not in fiscal_country_codes or not is_company">
                    <field name="l10n_ar_partner_tax_ids" nolabel="1" colspan="2" widget="auto_save_res_partner">
                        <list editable="top">
                            <field name="tax_id" options="{'no_create': True, 'no_open': True}" domain="[('l10n_ar_withholding_payment_type', '=', 'supplier')]"/>
                            <field name="from_date" optional="hide"/>
                            <field name="to_date" optional="hide"/>
                            <field name="ref" optional="hide"/>
                        </list>
                    </field>
                </group>
            </group>
        </field>
    </record>

</odoo>

```

## File: wizards\account_payment_register.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from odoo import models, fields, api, Command, _
from odoo.exceptions import ValidationError
from odoo.exceptions import UserError
from datetime import datetime

_logger = logging.getLogger(__name__)


class AccountPaymentRegister(models.TransientModel):
    _inherit = 'account.payment.register'

    l10n_ar_withholding_ids = fields.One2many(
        'l10n_ar.payment.register.withholding', 'payment_register_id', string="Withholdings",
        compute="_compute_l10n_ar_withholding_ids", readonly=False, store=True)
    l10n_ar_net_amount = fields.Monetary(compute='_compute_l10n_ar_net_amount', readonly=True, help="Net amount after withholdings")
    l10n_ar_adjustment_warning = fields.Boolean(compute="_compute_l10n_ar_adjustment_warning")

    @api.depends('l10n_latam_move_check_ids.amount', 'amount', 'l10n_ar_net_amount', 'l10n_latam_new_check_ids.amount', 'payment_method_code')
    def _compute_l10n_ar_adjustment_warning(self):
        wizard_register = self
        for wizard in self:
            checks = wizard.l10n_latam_new_check_ids if wizard.filtered(lambda x: x._is_latam_check_payment(check_subtype='new_check')) else wizard.l10n_latam_move_check_ids
            checks_amount = sum(checks.mapped('amount'))
            if checks_amount and wizard.l10n_ar_net_amount != checks_amount:
                wizard.l10n_ar_adjustment_warning = True
                wizard_register -= wizard
        wizard_register.l10n_ar_adjustment_warning = False

    @api.depends('amount', 'l10n_ar_withholding_ids.amount')
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
            # create withholding amount applied move line only if amount != 0
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
                'name': nice_base_label,
                'tax_ids': [Command.set(withholding_lines.mapped('tax_id').ids)],
                'account_id': account_id,
                'balance': cc_base_amount,
                'amount_currency': base_amount,
            })
            payment_vals['write_off_line_vals'].append({
                'currency_id': self.currency_id.id,  # Counterpart 0 operation
                'name': nice_base_label,
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

    @api.depends('partner_id', 'payment_date')
    def _compute_l10n_ar_withholding_ids(self):
        for wizard in self:
            date = wizard.payment_date or fields.Date.context_today(self)
            partner_taxes = self.env['l10n_ar.partner.tax'].search([
                *self.env['l10n_ar.partner.tax']._check_company_domain(wizard.company_id),
                '|', ('from_date', '>=', date), ('from_date', '=', False),
                '|', ('to_date', '<=', date), ('to_date', '=', False),
                ('partner_id', '=', wizard.partner_id.commercial_partner_id.id),
                ('tax_id.l10n_ar_withholding_payment_type', '=', wizard.partner_type)
            ])
            wizard.l10n_ar_withholding_ids = [Command.clear()] + [Command.create({'tax_id': x.tax_id.id}) for x in partner_taxes]

    def action_create_payments(self):
        if self.l10n_ar_withholding_ids and not self.payment_method_line_id.payment_account_id:
            raise ValidationError(_("A payment cannot have withholding if the payment method has no outstanding accounts"))
        return super().action_create_payments()

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
            <page name="latam_checks_page" position="after">
                <page name="withholdings_page" string="Withholdings" invisible="country_code != 'AR'">
                    <field name="l10n_ar_withholding_ids" nolabel="1" colspan="2">
                        <list editable="bottom">
                            <field name="withholding_sequence_id" column_invisible="True"/>
                            <field name="company_id" column_invisible="True"/>
                            <field name="currency_id" column_invisible="True"/>
                            <field name="tax_id" options="{'no_open': True, 'no_create': True}"/>
                            <field name="name" readonly="withholding_sequence_id"/>
                            <field name="base_amount"/>
                            <field name="amount"/>
                        </list>
                    </field>
                </page>
            </page>
            <group>
                <group colspan="2" invisible="country_code != 'AR' or not can_edit_wizard or (can_group_payments and not group_payment)">
                    <label for="l10n_ar_net_amount" string="Net Amount" invisible="l10n_latam_move_check_ids or l10n_latam_new_check_ids"/>
                    <label for="l10n_ar_net_amount" string="Check amount" invisible="not l10n_latam_move_check_ids and not l10n_latam_new_check_ids"/>
                    <field name="l10n_ar_net_amount" nolabel="1" />
                    <field name="l10n_ar_adjustment_warning" invisible="True"/>
                    <p colspan="2" invisible="not l10n_ar_adjustment_warning" class="alert alert-warning" role="alert">
                            Adjust total amount or withholdings amount so that the check amount is the correct one.
                    </p>
                </group>
            </group>
            <div role="alert" position="after">
                <div role="alert" class="alert alert-info"
                        invisible="country_code != 'AR' or can_edit_wizard and (not can_group_payments or can_group_payments and group_payment)">
                    <p>You can't register withholdings when paying invoices of different partners or same partner without grouping</p>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

## File: wizards\l10n_ar_payment_register_withholding.py

```python
# pylint: disable=protected-access
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging

from dateutil.relativedelta import relativedelta
from datetime import datetime
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
    base_amount = fields.Monetary(required=True, compute='_compute_base_amount', store=True, readonly=False)
    amount = fields.Monetary(required=True, compute='_compute_amount', store=True, readonly=False)

    def _tax_compute_all_helper(self):
        self.ensure_one()
        # Computes the withholding tax amount provided a base and a tax
        # It is equivalent to: amount = self.base * self.tax_id.amount / 100

        # if it is earnings withholding, then we accumulate the tax base for the period
        if self.tax_id.l10n_ar_tax_type in ['earnings', 'earnings_scale']:
            to_date = self.payment_register_id.payment_date or datetime.date.today()
            from_date = to_date + relativedelta(day=1)
            # We search for the payments in the same month of the same regimen and the same code.
            domain_same_period_withholdings = [
                *self.env['account.move.line']._check_company_domain(self.tax_id.company_id),
                ('parent_state', '=', 'posted'),
                ('tax_line_id.l10n_ar_code', '=', self.tax_id.l10n_ar_code),
                ('tax_line_id.l10n_ar_tax_type', 'in', ['earnings', 'earnings_scale']),
                ('partner_id', '=', self.payment_register_id.partner_id.commercial_partner_id.id),
                ('date', '<=', to_date), ('date', '>=', from_date)]
            if same_period_partner_withholdings := self.env['account.move.line']._read_group(domain_same_period_withholdings, ['partner_id'], ['balance:sum']):
                same_period_withholdings = abs(same_period_partner_withholdings[0][1])
            else:
                same_period_withholdings = 0.0
            domain_same_period_base = [
                *self.env['account.move.line']._check_company_domain(self.tax_id.company_id),
                ('parent_state', '=', 'posted'),
                ('tax_ids.l10n_ar_code', '=', self.tax_id.l10n_ar_code),
                ('tax_ids.l10n_ar_tax_type', 'in', ['earnings', 'earnings_scale']),
                ('partner_id', '=', self.payment_register_id.partner_id.commercial_partner_id.id),
                ('date', '<=', to_date), ('date', '>=', from_date)]
            if same_period_partner_base := self.env['account.move.line']._read_group(domain_same_period_base, ['partner_id'], ['balance:sum']):
                same_period_base = abs(same_period_partner_base[0][1])
            else:
                same_period_base = 0.0
            net_amount = self.base_amount + same_period_base
        else:
            net_amount = self.base_amount
        net_amount = max(0, net_amount - self.tax_id.l10n_ar_non_taxable_amount)
        taxes_res = self.tax_id.compute_all(
            net_amount,
            currency=self.payment_register_id.currency_id,
            quantity=1.0,
            product=False,
            partner=False,
            is_refund=False,
        )
        tax_amount = taxes_res['taxes'][0]['amount']
        tax_account_id = taxes_res['taxes'][0]['account_id']
        tax_repartition_line_id = taxes_res['taxes'][0]['tax_repartition_line_id']

        if self.tax_id.l10n_ar_tax_type in ['earnings', 'earnings_scale']:
            # if it is earnings scale we calculate according to the scale.
            if self.tax_id.l10n_ar_tax_type == 'earnings_scale':
                escala = self.env['l10n_ar.earnings.scale.line'].search([
                    ('scale_id', '=', self.tax_id.l10n_ar_scale_id.id),
                    ('excess_amount', '<=', net_amount),
                    ('to_amount', '>', net_amount),
                ], limit=1)
                tax_amount = ((net_amount - escala.excess_amount) * escala.percentage / 100) + escala.fixed_amount
            # deduct withholdings from the same period
            tax_amount -= same_period_withholdings

        l10n_ar_minimum_threshold = self.tax_id.l10n_ar_minimum_threshold
        if l10n_ar_minimum_threshold > tax_amount:
            tax_amount = 0.0
        return tax_amount, tax_account_id, tax_repartition_line_id

    @api.depends('base_amount', 'tax_id')
    def _compute_amount(self):
        for line in self:
            if not line.tax_id:
                line.amount = 0.0
            else:
                line.amount = line._tax_compute_all_helper()[0]

    @api.depends('payment_register_id.amount', 'tax_id')
    def _compute_base_amount(self):
        for wth in self:
            if wth.tax_id.l10n_ar_tax_type == 'iibb_total':
                wth.base_amount = wth.payment_register_id.amount
            else:
                wth.base_amount = wth.payment_register_id.amount * sum(wth.payment_register_id.line_ids.mapped('move_id.amount_untaxed')) / sum(wth.payment_register_id.line_ids.mapped("move_id.amount_total"))

```

## File: wizards\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_payment_register
from . import l10n_ar_payment_register_withholding

```

