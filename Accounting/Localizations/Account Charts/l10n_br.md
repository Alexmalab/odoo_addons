# Odoo Module: l10n_br

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import demo
from . import models
from . import wizard

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Brazilian - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/brazil.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['br'],
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Base module for the Brazilian localization
==========================================

This module consists of:

- Generic Brazilian chart of accounts
- Brazilian taxes such as:

  - IPI
  - ICMS
  - PIS
  - COFINS
  - ISS
  - IR
  - CSLL

- Document Types as NFC-e, NFS-e, etc.
- Identification Documents as CNPJ and CPF

In addition to this module, the Brazilian Localizations is also
extended and complemented with several additional modules.

Brazil - Accounting Reports (l10n_br_reports)
---------------------------------------------
Adds a simple tax report that helps check the tax amount per tax group
in a given period of time. Also adds the P&L and BS adapted for the
Brazilian market.

Avatax Brazil (l10n_br_avatax)
------------------------------
Add Brazilian tax calculation via Avatax and all necessary fields needed to
configure Odoo in order to properly use Avatax and send the needed fiscal
information to retrieve the correct taxes.

Avatax for SOs in Brazil (l10n_br_avatax_sale)
----------------------------------------------
Same as the l10n_br_avatax module with the extension to the sales order module.

Electronic invoicing through Avatax (l10n_br_edi)
-------------------------------------------------
Create electronic sales invoices with Avatax.
""",
    'author': 'Akretion, Odoo Brasil',
    'depends': [
        'account',
        'base_vat',
        'l10n_latam_base',
        'l10n_latam_invoice_document',
    ],
    'data': [
        'data/account_tax_report_data.xml',
        'data/l10n_latam.identification.type.csv',
        'data/l10n_latam.document.type.csv',
        'views/account_view.xml',
        'views/account_fiscal_position_views.xml',
        'views/res_company_views.xml',
        'views/res_partner_views.xml',
        'views/account_journal_views.xml',
    ],
    'demo': [
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
        <field name="country_id" ref="base.br"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_icms" model="account.report.line">
                <field name="name">ICMS</field>
                <field name="aggregation_formula">TRIBUTADA_INTEGRALMENTE.balance + TRIBUTADA_E_COM_COBRANCA_DO_ICMS_POR_SUBSTITUICAO_TRIBUTARIA.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_icms_tributada" model="account.report.line">
                        <field name="name">Taxed in full</field>
                        <field name="code">TRIBUTADA_INTEGRALMENTE</field>
                        <field name="aggregation_formula">ICMS_1.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_icms_1" model="account.report.line">
                                <field name="name">ICMS base</field>
                                <field name="code">ICMS_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_icms_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ICMS_1</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_icms_tributada_com" model="account.report.line">
                        <field name="name">Taxed and with ICMS collection by tax substitution</field>
                        <field name="code">TRIBUTADA_E_COM_COBRANCA_DO_ICMS_POR_SUBSTITUICAO_TRIBUTARIA</field>
                        <field name="aggregation_formula">ICMS_2.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_icms_2" model="account.report.line">
                                <field name="name">ICMS tax</field>
                                <field name="code">ICMS_2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_icms_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ICMS_2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_icmsst" model="account.report.line">
                <field name="name">ICMS Subist</field>
                <field name="aggregation_formula">ICMSST_1.balance + ICMSST_2.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_icmsst_1" model="account.report.line">
                        <field name="name">ICMSST base</field>
                        <field name="code">ICMSST_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_icmsst_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ICMSST_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_icmsst_2" model="account.report.line">
                        <field name="name">ICMSST tax</field>
                        <field name="code">ICMSST_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_icmsst_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ICMSST_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_irpj" model="account.report.line">
                <field name="name">IRPJ</field>
                <field name="aggregation_formula">IRPJ_1.balance + IRPJ_2.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_irpj_1" model="account.report.line">
                        <field name="name">IRPJ base</field>
                        <field name="code">IRPJ_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_irpj_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">IRPJ_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_irpj_2" model="account.report.line">
                        <field name="name">IRPJ tax</field>
                        <field name="code">IRPJ_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_irpj_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">IRPJ_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ir" model="account.report.line">
                <field name="name">IR</field>
                <field name="aggregation_formula">IR_1.balance + IR_2.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_ir_1" model="account.report.line">
                        <field name="name">IR base</field>
                        <field name="code">IR_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">IR_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ir_2" model="account.report.line">
                        <field name="name">IR tax</field>
                        <field name="code">IR_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_ir_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">IR_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_issqn" model="account.report.line">
                <field name="name">ISSQN</field>
                <field name="aggregation_formula">ISSQN_1.balance + ISSQN_2.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_issqn_1" model="account.report.line">
                        <field name="name">ISSQN base</field>
                        <field name="code">ISSQN_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_issqn_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ISSQN_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_issqn_2" model="account.report.line">
                        <field name="name">ISSQN tax</field>
                        <field name="code">ISSQN_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_issqn_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">ISSQN_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_csll" model="account.report.line">
                <field name="name">CSLL</field>
                <field name="aggregation_formula">CSLL_1.balance + CSLL_2.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_csll_1" model="account.report.line">
                        <field name="name">CSLL base</field>
                        <field name="code">CSLL_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_csll_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CSLL_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_csll_2" model="account.report.line">
                        <field name="name">CSLL tax</field>
                        <field name="code">CSLL_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_csll_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CSLL_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_cofins" model="account.report.line">
                <field name="name">COFINS</field>
                <field name="aggregation_formula">COFINS_OPERACAO_TRIBUTAVEL_COM_ALIQUOTA_BASICA.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_cofins_oper_bas" model="account.report.line">
                        <field name="name">Taxable Transaction with Basic Rate</field>
                        <field name="code">COFINS_OPERACAO_TRIBUTAVEL_COM_ALIQUOTA_BASICA</field>
                        <field name="aggregation_formula">COFINS_1.balance + COFINS_2.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_cofins_1" model="account.report.line">
                                <field name="name">COFINS base</field>
                                <field name="code">COFINS_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_cofins_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">COFINS_1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_cofins_2" model="account.report.line">
                                <field name="name">COFINS tax</field>
                                <field name="code">COFINS_2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_cofins_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">COFINS_2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_pis" model="account.report.line">
                <field name="name">PIS</field>
                <field name="aggregation_formula">PIS_OPERACAO_TRIBUTAVEL_COM_ALIQUOTA_BASICA.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_pis_oper_tri_basica" model="account.report.line">
                        <field name="name">Taxable Transaction with Basic Rate</field>
                        <field name="code">PIS_OPERACAO_TRIBUTAVEL_COM_ALIQUOTA_BASICA</field>
                        <field name="aggregation_formula">PIS_1.balance + PIS_2.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_pis_1" model="account.report.line">
                                <field name="name">PIS base</field>
                                <field name="code">PIS_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_pis_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">PIS_1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_pis_2" model="account.report.line">
                                <field name="name">PIS tax</field>
                                <field name="code">PIS_2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_pis_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">PIS_2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ipi" model="account.report.line">
                <field name="name">IPI</field>
                <field name="code">BRTAX07</field>
                <field name="aggregation_formula">ENTRADA_COM_RECUPERACAO_DE_CREDITO.balance + ENTRADA_TRIBUTADA_COM_ALIQUOTA_ZERO.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_ipi_extrada_com" model="account.report.line">
                        <field name="name">Purchase with Credit Recovery</field>
                        <field name="code">ENTRADA_COM_RECUPERACAO_DE_CREDITO</field>
                        <field name="aggregation_formula">BRTAX07_1.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_ipi_1" model="account.report.line">
                                <field name="name">IPI base</field>
                                <field name="code">BRTAX07_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ipi_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IPI_1</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ipi_extrada_tributada" model="account.report.line">
                        <field name="name">Purchase taxed at a zero rate</field>
                        <field name="code">ENTRADA_TRIBUTADA_COM_ALIQUOTA_ZERO</field>
                        <field name="aggregation_formula">IPI_2.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_ipi_2" model="account.report.line">
                                <field name="name">IPI tax</field>
                                <field name="code">IPI_2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_ipi_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">IPI_2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ii" model="account.report.line">
                <field name="name">II</field>
                <field name="aggregation_formula">II_1.balance + II_2.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_ii_1" model="account.report.line">
                        <field name="name">II base</field>
                        <field name="code">II_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_ii_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_ii_2" model="account.report.line">
                        <field name="name">II tax</field>
                        <field name="code">II_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_ii_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">II_2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_inss" model="account.report.line">
                <field name="name">INSS</field>
                <field name="aggregation_formula">INSS_1.balance + INSS_2.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_inss_1" model="account.report.line">
                        <field name="name">INSS base</field>
                        <field name="code">INSS_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_inss_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">INSS_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_inss_2" model="account.report.line">
                        <field name="name">INSS tax</field>
                        <field name="code">INSS_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_inss_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">INSS_2</field>
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
id,sequence,code,country_id/id,name,internal_type,doc_code_prefix,active
dt_55,10,55,base.br,Electronic Invoice (NF-e),all,NFe,True
dt_57,20,57,base.br,Electronic Bill of Lading (CT-e),invoice,CTe,False
dt_58,30,58,base.br,Electronic Manifesto of Tax Documents (MDF-e),invoice,MDFe,False
dt_59,40,59,base.br,Electronic Tax Coupon (CF-e-SAT),invoice,CFeS,False
dt_60,50,60,base.br,Electronic Tax Coupon (CF-e-ECF),invoice,CFeE,False
dt_65,60,65,base.br,Electronic Invoice to the Final Consumer (NFC-e),all,NFCe,True
dt_SE,70,SE,base.br,Electronic Service Invoice - NFS-e,all,NFSe,True
dt_01,80,01,base.br,Invoice 1 / 1A,all,NF,False
dt_1B,90,1B,base.br,Single invoice,all,NFA,False
dt_02,100,02,base.br,In-Consumer Sales Invoice,all,NFC,False
dt_2D,110,2D,base.br,Tax Coupon,invoice,CF,False
dt_2E,120,2E,base.br,Tax Coupon-Ticket,invoice,TIQP,False
dt_04,130,04,base.br,Invoice from Producer,invoice,NFP,False
dt_06,140,06,base.br,Nota Fiscal / Electricity Bill,invoice,NFCE,False
dt_07,150,07,base.br,Invoice for Transport Service,invoice,NFST,False
dt_08,160,08,base.br,Ground Bill of Lading,invoice,CTRC,False
dt_8B,170,8B,base.br,Loose Ground Bill of Lading,invoice,CTCA,False
dt_09,180,09,base.br,Maritime Bill of Lading,invoice,CTAC,False
dt_10,190,10,base.br,Aircraft Knowledge,invoice,CA,False
dt_11,200,11,base.br,Railway Bill of Lading,invoice,CTFC,False
dt_13,210,13,base.br,Road Ticket,invoice,BPR,False
dt_14,220,14,base.br,Waterway Ticket,invoice,BPA,False
dt_15,230,15,base.br,e-Baggage Ticket,invoice,BPNB,False
dt_16,240,16,base.br,Railway Ticket,invoice,BPF,False
dt_18,250,18,base.br,Daily Movement Summary,invoice,RMD,False
dt_21,260,21,base.br,Invoice for de-Communication Service,invoice,NFSC,False
dt_22,270,22,base.br,Invoice for Telecommunication Service,invoice,NFST,False
dt_26,280,26,base.br,Bill of Lading Multimodal Transport,invoice,CTMC,False
dt_27,290,27,base.br,Invoice for Rail Transport De-Cargo,invoice,NFTFC,False
dt_28,300,28,base.br,Invoice / Gas Supply Channel Account,invoice,NFCFGC,False
dt_29,310,29,base.br,Invoice / Water Supply Account,invoice,NFCFAC,False

```

## File: data\l10n_latam.identification.type.csv

```csv
id,name,country_id:id,is_vat
cnpj,CNPJ,base.br,True
cpf,CPF,base.br,False

```

## File: data\template\account.account-br.csv

```csv
"id","code","name","account_type","reconcile","name@pt"
"account_template_101010101","1.01.01.01.01","Cash - Head Office","asset_cash","","Caixa Matriz"
"account_template_101010102","1.01.01.01.02","Cash - Branches","asset_cash","","Caixa Filiais"
"account_template_101010201","1.01.01.02.01","Banks Account Movement - In the Country","asset_current","","Bancos Conta Movimento - No País"
"account_template_101010202","1.01.01.02.02","Banks Account Movement - Abroad","asset_current","","Bancos Conta Movimento - No Exterior"
"account_template_101010401","1.01.01.04.01","Cash in Transit","asset_receivable","True","Numerários em Trânsito"
"account_template_101010402","1.01.01.04.02","Cash in Transit (PoS)","asset_receivable","True","Numerários em Trânsito (PoS)"
"account_template_101010501","1.01.01.05.01","Securities for Trading - Measured to Fair Value By Result (VJPR) - In the Country","asset_current","","Títulos para Negociação - Mensurados a Valor Justo Por Meio do Resultado (VJPR) - No País"
"account_template_101010502","1.01.01.05.02","Securities Available for Sale - In the Country","asset_current","","Títulos Disponíveis para Venda - No País"
"account_template_101010503","1.01.01.05.03","Held-to-Maturity Securities - In the Country","asset_current","","Títulos Mantidos até o Vencimento - No País"
"account_template_101010510","1.01.01.05.10","Debentures issued by Related Parties - In the Country","asset_current","","Debêntures emitidas por Partes Relacionadas - No País"
"account_template_101010511","1.01.01.05.11","Debentures Issued by Non-Related Parties - In the Country","asset_current","","Debêntures Emitidas por Partes Não Relacionadas - No País"
"account_template_101010515","1.01.01.05.15","Other Loans and Receivables - In the Country","asset_current","","Outros Empréstimos e Recebíveis - No País"
"account_template_101010550","1.01.01.05.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) - Securities - In the Country","asset_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Valores Mobiliários - No País"
"account_template_101010555","1.01.01.05.55","(-) Losses by Recuperable Value (Impairment)- Securities - In the Country","asset_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment)- Valores Mobiliários - No País"
"account_template_101010570","1.01.01.05.70","Subaccount - Fair Value Adjustment - Securities – No Hedge - In the Country","asset_current","","Subconta - Ajuste a Valor Justo - Valores Mobiliários – Não Hedge -No País"
"account_template_101010590","1.01.01.05.90","Subaccount – Initial Adoption - Securities – No Hedge - In the Country","asset_current","","Subconta – Adoção Inicial - Valores Mobiliários – Não Hedge - No País"
"account_template_101010601","1.01.01.06.01","Derivatives - Hedge Fair Value - In the Country","asset_current","","Derivativos - Hedge Valor Justo - No País"
"account_template_101010602","1.01.01.06.02","Derivatives - Cash Flow Hedge - In the Country","asset_current","","Derivativos - Hedge Fluxo de Caixa - No País"
"account_template_101010603","1.01.01.06.03","Derivatives - Hedge Foreign Investment - In the Country","asset_current","","Derivativos - Hedge Investimento no Exterior - No País"
"account_template_101010670","1.01.01.06.70","Subaccount - Fair Value Adjustment - Securities – Hedge - In the Country","asset_current","","Subconta - Ajuste a Valor Justo - Valores Mobiliários – Hedge - No País"
"account_template_101010690","1.01.01.06.90","Subaccount – Initial Adoption - Securities – Hedge - In the Country","asset_current","","Subconta – Adoção Inicial - Valores Mobiliários – Hedge - No País"
"account_template_101010901","1.01.01.09.01","Securities for Trading - Measured to Fair Value by Result (VJPR) - Abroad","asset_current","","Títulos para Negociação - Mensurados a Valor Justo por Meio de Resultado (VJPR) - No Exterior"
"account_template_101010902","1.01.01.09.02","Securities Available for Sale - Abroad","asset_current","","Títulos Disponíveis para Venda - No Exterior"
"account_template_101010903","1.01.01.09.03","Held-to-Maturity Securities - Abroad","asset_current","","Títulos Mantidos até o Vencimento - No Exterior"
"account_template_101010910","1.01.01.09.10","Debentures issued by Related Parties - Abroad","asset_current","","Debêntures emitidas por Partes Relacionadas - No Exterior"
"account_template_101010911","1.01.01.09.11","Debentures issued by Non-Related Parties - Abroad","asset_current","","Debêntures emitidas por Partes Não Relacionadas - No Exterior"
"account_template_101010915","1.01.01.09.15","Other Loans and Receivables - Abroad","asset_current","","Outros Empréstimos e Recebíveis - No Exterior"
"account_template_101010950","1.01.01.09.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) - Securities - Abroad","asset_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Valores Mobiliários - No Exterior"
"account_template_101010955","1.01.01.09.55","(-) Losses by Recuperable Value (Impairment)- Securities - Abroad","asset_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment)- Valores Mobiliários - No Exterior"
"account_template_101010970","1.01.01.09.70","Subaccount - Fair Value Adjustment - No Hedge - Abroad","asset_current","","Subconta - Ajuste a Valor Justo - Não Hedge - No Exterior"
"account_template_101010990","1.01.01.09.90","Subaccount – Initial Adoption - Securities – No Hedge - Abroad","asset_current","","Subconta – Adoção Inicial - Valores Mobiliários – Não Hedge - No Exterior"
"account_template_101011001","1.01.01.10.01","Derivatives - Hedge Fair Value - Abroad","asset_current","","Derivativos - Hedge Valor Justo - No Exterior"
"account_template_101011002","1.01.01.10.02","Derivatives - Cash Flow Hedge - Abroad","asset_current","","Derivativos - Hedge Fluxo de Caixa - No Exterior"
"account_template_101011003","1.01.01.10.03","Derivatives - Hedge Foreign Investment - Abroad","asset_current","","Derivativos - Hedge Investimento no Exterior - No Exterior"
"account_template_101011070","1.01.01.10.70","Subaccount - Fair Value Adjustment - Securities – Hedge - Abroad","asset_current","","Subconta - Ajuste a Valor Justo - Valores Mobiliários – Hedge - No Exterior"
"account_template_101011090","1.01.01.10.90","Subaccount – Initial Adoption - Securities – Hedge - Abroad","asset_current","","Subconta – Adoção Inicial - Valores Mobiliários – Hedge - No Exterior"
"account_template_101014001","1.01.01.40.01","Resources Abroad Resulting from Exports","asset_current","","Recursos no Exterior Decorrentes de Exportação"
"account_template_101019901","1.01.01.99.01","Other Bank and Cash accounts","asset_current","","Outras Disponibilidades"
"account_template_101020101","1.01.02.01.01","Advances to Suppliers - in the Country – Current","asset_current","","Adiantamentos a Fornecedores - no País – Circulante"
"account_template_101020102","1.01.02.01.02","Advances to Suppliers - Abroad – Current","asset_current","","Adiantamentos a Fornecedores - no Exterior – Circulante"
"account_template_101020103","1.01.02.01.03","Advances to Employees – Current","asset_current","","Adiantamentos a Funcionários – Circulante"
"account_template_101020104","1.01.02.01.04","Advances to Third Parties – Current","asset_current","","Adiantamentos a Terceiros – Circulante"
"account_template_101020198","1.01.02.01.98","Other Advances – Current","asset_current","","Outros Adiantamentos – Circulante"
"account_template_101020201","1.01.02.02.01","Trade Bills Receivable – Operations with Non-related Parties - in the Country","asset_current","","Duplicatas a Receber – Operações com Partes Não Relacionadas - no País"
"account_template_101020202","1.01.02.02.02","Trade Bills Receivable - Transactions with Non-Related Parties - Abroad","asset_current","","Duplicatas a Receber - Operações com Partes Não Relacionadas - no Exterior"
"account_template_101020203","1.01.02.02.03","Trade Bills Receivable - Operations with Related Parties - in the Country","asset_current","","Duplicatas a Receber - Operações com Partes Relacionadas - no País"
"account_template_101020204","1.01.02.02.04","Trade Bills Receivable - Operations with Related Parties - Abroad","asset_current","","Duplicatas a Receber - Operações com Partes Relacionadas - no Exterior"
"account_template_101020250","1.01.02.02.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) – Bills Receivable","asset_current","","(-) Juros a Apropriar Decorrentes de Ajuste da Valor Presente (AVP) – Duplicatas a Receber"
"account_template_101020252","1.01.02.02.52","(-) Estimated Losses in Debt Settlement Credits - Bills Receivable","asset_current","","(-) Perdas Estimadas em Créditos de Liquidação Duvidosa - Duplicatas a Receber"
"account_template_101020255","1.01.02.02.55","(-) Recuperable Value (Impairment) - Bills Receivable","asset_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Duplicatas a Receber"
"account_template_101020290","1.01.02.02.90","Subaccount – Initial Adoption - Trade Bills Receivable","asset_current","","Subconta – Adoção Inicial - Duplicatas a Receber"
"account_template_101020301","1.01.02.03.01","IPI to Recover","asset_current","","IPI a Recuperar"
"account_template_101020302","1.01.02.03.02","ICMS to Recover","asset_current","","ICMS a Recuperar"
"account_template_101020303","1.01.02.03.03","PIS to Recover - Basic Credit","asset_current","","PIS a Recuperar - Crédito Básico"
"account_template_101020304","1.01.02.03.04","PIS to Recover - Presumed Credit","asset_current","","PIS a Recuperar - Crédito Presumido"
"account_template_101020305","1.01.02.03.05","COFINS to Recover - Basic Credit","asset_current","","COFINS a Recuperar - Crédito Básico"
"account_template_101020306","1.01.02.03.06","COFINS to Recover - Presumed Credit","asset_current","","COFINS a Recuperar - Crédito Presumido"
"account_template_101020307","1.01.02.03.07","CIDE to Recover","asset_current","","CIDE a Recuperar"
"account_template_101020340","1.01.02.03.40","Other Taxes and Contributions to Recover","asset_current","","Outros Impostos e Contribuições a Recuperar"
"account_template_101020350","1.01.02.03.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) - Taxes to recover","asset_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Tributos a Recuperar"
"account_template_101020355","1.01.02.03.55","(-) Recuperable Value Reduction Loss (Impairment) - Tributes to Recover","asset_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Tributos a Recuperar"
"account_template_101020401","1.01.02.04.01","Withholding Income Tax (IRRF)","asset_current","","Imposto de Renda Retido na Fonte (IRRF)"
"account_template_101020402","1.01.02.04.02","IRPJ Collected by Estimate","asset_current","","IRPJ Recolhido por Estimativa"
"account_template_101020403","1.01.02.04.03","IRPJ Negative Balance","asset_current","","IRPJ Saldo Negativo"
"account_template_101020404","1.01.02.04.04","CSLL Retained in Source","asset_current","","CSLL Retida na Fonte"
"account_template_101020405","1.01.02.04.05","CSLL Received by Estimate","asset_current","","CSLL Recolhida por Estimativa"
"account_template_101020406","1.01.02.04.06","CSLL Negative Balance","asset_current","","CSLL Saldo Negativo"
"account_template_101020407","1.01.02.04.07","PIS/PASEP Withheld at Source","asset_current","","PIS/PASEP Retido na Fonte"
"account_template_101020408","1.01.02.04.08","PIS/PASEP to Compensate","asset_current","","PIS/PASEP a Compensar"
"account_template_101020409","1.01.02.04.09","COFINS Retained in Source","asset_current","","COFINS Retida na Fonte"
"account_template_101020410","1.01.02.04.10","COFINS to Compensate","asset_current","","COFINS a Compensar"
"account_template_101020411","1.01.02.04.11","IPI to Compensate","asset_current","","IPI a Compensar"
"account_template_101020412","1.01.02.04.12","IOF Compensation","asset_current","","IOF a Compensar"
"account_template_101020413","1.01.02.04.13","Import Tax Compensation","asset_current","","Imposto de Importação a Compensar"
"account_template_101020414","1.01.02.04.14","Export Tax to Compensate","asset_current","","Imposto de Exportação a Compensar"
"account_template_101020415","1.01.02.04.15","ITR Compensation","asset_current","","ITR a Compensar"
"account_template_101020416","1.01.02.04.16","CIDE to Compensate","asset_current","","CIDE a Compensar"
"account_template_101020417","1.01.02.04.17","Social Security Contribution Withheld on Provision of Services","asset_current","","Contribuição Previdenciária Retida na Prestação de Serviços"
"account_template_101020418","1.01.02.04.18","Social Security Contribution to be Offset","asset_current","","Contribuição Previdenciária a Compensar"
"account_template_101020440","1.01.02.04.40","Other Taxes to Compensate","asset_current","","Outros Tributos a Compensar"
"account_template_101020450","1.01.02.04.50","( - ) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) - Taxes to be Offset","asset_current","","( - ) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Tributos a Compensar"
"account_template_101020455","1.01.02.04.55","( - ) Losses due to Reduction to Refundable Value (Impairment)- Taxes to be Compensated","asset_current","","( - ) Perdas por Redução ao Valor Recuperável (Impairment)- Tributos a Compensar"
"account_template_101020901","1.01.02.09.01","Mutual Loans with Non-Related Parties – Current - In the Country","asset_current","","Mútuos com Partes Não Relacionadas – Circulante - No País"
"account_template_101020902","1.01.02.09.02","Mutual Loans with Non-Related Parties – Current - Abroad","asset_current","","Mútuos com Partes Não Relacionadas – Circulante - No Exterior"
"account_template_101020903","1.01.02.09.03","Dividends Receivable - Current - In the Country","asset_current","","Dividendos a Receber - Circulante - No País"
"account_template_101020904","1.01.02.09.04","Dividends Receivable - Current - Abroad","asset_current","","Dividendos a Receber - Circulante - No Exterior"
"account_template_101020905","1.01.02.09.05","Interest on Shareholders' Equity Receivable - Current","asset_current","","Juros Sobre o Capital Próprio a Receber - Circulante"
"account_template_101020906","1.01.02.09.06","Advance for Future Capital Increase – Asset - Current","asset_current","","Adiantamento para Futuro Aumento de Capital –Ativo - Circulante"
"account_template_101020907","1.01.02.09.07","Other Interests Receivable - Current","asset_current","","Outros Juros a Receber - Circulante"
"account_template_101020909","1.01.02.09.09","Contingent consideration assets - Business Combination - Current","asset_current","","Contraprestação Contingente Ativa - Combinação de Negócios - Circulante"
"account_template_101020910","1.01.02.09.10","More Credits to Receive - Current","asset_current","","Demais Créditos a Receber - Circulante"
"account_template_101020911","1.01.02.09.11","Deposits in Litigation - Current","asset_current","","Depósitos em Contencioso - Circulante"
"account_template_101020912","1.01.02.09.12","Other Credits in Litigation - Current","asset_current","","Outros Créditos em Contencioso - Circulante"
"account_template_101020920","1.01.02.09.20","Assigned Creditor Rights","asset_current","","Direitos Creditórios Cedidos"
"account_template_101020921","1.01.02.09.21","(-) Negative goodwill on transfer of securities","asset_current","","(-) Deságio na Cessão de Títulos"
"account_template_101020925","1.01.02.09.25","Creditor Rights Receivable","asset_current","","Direitos Creditórios a Receber"
"account_template_101020950","1.01.02.09.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) – Other Credits - Current","asset_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) – Outros Créditos - Circulante"
"account_template_101020955","1.01.02.09.55","(-) Refundable Value Losses (Impairment) - Other Credits - Current","asset_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Outros Créditos - Circulante"
"account_template_101020960","1.01.02.09.60","CPC 47 - Contract Targets - Current","asset_current","","CPC 47 - Atvos de Contrato - Circulante"
"account_template_101020970","1.01.02.09.70","Subaccount - Fair Value Adjustment – Other Credits - Current","asset_current","","Subconta - Ajuste a Valor Justo – Outros Créditos - Circulante"
"account_template_101030101","1.01.03.01.01","Goods for Resale","asset_current","","Mercadorias para Revenda"
"account_template_101030155","1.01.03.01.55","(-) Loss due to Adjustment to the Net Realizable Value - Stock Goods","asset_current","","(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Mercadorias"
"account_template_101030170","1.01.03.01.70","Subaccount - Fair Value Adjustment - Stock Goods","asset_current","","Subconta - Ajuste a Valor Justo - Estoque Mercadorias"
"account_template_101030175","1.01.03.01.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Stock Goods","asset_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Mercadorias"
"account_template_101030190","1.01.03.01.90","Subaccount – Initial Adoption - Stocks of Goods","asset_current","","Subconta – Adoção Inicial - Estoques de Mercadorias"
"account_template_101030201","1.01.03.02.01","Supplies (direct materials)","asset_current","","Insumos (materiais diretos)"
"account_template_101030202","1.01.03.02.02","Other Materials","asset_current","","Outros Materiais"
"account_template_101030203","1.01.03.02.03","Products for Construction","asset_current","","Produtos em Elaboração"
"account_template_101030204","1.01.03.02.04","Finished Products","asset_current","","Produtos Acabados"
"account_template_101030255","1.01.03.02.55","(-) Loss due to Adjustment to the Net Realizable Value - Stock Products","asset_current","","(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Produtos"
"account_template_101030270","1.01.03.02.70","Subaccount - Fair Value Adjustment - Stock Products","asset_current","","Subconta - Ajuste a Valor Justo - Estoque de Produtos"
"account_template_101030275","1.01.03.02.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Stock of Products","asset_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque de Produtos"
"account_template_101030290","1.01.03.02.90","Subaccount – Initial Adoption - Product Stocks","asset_current","","Subconta – Adoção Inicial - Estoques de Produtos"
"account_template_101030301","1.01.03.03.01","Lands - Real Estate Activity","asset_current","","Terrenos - Atividade Imobiliária"
"account_template_101030302","1.01.03.03.02","Real Estate Acquired for Resale - Real Estate Activity","asset_current","","Imóveis Adquiridos para Revenda - Atividade Imobiliária"
"account_template_101030303","1.01.03.03.03","Work in Progress - Real Estate Activity","asset_current","","Obras em Andamento - Atividade Imobiliária"
"account_template_101030304","1.01.03.03.04","Real Estate for Sale - Real Estate Activity","asset_current","","Imóveis à Venda - Atividade Imobiliária"
"account_template_101030305","1.01.03.03.05","Constructions in Progress in Real Estate For sale","asset_current","","Construções em Andamento de Imóveis Destinados à Venda"
"account_template_101030306","1.01.03.03.06","Construction Materials - Real Estate Activity","asset_current","","Materiais de Construção - Atividade Imobiliária"
"account_template_101030355","1.01.03.03.55","(-) Loss due to Adjustment to the Net Realizable Value - Stock Real Estate Activity","asset_current","","(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Atividade Imobiliária"
"account_template_101030375","1.01.03.03.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Stock Real Estate Activity","asset_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Atividade Imobiliária"
"account_template_101030390","1.01.03.03.90","Subaccount – Initial Adoption - Stocks – Real Estate Activity","asset_current","","Subconta – Adoção Inicial - Estoques – Atividade Imobiliária"
"account_template_101030401","1.01.03.04.01","Supplies (direct materials) - Long Maturity Stock","asset_current","","Insumos (materiais diretos) - Estoque Longa Maturação"
"account_template_101030402","1.01.03.04.02","Other Materials - Stock Long Maturation","asset_current","","Outros Materiais - Estoque Longa Maturação"
"account_template_101030403","1.01.03.04.03","Work in Progress - Long Maturation Stock","asset_current","","Produtos em Elaboração - Estoque Longa Maturação"
"account_template_101030404","1.01.03.04.04","Finished Products - Long Maturation Stock","asset_current","","Produtos Acabados - Estoque Longa Maturação"
"account_template_101030455","1.01.03.04.55","(-) Loss due to Adjustment to the Net Realizable Value - Stock Long Maturation","asset_current","","(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Longa Maturação"
"account_template_101030470","1.01.03.04.70","Subaccount - Fair Value Adjustment - Stock Long Maturation","asset_current","","Subconta - Ajuste a Valor Justo - Estoque Longa Maturação"
"account_template_101030475","1.01.03.04.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Stock Long Maturation","asset_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Longa Maturação"
"account_template_101030490","1.01.03.04.90","Subaccount – Initial Adoption - Stocks – Long Maturation","asset_current","","Subconta – Adoção Inicial - Estoques – Longa Maturação"
"account_template_101030501","1.01.03.05.01","Agricultural Products of Animal Origin","asset_current","","Produtos Agropecuários de Origem Animal"
"account_template_101030502","1.01.03.05.02","Agricultural Products of Plant Origin","asset_current","","Produtos Agropecuários de Origem Vegetal"
"account_template_101030503","1.01.03.05.03","Agricultural Supplies","asset_current","","Insumos Agropecuários"
"account_template_101030504","1.01.03.05.04","Other Materials - Rural Activity","asset_current","","Outros Materiais - Atividade Rural"
"account_template_101030555","1.01.03.05.55","(-) Loss due to Adjustment to Net Realisable Value - Inventory Rural Activity","asset_current","","(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Atividade Rural"
"account_template_101030570","1.01.03.05.70","Subaccount - Fair Value Adjustment - Stock Rural Activity","asset_current","","Subconta - Ajuste a Valor Justo - Estoque Atividade Rural"
"account_template_101030575","1.01.03.05.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Stock Rural Activity","asset_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Atividade Rural"
"account_template_101030590","1.01.03.05.90","Subaccount – Initial Adoption - Stocks – Rural Activity","asset_current","","Subconta – Adoção Inicial - Estoques – Atividade Rural"
"account_template_101030601","1.01.03.06.01","Materials Applied in Service Production","asset_current","","Materiais Aplicados na Produção de Serviços"
"account_template_101030602","1.01.03.06.02","Services in Construction","asset_current","","Serviços em Andamento"
"account_template_101030603","1.01.03.06.03","Finished Services","asset_current","","Serviços Acabados"
"account_template_101030655","1.01.03.06.55","(-) Loss due to Adjustment to the Net Realizable Value - Stock Services","asset_current","","(-) Perda por Ajuste ao Valor Realizável Líquido - Estoque Serviços"
"account_template_101030670","1.01.03.06.70","Subaccount - Fair Value Adjustment - Stock Services","asset_current","","Subconta - Ajuste a Valor Justo - Estoque Serviços"
"account_template_101030675","1.01.03.06.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Stock Services","asset_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Serviços"
"account_template_101030690","1.01.03.06.90","Subaccount – Initial Adoption - Stock Services","asset_current","","Subconta – Adoção Inicial - Estoque Serviços"
"account_template_101030701","1.01.03.07.01","Material in Storeroom","asset_current","","Material em Almoxarifado"
"account_template_101030702","1.01.03.07.02","Material Intended for Destruction","asset_current","","Material Destinado à Destruição"
"account_template_101030703","1.01.03.07.03","Scrap","asset_current","","Sucata"
"account_template_101030704","1.01.03.07.04","Other Stocks","asset_current","","Outros Estoques"
"account_template_101030755","1.01.03.07.55","(-) Loss due to Adjustment to the Net Realizable Value - Other Stocks","asset_current","","(-) Perda por Ajuste ao Valor Realizável Líquido - Estoques Outros"
"account_template_101030775","1.01.03.07.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Stock Other","asset_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Estoque Outros"
"account_template_101030790","1.01.03.07.90","Subaccount – Initial Adoption - Stock Other","asset_current","","Subconta – Adoção Inicial - Estoque Outros"
"account_template_101050101","1.01.05.01.01","Rent Paid in Advance","asset_prepayments","","Alugueis Pagos Antecipadamente"
"account_template_101050102","1.01.05.01.02","Insurance Premiums Payable","asset_prepayments","","Prêmios de Seguros a Apropriar"
"account_template_101050103","1.01.05.01.03","Financial Charges to be Appropriated","asset_prepayments","","Encargos Financeiros a Apropriar"
"account_template_101050109","1.01.05.01.09","Other Paid Costs and Expenditure Previously","asset_prepayments","","Outros Custos e Despesas Pagos Antecipadamente"
"account_template_101100101","1.01.10.01.01","Consumable Biological Asset - Animal Origin – For Fair Value","asset_current","","Ativo Biológico Consumível - Origem Animal – Pelo Valor Justo"
"account_template_101100102","1.01.10.01.02","Consumable Biological Asset - Plant Origin – For Fair Value","asset_current","","Ativo Biológico Consumível - Origem Vegetal – Pelo Valor Justo"
"account_template_101100170","1.01.10.01.70","Subaccount - Fair Value Adjustment (AVJ) – Consumable Biological Assets By Fair Value - Current","asset_current","","Subconta - Ajuste a Valor Justo (AVJ) – Ativos Biológicos Consumíveis Pelo Valor Justo - Circulante"
"account_template_101100175","1.01.10.01.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Biological Assets By Fair Value - Current","asset_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Ativos Biológicos Pelo Valor Justo - Circulante"
"account_template_101100190","1.01.10.01.90","Subaccount – Initial Adoption - Biological Assets","asset_current","","Subconta – Adoção Inicial - Ativo Biologico"
"account_template_101100201","1.01.10.02.01","Consumable Biological Asset - Animal Origin - For Cost","asset_current","","Ativo Biológico Consumível - Origem Animal - Pelo Custo"
"account_template_101100202","1.01.10.02.02","Consumable Biological Asset - Plant Origin - For Cost","asset_current","","Ativo Biológico Consumível - Origem Vegetal - Pelo Custo"
"account_template_101100255","1.01.10.02.55","( - ) Losses for Recuperable Value Reduction (Impairment) - Consumable Biological Assets - At Cost","asset_current","","( - ) Perdas por Redução ao Valor Recuperável (Impairment) - Ativos Biológicos Consumível - Pelo Custo"
"account_template_101100275","1.01.10.02.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Consumable Biological Assets - At Cost","asset_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Ativos Biológicos Consumíveis - Pelo Custo"
"account_template_101110101","1.01.11.01.01","Non-Current Assets Held For For sale","asset_current","","Ativo Não Circulante Mantido Para Venda"
"account_template_101110155","1.01.11.01.55","(-) Recuperable Value Reduction Losses (Impairment) - Non-Current Assets Held for Sale","asset_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Ativo Não Circulante Mantido para Venda"
"account_template_101110170","1.01.11.01.70","Subaccount - Fair Value Adjustment – Non Current Asset Held for Sale","asset_current","","Subconta - Ajuste a Valor Justo – Ativo Não Circulante Mantido para Venda"
"account_template_101110175","1.01.11.01.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Non-Current Asset Holding for Sale","asset_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Ativo Não Circulante Mantido para Venda"
"account_template_101110190","1.01.11.01.90","Subaccount - Initial Adoption - Non-Current Assets Held for Sale","asset_current","","Subconta – Adoção Inicial - Ativo Não circulante Mantido para Venda"
"account_template_102010101","1.02.01.01.01","Customers - Long Term","asset_non_current","","Clientes - Longo Prazo"
"account_template_102010102","1.02.01.01.02","Mutual Loans with Non-Related Parties - Asset- Long-term","asset_non_current","","Mútuos com Partes Não Relacionadas - Ativo- Longo Prazo"
"account_template_102010103","1.02.01.01.03","Mutual Loans with Related Parties - Asset - Long Term","asset_non_current","","Mútuos com Partes Relacionadas - Ativo - Longo Prazo"
"account_template_102010104","1.02.01.01.04","Advance for Future Capital Increase - Asset - Long Term","asset_non_current","","Adiantamento para Futuro Aumento de Capital - Ativo - Longo Prazo"
"account_template_102010107","1.02.01.01.07","Equity Securities Valued at Cost","asset_non_current","","Títulos Patrimoniais Avaliados pelo Custo - Longo Prazo"
"account_template_102010109","1.02.01.01.09","Contingent consideration assets - Business Combination – Long Term","asset_non_current","","Contraprestação Contingente Ativa - Combinação de Negócios – Longo Prazo"
"account_template_102010110","1.02.01.01.10","Other Securities - Abroad - Long Term","asset_non_current","","Outros Valores Mobiliários - No Exterior - Longo Prazo"
"account_template_102010119","1.02.01.01.19","(-) Other Rectifying Accounts - Credits and Values - Long Term","asset_non_current","","(-) Outras Contas Retificadoras – Créditos e Valores - Longo Prazo"
"account_template_102010120","1.02.01.01.20","Assigned Creditor Rights – Long Term","asset_non_current","","Direitos Creditórios Cedidos –Longo Prazo"
"account_template_102010121","1.02.01.01.21","(-) Negative goodwill on transfer of securities – Long term","asset_non_current","","(-) Deságio na Cessão de Títulos–Longo Prazo"
"account_template_102010125","1.02.01.01.25","Creditor Rights Receivable – Long Term","asset_non_current","","Direitos Creditórios a Receber–Longo Prazo"
"account_template_102010126","1.02.01.01.26","Trade Bills Receivable – Operations with Non-related Parties - in the Country – Long-term","asset_non_current","","Duplicatas a Receber – Operações com Partes Não Relacionadas - no País – Longo Prazo"
"account_template_102010127","1.02.01.01.27","Trade Bills Receivable - Operations with Non-Related Parties - Abroad – Long Term","asset_non_current","","Duplicatas a Receber - Operações com Partes Não Relacionadas - no Exterior – Longo Prazo"
"account_template_102010128","1.02.01.01.28","Trade Bills Receivable - Operations with Related Parties - in the Country – Long Term","asset_non_current","","Duplicatas a Receber - Operações com Partes Relacionadas - no País – Longo Prazo"
"account_template_102010129","1.02.01.01.29","Trade Bills Receivable - Operations with Related Parties - Abroad – Long Term","asset_non_current","","Duplicatas a Receber - Operações com Partes Relacionadas - no Exterior – Longo Prazo"
"account_template_102010150","1.02.01.01.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) - Long-Term Credits","asset_non_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Créditos –Longo Prazo"
"account_template_102010152","1.02.01.01.52","(-) Estimated Losses in Debt Settlement Credits – Long Term","asset_non_current","","(-) Perdas Estimadas em Créditos de Liquidação Duvidosa – Longo Prazo"
"account_template_102010155","1.02.01.01.55","(-) Refundable Value Losses (Impairment) - Credits and Values - Long Term","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Créditos e Valores - Longo Prazo"
"account_template_102010170","1.02.01.01.70","Subaccount - Fair Value Adjustment - Credits – Long Term","asset_non_current","","Subconta - Ajuste a Valor Justo - Créditos –Longo Prazo"
"account_template_102010190","1.02.01.01.90","Subaccount – Initial Adoption - Credits and Values – Long Term","asset_non_current","","Subconta – Adoção Inicial - Créditos e Valores – Longo Prazo"
"account_template_102010201","1.02.01.02.01","Securities for Trading - In the Country - Long Term","asset_non_current","","Títulos para Negociação - No País - Longo Prazo"
"account_template_102010202","1.02.01.02.02","Securities Available for Sale - In the Country - Long Term","asset_non_current","","Títulos Disponíveis para Venda - No País - Longo Prazo"
"account_template_102010203","1.02.01.02.03","Held-to-Maturity Securities - In the Country - Long Term","asset_non_current","","Títulos Mantidos até o Vencimento - No País - Longo Prazo"
"account_template_102010210","1.02.01.02.10","Debentures issued by Related Parties - In the Country - Long Term","asset_non_current","","Debêntures emitidas por Partes Relacionada - No País - Longo Prazo"
"account_template_102010211","1.02.01.02.11","Debentures issued by Non-Related Parties - In the Country - Long Term","asset_non_current","","Debêntures emitidas por Partes Não Relacionada - No País - Longo Prazo"
"account_template_102010214","1.02.01.02.14","Other Securities – In the Country - Long Term","asset_non_current","","Outros Valores Mobiliários – No País - Longo Prazo"
"account_template_102010215","1.02.01.02.15","Other Loans and Receivables – In the Country - Long Term","asset_non_current","","Outros Empréstimos e Recebíveis – No País - Longo Prazo"
"account_template_102010216","1.02.01.02.16","Other Interests Receivable – In the Country - Long Term","asset_non_current","","Outros Juros a Receber – No País - Longo Prazo"
"account_template_102010250","1.02.01.02.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) Securities – Country - Long Term","asset_non_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) Valores Mobiliários – No País - Longo Prazo"
"account_template_102010255","1.02.01.02.55","(-) Losses by Refundable Value (Impairment) - Securities - In the Country - Long Term","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Valores Mobiliários - No País - Longo Prazo"
"account_template_102010270","1.02.01.02.70","Subaccount - Fair Value Adjustment – Securities – In the Country - Long Term","asset_non_current","","Subconta - Ajuste a Valor Justo – Valores Mobiliários – No País - Longo Prazo"
"account_template_102010290","1.02.01.02.90","Subaccount – Initial Adoption - Securities - In the Country – Long Term","asset_non_current","","Subconta – Adoção Inicial - Valores Mobiliários - No País – Longo Prazo"
"account_template_102010301","1.02.01.03.01","Securities for Trading - Abroad - Long Term","asset_non_current","","Títulos para Negociação - No Exterior - Longo Prazo"
"account_template_102010302","1.02.01.03.02","Securities Available for Sale - Abroad - Long Term","asset_non_current","","Títulos Disponíveis para Venda - No Exterior - Longo Prazo"
"account_template_102010303","1.02.01.03.03","Held-to-Maturity Securities - Abroad - Long Term","asset_non_current","","Títulos Mantidos até o Vencimento - No Exterior - Longo Prazo"
"account_template_102010304","1.02.01.03.04","Debentures issued by Related Parties – Abroad - Long Term","asset_non_current","","Debêntures emitidas por Partes Relacionada – No Exterior - Longo Prazo"
"account_template_102010305","1.02.01.03.05","Debentures issued by Non-Related Parties - Abroad - Long Term","asset_non_current","","Debêntures emitidas por Partes Não Relacionada - No Exterior - Longo Prazo"
"account_template_102010306","1.02.01.03.06","Other Loans and Receivables – Abroad - Long Term","asset_non_current","","Outros Empréstimos e Recebíveis – No Exterior - Longo Prazo"
"account_template_102010350","1.02.01.03.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) Securities – Abroad - Long Term","asset_non_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) Valores Mobiliários – No Exterior - Longo Prazo"
"account_template_102010355","1.02.01.03.55","(-) Recuperable Value Reduction Losses (Impairment) - Securities - Abroad - Long Term","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Valores Mobiliário - No Exterior - Longo Prazo"
"account_template_102010370","1.02.01.03.70","Subaccount - Fair Value Adjustment – Securities – Abroad - Long Term","asset_non_current","","Subconta - Ajuste a Valor Justo – Valores Mobiliários – No Exterior - Longo Prazo"
"account_template_102010390","1.02.01.03.90","Subaccount – Initial Adoption - Securities - Abroad – Long Term","asset_non_current","","Subconta – Adoção Inicial - Valores Mobiliários - No Exterior – Longo Prazo"
"account_template_102010501","1.02.01.05.01","CSLL Tax Credits - Temporary Differences and Negative Calculation Base - Long Term","asset_non_current","","Créditos Fiscais CSLL - Diferenças Temporárias e Base de Cálculo Negativa - Longo Prazo"
"account_template_102010502","1.02.01.05.02","Tax Credits IRPJ - Temporary Differences and Tax Impairments - Long Term","asset_non_current","","Créditos Fiscais IRPJ - Diferenças Temporárias e Prejuízos Fiscais - Longo Prazo"
"account_template_102010701","1.02.01.07.01","Deposits in Litigation - Long term","asset_non_current","","Depósitos em Contencioso - Longo Prazo"
"account_template_102010710","1.02.01.07.10","Other Credits in Litigation - Long Term","asset_non_current","","Outros Créditos em Contencioso - Longo Prazo"
"account_template_102010755","1.02.01.07.55","( - ) Perdas por Recuperável Value (Impairment) - Credits in Litigation – Long Term","asset_non_current","","( - ) Perdas por Redução ao Valor Recuperável (Impairment) - Créditos em Contencioso – Longo Prazo"
"account_template_102010801","1.02.01.08.01","IPI to Recover - Long Term","asset_non_current","","IPI a Recuperar - Longo Prazo"
"account_template_102010802","1.02.01.08.02","ICMS to Recover - Long Term","asset_non_current","","ICMS a Recuperar - Longo Prazo"
"account_template_102010803","1.02.01.08.03","PIS to Recover - Basic Credit - Long Term","asset_non_current","","PIS a Recuperar - Crédito Básico - Longo Prazo"
"account_template_102010804","1.02.01.08.04","PIS to Recover - Presumed Credit - Long Term","asset_non_current","","PIS a Recuperar - Crédito Presumido - Longo Prazo"
"account_template_102010805","1.02.01.08.05","COFINS to Recover - Basic Credit - Long Term","asset_non_current","","COFINS a Recuperar - Crédito Básico - Longo Prazo"
"account_template_102010806","1.02.01.08.06","COFINS to Recover - Presumed Credit - Long Term","asset_non_current","","COFINS a Recuperar - Crédito Presumido - Longo Prazo"
"account_template_102010807","1.02.01.08.07","CIDE to Recover - Long Term","asset_non_current","","CIDE a Recuperar - Longo Prazo"
"account_template_102010840","1.02.01.08.40","Other Taxes and Contributions to Recover - Long Term","asset_non_current","","Outros Impostos e Contribuições a Recuperar - Longo Prazo"
"account_template_102010850","1.02.01.08.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) - Taxes to Recover - Long Term","asset_non_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Tributos a Recuperar - Longo Prazo"
"account_template_102010855","1.02.01.08.55","(-) Recuperable Value Reduction Loss (Impairment) - Taxes to Recover - Long Term","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Tributos a Recuperar - Longo Prazo"
"account_template_102010901","1.02.01.09.01","Rent Paid in Advance - Long Term","asset_non_current","","Alugueis pagos Antecipadamente - Longo Prazo"
"account_template_102010902","1.02.01.09.02","Insurance Premiums Payable - Long Term","asset_non_current","","Prêmios de Seguros a Apropriar - Longo Prazo"
"account_template_102010903","1.02.01.09.03","Financial Charges to be Appropriated – Long Term","asset_non_current","","Encargos Financeiros a Apropriar –Longo Prazo"
"account_template_102010909","1.02.01.09.09","Other Costs and Expenses Paid in Advance - Long Term","asset_non_current","","Outros Custos e Despesas Pagos Antecipadamente - Longo Prazo"
"account_template_102011001","1.02.01.10.01","Consumable Biological Asset - Animal Origin – For Fair Value - Long Term","asset_non_current","","Ativo Biológico Consumível - Origem Animal – Pelo Valor Justo - Longo Prazo"
"account_template_102011002","1.02.01.10.02","Consumable Biological Asset - Plant Origin – For Fair Value - Long Term","asset_non_current","","Ativo Biológico Consumível - Origem Vegetal – Pelo Valor Justo - Longo Prazo"
"account_template_102011010","1.02.01.10.10","Consumable Biological Asset - Animal Origin - For Cost - Long Term","asset_non_current","","Ativo Biológico Consumível - Origem Animal - Pelo Custo - Longo Prazo"
"account_template_102011011","1.02.01.10.11","Consumable Biological Asset - Plant Origin - For Cost - Long Term","asset_non_current","","Ativo Biológico Consumível - Origem Vegetal - Pelo Custo - Longo Prazo"
"account_template_102011055","1.02.01.10.55","( - ) Losses for Recuperable Value (Impairment) - Consumable Biological Assets – Long Term","asset_non_current","","( - ) Perdas por Redução ao Valor Recuperável (Impairment) - Ativos Biológicos Consumível – Longo Prazo"
"account_template_102011070","1.02.01.10.70","Subaccount - Fair Value Adjustment (AVJ) – Biological Assets - Long Term","asset_non_current","","Subconta - Ajuste a Valor Justo (AVJ) – Ativos Biológicos - Longo Prazo"
"account_template_102011075","1.02.01.10.75","Subaccount – Present Value Adjustment (AVP) - Biological Assets - Long Term","asset_non_current","","Subconta – Ajuste a Valor Presente (AVP) - Ativos Biológicos - Longo Prazo"
"account_template_102011090","1.02.01.10.90","Subaccount – Initial Adoption – Biological Assets – Long Term","asset_non_current","","Subconta – Adoção Inicial – Ativo Biológico – Longo Prazo"
"account_template_102011501","1.02.01.15.01","Other Credits - Long Term","asset_non_current","","Outros Créditos - Longo Prazo"
"account_template_102011560","1.02.01.15.60","CPC 47 - Contract Targets - Long Term","asset_non_current","","CPC 47 - Atvos de Contrato - Longo Prazo"
"account_template_102020101","1.02.02.01.01","Permanent Stakes in Affiliates - In the Country","asset_non_current","","Participações Permanentes em Controladas - no País"
"account_template_102020104","1.02.02.01.04","Permanent Stakes in Affiliates - In the Country","asset_non_current","","Participações Permanentes em Controladas - no País"
"account_template_102020105","1.02.02.01.05","Permanent Participations in Joint Ventures and Specific Purpose Societies (SPE) - in the Country","asset_non_current","","Participações Permanentes em Joint Ventures e Sociedades de Propósito Específico (SPE) - no País"
"account_template_102020106","1.02.02.01.06","Participation Permanent in Other Societies of the same Group or Common Control - Evaluated by the Equity Method (MEP) - in the Country","asset_non_current","","Participações Permanentes em Outras Sociedades do Mesmo Grupo ou Controle Comum - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no País"
"account_template_102020107","1.02.02.01.07","Holdings in Limited Partnerships (SCP) - Ostensive Partner - Evaluated by the Equity Method (MEP) - in the Country","asset_non_current","","Participações em Sociedades em Conta de Participação (SCP) - Sócio Ostensivo - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no País"
"account_template_102020108","1.02.02.01.08","Holdings in Limited Partnerships (SCP) - Participating Partner - In the Country","asset_non_current","","Participações em Sociedades em Conta de Participação (SCP) - Sócio Participante - no País"
"account_template_102020110","1.02.02.01.10","Goodwill in Investments - In the Country","asset_non_current","","Goodwill em Investimentos - no País"
"account_template_102020111","1.02.02.01.11","Capital Gains on Investments - In the Country","asset_non_current","","Mais Valia em Investimentos - no País"
"account_template_102020112","1.02.02.01.12","(-) Loss of Value in Investments - in the Country","asset_non_current","","(-) Menos Valia em Investimentos - no País"
"account_template_102020120","1.02.02.01.20","Goodwill on Investments Generated until 31/12/2009 - in the Country","asset_non_current","","Ágios em Investimentos Gerados até 31/12/2009 - no País"
"account_template_102020121","1.02.02.01.21","(-) Negative Goodwill on Investments Generated until 31/12/2009 - in the Country","asset_non_current","","(-) Deságios em Investimentos Gerados até 31/12/2009 - no País"
"account_template_102020140","1.02.02.01.40","(-) Profits to be Appropriated from Sales with Subsidiaries -In the Country","asset_non_current","","(-) Lucros a Apropriar em Vendas com Controladas -no País"
"account_template_102020141","1.02.02.01.41","(-) Profits to be Appropriated on Sales with Affiliates - In the Country","asset_non_current","","(-) Lucros a Apropriar em Vendas com Coligadas -no País"
"account_template_102020142","1.02.02.01.42","(-) Profits to be Appropriated in Sales with Joint Ventures - In the Country","asset_non_current","","(-) Lucros a Apropriar em Vendas com Joint Ventures -no País"
"account_template_102020143","1.02.02.01.43","(-) Profits to be appropriated from Sales with other companies - Evaluated by Equity Method of Accounting (MEP) - in the Country","asset_non_current","","(-) Lucros a Apropriar em Vendas com outras sociedades - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no País"
"account_template_102020155","1.02.02.01.55","(-) Losses due to Reduction to the Recoverable Value (Impairment) of Interests Permanent Assured by the Equity Method - in the Country","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) de Participações Permanentes Avaliadas pelo Método de Equivalência Patrimonial - no País"
"account_template_102020156","1.02.02.01.56","(-) Goodwill Refundable Value (Impairment) Losses in Investments - in the Country","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) do Goodwill em Investimentos - no País"
"account_template_102020160","1.02.02.01.60","Subaccount - Fair Value Adjustment (AVJ) Reflected - Gain or Loss in Invested","asset_non_current","","Subconta - Ajuste a Valor Justo (AVJ) Reflexo - Ganho ou Perda na Investida"
"account_template_102020165","1.02.02.01.65","Subaccount - Fair Value Adjustment (AVJ) of Capital Subscription - Gain or Loss of Capital","asset_non_current","","Subconta - Ajuste a Valor Justo (AVJ) de Subscrição de Capital - Ganho ou Perda de Capital"
"account_template_102020175","1.02.02.01.75","(-) Subaccount - Present Value Adjustment (AVP) of Corporate Participation","asset_non_current","","(-) Subconta - Ajuste a Valor Presente (AVP) de Participação Societária"
"account_template_102020180","1.02.02.01.80","Subaccount - Surplus Value of Previous Participation - Stages","asset_non_current","","Subconta - Mais Valia da Participação Anterior - Estágios"
"account_template_102020181","1.02.02.01.81","(-) Subaccount - Less Value of Previous Holding - Stages","asset_non_current","","(-) Subconta - Menos Valia da Participação Anterior - Estágios"
"account_template_102020182","1.02.02.01.82","Subaccount - Goodwill on Previous Participation - Stages","asset_non_current","","Subconta - Goodwill da Participação Anterior - Estágios"
"account_template_102020184","1.02.02.01.84","Subaccount - Variation in Capital Gains of Previous Holdings - Stages","asset_non_current","","Subconta - Variação de Mais Valia da Participação Anterior - Estágios"
"account_template_102020185","1.02.02.01.85","(-) Subaccount - Variation of Losses on Previous Participation - Stages","asset_non_current","","(-) Subconta - Variação de Menos Valia da Participação Anterior - Estágios"
"account_template_102020186","1.02.02.01.86","Subaccount - Variation of Goodwill on Previous Participation - Stages","asset_non_current","","Subconta - Variação de Goodwill da Participação Anterior - Estágios"
"account_template_102020201","1.02.02.02.01","Permanent Stakes in Affiliates - Abroad","asset_non_current","","Participações Permanentes em Controladas - no Exterior"
"account_template_102020202","1.02.02.02.02","Subaccount - Taxation on a Universal Basis (TBU) - Direct Subsidiaries - Abroad","asset_non_current","","Subconta - Tributação em Base Universais (TBU) - Controladas Diretas - No Exterior"
"account_template_102020203","1.02.02.02.03","Subaccount - Taxation on a Universal Basis (TBU) - Indirect Subsidiaries - Abroad","asset_non_current","","Subconta - Tributação em Base Universais (TBU) - Controladas Indiretas - no Exterior"
"account_template_102020204","1.02.02.02.04","Permanent Interest in Associates - Valued by the Equity Method (MEP) - Abroad","asset_non_current","","Participações Permanentes em Coligadas - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no Exterior"
"account_template_102020205","1.02.02.02.05","Permanent Interests in Joint Ventures - Valued by the Equity Method - Abroad","asset_non_current","","Participações Permanentes em Joint Ventures - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no Exterior"
"account_template_102020206","1.02.02.02.06","Permanent Participation in Other Companies of the Same Group or Common Control - Measured by the Equity Method (MEP) - Abroad","asset_non_current","","Participações Permanentes em Outras Sociedades do Mesmo Grupo ou Controle Comum - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no Exterior"
"account_template_102020210","1.02.02.02.10","Goodwill in Investments - Abroad","asset_non_current","","Goodwill em Investimentos - no Exterior"
"account_template_102020211","1.02.02.02.11","Capital Gains on Investments - Abroad","asset_non_current","","Mais Valia em Investimentos - no Exterior"
"account_template_102020212","1.02.02.02.12","(-) Loss of Value in Investments - Abroad","asset_non_current","","(-) Menos Valia em Investimentos - no Exterior"
"account_template_102020220","1.02.02.02.20","Goodwill on Investments Generated until 31/12/2009 - Abroad","asset_non_current","","Ágios em Investimentos Gerados até 31/12/2009 - no Exterior"
"account_template_102020221","1.02.02.02.21","(-) Negative Goodwill on Investments Generated until 31/12/2009 - Abroad","asset_non_current","","(-) Deságios em Investimentos Gerados até 31/12/2009 - no Exterior"
"account_template_102020240","1.02.02.02.40","(-) Profits to be Appropriated from Sales with Subsidiaries - Abroad","asset_non_current","","(-) Lucros a Apropriar em Vendas com Controladas - no Exterior"
"account_template_102020241","1.02.02.02.41","(-) Profits to be Appropriated on Sales with Affiliates - Abroad","asset_non_current","","(-) Lucros a Apropriar em Vendas com Coligadas - no Exterior"
"account_template_102020242","1.02.02.02.42","(-) Profits to be Appropriated in Sales with Joint Ventures - Abroad","asset_non_current","","(-) Lucros a Apropriar em Vendas com Joint Ventures - no Exterior"
"account_template_102020243","1.02.02.02.43","(-) Profits to be Appropriated from Sales with other companies - Evaluated by Equity Method of Accounting (MEP) - Abroad","asset_non_current","","(-) Lucros a Apropriar em Vendas com outras sociedades - Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no Exterior"
"account_template_102020255","1.02.02.02.55","(-) Losses by Recuperable Value Reduction (Impairment) on Permanent Participations Valued by the Equity Method (MEP) - Abroad","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) em Participações Permanentes Avaliadas pelo Método de Equivalência Patrimonial (MEP) - no Exterior"
"account_template_102020256","1.02.02.02.56","(-) Goodwill Recuperable Value (Impairment) Losses in Investments - Abroad","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) do Goodwill em Investimentos - no Exterior"
"account_template_102020260","1.02.02.02.60","Subaccount - Fair Value Adjustment (AVJ) Reflected - Gain or Loss in Invested – Abroad","asset_non_current","","Subconta - Ajuste a Valor Justo (AVJ) Reflexo - Ganho ou Perda na Investida – no Exterior"
"account_template_102020265","1.02.02.02.65","Subaccount - Fair Value Adjustment (AVJ) of Capital Subscription - Gain or Loss of Capital - Abroad","asset_non_current","","Subconta - Ajuste a Valor Justo (AVJ) de Subscrição de Capital - Ganho ou Perda de Capital – no Exterior"
"account_template_102020275","1.02.02.02.75","(-) Subaccount - Present Value Adjustment (AVP) of Corporate Participation – Abroad","asset_non_current","","(-) Subconta - Ajuste a Valor Presente (AVP) de Participação Societária – no Exterior"
"account_template_102020280","1.02.02.02.80","Subaccount - Surplus Value of Previous Participation - Stages – Abroad","asset_non_current","","Subconta - Mais Valia da Participação Anterior - Estágios – no Exterior"
"account_template_102020281","1.02.02.02.81","(-) Subaccount - Less Value of Previous Holding - Stages – Abroad","asset_non_current","","(-) Subconta - Menos Valia da Participação Anterior - Estágios – no Exterior"
"account_template_102020282","1.02.02.02.82","Subaccount - Goodwill on Previous Participation - Stages – Abroad","asset_non_current","","Subconta - Goodwill da Participação Anterior - Estágios – no Exterior"
"account_template_102020284","1.02.02.02.84","Subaccount - Variation in Capital Gains of Previous Holdings - Stages – Abroad","asset_non_current","","Subconta - Variação de Mais Valia da Participação Anterior - Estágios – no Exterior"
"account_template_102020285","1.02.02.02.85","(-) Subaccount - Variation of Losses on Previous Participation - Stages – Abroad","asset_non_current","","(-) Subconta - Variação de Menos Valia da Participação Anterior - Estágios – no Exterior"
"account_template_102020286","1.02.02.02.86","Subaccount - Variation of Goodwill on Previous Participation - Stages – Abroad","asset_non_current","","Subconta - Variação de Goodwill da Participação Anterior - Estágios – no Exterior"
"account_template_102020301","1.02.02.03.01","Real Estate Owned in Construction - Investment Properties","asset_non_current","","Imóveis Próprios em Construção - Propriedades para Investimento"
"account_template_102020305","1.02.02.03.05","Real Estate – By Cost - Investment Properties","asset_non_current","","Imóveis Próprios – Pelo Custo - Propriedades para Investimento"
"account_template_102020306","1.02.02.03.06","Real Estate – Fair Value - Investment Properties","asset_non_current","","Imóveis Próprios – Valor Justo - Propriedades para Investimento"
"account_template_102020308","1.02.02.03.08","Real Estate Rental Object – Investment Properties","asset_non_current","","Imóveis Objeto de Arrendamento – Propriedades para Investimento"
"account_template_102020330","1.02.02.03.30","(-) Accrued Depreciation – Investment Properties","asset_non_current","","(-) Depreciação Acumuladas – Propriedades para Investimento"
"account_template_102020355","1.02.02.03.55","(-) Refundable Value Losses (Impairment) - Investment Properties","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Propriedades para Investimento"
"account_template_102020370","1.02.02.03.70","Subaccount - Fair Value Adjustment – Investment Properties","asset_non_current","","Subconta - Ajuste a Valor Justo – Propriedades para Investimentos"
"account_template_102020371","1.02.02.03.71","Subaccount - Fair Value Adjustment – Accrued Depreciation – Property for Investment","asset_non_current","","Subconta - Ajuste Valor Justo – Depreciação Acumulada – Propriedade para Investimento"
"account_template_102020375","1.02.02.03.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Investment Properties","asset_non_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Propriedades para Investimento"
"account_template_102020376","1.02.02.03.76","Subaccount - Present Value Adjustment – Accrued Depreciation - Investment Properties","asset_non_current","","Subconta - Ajuste Valor Presente – Depreciação Acumulada - Propriedades para Investimento"
"account_template_102020390","1.02.02.03.90","Subaccount – Initial Adoption – Investment Properties","asset_non_current","","Subconta – Adoção Inicial – Propriedades para Investimento"
"account_template_102020391","1.02.02.03.91","Subaccount – Initial Adoption – Accumulated Depreciation - Properties for Investment","asset_non_current","","Subconta – Adoção Inicial – Depreciação Acumulada - Propriedades para Investimento"
"account_template_102020395","1.02.02.03.95","Subaccount – Initial Adoption - Different Depreciation Rate - Investment Properties","asset_non_current","","Subconta – Adoção Inicial - Taxa de Depreciação Diferente - Propriedades para Investimento"
"account_template_102021003","1.02.02.10.03","Investments Deriving from Tax Incentives","asset_non_current","","Investimentos Decorrentes de Incentivos Fiscais"
"account_template_102021010","1.02.02.10.10","Other Permanent Investments","asset_non_current","","Outros Investimentos Permanentes"
"account_template_102021020","1.02.02.10.20","(-) Other Adjusting Accounts - Other Permanent Investments","asset_non_current","","(-) Outras Contas Retificadoras ­– Outros Investimentos Permanentes"
"account_template_102021050","1.02.02.10.50","( - ) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) – Other Permanent Investments","asset_non_current","","( - ) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) – Outros Investimentos Permanentes"
"account_template_102021055","1.02.02.10.55","(-) Refundable Value Losses (Impairment) - Other Investments","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Outros Investimentos"
"account_template_102021070","1.02.02.10.70","Subaccount - Fair Value Adjustment - Other Permanent Investments","asset_non_current","","Subconta - Ajuste a Valor Justo - Outros Investimentos Permanentes"
"account_template_102021090","1.02.02.10.90","Subaccount – Initial Adoption - Other Permanent Investments","asset_non_current","","Subconta – Adoção Inicial - Outros Investimentos Permanentes"
"account_template_102030101","1.02.03.01.01","Land","asset_fixed","","Terrenos"
"account_template_102030102","1.02.03.01.02","Buildings and Constructions","asset_fixed","","Edifícios e Construções"
"account_template_102030103","1.02.03.01.03","Constructions in Progress - Real Estate","asset_fixed","","Construções em Andamento - Imóvel Próprio"
"account_template_102030104","1.02.03.01.04","Other Tangible Fixed Assets in Progress","asset_fixed","","Outras Imobilizações em Andamento"
"account_template_102030105","1.02.03.01.05","Goods in Third Party Real Estate","asset_fixed","","Benfeitorias em Imóveis de Terceiros"
"account_template_102030106","1.02.03.01.06","Machinery, Equipment and Industrial Facilities","asset_fixed","","Máquinas, Equipamentos e Instalações Industriais"
"account_template_102030107","1.02.03.01.07","Furniture, Kitchenware and Commercial Facilities","asset_fixed","","Móveis, Utensílios e Instalações Comerciais"
"account_template_102030108","1.02.03.01.08","Vehicles","asset_fixed","","Veículos"
"account_template_102030109","1.02.03.01.09","Vessels","asset_fixed","","Embarcações"
"account_template_102030110","1.02.03.01.10","Aircraft","asset_fixed","","Aeronaves"
"account_template_102030111","1.02.03.01.11","Mineral Resources","asset_fixed","","Recursos Minerais"
"account_template_102030112","1.02.03.01.12","Ducts and Pipes","asset_fixed","","Dutos e Tubulações"
"account_template_102030113","1.02.03.01.13","Electricity Transmission Lines","asset_fixed","","Linhas de Transmissão Elétrica"
"account_template_102030114","1.02.03.01.14","Antennas and Transmission Towers","asset_fixed","","Antenas e Torres de Transmissão"
"account_template_102030115","1.02.03.01.15","Employees in Rural Activity","asset_fixed","","Máquinas Empregadas na Atividade Rural"
"account_template_102030116","1.02.03.01.16","Tractors and Vehicles Employees in Rural Activity","asset_fixed","","Tratores e Demais Veículos Empregados na Atividade Rural"
"account_template_102030128","1.02.03.01.28","Other Fixed Assets for Acquisition","asset_fixed","","Outras Imobilizações por Aquisição"
"account_template_102030130","1.02.03.01.30","(-) Accrued Depreciation - Fixed Assets","asset_fixed","","(-) Depreciação Acumulada - Imobilizado"
"account_template_102030131","1.02.03.01.31","(-) Accumulated Amortization - Fixed Assets","asset_fixed","","(-) Amortização Acumulada - Imobilizado"
"account_template_102030132","1.02.03.01.32","(-) Accumulated Depletion - Fixed Assets","asset_fixed","","(-) Exaustão Acumulada - Imobilizado"
"account_template_102030155","1.02.03.01.55","(-) Refundable Value Losses (Impairment) - Fixed Assets","asset_fixed","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Imobilizado"
"account_template_102030175","1.02.03.01.75","(-) Subaccount - Present Value Adjustment – Fixed Assets","asset_fixed","","(-) Subconta - Ajuste Valor Presente – Imobilizado"
"account_template_102030176","1.02.03.01.76","Subaccount - Present Value Adjustment – Accrued Depreciation - Fixed Assets","asset_fixed","","Subconta - Ajuste Valor Presente – Depreciação Acumulada - Imobilizado"
"account_template_102030177","1.02.03.01.77","Subaccount - Present Value Adjustment – Accrued Amortization - Fixed Assets","asset_fixed","","Subconta - Ajuste Valor Presente – Amortização Acumulada - Imobilizado"
"account_template_102030178","1.02.03.01.78","Subaccount - Present Value Adjustment – Accumulated Depletion - Fixed Assets","asset_fixed","","Subconta - Ajuste Valor Presente – Exaustão Acumulada - Imobilizado"
"account_template_102030190","1.02.03.01.90","Subaccount – Initial Adoption - Fixed Assets","asset_fixed","","Subconta – Adoção Inicial - Imobilizado"
"account_template_102030191","1.02.03.01.91","Subaccount – Initial Adoption – Accumulated Depreciation - Fixed Assets","asset_fixed","","Subconta – Adoção Inicial – Depreciação Acumulada - Imobilizado"
"account_template_102030192","1.02.03.01.92","Subaccount – Initial Adoption – Accumulated Amortization - Fixed Assets","asset_fixed","","Subconta – Adoção Inicial – Amortização Acumulada - Imobilizado"
"account_template_102030193","1.02.03.01.93","Subaccount – Initial Adoption – Accumulated Depletion - Fixed Assets","asset_fixed","","Subconta – Adoção Inicial – Exaustão Acumulada - Imobilizado"
"account_template_102030195","1.02.03.01.95","Subaccount – Initial Adoption – Different Depreciation Rate - Fixed Assets","asset_fixed","","Subconta – Adoção Inicial – Taxa Depreciação Diferente - Imobilizado"
"account_template_102030201","1.02.03.02.01","Vehicles","asset_fixed","","Veículos"
"account_template_102030202","1.02.03.02.02","Vessels","asset_fixed","","Embarcações"
"account_template_102030203","1.02.03.02.03","Aircraft","asset_fixed","","Aeronaves"
"account_template_102030204","1.02.03.02.04","Machinery, Equipment and Industrial Facilities","asset_fixed","","Máquinas, Equipamentos e Instalações Industriais"
"account_template_102030205","1.02.03.02.05","Furniture, Kitchenware and Commercial Facilities","asset_fixed","","Móveis, Utensílios e Instalações Comerciais"
"account_template_102030206","1.02.03.02.06","Real estate","asset_fixed","","Imóveis"
"account_template_102030209","1.02.03.02.09","Other Rentals","asset_fixed","","Outras Imobilizações por Arrendamento"
"account_template_102030230","1.02.03.02.30","(-) Accrued Depreciation - Fixed Assets - Property rental object","asset_fixed","","(-) Depreciação Acumulada - Imobilizado - Bens objeto de arrendamento"
"account_template_102030231","1.02.03.02.31","(-) Accumulated Amortization - Fixed Assets - Property rental object","asset_fixed","","(-) Amortização Acumulada - Imobilizado - Bens objeto de arrendamento"
"account_template_102030232","1.02.03.02.32","(-) Accumulated Depletion - Fixed Assets - Property rental object","asset_fixed","","(-) Exaustão Acumulada - Imobilizado - Bens objeto de arrendamento"
"account_template_102030255","1.02.03.02.55","(-) Refundable Value Losses (Impairment) - Fixed Assets - Rental property","asset_fixed","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Imobilizado - Bens objeto de arrendamento"
"account_template_102030401","1.02.03.04.01","Biological Production Asset - Animal Origin – For Fair Value","asset_non_current","","Ativo Biológico de Produção - Origem Animal – Pelo Valor Justo"
"account_template_102030402","1.02.03.04.02","Biological Production Asset - Plant Origin – For Fair Value","asset_non_current","","Ativo Biológico de Produção - Origem Vegetal – Pelo Valor Justo"
"account_template_102030403","1.02.03.04.03","Biological Production Asset - Animal Origin – For Cost","asset_non_current","","Ativo Biológico de Produção - Origem Animal – Pelo Custo"
"account_template_102030404","1.02.03.04.04","Biological Production - Plant Origin - By Cost","asset_non_current","","Ativo Biológico de Produção - Origem Vegetal - Pelo Custo"
"account_template_102030430","1.02.03.04.30","(-) Accrued Depreciation – Biological Production Assets","asset_non_current","","(-) Depreciação Acumulada Ativos Biológicos de Produção"
"account_template_102030432","1.02.03.04.32","(-) Accumulated Depletion Biological Production Assets","asset_non_current","","(-) Exaustão Acumulada Ativos Biológicos de Produção"
"account_template_102030455","1.02.03.04.55","(-) Recuperable Value Reduction Losses (Impairment) - Biological Production Assets","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Ativos Biológicos de Produção"
"account_template_102030470","1.02.03.04.70","Subaccount - Fair Value Adjustment (AVJ) – Biological Production Assets","asset_non_current","","Subconta - Ajuste a Valor Justo (AVJ) – Ativos Biológicos de Produção"
"account_template_102030475","1.02.03.04.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Biological Production Assets","asset_non_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Ativos Biológicos de Produção"
"account_template_102030476","1.02.03.04.76","Subaccount - Present Value Adjustment – Accrued Depreciation - Biological Production Assets","asset_non_current","","Subconta - Ajuste Valor Presente – Depreciação Acumulada - Ativos Biológicos de Produção"
"account_template_102030478","1.02.03.04.78","Subaccount - Present Value Adjustment – Accumulated Depletion - Biological Production Assets","asset_non_current","","Subconta - Ajuste Valor Presente – Exaustão Acumulada - Ativos Biológicos de Produção"
"account_template_102030490","1.02.03.04.90","Subaccount – Initial Adoption - Biological Production Assets","asset_non_current","","Subconta – Adoção Inicial - Ativos Biológicos de Produção"
"account_template_102030491","1.02.03.04.91","Subaccount – Initial Adoption – Accumulated Depreciation – Biological Production Assets","asset_non_current","","Subconta – Adoção Inicial – Depreciação Acumulada – Ativo Biológico de Produção"
"account_template_102030493","1.02.03.04.93","Subaccount – Initial Adoption – Accumulated Depletion – Biological Production Assets","asset_non_current","","Subconta – Adoção Inicial – Exaustão Acumulada – Ativo Biológico de Produção"
"account_template_102030495","1.02.03.04.95","Subaccount – Initial Adoption – Different Depreciation Rate - Biological Production Assets","asset_non_current","","Subconta – Adoção Inicial – Taxa Depreciação Diferente - Ativos Biológicos de Produção"
"account_template_102030503","1.02.03.05.03","Fixed Assets Received as Government Grants","asset_non_current","","Imobilizados Recebidos em Subvenções Governamentais"
"account_template_102030504","1.02.03.05.04","(-) Reduction of Fixed Assets Received in Government Subsidies","asset_non_current","","(-) Redutoras de Imobilizados Recebidos em Subvenções Governamentais"
"account_template_102030528","1.02.03.05.28","Other Real Estate","asset_non_current","","Outros Imobilizados"
"account_template_102030529","1.02.03.05.29","(-) Other Fixed Asset Reducing Accounts","asset_non_current","","(-) Outras Contas Redutoras do Imobilizado"
"account_template_102030530","1.02.03.05.30","(-) Other Accumulated Depreciation, Amortisation and Depletion","asset_non_current","","(-) Outras Depreciações, Amortizações e Quotas de Exaustão Acumuladas"
"account_template_102030570","1.02.03.05.70","Subaccount - Fair Value Adjustment – Other Fixed Assets","asset_non_current","","Subconta - Ajuste a Valor Justo – Outros Imobilizados"
"account_template_102030575","1.02.03.05.75","(-) Subaccount - Present Value Adjustment – Fixed Assets","asset_non_current","","(-) Subconta - Ajuste Valor Presente – Imobilizado"
"account_template_102030576","1.02.03.05.76","Subaccount - Present Value Adjustment – Depreciation, Amort., Accrued Depletion – Other Fixed Assets","asset_non_current","","Subconta - Ajuste Valor Presente – Depreciação, Amort., Exaustão Acumulada – Outros Imobilizados"
"account_template_102030590","1.02.03.05.90","Subaccount – Initial Adoption - Other Real Estate","asset_non_current","","Subconta – Adoção Inicial - Outros Imobilizados"
"account_template_102030591","1.02.03.05.91","Subaccount – Initial Adoption – Depreciation, Amort., Accumulated Depletion – Other Fixed Assets","asset_non_current","","Subconta – Adoção Inicial – Depreciação, Amort., Exaustão Acumulada – Outros Imobilizados"
"account_template_102030592","1.02.03.05.92","Subaccount – Initial Adoption – Accumulated Amortization - Intangible Assets","asset_non_current","","Subconta – Adoção Inicial – Amortização Acumulada - Intangível"
"account_template_102030595","1.02.03.05.95","Subaccount – Initial Adoption – Different Depreciation Rate - Other Fixed Assets","asset_non_current","","Subconta – Adoção Inicial – Taxa Depreciação Diferente - Outros Imobilizados"
"account_template_102050101","1.02.05.01.01","Brands","asset_non_current","","Marcas"
"account_template_102050102","1.02.05.01.02","Industrial Patents and Secrets","asset_non_current","","Patentes e Segredos Industriais"
"account_template_102050103","1.02.05.01.03","Public Services Exploration Rights","asset_non_current","","Direitos de Exploração de Serviços Públicos"
"account_template_102050104","1.02.05.01.04","Forest Resources Exploration Rights","asset_non_current","","Direitos de Exploração de Recursos Florestais"
"account_template_102050105","1.02.05.01.05","Mineral Resources Exploration Rights","asset_non_current","","Direitos de Exploração de Recursos Minerais"
"account_template_102050106","1.02.05.01.06","Water Resources Exploration Rights","asset_non_current","","Direitos de Exploração de Recursos Hídricos"
"account_template_102050107","1.02.05.01.07","Copyright","asset_non_current","","Direitos Autorais"
"account_template_102050108","1.02.05.01.08","Cultural heritage","asset_non_current","","Patrimônio Cultural"
"account_template_102050109","1.02.05.01.09","Trade Fund","asset_non_current","","Fundo de Comércio"
"account_template_102050110","1.02.05.01.10","Computer Software or Programs","asset_non_current","","Software ou Programas de Computador"
"account_template_102050111","1.02.05.01.11","Rental Contracts","asset_non_current","","Contratos de Aluguel"
"account_template_102050112","1.02.05.01.12","Franchise Agreements","asset_non_current","","Contratos de Franquias"
"account_template_102050113","1.02.05.01.13","Product or Service Development","asset_non_current","","Desenvolvimento de Produtos ou Serviços"
"account_template_102050114","1.02.05.01.14","Right Reacquired","asset_non_current","","Direito Readquirido"
"account_template_102050115","1.02.05.01.15","Operating Lease taken out by the Acquired on More Favourable Terms","asset_non_current","","Leasing Operacional Contratado pela Adquirida em Condições Mais Favoráveis"
"account_template_102050116","1.02.05.01.16","Intangible Assets Not Recognised in the Acquired","asset_non_current","","Intangíveis Não Reconhecidos na Adquirida"
"account_template_102050117","1.02.05.01.17","Intangible Assets Received in Government Grants","asset_non_current","","Intangíveis Recebidos em Subvenções Governamentais"
"account_template_102050118","1.02.05.01.18","(-) Reduction of Intangible Assets Received in Government Subsidies","asset_non_current","","(-) Redutora de Intangíveis Recebidos em Subvenções Governamentais"
"account_template_102050120","1.02.05.01.20","(-) Accumulated Amortization - Intangible Assets","asset_non_current","","(-) Amortização Acumulada - Intangível"
"account_template_102050121","1.02.05.01.21","Goodwill – Intangible Assets","asset_non_current","","Goodwill – Intangível"
"account_template_102050128","1.02.05.01.28","Other Intangible Assets","asset_non_current","","Outros Intangíveis"
"account_template_102050129","1.02.05.01.29","(-) Other Accounts Reducing Intangible Assets","asset_non_current","","(-) Outras Contas Redutoras do Intangível"
"account_template_102050155","1.02.05.01.55","(-) Losses by Recuperable Value (Impairment) - Intangible","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Intangível"
"account_template_102050156","1.02.05.01.56","(-) Goodwill Recuperable Value (Impairment) Losses - Intangible","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) do Goodwill - Intangível"
"account_template_102050175","1.02.05.01.75","( - ) Subaccount - Adjustment to Present Value (AVP) - Intangible","asset_non_current","","( - ) Subconta – Ajuste a Valor Presente (AVP) - Intangível"
"account_template_102050177","1.02.05.01.77","Subaccount - Present Value Adjustment – Accrued Amortization - Intangible Assets","asset_non_current","","Subconta - Ajuste Valor Presente – Amortização Acumulada - Intangível"
"account_template_102050190","1.02.05.01.90","Subaccount – Initial Adoption - Intangible Assets","asset_non_current","","Subconta – Adoção Inicial - Intangível"
"account_template_102050192","1.02.05.01.92","Subaccount – Initial Adoption – Accumulated Amortization - Intangible Assets","asset_non_current","","Subconta – Adoção Inicial – Amortização Acumulada - Intangível"
"account_template_102060101","1.02.06.01.01","Pre-Operational or Pre-Industrial Expenses - Deferred Assets","asset_non_current","","Despesas Pré-Operacionais ou Pré-Industriais – Ativo Diferido"
"account_template_102060102","1.02.06.01.02","Expenses with Scientific or Technological Research - Deferred Assets","asset_non_current","","Despesas com Pesquisas Científicas ou Tecnológicas – Ativo Diferido"
"account_template_102060103","1.02.06.01.03","Other Investments in Depreciable Expenses - Deferred Assets","asset_non_current","","Demais Aplicações em Despesas Amortizáveis – Ativo Diferido"
"account_template_102060131","1.02.06.01.31","(-) Accumulated Amortization - Deferred Assets","asset_non_current","","(-) Amortização Acumulada - Ativo Diferido"
"account_template_102060156","1.02.06.01.56","(-) Losses by Recuperable Value (Impairment) - Deferred Assets","asset_non_current","","(-) Perdas por Redução ao Valor Recuperável (Impairment) - Ativo Diferido"
"account_template_102060190","1.02.06.01.90","Subaccount – Initial Adoption – Deferred Assets","asset_non_current","","Subconta – Adoção Inicial – Ativo Diferido"
"account_template_201010101","2.01.01.01.01","Salaries and Remunerations Payable","liability_current","","Salários e Remunerações a Pagar"
"account_template_201010102","2.01.01.01.02","Profit Sharing Payable","liability_current","","Participações no Resultado a Pagar"
"account_template_201010103","2.01.01.01.03","INSS To Collect","liability_current","","INSS a Recolher"
"account_template_201010104","2.01.01.01.04","FGTS to Collect","liability_current","","FGTS a Recolher"
"account_template_201010105","2.01.01.01.05","Non-Monetary Benefits","liability_current","","Benefícios Não Monetários"
"account_template_201010150","2.01.01.01.50","Post-Employment Benefits","liability_current","","Benefícios Pós-Emprego"
"account_template_201010155","2.01.01.01.55","Other Long Term Benefits","liability_current","","Outros Benefícios de Longo Prazo"
"account_template_201010160","2.01.01.01.60","Rescission Benefits","liability_current","","Benefícios Rescisórios"
"account_template_201010109","2.01.01.01.09","Other Charges Payable","liability_current","","Demais Encargos a Recolher"
"account_template_201010301","2.01.01.03.01","Suppliers - Operations with Non-Related Parties - In the Country – Current","liability_payable","True","Fornecedores - Operações com Partes Não Relacionadas - No País – Circulante"
"account_template_201010302","2.01.01.03.02","Suppliers - Operations with Non-Related Parties - Abroad – Current","liability_payable","True","Fornecedores - Operações com Partes Não Relacionadas - No Exterior – Circulante"
"account_template_201010303","2.01.01.03.03","Suppliers - Operations with Related Parties - In the Country – Current","liability_payable","True","Fornecedores - Operações com Partes Relacionadas - No País – Circulante"
"account_template_201010304","2.01.01.03.04","Suppliers - Operations with Related Parties - Abroad – Current","liability_payable","True","Fornecedores - Operações com Partes Relacionadas - No Exterior – Circulante"
"account_template_201010350","2.01.01.03.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) - Current Suppliers","liability_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Fornecedores Circulante"
"account_template_201010390","2.01.01.03.90","Subaccount – Initial Adoption - Suppliers - Current","liability_current","","Subconta – Adoção Inicial - Fornecedores - Circulante"
"account_template_201010501","2.01.01.05.01","Advances from Customers - In the Country","liability_payable","True","Adiantamentos de Clientes - no País"
"account_template_201010502","2.01.01.05.02","Advances from Customers - Abroad","liability_payable","True","Adiantamentos de Clientes - no Exterior"
"account_template_201010550","2.01.01.05.50","(-) Unearned Interest Arising from Adjustment to Present Value (AVP) - Accounts Payable - Current","liability_payable","True","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Contas a Pagar - Circulante"
"account_template_201010590","2.01.01.05.90","Subaccount – Initial Adoption - Accounts Payable - Current","liability_payable","True","Subconta – Adoção Inicial - Contas a Pagar - Circulante"
"account_template_201010701","2.01.01.07.01","Discounted Trade Bills – Current","liability_current","","Duplicatas Descontadas – Circulante"
"account_template_201010702","2.01.01.07.02","Loans or Financing - In the Country - Current","liability_current","","Empréstimos ou Financiamentos - no País - Circulante"
"account_template_201010703","2.01.01.07.03","Loans or Financing - Abroad – Current","liability_current","","Empréstimos ou Financiamentos - no Exterior – Circulante"
"account_template_201010704","2.01.01.07.04","Exchange Contract Advances – Current","liability_current","","Adiantamentos de Contrato de Câmbio – Circulante"
"account_template_201010705","2.01.01.07.05","Rental - in the Country – Current","liability_current","","Arrendamento - no País – Circulante"
"account_template_201010706","2.01.01.07.06","Rent - Abroad - Current","liability_current","","Arrendamento - no Exterior - Circulante"
"account_template_201010750","2.01.01.07.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) - Loans and Financing - Current","liability_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Empréstimos e Financiamentos - Circulante"
"account_template_201010790","2.01.01.07.90","Subaccount – Initial Adoption - Loans and Financing - Current","liability_current","","Subconta – Adoção Inicial - Empréstimos e Financiamentos - Circulante"
"account_template_201010901","2.01.01.09.01","IRRF to Collect – Current","liability_current","","IRRF a Recolher – Circulante"
"account_template_201010902","2.01.01.09.02","IPI to Collect – Current","liability_current","","IPI a Recolher – Circulante"
"account_template_201010903","2.01.01.09.03","ICMS to Collect – Current","liability_current","","ICMS a Recolher – Circulante"
"account_template_201010904","2.01.01.09.04","PIS payable – Current","liability_current","","PIS a Recolher – Circulante"
"account_template_201010905","2.01.01.09.05","COFINS to Collect – Current","liability_current","","COFINS a Recolher – Circulante"
"account_template_201010906","2.01.01.09.06","IOF to Collect – Current","liability_current","","IOF a Recolher – Circulante"
"account_template_201010907","2.01.01.09.07","CIDE to collect – Current","liability_current","","CIDE a Recolher – Circulante"
"account_template_201010908","2.01.01.09.08","Municipal Taxes to Collect – Current","liability_current","","Tributos Municipais a Recolher – Circulante"
"account_template_201010909","2.01.01.09.09","Special Installments Payable - Federal Taxes – Current","liability_current","","Parcelamentos Especiais a Recolher - Tributos Federais – Circulante"
"account_template_201010910","2.01.01.09.10","Special Installments Payable - State and Municipal taxes – Current","liability_current","","Parcelamentos Especiais a Recolher - Tributos Estaduais e Municipais – Circulante"
"account_template_201010911","2.01.01.09.11","Social Security Contributions to be Paid - Payroll Exoneration – Current","liability_current","","Contribuições Previdenciárias a Recolher – Desoneração da Folha de Pagamento – Circulante"
"account_template_201010912","2.01.01.09.12","Taxes Retained to Collect – Current","liability_current","","Tributos Retidos a Recolher – Circulante"
"account_template_201010913","2.01.01.09.13","IRPJ to Collect – Current","liability_current","","IRPJ a Recolher – Circulante"
"account_template_201010914","2.01.01.09.14","CSLL to Collect – Current","liability_current","","CSLL a Recolher – Circulante"
"account_template_201010928","2.01.01.09.28","Other Taxes to Collect – Current","liability_current","","Outros Tributos a Recolher – Circulante"
"account_template_201011101","2.01.01.11.01","Derivatives - Hedge Fair Value - In the Country","liability_current","","Derivativos - Hedge Valor Justo - No País"
"account_template_201011102","2.01.01.11.02","Derivatives - Cash Flow Hedge - In the Country","liability_current","","Derivativos - Hedge Fluxo de Caixa - No País"
"account_template_201011103","2.01.01.11.03","Derivatives - Hedge Foreign Investment - In the Country","liability_current","","Derivativos - Hedge Investimento no Exterior - No País"
"account_template_201011170","2.01.01.11.70","Subaccount - Fair Value Adjustment - Securities – Hedge - In the Country","liability_current","","Subconta - Ajuste a Valor Justo - Valores Mobiliários – Hedge - No País"
"account_template_201011190","2.01.01.11.90","Subaccount – Initial Adoption - Securities – Hedge - In the Country","liability_current","","Subconta – Adoção Inicial - Valores Mobiliários – Hedge - No País"
"account_template_201011201","2.01.01.12.01","Derivatives - Hedge Fair Value - Abroad","liability_current","","Derivativos - Hedge Valor Justo - No Exterior"
"account_template_201011202","2.01.01.12.02","Derivatives - Cash Flow Hedge - Abroad","liability_current","","Derivativos - Hedge Fluxo de Caixa - No Exterior"
"account_template_201011203","2.01.01.12.03","Derivatives - Hedge Foreign Investment - Abroad","liability_current","","Derivativos - Hedge Investimento no Exterior - No Exterior"
"account_template_201011270","2.01.01.12.70","Subaccount - Fair Value Adjustment - Securities – Hedge - Abroad","liability_current","","Subconta - Ajuste a Valor Justo - Valores Mobiliários – Hedge - No Exterior"
"account_template_201011290","2.01.01.12.90","Subaccount – Initial Adoption - Securities – Hedge - Abroad","liability_current","","Subconta – Adoção Inicial - Valores Mobiliários – Hedge - No Exterior"
"account_template_201011301","2.01.01.13.01","Debentures Payable – Current","liability_current","","Debêntures a Pagar – Circulante"
"account_template_201011302","2.01.01.13.02","Premium in the Emission of Debentures – Current","liability_current","","Prêmio na Emissão de Debêntures – Circulante"
"account_template_201011304","2.01.01.13.04","Promissory Notes Payable","liability_current","","Notas Promissórias a Pagar"
"account_template_201011305","2.01.01.13.05","Bonds Paying","liability_current","","Bonds a Pagar"
"account_template_201011306","2.01.01.13.06","Real Estate Receiver Certificates - CRI","liability_current","","Certificados de Recebíveis Imobiliários - CRI"
"account_template_201011307","2.01.01.13.07","Certificates of Agribusiness Receivers - CRA","liability_current","","Certificados de Recebíveis do Agronegócio - CRA"
"account_template_201011325","2.01.01.13.25","Other Debt Securities Payable – For Amortized Cost - Current","liability_current","","Outros Títulos de Dívida a Pagar – Pelo Custo Amortizado - Circulante"
"account_template_201011327","2.01.01.13.27","(-) Costs to Amortize – Debt Securities - Current","liability_current","","(-) Custos a Amortizar – Títulos de Dívida - Circulante"
"account_template_201011328","2.01.01.13.28","Other Debt Securities Payable – For Fair Value (VJPR) - Current","liability_current","","Outros Títulos de Dívida a Pagar – Pelo Pelo Valor Justo(VJPR) - Circulante"
"account_template_201011329","2.01.01.13.29","(-) Negative goodwill to appropriate - Debt securities - Current","liability_current","","(-) Deságio a Apropriar – Títulos de Dívida - Circulante"
"account_template_201011350","2.01.01.13.50","(-) Accrued interest due to adjustment to present value (AVP) - Debt securities - Current","liability_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Títulos de Dívida - Circulante"
"account_template_201011370","2.01.01.13.70","Subaccount - Fair Value Adjustment – Debt securities payable - Current","liability_current","","Subconta - Ajuste a Valor Justo – Títulos de Dívida a Pagar - Circulante"
"account_template_201011390","2.01.01.13.90","Subaccount – Initial Adoption - Debt Securities - Current","liability_current","","Subconta – Adoção Inicial - Títulos de Dívida - Circulante"
"account_template_201011501","2.01.01.15.01","Provision for Income Tax","liability_current","","Provisão para o Imposto de Renda"
"account_template_201011502","2.01.01.15.02","Provision for Social Contribution on Net Income","liability_current","","Provisão para a Contribuição Social sobre o Lucro Líquido"
"account_template_201011503","2.01.01.15.03","Holidays Payable","liability_current","","Férias a Pagar"
"account_template_201011504","2.01.01.15.04","13th Salary Payable","liability_current","","13º Salário a Pagar"
"account_template_201011505","2.01.01.15.05","Labor Provisions - Current","liability_current","","Provisões de Natureza Trabalhista - Circulante"
"account_template_201011506","2.01.01.15.06","Tax Provisions – Current","liability_current","","Provisões de Natureza Tributária – Circulante"
"account_template_201011507","2.01.01.15.07","Provisions of a civil nature - Current","liability_current","","Provisões de Natureza Cível – Circulante"
"account_template_201011528","2.01.01.15.28","Other Provisions","liability_current","","Outras Provisões"
"account_template_201011550","2.01.01.15.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) - Provisions - Current","liability_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Provisões - Circulante"
"account_template_201011701","2.01.01.17.01","Mutual Loans – Non-Related Parties – In the Country - Current","liability_current","","Mútuos – Partes Não Relacionadas – No País - Ciculante"
"account_template_201011702","2.01.01.17.02","Mutual Loans - Non-Related Parties – Abroad - Current","liability_current","","Mútuos - Partes Não Relacionadas – No Exterior - Ciculante"
"account_template_201011703","2.01.01.17.03","Mutual Loans – Related Parties – In the Country – Current","liability_current","","Mútuos – Partes Relacionadas – No País – Circulante"
"account_template_201011704","2.01.01.17.04","Mutual Loans - Related Parties – Abroad - Current","liability_current","","Mútuos - Partes Relacionadas – No Exterior - Circulante"
"account_template_201011709","2.01.01.17.09","Contingent consideration Liabilities - Business Combination - Current","liability_current","","Contraprestação Contingente Passiva - Combinação de Negócios - Circulante"
"account_template_201011710","2.01.01.17.10","Contingent Liabilities Assumed in Business Combinations - Current","liability_current","","Passivo Contingente Assumido em Combinação de Negócios - Circulante"
"account_template_201011711","2.01.01.17.11","Future Delivery - Current","liability_current","","Faturamento para Entrega Futura - Circulante"
"account_template_201011712","2.01.01.17.12","Interest on Shareholders' Equity Payable - Current","liability_current","","Juros sobre o Capital Próprio a Pagar - Circulante"
"account_template_201011713","2.01.01.17.13","Dividends Payable – Current","liability_current","","Dividendos a Pagar – Circulante"
"account_template_201011715","2.01.01.17.15","Cost Control Account Hired - Current","liability_current","","Conta de Controle de Custo Contratado - Circulante"
"account_template_201011716","2.01.01.17.16","Cost Control Account Ordained - Current","liability_current","","Conta de Controle de Custo Orçado - Circulante"
"account_template_201011725","2.01.01.17.25","Creditor Rights Payable - Current","liability_current","","Direitos Creditórios a Pagar - Circulante"
"account_template_201011728","2.01.01.17.28","Other Liabilities – Current","liability_current","","Outras Obrigações – Circulante"
"account_template_201011750","2.01.01.17.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) – Accounts Payable - Current","liability_current","","(-)Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) – Outras Contas a Pagar - Circulante"
"account_template_201011760","2.01.01.17.60","CPC 47 - Contract Liabilities - Current","liability_current","","CPC 47 - Passivos de Contrato - Circulante"
"account_template_201011901","2.01.01.19.01","Deferred Income","liability_current","","Receitas Diferidas"
"account_template_201011902","2.01.01.19.02","(-) Costs Corresponding to Deferred Income","liability_current","","(-) Custos Correspondentes às Receitas Diferidas"
"account_template_201011903","2.01.01.19.03","Government Grants to be Appropriated","liability_current","","Subvenção Governamental a Apropriar"
"account_template_201011950","2.01.01.19.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) – Deferred Income","liability_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) – Receitas Diferidas"
"account_template_202010101","2.02.01.01.01","Suppliers - In the Country - Long Term","liability_non_current","","Fornecedores - No País - Longo Prazo"
"account_template_202010102","2.02.01.01.02","Suppliers - Abroad - Long Term","liability_non_current","","Fornecedores - No Exterior - Longo Prazo"
"account_template_202010103","2.02.01.01.03","Creditors for Financing - Long Term","liability_non_current","","Credores por Financiamento - Longo Prazo"
"account_template_202010104","2.02.01.01.04","Securities Payable - Long Term","liability_non_current","","Títulos a Pagar - Longo Prazo"
"account_template_202010105","2.02.01.01.05","Discounted Trade Bills - Long Term","liability_non_current","","Duplicatas Descontadas - Longo Prazo"
"account_template_202010106","2.02.01.01.06","Loans or Financing - In the Country - Long term","liability_non_current","","Empréstimos ou Financiamentos - no País - Longo Prazo"
"account_template_202010107","2.02.01.01.07","Loans or Financing - Abroad - Long Term","liability_non_current","","Empréstimos ou Financiamentos - no Exterior - Longo Prazo"
"account_template_202010108","2.02.01.01.08","Exchange Contract Advances - Long Term","liability_non_current","","Adiantamentos de Contrato de Câmbio - Longo Prazo"
"account_template_202010109","2.02.01.01.09","Lease In the Country - Long Term","liability_non_current","","Arrendamento no País - Longo Prazo"
"account_template_202010110","2.02.01.01.10","Lease Abroad - Long Term","liability_non_current","","Arrendamento no Exterior - Longo Prazo"
"account_template_202010111","2.02.01.01.11","Advances from Customers - In the Country – Long Term","liability_non_current","","Adiantamentos de Clientes - no País – Longo Prazo"
"account_template_202010112","2.02.01.01.12","Advances from Customers - Abroad – Long Term","liability_non_current","","Adiantamentos de Clientes - no Exterior – Longo Prazo"
"account_template_202010150","2.02.01.01.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) – Loans and Financing - Long Term","liability_non_current","","(-)Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) – Empréstimos e Financiamentos - Longo Prazo"
"account_template_202010170","2.02.01.01.70","Subaccount - Fair Value Adjustment - Loans and Financing - Long Term","liability_non_current","","Subconta - Ajuste a Valor Justo - Empréstimos e Financiamentos - Longo Prazo"
"account_template_202010190","2.02.01.01.90","Subaccount – Initial Adoption - Loans and Financing - Long Term","liability_non_current","","Subconta – Adoção Inicial - Empréstimos e Financiamentos - Longo Prazo"
"account_template_202010201","2.02.01.02.01","Post Employment Benefits - Long Term","liability_non_current","","Benefícios Pós Emprego - Longo Prazo"
"account_template_202010202","2.02.01.02.02","Other Long Term Benefits - Long Term","liability_non_current","","Outro Benefícios de Longo Prazo - Longo Prazo"
"account_template_202010203","2.02.01.02.03","Rescission Benefits - Long Term","liability_non_current","","Benefícios Rescisórios - Longo Prazo"
"account_template_202010301","2.02.01.03.01","Special and Ordinary Installments Payable - Federal Taxes - Long Term","liability_non_current","","Parcelamentos Especiais e Ordinários a Recolher - Tributos Federais - Longo Prazo"
"account_template_202010302","2.02.01.03.02","Special and Ordinary Installments Payable - State and Municipal Taxes - Long Term","liability_non_current","","Parcelamentos Especiais e Ordinários a Recolher - Tributos Estaduais e Municipais - Longo Prazo"
"account_template_202010328","2.02.01.03.28","Other Taxes to Collect - Long Term","liability_non_current","","Outros Tributos a Recolher - Longo Prazo"
"account_template_202010501","2.02.01.05.01","IRPJ - Temporary Differences - Long Term","liability_non_current","","Débitos Fiscais IRPJ - Diferenças Temporárias - Longo Prazo"
"account_template_202010502","2.02.01.05.02","CSLL - Temporary Differences - Long Term","liability_non_current","","Débitos Fiscais CSLL - Diferenças Temporárias - Longo Prazo"
"account_template_202010701","2.02.01.07.01","Debentures Payable - Long Term","liability_non_current","","Debêntures a Pagar - Longo Prazo"
"account_template_202010702","2.02.01.07.02","Premium on the Emission of Debentures - Long Term","liability_non_current","","Prêmio na Emissão de Debêntures - Longo Prazo"
"account_template_202010704","2.02.01.07.04","Promissory Notes Payable – Long Term","liability_non_current","","Notas Promissórias a Pagar – Longo Prazo"
"account_template_202010705","2.02.01.07.05","Bonds Paying","liability_non_current","","Bonds a Pagar"
"account_template_202010706","2.02.01.07.06","Real Estate Receiver Certificates (CRI) – Long Term","liability_non_current","","Certificados de Recebíveis Imobiliários(CRI) – Longo Prazo"
"account_template_202010707","2.02.01.07.07","Certificates of Agribusiness Receivers (CRA) – Long Term","liability_non_current","","Certificados de Recebíveis do Agronegócio(CRA) – Longo Prazo"
"account_template_202010725","2.02.01.07.25","Other Debt Securities Payable - For Amortised Cost - Long Term","liability_non_current","","Outros Títulos de Dívida a Pagar - – Pelo Custo Amortizado - Longo Prazo"
"account_template_202010727","2.02.01.07.27","(-) Costs to Amortize - Debt Securities - Long Term","liability_non_current","","(-) Custos a Amortizar - Títulos de Dívida - Longo Prazo"
"account_template_202010728","2.02.01.07.28","Other Debt Securities Payable – For Fair Value(VJPR) – Long Term","liability_non_current","","Outros Títulos de Dívida a Pagar – Pelo Pelo Valor Justo(VJPR) - – Longo Prazo"
"account_template_202010729","2.02.01.07.29","(-) Negative goodwill - Debentures - Long Term","liability_non_current","","(-) Deságio a Apropriar - Debêntures - Longo Prazo"
"account_template_202010770","2.02.01.07.70","Subaccount - Fair Value Adjustment – Debt securities payable – Long Term","liability_non_current","","Subconta - Ajuste a Valor Justo – Títulos de Dívida a Pagar - – Longo Prazo"
"account_template_202010901","2.02.01.09.01","Labor Provisions - Long Term","liability_non_current","","Provisões de Natureza Trabalhista - Longo Prazo"
"account_template_202010902","2.02.01.09.02","Tax Provisions - Long Term","liability_non_current","","Provisões de Natureza Tributária - Longo Prazo"
"account_template_202010903","2.02.01.09.03","Nature Forecasts - Long-term","liability_non_current","","Provisões de Natureza Cível - Longo Prazo"
"account_template_202010928","2.02.01.09.28","Other Provisions - Long Term","liability_non_current","","Outras Provisões - Longo Prazo"
"account_template_202011001","2.02.01.10.01","IRRF to Collect - Long Term","liability_non_current","","IRRF a Recolher - Longo Prazo"
"account_template_202011002","2.02.01.10.02","IPI to Collect - Long Term","liability_non_current","","IPI a Recolher - Longo Prazo"
"account_template_202011003","2.02.01.10.03","ICMS to Collect - Long Term","liability_non_current","","ICMS a Recolher - Longo Prazo"
"account_template_202011004","2.02.01.10.04","PIS payable - Long term","liability_non_current","","PIS a Recolher - Longo Prazo"
"account_template_202011005","2.02.01.10.05","COFINS to Collect - Long Term","liability_non_current","","COFINS a Recolher - Longo Prazo"
"account_template_202011006","2.02.01.10.06","IOF to Collect - Long Term","liability_non_current","","IOF a Recolher - Longo Prazo"
"account_template_202011007","2.02.01.10.07","CIDE to Collect - Long Term","liability_non_current","","CIDE a Recolher - Longo Prazo"
"account_template_202011008","2.02.01.10.08","Municipal Taxes to Collect - Long Term","liability_non_current","","Tributos Municipais a Recolher - Longo Prazo"
"account_template_202011009","2.02.01.10.09","Special Installments Payable - Federal Taxes – Long Term","liability_non_current","","Parcelamentos Especiais a Recolher - Tributos Federais – Longo Prazo"
"account_template_202011010","2.02.01.10.10","Special Installments Payable - State and Municipal taxes – Long Term","liability_non_current","","Parcelamentos Especiais a Recolher - Tributos Estaduais e Municipais – Longo Prazo"
"account_template_202011011","2.02.01.10.11","Contribution payable - Payroll Exoneration - Long Term","liability_non_current","","Contribuição a Recolher - Desoneração da Folha de Pagamento - Longo Prazo"
"account_template_202011028","2.02.01.10.28","Other Taxes to Collect - Long Term","liability_non_current","","Outros Tributos a Recolher - Longo Prazo"
"account_template_202011101","2.02.01.11.01","Mutual Loans - Non-Related Parties – Country - Long Term","liability_non_current","","Mútuos - Partes Não Relacionadas – No País - Longo Prazo"
"account_template_202011102","2.02.01.11.02","Mutual Loans - Non-Related Parties - Abroad - Long Term","liability_non_current","","Mútuos - Partes Não Relacionadas - No Exterior - Longo Prazo"
"account_template_202011103","2.02.01.11.03","Mutual Loans – Related Parties – In the Country - Long Term","liability_non_current","","Mútuos – Partes Relacionadas – No País - Longo Prazo"
"account_template_202011104","2.02.01.11.04","Mutual Loans - Related Parties – Abroad - Long Term","liability_non_current","","Mútuos - Partes Relacionadas – No Exterior - Longo Prazo"
"account_template_202011110","2.02.01.11.10","Contingent Liabilities Assumed in Business Combinations - Long Term","liability_non_current","","Passivo Contingente Assumido em Combinação de Negócios - Longo Prazo"
"account_template_202011113","2.02.01.11.13","Advance for Future Capital Increase - Liabilities - Long Term","liability_non_current","","Adiantamento para Futuro Aumento de Capital - Passivo - Longo Prazo"
"account_template_202011115","2.02.01.11.15","Contracted Cost Control Account - Long Term","liability_non_current","","Conta de Controle de Custo Contratado - Longo Prazo"
"account_template_202011116","2.02.01.11.16","Budgeted Cost Control Account - Long Term","liability_non_current","","Conta de Controle de Custo Orçado - Longo Prazo"
"account_template_202011122","2.02.01.11.22","Creditor Rights Payable – Long Term","liability_non_current","","Direitos Creditórios a Pagar – Longo Prazo"
"account_template_202011128","2.02.01.11.28","Other Liabilities - Long Term","liability_non_current","","Outras Obrigações - Longo Prazo"
"account_template_202011150","2.02.01.11.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) Other Obligations - Long Term","liability_non_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) Outras Obrigações - Longo Prazo"
"account_template_202011160","2.02.01.11.60","CPC 47 - Contract Liabilities - Long Term","liability_non_current","","CPC 47 - Passivos de Contrato - Longo Prazo"
"account_template_202011170","2.02.01.11.70","Subaccount - Fair Value Adjustment – Other Liabilities - Long Term","liability_non_current","","Subconta - Ajuste a Valor Justo – Outras Obrigações - Longo Prazo"
"account_template_202011190","2.02.01.11.90","Subaccount – Initial Adoption - Other Obligations - Long Term","liability_non_current","","Subconta – Adoção Inicial - Outras Obrigações - Longo Prazo"
"account_template_202012101","2.02.01.21.01","Deferred Income","liability_non_current","","Receitas Diferidas"
"account_template_202012102","2.02.01.21.02","(-) Costs Corresponding to Deferred Income","liability_non_current","","(-) Custos Correspondentes às Receitas Diferidas"
"account_template_202012103","2.02.01.21.03","Government Grants to be Appropriated","liability_non_current","","Subvenção Governamental a Apropriar"
"account_template_202012150","2.02.01.21.50","(-) Interest to be Appropriated Arising from Adjustment to Present Value (AVP) - Revenue Deferred - Long Term","liability_non_current","","(-) Juros a Apropriar Decorrentes de Ajuste a Valor Presente (AVP) - Receita Diferida - Longo Prazo"
"account_template_202012170","2.02.01.21.70","Subaccount - Fair Value Adjustment – Deferred Income - Long Term","liability_non_current","","Subconta - Ajuste a Valor Justo – Receita Diferida - Longo Prazo"
"account_template_202012190","2.02.01.21.90","Subaccount – Initial Adoption - Deferred Income - Long Term","liability_non_current","","Subconta – Adoção Inicial - Receita Diferida - Longo Prazo"
"account_template_203010101","2.03.01.01.01","Subscribed Capital of Households and Residents in the Country","equity","","Capital Subscrito de Domiciliados e Residentes no País"
"account_template_203010121","2.03.01.01.21","(-) Capital to Integralize of Households and Residents in the Country","equity","","(-) Capital a Integralizar de Domiciliados e Residentes no País"
"account_template_203010201","2.03.01.02.01","Subscribed Capital of Households and Residents Abroad","equity","","Capital Subscrito de Domiciliados e Residentes no Exterior"
"account_template_203010210","2.03.01.02.10","(-) Capital to Integralize of Households and Residents abroad","equity","","(-) Capital a Integralizar de Domiciliados e Residentes no Exterior"
"account_template_203011001","2.03.01.10.01","(-) Expenses with Issuance of Shares","equity","","(-) Gastos com Emissão de Ações"
"account_template_203020101","2.03.02.01.01","Premium on Issuance of Shares","equity","","Ágio na Emissão de Ações"
"account_template_203020102","2.03.02.01.02","Special Reserve for Goodwill on Merger","equity","","Reserva Especial de Ágio na Incorporação"
"account_template_203020103","2.03.02.01.03","Sale of Participating Interest and Subscription Bonus","equity","","Alienação de Partes Beneficiárias e Bônus de Subscrição"
"account_template_203020111","2.03.02.01.11","Donations and Investment Grants (Reserve constituted until 31/12/2007)","equity","","Doações e Subvenções para Investimentos (Reserva constituída até 31/12/2007)"
"account_template_203020112","2.03.02.01.12","Premium Received on the Issue of Debentures (Reserve established until 31/12/2007)","equity","","Prêmio Recebido na Emissão de Debêntures (Reserva constituída até 31/12/2007)"
"account_template_203020199","2.03.02.01.99","Other Capital Reserves","equity","","Outras Reservas de Capital"
"account_template_203020201","2.03.02.02.01","Revaluation Reserve","equity","","Reserva de Reavaliação"
"account_template_203020202","2.03.02.02.02","Revaluation Reserve Reflected","equity","","Reserva de Reavaliação Reflexa"
"account_template_203020301","2.03.02.03.01","Legal Reserve","equity","","Reserva Legal"
"account_template_203020302","2.03.02.03.02","State Reserve","equity","","Reserva Estatutária"
"account_template_203020303","2.03.02.03.03","Reserves for Contingency","equity","","Reserva para Contingência"
"account_template_203020304","2.03.02.03.04","Tax Incentive Reserve","equity","","Reserva de Incentivos Fiscais"
"account_template_203020305","2.03.02.03.05","Expansion Profit Reserve","equity","","Reserva de Lucros para Expansão"
"account_template_203020306","2.03.02.03.06","Unrealised Profit Reserve","equity","","Reserva de Lucros a Realizar"
"account_template_203020307","2.03.02.03.07","Special Reserve for Undistributed Mandatory Dividend","equity","","Reserva Especial para Dividendo Obrigatório não Distribuído"
"account_template_203020308","2.03.02.03.08","Premium Reserve in the Issue of Debentures","equity","","Reserva de Prêmio na Emissão de Debêntures"
"account_template_203020309","2.03.02.03.09","Reserve for Capital Increase (Law No. 9,249/1995, Art. 9, § 9)","equity","","Reserva para Aumento de Capital (Lei nº 9.249/1995, art. 9º, § 9º)"
"account_template_203020399","2.03.02.03.99","Other Revenue Reserves","equity","","Outras Reservas de Lucros"
"account_template_203030101","2.03.03.01.01","Offsetting entry of Financial Assets Available for Sale","equity","","Contrapartidas Ajustes Ativos Financeiros Disponíveis para Venda"
"account_template_203030102","2.03.03.01.02","Offsetting entries to hedging operations","equity","","Contrapartidas Ajustes em Operações de Hedge"
"account_template_203030103","2.03.03.01.03","Offsetting entry of the Positive Difference of Real Estate Transferred to Properties for Investment","equity","","Contrapartida da Diferença Positiva de Ativo Imobilizado Transferido para Propriedades para Investimento"
"account_template_203030104","2.03.03.01.04","Benefit Plans Adjustment Breakdown to Employees","equity","","Contrapartida de Ajustes dos Planos de Benefícios a Empregados"
"account_template_203030105","2.03.03.01.05","Offsetting entry of Fixed Assets and Investment Property Adjustments - CPC Initial Adoption","equity","","Contrapartida de Ajustes do Ativo Imobilizado e de Propriedades para Investimento - Adoção Inicial CPC"
"account_template_203030106","2.03.03.01.06","Accumulated Conversion Adjustments","equity","","Ajustes Acumulados de Conversão"
"account_template_203030130","2.03.03.01.30","(-) Negative Equity Valuation Adjustments","equity","","(-) Ajustes de Avaliação Patrimonial Negativos"
"account_template_203030190","2.03.03.01.90","Other Comprehensive Income - Equity Valuation Adjustments","equity","","Outros Resultados Abrangentes - Ajustes de Avaliação Patrimonial"
"account_template_203030201","2.03.03.02.01","Offsetting entry of Financial Assets Available for Sale - Reflected","equity","","Contrapartidas Ajustes Ativos Financeiros Disponíveis para Venda - Reflexa"
"account_template_203030202","2.03.03.02.02","Offsetting entries to hedging operations - Reflected","equity","","Contrapartidas Ajustes em Operações de Hedge - Reflexa"
"account_template_203030203","2.03.03.02.03","Offsetting entry Positive Difference of Fixed Assets Transferred to Investment Properties - Reflected","equity","","Contrapartida Diferença Positiva de Ativo Imobilizado Transferido para Propriedades para Investimento - Reflexa"
"account_template_203030204","2.03.03.02.04","Offsetting Entry Employee Benefit Plans - Reflected","equity","","Contrapartida Ajustes Planos de Benefícios a Empregados - Reflexa"
"account_template_203030205","2.03.03.02.05","Offsetting Entry Fixed assets and Investment Properties - Initial Adoption CPC - Reflected","equity","","Contrapartida Ajustes Imobilizado e Propriedades para Investimento - Adoção Inicial CPC - Reflexa"
"account_template_203030206","2.03.03.02.06","Accumulated Conversion Adjustments - Reflected","equity","","Ajustes Acumulados de Conversão - Reflexa"
"account_template_203030230","2.03.03.02.30","(-) Negative Equity Valuation Adjustments - Reflected","equity","","(-) Ajustes de Avaliação Patrimonial Negativos - Reflexa"
"account_template_203030290","2.03.03.02.90","Other Comprehensive Income - Equity Valuation Adjustments - Reflected","equity","","Outros Resultados Abrangentes - Ajustes de Avaliação Patrimonial - Reflexa"
"account_template_203040101","2.03.04.01.01","Accrued profits and/or Balance to the Provision of the Assembly","equity","","Lucros Acumulados e/ou Saldo à Disposição da Assembleia"
"account_template_203040105","2.03.04.01.05","Contracting Contingent - Business Combination - Shareholders' Equity","equity","","Contraprestação Contingente - Combinação de Negócios - Patrimônio Líquido"
"account_template_203040110","2.03.04.01.10","Prior Years Adjustments","equity","","Ajustes de Exercícios Anteriores"
"account_template_203040111","2.03.04.01.11","(-) Accumulated Losses","equity","","(-) Prejuízos Acumulados"
"account_template_203040112","2.03.04.01.12","(-) Shares in Treasury","equity","","(-) Ações em Tesouraria"
"account_template_203040115","2.03.04.01.15","(-) Capital Transactions","equity","","(-) Transações de Capital"
"account_template_203040190","2.03.04.01.90","Unclassified Shareholders' Equity Accounts","equity","","Contas do Patrimônio Líquido Não Classificadas"
"account_template_30101010101","3.01.01.01.01.01","Direct Export Revenue of Goods and Products","income","","Receita de Exportação Direta de Mercadorias e Produtos"
"account_template_30101010102","3.01.01.01.01.02","Income from Sales of Merchandise and Products to Exporting Commercial Companies for Specific Export Purposes","income","","Receita de Vendas de Mercadorias e Produtos a Comercial Exportadora com Fim Específico de Exportação"
"account_template_30101010103","3.01.01.01.01.03","Services Export Revenue","income","","Receita de Exportação de Serviços"
"account_template_30101010104","3.01.01.01.01.04","Revenues from sales of own-manufactured products on the domestic market","income","","Receita da Venda de Produtos de Fabricação Própria no Mercado Interno"
"account_template_30101010105","3.01.01.01.01.05","Revenue from the Resale of Goods in the Internal Market","income","","Receita da Revenda de Mercadorias no Mercado Interno"
"account_template_30101010106","3.01.01.01.01.06","Revenue from Services Rendered in the Internal Market","income","","Receita da Prestação de Serviços no Mercado Interno"
"account_template_30101010107","3.01.01.01.01.07","Income from the Sale of Real Estate Units","income","","Receita da Venda de Unidades Imobiliárias"
"account_template_30101010108","3.01.01.01.01.08","Revenue from the Rental of Movable and Immovable Property","income","","Receita da Locação de Bens Móveis e Imóveis"
"account_template_30101010120","3.01.01.01.01.20","Construction Contract Revenue","income","","Receita de Contrato de Construção"
"account_template_30101010125","3.01.01.01.01.25","Revenue from Public Service Exploitation Right","income","","Receita de Direito de Exploração Serviço Público"
"account_template_30101010130","3.01.01.01.01.30","Income from Securitization of Credits","income","","Receita de Securitização de Créditos"
"account_template_30101010198","3.01.01.01.01.98","Other General Activity Income","income","","Outras Receitas da Atividade Geral"
"account_template_30101010201","3.01.01.01.02.01","(-) Canceled Sales and Sales Returns","expense","","(-) Vendas Canceladas e Devoluções de Vendas"
"account_template_30101010202","3.01.01.01.02.02","(-) Unconditional Discounts and Rebates","expense","","(-) Descontos Incondicionais e Abatimentos"
"account_template_30101010203","3.01.01.01.02.03","(-) ICMS","expense","","(-) ICMS"
"account_template_30101010204","3.01.01.01.02.04","(-) COFINS On Gross Revenue","expense","","(-) COFINS Sobre Receita Bruta"
"account_template_30101010205","3.01.01.01.02.05","(-) PIS/PASEP On Gross Revenue","expense","","(-) PIS/PASEP Sobre Receita Bruta"
"account_template_30101010206","3.01.01.01.02.06","(-) ISS","expense","","(-) ISS"
"account_template_30101010209","3.01.01.01.02.09","(-) Other taxes and contributions on sales and services","expense","","(-) Demais Impostos e Contribuições Incidentes sobre Vendas e Serviços"
"account_template_30101010210","3.01.01.01.02.10","(-) Adjustment to Present Value on Gross Revenue","expense","","(-) Ajuste a Valor Presente sobre Receita Bruta"
"account_template_30101010260","3.01.01.01.02.60","(-) CPC 47 - Contractual Modifications","expense","","(-) CPC 47 - Modificações Contratuais"
"account_template_30101010262","3.01.01.01.02.62","(-) CPC 47 - Recognition of Contract Liabilities - Guarantees","expense","","(-) CPC 47 - Reconhecimento de Passivos de Contrato - Garantias"
"account_template_30101010264","3.01.01.01.02.64","(-) CPC 47 - Recognition of Contract Liabilities - Unexercised Rights","expense","","(-) CPC 47 - Reconhecimento de Passivos de Contrato - Direitos não Exercidos"
"account_template_30101010266","3.01.01.01.02.66","(-) CPC 47 - Recognition of Contract Liabilities - Custody Services - Sales for Future Delivery","expense","","(-) CPC 47 - Reconhecimento de Passivos de Contrato - Serviços de Custódia - Vendas para Entrega Futura"
"account_template_30101010268","3.01.01.01.02.68","(-) CPC 47 - Transaction Price - Variable Considerations","expense","","(-) CPC 47 - Preço de Transação - Contraprestações Variáveis"
"account_template_30101010270","3.01.01.01.02.70","(-) CPC 47 - Transaction Price - Variable Consideration Revaluations","expense","","(-) CPC 47 - Preço de Transação - Reavaliações de Contraprestação Variável"
"account_template_30101010272","3.01.01.01.02.72","(-) CPC 47 - Transaction Price - Consideration Paid or Payable","expense","","(-) CPC 47 - Preço de Transação - Contraprestações Pagas ou a Pagar"
"account_template_30101010274","3.01.01.01.02.74","(-) CPC 47 - Transaction Price - Performance Bonds","expense","","(-) CPC 47 - Preço de Transação - Obrigações de Desempenho"
"account_template_30101010276","3.01.01.01.02.76","(-) CPC 47 - Criteria Divergent from Tax Legislation - Non-Receiptof Consideration","expense","","(-) CPC 47 - Critérios Divergentes da Legislação Tributária - Não Recebimento de Contraprestação"
"account_template_30101010278","3.01.01.01.02.78","(-) CPC 47 - Criteria Diverging from Tax Legislation - Contract Liabilities - Right of Return","expense","","(-) CPC 47 - Critérios Divergentes da Legislação Tributária - Passivos de Contrato - Direito à Devolução"
"account_template_30101010280","3.01.01.01.02.80","(-) CPC 47 - Criteria Divergent from Tax Legislation - Contract Liabilities - Optional Right of Acquisition","expense","","(-) CPC 47 - Critérios Divergentes da Legislação Tributária - Passivos de Contrato - Direito de Aquisição Opcional"
"account_template_30101030101","3.01.01.03.01.01","(-) Cost of Own Products Sold","expense_direct_cost","","(-) Custo dos Produtos de Fabricação Própria Vendidos"
"account_template_30101030102","3.01.01.03.01.02","(-) Cost of Goods","expense_direct_cost","","(-) Custo das Mercadorias Revendidas"
"account_template_30101030103","3.01.01.03.01.03","(-) Cost of Services","expense_direct_cost","","(-) Custo dos Serviços Prestados"
"account_template_30101030104","3.01.01.03.01.04","(-) Cost of Real Estate Units Sold","expense_direct_cost","","(-) Custo das Unidades Imobiliárias Vendidas"
"account_template_30101030110","3.01.01.03.01.10","(-) Cost of Goods","expense_direct_cost","","(-) Custo das Mercadorias Revendidas"
"account_template_30101030120","3.01.01.03.01.20","(-) Construction Cost","expense_direct_cost","","(-) Custo de Construção"
"account_template_30101030130","3.01.01.03.01.30","(-) Cost of Securitization Operation","expense_direct_cost","","(-) Custo de Operação de Securitização"
"account_template_30101050101","3.01.01.05.01.01","Exchange Variations Assets","income_other","","Variações Cambiais Ativas"
"account_template_30101050102","3.01.01.05.01.02","Earnings Accrued in the Variable Income Market, except Day-Trade","income_other","","Ganhos Auferidos no Mercado de Renda Variável, exceto Day-Trade"
"account_template_30101050103","3.01.01.05.01.03","Gains in Day-Trade Operations","income_other","","Ganhos em Operações Day-Trade"
"account_template_30101050104","3.01.01.05.01.04","Interest Income on Equity","income_other","","Receitas de Juros sobre o Capital Próprio"
"account_template_30101050105","3.01.01.05.01.05","Other Financial Income","income_other","","Outras Receitas Financeiras"
"account_template_30101050106","3.01.01.05.01.06","Positive Results of Equity Investments Measured by the Equity Method","income_other","","Resultados Positivos em Participações Societárias Avaliadas pelo Método de Equivalência Patrimonial"
"account_template_30101050107","3.01.01.05.01.07","Positive Results in SCPs Evaluated by the Equity Method","income_other","","Resultados Positivos em SCP Avaliadas pelo Método de Equivalência Patrimonial"
"account_template_30101050108","3.01.01.05.01.08","Income and Capital Gains from Overseas Earnings","income_other","","Rendimentos e Ganhos de Capital Auferidos no Exterior"
"account_template_30101050109","3.01.01.05.01.09","Reversal of Estimated Losses Deriving from Recuperability Test (Impairment)","income_other","","Reversão das Perdas Estimadas Decorrentes de Teste de Recuperabilidade (Impairment)"
"account_template_30101050110","3.01.01.05.01.10","Reversal of the Balances of Provisions","income_other","","Reversão dos Saldos das Provisões"
"account_template_30101050111","3.01.01.05.01.11","Premiums Received in the Emission of Debentures","income_other","","Prêmios Recebidos na Emissão de Debêntures"
"account_template_30101050112","3.01.01.05.01.12","Donations and Subventions for Costing or Operations","income_other","","Doações e Subvenções para Custeio ou Operações"
"account_template_30101050113","3.01.01.05.01.13","Donations and Investment Grants","income_other","","Doações e Subvenções para Investimentos"
"account_template_30101050114","3.01.01.05.01.14","Income from Reclassification of Equity Valuation Adjustments","income_other","","Receitas de Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_30101050115","3.01.01.05.01.15","Income from Reclassification of Equity Valuation Adjustments - Reflected","income_other","","Receitas de Reclassificação de Ajustes de Avaliação Patrimonial - Reflexo"
"account_template_30101050116","3.01.01.05.01.16","Financial Income Resulting from Present Value Adjustments","income_other","","Receitas Financeiras Decorrentes de Ajustes ao Valor Presente"
"account_template_30101050117","3.01.01.05.01.17","Gain By Buying Advantageous Investments","income_other","","Ganho Por Compra Vantajosa em Investimentos"
"account_template_30101050118","3.01.01.05.01.18","Amortisation of capital losses","income_other","","Amortização de Menos-Valia"
"account_template_30101050119","3.01.01.05.01.19","Real Estate Rental Income - Non Core Activity","income_other","","Receita de Aluguel de Bens Imóveis - Atividade Não Principal"
"account_template_30101050120","3.01.01.05.01.20","Revenue from Rental of Movable Property - Non Core Activity","income_other","","Receita de Aluguel de Bens Móveis - Atividade Não Principal"
"account_template_30101050121","3.01.01.05.01.21","IPI Presumed Credits","income_other","","Créditos Presumidos de IPI"
"account_template_30101050122","3.01.01.05.01.22","PIS/COFINS Presumed Credits","income_other","","Créditos Presumidos de PIS/COFINS"
"account_template_30101050123","3.01.01.05.01.23","Other Presumed Tax Credits","income_other","","Outros Créditos Fiscais Presumidos"
"account_template_30101050124","3.01.01.05.01.24","Fines and Other Benefits Received","income_other","","Multas e Outras Vantagens Recebidas"
"account_template_30101050125","3.01.01.05.01.25","Profits and Dividends Derived from Equity Holdings Valued at Acquisition Cost","income_other","","Lucros e Dividendos Derivados de Participações Societárias Avaliadas pelo Custos de Aquisição"
"account_template_30101050126","3.01.01.05.01.26","Securities Lending Income","income_other","","Receitas com Empréstimos de Valores Mobiliários"
"account_template_30101050127","3.01.01.05.01.27","Income from Loan Transactions - Related Parties","income_other","","Rendimentos Auferidos em Operações de Mútuo – Partes Relacionadas"
"account_template_30101050128","3.01.01.05.01.28","Income from Loan Transactions - Non-Related Parties","income_other","","Rendimentos Auferidos em Operações de Mútuo – Partes Não Relacionadas"
"account_template_30101050129","3.01.01.05.01.29","Income from Debentures - Issuer Related Parties","income_other","","Rendimentos Auferidos com Debêntures - Emitente Partes  Relacionadas"
"account_template_30101050130","3.01.01.05.01.30","Income from Debentures - Issuer Non-Related Parties","income_other","","Rendimentos Auferidos com Debêntures - Emitente Partes Não Relacionadas"
"account_template_30101050131","3.01.01.05.01.31","Income from Government Securities","income_other","","Rendimentos Auferidos com Títulos Públicos"
"account_template_30101050132","3.01.01.05.01.32","Interest with Other Financial Assets Measured by Amortized Cost","income_other","","Juros Auferidos com Outros Ativos Financeiros Mensurados Pelo Custo Amortizado"
"account_template_30101050133","3.01.01.05.01.33","Gain from Adjustments to Fair Value - Financial Instruments for Trading - Non-Hedge - Fair Value by Result (VJPR).","income_other","","Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  para Negociação - Não Hedge – Valor Justo pelo Resultado (VJPR)."
"account_template_30101050134","3.01.01.05.01.34","Gain on Fair Value Adjustments - Available-for-sale Financial Instruments - Reclassification of Valuation Adjustments","income_other","","Ganho de Ajustes a Valor Justo  - Instrumentos Financeiros  Disponíveis para Venda - Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_30101050135","3.01.01.05.01.35","Gain from Fair Value Adjustments - Fair Value Hedge Financial Instruments","income_other","","Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  de Hedge  de Valor Justo"
"account_template_30101050136","3.01.01.05.01.36","Fair Value Adjustment Gain - Hedge Financial Instruments -Reclassification of Equity Valuation Adjustments","income_other","","Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  de Hedge -  Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_30101050137","3.01.01.05.01.37","Gain from Fair Value Adjustments - Fair Value Hedge Item","income_other","","Ganho de Ajustes a Valor Justo - Item Objeto de Hedge de Valor Justo"
"account_template_30101050138","3.01.01.05.01.38","Fair Value Adjustment Gain - Property for Investment","income_other","","Ganho de Ajustes a Valor Justo - Propriedade para Investimento"
"account_template_30101050139","3.01.01.05.01.39","Fair Value Adjustment Gain - Consumable Biological Asset","income_other","","Ganho de Ajustes a Valor Justo - Ativo Biológico Consumível"
"account_template_30101050140","3.01.01.05.01.40","Gain from Adjustments to Fair Value - Biological Production Asset","income_other","","Ganho de Ajustes a Valor Justo - Ativo Biológico de Produção"
"account_template_30101050141","3.01.01.05.01.41","Fair Value Adjustment Gain - Non Current Assets Held for Sale","income_other","","Ganho de Ajustes a Valor Justo - Ativos Não Circulantes Mantidos para Venda"
"account_template_30101050142","3.01.01.05.01.42","Fair Value Adjustment Gain - Capital Subscription with other Goods","income_other","","Ganho de Ajustes a Valor Justo - Subscrição de Capital com demais Bens"
"account_template_30101050143","3.01.01.05.01.43","Gain from Adjustments to Fair Value - Capital Subscription with Equity Interest","income_other","","Ganho de Ajustes a Valor Justo - Subscrição de Capital com Participação Societária"
"account_template_30101050144","3.01.01.05.01.44","Gain on Fair Value Adjustments - Acquisition of Equity Interest in stages","income_other","","Ganho de Ajustes a Valor Justo - Aquisição de Participação Societária em Estágios"
"account_template_30101050145","3.01.01.05.01.45","Fair Value Adjustment Gain - Due to Asset or Liabilities Exchange","income_other","","Ganho de Ajustes a Valor Justo - Decorrente de Permuta de Ativos ou Passivos"
"account_template_30101050146","3.01.01.05.01.46","Fair Value Adjustment Gain - Other Operations","income_other","","Ganho de Ajustes a Valor Justo - Outras Operações"
"account_template_30101050148","3.01.01.05.01.48","Cash Discount Gain","income_other","","Cash Discount Gain"
"account_template_30101050199","3.01.01.05.01.99","Other Operating Income","income_other","","Outras Receitas Operacionais"
"account_template_30101070101","3.01.01.07.01.01","(-) Compensation to Managers and Board of Directors","expense","","(-) Remuneração a Dirigentes e a Conselho de Administração"
"account_template_30101070102","3.01.01.07.01.02","(-) Ordained, Salaries, Gratifications and Other Remunerations to Employees","expense","","(-) Ordenados, Salários, Gratificações e Outras Remunerações a Empregados"
"account_template_30101070103","3.01.01.07.01.03","(-) Other Personnel Expenses","expense","","(-) Outros Gastos com Pessoal"
"account_template_30101070104","3.01.01.07.01.04","(-) Other Services Provided by Individuals or Legal Entities","expense","","(-) Outros Serviços Prestados por Pessoa Física ou Jurídica"
"account_template_30101070105","3.01.01.07.01.05","(-) Social Charges - Social Security","expense","","(-) Encargos Sociais - Previdência Social"
"account_template_30101070106","3.01.01.07.01.06","(-) Social Charges - FGTS","expense","","(-) Encargos Sociais - FGTS"
"account_template_30101070107","3.01.01.07.01.07","(-) Social Charges – Other","expense","","(-) Encargos Sociais – Outros"
"account_template_30101070108","3.01.01.07.01.08","(-) Donations and Sponsors of Cultural and Artistic Character (Law No. 8.313/1991)","expense","","(-) Doações e Patrocínios de Caráter Cultural e Artístico (Lei no 8.313/1991)"
"account_template_30101070109","3.01.01.07.01.09","(-) Operations of Acquisition of Culture Voucher (Law no 12,761/2012, art. 10).","expense","","(-) Operações de Aquisição de Vale Cultura (Lei no 12.761/2012, art. 10)."
"account_template_30101070110","3.01.01.07.01.10","(-) Donations to Teaching and Research Institutions (Law No. 9,249/1995, Art.13, § 2o)","expense","","(-) Doações a Instituições de Ensino e Pesquisa (Lei nº 9.249/1995, art.13, § 2º)"
"account_template_30101070111","3.01.01.07.01.11","(-) Donations to Civil Entities","expense","","(-) Doações a Entidades Civis"
"account_template_30101070112","3.01.01.07.01.12","(-) Other Contributions, Donations and Sponsors","expense","","(-) Outras Contribuições, Doações e Patrocínios"
"account_template_30101070113","3.01.01.07.01.13","(-) Worker's Food","expense","","(-) Alimentação do Trabalhador"
"account_template_30101070114","3.01.01.07.01.14","(-) PIS/PASEP","expense","","(-) PIS/PASEP"
"account_template_30101070115","3.01.01.07.01.15","(-) COFINS","expense","","(-) COFINS"
"account_template_30101070116","3.01.01.07.01.16","(-) Further Taxes, Fees and Contributions, except IR and CSLL","expense","","(-) Demais Impostos, Taxas e Contribuições, exceto IR e CSLL"
"account_template_30101070117","3.01.01.07.01.17","(-) Lease","expense","","(-) Arrendamento Mercantil"
"account_template_30101070118","3.01.01.07.01.18","(-) Rentals","expense","","(-) Aluguéis"
"account_template_30101070119","3.01.01.07.01.19","(-) Expenditure with Vehicles and Goods and Facilities Conservation","expense","","(-) Despesas com Veículos e de Conservação de Bens e Instalações"
"account_template_30101070120","3.01.01.07.01.20","(-) Advertising, Publicity and Sponsorship","expense","","(-) Propaganda, Publicidade e Patrocínio"
"account_template_30101070121","3.01.01.07.01.21","(-) Advertising, Publicity and Sponsorship of Assoc. Sports Maintaining Professional Football Team","expense","","(-) Propaganda, Publicidade e Patrocínio de Assoc. Desportivas que Mantenha Equipe de Futebol Profissional"
"account_template_30101070122","3.01.01.07.01.22","(-) Fines","expense","","(-) Multas"
"account_template_30101070123","3.01.01.07.01.23","(-) Depreciation Charges","expense_depreciation","","(-) Encargos de Depreciação"
"account_template_30101070124","3.01.01.07.01.24","(-) Amortization Charges","expense_depreciation","","(-) Encargos de Amortização"
"account_template_30101070125","3.01.01.07.01.25","(-) Losses in Credit Operations","expense","","(-) Perdas em Operações de Crédito"
"account_template_30101070126","3.01.01.07.01.26","(-) Vacation Provisions","expense","","(-) Provisões para Férias"
"account_template_30101070127","3.01.01.07.01.27","(-) Provisions for 13th Employee Salary","expense","","(-) Provisões para 13º Salário de Empregados"
"account_template_30101070128","3.01.01.07.01.28","(-) Provision for Book Stock Loss","expense","","(-) Provisão para Perda de Estoque de Livros"
"account_template_30101070129","3.01.01.07.01.29","(-) Further Provisions","expense","","(-) Demais Provisões"
"account_template_30101070130","3.01.01.07.01.30","(-) Gratifications to Administrators","expense","","(-) Gratificações a Administradores"
"account_template_30101070131","3.01.01.07.01.31","(-) Royalties and Technical Assistance - In the COUNTRY","expense","","(-) Royalties e Assistência Técnica - no PAÍS"
"account_template_30101070132","3.01.01.07.01.32","(-) Royalties and Technical Assistance - ABROAD","expense","","(-) Royalties e Assistência Técnica - no EXTERIOR"
"account_template_30101070133","3.01.01.07.01.33","(-) Medical, Dental and Pharmaceutical Assistance to Employees","expense","","(-) Assistência Médica, Odontológica e Farmacêutica a Empregados"
"account_template_30101070134","3.01.01.07.01.34","(-) Scientific and Technological Research","expense","","(-) Pesquisas Científicas e Tecnológicas"
"account_template_30101070135","3.01.01.07.01.35","(-) Goods of Small Unit Value or Useful Life of up to one Year Deducted as Expense","expense","","(-) Bens de Pequeno Valor Unitário ou de Vida Útil de até um Ano Deduzidos como Despesa"
"account_template_30101070136","3.01.01.07.01.36","(-) Electric Energy Expenditure","expense","","(-) Despesas com Energia Elétrica"
"account_template_30101070137","3.01.01.07.01.37","(-) Water and Sewage Expenditure","expense","","(-) Despesas com Água e Esgoto"
"account_template_30101070138","3.01.01.07.01.38","(-) Telephone and Internet expenditure","expense","","(-) Despesas com Telefone e Internet"
"account_template_30101070139","3.01.01.07.01.39","(-) Postal and Courier Expenses","expense","","(-) Despesas com Correios e Malotes"
"account_template_30101070140","3.01.01.07.01.40","(-) Insurance Expenditure","expense","","(-) Despesas com Seguros"
"account_template_30101070141","3.01.01.07.01.41","(-) Social Security Benefits to Employees","expense","","(-) Benefícios Previdenciários a Empregados"
"account_template_30101070142","3.01.01.07.01.42","(-) Individual Advisor Fund - FAPI","expense","","(-) Fundo de Aposentadora Individual - FAPI"
"account_template_30101070143","3.01.01.07.01.43","(-) Saving and Investment Plans - PAIT","expense","","(-) Planos de Poupança e Investimento - PAIT"
"account_template_30101070144","3.01.01.07.01.44","(-) Comprehensive Research and Development in the Rota 2030 Program","expense","","(-) Pesquisa e Desenvolvimento Abrangidas no Programa Rota 2030"
"account_template_30101090101","3.01.01.09.01.01","(-) Passive Exchange Variations","expense","","(-) Variações Cambiais Passivas"
"account_template_30101090102","3.01.01.09.01.02","(-) Incurred Losses in Variable Income Market except Day-Trade","expense","","(-) Perdas Incorridas no Mercado de Renda Variável, exceto Day-Trade"
"account_template_30101090103","3.01.01.09.01.03","(-) Day-Trade Operations Losses","expense","","(-) Perdas em Operações Day-Trade"
"account_template_30101090104","3.01.01.09.01.04","(-) Interest expenditure on equity","expense","","(-) Despesas de Juros sobre o Capital Próprio"
"account_template_30101090105","3.01.01.09.01.05","(-) Debenture Remuneration Expenses","expense","","(-) Despesas de Remuneração de Debêntures"
"account_template_30101090106","3.01.01.09.01.06","(-) Interest on Loans from Related Persons or Located in a Country with Favoured Taxation","expense","","(-) Juros com Empréstimos de Pessoas Vinculadas ou Situadas em País com Tributação favorecida"
"account_template_30101090107","3.01.01.09.01.07","(-) Financial Expenditure for Rent","expense","","(-) Despesas Financeiras Relativas a Arrendamento"
"account_template_30101090108","3.01.01.09.01.08","(-) Other Financial Expenses","expense","","(-) Outras Despesas Financeiras"
"account_template_30101090109","3.01.01.09.01.09","(-) Negative Results in Corporate Participation Valued by the Equity Method","expense","","(-) Resultados Negativos em Participações Societárias Avaliadas pelo Método de Equivalência Patrimonial"
"account_template_30101090110","3.01.01.09.01.10","(-) Negative results in SCP Valued by the Equity Method","expense","","(-) Resultados Negativos em SCP Avaliadas pelo Método de Equivalência Patrimonial"
"account_template_30101090111","3.01.01.09.01.11","(-) Loss in Operations Directed abroad","expense","","(-) Perdas em Operações Realizadas no Exterior"
"account_template_30101090112","3.01.01.09.01.12","(-) Estimated Losses Deriving from Recoverability Test (Impairment)","expense","","(-) Perdas Estimadas Decorrentes de Teste de Recuperabilidade (Impairment)"
"account_template_30101090113","3.01.01.09.01.13","(-) Expenses from Reclassification of Equity Valuation Adjustments","expense","","(-) Despesas de Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_30101090114","3.01.01.09.01.14","(-) Expenses from Reclassification of Equity Valuation Adjustments - Reflected","expense","","(-) Despesas de Reclassificação de Ajustes de Avaliação Patrimonial -Reflexo"
"account_template_30101090115","3.01.01.09.01.15","(-) Financial Expenses Resulting from Present Value Adjustments","expense","","(-) Despesas Financeiras Decorrentes dos Ajustes ao Valor Presente"
"account_template_30101090116","3.01.01.09.01.16","(-) Benefit Depreciation Charges Rental Object","expense_depreciation","","(-) Encargos de Depreciação de Bens Objeto de Arrendamento"
"account_template_30101090117","3.01.01.09.01.17","(-) Capital Gain Amortization Charges","expense_depreciation","","(-) Encargos de Amortização de Mais - Valia"
"account_template_30101090118","3.01.01.09.01.18","(-) Real Estate Rentals- Renter Related Part","expense","","(-) Aluguéis de Bens Imóveis- Locador  Parte Relacionada"
"account_template_30101090119","3.01.01.09.01.19","(-) Leases of Real Estate Landlord Unrelated Party","expense","","(-) Aluguéis de Bens Imóveis Locador  Parte Não Relacionada"
"account_template_30101090120","3.01.01.09.01.20","(-) Expenditure with Securities Loans","expense","","(-) Despesas com Empréstimos de Valores Mobiliários"
"account_template_30101090121","3.01.01.09.01.21","(-) Brokerage and commission expenses","expense","","(-) Despesas com Corretagem e Emolumentos"
"account_template_30101090122","3.01.01.09.01.22","(-) Expenses with Negative Goodwill on Transfer of Securities","expense","","(-) Despesas  com Deságio na Cessão de Títulos"
"account_template_30101090123","3.01.01.09.01.23","(-) Expenditure in Mutual Operations – Related Party","expense","","(-) Despesas Incorridas em Operações de Mútuo – Parte Relacionada"
"account_template_30101090124","3.01.01.09.01.24","(-) Expenses Incurred in Loan Transactions - Unrelated Party","expense","","(-) Despesas Incorridas em Operações de Mútuo – Parte Não Relacionada"
"account_template_30101090125","3.01.01.09.01.25","(-) Expenditure incurred in Other Financial Liabilities Measured by Amortized Cost","expense","","(-) Despesas Incorridas  em Outros Passivos Financeiros Mensurados Pelo Custo Amortizado"
"account_template_30101090126","3.01.01.09.01.26","(-) Fair Value Adjustment Loss - Financial Instruments for Trading - No Hedge - Fair Value for Result","expense","","(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros para Negociação - Não Hedge - Valor Justo pelo Resultado"
"account_template_30101090127","3.01.01.09.01.27","(-) Fair Value Adjustment Loss - Available Financial Instruments for Sale - Reclassification of Equity Valuation Adjustments","expense","","(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros Disponíveis para Venda - Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_30101090128","3.01.01.09.01.28","(-) Fair Value Adjustment Loss - Fair Value Hedge Financial Instruments","expense","","(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros de Hedge de Valor Justo"
"account_template_30101090129","3.01.01.09.01.29","(-) Fair Value Adjustment Loss - Hedge Financial Instruments - Reclassification of Equity Valuation Adjustments","expense","","(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros de Hedge -  Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_30101090130","3.01.01.09.01.30","(-) Loss of Adjustment to Fair Value - Fair Value Hedge Item","expense","","(-) Perda de Ajuste a Valor Justo - Item Objeto de Hedge de Valor Justo"
"account_template_30101090131","3.01.01.09.01.31","(-) Loss of Adjustment to Fair Value - Property for Investment","expense","","(-) Perda de Ajuste a Valor Justo - Propriedade para Investimento"
"account_template_30101090132","3.01.01.09.01.32","(-) Fair Value Adjustment Loss - Consumable Biological Asset","expense","","(-) Perda de Ajuste a Valor Justo - Ativo Biológico Consumível"
"account_template_30101090133","3.01.01.09.01.33","(-) Fair Value Adjustment Loss - Biological Production Act","expense","","(-) Perda de Ajuste a Valor Justo - Ativo Biológico de Produção"
"account_template_30101090134","3.01.01.09.01.34","(-) Fair Value Adjustment Loss - Non-Current Assets Held for Sale","expense","","(-) Perda de Ajuste a Valor Justo - Ativos Não Circulantes Mantidos para Venda"
"account_template_30101090135","3.01.01.09.01.35","(-) Loss of Adjustment to Fair Value - Subscription of Capital with other Goods","expense","","(-) Perda de Ajuste a Valor Justo - Subscrição de Capital com demais Bens"
"account_template_30101090136","3.01.01.09.01.36","(-) Loss of Adjustment to Fair Value - Equity Subscription with Corporate Participation","expense","","(-) Perda de Ajuste a Valor Justo - Subscrição de Capital com Participação Societária"
"account_template_30101090137","3.01.01.09.01.37","(-) Fair Value Adjustment Loss - Acquisition of Equity Interest in Stages","expense","","(-) Perda de Ajuste a Valor Justo - Aquisição de Participação Societária em Estágios"
"account_template_30101090138","3.01.01.09.01.38","(-) Fair Value Adjustment Loss - Due to Exchange of Assets or Liabilities","expense","","(-) Perda de Ajuste a Valor Justo - Decorrente de Permuta de Ativos ou Passivos"
"account_template_30101090139","3.01.01.09.01.39","(-) Loss of Adjustment to Fair Value - Other Operations","expense","","(-) Perda de Ajuste a Valor Justo - Outras Operações"
"account_template_30101090199","3.01.01.09.01.99","(-) Other Operational Expenses","expense","","(-) Outras Despesas Operacionais"
"account_template_30101110101","3.01.01.11.01.01","Income from the sale of participations that are part of Current Assets or Long-Term Assets","income_other","","Receitas na Alienação de Participações Integrantes do Ativo Circulante ou do Ativo Realizável a Longo Prazo"
"account_template_30101110102","3.01.01.11.01.02","Income from the Sale of Goods and Rights from Non-Current Assets, Fixed Assets and Intangible Assets","income_other","","Receitas de Alienações de Bens e Direitos do Ativo Não Circulante Investimentos, Imobilizado e Intangível"
"account_template_30101110103","3.01.01.11.01.03","Capital Gains by Percentage Change in Equity Interest Evaluated by Shareholders' Equity","income_other","","Ganhos de Capital por Variação Percentual em Participação Societária Avaliada pelo Patrimônio Líquido"
"account_template_30101110104","3.01.01.11.01.04","(-) Book Value of Shares Comprising Current Assets or Long-Term Assets Sold","expense","","(-) Valor Contábil de Participações Integrantes do Ativo Circulante ou do Ativo Realizável a Longo Prazo Alienadas"
"account_template_30101110105","3.01.01.11.01.05","(-) Book Value of Assets and Rights of Non-Current Assets Investments, Intangible Assets and Fixed Assets Sold","expense","","(-) Valor Contábil dos Bens e Direitos do Ativo Não Circulante Investimentos, Intangível e Imobilizado Alienados"
"account_template_30101110106","3.01.01.11.01.06","(-) Percentage of Capital by Percent Variation in Societal Participation Measured by Net Equity","expense","","(-) Perdas de Capital por Variação Percentual em Participação Societária Avaliada pelo Patrimônio Líquido"
"account_template_30101110107","3.01.01.11.01.07","Income from Discontinued Operations","income_other","","Receitas de Operações Descontinuadas"
"account_template_30101110108","3.01.01.11.01.08","(-) Operations Expenditure Discontinued","expense","","(-) Despesas de Operações Descontinuadas"
"account_template_30105010101","3.01.05.01.01.01","(-) Employees' participation","expense","","(-) Participações de Empregados"
"account_template_30105010102","3.01.05.01.01.02","(-) Contributions for Employees' Assistance or Welfare","expense","","(-) Contribuições para Assistência ou Previdência de Empregados"
"account_template_30105010198","3.01.05.01.01.98","(-) Other Employee Participations","expense","","(-) Outras Participações de Empregados"
"account_template_30105010301","3.01.05.01.03.01","(-) Participation of Administrators and Beneficiary Parties","expense","","(-) Participações de Administradores e Partes Beneficiárias"
"account_template_30105010302","3.01.05.01.03.02","(-) Interests in Debentures","expense","","(-) Participações de Debêntures"
"account_template_30105010398","3.01.05.01.03.98","(-) Other Participations","expense","","(-) Outras Participações"
"account_template_30201010101","3.02.01.01.01.01","(-) Provision for Social Contribution on Net Profit (General Activity)","expense","","(-) Provisão para Contribuição Social sobre o Lucro Líquido (Atividade Geral)"
"account_template_30201010102","3.02.01.01.01.02","(-) Provision for Income Tax - Legal Person (General and Rural Activity)","expense","","(-) Provisão para Imposto de Renda - Pessoa Jurídica (Atividade Geral e Rural)"
"account_template_30201010111","3.02.01.01.01.11","(-) Provision for Social Contribution on Net Profit - Deferred Profits (General Activity)","expense","","(-) Provisão para Contribuição Social sobre o Lucro Líquido - Lucros Diferidos (Atividade Geral)"
"account_template_30201010112","3.02.01.01.01.12","(-) Provision for Income Tax - Legal Person - Deferred Profits (General and Rural Activity)","expense","","(-) Provisão para Imposto de Renda - Pessoa Jurídica - Lucros Diferidos (Atividade Geral e Rural)"
"account_template_31101010101","3.11.01.01.01.01","Rural Activity Income - Direct Export","income","","Receita da Atividade Rural - Exportação Direta"
"account_template_31101010102","3.11.01.01.01.02","Sale to Exporting Commercial Company with Specific Export Purpose","income","","Receita da Atividade Rural - Venda a Comercial Exportadora com Fim Específico de Exportação"
"account_template_31101010103","3.11.01.01.01.03","Rural Activity Income - Internal Market","income","","Receita da Atividade Rural - Mercado Interno"
"account_template_31101010201","3.11.01.01.02.01","(-) Canceled Sales and Sales Returns","expense","","(-) Vendas Canceladas e Devoluções de Vendas"
"account_template_31101010202","3.11.01.01.02.02","(-) Unconditional Discounts and Rebates","expense","","(-) Descontos Incondicionais e Abatimentos"
"account_template_31101010203","3.11.01.01.02.03","(-) ICMS","expense","","(-) ICMS"
"account_template_31101010204","3.11.01.01.02.04","(-) Cofins On Gross Revenue","expense","","(-) Cofins Sobre Receita Bruta"
"account_template_31101010205","3.11.01.01.02.05","(-) PIS/Pasep On Gross Revenue","expense","","(-) PIS/Pasep Sobre Receita Bruta"
"account_template_31101010206","3.11.01.01.02.06","(-) ISS","expense","","(-) ISS"
"account_template_31101010209","3.11.01.01.02.09","(-) Other taxes and contributions on sales and services","expense","","(-) Demais Impostos e Contribuições Incidentes sobre Vendas e Serviços"
"account_template_31101010210","3.11.01.01.02.10","(-) Adjustment to Present Value on Gross Revenue","expense","","(-) Ajuste a Valor Presente sobre Receita Bruta"
"account_template_31101030101","3.11.01.03.01.01","(-) Cost of Goods and Products Sold from Rural Activity","expense","","(-) Custo dos Bens e Produtos Vendidos da Atividade Rural"
"account_template_31101050101","3.11.01.05.01.01","Exchange Variations Assets","income_other","","Variações Cambiais Ativas"
"account_template_31101050102","3.11.01.05.01.02","Earnings Accrued in the Variable Income Market, except Day-Trade","income_other","","Ganhos Auferidos no Mercado de Renda Variável, exceto Day-Trade"
"account_template_31101050103","3.11.01.05.01.03","Gains in Day-Trade Operations","income_other","","Ganhos em Operações Day-Trade"
"account_template_31101050104","3.11.01.05.01.04","Interest Income on Equity","income_other","","Receitas de Juros sobre o Capital Próprio"
"account_template_31101050105","3.11.01.05.01.05","Other Financial Income","income_other","","Outras Receitas Financeiras"
"account_template_31101050106","3.11.01.05.01.06","Positive Results of Equity Investments Measured by the Equity Method","income_other","","Resultados Positivos em Participações Societárias Avaliadas pelo Método de Equivalência Patrimonial"
"account_template_31101050107","3.11.01.05.01.07","Positive Results in SCPs Evaluated by the Equity Method","income_other","","Resultados Positivos em SCP Avaliadas pelo Método de Equivalência Patrimonial"
"account_template_31101050108","3.11.01.05.01.08","Income and Capital Gains from Overseas Earnings","income_other","","Rendimentos e Ganhos de Capital Auferidos no Exterior"
"account_template_31101050109","3.11.01.05.01.09","Reversal of Estimated Losses Deriving from Recuperability Test (Impairment)","income_other","","Reversão das Perdas Estimadas Decorrentes de Teste de Recuperabilidade (Impairment)"
"account_template_31101050110","3.11.01.05.01.10","Reversal of the Balances of Provisions","income_other","","Reversão dos Saldos das Provisões"
"account_template_31101050111","3.11.01.05.01.11","Premiums Received in the Emission of Debentures","income_other","","Prêmios Recebidos na Emissão de Debêntures"
"account_template_31101050112","3.11.01.05.01.12","Donations and Subventions for Costing or Operations","income_other","","Doações e Subvenções para Custeio ou Operações"
"account_template_31101050113","3.11.01.05.01.13","Income from Reclassification of Equity Valuation Adjustments","income_other","","Receitas de Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_31101050114","3.11.01.05.01.14","Income from Reclassification of Equity Valuation Adjustments - Reflected","income_other","","Receitas de Reclassificação de Ajustes de Avaliação Patrimonial - Reflexo"
"account_template_31101050116","3.11.01.05.01.16","Financial Income Resulting from Present Value Adjustments","income_other","","Receitas Financeiras Decorrentes de Ajustes ao Valor Presente"
"account_template_31101050117","3.11.01.05.01.17","Gain By Buying Advantageous Investments","income_other","","Ganho Por Compra Vantajosa em Investimentos"
"account_template_31101050118","3.11.01.05.01.18","Amortisation of capital losses","income_other","","Amortização de Menos-Valia"
"account_template_31101050119","3.11.01.05.01.19","Real Estate Rental Income - Non Core Activity","income_other","","Receita de Aluguel de Bens Imóveis - Atividade Não Principal"
"account_template_31101050120","3.11.01.05.01.20","Revenue from Rental of Movable Property - Non Core Activity","income_other","","Receita de Aluguel de Bens Móveis - Atividade Não Principal"
"account_template_31101050121","3.11.01.05.01.21","IPI Presumed Credits","income_other","","Créditos Presumidos de IPI"
"account_template_31101050122","3.11.01.05.01.22","PIS/COFINS Presumed Credits","income_other","","Créditos Presumidos de PIS/COFINS"
"account_template_31101050123","3.11.01.05.01.23","Other Presumed Tax Credits","income_other","","Outros Créditos Fiscais Presumidos"
"account_template_31101050124","3.11.01.05.01.24","Fines and Other Benefits Received","income_other","","Multas e Outras Vantagens Recebidas"
"account_template_31101050125","3.11.01.05.01.25","Profits and Dividends Derived from Equity Holdings Valued at Acquisition Cost","income_other","","Lucros e Dividendos Derivados de Participações Societárias Avaliadas pelo Custos de Aquisição"
"account_template_31101050126","3.11.01.05.01.26","Securities Lending Income","income_other","","Receitas com Empréstimos de Valores Mobiliários"
"account_template_31101050127","3.11.01.05.01.27","Income from Loan Transactions - Related Parties","income_other","","Rendimentos Auferidos em Operações de Mútuo – Partes Relacionadas"
"account_template_31101050128","3.11.01.05.01.28","Income from Loan Transactions - Non-Related Parties","income_other","","Rendimentos Auferidos em Operações de Mútuo – Partes Não Relacionadas"
"account_template_31101050129","3.11.01.05.01.29","Income from Debentures - Issuer Related Parties","income_other","","Rendimentos Auferidos com Debêntures - Emitente Partes  Relacionadas"
"account_template_31101050130","3.11.01.05.01.30","Income from Debentures - Issuer Non-Related Parties","income_other","","Rendimentos Auferidos com Debêntures - Emitente Partes Não Relacionadas"
"account_template_31101050131","3.11.01.05.01.31","Income from Government Securities","income_other","","Rendimentos Auferidos com Títulos Públicos"
"account_template_31101050132","3.11.01.05.01.32","Interest with Other Financial Assets Measured by Amortized Cost","income_other","","Juros Auferidos com Outros Ativos Financeiros Mensurados Pelo Custo Amortizado"
"account_template_31101050133","3.11.01.05.01.33","Gain from Adjustments to Fair Value - Financial Instruments for Trading - Non-Hedge - Fair Value by Result (VJPR).","income_other","","Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  para Negociação - Não Hedge – Valor Justo pelo Resultado (VJPR)."
"account_template_31101050134","3.11.01.05.01.34","Gain on Fair Value Adjustments - Available-for-sale Financial Instruments - Reclassification of Valuation Adjustments","income_other","","Ganho de Ajustes a Valor Justo  - Instrumentos Financeiros  Disponíveis para Venda - Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_31101050135","3.11.01.05.01.35","Gain from Fair Value Adjustments - Fair Value Hedge Financial Instruments","income_other","","Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  de Hedge  de Valor Justo"
"account_template_31101050136","3.11.01.05.01.36","Fair Value Adjustment Gain - Hedge Financial Instruments -Reclassification of Equity Valuation Adjustments","income_other","","Ganho de Ajustes a Valor Justo - Instrumentos Financeiros  de Hedge -  Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_31101050137","3.11.01.05.01.37","Gain from Fair Value Adjustments - Fair Value Hedge Item","income_other","","Ganho de Ajustes a Valor Justo - Item Objeto de Hedge de Valor Justo"
"account_template_31101050138","3.11.01.05.01.38","Fair Value Adjustment Gain - Property for Investment","income_other","","Ganho de Ajustes a Valor Justo - Propriedade para Investimento"
"account_template_31101050139","3.11.01.05.01.39","Fair Value Adjustment Gain - Consumable Biological Asset","income_other","","Ganho de Ajustes a Valor Justo - Ativo Biológico Consumível"
"account_template_31101050140","3.11.01.05.01.40","Gain from Adjustments to Fair Value - Biological Production Asset","income_other","","Ganho de Ajustes a Valor Justo - Ativo Biológico de Produção"
"account_template_31101050141","3.11.01.05.01.41","Fair Value Adjustment Gain - Non Current Assets Held for Sale","income_other","","Ganho de Ajustes a Valor Justo - Ativos Não Circulantes Mantidos para Venda"
"account_template_31101050142","3.11.01.05.01.42","Fair Value Adjustment Gain - Capital Subscription with other Goods","income_other","","Ganho de Ajustes a Valor Justo - Subscrição de Capital com demais Bens"
"account_template_31101050143","3.11.01.05.01.43","Gain from Adjustments to Fair Value - Capital Subscription with Equity Interest","income_other","","Ganho de Ajustes a Valor Justo - Subscrição de Capital com Participação Societária"
"account_template_31101050144","3.11.01.05.01.44","Gain on Fair Value Adjustments - Acquisition of Equity Interest in stages","income_other","","Ganho de Ajustes a Valor Justo - Aquisição de Participação Societária em Estágios"
"account_template_31101050145","3.11.01.05.01.45","Fair Value Adjustment Gain - Due to Asset or Liabilities Exchange","income_other","","Ganho de Ajustes a Valor Justo - Decorrente de Permuta de Ativos ou Passivos"
"account_template_31101050146","3.11.01.05.01.46","Fair Value Adjustment Gain - Other Operations","income_other","","Ganho de Ajustes a Valor Justo - Outras Operações"
"account_template_31101050147","3.11.01.05.01.47","Donations and Investment Grants","income_other","","Doações e Subvenções para Investimentos"
"account_template_31101050199","3.11.01.05.01.99","Other Operating Income","income_other","","Outras Receitas Operacionais"
"account_template_31101070101","3.11.01.07.01.01","(-) Compensation to Managers and Board of Directors","expense","","(-) Remuneração a Dirigentes e a Conselho de Administração"
"account_template_31101070102","3.11.01.07.01.02","(-) Ordained, Salaries, Gratifications and Other Remunerations to Employees","expense","","(-) Ordenados, Salários, Gratificações e Outras Remunerações a Empregados"
"account_template_31101070103","3.11.01.07.01.03","(-) Other Personnel Expenses","expense","","(-) Outros Gastos com Pessoal"
"account_template_31101070104","3.11.01.07.01.04","(-) Other Services Provided by Individuals or Legal Entities","expense","","(-) Outros Serviços Prestados por Pessoa Física ou Jurídica"
"account_template_31101070105","3.11.01.07.01.05","(-) Social Charges - Social Security","expense","","(-) Encargos Sociais - Previdência Social"
"account_template_31101070106","3.11.01.07.01.06","(-) Social Charges - FGTS","expense","","(-) Encargos Sociais - FGTS"
"account_template_31101070107","3.11.01.07.01.07","(-) Social Charges – Other","expense","","(-) Encargos Sociais – Outros"
"account_template_31101070108","3.11.01.07.01.08","(-) Donations and Sponsors of Cultural and Artistic Character (Law No. 8.313/1991)","expense","","(-) Doações e Patrocínios de Caráter Cultural e Artístico (Lei no 8.313/1991)"
"account_template_31101070109","3.11.01.07.01.09","(-) Donations of Acquisition of Vale-Cultura (Law no 12.761/2012, art. 10)","expense","","(-) Doações de Aquisição de Vale-Cultura (Lei no 12.761/2012, art. 10)"
"account_template_31101070110","3.11.01.07.01.10","(-) Donations to Teaching and Research Institutions (Law No. 9,249/1995, Art.13, § 2o)","expense","","(-) Doações a Instituições de Ensino e Pesquisa (Lei nº 9.249/1995, art.13, § 2º)"
"account_template_31101070111","3.11.01.07.01.11","(-) Donations to Civil Entities","expense","","(-) Doações a Entidades Civis"
"account_template_31101070112","3.11.01.07.01.12","(-) Other Contributions, Donations and Sponsors","expense","","(-) Outras Contribuições, Doações e Patrocínios"
"account_template_31101070113","3.11.01.07.01.13","(-) Worker's Food","expense","","(-) Alimentação do Trabalhador"
"account_template_31101070114","3.11.01.07.01.14","(-) PIS/PASEP","expense","","(-) PIS/PASEP"
"account_template_31101070115","3.11.01.07.01.15","(-) COFINS","expense","","(-) COFINS"
"account_template_31101070116","3.11.01.07.01.16","(-) Further Taxes, Fees and Contributions, except IR and CSLL","expense","","(-) Demais Impostos, Taxas e Contribuições, exceto IR e CSLL"
"account_template_31101070117","3.11.01.07.01.17","(-) Lease","expense","","(-) Arrendamento Mercantil"
"account_template_31101070118","3.11.01.07.01.18","(-) Rentals","expense","","(-) Aluguéis"
"account_template_31101070119","3.11.01.07.01.19","(-) Expenditure with Vehicles and Goods and Facilities Conservation","expense","","(-) Despesas com Veículos e de Conservação de Bens e Instalações"
"account_template_31101070120","3.11.01.07.01.20","(-) Advertising, Publicity and Sponsorship","expense","","(-) Propaganda, Publicidade e Patrocínio"
"account_template_31101070121","3.11.01.07.01.21","(-) Advertising, Publicity and Sponsorship of Assoc. Sports Maintaining Professional Football Team","expense","","(-) Propaganda, Publicidade e Patrocínio de Assoc. Desportivas que Mantenha Equipe de Futebol Profissional"
"account_template_31101070122","3.11.01.07.01.22","(-) Fines","expense","","(-) Multas"
"account_template_31101070123","3.11.01.07.01.23","(-) Depreciation Charges","expense_depreciation","","(-) Encargos de Depreciação"
"account_template_31101070124","3.11.01.07.01.24","(-) Amortization Charges","expense_depreciation","","(-) Encargos de Amortização"
"account_template_31101070125","3.11.01.07.01.25","(-) Losses in Credit Operations","expense","","(-) Perdas em Operações de Crédito"
"account_template_31101070126","3.11.01.07.01.26","(-) Vacation Provisions","expense","","(-) Provisões para Férias"
"account_template_31101070127","3.11.01.07.01.27","(-) Provisions for 13th Employee Salary","expense","","(-) Provisões para 13º Salário de Empregados"
"account_template_31101070128","3.11.01.07.01.28","(-) Stock Loss Provision","expense","","(-) Provisão para Perda de Estoque"
"account_template_31101070129","3.11.01.07.01.29","(-) Further Provisions","expense","","(-) Demais Provisões"
"account_template_31101070130","3.11.01.07.01.30","(-) Gratifications to Administrators","expense","","(-) Gratificações a Administradores"
"account_template_31101070131","3.11.01.07.01.31","(-) Royalties and Technical Assistance - In the COUNTRY","expense","","(-) Royalties e Assistência Técnica - no PAÍS"
"account_template_31101070132","3.11.01.07.01.32","(-) Royalties and Technical Assistance - ABROAD","expense","","(-) Royalties e Assistência Técnica - no EXTERIOR"
"account_template_31101070133","3.11.01.07.01.33","(-) Medical, Dental and Pharmaceutical Assistance to Employees","expense","","(-) Assistência Médica, Odontológica e Farmacêutica a Empregados"
"account_template_31101070134","3.11.01.07.01.34","(-) Scientific and Technological Research","expense","","(-) Pesquisas Científicas e Tecnológicas"
"account_template_31101070135","3.11.01.07.01.35","(-) Goods of Small Unit Value or Useful Life of up to one Year Deducted as Expense","expense","","(-) Bens de Pequeno Valor Unitário ou de Vida Útil de até um Ano Deduzidos como Despesa"
"account_template_31101070136","3.11.01.07.01.36","(-) Electric Energy Expenditure","expense","","(-) Despesas com Energia Elétrica"
"account_template_31101070137","3.11.01.07.01.37","(-) Water and Sewage Expenditure","expense","","(-) Despesas com Água e Esgoto"
"account_template_31101070138","3.11.01.07.01.38","(-) Telephone and Internet expenditure","expense","","(-) Despesas com Telefone e Internet"
"account_template_31101070139","3.11.01.07.01.39","(-) Postal and Courier Expenses","expense","","(-) Despesas com Correios e Malotes"
"account_template_31101070140","3.11.01.07.01.40","(-) Insurance Expenditure","expense","","(-) Despesas com Seguros"
"account_template_31101090101","3.11.01.09.01.01","(-) Passive Exchange Variations","expense","","(-) Variações Cambiais Passivas"
"account_template_31101090102","3.11.01.09.01.02","(-) Incurred Losses in Variable Income Market except Day-Trade","expense","","(-) Perdas Incorridas no Mercado de Renda Variável, exceto Day-Trade"
"account_template_31101090103","3.11.01.09.01.03","(-) Day-Trade Operations Losses","expense","","(-) Perdas em Operações Day-Trade"
"account_template_31101090104","3.11.01.09.01.04","(-) Interest expenditure on equity","expense","","(-) Despesas de Juros sobre o Capital Próprio"
"account_template_31101090105","3.11.01.09.01.05","(-) Debenture Remuneration Expenses","expense","","(-) Despesas de Remuneração de Debêntures"
"account_template_31101090106","3.11.01.09.01.06","(-) Interest on Loans from Related Persons or Located in a Country with Favoured Taxation","expense","","(-) Juros com Empréstimos de Pessoas Vinculadas ou Situadas em País com Tributação favorecida"
"account_template_31101090107","3.11.01.09.01.07","(-) Financial Expenditure for Rent","expense","","(-) Despesas Financeiras Relativas a Arrendamento"
"account_template_31101090108","3.11.01.09.01.08","(-) Other Financial Expenses","expense","","(-) Outras Despesas Financeiras"
"account_template_31101090109","3.11.01.09.01.09","(-) Negative Results in Corporate Participation Valued by the Equity Method","expense","","(-) Resultados Negativos em Participações Societárias Avaliadas pelo Método de Equivalência Patrimonial"
"account_template_31101090110","3.11.01.09.01.10","(-) Negative results in SCP Valued by the Equity Method","expense","","(-) Resultados Negativos em SCP Avaliadas pelo Método de Equivalência Patrimonial"
"account_template_31101090111","3.11.01.09.01.11","(-) Loss in Operations Directed abroad","expense","","(-) Perdas em Operações Realizadas no Exterior"
"account_template_31101090112","3.11.01.09.01.12","(-) Estimated Losses Deriving from Recoverability Test (Impairment)","expense","","(-) Perdas Estimadas Decorrentes de Teste de Recuperabilidade (Impairment)"
"account_template_31101090113","3.11.01.09.01.13","(-) Expenses from Reclassification of Equity Valuation Adjustments","expense","","(-) Despesas de Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_31101090114","3.11.01.09.01.14","(-) Expenses from Reclassification of Equity Valuation Adjustments - Reflected","expense","","(-) Despesas de Reclassificação de Ajustes de Avaliação Patrimonial -Reflexo"
"account_template_31101090115","3.11.01.09.01.15","(-) Financial Expenses Resulting from Present Value Adjustments","expense_depreciation","","(-) Despesas Financeiras Decorrentes dos Ajustes ao Valor Presente"
"account_template_31101090116","3.11.01.09.01.16","(-) Benefit Depreciation Charges Rental Object","expense_depreciation","","(-) Encargos de Depreciação de Bens Objeto de Arrendamento"
"account_template_31101090117","3.11.01.09.01.17","(-) Capital Gain Amortization Charges","expense_depreciation","","(-) Encargos de Amortização de Mais - Valia"
"account_template_31101090118","3.11.01.09.01.18","(-) Real Estate Rentals- Renter Related Part","expense","","(-) Aluguéis de Bens Imóveis- Locador  Parte Relacionada"
"account_template_31101090119","3.11.01.09.01.19","(-) Leases of Real Estate Landlord Unrelated Party","expense","","(-) Aluguéis de Bens Imóveis Locador  Parte Não Relacionada"
"account_template_31101090120","3.11.01.09.01.20","(-) Expenditure with Securities Loans","expense","","(-) Despesas com Empréstimos de Valores Mobiliários"
"account_template_31101090121","3.11.01.09.01.21","(-) Brokerage and commission expenses","expense","","(-) Despesas com Corretagem e Emolumentos"
"account_template_31101090122","3.11.01.09.01.22","(-) Expenses with Negative Goodwill on Transfer of Securities","expense","","(-) Despesas  com Deságio na Cessão de Títulos"
"account_template_31101090123","3.11.01.09.01.23","(-) Expenditure in Mutual Operations – Related Party","expense","","(-) Despesas Incorridas em Operações de Mútuo – Parte Relacionada"
"account_template_31101090124","3.11.01.09.01.24","(-) Expenses Incurred in Loan Transactions - Unrelated Party","expense","","(-) Despesas Incorridas em Operações de Mútuo – Parte Não Relacionada"
"account_template_31101090125","3.11.01.09.01.25","(-) Expenditure incurred in Other Financial Liabilities Measured by Amortized Cost","expense","","(-) Despesas Incorridas  em Outros Passivos Financeiros Mensurados Pelo Custo Amortizado"
"account_template_31101090126","3.11.01.09.01.26","(-) Fair Value Adjustment Loss - Financial Instruments for Trading - No Hedge - Fair Value for Result","expense","","(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros para Negociação - Não Hedge - Valor Justo pelo Resultado"
"account_template_31101090127","3.11.01.09.01.27","(-) Fair Value Adjustment Loss - Available Financial Instruments for Sale - Reclassification of Equity Valuation Adjustments","expense","","(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros Disponíveis para Venda - Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_31101090128","3.11.01.09.01.28","(-) Fair Value Adjustment Loss - Fair Value Hedge Financial Instruments","expense","","(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros de Hedge de Valor Justo"
"account_template_31101090129","3.11.01.09.01.29","(-) Fair Value Adjustment Loss - Hedge Financial Instruments - Reclassification of Equity Valuation Adjustments","expense","","(-) Perda de Ajuste a Valor Justo - Instrumentos Financeiros de Hedge -  Reclassificação de Ajustes de Avaliação Patrimonial"
"account_template_31101090130","3.11.01.09.01.30","(-) Loss of Adjustment to Fair Value - Fair Value Hedge Item","expense","","(-) Perda de Ajuste a Valor Justo - Item Objeto de Hedge de Valor Justo"
"account_template_31101090131","3.11.01.09.01.31","(-) Loss of Adjustment to Fair Value - Property for Investment","expense","","(-) Perda de Ajuste a Valor Justo - Propriedade para Investimento"
"account_template_31101090132","3.11.01.09.01.32","(-) Fair Value Adjustment Loss - Consumable Biological Asset","expense","","(-) Perda de Ajuste a Valor Justo - Ativo Biológico Consumível"
"account_template_31101090133","3.11.01.09.01.33","(-) Fair Value Adjustment Loss - Biological Production Act","expense","","(-) Perda de Ajuste a Valor Justo - Ativo Biológico de Produção"
"account_template_31101090134","3.11.01.09.01.34","(-) Fair Value Adjustment Loss - Non-Current Assets Held for Sale","expense","","(-) Perda de Ajuste a Valor Justo - Ativos Não Circulantes Mantidos para Venda"
"account_template_31101090135","3.11.01.09.01.35","(-) Loss of Adjustment to Fair Value - Subscription of Capital with other Goods","expense","","(-) Perda de Ajuste a Valor Justo - Subscrição de Capital com demais Bens"
"account_template_31101090136","3.11.01.09.01.36","(-) Loss of Adjustment to Fair Value - Equity Subscription with Corporate Participation","expense","","(-) Perda de Ajuste a Valor Justo - Subscrição de Capital com Participação Societária"
"account_template_31101090137","3.11.01.09.01.37","(-) Fair Value Adjustment Loss - Acquisition of Equity Interest in Stages","expense","","(-) Perda de Ajuste a Valor Justo - Aquisição de Participação Societária em Estágios"
"account_template_31101090138","3.11.01.09.01.38","(-) Fair Value Adjustment Loss - Due to Exchange of Assets or Liabilities","expense","","(-) Perda de Ajuste a Valor Justo - Decorrente de Permuta de Ativos ou Passivos"
"account_template_31101090139","3.11.01.09.01.39","(-) Loss of Adjustment to Fair Value - Other Operations","expense","","(-) Perda de Ajuste a Valor Justo - Outras Operações"
"account_template_31101090199","3.11.01.09.01.99","(-) Other Operational Expenses","expense","","(-) Outras Despesas Operacionais"
"account_template_31101110101","3.11.01.11.01.01","Income from the Sale of Assets under Current Assets or Long-Term Assets","income","","Receitas na Alienação de Bens Integrantes do Ativo Circulante ou do Ativo Realizável a Longo Prazo"
"account_template_31101110102","3.11.01.11.01.02","Income from Disposal of Non-Current Assets","income","","Receitas de Alienações de Bens do Ativo Não Circulante"
"account_template_31101110104","3.11.01.11.01.04","(-) Book Value of Assets Comprising Current Assets or Disposed of Long-Term Receivable Assets","expense","","(-) Valor Contábil de Bens Integrantes do Ativo Circulante ou do Ativo Realizável a Longo Prazo Alienados"
"account_template_31101110105","3.11.01.11.01.05","(-) Book Value of Sold Non-Current Assets","expense","","(-) Valor Contábil dos Bens do Ativo Não Circulante Alienados"
"account_template_31105010102","3.11.05.01.01.02","(-) Contributions for Employees' Assistance or Welfare","expense","","(-) Contribuições para Assistência ou Previdência de Empregados"
"account_template_31105010199","3.11.05.01.01.99","(-) Other Employee Participations","expense","","(-) Outras Participações de Empregados"
"account_template_31105010301","3.11.05.01.03.01","(-) Participation of Administrators and Beneficiary Parties","expense","","(-) Participações de Administradores e Partes Beneficiárias"
"account_template_31105010302","3.11.05.01.03.02","(-) Interests in Debentures","expense","","(-) Participações de Debêntures"
"account_template_31105010399","3.11.05.01.03.99","(-) Other Participations","expense","","(-) Outras Participações"
"account_template_31201010101","3.12.01.01.01.01","Social Contribution on Net Profit (Rural Activity)","expense","","Contribuição Social sobre o Lucro Líquido (Atividade Rural)"
"account_template_31201010111","3.12.01.01.01.11","Social Contribution on Net Profit - Deferred Profits (Rural Activity)","expense","","Contribuição Social sobre o Lucro Líquido - Lucros Diferidos (Atividade Rural)"
"br_3_01_01_05_01_47","3.01.01.05.01.47","Foreign Exchange Gain","income_other","","Ganho Cambial"
"br_3_11_01_09_01_40","3.11.01.09.01.40","Foreign Exchange Loss","expense","","Perda Cambial"

```

## File: data\template\account.fiscal.position-br.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","l10n_br_fp_type","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id","name@pt"
"fiscal_position_template_1","1","Internal (within one state)","1","1","base.br","internal","tax_template_out_icms_externo17","tax_template_out_icms_interno17","","","Internal (within one state)"
"","","","","","","","tax_template_in_icms_externo17","tax_template_in_icms_interno17","","",""
"fiscal_position_template_2","2","Foreign","1","","","","tax_template_out_icms_interno17","tax_template_out_icms_externo","","","Foreign"
"","","","","","","","tax_template_out_icms_externo17","tax_template_out_icms_externo","","",""
"","","","","","","","tax_template_out_ipi10","tax_template_out_ipi","","",""
"","","","","","","","tax_template_in_icms_interno17","tax_template_in_icms_interno","","",""
"","","","","","","","tax_template_in_icms_externo17","tax_template_in_icms_externo","","",""
"","","","","","","","tax_template_in_ipi10","tax_template_in_ipi","","",""
"","","","","","","","","","account_template_30101010105","account_template_30101010101",""
"","","","","","","","","","account_template_30101010106","account_template_30101010103",""
"fiscal_position_template_ss_nnm","3","South and Southeast to North, Northeast, and Midwest","1","1","base.br","ss_nnm","tax_template_out_icms_externo17","tax_template_out_icms_externo7","","","South and Southeast to North, Northeast, and Midwest"
"","","","","","","","tax_template_out_icms_interno17","tax_template_out_icms_externo7","","",""
"","","","","","","","tax_template_in_icms_externo17","tax_template_in_icms_externo7","","",""
"","","","","","","","tax_template_in_icms_interno17","tax_template_in_icms_externo7","","",""
"fiscal_position_template_interstate","4","Interstate","1","1","base.br","interstate","tax_template_out_icms_externo17","tax_template_out_icms_externo12","","","Interstate"
"","","","","","","","tax_template_out_icms_interno17","tax_template_out_icms_externo12","","",""
"","","","","","","","tax_template_in_icms_externo17","tax_template_in_icms_externo12","","",""
"","","","","","","","tax_template_in_icms_interno17","tax_template_in_icms_externo12","","",""

```

## File: data\template\account.tax-br.csv

```csv
"id","active","description","invoice_label","name","amount","type_tax_use","price_include","tax_discount","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@pt"
"tax_template_out_icms_interno17","True","ICMS Internal Sale 17%","ICMS Internal 17%","17% ICMS I","17.0","sale","False","1","tax_group_icms_17","base","invoice","+ICMS_1","","ICMS Saída Interno 17%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_out_icms_externo17","True","ICMS External Sale 17%","ICMS External 17%","17% ICMS E","17.0","sale","False","1","tax_group_icms_17","base","invoice","+ICMS_1","","ICMS Saída Externo 17%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_in_icms_interno17","True","ICMS Internal Purchase 17%","ICMS Internal 17%","17% ICMS I","17.0","purchase","False","1","tax_group_icms_17","base","invoice","-ICMS_1","","ICMS Entrada Interno 17%"
"","","","","","","","","","","tax","invoice","-ICMS_2","account_template_101020302",""
"","","","","","","","","","","base","refund","+ICMS_1","",""
"","","","","","","","","","","tax","refund","+ICMS_2","account_template_201010903",""
"tax_template_in_icms_externo17","True","ICMS External Purchase 17%","ICMS External 17%","17% ICMS E","17.0","purchase","False","1","tax_group_icms_17","base","invoice","-ICMS_1","","ICMS Entrada Externo 17%"
"","","","","","","","","","","tax","invoice","-ICMS_2","account_template_101020302",""
"","","","","","","","","","","base","refund","+ICMS_1","",""
"","","","","","","","","","","tax","refund","+ICMS_2","account_template_201010903",""
"tax_template_out_icms_interno","True","ICMS Internal Sale 0%","ICMS Internal market","0% ICMS I","0.0","sale","False","1","tax_group_icms_0","base","invoice","+ICMS_2","","ICMS Saída Interno 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-ICMS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_icms_externo","True","ICMS External Sale 0%","ICMS External","0% ICMS E","0.0","sale","False","1","tax_group_icms_0","base","invoice","+ICMS_2","","ICMS Saída Externo 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-ICMS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_icms_interno","True","ICMS Internal Purchase 0%","ICMS Internal market","0% ICMS I","0.0","purchase","False","1","tax_group_icms_0","base","invoice","-ICMS_2","","ICMS Entrada Interno 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+ICMS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_icms_externo","True","ICMS External Purchase 0%","ICMS External","0% ICMS E","0.0","purchase","False","1","tax_group_icms_0","base","invoice","-ICMS_2","","ICMS Entrada Externo 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+ICMS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_icms_externo7","True","ICMS External Sale 7%","ICMS External 7%","7% ICMS E","7.0","sale","False","1","tax_group_icms_7","base","invoice","+ICMS_1","","ICMS Saída Externo 7%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_in_icms_externo7","True","ICMS External Purchase 7%","ICMS External 7%","7% ICMS E","7.0","purchase","","1","tax_group_icms_7","base","invoice","+ICMS_1","","ICMS Entrada Externo 7%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_out_icms_externo12","True","ICMS External Sale 12%","ICMS External 12%","12% ICMS E","12.0","sale","False","1","tax_group_icms_12","base","invoice","+ICMS_1","","ICMS Saída Externo 12%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_in_icms_externo12","True","ICMS External Purchase 12%","ICMS External 12%","12% ICMS E","12.0","purchase","","1","tax_group_icms_12","base","invoice","+ICMS_1","","ICMS Entrada Externo 12%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_out_icms_subist","True","ICMS Subist Sale 0%","ICMS Subist","0% ICMS S","0.0","sale","False","0","tax_group_icms_0","base","invoice","+ICMSST_1","","ICMS Saída Subist 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-ICMSST_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_icms_subist","True","ICMS Subist Purchase 0%","ICMS Subist","0% ICMS S","0.0","purchase","False","0","tax_group_icms_0","base","invoice","-ICMSST_1","","ICMS Entrada Subist 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+ICMSST_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_ipi10","True","IPI Sale 10%","IPI 10%","10% IPI","10.0","sale","False","0","tax_group_ipi_10","base","invoice","+IPI_1","","IPI Saída 10%"
"","","","","","","","","","","tax","invoice","+IPI_2","account_template_201010902",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","-IPI_2","account_template_101020301",""
"tax_template_in_ipi10","True","IPI Purchase 10%","IPI 10%","10% IPI","10.0","purchase","False","0","tax_group_ipi_10","base","invoice","-IPI_1","","IPI Entrada 10%"
"","","","","","","","","","","tax","invoice","-IPI_2","account_template_101020301",""
"","","","","","","","","","","base","refund","+IPI_1","",""
"","","","","","","","","","","tax","refund","+IPI_2","account_template_201010902",""
"tax_template_out_ipi","True","IPI Sale 0%","IPI","0% IPI","0.0","sale","False","0","tax_group_ipi_0","base","invoice","+IPI_1","","IPI Saída 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_ipi","True","IPI Purchase 0%","IPI","0% IPI","0.0","purchase","False","0","tax_group_ipi_0","base","invoice","-IPI_1","","IPI Entrada 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+IPI_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_pis","True","PIS Sale 0%","PIS","0% PIS","0.0","sale","False","1","tax_group_pis_0","base","invoice","+PIS_2","","PIS Saída 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-PIS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_pis065","True","PIS Sale 0.65%","PIS 0,65%","0.65% PIS","0.65","sale","False","1","tax_group_pis_065","base","invoice","+PIS_1","","PIS Saída 0,65%"
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_101020303",""
"tax_template_in_pis","True","PIS Purchase 0%","PIS","0% PIS","0.0","purchase","False","1","tax_group_pis_0","base","invoice","-PIS_2","","PIS Entrada 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+PIS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_pis065","True","PIS Purchase 0.65%","PIS 0,65%","0.65% PIS","0.65","purchase","False","1","tax_group_pis_065","base","invoice","-PIS_1","","PIS Entrada 0,65%"
"","","","","","","","","","","tax","invoice","-PIS_2","account_template_101020303",""
"","","","","","","","","","","base","refund","+PIS_1","",""
"","","","","","","","","","","tax","refund","+PIS_2","account_template_201010904",""
"tax_template_out_cofins","True","COFINS Sale 0%","COFINS","0% COFINS","0.0","sale","False","1","tax_group_cofins_0","base","invoice","+COFINS_1","","COFINS Saída 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_cofins3","True","COFINS Sale 3%","COFINS 3%","3% COFINS","3.0","sale","False","1","tax_group_cofins_3","base","invoice","+COFINS_1","","COFINS Saída 3%"
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_101020305",""
"tax_template_in_cofins","True","COFINS Purchase 0%","COFINS","0% COFINS","0.0","purchase","False","1","tax_group_cofins_0","base","invoice","-COFINS_1","","COFINS Entrada 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+COFINS_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_cofins3","True","COFINS Purchase 3%","COFINS 3%","3% COFINS","3.0","purchase","False","1","tax_group_cofins_3","base","invoice","-COFINS_1","","COFINS Entrada 3%"
"","","","","","","","","","","tax","invoice","-COFINS_2","account_template_101020305",""
"","","","","","","","","","","base","refund","+COFINS_1","",""
"","","","","","","","","","","tax","refund","+COFINS_2","account_template_201010905",""
"tax_template_out_irpj","True","IRPJ 0%","IRPJ","0% IRPJ","0.0","sale","False","1","tax_group_irpj_0","base","invoice","+IRPJ_1","","IRPJ 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-IRPJ_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_ir","True","IR 0%","IR","0% IR","0.0","sale","False","1","tax_group_ir_0","base","invoice","+IR_1","","IR 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-IR_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_issqn2","True","ISSQN Sale 2%","ISSQN 2%","2% ISSQN","2.0","sale","False","1","tax_group_issqn_2","base","invoice","+ISSQN_1","","ISSQN Saída 2%"
"","","","","","","","","","","tax","invoice","+ISSQN_2","account_template_101020340",""
"","","","","","","","","","","base","refund","-ISSQN_1","",""
"","","","","","","","","","","tax","refund","-ISSQN_2","account_template_201010928",""
"tax_template_in_issqn2","True","ISSQN Purchase 2%","ISSQN 2%","2% ISSQN","2.0","purchase","False","1","tax_group_issqn_2","base","invoice","-ISSQN_1","","ISSQN Entrada 2%"
"","","","","","","","","","","tax","invoice","-ISSQN_2","account_template_101020340",""
"","","","","","","","","","","base","refund","+ISSQN_1","",""
"","","","","","","","","","","tax","refund","+ISSQN_2","account_template_201010928",""
"tax_template_out_csll","True","CSLL 0%","CSLL","0% CSLL","0.0","sale","False","1","tax_group_csll_0","base","invoice","+CSLL_1","","CSLL 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-CSLL_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_ii0","True","II Sale 0%","II","0% II","0.0","sale","False","0","tax_group_ii_0","base","invoice","+II_1","","II Saída 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-II_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_ii0","True","II Purchase 0%","II","0% II","0.0","purchase","False","0","tax_group_ii_0","base","invoice","+II_1","","II Entrada 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-II_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_inss0","True","INSS Sale 0%","INSS","0% INSS","0.0","sale","False","0","tax_group_inss_0","base","invoice","+INSS_1","","INSS Saída 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_inss0","True","INSS Purchase 0%","INSS","0% INSS","0.0","purchase","False","0","tax_group_inss_0","base","invoice","+INSS_1","","INSS Entrada 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_aproxtrib_fed_incl_goods","False","","Approximated Federal Taxation Incl.","Approximated Federal Taxation Incl.","1.00","sale","True","","tax_group_aproxtrib_fed_incl_goods","base","invoice","","","Tributação Federal Aproximada Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_aproxtrib_fed_excl_goods","False","","Approximated Federal Taxation Excl.","Approximated Federal Taxation Excl.","1.00","sale","False","","tax_group_aproxtrib_fed_excl_goods","base","invoice","","","Tributação Federal Aproximada Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_aproxtrib_state_incl_goods","False","","Approximated State Taxation Incl.","Approximated State Taxation Incl.","1.00","sale","True","","tax_group_aproxtrib_state_incl_goods","base","invoice","","","Tributação Estadual Aproximada Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_aproxtrib_state_excl_goods","False","","Approximated State Taxation Excl.","Approximated State Taxation Excl.","1.00","sale","False","","tax_group_aproxtrib_state_excl_goods","base","invoice","","","Tributação Estadual Aproximada Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_cofins_incl_goods","False","","COFINS Incl.","COFINS Incl.","1.00","sale","True","","tax_group_cofins_incl_goods","base","invoice","+COFINS_1","","COFINS Incl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_excl_goods","False","","COFINS Excl.","COFINS Excl.","1.00","sale","False","","tax_group_cofins_excl_goods","base","invoice","+COFINS_1","","COFINS Excl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_deson_incl_goods","False","","COFINS Exemption Incl.","COFINS Exemption Incl.","1.00","sale","True","","tax_group_cofins_deson_incl_goods","base","invoice","+COFINS_1","","COFINS Desoneração Incl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_deson_excl_goods","False","","COFINS Exemption Excl.","COFINS Exemption Excl.","1.00","sale","False","","tax_group_cofins_deson_excl_goods","base","invoice","+COFINS_1","","COFINS Desoneração Excl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_st_incl_goods","False","","COFINS ST Incl.","COFINS ST Incl.","1.00","sale","True","","tax_group_cofins_st_incl_goods","base","invoice","+COFINS_1","","COFINS ST Incl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_st_excl_goods","False","","COFINS ST Excl.","COFINS ST Excl.","1.00","sale","False","","tax_group_cofins_st_excl_goods","base","invoice","+COFINS_1","","COFINS ST Excl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_icms_incl_goods","False","","ICMS Incl.","ICMS Incl.","1.00","sale","True","","tax_group_icms_incl_goods","base","invoice","+ICMS_1","","ICMS Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_excl_goods","False","","ICMS Excl.","ICMS Excl.","1.00","sale","False","","tax_group_icms_excl_goods","base","invoice","+ICMS_1","","ICMS Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_credsn_incl_goods","False","","ICMS CredSN Incl.","ICMS CredSN Incl.","1.00","sale","True","","tax_group_icms_credsn_incl_goods","base","invoice","+ICMS_1","","ICMS CredSN Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_credsn_excl_goods","False","","ICMS CredSN Excl.","ICMS CredSN Excl.","1.00","sale","False","","tax_group_icms_credsn_excl_goods","base","invoice","+ICMS_1","","ICMS CredSN Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_deson_incl_goods","False","","ICMS Exemption Incl.","ICMS Exemption Incl.","1.00","sale","True","","tax_group_icms_deson_incl_goods","base","invoice","+ICMS_1","","ICMS Desoneração Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_deson_excl_goods","False","","ICMS Exemption Excl.","ICMS Exemption Excl.","1.00","sale","False","","tax_group_icms_deson_excl_goods","base","invoice","+ICMS_1","","ICMS Desoneração Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_dest_incl_goods","False","","ICMS DIFA Recipient Incl.","ICMS DIFA Recipient Incl.","1.00","sale","True","","tax_group_icms_difa_dest_incl_goods","base","invoice","+ICMS_1","","ICMS DIFA Destinatário Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_dest_excl_goods","False","","ICMS DIFA Recipient Excl.","ICMS DIFA Recipient Excl.","1.00","sale","False","","tax_group_icms_difa_dest_excl_goods","base","invoice","+ICMS_1","","ICMS DIFA Destinatário Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_fcp_incl_goods","False","","ICMS DIFA FCP Incl.","ICMS DIFA FCP Incl.","1.00","sale","True","","tax_group_icms_difa_fcp_incl_goods","base","invoice","+ICMS_1","","ICMS DIFA FCP Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_fcp_excl_goods","False","","ICMS DIFA FCP Excl.","ICMS DIFA FCP Excl.","1.00","sale","False","","tax_group_icms_difa_fcp_excl_goods","base","invoice","+ICMS_1","","ICMS DIFA FCP Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_remet_incl_goods","False","","ICMS DIFA Sender Incl.","ICMS DIFA Sender Incl.","1.00","sale","True","","tax_group_icms_difa_remet_incl_goods","base","invoice","+ICMS_1","","ICMS DIFA Remetente Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_remet_excl_goods","False","","ICMS DIFA Sender Excl.","ICMS DIFA Sender Excl.","1.00","sale","False","","tax_group_icms_difa_remet_excl_goods","base","invoice","+ICMS_1","","ICMS DIFA Remetente Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_eff_incl_goods","False","","ICMS EFF Incl.","ICMS EFF Incl.","1.00","sale","True","","tax_group_icms_eff_incl_goods","base","invoice","+ICMS_1","","ICMS EFF Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_eff_excl_goods","False","","ICMS EFF Excl.","ICMS EFF Excl.","1.00","sale","False","","tax_group_icms_eff_excl_goods","base","invoice","+ICMS_1","","ICMS EFF Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_fcp_incl_goods","False","","ICMS FCP Incl.","ICMS FCP Incl.","1.00","sale","True","","tax_group_icms_fcp_incl_goods","base","invoice","+ICMS_1","","ICMS FCP Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_fcp_excl_goods","False","","ICMS FCP Excl.","ICMS FCP Excl.","1.00","sale","False","","tax_group_icms_fcp_excl_goods","base","invoice","+ICMS_1","","ICMS FCP Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_own_payer_incl_goods","False","","ICMS Own Issuer Incl.","ICMS Own Issuer Incl.","1.00","sale","True","","tax_group_icms_own_payer_incl_goods","base","invoice","+ICMS_1","","ICMS Próprio Emitente Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_own_payer_excl_goods","False","","ICMS Own Issuer Excl.","ICMS Own Issuer Excl.","1.00","sale","False","","tax_group_icms_own_payer_excl_goods","base","invoice","+ICMS_1","","ICMS Próprio Emitente Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_part_incl_goods","False","","ICMS Sharing Incl.","ICMS Sharing Incl.","1.00","sale","True","","tax_group_icms_part_incl_goods","base","invoice","+ICMS_1","","ICMS Partilha Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_part_excl_goods","False","","ICMS Sharing Excl.","ICMS Sharing Excl.","1.00","sale","False","","tax_group_icms_part_excl_goods","base","invoice","+ICMS_1","","ICMS Partilha Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_rf_incl_goods","False","","ICMS RF Incl.","ICMS RF Incl.","1.00","sale","True","","tax_group_icms_rf_incl_goods","base","invoice","+ICMS_1","","ICMS RF Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_rf_excl_goods","False","","ICMS RF Excl.","ICMS RF Excl.","1.00","sale","False","","tax_group_icms_rf_excl_goods","base","invoice","+ICMS_1","","ICMS RF Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_incl_goods","False","","ICMS ST Incl.","ICMS ST Incl.","1.00","sale","True","","tax_group_icms_st_incl_goods","base","invoice","+ICMS_1","","ICMS ST Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_excl_goods","False","","ICMS ST Excl.","ICMS ST Excl.","1.00","sale","False","","tax_group_icms_st_excl_goods","base","invoice","+ICMS_1","","ICMS ST Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_fcp_incl_goods","False","","ICMS ST FCP Incl.","ICMS ST FCP Incl.","1.00","sale","True","","tax_group_icms_st_fcp_incl_goods","base","invoice","+ICMS_1","","ICMS ST FCP Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_fcp_excl_goods","False","","ICMS ST FCP Excl.","ICMS ST FCP Excl.","1.00","sale","False","","tax_group_icms_st_fcp_excl_goods","base","invoice","+ICMS_1","","ICMS ST FCP Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_fcppart_incl_goods","False","","ICMS ST FCP Sharing Incl.","ICMS ST FCP Sharing Incl.","1.00","sale","True","","tax_group_icms_st_fcppart_incl_goods","base","invoice","+ICMS_1","","ICMS ST FCP Partilha Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_fcppart_excl_goods","False","","ICMS ST FCP Sharing Excl.","ICMS ST FCP Sharing Excl.","1.00","sale","False","","tax_group_icms_st_fcppart_excl_goods","base","invoice","+ICMS_1","","ICMS ST FCP Partilha Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_part_incl_goods","False","","ICMS ST Sharing Incl.","ICMS ST Sharing Incl.","1.00","sale","True","","tax_group_icms_st_part_incl_goods","base","invoice","+ICMS_1","","ICMS ST Partilha Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_part_excl_goods","False","","ICMS ST Sharing Excl.","ICMS ST Sharing Excl.","1.00","sale","False","","tax_group_icms_st_part_excl_goods","base","invoice","+ICMS_1","","ICMS ST Partilha Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_sd_incl_goods","False","","ICMS ST SD Incl.","ICMS ST SD Incl.","1.00","sale","True","","tax_group_icms_st_sd_incl_goods","base","invoice","+ICMS_1","","ICMS ST SD Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_sd_excl_goods","False","","ICMS ST SD Excl.","ICMS ST SD Excl.","1.00","sale","False","","tax_group_icms_st_sd_excl_goods","base","invoice","+ICMS_1","","ICMS ST SD Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_sd_fcp_incl_goods","False","","ICMS ST SD FCP Incl.","ICMS ST SD FCP Incl.","1.00","sale","True","","tax_group_icms_st_sd_fcp_incl_goods","base","invoice","+ICMS_1","","ICMS ST SD FCP Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_sd_fcp_excl_goods","False","","ICMS ST SD FCP Excl.","ICMS ST SD FCP Excl.","1.00","sale","False","","tax_group_icms_st_sd_fcp_excl_goods","base","invoice","+ICMS_1","","ICMS ST SD FCP Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_ii_incl_goods","False","","II - Import Tax Incl.","II - Import Tax Incl.","1.00","sale","True","","tax_group_ii_incl_goods","base","invoice","+II_1","","II - Imposto de Importação Incl."
"","","","","","","","","","","tax","invoice","+II_2","account_template_201010928",""
"","","","","","","","","","","base","refund","-II_1","",""
"","","","","","","","","","","tax","refund","-II_2","account_template_201010928",""
"tax_template_out_ii_excl_goods","False","","II - Import Tax Excl.","II - Import Tax Excl.","1.00","sale","False","","tax_group_ii_excl_goods","base","invoice","+II_1","","II - Imposto de Importação Excl."
"","","","","","","","","","","tax","invoice","+II_2","account_template_201010928",""
"","","","","","","","","","","base","refund","-II_1","",""
"","","","","","","","","","","tax","refund","-II_2","account_template_201010928",""
"tax_template_out_iof_incl_goods","False","","IOF - Tax on Financial Operations Incl.","IOF - Tax on Financial Operations Incl.","1.00","sale","True","","tax_group_iof_incl_goods","base","invoice","","","IOF - Imposto sobre Operações Financeiras Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010906",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010906",""
"tax_template_out_iof_excl_goods","False","","IOF - Tax on Financial Operations Excl.","IOF - Tax on Financial Operations Excl.","1.00","sale","False","","tax_group_iof_excl_goods","base","invoice","","","IOF - Imposto sobre Operações Financeiras Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010906",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010906",""
"tax_template_out_ipi_incl_goods","False","","IPI Incl.","IPI Incl.","1.00","sale","True","","tax_group_ipi_incl_goods","base","invoice","+IPI_1","","IPI Incl."
"","","","","","","","","","","tax","invoice","+IPI_2","account_template_201010902",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","-IPI_2","account_template_201010902",""
"tax_template_out_ipi_excl_goods","False","","IPI Excl.","IPI Excl.","1.00","sale","False","","tax_group_ipi_excl_goods","base","invoice","+IPI_1","","IPI Excl."
"","","","","","","","","","","tax","invoice","+IPI_2","account_template_201010902",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","-IPI_2","account_template_201010902",""
"tax_template_out_ipi_returned_incl_goods","False","","IPI Returned Incl.","IPI Returned Incl.","1.00","sale","True","","tax_group_ipi_returned_incl_goods","base","invoice","+IPI_1","","IPI Retornado Incl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010902",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","-IPI_2","account_template_201010902",""
"tax_template_out_ipi_returned_excl_goods","False","","IPI Returned Excl.","IPI Returned Excl.","1.00","sale","False","","tax_group_ipi_returned_excl_goods","base","invoice","+IPI_1","","IPI Retornado Excl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010902",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","-IPI_2","account_template_201010902",""
"tax_template_out_pis_incl_goods","False","","PIS Incl.","PIS Incl.","1.00","sale","True","","tax_group_pis_incl_goods","base","invoice","+PIS_1","","PIS Incl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_excl_goods","False","","PIS Excl.","PIS Excl.","1.00","sale","False","","tax_group_pis_excl_goods","base","invoice","+PIS_1","","PIS Excl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_deson_incl_goods","False","","PIS Exemption Incl.","PIS Exemption Incl.","1.00","sale","True","","tax_group_pis_deson_incl_goods","base","invoice","+PIS_1","","PIS Desoneração Incl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_deson_excl_goods","False","","PIS Exemption Excl.","PIS Exemption Excl.","1.00","sale","False","","tax_group_pis_deson_excl_goods","base","invoice","+PIS_1","","PIS Desoneração Excl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_st_incl_goods","False","","PIS ST Incl.","PIS ST Incl.","1.00","sale","True","","tax_group_pis_st_incl_goods","base","invoice","+PIS_1","","PIS ST Incl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_st_excl_goods","False","","PIS ST Excl.","PIS ST Excl.","1.00","sale","False","","tax_group_pis_st_excl_goods","base","invoice","+PIS_1","","PIS ST Excl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_aproxtrib_city_incl_service","False","","Approximated City Taxation Incl.","Approximated City Taxation Incl.","1","sale","True","","tax_group_aproxtrib_city_incl_services","base","invoice","","","Tributação Municipal Aproximada Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010908",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010908",""
"tax_template_out_aproxtrib_city_excl_service","False","","Approximated City Taxation Excl.","Approximated City Taxation Excl.","1","sale","False","","tax_group_aproxtrib_city_excl_services","base","invoice","","","Tributação Municipal Aproximada Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010908",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010908",""
"tax_template_out_pis_rf_incl_service","False","","PIS RF Incl.","PIS RF Incl.","1","sale","True","","tax_group_pis_rf_incl_services","base","invoice","+PIS_1","","PIS RF Incl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_rf_excl_service","False","","PIS RF Excl.","PIS RF Excl.","1","sale","False","","tax_group_pis_rf_excl_services","base","invoice","+PIS_1","","PIS RF Excl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_cofins_rf_incl_service","False","","COFINS RF Incl.","COFINS RF Incl.","1","sale","True","","tax_group_cofins_rf_incl_services","base","invoice","+COFINS_1","","COFINS RF Incl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_rf_excl_service","False","","COFINS RF Excl.","COFINS RF Excl.","1","sale","False","","tax_group_cofins_rf_excl_services","base","invoice","+COFINS_1","","COFINS RF Excl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_csll_incl_service","False","","CSLL Incl.","CSLL Incl.","1","sale","True","","tax_group_csll_incl_services","base","invoice","+CSLL_1","","CSLL Incl."
"","","","","","","","","","","tax","invoice","+CSLL_2","account_template_201010914",""
"","","","","","","","","","","base","refund","-CSLL_1","",""
"","","","","","","","","","","tax","refund","-CSLL_2","account_template_201010914",""
"tax_template_out_csll_excl_service","False","","CSLL Excl.","CSLL Excl.","1","sale","False","","tax_group_csll_excl_services","base","invoice","+CSLL_1","","CSLL Excl."
"","","","","","","","","","","tax","invoice","+CSLL_2","account_template_201010914",""
"","","","","","","","","","","base","refund","-CSLL_1","",""
"","","","","","","","","","","tax","refund","-CSLL_2","account_template_201010914",""
"tax_template_out_csll_rf_incl_service","False","","CSLL RF Incl.","CSLL RF Incl.","1","sale","True","","tax_group_csll_rf_incl_services","base","invoice","+CSLL_1","","CSLL RF Incl."
"","","","","","","","","","","tax","invoice","+CSLL_2","account_template_201010914",""
"","","","","","","","","","","base","refund","-CSLL_1","",""
"","","","","","","","","","","tax","refund","-CSLL_2","account_template_201010914",""
"tax_template_out_csll_rf_excl_service","False","","CSLL RF Excl.","CSLL RF Excl.","1","sale","False","","tax_group_csll_rf_excl_services","base","invoice","+CSLL_1","","CSLL RF Excl."
"","","","","","","","","","","tax","invoice","+CSLL_2","account_template_201010914",""
"","","","","","","","","","","base","refund","-CSLL_1","",""
"","","","","","","","","","","tax","refund","-CSLL_2","account_template_201010914",""
"tax_template_out_iss_incl_service","False","","ISS Incl.","ISS Incl.","1","sale","True","","tax_group_iss_incl_services","base","invoice","+ISSQN_1","","ISS Incl."
"","","","","","","","","","","tax","invoice","+ISSQN_2","account_template_201010908",""
"","","","","","","","","","","base","refund","-ISSQN_1","",""
"","","","","","","","","","","tax","refund","-ISSQN_2","account_template_201010908",""
"tax_template_out_iss_excl_service","False","","ISS Excl.","ISS Excl.","1","sale","False","","tax_group_iss_excl_services","base","invoice","+ISSQN_1","","ISS Excl."
"","","","","","","","","","","tax","invoice","+ISSQN_2","account_template_201010908",""
"","","","","","","","","","","base","refund","-ISSQN_1","",""
"","","","","","","","","","","tax","refund","-ISSQN_2","account_template_201010908",""
"tax_template_out_iss_rf_incl_service","False","","ISS RF Incl.","ISS RF Incl.","1","sale","True","","tax_group_iss_rf_incl_services","base","invoice","+ISSQN_1","","ISS RF Incl."
"","","","","","","","","","","tax","invoice","+ISSQN_2","account_template_201010908",""
"","","","","","","","","","","base","refund","-ISSQN_1","",""
"","","","","","","","","","","tax","refund","-ISSQN_2","account_template_201010908",""
"tax_template_out_iss_rf_excl_service","False","","ISS RF Excl.","ISS RF Excl.","1","sale","False","","tax_group_iss_rf_excl_services","base","invoice","+ISSQN_1","","ISS RF Excl."
"","","","","","","","","","","tax","invoice","+ISSQN_2","account_template_201010908",""
"","","","","","","","","","","base","refund","-ISSQN_1","",""
"","","","","","","","","","","tax","refund","-ISSQN_2","account_template_201010908",""
"tax_template_out_ir_pj_incl_service","False","","IR PJ Incl.","IR PJ Incl.","1","sale","True","","tax_group_ir_pj_incl_services","base","invoice","+IRPJ_1","","IR PJ Incl."
"","","","","","","","","","","tax","invoice","+IRPJ_2","account_template_201010901",""
"","","","","","","","","","","base","refund","-IRPJ_1","",""
"","","","","","","","","","","tax","refund","-IRPJ_2","account_template_201010901",""
"tax_template_out_ir_pj_excl_service","False","","IR PJ Excl.","IR PJ Excl.","1","sale","False","","tax_group_ir_pj_excl_services","base","invoice","+IRPJ_1","","IR PJ Excl."
"","","","","","","","","","","tax","invoice","+IRPJ_2","account_template_201010901",""
"","","","","","","","","","","base","refund","-IRPJ_1","",""
"","","","","","","","","","","tax","refund","-IRPJ_2","account_template_201010901",""
"tax_template_out_ir_rf_incl_service","False","","IR RF Incl.","IR RF Incl.","1","sale","True","","tax_group_ir_rf_incl_services","base","invoice","+IR_1","","IR RF Incl."
"","","","","","","","","","","tax","invoice","+IR_2","account_template_201010901",""
"","","","","","","","","","","base","refund","-IR_1","",""
"","","","","","","","","","","tax","refund","-IR_2","account_template_201010901",""
"tax_template_out_ir_rf_excl_service","False","","IR RF Excl.","IR RF Excl.","1","sale","False","","tax_group_ir_rf_excl_services","base","invoice","+IR_1","","IR RF Excl."
"","","","","","","","","","","tax","invoice","+IR_2","account_template_201010901",""
"","","","","","","","","","","base","refund","-IR_1","",""
"","","","","","","","","","","tax","refund","-IR_2","account_template_201010901",""
"tax_template_out_cprb_incl_service","False","","CPRB Incl.","CPRB Incl.","1","sale","True","","tax_group_cprb_incl_services","base","invoice","","","CPRB Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_cprb_excl_service","False","","CPRB Excl.","CPRB Excl.","1","sale","False","","tax_group_cprb_excl_services","base","invoice","","","CPRB Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_cprb_rf_incl_service","False","","CPRB RF Incl.","CPRB RF Incl.","1","sale","True","","tax_group_cprb_rf_incl_services","base","invoice","","","CPRB RF Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_cprb_rf_excl_service","False","","CPRB RF Excl.","CPRB RF Excl.","1","sale","False","","tax_group_cprb_rf_excl_services","base","invoice","","","CPRB RF Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_inss_ar_incl_service","False","","INSS AR Incl.","INSS AR Incl.","1","sale","True","","tax_group_inss_ar_incl_services","base","invoice","+INSS_1","","INSS AR Incl."
"","","","","","","","","","","tax","invoice","+INSS_2","account_template_201010103",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","-INSS_2","account_template_201010103",""
"tax_template_out_inss_ar_excl_service","False","","INSS AR Excl.","INSS AR Excl.","1","sale","False","","tax_group_inss_ar_excl_services","base","invoice","+INSS_1","","INSS AR Excl."
"","","","","","","","","","","tax","invoice","+INSS_2","account_template_201010103",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","-INSS_2","account_template_201010103",""
"tax_template_out_inss_rf_incl_service","False","","INSS RF Incl.","INSS RF Incl.","1","sale","True","","tax_group_inss_rf_incl_services","base","invoice","+INSS_1","","INSS RF Incl."
"","","","","","","","","","","tax","invoice","+INSS_2","account_template_201010103",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","-INSS_2","account_template_201010103",""
"tax_template_out_inss_rf_excl_service","False","","INSS RF Excl.","INSS RF Excl.","1","sale","False","","tax_group_inss_rf_excl_services","base","invoice","+INSS_1","","INSS RF Excl."
"","","","","","","","","","","tax","invoice","+INSS_2","account_template_201010103",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","-INSS_2","account_template_201010103",""

```

## File: data\template\account.tax.group-br.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@pt"
"tax_group_icms_0","ICMS 0%","base.br","account_template_202011003","account_template_102010802","ICMS 0%"
"tax_group_icms_7","ICMS 7%","base.br","account_template_202011003","account_template_102010802","ICMS 7%"
"tax_group_icms_12","ICMS 12%","base.br","account_template_202011003","account_template_102010802","ICMS 12%"
"tax_group_icms_17","ICMS 17%","base.br","account_template_202011003","account_template_102010802","ICMS 17%"
"tax_group_irpj_0","IRPJ 0%","base.br","account_template_202011003","account_template_102010802","IRPJ 0%"
"tax_group_pis_0","PIS 0%","base.br","account_template_202011003","account_template_102010802","PIS 0%"
"tax_group_pis_065","PIS 0.65%","base.br","account_template_202011003","account_template_102010802","PIS 0.65%"
"tax_group_cofins_0","COFINS 0%","base.br","account_template_202011003","account_template_102010802","COFINS 0%"
"tax_group_cofins_3","COFINS 3%","base.br","account_template_202011003","account_template_102010802","COFINS 3%"
"tax_group_ir_0","IR 0%","base.br","account_template_202011003","account_template_102010802","IR 0%"
"tax_group_issqn_2","ISSQN 2%","base.br","account_template_202011003","account_template_102010802","ISSQN 2%"
"tax_group_csll_0","CSLL 0%","base.br","account_template_202011003","account_template_102010802","CSLL 0%"
"tax_group_ipi_0","IPI 0%","base.br","account_template_202011003","account_template_102010802","IPI 0%"
"tax_group_ipi_10","IPI 10%","base.br","account_template_202011003","account_template_102010802","IPI 10%"
"tax_group_ii_0","II","base.br","account_template_202011003","account_template_102010802","II"
"tax_group_inss_0","INSS","base.br","account_template_202011003","account_template_102010802","INSS"
"tax_group_aproxtrib_fed_incl_goods","Tributação Federal Aproximada Incl.","base.br","account_template_202011003","account_template_102010802","Tributação Federal Aproximada Incl."
"tax_group_aproxtrib_fed_excl_goods","Tributação Federal Aproximada Excl.","base.br","account_template_202011003","account_template_102010802","Tributação Federal Aproximada Excl."
"tax_group_aproxtrib_state_incl_goods","Tributação Estadual Aproximada Incl.","base.br","account_template_202011003","account_template_102010802","Tributação Estadual Aproximada Incl."
"tax_group_aproxtrib_state_excl_goods","Tributação Estadual Aproximada Excl.","base.br","account_template_202011003","account_template_102010802","Tributação Estadual Aproximada Excl."
"tax_group_cofins_incl_goods","COFINS Incl.","base.br","account_template_202011003","account_template_102010802","COFINS Incl."
"tax_group_cofins_excl_goods","COFINS Excl.","base.br","account_template_202011003","account_template_102010802","COFINS Excl."
"tax_group_cofins_deson_incl_goods","COFINS Desoneração Incl.","base.br","account_template_202011003","account_template_102010802","COFINS Desoneração Incl."
"tax_group_cofins_deson_excl_goods","COFINS Desoneração Excl.","base.br","account_template_202011003","account_template_102010802","COFINS Desoneração Excl."
"tax_group_cofins_st_incl_goods","COFINS ST Incl.","base.br","account_template_202011003","account_template_102010802","COFINS ST Incl."
"tax_group_cofins_st_excl_goods","COFINS ST Excl.","base.br","account_template_202011003","account_template_102010802","COFINS ST Excl."
"tax_group_icms_incl_goods","ICMS Incl.","base.br","account_template_202011003","account_template_102010802","ICMS Incl."
"tax_group_icms_excl_goods","ICMS Excl.","base.br","account_template_202011003","account_template_102010802","ICMS Excl."
"tax_group_icms_credsn_incl_goods","ICMS CredSN Incl.","base.br","account_template_202011003","account_template_102010802","ICMS CredSN Incl."
"tax_group_icms_credsn_excl_goods","ICMS CredSN Excl.","base.br","account_template_202011003","account_template_102010802","ICMS CredSN Excl."
"tax_group_icms_deson_incl_goods","ICMS Desoneração Incl.","base.br","account_template_202011003","account_template_102010802","ICMS Desoneração Incl."
"tax_group_icms_deson_excl_goods","ICMS Desoneração Excl.","base.br","account_template_202011003","account_template_102010802","ICMS Desoneração Excl."
"tax_group_icms_difa_dest_incl_goods","ICMS DIFA Destinatário Incl.","base.br","account_template_202011003","account_template_102010802","ICMS DIFA Destinatário Incl."
"tax_group_icms_difa_dest_excl_goods","ICMS DIFA Destinatário Excl.","base.br","account_template_202011003","account_template_102010802","ICMS DIFA Destinatário Excl."
"tax_group_icms_difa_fcp_incl_goods","ICMS DIFA FCP Incl.","base.br","account_template_202011003","account_template_102010802","ICMS DIFA FCP Incl."
"tax_group_icms_difa_fcp_excl_goods","ICMS DIFA FCP Excl.","base.br","account_template_202011003","account_template_102010802","ICMS DIFA FCP Excl."
"tax_group_icms_difa_remet_incl_goods","ICMS DIFA Remetente Incl.","base.br","account_template_202011003","account_template_102010802","ICMS DIFA Remetente Incl."
"tax_group_icms_difa_remet_excl_goods","ICMS DIFA Remetente Excl.","base.br","account_template_202011003","account_template_102010802","ICMS DIFA Remetente Excl."
"tax_group_icms_eff_incl_goods","ICMS EFF Incl.","base.br","account_template_202011003","account_template_102010802","ICMS EFF Incl."
"tax_group_icms_eff_excl_goods","ICMS EFF Excl.","base.br","account_template_202011003","account_template_102010802","ICMS EFF Excl."
"tax_group_icms_fcp_incl_goods","ICMS FCP Incl.","base.br","account_template_202011003","account_template_102010802","ICMS FCP Incl."
"tax_group_icms_fcp_excl_goods","ICMS FCP Excl.","base.br","account_template_202011003","account_template_102010802","ICMS FCP Excl."
"tax_group_icms_own_payer_incl_goods","ICMS Próprio Emitente Incl.","base.br","account_template_202011003","account_template_102010802","ICMS Próprio Emitente Incl."
"tax_group_icms_own_payer_excl_goods","ICMS Próprio Emitente Excl.","base.br","account_template_202011003","account_template_102010802","ICMS Próprio Emitente Excl."
"tax_group_icms_part_incl_goods","ICMS Partilha Incl.","base.br","account_template_202011003","account_template_102010802","ICMS Partilha Incl."
"tax_group_icms_part_excl_goods","ICMS Partilha Excl.","base.br","account_template_202011003","account_template_102010802","ICMS Partilha Excl."
"tax_group_icms_rf_incl_goods","ICMS RF Incl.","base.br","account_template_202011003","account_template_102010802","ICMS RF Incl."
"tax_group_icms_rf_excl_goods","ICMS RF Excl.","base.br","account_template_202011003","account_template_102010802","ICMS RF Excl."
"tax_group_icms_st_incl_goods","ICMS ST Incl.","base.br","account_template_202011003","account_template_102010802","ICMS ST Incl."
"tax_group_icms_st_excl_goods","ICMS ST Excl.","base.br","account_template_202011003","account_template_102010802","ICMS ST Excl."
"tax_group_icms_st_fcp_incl_goods","ICMS ST FCP Incl.","base.br","account_template_202011003","account_template_102010802","ICMS ST FCP Incl."
"tax_group_icms_st_fcp_excl_goods","ICMS ST FCP Excl.","base.br","account_template_202011003","account_template_102010802","ICMS ST FCP Excl."
"tax_group_icms_st_fcppart_incl_goods","ICMS ST FCP Partilha Incl.","base.br","account_template_202011003","account_template_102010802","ICMS ST FCP Partilha Incl."
"tax_group_icms_st_fcppart_excl_goods","ICMS ST FCP Partilha Excl.","base.br","account_template_202011003","account_template_102010802","ICMS ST FCP Partilha Excl."
"tax_group_icms_st_part_incl_goods","ICMS ST Partilha Incl.","base.br","account_template_202011003","account_template_102010802","ICMS ST Partilha Incl."
"tax_group_icms_st_part_excl_goods","ICMS ST Partilha Excl.","base.br","account_template_202011003","account_template_102010802","ICMS ST Partilha Excl."
"tax_group_icms_st_sd_incl_goods","ICMS ST SD Incl.","base.br","account_template_202011003","account_template_102010802","ICMS ST SD Incl."
"tax_group_icms_st_sd_excl_goods","ICMS ST SD Excl.","base.br","account_template_202011003","account_template_102010802","ICMS ST SD Excl."
"tax_group_icms_st_sd_fcp_incl_goods","ICMS ST SD FCP Incl.","base.br","account_template_202011003","account_template_102010802","ICMS ST SD FCP Incl."
"tax_group_icms_st_sd_fcp_excl_goods","ICMS ST SD FCP Excl.","base.br","account_template_202011003","account_template_102010802","ICMS ST SD FCP Excl."
"tax_group_ii_incl_goods","II - Imposto de Importação Incl.","base.br","account_template_202011003","account_template_102010802","II - Imposto de Importação Incl."
"tax_group_ii_excl_goods","II - Imposto de Importação Excl.","base.br","account_template_202011003","account_template_102010802","II - Imposto de Importação Excl."
"tax_group_iof_incl_goods","IOF - Imposto sobre Operações Financeiras Incl.","base.br","account_template_202011003","account_template_102010802","IOF - Imposto sobre Operações Financeiras Incl."
"tax_group_iof_excl_goods","IOF - Imposto sobre Operações Financeiras Excl.","base.br","account_template_202011003","account_template_102010802","IOF - Imposto sobre Operações Financeiras Excl."
"tax_group_ipi_incl_goods","IPI Incl.","base.br","account_template_202011003","account_template_102010802","IPI Incl."
"tax_group_ipi_excl_goods","IPI Excl.","base.br","account_template_202011003","account_template_102010802","IPI Excl."
"tax_group_ipi_returned_incl_goods","IPI Retornado Incl.","base.br","account_template_202011003","account_template_102010802","IPI Retornado Incl."
"tax_group_ipi_returned_excl_goods","IPI Retornado Excl.","base.br","account_template_202011003","account_template_102010802","IPI Retornado Excl."
"tax_group_pis_incl_goods","PIS Incl.","base.br","account_template_202011003","account_template_102010802","PIS Incl."
"tax_group_pis_excl_goods","PIS Excl.","base.br","account_template_202011003","account_template_102010802","PIS Excl."
"tax_group_pis_deson_incl_goods","PIS Desoneração Incl.","base.br","account_template_202011003","account_template_102010802","PIS Desoneração Incl."
"tax_group_pis_deson_excl_goods","PIS Desoneração Excl.","base.br","account_template_202011003","account_template_102010802","PIS Desoneração Excl."
"tax_group_pis_st_incl_goods","PIS ST Incl.","base.br","account_template_202011003","account_template_102010802","PIS ST Incl."
"tax_group_pis_st_excl_goods","PIS ST Excl.","base.br","account_template_202011003","account_template_102010802","PIS ST Excl."
"tax_group_aproxtrib_city_incl_services","Approximate Municipal Taxation Incl.","base.br","account_template_202011003","account_template_102010802","Tributação Municipal Aproximada Incl."
"tax_group_aproxtrib_city_excl_services","Approximate Municipal Taxation Incl.","base.br","account_template_202011003","account_template_102010802","Tributação Municipal Aproximada Excl."
"tax_group_pis_rf_incl_services","PIS RF Incl.","base.br","account_template_202011003","account_template_102010802","PIS RF Incl."
"tax_group_pis_rf_excl_services","PIS RF Excl.","base.br","account_template_202011003","account_template_102010802","PIS RF Excl."
"tax_group_cofins_rf_incl_services","COFINS RF Incl.","base.br","account_template_202011003","account_template_102010802","COFINS RF Incl."
"tax_group_cofins_rf_excl_services","COFINS RF Excl.","base.br","account_template_202011003","account_template_102010802","COFINS RF Excl."
"tax_group_csll_incl_services","CSLL Incl.","base.br","account_template_202011003","account_template_102010802","CSLL Incl."
"tax_group_csll_excl_services","CSLL Excl.","base.br","account_template_202011003","account_template_102010802","CSLL Excl."
"tax_group_csll_rf_incl_services","CSLL RF Incl.","base.br","account_template_202011003","account_template_102010802","CSLL RF Incl."
"tax_group_csll_rf_excl_services","CSLL RF Excl.","base.br","account_template_202011003","account_template_102010802","CSLL RF Excl."
"tax_group_iss_incl_services","ISS Incl.","base.br","account_template_202011003","account_template_102010802","ISS Incl."
"tax_group_iss_excl_services","ISS Excl.","base.br","account_template_202011003","account_template_102010802","ISS Excl."
"tax_group_iss_rf_incl_services","ISS RF Incl.","base.br","account_template_202011003","account_template_102010802","ISS RF Incl."
"tax_group_iss_rf_excl_services","ISS RF Excl.","base.br","account_template_202011003","account_template_102010802","ISS RF Excl."
"tax_group_ir_pj_incl_services","IR PJ Incl.","base.br","account_template_202011003","account_template_102010802","IR PJ Incl."
"tax_group_ir_pj_excl_services","IR PJ Excl.","base.br","account_template_202011003","account_template_102010802","IR PJ Excl."
"tax_group_ir_rf_incl_services","IR RF Incl.","base.br","account_template_202011003","account_template_102010802","IR RF Incl."
"tax_group_ir_rf_excl_services","IR RF Excl.","base.br","account_template_202011003","account_template_102010802","IR RF Excl."
"tax_group_cprb_incl_services","CPRB Incl.","base.br","account_template_202011003","account_template_102010802","CPRB Incl."
"tax_group_cprb_excl_services","CPRB Excl.","base.br","account_template_202011003","account_template_102010802","CPRB Excl."
"tax_group_cprb_rf_incl_services","CPRB RF Incl.","base.br","account_template_202011003","account_template_102010802","CPRB RF Incl."
"tax_group_cprb_rf_excl_services","CPRB RF Excl.","base.br","account_template_202011003","account_template_102010802","CPRB RF Excl."
"tax_group_inss_ar_incl_services","INSS AR Incl.","base.br","account_template_202011003","account_template_102010802","INSS AR Incl."
"tax_group_inss_ar_excl_services","INSS AR Excl.","base.br","account_template_202011003","account_template_102010802","INSS AR Excl."
"tax_group_inss_rf_incl_services","INSS RF Incl.","base.br","account_template_202011003","account_template_102010802","INSS RF Incl."
"tax_group_inss_rf_excl_services","INSS RF Excl.","base.br","account_template_202011003","account_template_102010802","INSS RF Excl."

```

## File: models\account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountTax(models.Model):
    """ Add fields used to define some brazilian taxes """
    _inherit = 'account.tax'

    tax_discount = fields.Boolean(string='Discount this Tax in Price',
                                  help="Mark it for (ICMS, PIS e etc.).")
    base_reduction = fields.Float(string='Redution', digits=0, required=True,
                                  help="Um percentual decimal em % entre 0-1.", default=0)
    amount_mva = fields.Float(string='MVA Percent', digits=0, required=True,
                              help="Um percentual decimal em % entre 0-1.", default=0)

```

## File: models\account_fiscal_position.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, fields, models

SOUTH_SOUTHEAST = {"PR", "RS", "SC", "SP", "ES", "MG", "RJ"}
NORTH_NORTHEAST_MIDWEST = {
    "AC", "AP", "AM", "PA", "RO", "RR", "TO", "AL", "BA", "CE",
    "MA", "PB", "PE", "PI", "RN", "SE", "DF", "GO", "MT", "MS"
}


class AccountFiscalPosition(models.Model):
    _inherit = 'account.fiscal.position'

    l10n_br_fp_type = fields.Selection(
        selection=[
            ('internal', 'Internal'),
            ('ss_nnm', 'South/Southeast selling to North/Northeast/Midwest'),
            ('interstate', 'Other interstate'),
        ],
        string='Interstate Fiscal Position Type',
    )

    @api.model
    def _get_fiscal_position(self, partner, delivery=None):
        if not delivery:
            delivery = partner

        if self.env.company.country_id.code != "BR" or delivery.country_id.code != 'BR':
            return super()._get_fiscal_position(partner, delivery=delivery)

        # manually set fiscal position on partner has a higher priority
        manual_fiscal_position = delivery.property_account_position_id or partner.property_account_position_id
        if manual_fiscal_position:
            return manual_fiscal_position

        # Taxation in Brazil depends on both the state of the partner and the state of the company
        if self.env.company.state_id == delivery.state_id:
            return self.search([('l10n_br_fp_type', '=', 'internal'), ('company_id', '=', self.env.company.id)], limit=1)
        if self.env.company.state_id.code in SOUTH_SOUTHEAST and delivery.state_id.code in NORTH_NORTHEAST_MIDWEST:
            return self.search([('l10n_br_fp_type', '=', 'ss_nnm'), ('company_id', '=', self.env.company.id)], limit=1)
        return self.search([('l10n_br_fp_type', '=', 'interstate'), ('company_id', '=', self.env.company.id)], limit=1)

```

## File: models\account_journal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    l10n_br_invoice_serial = fields.Char(
        'Series', copy=False,
        help='Brazil: Series number associated with this Journal. If more than one Series needs to be used, duplicate this Journal and assign the new Series to the duplicated Journal.'
    )

    @api.depends('l10n_br_invoice_serial')
    def _compute_display_name(self):
        res = super()._compute_display_name()
        for journal in self.filtered('l10n_br_invoice_serial'):
            journal.display_name = f'{journal.l10n_br_invoice_serial}-{journal.display_name}'

        return res

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountMove(models.Model):
    _inherit = "account.move"

    def _compute_l10n_latam_document_type(self):
        """ Override for debit notes. This sets the same document type as the one on the origin. Cannot
         override the defaults in the account.move.debit wizard because l10n_latam_invoice_document explicitly
         calls _compute_l10n_latam_document_type() after the debit note is created. """
        br_debit_notes = self.filtered(lambda m: m.state == "draft" and m.country_code == "BR" and m.debit_origin_id.l10n_latam_document_type_id)
        for move in br_debit_notes:
            move.l10n_latam_document_type_id = move.debit_origin_id.l10n_latam_document_type_id

        return super(AccountMove, self - br_debit_notes)._compute_l10n_latam_document_type()

    def _get_last_sequence_domain(self, relaxed=False):
        """ Override to give sequence names in the same journal their own, independent numbering. """
        where_string, param = super()._get_last_sequence_domain(relaxed)
        if self.country_code == "BR" and self.l10n_latam_use_documents:
            where_string += " AND l10n_latam_document_type_id = %(l10n_latam_document_type_id)s "
            param["l10n_latam_document_type_id"] = self.l10n_latam_document_type_id.id or 0
        return where_string, param

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = "res.company"

    # ==== Business fields ====
    l10n_br_cpf_code = fields.Char(string="CPF", help="Natural Persons Register.")
    l10n_br_ie_code = fields.Char(string="IE", help="State Tax Identification Number. Should contain 9-14 digits.") # each state has its own format. Not all of the validation rules can be easily found.
    l10n_br_im_code = fields.Char(string="IM", help="Municipal Tax Identification Number") # each municipality has its own format. There is no information about validation anywhere.
    l10n_br_nire_code = fields.Char(string="NIRE", help="State Commercial Identification Number. Should contain 11 digits.")

    def _localization_use_documents(self):
        self.ensure_one()
        return self.account_fiscal_country_id.code == "BR" or super()._localization_use_documents()

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_br_ie_code = fields.Char(string="IE", help="State Tax Identification Number. Should contain 9-14 digits.")
    l10n_br_im_code = fields.Char(string="IM", help="Municipal Tax Identification Number")
    l10n_br_isuf_code = fields.Char(string="SUFRAMA code", help="SUFRAMA registration number.")

```

## File: models\template_br.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('br')
    def _get_br_template_data(self):
        return {
            'code_digits': '6',
            'property_account_receivable_id': 'account_template_101010401',
            'property_account_payable_id': 'account_template_201010301',
            'property_account_expense_categ_id': 'account_template_30101030101',
            'property_account_income_categ_id': 'account_template_30101010105',
        }

    @template('br', 'res.company')
    def _get_br_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.br',
                'bank_account_code_prefix': '1.01.01.02.00',
                'cash_account_code_prefix': '1.01.01.01.00',
                'transfer_account_code_prefix': '1.01.01.12.00',
                'account_default_pos_receivable_account_id': 'account_template_101010402',
                'income_currency_exchange_account_id': 'br_3_01_01_05_01_47',
                'expense_currency_exchange_account_id': 'br_3_11_01_09_01_40',
                'account_journal_early_pay_discount_loss_account_id': 'account_template_31101010202',
                'account_journal_early_pay_discount_gain_account_id': 'account_template_30101050148',
                'account_sale_tax_id': 'tax_template_out_icms_interno17',
                'account_purchase_tax_id': 'tax_template_in_icms_interno17',
            },
        }

    @template('br', 'account.journal')
    def _get_br_account_journal(self):
        return {
            'sale': {
                'l10n_br_invoice_serial': '1',
                'refund_sequence': False,
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_br
from . import account
from . import account_journal
from . import account_move
from . import account_fiscal_position
from . import res_partner
from . import res_company

```

## File: views\account_fiscal_position_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_account_position_form" model="ir.ui.view">
        <field name="name">account.fiscal.position.form</field>
        <field name="model">account.fiscal.position</field>
        <field name="inherit_id" ref="account.view_account_position_form"/>
        <field name="arch" type="xml">
            <field name="auto_apply" position="after">
                <field name="l10n_br_fp_type" options="{'no_open': True, 'no_create': True}" invisible="country_id != %(base.br)d"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\account_journal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_journal_form" model="ir.ui.view">
            <field name="model">account.journal</field>
            <field name="name">account.journal.form</field>
            <field name="inherit_id" ref="l10n_latam_invoice_document.view_account_journal_form"/>
            <field name="arch" type="xml">
                <field name="type" position="after">
                    <field name="l10n_br_invoice_serial" invisible="not l10n_latam_use_documents or country_code != 'BR'"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\account_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
		<record model="ir.ui.view" id="view_l10n_br_account_tax_form">
			<field name="name">l10n_br_account.tax.form</field>
			<field name="model">account.tax</field>
			<field name="inherit_id" ref="account.view_tax_form"/>
			<field name="arch" type="xml">
				<field position="after" name="price_include">
					<field name="tax_discount" invisible="country_code != 'BR'"/>
			    </field>
			    <field position="after" name="tax_discount">
					<field name="base_reduction" widget="monetary" invisible="country_code != 'BR'"/>
					<field name="amount_mva" widget="monetary" invisible="country_code != 'BR'"/>
			    </field>
			</field>
		</record>
</odoo>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_company_form" model="ir.ui.view">
        <field name="name">res.company.form</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_br_cpf_code" invisible="country_id != %(base.br)d"/>
                <field name="l10n_br_ie_code" invisible="country_id != %(base.br)d"/>
                <field name="l10n_br_im_code" invisible="country_id != %(base.br)d"/>
                <field name="l10n_br_nire_code" invisible="country_id != %(base.br)d"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="br_partner_tax_fields_form" model="ir.ui.view">
            <field name="name">res.partner.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="account.view_partner_property_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vat']" position="after">
                    <field name="l10n_br_ie_code" invisible="'BR' not in fiscal_country_codes"/>
                    <field name="l10n_br_im_code" invisible="'BR' not in fiscal_country_codes"/>
                    <field name="l10n_br_isuf_code" invisible="'BR' not in fiscal_country_codes"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\account_move_reversal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class AccountMoveReversal(models.TransientModel):
    _inherit = "account.move.reversal"

    def _compute_document_type(self):
        """ If a l10n_latam_document_type_id was set, change it in the case of Brazil to be
        the same as the move that is being reversed.
        """
        res = super()._compute_document_type()
        for reversal in self.filtered("l10n_latam_document_type_id"):
            # LATAM invoices are guaranteed to be just one by _compute_documents_info().
            move = reversal.move_ids[0]
            if move.country_code == "BR":
                reversal.l10n_latam_document_type_id = move.l10n_latam_document_type_id

        return res

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move_reversal

```

