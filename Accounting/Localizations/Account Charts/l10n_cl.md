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
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Chile - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['cl'],
    'version': '3.1',
    'description': """
Chilean accounting chart and tax localization.
Plan contable chileno e impuestos de acuerdo a disposiciones vigentes.
    """,
    'author': 'Blanco Martín & Asociados',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/chile.html',
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'contacts',
        'base_vat',
        'l10n_latam_base',
        'l10n_latam_invoice_document',
        'uom',
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'views/account_move_view.xml',
        'views/account_tax_view.xml',
        'views/res_bank_view.xml',
        'views/res_country_view.xml',
        'views/res_company_view.xml',
        'views/report_invoice.xml',
        'views/res_partner.xml',
        'views/res_config_settings_view.xml',
        'data/l10n_cl_chart_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_tags_data.xml',
        'data/l10n_latam_identification_type_data.xml',
        'data/l10n_latam.document.type.csv',
        'data/product_data.xml',
        'data/uom_data.xml',
        'data/res.currency.csv',
        'data/res_currency_data.xml',
        'data/res.bank.csv',
        'data/res.country.csv',
        'data/res_partner.xml',
    ],
    'demo': [
        'demo/partner_demo.xml',
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.cl"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_base_imponible_ventas" model="account.report.line">
                <field name="name">Taxable Sales Base</field>
                <field name="expression_ids">
                    <record id="tax_report_base_imponible_ventas_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Base Imponible Ventas</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_ventas_exentas" model="account.report.line">
                        <field name="name">Exempt Sales</field>
                        <field name="expression_ids">
                            <record id="tax_report_ventas_exentas_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Ventas Exentas</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_impuestos_renta" model="account.report.line">
                        <field name="name">First Category Income Taxes Payable</field>
                        <field name="expression_ids">
                            <record id="tax_report_impuestos_renta_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Impuesto a la Renta Primera Categoría a Pagar</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_retencion_total_compras" model="account.report.line">
                <field name="name">Total retention (purchases)</field>
                <field name="expression_ids">
                    <record id="tax_report_retencion_total_compras_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Retención Total (compras)</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ventas_netas_gravadas_c_iva" model="account.report.line">
                <field name="name">Net Sales Taxed with VAT</field>
                <field name="expression_ids">
                    <record id="tax_report_ventas_netas_gravadas_c_iva_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ventas Netas Gravadas con IVA</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_impuestos_originados_venta" model="account.report.line">
                <field name="name">Sales Tax</field>
                <field name="expression_ids">
                    <record id="tax_report_impuestos_originados_venta_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Impuesto Originado por la Venta</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_iva_debito_fiscal" model="account.report.line">
                <field name="name">VAT Tax Debit</field>
                <field name="expression_ids">
                    <record id="tax_report_iva_debito_fiscal_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">IVA Debito Fiscal</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ppm" model="account.report.line">
                <field name="name">PPM</field>
                <field name="expression_ids">
                    <record id="tax_report_ppm_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">PPM</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_netas_gr_iva_recup" model="account.report.line">
                <field name="name">Net Purchases Taxed with VAT (recoverable)</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_netas_gr_iva_recup_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras Netas Gravadas Con IVA (recuperable)</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_netas_gr_iva_uso_comun" model="account.report.line">
                <field name="name">Purchase Engraved Nets With VAT Communal Use</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_netas_gr_iva_uso_comun_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compra Netas Gravadas Con IVA Uso Comun</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_netas_gr_iva_no_recuperable" model="account.report.line">
                <field name="name">Purchases Non-recoverable VAT</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_netas_gr_iva_no_recuperable_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras IVA No Recuperable</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_supermercado" model="account.report.line">
                <field name="name">Supermarket Shopping</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_supermercado_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras De Supermercado</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_activo_fijo" model="account.report.line">
                <field name="name">Purchases of fixed assets</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_activo_fijo_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras de Activo Fijo</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_activo_fijo_uso_comun" model="account.report.line">
                <field name="name">Purchases of Fixed Assets Common Use</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_activo_fijo_uso_comun_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras de Activo Fijo Uso Común</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_activo_fijo_no_recup" model="account.report.line">
                <field name="name">Purchases of non-recoverable fixed assets</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_activo_fijo_no_recup_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras de Activo Fijo No Recuperable</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_no_gravadas_iva" model="account.report.line">
                <field name="name">Purchases Not Taxed With VAT</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_no_gravadas_iva_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras No Gravadas Con IVA</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_impuestos_pagados_compra" model="account.report.line">
                <field name="name">Taxes Paid on Purchase</field>
                <field name="expression_ids">
                    <record id="tax_report_impuestos_pagados_compra_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Impuestos Pagados en la Compra</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_iva_recup" model="account.report.line">
                <field name="name">VAT Paid Purchases Recoverable</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_iva_recup_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">IVA Pagado Compras Recuperables</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_iva_uso_comun" model="account.report.line">
                <field name="name">VAT Paid Purchases Common Use</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_iva_uso_comun_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">IVA Pagado Compras Uso Común</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_iva_no_recup" model="account.report.line">
                <field name="name">VAT Paid Not Recoverable</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_iva_no_recup_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">IVA Pagado No Recuperable</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_iva_supermercado" model="account.report.line">
                <field name="name">VAT Paid Supermarket Purchases</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_iva_supermercado_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">IVA Pagado Compras Supermercado</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_iva_activo_fijo" model="account.report.line">
                <field name="name">Purchases Fixed Assets</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_iva_activo_fijo_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras Activo Fijo</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_iva_activo_fijo_uso_comun" model="account.report.line">
                <field name="name">Purchases of Fixed Assets Common Use</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_iva_activo_fijo_uso_comun_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras Activo Fijo Uso Común</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_compras_iva_activo_fijo_no_recup" model="account.report.line">
                <field name="name">Purchases of Non Recoverable Fixed Assets</field>
                <field name="expression_ids">
                    <record id="tax_report_compras_iva_activo_fijo_no_recup_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras Activo Fijo No Recuperables</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_retencion_segunda_categ" model="account.report.line">
                <field name="name">Second Category Withholding</field>
                <field name="expression_ids">
                    <record id="tax_report_retencion_segunda_categ_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Retención Segunda Categoría</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_base_retencion_segunda_categ" model="account.report.line">
                <field name="name">Second Category Withholding Base</field>
                <field name="expression_ids">
                    <record id="tax_report_base_retencion_segunda_categ_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Base Retención Segunda Categoría</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_base_ila_compras" model="account.report.line">
                <field name="name">ILA Withholding Base (purchases)</field>
                <field name="expression_ids">
                    <record id="tax_report_base_ila_compras_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Base Retenciones ILA (compras)</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_tax_ila_compras" model="account.report.line">
                <field name="name">Ret Suffered Tax ILA (purchases)</field>
                <field name="expression_ids">
                    <record id="tax_report_tax_ila_compras_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Retenciones ILA (compras)</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_base_ila_ventas" model="account.report.line">
                <field name="name">ILA Withholding Base (sales)</field>
                <field name="expression_ids">
                    <record id="tax_report_base_ila_ventas_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Base Retenciones ILA (ventas)</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_tax_ila_ventas" model="account.report.line">
                <field name="name">ILA Tax Ret Practiced (sales)</field>
                <field name="expression_ids">
                    <record id="tax_report_tax_ila_ventas_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Retenciones ILA (ventas)</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_base_compras_combustibles" model="account.report.line">
                <field name="name">Fuel Purchases (Base)</field>
                <field name="expression_ids">
                    <record id="tax_report_base_compras_combustibles_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Compras Des Combustibles</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_tax_compras_combustibles" model="account.report.line">
                <field name="name">Fuel Purchases (Tax)</field>
                <field name="expression_ids">
                    <record id="tax_report_tax_compras_combustibles_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">IEC Compras Des Combustibles</field>
                    </record>
                </field>
            </record>
        </field>
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
            <field name="name">Ventas - Amount VAT</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_mnt_fuera_plazo" model="account.account.tag">
            <!-- TotIVAFueraPlazo -->
            <field name="name">Sales - VAT Out of Time</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_iva_propio" model="account.account.tag">
            <!-- TotIVAPropio -->
            <field name="name">Sales - Own VAT</field>
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
            <field name="name">Sales - VAT Law 18211</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_otros_imp" model="account.account.tag">
            <!-- TotOtrosImp -->
            <field name="name">Sales - Other Taxes</field>
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
            <field name="name">Sales - Special Credit 65% Construction Companies</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_dep_env" model="account.account.tag">
            <!-- TotDepEnvase -->
            <field name="name">Sales - Container Depot</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_sale_iva_comisiones" model="account.account.tag">
            <!-- TotValComIVA -->
            <field name="name">Sales - VAT Commissions and Other Charges</field>
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
            <field name="name">Purchases - Amount of Recoverable VAT</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva_actf" model="account.account.tag">
            <!-- TotMntIVAActivoFijo -->
            <field name="name">Purchases - Fixed VAT Amount</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva_actf_uso_comun" model="account.account.tag">
            <!-- TotMntIVAActivoFijo -->
            <field name="name">Purchases - Amount of Active Fixed VAT (Common Use)</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva_actf_no_recup" model="account.account.tag">
            <!-- TotMntIVAActivoFijo -->
            <field name="name">Purchases - Amount of Active Fixed VAT (Non Deductible)</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>


        <record id="tag_cl_purchase_mnt_iva_no_rec" model="account.account.tag">
            <!-- TotMntIVANoRec -->
            <field name="name">Purchases - Non Recoverable VAT Amount</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva_uso_comun" model="account.account.tag">
            <!-- TotMntIVANoRec -->
            <field name="name">Purchases - Amount VAT Commun Use</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iva_supermercado" model="account.account.tag">
            <field name="name">Shopping - VAT amount for Supermarket purchases</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_otros_imp" model="account.account.tag">
            <!-- TotOtrosImp -->
            <field name="name">Purchases - Amount Other Taxes</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_imp_sin_credito" model="account.account.tag">
            <!-- TotImpSinCredito -->
            <field name="name">Purchases - Taxes Without Credit</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_iva_no_ret" model="account.account.tag">
            <!-- TotIVANoRetenido -->
            <field name="name">Purchases - VAT Not Withheld Purchasing fac</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_imp_42" model="account.account.tag">
            <!-- TotCredImp -->
            <field name="name">Purchases - Credit Imp Art 42</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_imp_sin_cred" model="account.account.tag">
            <!-- TotImpSinCredito -->
            <field name="name">Purchases - Tax No Credit</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_iva_no_reten" model="account.account.tag">
            <!-- TotIVANoRetenido -->
            <field name="name">Purchases - VAT Not Withheld</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_imp_vehic" model="account.account.tag">
            <!-- TotImpVehiculo -->
            <field name="name">Purchases - Motor Vehicle Tax</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.cl"/>
        </record>

        <record id="tag_cl_purchase_mnt_iec" model="account.account.tag">
            <!-- TotMntIEC -->
            <field name="name">Purchases - Specific Fuel Tax</field>
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
        <field name="name">Sales - Total Amount Exempt or Unrecorded</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_mnt_neto" model="account.account.tag">
        <!-- TotMntNeto -->
        <field name="name">Sales - Total Net Amount</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_valor_neto_comis" model="account.account.tag">
        <!-- TotValComNeto -->
        <field name="name">Sales - Total Net Value Commissions and Other Charges</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_valor_comisiones_no_afecto" model="account.account.tag">
        <!-- TotValComExe -->
        <field name="name">Sales - Total Net Value Commissions and Other Unaffected Charges</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_monto_no_facturable" model="account.account.tag">
        <!-- TotMntNoFact -->
        <field name="name">Sales - Total Non-Billable Amount</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_exento_vta_pasajes_nacional" model="account.account.tag">
        <!-- TotPsjNac -->
        <field name="name">Sales - Exempt Domestic Ticket Sales</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_sale_exento_vta_pasajes_internacional" model="account.account.tag">
        <!-- TotPsjInt -->
        <field name="name">Sales - Exempt Sales of International Tickets</field>
        <field name="applicability">accounts</field>
    </record>

    <!-- Account Taxes Tags Commpras -->
    <record id="tag_cl_purchase_mnt_exe" model="account.account.tag">
        <!-- TotMntExe -->
        <field name="name">Shopping Cart - Total Exempt or Unrecorded Amount</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_mnt_neto" model="account.account.tag">
        <!-- TotMntNeto -->
        <field name="name">Purchases - Total Net Amount</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_mnt_neto_uso_comun" model="account.account.tag">
        <!-- TotMntNeto -->
        <field name="name">Purchases - Total Net Amount Common Use</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_mnt_neto_no_recup" model="account.account.tag">
        <!-- TotMntNeto -->
        <field name="name">Purchases - Total Net Amount not Recoverable</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_mnt_neto_supermercado" model="account.account.tag">
        <!-- TotMntNeto -->
        <field name="name">Purchases - Total Net Amount Supermarket Purchases</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_mnt_neto_actf" model="account.account.tag">
        <!-- TotMntActivoFijo -->
        <field name="name">Purchases - Total Nets Amount Fixed Assets</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_tab_puros" model="account.account.tag">
        <!-- TotTabPuros -->
        <field name="name">Purchases - Total Manufactured Cigars</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_tab_cigar" model="account.account.tag">
        <!-- TotTabCigarrillos -->
        <field name="name">Purchases - Total Manufactured Tobacco Cigarettes</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_purchase_tab_elab" model="account.account.tag">
        <!-- TotTabCigarrillos -->
        <field name="name">Purchases - Total Manufactured Tobacco Products</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_remanente_cf" model="account.account.tag">
        <field name="name">Remaining Tax Credit</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_impuesto_unico_trabajadores" model="account.account.tag">
        <field name="name">Single Tax Workers</field>
        <field name="applicability">accounts</field>
    </record>

    <!-- honorarios y otros -->
    <record id="tag_cl_fees_amount" model="account.account.tag">
        <field name="name">Fees - Amounts Subject to 2 Category Income Tax Withholding</field>
        <field name="applicability">accounts</field>
    </record>

    <record id="tag_cl_specific_fuel_tax" model="account.account.tag">
        <field name="name">Specific Fuel Tax</field>
        <field name="applicability">accounts</field>
    </record>

</odoo>

```

## File: data\l10n_latam.document.type.csv

```csv
id,sequence,code,name,report_name,internal_type,doc_code_prefix,country_id/id,l10n_cl_active,active
dc_a_f_dte,1,33,Electronic Invoice,INVOICE,invoice,FAC,base.cl,True,True
dc_y_f_dte,2,34,Unaffected or Exempt Electronic Invoice,F-EXENTA,invoice,FNA,base.cl,True,True
dc_nc_f_dte,3,61,Electronic Credit Note,CREDIT NOTE,credit_note,N/C,base.cl,True,True
dc_nd_f_dte,4,56,Electronic Debit Note,DEBIT NOTE,debit_note,N/D,base.cl,True,True
dc_b_f_dte,5,39,Electronic Receipt,BEL,invoice,BEL,base.cl,True,True
dc_m_d_dtn,6,71,Electronic Fee Slips,BHE,invoice,BHE,base.cl,True,True
dc_b_e_dtn,7,38,Exempt Receipt,BEX,invoice,BEX,base.cl,False,False
dc_gd_dte,8,52,Electronic Dispatch Guide,GDE,stock_picking,GDE,base.cl,False,True
dc_I_f_dtn,10,29,Invoice of Initiation,FAI,invoice,FAI,base.cl,False,False
dc_a_f_dtn,10,30,Invoice,"INVOICE",invoice,FAC,base.cl,False,False
dc_y_f_dtn,10,32,Invoice of Sales and Services not Affected or Exempt from VAT,F-EXENTA,invoice,FNA,base.cl,False,False
dc_b_f_dtn,10,35,Bill of Sale,BOL,invoice,BOL,base.cl,False,False
dc_l_f_dtn,10,40,Invoice Settlement,L-FACTURAM,invoice,FAL,base.cl,False,False
dc_b_e_dte,10,41,Electronic Exempt Receipt,BXE,invoice,BXE,base.cl,True,True
dc_l_f_dte,10,43,Electronic Invoice Settlement,L-FACTURAE,invoice,FAL,base.cl,False,False
dc_fc_f_dtn,10,45,Purchase Invoice,"INVOICE",invoice_in,FAC,base.cl,False,False
dc_fc_f_dte,10,46,Electronic Purchase Invoice,"INVOICE",invoice_in,FAC,base.cl,False,True
dc_gd,10,50,Dispatch Guide,GD,stock_picking,GD,base.cl,False,False
dc_nd_f_dtn,10,55,Debit Note,DEBIT NOTE,debit_note,N/D,base.cl,False,False
dc_nc_f_dtn,10,60,Credit Note,CREDIT NOTE,credit_note,N/C,base.cl,False,False
dc_m_f_dtn,10,70,Fee Slips,BHO,invoice,BHO,base.cl,False,False
dc_li,10,103,Liquidation,LIQ,,LIQ,base.cl,False,True
dc_s_f_dtn,10,108,SRF Invoice Registration Request,SOL REGISTRO,,REG,base.cl,False,True
dc_fe_dte,10,110,Electronic Export Invoice,FCXE,invoice,FCXE,base.cl,False,True
dc_ndex_dte,10,111,Electronic Export Debit Note,NDXE,debit_note,NDXE,base.cl,False,True
dc_ncex_dte,10,112,Electronic Export Credit Note,NCXE,credit_note,NCXE,base.cl,False,True
dc_oc,10,300,Purchase Order,OC,,OC,base.cl,False,True
dc_tca_f_dtn,10,500,Exchange Rate Increase Adjustment (code 500),TC-A,,TCA,base.cl,False,True
dc_tcd_f_dtn,10,501,Exchange Rate Decrease Adjustment (code 501),TC-D,,TCD,base.cl,False,True
dc_odc,10,801,Purchase Order,OC,,OC,base.cl,False,True
dc_ndp,10,802,Order Note,NP,,NP,base.cl,False,True
dc_cont,10,803,Contract,CONT,,CONT,base.cl,False,True
dc_resol,10,804,Resolution,RES,,RES,base.cl,False,True
dc_prchc,10,805,Chile Purchase Process,PCHC,,PCHC,base.cl,False,True
dc_fichc,10,806,Chile Purchase Form,FCHC,,FCHC,base.cl,False,True
dc_dus,10,807,Single Exit Document (DUS),DUS,,DUS,base.cl,False,True
dc_bl_cemb,10,808,B/L (Bill of Lading),B/L,,B/L,base.cl,False,True
dc_awb,10,809,AWB Airway Bill,AWB,,AWB,base.cl,False,True
dc_mic_dta,10,810,MIC/DTA,MDT,,MDT,base.cl,False,True
dc_cpor,10,811,Bill of Lading,CPR,,CPR,base.cl,False,True
dc_res_sna,10,812,Resolution of the SNA where it qualifies Export Services,RSN,,RSN,base.cl,False,True
dc_pasap,10,813,Passport,PSP,,PSP,base.cl,False,True
dc_cd_bol,10,814,Certificate of Deposit Bolsa Prod. Chile.,CRTD,,CRTD,base.cl,False,True
dc_vp_pren,10,815,Pledge Voucher Bolsa Prod. Chile,VLPR,,VLPR,base.cl,False,True
dc_ftf_f_dtn,10,901,Invoice for sales to companies in the preferential territory ( Ex. Res. No. 1057,"INVOICE",invoice,FAC,base.cl,False,False
dc_cem_ma,10,902,Bill of Lading (Sea or Air),CEM,,CEM,base.cl,False,True
dc_ftt_f_dtn,10,904,Transfer Invoice,FACT TR,invoice,FTR,base.cl,False,False
dc_frr_f_dtn,10,905,Reissue Invoice,FACT RX,invoice,FRX,base.cl,False,False
dc_bzf_f_dtn,10,906,ZF Modules Sale Ballots (all),BOLETA ZF,invoice,BZF,base.cl,False,False
dc_fzf_f_dtn,10,907,Sales Invoices ZF Module (all),"INVOICE ZF",invoice,FZF,base.cl,False,False
dc_dizf_f_dtn,10,911,Declaration of Entry to Primary Free Trade Zone,DEC ING,invoice_in,DIN,base.cl,False,False
dc_din_f_dtn,10,914,Income Statement (DIN),DEC ING,invoice_in,DIN,base.cl,False,True
dc_res_vn_sf,10,919,Summary of Domestic Sales without Invoice,PASJ,,PASJ,base.cl,False,True
dc_chq,10,CHQ,Cheque,CHQ,,CHQ,base.cl,False,True
dc_hem,10,HEM,Material Entry Sheet (HEM),HEM,,HEM,base.cl,False,True
dc_hes,10,HES,Service Entry Sheet (HES),HES,,HES,base.cl,False,True
dc_migo,10,MIG,Movement of Goods (MIGO),MIGO,,MIGO,base.cl,False,True
dc_pag,10,PAG,Promissory note,PAG,,PAG,base.cl,False,True

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
            <field name='description'>Registration</field>
            <field name='sequence'>12</field>
            <field name='country_id' ref="base.cl"/>
        </record>
        <record model='l10n_latam.identification.type' id='it_DNI'>
            <field name='name'>ID CARD</field>
            <field name='description'>Foreign ID</field>
            <field name='sequence'>13</field>
            <field name='country_id' ref="base.cl"/>
        </record>
    </data>
</odoo>

```

## File: data\product_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

    <record id="product_product_ad_valorem" model="product.product">
        <field name="name">Ad-Valorem</field>
        <field name="type">service</field>
        <field name="default_code">AD_VALOREM</field>
        <field name="description">Charge for Ad-Valorem calculation in DIN</field>
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
            <field name="currency_unit_label">Development Unit</field>
            <field name="l10n_cl_short_name">Development Unit</field>
        </record>

        <record id="UTM" model="res.currency">
            <field name="name">UTM</field>
            <field name="symbol">UTM</field>
            <field name="rounding">0.01</field>
            <field name="position">after</field>
            <field name="currency_unit_label">Monthly Tax Unit</field>
            <field name="l10n_cl_short_name">Monthly Tax Unit</field>
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
            <field name='name'>Internal Revenue Service</field>
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
        <field name="name">Energy</field>
      </record>
      <record id="uom_categ_others" model="uom.category">
        <field name="name">Others</field>
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

## File: data\template\account.account-cl.csv

```csv
"id","code","name","account_type","reconcile","tag_ids","name@es"
"account_11700","117000","Transfer Account","asset_current","True","","Cuenta de Transferencia"
"account_110210","110210","Foreign Currency Deposits","asset_current","False","","Depósitos en Divisas"
"account_110220","110220","Shares","asset_current","False","","Acciones"
"account_110310","110310","Customers","asset_receivable","True","","Clientes"
"account_110315","110315","Factoring Receivable Account","asset_receivable","True","","Facturas cedidas por cobrar (Factoring)"
"account_11320","110320","Advances Suppliers","asset_current","True","","Anticipo Proveedores"
"account_11410","110410","Checks Receivable","asset_current","True","","Cheques a Fecha Por Cobrar"
"account_110420","110420","Sundry Accounts Receivable","asset_receivable","True","","Deudores Varios"
"account_110421","110421","Accounts receivable (Pos)","asset_receivable","True","","Deudores por Ventas (Pos)"
"account_110430","110430","Guaranty Ballots","asset_current","True","","Boletas en Garantía"
"account_110440","110440","Portfolio","asset_current","True","","Letras en Cartera"
"account_110450","110450","Protested Documents","asset_current","True","","Documentos Protestados"
"account_110510","110510","Salary Advance","asset_current","True","","Anticipo de Sueldo"
"account_110520","110520","Loans granted","asset_current","True","","Préstamos otorgados"
"account_110530","110530","Per Expense Advances","asset_prepayments","True","","Anticipos de Viáticos"
"account_110540","110540","Funds x Yield","asset_current","True","","Fondos x Rendir"
"account_110550","110550","Advance of Fees","asset_prepayments","True","","Anticipo de Honorarios"
"account_110560","110560","Advance of Christmas Bonus","asset_prepayments","True","","Anticipo de Aguinaldo"
"account_110570","110570","Family Allowance","asset_current","True","","Asignación Familiar"
"account_110580","110580","Advance Taxes","asset_current","False","","Anticipo de Impuestos"
"account_110585","110585","Prepaid Rents","asset_current","False","","Alquileres Pagados por Adelantado"
"account_110590","110590","Interest Paid in Advance","asset_current","False","","Intereses Pagados por Adelantado"
"account_110610","110610","Merchandise","asset_current","False","","Mercaderías"
"account_110612","110612","Raw Materials","asset_current","False","","Materia Prima"
"account_110615","110615","Materials","asset_current","False","","Materiales"
"account_110620","110620","Inputs","asset_current","False","","Insumos"
"account_110625","110625","Equipment","asset_current","False","","Equipos"
"account_110630","110630","Spare parts","asset_current","False","","Repuestos"
"account_110640","110640","Stocks in Transit","asset_current","True","","Existencias en Tránsito"
"account_110650","110650","Manufactured Products","asset_current","False","","Productos Fabricados"
"account_110660","110660","Products In Process","asset_current","False","","Productos En Proceso"
"account_110670","110670","Packaging","asset_current","False","","Envases"
"account_110710","110710","VAT Tax Credit","asset_current","False","","IVA Crédito Fiscal"
"account_110720","110720","Remaining Tax Credit","asset_current","False","","Remanente Crédito Fiscal"
"account_110730","110730","Property, plant and equipment","asset_current","False","","Crédito por Activo Fijo"
"account_110740","110740","P.P.M. / Art 33 BIS","asset_receivable","True","","P.P.M. / Art 33 BIS"
"account_110750","110750","Sence Credit","asset_current","False","","Crédito Sence"
"account_110760","110760","Specific Fuel Tax","asset_current","False","l10n_cl.tag_cl_specific_fuel_tax","Impuesto Específico a los Combustibles"
"account_110810","110810","Foreign Trade Advances","asset_current","False","","Anticipos Comercio Exterior"
"account_110820","110820","Prepaid Expenses","asset_current","False","","Gastos Anticipados"
"account_110830","110830","Organization and Start-up","asset_current","False","","Organización y Puesta en Marcha"
"account_110910","110910","Member Retreats","asset_current","False","","Retiros Socios"
"account_110920","110920","Withdrawals Reinvestment","asset_current","False","","Retiros Reinversión"
"account_111010","111010","Reorganization Expenses","asset_current","False","","Gastos Reorganización"
"account_111110","111110","Obligated Accounts Members","asset_current","False","","Cuentas Obligadas Socios"
"account_111210","111210","Tax Expenses","asset_current","False","","Gastos Tributarios"
"account_111310","111310","Imports Account","asset_current","False","","Cuenta Importaciones"
"account_111410","111410","Rejected Expenses","asset_current","False","","Gastos Rechazados"
"account_120110","120110","Delinquent Debtors","asset_receivable","True","","Deudores Morosos"
"account_120120","120120","Debtors in Judicial Management","asset_receivable","True","","Deudores en Gestión Judicial"
"account_121110","121110","Office Equipment and Furniture","asset_fixed","False","","Equipos y Mobiliario de Oficina"
"account_121120","121120","Computer Equipment","asset_fixed","False","","Equipos Computacionales"
"account_121130","121130","Vehicles","asset_fixed","False","","Vehículos"
"account_121140","121140","Machinery","asset_fixed","False","","Maquinaria"
"account_121210","121210","Real Estate","asset_fixed","False","","Bienes Raíces"
"account_121310","121310","Depreciation Accum Office Equipment and Furniture","asset_fixed","False","","Depreciación Acum Equipos y Mob de Oficina"
"account_121320","121320","Depreciation Acum Computer Equipment","asset_fixed","False","","Depreciación Acum Equipos Computacionales"
"account_121330","121330","Depreciation Accum Other Assets","asset_fixed","False","","Depreciación Acum Otros Activos"
"account_130110","130110","Intangible Assets - Right of Keys","asset_non_current","False","","Activo Intangible - Derecho de Llaves"
"account_130112","130112","Intangible Assets - Concessions and Franchises","asset_non_current","False","","Activo Intangible - Concesiones y Franquicias"
"account_130114","130114","Intangible Assets - Trademarks and Patents","asset_non_current","False","","Activo Intangible - Marcas y Patentes de Invención"
"account_130116","130116","Intangible Assets - (-) Accumulated Amortization","asset_non_current","False","","Activo Intangible - (-) Amortización Acumulada"
"account_130120","130120","Rights Other Companies","asset_non_current","False","","Derechos Otras Empresas"
"account_130130","130130","Accounts Receivable from Individuals and Related Companies Long term","asset_non_current","False","","Cuentas por Cobrar a Personas y Empresas Relacionadas Largo Plazo"
"account_130140","130140","Long-Term Notes and Accounts Receivable","asset_non_current","False","","Documentos y Cuentas por Cobrar a Largo Plazo"
"account_130150","130150","Guarantees of Long-Term Obligations and Third-Party Obligations","asset_non_current","False","","Garantías de Obligaciones a Largo Plazo y de Obligaciones de Terceros"
"account_130160","130160","Equity Securities Commodity Exchanges","asset_non_current","False","","Títulos Patrimoniales Bolsas de Productos"
"account_190110","190110","Guaranty Slips","asset_non_current","False","","Boletas de Garantía"
"account_190210","190210","Discounted Letters","asset_non_current","False","","Letras Descontadas"
"account_190310","190310","Collateral Documents","asset_non_current","False","","Documentos en Garantía"
"account_190410","190410","Subscribed Shares","asset_non_current","False","","Acciones Suscritas"
"account_210110","210110","Corporate Credit Card","liability_current","False","","Tarjeta de Crédito Corporativa"
"account_210120","210120","Bank Line of Credit","liability_current","False","","Linea de Crédito Bancaria"
"account_210140","210140","Commercial Credit with Term","liability_current","False","","Crédito Comercial C/Plazo"
"account_210150","210150","Short-Term Leasing Obligations","liability_current","False","","Obligaciones Leasing Corto Plazo"
"account_210160","210160","Mortgage loan with term","liability_current","False","","Crédito Hipotecario C/Plazo"
"account_210170","210170","Commercial Credit LP Venc.  with Term","liability_current","False","","Crédito Comercial LP Venc. C/Plazo"
"account_210180","210180","Dividends Payable","liability_current","False","","Dividendos por Pagar"
"account_210190","210190","Interest Accrued on Credit Purchases","liability_payable","True","","Intereses a Devengar por Compras al Crédito"
"account_210195","210195","Interest payable","liability_current","False","","Intereses a pagar"
"account_210210","210210","Suppliers","liability_payable","True","","Proveedores"
"account_210220","210220","Customer Advances","liability_payable","True","","Anticipo de Clientes"
"account_210230","210230","Invoices Receivable","liability_current","True","","Facturas por Recibir"
"account_210310","210310","Checks Drawn and Uncollected","liability_current","False","","Cheques Girados y No Cobrados"
"account_210320","210320","Collateral Documents","liability_current","False","","Documentos en Garantía"
"account_210330","210330","Warranty Bond","liability_current","False","","Boleta en Garantia"
"account_210410","210410","AFP x Pay","liability_current","False","","AFP x Pagar"
"account_210420","210420","C.C.A.F. x Payable","liability_current","False","","C.C.A.F. x Pagar"
"account_210430","210430","INP x Pagar","liability_current","False","","INP x Pagar"
"account_210440","210440","ISAPRES x Pagar","liability_current","False","","ISAPRES x Pagar"
"account_210450","210450","Mutual Insurance Paid","liability_current","False","","Mutual Seg. x Pagar"
"account_210510","210510","Salaries Payable","liability_current","False","","Sueldos por Pagar"
"account_210520","210520","Fees Payable","liability_current","False","","Honorarios por Pagar"
"account_210550","210550","Renditions Payable","liability_current","False","","Rendiciones por Pagar"
"account_210560","210560","Vacation Provision","liability_current","False","","Provisión de Vacaciones"
"account_210565","210565","Provision for severance payments","liability_current","False","","Provisión por Finiquitos"
"account_210610","210610","PPM Tax Provision","liability_non_current","False","","Provisión de Impuesto PPM"
"account_210620","210620","Other Provisions","liability_current","False","","Otras Provisiones"
"account_210710","210710","VAT Tax Debit","liability_current","False","","IVA Débito Fiscal"
"account_210720","210720","PPM Payable","liability_current","False","","PPM por Pagar"
"account_210730","210730","Single Tax Workers","liability_current","False","","Impuesto Único Trabajadores"
"account_210740","210740","Second Category Withholding Tax","liability_current","False","","Impuesto Retención Segunda Categoría"
"account_210715","210715","VAT withheld from third parties","liability_current","False","","IVA Retenido a terceros"
"account_210750","210750","1st Category Income Tax Payable","liability_current","False","","Impuesto Renta 1a Categoría por Pagar"
"account_210760","210760","Monthly Taxes Payable","liability_current","True","","Impuestos Mensuales por Pagar"
"account_220120","220120","Leasing Obligations Term","liability_current","False","","Obligaciones Leasing L/Plazo"
"account_220130","220130","Commercial Credit without Term","liability_current","False","","Crédito Comercial L/Plazo"
"account_220210","220210","Provision for Years of Service Indemnity","liability_current","False","","Provisión Indemnización por Años de Servicio"
"account_220310","220310","Current Account Related Company","liability_non_current","False","","Cta Corriente Empresa Relacionada"
"account_220320","220320","Long-term deferred taxes","liability_non_current","False","","Impuesto Diferido Largo Plazo"
"account_220330","220330","Other Liabilities","liability_current","False","","Otros Pasivos"
"account_230110","230110","Capital Stock","equity","False","","Capital Social"
"account_230120","230120","Shares in Circulation","equity","False","","Acciones en Circulación"
"account_230130","230130","Dividends to be Distributed in Shares","equity","False","","Dividendos a Distribuir en Acciones"
"account_230140","230140","Stock Issuance Discount","equity","False","","Descuento de Emisión de Acciones"
"account_230210","230210","Revaluation of shareholders' equity","equity","False","","Revalorización Capital Propio"
"account_230220","230220","Other Revaluations","equity","False","","Otras Revalorizaciones"
"account_230230","230230","Revaluation of fixed assets","equity","False","","Revalorización Activo Fijo"
"account_230240","230240","Value Fluctuation","equity","False","","Fluctuación de Valores"
"account_230250","230250","Legal Reserve","equity","False","","Reserva Legal"
"account_230260","230260","Statutory Reserve","equity","False","","Reserva Estatutaria"
"account_230270","230270","Optional Reserve","equity","False","","Reserva Facultativa"
"account_230280","230280","Fixed Asset Renewal Reserve","equity","False","","Reserva para Renovación de Activo Fijo"
"account_230290","230290","Accumulated Results","equity","False","","Resultados Acumulados"
"account_230300","230300","Profit and Loss for the Year","equity","False","","Utilidades y Pérdidas del Ejercicio"
"account_230310","230310","Prior Year Retained Earnings","equity","False","","Resultados Acumulados del Ejercicio Anterior"
"account_230510","230510","Income for the year","equity","False","","Resultado del Ejercicio"
"account_290110","290110","Bridge Account","liability_non_current","True","","Cuenta Puente"
"account_290210","290210","Responsible for Guarantee Vouchers","liability_non_current","False","","Responsable Boletas Garantía"
"account_290220","290220","Responsible Documents Warranty","liability_current","False","","Responsable Documentos Garantía"
"account_290230","290230","Responsible for Discounted Letters","liability_non_current","False","","Responsable Letras Descontadas"
"account_310110","310110","Consulting revenues","income","False","","Ingresos por Consultoría"
"account_310115","310115","Product Sales","income","False","","Ventas de Productos"
"account_310120","310120","Sales of Services","income","False","","Ventas de Servicios"
"account_310122","310122","National/International Ticket Sales and/or Unaffected Commissions","income","False","","Ventas de Pasajes Nacionales/Internacionales y/o Comisiones No Afectas"
"account_310125","310125","Export Sales","income","False","","Ventas de Exportación"
"account_310130","310130","Commissions earned on sales","income","False","","Comisiones Percibidas por Ventas"
"account_320210","320210","Profit from sale of fixed assets","income","False","","Utilidad Venta Activo Fijo"
"account_320220","320220","Interest Received on Loans Granted","income","False","","Intereses Percibidos Sobre Préstamos Otorgados"
"account_320225","320225","Leases earned, obtained, received","income","False","","Arriendos ganados, obtenidos, percibidos"
"account_320230","320230","Discounts earned, obtained, received","income","False","","Descuentos ganados, obtenidos, percibidos"
"account_320235","320235","Interest on investments","income","False","","Intereses sobre Inversiones"
"account_320240","320240","Gain on sale of shares","income","False","","Ganancia Venta de Acciones"
"account_320245","320245","Recovery of Arrears","income","False","","Recupero de Rezagos"
"account_320250","320250","Recovery of uncollectible accounts receivable","income","False","","Recupero de Deudores Incobrables"
"account_320255","320255","Donations obtained, earned, received","income","False","","Donaciones obtenidas, ganandas, percibidas"
"account_320260","320260","Gain on sale of permanent investments","income","False","","Ganancia Venta Inversiones Permanentes"
"account_320265","320265","Exchange rate difference Gain","income","False","","Diferencia tipo de cambio Ganancia"
"account_320270","320270","Long-term loans","income_other","False","","Cŕeditos a Largo Plazo"
"account_320275","320275","Other Income","income_other","False","","Otros Ingresos"
"account_320280","320280","Price-level restatement Assets","income_other","False","","Corrección Monetaria Activos"
"account_320285","320285","Price-level restatement Leasing assets","income_other","False","","Corrección Monetaria Bienes Leasing"
"account_410110","410110","Remunerations Operation","expense","False","","Remuneraciones Operación"
"account_410115","410115","Employer's contribution Operation","expense","False","","Aporte Patronal Operación"
"account_410120","410120","Per diem","expense","False","","Viáticos"
"account_410125","410125","Training","expense","False","","Capacitación"
"account_410130","410130","Fees Paid","expense","False","","Honorarios Pagados"
"account_410135","410135","Consultancies","expense","False","","Asesorías"
"account_410140","410140","Office Leases","expense","False","","Arriendos Oficina"
"account_410145","410145","Warehouse Leases","expense","False","","Arriendos Bodega"
"account_410150","410150","Communications","expense","False","","Comunicaciones"
"account_410155","410155","Legal Services","expense","False","","Servicios Legales"
"account_410160","410160","Asset Maintenance and Repair","expense","False","","Manutención y Reparación de Activos"
"account_410165","410165","Stationery-Sanitary-Miscellaneous Expenses","expense","False","","Papelería-Aseo-Gastos Diversos"
"account_410170","410170","Remunerations Administration","expense","False","","Remuneraciones Administración"
"account_410175","410175","Employer's contribution Administration","expense","False","","Aporte Patronal Administración"
"account_410180","410180","Electricity","expense","False","","Electricidad"
"account_410185","410185","Common Expenses","expense","False","","Gastos Comunes"
"account_410190","410190","Bank charges","off_balance","False","","Gastos Bancarios"
"account_410195","410195","Exchange Rate Difference Loss","expense","False","","Diferencia Tipo de Cambio Perdida"
"account_410200","410200","Monetary Correction","expense","False","","Corrección Monetaria"
"account_410205","410205","Tax Fines","expense","False","","Multas Fiscales"
"account_410210","410210","Income Tax 1st Category","expense","False","","Impuesto a la Renta 1ra Categoría"
"account_410215","410215","Loss on sale of assets","expense","False","","Pérdida por Venta Activo"
"account_410220","410220","Prior Year Adjustment","expense","False","","Ajuste Ejercicio Anterior"
"account_410225","410225","Donation","expense","False","","Donación"
"account_410230","410230","Purchases Products 1st Category","expense","False","","Compras Productos 1ra Categoría"
"account_410231","410231","Net Purchases (VAT Common Use)","expense","False","","Compras Netas (IVA Uso Común)"
"account_410232","410232","Net Purchases (Non-recoverable VAT)","expense","False","","Compras Netas (IVA No Recuperable)"
"account_410233","410233","Supermarket Shopping","expense","False","","Compras de Supermercado"
"account_410235","410235","Cost of Goods Sold - 1st Category Prod","expense_direct_cost","False","","Costo de Mercaderías Vendidas - Prod. 1ra Categoría"
"account_410240","410240","Imports","expense_direct_cost","False","","Importaciones"
"account_410245","410245","CIF","expense_direct_cost","False","","CIF"
"account_410250","410250","Purchases of raw materials","expense_direct_cost","False","","Compras de Materia Prima"
"account_410255","410255","Direct Labor","expense","False","","Mano de Obra Directa"
"account_420110","420110","Merchandise Received on Consignment","expense","False","","Mercaderias Recibidas en Consignación"
"account_440210","440210","Customer for Goods Received on Consignment","expense","False","","Comitente por Mercaderias Recibidas en Consignación"
"account_420120","420120","Fixed asset depreciation expense","expense","False","","Gastos en Depreciación de Activo Fijo"
"account_420140","420140","Amortization Expenses","expense","False","","Gastos en Amortización"
"account_420150","420150","Claims Expenses","expense","False","","Gastos en Siniestros"
"account_420170","420170","Warranties Granted","off_balance","False","","Garantias Otorgadas"
"account_420220","420220","Expenses Other Taxes","expense","False","","Gastos Otros Impuestos"

```

## File: data\template\account.fiscal.position-cl.csv

```csv
"id","name","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id","name@es"
"afpt_non_recoverable_vat_1","Purchases - intended to generate non-taxable or exempt transactions","OTAX_19","iva_compra_no_recup","","","Compras - destinadas a generar operaciones no gravadas o exentas"
"afpt_non_recoverable_vat_2","Purchasing - Invoices from suppliers registered after the due date","OTAX_19","iva_compra_no_recup","","","Compras - Facturas de proveedores registrados fuera de plazo"
"afpt_non_recoverable_vat_3","Purchases - Rejected expenses","OTAX_19","iva_compra_no_recup","","","Compras - Gastos rechazados"
"afpt_non_recoverable_vat_4","Purchases - Free deliveries (prizes, bonuses, etc.) received","OTAX_19","iva_compra_no_recup","","","Compras - Entregas gratuitas (premios, bonificaciones, etc.) recibidos"
"","","","","account_410235","account_410165",""
"","","","","account_410230","account_410165",""
"afpt_non_recoverable_vat_9","Purchases - Others","OTAX_19","iva_compra_no_recup","","","Compras - Otros"
"afpt_fixed_asset","Purchases - Fixed Assets","OTAX_19","iva_activo_fijo_uso_comun","","","Compras - Activo Fijo"
"","","","","account_410230","account_121140",""
"","","","","account_410235","account_121140",""
"afpt_purchase_exempt","Purchases - Exempt","OTAX_19","","","","Compras - Exentas"
"","","","","account_410230","account_410130",""
"afpt_purchase_supermarket","Shopping - Supermarket","OTAX_19","iva_supermercado_recup","","","Compras - Supermercado"
"","","","","account_410230","account_410233",""
"","","","","account_410235","account_410233",""
"afpt_sale_exempt","Sales - Exempt","ITAX_19","","","","Ventas - Exentas"
"","","","","account_310115","account_310120",""

```

## File: data\template\account.tax-cl.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","l10n_cl_sii_code","tax_group_id","active","sequence","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@es","name@es","invoice_label@es"
"ITAX_19","19% VAT","VAT 19% Sale","VAT 19% Sale","19.0","percent","sale","14","tax_group_iva_19","","","base","invoice","+Ventas Netas Gravadas con IVA","","","IVA 19% Venta","19% IVA","IVA 19% Vta"
"","","","","","","","","","","","tax","invoice","+IVA Debito Fiscal","account_210710","","","",""
"","","","","","","","","","","","base","refund","-Ventas Netas Gravadas con IVA","","","","",""
"","","","","","","","","","","","tax","refund","-IVA Debito Fiscal","account_210710","","","",""
"OTAX_19","19% VAT","VAT 19% Purchase","VAT 19% Purchase","19.0","percent","purchase","14","tax_group_iva_19","","","base","invoice","+Compras Netas Gravadas Con IVA (recuperable)","","","IVA 19% Compra","19% IVA","IVA 19% Comp"
"","","","","","","","","","","","tax","invoice","+IVA Pagado Compras Recuperables","account_110710","","","",""
"","","","","","","","","","","","base","refund","-Compras Netas Gravadas Con IVA (recuperable)","","","","",""
"","","","","","","","","","","","tax","refund","-IVA Pagado Compras Recuperables","account_110710","","","",""
"I_IU2C","10.75% WH","Withholding 2nd Category 2020","Ret. 2da 2020","-10.75","percent","purchase","15","tax_group_2da_categ","False","2","base","invoice","+Base Retención Segunda Categoría","","","Ret. 2da Categoría 2020","10.75% Ret. 2da 2020","Ret. 2da 2020"
"","","","","","","","","","","","tax","invoice","+Retención Segunda Categoría","account_210740","","","",""
"","","","","","","","","","","","base","refund","-Base Retención Segunda Categoría","","","","",""
"","","","","","","","","","","","tax","refund","-Retención Segunda Categoría","account_210740","","","",""
"I_IR2C_2021","11.5% WH","Withholding 2nd Category 2021","Ret. 2da 2021","-11.5","percent","purchase","15","tax_group_2da_categ","False","2","base","invoice","+Base Retención Segunda Categoría","","100","Ret. 2da Categoría 2021","11.5% Ret. 2da 2021","Ret. 2da 2021"
"","","","","","","","","","","","tax","invoice","+Retención Segunda Categoría","account_210740","100","","",""
"","","","","","","","","","","","base","refund","-Base Retención Segunda Categoría","","100","","",""
"","","","","","","","","","","","tax","refund","-Retención Segunda Categoría","account_210740","100","","",""
"I_IR2C_2022","12.25% WH","Withholding 2nd Category 2022","Ret. 2da 2022","-12.25","percent","purchase","15","tax_group_2da_categ","","2","base","invoice","+Base Retención Segunda Categoría","","100","Ret. 2da Categoría 2022","12.25% Ret. 2da 2022","Ret. 2da 2022"
"","","","","","","","","","","","tax","invoice","+Retención Segunda Categoría","account_210740","100","","",""
"","","","","","","","","","","","base","refund","-Base Retención Segunda Categoría","","100","","",""
"","","","","","","","","","","","tax","refund","-Retención Segunda Categoría","account_210740","100","","",""
"I_IR2C_2023","13% WH","Withholding 2nd Category 2023","Ret. 2da 2023","-13.0","percent","purchase","15","tax_group_2da_categ","","2","base","invoice","+Base Retención Segunda Categoría","","100","Ret. 2da Categoría 2023","13% Ret. 2da 2023","Ret. 2da 2023"
"","","","","","","","","","","","tax","invoice","+Retención Segunda Categoría","account_210740","100","","",""
"","","","","","","","","","","","base","refund","-Base Retención Segunda Categoría","","100","","",""
"","","","","","","","","","","","tax","refund","-Retención Segunda Categoría","account_210740","100","","",""
"I_IR2C_2024","13.75% WH","Withholding 2nd Category 2024","Ret. 2da 2024","-13.75","percent","purchase","15","tax_group_2da_categ","","2","base","invoice","+Base Retención Segunda Categoría","","100","Ret. 2da Categoría 2024","13.75% Ret. 2da 2024","Ret. 2da 2024"
"","","","","","","","","","","","tax","invoice","+Retención Segunda Categoría","account_210740","100","","",""
"","","","","","","","","","","","base","refund","-Base Retención Segunda Categoría","","100","","",""
"","","","","","","","","","","","tax","refund","-Retención Segunda Categoría","account_210740","100","","",""
"I_IR2C_2025","14.5% WH","Withholding 2nd Category 2025","Ret. 2da 2025","-14.5","percent","purchase","15","tax_group_2da_categ","","2","base","invoice","+Base Retención Segunda Categoría","","100","Ret. 2da Categoría 2025","14.5% Ret. 2da 2025","Ret. 2da 2025"
"","","","","","","","","","","","tax","invoice","+Retención Segunda Categoría","account_210740","100","","",""
"","","","","","","","","","","","base","refund","-Base Retención Segunda Categoría","","100","","",""
"","","","","","","","","","","","tax","refund","-Retención Segunda Categoría","account_210740","100","","",""
"I_IR2C_2026","15.25% WH","Withholding 2nd Category 2026","Ret. 2da 2026","-15.25","percent","purchase","15","tax_group_2da_categ","","2","base","invoice","+Base Retención Segunda Categoría","","100","Ret. 2da Categoría 2026","15.25% Ret. 2da 2026","Ret. 2da 2026"
"","","","","","","","","","","","tax","invoice","+Retención Segunda Categoría","account_210740","100","","",""
"","","","","","","","","","","","base","refund","-Base Retención Segunda Categoría","","100","","",""
"","","","","","","","","","","","tax","refund","-Retención Segunda Categoría","account_210740","100","","",""
"I_RTI","19% WH","Total VAT withholding","Total VAT withholding","-19.0","percent","purchase","15","tax_group_retenciones","","2","base","invoice","+Retención Total (compras)","","","Retención Total IVA","19% Ret. Tot. IVA","Retención total IVA"
"","","","","","","","","","","","tax","invoice","+Retención Total (compras)","account_210715","","","",""
"","","","","","","","","","","","base","refund","-Retención Total (compras)","","","","",""
"","","","","","","","","","","","tax","refund","-Retención Total (compras)","account_210715","","","",""
"especifico_compra","63% Spec","Specific Purchase","Specific purchase","63.0","percent","purchase","29","tax_group_impuestos_especificos","","5","base","invoice","","","","Específico Compra","63% Espec. Comp.","Espec. Comp"
"","","","","","","","","","","","tax","invoice","","account_420220","","","",""
"","","","","","","","","","","","base","refund","","","","","",""
"","","","","","","","","","","","tax","refund","","account_420220","","","",""
"iva_activo_fijo","19% F A","VAT Purchase 19% Fixed Assets","VAT 19% fixed assets","19.0","percent","purchase","14","tax_group_iva_19","","6","base","invoice","+Compras de Activo Fijo","","","IVA Compra 19% Activo Fijo","19% IVA Act. F","IVA 19% ActF"
"","","","","","","","","","","","tax","invoice","+Compras Activo Fijo","account_110730","","","",""
"","","","","","","","","","","","base","refund","-Compras de Activo Fijo","","","","",""
"","","","","","","","","","","","tax","refund","-Compras Activo Fijo","account_110730","","","",""
"iva_activo_fijo_uso_comun","19% F A C","VAT Purchase 19% Fixed Assets Common Use","VAT 19% fixed assets common use","19.0","percent","purchase","14","tax_group_iva_19","","6","base","invoice","+Compras de Activo Fijo Uso Común","","","IVA Compra 19% Act. Fijo Uso Común","19% IVA Act. FUC","IVA 19% ActFUC"
"","","","","","","","","","","","tax","invoice","+Compras Activo Fijo Uso Común","account_110730","","","",""
"","","","","","","","","","","","base","refund","-Compras de Activo Fijo Uso Común","","","","",""
"","","","","","","","","","","","tax","refund","-Compras Activo Fijo Uso Común","account_110730","","","",""
"iva_activo_fijo_uso_no_recup","19% F A NR","VAT Purchase 19% Fixed Assets Not Recoverable","VAT 19% fixed assets non-recoverable use","19.0","percent","purchase","14","tax_group_iva_19","","6","base","invoice","+Compras de Activo Fijo No Recuperable","","","IVA Compra 19% Activo Fijo No Recup","19% IVA Act. FNR","IVA 19% ActFNR"
"","","","","","","","","","","","tax","invoice","+Compras Activo Fijo No Recuperables","account_420220","","","",""
"","","","","","","","","","","","base","refund","-Compras de Activo Fijo No Recuperable","","","","",""
"","","","","","","","","","","","tax","refund","-Compras Activo Fijo No Recuperables","account_420220","","","",""
"ila_a_100_p","10% ILA","Beb. Analc. 10% (Purchases)","ILA P 10%","10.0","percent","purchase","27","tax_group_ila","","7","base","invoice","+Base Retenciones ILA (compras)","","","Beb. Analc. 10% (Compras)","10% ILA","ILA C 10%"
"","","","","","","","","","","","tax","invoice","+Retenciones ILA (compras)","account_420220","","","",""
"","","","","","","","","","","","base","refund","-Base Retenciones ILA (compras)","","","","",""
"","","","","","","","","","","","tax","refund","-Retenciones ILA (compras)","account_420220","","","",""
"ila_a_180_p","18% ILA","Beb. Analc 18% (Purchases)","ILA P 18%","18.0","percent","purchase","26","tax_group_ila","","7","base","invoice","+Base Retenciones ILA (compras)","","","Beb. Analc 18% (Compras)","18% ILA","ILA C 18%"
"","","","","","","","","","","","tax","invoice","+Retenciones ILA (compras)","account_420220","","","",""
"","","","","","","","","","","","base","refund","-Base Retenciones ILA (compras)","","","","",""
"","","","","","","","","","","","tax","refund","-Retenciones ILA (compras)","account_420220","","","",""
"ila_v_205_p","20.5% ILA","Wines (Purchases)","ILA P 20.5%","20.5","percent","purchase","25","tax_group_ila","","7","base","invoice","+Base Retenciones ILA (compras)","","","Vinos (Compras)","20.5% ILA","ILA C 20.5%"
"","","","","","","","","","","","tax","invoice","+Retenciones ILA (compras)","account_420220","","","",""
"","","","","","","","","","","","base","refund","-Base Retenciones ILA (compras)","","","","",""
"","","","","","","","","","","","tax","refund","-Retenciones ILA (compras)","account_420220","","","",""
"ila_l_315_p","31.5% ILA","Liquors 31.5% (Purchases)","ILA P 31.5%","31.5","percent","purchase","24","tax_group_ila","","7","base","invoice","+Base Retenciones ILA (compras)","","","Licores 31.5% (Compras)","31.5% ILA","ILA C 31.5%"
"","","","","","","","","","","","tax","invoice","+Retenciones ILA (compras)","account_420220","","","",""
"","","","","","","","","","","","base","refund","-Base Retenciones ILA (compras)","","","","",""
"","","","","","","","","","","","tax","refund","-Retenciones ILA (compras)","account_420220","","","",""
"ila_a_100_s","10% ILA","Beb. Analc. 10% (Sales)","ILA S 10%","10.0","percent","sale","27","tax_group_ila","","7","base","invoice","+Base Retenciones ILA (ventas)","","","Beb. Analc. 10% (Ventas)","10% ILA","ILA V 10%"
"","","","","","","","","","","","tax","invoice","+Retenciones ILA (ventas)","account_210760","","","",""
"","","","","","","","","","","","base","refund","-Base Retenciones ILA (ventas)","","","","",""
"","","","","","","","","","","","tax","refund","-Retenciones ILA (ventas)","account_210760","","","",""
"ila_a_180_s","18% ILA","Beb. Analc 18% (Sales)","ILA S 18%","18.0","percent","sale","26","tax_group_ila","","7","base","invoice","+Base Retenciones ILA (ventas)","","","Beb. Analc 18% (Ventas)","18% ILA","ILA V 18%"
"","","","","","","","","","","","tax","invoice","+Retenciones ILA (ventas)","account_210760","","","",""
"","","","","","","","","","","","base","refund","-Base Retenciones ILA (ventas)","","","","",""
"","","","","","","","","","","","tax","refund","-Retenciones ILA (ventas)","account_210760","","","",""
"ila_v_205_s","20.5% ILA","Wines (Sales)","ILA S 20.5%","20.5","percent","sale","25","tax_group_ila","","7","base","invoice","+Base Retenciones ILA (ventas)","","","Vinos (Ventas)","20.5% ILA","ILA V 20.5%"
"","","","","","","","","","","","tax","invoice","+Retenciones ILA (ventas)","account_210760","","","",""
"","","","","","","","","","","","base","refund","-Base Retenciones ILA (ventas)","","","","",""
"","","","","","","","","","","","tax","refund","-Retenciones ILA (ventas)","account_210760","","","",""
"ila_l_315_s","31.5% ILA","Liquors 31.5% (Sales)","ILA S 31.5%","31.5","percent","sale","24","tax_group_ila","","7","base","invoice","+Base Retenciones ILA (ventas)","","","Licores 31.5% (Ventas)","31.5% ILA","ILA V 31.5%"
"","","","","","","","","","","","tax","invoice","+Retenciones ILA (ventas)","account_210760","","","",""
"","","","","","","","","","","","base","refund","-Base Retenciones ILA (ventas)","","","","",""
"","","","","","","","","","","","tax","refund","-Retenciones ILA (ventas)","account_210760","","","",""
"iva_compra_no_recup","19% NR","VAT Purchase 19% Non-recoverable","VAT 19% non-recoverable purchase","19.0","percent","purchase","14","tax_group_iva_19","","6","base","invoice","+Compras IVA No Recuperable","","","IVA Compra 19% No Recup.","19% IVA NR","IVA 19% NoR"
"","","","","","","","","","","","tax","invoice","+IVA Pagado No Recuperable","account_420220","","","",""
"","","","","","","","","","","","base","refund","-Compras IVA No Recuperable","","","","",""
"","","","","","","","","","","","tax","refund","-IVA Pagado No Recuperable","account_420220","","","",""
"iva_compra_uso_comun","19% C","VAT Purchase 19% Common Use","VAT 19% purchase for public use","19.0","percent","purchase","14","tax_group_iva_19","","6","base","invoice","+Compra Netas Gravadas Con IVA Uso Comun","","","IVA Compra 19% Uso Común","19% IVA UC","IVA 19% CUC"
"","","","","","","","","","","","tax","invoice","+IVA Pagado Compras Uso Común","account_110730","","","",""
"","","","","","","","","","","","base","refund","-Compra Netas Gravadas Con IVA Uso Comun","","","","",""
"","","","","","","","","","","","tax","refund","-IVA Pagado Compras Uso Común","account_110730","","","",""
"iva_supermercado_recup","19% SM","VAT Purchase 19% Supermarket","VAT 19% supermarket recup","19.0","percent","purchase","14","tax_group_iva_19","","6","base","invoice","+Compras De Supermercado","","","IVA Compra 19% Supermercado Recup","19% IVA SupMRec","IVA 19% SupMRec"
"","","","","","","","","","","","tax","invoice","+IVA Pagado Compras Supermercado","account_110710","","","",""
"","","","","","","","","","","","base","refund","-Compras De Supermercado","","","","",""
"","","","","","","","","","","","tax","refund","-IVA Pagado Compras Supermercado","account_110710","","","",""
"iec_gasoline","IEC Gas","Specific Tax Gasoline","IEC Gasolina","1.0","fixed","purchase","35","tax_group_impuestos_especificos","","5","base","invoice","+Compras Des Combustibles","","","Impuesto Específico Gasolina","IEC Gasolina","IEC Gas"
"","","","","","","","","","","","tax","invoice","+IEC Compras Des Combustibles","account_110760","","","",""
"","","","","","","","","","","","base","refund","-Compras Des Combustibles","","","","",""
"","","","","","","","","","","","tax","refund","-IEC Compras Des Combustibles","account_110760","","","",""
"iec_diesel","IEC Die","Specific Tax Diesel","IEC Diesel","1.0","fixed","purchase","28","tax_group_impuestos_especificos","","5","base","invoice","+Compras Des Combustibles","","","Impuesto Específico Diesel","IEC Diesel","IEC Die"
"","","","","","","","","","","","tax","invoice","+IEC Compras Des Combustibles","account_110760","","","",""
"","","","","","","","","","","","base","refund","-Compras Des Combustibles","","","","",""
"","","","","","","","","","","","tax","refund","-IEC Compras Des Combustibles","account_110760","","","",""

```

## File: data\template\account.tax.group-cl.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@es"
"tax_group_iva_19","VAT 19%","base.cl","account_210760","account_210760","IVA 19%"
"tax_group_impuestos_especificos","Specific Taxes","base.cl","account_210760","account_210760","Impuestos Específicos"
"tax_group_ila","ILA","base.cl","account_210760","account_210760","ILA"
"tax_group_2da_categ","2nd Category Withholding Tax","base.cl","account_210760","account_210760","Retención de 2da Categoría"
"tax_group_retenciones","Withholdings","base.cl","account_210760","account_210760","Retenciones"

```

## File: migrations\3.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'cl')], order="parent_path"):
        env['account.chart.template'].try_loading('cl', company)

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.exceptions import ValidationError
from odoo import models, fields, api, _
from odoo.tools.misc import formatLang
from odoo.tools.float_utils import float_repr, float_round

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
            domain += [('code', '=', '71')]
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
                            'Document types for foreign customers must be export type (codes 110, 111 or 112) or you should define the customer as an end consumer and use receipts (codes 39 or 41)'))
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

    def _format_lang_totals(self, value, currency):
        return formatLang(self.env, value, currency_obj=currency)

    def _l10n_cl_get_invoice_totals_for_report(self):
        self.ensure_one()
        include_sii = self._l10n_cl_include_sii()
        tax_totals = self.tax_totals
        if not include_sii:
            return tax_totals

        tax_group_ids = {
            tax_group['id']
            for subtotal in tax_totals['subtotals']
            for tax_group in subtotal['tax_groups']
        }
        tax_group_ids_to_exclude = self.env['account.tax.group'].browse(tax_group_ids).filtered(lambda x: x.l10n_cl_sii_code == 14).ids
        if tax_group_ids_to_exclude:
            return self.env['account.tax']._exclude_tax_groups_from_tax_totals_summary(tax_totals, tax_group_ids_to_exclude)
        return tax_totals

    def _l10n_cl_include_sii(self):
        self.ensure_one()
        return self.l10n_latam_document_type_id.code in ['39', '41', '110', '111', '112', '34']

    def _is_manual_document_number(self):
        if self.journal_id.company_id.country_id.code == 'CL':
            return self.journal_id.type == 'purchase' and not self.l10n_latam_document_type_id._is_doc_type_vendor()
        return super()._is_manual_document_number()

    def _l10n_cl_get_amounts(self):
        """
        This method is used to calculate the amount and taxes required in the Chilean localization electronic documents.
        """
        self.ensure_one()
        global_discounts = self.invoice_line_ids.filtered(lambda x: x.price_subtotal < 0)
        export = self.l10n_latam_document_type_id._is_doc_type_export()
        main_currency = self.company_id.currency_id if not export else self.currency_id
        key_main_currency = 'amount_currency' if export else 'balance'
        sign_main_currency = -1 if self.move_type == 'out_invoice' else 1
        currency_round_main_currency = self.currency_id if export else self.company_id.currency_id
        currency_round_other_currency = self.company_id.currency_id if export else self.currency_id
        total_amount_main_currency = currency_round_main_currency.round(self.amount_total) if export \
            else (currency_round_main_currency.round(abs(self.amount_total_signed)))
        other_currency = self.currency_id != self.company_id.currency_id
        values = {
            'main_currency': main_currency,
            'vat_amount': 0,
            'subtotal_amount_taxable': 0,
            'subtotal_amount_exempt': 0, 'total_amount': total_amount_main_currency,
            'main_currency_round': currency_round_main_currency.decimal_places,
            'main_currency_name': self._l10n_cl_normalize_currency_name(
                currency_round_main_currency.name) if export else False
        }
        vat_percent = 0

        if other_currency:
            key_other_currency = 'balance' if export else 'amount_currency'
            values['second_currency'] = {
                'subtotal_amount_taxable': 0,
                'subtotal_amount_exempt': 0,
                'vat_amount': 0,
                'total_amount': currency_round_other_currency.round(abs(self.amount_total_signed))
                    if export else currency_round_other_currency.round(self.amount_total),
                'round_currency': currency_round_other_currency.decimal_places,
                'name': self._l10n_cl_normalize_currency_name(currency_round_other_currency.name),
                'rate': round(abs(self.amount_total_signed) / self.amount_total, 4),
            }
        for line in self.line_ids:
            if line.tax_line_id and line.tax_line_id.l10n_cl_sii_code == 14:
                values['vat_amount'] += line[key_main_currency] * sign_main_currency
                if other_currency:
                    values['second_currency']['vat_amount'] += line[key_other_currency] * sign_main_currency
                vat_percent = max(vat_percent, line.tax_line_id.amount)
            if line.display_type == 'product':
                if line.tax_ids.filtered(lambda x: x.l10n_cl_sii_code == 14):
                    values['subtotal_amount_taxable'] += line[key_main_currency] * sign_main_currency
                    if other_currency:
                        values['second_currency']['subtotal_amount_taxable'] += line[key_other_currency] * sign_main_currency
                elif not line.tax_ids:
                    values['subtotal_amount_exempt'] += line[key_main_currency] * sign_main_currency
                    if other_currency:
                        values['second_currency']['subtotal_amount_exempt'] += line[key_other_currency] * sign_main_currency
        values['global_discounts'] = []
        for gd in global_discounts:
            main_value = currency_round_main_currency.round(abs(gd.price_subtotal)) if \
                (not other_currency and not export) or (other_currency and export) else \
                currency_round_main_currency.round(abs(gd.balance))
            second_value = currency_round_other_currency.round(abs(gd.balance)) if other_currency and export else \
                currency_round_other_currency.round(abs(gd.price_subtotal))
            values['global_discounts'].append(
                {
                    'name': gd.name,
                    'global_discount_main_value': main_value,
                    'global_discount_second_value': second_value if second_value != main_value else False,
                    'tax_ids': gd.tax_ids,
                }
            )
        values['vat_percent'] = '%.2f' % vat_percent if vat_percent > 0 else False
        return values

    def _l10n_cl_get_withholdings(self):
        """
        This method calculates the section of withholding taxes, or 'other' taxes for the Chilean electronic invoices.
        These taxes are not VAT taxes in general; they are special taxes (for example, alcohol or sugar-added beverages,
        withholdings for meat processing, fuel, etc.
        The taxes codes used are included here:
        [15, 17, 18, 19, 24, 25, 26, 27, 271]
        http://www.sii.cl/declaraciones_juradas/ddjj_3327_3328/cod_otros_imp_retenc.pdf
        The need of the tax is not just the amount, but the code of the tax, the percentage amount and the amount
        :return:
        """
        self.ensure_one()
        tax = [{'tax_code': line.tax_line_id.l10n_cl_sii_code,
                'tax_name': line.tax_line_id.name,
                'tax_base': abs(sum(self.invoice_line_ids.filtered(
                    lambda x: line.tax_line_id.l10n_cl_sii_code in x.tax_ids.mapped('l10n_cl_sii_code')).mapped(
                    'balance'))),
                'tax_percent': abs(line.tax_line_id.amount),
                'tax_amount_currency': self.currency_id.round(abs(line.amount_currency)),
                'tax_amount': self.currency_id.round(abs(line.balance))} for line in self.line_ids.filtered(
            lambda x: x.tax_group_id.id in [self.env['account.chart.template'].with_company(self.company_id).ref('tax_group_ila').id,
                                            self.env['account.chart.template'].with_company(self.company_id).ref('tax_group_retenciones').id])]
        return tax

    def _float_repr_float_round(self, value, decimal_places):
        return float_repr(float_round(value, decimal_places), decimal_places)

    def _compute_tax_totals(self):
        # OVERRIDE 'account'
        super()._compute_tax_totals()
        for move in self:
            if move.tax_totals and move._get_name_invoice_report() == 'l10n_cl.report_invoice_document':
                # Disable the recap of tax totals in company currency at the bottom right of the invoice,
                # since this info is already present in our custom tax totals grid.
                move.tax_totals['display_in_company_currency'] = False

```

## File: models\account_move_line.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.tools.float_utils import float_repr


class AccountMoveLine(models.Model):

    _inherit = 'account.move.line'

    def _l10n_cl_prices_and_taxes(self):
        """ this method is preserved here to allow compatibility with old templates,
        Nevertheless it will be deprecated in future versions, since it had been replaced by
        the method _l10n_cl_get_line_amounts, which is the same method used to calculate
        the values for the XML (DTE) file
        """
        self.ensure_one()
        invoice = self.move_id
        included_taxes = self.tax_ids.filtered(lambda x: x.l10n_cl_sii_code == 14) if self.move_id._l10n_cl_include_sii() else self.tax_ids
        if not included_taxes:
            price_unit = self.tax_ids.compute_all(
                self.price_unit,
                currency=invoice.currency_id,
                product=self.product_id,
                partner=invoice.partner_id,
                rounding_method='round_globally',
            )
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

    def _l10n_cl_get_line_amounts(self):
        """
        This method is used to calculate the amount and taxes of the lines required in the Chilean localization
        electronic documents.
        """
        # If in this fix we should check for boletas, we have the following cases, and how this affects the xml
        # for facturas and boletas:

        # 1. local invoice in same currency tax not included in price
        # 2. local invoice in same currency tax included in price (there is difference of -1 peso in amount_untaxed
        # and +1 peso in vat tax amount. The lines are OK
        # 3. local invoice in different currency tax not included in price
        # 4. local invoice in different currency tax include in price -> this is the most problematic case because
        # 5. foreign invoice in different currency (without tax)
        if self.display_type != 'product':
            return {
                'price_subtotal': 0,
            }
        line_sign = self.price_subtotal / abs(self.price_subtotal) if self.price_subtotal else 0
        domestic_invoice_other_currency = self.move_id.currency_id != self.move_id.company_id.currency_id and not \
            self.move_id.l10n_latam_document_type_id._is_doc_type_export()
        export = self.move_id.l10n_latam_document_type_id._is_doc_type_export()
        if not export:
            # This is to manage case 1, 2, 3 and 4
            # cases 1 and 2: domestic invoice in same currency and cases 3 and 4 with other currency
            main_currency = self.move_id.company_id.currency_id
            main_currency_field = 'balance'
            second_currency_field = 'price_subtotal'
            second_currency = self.currency_id
            main_currency_rate = 1
            second_currency_rate = 1 / self.move_id.invoice_currency_rate if self.move_id.invoice_currency_rate else 1
            inverse_rate = second_currency_rate if domestic_invoice_other_currency else main_currency_rate
        else:
            # This is to manage case 5 (export docs)
            main_currency = self.currency_id
            second_currency = self.move_id.company_id.currency_id
            main_currency_field = 'price_subtotal'
            second_currency_field = 'balance'
            inverse_rate = 1 / self.move_id.invoice_currency_rate if self.move_id.invoice_currency_rate else 1
        price_subtotal = abs(self[main_currency_field]) * line_sign
        if self.quantity and self.discount != 100.0:
            price_unit = (price_subtotal / abs(self.quantity)) / (1 - self.discount / 100)
            if self.move_id.l10n_latam_document_type_id._is_doc_type_electronic_ticket():
                price_item_document = (self.price_total / abs(self.quantity)) / (1 - self.discount / 100)
                price_line_document = self.price_total
            else:
                price_item_document = price_unit
                price_line_document = price_subtotal
        else:
            price_item_document = price_line_document = 0.0
            price_unit = self.price_unit

        if self.discount == 100:
            price_before_discount = price_unit * self.quantity
        else:
            price_before_discount = price_subtotal / (1 - self.discount / 100)
        discount_amount = price_before_discount * self.discount / 100
        values = {
            'decimal_places': main_currency.decimal_places,
            'price_item': round(price_unit, 6),
            'price_item_document': round(price_item_document, 2),
            'price_line_document': price_line_document,
            'total_discount': main_currency.round(discount_amount),
            'price_subtotal': main_currency.round(price_subtotal),
            'exempt': bool(not self.tax_ids),
            'main_currency': main_currency,
        }
        if domestic_invoice_other_currency or export:
            price_subtotal_second = abs(self[second_currency_field]) * line_sign
            if self.quantity and self.discount != 100.0:
                price_unit_second = (price_subtotal_second / abs(self.quantity)) / (1 - self.discount / 100)
            else:
                price_unit_second = self.price_unit
            discount_amount_second = price_unit_second * self.quantity - price_subtotal_second
            values['second_currency'] = {
                'price': second_currency.round(price_unit_second),
                'currency_name': self.move_id._format_length(second_currency.name, 3),
                'conversion_rate': round(inverse_rate, 4),
                'amount_discount': second_currency.round(discount_amount_second),
                'total_amount': second_currency.round(price_subtotal_second),
                'round_currency': second_currency.decimal_places,
            }

        values['line_description'] = '%s (%s: %s @ %s)' % (
            self.name,
            values['second_currency']['currency_name'],
            float_repr(values['second_currency']['price'], values['second_currency']['round_currency']),
            self.move_id._float_repr_float_round(values['second_currency']['conversion_rate'], values['second_currency']['round_currency']),
        ) if values.get('second_currency') and not self.l10n_latam_document_type_id._is_doc_type_export() else self.name
        return values

```

## File: models\account_tax.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_cl_sii_code = fields.Integer('SII Code', aggregator=False)

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
            ('receipt_invoice', 'Receipt Invoice'),
            ('stock_picking', 'Stock Delivery'),
        ],
    )
    l10n_cl_active = fields.Boolean(
        'Active in localization', help='This boolean enables document to be included on invoicing')

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

    def _is_doc_type_export(self):
        return self.code in ['110', '111', '112'] and self.country_id.code == 'CL'

    def _is_doc_type_electronic_ticket(self):
        return self.code in ['39', '41'] and self.country_id.code == 'CL'

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class ResCompany(models.Model):
    _inherit = "res.company"

    l10n_cl_activity_description = fields.Char(
        string='Company Activity Description', related='partner_id.l10n_cl_activity_description', readonly=False)

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

    l10n_cl_currency_code = fields.Char('Currency Code', translate=True)
    l10n_cl_short_name = fields.Char('Short Name', translate=True)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import stdnum
from odoo import api, fields, models


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    l10n_cl_sii_taxpayer_type = fields.Selection(
        [
            ('1', 'VAT Affected (1st Category)'),
            ('2', 'Fees Receipt Issuer (2nd category)'),
            ('3', 'End Consumer'),
            ('4', 'Foreigner'),
        ],
        string='Taxpayer Type',
        index='btree_not_null',
        help='1 - VAT Affected (1st Category) (Most of the cases)\n'
             '2 - Fees Receipt Issuer (Applies to suppliers who issue fees receipt)\n'
             '3 - End consumer (only receipts)\n'
             '4 - Foreigner')
    l10n_cl_activity_description = fields.Char(string='Activity Description', help="Chile: Economic activity.")

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

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('vat'):
                vals['vat'] = self._format_vat_cl(vals)
        return super().create(vals_list)

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

    def _get_fiscal_country_codes(self):
        return ','.join(self.env.companies.mapped('account_fiscal_country_id.code'))

    l10n_cl_sbif_code = fields.Char('Cod. SBIF', size=10)
    fiscal_country_codes = fields.Char(store=False, default=_get_fiscal_country_codes)

```

## File: models\template_cl.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cl')
    def _get_cl_template_data(self):
        return {
            'code_digits': '6',
            'property_account_receivable_id': 'account_110310',
            'property_account_payable_id': 'account_210210',
            'property_account_expense_categ_id': 'account_410235',
            'property_account_income_categ_id': 'account_310115',
            'property_stock_account_input_categ_id': 'account_210230',
            'property_stock_account_output_categ_id': 'account_110640',
            'property_stock_valuation_account_id': 'account_110610',
        }

    @template('cl', 'res.company')
    def _get_cl_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.cl',
                'bank_account_code_prefix': '1101',
                'cash_account_code_prefix': '1101',
                'transfer_account_code_prefix': '117',
                'account_default_pos_receivable_account_id': 'account_110421',
                'income_currency_exchange_account_id': 'account_320265',
                'expense_currency_exchange_account_id': 'account_410195',
                'tax_calculation_rounding_method': 'round_globally',
                'account_sale_tax_id': 'ITAX_19',
                'account_purchase_tax_id': 'OTAX_19',
            },
        }

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_cl
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
                <attribute name="invisible" add="(not l10n_latam_use_documents or not posted_before or state != 'draft' or country_code != 'CL')" separator=" and "/>
                <attribute name="readonly">posted_before and state != 'draft'</attribute>
                <attribute name="required">l10n_latam_manual_document_number</attribute>
            </field>
        </field>
    </record>

    <record id="view_complete_invoice_refund_tree" model="ir.ui.view">
        <field name="name">account.move.list2</field>
        <field name="model">account.move</field>
        <field name="arch" type="xml">
            <list decoration-info="state == 'draft'" default_order="create_date" string="Invoices and Refunds" decoration-muted="state == 'cancel'" js_class="account_tree">
                <field name="l10n_latam_document_type_id_code"/>
                <field name="l10n_latam_document_number" string="Folio" readonly="state != 'draft'"/>
                <field name="partner_id_vat"/>
                <field name="partner_id" readonly="state != 'draft'"/>
                <field name="invoice_date" optional="show" readonly="state != 'draft'"/>
                <field name="invoice_date_due" optional="show"/>
                <field name="date" string="Accounting Date" optional="show" readonly="state in ['cancel', 'posted']"/>
                <field name="payment_reference" optional="hide"/>
                <field name="invoice_user_id" optional="show" column_invisible="context.get('default_move_type') not in ('out_invoice', 'out_refund', 'out_receipt')" string="Sales Person"/>
                <field name="company_id" groups="base.group_multi_company" options="{'no_create': True}" optional="show"/>
                <field name="invoice_origin" optional="show" string="Source Document"/>
                <field name="amount_untaxed_signed" string="Amount Untaxed" sum="Total" optional="show"/>
                <field name="amount_tax_signed" string="Tax" sum="Total" optional="show"/>
                <field name="amount_total_signed" string="Total" sum="Total" optional="show"/>
                <field name="amount_residual_signed" string="Amount Due" sum="Amount Due" optional="show"/>
                <field name="currency_id" column_invisible="True" readonly="state in ['cancel', 'posted']"/>
                <field name="company_currency_id" column_invisible="True"/>
                <field name="state" optional="show"/>
                <field name="payment_state" optional="hide"/>
                <field name="move_type" column_invisible="context.get('default_move_type', True)"/>
            </list>
        </field>
    </record>

    <record model="ir.actions.act_window" id="sale_invoices_credit_notes">
        <field name="name">Sale Invoices and Credit Notes</field>
        <field name="view_id" ref="view_complete_invoice_refund_tree"/>
        <field name="res_model">account.move</field>
        <field name="domain">[('move_type', 'in', ['out_invoice', 'out_refund'])]</field>
        <field name="context">{'default_move_type': 'out_invoice'}</field>
        <field name="target">current</field>
        <field name="view_mode">list,form</field>
    </record>

    <record model="ir.actions.act_window" id="vendor_bills_and_refunds">
        <field name="name">Vendor Bills and Refunds</field>
        <field name="view_id" ref="view_complete_invoice_refund_tree"/>
        <field name="res_model">account.move</field>
        <field name="domain">[('move_type', 'in', ['in_invoice', 'in_refund'])]</field>
        <field name="context">{'default_move_type': 'in_invoice'}</field>
        <field name="target">current</field>
        <field name="view_mode">list,form</field>
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
                    <field name="l10n_cl_sii_code" options="{'format': false}" invisible="country_code != 'CL'"/>
                </field>
            </field>
        </record>

        <record id="view_tax_sii_code_tree" model="ir.ui.view">
            <field name="name">account.tax.sii.code.list</field>
            <field name="model">account.tax</field>
            <field name="inherit_id" ref="account.view_tax_tree" />
            <field name="arch" type="xml">
                <field name="name" position="before">
                    <field name="l10n_cl_sii_code" options="{'format': false}" optional="hide"/>
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
            <field name="internal_type" position="before">
                <field name="l10n_cl_active" widget="boolean_toggle"/>
            </field>
        </field>
    </record>

    <record id="l10n_cl_latam_document_type_view_tree" model="ir.ui.view">
        <field name="name">l10n_cl_latam_document_type_view_tree</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_document_type_tree"/>
        <field name="arch" type="xml">
            <field name="sequence" position="attributes">
                <attribute name="widget">handle</attribute>
            </field>
            <field name="active" position="before">
                <field name="l10n_cl_active" widget="boolean_toggle"/>
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

        <div class="mb-3">
            <div class="row">
                <div name="left-upper-side" class="col-8">
                    <img t-if="o.company_id.logo" t-att-src="image_data_uri(o.company_id.logo)"
                         style="max-height: 45px;" alt="Logo"/>
                    <br/>
                    <strong>
                        <span t-field="o.company_id.partner_id.name"/>
                    </strong>
                    <br/>
                    <span name="company_activity" class="fst-italic" t-field="o.company_id.report_header"/>
                    <div/>
                    <t t-out="' - '.join([item for item in [
                        ', '.join([item for item in [header_address.street, header_address.street2] if item]),
                        header_address.city,
                        header_address.state_id and header_address.state_id.name,
                        header_address.zip,
                        header_address.country_id and header_address.country_id.name] if item])"/>
                    <span t-if="header_address.phone">
                        <br/>
                    </span>
                    <span t-if="header_address.phone" style="white-space: nowrap;"
                          t-out="'Tel: ' + header_address.phone"/>
                    <span t-if="header_address.website">
                        <span t-att-style="'color: %s;' % o.company_id.primary_color"
                              t-out="'- Web: %s' %' - '.join([item for item in [header_address.website.replace('https://', '').replace('http://', ''), header_address.email] if item])"/>
                    </span>
                </div>
                <div name="right-upper-side" class="col-4">
                    <div class="row">
                        <div name="right-upper-side" class="col-12">
                            <div class="row border border-4 border-dark" style="max-width: 100%;">
                                <div class="col-12 text-center">
                                    <h6 style="color: black;">
                                        <strong>
                                            <br/>
                                            <span style="font-family:arial; line-height: 180%;">RUT:</span>
                                            <t t-if="o.company_id.partner_id.vat">
                                                <span style="font-family:arial;" t-out="o.company_id.partner_id._format_dotted_vat_cl(o.company_id.partner_id.vat)"/>
                                            </t>
                                            <br/>
                                            <span style="font-family:arial;" class="text-uppercase" t-out="report_name"/>
                                            <br/>
                                            <span>Nº:</span>
                                            <span style="font-family:arial;line-height: 200%;" t-out="report_number"/>
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
                <span t-out="o.invoice_date" t-options='{"widget": "date"}'/>
                <br/>

                <strong>Customer:</strong>
                <span t-out="o.partner_id.commercial_partner_id.name or o.partner_id.name"/>
                <br/>

                <t t-if="o.partner_id.vat and o.partner_id.l10n_latam_identification_type_id">
                    <strong>
                        <t t-out="o.partner_id.l10n_latam_identification_type_id.name or o.company_id.account_fiscal_country_id.vat_label" id="inv_tax_id_label"/>:
                    </strong>
                    <span t-out="o.partner_id.vat"/>
                    <br/>
                </t>
                <strong>Address:</strong>
                <span t-out="o.partner_id._display_address(without_company=True)"/>
            </div>
            <div class="col-6">
                <strong>Due Date:</strong>
                <span t-out="o.invoice_date_due" t-options='{"widget": "date"}'/>
                <br/>

                <t t-if="o.invoice_incoterm_id">
                    <br/>
                    <strong>Incoterm:</strong>
                    <span t-field="o.invoice_incoterm_id.name"/>
                </t>

                <t t-if="o.partner_shipping_id and o.partner_id not in o.partner_shipping_id" >
                    <br/>
                    <strong>Delivery Address:</strong>
                    <span t-out="o.partner_shipping_id._display_address(without_company=True)"/>
                </t>
                <br/>
                <strong>GIRO:</strong>
                <span t-out="o.partner_id.industry_id.name or ''"/>
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
        <xpath expr="//div[@name='address_not_same_as_shipping']" position="replace">
            <div name="address_not_same_as_shipping"/>
        </xpath>
        <xpath expr="//div[@name='address_same_as_shipping']" position="replace">
            <div name="address_same_as_shipping"/>
        </xpath>
        <xpath expr="//div[@name='no_shipping']" position="replace">
            <div name="no_shipping"/>
        </xpath>

        <xpath expr="//t[@t-set='layout_document_title']" position="replace"/>

        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" position="before">
            <t t-set="line_amounts" t-value="line._l10n_cl_get_line_amounts()"/>
        </t>

        <xpath expr="//span[@t-field='line.price_unit']" position="before">
            <t t-if="'second_currency' in line_amounts" t-set="line_second_currency_round" t-value="line_amounts['second_currency']['round_currency']"/>
        </xpath>

        <xpath expr="//span[@t-field='line.price_unit']" position="attributes">
            <attribute name="t-field"></attribute>
            <attribute name="t-out">line_amounts['price_item_document']</attribute>
            <attribute name="t-options">{"widget": "float", "precision": 2}</attribute>
        </xpath>

        <xpath expr="//th[@name='th_discount']" position="after">
            <th name="th_discount_currency" t-if="display_discount" t-attf-class="text-end {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}">
                <span>Disc.</span>
            </th>
        </xpath>

        <td name="td_discount" t-if="display_discount" t-attf-class="text-end {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}" position="after">
            <td name="td_discount_currency" t-if="display_discount" t-attf-class="text-end {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}" position="after">
                <span class="text-nowrap" t-out="line_amounts['total_discount']" t-options="{'widget': 'monetary', 'display_currency': line_amounts['main_currency']}"/>
            </td>
        </td>


        <xpath expr="//span[@id='line_tax_ids']" position="attributes">
            <attribute name="t-out">', '.join(map(lambda x: (x.description or x.name), line.tax_lines))</attribute>
        </xpath>

        <span t-field="line.price_subtotal" position="attributes">
            <attribute name="t-field"/>
            <attribute name="t-out">line_amounts['price_line_document']</attribute>
            <attribute name="t-options">{"widget": "monetary", "display_currency": line_amounts['main_currency']}</attribute>
        </span>

        <xpath expr="//th[@name='th_taxes']" position="replace"/>
        <xpath expr="//span[@id='line_tax_ids']/.." position="replace"/>

        <!-- To remove? <div name="payment_term" position="replace"/>-->
        <xpath expr="//span[@t-field='o.payment_reference']/../.." position="replace"/>

        <!-- replace information section and usage chilean style -->
        <div id="informations" position="replace">
            <t t-call="l10n_cl.informations"/>
        </div>

        <xpath expr="//div[@id='right-elements']" position="after">
            <div name="stamp" class="col-6 text-center"/>
        </xpath>

        <xpath expr="//div[@id='right-elements']" position="inside">
            <div class="row">
                <div class="col-12 text-center" t-if="o.l10n_latam_document_type_id.code == '39'" name="vat_boleta">
                    The VAT tax of this boleta is: <span t-out="o._l10n_cl_get_amounts()['vat_amount']" t-options="{'widget': 'monetary', 'display_currency': o.currency_id}"/>.
                </div>
                <div name="transferable-table" class="col-6"/>
                <div name="transferable-legend" class="col-6 text-end"/>
            </div>
        </xpath>

        <xpath expr="//span[@t-field='line.name']" position="replace">
            <t t-set="all_taxes" t-value="'all_taxes'"/>
            <t t-if="'second_currency' in line_amounts" t-set="line_second_currency_round" t-value="line_amounts['second_currency']['round_currency']"/>
            <span t-out="line_amounts['line_description']" t-options="{'widget': 'text'}"/>
        </xpath>

        <t t-call="account.document_tax_totals" position="attributes">
            <attribute name="t-call">l10n_cl.tax_totals_widget</attribute>
        </t>

    </template>

    <!-- FIXME: Temp fix to allow fetching invoice_documemt in Studio Reports with localisation -->
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr="//t[@t-call='account.report_invoice_document']" position="after">
            <t t-if="o._get_name_invoice_report() == 'l10n_cl.report_invoice_document'"
                t-call="l10n_cl.report_invoice_document" t-lang="lang"/>
        </xpath>

    </template>

    <template id="tax_totals_widget" inherit_id="account.document_tax_totals" primary="True">
        <t t-foreach="tax_totals['subtotals']" t-as="subtotal" position="replace">
            <t t-set="subtotal_amounts" t-value="o._l10n_cl_get_amounts()"/>
            <t t-set="withholdings" t-value="o._l10n_cl_get_withholdings()"/>
            <t t-if="subtotal_amounts['subtotal_amount_taxable'] != 0.0">
                <tr class="border-black is-subtotal"><td><strong>Net Amount</strong></td><td class="text-end oe_subtotal_footer_separator" t-out="subtotal_amounts['subtotal_amount_taxable']" t-options="{'widget': 'monetary', 'display_currency': subtotal_amounts['main_currency']}"/></tr>
            </t>
            <t t-if="subtotal_amounts['subtotal_amount_exempt'] != 0.0">
                <tr class="border-black is-subtotal">
                    <td><strong>Exempt Amount</strong></td>
                    <td class="text-end oe_subtotal_footer_separator" t-out="subtotal_amounts['subtotal_amount_exempt']"
                        t-options="{'widget': 'monetary', 'display_currency': subtotal_amounts['main_currency']}"/>
                </tr>
            </t>
            <t t-if="subtotal_amounts['vat_amount'] != 0.0">
                <tr>
                    <td>VAT <t t-esc="subtotal_amounts['vat_percent']"/>%</td>
                    <td class="text-end" t-out="subtotal_amounts['vat_amount']"
                        t-options="{'widget': 'monetary', 'display_currency': subtotal_amounts['main_currency']}"/></tr>
            </t>
            <t t-foreach="withholdings" t-as="wh">
                <tr>
                    <td t-out="'%s (base %s)' % (wh['tax_name'], o._format_lang_totals(wh['tax_base'], subtotal_amounts['main_currency']))"/>
                    <td class="text-end" t-out="wh['tax_amount']" t-options="{'widget': 'monetary', 'display_currency': subtotal_amounts['main_currency']}"/></tr>
            </t>
        </t>
        <tr class="o_total" position="replace">
            <tr class="o_total">
                <td><strong>Total</strong></td>
                <td class="text-end">
                    <span t-if="0" t-out="subtotal_amounts['total_amount']" t-options="{'widget': 'monetary', 'display_currency': subtotal_amounts['main_currency']}"/>
                    <strong t-out="subtotal_amounts['total_amount']" t-options="{'widget': 'monetary', 'display_currency': subtotal_amounts['main_currency']}"/>
                </td>
            </tr>
        </tr>
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
                    <field name="fiscal_country_codes" invisible="1"/>
                    <field name="l10n_cl_sbif_code" invisible="'CL' not in fiscal_country_codes"/>
                </field>
            </field>
        </record>

        <record model="ir.ui.view" id="view_res_bank_tree">
            <field name="name">bank.bank.list</field>
            <field name="model">res.bank</field>
            <field name="inherit_id" ref="base.view_res_bank_tree" />
            <field name="arch" type="xml">
                <field name="name" position="before">
                    <field name="l10n_cl_sbif_code" optional="hide"/>
                </field>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\res_company_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_company_l10n_cl_form" model="ir.ui.view">
            <field name="model">res.company</field>
            <field name="name">view.company.l10n.cl.form</field>
            <field name="inherit_id" ref="base.view_company_form" />
            <field name="arch" type="xml">
                <field name="vat" position="after">
                    <field name="l10n_cl_activity_description" placeholder="Activity Description"
                        invisible="country_id != %(base.cl)d"
                        required="country_id == %(base.cl)d"/>
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
            <xpath expr="//block[@id='invoicing_settings']" position="after">
                <block title="Chilean Localization" id="l10n_cl_section" invisible="country_code != 'CL'">
                    <!-- inside empty to add configuration of tags -->
                </block>
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
        <field name="name">res.country.list</field>
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
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="model">res.partner</field>
        <field name="arch" type="xml">
            <field name="street2" position="attributes">
                <attribute name="placeholder">Additional data address and city</attribute>
            </field>
            <field name="city" position="attributes">
                <attribute name="placeholder">Commune</attribute>
            </field>
            <field name="state_id" position="attributes">
                <attribute name="placeholder">Region</attribute>
            </field>
            <field name="vat" position="after">
                <field name="l10n_cl_sii_taxpayer_type" invisible="'CL' not in fiscal_country_codes" readonly="parent_id"/>
                <field name="l10n_cl_activity_description" placeholder="Activity Description" invisible="'CL' not in fiscal_country_codes"/>
            </field>
        </field>
    </record>

</odoo>

```

