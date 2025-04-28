# Odoo Module: l10n_cl

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from . import demo

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
# Copyright (c) 2019 - Blanco Martín & Asociados. https://www.bmya.cl
{
    'name': 'Chile - Accounting',
    'version': "3.0",
    'description': """
Chilean accounting chart and tax localization.
Plan contable chileno e impuestos de acuerdo a disposiciones vigentes
    """,
    'author': 'Blanco Martín & Asociados',
    'website': 'https://www.odoo.com/documentation/15.0/applications/finance/accounting/fiscal_localizations/localizations/chile.html',
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'contacts',
        'base_address_city',
        'base_vat',
        'l10n_latam_base',
        'l10n_latam_invoice_document',
        'uom',
        ],
    'data': [
        'views/account_move_view.xml',
        'views/account_tax_view.xml',
        'views/res_bank_view.xml',
        'views/res_country_view.xml',
        'views/report_invoice.xml',
        'views/res_partner.xml',
        'views/res_config_settings_view.xml',
        'data/l10n_cl_chart_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_tags_data.xml',
        'data/account_tax_data.xml',
        'data/l10n_latam_identification_type_data.xml',
        'data/l10n_latam.document.type.csv',
        'data/menuitem_data.xml',
        'data/product_data.xml',
        'data/uom_data.xml',
        'data/res.currency.csv',
        'data/res_currency_data.xml',
        'data/res.bank.csv',
        'data/res.country.csv',
        'data/res_partner.xml',
        'data/account_fiscal_template.xml',
        'data/account_chart_template_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
        'demo/partner_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_cl.cl_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_fiscal_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="afpt_non_recoverable_vat_1" model="account.fiscal.position.template">
            <field name="name">Compras - destinadas a generar operaciones no gravadas o exentas</field>
            <field name="chart_template_id" ref="l10n_cl.cl_chart_template"/>
        </record>
        <record id="afpt_non_recoverable_vat_1_tax" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="l10n_cl.afpt_non_recoverable_vat_1"/>
            <field name="tax_src_id" ref="l10n_cl.OTAX_19"/>
            <field name="tax_dest_id" ref="l10n_cl.iva_compra_no_recup"/>
        </record>

        <record id="afpt_non_recoverable_vat_2" model="account.fiscal.position.template">
            <field name="name">Compras - Facturas de proveedores registrados fuera de plazo</field>
            <field name="chart_template_id" ref="l10n_cl.cl_chart_template"/>
        </record>
        <record id="afpt_non_recoverable_vat_2_tax" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="afpt_non_recoverable_vat_2"/>
            <field name="tax_src_id" ref="l10n_cl.OTAX_19"/>
            <field name="tax_dest_id" ref="l10n_cl.iva_compra_no_recup"/>
        </record>

        <record id="afpt_non_recoverable_vat_3" model="account.fiscal.position.template">
            <field name="name">Compras - Gastos rechazados</field>
            <field name="chart_template_id" ref="l10n_cl.cl_chart_template"/>
        </record>
        <record id="afpt_non_recoverable_vat_3_tax" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="afpt_non_recoverable_vat_3"/>
            <field name="tax_src_id" ref="l10n_cl.OTAX_19"/>
            <field name="tax_dest_id" ref="l10n_cl.iva_compra_no_recup"/>
        </record>

        <record id="afpt_non_recoverable_vat_4" model="account.fiscal.position.template">
            <field name="name">Compras - Entregas gratuitas (premios, bonificaciones, etc.) recibidos</field>
            <field name="chart_template_id" ref="l10n_cl.cl_chart_template"/>
        </record>
        <record id="afpt_non_recoverable_vat_4_tax" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="afpt_non_recoverable_vat_4"/>
            <field name="tax_src_id" ref="l10n_cl.OTAX_19"/>
            <field name="tax_dest_id" ref="l10n_cl.iva_compra_no_recup"/>
       </record>
        <record id="afpt_non_recoverable_vat_4_account_1" model="account.fiscal.position.account.template">
            <field name="position_id" ref="afpt_non_recoverable_vat_4"/>
            <field name="account_src_id" ref="l10n_cl.account_410235"/>
            <field name="account_dest_id" ref="l10n_cl.account_410165"/>
        </record>
        <record id="afpt_non_recoverable_vat_4_account_2" model="account.fiscal.position.account.template">
            <field name="position_id" ref="afpt_non_recoverable_vat_4"/>
            <field name="account_src_id" ref="l10n_cl.account_410230"/>
            <field name="account_dest_id" ref="l10n_cl.account_410165"/>
       </record>

        <record id="afpt_non_recoverable_vat_9" model="account.fiscal.position.template">
            <field name="name">Compras - Otros</field>
            <field name="chart_template_id" ref="l10n_cl.cl_chart_template"/>
        </record>
        <record id="afpt_non_recoverable_vat_9_tax" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="afpt_non_recoverable_vat_9"/>
            <field name="tax_src_id" ref="l10n_cl.OTAX_19"/>
            <field name="tax_dest_id" ref="l10n_cl.iva_compra_no_recup"/>
        </record>

        <record id="afpt_fixed_asset" model="account.fiscal.position.template">
            <field name="name">Compras - Activo Fijo</field>
            <field name="chart_template_id" ref="l10n_cl.cl_chart_template"/>
        </record>
        <record id="afpt_fixed_asset_tax" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="afpt_fixed_asset"/>
            <field name="tax_src_id" ref="l10n_cl.OTAX_19"/>
            <field name="tax_dest_id" ref="l10n_cl.iva_activo_fijo_uso_comun"/>
        </record>
        <record id="afpt_fixed_asset_account_1" model="account.fiscal.position.account.template">
            <field name="position_id" ref="afpt_fixed_asset"/>
            <field name="account_src_id" ref="l10n_cl.account_410230"/>
            <field name="account_dest_id" ref="l10n_cl.account_121140"/>
        </record>
        <record id="afpt_fixed_asset_account_2" model="account.fiscal.position.account.template">
            <field name="position_id" ref="afpt_fixed_asset"/>
            <field name="account_src_id" ref="l10n_cl.account_410235"/>
            <field name="account_dest_id" ref="l10n_cl.account_121140"/>
        </record>

        <record id="afpt_purchase_exempt" model="account.fiscal.position.template">
            <field name="name">Compras - Exentas</field>
            <field name="chart_template_id" ref="l10n_cl.cl_chart_template"/>
        </record>
        <record id="afpt_purchase_exempt_tax" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="afpt_purchase_exempt"/>
            <field name="tax_src_id" ref="l10n_cl.OTAX_19"/>
        </record>
        <record id="afpt_purchase_exempt_account" model="account.fiscal.position.account.template">
            <field name="position_id" ref="afpt_purchase_exempt"/>
            <field name="account_src_id" ref="l10n_cl.account_410230"/>
            <field name="account_dest_id" ref="l10n_cl.account_410130"/>
        </record>

        <record id="afpt_purchase_supermarket" model="account.fiscal.position.template">
            <field name="name">Compras - Supermercado</field>
            <field name="chart_template_id" ref="l10n_cl.cl_chart_template"/>
        </record>
        <record id="afpt_purchase_supermarket_tax" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="afpt_purchase_supermarket"/>
            <field name="tax_src_id" ref="l10n_cl.OTAX_19"/>
            <field name="tax_dest_id" ref="l10n_cl.iva_supermercado_recup"/>
        </record>
        <record id="afpt_purchase_supermarket_account" model="account.fiscal.position.account.template">
            <field name="position_id" ref="afpt_purchase_supermarket"/>
            <field name="account_src_id" ref="l10n_cl.account_410230"/>
            <field name="account_dest_id" ref="l10n_cl.account_410233"/>
        </record>
        <record id="afpt_purchase_supermarket_account_1" model="account.fiscal.position.account.template">
            <field name="position_id" ref="afpt_purchase_supermarket"/>
            <field name="account_src_id" ref="l10n_cl.account_410235"/>
            <field name="account_dest_id" ref="l10n_cl.account_410233"/>
        </record>

        <record id="afpt_sale_exempt" model="account.fiscal.position.template">
            <field name="name">Ventas - Exentas</field>
            <field name="chart_template_id" ref="l10n_cl.cl_chart_template"/>
        </record>
        <record id="afpt_sale_exempt_tax" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="afpt_sale_exempt"/>
            <field name="tax_src_id" ref="l10n_cl.ITAX_19"/>
        </record>
        <record id="afpt_sale_exempt_account" model="account.fiscal.position.account.template">
            <field name="position_id" ref="afpt_sale_exempt"/>
            <field name="account_src_id" ref="l10n_cl.account_310115"/>
            <field name="account_dest_id" ref="l10n_cl.account_310120"/>
        </record>

    </data>
</odoo>
```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Account Tags -->

    <record id="ITAX_19" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">IVA 19% Venta</field>
        <field name="description">IVA 19% Vta</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="l10n_cl_sii_code">14</field>
        <field name="tax_group_id" ref="tax_group_iva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_ventas_netas_gravadas_c_iva')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_210710'),
                'plus_report_line_ids': [ref('tax_report_iva_debito_fiscal')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_ventas_netas_gravadas_c_iva')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_210710'),
                'minus_report_line_ids': [ref('tax_report_iva_debito_fiscal')],
            }),
        ]"/>
    </record>

    <record id="OTAX_19" model="account.tax.template">
      <field name="chart_template_id" ref="cl_chart_template"/>
      <field name="name">IVA 19% Compra</field>
      <field name="description">IVA 19% Comp</field>
      <field name="amount">19</field>
      <field name="amount_type">percent</field>
      <field name="type_tax_use">purchase</field>
      <field name="l10n_cl_sii_code">14</field>
      <field name="tax_group_id" ref="tax_group_iva_19"/>
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_compras_netas_gr_iva_recup')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_110710'),
                'plus_report_line_ids': [ref('tax_report_compras_iva_recup')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_compras_netas_gr_iva_recup')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_110710'),
                'minus_report_line_ids': [ref('tax_report_compras_iva_recup')],
            }),
        ]"/>
    </record>

    <record id="I_IU2C" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Ret. 2da Categoría</field>
        <field name="description">Ret. 2da Cat</field>
        <field name="amount">-10.75</field>
        <field name="sequence" eval="2"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="l10n_cl_sii_code">15</field>
        <field name="tax_group_id" ref="tax_group_2da_categ"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_retencion_segunda_categ')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_210740'),
                'plus_report_line_ids': [ref('tax_report_retencion_segunda_categ')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_retencion_segunda_categ')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_210740'),
                'minus_report_line_ids': [ref('tax_report_retencion_segunda_categ')],
            }),
        ]"/>
    </record>

    <record id="I_RTI" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Retención Total IVA</field>
        <field name="description">Retención total IVA</field>
        <field name="amount">-19.00</field>
        <field name="sequence" eval="2"/>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="l10n_cl_sii_code">15</field>
        <field name="tax_group_id" ref="tax_group_retenciones"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_retencion_total_compras')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_210715'),
                'plus_report_line_ids': [ref('tax_report_retencion_total_compras')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_retencion_total_compras')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_210715'),
                'minus_report_line_ids': [ref('tax_report_retencion_total_compras')],
            }),
        ]"/>
    </record>

    <record id="especifico_compra" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Específico Compra</field>
        <field name="description">Espec. Comp</field>
        <field name="amount">63</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="l10n_cl_sii_code">29</field>
        <field name="sequence" eval="5"/>
        <field name="tax_group_id" ref="tax_group_impuestos_especificos"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('account_420220'),
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
                'account_id': ref('account_420220'),
            }),
        ]"/>
    </record>

    <record id="iva_activo_fijo" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">IVA Compra 19% Activo Fijo</field>
        <field name="description">IVA 19% ActF</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="l10n_cl_sii_code">14</field>
        <field name="sequence" eval="6"/>
        <field name="tax_group_id" ref="tax_group_iva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_compras_activo_fijo')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_compras_iva_activo_fijo')],
                'account_id': ref('account_110730'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_compras_activo_fijo')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_compras_iva_activo_fijo')],
                'account_id': ref('account_110730'),
            }),
        ]"/>
    </record>

    <record id="iva_activo_fijo_uso_comun" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">IVA Compra 19% Act. Fijo Uso Común</field>
        <field name="description">IVA 19% ActFUC</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="l10n_cl_sii_code">14</field>
        <field name="sequence" eval="6"/>
        <field name="tax_group_id" ref="tax_group_iva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_compras_activo_fijo_uso_comun')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_compras_iva_activo_fijo_uso_comun')],
                'account_id': ref('account_110730'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_compras_activo_fijo_uso_comun')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_compras_iva_activo_fijo_uso_comun')],
                'account_id': ref('account_110730'),
            }),
        ]"/>
    </record>

    <record id="iva_activo_fijo_uso_no_recup" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">IVA Compra 19% Activo Fijo No Recup</field>
        <field name="description">IVA 19% ActFNR</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="l10n_cl_sii_code">14</field>
        <field name="sequence" eval="6"/>
        <field name="tax_group_id" ref="tax_group_iva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_compras_activo_fijo_no_recup')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_compras_iva_activo_fijo_no_recup')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_compras_activo_fijo_no_recup')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_compras_iva_activo_fijo_no_recup')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
    </record>

    <record id="ila_a_100_p" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Beb. Analc. 10% (Compras)</field>
        <field name="description">ILA C 10%</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="l10n_cl_sii_code">27</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="7"/>
        <field name="tax_group_id" ref="tax_group_ila"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_ila_compras')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_tax_ila_compras')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_ila_compras')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_tax_ila_compras')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
    </record>

    <record id="ila_a_180_p" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Beb. Analc 18% (Compras)</field>
        <field name="description">ILA C 18%</field>
        <field name="amount">18</field>
        <field name="amount_type">percent</field>
        <field name="l10n_cl_sii_code">26</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="7"/>
        <field name="tax_group_id" ref="tax_group_ila"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_ila_compras')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_tax_ila_compras')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_ila_compras')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_tax_ila_compras')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
    </record>

    <record id="ila_v_205_p" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Vinos (Compras)</field>
        <field name="description">ILA C 20.5%</field>
        <field name="amount">20.5</field>
        <field name="amount_type">percent</field>
        <field name="l10n_cl_sii_code">25</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="7"/>
        <field name="tax_group_id" ref="tax_group_ila"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_ila_compras')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_tax_ila_compras')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_ila_compras')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_tax_ila_compras')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
    </record>

    <record id="ila_l_315_p" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Licores 31.5% (Compras)</field>
        <field name="description">ILA C 31.5%</field>
        <field name="amount">31.5</field>
        <field name="amount_type">percent</field>
        <field name="l10n_cl_sii_code">24</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="7"/>
        <field name="tax_group_id" ref="tax_group_ila"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_ila_compras')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_tax_ila_compras')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_ila_compras')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_tax_ila_compras')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
    </record>

    <record id="ila_a_100_s" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Beb. Analc. 10% (Ventas)</field>
        <field name="description">ILA V 10%</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="l10n_cl_sii_code">27</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="7"/>
        <field name="tax_group_id" ref="tax_group_ila"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_ila_ventas')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_tax_ila_ventas')],
                'account_id': ref('account_210760'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_ila_ventas')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_tax_ila_ventas')],
                'account_id': ref('account_210760'),
            }),
        ]"/>
    </record>

    <record id="ila_a_180_s" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Beb. Analc 18% (Ventas)</field>
        <field name="description">ILA V 18%</field>
        <field name="amount">18</field>
        <field name="amount_type">percent</field>
        <field name="l10n_cl_sii_code">26</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="7"/>
        <field name="tax_group_id" ref="tax_group_ila"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_ila_ventas')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_tax_ila_ventas')],
                'account_id': ref('account_210760'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_ila_ventas')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_tax_ila_ventas')],
                'account_id': ref('account_210760'),
            }),
        ]"/>
    </record>

    <record id="ila_v_205_s" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Vinos (Ventas)</field>
        <field name="description">ILA V 20.5%</field>
        <field name="amount">20.5</field>
        <field name="amount_type">percent</field>
        <field name="l10n_cl_sii_code">25</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="7"/>
        <field name="tax_group_id" ref="tax_group_ila"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_ila_ventas')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_tax_ila_ventas')],
                'account_id': ref('account_210760'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_ila_ventas')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_tax_ila_ventas')],
                'account_id': ref('account_210760'),
            }),
        ]"/>
    </record>

    <record id="ila_l_315_s" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">Licores 31.5% (Ventas)</field>
        <field name="description">ILA V 31.5%</field>
        <field name="amount">31.5</field>
        <field name="amount_type">percent</field>
        <field name="l10n_cl_sii_code">24</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="7"/>
        <field name="tax_group_id" ref="tax_group_ila"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_base_ila_ventas')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_tax_ila_ventas')],
                'account_id': ref('account_210760'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_base_ila_ventas')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_tax_ila_ventas')],
                'account_id': ref('account_210760'),
            }),
        ]"/>
    </record>

    <record id="iva_compra_no_recup" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">IVA Compra 19% No Recup.</field>
        <field name="description">IVA 19% NoR</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="l10n_cl_sii_code">14</field>
        <field name="sequence" eval="6"/>
        <field name="tax_group_id" ref="tax_group_iva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_compras_netas_gr_iva_no_recuperable')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_compras_iva_no_recup')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_compras_netas_gr_iva_no_recuperable')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_compras_iva_no_recup')],
                'account_id': ref('account_420220'),
            }),
        ]"/>
    </record>

    <record id="iva_compra_uso_comun" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">IVA Compra 19% Uso Común</field>
        <field name="description">IVA 19% CUC</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="l10n_cl_sii_code">14</field>
        <field name="sequence" eval="6"/>
        <field name="tax_group_id" ref="tax_group_iva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_compras_netas_gr_iva_uso_comun')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_compras_iva_uso_comun')],
                'account_id': ref('account_110730'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_compras_netas_gr_iva_uso_comun')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_compras_iva_uso_comun')],
                'account_id': ref('account_110730'),
            }),
        ]"/>
    </record>

    <record id="iva_supermercado_recup" model="account.tax.template">
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="name">IVA Compra 19% Superm.</field>
        <field name="description">IVA 19% SupMRec</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="l10n_cl_sii_code">14</field>
        <field name="sequence" eval="6"/>
        <field name="tax_group_id" ref="tax_group_iva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_compras_supermercado')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_compras_iva_supermercado')],
                'account_id': ref('account_110710'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_compras_supermercado')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_compras_iva_supermercado')],
                'account_id': ref('account_110710'),
            }),
        ]"/>
    </record>

</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_iva_19" model="account.tax.group">
            <field name="name">IVA 19%</field>
            <field name="country_id" ref="base.cl"/>
        </record>
        <record id="tax_group_impuestos_especificos" model="account.tax.group">
            <field name="name">Impuestos Específicos</field>
            <field name="country_id" ref="base.cl"/>
        </record>
        <record id="tax_group_ila" model="account.tax.group">
            <field name="name">ILA</field>
            <field name="country_id" ref="base.cl"/>
        </record>
        <record id="tax_group_2da_categ" model="account.tax.group">
            <field name="name">Retención de 2da Categoría</field>
            <field name="country_id" ref="base.cl"/>
        </record>
        <record id="tax_group_retenciones" model="account.tax.group">
            <field name="name">Retenciones</field>
            <field name="country_id" ref="base.cl"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.cl"/>
    </record>

    <record id="tax_report_base_imponible_ventas" model="account.tax.report.line">
        <field name="name">Base Imponible Ventas</field>
        <field name="tag_name">Base Imponible Ventas</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_ventas_netas_gravadas_c_iva" model="account.tax.report.line">
        <field name="name">Ventas Netas Gravadas con IVA</field>
        <field name="tag_name">Ventas Netas Gravadas con IVA</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_ventas_exentas" model="account.tax.report.line">
        <field name="name">Ventas Exentas</field>
        <field name="tag_name">Ventas Exentas</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_base_imponible_ventas"/>
    </record>

    <record id="tax_report_impuestos_renta" model="account.tax.report.line">
        <field name="name">Impuesto a la Renta Primera Categoría a Pagar</field>
        <field name="tag_name">Impuesto a la Renta Primera Categoría a Pagar</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_base_imponible_ventas"/>
    </record>

    <record id="tax_report_retencion_total_compras" model="account.tax.report.line">
        <field name="name">Retención Total (compras)</field>
        <field name="tag_name">Retención Total (compras)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_impuestos_originados_venta" model="account.tax.report.line">
        <field name="name">Impuesto Originado por la Venta</field>
        <field name="tag_name">Impuesto Originado por la Venta</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_iva_debito_fiscal" model="account.tax.report.line">
        <field name="name">IVA Debito Fiscal</field>
        <field name="tag_name">IVA Debito Fiscal</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_ppm" model="account.tax.report.line">
        <field name="name">PPM</field>
        <field name="tag_name">PPM</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_netas_gr_iva_recup" model="account.tax.report.line">
        <field name="name">Compras Netas Gravadas Con IVA (recuperable)</field>
        <field name="tag_name">Compras Netas Gravadas Con IVA (recuperable)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_netas_gr_iva_uso_comun" model="account.tax.report.line">
        <field name="name">Compra Netas Gravadas Con IVA Uso Comun</field>
        <field name="tag_name">Compra Netas Gravadas Con IVA Uso Comun</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_netas_gr_iva_no_recuperable" model="account.tax.report.line">
        <field name="name">Compras IVA No Recuperable</field>
        <field name="tag_name">Compras IVA No Recuperable</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_supermercado" model="account.tax.report.line">
        <field name="name">Compras De Supermercado</field>
        <field name="tag_name">Compras De Supermercado</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_activo_fijo" model="account.tax.report.line">
        <field name="name">Compras de Activo Fijo</field>
        <field name="tag_name">Compras de Activo Fijo</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_activo_fijo_uso_comun" model="account.tax.report.line">
        <field name="name">Compras de Activo Fijo Uso Común</field>
        <field name="tag_name">Compras de Activo Fijo Uso Común</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_activo_fijo_no_recup" model="account.tax.report.line">
        <field name="name">Compras de Activo Fijo No Recuperable</field>
        <field name="tag_name">Compras de Activo Fijo No Recuperable</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_no_gravadas_iva" model="account.tax.report.line">
        <field name="name">Compras No Gravadas Con IVA</field>
        <field name="tag_name">Compras No Gravadas Con IVA</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_impuestos_pagados_compra" model="account.tax.report.line">
        <field name="name">Impuestos Pagados en la Compra</field>
        <field name="tag_name">Impuestos Pagados en la Compra</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_iva_recup" model="account.tax.report.line">
        <field name="name">IVA Pagado Compras Recuperables</field>
        <field name="tag_name">IVA Pagado Compras Recuperables</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_iva_uso_comun" model="account.tax.report.line">
        <field name="name">IVA Pagado Compras Uso Común</field>
        <field name="tag_name">IVA Pagado Compras Uso Común</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_iva_no_recup" model="account.tax.report.line">
        <field name="name">IVA Pagado No Recuperable</field>
        <field name="tag_name">IVA Pagado No Recuperable</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_iva_supermercado" model="account.tax.report.line">
        <field name="name">IVA Pagado Compras Supermercado</field>
        <field name="tag_name">IVA Pagado Compras Supermercado</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_iva_activo_fijo" model="account.tax.report.line">
        <field name="name">Compras Activo Fijo</field>
        <field name="tag_name">Compras Activo Fijo</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_iva_activo_fijo_uso_comun" model="account.tax.report.line">
        <field name="name">Compras Activo Fijo Uso Común</field>
        <field name="tag_name">Compras Activo Fijo Uso Común</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_compras_iva_activo_fijo_no_recup" model="account.tax.report.line">
        <field name="name">Compras Activo Fijo No Recuperables</field>
        <field name="tag_name">Compras Activo Fijo No Recuperables</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_retencion_segunda_categ" model="account.tax.report.line">
        <field name="name">Retención Segunda Categoría</field>
        <field name="tag_name">Retención Segunda Categoría</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_base_retencion_segunda_categ" model="account.tax.report.line">
        <field name="name">Base Retención Segunda Categoría</field>
        <field name="tag_name">Base Retención Segunda Categoría</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_base_ila_compras" model="account.tax.report.line">
        <field name="name">Base Retenciones ILA (compras)</field>
        <field name="tag_name">Base Retenciones ILA (compras)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_tax_ila_compras" model="account.tax.report.line">
        <field name="name">Impuesto Ret Sufrida ILA (compras)</field>
        <field name="tag_name">Retenciones ILA (compras)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_base_ila_ventas" model="account.tax.report.line">
        <field name="name">Base Retenciones ILA (ventas)</field>
        <field name="tag_name">Base Retenciones ILA (ventas)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_tax_ila_ventas" model="account.tax.report.line">
        <field name="name">Impuesto Ret Practicadas ILA (ventas)</field>
        <field name="tag_name">Retenciones ILA (ventas)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

</odoo>

```

## File: data\account_tax_tags_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="tag_cl_sale_mnt_iva" model="account.account.tag">
            <!-- TotMntIVA -->
            <field name="name">Ventas - Monto IVA</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_mnt_fuera_plazo" model="account.account.tag">
            <!-- TotIVAFueraPlazo -->
            <field name="name">Ventas - IVA Fuera de Plazo</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_iva_propio" model="account.account.tag">
            <!-- TotIVAPropio -->
            <field name="name">Ventas - IVA Propio</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_iva_terceros" model="account.account.tag">
            <!-- TotIVATerceros -->
            <field name="name">Ventas - IVA Terceros</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_iva_18211" model="account.account.tag">
            <!-- TotLey18211 -->
            <field name="name">Ventas - IVA Ley 18211</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_otros_imp" model="account.account.tag">
            <!-- TotOtrosImp -->
            <field name="name">Ventas - Otros Impuestos</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_iva_ret_total" model="account.account.tag">
            <!-- TotIVARetTotal -->
            <field name="name">Ventas - IVA Retenido Total</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_iva_ret_parcial" model="account.account.tag">
            <!-- TotIVARetParcial -->
            <field name="name">Ventas - IVA Retenido Parcial</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_cred_ec" model="account.account.tag">
            <!-- TotCredEC -->
            <field name="name">Ventas - Credito Especial 65% Empresas Constructoras</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_dep_env" model="account.account.tag">
            <!-- TotDepEnvase -->
            <field name="name">Ventas - Depósito de Envases</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_iva_comisiones" model="account.account.tag">
            <!-- TotValComIVA -->
            <field name="name">Ventas - IVA Comisiones y Otros Cargos</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_iva_no_retenido" model="account.account.tag">
            <!-- TotOpIVANoRetenido -->
            <field name="name">Ventas - IVA No Retenido</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva" model="account.account.tag">
            <!-- TotMntIVA -->
            <field name="name">Compras - Monto IVA Recuperable</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva_actf" model="account.account.tag">
            <!-- TotMntIVAActivoFijo -->
            <field name="name">Compras - Monto IVA Activo Fijo</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva_actf_uso_comun" model="account.account.tag">
            <!-- TotMntIVAActivoFijo -->
            <field name="name">Compras - Monto IVA Activo Fijo (Uso Común)</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva_actf_no_recup" model="account.account.tag">
            <!-- TotMntIVAActivoFijo -->
            <field name="name">Compras - Monto IVA Activo Fijo (Uso Común)</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>


        <record id="tag_cl_purchase_mnt_iva_no_rec" model="account.account.tag">
            <!-- TotMntIVANoRec -->
            <field name="name">Compras - Monto IVA No Recuperable</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva_uso_comun" model="account.account.tag">
            <!-- TotMntIVANoRec -->
            <field name="name">Compras - Monto IVA Uso Comun</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva_supermercado" model="account.account.tag">
            <field name="name">Compras - Monto IVA Compras Supermercado</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_otros_imp" model="account.account.tag">
            <!-- TotOtrosImp -->
            <field name="name">Compras - Monto Otros Impuestos</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_imp_sin_credito" model="account.account.tag">
            <!-- TotImpSinCredito -->
            <field name="name">Compras - Impuestos Sin Crédito</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_iva_no_ret" model="account.account.tag">
            <!-- TotIVANoRetenido -->
            <field name="name">Compras - IVA No Retenido Fac de compra</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_imp_42" model="account.account.tag">
            <!-- TotCredImp -->
            <field name="name">Compras - Credito Imp Art 42</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_imp_sin_cred" model="account.account.tag">
            <!-- TotImpSinCredito -->
            <field name="name">Compras - Impuesto Sin Credito</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_iva_no_reten" model="account.account.tag">
            <!-- TotIVANoRetenido -->
            <field name="name">Compras - IVA No Retenido</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_imp_vehic" model="account.account.tag">
            <!-- TotImpVehiculo -->
            <field name="name">Compras - Impuesto a Vehículos Automotores</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

    </data>
</odoo>
```

## File: data\l10n_cl_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Account Tags -->
    <!-- Account Taxes Tags Ventas -->
    <record id="tag_cl_sale_mnt_exe" model="account.account.tag">
        <!-- TotMntExe -->
        <field name="name">Ventas - Total Monto Exento o No Gravado</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_mnt_neto" model="account.account.tag">
        <!-- TotMntNeto -->
        <field name="name">Ventas - Total Monto Neto</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_valor_neto_comis" model="account.account.tag">
        <!-- TotValComNeto -->
        <field name="name">Ventas - Total Valor Neto Comisiones y Otros Cargos</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_valor_comisiones_no_afecto" model="account.account.tag">
        <!-- TotValComExe -->
        <field name="name">Ventas - Total Valor Neto Comisiones y Otros Cargos No Afectos</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_monto_no_facturable" model="account.account.tag">
        <!-- TotMntNoFact -->
        <field name="name">Ventas - Total Monto No Facturable</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_exento_vta_pasajes_nacional" model="account.account.tag">
        <!-- TotPsjNac -->
        <field name="name">Ventas - Exento Ventas de Pasajes Nacional</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_exento_vta_pasajes_internacional" model="account.account.tag">
        <!-- TotPsjInt -->
        <field name="name">Ventas - Exento Ventas de Pasajes Internacional</field>
        <field name="applicability">accounts</field>
    </record>

    <!-- Account Taxes Tags Commpras -->
    <record id="tag_cl_purchase_mnt_exe" model="account.account.tag">
        <!-- TotMntExe -->
        <field name="name">Compras - Total Monto Exento o No Gravado</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_mnt_neto" model="account.account.tag">
        <!-- TotMntNeto -->
        <field name="name">Compras - Total Monto Neto</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_mnt_neto_uso_comun" model="account.account.tag">
        <!-- TotMntNeto -->
        <field name="name">Compras - Total Monto Neto Uso Común</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_mnt_neto_no_recup" model="account.account.tag">
        <!-- TotMntNeto -->
        <field name="name">Compras - Total Monto Neto No Recuperable</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_mnt_neto_supermercado" model="account.account.tag">
        <!-- TotMntNeto -->
        <field name="name">Compras - Total Monto Neto Compras de Supermercado</field>
        <field name="applicability">accounts</field>
    </record>


    <record id="tag_cl_purchase_mnt_neto_actf" model="account.account.tag">
        <!-- TotMntActivoFijo -->
        <field name="name">Compras - Total Monto Neto Activo Fijo</field>
        <field name="applicability">accounts</field>
    </record>


    <record id="tag_cl_purchase_tab_puros" model="account.account.tag">
        <!-- TotTabPuros -->
        <field name="name">Compras - Total Tabacos Manufacturados Puros</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_tab_cigar" model="account.account.tag">
        <!-- TotTabCigarrillos -->
        <field name="name">Compras - Total Tabacos Manufacturados Cigarrillos</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_tab_elab" model="account.account.tag">
        <!-- TotTabCigarrillos -->
        <field name="name">Compras - Total Tabacos Manufacturados Elaborados</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_remanente_cf" model="account.account.tag">
        <field name="name">Remantente de Crédito Fiscal</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_impuesto_unico_trabajadores" model="account.account.tag">
        <field name="name">Impuesto Unico Trabajadores</field>
        <field name="applicability">accounts</field>
    </record>

    <!-- honorarios y otros -->
    <record id="tag_cl_fees_amount" model="account.account.tag">
        <field name="name">Honorarios -  Montos Sujetos a Retención Renta 2 Categoría</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="cl_chart_template" model="account.chart.template">
        <field name="name">Chile - Plan de Cuentas</field>
        <field name="bank_account_code_prefix">1101</field>
        <field name="cash_account_code_prefix">1101</field>
        <field name="transfer_account_code_prefix">117</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.CLP"/>
        <field name="use_anglo_saxon" eval="True" />
        <field name="country_id" ref="base.cl"/>
    </record>

    <record id="account_11700" model="account.account.template">
        <field name="code">117000</field>
        <field name="name">Cuenta de Transferencia</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110210" model="account.account.template">
        <field name="code">110210</field>
        <field name="name">Depósitos en Divisas</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110220" model="account.account.template">
        <field name="code">110220</field>
        <field name="name">Acciones</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110310" model="account.account.template">
        <field name="code">110310</field>
        <field name="name">Clientes</field>
        <field ref="account.data_account_type_receivable" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_11320" model="account.account.template">
        <field name="code">110320</field>
        <field name="name">Anticipo Proveedores</field>
        <field ref="account.data_account_type_receivable" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_11410" model="account.account.template">
        <field name="code">110410</field>
        <field name="name">Cheques a Fecha Por Cobrar</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110420" model="account.account.template">
        <field name="code">110420</field>
        <field name="name">Deudores Varios</field>
        <field ref="account.data_account_type_receivable" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110421" model="account.account.template">
        <field name="code">110421</field>
        <field name="name">Deudores por Ventas (Pos)</field>
        <field ref="account.data_account_type_receivable" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>


    <record id="account_110430" model="account.account.template">
        <field name="code">110430</field>
        <field name="name">Boletas en Garantía</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110440" model="account.account.template">
        <field name="code">110440</field>
        <field name="name">Letras en Cartera</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110450" model="account.account.template">
        <field name="code">110450</field>
        <field name="name">Documentos Protestados</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110510" model="account.account.template">
        <field name="code">110510</field>
        <field name="name">Anticipo de Sueldo</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110520" model="account.account.template">
        <field name="code">110520</field>
        <field name="name">Préstamos otorgados</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110530" model="account.account.template">
        <field name="code">110530</field>
        <field name="name">Anticipos de Viáticos</field>
        <field ref="account.data_account_type_prepayments" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110540" model="account.account.template">
        <field name="code">110540</field>
        <field name="name">Fondos x Rendir</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110550" model="account.account.template">
        <field name="code">110550</field>
        <field name="name">Anticipo de Honorarios</field>
        <field ref="account.data_account_type_prepayments" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110560" model="account.account.template">
        <field name="code">110560</field>
        <field name="name">Anticipo de Aguinaldo</field>
        <field ref="account.data_account_type_prepayments" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110570" model="account.account.template">
        <field name="code">110570</field>
        <field name="name">Asignación Familiar</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110580" model="account.account.template">
        <field name="code">110580</field>
        <field name="name">Anticipo de Impuestos</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110585" model="account.account.template">
        <field name="code">110585</field>
        <field name="name">Alquileres Pagados por Adelantado</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110590" model="account.account.template">
        <field name="code">110590</field>
        <field name="name">Intereses Pagados por Adelantado</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110610" model="account.account.template">
        <field name="code">110610</field>
        <field name="name">Mercaderías</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110612" model="account.account.template">
        <field name="code">110612</field>
        <field name="name">Materia Prima</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110615" model="account.account.template">
        <field name="code">110615</field>
        <field name="name">Materiales</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110620" model="account.account.template">
        <field name="code">110620</field>
        <field name="name">Insumos</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110625" model="account.account.template">
        <field name="code">110625</field>
        <field name="name">Equipos</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110630" model="account.account.template">
        <field name="code">110630</field>
        <field name="name">Repuestos</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110640" model="account.account.template">
        <field name="code">110640</field>
        <field name="name">Existencias en Tránsito</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110650" model="account.account.template">
        <field name="code">110650</field>
        <field name="name">Productos Fabricados</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110660" model="account.account.template">
        <field name="code">110660</field>
        <field name="name">Productos En Proceso</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110670" model="account.account.template">
        <field name="code">110670</field>
        <field name="name">Envases</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110710" model="account.account.template">
        <field name="code">110710</field>
        <field name="name">IVA Crédito Fiscal</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110720" model="account.account.template">
        <field name="code">110720</field>
        <field name="name">Remanente Crédito Fiscal</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids" eval="[(6,0,[ref('tag_cl_remanente_cf')])]"/>
    </record>

    <record id="account_110730" model="account.account.template">
        <field name="code">110730</field>
        <field name="name">Crédito por Activo Fijo</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110740" model="account.account.template">
        <field name="code">110740</field>
        <field name="name">P.P.M. / Art 33 BIS</field>
        <field ref="account.data_account_type_receivable" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110750" model="account.account.template">
        <field name="code">110750</field>
        <field name="name">Crédito Sence</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110810" model="account.account.template">
        <field name="code">110810</field>
        <field name="name">Anticipos Comercio Exterior</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110820" model="account.account.template">
        <field name="code">110820</field>
        <field name="name">Gastos Anticipados</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110830" model="account.account.template">
        <field name="code">110830</field>
        <field name="name">Organización y Puesta en Marcha</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110910" model="account.account.template">
        <field name="code">110910</field>
        <field name="name">Retiros Socios</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_110920" model="account.account.template">
        <field name="code">110920</field>
        <field name="name">Retiros Reinversión</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_111010" model="account.account.template">
        <field name="code">111010</field>
        <field name="name">Gastos Reorganización</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_111110" model="account.account.template">
        <field name="code">111110</field>
        <field name="name">Cuentas Obligadas Socios</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_111210" model="account.account.template">
        <field name="code">111210</field>
        <field name="name">Gastos Tributarios</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_111310" model="account.account.template">
        <field name="code">111310</field>
        <field name="name">Cuenta Importaciones</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_111410" model="account.account.template">
        <field name="code">111410</field>
        <field name="name">Gastos Rechazados</field>
        <field ref="account.data_account_type_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_120110" model="account.account.template">
        <field name="code">120110</field>
        <field name="name">Deudores Morosos</field>
        <field ref="account.data_account_type_receivable" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_120120" model="account.account.template">
        <field name="code">120120</field>
        <field name="name">Deudores en Gestión Judicial</field>
        <field ref="account.data_account_type_receivable" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_121110" model="account.account.template">
        <field name="code">121110</field>
        <field name="name">Equipos y Mobiliario de Oficina</field>
        <field ref="account.data_account_type_fixed_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto_actf')])]"/>
    </record>

    <record id="account_121120" model="account.account.template">
        <field name="code">121120</field>
        <field name="name">Equipos Computacionales</field>
        <field ref="account.data_account_type_fixed_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto_actf')])]"/>
    </record>

    <record id="account_121130" model="account.account.template">
        <field name="code">121130</field>
        <field name="name">Vehículos</field>
        <field ref="account.data_account_type_fixed_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto_actf')])]"/>
    </record>

    <record id="account_121140" model="account.account.template">
        <field name="code">121140</field>
        <field name="name">Maquinaria</field>
        <field ref="account.data_account_type_fixed_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto_actf')])]"/>
    </record>

    <record id="account_121210" model="account.account.template">
        <field name="code">121210</field>
        <field name="name">Bienes Raíces</field>
        <field ref="account.data_account_type_fixed_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto_actf')])]"/>
    </record>

    <record id="account_121310" model="account.account.template">
        <field name="code">121310</field>
        <field name="name">Depreciación Acum Equipos y Mob de Oficina</field>
        <field ref="account.data_account_type_fixed_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_121320" model="account.account.template">
        <field name="code">121320</field>
        <field name="name">Depreciación Acum Equipos Computacionales</field>
        <field ref="account.data_account_type_fixed_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_121330" model="account.account.template">
        <field name="code">121330</field>
        <field name="name">Depreciación Acum Otros Activos</field>
        <field ref="account.data_account_type_fixed_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto_actf')])]"/>
    </record>

    <record id="account_130110" model="account.account.template">
        <field name="code">130110</field>
        <field name="name">Activo Intangible - Derecho de Llaves</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_130112" model="account.account.template">
        <field name="code">130112</field>
        <field name="name">Activo Intangible - Concesiones y Franquicias</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_130114" model="account.account.template">
        <field name="code">130114</field>
        <field name="name">Activo Intangible - Marcas y Patentes de Invención</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_130116" model="account.account.template">
        <field name="code">130116</field>
        <field name="name">Activo Intangible - (-) Amortización Acumulada</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_130120" model="account.account.template">
        <field name="code">130120</field>
        <field name="name">Derechos Otras Empresas</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_130130" model="account.account.template">
        <field name="code">130130</field>
        <field name="name">Cuentas por Cobrar a Personas y Empresas Relacionadas Largo Plazo</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_130140" model="account.account.template">
        <field name="code">130140</field>
        <field name="name">Documentos y Cuentas por Cobrar a Largo Plazo</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_130150" model="account.account.template">
        <field name="code">130150</field>
        <field name="name">Garantías de Obligaciones a Largo Plazo y de Obligaciones de Terceros</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_130160" model="account.account.template">
        <field name="code">130160</field>
        <field name="name">Títulos Patrimoniales Bolsas de Productos</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_190110" model="account.account.template">
        <field name="code">190110</field>
        <field name="name">Boletas de Garantía</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_190210" model="account.account.template">
        <field name="code">190210</field>
        <field name="name">Letras Descontadas</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_190310" model="account.account.template">
        <field name="code">190310</field>
        <field name="name">Documentos en Garantía</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_190410" model="account.account.template">
        <field name="code">190410</field>
        <field name="name">Acciones Suscritas</field>
        <field ref="account.data_account_type_non_current_assets" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210110" model="account.account.template">
        <field name="code">210110</field>
        <field name="name">Tarjeta de Crédito Corporativa</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210120" model="account.account.template">
        <field name="code">210120</field>
        <field name="name">Linea de Crédito Bancaria</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210140" model="account.account.template">
        <field name="code">210140</field>
        <field name="name">Crédito Comercial C/Plazo</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210150" model="account.account.template">
        <field name="code">210150</field>
        <field name="name">Obligaciones Leasing Corto Plazo</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210160" model="account.account.template">
        <field name="code">210160</field>
        <field name="name">Crédito Hipotecario C/Plazo</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210170" model="account.account.template">
        <field name="code">210170</field>
        <field name="name">Crédito Comercial LP Venc. C/Plazo</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210180" model="account.account.template">
        <field name="code">210180</field>
        <field name="name">Dividendos por Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210190" model="account.account.template">
        <field name="code">210190</field>
        <field name="name">Intereses a Devengar por Compras al Crédito</field>
        <field ref="account.data_account_type_payable" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210195" model="account.account.template">
        <field name="code">210195</field>
        <field name="name">Intereses a pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210210" model="account.account.template">
        <field name="code">210210</field>
        <field name="name">Proveedores</field>
        <field ref="account.data_account_type_payable" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>


    <record id="account_210220" model="account.account.template">
        <field name="code">210220</field>
        <field name="name">Anticipo de Clientes</field>
        <field ref="account.data_account_type_payable" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210230" model="account.account.template">
        <field name="code">210230</field>
        <field name="name">Facturas por Recibir</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210310" model="account.account.template">
        <field name="code">210310</field>
        <field name="name">Cheques Girados y No Cobrados</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210320" model="account.account.template">
        <field name="code">210320</field>
        <field name="name">Documentos en Garantía</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210330" model="account.account.template">
        <field name="code">210330</field>
        <field name="name">Boleta en Garantia</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210410" model="account.account.template">
        <field name="code">210410</field>
        <field name="name">AFP x Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210420" model="account.account.template">
        <field name="code">210420</field>
        <field name="name">C.C.A.F. x Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210430" model="account.account.template">
        <field name="code">210430</field>
        <field name="name">INP x Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210440" model="account.account.template">
        <field name="code">210440</field>
        <field name="name">ISAPRES x Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210450" model="account.account.template">
        <field name="code">210450</field>
        <field name="name">Mutual Seg. x Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210510" model="account.account.template">
        <field name="code">210510</field>
        <field name="name">Sueldos por Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210520" model="account.account.template">
        <field name="code">210520</field>
        <field name="name">Honorarios por Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210550" model="account.account.template">
        <field name="code">210550</field>
        <field name="name">Rendiciones por Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210560" model="account.account.template">
        <field name="code">210560</field>
        <field name="name">Provisión de Vacaciones</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210565" model="account.account.template">
        <field name="code">210565</field>
        <field name="name">Provisión por Finiquitos</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210610" model="account.account.template">
        <field name="code">210610</field>
        <field name="name">Provisión de Impuesto PPM</field>
        <field ref="account.data_account_type_non_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210620" model="account.account.template">
        <field name="code">210620</field>
        <field name="name">Otras Provisiones</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210710" model="account.account.template">
        <field name="code">210710</field>
        <field name="name">IVA Débito Fiscal</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210720" model="account.account.template">
        <field name="code">210720</field>
        <field name="name">PPM por Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210730" model="account.account.template">
        <field name="code">210730</field>
        <field name="name">Impuesto Único Trabajadores</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids" eval="[(6,0,[ref('tag_cl_impuesto_unico_trabajadores')])]"/>
    </record>

    <record id="account_210740" model="account.account.template">
        <field name="code">210740</field>
        <field name="name">Impuesto Retención Segunda Categoría</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210715" model="account.account.template">
        <field name="code">210715</field>
        <field name="name">IVA Retenido a terceros</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210750" model="account.account.template">
        <field name="code">210750</field>
        <field name="name">Impuesto Renta 1a Categoría por Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_210760" model="account.account.template">
        <field name="code">210760</field>
        <field name="name">Impuestos Mensuales por Pagar</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_220120" model="account.account.template">
        <field name="code">220120</field>
        <field name="name">Obligaciones Leasing L/Plazo</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_220130" model="account.account.template">
        <field name="code">220130</field>
        <field name="name">Crédito Comercial L/Plazo</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_220210" model="account.account.template">
        <field name="code">220210</field>
        <field name="name">Provisión Indemnización por Años de Servicio</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_220310" model="account.account.template">
        <field name="code">220310</field>
        <field name="name">Cta Corriente Empresa Relacionada</field>
        <field ref="account.data_account_type_non_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_220320" model="account.account.template">
        <field name="code">220320</field>
        <field name="name">Impuesto Diferido Largo Plazo</field>
        <field ref="account.data_account_type_non_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_220330" model="account.account.template">
        <field name="code">220330</field>
        <field name="name">Otros Pasivos</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230110" model="account.account.template">
        <field name="code">230110</field>
        <field name="name">Capital Social</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230120" model="account.account.template">
        <field name="code">230120</field>
        <field name="name">Acciones en Circulación</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230130" model="account.account.template">
        <field name="code">230130</field>
        <field name="name">Dividendos a Distribuir en Acciones</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230140" model="account.account.template">
        <field name="code">230140</field>
        <field name="name">Descuento de Emisión de Acciones</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230210" model="account.account.template">
        <field name="code">230210</field>
        <field name="name">Revalorización Capital Propio</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230220" model="account.account.template">
        <field name="code">230220</field>
        <field name="name">Otras Revalorizaciones</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230230" model="account.account.template">
        <field name="code">230230</field>
        <field name="name">Revalorización Activo Fijo</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230240" model="account.account.template">
        <field name="code">230240</field>
        <field name="name">Fluctuación de Valores</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230250" model="account.account.template">
        <field name="code">230250</field>
        <field name="name">Reserva Legal</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230260" model="account.account.template">
        <field name="code">230260</field>
        <field name="name">Reserva Estatutaria</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230270" model="account.account.template">
        <field name="code">230270</field>
        <field name="name">Reserva Facultativa</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230280" model="account.account.template">
        <field name="code">230280</field>
        <field name="name">Reserva para Renovación de Activo Fijo</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230290" model="account.account.template">
        <field name="code">230290</field>
        <field name="name">Resultados Acumulados</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230300" model="account.account.template">
        <field name="code">230300</field>
        <field name="name">Utilidades y Pérdidas del Ejercicio</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230310" model="account.account.template">
        <field name="code">230310</field>
        <field name="name">Resultados Acumulados del Ejercicio Anterior</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_230510" model="account.account.template">
        <field name="code">230510</field>
        <field name="name">Resultado del Ejercicio</field>
        <field ref="account.data_account_type_equity" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_290110" model="account.account.template">
        <field name="code">290110</field>
        <field name="name">Cuenta Puente</field>
        <field ref="account.data_account_type_non_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="True"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_290210" model="account.account.template">
        <field name="code">290210</field>
        <field name="name">Responsable Boletas Garantía</field>
        <field ref="account.data_account_type_non_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_290220" model="account.account.template">
        <field name="code">290220</field>
        <field name="name">Responsable Documentos Garantía</field>
        <field ref="account.data_account_type_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_290230" model="account.account.template">
        <field name="code">290230</field>
        <field name="name">Responsable Letras Descontadas</field>
        <field ref="account.data_account_type_non_current_liabilities" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_310110" model="account.account.template">
        <field name="code">310110</field>
        <field name="name">Ingresos por Consultoría</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_310115" model="account.account.template">
        <field name="code">310115</field>
        <field name="name">Ventas de Productos</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[
                         ref('tag_cl_sale_mnt_neto'),
                         ref('tag_cl_sale_valor_neto_comis')
               ])]"/>
    </record>

    <record id="account_310120" model="account.account.template">
        <field name="code">310120</field>
        <field name="name">Ventas de Servicios</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[
                         ref('tag_cl_sale_mnt_neto'),
                         ref('tag_cl_sale_valor_neto_comis')
               ])]"/>
    </record>

    <record id="account_310122" model="account.account.template">
        <field name="code">310122</field>
        <field name="name">Ventas de Pasajes Nacionales/Internacionales y/o Comisiones No Afectas</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[
                         ref('tag_cl_sale_valor_comisiones_no_afecto'),
                         ref('tag_cl_sale_exento_vta_pasajes_nacional'),
                         ref('tag_cl_sale_exento_vta_pasajes_internacional')
               ])]"/>
    </record>

    <record id="account_310125" model="account.account.template">
        <field name="code">310125</field>
        <field name="name">Ventas de Exportación</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="tag_ids"
               eval="[(6,0,[
                        ref('tag_cl_sale_mnt_exe'),
               ])]"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_310130" model="account.account.template">
        <field name="code">310130</field>
        <field name="name">Comisiones Percibidas por Ventas</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320210" model="account.account.template">
        <field name="code">320210</field>
        <field name="name">Utilidad Venta Activo Fijo</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320220" model="account.account.template">
        <field name="code">320220</field>
        <field name="name">Intereses Percibidos Sobre Préstamos Otorgados</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320225" model="account.account.template">
        <field name="code">320225</field>
        <field name="name">Arriendos ganados, obtenidos, percibidos</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320230" model="account.account.template">
        <field name="code">320230</field>
        <field name="name">Descuentos ganados, obtenidos, percibidos</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320235" model="account.account.template">
        <field name="code">320235</field>
        <field name="name">Intereses sobre Inversiones</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320240" model="account.account.template">
        <field name="code">320240</field>
        <field name="name">Ganancia Venta de Acciones</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320245" model="account.account.template">
        <field name="code">320245</field>
        <field name="name">Recupero de Rezagos</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320250" model="account.account.template">
        <field name="code">320250</field>
        <field name="name">Recupero de Deudores Incobrables</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320255" model="account.account.template">
        <field name="code">320255</field>
        <field name="name">Donaciones obtenidas, ganandas, percibidas</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320260" model="account.account.template">
        <field name="code">320260</field>
        <field name="name">Ganancia Venta Inversiones Permanentes</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320265" model="account.account.template">
        <field name="code">320265</field>
        <field name="name">Diferencia tipo de cambio (Ganancia)</field>
        <field ref="account.data_account_type_revenue" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320270" model="account.account.template">
        <field name="code">320270</field>
        <field name="name">Cŕeditos a Largo Plazo</field>
        <field ref="account.data_account_type_other_income" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320275" model="account.account.template">
        <field name="code">320275</field>
        <field name="name">Otros Ingresos</field>
        <field ref="account.data_account_type_other_income" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320280" model="account.account.template">
        <field name="code">320280</field>
        <field name="name">Corrección Monetaria Activos</field>
        <field ref="account.data_account_type_other_income" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_320285" model="account.account.template">
        <field name="code">320285</field>
        <field name="name">Corrección Monetaria Bienes Leasing</field>
        <field ref="account.data_account_type_other_income" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410110" model="account.account.template">
        <field name="code">410110</field>
        <field name="name">Remuneraciones Operación</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410115" model="account.account.template">
        <field name="code">410115</field>
        <field name="name">Aporte Patronal Operación</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410120" model="account.account.template">
        <field name="code">410120</field>
        <field name="name">Viáticos</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410125" model="account.account.template">
        <field name="code">410125</field>
        <field name="name">Capacitación</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410130" model="account.account.template">
        <field name="code">410130</field>
        <field name="name">Honorarios Profesionales</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410135" model="account.account.template">
        <field name="code">410135</field>
        <field name="name">Asesorías</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410140" model="account.account.template">
        <field name="code">410140</field>
        <field name="name">Arriendos Oficina</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410145" model="account.account.template">
        <field name="code">410145</field>
        <field name="name">Arriendos Bodega</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410150" model="account.account.template">
        <field name="code">410150</field>
        <field name="name">Comunicaciones</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410155" model="account.account.template">
        <field name="code">410155</field>
        <field name="name">Servicios Legales</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410160" model="account.account.template">
        <field name="code">410160</field>
        <field name="name">Manutención y Reparación de Activos</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410165" model="account.account.template">
        <field name="code">410165</field>
        <field name="name">Papelería-Aseo-Gastos Diversos</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410170" model="account.account.template">
        <field name="code">410170</field>
        <field name="name">Remuneraciones Administración</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410175" model="account.account.template">
        <field name="code">410175</field>
        <field name="name">Aporte Patronal Administración</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410180" model="account.account.template">
        <field name="code">410180</field>
        <field name="name">Electricidad</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410185" model="account.account.template">
        <field name="code">410185</field>
        <field name="name">Gastos Comunes</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410190" model="account.account.template">
        <field name="code">410190</field>
        <field name="name">Gastos Bancarios</field>
        <field ref="account.data_account_off_sheet" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410195" model="account.account.template">
        <field name="code">410195</field>
        <field name="name">Diferencia Tipo de Cambio (Pérdida)</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410200" model="account.account.template">
        <field name="code">410200</field>
        <field name="name">Corrección Monetaria</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410205" model="account.account.template">
        <field name="code">410205</field>
        <field name="name">Multas Fiscales</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410210" model="account.account.template">
        <field name="code">410210</field>
        <field name="name">Impuesto a la Renta 1ra Categoría</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410215" model="account.account.template">
        <field name="code">410215</field>
        <field name="name">Pérdida por Venta Activo</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410220" model="account.account.template">
        <field name="code">410220</field>
        <field name="name">Ajuste Ejercicio Anterior</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410225" model="account.account.template">
        <field name="code">410225</field>
        <field name="name">Donación</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410230" model="account.account.template">
        <field name="code">410230</field>
        <field name="name">Compras Productos 1ra Categoría</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto'), ref('tag_cl_purchase_tab_puros'), ref('tag_cl_purchase_tab_cigar')])]"/>
    </record>
    <record id="account_410231" model="account.account.template">
        <field name="code">410231</field>
        <field name="name">Compras Netas (IVA Uso Común)</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto_uso_comun')])]"/>
    </record>
    <record id="account_410232" model="account.account.template">
        <field name="code">410232</field>
        <field name="name">Compras Netas (IVA No Recuperable)</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto_no_recup')])]"/>
    </record>
    <record id="account_410233" model="account.account.template">
        <field name="code">410233</field>
        <field name="name">Compras de Supermercado</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto_supermercado')])]"/>
    </record>

    <record id="account_410235" model="account.account.template">
        <field name="code">410235</field>
        <field name="name">Costo de Mercaderías Vendidas - Prod. 1ra Categoría</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
        <field name="tag_ids"
               eval="[(6,0,[ref('tag_cl_purchase_mnt_neto'), ref('tag_cl_purchase_tab_puros'), ref('tag_cl_purchase_tab_cigar')])]"/>
    </record>

    <record id="account_410240" model="account.account.template">
        <field name="code">410240</field>
        <field name="name">Importaciones</field>
        <field ref="account.data_account_type_direct_costs" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410245" model="account.account.template">
        <field name="code">410245</field>
        <field name="name">CIF</field>
        <field ref="account.data_account_type_direct_costs" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410250" model="account.account.template">
        <field name="code">410250</field>
        <field name="name">Compras de Materia Prima</field>
        <field ref="account.data_account_type_direct_costs" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_410255" model="account.account.template">
        <field name="code">410255</field>
        <field name="name">Mano de Obra Directa</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_420110" model="account.account.template">
        <field name="code">420110</field>
        <field name="name">Mercaderias Recibidas en Consignación</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_440210" model="account.account.template">
        <field name="code">440210</field>
        <field name="name">Comitente por Mercaderias Recibidas en Consignación</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_420120" model="account.account.template">
        <field name="code">420120</field>
        <field name="name">Gastos en Depreciación de Activo Fijo</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_420140" model="account.account.template">
        <field name="code">420140</field>
        <field name="name">Gastos en Amortización</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_420150" model="account.account.template">
        <field name="code">420150</field>
        <field name="name">Gastos en Siniestros</field>
        <field ref="account.data_account_type_expenses" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_420170" model="account.account.template">
        <field name="code">420170</field>
        <field name="name">Garantias Otorgadas</field>
        <field ref="account.data_account_off_sheet" name="user_type_id"/>
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="account_420220" model="account.account.template">
        <field name="code">420220</field>
        <field name="name">Gastos Otros Impuestos</field>
        <field name="user_type_id" ref="account.data_account_type_expenses" />
        <field name="reconcile" eval="False"/>
        <field name="chart_template_id" ref="cl_chart_template"/>
    </record>

    <record id="cl_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="account_110310"/>
        <field name="property_account_payable_id" ref="account_210210"/>
        <field name="property_account_expense_categ_id" ref="account_410235"/>
        <field name="property_account_income_categ_id" ref="account_310115"/>
        <field name="income_currency_exchange_account_id" ref="account_320265"/>
        <field name="expense_currency_exchange_account_id" ref="account_410195"/>
        <field name="default_pos_receivable_account_id" ref="account_110421"/>
        <field name="property_stock_account_input_categ_id" ref="account_210230"/>
        <field name="property_stock_account_output_categ_id" ref="account_110640"/>
        <field name="property_stock_valuation_account_id" ref="account_110610"/>
    </record>

</odoo>

```

## File: data\l10n_latam.document.type.csv

```csv
id,sequence,code,name,report_name,internal_type,doc_code_prefix,country_id/id,active
dc_a_f_dte,1,33,Factura Electrónica,FACTURA,invoice,FAC,base.cl,True
dc_y_f_dte,2,34,Factura no Afecta o Exenta Electrónica,F-EXENTA,invoice,FNA,base.cl,True
dc_nc_f_dte,3,61,Nota de Crédito Electrónica,NOTA DE CREDITO,credit_note,N/C,base.cl,True
dc_nd_f_dte,4,56,Nota de Débito Electrónica,NOTA DE DEBITO,debit_note,N/D,base.cl,True
dc_b_f_dte,5,39,Boleta Electrónica,BEL,invoice,BEL,base.cl,True
dc_m_d_dtn,6,71,Boleta de Honorarios Electrónica,BHE,invoice,BHE,base.cl,True
dc_b_e_dtn,7,38,Boleta exenta,BEX,invoice,BEX,base.cl,False
dc_I_f_dtn,10,29,Factura de Inicio,FAI,invoice,FAI,base.cl,False
dc_a_f_dtn,10,30,Factura,FACTURA,invoice,FAC,base.cl,False
dc_y_f_dtn,10,32,Factura de Ventas y Servicios no Afectos o Exentos de IVA,F-EXENTA,invoice,FNA,base.cl,False
dc_l_f_dtn,10,40,Liquidación Factura,L-FACTURAM,invoice,FAL,base.cl,False
dc_l_f_dte,10,43,Liquidación Factura Electrónica,L-FACTURAE,invoice,FAL,base.cl,False
dc_fc_f_dtn,10,45,Factura de Compra,FACTURA,invoice_in,FAC,base.cl,False
dc_fc_f_dte,10,46,Factura de Compra Electrónica,FACTURA,invoice_in,FAC,base.cl,True
dc_nd_f_dtn,10,55,Nota de Débito,NOTA DE DEBITO,debit_note,N/D,base.cl,False
dc_nc_f_dtn,10,60,Nota de Crédito,NOTA DE CREDITO,credit_note,N/C,base.cl,False
dc_s_f_dtn,10,108,SRF Solicitud de Registro de Factura,SOL REGISTRO,,REG,base.cl,False
dc_ftf_f_dtn,10,901,Factura de ventas a empresas del territorio preferencial ( Res. Ex. N° 1057,FACTURA,invoice,FAC,base.cl,False
dc_din_f_dtn,10,914,Declaración de Ingreso (DIN),DEC ING,invoice_in,DIN,base.cl,False
dc_dizf_f_dtn,10,911,Declaración de Ingreso a Zona Franca Primaria,DEC ING,invoice_in,DIN,base.cl,False
dc_ftt_f_dtn,10,904,Factura de Traspaso,FACT TR,invoice,FTR,base.cl,False
dc_frr_f_dtn,10,905,Factura de Reexpedición,FACT RX,invoice,FRX,base.cl,False
dc_bzf_f_dtn,10,906,Boletas Venta Módulos ZF (todas),BOLETA ZF,invoice,BZF,base.cl,False
dc_fzf_f_dtn,10,907,Facturas Venta Módulo ZF (todas),FACTURA ZF,invoice,FZF,base.cl,False
dc_tca_f_dtn,10,500,Ajuste aumento Tipo de Cambio (código 500),TC-A,,TCA,base.cl,False
dc_tcd_f_dtn,10,501,Ajuste disminución Tipo de Cambio (código 501),TC-D,,TCD,base.cl,False
dc_b_f_dtn,10,35,Boleta de Venta,BOL,invoice,BOL,base.cl,False
dc_b_e_dte,10,41,Boleta Exenta Electrónica,BXE,invoice,BXE,base.cl,False
dc_m_f_dtn,10,70,Boleta de Honorarios,BHO,invoice,BHO,base.cl,False
dc_hes,10,HES,Hoja de Entrada de Servicio (HES),HES,,HES,base.cl,False
dc_hem,10,HEM,Hoja de Entrada de Materiales (HEM),HEM,,HEM,base.cl,False
dc_migo,10,MIG,Movimiento de Mercancías (MIGO),MIGO,,MIGO,base.cl,False
dc_li,10,103,Liquidación,LIQ,,LIQ,base.cl,False
dc_fe_dte,10,110,Factura de Exportación Electrónica,FCXE,invoice,FCXE,base.cl,False
dc_ndex_dte,10,111,Nota de Débito de Exportación Electrónica,NDXE,debit_note,NDXE,base.cl,False
dc_ncex_dte,10,112,Nota de Crédito de Exportación Electrónica,NCXE,credit_note,NCXE,base.cl,False
dc_odc,10,801,Orden de Compra,OC,,OC,base.cl,False
dc_ndp,10,802,Nota de pedido,NP,,NP,base.cl,False
dc_cont,10,803,Contrato,CONT,,CONT,base.cl,False
dc_resol,10,804,Resolución,RES,,RES,base.cl,False
dc_prchc,10,805,Proceso ChileCompra,PCHC,,PCHC,base.cl,False
dc_fichc,10,806,Ficha ChileCompra,FCHC,,FCHC,base.cl,False
dc_dus,10,807,DUS,DUS,,DUS,base.cl,False
dc_bl_cemb,10,808,B/L (Conocimiento de embarque),B/L,,B/L,base.cl,False
dc_awb,10,809,AWB Airway Bill,AWB,,AWB,base.cl,False
dc_mic_dta,10,810,MIC/DTA,MDT,,MDT,base.cl,False
dc_cpor,10,811,Carta de Porte,CPR,,CPR,base.cl,False
dc_res_sna,10,812,Resolución del SNA donde califica Servicios de Exportación,RSN,,RSN,base.cl,False
dc_pasap,10,813,Pasaporte,PSP,,PSP,base.cl,False
dc_cd_bol,10,814,Certificado de Depósito Bolsa Prod. Chile.,CRTD,,CRTD,base.cl,False
dc_vp_pren,10,815,Vale de Prenda Bolsa Prod. Chile,VLPR,,VLPR,base.cl,False
dc_oc,300,10,Orden de Compra,OC,,OC,base.cl,False

```

## File: data\l10n_latam_identification_type_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="False">
        <record model='l10n_latam.identification.type' id='it_RUT'>
            <field name='name'>RUT</field>
            <field name='description'>RUT</field>
            <field name='sequence'>11</field>
            <field name='is_vat' eval='True'/>
            <field name='country_id' ref="base.cl"/>
        </record>
        <record model='l10n_latam.identification.type' id='it_RUN'>
            <field name='name'>RUN</field>
            <field name='description'>Cédula</field>
            <field name='sequence'>12</field>
            <field name='country_id' ref="base.cl"/>
        </record>
        <record model='l10n_latam.identification.type' id='it_DNI'>
            <field name='name'>DNI</field>
            <field name='description'>DNI Extranjero</field>
            <field name='sequence'>13</field>
            <field name='country_id' ref="base.cl"/>
        </record>
    </data>
</odoo>

```

## File: data\menuitem_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_cl_statements_menu" name="Chile" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
</odoo>

```

## File: data\product_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

    <record id="product_product_ad_valorem" model="product.product">
        <field name="name">Ad-Valorem</field>
        <field name="detailed_type">service</field>
        <field name="default_code">AD_VALOREM</field>
        <field name="description">Cargo para calculo de Ad-Valorem en DIN</field>
        <field name="sale_ok" eval="False"/>
        <field name="uom_id" ref="uom.product_uom_unit"/>
    </record>

</odoo>

```

## File: data\res.bank.csv

```csv
id,name,l10n_cl_sbif_code,country
bank_001_0,Banco de Chile,001,Chile
bank_001_1,Banco Edwards,001,Chile
bank_001_2,Citi,001,Chile
bank_001_3,Atlas,001,Chile
bank_001_4,CrediChile,001,Chile
bank_009_0,Banco Internacional,009,Chile
bank_012_0,Banco Estado,012,Chile
bank_014_0,Scotiabank Chile,014,Chile
bank_016_0,Banco De Credito e Inversiones (BCI),016,Chile
bank_016_1,Tbanc,016,Chile
bank_016_2,BCI Nova,016,Chile
bank_027_0,Corpbanca,027,Chile
bank_027_1,Banco Condell,027,Chile
bank_028_0,Banco Bice,028,Chile
bank_031_0,Hsbc Bank,031,Chile
bank_037_0,Banco Santander-Chile,037,Chile
bank_037_1,Banefe,037,Chile
bank_039_0,Banco Itaú Chile,039,Chile
bank_049_0,Banco Security,049,Chile
bank_051_0,Banco Falabella,051,Chile
bank_052_0,Deutsche Bank,052,Chile
bank_053_0,Banco Ripley,053,Chile
bank_054_0,Rabobank Chile (Ex Hns Banco),054,Chile
bank_055_0,Banco Consorcio (Ex Banco Monex),055,Chile
bank_056_0,Banco Penta,056,Chile
bank_057_0,Banco Paris,057,Chile
bank_504_0,Banco Bilbao Vizcaya Argentaria Chile (BBVA),504,Chile
bank_504_1,BBVA Express,504,Chile
bank_059_0,Banco Btg Pactual Chile,059,Chile
bank_017_0,Banco Do Brasil S.A.,017,Chile
bank_041_0,Jp Morgan Chase Bank  N. A.,041,Chile
bank_043_0,Banco De La Nacion Argentina,043,Chile

```

## File: data\res.country.csv

```csv
id,l10n_cl_customs_code
base.ad,525
base.af,308
base.ag,240
base.al,518
base.am,540
base.ao,140
base.ar,224
base.at,509
base.au,406
base.aw,243
base.az,541
base.bb,204
base.bd,321
base.be,514
base.bf,161
base.bg,527
base.bi,141
base.bj,150
base.bm,244
base.bo,221
base.br,220
base.bs,207
base.bw,113
base.by,542
base.ca,226
base.cg,144
base.ch,508
base.ci,107
base.ck,427
base.cl,997
base.cn,336
base.co,202
base.cr,211
base.cu,209
base.cv,129
base.cy,305
base.de,563
base.dj,155
base.dk,507
base.dm,231
base.dz,127
base.ec,218
base.ee,549
base.eg,124
base.er,163
base.es,517
base.fi,512
base.fj,401
base.fm,417
base.fr,505
base.ga,145
base.gd,232
base.ge,550
base.gg,566
base.gh,108
base.gi,565
base.gl,253
base.gm,102
base.gn,104
base.gq,147
base.gr,520
base.gt,215
base.gu,425
base.gy,217
base.hn,214
base.hr,547
base.ht,208
base.id,328
base.ie,506
base.il,306
base.in,317
base.iq,307
base.ir,309
base.is,516
base.it,504
base.je,568
base.jm,205
base.jo,301
base.jp,331
base.ke,137
base.kh,315
base.ki,416
base.kp,334
base.kr,333
base.kw,303
base.la,316
base.li,534
base.lk,314
base.lr,106
base.ls,114
base.lt,554
base.lv,553
base.ly,125
base.ma,128
base.mc,535
base.md,556
base.me,561
base.mg,120
base.mh,164
base.ml,133
base.mn,337
base.mo,345
base.mp,424
base.mq,250
base.mr,134
base.mt,523
base.mu,119
base.mw,115
base.mx,216
base.my,329
base.mz,121
base.na,159
base.nc,423
base.ne,131
base.ng,111
base.ni,212
base.nl,515
base.no,513
base.np,320
base.nr,402
base.nu,421
base.om,304
base.pa,210
base.pe,219
base.pf,422
base.ph,335
base.pk,324
base.pl,528
base.pr,251
base.pt,501
base.py,222
base.qa,312
base.ro,519
base.rs,546
base.rw,142
base.sc,156
base.sd,123
base.se,511
base.sg,332
base.si,548
base.sl,105
base.sm,536
base.sn,101
base.so,138
base.sr,235
base.sv,213
base.sy,310
base.td,130
base.tg,109
base.tm,558
base.tt,203
base.tv,419
base.tw,330
base.tz,135
base.ua,559
base.ug,136
base.uk,510
base.us,225
base.uy,223
base.uz,560
base.vc,234
base.ve,201
base.vn,325
base.vu,415
base.za,112
base.zm,117
base.zw,116

```

## File: data\res.currency.csv

```csv
id,l10n_cl_currency_code,l10n_cl_short_name
base.AED,139,DIRHAM
base.ARS,1,PESO
base.AUD,36,DOLAR AUST
base.BOB,4,BOLIVIANO
base.BRL,5,CRUZEIRO REAL
base.CAD,6,DOLAR CAN
base.CHF,82,FRANCO SZ
base.CLP,200,PESO CL
base.CNY,48,RENMINBI
base.COP,129,PESO COL
base.EUR,142,EURO
base.GBP,102,LIBRA EST
base.HKD,127,DOLAR HK
base.INR,137,RUPIA
base.JPY,72,YEN
base.MXN,132,PESO MEX
base.NOK,96,CORONA NOR
base.NZD,97,DOLAR NZ
base.PEN,24,NUEVO SOL
base.PYG,23,GUARANI
base.SEK,113,CORONA SC
base.SGD,136,DOLAR SIN
base.TWD,138,DOLAR TAI
base.USD,13,DOLAR USA
base.UYU,26,PESO URUG
base.VEF,134,BOLIVAR
base.ZAR,128,RAND

```

## File: data\res_currency_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="UF" model="res.currency">
            <field name="name">UF</field>
            <field name="symbol">UF</field>
            <field name="rounding">0.01</field>
            <field name="position">after</field>
            <field name="currency_unit_label">Unidad de Fomento</field>
            <field name="l10n_cl_short_name">Unidad de Fomento</field>
        </record>

        <record id="UTM" model="res.currency">
            <field name="name">UTM</field>
            <field name="symbol">UTM</field>
            <field name="rounding">0.01</field>
            <field name="position">after</field>
            <field name="currency_unit_label">Unidad Tributaria Mensual</field>
            <field name="l10n_cl_short_name">Unidad Tributaria Mensual</field>
        </record>

        <record id="OTR" model="res.currency">
            <field name="name">OTR</field>
            <field name="symbol">OTR</field>
            <field name="rounding">1</field>
            <field name="l10n_cl_currency_code">900</field>
            <field name="l10n_cl_short_name">OTR</field>
        </record>
    </data>
</odoo>

```

## File: data\res_partner.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="True">

        <record model='res.partner' id='par_cfa'>
            <field name='name'>Consumidor Final Anónimo</field>
            <field name='l10n_cl_sii_taxpayer_type'>3</field>
            <field name='l10n_latam_identification_type_id' ref='l10n_cl.it_RUT'/>
            <field name='vat'>66666666-6</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record model='res.partner' id='par_tgr'>
            <field name='name'>Tesorería General de la República</field>
            <field name="company_type">company</field>
            <field name='l10n_cl_sii_taxpayer_type'>1</field>
            <field name='l10n_latam_identification_type_id' ref='l10n_cl.it_RUT'/>
            <field name='vat'>60805000-0</field>
            <field name="image_1920" type="base64" file="l10n_cl/static/tgr_logo.png"/>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record model='res.partner' id='par_sii'>
            <field name='name'>Servicio de Impuestos Internos</field>
            <field name="company_type">company</field>
            <field name='l10n_cl_sii_taxpayer_type'>1</field>
            <field name='l10n_latam_identification_type_id' ref='l10n_cl.it_RUT'/>
            <field name='vat'>60803000-K</field>
            <field name="image_1920" type="base64" file="l10n_cl/static/sii_logo.jpeg"/>
            <field name="country_id" ref="base.cl"/>
        </record>

    </data>
</odoo>

```

## File: data\uom_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="True">
      <record id="uom_categ_energy" model="uom.category">
        <field name="name">Energía</field>
      </record>
      <record id="uom_categ_others" model="uom.category">
        <field name="name">Otros</field>
      </record>

      <record id="uom.product_uom_unit" model="uom.uom">
        <field name="l10n_cl_sii_code">10</field>
      </record>

      <record id="uom.product_uom_dozen" model="uom.uom">
        <field name="l10n_cl_sii_code">11</field>
      </record>

      <record id="uom.product_uom_meter" model="uom.uom">
        <field name="l10n_cl_sii_code">14</field>
      </record>

      <record id="uom.product_uom_foot" model="uom.uom">
        <field name="l10n_cl_sii_code">13</field>
      </record>

      <record id="uom.product_uom_kgm" model="uom.uom">
        <field name="l10n_cl_sii_code">6</field>
      </record>

      <record id="uom.product_uom_litre" model="uom.uom">
        <field name="l10n_cl_sii_code">9</field>
      </record>

      <record id="product_uom_sum" model="uom.uom">
        <field name="l10n_cl_sii_code">0</field>
        <field name="name">S.U.M</field>
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="uom_type">smaller</field>
      </record>

      <record id="product_uom_tmb" model="uom.uom">
        <field name="l10n_cl_sii_code">1</field>
        <field name="name">TMB</field>
        <field name="category_id" ref="uom.product_uom_categ_kgm"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_u" model="uom.uom">
        <field name="l10n_cl_sii_code">12</field>
        <field name="name">U(JGO)</field>
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_mt2" model="uom.uom">
        <field name="l10n_cl_sii_code">15</field>
        <field name="name">MT2</field>
        <field name="category_id" ref="uom.uom_categ_length"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_mcub" model="uom.uom">
        <field name="l10n_cl_sii_code">16</field>
        <field name="name">MCUB</field>
        <field name="category_id" ref="uom.product_uom_categ_vol"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_par" model="uom.uom">
        <field name="l10n_cl_sii_code">17</field>
        <field name="name">PAR</field>
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_knfc" model="uom.uom">
        <field name="l10n_cl_sii_code">18</field>
        <field name="name">KNFC</field>
        <field name="category_id" ref="uom.product_uom_categ_kgm"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_carton" model="uom.uom">
        <field name="l10n_cl_sii_code">19</field>
        <field name="name">CARTON</field>
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_qmb" model="uom.uom">
        <field name="l10n_cl_sii_code">2</field>
        <field name="name">QMB</field>
        <field name="category_id" ref="uom.product_uom_categ_kgm"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_kwh" model="uom.uom">
        <field name="l10n_cl_sii_code">20</field>
        <field name="name">KWH</field>
        <field name="category_id" ref="uom_categ_energy"/>
      </record>

      <record id="product_uom_bar" model="uom.uom">
        <field name="l10n_cl_sii_code">23</field>
        <field name="name">BAR</field>
        <field name="category_id" ref="uom.product_uom_categ_vol"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_mm" model="uom.uom">
        <field name="l10n_cl_sii_code">24</field>
        <field name="name">M2/1MM</field>
        <field name="category_id" ref="uom.uom_categ_length"/>
        <field name="uom_type">smaller</field>
      </record>

      <record id="product_uom_mkwh" model="uom.uom">
        <field name="l10n_cl_sii_code">3</field>
        <field name="name">MKWH</field>
        <field name="category_id" ref="uom_categ_energy"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_tmn" model="uom.uom">
        <field name="l10n_cl_sii_code">4</field>
        <field name="name">TMN</field>
        <field name="category_id" ref="uom.product_uom_categ_kgm"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_qnt" model="uom.uom">
        <field name="l10n_cl_sii_code">5</field>
        <field name="name">QNT</field>
        <field name="category_id" ref="uom.product_uom_categ_kgm"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="uom.product_uom_gram" model="uom.uom">
        <field name="l10n_cl_sii_code">7</field>
        <field name="uom_type">smaller</field>
      </record>

      <record id="product_uom_hl" model="uom.uom">
        <field name="l10n_cl_sii_code">8</field>
        <field name="name">HL</field>
        <field name="category_id" ref="uom.product_uom_categ_vol"/>
        <field name="uom_type">bigger</field>
      </record>

      <record id="product_uom_sum_99" model="uom.uom">
        <field name="l10n_cl_sii_code">99</field>
        <field name="name">S.U.M</field>
        <field name="category_id" ref="uom.product_uom_categ_unit"/>
        <field name="uom_type">bigger</field>
      </record>
    </data>
</odoo>

```

## File: models\account_chart_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.http import request


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def _load(self, sale_tax_rate, purchase_tax_rate, company):
        """ Set tax calculation rounding method required in Chilean localization"""
        res = super()._load(sale_tax_rate, purchase_tax_rate, company)
        if company.account_fiscal_country_id.code == 'CL':
            company.write({'tax_calculation_rounding_method': 'round_globally'})
        return res

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.exceptions import ValidationError
from odoo import models, fields, api, _
from odoo.osv import expression

SII_VAT = '60805000-0'


class AccountMove(models.Model):
    _inherit = "account.move"

    partner_id_vat = fields.Char(related='partner_id.vat', string='VAT No')
    l10n_latam_internal_type = fields.Selection(
        related='l10n_latam_document_type_id.internal_type', string='L10n Latam Internal Type')

    def _get_l10n_latam_documents_domain(self):
        self.ensure_one()
        if self.journal_id.company_id.account_fiscal_country_id != self.env.ref('base.cl') or not \
                self.journal_id.l10n_latam_use_documents:
            return super()._get_l10n_latam_documents_domain()
        if self.journal_id.type == 'sale':
            domain = [('country_id.code', '=', 'CL')]
            if self.move_type in ['in_invoice', 'out_invoice']:
                domain += [('internal_type', 'in', ['invoice', 'debit_note', 'invoice_in'])]
            elif self.move_type in ['in_refund', 'out_refund']:
                domain += [('internal_type', '=', 'credit_note')]
            if self.company_id.partner_id.l10n_cl_sii_taxpayer_type == '1':
                domain += [('code', '!=', '71')]  # Companies with VAT Affected doesn't have "Boleta de honorarios Electrónica"
            return domain
        if self.move_type == 'in_refund':
            internal_types_domain = ('internal_type', '=', 'credit_note')
        else:
            internal_types_domain = ('internal_type', 'in', ['invoice', 'debit_note', 'invoice_in'])
        domain = [
            ('country_id.code', '=', 'CL'),
            internal_types_domain,
        ]
        if self.partner_id.l10n_cl_sii_taxpayer_type == '1' and self.partner_id_vat != '60805000-0':
            domain += [('code', 'not in', ['39', '70', '71', '914', '911'])]
        elif self.partner_id.l10n_cl_sii_taxpayer_type == '1' and self.partner_id_vat == '60805000-0':
            domain += [('code', 'not in', ['39', '70', '71'])]
        elif self.partner_id.l10n_cl_sii_taxpayer_type == '2':
            domain += [('code', 'in', ['70', '71', '56', '61'])]
        elif self.partner_id.l10n_cl_sii_taxpayer_type == '3':
            domain += [('code', 'in', ['35', '38', '39', '41', '56', '61'])]
        elif self.partner_id.country_id.code != 'CL' or self.partner_id.l10n_cl_sii_taxpayer_type == '4':
            domain += [('code', '=', '46')]
        else:
            domain += [('code', 'in', [])]
        return domain

    def _check_document_types_post(self):
        for rec in self.filtered(
                lambda r: r.company_id.account_fiscal_country_id.code == "CL" and
                          r.journal_id.type in ['sale', 'purchase']):
            tax_payer_type = rec.partner_id.l10n_cl_sii_taxpayer_type
            vat = rec.partner_id.vat
            country_id = rec.partner_id.country_id
            latam_document_type_code = rec.l10n_latam_document_type_id.code
            if (rec.journal_id.type == 'purchase' and tax_payer_type == '4' and country_id.code != 'CL' and
                latam_document_type_code == '61' and
               '46' in rec.l10n_cl_reference_ids.mapped('l10n_cl_reference_doc_type_selection')):
                continue
            if (not tax_payer_type or not vat) and (country_id.code == "CL" and latam_document_type_code
                                                  and latam_document_type_code not in ['35', '38', '39', '41']):
                raise ValidationError(_('Tax payer type and vat number are mandatory for this type of '
                                        'document. Please set the current tax payer type of this customer'))
            if rec.journal_id.type == 'sale' and rec.journal_id.l10n_latam_use_documents:
                if country_id.code != "CL":
                    if not ((tax_payer_type == '4' and latam_document_type_code in ['110', '111', '112']) or (
                            tax_payer_type == '3' and latam_document_type_code in ['39', '41', '61', '56'])):
                        raise ValidationError(_(
                            'Document types for foreign customers must be export type (codes 110, 111 or 112) or you '
                            'should define the customer as an end consumer and use receipts (codes 39 or 41)'))
            if rec.journal_id.type == 'purchase' and rec.journal_id.l10n_latam_use_documents:
                if vat != SII_VAT and latam_document_type_code == '914':
                    raise ValidationError(_('The DIN document is intended to be used only with RUT 60805000-0'
                                            ' (Tesorería General de La República)'))
                if not tax_payer_type or not vat:
                    if country_id.code == "CL" and latam_document_type_code not in [
                            '35', '38', '39', '41']:
                        raise ValidationError(_('Tax payer type and vat number are mandatory for this type of '
                                                'document. Please set the current tax payer type of this supplier'))
                if tax_payer_type == '2' and latam_document_type_code not in ['70', '71', '56', '61']:
                    raise ValidationError(_('The tax payer type of this supplier is incorrect for the selected type'
                                            ' of document.'))
                if tax_payer_type in ['1', '3']:
                    if latam_document_type_code in ['70', '71']:
                        raise ValidationError(_('The tax payer type of this supplier is not entitled to deliver '
                                                'fees documents'))
                    if latam_document_type_code in ['110', '111', '112']:
                        raise ValidationError(_('The tax payer type of this supplier is not entitled to deliver '
                                                'imports documents'))
                if (tax_payer_type == '4' or country_id.code != "CL") and latam_document_type_code != '46':
                    raise ValidationError(_('You need a journal without the use of documents for foreign '
                                            'suppliers'))

    @api.onchange('journal_id')
    def _l10n_cl_onchange_journal(self):
        if self.company_id.country_id.code == 'CL':
            self.l10n_latam_document_type_id = False

    def _post(self, soft=True):
        self._check_document_types_post()
        return super()._post(soft)

    def _l10n_cl_get_formatted_sequence(self, number=0):
        return '%s %06d' % (self.l10n_latam_document_type_id.doc_code_prefix, number)

    def _get_starting_sequence(self):
        """ If use documents then will create a new starting sequence using the document type code prefix and the
        journal document number with a 6 padding number """
        if self.journal_id.l10n_latam_use_documents and self.company_id.account_fiscal_country_id.code == "CL":
            if self.l10n_latam_document_type_id:
                return self._l10n_cl_get_formatted_sequence()
        return super()._get_starting_sequence()

    def _get_last_sequence_domain(self, relaxed=False):
        where_string, param = super(AccountMove, self)._get_last_sequence_domain(relaxed)
        if self.company_id.account_fiscal_country_id.code == "CL" and self.l10n_latam_use_documents:
            where_string = where_string.replace('journal_id = %(journal_id)s AND', '')
            where_string += ' AND l10n_latam_document_type_id = %(l10n_latam_document_type_id)s AND ' \
                            'company_id = %(company_id)s AND move_type IN %(move_type)s'

            param['company_id'] = self.company_id.id or False
            param['l10n_latam_document_type_id'] = self.l10n_latam_document_type_id.id or 0
            param['move_type'] = (('in_invoice', 'in_refund') if
                  self.l10n_latam_document_type_id._is_doc_type_vendor() else ('out_invoice', 'out_refund'))
        return where_string, param

    def _get_name_invoice_report(self):
        self.ensure_one()
        if self.l10n_latam_use_documents and self.company_id.account_fiscal_country_id.code == 'CL':
            return 'l10n_cl.report_invoice_document'
        return super()._get_name_invoice_report()

    def _l10n_cl_get_invoice_totals_for_report(self):
        self.ensure_one()
        tax_ids_filter = tax_line_id_filter = None
        include_sii = self._l10n_cl_include_sii()

        if include_sii:
            tax_ids_filter = (lambda aml, tax: bool(tax.l10n_cl_sii_code != 14))
            tax_line_id_filter = (lambda aml, tax: bool(tax.l10n_cl_sii_code != 14))

        tax_lines_data = self._prepare_tax_lines_data_for_totals_from_invoice(
            tax_ids_filter=tax_ids_filter, tax_line_id_filter=tax_line_id_filter)

        if include_sii:
            amount_untaxed = self.currency_id.round(
                self.amount_total - sum([x['tax_amount'] for x in tax_lines_data if 'tax_amount' in x]))
        else:
            amount_untaxed = self.amount_untaxed
        return self._get_tax_totals(self.partner_id, tax_lines_data, self.amount_total, amount_untaxed, self.currency_id)

    def _l10n_cl_include_sii(self):
        self.ensure_one()
        return self.l10n_latam_document_type_id.code in ['39', '41', '110', '111', '112', '34']

    def _is_manual_document_number(self):
        if self.journal_id.company_id.country_id.code == 'CL':
            return self.journal_id.type == 'purchase' and not self.l10n_latam_document_type_id._is_doc_type_vendor()
        return super()._is_manual_document_number()

```

## File: models\account_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountMoveLine(models.Model):

    _inherit = 'account.move.line'

    def _l10n_cl_prices_and_taxes(self):
        self.ensure_one()
        invoice = self.move_id
        included_taxes = self.tax_ids.filtered(lambda x: x.l10n_cl_sii_code == 14) if self.move_id._l10n_cl_include_sii() else self.tax_ids
        if not included_taxes:
            price_unit = self.tax_ids.with_context(round=False).compute_all(
                self.price_unit, invoice.currency_id, 1.0, self.product_id, invoice.partner_id)
            price_unit = price_unit['total_excluded']
            price_subtotal = self.price_subtotal
        else:
            price_unit = included_taxes.compute_all(
                self.price_unit, invoice.currency_id, 1.0, self.product_id, invoice.partner_id)['total_included']
            price = self.price_unit * (1 - (self.discount or 0.0) / 100.0)
            price_subtotal = included_taxes.compute_all(
                price, invoice.currency_id, self.quantity, self.product_id, invoice.partner_id)['total_included']
        price_net = price_unit * (1 - (self.discount or 0.0) / 100.0)

        return {
            'price_unit': price_unit,
            'price_subtotal': price_subtotal,
            'price_net': price_net
        }

```

## File: models\account_tax.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountTax(models.Model):
    _name = 'account.tax'
    _inherit = 'account.tax'

    l10n_cl_sii_code = fields.Integer('SII Code', group_operator=False)


class AccountTaxTemplate(models.Model):
    _name = 'account.tax.template'
    _inherit = 'account.tax.template'

    l10n_cl_sii_code = fields.Integer('SII Code')

    def _get_tax_vals(self, company, tax_template_to_tax):
        self.ensure_one()
        vals = super(AccountTaxTemplate, self)._get_tax_vals(company, tax_template_to_tax)
        vals.update({
            'l10n_cl_sii_code': self.l10n_cl_sii_code,
        })
        if self.tax_group_id:
            vals['tax_group_id'] = self.tax_group_id.id
        return vals

```

## File: models\l10n_latam_document_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class L10nLatamDocumentType(models.Model):

    _inherit = 'l10n_latam.document.type'

    internal_type = fields.Selection(
        selection_add=[
            ('invoice', 'Invoices'),
            ('invoice_in', 'Purchase Invoices'),
            ('debit_note', 'Debit Notes'),
            ('credit_note', 'Credit Notes'),
            ('receipt_invoice', 'Receipt Invoice')])

    def _format_document_number(self, document_number):
        """ Make validation of Import Dispatch Number
          * making validations on the document_number. If it is wrong it should raise an exception
          * format the document_number against a pattern and return it
        """
        self.ensure_one()
        if self.country_id.code != "CL":
            return super()._format_document_number(document_number)

        if not document_number:
            return False

        return document_number.zfill(6)

    def _is_doc_type_vendor(self):
        return self.code == '46'

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class ResCompany(models.Model):
    _inherit = "res.company"

    def _localization_use_documents(self):
        """ Chilean localization use documents """
        self.ensure_one()
        return self.account_fiscal_country_id.code == "CL" or super()._localization_use_documents()

```

## File: models\res_country.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResPartner(models.Model):
    _name = 'res.country'
    _inherit = 'res.country'

    l10n_cl_customs_code = fields.Char('Customs Code')
    l10n_cl_customs_name = fields.Char('Customs Name')
    l10n_cl_customs_abbreviation = fields.Char('Customs Abbreviation')

```

## File: models\res_currency.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import _, api, fields, models


class ResCurrency(models.Model):
    _name = "res.currency"
    _inherit = "res.currency"

    l10n_cl_currency_code = fields.Char('Currency Code')
    l10n_cl_short_name = fields.Char('Short Name')

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import stdnum
from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    _sii_taxpayer_types = [
        ('1', _('VAT Affected (1st Category)')),
        ('2', _('Fees Receipt Issuer (2nd category)')),
        ('3', _('End Consumer')),
        ('4', _('Foreigner')),
    ]

    l10n_cl_sii_taxpayer_type = fields.Selection(
        _sii_taxpayer_types, 'Taxpayer Type', index=True,
        help='1 - VAT Affected (1st Category) (Most of the cases)\n'
             '2 - Fees Receipt Issuer (Applies to suppliers who issue fees receipt)\n'
             '3 - End consumer (only receipts)\n'
             '4 - Foreigner')

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['l10n_cl_sii_taxpayer_type']

    def _format_vat_cl(self, values):
        identification_types = [self.env.ref('l10n_latam_base.it_vat').id, self.env.ref('l10n_cl.it_RUT').id,
                                self.env.ref('l10n_cl.it_RUN').id]
        country = self.env["res.country"].browse(values.get('country_id'))
        identification_type = self.env['l10n_latam.identification.type'].browse(
            values.get('l10n_latam_identification_type_id')
        )
        partner_country_is_chile = country.code == "CL" or identification_type.country_id.code == "CL"
        if partner_country_is_chile and \
                values.get('l10n_latam_identification_type_id') in identification_types and values.get('vat') and\
                stdnum.util.get_cc_module('cl', 'vat').is_valid(values['vat']):
            return stdnum.util.get_cc_module('cl', 'vat').format(values['vat']).replace('.', '').replace(
                'CL', '').upper()
        else:
            return values['vat']

    def _format_dotted_vat_cl(self, vat):
        vat_l = vat.split('-')
        n_vat, n_dv = vat_l[0], vat_l[1]
        return '%s-%s' % (format(int(n_vat), ',d').replace(',', '.'), n_dv)

    @api.model
    def create(self, values):
        if values.get('vat'):
            values['vat'] = self._format_vat_cl(values)
        return super().create(values)

    def write(self, values):
        if any(field in values for field in ['vat', 'l10n_latam_identification_type_id', 'country_id']):
            for record in self:
                vat_values = {
                    'vat': values.get('vat', record.vat),
                    'l10n_latam_identification_type_id': values.get(
                        'l10n_latam_identification_type_id', record.l10n_latam_identification_type_id.id),
                    'country_id': values.get('country_id', record.country_id.id)
                }
                values['vat'] = self._format_vat_cl(vat_values)
        return super().write(values)

```

## File: models\res_partner_bank.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResBank(models.Model):
    _name = 'res.bank'
    _inherit = 'res.bank'

    l10n_cl_sbif_code = fields.Char('Cod. SBIF', size=10)

```

## File: models\uom_uom.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _


class UomUom(models.Model):
    _inherit = 'uom.uom'

    l10n_cl_sii_code = fields.Char('SII Code')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_chart_template
from . import account_move
from . import account_move_line
from . import account_tax
from . import l10n_latam_document_type
from . import res_company
from . import res_country
from . import res_currency
from . import res_partner
from . import res_partner_bank
from . import uom_uom

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="6.44" y="6.23" width="52.36" height="34.87" maskUnits="userSpaceOnUse">
      <rect x="6.44" y="7.48" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
    </mask>
    <symbol id="c" data-name="account icon" viewBox="0 0 106 106">
      <g style="mask: url(#a)">
        <g>
          <path d="M0,0H106V106H0Z" style="fill: #5a5a64;fill-rule: evenodd"/>
          <path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill: #fff;fill-opacity: 0.382999986410141;fill-rule: evenodd"/>
          <path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity: 0.382999986410141;fill-rule: evenodd"/>
          <g>
            <path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill: #393939;fill-rule: evenodd;opacity: 0.324000000953674;isolation: isolate"/>
            <g style="opacity: 0.30000000000000004">
              <g>
                <path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/>
                <path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/>
                <path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/>
                <path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/>
                <path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/>
                <path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/>
                <path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/>
              </g>
              <path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/>
            </g>
            <path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill: #a8a9ab"/>
            <g>
              <path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill: #a8a9ab"/>
              <path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill: #a8a9ab"/>
              <path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill: #a8a9ab"/>
              <path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill: #a8a9ab"/>
              <path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill: #a8a9ab"/>
              <path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill: #a8a9ab"/>
              <path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill: #a8a9ab"/>
            </g>
          </g>
        </g>
      </g>
    </symbol>
  </defs>
  <g>
    <use width="106" height="106" transform="translate(-0.07 0)" xlink:href="#c"/>
    <rect x="6.76" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="1500" height="1000" transform="translate(6.44 6.23) scale(0.03 0.03)" xlink:href="data:image/png;base64,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AAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAATOmbl3IwAAAAAAYOn5AWfU2y7vlCiFAAAAAElFTkSuQmCC"/>
    </g>
  </g>
</svg>

```

## File: views\account_move_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_move_form_inherit_l10n_cl" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n.cl</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <form>
                <field name="l10n_latam_internal_type" invisible="1"/>
            </form>
        </field>
    </record>

    <record id="view_latam_form_inherit_l10n_cl" model="ir.ui.view">
        <field name="name">account.move.latam.form.inherit.l10n.cl</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_move_form"/>
        <field name="arch" type="xml">
            <field name="l10n_latam_document_number" position="attributes">
                <attribute name="attrs">{
                    'invisible': [
                    '|',
                        ('l10n_latam_use_documents', '=', False),
                        ('l10n_latam_manual_document_number', '=', False),
                    '|', '|',
                        ('l10n_latam_use_documents', '=', False),
                        ('highest_name', '!=', False),
                        ('state', '!=', 'draft'),
                    '|', '|', '|',
                        ('l10n_latam_use_documents', '=', False),
                        ('posted_before', '=', False),
                        ('state', '!=', 'draft'),
                        ('country_code', '!=', 'CL')
                    ],
                    'readonly': [('posted_before', '=', True), ('state', '!=', 'draft')],
                    'required': ['|', ('l10n_latam_manual_document_number', '=', True), ('highest_name', '=', False)]}
                </attribute>
            </field>
        </field>
    </record>

    <record id="view_complete_invoice_refund_tree" model="ir.ui.view">
        <field name="name">account.move.tree2</field>
        <field name="model">account.move</field>
        <field name="arch" type="xml">
            <tree decoration-info="state == 'draft'" default_order="create_date" string="Invoices and Refunds" decoration-muted="state == 'cancel'" js_class="account_tree">
                <field name="l10n_latam_document_type_id_code"/>
                <field name="l10n_latam_document_number" string="Folio"/>
                <field name="partner_id_vat"/>
                <field name="partner_id"/>
                <field name="invoice_date" optional="show"/>
                <field name="invoice_date_due" optional="show"/>
                <field name="date" string="Accounting Date" optional="show"/>
                <field name="payment_reference" optional="hide"/>
                <field name="invoice_user_id" optional="show" invisible="context.get('default_move_type') not in ('out_invoice', 'out_refund','out_receipt')" string="Sales Person"/>
                <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" optional="show"/>
                <field name="invoice_origin" optional="show" string="Source Document"/>
                <field name="amount_untaxed_signed" string="Amount Untaxed" sum="Total" optional="show"/>
                <field name="amount_tax_signed" string="Tax" sum="Total" optional="show"/>
                <field name="amount_total_signed" string="Total" sum="Total" optional="show"/>
                <field name="amount_residual_signed" string="Amount Due" sum="Amount Due" optional="show"/>
                <field name="currency_id" invisible="1"/>
                <field name="company_currency_id" invisible="1"/>
                <field name="state" optional="show"/>
                <field name="payment_state" optional="hide"/>
                <field name="move_type" invisible="context.get('default_move_type', True)"/>
            </tree>
        </field>
    </record>

    <record model="ir.actions.act_window" id="sale_invoices_credit_notes">
        <field name="name">Sale Invoices and Credit Notes</field>
        <field name="view_id" ref="view_complete_invoice_refund_tree"/>
        <field name="res_model">account.move</field>
        <field name="domain">[('move_type', 'in', ['out_invoice', 'out_refund'])]</field>
        <field name="context">{'default_move_type': 'out_invoice'}</field>
        <field name="target">current</field>
        <field name="view_mode">tree,form</field>
    </record>

    <record model="ir.actions.act_window" id="vendor_bills_and_refunds">
        <field name="name">Vendor Bills and Refunds</field>
        <field name="view_id" ref="view_complete_invoice_refund_tree"/>
        <field name="res_model">account.move</field>
        <field name="domain">[('move_type', 'in', ['in_invoice', 'in_refund'])]</field>
        <field name="context">{'default_move_type': 'in_invoice'}</field>
        <field name="target">current</field>
        <field name="view_mode">tree,form</field>
    </record>

    <menuitem id="menu_sale_invoices_credit_notes" parent="account.menu_finance_receivables" sequence="3" action="sale_invoices_credit_notes" name="Sale Invoices and Credit Notes (CL)"/>
    <menuitem id="menu_vendor_bills_and_refunds" parent="account.menu_finance_payables" sequence="3" action="vendor_bills_and_refunds" name="Vendor Bills and Refunds (CL)"/>

</odoo>

```

## File: views\account_tax_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_tax_form" model="ir.ui.view">
            <field name="name">account.tax.form</field>
            <field name="model">account.tax</field>
            <field name="inherit_id" ref="account.view_tax_form"/>
            <field name="arch" type="xml">
                <field name="name" position="after">
                    <field name="l10n_cl_sii_code" options="{'format': false}" attrs="{'invisible': [('country_code', '!=', 'CL')]}"/>
                </field>
            </field>
        </record>

        <record id="view_tax_sii_code_tree" model="ir.ui.view">
            <field name="name">account.tax.sii.code.tree</field>
            <field name="model">account.tax</field>
            <field name="inherit_id" ref="account.view_tax_tree" />
            <field name="arch" type="xml">
                <field name="name" position="before">
                    <field name="l10n_cl_sii_code" options="{'format': false}"/>
                </field>
            </field>
        </record>

        <record id="view_account_tax_template_form" model="ir.ui.view">
            <field name="name">account.tax.template.form</field>
            <field name="model">account.tax.template</field>
            <field name="inherit_id" ref="account.view_account_tax_template_form"/>
            <field name="arch" type="xml">
                <field name="name" position="after">
                    <field name="l10n_cl_sii_code" options="{'format': false}"/>
                </field>
            </field>
        </record>

        <record id="view_account_tax_template_tree" model="ir.ui.view">
            <field name="name">account.tax.template.sii.tree</field>
            <field name="model">account.tax.template</field>
            <field name="inherit_id" ref="account.view_account_tax_template_tree" />
            <field name="arch" type="xml">
                <field name="name" position="before">
                    <field name="l10n_cl_sii_code" options="{'format': false}"/>
                </field>
            </field>
        </record>
    </data>
</odoo>
```

## File: views\l10n_latam_document_type_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_cl_latam_document_type_view" model="ir.ui.view">
        <field name="name">l10n.cl.latam.document.type.view</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_document_type_form"/>
        <field name="arch" type="xml">
            <field name="sequence" position="attributes">
                    <attribute name="widget">handle</attribute>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- this header can be used on any Chilean report -->
    <template id="custom_header">

        <t t-set="report_date" t-value="o.invoice_date"/>
        <t t-set="report_number" t-value="int(o.l10n_latam_document_number)"/>
        <t t-set="pre_printed_report" t-value="report_type == 'pdf'"/>
        <t t-set="report_name" t-value="o.l10n_latam_document_type_id.name"/>
        <t t-set="header_address" t-value="o.company_id.partner_id"/>
        <t t-set="custom_footer">
            <t t-call="l10n_cl.custom_footer"/>
        </t>

        <div>
            <div class="row">
                <div name="left-upper-side" class="col-8">
                    <img t-if="o.company_id.logo" t-att-src="image_data_uri(o.company_id.logo)"
                         style="max-height: 45px;" alt="Logo"/>
                    <br/>
                    <strong>
                        <span t-field="o.company_id.partner_id.name"/>
                    </strong>
                    <br/>
                    <span name="company_activity" class="font-italic" t-field="o.company_id.report_header"/>
                    <div/>
                    <t t-esc="' - '.join([item for item in [
                        ', '.join([item for item in [header_address.street, header_address.street2] if item]),
                        header_address.city,
                        header_address.state_id and header_address.state_id.name,
                        header_address.zip,
                        header_address.country_id and header_address.country_id.name] if item])"/>
                    <span t-if="header_address.phone">
                        <br/>
                    </span>
                    <span t-if="header_address.phone" style="white-space: nowrap;"
                          t-esc="'Tel: ' + header_address.phone"/>
                    <span t-if="header_address.website">
                        <span t-att-style="'color: %s;' % o.company_id.primary_color"
                              t-esc="'- Web: %s' %' - '.join([item for item in [header_address.website.replace('https://', '').replace('http://', ''), header_address.email] if item])"/>
                    </span>
                </div>
                <div name="right-upper-side" class="col-4">
                    <div class="row">
                        <div name="right-upper-side" class="col-12">
                            <div class="row border border-dark">
                                <div class="col-12 text-center">
                                    <h6 t-att-style="'color: %s;' % o.company_id.primary_color">
                                        <strong t-att-style="'color: %s;' % o.company_id.primary_color">
                                            <br/>
                                            <span style="line-height: 180%;">RUT:</span>
                                            <t t-if="o.company_id.partner_id.vat">
                                                <span t-esc="o.company_id.partner_id._format_dotted_vat_cl(o.company_id.partner_id.vat)"/>
                                            </t>
                                            <br/>
                                            <span class="text-uppercase" t-esc="report_name"/>
                                            <br/>
                                            <span>Nº:</span>
                                            <span style="line-height: 200%;" t-esc="report_number"/>
                                        </strong>
                                    </h6>
                                </div>
                            </div>
                            <div class="row text-center">
                                <div class="col-12 text-center" t-att-style="'color: %s;' % o.company_id.primary_color"
                                     name="regional-office"/>
                            </div>
                        </div>
                    </div>
                </div>

            </div>

        </div>
    </template>


    <template id="informations">
        <div id="informations" class="row mt8 mb8">
            <div class="col-6">
                <strong>
                    <span t-att-style="'color: %s;' % o.company_id.secondary_color">Date:</span>
                </strong>
                <span t-esc="o.invoice_date" t-options='{"widget": "date"}'/>
                <br/>

                <strong>Customer:</strong>
                <span t-field="o.partner_id.name"/>
                <br/>

                <t t-if="o.partner_id.vat and o.partner_id.l10n_latam_identification_type_id">
                    <strong>
                        <t t-esc="o.partner_id.l10n_latam_identification_type_id.name or o.company_id.account_fiscal_country_id.vat_label" id="inv_tax_id_label"/>:
                    </strong>
                    <span t-esc="o.partner_id.vat"/>
                    <br/>
                </t>
                <strong>GIRO:</strong>
                <span t-esc="o.partner_id.industry_id.name or ''"/>
                <br/>
            </div>
            <div class="col-6">
                <strong>Due Date:</strong>
                <span t-esc="o.invoice_date_due" t-options='{"widget": "date"}'/>
                <br/>
                <strong>Address:</strong>
                <span t-field="o.partner_id"
                      t-options="{'widget': 'contact', 'fields': ['address'], 'no_marker': true, 'no_tag_br': True}"/>

                <strong>Payment Terms:</strong>
                <span t-esc="o.invoice_payment_term_id.name or ''"/>


                <t t-if="o.invoice_incoterm_id">
                    <br/>
                    <strong>Incoterm:</strong>
                    <span t-field="o.invoice_incoterm_id.name"/>
                </t>

            </div>
        </div>
        <div id="references" class="row">
            <div name="references" class="col-12 text-center"/>
        </div>
    </template>

    <template id="custom_footer">
        <div name="footer_left_column" class="col-8 text-center"/>
    </template>

    <template id="report_invoice_document" inherit_id="account.report_invoice_document" primary="True">

        <t t-set="o" position="after">
            <t t-set="custom_header" t-value="'l10n_cl.custom_header'"/>
        </t>

        <!-- remove default partner address -->
        <t t-set="address" position="replace"/>

        <xpath expr="//h2" position="replace"/>

        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" position="before">
            <t t-set="l10n_cl_values" t-value="line._l10n_cl_prices_and_taxes()"/>
        </t>

        <xpath expr="//span[@t-field='line.price_unit']" position="attributes">
            <attribute name="t-field"></attribute>
            <attribute name="t-esc">line.price_unit</attribute>
            <attribute name="t-options">{"widget": "monetary", "display_currency": o.currency_id}</attribute>
        </xpath>

        <xpath expr="//span[@id='line_tax_ids']" position="attributes">
            <attribute name="t-esc">', '.join(map(lambda x: (x.description or x.name), line.tax_lines))</attribute>
        </xpath>

        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" position="attributes">
            <attribute name="t-value">current_subtotal + l10n_cl_values['price_subtotal']</attribute>
        </t>

        <xpath expr="//th[@name='th_subtotal']/span[@groups='account.group_show_line_subtotals_tax_included']" position="replace">
            <span groups="account.group_show_line_subtotals_tax_included">Amount</span>
        </xpath>

        <span t-field="line.price_subtotal" position="attributes">
            <attribute name="t-field"></attribute>
            <attribute name="t-esc">l10n_cl_values['price_subtotal']</attribute>
            <attribute name="t-options">{"widget": "monetary", "display_currency": o.currency_id}</attribute>
        </span>

        <t t-set="tax_totals" position="attributes">
            <attribute name="t-value">o._l10n_cl_get_invoice_totals_for_report()</attribute>
        </t>

        <xpath expr="//th[@name='th_taxes']" position="replace"/>
        <xpath expr="//span[@id='line_tax_ids']/.." position="replace"/>

        <p name="payment_term" position="replace"/>
        <xpath expr="//span[@t-field='o.payment_reference']/../.." position="replace"/>

        <!-- replace information section and usage chilean style -->
        <div id="informations" position="replace">
            <t t-call="l10n_cl.informations"/>
        </div>

        <!--  we remove the ml auto and also give more space to avoid multiple lines on tax detail -->
        <xpath expr="//div[@id='total']/div" position="attributes">
            <attribute name="t-attf-class">#{'col-6' if report_type != 'html' else 'col-sm-7 col-md-6'}</attribute>
        </xpath>

        <xpath expr="//div[@id='total']/div" position="before">
            <div t-attf-class="#{'col-6' if report_type != 'html' else 'col-sm-7 col-md-6'}"/>
        </xpath>

        <xpath expr="//div[@id='total']" position="after">
            <div class="row">
                <div name="stamp" class="col-4 text-center"/>
                <div name="transferable-table" class="col-4"/>
                <div name="transferable-legend" class="col-4 pull-right"/>
            </div>
        </xpath>

    </template>

    <!-- FIXME: Temp fix to allow fetching invoice_documemt in Studio Reports with localisation -->
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-if="o._get_name_invoice_report() == 'l10n_cl.report_invoice_document'"
                t-call="l10n_cl.report_invoice_document" t-lang="lang"/>
        </xpath>
    </template>

    <template id="report_invoice_with_payments" inherit_id="account.report_invoice_with_payments">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-if="o._get_name_invoice_report() == 'l10n_cl.report_invoice_document'"
                t-call="l10n_cl.report_invoice_document" t-lang="lang"/>
        </xpath>
    </template>

</odoo>

```

## File: views\res_bank_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record model="ir.ui.view" id="view_res_bank_form">
            <field name="name">res.bank.form</field>
            <field name="model">res.bank</field>
            <field name="inherit_id" ref="base.view_res_bank_form" />
            <field name="arch" type="xml">
                <field name="name" position="before">
                    <field name="l10n_cl_sbif_code" />
                </field>
            </field>
        </record>

        <record model="ir.ui.view" id="view_res_bank_tree">
            <field name="name">bank.bank.tree</field>
            <field name="model">res.bank</field>
            <field name="inherit_id" ref="base.view_res_bank_tree" />
            <field name="arch" type="xml">
                <field name="name" position="before">
                    <field name="l10n_cl_sbif_code" />
                </field>
            </field>
        </record>

    </data>
</odoo>
```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.chilean.loc</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='invoicing_settings']" position="after">
                <div id="l10n_cl_title" attrs="{'invisible': True}">
                    <h2>Chilean Localization</h2>
                </div>
                <div id="l10n_cl_section" class="row mt16 o_settings_container" attrs="{'invisible': [('country_code', '!=', 'CL')]}">
                    <!-- inside empty to add configuration of tags -->
                </div>
            </xpath>
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
                <field name="l10n_cl_customs_name"/>
                <field name="l10n_cl_customs_code"/>
                <field name="l10n_cl_customs_abbreviation"/>
            </field>
        </field>
    </record>

    <record id="view_res_country_tree" model="ir.ui.view">
        <field name="name">res.country.tree</field>
        <field name="model">res.country</field>
        <field name="inherit_id" ref="base.view_country_tree"/>
        <field name="arch" type="xml">
            <field name="code" position="after">
                <field name="l10n_cl_customs_name"/>
                <field name="l10n_cl_customs_code"/>
                <field name="l10n_cl_customs_abbreviation"/>
            </field>
        </field>
    </record>
</odoo>
```

## File: views\res_partner.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_move_form" model="ir.ui.view">
        <field name="name">res.partner.placeholders.l10n_cl.form</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="model">res.partner</field>
        <field name="arch" type="xml">
            <field name="street2" position="attributes">
                <attribute name="placeholder">Datos adic. dirección y Ciudad</attribute>
            </field>
            <field name="city" position="attributes">
                <attribute name="placeholder">Comuna</attribute>
            </field>
            <field name="state_id" position="attributes">
                <attribute name="placeholder">Región</attribute>
            </field>
            <field name="vat" position="after">
                <field name="l10n_cl_sii_taxpayer_type" attrs="{'readonly': [('parent_id', '!=', False)]}"/>
            </field>
        </field>
    </record>

</odoo>

```

