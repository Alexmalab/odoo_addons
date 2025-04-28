# Odoo Module: l10n_uy

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from . import demo

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Uruguay - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['uy'],
    'version': '0.1',
    'author': 'Uruguay l10n Team, Guillem Barba, ADHOC',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
General Chart of Accounts.
==========================

This module adds accounting functionalities for the Uruguayan localization, representing the minimum required configuration for a company to operate in Uruguay under the regulations and guidelines provided by the DGI (Dirección General Impositiva).

Among the functionalities are:

* Uruguayan Generic Chart of Account
* Pre-configured VAT Taxes and Tax Groups.
* Legal document types in Uruguay.
* Valid contact identification types in Uruguay.
* Configuration and activation of Uruguayan Currencies  (UYU, UYI - Unidad Indexada Uruguaya).
* Frequently used default contacts already configured: DGI, Consumidor Final Uruguayo.

Configuration
-------------

Demo data for testing:

* Uruguayan company named "UY Company" with the Uruguayan chart of accounts already installed, pre configured taxes, document types and identification types.
* Uruguayan contacts for testing:

   * IEB Internacional
   * Consumidor Final Anónimo Uruguayo.

""",
    'depends': [
        'account',
        'l10n_latam_invoice_document',
        'l10n_latam_base',
    ],
    'data': [
        'data/account_tax_report_data.xml',
        'data/l10n_latam.document.type.csv',
        'data/l10n_latam_identification_type_data.xml',
        'data/res_partner_data.xml',
        'views/account_tax_views.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
        'demo/res_currency_rate_demo.xml',
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
        <field name="country_id" ref="base.uy"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_base_impb" model="account.report.line">
                <field name="name">Taxable income</field>
                <field name="aggregation_formula">BASE_IMPONIBLE_COMPRAS.balance + BASE_IMPONIBLE_VENTAS.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_base_impb_cmprs" model="account.report.line">
                        <field name="name">Tax Base Purchases</field>
                        <field name="code">BASE_IMPONIBLE_COMPRAS</field>
                        <field name="aggregation_formula">UYTAX_010101.balance + UYTAX_020101.balance + UYTAX_030101.balance + UYTAX_040101.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_base_impb_cmprs_22" model="account.report.line">
                                <field name="name">Base Purchases 22%</field>
                                <field name="code">UYTAX_010101</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_base_impb_cmprs_22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Base Purchases 22%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_base_impb_cmprs_10" model="account.report.line">
                                <field name="name">Base Purchases 10%</field>
                                <field name="code">UYTAX_020101</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_base_impb_cmprs_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Base Purchases 10%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_base_impb_cmprs_0" model="account.report.line">
                                <field name="name">Base Purchases 0%</field>
                                <field name="code">UYTAX_030101</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_base_impb_cmprs_0_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Base Purchases 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_impb_cmprs" model="account.report.line">
                                <field name="name">Tax Base Purchases</field>
                                <field name="code">UYTAX_040101</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_impb_cmprs_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Tax Base Purchases</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_base_impb_vnts" model="account.report.line">
                        <field name="name">Taxable Sales Base</field>
                        <field name="code">BASE_IMPONIBLE_VENTAS</field>
                        <field name="aggregation_formula">UYTAX_010201.balance + UYTAX_020201.balance + UYTAX_030201.balance + UYTAX_040201.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_base_impb_vnts_22" model="account.report.line">
                                <field name="name">Base Sales 22%</field>
                                <field name="code">UYTAX_010201</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_base_impb_vnts_22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Base Sales 22%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_base_impb_vnts_10" model="account.report.line">
                                <field name="name">Base Sales 10%</field>
                                <field name="code">UYTAX_020201</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_base_impb_vnts_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Base Sales 10%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_base_impb_vnts_0" model="account.report.line">
                                <field name="name">Base Sales 0%</field>
                                <field name="code">UYTAX_030201</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_base_impb_vnts_0_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Base Sales 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_impb_vnts" model="account.report.line">
                                <field name="name">Taxable Sales Base</field>
                                <field name="code">UYTAX_040201</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_impb_vnts_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Taxable Sales Base</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_sldo_iva" model="account.report.line">
                <field name="name">VAT balance</field>
                <field name="aggregation_formula">IVA_COMPRAS__PAGADO.balance + UYTAX_040102.balance + IVA_VENTAS__PERCIBIDO.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_iva_cmprs_pagdo" model="account.report.line">
                        <field name="name">VAT Purchases - paid</field>
                        <field name="code">IVA_COMPRAS__PAGADO</field>
                        <field name="aggregation_formula">UYTAX_010102.balance + UYTAX_020102.balance + COMPRAS_EXENTO_IVA.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_iva_cmprs_22" model="account.report.line">
                                <field name="name">VAT Purchases 22%</field>
                                <field name="code">UYTAX_010102</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_iva_cmprs_22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT Purchases 22%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_iva_cmprs_10" model="account.report.line">
                                <field name="name">VAT Purchases 10%</field>
                                <field name="code">UYTAX_020102</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_iva_cmprs_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT Purchases 10%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_cmprs_exnto_iva" model="account.report.line">
                                <field name="name">Purchases Exempt from VAT</field>
                                <field name="code">COMPRAS_EXENTO_IVA</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_cmprs_exnto_iva_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_cmprs_pagdo" model="account.report.line">
                        <field name="name">VAT Purchases - paid</field>
                        <field name="code">UYTAX_040102</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_cmprs_pagdo_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">VAT Purchases - paid</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_iva_vnts_prcbdo" model="account.report.line">
                        <field name="name">VAT Sales - received</field>
                        <field name="code">IVA_VENTAS__PERCIBIDO</field>
                        <field name="aggregation_formula">UYTAX_010202.balance + UYTAX_020202.balance + VENTAS_EXENTO_IVA.balance + UYTAX_040202.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_iva_vnts_22" model="account.report.line">
                                <field name="name">Sales VAT 22%</field>
                                <field name="code">UYTAX_010202</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_iva_vnts_22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Sales VAT 22%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_iva_vnts_10" model="account.report.line">
                                <field name="name">Sales VAT 10%</field>
                                <field name="code">UYTAX_020202</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_iva_vnts_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Sales VAT 10%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_vnts_iva" model="account.report.line">
                                <field name="name">Sales VAT exempt</field>
                                <field name="code">VENTAS_EXENTO_IVA</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_vnts_iva_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_vnts_prcbdo" model="account.report.line">
                                <field name="name">VAT Sales - received</field>
                                <field name="code">UYTAX_040202</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_vnts_prcbdo_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT Sales - received</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_latam.document.type.csv

```csv
"id","code","name","internal_type","doc_code_prefix","country_id/id"
"dc_inv",0,"Invoice","invoice","FC","base.uy"
"dc_boleta_venta_contado",0,"Boleta","invoice","BO","base.uy"
"dc_cn_inv",0,"Credit Note","credit_note","NC","base.uy"
"dc_dn_inv",0,"Debit Note","debit_note","ND","base.uy"
"dc_recibo_cobranza",0,"Collection Receipt","invoice","RC","base.uy"
"dc_inv_expo",0,"Export Invoice","invoice","FCE","base.uy"
"dc_nc_expo",0,"Export Credit Note","credit_note","NCE","base.uy"
"dc_nd_expo",0,"Export Debit Note","debit_note","NDE","base.uy"
"dc_remito",0,"Delivery Guide",,"REM","base.uy"
"dc_e_ticket",101,"e-Ticket","invoice","e-TK","base.uy"
"dc_cn_e_ticket",102,"e-Ticket Credit Note","credit_note","e-NCTK","base.uy"
"dc_dn_e_ticket",103,"e-Ticket Debit Note","debit_note","e-NDTK","base.uy"
"dc_e_inv",111,"e-Invoice","invoice","e-FC","base.uy"
"dc_cn_e_inv",112,"e-Invoice Credit Note","credit_note","e-NC","base.uy"
"dc_dn_e_inv",113,"e-Invoice Debit Note","debit_note","e-ND","base.uy"
"dc_e_inv_exp",121,"Export e-Invoice","invoice","e-FCE","base.uy"
"dc_cn_e_inv_exp",122,"Export e-Invoice Credit Note","credit_note","e-NCE","base.uy"
"dc_dn_e_inv_exp",123,"Export e-Invoice Debit Note","debit_note","e-NDE","base.uy"
"dc_e_remito_expo",124,"Export e-Delivery Guide",,"e-REME","base.uy"
"dc_e_ticket_venta_por_cuenta_ajena",131,"e-Ticket Sale By Third Party","invoice","e-TK-CA","base.uy"
"dc_nota_de_credito_e_ticket_venta_por_cuenta_ajena",132,"e-Ticket Sale By Third Party Credit Note","credit_note","e-NCTK-CA","base.uy"
"dc_nota_de_debito_e_ticket_venta_por_cuenta_ajena",133,"e-Ticket Sale By Third Party Debit Note","debit_note","e-NDTK-CA","base.uy"
"dc_e_factura_venta_por_cuenta_ajena",141,"e-Invoice Sale By Third Party","invoice","e-FC-CA","base.uy"
"dc_nota_de_credito_e_factura_venta_por_cuenta_ajena",142,"e-Invoice Sale By Third Party Credit Note","credit_note","e-NC-CA","base.uy"
"dc_nota_de_debito_e_factura_venta_por_cuenta_ajena",143,"e-Invoice Sale By Third Party Debit Note","debit_note","e-ND-CA","base.uy"
"dc_e_boleta",151,"e-Boleta","invoice","e-BO","base.uy"
"dc_nota_de_credito_e_boleta",152,"Credit Note e-Boleta","credit_note","e-BO-NC","base.uy"
"dc_nota_de_debito_e_boleta",153,"Debit Note e-Boleta","debit_note","e-BO-ND","base.uy"
"dc_e_remito",181,"e-Delivery Guide",,"e-REM","base.uy"
"dc_e_resguardo",182,"e-Resguardo",,"e-RES","base.uy"
"dc_e_ticket_cont",201,"Contingency e-Ticket","invoice","e-TK-C","base.uy"
"dc_cn_e_ticket_cont",202,"Contingency e-Ticket Credit Note","credit_note","e-NCTK-C","base.uy"
"dc_dn_e_ticket_cont",203,"Contingency e-Ticket Debit Note","debit_note","e-NDTK-C","base.uy"
"dc_e_factura_contingencia",211,"Contingency e-Invoice","invoice","e-FC-C","base.uy"
"dc_nota_de_crédito_de_e_factura_contingencia",212,"Contingency e-Invoice Credit Note","credit_note","e-NC-C","base.uy"
"dc_nota_de_débito_de_e_factura_contingencia",213,"Contingency e-Invoice Debit Note","debit_note","e-ND-C","base.uy"
"dc_e_factura_exportación_contingencia",221,"Contingency Export e-Invoice","invoice","e-FCE-C","base.uy"
"dc_nota_de_crédito_de_e_factura_exportación_contingencia",222,"Contingency Export e-Invoice Credit Note","credit_note","e-NCE-C","base.uy"
"dc_nota_de_débito_de_e_factura_exportación_contingencia",223,"Contingency Export e-Invoice Debit Note","debit_note","e-NDE-C","base.uy"
"dc_e_remito_de_exportación_contingencia",224,"Contingency Export e-Delivery Guide",,"e-REME-C","base.uy"
"dc_e_ticket_venta_por_cuenta_ajena_contingencia",231,"Contingency e-Ticket Sale By Third Party","invoice","e-TK-CAC","base.uy"
"dc_nota_de_crédito_de_e_ticket_venta_por_cuenta_ajena_contingencia",232,"Contingency e-Ticket Sale By Third Party Credit Note","credit_note","e-NCTK-CAC","base.uy"
"dc_nota_de_débito_de_e_ticket_venta_por_cuenta_ajena_contingencia",233,"Contingency e-Ticket Sale By Third Party Debit Note","debit_note","e-NDTK-CAC","base.uy"
"dc_e_factura_venta_por_cuenta_ajena_contingencia",241,"Contingency e-Invoice Sale By Third Party","invoice","e-FC-CAC","base.uy"
"dc_nota_de_crédito_de_e_factura_venta_por_cuenta_ajena_contingencia",242,"Contingency e-Invoice Sale By Third Party Credit Note","credit_note","e-NC-CAC","base.uy"
"dc_nota_de_débito_de_e_factura_venta_por_cuenta_ajena_contingencia",243,"Contingency e-Invoice Sale By Third Party Debit Note","debit_note","e-ND-CAC","base.uy"
"dc_e_boleta_contingencia",251,"Contingency e-Boleta","invoice","e-BO-C","base.uy"
"dc_nota_de_credito_e_boleta_contingencia",252,"Contingency e-Boleta Credit Note","credit_note","e-BO-NC-C","base.uy"
"dc_nota_de_debito_e_boleta_contingencia",253,"Contingency e-Boleta Debit Note","debit_note","e-BO-ND-C","base.uy"
"dc_e_remito_contingencia",281,"Contingency e-Delivery Guide",,"e-REM-C","base.uy"
"dc_e_resguardo_contingencia",282,"Contingency e-Resguardo",,"e-RES-C","base.uy"

```

## File: data\l10n_latam_identification_type_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>

    <record model="l10n_latam.identification.type" id="it_nie">
        <field name="name">NIE</field>
        <field name="description">Foreigner Identity Number</field>
        <field name='country_id' ref='base.uy'/>
        <field name="sequence">10</field>
        <field name="l10n_uy_dgi_code">1</field>
    </record>

    <record model="l10n_latam.identification.type" id="it_rut">
        <field name="name">RUT / RUC</field>
        <field name="description">Unique Tax Registry / Unique Taxpayer Registry</field>
        <field name='country_id' ref='base.uy'/>
        <field name='is_vat' eval='True'/>
        <field name="sequence">30</field>
        <field name="l10n_uy_dgi_code">2</field>
    </record>

    <record model="l10n_latam.identification.type" id="it_ci">
        <field name="name">CI</field>
        <field name="description">Identification Card</field>
        <field name='country_id' ref='base.uy'/>
        <field name="sequence">40</field>
        <field name="l10n_uy_dgi_code">3</field>
    </record>

    <record model="l10n_latam.identification.type" id="it_other">
        <field name="name">OTR</field>
        <field name="description">Others</field>
        <field name='country_id' ref='base.uy'/>
        <field name="sequence">50</field>
        <field name="l10n_uy_dgi_code">4</field>
    </record>

    <record model="l10n_latam.identification.type" id="it_pass">
        <field name="name">PAS</field>
        <field name="description">Passport (all countries)</field>
        <field name='country_id' ref='base.uy'/>
        <field name="sequence">60</field>
        <field name="l10n_uy_dgi_code">5</field>
    </record>

    <record model="l10n_latam.identification.type" id="it_dni">
        <field name="name">DNI</field>
        <field name="description">National identity document of Argentina, Brazil, Chile or Paraguay</field>
        <field name='country_id' ref='base.uy'/>
        <field name="sequence">70</field>
        <field name="l10n_uy_dgi_code">6</field>
    </record>

    <record model="l10n_latam.identification.type" id="it_nife">
        <field name="name">NIFE</field>
        <field name="description">Foreign tax identification number</field>
        <field name='country_id' ref='base.uy'/>
        <field name="sequence">80</field>
        <field name="l10n_uy_dgi_code">7</field>
    </record>

    <record model="l10n_latam.identification.type" id="l10n_latam_base.it_vat">
        <field name="l10n_uy_dgi_code">7</field>
    </record>
    <record model="l10n_latam.identification.type" id="l10n_latam_base.it_pass">
        <field name="l10n_uy_dgi_code">5</field>
    </record>
    <record model="l10n_latam.identification.type" id="l10n_latam_base.it_fid">
        <field name="l10n_uy_dgi_code">4</field>
    </record>

</odoo>

```

## File: data\res_partner_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>

    <!-- DGI Is the Fiscal office on UY: Direccion General Impositiva -->
    <record id="partner_dgi" model="res.partner">
        <field name="name">DGI</field>
        <field name="is_company" eval="True"/>
        <field name='l10n_latam_identification_type_id' ref='it_rut'/>
        <field name='website'>https://www.dgi.gub.uy</field>
        <field name='country_id' ref='base.uy'/>
    </record>

    <record model='res.partner' id='partner_cfu'>
        <field name='name'>Consumidor Final Anónimo</field>
        <field name='l10n_latam_identification_type_id' ref='it_dni'/>
        <field name='country_id' ref='base.uy'/>
    </record>

</odoo>

```

## File: data\template\account.account-uy.csv

```csv
"id","name","code","account_type","reconcile","name@es"
"uy_code_11300","Sale Debtors","11300","asset_receivable","True","Deudores por Ventas"
"uy_code_11303","Notes Receivable MN","11303","asset_current","False","Documentos a Cobrar MN"
"uy_code_11304","Notes Receivable ME","11304","asset_current","False","Documentos a Cobrar ME"
"uy_code_11305","Checks in portfolio MN","11305","asset_current","False","Cheques en Cartera MN"
"uy_code_11306","Checks in ME Portfolio","11306","asset_current","False","Cheques en Cartera ME"
"uy_code_11307","Sundry Debtors (PoS)","11307","asset_receivable","True","Deudores Varios (PoS)"
"uy_code_11311","Allowance for Uncollectible Accounts Receivable","11311","asset_current","False","Prevision para Deudores Incobrables"
"uy_code_11312","Provisions for expenses and bonuses","11312","asset_current","False","Prevision p/dtos y Bonificaciones"
"uy_code_11313","Interest received in advance","11313","asset_current","False","Intereses percibidos por adelantado"
"uy_code_11401","Advances to Suppliers","11401","liability_current","False","Anticipos a Proveedores"
"uy_code_11404","Security Deposits","11404","asset_current","False","Depositos en Garantia"
"uy_code_11405","Advance payments","11405","liability_current","False","Pagos adelantados"
"uy_code_11406","Debit balances of Directors' accounts receivable","11406","asset_current","False","Saldos Deudor de ctas de Directores"
"uy_code_11501","VAT Minimum Purchases","11501","asset_current","False","Iva Compras Mínima"
"uy_code_11502","VAT Basic Purchases","11502","asset_current","False","Iva Compras Básica"
"uy_code_11503","VAT Exempt Purchases","11503","asset_current","False","Iva Compras Exento"
"uy_code_11505","VAT to be credited on purchases","11505","asset_receivable","True","IVA acreditable en compras"
"uy_code_11506","VAT Withholdings","11506","asset_current","False","Iva Retenciones"
"uy_code_11507","Import VAT","11507","asset_current","False","Iva Importación"
"uy_code_11508","Advance Import Tax","11508","asset_current","False","Iva Anticipo Importación"
"uy_code_11601","Tax on Income from Economic Activities - Advance","11601","asset_current","False","Irae Anticipo"
"uy_code_11602","Equity Advances","11602","asset_current","False","Patrimonio Anticipo"
"uy_code_11603","Icosa Advance","11603","asset_current","False","Icosa Anticipo"
"uy_code_11621","Various","11621","asset_current","False","Diversos"
"uy_code_11632","Income received in advance","4110","income","False","Ingresos percibidos por adelantado"
"uy_code_21321","Deferred Revenue","21321","liability_current","False","Ingreso diferidos"
"uy_code_11701","Resale Merchandise","11701","asset_current","False","Mercaderia de Reventa"
"uy_code_11702","Finished Products","11702","asset_current","False","Productos Terminados"
"uy_code_11703","Products in Process","11703","asset_current","False","Productos en Proceso"
"uy_code_11704","Raw Materials","11704","asset_current","False","Materias Primas"
"uy_code_11705","Materials and Supplies","11705","asset_current","False","Materiales y Suministros"
"uy_code_11706","Imports in process","11706","asset_current","False","Importaciones en tramite"
"uy_code_11711","Allowance for impairment","11711","asset_current","False","Previsión para desvalorizaciones"
"uy_code_12101","Long-term loans","12101","asset_current","False","Créditos a Largo Plazo"
"uy_code_12201","Non-current assets","12201","asset_fixed","False","Activos fijos"
"uy_code_12302","Real Estate","12302","asset_current","False","Inmuebles"
"uy_code_12303","Original and revaluated values as per appendix","12303","asset_current","False","Valores orig. y revaluados s/anexo"
"uy_code_12304","Less: Amort. Accum.","12304","asset_current","False","Menos: Amort. Acum."
"uy_code_12305","Securities and Shares","12305","asset_current","False","Titulos y Acciones"
"uy_code_12311","Allowance for Impairment","12311","expense","False","Prevision para Desvalorizaciones"
"uy_code_12312","Interest received in advance","12312","income","False","Intereses percibidos por adelantado"
"uy_code_12401","Furniture and Fixtures","12401","asset_fixed","False","Muebles y Útiles"
"uy_code_12402","Real Estate","12402","asset_fixed","False","Inmuebles"
"uy_code_12403","Machines and Tools","12403","asset_fixed","False","Maquinas y Herramientas"
"uy_code_12404","Vehicles","12404","asset_fixed","False","Vehículos"
"uy_code_12420","Furniture and Fixtures Amort.","12420","asset_current","False","Amort.Ac.Mueb.y Utiles"
"uy_code_12421","Real Estate Amort.","12421","asset_current","False","Amort.Ac.Inmuebles"
"uy_code_12422","Machinery and Equipment Amort.","12422","asset_current","False","Amort.Ac.Maq.y Herram."
"uy_code_12423","Vehicle Amort. Shrinkage","12423","asset_current","False","Amort.Ac.Vehiculos"
"uy_code_12501","Patents, trademarks and licenses","12501","asset_fixed","False","Patentes, marcas y licencias"
"uy_code_12502","Research expenses","12502","asset_fixed","False","Gastos de investigacion"
"uy_code_12520","Accumulated Depreciation","12520","asset_current","False","Amortizaciones Acumuladas"
"uy_code_11407","Deferred Expense","11407","asset_current","False","Gastos diferidos"
"uy_code_21100","Sundry creditors (def)","21100","liability_payable","True","Acreedores Varios (def)"
"uy_code_21101","Suppliers by Imports","21101","liability_payable","True","Proveedores por Importaciones"
"uy_code_21102","Debts. Import Exchange Contracts","21102","liability_current","False","Deuds. Contratos de Cambio Import."
"uy_code_21103","Plaza Suppliers","21103","liability_payable","True","Proveedores de Plaza"
"uy_code_21104","Notes Payable ds/Commercial","21104","liability_payable","True","Documentos a Pagar ds/Comerciales"
"uy_code_21120","Interest due ds/Commercial","21120","liability_current","False","Intereses a vencer ds/Comerciales"
"uy_code_21201","Bank Loans","21201","liability_current","False","Prestamos Bancarios"
"uy_code_21202","Obligations","21202","liability_current","False","Obligaciones"
"uy_code_21203","Notes payable MN payable ds/Financials","21203","liability_payable","True","Documentos a pagar MN a pagar ds/Financieras"
"uy_code_21204","Notes payable EM payable ds/Financials","21204","liability_payable","True","Documentos a pagar ME a pagar ds/Financieras"
"uy_code_21220","Ints. due ds/Financials","53011","expense","False","Ints. a vencer ds/Financieras"
"uy_code_21301","Advance Collections","21301","liability_current","False","Cobros Anticipados"
"uy_code_21302","Dividends Payable","21302","liability_payable","True","Dividendos a Pagar"
"uy_code_21305","Wages and salaries payable","21305","liability_payable","True","Sueldos y Jornales a pagar"
"uy_code_21306","Social Creditors","21306","liability_current","False","Acreedores por Cargas Sociales"
"uy_code_21307","Tax creditors","21307","liability_current","False","Acreedores fiscales"
"uy_code_21308","Credit balances Payable to Directors' Accounts","21308","liability_current","False","Saldos Acreedores Cuentas Directores"
"uy_code_21320","Other debts","21320","liability_current","False","Otras deudas"
"uy_code_21401","Minimum Sales Tax","21401","liability_current","False","Iva Ventas Mínima"
"uy_code_21402","Basic Sales Tax","21402","liability_current","False","Iva Ventas Básica"
"uy_code_21403","Tax to be Paid","21403","liability_payable","True","Iva a Pagar"
"uy_code_21404","VAT Withholding","21404","liability_current","False","Iva Retenido"
"uy_code_21405","Exempt Sales Tax","21405","liability_current","False","Iva Ventas Exento"
"uy_code_21411","Withholding tax","21411","liability_current","False","Irpf Retenido"
"uy_code_21421","Social Security Bank of Uruguay","21421","liability_current","False","Bps"
"uy_code_21501","Tax on Income from Economic Activities - of the Exercise","21501","liability_current","False","Irae del Ejercicio"
"uy_code_21502","Tax on Income from Economic Activities - to Pay","21502","liability_payable","True","Irae a Pagar"
"uy_code_21503","Tax on Income from Economic Activities - Advance Payable","21503","liability_payable","True","Irae Anticipo a Pagar"
"uy_code_21601","Net Assets for the Year","21601","liability_current","False","Patrimonio del Ejercicio"
"uy_code_21602","Assets Payable","21602","liability_payable","True","Patrimonio a Pagar"
"uy_code_21603","Equity Advances Payable","21603","liability_payable","True","Patrimonio Anticipo a Pagar"
"uy_code_21701","Exercise Icosa","21701","liability_current","False","Icosa del Ejercicio"
"uy_code_21702","Icosa to be Paid","21702","liability_payable","True","Icosa a Pagar"
"uy_code_21703","Icosa Advance Payable","21703","liability_payable","True","Icosa Anticipo a Pagar"
"uy_code_21801","Third-party liability","21801","liability_current","False","Responsabilidad frente a terceros"
"uy_code_22101","Commercial Debts","22101","liability_current","False","Deudas Comerciales"
"uy_code_22201","Financial Debts","22201","liability_payable","True","Deudas Financieras"
"uy_code_22301","Sundry Debts","22301","liability_current","False","Deudas Diversas"
"uy_code_22401","Non-Current Provisions","22401","liability_current","False","Previsiones No Corrientes"
"uy_code_3111","Integrated Capital","3111","equity","False","Capital Integrado"
"uy_code_3221","Tax revaluations","3221","equity","False","Revaluaciones fiscales"
"uy_code_3222","Voluntary revaluations","3222","equity","False","Revaluaciones voluntarias"
"uy_code_3311","Legal Reserves","3311","equity","False","Reservas Legales"
"uy_code_3312","Voluntary Reserves","3312","equity","False","Reservas Voluntarias"
"uy_code_3313","Profit and loss","3313","equity","False","Pérdidas y Ganancias"
"uy_code_33201","Results for the year","33201","equity","False","Resultados del ejercicio"
"uy_code_33220","Interim dividends","33220","equity","False","Dividendos provisorios"
"uy_code_4101","Exempt Sales","4100","income","False","Ventas Exentas"
"uy_code_4102","Sales Basic Rate","4101","income","False","Ventas Tasa Básica"
"uy_code_4103","Sales Minimum rate","4102","income","False","Ventas Tasa Mínima"
"uy_code_4104","Export Sales","4103","income","False","Ventas por Exportaciones"
"uy_code_4201","Extraordinary sales","4201","income","False","Ventas extraordinarias"
"uy_code_4301","Interest earned","4301","income","False","Intereses ganados"
"uy_code_4302","Exchange Differences Earned","4302","income","False","Diferencias de Cambio ganadas"
"uy_code_4303","Discounts Obtained","4303","income","False","Descuentos Obtenidos"
"uy_code_5100","Miscellaneous (def)","5100","expense","False","Gastos Varios (def)"
"uy_code_5101","Salaries and Wages","5101","expense","False","Sueldos y Jornales"
"uy_code_5102","Social Charges","5102","expense","False","Cargas Sociales"
"uy_code_5103","Insurance","5103","expense","False","Seguros"
"uy_code_5104","Stationery","5104","expense","False","Papelería"
"uy_code_5105","Fuel","5105","expense","False","Combustible"
"uy_code_5106","Freight","5106","expense","False","Fletes"
"uy_code_5107","Vehicle Maintenance","5107","expense","False","Mantenimiento Vehículos"
"uy_code_5108","Professional Fees","5108","expense","False","Honorarios Profesionales"
"uy_code_5109","Contracted Services","5109","expense","False","Servicios Contratados"
"uy_code_5110","Electric Power and Running Water","5110","expense","False","Energía Eléctrica y Aguas Corrientes"
"uy_code_5111","Communications and Telephone Services","5111","expense","False","Comunicaciones y Servicios Telefónicos"
"uy_code_5112","Rentals","5112","expense","False","Alquileres"
"uy_code_5114","Advertising","5114","expense","False","Publicidad"
"uy_code_5115","Representation","5115","expense","False","Representación"
"uy_code_5203","Contributions","5203","expense","False","Contribuciones"
"uy_code_5204","Withholdings","5204","expense","False","Retenciones"
"uy_code_5205","Others","5205","expense","False","Otros"
"uy_code_5301","Interest and Bank Charges","5301","expense","False","Intereses y Gastos Bancarios"
"uy_code_5302","Exchange Differences lost","5302","expense","False","Diferencias de Cambio perdidas"
"uy_code_5303","Discounts Granted","5303","expense","False","Descuentos Concedidos"
"uy_code_5304","Tax Penalties and Surcharges","5304","expense","False","Multas y Recargos Fiscales"
"uy_code_5401","Cost of Goods","5401","expense","False","Costo de Mercaderías"
"uy_code_5402","Cost of sales of property, plant and equipment","5402","expense","False","Costo de Venta de Bienes de Uso"
"uy_code_5500","Amortizations","5500","expense","False","Amortizaciones"
"uy_code_61","Shares to be issued","61","equity","False","Acciones a Emitir"
"uy_code_62","Stock subscribers","62","equity","False","Suscriptores de acciones"
"uy_code_71","Authorized Capital to be Subscribed","71","equity","False","Capital Autorizado a Suscribir"
"uy_code_72","Subscribed capital","72","equity","False","Capital suscripto"

```

## File: data\template\account.fiscal.position-uy.csv

```csv
"id","name","auto_apply","sequence","country_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"account_fiscal_position_exportation","Exportación","1","10",,"vat1","vat3"
"","","","","","vat2","vat3"
"","","","","","vat4","vat6"
"","","","","","vat5","vat6"
"account_fiscal_position_local_uruguay","Local – Uruguay","1","15","base.uy","",""

```

## File: data\template\account.tax-uy.csv

```csv
id,name,description,invoice_label,l10n_uy_tax_category,amount,amount_type,type_tax_use,tax_group_id,repartition_line_ids/repartition_type,repartition_line_ids/document_type,repartition_line_ids/tag_ids,repartition_line_ids/account_id,price_include,description@es,invoice_label@es
vat1,22%,VAT Sales (22%),VAT Sales (22%),vat,22,percent,sale,tax_group_iva_22,base,invoice,+Base Sales 22%,,,IVA Ventas (22%),IVA Ventas (22%)
,,,,,,,,,tax,invoice,+Sales VAT 22%,uy_code_21402,,,
,,,,,,,,,base,refund,-Taxable Sales Base,,,,
,,,,,,,,,tax,refund,-VAT Sales - received,uy_code_21402,,,
vat2,10%,VAT Sales (10%),VAT Sales (10%),vat,10,percent,sale,tax_group_iva_10,base,invoice,+Base Sales 10%,,,IVA Ventas (10%),IVA Ventas (10%)
,,,,,,,,,tax,invoice,+Sales VAT 10%,uy_code_21401,,,
,,,,,,,,,base,refund,-Taxable Sales Base,,,,
,,,,,,,,,tax,refund,-VAT Sales - received,uy_code_21401,,,
vat3,0% EXEMPT,VAT Exempt Sales,VAT Exempt Sales,vat,0,percent,sale,tax_group_exenton,base,invoice,+Base Sales 0%,,,Ventas Exentos IVA,Ventas Exentos IVA
,,,,,,,,,tax,invoice,,uy_code_21405,,,
,,,,,,,,,base,refund,-Taxable Sales Base,,,,
,,,,,,,,,tax,refund,,uy_code_21405,,,
vat4,22%,VAT Purchases (22%),VAT Purchases (22%),vat,22,percent,purchase,tax_group_iva_22,base,invoice,+Base Purchases 22%,,,IVA Compras (22%),IVA Compras (22%)
,,,,,,,,,tax,invoice,+VAT Purchases 22%,uy_code_11502,,,
,,,,,,,,,base,refund,-Tax Base Purchases,,,,
,,,,,,,,,tax,refund,-VAT Purchases - paid,uy_code_11502,,,
vat5,10%,VAT Purchases (10%),VAT Purchases (10%),vat,10,percent,purchase,tax_group_iva_10,base,invoice,+Base Purchases 10%,,,IVA Compras (10%),IVA Compras (10%)
,,,,,,,,,tax,invoice,+VAT Purchases 10%,uy_code_11501,,,
,,,,,,,,,base,refund,-Tax Base Purchases,,,,
,,,,,,,,,tax,refund,-VAT Purchases - paid,uy_code_11501,,,
vat6,0% EXEMPT,Purchases Exempt from VAT,Purchases Exempt from VAT,vat,0,percent,purchase,tax_group_exenton,base,invoice,+Base Purchases 0%,,,Compras Exento IVA,Compras Exentos IVA
,,,,,,,,,tax,invoice,,uy_code_11503,,,
,,,,,,,,,base,refund,-Tax Base Purchases,,,,
,,,,,,,,,tax,refund,,uy_code_11503,,,
vat7,22% included,VAT Included Sales (22%),VAT Included Sales (22%),vat,22,percent,sale,tax_group_iva_22,base,invoice,+Base Sales 22%,,True,IVA Ventas Incluído (22%),IVA Ventas Incluído (22%)
,,,,,,,,,tax,invoice,+Sales VAT 22%,uy_code_21402,,,
,,,,,,,,,base,refund,-Taxable Sales Base,,,,
,,,,,,,,,tax,refund,-VAT Sales - received,uy_code_21402,,,
vat8,10% included,VAT Included Sales (10%),VAT Included Sales (10%),vat,10,percent,sale,tax_group_iva_10,base,invoice,+Base Sales 10%,,True,IVA Ventas Incluído (10%),IVA Ventas Incluído (10%)
,,,,,,,,,tax,invoice,+Sales VAT 10%,uy_code_21401,,,
,,,,,,,,,base,refund,-Taxable Sales Base,,,,
,,,,,,,,,tax,refund,-VAT Sales - received,uy_code_21401,,,
vat9,22% included,VAT Included Purchases (22%),VAT Included Purchases (22%),vat,22,percent,purchase,tax_group_iva_22,base,invoice,+Base Purchases 22%,,True,IVA Compras Incluído (22%),IVA Compras Incluído (22%)
,,,,,,,,,tax,invoice,+VAT Purchases 22%,uy_code_11502,,,
,,,,,,,,,base,refund,-Tax Base Purchases,,,,
,,,,,,,,,tax,refund,-VAT Purchases - paid,uy_code_11502,,,
vat10,10% included,VAT Included Purchases (10%),VAT Included Purchases (10%),vat,10,percent,purchase,tax_group_iva_10,base,invoice,+Base Purchases 10%,,True,IVA Compras Incluído (10%),IVA Compras Incluído (10%)
,,,,,,,,,tax,invoice,+VAT Purchases 10%,uy_code_11501,,,
,,,,,,,,,base,refund,-Tax Base Purchases,,,,
,,,,,,,,,tax,refund,-VAT Purchases - paid,uy_code_11501,,,

```

## File: data\template\account.tax.group-uy.csv

```csv
"id","name","country_id","name@es"
"tax_group_iva_10","VAT 10%","base.uy","IVA 10%"
"tax_group_iva_22","VAT 22%","base.uy","IVA 22%"
"tax_group_exenton","EXEMPT","base.uy","EXENTOS"

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models

# Let us match the document types to properly suggest the DN and CN documents
# NOTE: this can be avoided if we have an extra subclassification of UY documents
UY_DOC_SUBTYPES = [
    ["0"],  # not electronic
    ["101", "102", "103", "201", "202", "203"],  # e-ticket
    ["111", "112", "113", "211", "212", "213"],  # e-invoice
    ["121", "122", "123", "221", "222", "223"],  # e-inv-expo
    ["151", "152", "153", "251", "252", "253"],  # e-boleta (not implemented yet)
]


class AccountMove(models.Model):

    _inherit = 'account.move'

    def _get_starting_sequence(self):
        """ If use documents then will create a new starting sequence using the document type code prefix and the
        journal document number with a 8 padding number """
        if self.journal_id.l10n_latam_use_documents and self.company_id.account_fiscal_country_id.code == "UY" and self.l10n_latam_document_type_id:
            return self._l10n_uy_get_formatted_sequence()
        return super()._get_starting_sequence()

    def _l10n_uy_get_formatted_sequence(self, number=0):
        return "%s A%07d" % (self.l10n_latam_document_type_id.doc_code_prefix, number)

    def _get_last_sequence_domain(self, relaxed=False):
        where_string, param = super(AccountMove, self)._get_last_sequence_domain(relaxed)
        if self.company_id.account_fiscal_country_id.code == "UY" and self.l10n_latam_use_documents:
            where_string += " AND l10n_latam_document_type_id = %(l10n_latam_document_type_id)s"
            param['l10n_latam_document_type_id'] = self.l10n_latam_document_type_id.id or 0
        return where_string, param

    def _get_l10n_latam_documents_domain(self):
        """ If this is a reversal or debit, suggest only related subtypes """
        self.ensure_one()
        domain = super()._get_l10n_latam_documents_domain()
        if self.country_code == "UY" and (original_move := self.reversed_entry_id or self.debit_origin_id):
            matching_subtype_codes = [
                subtype for subtype in UY_DOC_SUBTYPES
                if original_move.l10n_latam_document_type_id.code in subtype
            ]
            if matching_subtype_codes:
                # restrict to the codes from the subtype matching the one of the original_move (e.g. 'e-ticket')
                codes = self.env["l10n_latam.document.type"].search(domain).mapped('code')
                allowed_codes = set(codes).intersection(set(matching_subtype_codes[0]))
                domain += [("code", "in", tuple(allowed_codes))]
        return domain

```

## File: models\account_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountTax(models.Model):

    _inherit = "account.tax"

    l10n_uy_tax_category = fields.Selection([
        ('vat', 'VAT'),
    ], string="Tax Category", help="UY: Use to group the transactions in the Financial Reports required by DGI")

```

## File: models\l10n_latam_document_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import _, models
from odoo.exceptions import UserError
import re


class L10nAccountDocumentType(models.Model):

    _inherit = 'l10n_latam.document.type'

    def _format_document_number(self, document_number):
        """ format and validate the document_number"""
        self.ensure_one()
        if self.country_id.code != "UY":
            return super()._format_document_number(document_number)

        if not document_number:
            return False

        if self.code == "0":
            return document_number

        document_number = document_number.strip()
        number_part = re.findall(r'[\d]+', document_number)
        serie_part = re.findall(r'^[A-Za-z]+', document_number)
        if not serie_part or len(serie_part) > 1 or len(serie_part[0]) > 2 \
           or not number_part or len(number_part) > 1 or len(number_part[0]) > 7:
            raise UserError(_(
                "%(document_number)s is not a valid value for %(document_type)s.\n"
                "The document number must be entered with a maximum of 2 letters for the first part "
                "and 7 numbers for the second. The following are examples of valid document numbers:\n"
                "- XX0000001\n - YY0000123\n - A0000001",
                document_number=document_number,
                document_type=self.name,
            ))
        return serie_part[0].upper() + number_part[0].zfill(7)

```

## File: models\l10n_latam_identification_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields


class L10nLatamIdentificationType(models.Model):

    _inherit = "l10n_latam.identification.type"

    l10n_uy_dgi_code = fields.Char('DGI Code')

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class ResCompany(models.Model):

    _inherit = 'res.company'

    def _localization_use_documents(self):
        """ Uruguayan localization use documents """
        self.ensure_one()
        return self.account_fiscal_country_id.code == "UY" or super()._localization_use_documents()

```

## File: models\res_partner.py

```python
import logging
import re

from odoo import api, models, _

from odoo.exceptions import ValidationError

_logger = logging.getLogger(__name__)


class ResPartner(models.Model):
    _inherit = "res.partner"

    @api.constrains("vat", "l10n_latam_identification_type_id")
    def check_vat(self):
        # EXTEND account/base_vat
        """ Add validation of UY document types CI and NIE """
        ci_nie_types = self.filtered(
            lambda p: p.l10n_latam_identification_type_id.l10n_uy_dgi_code in ("1", "3")
            and p.l10n_latam_identification_type_id.country_id.code == "UY" and p.vat)
        for partner in ci_nie_types:
            if not partner._l10n_uy_ci_nie_is_valid():
                raise ValidationError(self._l10n_uy_build_vat_error_message(partner))
        return super(ResPartner, self - ci_nie_types).check_vat()

    @api.model
    def _l10n_uy_build_vat_error_message(self, partner):
        """ Similar to _build_vat_error_message but using latam doc type name instead of vat_label
        NOTE: maybe can be implemented in master to l10n_latam_base for the use of different doc types """
        vat_label = _("CI/NIE")
        expected_format = _("3:402.010-2 or 93:402.010-1 (CI or NIE)")

        # Catch use case where the record label is about the public user (name: False)
        if partner.name:
            msg = "\n" + _(
                "The %(vat_label)s number [%(wrong_vat)s] for %(partner_label)s does not seem to be valid."
                "\nNote: the expected format is %(expected_format)s",
                vat_label=vat_label,
                wrong_vat=partner.vat,
                partner_label=_("partner [%s]", partner.name),
                expected_format=expected_format,
            )
        else:
            msg = "\n" + _(
                "The %(vat_label)s number [%(wrong_vat)s] does not seem to be valid."
                "\nNote: the expected format is %(expected_format)s",
                vat_label=vat_label,
                wrong_vat=partner.vat,
                expected_format=expected_format,
            )
        return msg

    def _l10n_uy_ci_nie_is_valid(self):
        """ Check if the partner's CI or NIE number is a valid one.

        CI:
            1) The ID number is taken up to the second to last position, that is, the first 6 or 7 digits.
            2) Each digit is multiplied by a different factor starting from right to left, the factors are:
                2, 9, 8, 7, 6, 3, 4.
            3) The products obtained are added:
            4) The base module 10 is calculated on this result to obtain the check digit, expressed in another way,
            the next number ending in zero is taken that follows the result of the addition (for the example
            would be 60) subtracting the sum itself: 60 - 59 = 1. The verification digit of the example ID is 1.

            NOTE: If the ID has fewer digits, it is preceded with zeros and the mechanism described above is applied

        NIE:
            The calculation for the NIE is the same as that used for the CI. The only difference is that we skip the
            first number

        Both algorithms where extracted from Uruware's Technical Manual (section 9.2 and 9.3)

        Return: False is not valid, True is valid
        """
        self.ensure_one()

        # The VAT must consist only numbers (format could have these characters ":., " we can skip them later)
        invalid_chars = re.findall(r"[^0-9:., \-]", self.vat)
        if invalid_chars:
            return False

        ci_nie_number = re.sub("[^0-9]", "", self.vat)

        # we get the validation digit, if NIE doc type we skip the first digit
        is_nie = self.l10n_latam_identification_type_id.l10n_uy_dgi_code == "1"
        verif_digit = int(ci_nie_number[-1])
        ci_nie_number = ci_nie_number[1:-1] if is_nie else ci_nie_number[0:-1]

        # If number is < 7 digits we add 0 to the left
        ci_nie_number = "%07d" % int(ci_nie_number)

        # If NIE > 7 digits is not valid
        if len(ci_nie_number) > 7:
            return False

        verification_vector = (2, 9, 8, 7, 6, 3, 4)
        num_sum = sum(int(ci_nie_number[i]) * verification_vector[i] for i in range(7))

        res = -num_sum % 10
        return res == verif_digit

```

## File: models\template_uy.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('uy')
    def _get_uy_template_data(self):
        return {
            'property_account_receivable_id': 'uy_code_11300',
            'property_account_payable_id': 'uy_code_21100',
            'property_account_income_categ_id': 'uy_code_4102',
            'property_account_expense_categ_id': 'uy_code_5100',
            'code_digits': '6',
            'name': _('Uruguayan Generic Chart of Accounts'),
        }

    @template('uy', 'res.company')
    def _get_uy_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.uy',
                'bank_account_code_prefix': '1111',
                'cash_account_code_prefix': '1112',
                'transfer_account_code_prefix': '11120',
                'account_default_pos_receivable_account_id': 'uy_code_11307',
                'income_currency_exchange_account_id': 'uy_code_4302',
                'expense_currency_exchange_account_id': 'uy_code_5302',
                'account_journal_early_pay_discount_loss_account_id': 'uy_code_5303',
                'account_journal_early_pay_discount_gain_account_id': 'uy_code_4303',
                'account_sale_tax_id': 'vat1',
                'account_purchase_tax_id': 'vat4',
                'deferred_expense_account_id': 'uy_code_11407',
                'deferred_revenue_account_id': 'uy_code_21321',
            },
        }

    @template('uy', 'account.journal')
    def _get_uy_account_journal(self):
        return {
            'sale': {
                "name": _("Customer Invoices"),
                "code": "0001",
                "l10n_latam_use_documents": True,
                "refund_sequence": False,
            },
            'purchase': {
                "name": _("Vendor Bills"),
                "code": "0002",
                "l10n_latam_use_documents": True,
                "refund_sequence": False,
            },
        }

    def _load(self, template_code, company, install_demo):
        """ Set companies rut as the company identification type  after install the chart of account,
        this one is the uruguayan vat """
        res = super()._load(template_code, company, install_demo)
        if template_code == 'uy':
            company.partner_id.l10n_latam_identification_type_id = self.env.ref('l10n_uy.it_rut')
        return res

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move
from . import account_tax
from . import l10n_latam_identification_type
from . import res_company
from . import res_partner
from . import template_uy
from . import l10n_latam_document_type

```

## File: views\account_tax_views.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>

    <record id="view_tax_form" model="ir.ui.view">
        <field name="name">account.tax.inherit.view.form</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <field name="tax_group_id" position="after">
                <field name="l10n_uy_tax_category" invisible="country_code != 'UY'"/>
            </field>
        </field>
    </record>

</odoo>

```

