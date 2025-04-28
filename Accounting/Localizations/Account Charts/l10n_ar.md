# Odoo Module: l10n_ar

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
from . import demo

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Argentina - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/argentina.html',
    'countries': ['ar'],
    'icon': '/base/static/img/country_flags/ar.png',
    'version': "3.7",
    'description': """
Functional
----------

This module add accounting features for the Argentinean localization, which represent the minimal configuration needed for a company  to operate in Argentina and under the AFIP (Administración Federal de Ingresos Públicos) regulations and guidelines.

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
  * Servicios Globales (IVA Sujeto Exento)
  * Gritti (Monotributo)
  * Montana Sur. IVA Liberado in Zona Franca
  * Barcelona food (Cliente del Exterior)
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

* Argentinean Taxes and Account Tax Groups (VAT taxes with the existing aliquots and other types)
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
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'security/ir.model.access.csv',
        'data/l10n_latam_identification_type_data.xml',
        'data/l10n_ar_afip_responsibility_type_data.xml',
        'data/account_chart_template_data2.xml',
        'data/uom_uom_data.xml',
        'data/l10n_latam.document.type.csv',
        'data/l10n_latam.document.type.xml',
        'data/res_partner_data.xml',
        'data/res.currency.csv',
        'data/res.country.csv',
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
        'views/res_config_settings_view.xml',
        'report/invoice_report_view.xml',
    ],
    'demo': [
        'demo/exento_demo.xml',
        'demo/mono_demo.xml',
        'demo/respinsc_demo.xml',
        'demo/res_partner_demo.xml',
        'demo/product_product_demo.xml',
        'demo/account_customer_invoice_demo.xml',
        'demo/account_customer_refund_demo.xml',
        'demo/account_supplier_invoice_demo.xml',
        'demo/account_supplier_refund_demo.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\account_chart_template_data2.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_ar_statements_menu" name="Argentinean Statements" parent="account.menu_finance_reports" sequence="5" groups="account.group_account_readonly"/>
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
        <field name='name'>(Depreciado) IVA Responsable no Inscripto</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVANR'>
        <field name='code'>3</field>
        <field name='name'>(Depreciado) IVA no Responsable</field>
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
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_EXT_Prov'>
        <field name='code'>8</field>
        <field name='name'>Proveedor del Exterior</field>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_EXT'>
        <field name='code'>9</field>
        <field name='name'>Cliente del Exterior</field>
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVA_LIB'>
        <field name='code'>10</field>
        <field name='name'>IVA Liberado – Ley Nº 19.640</field>
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVARI_AP'>
        <field name='code'>11</field>
        <field name='name'>(Depreciado) IVA Responsable Inscripto – Agente de Percepción</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_EVENTUAL'>
        <field name='code'>12</field>
        <field name='name'>(Depreciado) Pequeño Contribuyente Eventual</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_MON_SOCIAL'>
        <field name='code'>13</field>
        <field name='name'>Monotributista Social</field>
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_EVENTUAL_SOCIAL'>
        <field name='code'>14</field>
        <field name='name'>(Depreciado) Pequeño Contribuyente Eventual Social</field>
        <field name='active' eval="False"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_IVA_NO_ALC'>
        <field name='code'>15</field>
        <field name='name'>IVA No Alcanzado</field>
        <field name='active' eval="True"/>
    </record>
    <record model='l10n_ar.afip.responsibility.type' id='res_MON_Ind'>
        <field name='code'>16</field>
        <field name='name'>Monotributo Trabajador / Independiente Promovido</field>
    </record>
</odoo>

```

## File: data\l10n_latam.document.type.csv

```csv
id,sequence,code,name,l10n_ar_letter,report_name,internal_type,doc_code_prefix,country_id/id,purchase_aliquots
dc_a_f,10,1,INVOICES A,A,INVOICE,invoice,FA-A,base.ar,not_zero
dc_a_nd,20,2,DEBIT MEMOS A,A,DEBIT NOTE,debit_note,ND-A,base.ar,not_zero
dc_a_nc,30,3,CREDIT NOTES A,A,CREDIT NOTE,credit_note,NC-A,base.ar,not_zero
dc_a_r,40,4,RECEIPT A,A,RECEIPT,invoice,RE-A,base.ar,not_zero
dc_a_nvc,50,5,CASH SALES NOTES A,A,,invoice,NVC-A,base.ar,not_zero
dc_b_f,60,6,INVOICES B,B,INVOICE,invoice,FA-B,base.ar,zero
dc_b_nd,70,7,DEBIT MEMOS B,B,DEBIT NOTE,debit_note,ND-B,base.ar,zero
dc_b_nc,80,8,CREDIT NOTES B,B,CREDIT NOTE,credit_note,NC-B,base.ar,zero
dc_b_r,90,9,RECEIPT B,B,RECEIPT,invoice,RE-B,base.ar,zero
dc_b_nvc,100,10,CASH SALES NOTES B,B,,invoice,NVC-B,base.ar,zero
dc_c_f,110,11,INVOICES C,C,INVOICE,invoice,FA-C,base.ar,zero
dc_c_nd,120,12,DEBIT NOTES C,C,DEBIT NOTE,debit_note,ND-C,base.ar,zero
dc_c_nc,130,13,CREDIT NOTES C,C,CREDIT NOTE,credit_note,NC-C,base.ar,zero
dc_aduana,140,14,CUSTOMS DOCUMENT,,,,,base.ar,
dc_c_r,150,15,RECEIPT C,C,RECEIPT,invoice,RE-C,base.ar,zero
dc_c_nvc,160,16,CASH SALES NOTES C,C,,invoice,NVC-C,base.ar,zero
dc_e_f,170,19,EXPORT INVOICES,E,INVOICE,invoice,FA-E,base.ar,not_zero
dc_e_nd,180,20,DEBIT NOTES FOR FOREIGN OPERATIONS,E,DEBIT NOTE,debit_note,ND-E,base.ar,not_zero
dc_e_nc,190,21,CREDIT NOTES FOR FOREIGN OPERATIONS,E,CREDIT NOTE,credit_note,NC-E,base.ar,not_zero
dc_e_fs,200,22,INVOICES - SIMPLIFIED EXPORT PERMIT - DTO. 855/97,E,,,,base.ar,not_zero
dc_usados,210,30,PROOF OF PURCHASE OF USED GOODS,,,invoice,CBU,base.ar,not_zero
dc_mandato,220,31,MANDATE - CONSIGNMENT,,,,,base.ar,
dc_reciclado,230,32,RECEIPTS FOR RECYCLING MATERIALS,,,invoice,CRM,base.ar,not_zero
dc_a_rg1415,240,34,"VOUCHERS A OF SECTION A, INC. F), G.R. Nº 1415",A,,invoice,CA-A,base.ar,not_zero
dc_b_rg1415,250,35,"VOUCHERS B OF ANNEX I, SECTION A, INC. F), G.R. Nº 1415",B,,invoice,CA-B,base.ar,zero
dc_c_rg1415,260,36,"VOUCHERS C OF ANNEX I, SECTION A, INC. F), G.R. Nº 1415",C,,invoice,CA-C,base.ar,zero
dc_nd_rg1415,270,37,DEBIT NOTES OR EQUIVALENT DOCUMENT COMPLYING WITH G.R. NO. 1415,,,debit_note,ND1415,base.ar,not_zero
dc_nc_rg1415,280,38,CREDIT NOTES OR EQUIVALENT DOCUMENT COMPLYING WITH G.R. NO. 1415,,,credit_note,NC1415,base.ar,not_zero
dc_a_o_rg1415,290,39,OTHER COMPROBANTES A QUE CUMPLEN CON LA R.G. Nº 1415,A,,invoice,OC-A,base.ar,not_zero
dc_b_o_rg1415,300,40,OTHER COMPROBANTES B QUE CUMPLAN CON LA R.G. Nº 1415,B,,invoice,OC-B,base.ar,zero
dc_c_o_rg1415,310,41,OTHER COMPROBANTES C QUE CUMPLAN CON LA R.G. Nº 1415,C,,invoice,OC-C,base.ar,zero
dc_a_rf,320,50,RECEIPT OF INVOICE UNDER THE CREDIT INVOICE SYSTEM,A,,,,base.ar,not_zero
dc_m_f,330,51,INVOICES M,M,INVOICE,invoice,FA-M,base.ar,not_zero
dc_m_nd,340,52,DEBIT MEMOS M,M,DEBIT NOTE,debit_note,ND-M,base.ar,not_zero
dc_m_nc,350,53,CREDIT NOTES M,M,CREDIT NOTE,credit_note,NC-M,base.ar,not_zero
dc_m_r,360,54,RECEIPTS M,M,RECEIPT,invoice,RE-M,base.ar,not_zero
dc_m_nvc,370,55,CASH SALES NOTES M,M,,invoice,NVC-M,base.ar,not_zero
dc_m_rg1415,380,56,"VOUCHERS M OF ANNEX I, SECTION A, INC. F), G.R. NO. 1415",M,,invoice,CA-M,base.ar,not_zero
dc_m_o_rg1415,390,57,OTHER COMPROBANTES M QUE CUMPLAN CON LA R.G. Nº 1415,M,,invoice,OC-M,base.ar,not_zero
dc_m_cvl,400,58,SALES ACCOUNTS AND LIQUID PRODUCT M,M,,invoice,LP-M,base.ar,not_zero
dc_m_l,410,59,LIQUIDATIONS M,M,LIQUIDATION,invoice,LI-M,base.ar,not_zero
dc_a_cvl,420,60,SALES ACCOUNTS AND LIQUID PRODUCT A,A,LIQUID PRODUCT SALES ACCOUNT,invoice,LP-A,base.ar,not_zero
dc_b_cvl,430,61,SALES ACCOUNTS AND LIQUID PRODUCT B,B,LIQUID PRODUCT SALES ACCOUNT,invoice,LP-B,base.ar,zero
dc_a_l,440,63,LIQUIDATIONS A,A,LIQUIDATION,invoice,LI-A,base.ar,not_zero
dc_b_l,450,64,LIQUIDATIONS B,B,LIQUIDATION,invoice,LI-B,base.ar,zero
dc_nc,460,65,"PROOF CREDIT NOTES WITH CODE. 34, 39, 58, 59, 60, 63, 96, 97,",,,,,base.ar,
dc_desp_imp,470,66,IMPORT CLEARANCE,,,invoice,DI,base.ar,not_zero
dc_imp_serv,480,67,IMPORT OF SERVICES,,,,,base.ar,
dc_c_l,490,68,LIQUIDATION C,C,LIQUIDATION,invoice,LI-C,base.ar,not_zero
dc_rfc,500,70,CREDIT INVOICE RECEIPTS,,,,,base.ar,not_zero
dc_cfcp,510,71,TAX CREDIT FOR EMPLOYER CONTRIBUTIONS,,,,,base.ar,
dc_f1116,520,73,FORM 1116 RT,,,,,base.ar,
dc_cptag,530,74,WAYBILL FOR MOTOR TRANSPORT FOR GRAINS,,,,,base.ar,
dc_cptfg,540,75,WAYBILL FOR RAIL TRANSPORT FOR GRAINS,,,,,base.ar,
dc_zeta,550,80,DAILY CLOSING REPORT (ZETA) - FISCAL CONTROLLERS,,ZETA,invoice,CI-Z,base.ar,zero
dc_a_t,560,81,TICKET INVOICE A,A,INVOICE TICKET,invoice,TF-A,base.ar,not_zero
dc_b_t,570,82,TICKET - INVOICE B,B,INVOICE TICKET,invoice,TF-B,base.ar,zero
dc_t,580,83,TICKET,,TICKET,invoice,TI-X,base.ar,zero
dc_sp_c,590,84,UTILITY BILL VOUCHER FINANCIAL INTEREST,,,,,base.ar,
dc_sp_nc,600,85,CREDIT NOTE PUBLIC SERVICES/TAX CONTROLLERS,,,,,base.ar,
dc_sp_nd,610,86,UTILITY DEBIT MEMO,,,,,base.ar,
dc_oc_se,620,87,OTHER VOUCHERS - FOREIGN SERVICES,,,,,base.ar,
dc_oc_c,630,88,ELECTRONIC MAILING,,,,,base.ar,
dc_oc_nd,640,89,DATA OVERVIEW,,,,,base.ar,
dc_oc_nc,650,90,OTHER VOUCHERS - EXCEPTED DOCUMENTS - CREDIT NOTES,,,credit_note,OC,base.ar,not_zero
dc_r_r,660,91,MAILING R,R,,,,base.ar,
dc_ac_inc_df,670,92,ACCOUNTING ADJUSTMENTS THAT INCREASE TAX LIABILITIES,,,,,base.ar,
dc_ac_dis_df,680,93,ACCOUNTING ADJUSTMENTS THAT REDUCE TAX LIABILITIES,,,,,base.ar,
dc_ac_inc_cf,690,94,ACCOUNTING ADJUSTMENTS THAT INCREASE TAX CREDITS,,,,,base.ar,
dc_ac_dis_cf,700,95,ACCOUNTING ADJUSTMENTS THAT DECREASE THE TAX CREDIT,,,,,base.ar,
dc_f1116b,710,96,FORM 1116 B,,,,,base.ar,
dc_f1116c,720,97,FORM 1116 C,,,,,base.ar,
dc_oc_ncrg3419,730,99,OTHER VOUCHERS THAT DO NOT COMPLY WITH OR ARE EXEMPT FROM G.R. NO. 1415 AND AMENDMENTS THERETO,,,invoice,OC-X,base.ar,not_zero
dc_aa_dj_pos,740,101,ANNUAL ADJUSTMENT ARISING FROM THE POSITIVE VAT D J,,,,,base.ar,
dc_aa_dj_neg,750,102,ANNUAL ADJUSTMENT ARISING FROM THE NEGATIVE VAT D J,,,,,base.ar,
dc_na,760,103,ASSIGNMENT NOTE,,,,,base.ar,
dc_nca,770,104,ASSIGNMENT CREDIT NOTE,,,,,base.ar,
dc_remito_x,790,94,MAILING X,X,MAILING,,RM-X,base.ar,
dc_liq_s_a,800,17,LIQUIDATION OF CLASS A UTILITIES,A,,invoice,LS-A,base.ar,not_zero
dc_liq_s_b,810,18,LIQUIDATION OF CLASS B UTILITIES,B,,invoice,LS-B,base.ar,zero
dc_com_a_m,820,23,"PRIMARY PURCHASE ""A"" VOUCHERS FOR THE MARINE FISHING SECTOR",,,,,base.ar,
dc_con_a_m,830,24,"PRIMARY CONSIGNMENT ""A"" VOUCHERS FOR THE MARINE FISHING SECTOR",,,,,base.ar,
dc_com_b_m,840,25,"PRIMARY PURCHASE ""B"" VOUCHERS FOR THE MARINE FISHING SECTOR",,,,,base.ar,
dc_con_b_m,850,26,"PRIMARY CONSIGNMENT ""B"" VOUCHERS FOR THE MARINE FISHING SECTOR",,,,,base.ar,
dc_liq_uci_a,860,27,CLASS A SINGLE COMMERCIAL TAX SETTLEMENT,A,,invoice,LU-A,base.ar,not_zero
dc_liq_uci_b,870,28,CLASS B SINGLE COMMERCIAL TAX SETTLEMENT,B,,invoice,LU-B,base.ar,zero
dc_liq_uci_c,880,29,CLASS C SINGLE COMMERCIAL TAX SETTLEMENT,C,,invoice,LU-C,base.ar,zero
dc_liq_prim_gr,890,33,PRIMARY GRAIN SETTLEMENT,,,invoice,LPG,base.ar,not_zero
dc_nc_liq_uci_a,900,43,CREDIT NOTE SINGLE COMMERCIAL TAX SETTLEMENT CLASS B,B,,credit_note,NCLU-B,base.ar,zero
dc_nc_liq_uci_b,910,44,CREDIT NOTE SINGLE COMMERCIAL TAX SETTLEMENT CLASS C,C,,credit_note,NCLU-C,base.ar,zero
dc_nd_liq_uci_a,920,45,DEBIT NOTE SINGLE COMMERCIAL TAX SETTLEMENT CLASS A,A,,debit_note,NDLU-A,base.ar,not_zero
dc_nd_liq_uci_b,930,46,DEBIT NOTE SINGLE COMMERCIAL TAX SETTLEMENT CLASS B,B,,debit_note,NDLU-B,base.ar,zero
dc_nd_liq_uci_c,940,47,DEBIT NOTE SINGLE COMMERCIAL TAX SETTLEMENT CLASS C,C,,debit_note,NDLU-C,base.ar,zero
dc_nc_liq_uci_c,950,48,CREDIT NOTE SINGLE COMMERCIAL TAX SETTLEMENT CLASS A,A,,credit_note,NCLU-A,base.ar,not_zero
dc_bs_no_reg,960,49,PROOFS OF PURCHASE OF NON-REGISTRABLE GOODS TO FINAL CONSUMERS,,,invoice,BNR,base.ar,not_zero
dc_t_nc,970,110,CREDIT NOTE TICKET,,CREDIT NOTE TICKET,credit_note,TC-X,base.ar,not_zero
dc_t_c,980,111,TICKET INVOICE C,C,INVOICE TICKET,invoice,TF-C,base.ar,zero
dc_t_nc_a,990,112,CREDIT NOTE TICKET A,A,CREDIT NOTE TICKET,credit_note,TN-A,base.ar,not_zero
dc_t_nc_b,1000,113,CREDIT NOTE TICKET B,B,CREDIT NOTE TICKET,credit_note,TN-B,base.ar,zero
dc_t_nc_c,1010,114,CREDIT NOTE TICKET C,C,CREDIT NOTE TICKET,credit_note,TN-C,base.ar,zero
dc_t_nd_a,1020,115,DEBIT NOTE TICKET A,A,DEBIT NOTE TICKET,debit_note,TD-A,base.ar,not_zero
dc_t_nd_b,1030,116,DEBIT NOTE TICKET B,B,DEBIT NOTE TICKET,debit_note,TD-B,base.ar,zero
dc_t_nd_c,1040,117,DEBIT NOTE TICKET C,C,DEBIT NOTE TICKET,debit_note,TD-C,base.ar,zero
dc_t_m,1050,118,TICKET INVOICE M,M,INVOICE,invoice,TF-M,base.ar,not_zero
dc_t_nc_m,1060,119,CREDIT NOTE TICKET M,M,CREDIT NOTE,credit_note,TC-M,base.ar,not_zero
dc_t_nd_m,1070,120,DEBIT NOTE TICKET M,M,DEBIT NOTE,debit_note,TD-M,base.ar,not_zero
dc_liq_sec_gr,1080,331,SECONDARY GRAIN LIQUIDATION,,,invoice,LSG,base.ar,not_zero
dc_cert_ele_gr,1090,332,ELECTRONIC CERTIFICATION (GRAINS),,,,,base.ar,
dc_fce_a_f,1100,201,ELECTRONIC CREDIT INVOICE FOR SMBs (FCE) TO,A,ELECTRONIC CREDIT INVOICE,invoice,FCE-A,base.ar,not_zero
dc_fce_a_nd,1110,202,ELECTRONIC DEBIT NOTE SMMEs (FCE) A,A,ELECTRONIC DEBIT MEMO,debit_note,NDE-A,base.ar,not_zero
dc_fce_a_nc,1120,203,ELECTRONIC CREDIT NOTE SME SMEs (FCE) A,A,ELECTRONIC CREDIT NOTE,credit_note,NCE-A,base.ar,not_zero
dc_fce_b_f,1130,206,ELECTRONIC CREDIT INVOICE FOR SMBs (ECF) B,B,ELECTRONIC CREDIT INVOICE,invoice,FCE-B,base.ar,zero
dc_fce_b_nd,1140,207,ELECTRONIC DEBIT NOTE SME's (FCE) B,B,ELECTRONIC DEBIT MEMO,debit_note,NDE-B,base.ar,zero
dc_fce_b_nc,1150,208,ELECTRONIC CREDIT NOTICE FOR SMBs (FCE) B,B,ELECTRONIC CREDIT NOTE,credit_note,NCE-B,base.ar,zero
dc_fce_c_f,1160,211,ELECTRONIC CREDIT INVOICE FOR SMBs (FCE) C,C,ELECTRONIC CREDIT INVOICE,invoice,FCE-C,base.ar,zero
dc_fce_c_nd,1170,212,ELECTRONIC DEBIT NOTE SME's (FCE) C,C,ELECTRONIC DEBIT MEMO,debit_note,NDE-C,base.ar,zero
dc_fce_c_nc,1180,213,ELECTRONIC CREDIT NOTE SME's (FCE) C,C,ELECTRONIC CREDIT NOTE,credit_note,NCE-D,base.ar,zero
fa_exterior,195,,INVOICES AND RECEIPTS FROM ABROAD,I,,invoice,FA-I,base.ar,zero
nc_exterior,196,,FOREIGN CREDIT NOTES AND REIMBURSEMENTS,I,,credit_note,NC-I,base.ar,zero
dc_liq_cpst_a,1190,150,LIQUIDATION OF PRIMARY PURCHASE FOR THE TOBACCO SECTOR A,A,,invoice,LCT-A,base.ar,not_zero
dc_liq_cpst_b,1200,151,PRIMARY PURCHASE LIQUIDATION FOR THE TOBACCO SECTOR B,B,,invoice,LCT-B,base.ar,zero
dc_cvl_sa_a,1210,157,SALES AND CASH ACCOUNT PRODUCT A - POULTRY SECTOR,A,,invoice,CVA-A,base.ar,not_zero
dc_cvl_sa_b,1220,158,SALES AND CASH ACCOUNT PRODUCT B - POULTRY SECTOR,B,,invoice,CVA-B,base.ar,zero
dc_liq_c_sa_a,1230,159,PURCHASE LIQUIDATION A - POULTRY SECTOR,A,,invoice,LCA-A,base.ar,not_zero
dc_liq_c_sa_b,1240,160,PURCHASE LIQUIDATION B - POULTRY SECTOR,B,,invoice,LCA-B,base.ar,zero
dc_liq_cd_sa_a,1250,161,DIRECT PURCHASE LIQUIDATION A - POULTRY SECTOR,A,,invoice,LCDA-A,base.ar,not_zero
dc_liq_cd_sa_b,1260,162,DIRECT PURCHASE LIQUIDATION B - POULTRY SECTOR,B,,invoice,LCDA-B,base.ar,zero
dc_liq_cd_sa_c,1270,163,DIRECT PURCHASE LIQUIDATION C - POULTRY SECTOR,C,,invoice,LCDA-C,base.ar,zero
dc_liq_vd_sa_a,1280,164,LIQUIDATION OF DIRECT SALES A - POULTRY SECTOR,A,,invoice,LVDA-A,base.ar,not_zero
dc_liq_vd_sa_b,1290,165,LIQUIDATION OF DIRECT SALES B - POULTRY SECTOR,B,,invoice,LVDA-B,base.ar,zero
dc_liq_ccpc_a,1300,166,LIQUIDATION OF HIRING OF BREEDING GRILL CHICKENS A,A,,invoice,LCCP-A,base.ar,not_zero
dc_liq_ccpc_b,1310,167,LIQUIDATION OF HIRING OF BREEDING GRILL CHICKENS B,B,,invoice,LCCP-B,base.ar,zero
dc_liq_ccpc_c,1320,168,LIQUIDATION OF HIRING OF BREEDING GRILL CHICKENS C,C,,invoice,LCCP-C,base.ar,zero
dc_liq_cpc_a,1330,169,LIQUIDATION OF BREEDING BROILER CHICKENS A,A,,invoice,LCPP-A,base.ar,not_zero
dc_liq_cpc_b,1340,170,LIQUIDATION OF BREEDING BROILER CHICKENS B,B,,invoice,LCPP-B,base.ar,zero
dc_liq_cca_a,1350,171,SUGAR CANE PURCHASE LIQUIDATION A,A,,invoice,LCCA-A,base.ar,not_zero
dc_liq_cca_b,1360,172,SUGAR CANE PURCHASE LIQUIDATION A,B,,invoice,LCCA-B,base.ar,zero
dc_cvl_sp_a,1370,180,SALES AND CASH ACCOUNT PRODUCT A - LIVESTOCK SECTOR,A,,invoice,CVP-A,base.ar,not_zero
dc_cvl_sp_b,1380,182,SALES AND CASH ACCOUNT PRODUCT B - LIVESTOCK SECTOR,B,,invoice,CVP-B,base.ar,zero
dc_liq_c_sp_a,1390,183,PURCHASE LIQUIDATION A - LIVESTOCK SECTOR,A,,invoice,LCP-A,base.ar,not_zero
dc_liq_c_sp_b,1400,185,PURCHASE LIQUIDATION B - LIVESTOCK SECTOR,B,,invoice,LCP-B,base.ar,zero
dc_liq_cd_sp_a,1410,186,DIRECT PURCHASE LIQUIDATION A - LIVESTOCK SECTOR,A,,invoice,LCDP-A,base.ar,not_zero
dc_liq_cd_sp_b,1420,188,DIRECT PURCHASE LIQUIDATION B - LIVESTOCK SECTOR,B,,invoice,LCDP-B,base.ar,zero
dc_liq_cd_sp_c,1430,189,DIRECT PURCHASE LIQUIDATION C - LIVESTOCK SECTOR,C,,invoice,LCDP-C,base.ar,zero
dc_liq_vd_sp_a,1440,190,LIQUIDATION OF DIRECT SALE A - LIVESTOCK SECTOR,A,,invoice,LVDP-A,base.ar,not_zero
dc_liq_vd_sp_b,1450,191,LIQUIDATION OF DIRECT SALES B - LIVESTOCK SECTOR,B,,invoice,LVDP-B,base.ar,zero

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
        <field name='description'>Unique Tax Identification Code</field>
        <field name='country_id' ref='base.ar'/>
        <field name='is_vat' eval='True'/>
        <field name='l10n_ar_afip_code'>80</field>
        <field name='sequence'>10</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_dni'>
        <field name='name'>DNI</field>
        <field name='description'>National Identity Card</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>96</field>
        <field name='sequence'>20</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CUIL'>
        <field name='name'>CUIL</field>
        <field name='description'>Unique Labor Identification Code</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>86</field>
        <field name='sequence'>30</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_Sigd'>
        <field name='name'>Sigd</field>
        <field name='description'>Unidentified/global daily sales</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>99</field>
        <field name='sequence'>110</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_CPF'>
        <field name='name'>CPF</field>
        <field name='description'>CI Federal Police</field>
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
        <field name='description'>pending</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>92</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_AN'>
        <field name='name'>AN</field>
        <field name='description'>Birth certificate</field>
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
        <field name='description'>Migration Certificate</field>
        <field name='country_id' ref='base.ar'/>
        <field name='l10n_ar_afip_code'>30</field>
    </record>
    <record model='l10n_latam.identification.type' id='it_UpApP'>
        <field name='name'>UpApP</field>
        <field name='description'>Used by Anses for Padrón</field>
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
base.EGP,046
base.GBP,021
base.GTQ,055
base.HKD,051
base.HNL,063
base.HUF,056
base.ILS,030
base.INR,062
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

## File: data\template\account.account-ar_base.csv

```csv
"id","code","account_type","name","reconcile","name@es"
"base_cheques_de_terceros","1.1.1.02.010","asset_cash","Third Party Checks","False","Cheques de Terceros"
"base_cheques_de_terceros_rechazados","1.1.1.02.020","asset_current","Rejected Third Party Checks","False","Cheques de Terceros Rechazados"
"base_fondo_comun_de_inversion","1.1.2.01.010","asset_current","Mutual fund","False","Fondo común de inversión"
"base_plazo_fijo","1.1.2.01.020","asset_current","Fixed term","False","Plazo fijo"
"base_deudores_por_ventas","1.1.3.01.010","asset_receivable","Sales receivables","True","Créditos por ventas"
"base_deudores_por_ventas_pos","1.1.3.01.020","asset_receivable","Sales receivables (PoS)","True","Créditos por ventas (PoS)"
"base_ret_percepcion_tasa_municipal","1.1.4.01.010","asset_current","Ret/Perception Municipal Tax","False","Ret/Percepción Tasa Municipal"
"base_saldo_a_favor_tasa_municipal","1.1.4.01.020","asset_current","Balance in favor Municipal Tax","False","Saldo a favor Tasa Municipal"
"base_saldo_favor_iibb_caba","1.1.4.02.010","asset_current","Balance in favor IIBB CABA","False","Saldo a favor IIBB CABA"
"base_retencion_iibb_caba_sufrida","1.1.4.02.020","asset_current","IIBB CABA withholding incurred","False","Retención IIBB CABA sufrida"
"base_percepcion_iibb_caba_sufrida","1.1.4.02.030","asset_current","Perception of IIBB CABA incurred","False","Percepción IIBB CABA sufrida"
"base_saldo_favor_iibb_ba","1.1.4.02.040","asset_current","Balance in favor IIBB Buenos Aires","False","Saldo a favor IIBB Buenos Aires"
"base_retencion_iibb_ba_sufrida","1.1.4.02.050","asset_current","IIBB Buenos Aires withholding tax incurred","False","Retención IIBB Buenos Aires sufrida"
"base_percepcion_iibb_ba_sufrida","1.1.4.02.060","asset_current","Perception IIBB Buenos Aires incurred","False","Percepción IIBB Buenos Aires sufrida"
"base_saldo_favor_iibb_ca","1.1.4.02.070","asset_current","Balance in favor IIBB Catamarca","False","Saldo a favor IIBB Catamarca"
"base_retencion_iibb_ca_sufrida","1.1.4.02.080","asset_current","IIBB withholding Catamarca incurred","False","Retención IIBB Catamarca sufrida"
"base_percepcion_iibb_ca_sufrida","1.1.4.02.090","asset_current","Perception IIBB Catamarca incurred","False","Percepción IIBB Catamarca sufrida"
"base_saldo_favor_iibb_co","1.1.4.02.100","asset_current","Balance in favor IIBB Córdoba","False","Saldo a favor IIBB Córdoba"
"base_retencion_iibb_co_sufrida","1.1.4.02.110","asset_current","IIBB Córdoba withholding tax incurred","False","Retención IIBB Córdoba sufrida"
"base_percepcion_iibb_co_sufrida","1.1.4.02.120","asset_current","Perception IIBB Córdoba incurred","False","Percepción IIBB Córdoba sufrida"
"base_saldo_favor_iibb_rr","1.1.4.02.130","asset_current","Balance in favor IIBB Corrientes","False","Saldo a favor IIBB Corrientes"
"base_retencion_iibb_rr_sufrida","1.1.4.02.140","asset_current","IIBB Corrientes withholding incurred","False","Retención IIBB Corrientes sufrida"
"base_percepcion_iibb_rr_sufrida","1.1.4.02.150","asset_current","Perception IIBB Corrientes incurred","False","Percepción IIBB Corrientes sufrida"
"base_saldo_favor_iibb_er","1.1.4.02.160","asset_current","Balance in favor IIBB Entre Ríos","False","Saldo a favor IIBB Entre Ríos"
"base_retencion_iibb_er_sufrida","1.1.4.02.170","asset_current","IIBB withholding Entre Ríos incurred","False","Retención IIBB Entre Ríos sufrida"
"base_percepcion_iibb_er_sufrida","1.1.4.02.180","asset_current","Perception IIBB Entre Ríos incurred","False","Percepción IIBB Entre Ríos sufrida"
"base_saldo_favor_iibb_ju","1.1.4.02.190","asset_current","Balance in favor IIBB Jujuy","False","Saldo a favor IIBB Jujuy"
"base_retencion_iibb_ju_sufrida","1.1.4.02.200","asset_current","IIBB Jujuy withholding incurred","False","Retención IIBB Jujuy sufrida"
"base_percepcion_iibb_ju_sufrida","1.1.4.02.210","asset_current","Perception IIBB Jujuy incurred","False","Percepción IIBB Jujuy sufrida"
"base_saldo_favor_iibb_za","1.1.4.02.220","asset_current","Balance in favor IIBB Mendoza","False","Saldo a favor IIBB Mendoza"
"base_retencion_iibb_za_sufrida","1.1.4.02.230","asset_current","IIBB Mendoza withholding incurred","False","Retención IIBB Mendoza sufrida"
"base_percepcion_iibb_za_sufrida","1.1.4.02.240","asset_current","Perception IIBB Mendoza incurred","False","Percepción IIBB Mendoza sufrida"
"base_saldo_favor_iibb_lr","1.1.4.02.250","asset_current","Balance in favor IIBB La Rioja","False","Saldo a favor IIBB La Rioja"
"base_retencion_iibb_lr_sufrida","1.1.4.02.260","asset_current","IIBB withholding La Rioja incurred","False","Retención IIBB La Rioja sufrida"
"base_percepcion_iibb_lr_sufrida","1.1.4.02.270","asset_current","Perception IIBB La Rioja incurred","False","Percepción IIBB La Rioja sufrida"
"base_saldo_favor_iibb_sa","1.1.4.02.280","asset_current","Balance in favor IIBB Salta","False","Saldo a favor IIBB Salta"
"base_retencion_iibb_sa_sufrida","1.1.4.02.290","asset_current","IIBB withholding IIBB Salta incurred","False","Retención IIBB Salta sufrida"
"base_percepcion_iibb_sa_sufrida","1.1.4.02.300","asset_current","Perception IIBB Salta incurred","False","Percepción IIBB Salta sufrida"
"base_saldo_favor_iibb_nn","1.1.4.02.310","asset_current","Balance in favor IIBB San Juan","False","Saldo a favor IIBB San Juan"
"base_retencion_iibb_nn_sufrida","1.1.4.02.320","asset_current","IIBB withholding San Juan incurred","False","Retención IIBB San Juan sufrida"
"base_percepcion_iibb_nn_sufrida","1.1.4.02.330","asset_current","Perception IIBB San Juan incurred","False","Percepción IIBB San Juan sufrida"
"base_saldo_favor_iibb_sl","1.1.4.02.340","asset_current","Balance in favor IIBB San Luis","False","Saldo a favor IIBB San Luis"
"base_retencion_iibb_sl_sufrida","1.1.4.02.350","asset_current","IIBB withholding San Luis incurred","False","Retención IIBB San Luis sufrida"
"base_percepcion_iibb_sl_sufrida","1.1.4.02.360","asset_current","Perception IIBB San Luis incurred","False","Percepción IIBB San Luis sufrida"
"base_saldo_favor_iibb_sf","1.1.4.02.370","asset_current","Balance in favor IIBB Santa Fe","False","Saldo a favor IIBB Santa Fe"
"base_retencion_iibb_sf_sufrida","1.1.4.02.380","asset_current","Withholding IIBB Santa Fe incurred","False","Retención IIBB Santa Fe sufrida"
"base_percepcion_iibb_sf_sufrida","1.1.4.02.390","asset_current","Perception IIBB Santa Fe incurred","False","Percepción IIBB Santa Fe sufrida"
"base_saldo_favor_iibb_se","1.1.4.02.400","asset_current","Balance in favor IIBB Santiago del Estero","False","Saldo a favor IIBB Santiago del Estero"
"base_retencion_iibb_se_sufrida","1.1.4.02.410","asset_current","IIBB Santiago del Estero withholding incurred","False","Retención IIBB Santiago del Estero sufrida"
"base_percepcion_iibb_se_sufrida","1.1.4.02.420","asset_current","Perception IIBB Santiago del Estero incurred","False","Percepción IIBB Santiago del Estero sufrida"
"base_saldo_favor_iibb_tn","1.1.4.02.430","asset_current","Balance in favor IIBB Tucumán","False","Saldo a favor IIBB Tucumán"
"base_retencion_iibb_tn_sufrida","1.1.4.02.440","asset_current","IIBB withholding Tucumán incurred","False","Retención IIBB Tucumán sufrida"
"base_percepcion_iibb_tn_sufrida","1.1.4.02.450","asset_current","Perception IIBB Tucumán incurred","False","Percepción IIBB Tucumán sufrida"
"base_saldo_favor_iibb_ha","1.1.4.02.460","asset_current","Balance in favor IIBB Chaco","False","Saldo a favor IIBB Chaco"
"base_retencion_iibb_ha_sufrida","1.1.4.02.470","asset_current","IIBB Chaco withholding incurred","False","Retención IIBB Chaco sufrida"
"base_percepcion_iibb_ha_sufrida","1.1.4.02.480","asset_current","Perception IIBB Chaco incurred","False","Percepción IIBB Chaco sufrida"
"base_saldo_favor_iibb_ct","1.1.4.02.490","asset_current","Balance in favor IIBB Chubut","False","Saldo a favor IIBB Chubut"
"base_retencion_iibb_ct_sufrida","1.1.4.02.500","asset_current","IIBB Chubut withholding incurred","False","Retención IIBB Chubut sufrida"
"base_percepcion_iibb_ct_sufrida","1.1.4.02.510","asset_current","Perception IIBB Chubut incurred","False","Percepción IIBB Chubut sufrida"
"base_saldo_favor_iibb_fo","1.1.4.02.520","asset_current","Balance in favor IIBB Formosa","False","Saldo a favor IIBB Formosa"
"base_retencion_iibb_fo_sufrida","1.1.4.02.530","asset_current","IIBB Formosa withholding incurred","False","Retención IIBB Formosa sufrida"
"base_percepcion_iibb_fo_sufrida","1.1.4.02.540","asset_current","Perception IIBB Formosa incurred","False","Percepción IIBB Formosa sufrida"
"base_saldo_favor_iibb_mi","1.1.4.02.550","asset_current","Balance in favor IIBB Misiones","False","Saldo a favor IIBB Misiones"
"base_retencion_iibb_mi_sufrida","1.1.4.02.560","asset_current","IIBB Misiones withholding incurred","False","Retención IIBB Misiones sufrida"
"base_percepcion_iibb_mi_sufrida","1.1.4.02.570","asset_current","Perception of IIBB Misiones incurred","False","Percepción IIBB Misiones sufrida"
"base_saldo_favor_iibb_ne","1.1.4.02.580","asset_current","Balance in favor IIBB Neuquén","False","Saldo a favor IIBB Neuquén"
"base_retencion_iibb_ne_sufrida","1.1.4.02.590","asset_current","IIBB withholding Neuquén incurred","False","Retención IIBB Neuquén sufrida"
"base_percepcion_iibb_ne_sufrida","1.1.4.02.600","asset_current","Perception IIBB Neuquén incurred","False","Percepción IIBB Neuquén sufrida"
"base_saldo_favor_iibb_lp","1.1.4.02.610","asset_current","Balance in favor IIBB La Pampa","False","Saldo a favor IIBB La Pampa"
"base_retencion_iibb_lp_sufrida","1.1.4.02.620","asset_current","IIBB withholding La Pampa incurred","False","Retención IIBB La Pampa sufrida"
"base_percepcion_iibb_lp_sufrida","1.1.4.02.630","asset_current","Perception of IIBB La Pampa incurred","False","Percepción IIBB La Pampa sufrida"
"base_saldo_favor_iibb_rn","1.1.4.02.640","asset_current","Balance in favor IIBB Río Negro","False","Saldo a favor IIBB Río Negro"
"base_retencion_iibb_rn_sufrida","1.1.4.02.650","asset_current","IIBB withholding Rio Negro incurred","False","Retención IIBB Río Negro sufrida"
"base_percepcion_iibb_rn_sufrida","1.1.4.02.660","asset_current","Perception of IIBB Río Negro incurred","False","Percepción IIBB Río Negro sufrida"
"base_saldo_favor_iibb_az","1.1.4.02.670","asset_current","Balance in favor IIBB Santa Cruz","False","Saldo a favor IIBB Santa Cruz"
"base_retencion_iibb_az_sufrida","1.1.4.02.680","asset_current","IIBB withholding Santa Cruz incurred","False","Retención IIBB Santa Cruz sufrida"
"base_percepcion_iibb_az_sufrida","1.1.4.02.690","asset_current","Perception IIBB Santa Cruz incurred","False","Percepción IIBB Santa Cruz sufrida"
"base_saldo_favor_iibb_tf","1.1.4.02.700","asset_current","Balance in favor IIBB Tierra del Fuego","False","Saldo a favor IIBB Tierra del Fuego"
"base_retencion_iibb_tf_sufrida","1.1.4.02.710","asset_current","IIBB withholding Tierra del Fuego incurred","False","Retención IIBB Tierra del Fuego sufrida"
"base_percepcion_iibb_tf_sufrida","1.1.4.02.720","asset_current","Perception IIBB Tierra del Fuego incurred","False","Percepción IIBB Tierra del Fuego sufrida"
"base_sircreb","1.1.4.02.730","asset_current","SIRCREB","False","SIRCREB"
"base_saldo_a_favor_suss","1.1.4.03.010","asset_current","Balance in favor SUSS","False","Saldo a favor SUSS"
"base_retencion_suss_sufrida","1.1.4.03.020","asset_current","SUSS Withholding incurred","False","Retención SUSS Sufrida"
"base_imp_debitos_computables","1.1.4.05.050","asset_current","Tax on Computable Debits","False","Impuesto a los Débitos Computables"
"base_otros_creditos","1.1.5.01.010","asset_receivable","Sundry Accounts Receivable","True","Deudores Varios"
"base_materia_prima","1.1.6.01.010","asset_current","Raw Materials","False","Materia Prima"
"base_productos_en_proceso","1.1.6.01.020","asset_current","Products in process","False","Productos en proceso"
"base_productos_terminados","1.1.6.01.030","asset_current","Finished products","False","Productos terminados"
"base_mercaderia_reventa","1.1.6.01.040","asset_current","Resale merchandise","False","Mercaderia de reventa"
"base_anticipo_proveedores","1.1.6.01.050","asset_current","Advances to Suppliers","False","Anticipo a Proveedores"
"base_instalaciones","1.2.1.01.010","asset_fixed","Facilities","False","Instalaciones"
"base_amortizacion_acumulada_instalaciones","1.2.1.01.020","asset_fixed","Accumulated depreciation of facilities","False","Amortización acumulada instalaciones"
"base_maq_y_equipos","1.2.1.02.010","asset_fixed","Machinery and equipment","False","Maquinarias y equipos"
"base_amortizacion_acumulada_maq_y_equipos","1.2.1.02.020","asset_fixed","Accumulated depreciation of machinery and equipment","False","Amortización acumulada maquinarias y equipos"
"base_muebles_y_utiles","1.2.1.03.010","asset_fixed","Furniture and fixtures","False","Muebles y útiles"
"base_amortizacion_acumulada_muebles_utiles","1.2.1.03.020","asset_fixed","Accumulated depreciation furniture and fixtures","False","Amortización acumulada muebles y útiles"
"base_rodados","1.2.1.04.010","asset_fixed","Vehicules","False","Rodados"
"base_amortizacion_acumulada_rodados","1.2.1.04.020","asset_fixed","Accumulated depreciation on wheeled vehicles","False","Amortización acumulada rodados"
"base_derechos_de_marca","1.2.2.01.010","asset_fixed","Trademark rights","False","Derechos de marca"
"base_amortizacion_acumulada_derechos_de_marca","1.2.2.01.020","asset_fixed","Accumulated amortization Trademark rights","False","Amortización acumulada Derechos de marca"
"base_proveedores","2.1.1.01.010","liability_payable","Suppliers","True","Proveedores"
"base_cheques_diferidos","2.1.1.01.020","liability_current","Deferred checks payable","True","Cheques diferidos a pagar"
"base_cheques_rechazados","2.1.1.01.030","liability_current","Rejected Checks","False","Cheques Rechazados"
"base_anticipos_de_clientes","2.1.1.01.040","liability_current","Customer advances","False","Anticipos de clientes"
"base_alquileres_a_pagar","2.1.1.01.050","liability_payable","Rent to be paid","True","Alquileres a pagar"
"base_intereses_a_pagar","2.1.2.01.010","liability_current","Interest payable","False","Intereses a pagar"
"base_giro_en_descubierto","2.1.2.01.020","liability_current","Overdraft Bank xxxxx","False","Giro en descubierto Banco xxxxx"
"base_otras_deudas_documentadas","2.1.2.01.030","liability_current","Other documented debts","False","Otras deudas documentadas"
"base_prestamo_banco_x","2.1.2.01.040","liability_non_current","Loan Bank xxxxx","False","Prestamo Banco xxxxx"
"base_acreedores_varios","2.1.2.01.050","liability_payable","Other accounts payable","True","Acreedores varios"
"base_tasa_municipal_a_pagar","2.1.3.01.010","liability_payable","Municipal tax payable","True","Tasa Municipal a pagar"
"base_plan_tasa_municipal_a_pagar","2.1.3.01.020","liability_payable","Plan Municipal Tax to be paid","True","Plan Tasa Municipal a pagar"
"base_iibb_a_pagar","2.1.3.02.010","liability_payable","IIBB a pagar","True","IIBB a pagar"
"base_plan_de_iibb_a_pagar","2.1.3.02.520","liability_payable","IIBB plan to be paid","True","Plan de IIBB a pagar"
"base_sueldos_a_pagar","2.1.4.01.010","liability_payable","Salaries to be paid","True","Sueldos a pagar"
"base_suss_a_pagar","2.1.4.01.020","liability_payable","Social Laws to be paid","True","Leyes Sociales a pagar"
"base_seguro_de_vida_a_pagar","2.1.4.01.030","liability_payable","Life Insurance to be Paid","True","Seguro de Vida a Pagar"
"base_art_a_pagar","2.1.4.01.040","liability_payable","ART to be Paid","True","ART a Pagar"
"base_aportes_sindicales_a_pagar","2.1.4.01.050","liability_payable","Union to be paid","True","Sindicato a pagar"
"base_embargos_a_depositar","2.1.4.01.060","liability_payable","Liens to be deposited","True","Embargos a depositar"
"base_cta_particular_socio_x","2.1.5.01.010","liability_non_current","Private account partner x","False","Cta particular socio x"
"base_proveedores_largo_plazo","2.2.1.01.010","liability_non_current","Long-term debt Suppliers","False","Deudas Proveedores a largo plazo"
"base_prevision_sac_a_pagar","2.2.2.01.010","liability_current","Projected SAC Payable","False","Previsión SAC a Pagar"
"base_prevision_para_despidos","2.2.2.01.020","liability_current","Provision for Layoffs","False","Previsión para Despidos"
"base_prevision_gastos","2.2.2.01.030","liability_current","Forecast expenses","False","Prevision gastos"
"base_capital","3.1.1.01.010","equity","Subscribed capital","False","Capital suscripto"
"base_ajustes_de_capital","3.1.1.01.020","equity","Capital adjustments","False","Ajustes de capital"
"base_aportes_no_capitalizados","3.1.1.01.030","equity","Uncapitalized contributions","False","Aportes no capitalizados"
"base_reserva_legal","3.2.1.01.010","equity","Legal reserve","False","Reserva legal"
"base_reserva_estatuitaria","3.2.1.01.020","equity","Statutory reserve","False","Reserva estatuitaria"
"base_futuras_inversiones","3.2.1.01.030","equity","Future investment reserves","False","Reservas futuras inversiones"
"base_reservas_facultativas","3.2.1.01.040","equity","Optional reserves","False","Reservas facultativas"
"base_resultado_del_ejercicio","3.3.1.01.010","equity_unaffected","Income for the year","False","Resultado del ejercicio"
"base_resultado_acumulados","3.3.1.01.020","equity","Results of prior years","False","Resultados de ejercicios anteriores"
"base_ajuste_resultados","3.3.1.01.030","equity","Adjustment of prior years' results","False","Ajuste resultados ejercicios anteriores"
"base_venta_de_mercaderia","4.1.1.01.010","income","Sale of merchandise","False","Venta de mercadería"
"base_venta_de_servicios","4.1.1.01.020","income","Sale of services","False","Venta de servicios"
"base_resultado_intereses_ganados","4.2.1.01.010","income_other","Interest earned","False","Intereses ganados"
"base_diferencias_de_cambio","4.2.1.01.020","income_other","Exchange differences","False","Diferencias de cambio"
"base_ajuste_por_redondeo","4.2.1.01.030","income_other","Rounding adjustment","False","Ajuste por redondeo"
"base_resultado_venta_bienes_de_uso","4.3.1.01.010","income_other","Profit/(loss) on sale of property, plant and equipment","False","Resultado venta bienes de uso"
"base_recupero_de_gastos","4.3.1.01.020","income_other","Cost recovery","False","Recupero de gastos"
"base_aportes_no_reembolsables","4.3.1.01.030","income_other","Non-reimbursable contributions (subsidies)","False","Aportes no reeombolsables (subsidios)"
"base_cmv","5.1.1.01.010","expense","Cost of Goods Sold","False","Costo de Mercadería Vendida"
"base_descuentos_obtenidos","5.1.1.01.020","expense","Discounts Obtained","False","Descuentos Obtenidos"
"base_compra_mercaderia","5.1.1.01.030","expense","Purchase of merchandise","False","Compra de mercadería"
"base_haberes_produccion","5.1.2.01.010","expense","Salaries and SAC Production","False","Sueldos y SAC Producción"
"base_cargas_sociales_produccion","5.1.2.01.020","expense","Social Charges Production","False","Cargas Sociales Producción"
"base_gastos_varios_produccion","5.1.2.01.030","expense","Miscellaneous Production","False","Gastos Varios Producción"
"base_alquileres_produccion","5.1.2.01.040","expense","Rentals Production","False","Alquileres Producción"
"base_servicio_de_luz_produccion","5.1.2.01.050","expense","Electrical Service Production","False","Servicio Eléctrico Producción"
"base_servicio_de_agua_produccion","5.1.2.01.060","expense","Water Service Production","False","Servicio de Agua Producción"
"base_servicio_de_gas_produccion","5.1.2.01.070","expense","Gas Service Production","False","Servicio de Gas Producción"
"base_impuesto_inmobiliario_produccion","5.1.2.01.080","expense","Real Estate Tax Production","False","Impuesto Inmobiliario Producción"
"base_mantenimiento_y_reparaciones_produccion","5.1.2.01.090","expense","Maintenance and Repairs Production","False","Mantenimiento y Reparaciones Producción"
"base_higiene_y_seguridad","5.1.2.01.100","expense","Hygiene and Safety","False","Higiene y Seguridad"
"base_honorarios_produccion","5.1.2.01.110","expense","Production Fees","False","Honorarios Producción"
"base_mantenimiento_y_limpieza","5.1.2.01.120","expense","Maintenance and cleaning","False","Mantenimiento y limpieza"
"base_seguros_produccion","5.1.2.01.130","expense","Insurance Production","False","Seguros Producción"
"base_haberes_comerciales","5.2.1.01.010","expense","Salaries and SAC Commercial","False","Sueldos y SAC Comercial"
"base_cargas_sociales_comerciales","5.2.1.01.020","expense","Commercial Social Charges","False","Cargas Sociales Comercial"
"base_gastos_varios_comerciales","5.2.1.01.030","expense","Miscellaneous Commercial","False","Gastos Varios Comercial"
"base_alquileres_comerciales","5.2.1.01.040","expense","Commercial Rentals","False","Alquileres Comercial"
"base_movilidad_y_viaticos","5.2.1.01.050","expense","Mobility and per diems","False","Movilidad y viáticos"
"base_publicidad","5.2.1.01.060","expense","Advertising","False","Publicidad"
"base_comisiones","5.2.1.01.070","expense","Commissions Paid","False","Comisiones Pagadas"
"base_servicios_de_luz_ecomercial","5.2.1.01.080","expense","Commercial Electric Service","False","Servicio Eléctrico Comercial"
"base_servicio_de_agua_comercial","5.2.1.01.090","expense","Commercial Water Service","False","Servicio de Agua Comercial"
"base_servicio_de_gas_comercial","5.2.1.01.100","expense","Commercial Gas Service","False","Servicio de Gas Comercial"
"base_honorarios_comercial","5.2.1.01.110","expense","Commercial Fees","False","Honorarios Comercial"
"base_patentes_comercial","5.2.1.01.120","expense","Patents Commercial","False","Patentes Comercial"
"base_seguros_comercial","5.2.1.01.130","expense","Commercial Insurance","False","Seguros Comercial"
"base_haberes_administrativos","5.3.1.01.010","expense","Administrative Salaries and SAC","False","Sueldos y SAC Administrativos"
"base_cargas_sociales_administrativos","5.3.1.01.020","expense","Administrative Social Charges","False","Cargas Sociales Administrativos"
"base_gastos_varios_administrativos","5.3.1.01.030","expense","Miscellaneous Administrative Expenses","False","Gastos varios Administrativos"
"base_alquileres_administrativos","5.3.1.01.040","expense","Administrative Rents","False","Alquileres Administrativos"
"base_servicio_de_luz_administrativos","5.3.1.01.050","expense","Electrical Service Administrative","False","Servicio Eléctrico Administrativos"
"base_servicio_de_agua_administrativos","5.3.1.01.060","expense","Water Service Administrative","False","Servicio de Agua Administrativos"
"base_servicio_de_gas_administrativos","5.3.1.01.070","expense","Administrative Gas Service","False","Servicio de Gas Administrativos"
"base_servicio_de_internet","5.3.1.01.080","expense","Internet Service","False","Servicio de Internet"
"base_sistema_y_software","5.3.1.01.090","expense","System and Software","False","Sistema y Software"
"base_cadeteria_y_franqueo","5.3.1.01.100","expense","Cadastre and postage","False","Cadeteria y franqueo"
"base_honorarios_administracion","5.3.1.01.110","expense","Administration Fees","False","Honorarios Administración"
"base_articulos_de_libreria","5.3.1.01.120","expense","Bookstore items","False","Artículos de librería"
"base_seguros_administracion","5.3.1.01.130","expense","Insurance Administration","False","Seguros Administración"
"base_sellados_y_certificaciones","5.3.1.01.140","expense","Seals and Certifications","False","Sellados y Certificaciones"
"base_tasa_municipal","5.4.1.01.010","expense","Municipal Tax","False","Tasa Municipal"
"base_impuestos_iibb_caba","5.4.2.01.010","expense","IIBB CABA","False","IIBB CABA"
"base_impuestos_iibb_ba","5.4.2.01.020","expense","IIBB ARBA","False","IIBB ARBA"
"base_impuestos_iibb_ca","5.4.2.01.030","expense","IIBB Catamarca","False","IIBB Catamarca"
"base_impuestos_iibb_co","5.4.2.01.040","expense","IIBB Córdoba","False","IIBB Córdoba"
"base_impuestos_iibb_rr","5.4.2.01.050","expense","IIBB Corrientes","False","IIBB Corrientes"
"base_impuestos_iibb_er","5.4.2.01.060","expense","IIBB Entre Ríos","False","IIBB Entre Ríos"
"base_impuestos_iibb_ju","5.4.2.01.070","expense","IIBB Jujuy","False","IIBB Jujuy"
"base_impuestos_iibb_za","5.4.2.01.080","expense","IIBB Mendoza","False","IIBB Mendoza"
"base_impuestos_iibb_lr","5.4.2.01.090","expense","IIBB La Rioja","False","IIBB La Rioja"
"base_impuestos_iibb_sa","5.4.2.01.100","expense","IIBB Salta","False","IIBB Salta"
"base_impuestos_iibb_nn","5.4.2.01.110","expense","IIBB San Juan","False","IIBB San Juan"
"base_impuestos_iibb_sl","5.4.2.01.120","expense","IIBB San Luis","False","IIBB San Luis"
"base_impuestos_iibb_sf","5.4.2.01.130","expense","IIBB Santa Fe","False","IIBB Santa Fe"
"base_impuestos_iibb_se","5.4.2.01.140","expense","IIBB Santiago del Estero","False","IIBB Santiago del Estero"
"base_impuestos_iibb_tn","5.4.2.01.150","expense","IIBB Tucumán","False","IIBB Tucumán"
"base_impuestos_iibb_ha","5.4.2.01.160","expense","IIBB Chaco","False","IIBB Chaco"
"base_impuestos_iibb_ct","5.4.2.01.170","expense","IIBB Chubut","False","IIBB Chubut"
"base_impuestos_iibb_fo","5.4.2.01.180","expense","IIBB Formosa","False","IIBB Formosa"
"base_impuestos_iibb_mi","5.4.2.01.190","expense","IIBB Misiones","False","IIBB Misiones"
"base_impuestos_iibb_ne","5.4.2.01.200","expense","IIBB Neuquén","False","IIBB Neuquén"
"base_impuestos_iibb_lp","5.4.2.01.210","expense","IIBB La Pampa","False","IIBB La Pampa"
"base_impuestos_iibb_rn","5.4.2.01.220","expense","IIBB Río Negro","False","IIBB Río Negro"
"base_impuestos_iibb_az","5.4.2.01.230","expense","IIBB Santa Cruz","False","IIBB Santa Cruz"
"base_impuestos_iibb_tf","5.4.2.01.240","expense","IIBB Tierra del Fuego","False","IIBB Tierra del Fuego"
"base_impuestos_debitos_y_creditos","5.4.3.01.010","expense","Taxes on bank debits and credits","False","Impuestos a los débitos y créditos bancarios"
"base_resultado_intereses_y_recargos","5.6.1.01.020","expense","Interest on loans","False","Intereses por préstamos"
"base_intereses_por_descubierto","5.6.1.01.030","expense","Overdraft interest","False","Intereses por descubierto"
"base_intereses_por_venta_de_valores","5.6.1.01.040","expense","Interest on sale of securities","False","Intereses por venta de valores"
"base_intereses_fiscales","5.6.1.01.050","expense","Tax interest","False","Intereses fiscales"
"base_gastos_bancarios","5.6.1.01.060","expense","Bank charges","False","Gastos Bancarios"
"base_r_e_c_p_a_m","5.6.1.01.070","expense","R.E.C.P.A.M.","False","R.E.C.P.A.M."
"base_amortizacion_instalaciones","5.7.1.01.010","expense_depreciation","Amortization of facilities","False","Amortización instalaciones"
"base_amortizacion_maq_y_equipos","5.7.1.01.020","expense_depreciation","Depreciation of machinery and equipment","False","Amortización maquinarias y equipos"
"base_amortizacion_muebles_utiles","5.7.1.01.030","expense_depreciation","Depreciation of furniture and fixtures","False","Amortización muebles y útiles"
"base_amortizacion_rodados","5.7.1.01.040","expense_depreciation","Amortization of rolling stock","False","Amortización rodados"
"base_amortizacion_derechos_de_marca","5.7.1.01.050","expense_depreciation","Amortization of trademark rights","False","Amortización Derechos de marca"
"base_contrapartida_auxiliar","6.0.0.00.010","asset_current","Auxiliary Counterpart","False","Contrapartida Auxiliar"
"base_default_vat","9.9.9.99.999","liability_current","Default VAT Payable/Receivable Account","False","Cuenta predeterminada de IVA por pagar/por cobrar"

```

## File: data\template\account.account-ar_ex.csv

```csv
"id","code","account_type","name","reconcile",name@es
"base_anticipo_ganancias","1.1.4.05.010","asset_current","Earnings advance","False",Anticipo ganancias
"base_percepcion_ganancias_sufrida","1.1.4.05.020","asset_current","Perceptions of Earnings incurred","False",Percepciones de Ganancias Sufridas
"base_retencion_ganancias_sufrida","1.1.4.05.030","asset_current","Withholdings of Profits incurred","False",Retenciones de Ganancias Sufridas
"base_saldo_favor_ganancias","1.1.4.05.040","asset_current","Profit balance","False",Saldo a favor Ganancias
"ri_retencion_sicore_a_pagar","2.1.3.02.020","liability_payable","SICORE to be paid","True",SICORE a pagar
"ri_retencion_iibb_caba_aplicada","2.1.3.02.030","liability_current","IIBB CABA withholding applied","False",Retención IIBB CABA aplicada
"ri_percepcion_iibb_caba_aplicada","2.1.3.02.040","liability_current","Perception of IIBB CABA applied","False",Percepción IIBB CABA aplicada
"ri_retencion_iibb_ba_aplicada","2.1.3.02.050","liability_current","IIBB ARBA withholding applied","False",Retención IIBB ARBA aplicada
"ri_percepcion_iibb_ba_aplicada","2.1.3.02.060","liability_current","Perception IIBB ARBA applied","False",Percepción IIBB ARBA aplicada
"ri_retencion_iibb_ca_aplicada","2.1.3.02.070","liability_current","IIBB withholding Catamarca applied","False",Retención IIBB Catamarca aplicada
"ri_percepcion_iibb_ca_aplicada","2.1.3.02.080","liability_current","Perception IIBB Catamarca applied","False",Percepción IIBB Catamarca aplicada
"ri_retencion_iibb_co_aplicada","2.1.3.02.090","liability_current","IIBB Córdoba withholding applied","False",Retención IIBB Córdoba aplicada
"ri_percepcion_iibb_co_aplicada","2.1.3.02.100","liability_current","Perception IIBB Córdoba applied","False",Percepción IIBB Córdoba aplicada
"ri_retencion_iibb_rr_aplicada","2.1.3.02.110","liability_current","IIBB Corrientes withholding applied","False",Retención IIBB Corrientes aplicada
"ri_percepcion_iibb_rr_aplicada","2.1.3.02.120","liability_current","Perception of IIBB Corrientes applied","False",Percepción IIBB Corrientes aplicada
"ri_retencion_iibb_er_aplicada","2.1.3.02.130","liability_current","IIBB withholding Entre Río applied","False",Retención IIBB Entre Río aplicada
"ri_percepcion_iibb_er_aplicada","2.1.3.02.140","liability_current","Perception of IIBB Entre Río applied","False",Percepción IIBB Entre Río aplicada
"ri_retencion_iibb_ju_aplicada","2.1.3.02.150","liability_current","IIBB Jujuy withholding applied","False",Retención IIBB Jujuy aplicada
"ri_percepcion_iibb_ju_aplicada","2.1.3.02.160","liability_current","Perception IIBB Jujuy applied","False",Percepción IIBB Jujuy aplicada
"ri_retencion_iibb_za_aplicada","2.1.3.02.170","liability_current","IIBB Mendoza withholding applied","False",Retención IIBB Mendoza aplicada
"ri_percepcion_iibb_za_aplicada","2.1.3.02.180","liability_current","Perception IIBB Mendoza applied","False",Percepción IIBB Mendoza aplicada
"ri_retencion_iibb_lr_aplicada","2.1.3.02.190","liability_current","IIBB withholding La Rioja applied","False",Retención IIBB La Rioja aplicada
"ri_percepcion_iibb_lr_aplicada","2.1.3.02.200","liability_current","Perception IIBB La Rioja applied","False",Percepción IIBB La Rioja aplicada
"ri_retencion_iibb_sa_aplicada","2.1.3.02.210","liability_current","IIBB withholding IIBB Salta applied","False",Retención IIBB Salta aplicada
"ri_percepcion_iibb_sa_aplicada","2.1.3.02.220","liability_current","Perception IIBB Salta applied","False",Percepción IIBB Salta aplicada
"ri_retencion_iibb_nn_aplicada","2.1.3.02.230","liability_current","IIBB withholding San Juan applied","False",Retención IIBB San Juan aplicada
"ri_percepcion_iibb_nn_aplicada","2.1.3.02.240","liability_current","Perception IIBB San Juan applied","False",Percepción IIBB San Juan aplicada
"ri_retencion_iibb_sl_aplicada","2.1.3.02.250","liability_current","IIBB withholding San Luis applied","False",Retención IIBB San Luis aplicada
"ri_percepcion_iibb_sl_aplicada","2.1.3.02.260","liability_current","Perception IIBB San Luis applied","False",Percepción IIBB San Luis aplicada
"ri_retencion_iibb_sf_aplicada","2.1.3.02.270","liability_current","Withholding IIBB Santa Fe applied","False",Retención IIBB Santa Fe aplicada
"ri_percepcion_iibb_sf_aplicada","2.1.3.02.280","liability_current","Perception IIBB Santa Fe applied","False",Percepción IIBB Santa Fe aplicada
"ri_retencion_iibb_se_aplicada","2.1.3.02.290","liability_current","IIBB Santiago del Estero withholding applied","False",Retención IIBB Santiago del Estero aplicada
"ri_percepcion_iibb_se_aplicada","2.1.3.02.300","liability_current","Perception IIBB Santiago del Estero applied","False",Percepción IIBB Santiago del Estero aplicada
"ri_retencion_iibb_tn_aplicada","2.1.3.02.310","liability_current","IIBB withholding Tucumán applied","False",Retención IIBB Tucumán aplicada
"ri_percepcion_iibb_tn_aplicada","2.1.3.02.320","liability_current","Perception IIBB Tucumán applied","False",Percepción IIBB Tucumán aplicada
"ri_retencion_iibb_ha_aplicada","2.1.3.02.330","liability_current","IIBB Chaco withholding applied","False",Retención IIBB Chaco aplicada
"ri_percepcion_iibb_ha_aplicada","2.1.3.02.340","liability_current","Perception IIBB Chaco applied","False",Percepción IIBB Chaco aplicada
"ri_retencion_iibb_ct_aplicada","2.1.3.02.350","liability_current","IIBB Chubut withholding applied","False",Retención IIBB Chubut aplicada
"ri_percepcion_iibb_ct_aplicada","2.1.3.02.360","liability_current","Perception IIBB Chubut applied","False",Percepción IIBB Chubut aplicada
"ri_retencion_iibb_fo_aplicada","2.1.3.02.370","liability_current","IIBB Formosa withholding applied","False",Retención IIBB Formosa aplicada
"ri_percepcion_iibb_fo_aplicada","2.1.3.02.380","liability_current","Perception IIBB Formosa applied","False",Percepción IIBB Formosa aplicada
"ri_retencion_iibb_mi_aplicada","2.1.3.02.390","liability_current","IIBB Misiones withholding applied","False",Retención IIBB Misiones aplicada
"ri_percepcion_iibb_mi_aplicada","2.1.3.02.400","liability_current","Perception IIBB Misiones applied","False",Percepción IIBB Misiones aplicada
"ri_retencion_iibb_ne_aplicada","2.1.3.02.410","liability_current","IIBB withholding Neuquén applied","False",Retención IIBB Neuquén aplicada
"ri_percepcion_iibb_ne_aplicada","2.1.3.02.420","liability_current","Perception IIBB Neuquén applied","False",Percepción IIBB Neuquén aplicada
"ri_retencion_iibb_lp_aplicada","2.1.3.02.430","liability_current","IIBB withholding La Pampa applied","False",Retención IIBB La Pampa aplicada
"ri_percepcion_iibb_lp_aplicada","2.1.3.02.440","liability_current","Perception of IIBB La Pampa applied","False",Percepción IIBB La Pampa aplicada
"ri_retencion_iibb_rn_aplicada","2.1.3.02.450","liability_current","IIBB withholding Rio Negro applied","False",Retención IIBB Río Negro aplicada
"ri_percepcion_iibb_rn_aplicada","2.1.3.02.460","liability_current","Perception IIBB Río Negro applied","False",Percepción IIBB Río Negro aplicada
"ri_retencion_iibb_az_aplicada","2.1.3.02.470","liability_current","IIBB withholding Santa Cruz applied","False",Retención IIBB Santa Cruz aplicada
"ri_percepcion_iibb_az_aplicada","2.1.3.02.480","liability_current","Perception IIBB Santa Cruz applied","False",Percepción IIBB Santa Cruz aplicada
"ri_retencion_iibb_tf_aplicada","2.1.3.02.490","liability_current","IIBB withholding Tierra del Fuego applied","False",Retención IIBB Tierra del Fuego aplicada
"ri_percepcion_iibb_tf_aplicada","2.1.3.02.500","liability_current","Perception IIBB Tierra del Fuego applied","False",Percepción IIBB Tierra del Fuego aplicada
"ri_retencion_iibb_a_pagar","2.1.3.02.510","liability_payable","Withholding/Perception IIBB to be paid","True",Retención/Percepción IIBB a pagar
"ri_retencion_iva_aplicada","2.1.3.03.030","liability_current","VAT withholding applied","False",Retención IVA aplicada
"ri_percepcion_iva_aplicada","2.1.3.03.040","liability_current","Perception VAT applied","False",Percepción IVA aplicada
"base_impuesto_ganancias_a_pagar","2.1.3.04.010","liability_payable","Income tax payable","True",Impuesto a las ganancias a pagar
"ri_retencion_ganancias_aplicada","2.1.3.04.020","liability_current","Withholding applied to earnings","False",Retención ganancias aplicada
"ri_percepcion_ganancias_aplicada","2.1.3.04.030","liability_current","Perception applied earnings","False",Percepción ganancias aplicada
"base_provision_imp_a_las_ganancias","2.1.3.04.040","liability_current","Provision for income tax","True",Provisión Imp a las Ganancias
"base_anticipos_imp_a_las_ganancias_a_pagar","2.1.3.04.050","liability_current","Income Tax Advances Payable","True",Anticipos Imp a las Ganancias a Pagar
"base_planes_a_pagar_afip","2.1.3.04.060","liability_non_current","Plan Earnings to be paid","False",Plan Ganancias a pagar
"base_impuestos_a_las_ganancias","5.5.1.01.010","expense","Income Taxes","False",Impuestos a las ganancias

```

## File: data\template\account.account-ar_ri.csv

```csv
"id","code","account_type","name","reconcile",name@es
"ri_iva_credito_fiscal","1.1.4.04.010","asset_current","VAT tax credit","False",IVA crédito fiscal
"ri_percepcion_iva_sufrida","1.1.4.04.020","asset_current","Perception VAT incurred","False",Percepción IVA Sufrida
"ri_retencion_iva_sufrida","1.1.4.04.030","asset_current","VAT withholding incurred","False",Retención IVA Sufrida
"ri_iva_saldo_tecnico_favor","1.1.4.04.040","asset_current","VAT Technical balance","False",IVA Saldo técnico
"ri_iva_saldo_libre_disponibilidad","1.1.4.04.050","asset_current","VAT Unrestricted balance","False",IVA Saldo Libre Disponibilidad
"ri_iva_debito_fiscal","2.1.3.03.010","liability_current","VAT tax debit","False",IVA débito fiscal
"ri_iva_saldo_a_pagar","2.1.3.03.020","liability_payable","VAT balance payable","True",IVA saldo a pagar
"base_plan_iva_a_pagar","2.1.3.03.050","liability_current","VAT plan to be paid","False",Plan IVA a pagar

```

## File: data\template\account.fiscal.position-ar_ri.csv

```csv
"id","name","auto_apply","l10n_ar_afip_responsibility_type_ids","tax_ids/tax_src_id","tax_ids/tax_dest_id",name@es
"fiscal_position_template_exempt_operations","Purchases / Sales abroad","1","l10n_ar.res_EXT","ri_tax_vat_0_ventas","ri_tax_vat_exento_ventas",Compras / Ventas al exterior
"","","","","ri_tax_vat_10_ventas","ri_tax_vat_exento_ventas",""
"","","","","ri_tax_vat_21_ventas","ri_tax_vat_exento_ventas",""
"","","","","ri_tax_vat_27_ventas","ri_tax_vat_exento_ventas",""
"","","","","ri_tax_vat_0_compras","ri_tax_vat_no_corresponde_compras",""
"","","","","ri_tax_vat_10_compras","ri_tax_vat_no_corresponde_compras",""
"","","","","ri_tax_vat_21_compras","ri_tax_vat_no_corresponde_compras",""
"","","","","ri_tax_vat_27_compras","ri_tax_vat_no_corresponde_compras",""
"","","","","ri_tax_vat_exento_compras","ri_tax_vat_no_corresponde_compras",""
"","","","","ri_tax_vat_no_gravado_compras","ri_tax_vat_no_corresponde_compras",""
"fiscal_position_template_free_zone","Purchases / Sales Free Trade Zone","1","l10n_ar.res_IVA_LIB","ri_tax_vat_0_ventas","ri_tax_vat_exento_ventas",Compras / Ventas Zona Franca
"","","","","ri_tax_vat_10_ventas","ri_tax_vat_exento_ventas",""
"","","","","ri_tax_vat_21_ventas","ri_tax_vat_exento_ventas",""
"","","","","ri_tax_vat_27_ventas","ri_tax_vat_exento_ventas",""
"","","","","ri_tax_vat_0_compras","ri_tax_vat_exento_compras",""
"","","","","ri_tax_vat_10_compras","ri_tax_vat_exento_compras",""
"","","","","ri_tax_vat_21_compras","ri_tax_vat_exento_compras",""
"","","","","ri_tax_vat_27_compras","ri_tax_vat_exento_compras",""
"fiscal_position_template_iva_no_corresponde","Purchases VAT in the correspondent","1","l10n_ar.res_IVAE,l10n_ar.res_RM","ri_tax_vat_0_compras","ri_tax_vat_no_corresponde_compras",Compras IVA no corresponde
"","","","","ri_tax_vat_10_compras","ri_tax_vat_no_corresponde_compras",""
"","","","","ri_tax_vat_21_compras","ri_tax_vat_no_corresponde_compras",""
"","","","","ri_tax_vat_27_compras","ri_tax_vat_no_corresponde_compras",""
"","","","","ri_tax_vat_exento_compras","ri_tax_vat_no_corresponde_compras",""
"","","","","ri_tax_vat_no_gravado_compras","ri_tax_vat_no_corresponde_compras",""

```

## File: data\template\account.group-ar_base.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@es"
"account_group_activo","1","","Active","Activo"
"account_group_activo_corriente","1.1","","Current Assets","Activo Corriente"
"account_group_cajas_y_bancos","1.1.1","","Banks and savings","Cajas y Bancos"
"account_group_caja","1.1.1.01","","Savings","Caja"
"account_group_bancos","1.1.1.02","","Banks","Bancos"
"account_group_inversiones","1.1.2","","Investments","Inversiones"
"account_group_creditos_por_ventas","1.1.3","","Sales receivables","Créditos por ventas"
"account_group_creditos_fiscales","1.1.4","","Tax credits","Créditos fiscales"
"account_group_creditos_fiscales_municipales","1.1.4.01","","Municipal Tax Credits","Créditos Fiscales Municipales"
"account_group_creditos_fiscales_iibb","1.1.4.02","","IIBB Tax Credits","Créditos Fiscales IIBB"
"account_group_creditos_fiscales_suss","1.1.4.03","","SUSS Tax Credits","Créditos Fiscales SUSS"
"account_group_creditos_fiscales_iva","1.1.4.04","","VAT Tax Credits","Créditos Fiscales IVA"
"account_group_creditos_fiscales_ganancias","1.1.4.05","","Tax Credits Earnings","Créditos Fiscales Ganancias"
"account_group_otros_creditos","1.1.5","","Other credits","Otros créditos"
"account_group_bienes_de_cambio","1.1.6","","Exchange Assets","Bienes de Cambio"
"account_group_activo_no_corriente","1.2","","Non-current assets","Activo no corriente"
"account_group_activos_fijos","1.2.1","","Fixed Assets","Activos Fijos"
"account_group_instalaciones","1.2.1.01","","Facilities","Instalaciones"
"account_group_maquinarias_y_equipos","1.2.1.02","","Machinery and Equipment","Maquinarias y Equipos"
"account_group_muebles_y_útiles","1.2.1.03","","Furniture and Fixtures","Muebles y Útiles"
"account_group_rodados","1.2.1.04","","Vehicules","Rodados"
"account_group_activos_intangibles","1.2.2","","Intangible Assets","Activos Intangibles"
"account_group_derechos_de_marcas","1.2.2.01","","Trademark Rights","Derechos de Marcas"
"account_group_pasivo","2","","Passive","Pasivo"
"account_group_pasivo_corriente","2.1","","Current Liabilities","Pasivo Corriente"
"account_group_deudas_comerciales","2.1.1","","Commercial Debts","Deudas Comerciales"
"account_group_deudas_financieras","2.1.2","","Financial Debts","Deudas Financieras"
"account_group_deudas_fiscales","2.1.3","","Tax Debts","Deudas Fiscales"
"account_group_deudas_tasas_municipales","2.1.3.01","","Municipal Tax Debts","Deudas Tasas Municipales"
"account_group_deudas_iibb","2.1.3.02","","IIBB Debts","Deudas IIBB"
"account_group_deudas_iva","2.1.3.03","","VAT debts","Deudas IVA"
"account_group_deudas_imp_ganancias","2.1.3.04","","Income Tax Debts","Deudas Imp. Ganancias"
"account_group_remueraciones_y_cargas_sociales","2.1.4","","Remunerations and Social Charges","Remuneraciones y Cargas Sociales"
"account_group_cuentas_particulares","2.1.5","","Private Accounts","Cuentas Particulares"
"account_group_pasivo_no_corriente","2.2","","Non-Current Liabilities","Pasivo no Corriente"
"account_group_deudas_comerciales_a_largo_plazo","2.2.1","","Long-Term Commercial Debts","Deudas Comerciales a Largo Plazo"
"account_group_previsiones","2.2.2","","Forecasts","Previsiones"
"account_group_patrimonio_neto","3","","Equity","Patrimonio Neto"
"account_group_capital_social","3.1","","Capital Social","Capital Social"
"account_group_reservas","3.2","","Reservations","Reservas"
"account_group_resultados","3.3","","Results","Resultados"
"account_group_ingresos","4","","Revenues","Ingresos"
"account_group_ingresos_por_ventas","4.1","","Sales revenue","Ingresos por ventas"
"account_group_ingresos_por_resultados_financieros","4.2","","Income from financial results","Ingresos por resultados financieros"
"account_group_ingresos_extraordinarios","4.3","","Extraordinary Income","Ingresos Extraordinarios"
"account_group_egresos","5","","Expenses","Egresos"
"account_group_gastos_operativos","5.1","","Operating Expenses","Gastos Operativos"
"account_group_costo_de_mercadería_vendida","5.1.1","","Cost of Goods Sold","Costo de Mercadería Vendida"
"account_group_gastos_de_producción","5.1.2","","Production Expenses","Gastos de Producción"
"account_group_gastos_comerciales","5.2","","Commercial Expenses","Gastos Comerciales"
"account_group_gastos_administrativos","5.3","","Administrative Expenses","Gastos Administrativos"
"account_group_impuestos","5.4","","Taxes","Impuestos"
"account_group_tasas_municipales","5.4.1","","Municipal Taxes","Tasas Municipales"
"account_group_iibb","5.4.2","","IIBB","IIBB"
"account_group_otros_impuestos","5.4.3","","Other Taxes","Otros Impuestos"
"account_group_imp_a_las_ganancias","5.5","","Income Tax","Imp a las Ganancias"
"account_group_gastos_financieros","5.6","","Financial Expenses","Gastos Financieros"
"account_group_cuentas_puentes","6","","BRIDGE ACCOUNTS","CUENTAS PUENTES"

```

## File: data\template\account.tax-ar_base.csv

```csv
"id","name","description","invoice_label","sequence","amount_type","amount","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","name@es","description@es","invoice_label@es"
"ri_tax_percepcion_iibb_caba_sufrida","P. IIBB CABA","Perception of IIBB Ciudad Autónoma de Buenos Aires","Perc IIBB Ciudad Autónoma de Buenos Aires","4","fixed","1","purchase","tax_group_percepcion_iibb_caba","base","invoice","","","Percepción IIBB Ciudad Autónoma de Buenos Aires","Perc IIBB Ciudad Autónoma de Buenos Aires"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_caba_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_caba_sufrida","","",""
"ri_tax_percepcion_iibb_ba_sufrida","P. IIBB BA","Perception IIBB Buenos Aires","Perc IIBB Buenos Aires","4","fixed","1","purchase","tax_group_percepcion_iibb_ba","base","invoice","","","Percepción IIBB Buenos Aires","Perc IIBB Buenos Aires"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_ba_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_ba_sufrida","","",""
"ri_tax_percepcion_iibb_ca_sufrida","P. IIBB C","Perception IIBB Catamarca","Perc IIBB Catamarca","4","fixed","1","purchase","tax_group_percepcion_iibb_ca","base","invoice","","","Percepción IIBB Catamarca","Perc IIBB Catamarca"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_ca_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_ca_sufrida","","",""
"ri_tax_percepcion_iibb_co_sufrida","P. IIBB CBA","Perception IIBB Córdoba","Perc IIBB Córdoba","4","fixed","1","purchase","tax_group_percepcion_iibb_co","base","invoice","","","Percepción IIBB Córdoba","Perc IIBB Córdoba"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_co_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_co_sufrida","","",""
"ri_tax_percepcion_iibb_rr_sufrida","P. IIBB CTS","Perception IIBB Corrientes","Perc IIBB Corrientes","4","fixed","1","purchase","tax_group_percepcion_iibb_rr","base","invoice","","","Percepción IIBB Corrientes","Perc IIBB Corrientes"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_rr_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_rr_sufrida","","",""
"ri_tax_percepcion_iibb_er_sufrida","P. IIBB ER","Perception IIBB Entre Ríos","Perc IIBB Entre Ríos","4","fixed","1","purchase","tax_group_percepcion_iibb_er","base","invoice","","","Percepción IIBB Entre Ríos","Perc IIBB Entre Ríos"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_er_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_er_sufrida","","",""
"ri_tax_percepcion_iibb_ju_sufrida","P. IIBB J","Perception IIBB Jujuy","Perc IIBB Jujuy","4","fixed","1","purchase","tax_group_percepcion_iibb_ju","base","invoice","","","Percepción IIBB Jujuy","Perc IIBB Jujuy"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_ju_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_ju_sufrida","","",""
"ri_tax_percepcion_iibb_za_sufrida","P. IIBB MZA","Perception IIBB Mendoza","Perc IIBB Mendoza","4","fixed","1","purchase","tax_group_percepcion_iibb_za","base","invoice","","","Percepción IIBB Mendoza","Perc IIBB Mendoza"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_za_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_za_sufrida","","",""
"ri_tax_percepcion_iibb_lr_sufrida","P. IIBB LR","Perception IIBB La Rioja","Perc IIBB La Rioja","4","fixed","1","purchase","tax_group_percepcion_iibb_lr","base","invoice","","","Percepción IIBB La Rioja","Perc IIBB La Rioja"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_lr_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_lr_sufrida","","",""
"ri_tax_percepcion_iibb_sa_sufrida","P. IIBB S","Perception IIBB Salta","Perc IIBB Salta","4","fixed","1","purchase","tax_group_percepcion_iibb_sa","base","invoice","","","Percepción IIBB Salta","Perc IIBB Salta"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_sa_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_sa_sufrida","","",""
"ri_tax_percepcion_iibb_nn_sufrida","P. IIBB SJ","Perception IIBB San Juan","Perc IIBB San Juan","4","fixed","1","purchase","tax_group_percepcion_iibb_nn","base","invoice","","","Percepción IIBB San Juan","Perc IIBB San Juan"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_nn_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_nn_sufrida","","",""
"ri_tax_percepcion_iibb_sl_sufrida","P. IIBB SL","Perception IIBB San Luis","Perc IIBB San Luis","4","fixed","1","purchase","tax_group_percepcion_iibb_sl","base","invoice","","","Percepción IIBB San Luis","Perc IIBB San Luis"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_sl_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_sl_sufrida","","",""
"ri_tax_percepcion_iibb_sf_sufrida","P. IIBB SF","Perception IIBB Santa Fe","Perc IIBB Santa Fe","4","fixed","1","purchase","tax_group_percepcion_iibb_sf","base","invoice","","","Percepción IIBB Santa Fe","Perc IIBB Santa Fe"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_sf_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_sf_sufrida","","",""
"ri_tax_percepcion_iibb_se_sufrida","P. IIBB SE","Perception IIBB Santiago del Estero","Perc IIBB Santiago del Estero","4","fixed","1","purchase","tax_group_percepcion_iibb_se","base","invoice","","","Percepción IIBB Santiago del Estero","Perc IIBB Santiago del Estero"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_se_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_se_sufrida","","",""
"ri_tax_percepcion_iibb_tn_sufrida","P. IIBB T","Perception IIBB Tucumán","Perc IIBB Tucumán","4","fixed","1","purchase","tax_group_percepcion_iibb_tn","base","invoice","","","Percepción IIBB Tucumán","Perc IIBB Tucumán"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_tn_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_tn_sufrida","","",""
"ri_tax_percepcion_iibb_ha_sufrida","P. IIBB CHO","Perception IIBB Chaco","Perc IIBB Chaco","4","fixed","1","purchase","tax_group_percepcion_iibb_ha","base","invoice","","","Percepción IIBB Chaco","Perc IIBB Chaco"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_ha_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_ha_sufrida","","",""
"ri_tax_percepcion_iibb_ct_sufrida","P. IIBB CHT","Perception IIBB Chubut","Perc IIBB Chubut","4","fixed","1","purchase","tax_group_percepcion_iibb_ct","base","invoice","","","Percepción IIBB Chubut","Perc IIBB Chubut"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_ct_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_ct_sufrida","","",""
"ri_tax_percepcion_iibb_fo_sufrida","P. IIBB F","Perception IIBB Formosa","Perc IIBB Formosa","4","fixed","1","purchase","tax_group_percepcion_iibb_fo","base","invoice","","","Percepción IIBB Formosa","Perc IIBB Formosa"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_fo_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_fo_sufrida","","",""
"ri_tax_percepcion_iibb_mi_sufrida","P. IIBB MS","Perception IIBB Misiones","Perc IIBB Misiones","4","fixed","1","purchase","tax_group_percepcion_iibb_mi","base","invoice","","","Percepción IIBB Misiones","Perc IIBB Misiones"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_mi_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_mi_sufrida","","",""
"ri_tax_percepcion_iibb_ne_sufrida","P. IIBB N","Perception IIBB Neuquén","Perc IIBB Neuquén","4","fixed","1","purchase","tax_group_percepcion_iibb_ne","base","invoice","","","Percepción IIBB Neuquén","Perc IIBB Neuquén"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_ne_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_ne_sufrida","","",""
"ri_tax_percepcion_iibb_lp_sufrida","P. IIBB LP","Perception IIBB La Pampa","Perc IIBB La Pampa","4","fixed","1","purchase","tax_group_percepcion_iibb_lp","base","invoice","","","Percepción IIBB La Pampa","Perc IIBB La Pampa"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_lp_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_lp_sufrida","","",""
"ri_tax_percepcion_iibb_rn_sufrida","P. IIBB RN","Perception IIBB Río Negro","Perc IIBB Río Negro","4","fixed","1","purchase","tax_group_percepcion_iibb_rn","base","invoice","","","Percepción IIBB Río Negro","Perc IIBB Río Negro"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_rn_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_rn_sufrida","","",""
"ri_tax_percepcion_iibb_az_sufrida","P. IIBB SC","Perception IIBB Santa Cruz","Perc IIBB Santa Cruz","4","fixed","1","purchase","tax_group_percepcion_iibb_az","base","invoice","","","Percepción IIBB Santa Cruz","Perc IIBB Santa Cruz"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_az_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_az_sufrida","","",""
"ri_tax_percepcion_iibb_tf_sufrida","P. IIBB TAIS","Perception IIBB Tierra del Fuego","Perc IIBB Tierra del Fuego","4","fixed","1","purchase","tax_group_percepcion_iibb_tf","base","invoice","","","Percepción IIBB Tierra del Fuego","Perc IIBB Tierra del Fuego"
"","","","","","","","","","tax","invoice","base_percepcion_iibb_tf_sufrida","","",""
"","","","","","","","","","base","refund","","","",""
"","","","","","","","","","tax","refund","base_percepcion_iibb_tf_sufrida","","",""

```

## File: data\template\account.tax-ar_ex.csv

```csv
"id","name","description","invoice_label","sequence","active","amount_type","amount","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","name@es","description@es","invoice_label@es"
"ri_tax_percepcion_iva_aplicada","Perc VAT 0%","Perception VAT","Perc VAT","4","False","percent","0","sale","tax_group_percepcion_iva","base","invoice","","Perc IVA","Percepción IVA","Perc IVA"
"","","","","","","","","","","tax","invoice","ri_percepcion_iva_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iva_aplicada","","",""
"ri_tax_percepcion_ganancias_aplicada","Perc Profits 0%","Perception of Profits","Perc Profits","4","False","percent","0","sale","tax_group_percepcion_ganancias","base","invoice","","Perc Gananc","Percepción Ganancias","Perc Ganancias"
"","","","","","","","","","","tax","invoice","ri_percepcion_ganancias_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_ganancias_aplicada","","",""
"ri_tax_percepcion_ganancias_sufrida","Perc Profits","Perception of Profits","Perc Profits","4","","fixed","1","purchase","tax_group_percepcion_ganancias","base","invoice","","Perc Gananc","Percepción Ganancias","Perc Ganancias"
"","","","","","","","","","","tax","invoice","base_percepcion_ganancias_sufrida","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","base_percepcion_ganancias_sufrida","","",""
"ri_tax_percepcion_iibb_caba_aplicada","P. IIBB CABA 0%","Perception IIBB Ciudad Autónoma de Buenos Aires","Perc IIBB Ciudad Autónoma de Buenos Aires","4","False","percent","0","sale","tax_group_percepcion_iibb_caba","base","invoice","","","Percepción IIBB Ciudad Autónoma de Buenos Aires","Perc IIBB Ciudad Autónoma de Buenos Aires"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_caba_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_caba_aplicada","","",""
"ri_tax_percepcion_iibb_ba_aplicada","P. IIBB BA 0%","Perception IIBB Buenos Aires","Perc IIBB Buenos Aires","4","False","percent","0","sale","tax_group_percepcion_iibb_ba","base","invoice","","","Percepción IIBB Buenos Aires","Perc IIBB Buenos Aires"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_ba_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_ba_aplicada","","",""
"ri_tax_percepcion_iibb_ca_aplicada","P. IIBB C 0%","Perception IIBB Catamarca","Perc IIBB Catamarca","4","False","percent","0","sale","tax_group_percepcion_iibb_ca","base","invoice","","","Percepción IIBB Catamarca","Perc IIBB Catamarca"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_ca_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_ca_aplicada","","",""
"ri_tax_percepcion_iibb_co_aplicada","P. IIBB CBA 0%","Perception IIBB Córdoba","Perc IIBB Córdoba","4","False","percent","0","sale","tax_group_percepcion_iibb_co","base","invoice","","","Percepción IIBB Córdoba","Perc IIBB Córdoba"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_co_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_co_aplicada","","",""
"ri_tax_percepcion_iibb_rr_aplicada","P. IIBB CTS 0%","Perception of IIBB Corrientes","Perc IIBB Corrientes","4","False","percent","0","sale","tax_group_percepcion_iibb_rr","base","invoice","","","Percepción IIBB Corrientes","Perc IIBB Corrientes"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_rr_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_rr_aplicada","","",""
"ri_tax_percepcion_iibb_er_aplicada","P. IIBB ER 0%","Perception of IIBB Entre Ríos","Perc IIBB Entre Ríos","4","False","percent","0","sale","tax_group_percepcion_iibb_er","base","invoice","","","Percepción IIBB Entre Ríos","Perc IIBB Entre Ríos"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_er_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_er_aplicada","","",""
"ri_tax_percepcion_iibb_ju_aplicada","P. IIBB J 0%","Perception IIBB Jujuy","Perc IIBB Jujuy","4","False","percent","0","sale","tax_group_percepcion_iibb_ju","base","invoice","","","Percepción IIBB Jujuy","Perc IIBB Jujuy"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_ju_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_ju_aplicada","","",""
"ri_tax_percepcion_iibb_za_aplicada","P. IIBB MZA 0%","Perception IIBB Mendoza","Perc IIBB Mendoza","4","False","percent","0","sale","tax_group_percepcion_iibb_za","base","invoice","","","Percepción IIBB Mendoza","Perc IIBB Mendoza"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_za_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_za_aplicada","","",""
"ri_tax_percepcion_iibb_lr_aplicada","P. IIBB LR 0%","Perception IIBB La Rioja","Perc IIBB La Rioja","4","False","percent","0","sale","tax_group_percepcion_iibb_lr","base","invoice","","","Percepción IIBB La Rioja","Perc IIBB La Rioja"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_lr_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_lr_aplicada","","",""
"ri_tax_percepcion_iibb_sa_aplicada","P. IIBB S 0%","Perception IIBB Salta","Perc IIBB Salta","4","False","percent","0","sale","tax_group_percepcion_iibb_sa","base","invoice","","","Percepción IIBB Salta","Perc IIBB Salta"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_sa_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_sa_aplicada","","",""
"ri_tax_percepcion_iibb_nn_aplicada","P. IIBB SJ 0%","Perception IIBB San Juan","Perc IIBB San Juan","4","False","percent","0","sale","tax_group_percepcion_iibb_nn","base","invoice","","","Percepción IIBB SJ","Perc IIBB San Juan"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_nn_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_nn_aplicada","","",""
"ri_tax_percepcion_iibb_sl_aplicada","P. IIBB SL 0%","Perception IIBB San Luis","Perc IIBB San Luis","4","False","percent","0","sale","tax_group_percepcion_iibb_sl","base","invoice","","","Percepción IIBB SL","Perc IIBB San Luis"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_sl_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_sl_aplicada","","",""
"ri_tax_percepcion_iibb_sf_aplicada","P. IIBB SF 0%","Perception IIBB Santa Fe","Perc IIBB Santa Fe","4","False","percent","0","sale","tax_group_percepcion_iibb_sf","base","invoice","","","Percepción IIBB SF","Perc IIBB Santa Fe"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_sf_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_sf_aplicada","","",""
"ri_tax_percepcion_iibb_se_aplicada","P. IIBB SE 0%","Perception IIBB Santiago del Estero","Perc IIBB Santiago del Estero","4","False","percent","0","sale","tax_group_percepcion_iibb_se","base","invoice","","","Percepción IIBB Santiago del Estero","Perc IIBB Santiago del Estero"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_se_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_se_aplicada","","",""
"ri_tax_percepcion_iibb_tn_aplicada","P. IIBB T 0%","Perception IIBB Tucumán","Perc IIBB Tucumán","4","False","percent","0","sale","tax_group_percepcion_iibb_tn","base","invoice","","","Percepción IIBB Tucumán","Perc IIBB Tucumán"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_tn_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_tn_aplicada","","",""
"ri_tax_percepcion_iibb_ha_aplicada","P. IIBB CHO 0%","Perception IIBB Chaco","Perc IIBB Chaco","4","False","percent","0","sale","tax_group_percepcion_iibb_ha","base","invoice","","","Percepción IIBB Chaco","Perc IIBB Chaco"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_ha_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_ha_aplicada","","",""
"ri_tax_percepcion_iibb_ct_aplicada","P. IIBB CHT 0%","Perception IIBB Chubut","Perc IIBB Chubut","4","False","percent","0","sale","tax_group_percepcion_iibb_ct","base","invoice","","","Percepción IIBB Chubut","Perc IIBB Chubut"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_ct_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_ct_aplicada","","",""
"ri_tax_percepcion_iibb_fo_aplicada","P. IIBB F 0%","Perception IIBB Formosa","Perc IIBB Formosa","4","False","percent","0","sale","tax_group_percepcion_iibb_fo","base","invoice","","","Percepción IIBB Formosa","Perc IIBB Formosa"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_fo_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_fo_aplicada","","",""
"ri_tax_percepcion_iibb_mi_aplicada","P. IIBB MS 0%","Perception IIBB Misiones","Perc IIBB Misiones","4","False","percent","0","sale","tax_group_percepcion_iibb_mi","base","invoice","","","Percepción IIBB Misiones","Perc IIBB Misiones"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_mi_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_mi_aplicada","","",""
"ri_tax_percepcion_iibb_ne_aplicada","P. IIBB N 0%","Perception IIBB Neuquén","Perc IIBB Neuquén","4","False","percent","0","sale","tax_group_percepcion_iibb_ne","base","invoice","","","Percepción IIBB Neuquén","Perc IIBB Neuquén"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_ne_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_ne_aplicada","","",""
"ri_tax_percepcion_iibb_lp_aplicada","P. IIBB LP 0%","Perception IIBB La Pampa","Perc IIBB La Pampa","4","False","percent","0","sale","tax_group_percepcion_iibb_lp","base","invoice","","","Percepción IIBB La Pampa","Perc IIBB La Pampa"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_lp_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_lp_aplicada","","",""
"ri_tax_percepcion_iibb_rn_aplicada","P. IIBB RN 0%","Perception IIBB Río Negro","Perc IIBB Río Negro","4","False","percent","0","sale","tax_group_percepcion_iibb_rn","base","invoice","","","Percepción IIBB Río Negro","Perc IIBB Río Negro"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_rn_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_rn_aplicada","","",""
"ri_tax_percepcion_iibb_az_aplicada","P. IIBB SC 0%","Perception IIBB Santa Cruz","Perc IIBB Santa Cruz","4","False","percent","0","sale","tax_group_percepcion_iibb_az","base","invoice","","","Percepción IIBB Santa Cruz","Perc IIBB Santa Cruz"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_az_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_az_aplicada","","",""
"ri_tax_percepcion_iibb_tf_aplicada","P. IIBB TAIS 0%","Perception IIBB Tierra del Fuego","Perc IIBB Tierra del Fuego","4","False","percent","0","sale","tax_group_percepcion_iibb_tf","base","invoice","","","Percepción IIBB Tierra del Fuego","Perc IIBB Tierra del Fuego"
"","","","","","","","","","","tax","invoice","ri_percepcion_iibb_tf_aplicada","","",""
"","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","tax","refund","ri_percepcion_iibb_tf_aplicada","","",""

```

## File: data\template\account.tax-ar_ri.csv

```csv
"id","description","invoice_label","name","active","sequence","amount_type","amount","tax_group_id","type_tax_use","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","name@es","description@es","invoice_label@es",
ri_tax_vat_no_corresponde_ventas,"VAT Not Applicable","VAT Not Applicable","0% NA","False","2","fixed","0.0","tax_group_iva_no_corresponde","sale","base","invoice","","IVA No Corresp","IVA No Corresponde","IVA No Corresponde",
"","","","","","","","","","","tax","invoice","ri_iva_debito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_credito_fiscal","","","",
ri_tax_vat_no_corresponde_compras,"VAT Not Applicable","VAT Not Applicable","0% NA","","2","fixed","0.0","tax_group_iva_no_corresponde","purchase","base","invoice","","IVA No Corresp","IVA No Corresponde","IVA No Corresponde",
"","","","","","","","","","","tax","invoice","ri_iva_credito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_debito_fiscal","","","",
ri_tax_vat_no_gravado_ventas,"VAT Not Taxed","VAT Not Taxed","0% NT","","2","fixed","0.0","tax_group_iva_no_gravado","sale","base","invoice","","IVA No Grav","IVA No Gravado","IVA No Gravado",
"","","","","","","","","","","tax","invoice","ri_iva_debito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_credito_fiscal","","","",
ri_tax_vat_no_gravado_compras,"VAT Not Taxed","VAT Not Taxed","0% NT","","2","fixed","0.0","tax_group_iva_no_gravado","purchase","base","invoice","","IVA No Grav","IVA No Gravado","IVA No Gravado",
"","","","","","","","","","","tax","invoice","ri_iva_credito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_debito_fiscal","","","",
ri_tax_vat_exento_ventas,"Exempt","VAT Exempt","0% EXEMPT","","2","fixed","0.0","tax_group_iva_exento","sale","base","invoice","","IVA Exen","IVA Exento","IVA Exento",
"","","","","","","","","","","tax","invoice","ri_iva_debito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_credito_fiscal","","","",
ri_tax_vat_exento_compras,"Exempt","VAT Exempt","0% EXEMPT","","2","fixed","0.0","tax_group_iva_exento","purchase","base","invoice","","IVA Exen","IVA Exento","IVA Exento",
"","","","","","","","","","","tax","invoice","ri_iva_credito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_debito_fiscal","","","",
ri_tax_vat_0_ventas,"","VAT 0%","VAT 0%","","2","percent","0.0","tax_group_iva_0","sale","base","invoice","","IVA 0%","IVA 0%","IVA 0%",
"","","","","","","","","","","tax","invoice","ri_iva_debito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_credito_fiscal","","","",
ri_tax_vat_0_compras,"","VAT 0%","VAT 0%","","2","percent","0.0","tax_group_iva_0","purchase","base","invoice","","IVA 0%","IVA 0%","IVA 0%",
"","","","","","","","","","","tax","invoice","ri_iva_credito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_debito_fiscal","","","",
ri_tax_vat_10_ventas,"","VAT 10.5%","VAT 10.5%","","2","percent","10.5","tax_group_iva_105","sale","base","invoice","","IVA 10.5%","IVA 10.5%","IVA 10.5%",
"","","","","","","","","","","tax","invoice","ri_iva_debito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_credito_fiscal","","","",
ri_tax_vat_10_compras,"","VAT 10.5%","VAT 10.5%","","2","percent","10.5","tax_group_iva_105","purchase","base","invoice","","IVA 10.5%","IVA 10.5%","IVA 10.5%",
"","","","","","","","","","","tax","invoice","ri_iva_credito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_debito_fiscal","","","",
ri_tax_vat_21_ventas,"","VAT 21%","VAT 21%","","1","percent","21.0","tax_group_iva_21","sale","base","invoice","","IVA 21%","IVA 21%","IVA 21%",
"","","","","","","","","","","tax","invoice","ri_iva_debito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_credito_fiscal","","","",
ri_tax_vat_21_compras,"","VAT 21%","VAT 21%","","1","percent","21.0","tax_group_iva_21","purchase","base","invoice","","IVA 21%","IVA 21%","IVA 21%",
"","","","","","","","","","","tax","invoice","ri_iva_credito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_debito_fiscal","","","",
ri_tax_vat_27_ventas,"","VAT 27%","VAT 27%","","3","percent","27.0","tax_group_iva_27","sale","base","invoice","","IVA 27%","IVA 27%","IVA 27%",
"","","","","","","","","","","tax","invoice","ri_iva_debito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_credito_fiscal","","","",
ri_tax_vat_27_compras,"","VAT 27%","VAT 27%","","3","percent","27.0","tax_group_iva_27","purchase","base","invoice","","IVA 27%","IVA 27%","IVA 27%",
"","","","","","","","","","","tax","invoice","ri_iva_credito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_debito_fiscal","","","",
ri_tax_vat_25_ventas,"","VAT 2.5%","VAT 2.5%","False","9","percent","2.5","tax_group_iva_025","sale","base","invoice","","IVA 2.5","IVA 2,5%","IVA 2.5",
"","","","","","","","","","","tax","invoice","ri_iva_debito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_credito_fiscal","","","",
ri_tax_vat_25_compras,"","VAT 2.5%","VAT 2.5%","False","9","percent","2.5","tax_group_iva_025","purchase","base","invoice","","IVA 2.5","IVA 2,5%","IVA 2.5",
"","","","","","","","","","","tax","invoice","ri_iva_credito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_debito_fiscal","","","",
ri_tax_vat_5_ventas,"","VAT 5%","VAT 5%","False","10","percent","5.0","tax_group_iva_5","sale","base","invoice","","IVA 5%","IVA 5%","IVA 5%",
"","","","","","","","","","","tax","invoice","ri_iva_debito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_credito_fiscal","","","",
ri_tax_vat_5_compras,"","VAT 5%","VAT 5%","False","10","percent","5.0","tax_group_iva_5","purchase","base","invoice","","IVA 5%","IVA 5%","IVA 5%",
"","","","","","","","","","","tax","invoice","ri_iva_credito_fiscal","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_iva_debito_fiscal","","","",
ri_tax_percepcion_iva_sufrida,"Perception VAT","Perc VAT","Perc VAT","","4","fixed","1.0","tax_group_percepcion_iva","purchase","base","invoice","","Perc IVA","Percepción IVA","Perc IVA",
"","","","","","","","","","","tax","invoice","ri_percepcion_iva_sufrida","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_percepcion_iva_sufrida","","","",
ri_tax_ganancias_iva_adicional,"VAT Additional 20%","VAT Additional 20%","VAT 20%","","4","percent","20.0","tax_group_percepcion_iva","purchase","base","invoice","","IVA Adic 20%","IVA Adicional 20%","IVA Adicional 20%",
"","","","","","","","","","","tax","invoice","ri_percepcion_iva_sufrida","","","",
"","","","","","","","","","","base","refund","","","","",
"","","","","","","","","","","tax","refund","ri_percepcion_iva_sufrida","","","",

```

## File: data\template\account.tax.group-ar_base.csv

```csv
"id","name","l10n_ar_vat_afip_code","country_id","sequence","l10n_ar_tribute_afip_code","tax_payable_account_id","tax_receivable_account_id","name@es"
"tax_group_iva_21","VAT 21%","5","base.ar","","","base_default_vat","base_default_vat","IVA 21%"
"tax_group_iva_27","VAT 27%","6","base.ar","","","base_default_vat","base_default_vat","IVA 27%"
"tax_group_iva_105","VAT 10.5%","4","base.ar","","","base_default_vat","base_default_vat","IVA 10,5%"
"tax_group_iva_025","VAT 2.5%","9","base.ar","","","base_default_vat","base_default_vat","IVA 2,5%"
"tax_group_iva_no_corresponde","VAT Not Applicable","0","base.ar","","","base_default_vat","base_default_vat","IVA No Corresponde"
"tax_group_iva_no_gravado","VAT Untaxed","1","base.ar","","","base_default_vat","base_default_vat","IVA No Gravado"
"tax_group_iva_exento","VAT Exempt","2","base.ar","","","base_default_vat","base_default_vat","IVA Exento"
"tax_group_iva_0","VAT 0%","3","base.ar","","","base_default_vat","base_default_vat","IVA 0%"
"tax_group_iva_5","VAT 5%","8","base.ar","","","base_default_vat","base_default_vat","IVA 5%"
"tax_group_otros_impuestos","Other Taxes","","base.ar","20","99","base_default_vat","base_default_vat","Otros Impuestos"
"tax_impuestos_internos","Internal Taxes","","base.ar","15","04","base_default_vat","base_default_vat","Impuestos internos"
"tax_group_national_taxes","National Taxes","","base.ar","30","01","base_default_vat","base_default_vat","Impuestos Nacionales"
"tax_group_percepcion_iva","VAT Perception","","base.ar","","06","base_default_vat","base_default_vat","Percepción del IVA"
"tax_group_percepcion_iibb_caba","Perc IIBB CABA","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB CABA"
"tax_group_percepcion_iibb_ba","Perc IIBB ARBA","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB ARBA"
"tax_group_percepcion_iibb_ca","Perc IIBB Catamarca","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Catamarca"
"tax_group_percepcion_iibb_co","Perc IIBB Córdoba","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Córdoba"
"tax_group_percepcion_iibb_rr","Perc IIBB Corrientes","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Corrientes"
"tax_group_percepcion_iibb_er","Perc IIBB Entre Ríos","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Entre Ríos"
"tax_group_percepcion_iibb_ju","Perc IIBB Jujuy","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Jujuy"
"tax_group_percepcion_iibb_za","Perc IIBB Mendoza","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Mendoza"
"tax_group_percepcion_iibb_lr","Perc IIBB La Rioja","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB La Rioja"
"tax_group_percepcion_iibb_sa","Perc IIBB Salta","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Salta"
"tax_group_percepcion_iibb_nn","Perc IIBB San Juan","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB San Juan"
"tax_group_percepcion_iibb_sl","Perc IIBB San Luis","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB San Luis"
"tax_group_percepcion_iibb_sf","Perc IIBB Santa Fe","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Santa Fe"
"tax_group_percepcion_iibb_se","Perc IIBB Santiago del Estero","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Santiago del Estero"
"tax_group_percepcion_iibb_tn","Perc IIBB Tucumán","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Tucumán"
"tax_group_percepcion_iibb_ha","Perc IIBB Chaco","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Chaco"
"tax_group_percepcion_iibb_ct","Perc IIBB Chubut","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Chubut"
"tax_group_percepcion_iibb_fo","Perc IIBB Formosa","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Formosa"
"tax_group_percepcion_iibb_mi","Perc IIBB Misiones","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Misiones"
"tax_group_percepcion_iibb_ne","Perc IIBB Neuquén","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Neuquén"
"tax_group_percepcion_iibb_lp","Perc IIBB La Pampa","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB La Pampa"
"tax_group_percepcion_iibb_rn","Perc IIBB Río Negro","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Río Negro"
"tax_group_percepcion_iibb_az","Perc IIBB Santa Cruz","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Santa Cruz"
"tax_group_percepcion_iibb_tf","Perc IIBB Tierra del Fuego","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Tierra del Fuego"
"tax_group_percepcion_iibb","IIBB Perceptions","","base.ar","25","07","base_default_vat","base_default_vat","IIBB Percepciones"
"tax_group_percepcion_municipal","Municipal Taxes Perceptions","","base.ar","","08","base_default_vat","base_default_vat","Percepción de los impuestos municipales"
"tax_group_percepcion_ganancias","Profit Perceptions","","base.ar","","09","base_default_vat","base_default_vat","Percepción de los beneficios"
"tax_group_otras_percepciones","Other Perceptions","","base.ar","","09","base_default_vat","base_default_vat","Otras percepciones"

```

## File: data\template\account.tax.group-ar_ex.csv

```csv
"id","name","l10n_ar_vat_afip_code","country_id","sequence","l10n_ar_tribute_afip_code","tax_payable_account_id","tax_receivable_account_id","name@es"
"tax_group_iva_21","VAT 21%","5","base.ar","","","base_default_vat","base_default_vat","IVA 21%"
"tax_group_iva_27","VAT 27%","6","base.ar","","","base_default_vat","base_default_vat","IVA 27%"
"tax_group_iva_105","VAT 10.5%","4","base.ar","","","base_default_vat","base_default_vat","IVA 10,5%"
"tax_group_iva_025","VAT 2.5%","9","base.ar","","","base_default_vat","base_default_vat","IVA 2,5%"
"tax_group_iva_no_corresponde","VAT Not Applicable","0","base.ar","","","base_default_vat","base_default_vat","IVA No Corresponde"
"tax_group_iva_no_gravado","VAT Untaxed","1","base.ar","","","base_default_vat","base_default_vat","IVA No Gravado"
"tax_group_iva_exento","VAT Exempt","2","base.ar","","","base_default_vat","base_default_vat","IVA Exento"
"tax_group_iva_0","VAT 0%","3","base.ar","","","base_default_vat","base_default_vat","IVA 0%"
"tax_group_iva_5","VAT 5%","8","base.ar","","","base_default_vat","base_default_vat","IVA 5%"
"tax_group_otros_impuestos","Other Taxes","","base.ar","20","99","base_default_vat","base_default_vat","Otros Impuestos"
"tax_impuestos_internos","Internal Taxes","","base.ar","15","04","base_default_vat","base_default_vat","Impuestos internos"
"tax_group_national_taxes","National Taxes","","base.ar","30","01","base_default_vat","base_default_vat","Impuestos Nacionales"
"tax_group_percepcion_iva","VAT Perception","","base.ar","","06","base_default_vat","base_default_vat","Percepción del IVA"
"tax_group_percepcion_iibb_caba","Perc IIBB CABA","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB CABA"
"tax_group_percepcion_iibb_ba","Perc IIBB ARBA","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB ARBA"
"tax_group_percepcion_iibb_ca","Perc IIBB Catamarca","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Catamarca"
"tax_group_percepcion_iibb_co","Perc IIBB Córdoba","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Córdoba"
"tax_group_percepcion_iibb_rr","Perc IIBB Corrientes","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Corrientes"
"tax_group_percepcion_iibb_er","Perc IIBB Entre Ríos","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Entre Ríos"
"tax_group_percepcion_iibb_ju","Perc IIBB Jujuy","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Jujuy"
"tax_group_percepcion_iibb_za","Perc IIBB Mendoza","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Mendoza"
"tax_group_percepcion_iibb_lr","Perc IIBB La Rioja","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB La Rioja"
"tax_group_percepcion_iibb_sa","Perc IIBB Salta","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Salta"
"tax_group_percepcion_iibb_nn","Perc IIBB San Juan","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB San Juan"
"tax_group_percepcion_iibb_sl","Perc IIBB San Luis","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB San Luis"
"tax_group_percepcion_iibb_sf","Perc IIBB Santa Fe","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Santa Fe"
"tax_group_percepcion_iibb_se","Perc IIBB Santiago del Estero","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Santiago del Estero"
"tax_group_percepcion_iibb_tn","Perc IIBB Tucumán","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Tucumán"
"tax_group_percepcion_iibb_ha","Perc IIBB Chaco","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Chaco"
"tax_group_percepcion_iibb_ct","Perc IIBB Chubut","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Chubut"
"tax_group_percepcion_iibb_fo","Perc IIBB Formosa","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Formosa"
"tax_group_percepcion_iibb_mi","Perc IIBB Misiones","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Misiones"
"tax_group_percepcion_iibb_ne","Perc IIBB Neuquén","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Neuquén"
"tax_group_percepcion_iibb_lp","Perc IIBB La Pampa","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB La Pampa"
"tax_group_percepcion_iibb_rn","Perc IIBB Río Negro","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Río Negro"
"tax_group_percepcion_iibb_az","Perc IIBB Santa Cruz","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Santa Cruz"
"tax_group_percepcion_iibb_tf","Perc IIBB Tierra del Fuego","","base.ar","","07","base_default_vat","base_default_vat","Perc IIBB Tierra del Fuego"
"tax_group_percepcion_iibb","IIBB Perceptions","","base.ar","25","07","base_default_vat","base_default_vat","IIBB Percepciones"
"tax_group_percepcion_municipal","Municipal Taxes Perceptions","","base.ar","","08","base_default_vat","base_default_vat","Percepción de los impuestos municipales"
"tax_group_percepcion_ganancias","Profit Perceptions","","base.ar","","09","base_default_vat","base_default_vat","Percepción de los beneficios"
"tax_group_otras_percepciones","Other Perceptions","","base.ar","","09","base_default_vat","base_default_vat","Otras percepciones"

```

## File: data\template\account.tax.group-ar_ri.csv

```csv
"id","name","l10n_ar_vat_afip_code","country_id","sequence","l10n_ar_tribute_afip_code","tax_payable_account_id","tax_receivable_account_id","name@es"
"tax_group_iva_21","VAT 21%","5","base.ar","","","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","IVA 21%"
"tax_group_iva_27","VAT 27%","6","base.ar","","","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","IVA 27%"
"tax_group_iva_105","VAT 10.5%","4","base.ar","","","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","IVA 10,5%"
"tax_group_iva_025","VAT 2.5%","9","base.ar","","","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","IVA 2,5%"
"tax_group_iva_no_corresponde","VAT Not Applicable","0","base.ar","","","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","IVA No Corresponde"
"tax_group_iva_no_gravado","VAT Untaxed","1","base.ar","","","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","IVA No Gravado"
"tax_group_iva_exento","VAT Exempt","2","base.ar","","","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","IVA Exento"
"tax_group_iva_0","VAT 0%","3","base.ar","","","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","IVA 0%"
"tax_group_iva_5","VAT 5%","8","base.ar","","","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","IVA 5%"
"tax_group_otros_impuestos","Other Taxes","","base.ar","20","99","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Otros Impuestos"
"tax_impuestos_internos","Internal Taxes","","base.ar","15","04","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Impuestos internos"
"tax_group_national_taxes","National Taxes","","base.ar","30","01","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Impuestos Nacionales"
"tax_group_percepcion_iva","VAT Perception","","base.ar","","06","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Percepción del IVA"
"tax_group_percepcion_iibb_caba","Perc IIBB CABA","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB CABA"
"tax_group_percepcion_iibb_ba","Perc IIBB ARBA","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB ARBA"
"tax_group_percepcion_iibb_ca","Perc IIBB Catamarca","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Catamarca"
"tax_group_percepcion_iibb_co","Perc IIBB Córdoba","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Córdoba"
"tax_group_percepcion_iibb_rr","Perc IIBB Corrientes","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Corrientes"
"tax_group_percepcion_iibb_er","Perc IIBB Entre Ríos","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Entre Ríos"
"tax_group_percepcion_iibb_ju","Perc IIBB Jujuy","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Jujuy"
"tax_group_percepcion_iibb_za","Perc IIBB Mendoza","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Mendoza"
"tax_group_percepcion_iibb_lr","Perc IIBB La Rioja","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB La Rioja"
"tax_group_percepcion_iibb_sa","Perc IIBB Salta","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Salta"
"tax_group_percepcion_iibb_nn","Perc IIBB San Juan","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB San Juan"
"tax_group_percepcion_iibb_sl","Perc IIBB San Luis","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB San Luis"
"tax_group_percepcion_iibb_sf","Perc IIBB Santa Fe","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Santa Fe"
"tax_group_percepcion_iibb_se","Perc IIBB Santiago del Estero","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Santiago del Estero"
"tax_group_percepcion_iibb_tn","Perc IIBB Tucumán","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Tucumán"
"tax_group_percepcion_iibb_ha","Perc IIBB Chaco","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Chaco"
"tax_group_percepcion_iibb_ct","Perc IIBB Chubut","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Chubut"
"tax_group_percepcion_iibb_fo","Perc IIBB Formosa","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Formosa"
"tax_group_percepcion_iibb_mi","Perc IIBB Misiones","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Misiones"
"tax_group_percepcion_iibb_ne","Perc IIBB Neuquén","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Neuquén"
"tax_group_percepcion_iibb_lp","Perc IIBB La Pampa","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB La Pampa"
"tax_group_percepcion_iibb_rn","Perc IIBB Río Negro","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Río Negro"
"tax_group_percepcion_iibb_az","Perc IIBB Santa Cruz","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Santa Cruz"
"tax_group_percepcion_iibb_tf","Perc IIBB Tierra del Fuego","","base.ar","","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Perc IIBB Tierra del Fuego"
"tax_group_percepcion_iibb","IIBB Perceptions","","base.ar","25","07","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","IIBB Percepciones"
"tax_group_percepcion_municipal","Municipal Taxes Perceptions","","base.ar","","08","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Percepción de los impuestos municipales"
"tax_group_percepcion_ganancias","Profit Perceptions","","base.ar","","09","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Percepción de los beneficios"
"tax_group_otras_percepciones","Other Perceptions","","base.ar","","09","ri_iva_saldo_a_pagar","ri_iva_saldo_tecnico_favor","Otras percepciones"

```

## File: models\account_chart_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, _
from odoo.exceptions import ValidationError
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @api.model
    def _get_ar_responsibility_match(self, chart_template):
        """ return responsibility type that match with the given chart_template code
        """
        match = {
            'ar_base': self.env.ref('l10n_ar.res_RM'),
            'ar_ex': self.env.ref('l10n_ar.res_IVAE'),
            'ar_ri': self.env.ref('l10n_ar.res_IVARI'),
        }
        return match.get(chart_template)

    def _load(self, template_code, company, install_demo,force_create=True):
        """ Set companies AFIP Responsibility and Country if AR CoA is installed, also set tax calculation rounding
        method required in order to properly validate match AFIP invoices.

        Also, raise a warning if the user is trying to install a CoA that does not match with the defined AFIP
        Responsibility defined in the company
        """
        coa_responsibility = self._get_ar_responsibility_match(template_code)
        if coa_responsibility:
            company.write({
                'l10n_ar_afip_responsibility_type_id': coa_responsibility.id,
                'country_id': self.env['res.country'].search([('code', '=', 'AR')]).id,
                'tax_calculation_rounding_method': 'round_globally',
            })

            current_identification_type = company.partner_id.l10n_latam_identification_type_id
            try:
                # set CUIT identification type (which is the argentinean vat) in the created company partner instead of
                # the default VAT type.
                company.partner_id.l10n_latam_identification_type_id = self.env.ref('l10n_ar.it_cuit')
            except ValidationError:
                # put back previous value if we could not validate the CUIT
                company.partner_id.l10n_latam_identification_type_id = current_identification_type

        res = super()._load(template_code, company, install_demo,force_create)

        # If Responsable Monotributista remove the default purchase tax
        if template_code in ('ar_base', 'ar_ex'):
            company.account_purchase_tax_id = self.env['account.tax']

        return res

    def try_loading(self, template_code, company, install_demo=False, force_create=True):
        # During company creation load template code corresponding to the AFIP Responsibility
        if not company:
            return
        if isinstance(company, int):
            company = self.env['res.company'].browse([company])
        if company.country_code == 'AR' and not company.chart_template:
            match = {
                self.env.ref('l10n_ar.res_RM'): 'ar_base',
                self.env.ref('l10n_ar.res_IVAE'): 'ar_ex',
                self.env.ref('l10n_ar.res_IVARI'): 'ar_ri',
            }
            template_code = match.get(company.l10n_ar_afip_responsibility_type_id, template_code)
        return super().try_loading(template_code, company, install_demo, force_create)

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

    def _get_fpos_ranking_functions(self, partner):
        if self.env.company.country_id.code != "AR":
            return super()._get_fpos_ranking_functions(partner)
        return [
            ('l10n_ar_afip_responsibility_type_id', lambda fpos: (
                partner.l10n_ar_afip_responsibility_type_id in fpos.l10n_ar_afip_responsibility_type_ids
            ))
        ] + super()._get_fpos_ranking_functions(partner)

```

## File: models\account_journal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import UserError, ValidationError, RedirectWarning


class AccountJournal(models.Model):

    _inherit = "account.journal"

    l10n_ar_afip_pos_system = fields.Selection(
        selection='_get_l10n_ar_afip_pos_types_selection', string='AFIP POS System',
        compute='_compute_l10n_ar_afip_pos_system', store=True, readonly=False,
        help="Argentina: Specify which type of system will be used to create the electronic invoice. This will depend on the type of invoice to be created.",
    )
    l10n_ar_afip_pos_number = fields.Integer(
        'AFIP POS Number', help='This is the point of sale number assigned by AFIP in order to generate invoices')
    company_partner = fields.Many2one('res.partner', related='company_id.partner_id')
    l10n_ar_afip_pos_partner_id = fields.Many2one(
        'res.partner', 'AFIP POS Address', help='This is the address used for invoice reports of this POS',
        domain="['|', ('id', '=', company_partner), '&', ('id', 'child_of', company_partner), ('type', '!=', 'contact')]"
    )
    l10n_ar_is_pos = fields.Boolean(
        compute="_compute_l10n_ar_is_pos", store=True, readonly=False,
        string="Is AFIP POS?",
        help="Argentina: Specify if this Journal will be used to send electronic invoices to AFIP.",
    )

    @api.depends('country_code', 'type', 'l10n_latam_use_documents')
    def _compute_l10n_ar_is_pos(self):
        for journal in self:
            journal.l10n_ar_is_pos = journal.country_code == 'AR' and journal.type == 'sale' and journal.l10n_latam_use_documents

    @api.depends('l10n_ar_is_pos')
    def _compute_l10n_ar_afip_pos_system(self):
        for journal in self:
            journal.l10n_ar_afip_pos_system = journal.l10n_ar_is_pos and journal.l10n_ar_afip_pos_system

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
                '4': ['C'],
                '5': [],
                '6': ['C', 'E'],
                '7': ['B', 'C', 'I'],
                '8': ['B', 'C', 'I'],
                '9': ['I'],
                '10': [],
                '13': ['C', 'E'],
                '15': [],
                '16': [],
            },
            'received': {
                '1': ['A', 'B', 'C', 'E', 'M', 'I'],
                '4': ['B', 'C', 'I'],
                '5': ['B', 'C', 'I'],
                '6': ['A', 'B', 'C', 'M', 'I'],
                '7': ['B', 'C', 'I'],
                '8': ['E', 'B', 'C'],
                '9': ['E', 'B', 'C'],
                '10': ['E', 'B', 'C'],
                '13': ['A', 'B', 'C', 'M', 'I'],
                '15': ['B', 'C', 'I'],
                '16': ['A', 'C', 'M'],
            },
        }
        if not self.company_id.l10n_ar_afip_responsibility_type_id:
            action = self.env.ref('base.action_res_company_form')
            msg = _('Can not create chart of account until you configure your company AFIP Responsibility and VAT.')
            raise RedirectWarning(msg, action.id, _('Go to Companies'))

        letters = letters_data['issued' if self.l10n_ar_is_pos else 'received'][
            self.company_id.l10n_ar_afip_responsibility_type_id.code]
        if counterpart_partner:
            counterpart_letters = letters_data['issued' if not self.l10n_ar_is_pos else 'received'].get(
                counterpart_partner.l10n_ar_afip_responsibility_type_id.code, [])
            letters = list(set(letters) & set(counterpart_letters))
        return letters

    def _get_journal_codes_domain(self):
        self.ensure_one()
        return self._get_codes_per_journal_type(self.l10n_ar_afip_pos_system)

    @api.model
    def _get_codes_per_journal_type(self, afip_pos_system):
        usual_codes = ['1', '2', '3', '6', '7', '8', '11', '12', '13']
        mipyme_codes = ['201', '202', '203', '206', '207', '208', '211', '212', '213']
        invoice_m_code = ['51', '52', '53']
        receipt_m_code = ['54']
        receipt_codes = ['4', '9', '15']
        expo_codes = ['19', '20', '21']
        zeta_codes = ['80', '83']
        codes_issuer_is_supplier = [
            '23', '24', '25', '26', '27', '28', '33', '43', '45', '46', '48', '58', '60', '61', '150', '151', '157',
            '158', '161', '162', '164', '166', '167', '171', '172', '180', '182', '186', '188', '332']
        codes = []
        if (self.type == 'sale' and not self.l10n_ar_is_pos) or (self.type == 'purchase' and afip_pos_system in ['II_IM', 'RLI_RLM']):
            codes = codes_issuer_is_supplier
        elif self.type == 'purchase' and afip_pos_system == 'RAW_MAW':
            # electronic invoices (wsfev1) (intersection between available docs on ws and codes_issuer_is_supplier)
            codes = ['60', '61']
        elif self.type == 'purchase':
            return [('code', 'not in', codes_issuer_is_supplier)]
        elif afip_pos_system == 'II_IM':
            # pre-printed invoice
            codes = usual_codes + receipt_codes + expo_codes + invoice_m_code + receipt_m_code
        elif afip_pos_system == 'RAW_MAW':
            # electronic/online invoice
            codes = usual_codes + receipt_codes + invoice_m_code + receipt_m_code + mipyme_codes
        elif afip_pos_system == 'RLI_RLM':
            codes = usual_codes + receipt_codes + invoice_m_code + receipt_m_code + mipyme_codes + zeta_codes
        elif afip_pos_system in ['CPERCEL', 'CPEWS']:
            # invoice with detail
            codes = usual_codes + invoice_m_code
        elif afip_pos_system in ['BFERCEL', 'BFEWS']:
            # Bonds invoice
            codes = usual_codes + mipyme_codes
        elif afip_pos_system in ['FEERCEL', 'FEEWS', 'FEERCELP']:
            codes = expo_codes
        return [('code', 'in', codes)]

    @api.constrains('l10n_ar_afip_pos_system')
    def _check_afip_pos_system(self):
        journals = self.filtered(
            lambda j: j.l10n_ar_is_pos and j.type == 'purchase' and
            j.l10n_ar_afip_pos_system not in ['II_IM', 'RLI_RLM', 'RAW_MAW'])
        if journals:
            raise ValidationError("\n".join(
                _("The pos system %(system)s can not be used on a purchase journal (id %(id)s)", system=x.l10n_ar_afip_pos_system, id=x.id)
                for x in journals
            ))

    @api.constrains('l10n_ar_afip_pos_number')
    def _check_afip_pos_number(self):
        if self.filtered(lambda j: j.l10n_ar_is_pos and j.l10n_ar_afip_pos_number == 0):
            raise ValidationError(_('Please define an AFIP POS number'))

        if self.filtered(lambda j: j.l10n_ar_is_pos and j.l10n_ar_afip_pos_number > 99999):
            raise ValidationError(_('Please define a valid AFIP POS number (5 digits max)'))

    @api.onchange('l10n_ar_afip_pos_number', 'type')
    def _onchange_set_short_name(self):
        """ Will define the AFIP POS Address field domain taking into account the company configured in the journal
        The short code of the journal only admit 5 characters, so depending on the size of the pos_number (also max 5)
        we add or not a prefix to identify sales journal.
        """
        if self.type == 'sale' and self.l10n_ar_afip_pos_number:
            self.code = "%05i" % self.l10n_ar_afip_pos_number

    def write(self, vals):
        protected_fields = ('type', 'l10n_ar_afip_pos_system', 'l10n_ar_afip_pos_number', 'l10n_latam_use_documents')
        fields_to_check = [field for field in protected_fields if field in vals]

        if fields_to_check:
            self._cr.execute("SELECT DISTINCT(journal_id) FROM account_move WHERE posted_before = True")
            res = self._cr.fetchall()
            journal_with_entry_ids = [journal_id for journal_id, in res]

            for journal in self:
                if (
                    journal.company_id.account_fiscal_country_id.code != "AR"
                    or journal.type not in ['sale', 'purchase']
                    or journal.id not in journal_with_entry_ids
                ):
                    continue

                for field in fields_to_check:
                    # Wouldn't work if there was a relational field, as we would compare an id with a recordset.
                    if vals[field] != journal[field]:
                        raise UserError(_("You can not change %s journal's configuration if it already has validated invoices", journal.name))

        return super().write(vals)

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api, _
from odoo.osv import expression
from odoo.exceptions import UserError, RedirectWarning, ValidationError
from odoo.tools.misc import formatLang
from dateutil.relativedelta import relativedelta
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

    # Mostly used on reports
    l10n_ar_afip_concept = fields.Selection(
        compute='_compute_l10n_ar_afip_concept', selection='_get_afip_invoice_concepts', string="AFIP Concept",
        help="A concept is suggested regarding the type of the products on the invoice.")
    l10n_ar_afip_service_start = fields.Date(string='AFIP Service Start Date')
    l10n_ar_afip_service_end = fields.Date(string='AFIP Service End Date')

    def _is_manual_document_number(self):
        """ Document number should be manual input by user when the journal use documents and

        * if sales journal and not a AFIP pos (liquido producto case)
        * if purchase journal and not a AFIP pos (regular case of vendor bills)

        All the other cases the number should be automatic set, wiht only one exception, for pre-printed/online AFIP
        POS type, the first numeber will be always set manually by the user and then will be computed automatically
        from there """
        if self.country_code != 'AR':
            return super()._is_manual_document_number()

        # NOTE: There is a corner case where 2 sales documents can have the same number for the same DOC from a
        # different vendor, in that case, the user can create a new Sales Liquido Producto Journal
        return self.l10n_latam_use_documents and self.journal_id.type in ['purchase', 'sale'] and \
            not self.journal_id.l10n_ar_is_pos

    @api.constrains('move_type', 'journal_id')
    def _check_moves_use_documents(self):
        """ Do not let to create not invoices entries in journals that use documents """
        not_invoices = self.filtered(lambda x: x.company_id.account_fiscal_country_id.code == "AR" and x.journal_id.type in ['sale', 'purchase'] and x.l10n_latam_use_documents and not x.is_invoice())
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
        recs_afip = self.filtered(lambda x: x.company_id.account_fiscal_country_id.code == "AR" and x.l10n_latam_use_documents)
        for rec in recs_afip:
            rec.l10n_ar_afip_concept = rec._get_concept()
        remaining = self - recs_afip
        remaining.l10n_ar_afip_concept = ''

    def _get_concept(self):
        """ Method to get the concept of the invoice considering the type of the products on the invoice """
        self.ensure_one()
        invoice_lines = self.invoice_line_ids.filtered(lambda x: x.display_type not in ('line_note', 'line_section'))
        product_types = set([x.product_id.type for x in invoice_lines if x.product_id])
        consumable = {'consu'}
        service = set(['service'])
        # on expo invoice you can mix services and products
        expo_invoice = self.l10n_latam_document_type_id.code in ['19', '20', '21']

        # WSFEX 1668 - If Expo invoice and we have a "IVA Liberado – Ley Nº 19.640" (Zona Franca) partner
        # then AFIP concept to use should be type "Others (4)"
        is_zona_franca = self.partner_id.l10n_ar_afip_responsibility_type_id == self.env.ref("l10n_ar.res_IVA_LIB")
        # Default value "product"
        afip_concept = '1'
        if expo_invoice and is_zona_franca:
            afip_concept = '4'
        elif product_types == service:
            afip_concept = '2'
        elif product_types - consumable and product_types - service and not expo_invoice:
            afip_concept = '3'
        return afip_concept

    @api.model
    def _get_l10n_ar_codes_used_for_inv_and_ref(self):
        """ List of document types that can be used as an invoice and refund. This list can be increased once needed
        and demonstrated. As far as we've checked document types of wsfev1 don't allow negative amounts so, for example
        document 61 could not be used as refunds. """
        return ['99', '186', '188', '189', '60']

    def _get_l10n_latam_documents_domain(self):
        self.ensure_one()
        domain = super()._get_l10n_latam_documents_domain()
        if self.journal_id.company_id.account_fiscal_country_id.code == "AR":
            letters = self.journal_id._get_journal_letter(counterpart_partner=self.partner_id.commercial_partner_id)
            domain += ['|', ('l10n_ar_letter', '=', False), ('l10n_ar_letter', 'in', letters)]
            domain = expression.AND([
                domain or [],
                self.journal_id._get_journal_codes_domain(),
            ])
            if self.move_type in ['out_refund', 'in_refund']:
                domain = ['|', ('code', 'in', self._get_l10n_ar_codes_used_for_inv_and_ref())] + domain
        return domain

    def _check_argentinean_invoice_taxes(self):

        # check vat on companies thats has it (Responsable inscripto)
        for inv in self.filtered(lambda x: x.company_id.l10n_ar_company_requires_vat):
            purchase_aliquots = 'not_zero'
            # we require a single vat on each invoice line except from some purchase documents
            if inv.move_type in ['in_invoice', 'in_refund'] and inv.l10n_latam_document_type_id.purchase_aliquots == 'zero':
                purchase_aliquots = 'zero'
            for line in inv.mapped('invoice_line_ids').filtered(lambda x: x.display_type not in ('line_section', 'line_note')):
                vat_taxes = line.tax_ids.filtered(lambda x: x.tax_group_id.l10n_ar_vat_afip_code)
                if len(vat_taxes) != 1:
                    raise UserError(_("There should be a single tax from the “VAT“ tax group per line, but this is not the case for line “%s”. Please add a tax to this line or check the tax configuration's advanced options for the corresponding field “Tax Group”.", line.name))

                elif purchase_aliquots == 'zero' and vat_taxes.tax_group_id.l10n_ar_vat_afip_code != '0':
                    raise UserError(_('On invoice id “%s” you must use VAT Not Applicable on every line.', inv.id))
                elif purchase_aliquots == 'not_zero' and vat_taxes.tax_group_id.l10n_ar_vat_afip_code == '0':
                    raise UserError(_('On invoice id “%s” you must use a VAT tax that is not VAT Not Applicable', inv.id))

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

    @api.onchange('partner_id')
    def _onchange_afip_responsibility(self):
        if self.company_id.account_fiscal_country_id.code == 'AR' and self.l10n_latam_use_documents and self.partner_id \
           and not self.partner_id.l10n_ar_afip_responsibility_type_id:
            return {'warning': {
                'title': _('Missing Partner Configuration'),
                'message': _('Please configure the AFIP Responsibility for "%s" in order to continue',
                    self.partner_id.name)}}

    @api.onchange('partner_id')
    def _onchange_partner_journal(self):
        """ This method is used when the invoice is created from the sale or subscription """
        expo_journals = ['FEERCEL', 'FEEWS', 'FEERCELP']
        for rec in self.filtered(lambda x: x.company_id.account_fiscal_country_id.code == "AR" and x.journal_id.type == 'sale'
                                 and x.l10n_latam_use_documents and x.partner_id.l10n_ar_afip_responsibility_type_id):
            res_code = rec.partner_id.l10n_ar_afip_responsibility_type_id.code
            domain = [
                *self.env['account.journal']._check_company_domain(rec.company_id),
                ('l10n_latam_use_documents', '=', True),
                ('type', '=', 'sale'),
            ]
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
        ar_invoices = self.filtered(lambda x: x.company_id.account_fiscal_country_id.code == "AR" and x.l10n_latam_use_documents)
        # We make validations here and not with a constraint because we want validation before sending electronic
        # data on l10n_ar_edi
        ar_invoices._check_argentinean_invoice_taxes()
        posted = super()._post(soft=soft)

        posted_ar_invoices = posted & ar_invoices
        posted_ar_invoices._set_afip_responsibility()
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

    @api.onchange('l10n_latam_document_type_id', 'l10n_latam_document_number', 'partner_id')
    def _inverse_l10n_latam_document_number(self):
        super()._inverse_l10n_latam_document_number()

        to_review = self.filtered(lambda x: (
            x.journal_id.l10n_ar_is_pos
            and x.l10n_latam_document_type_id
            and x.l10n_latam_document_number
            and (x.l10n_latam_manual_document_number or not x.highest_name)
            and x.l10n_latam_document_type_id.country_id.code == 'AR'
        ))
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
        if self.journal_id.l10n_latam_use_documents and self.company_id.account_fiscal_country_id.code == "AR":
            if self.l10n_latam_document_type_id:
                return self._get_formatted_sequence()
        return super()._get_starting_sequence()

    def _get_last_sequence_domain(self, relaxed=False):
        where_string, param = super(AccountMove, self)._get_last_sequence_domain(relaxed)
        if self.company_id.account_fiscal_country_id.code == "AR" and self.l10n_latam_use_documents:
            where_string += " AND l10n_latam_document_type_id = %(l10n_latam_document_type_id)s"
            param['l10n_latam_document_type_id'] = self.l10n_latam_document_type_id.id or 0
        return where_string, param

    def _l10n_ar_get_amounts(self, company_currency=False):
        """ Method used to prepare data to present amounts and taxes related amounts when creating an
        electronic invoice for argentinean and the txt files for digital VAT books. Only take into account the argentinean taxes """
        self.ensure_one()
        amount_field = company_currency and 'balance' or 'amount_currency'
        # if we use balance we need to correct sign (on price_subtotal is positive for refunds and invoices)
        sign = -1 if self.is_inbound() else 1

        # if we are on a document that works invoice and refund and it's a refund, we need to export it as negative
        sign = -sign if self.move_type in ('out_refund', 'in_refund') and\
            self.l10n_latam_document_type_id.code in self._get_l10n_ar_codes_used_for_inv_and_ref() else sign

        tax_lines = self.line_ids.filtered('tax_line_id')
        vat_taxes = tax_lines.filtered(lambda r: r.tax_line_id.tax_group_id.l10n_ar_vat_afip_code)

        vat_taxable = self.env['account.move.line']
        for line in self.invoice_line_ids:
            if any(tax.tax_group_id.l10n_ar_vat_afip_code and tax.tax_group_id.l10n_ar_vat_afip_code not in ['0', '1', '2'] for tax in line.tax_ids):
                vat_taxable |= line

        profits_tax_group = self.env['account.chart.template'].with_company(self.company_id).ref(
            'tax_group_percepcion_ganancias',
            raise_if_not_found=False,
        )
        if not profits_tax_group:
            raise RedirectWarning(
                message=_(
                    "A required tax group could not be found (XML ID: %s).\n"
                    "Please reload your chart template in order to reinstall the required tax group.\n\n"
                    "Note: You might have to relink your existing taxes to this new tax group.",
                    'tax_group_percepcion_ganancias',
                ),
                action=self.env.ref('account.action_account_config').id,
                button_text=_("Accounting Settings"),
            )

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

    def _get_vat(self):
        """ Applies on wsfe web service and in the VAT digital books """
        # if we are on a document that works invoice and refund and it's a refund, we need to export it as negative
        sign = -1 if self.move_type in ('out_refund', 'in_refund') and\
            self.l10n_latam_document_type_id.code in self._get_l10n_ar_codes_used_for_inv_and_ref() else 1

        res = []
        vat_taxable = self.env['account.move.line']
        # get all invoice lines that are vat taxable
        for line in self.line_ids:
            if any(tax.tax_group_id.l10n_ar_vat_afip_code and tax.tax_group_id.l10n_ar_vat_afip_code not in ['0', '1', '2'] for tax in line.tax_line_id) and line['amount_currency']:
                vat_taxable |= line
        for tax_group in vat_taxable.mapped('tax_group_id'):
            base_imp = sum(self.invoice_line_ids.filtered(lambda x: x.tax_ids.filtered(lambda y: y.tax_group_id.l10n_ar_vat_afip_code == tax_group.l10n_ar_vat_afip_code)).mapped('price_subtotal'))
            imp = abs(sum(vat_taxable.filtered(lambda x: x.tax_group_id.l10n_ar_vat_afip_code == tax_group.l10n_ar_vat_afip_code).mapped('amount_currency')))
            res += [{'Id': tax_group.l10n_ar_vat_afip_code,
                     'BaseImp': sign * base_imp,
                     'Importe': sign * imp}]

        # Report vat 0%
        vat_base_0 = sum(self.invoice_line_ids.filtered(lambda x: x.tax_ids.filtered(lambda y: y.tax_group_id.l10n_ar_vat_afip_code == '3')).mapped('price_subtotal'))
        if vat_base_0:
            res += [{'Id': '3', 'BaseImp': sign * vat_base_0, 'Importe': 0.0}]

        return res if res else []

    def _get_name_invoice_report(self):
        self.ensure_one()
        if self.l10n_latam_use_documents and self.company_id.account_fiscal_country_id.code == 'AR':
            return 'l10n_ar.report_invoice_document'
        return super()._get_name_invoice_report()

    def _l10n_ar_get_invoice_totals_for_report(self):
        """If the invoice document type indicates that vat should not be detailed in the printed report (result of _l10n_ar_include_vat()) then we overwrite tax_totals field so that includes taxes in the total amount, otherwise it would be showing amount_untaxed in the amount_total"""
        self.ensure_one()
        tax_totals = self.tax_totals
        include_vat = self._l10n_ar_include_vat()
        if not include_vat:
            return tax_totals

        tax_group_ids = {
            tax_group['id']
            for subtotal in tax_totals['subtotals']
            for tax_group in subtotal['tax_groups']
        }
        tax_group_ids_to_exclude = self.env['account.tax.group']\
            .browse(tax_group_ids)\
            .filtered(lambda tax_group: (
                self._l10n_ar_is_tax_group_other_national_ind_tax(tax_group)
                or self._l10n_ar_is_tax_group_vat(tax_group)
            )).ids
        if tax_group_ids_to_exclude:
            tax_totals = self.env['account.tax']._exclude_tax_groups_from_tax_totals_summary(tax_totals, tax_group_ids_to_exclude)
        return tax_totals

    def _l10n_ar_get_invoice_custom_tax_summary_for_report(self):
        """ Get a new tax details for RG 5614/2024 to show ARCA VAT and Other National Internal Taxes. """
        if self.l10n_latam_document_type_id.code not in ('6', '7', '8'):
            return []

        base_lines, _tax_lines = self._get_rounded_base_and_tax_lines()

        def grouping_function(base_line, tax_data):
            tax_group = tax_data['tax'].tax_group_id
            skip = False
            name = None
            if self._l10n_ar_is_tax_group_other_national_ind_tax(tax_group):
                name = _("Other National Ind. Taxes %s", base_line['currency_id'].symbol)
            elif self._l10n_ar_is_tax_group_vat(tax_group):
                name = _("VAT Content %s", base_line['currency_id'].symbol)
            else:
                skip = True
            return {
                'name': name,
                'skip': skip,
            }

        AccountTax = self.env['account.tax']
        base_lines_aggregated_values = AccountTax._aggregate_base_lines_tax_details(base_lines, grouping_function)
        values_per_grouping_key = AccountTax._aggregate_base_lines_aggregated_values(base_lines_aggregated_values)
        results = []
        for grouping_key, values in values_per_grouping_key.items():
            if (
                grouping_key
                and not grouping_key['skip']
                and not self.currency_id.is_zero(values['tax_amount_currency'])
            ):
                results.append({
                    'name': grouping_key['name'],
                    'tax_amount_currency': values['tax_amount_currency'],
                    'formatted_tax_amount_currency': formatLang(self.env, values['tax_amount_currency']),
                })
        return results

    def _l10n_ar_include_vat(self):
        self.ensure_one()
        return self.l10n_latam_document_type_id.l10n_ar_letter in ['B', 'C', 'X', 'R']

    @api.model
    def _l10n_ar_is_tax_group_other_national_ind_tax(self, tax_group):
        return tax_group.l10n_ar_tribute_afip_code in ('01', '04')

    @api.model
    def _l10n_ar_is_tax_group_vat(self, tax_group):
        return bool(tax_group.l10n_ar_vat_afip_code)

```

## File: models\account_move_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountMoveLine(models.Model):
    _inherit = 'account.move.line'

    def _l10n_ar_prices_and_taxes(self):
        self.ensure_one()
        invoice = self.move_id
        include_vat = invoice._l10n_ar_include_vat()

        AccountTax = self.env['account.tax']
        base_line = invoice._prepare_product_base_line_for_taxes_computation(self)
        if include_vat:
            base_line['tax_ids'] = self.tax_ids.filtered('tax_group_id.l10n_ar_vat_afip_code')
        AccountTax._add_tax_details_in_base_line(base_line, self.company_id, rounding_method='round_globally')

        tax_details = base_line['tax_details']
        discount = base_line['discount']
        price_unit = base_line['price_unit']
        quantity = base_line['quantity']
        if include_vat:
            raw_total = tax_details['raw_total_included_currency']
        else:
            raw_total = tax_details['raw_total_excluded_currency']

        if discount == 100.0:
            price_subtotal_before_discount = price_unit * quantity
        else:
            price_subtotal_before_discount = raw_total / (1 - discount / 100.0)

        if quantity:
            price_unit = price_subtotal_before_discount / quantity
            price_net = raw_total / quantity
        else:
            price_unit = 0.0
            price_net = 0.0

        return {
            'price_unit': price_unit,
            'price_subtotal': invoice.currency_id.round(raw_total),
            'price_net': price_net,
        }

```

## File: models\account_tax_group.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import _, api, fields, models
from odoo.exceptions import UserError


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

    @api.ondelete(at_uninstall=False)
    def check_uninstall_required(self):
        """
        Make sure we don't uninstall a required tax group
        """
        ar_companies = self.filtered(lambda g: g.company_id.chart_template.startswith('ar_')).mapped('company_id')
        profits_tax_group_ids = self.env['ir.model.data'].search([
            ('name', 'in', [f'{company.id}_tax_group_percepcion_ganancias' for company in ar_companies]),
            ('module', '=', 'account'),
        ]).mapped('res_id')
        if profit_tax_groups_to_be_deleted := self.filtered(lambda g: g.id in profits_tax_group_ids):
            raise UserError(
                _(
                    "The tax group '%s' can't be removed, since it is required in the Argentinian localization.",
                    profit_tax_groups_to_be_deleted[0].name,
                )
            )

```

## File: models\l10n_ar_afip_responsibility_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class L10nArAfipResponsibilityType(models.Model):

    _name = 'l10n_ar.afip.responsibility.type'
    _description = 'AFIP Responsibility Type'
    _order = 'sequence'

    name = fields.Char(required=True, index='trigram')
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

        if not self.code:
            return document_number

        # Import Dispatch Number Validator
        if self.code in ['66', '67']:
            if len(document_number) != 16:
                raise UserError(
                    _(
                        "%(value)s is not a valid value for %(field)s.\nThe number of import Dispatch must be 16 characters.",
                        value=document_number,
                        field=self.name,
                    ),
                )
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
            raise UserError(
                _(
                    "%(value)s is not a valid value for %(field)s.\nThe document number must be entered with a dash (-) and a maximum of 5 characters for the first part and 8 for the second. The following are examples of valid numbers:\n* 1-1\n* 0001-00000001\n* 00001-00000001",
                    value=document_number,
                    field=self.name,
                ),
            )

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
from odoo.exceptions import UserError

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
        """ Argentinean companies use round_globally as tax_calculation_rounding_method """
        for rec in self.filtered(lambda x: x.country_id.code == "AR"):
            rec.tax_calculation_rounding_method = 'round_globally'

    @api.depends('l10n_ar_afip_responsibility_type_id')
    def _compute_l10n_ar_company_requires_vat(self):
        recs_requires_vat = self.filtered(lambda x: x.l10n_ar_afip_responsibility_type_id.code == '1')
        recs_requires_vat.l10n_ar_company_requires_vat = True
        remaining = self - recs_requires_vat
        remaining.l10n_ar_company_requires_vat = False

    def _localization_use_documents(self):
        """ Argentinean localization use documents """
        self.ensure_one()
        return self.account_fiscal_country_id.code == "AR" or super()._localization_use_documents()

    def write(self, vals):
        if 'l10n_ar_afip_responsibility_type_id' in vals:
            for company in self:
                if vals['l10n_ar_afip_responsibility_type_id'] != company.l10n_ar_afip_responsibility_type_id.id and company.sudo()._existing_accounting():
                    raise UserError(_('Could not change the AFIP Responsibility of this company because there are already accounting entries.'))

        return super().write(vals)

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
        'Gross Income Type', help='Argentina: Type of gross income: exempt, local, multilateral.')
    l10n_ar_afip_responsibility_type_id = fields.Many2one(
        'l10n_ar.afip.responsibility.type', string='AFIP Responsibility Type', index='btree_not_null', help='Defined by AFIP to'
        ' identify the type of responsibilities that a person or a legal entity could have and that impacts in the'
        ' type of operations and requirements they need.')

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
                _logger.runbot("Argentinean VAT was not formatted: %s", repr(error))
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
        """ Since we validate more documents than the vat for Argentinean partners (CUIT - VAT AR, CUIL, DNI) we
        extend this method in order to process it. """
        # NOTE by the moment we include the CUIT (VAT AR) validation also here because we extend the messages
        # errors to be more friendly to the user. In a future when Odoo improve the base_vat message errors
        # we can change this method and use the base_vat.check_vat_ar method.s
        l10n_ar_partners = self.filtered(lambda p: p.l10n_latam_identification_type_id.l10n_ar_afip_code or p.country_code == 'AR')
        l10n_ar_partners.l10n_ar_identification_validation()
        return super(ResPartner, self - l10n_ar_partners).check_vat()

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['l10n_ar_afip_responsibility_type_id']

    def ensure_vat(self):
        """ This method is a helper that returns the VAT number is this one is defined if not raise an UserError.

        VAT is not mandatory field but for some Argentinean operations the VAT is required, for eg  validate an
        electronic invoice, build a report, etc.

        This method can be used to validate is the VAT is proper defined in the partner """
        self.ensure_one()
        if not self.l10n_ar_vat:
            raise UserError(_('No VAT configured for partner [%i] %s', self.id, self.name))
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
                _logger.runbot("Argentinean document was not validated: %s", repr(error))

            if not module:
                continue
            try:
                module.validate(rec.vat)
            except module.InvalidChecksum:
                raise ValidationError(_('The validation digit is not valid for "%s"',
                                        rec.l10n_latam_identification_type_id.name))
            except module.InvalidLength:
                raise ValidationError(_('Invalid length for "%s"', rec.l10n_latam_identification_type_id.name))
            except module.InvalidFormat:
                raise ValidationError(_('Only numbers allowed for "%s"', rec.l10n_latam_identification_type_id.name))
            except module.InvalidComponent:
                valid_cuit = ('20', '23', '24', '27', '30', '33', '34', '50', '51', '55')
                raise ValidationError(_('CUIT number must be prefixed with one of the following: %s', ', '.join(valid_cuit)))
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
            raise ValidationError(_('Invalid Length'))
        if not number.isdigit():
            raise ValidationError(_('Invalid Format'))
        if _check_digit(number[:7]) != number[7]:
            raise ValidationError(_('Invalid Checksum'))
        if _check_digit(number[8:-1]) != number[-1]:
            raise ValidationError(_('Invalid Checksum'))
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

## File: models\template_ar_base.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ar_base')
    def _get_ar_base_template_data(self):
        return {
            'property_account_receivable_id': 'base_deudores_por_ventas',
            'property_account_payable_id': 'base_proveedores',
            'property_account_expense_categ_id': 'base_compra_mercaderia',
            'property_account_income_categ_id': 'base_venta_de_mercaderia',
            'name': _('Generic Chart of Accounts Argentina Single Taxpayer / Basis'),
            'code_digits': '12',
        }

    @template('ar_base', 'res.company')
    def _get_ar_base_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.ar',
                'bank_account_code_prefix': '1.1.1.02.',
                'cash_account_code_prefix': '1.1.1.01.',
                'transfer_account_code_prefix': '6.0.00.00.',
                'account_default_pos_receivable_account_id': 'base_deudores_por_ventas_pos',
                'income_currency_exchange_account_id': 'base_diferencias_de_cambio',
                'expense_currency_exchange_account_id': 'base_diferencias_de_cambio',
            },
        }

    @template('ar_base', 'account.journal')
    def _get_ar_account_journal(self):
        """ In case of an Argentinean CoA, we modify the default values of the sales journal to be a preprinted journal"""
        return {
            'sale': {
                "name": "Ventas Preimpreso",
                "code": "0001",
                "l10n_ar_afip_pos_number": 1,
                "l10n_ar_afip_pos_partner_id": self.env.company.partner_id.id,
                "l10n_ar_afip_pos_system": 'II_IM',
                "refund_sequence": False,
            },
        }

```

## File: models\template_ar_ex.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ar_ex')
    def _get_ar_ex_template_data(self):
        return {
            'name': _('Argentine Generic Chart of Accounts for Exempt Individuals'),
            'parent': 'ar_base',
            'code_digits': '12',
        }

    @template('ar_ex', 'res.company')
    def _get_ar_ex_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.ar',
                'bank_account_code_prefix': '1.1.1.02.',
                'cash_account_code_prefix': '1.1.1.01.',
                'transfer_account_code_prefix': '6.0.00.00.',
                'account_default_pos_receivable_account_id': 'base_deudores_por_ventas_pos',
                'income_currency_exchange_account_id': 'base_diferencias_de_cambio',
                'expense_currency_exchange_account_id': 'base_diferencias_de_cambio',
            },
        }

```

## File: models\template_ar_ri.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ar_ri')
    def _get_ar_ri_template_data(self):
        return {
            'name': _('Argentine Generic Chart of Accounts for Registered Accountants'),
            'parent': 'ar_ex',
            'code_digits': '12',
        }

    @template('ar_ri', 'res.company')
    def _get_ar_ri_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.ar',
                'bank_account_code_prefix': '1.1.1.02.',
                'cash_account_code_prefix': '1.1.1.01.',
                'transfer_account_code_prefix': '6.0.00.00.',
                'account_default_pos_receivable_account_id': 'base_deudores_por_ventas_pos',
                'income_currency_exchange_account_id': 'base_diferencias_de_cambio',
                'expense_currency_exchange_account_id': 'base_diferencias_de_cambio',
                'account_sale_tax_id': 'ri_tax_vat_21_ventas',
                'account_purchase_tax_id': 'ri_tax_vat_21_compras',
            },
        }

```

## File: models\uom_uom.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class Uom(models.Model):

    _inherit = 'uom.uom'

    l10n_ar_afip_code = fields.Char('Code', help='Argentina: This code will be used on electronic invoice.')

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ar_ex
from . import template_ar_ri
from . import template_ar_base
from . import l10n_latam_identification_type
from . import l10n_ar_afip_responsibility_type
from . import account_journal
from . import account_tax_group
from . import account_fiscal_position
from . import l10n_latam_document_type
from . import res_partner
from . import res_country
from . import res_currency
from . import res_company
from . import res_partner_bank
from . import uom_uom
from . import account_chart_template
from . import account_move
from . import account_move_line

```

## File: report\invoice_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields
from odoo.tools import SQL


class AccountInvoiceReport(models.Model):

    _inherit = 'account.invoice.report'

    l10n_ar_state_id = fields.Many2one('res.country.state', 'Delivery Province', readonly=True)
    date = fields.Date(readonly=True, string="Accounting Date")

    _depends = {
        'account.move': ['partner_shipping_id', 'date'],
        'res.partner': ['state_id'],
    }

    def _select(self) -> SQL:
        return SQL("%s, contact_partner.state_id as l10n_ar_state_id, move.date",
                   super()._select())

    def _from(self) -> SQL:
        return SQL("%s LEFT JOIN res_partner contact_partner ON contact_partner.id = COALESCE(move.partner_shipping_id, move.partner_id)",
                   super()._from())

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
access_l10n_ar_afip_responsibility_type_all,l10n_ar.afip.responsibility.type.all,model_l10n_ar_afip_responsibility_type,base.group_user,1,0,0,0
access_l10n_ar_afip_responsibility_type_portal,l10n_ar.afip.responsibility.type.portal,model_l10n_ar_afip_responsibility_type,base.group_portal,1,0,0,0
access_l10n_ar_afip_responsibility_type_public,l10n_ar.afip.responsibility.type.public,model_l10n_ar_afip_responsibility_type,base.group_public,1,0,0,0

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
                <field name="l10n_ar_afip_responsibility_type_ids" options="{'no_open': True, 'no_create': True}" invisible="'AR' not in fiscal_country_codes or not auto_apply" groups="base.group_no_one" widget="many2many_tags"/>
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
                <field name="l10n_ar_is_pos" invisible="country_code != 'AR' or not l10n_latam_use_documents or type not in ['sale', 'purchase']"/>
                <field name="company_partner" invisible="1"/>
                <field name="l10n_ar_afip_pos_system" invisible="not l10n_ar_is_pos" required="l10n_ar_is_pos"/>
                <field name="l10n_ar_afip_pos_number" invisible="not l10n_ar_is_pos" required="l10n_ar_is_pos"/>
                <field name="l10n_ar_afip_pos_partner_id" invisible="not l10n_ar_is_pos" required="l10n_ar_is_pos"/>
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
            <group name="sale_info_group" position="inside">
                <field name='l10n_ar_afip_concept' invisible="not l10n_latam_use_documents or country_code != 'AR'"/>
                <label for="l10n_ar_afip_service_start" invisible="l10n_ar_afip_concept not in ('2', '3', '4') or country_code != 'AR'" string="Service Date"/>
                <div invisible="l10n_ar_afip_concept not in ('2', '3', '4') or country_code != 'AR'">
                    <field name="l10n_ar_afip_service_start" class="oe_inline" readonly="state != 'draft'"/> to <field name="l10n_ar_afip_service_end" class="oe_inline" readonly="state != 'draft'"/>
                </div>
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
        <field name="name">afip.responsibility.type.list</field>
        <field name="model">l10n_ar.afip.responsibility.type</field>
        <field name="arch" type="xml">
            <list string="AFIP Responsibility Type" decoration-muted="(not active)">
                <field name="name"/>
                <field name="code"/>
                <field name='active'/>
            </list>
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
        <field name="name">l10n_latam.document.type.list</field>
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
                <filter string="Argentinean Documents" name="localization" domain="[('country_id', '=', %(base.ar)d)]"/>
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
        <div class="mb-3">
            <div class="row">
                <div name="left-upper-side" class="col-5" t-if="not pre_printed_report">
                    <img t-if="o.company_id.logo" t-att-src="image_data_uri(o.company_id.logo)" style="max-height: 45px;" alt="Logo"/>
                </div>
                <div name="center-upper" class="col-2 text-center" t-att-style="'color: %s;' % o.company_id.primary_color">
                    <span style="display: inline-block; text-align: center; line-height: 8px;">
                        <h1 style="line-height: 35px;">
                            <strong><span t-out="not pre_printed_report and document_letter or '&#160;'"/></strong>
                        </h1>
                        <span style="font-size: x-small;" t-out="not pre_printed_report and document_legend or '&#160;'"/>
                    </span>
                </div>
                <div name="right-upper-side" class="col-5 text-end" style="padding-left: 0px;" t-if="not pre_printed_report">

                    <!-- (6) Titulo de Documento -->
                    <h4 t-att-style="'color: %s;' % o.company_id.primary_color"><strong>
                        <span t-out="report_name"/>
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
                        <t t-out="' - '.join([item for item in [
                            ', '.join([item for item in [header_address.street, header_address.street2] if item]),
                            header_address.city,
                            header_address.state_id and header_address.state_id.name,
                            header_address.zip,
                            header_address.country_id and header_address.country_id.name] if item])"/><span t-if="header_address.phone"> - </span><span t-if="header_address.phone" style="white-space: nowrap;" t-out="'Tel: ' + header_address.phone"/>
                        <br/>
                        <span t-att-style="'color: %s;' % o.company_id.primary_color" t-out="' - '.join([item for item in [(header_address.website or '').replace('https://', '').replace('http://', ''), header_address.email] if item])"/>
                    </t>
                </div>
                <div class="col-6 text-end" style="padding-left: 0px;">

                    <t t-if="not pre_printed_report">
                        <!-- (7) Numero punto venta - (8) numero de documento -->
                        <span t-att-style="'color: %s;' % o.company_id.secondary_color">Nro: </span><span t-out="report_number"/>
                    </t>
                    <br/>

                    <!-- (9) Fecha -->
                    <span t-att-style="'color: %s;' % o.company_id.secondary_color">Date: </span><span t-out="report_date" t-options='{"widget": "date"}'/>

                    <t t-if="not pre_printed_report">
                        <!-- (5) Condicion de IVA / Responsabilidad -->
                        <!-- (10) CUIT -->
                        <br/>
                        <span t-field="o.company_id.l10n_ar_afip_responsibility_type_id"/><span t-att-style="'color: %s;' % o.company_id.secondary_color"> - CUIT: </span><span t-field="o.company_id.partner_id.l10n_ar_formatted_vat"/>

                        <!-- (11) IIBB: -->
                        <!-- (12) Inicio de actividades -->
                        <br/><span t-att-style="'color: %s;' % o.company_id.secondary_color">IIBB: </span><span t-out="o.company_id.l10n_ar_gross_income_type == 'exempt' and 'Exento' or o.company_id.l10n_ar_gross_income_number"/><span t-att-style="'color: %s;' % o.company_id.secondary_color"> - Activities Start: </span><span t-field="o.company_id.l10n_ar_afip_start_date"/>
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
                    <div name="footer_left_column" class="col-8 text-start">
                    </div>
                    <div name="footer_right_column" class="col-4 text-end">
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
        <xpath expr="//div[@name='address_not_same_as_shipping']" position="replace">
            <div name="address_not_same_as_shipping"/>
        </xpath>
        <xpath expr="//div[@name='address_same_as_shipping']" position="replace">
            <div name="address_same_as_shipping"/>
        </xpath>
        <xpath expr="//div[@name='no_shipping']" position="replace">
            <div name="no_shipping"/>
        </xpath>

        <!-- remove default document title -->
        <xpath expr="//t[@t-set='layout_document_title']" position="replace"/>

        <!-- remove detail of taxes when currency != from company's currency -->
        <t t-call="account.document_tax_totals_company_currency_template" position="replace"/>

        <!-- NCM column for fiscal bond -->
        <th name="th_description" position="after">
            <th t-if="fiscal_bond" name="th_ncm_code" class="text-start"><span>NCM</span></th>
        </th>
        <td name="account_invoice_line_name" position="after">
            <td t-if="fiscal_bond" name="ncm_code"><span t-field="line.product_id.l10n_ar_ncm_code"/></td>
        </td>

        <!-- use latam prices (to include/exclude VAT) -->
        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" position="before">
            <t t-set="l10n_ar_values" t-value="line._l10n_ar_prices_and_taxes()"/>
        </t>
        <xpath expr="//span[@t-field='line.price_unit']" position="attributes">
            <attribute name="t-field"></attribute>
            <attribute name="t-out">l10n_ar_values['price_unit']</attribute>
            <attribute name="t-options">{"widget": "float", "display_currency": o.currency_id, "decimal_precision": "Product Price"}</attribute>
        </xpath>
        <t t-set="current_subtotal" t-value="current_subtotal + line.price_subtotal" position="attributes">
            <attribute name="t-value">current_subtotal + l10n_ar_values['price_subtotal']</attribute>
        </t>
        <!-- if b2c we still wants the latam subtotal -->
        <t t-set="current_total" t-value="current_total + line.price_total" position="attributes">
            <attribute name="t-value">current_subtotal + l10n_ar_values['price_subtotal']</attribute>
        </t>
        <!-- label amount for subtotal column on b2b and b2c -->
        <xpath expr="//th[@name='th_subtotal']/span" position="replace">
            <span>Amount</span>
        </xpath>
        <span t-field="line.price_subtotal" position="attributes">
            <attribute name="t-field"></attribute>
            <attribute name="t-out">l10n_ar_values['price_subtotal']</attribute>
            <attribute name="t-options">{"widget": "monetary", "display_currency": o.currency_id}</attribute>
        </span>

        <t t-set="tax_totals" position="attributes">
            <attribute name="t-value">o._l10n_ar_get_invoice_totals_for_report()</attribute>
        </t>

        <!-- use column vat instead of taxes and only if vat discriminated -->
        <xpath expr="//th[@name='th_taxes']" position="replace">
            <th name="th_taxes"
                t-attf-class="text-start {{ 'd-none d-md-table-cell' if report_type == 'html' else '' }}"
                t-if="not o._l10n_ar_include_vat()">
                <span t-if="o.company_id.country_id.code == 'AR'">% VAT</span>
                <span t-else="">Taxes</span>
            </th>
        </xpath>

        <xpath expr="//span[@id='line_tax_ids']/.." position="attributes">
            <attribute name="t-if">not o._l10n_ar_include_vat()</attribute>
        </xpath>
        <span id="line_tax_ids" position="attributes">
            <attribute name="t-out">', '.join(map(lambda x: (x.invoice_label or x.name), line.tax_ids.filtered(lambda x: x.tax_group_id.l10n_ar_vat_afip_code)))</attribute>
        </span>

        <!-- remove payment reference that is not used in Argentina -->
        <xpath expr="//span[@t-field='o.payment_reference']/../.." position="replace"/>

        <!-- replace information section and usage argentinean style -->
        <div id="informations" position="replace">
            <div id="informations" class="row mt8 mb8">
                <div class="col-6">

                    <!-- IDENTIFICACION (ADQUIRIENTE-LOCATARIO-PRESTARIO) -->

                    <!-- (14) Apellido uy Nombre: Denominicacion o Razon Soclial -->
                    <t t-if="o.is_sale_document(include_receipts=True)"><strong>Customer: </strong></t>
                    <t t-else=""><strong>Supplier: </strong></t>
                    <span t-field="o.partner_id.commercial_partner_id.name"/>

                    <!-- (15) Domicilio Comercial -->
                    <br/>
                    <span t-field="o.partner_id" t-options="{'widget': 'contact', 'fields': ['address'], 'no_marker': true, 'no_tag_br': True}"/>

                    <!-- (16) Responsabilidad AFIP -->
                    <strong>VAT Cond: </strong><span t-field="o.partner_id.l10n_ar_afip_responsibility_type_id"/>

                    <!-- (17) CUIT -->
                    <t t-if="o.partner_id.vat and o.partner_id.l10n_latam_identification_type_id and o.partner_id.l10n_latam_identification_type_id.l10n_ar_afip_code != '99'">
                        <br/><strong><t t-out="o.partner_id.l10n_latam_identification_type_id.name or o.company_id.account_fiscal_country_id.vat_label" id="inv_tax_id_label"/>:</strong> <span t-out="o.partner_id.l10n_ar_formatted_vat if o.partner_id.l10n_ar_vat else o.partner_id.vat"/>
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
                        <br/>
                        <strong>Incoterm:</strong>
                        <p t-if="o.incoterm_location">
                            <span t-field="o.invoice_incoterm_id.code"/> <br/>
                            <span t-field="o.incoterm_location"/>
                        </p>
                        <p t-else="" t-field="o.invoice_incoterm_id.name" class="m-0"/>
                    </t>

                </div>

            </div>
        </div>

        <xpath expr="//div[@id='payment_term']" position="before">
            <div class="mb-4">
                <t t-if="o.l10n_ar_afip_concept in ['2', '3', '4'] and o.l10n_ar_afip_service_start and o.l10n_ar_afip_service_end">
                    <strong>Invoiced period: </strong><span t-field="o.l10n_ar_afip_service_start"/> to <span t-field="o.l10n_ar_afip_service_end"/>
                </t>
                <t t-if="o.currency_id != o.company_id.currency_id">
                    <br/><strong>Currency: </strong><span t-out="'%s - %s' % (o.currency_id.name, o.currency_id.currency_unit_label)"/>
                    <br/><span>1 <t t-out="o.currency_id.name"/> = <t t-out="1 / o.invoice_currency_rate" t-options='{"widget": "float", "precision": 2}'/> <t t-out="o.company_currency_id.name"/></span>
                </t>
                <!-- Show CBU for FACTURA DE CREDITO ELECTRONICA MiPyMEs and NOTA DE DEBITO ELECTRONICA MiPyMEs -->
                <t t-if="o.l10n_latam_document_type_id.code in ['201', '206', '211', '202', '207', '212'] and o.partner_bank_id">
                    <br/><strong>CBU for payment: </strong><span t-out="o.partner_bank_id.acc_number or '' if o.partner_bank_id.acc_type == 'cbu' else ''"/>
                </t>

            </div>
        </xpath>

        <!-- Show total amount in letters for MiPyMEs document types according to the law
         http://biblioteca.afip.gob.ar/dcp/LEY_C_027440_2018_05_09 article 5.f -->
        <xpath expr="//div[@id='total']/div/table" position="after">
            <t t-if="o.l10n_latam_document_type_id.code in ['201', '202', '203', '206', '207', '208', '211', '212', '213']">
                <strong>Son: </strong><span t-out="o.currency_id.with_context(lang='es_AR').amount_to_text(o.amount_total)"/>
            </t>
        </xpath>

        <!-- RG 5003: Add legend for 'A' documents that have a Monotribuista receptor -->
        <div name="comment" position="after">
            <p t-if="o.partner_id.l10n_ar_afip_responsibility_type_id.code in ['6', '13'] and o.l10n_latam_document_type_id.l10n_ar_letter == 'A'" >
                The tax credit specified in this voucher may only be computed for purposes of the Tax Support and Inclusion Regime for Small Taxpayers of Law No. 27,618.
            </p>
        </div>

        <t t-call="account.document_tax_totals" position="attributes">
            <attribute name="t-call">l10n_ar.document_tax_totals</attribute>
        </t>

        <div id="qrcode" position="after">
            <!-- RG 5614/2024: Show ARCA VAT and Other National Internal Taxes -->
            <t t-set="l10n_ar_custom_tax_summary" t-value="o._l10n_ar_get_invoice_custom_tax_summary_for_report()"/>
            <div class="l10n_ar_tax_details" t-if="l10n_ar_custom_tax_summary">
                <table class="table table-sm table-borderless" style="page-break-inside: avoid;">
                    <th class="border-black" style="border-bottom: 1px solid" colspan="2">
                           Fiscal Transparency Regime for the Final Consumer (Law 27.743)
                    </th>
                    <t t-foreach="l10n_ar_custom_tax_summary" t-as="detail">
                        <tr>
                            <td class="text-end"><strong t-esc="detail['name']"/></td>
                            <td class="text-end">
                                <span
                                    class="oe_subtotal_footer_separator"
                                    t-out="detail['formatted_tax_amount_currency']"
                                />
                            </td>
                        </tr>
                    </t>
                </table>
            </div>
        </div>

    </template>

    <template id="document_tax_totals" inherit_id="account.document_tax_totals" primary="True">
        <xpath expr="//t[@t-foreach]/tr" position="attributes">
            <!-- Only for the Untaxed Amount  -->
            <attribute name="t-if">not o._l10n_ar_include_vat()</attribute>
        </xpath>
    </template>

    <!-- Workaround for Studio reports, see odoo/odoo#60660 -->
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-elif="o._get_name_invoice_report() == 'l10n_ar.report_invoice_document'"
               t-call="l10n_ar.report_invoice_document"
               t-lang="lang"/>
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
                <field name="l10n_ar_afip_responsibility_type_id" options="{'no_open': True, 'no_create': True}" invisible="country_code != 'AR'"/>
                <label for="l10n_ar_gross_income_number" string="Gross Income" invisible="country_code != 'AR'"/>
                <div invisible="country_code != 'AR'" name="gross_income">
                    <field name="l10n_ar_gross_income_type" class="oe_inline"/>
                    <field name="l10n_ar_gross_income_number" placeholder="Number..." class="oe_inline" invisible="l10n_ar_gross_income_type in [False, 'exempt']" required="l10n_ar_gross_income_type not in [False, 'exempt']"/>
                </div>
                <field name="l10n_ar_afip_start_date" invisible="country_code != 'AR'"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="res_config_settings_view_form">
        <field name="name">res.config.settings.view.form</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//app[@name='account']/block" position="after">
                <div  id="argentina_localization_section" invisible="1">
                    <block title="Argentinean Localization" id="argentina_localization" invisible="country_code != 'AR'">
                    </block>
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
                <field name="l10n_ar_afip_code" groups="base.group_no_one"/>
                <field name="l10n_ar_natural_vat"/>
                <field name="l10n_ar_legal_entity_vat"/>
                <field name="l10n_ar_other_vat"/>
            </field>
        </field>
    </record>

    <record id="view_res_country_tree" model="ir.ui.view">
        <field name="name">res.country.list</field>
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
                <field name="l10n_ar_afip_code" invisible="'AR' not in fiscal_country_codes"/>
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
        <field name="inherit_id" ref="l10n_latam_base.view_partner_latam_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']/.." position="after">
                <field name="l10n_ar_afip_responsibility_type_id"
                       invisible="'AR' not in fiscal_country_codes"
                       options="{'no_open': True, 'no_create': True}"
                       readonly="parent_id"/>
            </xpath>
        </field>
    </record>

    <record id="view_partner_property_form" model="ir.ui.view">
        <field name="name">res.partner.form</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">

            <field name="property_account_position_id" position="after">
                <label for="l10n_ar_gross_income_type" string="Gross Income" invisible="'AR' not in fiscal_country_codes"/>
                <div name="gross_income" invisible="'AR'not in fiscal_country_codes">
                    <field name="l10n_ar_gross_income_type" class="oe_inline"/>
                    <field name="l10n_ar_gross_income_number" placeholder="Number..." class="oe_inline" invisible="l10n_ar_gross_income_type not in ['multilateral', 'local']" required="l10n_ar_gross_income_type in ['multilateral', 'local']"/>
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
        <field name="name">uom.uom.list</field>
        <field name="model">uom.uom</field>
        <field name="inherit_id" ref="uom.product_uom_tree_view"/>
        <field name="arch" type="xml">
            <list>
                <field name="l10n_ar_afip_code" optional="hide"/>
            </list>
        </field>
    </record>

    <record id="product_uom_form_view" model="ir.ui.view">
        <field name="name">uom.uom.form</field>
        <field name="model">uom.uom</field>
        <field name="inherit_id" ref="uom.product_uom_form_view"/>
        <field name="arch" type="xml">
            <field name="rounding" position="after">
                <field name="l10n_ar_afip_code" invisible="'AR' not in fiscal_country_codes"/>
            </field>
        </field>
    </record>

    <record id="product_uom_categ_form_view" model="ir.ui.view">
        <field name="name">uom.category.form</field>
        <field name="model">uom.category</field>
        <field name="inherit_id" ref="uom.product_uom_categ_form_view"/>
        <field name="arch" type="xml">
            <field name="rounding" position="after">
                <field name="l10n_ar_afip_code" optional="hide"/>
            </field>
        </field>
    </record>

</odoo>

```

