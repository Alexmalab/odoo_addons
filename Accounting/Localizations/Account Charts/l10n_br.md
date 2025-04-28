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
    'version': '1.0',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/brazil.html',
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
        'account_qr_code_emv',
        'base_address_extended',
        'l10n_latam_base',
        'l10n_latam_invoice_document',
    ],
    'auto_install': ['account'],
    'data': [
        'security/ir.model.access.csv',
        'views/res_partner_views.xml',
        'data/account_tax_report_data.xml',
        'data/res_country_data.xml',
        'data/res.city.csv',
        'data/l10n_br.zip.range.csv',
        'data/l10n_latam.identification.type.csv',
        'data/l10n_latam.document.type.csv',
        'views/account_view.xml',
        'views/account_fiscal_position_views.xml',
        'views/ir_ui_menu_brazil.xml',
        'views/res_company_views.xml',
        'views/account_journal_views.xml',
        'views/res_bank_views.xml',
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

## File: data\l10n_br.zip.range.csv

```csv
id,city_id:id,start,end
zip_range_0001,city_br_320,75345-000,75349-999
zip_range_0002,city_br_321,38540-000,38549-999
zip_range_0003,city_br_322,72940-000,72959-999
zip_range_0004,city_br_323,35620-000,35620-999
zip_range_0005,city_br_190,68440-000,68444-999
zip_range_0006,city_br_324,63240-000,63249-999
zip_range_0007,city_br_325,46690-000,46699-999
zip_range_0008,city_br_326,48680-000,48699-999
zip_range_0009,city_br_327,86460-000,86464-999
zip_range_0010,city_br_328,89636-000,89637-999
zip_range_0011,city_br_329,68527-000,68529-999
zip_range_0012,city_br_330,89830-000,89831-999
zip_range_0013,city_br_331,35365-000,35366-999
zip_range_0014,city_br_332,53500-001,53599-999
zip_range_0015,city_br_333,77693-000,77694-999
zip_range_0016,city_br_334,35438-000,35438-999
zip_range_0017,city_br_292,65930-000,65934-999
zip_range_0018,city_br_335,48360-000,48369-999
zip_range_0019,city_br_336,68690-000,68694-999
zip_range_0020,city_br_337,62785-000,62789-999
zip_range_0021,city_br_338,62580-000,62589-999
zip_range_0022,city_br_339,59370-000,59373-999
zip_range_0023,city_br_340,64748-000,64749-999
zip_range_0024,city_br_341,96445-000,96449-999
zip_range_0025,city_br_342,63560-000,63569-999
zip_range_0026,city_br_343,78480-000,78489-999
zip_range_0027,city_br_344,69945-000,69949-999
zip_range_0028,city_br_345,75960-000,75969-999
zip_range_0029,city_br_346,59650-000,59654-999
zip_range_0030,city_br_347,35147-000,35147-999
zip_range_0031,city_br_348,17800-000,17809-999
zip_range_0032,city_br_349,76155-000,76159-999
zip_range_0033,city_br_350,15230-000,15239-999
zip_range_0034,city_br_351,83490-000,83499-999
zip_range_0035,city_br_352,48435-000,48439-999
zip_range_0036,city_br_353,56800-000,56819-999
zip_range_0037,city_br_354,59510-000,59512-999
zip_range_0038,city_br_355,29600-000,29614-999
zip_range_0039,city_br_356,65505-000,65509-999
zip_range_0040,city_br_357,56360-000,56379-999
zip_range_0041,city_br_358,68890-000,68899-999
zip_range_0042,city_br_359,55495-000,55499-999
zip_range_0043,city_br_360,64440-000,64444-999
zip_range_0044,city_br_361,88420-000,88429-999
zip_range_0045,city_br_362,89188-000,89189-999
zip_range_0046,city_br_363,68533-000,68534-999
zip_range_0047,city_br_364,39790-000,39794-999
zip_range_0048,city_br_365,78635-000,78637-999
zip_range_0049,city_br_366,57490-000,57499-999
zip_range_0050,city_br_367,58748-000,58749-999
zip_range_0051,city_br_368,64460-000,64464-999
zip_range_0052,city_br_369,79680-000,79689-999
zip_range_0053,city_br_370,38110-000,38119-999
zip_range_0054,city_br_371,89654-000,89659-999
zip_range_0055,city_br_372,65578-000,65579-999
zip_range_0056,city_br_373,29820-000,29829-999
zip_range_0057,city_br_374,48170-000,48179-999
zip_range_0058,city_br_375,73780-000,73789-999
zip_range_0059,city_br_376,75665-000,75669-999
zip_range_0060,city_br_377,59995-000,59999-999
zip_range_0061,city_br_378,55550-000,55554-999
zip_range_0062,city_br_379,99965-000,99969-999
zip_range_0063,city_br_380,13860-000,13869-999
zip_range_0064,city_br_381,37273-000,37274-999
zip_range_0065,city_br_382,55340-000,55344-999
zip_range_0066,city_br_383,13890-000,13899-999
zip_range_0067,city_br_384,89883-000,89884-999
zip_range_0068,city_br_385,13940-000,13949-999
zip_range_0069,city_br_386,18770-000,18774-999
zip_range_0070,city_br_387,13525-000,13529-999
zip_range_0071,city_br_388,39880-000,39884-999
zip_range_0072,city_br_389,89843-000,89844-999
zip_range_0073,city_br_137,72910-001,72929-999
zip_range_0074,city_br_390,88150-000,88159-999
zip_range_0075,city_br_391,39990-000,39994-999
zip_range_0076,city_br_392,96540-000,96544-999
zip_range_0077,city_br_393,17120-000,17149-999
zip_range_0078,city_br_394,83850-000,83859-999
zip_range_0079,city_br_395,29795-000,29799-999
zip_range_0080,city_br_396,58778-000,58779-999
zip_range_0081,city_br_397,77908-000,77909-999
zip_range_0082,city_br_398,35200-000,35219-999
zip_range_0083,city_br_399,45220-000,45224-999
zip_range_0084,city_br_400,63575-000,63579-999
zip_range_0085,city_br_401,37450-000,37451-999
zip_range_0086,city_br_402,98750-000,98757-999
zip_range_0087,city_br_403,37458-000,37459-999
zip_range_0088,city_br_404,58388-000,58389-999
zip_range_0089,city_br_405,58125-000,58127-999
zip_range_0090,city_br_406,58390-000,58392-999
zip_range_0091,city_br_407,55260-000,55269-999
zip_range_0092,city_br_408,64655-000,64659-999
zip_range_0093,city_br_199,48000-001,48107-999
zip_range_0094,city_br_409,18220-000,18224-999
zip_range_0095,city_br_410,37596-000,37599-999
zip_range_0096,city_br_411,65250-000,65254-999
zip_range_0097,city_br_412,62120-000,62129-999
zip_range_0098,city_br_413,58460-000,58462-999
zip_range_0099,city_br_414,79530-000,79539-999
zip_range_0100,city_br_415,45910-000,45919-999
zip_range_0101,city_br_416,65610-000,65614-999
zip_range_0102,city_br_417,98950-000,98954-999
zip_range_0103,city_br_418,29500-000,29539-999
zip_range_0104,city_br_419,97540-001,97559-999
zip_range_0105,city_br_420,64675-000,64677-999
zip_range_0106,city_br_421,98905-000,98909-999
zip_range_0107,city_br_422,36660-000,36669-999
zip_range_0108,city_br_423,68200-000,68209-999
zip_range_0109,city_br_424,59965-000,59969-999
zip_range_0110,city_br_425,72930-000,72939-999
zip_range_0111,city_br_426,37130-001,37139-999
zip_range_0112,city_br_427,29240-000,29254-999
zip_range_0113,city_br_428,19180-000,19189-999
zip_range_0114,city_br_429,36272-000,36274-999
zip_range_0115,city_br_430,88450-000,88459-999
zip_range_0116,city_br_431,58399-000,58399-999
zip_range_0117,city_br_432,58320-000,58321-999
zip_range_0118,city_br_433,55890-000,55899-999
zip_range_0119,city_br_434,77455-000,77457-999
zip_range_0120,city_br_435,45640-000,45640-999
zip_range_0121,city_br_436,77310-000,77314-999
zip_range_0122,city_br_437,68230-000,68249-999
zip_range_0123,city_br_438,39900-000,39911-999
zip_range_0124,city_br_439,59760-000,59769-999
zip_range_0125,city_br_253,83500-001,83534-999
zip_range_0126,city_br_440,99523-000,99524-999
zip_range_0127,city_br_441,75615-000,75619-999
zip_range_0128,city_br_442,35138-000,35139-999
zip_range_0129,city_br_443,98480-000,98499-999
zip_range_0130,city_br_444,37940-000,37944-999
zip_range_0131,city_br_445,78580-000,78586-999
zip_range_0132,city_br_446,76954-000,76955-999
zip_range_0133,city_br_447,15430-000,15439-999
zip_range_0134,city_br_241,68370-001,68379-999
zip_range_0135,city_br_448,65310-000,65314-999
zip_range_0136,city_br_449,85280-000,85299-999
zip_range_0137,city_br_450,63195-000,63199-999
zip_range_0138,city_br_451,37145-000,37147-999
zip_range_0139,city_br_452,55490-000,55494-999
zip_range_0140,city_br_453,14350-000,14389-999
zip_range_0141,city_br_454,69350-000,69354-999
zip_range_0142,city_br_455,99430-000,99434-999
zip_range_0143,city_br_456,16310-000,16339-999
zip_range_0144,city_br_457,65413-000,65414-999
zip_range_0145,city_br_458,65398-000,65399-999
zip_range_0146,city_br_459,76952-000,76953-999
zip_range_0147,city_br_460,78780-000,78784-999
zip_range_0148,city_br_461,89730-000,89734-999
zip_range_0149,city_br_462,78665-000,78667-999
zip_range_0150,city_br_463,36979-000,36979-999
zip_range_0151,city_br_464,59507-000,59507-999
zip_range_0152,city_br_465,95773-000,95774-999
zip_range_0153,city_br_466,78770-000,78772-999
zip_range_0154,city_br_467,76560-000,76579-999
zip_range_0155,city_br_468,36976-000,36978-999
zip_range_0156,city_br_469,64360-000,64364-999
zip_range_0157,city_br_470,78410-000,78414-999
zip_range_0158,city_br_471,87528-000,87529-999
zip_range_0159,city_br_472,76862-000,76862-999
zip_range_0160,city_br_473,73770-000,73779-999
zip_range_0161,city_br_474,87750-000,87759-999
zip_range_0162,city_br_475,65810-000,65819-999
zip_range_0163,city_br_476,87580-000,87594-999
zip_range_0164,city_br_477,36260-000,36264-999
zip_range_0165,city_br_478,29760-000,29769-999
zip_range_0166,city_br_479,62970-000,62979-999
zip_range_0167,city_br_480,78785-000,78789-999
zip_range_0168,city_br_481,87550-000,87554-999
zip_range_0169,city_br_482,64290-000,64294-999
zip_range_0170,city_br_483,18125-000,18129-999
zip_range_0171,city_br_484,69540-000,69549-999
zip_range_0172,city_br_485,35249-000,35249-999
zip_range_0173,city_br_486,15540-000,15549-999
zip_range_0174,city_br_487,19160-000,19179-999
zip_range_0175,city_br_488,17410-000,17419-999
zip_range_0176,city_br_489,17430-000,17439-999
zip_range_0177,city_br_490,35950-000,35959-999
zip_range_0178,city_br_160,94800-001,94899-999
zip_range_0179,city_br_491,77480-000,77482-999
zip_range_0180,city_br_492,76930-000,76931-999
zip_range_0181,city_br_493,39140-000,39149-999
zip_range_0182,city_br_494,64923-000,64924-999
zip_range_0183,city_br_495,73950-000,73969-999
zip_range_0184,city_br_496,86150-000,86159-999
zip_range_0185,city_br_497,69343-000,69344-999
zip_range_0186,city_br_498,79990-000,79994-999
zip_range_0187,city_br_499,68950-000,68959-999
zip_range_0188,city_br_500,65293-000,65293-999
zip_range_0189,city_br_501,87850-000,87859-999
zip_range_0190,city_br_502,55515-000,55519-999
zip_range_0191,city_br_503,96635-000,96639-999
zip_range_0192,city_br_504,76493-000,76494-999
zip_range_0193,city_br_505,64400-000,64409-999
zip_range_0194,city_br_506,65923-000,65923-999
zip_range_0195,city_br_507,45300-000,45304-999
zip_range_0196,city_br_508,69620-000,69629-999
zip_range_0197,city_br_509,44230-000,44244-999
zip_range_0198,city_br_510,44910-000,44914-999
zip_range_0199,city_br_124,13465-001,13479-999
zip_range_0200,city_br_511,76165-000,76169-999
zip_range_0201,city_br_512,14820-000,14824-999
zip_range_0202,city_br_513,15550-000,15559-999
zip_range_0203,city_br_514,98465-000,98469-999
zip_range_0204,city_br_515,62540-000,62549-999
zip_range_0205,city_br_516,76140-000,76144-999
zip_range_0206,city_br_517,58548-000,58549-999
zip_range_0207,city_br_518,13900-001,13909-999
zip_range_0208,city_br_519,49920-000,49929-999
zip_range_0209,city_br_520,35444-000,35446-999
zip_range_0210,city_br_521,85640-000,85649-999
zip_range_0211,city_br_522,57660-000,57669-999
zip_range_0212,city_br_523,45180-000,45189-999
zip_range_0213,city_br_524,85425-000,85429-999
zip_range_0214,city_br_525,68810-000,68814-999
zip_range_0215,city_br_526,65490-000,65494-999
zip_range_0216,city_br_527,13550-000,13559-999
zip_range_0217,city_br_528,69445-000,69449-999
zip_range_0218,city_br_529,77890-000,77892-999
zip_range_0219,city_br_046,67000-001,67199-999
zip_range_0220,city_br_062,75000-001,75159-999
zip_range_0221,city_br_530,68365-000,68369-999
zip_range_0222,city_br_531,65525-000,65529-999
zip_range_0223,city_br_532,79210-000,79214-999
zip_range_0224,city_br_533,79770-000,79779-999
zip_range_0225,city_br_534,29230-000,29239-999
zip_range_0226,city_br_535,89970-000,89979-999
zip_range_0227,city_br_536,46830-000,46834-999
zip_range_0228,city_br_537,86380-000,86384-999
zip_range_0229,city_br_538,48990-000,48999-999
zip_range_0230,city_br_539,37795-000,37799-999
zip_range_0231,city_br_540,16900-001,16919-999
zip_range_0232,city_br_541,95310-000,95314-999
zip_range_0233,city_br_542,37300-000,37304-999
zip_range_0234,city_br_543,18240-000,18244-999
zip_range_0235,city_br_544,39685-000,39687-999
zip_range_0236,city_br_545,79785-000,79789-999
zip_range_0237,city_br_546,55430-000,55434-999
zip_range_0238,city_br_547,88460-000,88469-999
zip_range_0239,city_br_548,47960-000,47969-999
zip_range_0240,city_br_549,64410-000,64414-999
zip_range_0241,city_br_550,77905-000,77907-999
zip_range_0242,city_br_551,59515-000,59516-999
zip_range_0243,city_br_175,23900-001,23969-999
zip_range_0244,city_br_552,44670-000,44679-999
zip_range_0245,city_br_553,86755-000,86759-999
zip_range_0246,city_br_554,75770-000,75779-999
zip_range_0247,city_br_555,18620-000,18639-999
zip_range_0248,city_br_556,19580-000,19589-999
zip_range_0249,city_br_557,76170-000,76179-999
zip_range_0250,city_br_558,64780-000,64781-999
zip_range_0251,city_br_559,88590-000,88597-999
zip_range_0252,city_br_560,88475-000,88484-999
zip_range_0253,city_br_561,69440-000,69444-999
zip_range_0254,city_br_562,95980-000,95984-999
zip_range_0255,city_br_563,48420-000,48429-999
zip_range_0256,city_br_564,83370-000,83389-999
zip_range_0257,city_br_565,63570-000,63574-999
zip_range_0258,city_br_566,64855-000,64857-999
zip_range_0259,city_br_567,44180-000,44189-999
zip_range_0260,city_br_568,36220-000,36224-999
zip_range_0261,city_br_569,88180-000,88189-999
zip_range_0262,city_br_570,35177-000,35178-999
zip_range_0263,city_br_571,44780-000,44789-999
zip_range_0264,city_br_572,79910-000,79919-999
zip_range_0265,city_br_573,59870-000,59879-999
zip_range_0266,city_br_574,83980-000,83999-999
zip_range_0267,city_br_575,95250-000,95254-999
zip_range_0268,city_br_576,36850-000,36854-999
zip_range_0269,city_br_577,58823-000,58823-999
zip_range_0270,city_br_578,12570-000,12579-999
zip_range_0271,city_br_579,15735-000,15739-999
zip_range_0272,city_br_040,74900-001,74999-999
zip_range_0273,city_br_580,75827-000,75827-999
zip_range_0274,city_br_581,77620-000,77629-999
zip_range_0275,city_br_582,79570-000,79579-999
zip_range_0276,city_br_583,28495-000,28499-999
zip_range_0277,city_br_584,29450-000,29459-999
zip_range_0278,city_br_585,78595-000,78599-999
zip_range_0279,city_br_586,18320-000,18324-999
zip_range_0280,city_br_587,65275-000,65275-999
zip_range_0281,city_br_588,89135-000,89135-999
zip_range_0282,city_br_589,59700-000,59729-999
zip_range_0283,city_br_590,48350-000,48359-999
zip_range_0284,city_br_591,75825-000,75826-999
zip_range_0285,city_br_592,45355-000,45359-999
zip_range_0286,city_br_231,86800-001,86819-999
zip_range_0287,city_br_593,69265-000,69279-999
zip_range_0288,city_br_594,62630-000,62639-999
zip_range_0289,city_br_595,49790-000,49799-999
zip_range_0290,city_br_596,79200-000,79209-999
zip_range_0291,city_br_597,61700-000,61759-999
zip_range_0292,city_br_598,89740-000,89744-999
zip_range_0293,city_br_599,58270-000,58272-999
zip_range_0294,city_br_600,35777-000,35779-999
zip_range_0295,city_br_036,49000-001,49099-999
zip_range_0296,city_br_601,18147-000,18149-999
zip_range_0297,city_br_602,48108-000,48109-999
zip_range_0298,city_br_603,62800-000,62809-999
zip_range_0299,city_br_604,46130-000,46139-999
zip_range_0300,city_br_152,16000-001,16129-999
zip_range_0301,city_br_605,48760-000,48769-999
zip_range_0302,city_br_606,36255-000,36259-999
zip_range_0303,city_br_607,62750-000,62754-999
zip_range_0304,city_br_608,53690-000,53699-999
zip_range_0305,city_br_609,18190-000,18194-999
zip_range_0306,city_br_610,29190-001,29199-999
zip_range_0307,city_br_611,75410-000,75419-999
zip_range_0308,city_br_612,39600-000,39609-999
zip_range_0309,city_br_613,76240-000,76244-999
zip_range_0310,city_br_614,75360-000,75369-999
zip_range_0311,city_br_615,77845-000,77847-999
zip_range_0312,city_br_616,77690-000,77692-999
zip_range_0313,city_br_617,77475-000,77477-999
zip_range_0314,city_br_618,78685-000,78689-999
zip_range_0315,city_br_170,77800-001,77839-999
zip_range_0316,city_br_619,78615-000,78619-999
zip_range_0317,city_br_620,65368-000,65369-999
zip_range_0318,city_br_621,77855-000,77859-999
zip_range_0319,city_br_622,76720-000,76729-999
zip_range_0320,city_br_261,38440-001,38459-999
zip_range_0321,city_br_623,77950-000,77957-999
zip_range_0322,city_br_624,65570-000,65577-999
zip_range_0323,city_br_625,79930-000,79934-999
zip_range_0324,city_br_626,48130-000,48139-999
zip_range_0325,city_br_627,96178-000,96179-999
zip_range_0326,city_br_628,65945-000,65947-999
zip_range_0327,city_br_629,14550-000,14569-999
zip_range_0328,city_br_630,18710-000,18719-999
zip_range_0329,city_br_631,37360-000,37369-999
zip_range_0330,city_br_632,12870-000,12899-999
zip_range_0331,city_br_127,57300-001,57319-999
zip_range_0332,city_br_633,77780-000,77782-999
zip_range_0333,city_br_634,36594-000,36599-999
zip_range_0334,city_br_254,86700-001,86719-999
zip_range_0335,city_br_635,38465-000,38469-999
zip_range_0336,city_br_636,84990-000,84999-999
zip_range_0337,city_br_637,38860-000,38869-999
zip_range_0338,city_br_638,86884-000,86889-999
zip_range_0339,city_br_639,78260-000,78264-999
zip_range_0340,city_br_640,89245-000,89246-999
zip_range_0341,city_br_641,58396-000,58396-999
zip_range_0342,city_br_642,88900-001,88913-999
zip_range_0343,city_br_121,14800-001,14812-999
zip_range_0344,city_br_230,13600-001,13609-999
zip_range_0345,city_br_643,62210-000,62214-999
zip_range_0346,city_br_644,65480-000,65484-999
zip_range_0347,city_br_645,93880-000,93889-999
zip_range_0348,city_br_646,63170-000,63179-999
zip_range_0349,city_br_647,56280-000,56299-999
zip_range_0350,city_br_232,28970-000,28989-999
zip_range_0351,city_br_648,58233-000,58234-999
zip_range_0352,city_br_649,87260-000,87264-999
zip_range_0353,city_br_650,45695-000,45699-999
zip_range_0354,city_br_651,99770-000,99789-999
zip_range_0355,city_br_652,62762-000,62763-999
zip_range_0356,city_br_653,44490-000,44499-999
zip_range_0357,city_br_654,49220-000,49229-999
zip_range_0358,city_br_198,83700-001,83729-999
zip_range_0359,city_br_655,35603-000,35603-999
zip_range_0360,city_br_281,38180-001,38184-999
zip_range_0361,city_br_656,37820-000,37854-999
zip_range_0362,city_br_657,17630-000,17649-999
zip_range_0363,city_br_658,35588-000,35589-999
zip_range_0364,city_br_659,56500-001,56519-999
zip_range_0365,city_br_660,37140-000,37141-999
zip_range_0366,city_br_661,25845-000,25849-999
zip_range_0367,city_br_662,17160-000,17179-999
zip_range_0368,city_br_663,58397-000,58397-999
zip_range_0369,city_br_664,59655-000,59659-999
zip_range_0370,city_br_665,49580-000,49599-999
zip_range_0371,city_br_666,58732-000,58732-999
zip_range_0372,city_br_667,58140-000,58144-999
zip_range_0373,city_br_668,12820-000,12829-999
zip_range_0374,city_br_669,18670-000,18674-999
zip_range_0375,city_br_670,78420-000,78424-999
zip_range_0376,city_br_671,76235-000,76239-999
zip_range_0377,city_br_672,59170-000,59172-999
zip_range_0378,city_br_673,36710-000,36719-999
zip_range_0379,city_br_674,39678-000,39679-999
zip_range_0380,city_br_675,38680-000,38689-999
zip_range_0381,city_br_676,78325-000,78329-999
zip_range_0382,city_br_677,76870-001,76879-999
zip_range_0383,city_br_678,15960-000,15969-999
zip_range_0384,city_br_679,86880-000,86883-999
zip_range_0385,city_br_680,28950-000,28959-999
zip_range_0386,city_br_681,88740-000,88744-999
zip_range_0387,city_br_682,63670-000,63679-999
zip_range_0388,city_br_683,64310-000,64314-999
zip_range_0389,city_br_684,58489-000,58491-999
zip_range_0390,city_br_685,64612-000,64612-999
zip_range_0391,city_br_686,64480-000,64489-999
zip_range_0392,city_br_687,28930-000,28939-999
zip_range_0393,city_br_688,77330-000,77349-999
zip_range_0394,city_br_689,95940-000,95944-999
zip_range_0395,city_br_690,96155-000,96159-999
zip_range_0396,city_br_691,95585-000,95587-999
zip_range_0397,city_br_692,96950-000,96989-999
zip_range_0398,city_br_693,96740-000,96744-999
zip_range_0399,city_br_694,96330-000,96359-999
zip_range_0400,city_br_695,89590-000,89594-999
zip_range_0401,city_br_696,13160-000,13164-999
zip_range_0402,city_br_697,76710-000,76719-999
zip_range_0403,city_br_698,07400-001,07499-999
zip_range_0404,city_br_699,89778-000,89779-999
zip_range_0405,city_br_700,95995-000,95996-999
zip_range_0406,city_br_701,89138-000,89139-999
zip_range_0407,city_br_702,15763-000,15764-999
zip_range_0408,city_br_703,86220-000,86224-999
zip_range_0409,city_br_704,63140-000,63144-999
zip_range_0410,city_br_318,19800-001,19819-999
zip_range_0411,city_br_705,69935-000,69939-999
zip_range_0412,city_br_706,85935-000,85939-999
zip_range_0413,city_br_707,58685-000,58689-999
zip_range_0414,city_br_708,64333-000,64334-999
zip_range_0415,city_br_709,36780-000,36783-999
zip_range_0416,city_br_710,86730-000,86749-999
zip_range_0417,city_br_711,57690-000,57699-999
zip_range_0418,city_br_712,87630-000,87639-999
zip_range_0419,city_br_713,69650-000,69659-999
zip_range_0420,city_br_714,88410-000,88419-999
zip_range_0421,city_br_715,39850-000,39854-999
zip_range_0422,city_br_188,12940-001,12954-999
zip_range_0423,city_br_716,29490-000,29499-999
zip_range_0424,city_br_717,77960-000,77969-999
zip_range_0425,city_br_718,68610-000,68616-999
zip_range_0426,city_br_719,39219-000,39219-999
zip_range_0427,city_br_720,98740-000,98749-999
zip_range_0428,city_br_721,99835-000,99837-999
zip_range_0429,city_br_722,45675-000,45679-999
zip_range_0430,city_br_723,15350-000,15354-999
zip_range_0431,city_br_724,76120-000,76124-999
zip_range_0432,city_br_725,63360-000,63379-999
zip_range_0433,city_br_726,89186-000,89187-999
zip_range_0434,city_br_727,68658-000,68659-999
zip_range_0435,city_br_728,77325-000,77327-999
zip_range_0436,city_br_729,69240-000,69249-999
zip_range_0437,city_br_730,16680-000,16699-999
zip_range_0438,city_br_731,16360-000,16369-999
zip_range_0439,city_br_732,18700-001,18709-999
zip_range_0440,city_br_733,68150-000,68164-999
zip_range_0441,city_br_734,64965-000,64967-999
zip_range_0442,city_br_735,75395-000,75395-999
zip_range_0443,city_br_736,65148-000,65149-999
zip_range_0444,city_br_737,77930-000,77939-999
zip_range_0445,city_br_738,77870-000,77879-999
zip_range_0446,city_br_307,65700-000,65703-999
zip_range_0447,city_br_739,65143-000,65144-999
zip_range_0448,city_br_740,65270-000,65271-999
zip_range_0449,city_br_741,65233-000,65234-999
zip_range_0450,city_br_742,15115-000,15119-999
zip_range_0451,city_br_743,37443-000,37444-999
zip_range_0452,city_br_260,96400-001,96444-999
zip_range_0453,city_br_744,68475-000,68479-999
zip_range_0454,city_br_745,58295-000,58296-999
zip_range_0455,city_br_746,59194-000,59195-999
zip_range_0456,city_br_747,47830-000,47844-999
zip_range_0457,city_br_748,68465-000,68469-999
zip_range_0458,city_br_749,44620-000,44629-999
zip_range_0459,city_br_750,64868-000,64869-999
zip_range_0460,city_br_751,63320-000,63339-999
zip_range_0461,city_br_752,29730-000,29744-999
zip_range_0462,city_br_753,16640-000,16649-999
zip_range_0463,city_br_754,35732-000,35735-999
zip_range_0464,city_br_755,76250-000,76254-999
zip_range_0465,city_br_756,88914-000,88914-999
zip_range_0466,city_br_757,89247-000,89247-999
zip_range_0467,city_br_214,88330-001,88339-999
zip_range_0468,city_br_758,88955-000,88959-999
zip_range_0469,city_br_759,88380-000,88384-999
zip_range_0470,city_br_760,95599-000,95599-999
zip_range_0471,city_br_761,88828-000,88829-999
zip_range_0472,city_br_762,83650-000,83699-999
zip_range_0473,city_br_763,15140-000,15144-999
zip_range_0474,city_br_316,65800-000,65804-999
zip_range_0475,city_br_764,38900-000,38909-999
zip_range_0476,city_br_765,63960-000,63969-999
zip_range_0477,city_br_766,12850-000,12869-999
zip_range_0478,city_br_767,58220-000,58224-999
zip_range_0479,city_br_768,39917-000,39919-999
zip_range_0480,city_br_769,37740-000,37749-999
zip_range_0481,city_br_770,89905-000,89905-999
zip_range_0482,city_br_771,79430-000,79439-999
zip_range_0483,city_br_772,86360-000,86369-999
zip_range_0484,city_br_773,77783-000,77784-999
zip_range_0485,city_br_774,68388-000,68389-999
zip_range_0486,city_br_775,48405-000,48409-999
zip_range_0487,city_br_776,95730-000,95734-999
zip_range_0488,city_br_777,18490-000,18499-999
zip_range_0489,city_br_778,35970-000,35983-999
zip_range_0490,city_br_779,99740-000,99749-999
zip_range_0491,city_br_780,65660-000,65664-999
zip_range_0492,city_br_781,78190-000,78194-999
zip_range_0493,city_br_782,36870-000,36877-999
zip_range_0494,city_br_783,96735-000,96739-999
zip_range_0495,city_br_784,58188-000,58194-999
zip_range_0496,city_br_785,59695-000,59699-999
zip_range_0497,city_br_242,36200-001,36209-999
zip_range_0498,city_br_786,63180-000,63184-999
zip_range_0499,city_br_787,16350-000,16359-999
zip_range_0500,city_br_788,86960-000,86969-999
zip_range_0501,city_br_239,68445-000,68449-999
zip_range_0502,city_br_789,59410-000,59419-999
zip_range_0503,city_br_790,69700-000,69729-999
zip_range_0504,city_br_791,17250-000,17259-999
zip_range_0505,city_br_792,47100-000,47114-999
zip_range_0506,city_br_793,89909-000,89909-999
zip_range_0507,city_br_794,17340-000,17349-999
zip_range_0508,city_br_795,64528-000,64529-999
zip_range_0509,city_br_796,46650-000,46669-999
zip_range_0510,city_br_797,55690-000,55694-999
zip_range_0511,city_br_798,58170-000,58172-999
zip_range_0512,city_br_799,58458-000,58459-999
zip_range_0513,city_br_800,57925-000,57929-999
zip_range_0514,city_br_801,29800-000,29819-999
zip_range_0515,city_br_802,57180-000,57199-999
zip_range_0516,city_br_803,58483-000,58484-999
zip_range_0517,city_br_804,78390-000,78397-999
zip_range_0518,city_br_805,18325-000,18329-999
zip_range_0519,city_br_806,45120-000,45129-999
zip_range_0520,city_br_807,65950-000,65961-999
zip_range_0521,city_br_808,78600-000,78609-999
zip_range_0522,city_br_809,98530-000,98534-999
zip_range_0523,city_br_810,86385-000,86389-999
zip_range_0524,city_br_811,44990-000,44999-999
zip_range_0525,city_br_812,77765-000,77769-999
zip_range_0526,city_br_813,27000-001,27174-999
zip_range_0527,city_br_814,97538-000,97539-999
zip_range_0528,city_br_815,96790-000,96799-999
zip_range_0529,city_br_816,99795-000,99799-999
zip_range_0530,city_br_817,45560-000,45569-999
zip_range_0531,city_br_818,11955-000,11959-999
zip_range_0532,city_br_819,49140-000,49149-999
zip_range_0533,city_br_820,99585-000,99589-999
zip_range_0534,city_br_821,35447-000,35449-999
zip_range_0535,city_br_172,27300-001,27399-999
zip_range_0536,city_br_822,88390-000,88394-999
zip_range_0537,city_br_823,85700-000,85707-999
zip_range_0538,city_br_824,95370-000,95379-999
zip_range_0539,city_br_825,64100-000,64104-999
zip_range_0540,city_br_826,62795-000,62799-999
zip_range_0541,city_br_185,47800-001,47819-999
zip_range_0542,city_br_827,64990-000,64992-999
zip_range_0543,city_br_828,69160-000,69179-999
zip_range_0544,city_br_829,65590-000,65599-999
zip_range_0545,city_br_830,55560-000,55564-999
zip_range_0546,city_br_250,14780-001,14789-999
zip_range_0547,city_br_831,14860-000,14869-999
zip_range_0548,city_br_832,63380-000,63399-999
zip_range_0549,city_br_833,44895-000,44899-999
zip_range_0550,city_br_834,76390-000,76392-999
zip_range_0551,city_br_835,64455-000,64459-999
zip_range_0552,city_br_836,45625-000,45629-999
zip_range_0553,city_br_837,48705-000,48709-999
zip_range_0554,city_br_838,77665-000,77669-999
zip_range_0555,city_br_839,62410-000,62419-999
zip_range_0556,city_br_840,99360-000,99369-999
zip_range_0557,city_br_841,36212-000,36212-999
zip_range_0558,city_br_088,06400-001,06499-999
zip_range_0559,city_br_842,17690-000,17699-999
zip_range_0560,city_br_843,79780-000,79784-999
zip_range_0561,city_br_844,57420-000,57424-999
zip_range_0562,city_br_845,64190-000,64199-999
zip_range_0563,city_br_846,14300-000,14339-999
zip_range_0564,city_br_847,79760-000,79764-999
zip_range_0565,city_br_848,62760-000,62761-999
zip_range_0566,city_br_066,17000-001,17119-999
zip_range_0567,city_br_849,58305-001,58309-999
zip_range_0568,city_br_850,14700-001,14719-999
zip_range_0569,city_br_851,62840-000,62849-999
zip_range_0570,city_br_852,62570-000,62579-999
zip_range_0571,city_br_853,79260-000,79269-999
zip_range_0572,city_br_854,85745-000,85749-999
zip_range_0573,city_br_855,75240-000,75244-999
zip_range_0574,city_br_856,35938-000,35939-999
zip_range_0575,city_br_857,65335-000,65339-999
zip_range_0576,city_br_858,86130-000,86139-999
zip_range_0577,city_br_859,64705-000,64709-999
zip_range_0578,city_br_860,89478-000,89479-999
zip_range_0579,city_br_861,65535-000,65539-999
zip_range_0580,city_br_862,57630-000,57634-999
zip_range_0581,city_br_012,66000-001,66999-999
zip_range_0582,city_br_863,58255-000,58259-999
zip_range_0583,city_br_864,55440-000,55449-999
zip_range_0584,city_br_865,58895-000,58899-999
zip_range_0585,city_br_866,64678-000,64679-999
zip_range_0586,city_br_867,56440-000,56459-999
zip_range_0587,city_br_043,26100-001,26199-999
zip_range_0588,city_br_868,36126-000,36129-999
zip_range_0589,city_br_869,45800-000,45806-999
zip_range_0590,city_br_870,89925-000,89929-999
zip_range_0591,city_br_871,45160-000,45169-999
zip_range_0592,city_br_006,30000-001,31999-999
zip_range_0593,city_br_872,55150-001,55169-999
zip_range_0594,city_br_873,57435-000,57439-999
zip_range_0595,city_br_874,35195-000,35197-999
zip_range_0596,city_br_875,35473-000,35475-999
zip_range_0597,city_br_876,68143-000,68144-999
zip_range_0598,city_br_877,64380-000,64387-999
zip_range_0599,city_br_878,65885-000,65887-999
zip_range_0600,city_br_879,89124-000,89125-999
zip_range_0601,city_br_880,68795-000,68797-999
zip_range_0602,city_br_881,69630-000,69639-999
zip_range_0603,city_br_882,99650-000,99654-999
zip_range_0604,city_br_883,16790-000,16799-999
zip_range_0605,city_br_884,59555-000,59559-999
zip_range_0606,city_br_248,95700-001,95714-999
zip_range_0607,city_br_885,65248-000,65249-999
zip_range_0608,city_br_886,39640-000,39641-999
zip_range_0609,city_br_887,39555-000,39557-999
zip_range_0610,city_br_888,58922-000,58924-999
zip_range_0611,city_br_889,18960-000,18969-999
zip_range_0612,city_br_890,65723-000,65724-999
zip_range_0613,city_br_891,77755-000,77759-999
zip_range_0614,city_br_892,11250-000,11299-999
zip_range_0615,city_br_893,64870-000,64872-999
zip_range_0616,city_br_894,39875-000,39877-999
zip_range_0617,city_br_895,69430-000,69434-999
zip_range_0618,city_br_896,56670-000,56699-999
zip_range_0619,city_br_897,64753-000,64754-999
zip_range_0620,city_br_060,32600-001,32699-999
zip_range_0621,city_br_898,55660-000,55664-999
zip_range_0622,city_br_899,36230-000,36234-999
zip_range_0623,city_br_900,36600-000,36603-999
zip_range_0624,city_br_901,88160-001,88179-999
zip_range_0625,city_br_902,16210-000,16219-999
zip_range_0626,city_br_903,35621-000,35621-999
zip_range_0627,city_br_255,16200-001,16209-999
zip_range_0628,city_br_904,08940-000,08969-999
zip_range_0629,city_br_905,48780-000,48789-999
zip_range_0630,city_br_906,84640-000,84659-999
zip_range_0631,city_br_071,89000-001,89099-999
zip_range_0632,city_br_907,29845-000,29849-999
zip_range_0633,city_br_908,37170-000,37174-999
zip_range_0634,city_br_909,87390-000,87394-999
zip_range_0635,city_br_910,85680-000,85684-999
zip_range_0636,city_br_911,14930-000,14934-999
zip_range_0637,city_br_912,64108-000,64109-999
zip_range_0638,city_br_913,45250-000,45254-999
zip_range_0639,city_br_914,59260-000,59269-999
zip_range_0640,city_br_915,58993-000,58993-999
zip_range_0641,city_br_916,85225-000,85229-999
zip_range_0642,city_br_917,63870-000,63899-999
zip_range_0643,city_br_918,58123-000,58124-999
zip_range_0644,city_br_059,69300-001,69339-999
zip_range_0645,city_br_919,85780-000,85789-999
zip_range_0646,city_br_920,98335-000,98337-999
zip_range_0647,city_br_921,98918-000,98918-999
zip_range_0648,city_br_922,98118-000,98119-999
zip_range_0649,city_br_923,65292-000,65292-999
zip_range_0650,city_br_924,98120-000,98124-999
zip_range_0651,city_br_925,69195-000,69199-999
zip_range_0652,city_br_926,95727-000,95729-999
zip_range_0653,city_br_927,46850-000,46859-999
zip_range_0654,city_br_928,57680-000,57689-999
zip_range_0655,city_br_929,69850-000,69859-999
zip_range_0656,city_br_930,64630-000,64634-999
zip_range_0657,city_br_931,17240-000,17249-999
zip_range_0658,city_br_932,37340-000,37349-999
zip_range_0659,city_br_933,88538-000,88539-999
zip_range_0660,city_br_934,39390-000,39396-999
zip_range_0661,city_br_935,83450-000,83479-999
zip_range_0662,city_br_936,59528-000,59529-999
zip_range_0663,city_br_937,56220-000,56229-999
zip_range_0664,city_br_938,79390-000,79399-999
zip_range_0665,city_br_939,18590-000,18599-999
zip_range_0666,city_br_940,18550-000,18559-999
zip_range_0667,city_br_941,55330-000,55339-999
zip_range_0668,city_br_942,35600-000,35602-999
zip_range_0669,city_br_943,65380-000,65384-999
zip_range_0670,city_br_944,55730-000,55739-999
zip_range_0671,city_br_945,28660-000,28679-999
zip_range_0672,city_br_946,88640-000,88649-999
zip_range_0673,city_br_947,76245-000,76249-999
zip_range_0674,city_br_948,37310-000,37329-999
zip_range_0675,city_br_949,58930-000,58932-999
zip_range_0676,city_br_950,64900-000,64904-999
zip_range_0677,city_br_951,59270-000,59274-999
zip_range_0678,city_br_952,95290-000,95299-999
zip_range_0679,city_br_953,89824-000,89824-999
zip_range_0680,city_br_954,47600-000,47609-999
zip_range_0681,city_br_955,37948-000,37949-999
zip_range_0682,city_br_956,45263-000,45264-999
zip_range_0683,city_br_957,65395-000,65397-999
zip_range_0684,city_br_958,75570-000,75579-999
zip_range_0685,city_br_959,35908-000,35909-999
zip_range_0686,city_br_960,78678-000,78679-999
zip_range_0687,city_br_961,35340-000,35344-999
zip_range_0688,city_br_962,28360-000,28374-999
zip_range_0689,city_br_963,29460-000,29469-999
zip_range_0690,city_br_964,89873-000,89873-999
zip_range_0691,city_br_965,85708-000,85709-999
zip_range_0692,city_br_966,68525-000,68526-999
zip_range_0693,city_br_967,77714-000,77715-999
zip_range_0694,city_br_968,12955-000,12959-999
zip_range_0695,city_br_969,65704-000,65704-999
zip_range_0696,city_br_970,95765-000,95767-999
zip_range_0697,city_br_971,64225-000,64227-999
zip_range_0698,city_br_972,98575-000,98579-999
zip_range_0699,city_br_973,37610-000,37614-999
zip_range_0700,city_br_974,88680-000,88699-999
zip_range_0701,city_br_975,95870-000,95874-999
zip_range_0702,city_br_976,37220-000,37222-999
zip_range_0703,city_br_977,58887-000,58889-999
zip_range_0704,city_br_978,86940-000,86944-999
zip_range_0705,city_br_979,18475-000,18479-999
zip_range_0706,city_br_980,85515-000,85519-999
zip_range_0707,city_br_981,88215-000,88219-999
zip_range_0708,city_br_982,35480-000,35484-999
zip_range_0709,city_br_983,69380-000,69389-999
zip_range_0710,city_br_984,64775-000,64777-999
zip_range_0711,city_br_985,75195-000,75199-999
zip_range_0712,city_br_986,38650-000,38653-999
zip_range_0713,city_br_987,46740-000,46749-999
zip_range_0714,city_br_988,46820-000,46824-999
zip_range_0715,city_br_989,79290-000,79299-999
zip_range_0716,city_br_990,68645-000,68646-999
zip_range_0717,city_br_991,55680-000,55689-999
zip_range_0718,city_br_992,39490-000,39491-999
zip_range_0719,city_br_993,58960-000,58969-999
zip_range_0720,city_br_994,76555-000,76559-999
zip_range_0721,city_br_995,58450-000,58454-999
zip_range_0722,city_br_996,95920-000,95922-999
zip_range_0723,city_br_997,64283-000,64284-999
zip_range_0724,city_br_998,49360-000,49389-999
zip_range_0725,city_br_999,46530-000,46539-999
zip_range_0726,city_br_1000,19740-000,19749-999
zip_range_0727,city_br_1001,17270-000,17279-999
zip_range_0728,city_br_1002,69200-000,69229-999
zip_range_0729,city_br_1003,58394-000,58394-999
zip_range_0730,city_br_1004,14955-000,14959-999
zip_range_0731,city_br_1005,37564-000,37565-999
zip_range_0732,city_br_1006,18675-000,18679-999
zip_range_0733,city_br_1007,86925-000,86929-999
zip_range_0734,city_br_1008,97850-000,97869-999
zip_range_0735,city_br_1009,37720-000,37729-999
zip_range_0736,city_br_208,18600-001,18619-999
zip_range_0737,city_br_1010,39595-000,39597-999
zip_range_0738,city_br_1011,46570-000,46574-999
zip_range_0739,city_br_1012,88295-000,88299-999
zip_range_0740,city_br_1013,98733-000,98734-999
zip_range_0741,city_br_1014,88750-000,88759-999
zip_range_0742,city_br_1015,89178-000,89179-999
zip_range_0743,city_br_1016,98560-000,98569-999
zip_range_0744,city_br_249,68600-000,68609-999
zip_range_0745,city_br_168,12900-001,12929-999
zip_range_0746,city_br_1017,85430-000,85439-999
zip_range_0747,city_br_1018,57830-000,57839-999
zip_range_0748,city_br_1019,36542-000,36543-999
zip_range_0749,city_br_1020,68148-000,68149-999
zip_range_0750,city_br_1021,79670-000,79679-999
zip_range_0751,city_br_1022,38779-000,38779-999
zip_range_0752,city_br_1023,87595-000,87599-999
zip_range_0753,city_br_1024,77735-000,77739-999
zip_range_0754,city_br_1025,69932-000,69933-999
zip_range_0755,city_br_1026,64265-000,64269-999
zip_range_0756,city_br_003,70000-001,72799-999
zip_range_0757,city_br_003,73000-001,73699-999
zip_range_0758,city_br_1027,39330-000,39334-999
zip_range_0759,city_br_1028,78350-000,78359-999
zip_range_0760,city_br_1029,16290-000,16299-999
zip_range_0761,city_br_1030,35189-000,35189-999
zip_range_0762,city_br_1031,75440-000,75449-999
zip_range_0763,city_br_1032,37530-000,37539-999
zip_range_0764,city_br_1033,55325-000,55329-999
zip_range_0765,city_br_1034,29630-000,29639-999
zip_range_0766,city_br_1035,56740-000,56749-999
zip_range_0767,city_br_1036,59219-000,59219-999
zip_range_0768,city_br_1037,77560-000,77564-999
zip_range_0769,city_br_1038,65520-000,65524-999
zip_range_0770,city_br_1039,16265-000,16269-999
zip_range_0771,city_br_1040,55170-000,55179-999
zip_range_0772,city_br_1041,65315-000,65319-999
zip_range_0773,city_br_1042,58890-000,58892-999
zip_range_0774,city_br_1043,64895-000,64897-999
zip_range_0775,city_br_1044,58880-000,58883-999
zip_range_0776,city_br_1045,49995-000,49999-999
zip_range_0777,city_br_1046,68521-000,68522-999
zip_range_0778,city_br_1047,63260-000,63269-999
zip_range_0779,city_br_1048,45325-000,45329-999
zip_range_0780,city_br_1049,47750-000,47759-999
zip_range_0781,city_br_1050,68488-000,68489-999
zip_range_0782,city_br_291,68800-000,68809-999
zip_range_0783,city_br_1051,76280-000,76284-999
zip_range_0784,city_br_1052,95790-000,95792-999
zip_range_0785,city_br_1053,14340-000,14349-999
zip_range_0786,city_br_1054,17380-000,17399-999
zip_range_0787,city_br_1055,47560-000,47579-999
zip_range_0788,city_br_1056,35460-000,35469-999
zip_range_0789,city_br_1057,46100-000,46109-999
zip_range_0790,city_br_1058,89634-000,89635-999
zip_range_0791,city_br_212,88350-001,88359-999
zip_range_0792,city_br_1059,37578-000,37579-999
zip_range_0793,city_br_1060,39230-000,39236-999
zip_range_0794,city_br_1061,55845-000,55849-999
zip_range_0795,city_br_1062,45615-000,45619-999
zip_range_0796,city_br_1063,35193-000,35193-999
zip_range_0797,city_br_1064,56520-000,56539-999
zip_range_0798,city_br_1065,69926-000,69926-999
zip_range_0799,city_br_1066,68670-000,68674-999
zip_range_0800,city_br_1067,18290-000,18299-999
zip_range_0801,city_br_1068,15290-000,15299-999
zip_range_0802,city_br_1069,65515-000,65519-999
zip_range_0803,city_br_1070,75660-000,75664-999
zip_range_0804,city_br_1071,65685-000,65689-999
zip_range_0805,city_br_1072,76152-000,76154-999
zip_range_0806,city_br_1073,77995-000,77999-999
zip_range_0807,city_br_1074,64230-000,64232-999
zip_range_0808,city_br_1075,64345-000,64349-999
zip_range_0809,city_br_1076,65393-000,65394-999
zip_range_0810,city_br_1077,73975-000,73979-999
zip_range_0811,city_br_1078,47120-000,47149-999
zip_range_0812,city_br_1079,65935-500,65935-999
zip_range_0813,city_br_1080,38660-000,38679-999
zip_range_0814,city_br_1081,76880-000,76886-999
zip_range_0815,city_br_1082,14570-000,14579-999
zip_range_0816,city_br_1083,39280-000,39289-999
zip_range_0817,city_br_1084,96750-000,96754-999
zip_range_0818,city_br_1085,69425-000,69429-999
zip_range_0819,city_br_1086,58326-000,58327-999
zip_range_0820,city_br_1087,79940-000,79949-999
zip_range_0821,city_br_1088,45130-000,45139-999
zip_range_0822,city_br_1089,58480-000,58482-999
zip_range_0823,city_br_1090,44345-000,44349-999
zip_range_0824,city_br_1091,38625-000,38629-999
zip_range_0825,city_br_1092,73870-000,73889-999
zip_range_0826,city_br_1093,64105-000,64107-999
zip_range_0827,city_br_1094,58100-001,58109-999
zip_range_0828,city_br_1095,76994-000,76994-999
zip_range_0829,city_br_149,54500-001,54599-999
zip_range_0830,city_br_142,28900-001,28929-999
zip_range_0831,city_br_1096,37880-000,37889-999
zip_range_0832,city_br_1097,17480-000,17489-999
zip_range_0833,city_br_1098,13315-000,13319-999
zip_range_0834,city_br_1099,56180-000,56189-999
zip_range_0835,city_br_1100,89500-001,89514-999
zip_range_0836,city_br_1101,12280-001,12299-999
zip_range_0837,city_br_1102,96570-000,96589-999
zip_range_0838,city_br_1103,76889-000,76889-999
zip_range_0839,city_br_1104,97450-000,97499-999
zip_range_0840,city_br_1105,78200-000,78236-999
zip_range_0841,city_br_1106,44300-000,44319-999
zip_range_0842,city_br_1107,75870-000,75879-999
zip_range_0843,city_br_1108,35765-000,35766-999
zip_range_0844,city_br_1109,76125-000,76129-999
zip_range_0845,city_br_1110,37545-000,37547-999
zip_range_0846,city_br_1111,39980-000,39989-999
zip_range_0847,city_br_1112,68840-000,68849-999
zip_range_0848,city_br_1113,68617-000,68617-999
zip_range_0849,city_br_1114,96500-001,96529-999
zip_range_0850,city_br_1115,58935-000,58939-999
zip_range_0851,city_br_1116,75560-000,75569-999
zip_range_0852,city_br_1117,38370-000,38379-999
zip_range_0853,city_br_1118,65165-000,65169-999
zip_range_0854,city_br_1119,12630-000,12689-999
zip_range_0855,city_br_1120,28680-000,28699-999
zip_range_0856,city_br_1121,55380-000,55384-999
zip_range_0857,city_br_218,94900-001,94999-999
zip_range_0858,city_br_1122,77915-000,77917-999
zip_range_0859,city_br_162,29300-001,29329-999
zip_range_0860,city_br_1123,58730-000,58731-999
zip_range_0861,city_br_1124,58230-000,58232-999
zip_range_0862,city_br_1125,58698-000,58699-999
zip_range_0863,city_br_1126,57570-000,57579-999
zip_range_0864,city_br_1127,99860-000,99869-999
zip_range_0865,city_br_1128,76960-001,76969-999
zip_range_0866,city_br_1129,13770-000,13779-999
zip_range_0867,city_br_1130,75813-000,75814-999
zip_range_0868,city_br_1131,46300-000,46309-999
zip_range_0869,city_br_1132,44730-000,44739-999
zip_range_0870,city_br_1133,35770-000,35773-999
zip_range_0871,city_br_1134,45265-000,45269-999
zip_range_0872,city_br_1135,34800-000,34989-999
zip_range_0873,city_br_1136,55360-000,55364-999
zip_range_0874,city_br_1137,46400-000,46424-999
zip_range_0875,city_br_1138,44880-000,44884-999
zip_range_0876,city_br_1139,86640-000,86649-999
zip_range_0877,city_br_1140,85415-000,85419-999
zip_range_0878,city_br_1141,16500-000,16569-999
zip_range_0879,city_br_1142,87565-000,87569-999
zip_range_0880,city_br_1143,19530-000,19559-999
zip_range_0881,city_br_1144,36832-000,36833-999
zip_range_0882,city_br_1145,75850-000,75854-999
zip_range_0883,city_br_1146,97930-000,97934-999
zip_range_0884,city_br_1147,89888-000,89889-999
zip_range_0885,city_br_1148,58253-000,58253-999
zip_range_0886,city_br_1149,98440-000,98449-999
zip_range_0887,city_br_1150,59592-000,59593-999
zip_range_0888,city_br_1151,59540-000,59543-999
zip_range_0889,city_br_1152,59300-000,59309-999
zip_range_0890,city_br_1153,07700-001,07749-999
zip_range_0891,city_br_1154,45420-000,45429-999
zip_range_0892,city_br_1155,19450-000,19469-999
zip_range_0893,city_br_1156,07750-001,07799-999
zip_range_0894,city_br_1157,65230-000,65232-999
zip_range_0895,city_br_1158,65210-000,65212-999
zip_range_0896,city_br_1159,11950-000,11954-999
zip_range_0897,city_br_1160,58900-000,58907-999
zip_range_0898,city_br_1161,64514-000,64515-999
zip_range_0899,city_br_1162,58855-000,58856-999
zip_range_0900,city_br_1163,15410-000,15419-999
zip_range_0901,city_br_1164,57770-000,57779-999
zip_range_0902,city_br_1165,64222-000,64223-999
zip_range_0903,city_br_1166,36560-000,36569-999
zip_range_0904,city_br_1167,14240-000,14249-999
zip_range_0905,city_br_1168,55375-000,55379-999
zip_range_0906,city_br_1169,68960-000,68972-999
zip_range_0907,city_br_1170,37780-000,37789-999
zip_range_0908,city_br_1171,58350-000,58353-999
zip_range_0909,city_br_1172,75690-000,75694-999
zip_range_0910,city_br_1173,75245-000,75249-999
zip_range_0911,city_br_1174,44750-000,44754-999
zip_range_0912,city_br_1175,64695-000,64699-999
zip_range_0913,city_br_1176,86820-000,86824-999
zip_range_0914,city_br_1177,89430-000,89439-999
zip_range_0915,city_br_1178,56930-000,56949-999
zip_range_0916,city_br_1179,45880-000,45889-999
zip_range_0917,city_br_092,42800-001,42849-999
zip_range_0918,city_br_1180,35555-000,35556-999
zip_range_0919,city_br_1181,58530-000,58534-999
zip_range_0920,city_br_1182,45445-000,45449-999
zip_range_0921,city_br_1183,37650-000,37654-999
zip_range_0922,city_br_1184,79420-000,79427-999
zip_range_0923,city_br_1185,96180-000,96189-999
zip_range_0924,city_br_203,54750-001,54799-999
zip_range_0925,city_br_1186,99165-000,99169-999
zip_range_0926,city_br_1187,86390-000,86399-999
zip_range_0927,city_br_1188,95480-000,95499-999
zip_range_0928,city_br_290,86180-001,86199-999
zip_range_0929,city_br_1189,86890-000,86894-999
zip_range_0930,city_br_312,88340-001,88349-999
zip_range_0931,city_br_1190,28430-000,28454-999
zip_range_0932,city_br_1191,37600-000,37604-999
zip_range_0933,city_br_1192,37420-000,37429-999
zip_range_0934,city_br_221,68400-000,68414-999
zip_range_0935,city_br_1193,62400-000,62409-999
zip_range_0936,city_br_1194,55665-000,55669-999
zip_range_0937,city_br_1195,39835-000,39836-999
zip_range_0938,city_br_1196,37400-000,37404-999
zip_range_0939,city_br_1197,57968-000,57969-999
zip_range_0940,city_br_1198,37730-000,37739-999
zip_range_0941,city_br_1199,95255-000,95259-999
zip_range_0942,city_br_1200,75396-000,75397-999
zip_range_0943,city_br_1201,65968-000,65969-999
zip_range_0944,city_br_1202,87345-000,87354-999
zip_range_0945,city_br_1203,98975-000,98979-999
zip_range_0946,city_br_1204,18245-000,18249-999
zip_range_0947,city_br_1205,85148-000,85149-999
zip_range_0948,city_br_055,58400-001,58449-999
zip_range_0949,city_br_1206,83430-000,83449-999
zip_range_0950,city_br_1207,38270-000,38279-999
zip_range_0951,city_br_1208,76440-000,76449-999
zip_range_0952,city_br_1209,78630-000,78634-999
zip_range_0953,city_br_014,13000-001,13139-999
zip_range_0954,city_br_1210,64730-000,64739-999
zip_range_0955,city_br_1211,99660-000,99664-999
zip_range_0956,city_br_1212,76410-000,76419-999
zip_range_0957,city_br_1213,57250-000,57254-999
zip_range_0958,city_br_1214,89294-000,89294-999
zip_range_0959,city_br_1215,75795-000,75799-999
zip_range_0960,city_br_1216,47220-000,47239-999
zip_range_0961,city_br_1217,64767-000,64767-999
zip_range_0962,city_br_1218,39338-000,39339-999
zip_range_0963,city_br_1219,37270-000,37272-999
zip_range_0964,city_br_1220,88580-000,88584-999
zip_range_0965,city_br_1221,93700-000,93799-999
zip_range_0966,city_br_1222,85450-000,85459-999
zip_range_0967,city_br_1223,49520-000,49524-999
zip_range_0968,city_br_1224,37165-000,37169-999
zip_range_0969,city_br_1225,83870-000,83879-999
zip_range_0970,city_br_1226,89980-000,89980-999
zip_range_0971,city_br_1227,38130-000,38139-999
zip_range_0972,city_br_1228,44790-000,44797-999
zip_range_0973,city_br_1229,57350-000,57359-999
zip_range_0974,city_br_017,79000-001,79129-999
zip_range_0975,city_br_1230,59680-000,59684-999
zip_range_0976,city_br_1231,64578-000,64579-999
zip_range_0977,city_br_217,83600-001,83649-999
zip_range_0978,city_br_1232,64148-000,64149-999
zip_range_0979,city_br_1233,75160-000,75164-999
zip_range_0980,city_br_1234,13230-001,13239-999
zip_range_0981,city_br_1235,83535-000,83539-999
zip_range_0982,city_br_1236,64280-000,64282-999
zip_range_0983,city_br_1237,87300-001,87319-999
zip_range_0984,city_br_1238,98570-000,98574-999
zip_range_0985,city_br_1239,76887-000,76887-999
zip_range_0986,city_br_1240,78360-000,78364-999
zip_range_0987,city_br_1241,59230-000,59234-999
zip_range_0988,city_br_1242,78840-000,78849-999
zip_range_0989,city_br_1243,38970-000,38979-999
zip_range_0990,city_br_1244,73840-000,73849-999
zip_range_0991,city_br_1245,99435-000,99439-999
zip_range_0992,city_br_1246,78307-000,78309-999
zip_range_0993,city_br_1247,12460-000,12489-999
zip_range_0994,city_br_042,28000-001,28179-999
zip_range_0995,city_br_1248,37160-000,37164-999
zip_range_0996,city_br_1249,77777-000,77779-999
zip_range_0997,city_br_1250,89620-000,89632-999
zip_range_0998,city_br_1251,19960-000,19969-999
zip_range_0999,city_br_1252,63150-000,63154-999
zip_range_1000,city_br_1253,76515-000,76519-999
zip_range_1001,city_br_1254,55930-000,55939-999
zip_range_1002,city_br_1255,37267-000,37269-999
zip_range_1003,city_br_1256,36592-000,36593-999
zip_range_1004,city_br_1257,68537-000,68539-999
zip_range_1005,city_br_1258,78658-000,78659-999
zip_range_1006,city_br_1259,11990-000,11999-999
zip_range_1007,city_br_1260,57530-000,57534-999
zip_range_1008,city_br_1261,47730-000,47739-999
zip_range_1009,city_br_1262,38380-000,38389-999
zip_range_1010,city_br_1263,44890-000,44894-999
zip_range_1011,city_br_1264,78640-000,78642-999
zip_range_1012,city_br_1265,12615-000,12619-999
zip_range_1013,city_br_1266,64833-000,64834-999
zip_range_1014,city_br_1267,45860-000,45864-999
zip_range_1015,city_br_1268,48710-000,48719-999
zip_range_1016,city_br_1269,43800-001,43849-999
zip_range_1017,city_br_1270,37280-000,37299-999
zip_range_1018,city_br_1271,76860-000,76860-999
zip_range_1019,city_br_1272,96930-000,96949-999
zip_range_1020,city_br_1273,46380-000,46389-999
zip_range_1021,city_br_1274,84470-000,84499-999
zip_range_1022,city_br_1275,98970-000,98974-999
zip_range_1023,city_br_1276,65280-000,65282-999
zip_range_1024,city_br_1277,19880-000,19899-999
zip_range_1025,city_br_1278,15930-000,15939-999
zip_range_1026,city_br_1279,45157-000,45159-999
zip_range_1027,city_br_1280,96495-000,96499-999
zip_range_1028,city_br_1281,85140-000,85144-999
zip_range_1029,city_br_1282,95680-000,95689-999
zip_range_1030,city_br_1283,88230-000,88239-999
zip_range_1031,city_br_1284,59190-000,59191-999
zip_range_1032,city_br_1285,96600-000,96609-999
zip_range_1033,city_br_1286,49880-000,49889-999
zip_range_1034,city_br_1287,55420-000,55429-999
zip_range_1035,city_br_1288,62700-000,62719-999
zip_range_1036,city_br_1289,49820-000,49829-999
zip_range_1037,city_br_1290,18990-000,18999-999
zip_range_1038,city_br_079,92000-001,92479-999
zip_range_1039,city_br_1291,89460-000,89477-999
zip_range_1040,city_br_1292,48840-000,48849-999
zip_range_1041,city_br_1293,69390-000,69399-999
zip_range_1042,city_br_1294,39703-000,39703-999
zip_range_1043,city_br_1295,85160-000,85161-999
zip_range_1044,city_br_1296,28500-000,28539-999
zip_range_1045,city_br_1297,65465-000,65467-999
zip_range_1046,city_br_1298,64890-000,64892-999
zip_range_1047,city_br_1299,48520-000,48539-999
zip_range_1048,city_br_1300,95933-000,95934-999
zip_range_1049,city_br_1301,69820-000,69829-999
zip_range_1050,city_br_1302,68700-001,68706-999
zip_range_1051,city_br_1303,85760-000,85769-999
zip_range_1052,city_br_1304,88548-000,88549-999
zip_range_1053,city_br_1305,18300-001,18309-999
zip_range_1054,city_br_1306,95308-000,95309-999
zip_range_1055,city_br_1307,95555-000,95559-999
zip_range_1056,city_br_1308,97753-000,97754-999
zip_range_1057,city_br_1309,96160-000,96169-999
zip_range_1058,city_br_1310,36834-000,36835-999
zip_range_1059,city_br_1311,57780-000,57799-999
zip_range_1060,city_br_1312,49700-000,49739-999
zip_range_1061,city_br_1313,95745-000,95747-999
zip_range_1062,city_br_1314,18195-000,18199-999
zip_range_1063,city_br_1315,44645-000,44649-999
zip_range_1064,city_br_1316,36290-000,36299-999
zip_range_1065,city_br_1317,39680-000,39684-999
zip_range_1066,city_br_1318,37993-000,37996-999
zip_range_1067,city_br_1319,58287-000,58288-999
zip_range_1068,city_br_1320,35730-000,35731-999
zip_range_1069,city_br_1321,44695-000,44697-999
zip_range_1070,city_br_1322,38360-000,38369-999
zip_range_1071,city_br_1323,89665-000,89666-999
zip_range_1072,city_br_1324,65735-000,65739-999
zip_range_1073,city_br_1325,62748-000,62749-999
zip_range_1074,city_br_1326,95935-000,95936-999
zip_range_1075,city_br_1327,35123-000,35124-999
zip_range_1076,city_br_1328,64270-000,64274-999
zip_range_1077,city_br_1329,39472-000,39474-999
zip_range_1078,city_br_1330,64763-000,64763-999
zip_range_1079,city_br_1331,85790-000,85794-999
zip_range_1080,city_br_1332,68650-000,68654-999
zip_range_1081,city_br_1333,37930-000,37939-999
zip_range_1082,city_br_1334,13360-000,13369-999
zip_range_1083,city_br_1335,88745-000,88749-999
zip_range_1084,city_br_1336,95552-000,95554-999
zip_range_1085,city_br_1337,69931-000,69931-999
zip_range_1086,city_br_1338,55365-000,55369-999
zip_range_1087,city_br_1339,36925-000,36929-999
zip_range_1088,city_br_1340,95515-000,95519-999
zip_range_1089,city_br_1341,69360-000,69369-999
zip_range_1090,city_br_1342,79270-000,79279-999
zip_range_1091,city_br_1343,64795-000,64797-999
zip_range_1092,city_br_219,11660-001,11679-999
zip_range_1093,city_br_1344,39810-000,39813-999
zip_range_1094,city_br_1345,45177-000,45179-999
zip_range_1095,city_br_1346,84145-000,84149-999
zip_range_1096,city_br_1347,36428-000,36429-999
zip_range_1097,city_br_1348,36280-000,36289-999
zip_range_1098,city_br_1349,36800-000,36809-999
zip_range_1099,city_br_1350,27998-000,27999-999
zip_range_1100,city_br_064,06300-001,06399-999
zip_range_1101,city_br_1351,35300-001,35322-999
zip_range_1102,city_br_1352,69500-000,69509-999
zip_range_1103,city_br_1353,58595-000,58599-999
zip_range_1104,city_br_1354,59780-000,59789-999
zip_range_1105,city_br_1355,64233-000,64234-999
zip_range_1106,city_br_1356,45900-000,45909-999
zip_range_1107,city_br_1357,99500-000,99522-999
zip_range_1108,city_br_1358,39665-000,39669-999
zip_range_1109,city_br_1359,48390-000,48399-999
zip_range_1110,city_br_1360,15570-000,15579-999
zip_range_1111,city_br_1361,28180-000,28199-999
zip_range_1112,city_br_1362,37582-000,37583-999
zip_range_1113,city_br_1363,69250-000,69254-999
zip_range_1114,city_br_1364,69255-000,69259-999
zip_range_1115,city_br_074,29140-001,29159-999
zip_range_1116,city_br_1365,62730-000,62735-999
zip_range_1117,city_br_1366,64590-000,64594-999
zip_range_1118,city_br_1367,46445-000,46445-999
zip_range_1119,city_br_1368,49550-000,49559-999
zip_range_1120,city_br_1369,62184-000,62189-999
zip_range_1121,city_br_1370,77453-000,77454-999
zip_range_1122,city_br_1371,63220-000,63229-999
zip_range_1123,city_br_1372,63530-000,63539-999
zip_range_1124,city_br_1373,78587-000,78589-999
zip_range_1125,city_br_1374,86420-000,86429-999
zip_range_1126,city_br_1375,95185-000,95189-999
zip_range_1127,city_br_1376,39864-000,39867-999
zip_range_1128,city_br_1377,99825-000,99829-999
zip_range_1129,city_br_1378,35878-000,35879-999
zip_range_1130,city_br_1379,28640-000,28649-999
zip_range_1131,city_br_1380,37225-000,37234-999
zip_range_1132,city_br_1381,35547-000,35549-999
zip_range_1133,city_br_1382,37472-000,37473-999
zip_range_1134,city_br_1383,35557-000,35559-999
zip_range_1135,city_br_1384,38840-000,38859-999
zip_range_1136,city_br_1385,37150-000,37159-999
zip_range_1137,city_br_1386,76340-000,76342-999
zip_range_1138,city_br_1387,77840-000,77844-999
zip_range_1139,city_br_1388,49740-000,49749-999
zip_range_1140,city_br_1389,35534-000,35535-999
zip_range_1141,city_br_1390,56820-000,56827-999
zip_range_1142,city_br_1391,59374-000,59374-999
zip_range_1143,city_br_1392,59665-000,59667-999
zip_range_1144,city_br_1393,62375-000,62379-999
zip_range_1145,city_br_1394,56420-000,56429-999
zip_range_1146,city_br_1395,38290-000,38294-999
zip_range_1147,city_br_1396,57535-000,57539-999
zip_range_1148,city_br_1397,69378-000,69379-999
zip_range_1149,city_br_1398,65980-000,65989-999
zip_range_1150,city_br_1399,55810-001,55819-999
zip_range_1151,city_br_1400,37245-000,37249-999
zip_range_1152,city_br_1401,58945-000,58949-999
zip_range_1153,city_br_1402,77985-000,77989-999
zip_range_1154,city_br_067,55000-001,55119-999
zip_range_1155,city_br_1403,65295-000,65298-999
zip_range_1156,city_br_1404,37760-000,37774-999
zip_range_1157,city_br_1405,37456-000,37457-999
zip_range_1158,city_br_1406,13700-000,13709-999
zip_range_1159,city_br_1407,36422-000,36423-999
zip_range_1160,city_br_1408,47300-000,47349-999
zip_range_1161,city_br_1409,99260-000,99264-999
zip_range_1162,city_br_1410,38460-000,38464-999
zip_range_1163,city_br_1411,62850-000,62859-999
zip_range_1164,city_br_078,85800-001,85824-999
zip_range_1165,city_br_1412,77680-000,77684-999
zip_range_1166,city_br_1413,95315-000,95319-999
zip_range_1167,city_br_1414,28860-000,28889-999
zip_range_1168,city_br_1415,55755-000,55759-999
zip_range_1169,city_br_1416,58238-000,58239-999
zip_range_1170,city_br_1417,37980-000,37989-999
zip_range_1171,city_br_1418,14260-000,14269-999
zip_range_1172,city_br_1419,79540-000,79549-999
zip_range_1173,city_br_157,68740-001,68747-999
zip_range_1174,city_br_1420,78345-000,78349-999
zip_range_1175,city_br_1421,76948-000,76949-999
zip_range_1176,city_br_1422,75925-000,75929-999
zip_range_1177,city_br_1423,29360-000,29369-999
zip_range_1178,city_br_1424,64340-000,64342-999
zip_range_1179,city_br_1425,16920-000,16939-999
zip_range_1180,city_br_1426,84160-001,84199-999
zip_range_1181,city_br_1427,44500-000,44519-999
zip_range_1182,city_br_1428,36770-001,36779-999
zip_range_1183,city_br_272,75700-001,75714-999
zip_range_1184,city_br_266,15800-001,15819-999
zip_range_1185,city_br_1429,85470-000,85477-999
zip_range_1186,city_br_1430,89670-000,89674-999
zip_range_1187,city_br_1431,63595-000,63599-999
zip_range_1188,city_br_1432,35969-000,35969-999
zip_range_1189,city_br_1433,36450-000,36454-999
zip_range_1190,city_br_1434,55400-000,55404-999
zip_range_1191,city_br_1435,15870-000,15879-999
zip_range_1192,city_br_1436,58715-000,58719-999
zip_range_1193,city_br_1437,47845-000,47849-999
zip_range_1194,city_br_1438,58884-000,58886-999
zip_range_1195,city_br_1439,48110-000,48119-999
zip_range_1196,city_br_1440,98770-000,98779-999
zip_range_1197,city_br_1441,39816-000,39816-999
zip_range_1198,city_br_1442,62297-000,62299-999
zip_range_1199,city_br_1443,75430-000,75439-999
zip_range_1200,city_br_1444,46575-000,46579-999
zip_range_1201,city_br_1445,58455-000,58457-999
zip_range_1202,city_br_1446,39526-000,39526-999
zip_range_1203,city_br_073,61600-001,61699-999
zip_range_1204,city_br_1447,73790-000,73794-999
zip_range_1205,city_br_1448,37440-000,37442-999
zip_range_1206,city_br_1449,89880-000,89881-999
zip_range_1207,city_br_192,65600-001,65609-999
zip_range_1208,city_br_048,95000-001,95149-999
zip_range_1209,city_br_1450,64228-000,64229-999
zip_range_1210,city_br_1451,59570-000,59574-999
zip_range_1211,city_br_1452,65260-000,65262-999
zip_range_1212,city_br_1453,15895-000,15899-999
zip_range_1213,city_br_1454,63400-000,63429-999
zip_range_1214,city_br_1455,56130-000,56139-999
zip_range_1215,city_br_1456,49930-000,49939-999
zip_range_1216,city_br_1457,35624-000,35624-999
zip_range_1217,city_br_1458,88598-000,88599-999
zip_range_1218,city_br_1459,99838-000,99839-999
zip_range_1219,city_br_1460,77723-000,77724-999
zip_range_1220,city_br_1461,86630-000,86634-999
zip_range_1221,city_br_1462,44940-000,44949-999
zip_range_1222,city_br_1463,35260-000,35264-999
zip_range_1223,city_br_1464,65267-000,65267-999
zip_range_1224,city_br_1465,38390-000,38399-999
zip_range_1225,city_br_1466,65288-000,65288-999
zip_range_1226,city_br_1467,65299-000,65299-999
zip_range_1227,city_br_1468,76997-000,76998-999
zip_range_1228,city_br_1469,76300-000,76303-999
zip_range_1229,city_br_1470,18760-000,18769-999
zip_range_1230,city_br_1471,18520-000,18529-999
zip_range_1231,city_br_1472,96395-000,96399-999
zip_range_1232,city_br_1473,83570-000,83589-999
zip_range_1233,city_br_1474,96535-000,96539-999
zip_range_1234,city_br_1475,59395-000,59399-999
zip_range_1235,city_br_1476,98340-000,98344-999
zip_range_1236,city_br_1477,96770-000,96789-999
zip_range_1237,city_br_1478,97900-000,97919-999
zip_range_1238,city_br_1479,88585-000,88589-999
zip_range_1239,city_br_1480,18285-000,18289-999
zip_range_1240,city_br_1481,85840-000,85844-999
zip_range_1241,city_br_1482,76195-000,76199-999
zip_range_1242,city_br_1483,55835-000,55839-999
zip_range_1243,city_br_1484,55636-000,55639-999
zip_range_1244,city_br_1485,57760-000,57769-999
zip_range_1245,city_br_1486,36110-000,36119-999
zip_range_1246,city_br_1487,36985-000,36989-999
zip_range_1247,city_br_1488,99530-000,99559-999
zip_range_1248,city_br_1489,77378-000,77379-999
zip_range_1249,city_br_1490,77575-000,77579-999
zip_range_1250,city_br_1491,39648-000,39649-999
zip_range_1251,city_br_1492,78195-000,78199-999
zip_range_1252,city_br_1493,39314-000,39314-999
zip_range_1253,city_br_1494,75828-000,75829-999
zip_range_1254,city_br_1495,88407-000,88409-999
zip_range_1255,city_br_1496,79560-000,79569-999
zip_range_1256,city_br_1497,65500-000,65504-999
zip_range_1257,city_br_114,89800-001,89816-999
zip_range_1258,city_br_1498,13515-000,13519-999
zip_range_1259,city_br_1499,96745-000,96749-999
zip_range_1260,city_br_1500,99960-000,99964-999
zip_range_1261,city_br_1501,62420-000,62429-999
zip_range_1262,city_br_1502,18970-000,18989-999
zip_range_1263,city_br_1503,68880-000,68889-999
zip_range_1264,city_br_1504,36630-000,36639-999
zip_range_1265,city_br_1505,98760-000,98764-999
zip_range_1266,city_br_1506,85560-000,85564-999
zip_range_1267,city_br_1507,63950-000,63959-999
zip_range_1268,city_br_1508,62875-000,62879-999
zip_range_1269,city_br_1509,48660-000,48679-999
zip_range_1270,city_br_1510,96255-000,96269-999
zip_range_1271,city_br_1511,76990-000,76992-999
zip_range_1272,city_br_1512,96193-000,96194-999
zip_range_1273,city_br_1513,87200-001,87214-999
zip_range_1274,city_br_1514,48410-000,48414-999
zip_range_1275,city_br_1515,87820-000,87829-999
zip_range_1276,city_br_1516,72880-001,72899-999
zip_range_1277,city_br_1517,65921-000,65921-999
zip_range_1278,city_br_1518,95595-000,95598-999
zip_range_1279,city_br_1519,48450-000,48454-999
zip_range_1280,city_br_1520,36265-000,36269-999
zip_range_1281,city_br_1521,99970-000,99979-999
zip_range_1282,city_br_1522,37997-000,37999-999
zip_range_1283,city_br_1523,39380-000,39386-999
zip_range_1284,city_br_1524,78540-000,78542-999
zip_range_1285,city_br_1525,35530-000,35533-999
zip_range_1286,city_br_1526,16250-000,16259-999
zip_range_1287,city_br_1527,85530-000,85539-999
zip_range_1288,city_br_1528,45638-000,45639-999
zip_range_1289,city_br_1529,69460-000,69469-999
zip_range_1290,city_br_1530,64235-000,64237-999
zip_range_1291,city_br_1531,64278-000,64279-999
zip_range_1292,city_br_1532,88845-000,88849-999
zip_range_1293,city_br_1533,64238-000,64239-999
zip_range_1294,city_br_1534,78680-000,78684-999
zip_range_1295,city_br_1535,72975-000,72979-999
zip_range_1296,city_br_1536,47680-000,47689-999
zip_range_1297,city_br_1537,69450-000,69459-999
zip_range_1298,city_br_273,65400-000,65412-999
zip_range_1299,city_br_1538,65620-000,65624-999
zip_range_1300,city_br_1539,36550-000,36554-999
zip_range_1301,city_br_1540,57325-000,57329-999
zip_range_1302,city_br_1541,64335-000,64339-999
zip_range_1303,city_br_1542,68785-000,68785-999
zip_range_1304,city_br_252,29700-001,29719-999
zip_range_1305,city_br_1543,78500-000,78504-999
zip_range_1306,city_br_1544,14770-000,14774-999
zip_range_1307,city_br_1545,65690-000,65692-999
zip_range_1308,city_br_1546,95895-000,95899-999
zip_range_1309,city_br_1547,73740-000,73749-999
zip_range_1310,city_br_1548,77760-000,77764-999
zip_range_1311,city_br_1549,77725-000,77729-999
zip_range_1312,city_br_1550,78335-000,78337-999
zip_range_1313,city_br_1551,14795-000,14799-999
zip_range_1314,city_br_130,83400-001,83419-999
zip_range_1315,city_br_1552,64885-000,64889-999
zip_range_1316,city_br_1553,64516-000,64517-999
zip_range_1317,city_br_1554,57975-000,57979-999
zip_range_1318,city_br_1555,86690-000,86699-999
zip_range_1319,city_br_1556,99460-000,99469-999
zip_range_1320,city_br_1557,76993-000,76993-999
zip_range_1321,city_br_1558,39770-000,39774-999
zip_range_1322,city_br_1559,77350-000,77352-999
zip_range_1323,city_br_1560,38250-000,38259-999
zip_range_1324,city_br_1561,25870-000,25879-999
zip_range_1325,city_br_1562,39628-000,39629-999
zip_range_1326,city_br_1563,78310-000,78319-999
zip_range_1327,city_br_1564,58970-000,58977-999
zip_range_1328,city_br_1565,37148-000,37149-999
zip_range_1329,city_br_1566,29960-000,29969-999
zip_range_1330,city_br_1567,36360-000,36369-999
zip_range_1331,city_br_1568,44320-000,44329-999
zip_range_1332,city_br_1569,38120-000,38129-999
zip_range_1333,city_br_1570,37527-000,37529-999
zip_range_1334,city_br_1571,36947-000,36949-999
zip_range_1335,city_br_1572,28740-000,28749-999
zip_range_1336,city_br_1573,44540-000,44549-999
zip_range_1337,city_br_1574,68540-000,68542-999
zip_range_1338,city_br_1575,64740-000,64744-999
zip_range_1339,city_br_1576,29370-000,29374-999
zip_range_1340,city_br_1577,48730-000,48749-999
zip_range_1341,city_br_1578,44245-000,44249-999
zip_range_1342,city_br_1579,65340-000,65344-999
zip_range_1343,city_br_1580,35858-000,35864-999
zip_range_1344,city_br_1581,35668-000,35668-999
zip_range_1345,city_br_1582,37430-000,37439-999
zip_range_1346,city_br_1583,77305-000,77307-999
zip_range_1347,city_br_1584,37548-000,37548-999
zip_range_1348,city_br_1585,13835-000,13839-999
zip_range_1349,city_br_1586,18570-000,18579-999
zip_range_1350,city_br_1587,89700-001,89729-999
zip_range_1351,city_br_1588,68685-000,68689-999
zip_range_1352,city_br_1589,58714-000,58714-999
zip_range_1353,city_br_1590,55940-000,55949-999
zip_range_1354,city_br_1591,48300-000,48309-999
zip_range_1355,city_br_1592,58322-000,58323-999
zip_range_1356,city_br_1593,46200-000,46204-999
zip_range_1357,city_br_1594,98290-000,98299-999
zip_range_1358,city_br_1595,39489-000,39489-999
zip_range_1359,city_br_1596,33500-000,33599-999
zip_range_1360,city_br_1597,78652-000,78654-999
zip_range_1361,city_br_1598,58535-000,58539-999
zip_range_1362,city_br_1599,37584-000,37585-999
zip_range_1363,city_br_1600,36415-000,36419-999
zip_range_1364,city_br_1601,35850-000,35857-999
zip_range_1365,city_br_1602,86320-000,86329-999
zip_range_1366,city_br_1603,38195-000,38199-999
zip_range_1367,city_br_1604,78254-000,78254-999
zip_range_1368,city_br_227,36400-000,36414-999
zip_range_1369,city_br_1605,86480-000,86489-999
zip_range_1370,city_br_1606,35240-000,35245-999
zip_range_1371,city_br_1607,37670-000,37679-999
zip_range_1372,city_br_1608,99680-000,99686-999
zip_range_1373,city_br_033,32000-001,32399-999
zip_range_1374,city_br_1609,83730-000,83749-999
zip_range_1375,city_br_1610,46620-000,46639-999
zip_range_1376,city_br_1611,37235-000,37239-999
zip_range_1377,city_br_1612,95955-000,95959-999
zip_range_1378,city_br_1613,57140-000,57149-999
zip_range_1379,city_br_1614,99528-000,99529-999
zip_range_1380,city_br_1615,39340-000,39349-999
zip_range_1381,city_br_1616,44250-000,44254-999
zip_range_1382,city_br_1617,85420-000,85422-999
zip_range_1383,city_br_1618,28540-000,28544-999
zip_range_1384,city_br_1619,13490-000,13494-999
zip_range_1385,city_br_1620,46280-000,46289-999
zip_range_1386,city_br_1621,89819-000,89819-999
zip_range_1387,city_br_1622,35780-000,35784-999
zip_range_1388,city_br_1623,37498-000,37499-999
zip_range_1389,city_br_1624,62160-000,62169-999
zip_range_1390,city_br_1625,58770-000,58774-999
zip_range_1391,city_br_1626,79460-000,79469-999
zip_range_1392,city_br_1627,47690-000,47699-999
zip_range_1393,city_br_1628,39200-000,39204-999
zip_range_1394,city_br_1629,86300-000,86309-999
zip_range_1395,city_br_1630,39710-000,39714-999
zip_range_1396,city_br_1631,16260-000,16264-999
zip_range_1397,city_br_1632,65415-000,65417-999
zip_range_1398,city_br_1633,38550-000,38569-999
zip_range_1399,city_br_1634,98735-000,98739-999
zip_range_1400,city_br_1635,98580-000,98589-999
zip_range_1401,city_br_1636,85557-000,85559-999
zip_range_1402,city_br_1637,59220-000,59224-999
zip_range_1403,city_br_301,35170-001,35176-999
zip_range_1404,city_br_1638,89840-000,89842-999
zip_range_1405,city_br_1639,59930-000,59939-999
zip_range_1406,city_br_1640,48590-000,48599-999
zip_range_1407,city_br_1641,64793-000,64794-999
zip_range_1408,city_br_1642,18745-000,18759-999
zip_range_1409,city_br_1643,89837-000,89837-999
zip_range_1410,city_br_1644,39635-000,39639-999
zip_range_1411,city_br_1645,36155-000,36156-999
zip_range_1412,city_br_1646,95726-000,95726-999
zip_range_1413,city_br_1647,79995-000,79999-999
zip_range_1414,city_br_1648,85550-000,85554-999
zip_range_1415,city_br_1649,36330-000,36334-999
zip_range_1416,city_br_1650,38990-000,39099-999
zip_range_1417,city_br_1651,37605-000,37609-999
zip_range_1418,city_br_1652,76145-000,76149-999
zip_range_1419,city_br_1653,35578-000,35579-999
zip_range_1420,city_br_1654,35345-000,35347-999
zip_range_1421,city_br_1655,88535-000,88537-999
zip_range_1422,city_br_1656,64980-000,64984-999
zip_range_1423,city_br_1657,55315-000,55319-999
zip_range_1424,city_br_1658,47650-000,47654-999
zip_range_1425,city_br_1659,55525-000,55529-999
zip_range_1426,city_br_1660,79300-001,79369-999
zip_range_1427,city_br_1661,72960-000,72974-999
zip_range_1428,city_br_1662,75680-000,75689-999
zip_range_1429,city_br_1663,13540-000,13549-999
zip_range_1430,city_br_1664,86970-000,86974-999
zip_range_1431,city_br_1665,76995-000,76996-999
zip_range_1432,city_br_1666,89278-000,89279-999
zip_range_1433,city_br_1667,57230-000,57239-999
zip_range_1434,city_br_1668,13150-000,13159-999
zip_range_1435,city_br_1669,15530-000,15539-999
zip_range_1436,city_br_1670,76937-000,76939-999
zip_range_1437,city_br_1671,79550-000,79555-999
zip_range_1438,city_br_1672,47900-000,47939-999
zip_range_1439,city_br_100,06700-001,06729-999
zip_range_1440,city_br_1673,95335-000,95339-999
zip_range_1441,city_br_1674,78330-000,78334-999
zip_range_1442,city_br_1675,39188-000,39189-999
zip_range_1443,city_br_1676,77750-000,77752-999
zip_range_1444,city_br_1677,99145-000,99149-999
zip_range_1445,city_br_1678,79400-000,79409-999
zip_range_1446,city_br_1679,58588-000,58589-999
zip_range_1447,city_br_1680,57320-000,57324-999
zip_range_1448,city_br_1681,63700-000,63739-999
zip_range_1449,city_br_229,63100-001,63139-999
zip_range_1450,city_br_1682,14140-000,14149-999
zip_range_1451,city_br_1683,45330-000,45339-999
zip_range_1452,city_br_145,88800-001,88819-999
zip_range_1453,city_br_1684,39885-000,39889-999
zip_range_1454,city_br_1685,48480-000,48484-999
zip_range_1455,city_br_1686,98640-000,98669-999
zip_range_1456,city_br_1687,37275-000,37277-999
zip_range_1457,city_br_1688,14460-000,14469-999
zip_range_1458,city_br_1689,96195-000,96199-999
zip_range_1459,city_br_1690,98368-000,98369-999
zip_range_1460,city_br_1691,77490-000,77492-999
zip_range_1461,city_br_1692,64995-000,64999-999
zip_range_1462,city_br_1693,39598-000,39599-999
zip_range_1463,city_br_1694,73850-000,73859-999
zip_range_1464,city_br_1695,36426-000,36427-999
zip_range_1465,city_br_1696,75230-000,75239-999
zip_range_1466,city_br_1697,37476-000,37477-999
zip_range_1467,city_br_1698,49270-000,49279-999
zip_range_1468,city_br_1699,64920-000,64922-999
zip_range_1469,city_br_1700,47950-000,47959-999
zip_range_1470,city_br_1701,76510-000,76514-999
zip_range_1471,city_br_1702,77463-000,77464-999
zip_range_1472,city_br_1703,62390-000,62399-999
zip_range_1473,city_br_1704,75635-000,75639-999
zip_range_1474,city_br_1705,35478-000,35479-999
zip_range_1475,city_br_1706,62595-000,62597-999
zip_range_1476,city_br_1707,98000-001,98059-999
zip_range_1477,city_br_1708,44380-000,44399-999
zip_range_1478,city_br_1709,58337-000,58337-999
zip_range_1479,city_br_1710,84620-000,84629-999
zip_range_1480,city_br_1711,19860-000,19864-999
zip_range_1481,city_br_1712,99665-000,99669-999
zip_range_1482,city_br_1713,12700-001,12759-999
zip_range_1483,city_br_1714,38735-000,38739-999
zip_range_1484,city_br_1715,85598-000,85599-999
zip_range_1485,city_br_1716,87400-000,87429-999
zip_range_1486,city_br_1717,69980-000,69981-999
zip_range_1487,city_br_1718,87650-000,87659-999
zip_range_1488,city_br_1719,95930-000,95932-999
zip_range_1489,city_br_1720,59375-000,59377-999
zip_range_1490,city_br_1721,37445-000,37446-999
zip_range_1491,city_br_1722,86855-000,86859-999
zip_range_1492,city_br_278,11500-001,11599-999
zip_range_1493,city_br_1723,58167-000,58169-999
zip_range_1494,city_br_031,78000-001,78109-999
zip_range_1495,city_br_1724,58175-000,58176-999
zip_range_1496,city_br_1725,58289-000,58290-999
zip_range_1497,city_br_1726,58208-000,58209-999
zip_range_1498,city_br_1727,76864-000,76865-999
zip_range_1499,city_br_1728,75760-000,75769-999
zip_range_1500,city_br_1729,55655-000,55659-999
zip_range_1501,city_br_1730,68398-000,68399-999
zip_range_1502,city_br_1731,49660-000,49669-999
zip_range_1503,city_br_1732,12530-000,12569-999
zip_range_1504,city_br_1733,89890-000,89890-999
zip_range_1505,city_br_1734,89886-000,89886-999
zip_range_1506,city_br_1735,35246-000,35247-999
zip_range_1507,city_br_1736,55460-000,55469-999
zip_range_1508,city_br_1737,48930-000,48949-999
zip_range_1509,city_br_1738,64960-000,64962-999
zip_range_1510,city_br_1739,68523-000,68523-999
zip_range_1511,city_br_008,80000-001,82999-999
zip_range_1512,city_br_1740,89520-000,89529-999
zip_range_1513,city_br_1741,84280-000,84284-999
zip_range_1514,city_br_1742,64905-000,64909-999
zip_range_1515,city_br_1743,59380-000,59389-999
zip_range_1516,city_br_1744,58291-000,58291-999
zip_range_1517,city_br_1745,39569-000,39569-999
zip_range_1518,city_br_1746,64595-000,64599-999
zip_range_1519,city_br_1747,58990-000,58992-999
zip_range_1520,city_br_1748,68815-000,68819-999
zip_range_1521,city_br_1749,64453-000,64454-999
zip_range_1522,city_br_1750,68210-000,68219-999
zip_range_1523,city_br_1751,68750-000,68759-999
zip_range_1524,city_br_1752,65268-000,65268-999
zip_range_1525,city_br_1753,78237-000,78239-999
zip_range_1526,city_br_1754,35789-000,35799-999
zip_range_1527,city_br_1755,56640-000,56669-999
zip_range_1528,city_br_1756,68973-000,68975-999
zip_range_1529,city_br_1757,73980-000,73989-999
zip_range_1530,city_br_1758,58173-000,58174-999
zip_range_1531,city_br_1759,75420-000,75429-999
zip_range_1532,city_br_1760,77910-000,77912-999
zip_range_1533,city_br_1761,45590-000,45599-999
zip_range_1534,city_br_1762,39130-000,39134-999
zip_range_1535,city_br_1763,99980-000,99989-999
zip_range_1536,city_br_1764,75730-000,75739-999
zip_range_1537,city_br_1765,65927-000,65927-999
zip_range_1538,city_br_1766,37514-000,37515-999
zip_range_1539,city_br_1767,37910-000,37919-999
zip_range_1540,city_br_1768,57480-000,57489-999
zip_range_1541,city_br_1769,38108-000,38109-999
zip_range_1542,city_br_1770,64390-000,64394-999
zip_range_1543,city_br_1771,78380-000,78389-999
zip_range_1544,city_br_1772,79790-000,79799-999
zip_range_1545,city_br_1773,63645-000,63649-999
zip_range_1546,city_br_1774,98528-000,98529-999
zip_range_1547,city_br_1775,13690-000,13699-999
zip_range_1548,city_br_1776,89910-000,89914-999
zip_range_1549,city_br_1777,36690-000,36699-999
zip_range_1550,city_br_1778,58695-000,58697-999
zip_range_1551,city_br_1779,35492-000,35494-999
zip_range_1552,city_br_1780,36210-000,36211-999
zip_range_1553,city_br_1781,97845-000,97849-999
zip_range_1554,city_br_063,09900-001,09999-999
zip_range_1555,city_br_1782,58994-000,58994-999
zip_range_1556,city_br_1783,85896-000,85897-999
zip_range_1557,city_br_1784,87990-000,87999-999
zip_range_1558,city_br_1785,85408-000,85409-999
zip_range_1559,city_br_1786,39100-000,39119-999
zip_range_1560,city_br_1787,78400-000,78409-999
zip_range_1561,city_br_1788,77300-000,77302-999
zip_range_1562,city_br_1789,42850-000,43699-999
zip_range_1563,city_br_1790,97180-000,97184-999
zip_range_1564,city_br_1791,35437-000,35437-999
zip_range_1565,city_br_1792,35984-000,35985-999
zip_range_1566,city_br_1793,89950-000,89969-999
zip_range_1567,city_br_1794,76260-000,76264-999
zip_range_1568,city_br_1795,15715-000,15717-999
zip_range_1569,city_br_1796,64785-000,64787-999
zip_range_1570,city_br_1797,49650-000,49659-999
zip_range_1571,city_br_1798,36546-000,36549-999
zip_range_1572,city_br_1799,36820-000,36827-999
zip_range_1573,city_br_1800,35265-000,35269-999
zip_range_1574,city_br_1801,29590-000,29599-999
zip_range_1575,city_br_1802,13780-000,13789-999
zip_range_1576,city_br_1803,39735-000,39739-999
zip_range_1577,city_br_131,35500-001,35516-999
zip_range_1578,city_br_1804,73865-000,73869-999
zip_range_1579,city_br_1805,77670-000,77672-999
zip_range_1580,city_br_1806,39995-000,39997-999
zip_range_1581,city_br_1807,37142-000,37142-999
zip_range_1582,city_br_1808,39912-000,39914-999
zip_range_1583,city_br_1809,15980-000,15989-999
zip_range_1584,city_br_1810,17300-000,17319-999
zip_range_1585,city_br_1811,93950-000,93989-999
zip_range_1586,city_br_1812,98385-000,98389-999
zip_range_1587,city_br_1813,79215-000,79219-999
zip_range_1588,city_br_1814,77685-000,77689-999
zip_range_1589,city_br_1815,99220-000,99239-999
zip_range_1590,city_br_1816,57560-000,57569-999
zip_range_1591,city_br_1817,85660-000,85669-999
zip_range_1592,city_br_1818,15740-000,15744-999
zip_range_1593,city_br_1819,78830-000,78834-999
zip_range_1594,city_br_1820,46165-000,46169-999
zip_range_1595,city_br_1821,38654-000,38657-999
zip_range_1596,city_br_1822,35148-000,35149-999
zip_range_1597,city_br_1823,68633-000,68634-999
zip_range_1598,city_br_1824,64620-000,64624-999
zip_range_1599,city_br_1825,96190-000,96192-999
zip_range_1600,city_br_1826,64790-000,64792-999
zip_range_1601,city_br_1827,35865-000,35874-999
zip_range_1602,city_br_1828,44560-000,44564-999
zip_range_1603,city_br_1829,96450-000,96459-999
zip_range_1604,city_br_1830,65765-000,65767-999
zip_range_1605,city_br_1831,95568-000,95571-999
zip_range_1606,city_br_1832,35440-000,35440-999
zip_range_1607,city_br_1833,37474-000,37475-999
zip_range_1608,city_br_1834,29260-000,29279-999
zip_range_1609,city_br_1835,64250-000,64252-999
zip_range_1610,city_br_1836,89155-000,89156-999
zip_range_1611,city_br_1837,36784-000,36787-999
zip_range_1612,city_br_1838,97280-000,97299-999
zip_range_1613,city_br_1839,58228-000,58229-999
zip_range_1614,city_br_1840,36213-000,36214-999
zip_range_1615,city_br_1841,35894-000,35899-999
zip_range_1616,city_br_1842,35610-000,35612-999
zip_range_1617,city_br_1843,29580-000,29589-999
zip_range_1618,city_br_1844,36513-000,36514-999
zip_range_1619,city_br_1845,37926-000,37926-999
zip_range_1620,city_br_1846,56355-000,56359-999
zip_range_1621,city_br_1847,79880-000,79889-999
zip_range_1622,city_br_1848,87485-000,87489-999
zip_range_1623,city_br_1849,13590-000,13599-999
zip_range_1624,city_br_1850,38530-000,38539-999
zip_range_1625,city_br_120,79800-000,79849-999
zip_range_1626,city_br_1851,87155-000,87159-999
zip_range_1627,city_br_1852,98925-000,98929-999
zip_range_1628,city_br_1853,89126-000,89127-999
zip_range_1629,city_br_1854,95967-000,95969-999
zip_range_1630,city_br_1855,59910-000,59919-999
zip_range_1631,city_br_1856,83590-000,83599-999
zip_range_1632,city_br_1857,75855-000,75859-999
zip_range_1633,city_br_1858,17900-000,17919-999
zip_range_1634,city_br_1859,17470-000,17474-999
zip_range_1635,city_br_1860,28650-000,28659-999
zip_range_1636,city_br_1861,58265-000,58267-999
zip_range_1637,city_br_1862,77485-000,77489-999
zip_range_1638,city_br_1863,14120-000,14139-999
zip_range_1639,city_br_1864,65625-000,65629-999
zip_range_1640,city_br_022,25000-001,25499-999
zip_range_1641,city_br_1865,36974-000,36975-999
zip_range_1642,city_br_1866,19830-000,19839-999
zip_range_1643,city_br_1867,29850-000,29879-999
zip_range_1644,city_br_1868,75945-000,75949-999
zip_range_1645,city_br_1869,75940-000,75944-999
zip_range_1646,city_br_1870,69880-000,69889-999
zip_range_1647,city_br_1871,79970-000,79974-999
zip_range_1648,city_br_1872,11960-000,11989-999
zip_range_1649,city_br_1873,68524-000,68524-999
zip_range_1650,city_br_1874,92990-000,92999-999
zip_range_1651,city_br_1875,64325-000,64329-999
zip_range_1652,city_br_1876,13350-000,13359-999
zip_range_1653,city_br_1877,64880-000,64884-999
zip_range_1654,city_br_1878,15823-000,15824-999
zip_range_1655,city_br_1879,45305-000,45309-999
zip_range_1656,city_br_1880,37110-000,37114-999
zip_range_1657,city_br_1881,58763-000,58764-999
zip_range_1658,city_br_1882,15425-000,15429-999
zip_range_1659,city_br_116,06800-001,06849-999
zip_range_1660,city_br_1883,06900-000,06949-999
zip_range_1661,city_br_1884,19350-000,19359-999
zip_range_1662,city_br_1885,95960-000,95964-999
zip_range_1663,city_br_1886,59905-000,59907-999
zip_range_1664,city_br_1887,45150-000,45154-999
zip_range_1665,city_br_1888,96610-000,96634-999
zip_range_1666,city_br_1889,85630-000,85634-999
zip_range_1667,city_br_1890,87270-000,87279-999
zip_range_1668,city_br_1891,35130-000,35134-999
zip_range_1669,city_br_1892,13165-000,13169-999
zip_range_1670,city_br_1893,39363-000,39364-999
zip_range_1671,city_br_1894,26650-000,26699-999
zip_range_1672,city_br_1895,99698-000,99699-999
zip_range_1673,city_br_1896,35324-000,35324-999
zip_range_1674,city_br_1897,98855-000,98859-999
zip_range_1675,city_br_1898,48180-000,48279-999
zip_range_1676,city_br_1899,89862-000,89864-999
zip_range_1677,city_br_1900,35490-000,35491-999
zip_range_1678,city_br_1901,85988-000,85989-999
zip_range_1679,city_br_1902,99645-000,99649-999
zip_range_1680,city_br_1903,69870-000,69879-999
zip_range_1681,city_br_1904,69934-000,69934-999
zip_range_1682,city_br_1905,59355-000,59359-999
zip_range_1683,city_br_1906,99920-000,99924-999
zip_range_1684,city_br_295,99700-001,99717-999
zip_range_1685,city_br_1907,63470-000,63474-999
zip_range_1686,city_br_1908,46180-000,46189-999
zip_range_1687,city_br_1909,88935-000,88939-999
zip_range_1688,city_br_1910,99140-000,99144-999
zip_range_1689,city_br_1911,99750-000,99759-999
zip_range_1690,city_br_1912,98390-000,98399-999
zip_range_1691,city_br_1913,89613-000,89617-999
zip_range_1692,city_br_1914,36555-000,36559-999
zip_range_1693,city_br_1915,55500-000,55509-999
zip_range_1694,city_br_1916,95380-000,95389-999
zip_range_1695,city_br_1917,35740-000,35759-999
zip_range_1696,city_br_1918,36830-000,36831-999
zip_range_1697,city_br_1919,58135-000,58139-999
zip_range_1698,city_br_1920,98635-000,98639-999
zip_range_1699,city_br_1921,87545-000,87549-999
zip_range_1700,city_br_1922,64180-000,64189-999
zip_range_1701,city_br_1923,77993-000,77994-999
zip_range_1702,city_br_1924,65750-000,65752-999
zip_range_1703,city_br_1925,85465-000,85469-999
zip_range_1704,city_br_1926,76974-000,76975-999
zip_range_1705,city_br_1927,39510-000,39515-999
zip_range_1706,city_br_1928,59180-000,59181-999
zip_range_1707,city_br_1929,37566-000,37566-999
zip_range_1708,city_br_1930,13990-000,13994-999
zip_range_1709,city_br_1931,18935-000,18939-999
zip_range_1710,city_br_1932,48370-000,48389-999
zip_range_1711,city_br_1933,99400-000,99429-999
zip_range_1712,city_br_1934,99930-000,99939-999
zip_range_1713,city_br_1935,49200-000,49219-999
zip_range_1714,city_br_1936,93600-001,93699-999
zip_range_1715,city_br_1937,93250-001,93299-999
zip_range_1716,city_br_1938,37542-000,37544-999
zip_range_1717,city_br_1939,13857-000,13859-999
zip_range_1718,city_br_1940,65975-000,65977-999
zip_range_1719,city_br_1941,95880-000,95884-999
zip_range_1720,city_br_1942,15650-000,15669-999
zip_range_1721,city_br_1943,36725-000,36729-999
zip_range_1722,city_br_1944,57625-000,57629-999
zip_range_1723,city_br_1945,35613-000,35616-999
zip_range_1724,city_br_1946,76485-000,76489-999
zip_range_1725,city_br_1947,19230-000,19249-999
zip_range_1726,city_br_1948,38525-000,38529-999
zip_range_1727,city_br_1949,96990-000,96999-999
zip_range_1728,city_br_1950,48500-000,48519-999
zip_range_1729,city_br_1951,19275-000,19279-999
zip_range_1730,city_br_1952,98860-000,98864-999
zip_range_1731,city_br_1953,36855-000,36859-999
zip_range_1732,city_br_274,45820-001,45833-999
zip_range_1733,city_br_1954,61760-000,61799-999
zip_range_1734,city_br_1955,36108-000,36109-999
zip_range_1735,city_br_1956,37640-000,37649-999
zip_range_1736,city_br_1957,59575-000,59577-999
zip_range_1737,city_br_1958,56230-000,56249-999
zip_range_1738,city_br_1959,58487-000,58488-999
zip_range_1739,city_br_1960,95333-000,95333-999
zip_range_1740,city_br_1961,76740-000,76759-999
zip_range_1741,city_br_1962,37144-000,37144-999
zip_range_1742,city_br_1963,36840-000,36843-999
zip_range_1743,city_br_1964,63185-000,63189-999
zip_range_1744,city_br_1965,68280-000,68284-999
zip_range_1745,city_br_1966,87325-000,87329-999
zip_range_1746,city_br_1967,95180-000,95184-999
zip_range_1747,city_br_1968,18870-000,18889-999
zip_range_1748,city_br_1969,64788-000,64789-999
zip_range_1749,city_br_1970,48415-000,48419-999
zip_range_1750,city_br_1971,77555-000,77557-999
zip_range_1751,city_br_1972,79700-000,79709-999
zip_range_1752,city_br_1973,86840-000,86844-999
zip_range_1753,city_br_1974,97220-000,97229-999
zip_range_1754,city_br_1975,89694-000,89699-999
zip_range_1755,city_br_1976,99655-000,99659-999
zip_range_1756,city_br_1977,76220-000,76229-999
zip_range_1757,city_br_202,83820-001,83839-999
zip_range_1758,city_br_1978,95875-000,95879-999
zip_range_1759,city_br_1979,69960-000,69969-999
zip_range_1760,city_br_1980,46446-000,46449-999
zip_range_1761,city_br_035,44000-001,44149-999
zip_range_1762,city_br_1981,57340-000,57349-999
zip_range_1763,city_br_1982,55715-000,55719-999
zip_range_1764,city_br_1983,49670-000,49679-999
zip_range_1765,city_br_1984,65995-000,65999-999
zip_range_1766,city_br_1985,39180-000,39184-999
zip_range_1767,city_br_1986,59795-000,59799-999
zip_range_1768,city_br_1987,39895-000,39899-999
zip_range_1769,city_br_1988,39237-000,39239-999
zip_range_1770,city_br_1989,95770-000,95772-999
zip_range_1771,city_br_1990,57220-000,57229-999
zip_range_1772,city_br_1991,78885-000,78886-999
zip_range_1773,city_br_1992,86950-000,86959-999
zip_range_1774,city_br_1993,84535-000,84549-999
zip_range_1775,city_br_1994,35135-000,35137-999
zip_range_1776,city_br_1995,53990-000,53999-999
zip_range_1777,city_br_1996,65964-000,65967-999
zip_range_1778,city_br_1997,59517-000,59517-999
zip_range_1779,city_br_1998,15940-000,15949-999
zip_range_1780,city_br_1999,15600-000,15619-999
zip_range_1781,city_br_2000,17455-000,17469-999
zip_range_1782,city_br_166,08500-001,08549-999
zip_range_1783,city_br_2001,68915-000,68917-999
zip_range_1784,city_br_2002,55880-000,55889-999
zip_range_1785,city_br_2003,35800-000,35809-999
zip_range_1786,city_br_2004,36815-000,36819-999
zip_range_1787,city_br_2005,84285-000,84289-999
zip_range_1788,city_br_2006,79428-000,79429-999
zip_range_1789,city_br_2007,77465-000,77469-999
zip_range_1790,city_br_2008,78290-000,78292-999
zip_range_1791,city_br_2009,44775-000,44779-999
zip_range_1792,city_br_2010,77795-000,77797-999
zip_range_1793,city_br_2011,45720-000,45724-999
zip_range_1794,city_br_2012,76105-000,76109-999
zip_range_1795,city_br_2013,57995-000,57999-999
zip_range_1796,city_br_2014,85618-000,85619-999
zip_range_1797,city_br_2015,89878-000,89878-999
zip_range_1798,city_br_2016,17870-000,17879-999
zip_range_1799,city_br_2017,87185-000,87189-999
zip_range_1800,city_br_2018,59335-000,59337-999
zip_range_1801,city_br_2019,15320-000,15329-999
zip_range_1802,city_br_2020,56850-000,56869-999
zip_range_1803,city_br_2021,95270-000,95274-999
zip_range_1804,city_br_2022,73890-000,73899-999
zip_range_1805,city_br_2023,64815-000,64819-999
zip_range_1806,city_br_2024,56400-000,56419-999
zip_range_1807,city_br_2025,87120-000,87129-999
zip_range_1808,city_br_2026,45740-000,45744-999
zip_range_1809,city_br_2027,68543-000,68544-999
zip_range_1810,city_br_2028,64563-000,64564-999
zip_range_1811,city_br_2029,35690-000,35693-999
zip_range_1812,city_br_2030,86165-000,86169-999
zip_range_1813,city_br_2031,64800-001,64814-999
zip_range_1814,city_br_2032,99910-000,99919-999
zip_range_1815,city_br_039,88000-001,88099-999
zip_range_1816,city_br_2033,86780-000,86789-999
zip_range_1817,city_br_2034,17830-000,17859-999
zip_range_1818,city_br_2035,19870-000,19879-999
zip_range_1819,city_br_2036,69670-000,69679-999
zip_range_1820,city_br_2037,99370-000,99379-999
zip_range_1821,city_br_2038,35570-000,35577-999
zip_range_1822,city_br_2039,97210-000,97219-999
zip_range_1823,city_br_268,73800-001,73819-999
zip_range_1824,city_br_2040,65943-000,65944-999
zip_range_1825,city_br_2041,85830-000,85832-999
zip_range_1826,city_br_2042,47990-000,47999-999
zip_range_1827,city_br_2043,89859-000,89859-999
zip_range_1828,city_br_2044,76470-000,76479-999
zip_range_1829,city_br_2045,38690-000,38699-999
zip_range_1830,city_br_2046,77470-000,77474-999
zip_range_1831,city_br_2047,95937-000,95939-999
zip_range_1832,city_br_2048,62115-000,62119-999
zip_range_1833,city_br_2049,88850-000,88859-999
zip_range_1834,city_br_004,60000-001,61599-999
zip_range_1835,city_br_2050,37905-000,37909-999
zip_range_1836,city_br_2051,77708-000,77709-999
zip_range_1837,city_br_2052,65805-000,65807-999
zip_range_1838,city_br_2053,98125-000,98129-999
zip_range_1839,city_br_2054,62815-000,62819-999
zip_range_1840,city_br_2055,65695-000,65699-999
zip_range_1841,city_br_2056,35760-000,35762-999
zip_range_1842,city_br_097,85850-001,85874-999
zip_range_1843,city_br_2057,85145-000,85147-999
zip_range_1844,city_br_2058,89580-000,89589-999
zip_range_1845,city_br_075,14400-001,14414-999
zip_range_1846,city_br_2059,64520-000,64524-999
zip_range_1847,city_br_2060,87570-000,87579-999
zip_range_1848,city_br_2061,64475-000,64479-999
zip_range_1849,city_br_2062,39644-000,39644-999
zip_range_1850,city_br_2063,85600-001,85609-999
zip_range_1851,city_br_2064,59902-000,59904-999
zip_range_1852,city_br_2065,39387-000,39389-999
zip_range_1853,city_br_2066,64683-000,64684-999
zip_range_1854,city_br_180,07900-001,07999-999
zip_range_1855,city_br_2067,39580-000,39589-999
zip_range_1856,city_br_2068,64645-000,64649-999
zip_range_1857,city_br_2069,39695-000,39699-999
zip_range_1858,city_br_209,07800-001,07899-999
zip_range_1859,city_br_2070,62340-000,62349-999
zip_range_1860,city_br_2071,98400-000,98409-999
zip_range_1861,city_br_2072,39840-000,39847-999
zip_range_1862,city_br_2073,35112-000,35112-999
zip_range_1863,city_br_2074,39708-000,39709-999
zip_range_1864,city_br_2075,58195-000,58199-999
zip_range_1865,city_br_2076,55780-000,55789-999
zip_range_1866,city_br_2077,49514-000,49516-999
zip_range_1867,city_br_2078,89530-000,89532-999
zip_range_1868,city_br_2079,38230-000,38239-999
zip_range_1869,city_br_2080,39870-000,39872-999
zip_range_1870,city_br_2081,64690-000,64694-999
zip_range_1871,city_br_2082,39558-000,39559-999
zip_range_1872,city_br_2083,38200-000,38209-999
zip_range_1873,city_br_2084,59890-000,59899-999
zip_range_1874,city_br_2085,29185-000,29189-999
zip_range_1875,city_br_2086,35736-000,35737-999
zip_range_1876,city_br_2087,16220-000,16229-999
zip_range_1877,city_br_2088,58492-000,58493-999
zip_range_1878,city_br_2089,17450-000,17454-999
zip_range_1879,city_br_2090,35250-000,35257-999
zip_range_1880,city_br_2091,59596-000,59597-999
zip_range_1881,city_br_2092,89838-000,89838-999
zip_range_1882,city_br_2093,55530-000,55534-999
zip_range_1883,city_br_2094,75184-000,75184-999
zip_range_1884,city_br_2095,39505-000,39507-999
zip_range_1885,city_br_2096,45450-000,45451-999
zip_range_1886,city_br_210,55290-001,55304-999
zip_range_1887,city_br_2097,49830-000,49859-999
zip_range_1888,city_br_2098,17400-000,17409-999
zip_range_1889,city_br_2099,95720-000,95725-999
zip_range_1890,city_br_2100,88495-000,88499-999
zip_range_1891,city_br_2101,68665-000,68669-999
zip_range_1892,city_br_2102,97690-000,97699-999
zip_range_1893,city_br_2103,89248-000,89248-999
zip_range_1894,city_br_2104,89110-001,89119-999
zip_range_1895,city_br_2105,15330-000,15339-999
zip_range_1896,city_br_2106,78875-000,78879-999
zip_range_1897,city_br_2107,99830-000,99834-999
zip_range_1898,city_br_2108,44650-000,44654-999
zip_range_1899,city_br_2109,14813-000,14814-999
zip_range_1900,city_br_2110,64613-000,64614-999
zip_range_1901,city_br_2111,95820-000,95832-999
zip_range_1902,city_br_2112,78620-000,78624-999
zip_range_1903,city_br_2113,84660-000,84899-999
zip_range_1904,city_br_2114,49750-000,49759-999
zip_range_1905,city_br_2115,15300-000,15309-999
zip_range_1906,city_br_2116,62738-000,62739-999
zip_range_1907,city_br_2117,99160-000,99164-999
zip_range_1908,city_br_2118,47450-000,47499-999
zip_range_1909,city_br_2119,16450-000,16479-999
zip_range_1910,city_br_2120,99900-000,99909-999
zip_range_1911,city_br_2121,64930-000,64939-999
zip_range_1912,city_br_2122,57360-000,57369-999
zip_range_1913,city_br_2123,98870-000,98894-999
zip_range_1914,city_br_2124,39592-000,39593-999
zip_range_1915,city_br_2125,16270-000,16289-999
zip_range_1916,city_br_2126,48620-000,48629-999
zip_range_1917,city_br_2127,78293-000,78294-999
zip_range_1918,city_br_2128,79730-000,79739-999
zip_range_1919,city_br_2129,55620-000,55629-999
zip_range_1920,city_br_2130,94380-000,94399-999
zip_range_1921,city_br_2131,65285-000,65287-999
zip_range_1922,city_br_2132,86938-000,86939-999
zip_range_1923,city_br_2133,35248-000,35248-999
zip_range_1924,city_br_2134,55900-000,55919-999
zip_range_1925,city_br_2135,36152-000,36154-999
zip_range_1926,city_br_2136,75170-000,75174-999
zip_range_1927,city_br_2137,75740-000,75749-999
zip_range_1928,city_br_2138,76380-001,76389-999
zip_range_1929,city_br_2139,68639-000,68639-999
zip_range_1930,city_br_010,74000-001,74899-999
zip_range_1931,city_br_2140,59173-000,59177-999
zip_range_1932,city_br_2141,75370-000,75374-999
zip_range_1933,city_br_2142,77695-000,77699-999
zip_range_1934,city_br_2143,76600-000,76629-999
zip_range_1935,city_br_2144,77770-000,77776-999
zip_range_1936,city_br_2145,75600-000,75602-999
zip_range_1937,city_br_2146,87360-000,87364-999
zip_range_1938,city_br_2147,85162-000,85167-999
zip_range_1939,city_br_2148,37680-000,37689-999
zip_range_1940,city_br_2149,65775-000,65779-999
zip_range_1941,city_br_2150,45540-000,45544-999
zip_range_1942,city_br_2151,39720-000,39722-999
zip_range_1943,city_br_2152,39120-000,39129-999
zip_range_1944,city_br_2153,75865-000,75869-999
zip_range_1945,city_br_2154,65770-000,65774-999
zip_range_1946,city_br_2155,88190-000,88199-999
zip_range_1947,city_br_2156,59790-000,59794-999
zip_range_1948,city_br_2157,65928-000,65928-999
zip_range_1949,city_br_2158,65780-000,65782-999
zip_range_1950,city_br_2159,76898-000,76899-999
zip_range_1951,city_br_2160,29720-000,29724-999
zip_range_1952,city_br_2161,65795-000,65799-999
zip_range_1953,city_br_2162,44350-000,44359-999
zip_range_1954,city_br_2163,65363-000,65364-999
zip_range_1955,city_br_2164,65284-000,65284-999
zip_range_1956,city_br_111,35000-001,35109-999
zip_range_1957,city_br_2165,62365-000,62369-999
zip_range_1958,city_br_2166,65785-000,65789-999
zip_range_1959,city_br_2167,49860-000,49869-999
zip_range_1960,city_br_2168,65940-000,65942-999
zip_range_1961,city_br_2169,95670-000,95679-999
zip_range_1962,city_br_2170,99605-000,99609-999
zip_range_1963,city_br_2171,96875-000,96877-999
zip_range_1964,city_br_2172,86845-000,86847-999
zip_range_1965,city_br_2173,56160-000,56162-999
zip_range_1966,city_br_2174,62430-000,62449-999
zip_range_1967,city_br_2175,63230-000,63239-999
zip_range_1968,city_br_2176,39570-000,39572-999
zip_range_1969,city_br_2177,88890-000,88899-999
zip_range_1970,city_br_2178,55640-001,55649-999
zip_range_1971,city_br_107,94000-001,94379-999
zip_range_1972,city_br_2179,88735-000,88739-999
zip_range_1973,city_br_2180,62190-000,62199-999
zip_range_1974,city_br_2181,59675-000,59677-999
zip_range_1975,city_br_2182,38470-000,38474-999
zip_range_1976,city_br_2183,95355-000,95359-999
zip_range_1977,city_br_2184,88360-000,88369-999
zip_range_1978,city_br_2185,29560-000,29579-999
zip_range_1979,city_br_2186,64840-000,64844-999
zip_range_1980,city_br_2187,92500-000,92849-999
zip_range_1981,city_br_2188,16430-000,16439-999
zip_range_1982,city_br_2189,16480-000,16499-999
zip_range_1983,city_br_2190,85980-000,85987-999
zip_range_1984,city_br_2191,14790-000,14794-999
zip_range_1985,city_br_2192,87880-000,87889-999
zip_range_1986,city_br_2193,61890-000,61899-999
zip_range_1987,city_br_2194,69895-000,69899-999
zip_range_1988,city_br_2195,76850-000,76856-999
zip_range_1989,city_br_2196,46205-000,46219-999
zip_range_1990,city_br_2197,59598-000,59599-999
zip_range_1991,city_br_2198,84435-000,84449-999
zip_range_1992,city_br_2199,46430-000,46437-999
zip_range_1993,city_br_2200,39740-000,39744-999
zip_range_1994,city_br_2201,37177-000,37189-999
zip_range_1995,city_br_2202,15110-000,15114-999
zip_range_1996,city_br_2203,18310-000,18314-999
zip_range_1997,city_br_2204,25940-001,25949-999
zip_range_1998,city_br_2205,86465-000,86469-999
zip_range_1999,city_br_2206,75350-000,75354-999
zip_range_2000,city_br_2207,99200-000,99214-999
zip_range_2001,city_br_2208,87810-000,87819-999
zip_range_2002,city_br_2209,14580-000,14599-999
zip_range_2003,city_br_2210,58200-000,58207-999
zip_range_2004,city_br_2211,16980-000,16999-999
zip_range_2005,city_br_2212,86620-000,86629-999
zip_range_2006,city_br_2213,15420-000,15424-999
zip_range_2007,city_br_2214,35436-000,35436-999
zip_range_2008,city_br_2215,89920-000,89924-999
zip_range_2009,city_br_2216,62380-000,62389-999
zip_range_2010,city_br_2217,39397-000,39397-999
zip_range_2011,city_br_2218,77700-000,77703-999
zip_range_2012,city_br_2219,76690-000,76699-999
zip_range_2013,city_br_2220,62766-000,62769-999
zip_range_2014,city_br_2221,89270-000,89274-999
zip_range_2015,city_br_2222,37810-000,37819-999
zip_range_2016,city_br_2223,36160-000,36164-999
zip_range_2017,city_br_2224,15680-000,15684-999
zip_range_2018,city_br_2225,97950-000,97959-999
zip_range_2019,city_br_2226,73910-000,73919-999
zip_range_2020,city_br_2227,85400-000,85407-999
zip_range_2021,city_br_2228,16570-000,16599-999
zip_range_2022,city_br_2229,78520-000,78524-999
zip_range_2023,city_br_243,29200-001,29229-999
zip_range_2024,city_br_165,85000-001,85139-999
zip_range_2025,city_br_2230,83390-000,83399-999
zip_range_2026,city_br_2231,36606-000,36607-999
zip_range_2027,city_br_2232,16700-000,16749-999
zip_range_2028,city_br_2233,08900-000,08939-999
zip_range_2029,city_br_2234,45840-000,45847-999
zip_range_2030,city_br_259,12500-001,12524-999
zip_range_2031,city_br_2235,83280-000,83299-999
zip_range_2032,city_br_2236,38570-000,38599-999
zip_range_2033,city_br_2237,18250-000,18254-999
zip_range_2034,city_br_2238,14840-000,14849-999
zip_range_2035,city_br_2239,64798-000,64799-999
zip_range_2036,city_br_2240,76374-000,76374-999
zip_range_2037,city_br_095,11400-001,11499-999
zip_range_2038,city_br_2241,89940-000,89949-999
zip_range_2039,city_br_013,07000-001,07399-999
zip_range_2040,city_br_2242,89817-000,89817-999
zip_range_2041,city_br_2243,14115-000,14119-999
zip_range_2042,city_br_2244,37800-000,37804-999
zip_range_2043,city_br_2245,79230-000,79239-999
zip_range_2044,city_br_2246,36515-000,36519-999
zip_range_2045,city_br_2247,65255-000,65259-999
zip_range_2046,city_br_2248,38730-000,38734-999
zip_range_2047,city_br_2249,78760-000,78769-999
zip_range_2048,city_br_2250,36525-000,36529-999
zip_range_2049,city_br_2251,38310-000,38319-999
zip_range_2050,city_br_2252,58356-000,58359-999
zip_range_2051,city_br_2253,58670-000,58674-999
zip_range_2052,city_br_2254,68300-000,68329-999
zip_range_2053,city_br_2255,77400-001,77449-999
zip_range_2054,city_br_2256,15355-000,15359-999
zip_range_2055,city_br_2257,95785-000,95789-999
zip_range_2056,city_br_2258,76670-000,76679-999
zip_range_2057,city_br_2259,37484-000,37484-999
zip_range_2058,city_br_2260,48445-000,48449-999
zip_range_2059,city_br_2261,17650-000,17669-999
zip_range_2060,city_br_2262,96310-000,96329-999
zip_range_2061,city_br_2263,89610-000,89612-999
zip_range_2062,city_br_2264,96888-000,96889-999
zip_range_2063,city_br_2265,62270-000,62279-999
zip_range_2064,city_br_2266,75340-000,75344-999
zip_range_2065,city_br_2267,76375-000,76379-999
zip_range_2066,city_br_2268,13825-000,13829-999
zip_range_2067,city_br_2269,85548-000,85549-999
zip_range_2068,city_br_2270,62880-001,62899-999
zip_range_2069,city_br_2271,98920-000,98924-999
zip_range_2070,city_br_125,13183-001,13189-999
zip_range_2071,city_br_2272,64470-000,64474-999
zip_range_2072,city_br_2273,96460-000,96469-999
zip_range_2073,city_br_2274,69800-000,69819-999
zip_range_2074,city_br_2275,98670-000,98674-999
zip_range_2075,city_br_2276,65180-000,65189-999
zip_range_2076,city_br_2277,17180-000,17189-999
zip_range_2077,city_br_2278,73920-000,73929-999
zip_range_2078,city_br_2279,17680-000,17689-999
zip_range_2079,city_br_2280,46860-000,46874-999
zip_range_2080,city_br_2281,35190-000,35192-999
zip_range_2081,city_br_2282,18775-000,18779-999
zip_range_2082,city_br_2283,55345-000,55349-999
zip_range_2083,city_br_2284,84900-000,84919-999
zip_range_2084,city_br_2285,96925-000,96929-999
zip_range_2085,city_br_2286,63970-000,63999-999
zip_range_2086,city_br_2287,14815-000,14819-999
zip_range_2087,city_br_2288,57890-000,57899-999
zip_range_2088,city_br_2289,29395-000,29397-999
zip_range_2089,city_br_2290,85478-000,85484-999
zip_range_2090,city_br_2291,36225-000,36226-999
zip_range_2091,city_br_2292,38950-000,38959-999
zip_range_2092,city_br_2293,99940-000,99949-999
zip_range_2093,city_br_2294,39350-000,39354-999
zip_range_2094,city_br_2295,89652-000,89653-999
zip_range_2095,city_br_2296,62360-000,62364-999
zip_range_2096,city_br_2297,58980-000,58984-999
zip_range_2097,city_br_2298,46390-000,46399-999
zip_range_2098,city_br_2299,45745-000,45749-999
zip_range_2099,city_br_2300,89640-000,89641-999
zip_range_2100,city_br_2301,46760-000,46764-999
zip_range_2101,city_br_2302,45290-000,45299-999
zip_range_2102,city_br_2303,62955-000,62959-999
zip_range_2103,city_br_2304,56580-000,56599-999
zip_range_2104,city_br_2305,44970-000,44989-999
zip_range_2105,city_br_2306,46540-000,46549-999
zip_range_2106,city_br_2307,86200-000,86209-999
zip_range_2107,city_br_2308,46840-000,46849-999
zip_range_2108,city_br_2309,15860-000,15869-999
zip_range_2109,city_br_2310,39455-000,39457-999
zip_range_2110,city_br_2311,37990-000,37992-999
zip_range_2111,city_br_2312,29670-000,29679-999
zip_range_2112,city_br_2313,95305-000,95307-999
zip_range_2113,city_br_2314,55390-000,55394-999
zip_range_2114,city_br_2315,89140-000,89144-999
zip_range_2115,city_br_2316,45500-000,45519-999
zip_range_2116,city_br_2317,45940-000,45949-999
zip_range_2117,city_br_2318,99320-000,99329-999
zip_range_2118,city_br_2319,19940-000,19959-999
zip_range_2119,city_br_2320,45580-000,45584-999
zip_range_2120,city_br_171,32400-001,32449-999
zip_range_2121,city_br_2321,98200-000,98229-999
zip_range_2122,city_br_2322,46700-000,46729-999
zip_range_2123,city_br_2323,14940-000,14954-999
zip_range_2124,city_br_2324,29540-000,29549-999
zip_range_2125,city_br_2325,44960-000,44969-999
zip_range_2126,city_br_2326,37790-000,37794-999
zip_range_2127,city_br_2327,37223-000,37224-999
zip_range_2128,city_br_2328,18150-000,18159-999
zip_range_2129,city_br_2329,47520-000,47529-999
zip_range_2130,city_br_2330,62810-000,62814-999
zip_range_2131,city_br_2331,88820-000,88827-999
zip_range_2132,city_br_2332,39318-000,39319-999
zip_range_2133,city_br_2333,87530-000,87534-999
zip_range_2134,city_br_2334,65170-000,65179-999
zip_range_2135,city_br_2335,15460-000,15469-999
zip_range_2136,city_br_2336,48725-000,48729-999
zip_range_2137,city_br_2337,63430-000,63459-999
zip_range_2138,city_br_2338,29280-000,29284-999
zip_range_2139,city_br_2339,59490-000,59499-999
zip_range_2140,city_br_2340,19640-000,19644-999
zip_range_2141,city_br_2341,57620-000,57624-999
zip_range_2142,city_br_2342,46490-000,46499-999
zip_range_2143,city_br_2343,17350-000,17359-999
zip_range_2144,city_br_2344,58775-000,58777-999
zip_range_2145,city_br_2345,14540-000,14549-999
zip_range_2146,city_br_2346,32900-000,32919-999
zip_range_2147,city_br_2347,68725-000,68729-999
zip_range_2148,city_br_2348,65345-000,65349-999
zip_range_2149,city_br_2349,65720-000,65722-999
zip_range_2150,city_br_2350,68430-000,68439-999
zip_range_2151,city_br_270,53600-001,53689-999
zip_range_2152,city_br_2351,12350-000,12379-999
zip_range_2153,city_br_2352,35695-000,35699-999
zip_range_2154,city_br_2353,45443-000,45444-999
zip_range_2155,city_br_2354,57280-000,57289-999
zip_range_2156,city_br_2355,95650-000,95659-999
zip_range_2157,city_br_2356,28960-000,28969-999
zip_range_2158,city_br_2357,45280-000,45289-999
zip_range_2159,city_br_2358,11920-000,11924-999
zip_range_2160,city_br_2359,86750-000,86754-999
zip_range_2161,city_br_2360,56840-000,56849-999
zip_range_2162,city_br_2361,38910-000,38929-999
zip_range_2163,city_br_2362,79960-000,79964-999
zip_range_2164,city_br_2363,63500-001,63514-999
zip_range_2165,city_br_2364,85423-000,85424-999
zip_range_2166,city_br_2365,37218-000,37219-999
zip_range_2167,city_br_2366,98700-000,98732-999
zip_range_2168,city_br_2367,11925-000,11929-999
zip_range_2169,city_br_2368,49990-000,49994-999
zip_range_2170,city_br_2369,53900-000,53989-999
zip_range_2171,city_br_2370,64224-000,64224-999
zip_range_2172,city_br_2371,15385-000,15389-999
zip_range_2173,city_br_2372,11630-000,11659-999
zip_range_2174,city_br_167,45650-001,45674-999
zip_range_2175,city_br_2373,88320-000,88329-999
zip_range_2176,city_br_2374,37175-000,37176-999
zip_range_2177,city_br_2375,95990-000,95994-999
zip_range_2178,city_br_2376,58745-000,58747-999
zip_range_2179,city_br_2377,88770-000,88779-999
zip_range_2180,city_br_2378,84250-000,84259-999
zip_range_2181,city_br_2379,95625-000,95629-999
zip_range_2182,city_br_2380,35323-000,35323-999
zip_range_2183,city_br_2381,88780-000,88789-999
zip_range_2184,city_br_2382,84430-000,84434-999
zip_range_2185,city_br_2383,88440-000,88442-999
zip_range_2186,city_br_2384,95885-000,95889-999
zip_range_2187,city_br_102,65900-001,65919-999
zip_range_2188,city_br_2385,85155-000,85159-999
zip_range_2189,city_br_2386,75550-000,75554-999
zip_range_2190,city_br_2387,56560-000,56564-999
zip_range_2191,city_br_2388,87670-000,87679-999
zip_range_2192,city_br_2389,37576-000,37577-999
zip_range_2193,city_br_2390,39536-000,39537-999
zip_range_2194,city_br_2391,89130-000,89134-999
zip_range_2195,city_br_112,13330-001,13349-999
zip_range_2196,city_br_2392,63640-000,63644-999
zip_range_2197,city_br_2393,98915-000,98917-999
zip_range_2198,city_br_2394,19560-000,19569-999
zip_range_2199,city_br_2395,38490-000,38499-999
zip_range_2200,city_br_2396,87235-000,87239-999
zip_range_2201,city_br_2397,15690-000,15699-999
zip_range_2202,city_br_2398,75955-000,75959-999
zip_range_2203,city_br_2399,49250-000,49259-999
zip_range_2204,city_br_2400,78295-000,78299-999
zip_range_2205,city_br_2401,58380-000,58381-999
zip_range_2206,city_br_2402,37215-000,37217-999
zip_range_2207,city_br_2403,56830-000,56839-999
zip_range_2208,city_br_2404,98765-000,98769-999
zip_range_2209,city_br_2405,48490-000,48499-999
zip_range_2210,city_br_2406,68770-000,68772-999
zip_range_2211,city_br_2407,57545-000,57549-999
zip_range_2212,city_br_2408,35330-000,35333-999
zip_range_2213,city_br_2409,35763-000,35764-999
zip_range_2214,city_br_2410,64535-000,64539-999
zip_range_2215,city_br_2411,75400-000,75409-999
zip_range_2216,city_br_2412,39243-000,39244-999
zip_range_2217,city_br_2413,79580-000,79589-999
zip_range_2218,city_br_2414,17760-000,17779-999
zip_range_2219,city_br_2415,89558-000,89559-999
zip_range_2220,city_br_2416,35198-000,35198-999
zip_range_2221,city_br_2417,75780-000,75789-999
zip_range_2222,city_br_2418,36950-000,36952-999
zip_range_2223,city_br_2419,59508-000,59509-999
zip_range_2224,city_br_2420,62215-000,62219-999
zip_range_2225,city_br_134,35150-001,35169-999
zip_range_2226,city_br_2421,63340-000,63359-999
zip_range_2227,city_br_2422,18950-000,18959-999
zip_range_2228,city_br_2423,95240-000,95249-999
zip_range_2229,city_br_2424,44680-000,44689-999
zip_range_2230,city_br_2425,18560-000,18569-999
zip_range_2231,city_br_2426,13537-000,13539-999
zip_range_2232,city_br_2427,38350-000,38359-999
zip_range_2233,city_br_2428,45570-000,45579-999
zip_range_2234,city_br_2429,15108-000,15109-999
zip_range_2235,city_br_2430,89669-000,89669-999
zip_range_2236,city_br_2431,44600-000,44609-999
zip_range_2237,city_br_2432,84450-000,84459-999
zip_range_2238,city_br_2433,76304-000,76304-999
zip_range_2239,city_br_2434,78578-000,78578-999
zip_range_2240,city_br_2435,64540-000,64544-999
zip_range_2241,city_br_2436,99925-000,99929-999
zip_range_2242,city_br_2437,69890-000,69894-999
zip_range_2243,city_br_2438,68637-000,68637-999
zip_range_2244,city_br_2439,55590-000,55599-999
zip_range_2245,city_br_2440,76200-000,76204-999
zip_range_2246,city_br_2441,87560-000,87564-999
zip_range_2247,city_br_2442,89899-000,89899-999
zip_range_2248,city_br_2443,18330-000,18359-999
zip_range_2249,city_br_2444,62250-000,62254-999
zip_range_2250,city_br_2445,14610-000,14619-999
zip_range_2251,city_br_2446,89832-000,89833-999
zip_range_2252,city_br_2447,56260-000,56279-999
zip_range_2253,city_br_2448,59315-000,59317-999
zip_range_2254,city_br_2449,62230-000,62249-999
zip_range_2255,city_br_2450,77553-000,77554-999
zip_range_2256,city_br_2451,37588-000,37588-999
zip_range_2257,city_br_2452,89790-000,89799-999
zip_range_2258,city_br_2453,47590-000,47599-999
zip_range_2259,city_br_2454,62980-000,62989-999
zip_range_2260,city_br_2455,69348-000,69349-999
zip_range_2261,city_br_2456,85833-000,85834-999
zip_range_2262,city_br_2457,13495-000,13499-999
zip_range_2263,city_br_2458,89891-000,89892-999
zip_range_2264,city_br_2459,98460-000,98464-999
zip_range_2265,city_br_2460,38510-000,38519-999
zip_range_2266,city_br_2461,45370-000,45374-999
zip_range_2267,city_br_2462,46770-000,46779-999
zip_range_2268,city_br_2463,69415-000,69424-999
zip_range_2269,city_br_2464,89680-000,89682-999
zip_range_2270,city_br_2465,14990-000,14999-999
zip_range_2271,city_br_2466,17880-000,17889-999
zip_range_2272,city_br_2467,46980-000,46989-999
zip_range_2273,city_br_2468,44255-000,44259-999
zip_range_2274,city_br_2469,84500-000,84529-999
zip_range_2275,city_br_2470,89856-000,89858-999
zip_range_2276,city_br_2471,62620-000,62629-999
zip_range_2277,city_br_2472,44900-000,44904-999
zip_range_2278,city_br_2473,87280-000,87289-999
zip_range_2279,city_br_2474,89440-000,89459-999
zip_range_2280,city_br_2475,68655-000,68657-999
zip_range_2281,city_br_2476,29398-000,29399-999
zip_range_2282,city_br_2477,64570-000,64572-999
zip_range_2283,city_br_2478,76205-000,76209-999
zip_range_2284,city_br_2479,89760-000,89764-999
zip_range_2285,city_br_2480,97185-000,97189-999
zip_range_2286,city_br_2481,58360-000,58369-999
zip_range_2287,city_br_309,49500-001,49511-999
zip_range_2288,city_br_2482,49290-000,49299-999
zip_range_2289,city_br_2483,45848-000,45849-999
zip_range_2290,city_br_2484,18440-000,18459-999
zip_range_2291,city_br_2485,46880-000,46899-999
zip_range_2292,city_br_2486,76630-000,76639-999
zip_range_2293,city_br_2487,49870-000,49879-999
zip_range_2294,city_br_275,35900-001,35907-999
zip_range_2295,city_br_2488,35280-000,35289-999
zip_range_2296,city_br_2489,35450-000,35459-999
zip_range_2297,city_br_139,24800-001,24889-999
zip_range_2298,city_br_161,45600-001,45614-999
zip_range_2299,city_br_2490,77720-000,77722-999
zip_range_2300,city_br_2491,39594-000,39594-999
zip_range_2301,city_br_2492,39470-000,39471-999
zip_range_2302,city_br_2493,45530-000,45539-999
zip_range_2303,city_br_308,69100-001,69113-999
zip_range_2304,city_br_2494,56430-000,56439-999
zip_range_2305,city_br_2495,97685-000,97689-999
zip_range_2306,city_br_2496,46790-000,46799-999
zip_range_2307,city_br_2497,45230-000,45239-999
zip_range_2308,city_br_2498,45585-000,45589-999
zip_range_2309,city_br_2499,45850-000,45854-999
zip_range_2310,city_br_2500,29690-000,29699-999
zip_range_2311,city_br_2501,47440-000,47449-999
zip_range_2312,city_br_264,23800-001,23859-999
zip_range_2313,city_br_2502,86670-000,86679-999
zip_range_2314,city_br_2503,35488-000,35489-999
zip_range_2315,city_br_2504,76650-000,76659-999
zip_range_2316,city_br_2505,76660-000,76669-999
zip_range_2317,city_br_2506,77920-000,77924-999
zip_range_2318,city_br_2507,18730-000,18739-999
zip_range_2319,city_br_2508,56550-000,56559-999
zip_range_2320,city_br_2509,62820-000,62822-999
zip_range_2321,city_br_2510,64565-000,64567-999
zip_range_2322,city_br_2511,89340-000,89369-999
zip_range_2323,city_br_2512,65948-000,65949-999
zip_range_2324,city_br_2513,39815-000,39815-999
zip_range_2325,city_br_2514,85880-000,85883-999
zip_range_2326,city_br_2515,61880-000,61889-999
zip_range_2327,city_br_247,68180-001,68192-999
zip_range_2328,city_br_2516,75815-000,75818-999
zip_range_2329,city_br_2517,59513-000,59514-999
zip_range_2330,city_br_109,88300-001,88319-999
zip_range_2331,city_br_2518,15840-000,15844-999
zip_range_2332,city_br_2519,17260-000,17269-999
zip_range_2333,city_br_2520,45730-000,45739-999
zip_range_2334,city_br_2521,37500-001,37507-999
zip_range_2335,city_br_2522,45630-000,45637-999
zip_range_2336,city_br_2523,28250-000,28299-999
zip_range_2337,city_br_2524,45836-000,45839-999
zip_range_2338,city_br_2525,39670-000,39677-999
zip_range_2339,city_br_2526,69510-000,69519-999
zip_range_2340,city_br_2527,36788-000,36789-999
zip_range_2341,city_br_2528,45455-000,45459-999
zip_range_2342,city_br_2529,39830-000,39834-999
zip_range_2343,city_br_2530,86375-000,86379-999
zip_range_2344,city_br_2531,45140-000,45149-999
zip_range_2345,city_br_2532,55920-000,55929-999
zip_range_2346,city_br_2533,87175-000,87179-999
zip_range_2347,city_br_2534,35820-000,35829-999
zip_range_2348,city_br_2535,37973-000,37974-999
zip_range_2349,city_br_2536,37466-000,37466-999
zip_range_2350,city_br_2537,48290-000,48299-999
zip_range_2351,city_br_277,11740-000,11749-999
zip_range_2352,city_br_2538,37464-000,37464-999
zip_range_2353,city_br_2539,78579-000,78579-999
zip_range_2354,city_br_2540,45970-000,45979-999
zip_range_2355,city_br_2541,35120-000,35122-999
zip_range_2356,city_br_2542,39625-000,39627-999
zip_range_2357,city_br_2543,18360-000,18379-999
zip_range_2358,city_br_2544,28570-000,28599-999
zip_range_2359,city_br_2545,76360-000,76364-999
zip_range_2360,city_br_2546,38240-000,38249-999
zip_range_2361,city_br_2547,62600-000,62609-999
zip_range_2362,city_br_2548,44460-000,44469-999
zip_range_2363,city_br_2549,45750-000,45759-999
zip_range_2364,city_br_2550,45855-000,45859-999
zip_range_2365,city_br_2551,35550-000,35554-999
zip_range_2366,city_br_189,06850-001,06889-999
zip_range_2367,city_br_2552,65485-000,65489-999
zip_range_2368,city_br_2553,85580-000,85584-999
zip_range_2369,city_br_2554,88220-000,88229-999
zip_range_2370,city_br_2555,29330-000,29344-999
zip_range_2371,city_br_2556,83560-000,83569-999
zip_range_2372,city_br_319,28300-000,28349-999
zip_range_2373,city_br_2557,56720-000,56739-999
zip_range_2374,city_br_2558,45700-000,45709-999
zip_range_2375,city_br_191,18200-001,18219-999
zip_range_2376,city_br_2559,37655-000,37659-999
zip_range_2377,city_br_2560,18400-001,18424-999
zip_range_2378,city_br_129,06650-001,06699-999
zip_range_2379,city_br_2561,48475-000,48479-999
zip_range_2380,city_br_228,62500-000,62529-999
zip_range_2381,city_br_2562,13970-001,13989-999
zip_range_2382,city_br_2563,69120-000,69129-999
zip_range_2383,city_br_2564,89896-000,89896-999
zip_range_2384,city_br_2565,76290-000,76299-999
zip_range_2385,city_br_2566,18385-000,18399-999
zip_range_2386,city_br_2567,77718-000,77719-999
zip_range_2387,city_br_2568,53700-000,53899-999
zip_range_2388,city_br_2569,45645-000,45649-999
zip_range_2389,city_br_2570,62740-000,62747-999
zip_range_2390,city_br_2571,89249-000,89249-999
zip_range_2391,city_br_2572,14900-000,14909-999
zip_range_2392,city_br_2573,79890-000,79899-999
zip_range_2393,city_br_2574,77740-000,77742-999
zip_range_2394,city_br_2575,58780-000,58783-999
zip_range_2395,city_br_2576,18480-000,18489-999
zip_range_2396,city_br_2577,49120-000,49129-999
zip_range_2397,city_br_2578,58275-000,58277-999
zip_range_2398,city_br_2579,76861-000,76861-999
zip_range_2399,city_br_2580,95997-000,95999-999
zip_range_2400,city_br_2581,17230-000,17239-999
zip_range_2401,city_br_2582,15390-000,15399-999
zip_range_2402,city_br_2583,76680-000,76689-999
zip_range_2403,city_br_069,08570-001,08599-999
zip_range_2404,city_br_2584,45340-000,45344-999
zip_range_2405,city_br_2585,97650-000,97669-999
zip_range_2406,city_br_2586,79965-000,79969-999
zip_range_2407,city_br_2587,55950-000,55999-999
zip_range_2408,city_br_2588,29620-000,29629-999
zip_range_2409,city_br_2589,45780-000,45789-999
zip_range_2410,city_br_2590,18460-000,18469-999
zip_range_2411,city_br_2591,62590-000,62594-999
zip_range_2412,city_br_2592,11760-000,11789-999
zip_range_2413,city_br_2593,75810-000,75812-999
zip_range_2414,city_br_2594,95538-000,95539-999
zip_range_2415,city_br_2595,27580-000,27599-999
zip_range_2416,city_br_2596,35685-000,35689-999
zip_range_2417,city_br_251,13250-001,13259-999
zip_range_2418,city_br_2597,99760-000,99769-999
zip_range_2419,city_br_2598,46875-000,46879-999
zip_range_2420,city_br_2599,18690-000,18699-999
zip_range_2421,city_br_2600,62720-000,62729-999
zip_range_2422,city_br_2601,58378-000,58379-999
zip_range_2423,city_br_2602,59855-000,59855-999
zip_range_2424,city_br_2603,37975-000,37979-999
zip_range_2425,city_br_2604,78510-000,78514-999
zip_range_2426,city_br_2605,68976-000,68979-999
zip_range_2427,city_br_2606,75450-000,75454-999
zip_range_2428,city_br_2607,64820-000,64824-999
zip_range_2429,city_br_2608,35680-001,35684-999
zip_range_2430,city_br_2609,87980-000,87989-999
zip_range_2431,city_br_2610,36440-000,36449-999
zip_range_2432,city_br_2611,39610-000,39614-999
zip_range_2433,city_br_2612,65939-000,65939-999
zip_range_2434,city_br_2613,78790-000,78794-999
zip_range_2435,city_br_2614,13530-000,13536-999
zip_range_2436,city_br_2615,14420-000,14429-999
zip_range_2437,city_br_2616,45350-000,45354-999
zip_range_2438,city_br_2617,48850-000,48859-999
zip_range_2439,city_br_2618,13715-000,13719-999
zip_range_2440,city_br_2619,45710-000,45719-999
zip_range_2441,city_br_173,13300-001,13314-999
zip_range_2442,city_br_2620,46640-000,46649-999
zip_range_2443,city_br_2621,45435-000,45435-999
zip_range_2444,city_br_2622,35220-000,35224-999
zip_range_2445,city_br_314,38300-001,38309-999
zip_range_2446,city_br_287,75500-001,75549-999
zip_range_2447,city_br_2623,37210-000,37214-999
zip_range_2448,city_br_2624,13295-000,13299-999
zip_range_2449,city_br_2625,68580-000,68584-999
zip_range_2450,city_br_2626,88400-000,88406-999
zip_range_2451,city_br_2627,38280-000,38287-999
zip_range_2452,city_br_2628,36390-000,36399-999
zip_range_2453,city_br_2629,14500-000,14529-999
zip_range_2454,city_br_2630,46438-000,46439-999
zip_range_2455,city_br_2631,29390-000,29394-999
zip_range_2456,city_br_2632,84460-000,84469-999
zip_range_2457,city_br_2633,86870-000,86879-999
zip_range_2458,city_br_2634,87525-000,87527-999
zip_range_2459,city_br_2635,87130-000,87139-999
zip_range_2460,city_br_2636,79740-000,79744-999
zip_range_2461,city_br_2637,76130-000,76134-999
zip_range_2462,city_br_2638,98160-000,98169-999
zip_range_2463,city_br_2639,93900-000,93939-999
zip_range_2464,city_br_032,54000-001,54499-999
zip_range_2465,city_br_2640,89677-000,89679-999
zip_range_2466,city_br_2641,47655-000,47664-999
zip_range_2467,city_br_2642,14775-000,14779-999
zip_range_2468,city_br_2643,84930-000,84934-999
zip_range_2469,city_br_2644,98350-000,98359-999
zip_range_2470,city_br_2645,14870-001,14899-999
zip_range_2471,city_br_2646,35830-000,35844-999
zip_range_2472,city_br_2647,59225-000,59226-999
zip_range_2473,city_br_2648,46310-000,46329-999
zip_range_2474,city_br_2649,58278-000,58279-999
zip_range_2475,city_br_2650,57430-000,57434-999
zip_range_2476,city_br_2651,68195-000,68197-999
zip_range_2477,city_br_122,12300-001,12349-999
zip_range_2478,city_br_2652,86400-000,86409-999
zip_range_2479,city_br_2653,15155-000,15159-999
zip_range_2480,city_br_2654,78820-000,78829-999
zip_range_2481,city_br_2655,39930-000,39934-999
zip_range_2482,city_br_2656,88950-000,88954-999
zip_range_2483,city_br_2657,44700-000,44709-999
zip_range_2484,city_br_2658,64755-000,64757-999
zip_range_2485,city_br_2659,37965-000,37967-999
zip_range_2486,city_br_2660,57960-000,57964-999
zip_range_2487,city_br_2661,99457-000,99459-999
zip_range_2488,city_br_2662,68590-000,68599-999
zip_range_2489,city_br_2663,11940-000,11949-999
zip_range_2490,city_br_2664,37590-000,37595-999
zip_range_2491,city_br_2665,99730-000,99734-999
zip_range_2492,city_br_2666,86610-000,86612-999
zip_range_2493,city_br_2667,45345-000,45349-999
zip_range_2494,city_br_2668,35188-000,35188-999
zip_range_2495,city_br_2669,96300-000,96309-999
zip_range_2496,city_br_2670,48960-000,48969-999
zip_range_2497,city_br_2671,29950-000,29959-999
zip_range_2498,city_br_2672,63480-000,63489-999
zip_range_2499,city_br_2673,97760-000,97769-999
zip_range_2500,city_br_2674,84200-000,84219-999
zip_range_2501,city_br_2675,63490-000,63499-999
zip_range_2502,city_br_2676,63475-000,63479-999
zip_range_2503,city_br_2677,44480-000,44489-999
zip_range_2504,city_br_2678,13820-000,13824-999
zip_range_2505,city_br_2679,62823-000,62839-999
zip_range_2506,city_br_2680,88715-000,88716-999
zip_range_2507,city_br_2681,39508-000,39509-999
zip_range_2508,city_br_2682,64575-000,64577-999
zip_range_2509,city_br_2683,15700-001,15709-999
zip_range_2510,city_br_2684,12270-000,12279-999
zip_range_2511,city_br_2685,39837-000,39839-999
zip_range_2512,city_br_2686,39440-000,39449-999
zip_range_2513,city_br_2687,75950-000,75954-999
zip_range_2514,city_br_2688,86900-000,86909-999
zip_range_2515,city_br_2689,48310-000,48329-999
zip_range_2516,city_br_2690,59594-000,59595-999
zip_range_2517,city_br_258,06600-001,06649-999
zip_range_2518,city_br_2691,59690-000,59694-999
zip_range_2519,city_br_2692,78490-000,78499-999
zip_range_2520,city_br_2693,87380-000,87389-999
zip_range_2521,city_br_2694,39480-000,39488-999
zip_range_2522,city_br_2695,35580-000,35581-999
zip_range_2523,city_br_2696,57950-000,57954-999
zip_range_2524,city_br_2697,49960-000,49969-999
zip_range_2525,city_br_2698,26400-001,26499-999
zip_range_2526,city_br_2699,59213-000,59213-999
zip_range_2527,city_br_2700,84920-000,84924-999
zip_range_2528,city_br_2701,49950-000,49959-999
zip_range_2529,city_br_2702,39335-000,39335-999
zip_range_2530,city_br_2703,79985-000,79989-999
zip_range_2531,city_br_2704,69495-000,69499-999
zip_range_2532,city_br_2705,87225-000,87229-999
zip_range_2533,city_br_2706,55409-000,55409-999
zip_range_2534,city_br_2707,95420-000,95479-999
zip_range_2535,city_br_2708,76330-000,76334-999
zip_range_2536,city_br_164,89250-001,89269-999
zip_range_2537,city_br_2709,79440-000,79449-999
zip_range_2538,city_br_2710,57425-000,57429-999
zip_range_2539,city_br_2711,63290-000,63299-999
zip_range_2540,city_br_2712,79240-000,79259-999
zip_range_2541,city_br_2713,86860-000,86864-999
zip_range_2542,city_br_2714,59544-000,59546-999
zip_range_2543,city_br_2715,59324-000,59326-999
zip_range_2544,city_br_2716,64495-000,64499-999
zip_range_2545,city_br_2717,59343-000,59346-999
zip_range_2546,city_br_2718,87690-000,87699-999
zip_range_2547,city_br_2719,89848-000,89849-999
zip_range_2548,city_br_2720,14680-000,14699-999
zip_range_2549,city_br_2721,98175-000,98179-999
zip_range_2550,city_br_2722,13240-000,13249-999
zip_range_2551,city_br_2723,76890-000,76897-999
zip_range_2552,city_br_294,75800-001,75809-999
zip_range_2553,city_br_2724,86210-000,86219-999
zip_range_2554,city_br_2725,55180-000,55189-999
zip_range_2555,city_br_2726,79720-000,79729-999
zip_range_2556,city_br_2727,63275-000,63279-999
zip_range_2557,city_br_2728,65693-000,65694-999
zip_range_2558,city_br_2729,56470-000,56479-999
zip_range_2559,city_br_2730,64275-000,64277-999
zip_range_2560,city_br_223,17200-001,17229-999
zip_range_2561,city_br_2731,77450-000,77452-999
zip_range_2562,city_br_2732,76210-000,76219-999
zip_range_2563,city_br_2733,78255-000,78259-999
zip_range_2564,city_br_2734,35497-000,35499-999
zip_range_2565,city_br_2735,39645-000,39647-999
zip_range_2566,city_br_2736,65962-000,65963-999
zip_range_2567,city_br_2737,35390-000,35399-999
zip_range_2568,city_br_2738,57255-000,57256-999
zip_range_2569,city_br_187,45200-001,45214-999
zip_range_2570,city_br_2739,39370-000,39372-999
zip_range_2571,city_br_2740,35767-000,35769-999
zip_range_2572,city_br_2741,39960-000,39969-999
zip_range_2573,city_br_2742,48540-000,48564-999
zip_range_2574,city_br_2743,58830-000,58831-999
zip_range_2575,city_br_2744,14450-000,14459-999
zip_range_2576,city_br_2745,29550-000,29559-999
zip_range_2577,city_br_2746,64830-000,64832-999
zip_range_2578,city_br_2747,37485-000,37487-999
zip_range_2579,city_br_2748,85835-000,85839-999
zip_range_2580,city_br_2749,75495-000,75499-999
zip_range_2581,city_br_244,76900-001,76915-999
zip_range_2582,city_br_2750,62598-000,62599-999
zip_range_2583,city_br_2751,45470-000,45479-999
zip_range_2584,city_br_2752,45225-000,45229-999
zip_range_2585,city_br_2753,89600-000,89608-999
zip_range_2586,city_br_2754,39890-000,39892-999
zip_range_2587,city_br_2755,35194-000,35194-999
zip_range_2588,city_br_2756,12980-000,12989-999
zip_range_2589,city_br_2757,55720-000,55729-999
zip_range_2590,city_br_2758,59550-000,59554-999
zip_range_2591,city_br_2759,64765-000,64766-999
zip_range_2592,city_br_2760,59880-000,59889-999
zip_range_2593,city_br_2761,44920-000,44924-999
zip_range_2594,city_br_2762,65922-000,65922-999
zip_range_2595,city_br_2763,35930-001,35934-999
zip_range_2596,city_br_2764,29680-000,29689-999
zip_range_2597,city_br_020,58000-001,58099-999
zip_range_2598,city_br_2765,38770-000,38778-999
zip_range_2599,city_br_2766,19680-000,19699-999
zip_range_2600,city_br_2767,39240-000,39242-999
zip_range_2601,city_br_2768,57980-000,57989-999
zip_range_2602,city_br_2769,55535-000,55539-999
zip_range_2603,city_br_2770,64170-000,64174-999
zip_range_2604,city_br_2771,86455-000,86459-999
zip_range_2605,city_br_2772,58928-000,58929-999
zip_range_2606,city_br_2773,64165-000,64167-999
zip_range_2607,city_br_2774,98180-000,98199-999
zip_range_2608,city_br_034,89200-001,89239-999
zip_range_2609,city_br_2775,39920-000,39924-999
zip_range_2610,city_br_2776,69975-000,69979-999
zip_range_2611,city_br_2777,89145-000,89147-999
zip_range_2612,city_br_2778,15200-000,15209-999
zip_range_2613,city_br_2779,59980-000,59986-999
zip_range_2614,city_br_2780,64110-000,64119-999
zip_range_2615,city_br_2781,39642-000,39643-999
zip_range_2616,city_br_2782,39775-000,39779-999
zip_range_2617,city_br_2783,65755-000,65757-999
zip_range_2618,city_br_2784,39575-000,39579-999
zip_range_2619,city_br_2785,75610-000,75614-999
zip_range_2620,city_br_2786,78575-000,78577-999
zip_range_2621,city_br_2787,58387-000,58387-999
zip_range_2622,city_br_2788,77753-000,77754-999
zip_range_2623,city_br_2789,35675-000,35679-999
zip_range_2624,city_br_2790,58660-000,58664-999
zip_range_2625,city_br_126,48900-001,48924-999
zip_range_2626,city_br_096,63000-001,63099-999
zip_range_2627,city_br_2791,64343-000,64344-999
zip_range_2628,city_br_2792,63580-000,63589-999
zip_range_2629,city_br_2793,55398-000,55399-999
zip_range_2630,city_br_2794,45834-000,45835-999
zip_range_2631,city_br_2795,59330-000,59334-999
zip_range_2632,city_br_2796,78320-000,78324-999
zip_range_2633,city_br_038,36000-001,36107-999
zip_range_2634,city_br_2797,64963-000,64964-999
zip_range_2635,city_br_2798,98130-000,98139-999
zip_range_2636,city_br_2799,17550-000,17559-999
zip_range_2637,city_br_2800,18535-000,18539-999
zip_range_2638,city_br_2801,65294-000,65294-999
zip_range_2639,city_br_2802,58640-000,58649-999
zip_range_2640,city_br_2803,57965-000,57967-999
zip_range_2641,city_br_2804,59188-000,59189-999
zip_range_2642,city_br_051,13200-001,13219-999
zip_range_2643,city_br_2805,86470-000,86479-999
zip_range_2644,city_br_2806,57270-000,57274-999
zip_range_2645,city_br_2807,17890-000,17899-999
zip_range_2646,city_br_2808,55395-000,55397-999
zip_range_2647,city_br_2809,89839-000,89839-999
zip_range_2648,city_br_2810,11800-000,11849-999
zip_range_2649,city_br_2811,06950-000,06999-999
zip_range_2650,city_br_2812,39590-000,39591-999
zip_range_2651,city_br_2813,87355-000,87359-999
zip_range_2652,city_br_2814,55480-000,55489-999
zip_range_2653,city_br_2815,64782-000,64782-999
zip_range_2654,city_br_2816,58330-000,58333-999
zip_range_2655,city_br_2817,58750-000,58752-999
zip_range_2656,city_br_2818,69520-000,69529-999
zip_range_2657,city_br_2819,37805-000,37809-999
zip_range_2658,city_br_2820,78340-000,78344-999
zip_range_2659,city_br_2821,68170-000,68179-999
zip_range_2660,city_br_2822,78810-000,78819-999
zip_range_2661,city_br_2823,44925-000,44929-999
zip_range_2662,city_br_2824,76270-000,76279-999
zip_range_2663,city_br_2825,87230-000,87234-999
zip_range_2664,city_br_2826,45622-000,45624-999
zip_range_2665,city_br_2827,46670-000,46689-999
zip_range_2666,city_br_2828,69660-000,69669-999
zip_range_2667,city_br_2829,79955-000,79959-999
zip_range_2668,city_br_2830,39467-000,39469-999
zip_range_2669,city_br_2831,86920-000,86924-999
zip_range_2670,city_br_2832,69830-000,69849-999
zip_range_2671,city_br_2833,89660-000,89662-999
zip_range_2672,city_br_2834,39825-000,39826-999
zip_range_2673,city_br_2835,79370-000,79379-999
zip_range_2674,city_br_2836,45215-000,45219-999
zip_range_2675,city_br_2837,38785-000,38789-999
zip_range_2676,city_br_317,49400-000,49479-999
zip_range_2677,city_br_182,88500-001,88534-999
zip_range_2678,city_br_2838,65715-000,65715-999
zip_range_2679,city_br_2839,65710-000,65711-999
zip_range_2680,city_br_2840,65712-000,65713-999
zip_range_2681,city_br_2841,65705-000,65705-999
zip_range_2682,city_br_2842,58835-000,58839-999
zip_range_2683,city_br_2843,64138-000,64139-999
zip_range_2684,city_br_2844,96920-000,96924-999
zip_range_2685,city_br_2845,59227-000,59229-999
zip_range_2686,city_br_2846,57330-000,57339-999
zip_range_2687,city_br_2847,77493-000,77494-999
zip_range_2688,city_br_2848,35590-000,35594-999
zip_range_2689,city_br_2849,58250-000,58252-999
zip_range_2690,city_br_2850,55840-000,55844-999
zip_range_2691,city_br_2851,59244-000,59244-999
zip_range_2692,city_br_2852,64258-000,64259-999
zip_range_2693,city_br_2853,59430-000,59439-999
zip_range_2694,city_br_2854,64768-000,64769-999
zip_range_2695,city_br_2855,55820-000,55824-999
zip_range_2696,city_br_2856,65683-000,65684-999
zip_range_2697,city_br_2857,55320-000,55324-999
zip_range_2698,city_br_2858,64388-000,64389-999
zip_range_2699,city_br_2859,64308-000,64309-999
zip_range_2700,city_br_2860,77613-000,77614-999
zip_range_2701,city_br_2861,55450-000,55459-999
zip_range_2702,city_br_2862,39360-000,39362-999
zip_range_2703,city_br_2863,99495-000,99499-999
zip_range_2704,city_br_2864,36345-000,36349-999
zip_range_2705,city_br_2865,38720-000,38729-999
zip_range_2706,city_br_2866,38755-000,38759-999
zip_range_2707,city_br_2867,56395-000,56399-999
zip_range_2708,city_br_2868,65718-000,65719-999
zip_range_2709,city_br_2869,59390-000,59394-999
zip_range_2710,city_br_2870,46425-000,46429-999
zip_range_2711,city_br_2871,59247-000,59249-999
zip_range_2712,city_br_2872,75819-000,75819-999
zip_range_2713,city_br_2873,33400-000,33499-999
zip_range_2714,city_br_2874,58117-000,58118-999
zip_range_2715,city_br_2875,95300-000,95304-999
zip_range_2716,city_br_2876,99340-000,99344-999
zip_range_2717,city_br_2877,12130-000,12139-999
zip_range_2718,city_br_2878,64465-000,64467-999
zip_range_2719,city_br_2879,88790-000,88797-999
zip_range_2720,city_br_2880,79920-000,79924-999
zip_range_2721,city_br_2881,45490-000,45499-999
zip_range_2722,city_br_2882,28350-000,28359-999
zip_range_2723,city_br_2883,95900-001,95914-999
zip_range_2724,city_br_2884,77645-000,77649-999
zip_range_2725,city_br_2885,98320-000,98322-999
zip_range_2726,city_br_2886,89828-000,89829-999
zip_range_2727,city_br_2887,65937-000,65937-999
zip_range_2728,city_br_2888,45950-000,45954-999
zip_range_2729,city_br_2889,46825-000,46829-999
zip_range_2730,city_br_2890,55385-000,55389-999
zip_range_2731,city_br_2891,45365-000,45369-999
zip_range_2732,city_br_2892,59535-000,59539-999
zip_range_2733,city_br_2893,59235-000,59239-999
zip_range_2734,city_br_2894,36980-000,36984-999
zip_range_2735,city_br_2895,48720-000,48724-999
zip_range_2736,city_br_2896,37480-000,37483-999
zip_range_2737,city_br_2897,78278-000,78279-999
zip_range_2738,city_br_2898,36455-000,36459-999
zip_range_2739,city_br_2899,64850-000,64854-999
zip_range_2740,city_br_2900,83750-000,83799-999
zip_range_2741,city_br_2901,44905-000,44909-999
zip_range_2742,city_br_2902,29615-000,29619-999
zip_range_2743,city_br_2903,36760-000,36769-999
zip_range_2744,city_br_2904,85275-000,85279-999
zip_range_2745,city_br_2905,68920-000,68923-999
zip_range_2746,city_br_2906,18500-000,18519-999
zip_range_2747,city_br_2907,49170-000,49179-999
zip_range_2748,city_br_2908,85300-001,85339-999
zip_range_2749,city_br_2909,39250-000,39259-999
zip_range_2750,city_br_2910,58820-000,58821-999
zip_range_2751,city_br_2911,89170-000,89171-999
zip_range_2752,city_br_148,42700-001,42799-999
zip_range_2753,city_br_2912,88880-000,88889-999
zip_range_2754,city_br_2913,77328-000,77329-999
zip_range_2755,city_br_2914,16850-000,16879-999
zip_range_2756,city_br_300,37200-000,37209-999
zip_range_2757,city_br_2915,63300-000,63309-999
zip_range_2758,city_br_2916,97390-000,97399-999
zip_range_2759,city_br_2917,12760-000,12799-999
zip_range_2760,city_br_2918,35657-000,35659-999
zip_range_2761,city_br_2919,89515-000,89517-999
zip_range_2762,city_br_2920,13610-001,13624-999
zip_range_2763,city_br_2921,39655-000,39659-999
zip_range_2764,city_br_2922,46960-000,46969-999
zip_range_2765,city_br_2923,18680-001,18689-999
zip_range_2766,city_br_2924,88445-000,88449-999
zip_range_2767,city_br_2925,36700-000,36709-999
zip_range_2768,city_br_2926,75190-000,75194-999
zip_range_2769,city_br_2927,86330-000,86339-999
zip_range_2770,city_br_2928,99690-000,99697-999
zip_range_2771,city_br_2929,37350-000,37359-999
zip_range_2772,city_br_2930,46330-000,46349-999
zip_range_2773,city_br_2931,86865-000,86869-999
zip_range_2774,city_br_2932,65728-000,65729-999
zip_range_2775,city_br_2933,36140-000,36144-999
zip_range_2776,city_br_094,13480-001,13489-999
zip_range_2777,city_br_2934,38295-000,38299-999
zip_range_2778,city_br_2935,55700-000,55714-999
zip_range_2779,city_br_2936,57260-000,57264-999
zip_range_2780,city_br_2937,68415-000,68419-999
zip_range_2781,city_br_2938,62930-000,62939-999
zip_range_2782,city_br_2939,85826-000,85829-999
zip_range_2783,city_br_2940,13950-000,13959-999
zip_range_2784,city_br_2941,89735-000,89739-999
zip_range_2785,city_br_2942,93940-000,93944-999
zip_range_2786,city_br_2943,95768-000,95769-999
zip_range_2787,city_br_177,29900-001,29919-999
zip_range_2788,city_br_2944,16400-001,16429-999
zip_range_2789,city_br_2945,58690-000,58694-999
zip_range_2790,city_br_2946,46140-000,46164-999
zip_range_2791,city_br_2947,77630-000,77634-999
zip_range_2792,city_br_2948,87900-000,87909-999
zip_range_2793,city_br_2949,86790-000,86799-999
zip_range_2794,city_br_2950,58254-000,58254-999
zip_range_2795,city_br_037,86000-001,86124-999
zip_range_2796,city_br_2951,39437-000,39439-999
zip_range_2797,city_br_2952,89182-000,89183-999
zip_range_2798,city_br_2953,12600-001,12614-999
zip_range_2799,city_br_2954,65895-000,65899-999
zip_range_2800,city_br_2955,15285-000,15289-999
zip_range_2801,city_br_2956,13290-000,13294-999
zip_range_2802,city_br_2957,78455-000,78459-999
zip_range_2803,city_br_2958,17780-000,17789-999
zip_range_2804,city_br_2959,58315-000,58319-999
zip_range_2805,city_br_2960,17475-000,17479-999
zip_range_2806,city_br_2961,78660-000,78662-999
zip_range_2807,city_br_2962,59805-000,59807-999
zip_range_2808,city_br_2963,14210-000,14229-999
zip_range_2809,city_br_2964,64220-000,64221-999
zip_range_2810,city_br_2965,65290-000,65291-999
zip_range_2811,city_br_288,47850-000,47859-999
zip_range_2812,city_br_2966,59940-000,59944-999
zip_range_2813,city_br_2967,36923-000,36924-999
zip_range_2814,city_br_2968,39336-000,39337-999
zip_range_2815,city_br_2969,89128-000,89129-999
zip_range_2816,city_br_2970,87290-000,87299-999
zip_range_2817,city_br_2971,16340-000,16349-999
zip_range_2818,city_br_2972,37240-000,37244-999
zip_range_2819,city_br_2973,86935-000,86937-999
zip_range_2820,city_br_2974,17420-000,17429-999
zip_range_2821,city_br_2975,86635-000,86639-999
zip_range_2822,city_br_2976,19750-000,19769-999
zip_range_2823,city_br_2977,35595-000,35599-999
zip_range_2824,city_br_2978,89609-000,89609-999
zip_range_2825,city_br_146,72800-001,72859-999
zip_range_2826,city_br_2979,64160-000,64164-999
zip_range_2827,city_br_2980,77903-000,77904-999
zip_range_2828,city_br_117,27900-001,27997-999
zip_range_2829,city_br_2981,59280-000,59289-999
zip_range_2830,city_br_2982,46805-000,46809-999
zip_range_2831,city_br_2983,97645-000,97649-999
zip_range_2832,city_br_2984,49565-000,49569-999
zip_range_2833,city_br_052,68900-001,68914-999
zip_range_2834,city_br_2985,55865-000,55869-999
zip_range_2835,city_br_2986,45760-000,45769-999
zip_range_2836,city_br_2987,17290-000,17299-999
zip_range_2837,city_br_2988,59500-000,59503-999
zip_range_2838,city_br_2989,15270-000,15274-999
zip_range_2839,city_br_2990,46500-000,46529-999
zip_range_2840,city_br_2991,15620-000,15624-999
zip_range_2841,city_br_016,57000-001,57099-999
zip_range_2842,city_br_2992,39873-000,39873-999
zip_range_2843,city_br_2993,99880-000,99889-999
zip_range_2844,city_br_2994,76868-000,76869-999
zip_range_2845,city_br_2995,37750-000,37756-999
zip_range_2846,city_br_2996,55740-000,55744-999
zip_range_2847,city_br_2997,89518-000,89519-999
zip_range_2848,city_br_2998,28545-000,28549-999
zip_range_2849,city_br_2999,48650-000,48659-999
zip_range_2850,city_br_3000,63860-000,63869-999
zip_range_2851,city_br_3001,64168-000,64169-999
zip_range_2852,city_br_3002,42600-000,42699-999
zip_range_2853,city_br_3003,37305-000,37309-999
zip_range_2854,city_br_3004,58740-000,58744-999
zip_range_2855,city_br_3005,68675-000,68679-999
zip_range_2856,city_br_3006,46255-000,46269-999
zip_range_2857,city_br_3007,89300-000,89339-999
zip_range_2858,city_br_3008,68722-000,68724-999
zip_range_2859,city_br_3009,65560-000,65569-999
zip_range_2860,city_br_3010,15310-000,15312-999
zip_range_2861,city_br_132,25900-001,25939-999
zip_range_2862,city_br_3011,45770-000,45779-999
zip_range_2863,city_br_3012,44630-000,44634-999
zip_range_2864,city_br_3013,18120-000,18124-999
zip_range_2865,city_br_3014,07600-000,07699-999
zip_range_2866,city_br_3015,75630-000,75634-999
zip_range_2867,city_br_3016,88260-000,88269-999
zip_range_2868,city_br_3017,57580-000,57599-999
zip_range_2869,city_br_3018,59945-000,59949-999
zip_range_2870,city_br_3019,89480-000,89489-999
zip_range_2871,city_br_3020,39690-000,39694-999
zip_range_2872,city_br_3021,46440-000,46444-999
zip_range_2873,city_br_3022,46110-000,46129-999
zip_range_2874,city_br_3023,49940-000,49944-999
zip_range_2875,city_br_3024,49570-000,49579-999
zip_range_2876,city_br_3025,84570-000,84599-999
zip_range_2877,city_br_3026,58713-000,58713-999
zip_range_2878,city_br_3027,58280-000,58286-999
zip_range_2879,city_br_3028,73970-000,73974-999
zip_range_2880,city_br_3029,87340-000,87344-999
zip_range_2881,city_br_3030,39516-000,39516-999
zip_range_2882,city_br_3031,95572-000,95574-999
zip_range_2883,city_br_315,69400-001,69414-999
zip_range_2884,city_br_3032,58995-000,58996-999
zip_range_2885,city_br_3033,69435-000,69439-999
zip_range_2886,city_br_3034,56565-000,56579-999
zip_range_2887,city_br_007,69000-001,69099-999
zip_range_2888,city_br_3035,69990-000,69999-999
zip_range_2889,city_br_3036,87160-000,87169-999
zip_range_2890,city_br_3037,86975-000,86989-999
zip_range_2891,city_br_3038,83800-000,83819-999
zip_range_2892,city_br_3039,18780-000,18789-999
zip_range_2893,city_br_3040,85628-000,85629-999
zip_range_2894,city_br_3041,39460-000,39464-999
zip_range_2895,city_br_3042,23860-000,23889-999
zip_range_2896,city_br_3043,85540-000,85547-999
zip_range_2897,city_br_3044,36900-000,36912-999
zip_range_2898,city_br_3045,36970-000,36971-999
zip_range_2899,city_br_3046,69280-000,69299-999
zip_range_2900,city_br_3047,64875-000,64879-999
zip_range_2901,city_br_3048,85260-000,85269-999
zip_range_2902,city_br_3049,69950-000,69954-999
zip_range_2903,city_br_3050,97640-000,97644-999
zip_range_2904,city_br_3051,45240-000,45249-999
zip_range_2905,city_br_3052,47160-000,47199-999
zip_range_2906,city_br_3053,35290-000,35297-999
zip_range_2907,city_br_3054,29770-000,29779-999
zip_range_2908,city_br_3055,95530-000,95534-999
zip_range_2909,city_br_3056,36640-000,36649-999
zip_range_2910,city_br_3057,57730-000,57739-999
zip_range_2911,city_br_3058,76490-000,76492-999
zip_range_2912,city_br_3059,69490-000,69494-999
zip_range_2913,city_br_105,68500-001,68514-999
zip_range_2914,city_br_3060,19430-000,19449-999
zip_range_2915,city_br_3061,65289-000,65289-999
zip_range_2916,city_br_3062,19840-000,19859-999
zip_range_2917,city_br_3063,88915-000,88919-999
zip_range_2918,city_br_3064,79150-000,79169-999
zip_range_2919,city_br_3065,68710-000,68718-999
zip_range_2920,city_br_128,61900-001,61939-999
zip_range_2921,city_br_3066,45360-000,45364-999
zip_range_2922,city_br_3067,57955-000,57959-999
zip_range_2923,city_br_3068,44420-000,44449-999
zip_range_2924,city_br_3069,55405-000,55408-999
zip_range_2925,city_br_3070,65714-000,65714-999
zip_range_2926,city_br_298,61940-001,61999-999
zip_range_2927,city_br_3071,65283-000,65283-999
zip_range_2928,city_br_3072,68760-000,68769-999
zip_range_2929,city_br_3073,15845-000,15849-999
zip_range_2930,city_br_3074,95793-000,95794-999
zip_range_2931,city_br_3075,29345-000,29349-999
zip_range_2932,city_br_3076,99150-000,99154-999
zip_range_2933,city_br_3077,45520-000,45529-999
zip_range_2934,city_br_3078,57520-000,57524-999
zip_range_2935,city_br_3079,89874-000,89874-999
zip_range_2936,city_br_3080,35666-000,35666-999
zip_range_2937,city_br_3081,58294-000,58294-999
zip_range_2938,city_br_3082,78535-000,78539-999
zip_range_2939,city_br_3083,99800-000,99809-999
zip_range_2940,city_br_3084,59970-000,59979-999
zip_range_2941,city_br_3085,46780-000,46789-999
zip_range_2942,city_br_3086,62560-000,62569-999
zip_range_2943,city_br_3087,64685-000,64687-999
zip_range_2944,city_br_3088,64845-000,64849-999
zip_range_2945,city_br_3089,85960-000,85979-999
zip_range_2946,city_br_3090,57160-000,57179-999
zip_range_2947,city_br_3091,29255-000,29259-999
zip_range_2948,city_br_3092,69983-000,69984-999
zip_range_2949,city_br_3093,89860-000,89861-999
zip_range_2950,city_br_3094,58345-000,58347-999
zip_range_2951,city_br_3095,37517-000,37519-999
zip_range_2952,city_br_3096,87480-000,87484-999
zip_range_2953,city_br_3097,86990-000,86999-999
zip_range_2954,city_br_3098,35420-000,35429-999
zip_range_2955,city_br_3099,92900-000,92989-999
zip_range_2956,city_br_3100,99790-000,99794-999
zip_range_2957,city_br_3101,77675-000,77679-999
zip_range_2958,city_br_3102,17810-000,17829-999
zip_range_2959,city_br_3103,57670-000,57679-999
zip_range_2960,city_br_154,24900-001,24999-999
zip_range_2961,city_br_3104,35115-000,35115-999
zip_range_2962,city_br_3105,29725-000,29729-999
zip_range_2963,city_br_3106,86825-000,86827-999
zip_range_2964,city_br_3107,87960-000,87969-999
zip_range_2965,city_br_123,17500-001,17539-999
zip_range_2966,city_br_3108,87470-000,87479-999
zip_range_2967,city_br_061,87000-001,87109-999
zip_range_2968,city_br_3109,15730-000,15734-999
zip_range_2969,city_br_3110,32470-000,32499-999
zip_range_2970,city_br_3111,85525-000,85529-999
zip_range_2971,city_br_3112,85955-000,85959-999
zip_range_2972,city_br_3113,36608-000,36609-999
zip_range_2973,city_br_285,67200-000,67999-999
zip_range_2974,city_br_3114,58819-000,58819-999
zip_range_2975,city_br_3115,35185-000,35187-999
zip_range_2976,city_br_3116,85615-000,85617-999
zip_range_2977,city_br_3117,37516-000,37516-999
zip_range_2978,city_br_3118,95923-000,95924-999
zip_range_2979,city_br_3119,85168-000,85169-999
zip_range_2980,city_br_3120,35606-000,35609-999
zip_range_2981,city_br_3121,62450-000,62459-999
zip_range_2982,city_br_3122,19500-000,19529-999
zip_range_2983,city_br_3123,59800-000,59804-999
zip_range_2984,city_br_3124,36972-000,36973-999
zip_range_2985,city_br_3125,49770-000,49779-999
zip_range_2986,city_br_3126,86910-000,86919-999
zip_range_2987,city_br_3127,75670-000,75679-999
zip_range_2988,city_br_3128,45870-000,45879-999
zip_range_2989,city_br_3129,62140-000,62149-999
zip_range_2990,city_br_3130,64573-000,64574-999
zip_range_2991,city_br_3131,58120-000,58122-999
zip_range_2992,city_br_3132,89108-000,89109-999
zip_range_2993,city_br_3133,97410-000,97417-999
zip_range_2994,city_br_3134,48280-000,48289-999
zip_range_2995,city_br_3135,57540-000,57544-999
zip_range_2996,city_br_3136,65510-000,65514-999
zip_range_2997,city_br_3137,39915-000,39916-999
zip_range_2998,city_br_3138,15990-001,15999-999
zip_range_2999,city_br_3139,58292-000,58293-999
zip_range_3000,city_br_3140,77593-000,77599-999
zip_range_3001,city_br_3141,85887-000,85887-999
zip_range_3002,city_br_3142,39755-000,39764-999
zip_range_3003,city_br_3143,35670-000,35674-999
zip_range_3004,city_br_3144,35110-000,35111-999
zip_range_3005,city_br_3145,36120-000,36122-999
zip_range_3006,city_br_3146,39478-000,39479-999
zip_range_3007,city_br_3147,64150-000,64154-999
zip_range_3008,city_br_3148,46480-000,46489-999
zip_range_3009,city_br_3149,65218-000,65219-999
zip_range_3010,city_br_3150,58128-000,58134-999
zip_range_3011,city_br_3151,83260-000,83279-999
zip_range_3012,city_br_3152,35367-000,35367-999
zip_range_3013,city_br_3153,99180-000,99189-999
zip_range_3014,city_br_3154,58832-000,58834-999
zip_range_3015,city_br_3155,95835-000,95839-999
zip_range_3016,city_br_3156,97935-000,97939-999
zip_range_3017,city_br_3157,85240-000,85249-999
zip_range_3018,city_br_3158,39527-000,39528-999
zip_range_3019,city_br_3159,65645-000,65649-999
zip_range_3020,city_br_3160,65468-000,65469-999
zip_range_3021,city_br_3161,89420-000,89429-999
zip_range_3022,city_br_3162,35720-000,35729-999
zip_range_3023,city_br_3163,76730-000,76739-999
zip_range_3024,city_br_3164,57910-000,57919-999
zip_range_3025,city_br_3165,78525-000,78527-999
zip_range_3026,city_br_3166,58737-000,58739-999
zip_range_3027,city_br_3167,38870-000,38879-999
zip_range_3028,city_br_057,09300-001,09399-999
zip_range_3029,city_br_3168,86828-000,86829-999
zip_range_3030,city_br_3169,69190-000,69194-999
zip_range_3031,city_br_3170,75930-000,75934-999
zip_range_3032,city_br_3171,77918-000,77919-999
zip_range_3033,city_br_3172,63210-000,63219-999
zip_range_3034,city_br_3173,59580-000,59581-999
zip_range_3035,city_br_3174,99890-000,99894-999
zip_range_3036,city_br_3175,68940-000,68944-999
zip_range_3037,city_br_3176,38930-000,38949-999
zip_range_3038,city_br_3177,45960-000,45969-999
zip_range_3039,city_br_3178,85884-000,85884-999
zip_range_3040,city_br_3179,68145-000,68147-999
zip_range_3041,city_br_3180,39620-000,39624-999
zip_range_3042,city_br_3181,88920-000,88924-999
zip_range_3043,city_br_3182,68490-000,68499-999
zip_range_3044,city_br_3183,26700-000,26899-999
zip_range_3045,city_br_3184,35270-000,35274-999
zip_range_3046,city_br_3185,15220-000,15224-999
zip_range_3047,city_br_3186,85998-000,85999-999
zip_range_3048,city_br_3187,36190-000,36194-999
zip_range_3049,city_br_3188,15625-000,15629-999
zip_range_3050,city_br_3189,62130-000,62139-999
zip_range_3051,city_br_3190,15748-000,15749-999
zip_range_3052,city_br_3191,35116-000,35116-999
zip_range_3053,city_br_176,26550-001,26599-999
zip_range_3054,city_br_3192,57990-000,57994-999
zip_range_3055,city_br_3193,59775-000,59779-999
zip_range_3056,city_br_3194,64130-000,64137-999
zip_range_3057,city_br_3195,44720-000,44729-999
zip_range_3058,city_br_3196,64445-000,64449-999
zip_range_3059,city_br_3197,26900-000,26949-999
zip_range_3060,city_br_3198,14530-000,14539-999
zip_range_3061,city_br_3199,45315-000,45319-999
zip_range_3062,city_br_3200,63250-000,63259-999
zip_range_3063,city_br_3201,65545-000,65549-999
zip_range_3064,city_br_3202,63635-000,63639-999
zip_range_3065,city_br_3203,64253-000,64254-999
zip_range_3066,city_br_3204,73730-000,73739-999
zip_range_3067,city_br_3205,29400-000,29449-999
zip_range_3068,city_br_3206,76450-000,76459-999
zip_range_3069,city_br_3207,57615-000,57619-999
zip_range_3070,city_br_3208,96755-000,96759-999
zip_range_3071,city_br_3209,39650-000,39654-999
zip_range_3072,city_br_3210,37447-000,37449-999
zip_range_3073,city_br_3211,75830-000,75834-999
zip_range_3074,city_br_3212,17320-000,17339-999
zip_range_3075,city_br_3213,76919-000,76919-999
zip_range_3076,city_br_3214,15580-000,15599-999
zip_range_3077,city_br_3215,39373-000,39377-999
zip_range_3078,city_br_3216,11850-000,11899-999
zip_range_3079,city_br_3217,28460-000,28469-999
zip_range_3080,city_br_3218,77650-000,77654-999
zip_range_3081,city_br_3219,65850-000,65859-999
zip_range_3082,city_br_3220,87840-000,87849-999
zip_range_3083,city_br_3221,36893-000,36894-999
zip_range_3084,city_br_3222,98540-000,98549-999
zip_range_3085,city_br_3223,36790-000,36792-999
zip_range_3086,city_br_3224,62530-000,62539-999
zip_range_3087,city_br_3225,79380-000,79389-999
zip_range_3088,city_br_3226,65495-000,65499-999
zip_range_3089,city_br_3227,56980-000,56999-999
zip_range_3090,city_br_3228,16800-000,16849-999
zip_range_3091,city_br_3229,44745-000,44749-999
zip_range_3092,city_br_3230,77660-000,77664-999
zip_range_3093,city_br_3231,45255-000,45259-999
zip_range_3094,city_br_3232,76926-000,76927-999
zip_range_3095,city_br_3233,19260-000,19272-999
zip_range_3096,city_br_3234,86615-000,86617-999
zip_range_3097,city_br_3235,15130-000,15139-999
zip_range_3098,city_br_3236,78280-000,78284-999
zip_range_3099,city_br_3237,15145-000,15149-999
zip_range_3100,city_br_3238,39465-000,39466-999
zip_range_3101,city_br_3239,89194-000,89195-999
zip_range_3102,city_br_3240,65265-000,65266-999
zip_range_3103,city_br_3241,85890-000,85891-999
zip_range_3104,city_br_3242,63200-000,63209-999
zip_range_3105,city_br_3243,68420-000,68429-999
zip_range_3106,city_br_3244,13730-001,13759-999
zip_range_3107,city_br_3245,89872-000,89872-999
zip_range_3108,city_br_3246,35470-000,35472-999
zip_range_3109,city_br_3247,35604-000,35605-999
zip_range_3110,city_br_3248,58375-000,58377-999
zip_range_3111,city_br_050,08700-001,08899-999
zip_range_3112,city_br_196,13840-001,13856-999
zip_range_3113,city_br_3249,13800-001,13819-999
zip_range_3114,city_br_3250,76135-000,76139-999
zip_range_3115,city_br_3251,49560-000,49564-999
zip_range_3116,city_br_3252,68450-000,68454-999
zip_range_3117,city_br_3253,68129-000,68129-999
zip_range_3118,city_br_3254,63610-000,63619-999
zip_range_3119,city_br_3255,13375-000,13379-999
zip_range_3120,city_br_3256,65360-000,65362-999
zip_range_3121,city_br_3257,15275-000,15279-999
zip_range_3122,city_br_3258,89893-000,89894-999
zip_range_3123,city_br_3259,11730-000,11739-999
zip_range_3124,city_br_3260,39215-000,39218-999
zip_range_3125,city_br_3261,64450-000,64452-999
zip_range_3126,city_br_3262,64650-000,64654-999
zip_range_3127,city_br_3263,37405-000,37406-999
zip_range_3128,city_br_3264,63780-000,63799-999
zip_range_3129,city_br_3265,58145-000,58149-999
zip_range_3130,city_br_3266,39495-000,39499-999
zip_range_3131,city_br_3267,29890-000,29899-999
zip_range_3132,city_br_3268,59198-000,59199-999
zip_range_3133,city_br_3269,99255-000,99259-999
zip_range_3134,city_br_3270,68220-000,68229-999
zip_range_3135,city_br_3271,59182-000,59183-999
zip_range_3136,city_br_3272,73830-000,73839-999
zip_range_3137,city_br_3273,38475-000,38479-999
zip_range_3138,city_br_3274,49690-000,49699-999
zip_range_3139,city_br_3275,64940-000,64944-999
zip_range_3140,city_br_3276,13910-000,13919-999
zip_range_3141,city_br_3277,95236-000,95239-999
zip_range_3142,city_br_3278,15910-000,15919-999
zip_range_3143,city_br_3279,15150-000,15154-999
zip_range_3144,city_br_3280,39500-000,39504-999
zip_range_3145,city_br_3281,14730-000,14734-999
zip_range_3146,city_br_3282,37115-000,37119-999
zip_range_3147,city_br_3283,95718-000,95719-999
zip_range_3148,city_br_3284,89618-000,89619-999
zip_range_3149,city_br_3285,38500-000,38509-999
zip_range_3150,city_br_3286,89380-000,89399-999
zip_range_3151,city_br_3287,17960-000,17969-999
zip_range_3152,city_br_3288,59217-000,59217-999
zip_range_3153,city_br_3289,77585-000,77589-999
zip_range_3154,city_br_3290,39893-000,39894-999
zip_range_3155,city_br_3291,58950-000,58954-999
zip_range_3156,city_br_3292,13190-000,13199-999
zip_range_3157,city_br_3293,76888-000,76888-999
zip_range_3158,city_br_3294,48800-000,48829-999
zip_range_3159,city_br_3295,37968-000,37969-999
zip_range_3160,city_br_3296,77673-000,77674-999
zip_range_3161,city_br_3297,37580-000,37581-999
zip_range_3162,city_br_3298,58500-000,58509-999
zip_range_3163,city_br_3299,12250-000,12259-999
zip_range_3164,city_br_3300,57440-000,57441-999
zip_range_3165,city_br_3301,95780-000,95782-999
zip_range_3166,city_br_3302,65936-000,65936-999
zip_range_3167,city_br_058,39400-001,39429-999
zip_range_3168,city_br_3303,76255-000,76259-999
zip_range_3169,city_br_3304,39547-000,39549-999
zip_range_3170,city_br_3305,75915-000,75919-999
zip_range_3171,city_br_3306,76465-000,76469-999
zip_range_3172,city_br_3307,62940-000,62954-999
zip_range_3173,city_br_3308,35628-000,35639-999
zip_range_3174,city_br_3309,62480-000,62499-999
zip_range_3175,city_br_3310,56150-000,56159-999
zip_range_3176,city_br_3311,87370-000,87379-999
zip_range_3177,city_br_3312,54800-000,54999-999
zip_range_3178,city_br_3313,99315-000,99319-999
zip_range_3179,city_br_3314,47580-000,47589-999
zip_range_3180,city_br_3315,83350-000,83369-999
zip_range_3181,city_br_3316,62550-000,62559-999
zip_range_3182,city_br_3317,75650-000,75659-999
zip_range_3183,city_br_3318,95577-000,95579-999
zip_range_3184,city_br_3319,14640-000,14659-999
zip_range_3185,city_br_3320,76355-000,76359-999
zip_range_3186,city_br_3321,64968-000,64969-999
zip_range_3187,city_br_3322,88830-000,88839-999
zip_range_3188,city_br_3323,39248-000,39249-999
zip_range_3189,city_br_3324,44850-000,44879-999
zip_range_3190,city_br_3325,64178-000,64179-999
zip_range_3191,city_br_3326,35875-000,35877-999
zip_range_3192,city_br_3327,88925-000,88929-999
zip_range_3193,city_br_3328,96150-000,96154-999
zip_range_3194,city_br_3329,93990-000,93994-999
zip_range_3195,city_br_3330,65160-000,65164-999
zip_range_3196,city_br_3331,46290-000,46299-999
zip_range_3197,city_br_3332,13260-000,13269-999
zip_range_3198,city_br_3333,76150-000,76151-999
zip_range_3199,city_br_108,59600-001,59649-999
zip_range_3200,city_br_3334,96270-000,96289-999
zip_range_3201,city_br_3335,14835-000,14839-999
zip_range_3202,city_br_3336,76700-000,76709-999
zip_range_3203,city_br_3337,68825-000,68829-999
zip_range_3204,city_br_3338,69340-000,69342-999
zip_range_3205,city_br_3339,62170-000,62179-999
zip_range_3206,city_br_3340,46750-000,46759-999
zip_range_3207,city_br_3341,95970-000,95971-999
zip_range_3208,city_br_3342,45930-000,45939-999
zip_range_3209,city_br_3343,29880-000,29884-999
zip_range_3210,city_br_3344,95230-000,95235-999
zip_range_3211,city_br_3345,99990-000,99999-999
zip_range_3212,city_br_3346,62764-000,62765-999
zip_range_3213,city_br_3347,58354-000,58355-999
zip_range_3214,city_br_3348,44885-000,44889-999
zip_range_3215,city_br_3349,44800-000,44829-999
zip_range_3216,city_br_3350,76530-000,76539-999
zip_range_3217,city_br_3351,79980-000,79984-999
zip_range_3218,city_br_3352,37620-000,37629-999
zip_range_3219,city_br_3353,86760-000,86769-999
zip_range_3220,city_br_3354,44575-000,44579-999
zip_range_3221,city_br_3355,29380-000,29389-999
zip_range_3222,city_br_3356,47115-000,47119-999
zip_range_3223,city_br_3357,29480-000,29489-999
zip_range_3224,city_br_302,36880-001,36892-999
zip_range_3225,city_br_3358,49780-000,49789-999
zip_range_3226,city_br_3359,57820-000,57829-999
zip_range_3227,city_br_3360,64175-000,64177-999
zip_range_3228,city_br_3361,77850-000,77854-999
zip_range_3229,city_br_3362,44340-000,44344-999
zip_range_3230,city_br_3363,16950-000,16979-999
zip_range_3231,city_br_3364,45480-000,45489-999
zip_range_3232,city_br_3365,36955-000,36959-999
zip_range_3233,city_br_3366,76540-000,76549-999
zip_range_3234,city_br_3367,37890-000,37899-999
zip_range_3235,city_br_3368,39718-000,39719-999
zip_range_3236,city_br_3369,19645-000,19679-999
zip_range_3237,city_br_3370,39860-000,39863-999
zip_range_3238,city_br_3371,99470-000,99489-999
zip_range_3239,city_br_3372,35117-000,35117-999
zip_range_3240,city_br_3373,19220-000,19229-999
zip_range_3241,city_br_024,59000-001,59139-999
zip_range_3242,city_br_3374,38658-000,38659-999
zip_range_3243,city_br_3375,37524-000,37526-999
zip_range_3244,city_br_3376,28380-000,28389-999
zip_range_3245,city_br_3377,77370-000,77374-999
zip_range_3246,city_br_3378,12180-000,12199-999
zip_range_3247,city_br_3379,58494-000,58496-999
zip_range_3248,city_br_3380,88370-001,88379-999
zip_range_3249,city_br_3381,79950-000,79954-999
zip_range_3250,city_br_3382,44400-000,44419-999
zip_range_3251,city_br_3383,77895-000,77899-999
zip_range_3252,city_br_3384,55800-000,55804-999
zip_range_3253,city_br_3385,64825-000,64829-999
zip_range_3254,city_br_3386,12960-000,12969-999
zip_range_3255,city_br_3387,36370-000,36389-999
zip_range_3256,city_br_3388,58817-000,58817-999
zip_range_3257,city_br_3389,64415-000,64419-999
zip_range_3258,city_br_3390,76180-000,76189-999
zip_range_3259,city_br_3391,49980-000,49984-999
zip_range_3260,city_br_3392,37250-000,37259-999
zip_range_3261,city_br_3393,75460-000,75469-999
zip_range_3262,city_br_3394,15120-000,15129-999
zip_range_3263,city_br_3395,69140-000,69149-999
zip_range_3264,city_br_3396,15190-000,15199-999
zip_range_3265,city_br_3397,99175-000,99179-999
zip_range_3266,city_br_3398,45440-000,45442-999
zip_range_3267,city_br_204,26500-001,26549-999
zip_range_3268,city_br_3399,65450-000,65454-999
zip_range_3269,city_br_3400,39553-000,39554-999
zip_range_3270,city_br_3401,79220-000,79229-999
zip_range_3271,city_br_3402,15240-000,15249-999
zip_range_3272,city_br_3403,76420-000,76439-999
zip_range_3273,city_br_3404,59164-000,59167-999
zip_range_3274,city_br_044,24000-001,24399-999
zip_range_3275,city_br_3405,78460-000,78469-999
zip_range_3276,city_br_3406,99600-000,99604-999
zip_range_3277,city_br_3407,48870-000,48879-999
zip_range_3278,city_br_3408,69355-000,69357-999
zip_range_3279,city_br_3409,78430-000,78434-999
zip_range_3280,city_br_3410,49540-000,49549-999
zip_range_3281,city_br_3411,49680-000,49689-999
zip_range_3282,city_br_3412,49600-000,49629-999
zip_range_3283,city_br_3413,86680-000,86689-999
zip_range_3284,city_br_3414,49890-000,49899-999
zip_range_3285,city_br_3415,64288-000,64289-999
zip_range_3286,city_br_3416,78170-000,78174-999
zip_range_3287,city_br_156,49150-000,49169-999
zip_range_3288,city_br_3417,64140-000,64144-999
zip_range_3289,city_br_3418,15210-000,15219-999
zip_range_3290,city_br_3419,87790-000,87799-999
zip_range_3291,city_br_3420,95985-000,95989-999
zip_range_3292,city_br_3421,79140-000,79149-999
zip_range_3293,city_br_3422,76345-000,76349-999
zip_range_3294,city_br_3423,86230-000,86239-999
zip_range_3295,city_br_3424,79750-000,79759-999
zip_range_3296,city_br_3425,95350-000,95354-999
zip_range_3297,city_br_3426,75750-000,75759-999
zip_range_3298,city_br_3427,85410-000,85414-999
zip_range_3299,city_br_3428,78565-000,78569-999
zip_range_3300,city_br_3429,95340-000,95344-999
zip_range_3301,city_br_3430,35298-000,35299-999
zip_range_3302,city_br_3431,99580-000,99584-999
zip_range_3303,city_br_3432,78860-000,78869-999
zip_range_3304,city_br_3433,76958-000,76959-999
zip_range_3305,city_br_3434,95950-000,95954-999
zip_range_3306,city_br_3435,18435-000,18439-999
zip_range_3307,city_br_3436,45270-000,45279-999
zip_range_3308,city_br_3437,78515-000,78519-999
zip_range_3309,city_br_3438,15773-000,15774-999
zip_range_3310,city_br_3439,98919-000,98919-999
zip_range_3311,city_br_3440,87330-000,87339-999
zip_range_3312,city_br_3441,15313-000,15314-999
zip_range_3313,city_br_3442,65808-000,65809-999
zip_range_3314,city_br_3443,76520-000,76524-999
zip_range_3315,city_br_3444,59215-000,59216-999
zip_range_3316,city_br_3445,35920-000,35929-999
zip_range_3317,city_br_3446,89865-000,89867-999
zip_range_3318,city_br_3447,87600-000,87629-999
zip_range_3319,city_br_3448,68618-000,68619-999
zip_range_3320,city_br_3449,85635-000,85639-999
zip_range_3321,city_br_3450,97770-000,97799-999
zip_range_3322,city_br_3451,14920-000,14929-999
zip_range_3323,city_br_3452,44642-000,44644-999
zip_range_3324,city_br_3453,86310-000,86314-999
zip_range_3325,city_br_3454,58178-000,58179-999
zip_range_3326,city_br_159,28600-001,28636-999
zip_range_3327,city_br_3455,76305-000,76309-999
zip_range_3328,city_br_3456,15440-000,15449-999
zip_range_3329,city_br_3457,78508-000,78509-999
zip_range_3330,city_br_3458,17950-000,17959-999
zip_range_3331,city_br_3459,93890-000,93899-999
zip_range_3332,city_br_3460,45452-000,45454-999
zip_range_3333,city_br_023,26000-001,26099-999
zip_range_3334,city_br_023,26200-000,26299-999
zip_range_3335,city_br_3461,76495-000,76499-999
zip_range_3336,city_br_3462,16940-000,16949-999
zip_range_3337,city_br_3463,65880-000,65884-999
zip_range_3338,city_br_3464,68585-000,68589-999
zip_range_3339,city_br_3465,89818-000,89818-999
zip_range_3340,city_br_3466,45390-000,45399-999
zip_range_3341,city_br_3467,78243-000,78244-999
zip_range_3342,city_br_3468,85350-000,85389-999
zip_range_3343,city_br_280,34000-001,34019-999
zip_range_3344,city_br_3469,87970-000,87979-999
zip_range_3345,city_br_3470,15340-000,15349-999
zip_range_3346,city_br_3471,76857-000,76859-999
zip_range_3347,city_br_3472,78415-000,78419-999
zip_range_3348,city_br_3473,78445-000,78449-999
zip_range_3349,city_br_3474,35113-000,35113-999
zip_range_3350,city_br_3475,78593-000,78594-999
zip_range_3351,city_br_3476,78450-000,78452-999
zip_range_3352,city_br_3477,78638-000,78639-999
zip_range_3353,city_br_3478,13380-001,13389-999
zip_range_3354,city_br_3479,78370-000,78379-999
zip_range_3355,city_br_3480,87490-000,87499-999
zip_range_3356,city_br_3481,63165-000,63169-999
zip_range_3357,city_br_3482,58798-000,58799-999
zip_range_3358,city_br_3483,77790-000,77794-999
zip_range_3359,city_br_3484,65274-000,65274-999
zip_range_3360,city_br_3485,69230-000,69239-999
zip_range_3361,city_br_3486,95275-000,95279-999
zip_range_3362,city_br_3487,97250-000,97279-999
zip_range_3363,city_br_3488,58184-000,58186-999
zip_range_3364,city_br_3489,95150-000,95174-999
zip_range_3365,city_br_3490,38160-000,38169-999
zip_range_3366,city_br_3491,39525-000,39525-999
zip_range_3367,city_br_3492,95320-000,95324-999
zip_range_3368,city_br_3493,85685-000,85699-999
zip_range_3369,city_br_3494,98758-000,98759-999
zip_range_3370,city_br_3495,46835-000,46839-999
zip_range_3371,city_br_3496,37860-000,37879-999
zip_range_3372,city_br_3497,73820-000,73824-999
zip_range_3373,city_br_3498,95260-000,95269-999
zip_range_3374,city_br_3499,77495-000,77499-999
zip_range_3375,city_br_3500,62200-000,62209-999
zip_range_3376,city_br_3501,86250-000,86269-999
zip_range_3377,city_br_3502,78548-000,78549-999
zip_range_3378,city_br_3503,64764-000,64764-999
zip_range_3379,city_br_3504,92480-000,92499-999
zip_range_3380,city_br_3505,85930-000,85932-999
zip_range_3381,city_br_296,35517-000,35529-999
zip_range_3382,city_br_3506,48460-000,48469-999
zip_range_3383,city_br_3507,85250-000,85259-999
zip_range_3384,city_br_3508,68730-000,68733-999
zip_range_3385,city_br_3509,88270-000,88294-999
zip_range_3386,city_br_3510,78888-000,78889-999
zip_range_3387,city_br_3511,34990-000,34999-999
zip_range_3388,city_br_3512,76924-000,76925-999
zip_range_3389,city_br_3513,29830-000,29842-999
zip_range_3390,city_br_3514,75470-000,75479-999
zip_range_3391,city_br_3515,88865-000,88869-999
zip_range_3392,city_br_3516,45920-000,45929-999
zip_range_3393,city_br_3517,78690-000,78694-999
zip_range_3394,city_br_3518,15885-000,15889-999
zip_range_3395,city_br_3519,77610-000,77612-999
zip_range_3396,city_br_3520,69730-000,69734-999
zip_range_3397,city_br_3521,77353-000,77359-999
zip_range_3398,city_br_3522,69260-000,69264-999
zip_range_3399,city_br_3523,98338-000,98339-999
zip_range_3400,city_br_3524,76285-000,76289-999
zip_range_3401,city_br_3525,96545-000,96569-999
zip_range_3402,city_br_3526,39820-000,39824-999
zip_range_3403,city_br_305,72860-001,72869-999
zip_range_3404,city_br_133,93300-001,93599-999
zip_range_3405,city_br_3527,46730-000,46739-999
zip_range_3406,city_br_3528,89998-000,89999-999
zip_range_3407,city_br_3529,14960-000,14979-999
zip_range_3408,city_br_3530,78570-000,78572-999
zip_range_3409,city_br_3531,76956-000,76957-999
zip_range_3410,city_br_3532,79745-000,79749-999
zip_range_3411,city_br_3533,86895-000,86899-999
zip_range_3412,city_br_3534,77318-000,77319-999
zip_range_3413,city_br_3535,57970-000,57974-999
zip_range_3414,city_br_3536,98955-000,98957-999
zip_range_3415,city_br_3537,78528-000,78529-999
zip_range_3416,city_br_3538,63740-000,63749-999
zip_range_3417,city_br_3539,39817-000,39817-999
zip_range_3418,city_br_3540,64530-000,64534-999
zip_range_3419,city_br_3541,76580-000,76589-999
zip_range_3420,city_br_3542,68193-000,68194-999
zip_range_3421,city_br_3543,68473-000,68474-999
zip_range_3422,city_br_3544,78674-000,78674-999
zip_range_3423,city_br_3545,64365-000,64369-999
zip_range_3424,city_br_3546,78625-000,78627-999
zip_range_3425,city_br_3547,98370-000,98379-999
zip_range_3426,city_br_3548,48455-000,48459-999
zip_range_3427,city_br_3549,99687-000,99689-999
zip_range_3428,city_br_3550,39568-000,39568-999
zip_range_3429,city_br_3551,14670-000,14679-999
zip_range_3430,city_br_3552,68250-000,68269-999
zip_range_3431,city_br_3553,62755-000,62759-999
zip_range_3432,city_br_3554,17540-000,17549-999
zip_range_3433,city_br_3555,64500-000,64509-999
zip_range_3434,city_br_3556,68470-000,68472-999
zip_range_3435,city_br_3557,68980-000,68989-999
zip_range_3436,city_br_3558,36145-000,36145-999
zip_range_3437,city_br_3559,18790-000,18799-999
zip_range_3438,city_br_3560,58760-000,58762-999
zip_range_3439,city_br_3561,65706-000,65706-999
zip_range_3440,city_br_3562,57442-000,57444-999
zip_range_3441,city_br_3563,59730-000,59739-999
zip_range_3442,city_br_3564,57470-000,57474-999
zip_range_3443,city_br_3565,64468-000,64469-999
zip_range_3444,city_br_3566,57390-000,57399-999
zip_range_3445,city_br_3567,39398-000,39399-999
zip_range_3446,city_br_3568,15400-000,15409-999
zip_range_3447,city_br_3569,37488-000,37489-999
zip_range_3448,city_br_076,53000-001,53399-999
zip_range_3449,city_br_3570,65223-000,65224-999
zip_range_3450,city_br_3571,48470-000,48474-999
zip_range_3451,city_br_3572,58160-000,58166-999
zip_range_3452,city_br_3573,35540-000,35542-999
zip_range_3453,city_br_3574,77558-000,77559-999
zip_range_3454,city_br_3575,47530-000,47559-999
zip_range_3455,city_br_3576,36250-000,36254-999
zip_range_3456,city_br_3577,57550-000,57559-999
zip_range_3457,city_br_3578,35655-000,35656-999
zip_range_3458,city_br_3579,15450-000,15459-999
zip_range_3459,city_br_3580,35439-000,35439-999
zip_range_3460,city_br_3581,17570-000,17579-999
zip_range_3461,city_br_3582,15480-000,15489-999
zip_range_3462,city_br_3583,68270-000,68279-999
zip_range_3463,city_br_3584,36828-000,36829-999
zip_range_3464,city_br_3585,75280-000,75339-999
zip_range_3465,city_br_3586,14620-000,14639-999
zip_range_3466,city_br_3587,88870-000,88879-999
zip_range_3467,city_br_3588,55745-000,55749-999
zip_range_3468,city_br_3589,56170-000,56179-999
zip_range_3469,city_br_3590,63520-000,63529-999
zip_range_3470,city_br_3591,84350-000,84399-999
zip_range_3471,city_br_026,06000-001,06299-999
zip_range_3472,city_br_3592,19770-000,19779-999
zip_range_3473,city_br_3593,95520-000,95529-999
zip_range_3474,city_br_3594,17700-000,17709-999
zip_range_3475,city_br_3595,88540-000,88542-999
zip_range_3476,city_br_3596,68640-000,68643-999
zip_range_3477,city_br_3597,48150-000,48169-999
zip_range_3478,city_br_3598,56200-000,56209-999
zip_range_3479,city_br_3599,68390-000,68397-999
zip_range_3480,city_br_304,19900-001,19919-999
zip_range_3481,city_br_3600,87170-000,87174-999
zip_range_3482,city_br_3601,89663-000,89664-999
zip_range_3483,city_br_3602,57525-000,57529-999
zip_range_3484,city_br_3603,36420-000,36421-999
zip_range_3485,city_br_3604,59347-000,59349-999
zip_range_3486,city_br_3605,37570-000,37575-999
zip_range_3487,city_br_3606,35400-000,35419-999
zip_range_3488,city_br_3607,76920-000,76922-999
zip_range_3489,city_br_3608,58560-000,58569-999
zip_range_3490,city_br_3609,89834-000,89834-999
zip_range_3491,city_br_3610,17920-000,17929-999
zip_range_3492,city_br_3611,75165-000,75169-999
zip_range_3493,city_br_3612,39855-000,39859-999
zip_range_3494,city_br_3613,85933-000,85934-999
zip_range_3495,city_br_3614,15685-000,15689-999
zip_range_3496,city_br_3615,44718-000,44719-999
zip_range_3497,city_br_3616,75715-000,75719-999
zip_range_3498,city_br_3617,17860-000,17869-999
zip_range_3499,city_br_3618,68485-000,68487-999
zip_range_3500,city_br_3619,62870-000,62874-999
zip_range_3501,city_br_3620,69345-000,69347-999
zip_range_3502,city_br_3621,61800-001,61879-999
zip_range_3503,city_br_3622,49970-000,49979-999
zip_range_3504,city_br_206,65130-000,65137-999
zip_range_3505,city_br_3623,62770-000,62779-999
zip_range_3506,city_br_3624,62180-000,62183-999
zip_range_3507,city_br_3625,73700-000,73729-999
zip_range_3508,city_br_3626,39573-000,39574-999
zip_range_3509,city_br_3627,64680-000,64682-999
zip_range_3510,city_br_3628,39818-000,39819-999
zip_range_3511,city_br_3629,64710-000,64719-999
zip_range_3512,city_br_3630,39517-000,39517-999
zip_range_3513,city_br_3631,89765-000,89769-999
zip_range_3514,city_br_3632,87140-000,87154-999
zip_range_3515,city_br_3633,99850-000,99854-999
zip_range_3516,city_br_3634,35622-000,35623-999
zip_range_3517,city_br_3635,88543-000,88544-999
zip_range_3518,city_br_3636,35582-000,35584-999
zip_range_3519,city_br_3637,36195-000,36199-999
zip_range_3520,city_br_3638,64898-000,64899-999
zip_range_3521,city_br_3639,57410-000,57419-999
zip_range_3522,city_br_3640,15470-000,15479-999
zip_range_3523,city_br_3641,75845-000,75849-999
zip_range_3524,city_br_3642,68535-000,68536-999
zip_range_3525,city_br_3643,62910-000,62919-999
zip_range_3526,city_br_141,88130-001,88139-999
zip_range_3527,city_br_3644,36750-000,36759-999
zip_range_3528,city_br_3645,89985-000,89989-999
zip_range_3529,city_br_3646,62780-000,62784-999
zip_range_3530,city_br_3647,55540-000,55549-999
zip_range_3531,city_br_3648,95540-000,95551-999
zip_range_3532,city_br_3649,15828-000,15829-999
zip_range_3533,city_br_3650,85555-000,85556-999
zip_range_3534,city_br_091,77000-001,77299-999
zip_range_3535,city_br_3651,46460-000,46469-999
zip_range_3536,city_br_3652,84130-000,84139-999
zip_range_3537,city_br_3653,88545-000,88547-999
zip_range_3538,city_br_3654,15720-000,15729-999
zip_range_3539,city_br_3655,98300-000,98319-999
zip_range_3540,city_br_3656,64925-000,64929-999
zip_range_3541,city_br_3657,57600-001,57614-999
zip_range_3542,city_br_3658,64420-000,64429-999
zip_range_3543,city_br_3659,65238-000,65244-999
zip_range_3544,city_br_3660,77798-000,77799-999
zip_range_3545,city_br_3661,46930-000,46959-999
zip_range_3546,city_br_3662,76190-000,76194-999
zip_range_3547,city_br_3663,77913-000,77914-999
zip_range_3548,city_br_3664,55310-000,55314-999
zip_range_3549,city_br_3665,77365-000,77367-999
zip_range_3550,city_br_3666,75210-000,75219-999
zip_range_3551,city_br_3667,75990-000,76099-999
zip_range_3552,city_br_3668,85270-000,85274-999
zip_range_3553,city_br_3669,19970-000,19989-999
zip_range_3554,city_br_3670,98430-000,98434-999
zip_range_3555,city_br_3671,89887-000,89887-999
zip_range_3556,city_br_3672,39945-000,39949-999
zip_range_3557,city_br_3673,85950-000,85954-999
zip_range_3558,city_br_3674,75580-000,75599-999
zip_range_3559,city_br_3675,98280-000,98289-999
zip_range_3560,city_br_3676,29750-000,29759-999
zip_range_3561,city_br_3677,55470-000,55479-999
zip_range_3562,city_br_3678,17980-000,17989-999
zip_range_3563,city_br_3679,96690-000,96699-999
zip_range_3564,city_br_3680,57400-000,57409-999
zip_range_3565,city_br_3681,35669-000,35669-999
zip_range_3566,city_br_3682,89370-000,89379-999
zip_range_3567,city_br_3683,64618-000,64619-999
zip_range_3568,city_br_3684,35660-001,35665-999
zip_range_3569,city_br_3685,26600-000,26649-999
zip_range_3570,city_br_3686,38600-000,38609-999
zip_range_3571,city_br_3687,62680-000,62684-999
zip_range_3572,city_br_297,68625-001,68631-999
zip_range_3573,city_br_3688,37120-000,37129-999
zip_range_3574,city_br_3689,19700-000,19739-999
zip_range_3575,city_br_3690,95360-000,95364-999
zip_range_3576,city_br_3691,25850-000,25869-999
zip_range_3577,city_br_3692,65670-000,65679-999
zip_range_3578,city_br_3693,12260-000,12269-999
zip_range_3579,city_br_3694,62685-000,62689-999
zip_range_3580,city_br_3695,89906-000,89907-999
zip_range_3581,city_br_3696,15825-000,15827-999
zip_range_3582,city_br_3697,79556-000,79559-999
zip_range_3583,city_br_3698,87780-000,87789-999
zip_range_3584,city_br_3699,96530-000,96534-999
zip_range_3585,city_br_3700,77600-000,77602-999
zip_range_3586,city_br_3701,37660-000,37669-999
zip_range_3587,city_br_3702,63680-000,63699-999
zip_range_3588,city_br_3703,46190-000,46199-999
zip_range_3589,city_br_3704,62736-000,62737-999
zip_range_3590,city_br_3705,59950-000,59954-999
zip_range_3591,city_br_3706,77360-000,77364-999
zip_range_3592,city_br_3707,87660-000,87669-999
zip_range_3593,city_br_205,83200-001,83254-999
zip_range_3594,city_br_3708,79500-000,79529-999
zip_range_3595,city_br_3709,75880-000,75889-999
zip_range_3596,city_br_3710,78590-000,78592-999
zip_range_3597,city_br_3711,18720-000,18729-999
zip_range_3598,city_br_3712,87680-000,87689-999
zip_range_3599,city_br_3713,15745-000,15747-999
zip_range_3600,city_br_3714,55355-000,55359-999
zip_range_3601,city_br_3715,78870-000,78874-999
zip_range_3602,city_br_3716,87700-001,87729-999
zip_range_3603,city_br_3717,79925-000,79929-999
zip_range_3604,city_br_3718,35774-000,35776-999
zip_range_3605,city_br_3719,17730-000,17739-999
zip_range_3606,city_br_3720,58575-000,58579-999
zip_range_3607,city_br_3721,47500-000,47519-999
zip_range_3608,city_br_3722,23970-000,23999-999
zip_range_3609,city_br_3723,59660-000,59662-999
zip_range_3610,city_br_106,68515-000,68517-999
zip_range_3611,city_br_3724,75980-000,75984-999
zip_range_3612,city_br_3725,59586-000,59587-999
zip_range_3613,city_br_3726,18640-000,18649-999
zip_range_3614,city_br_3727,95783-000,95784-999
zip_range_3615,city_br_3728,76979-000,76979-999
zip_range_3616,city_br_3729,59360-000,59369-999
zip_range_3617,city_br_3730,57475-000,57479-999
zip_range_3618,city_br_3731,69150-001,69159-999
zip_range_3619,city_br_3732,48430-000,48434-999
zip_range_3620,city_br_3733,57935-000,57939-999
zip_range_3621,city_br_3734,11930-000,11939-999
zip_range_3622,city_br_3735,15525-000,15529-999
zip_range_3623,city_br_3736,64970-000,64974-999
zip_range_3624,city_br_184,64200-001,64219-999
zip_range_3625,city_br_3737,56163-000,56169-999
zip_range_3626,city_br_115,59140-001,59161-999
zip_range_3627,city_br_3738,65640-000,65644-999
zip_range_3628,city_br_3739,95630-000,95649-999
zip_range_3629,city_br_3740,59218-000,59218-999
zip_range_3630,city_br_3741,37460-000,37463-999
zip_range_3631,city_br_3742,96908-000,96909-999
zip_range_3632,city_br_3743,35537-000,35539-999
zip_range_3633,city_br_3744,37330-000,37339-999
zip_range_3634,city_br_3745,35810-000,35814-999
zip_range_3635,city_br_3746,58734-000,58734-999
zip_range_3636,city_br_3747,59259-000,59259-999
zip_range_3637,city_br_3748,65680-000,65682-999
zip_range_3638,city_br_3749,64395-000,64399-999
zip_range_3639,city_br_3750,55650-000,55654-999
zip_range_3640,city_br_3751,57930-000,57934-999
zip_range_3641,city_br_3752,88980-000,88989-999
zip_range_3642,city_br_3753,96685-000,96689-999
zip_range_3643,city_br_147,99000-001,99139-999
zip_range_3644,city_br_279,37900-001,37904-999
zip_range_3645,city_br_3754,89687-000,89689-999
zip_range_3646,city_br_3755,65870-000,65879-999
zip_range_3647,city_br_3756,39378-000,39379-999
zip_range_3648,city_br_3757,85948-000,85949-999
zip_range_3649,city_br_3758,85500-001,85514-999
zip_range_3650,city_br_311,58700-001,58709-999
zip_range_3651,city_br_186,38700-001,38719-999
zip_range_3652,city_br_3759,64580-000,64584-999
zip_range_3653,city_br_3760,38740-001,38749-999
zip_range_3654,city_br_3761,36860-000,36869-999
zip_range_3655,city_br_3762,14415-000,14419-999
zip_range_3656,city_br_3763,59770-000,59774-999
zip_range_3657,city_br_3764,26950-000,26999-999
zip_range_3658,city_br_3765,45890-000,45899-999
zip_range_3659,city_br_3766,68545-000,68547-999
zip_range_3660,city_br_3767,77785-000,77789-999
zip_range_3661,city_br_3768,64295-000,64299-999
zip_range_3662,city_br_3769,59900-000,59901-999
zip_range_3663,city_br_3770,55825-000,55834-999
zip_range_3664,city_br_3771,69860-000,69869-999
zip_range_3665,city_br_3772,36544-000,36545-999
zip_range_3666,city_br_3773,84630-000,84634-999
zip_range_3667,city_br_3774,17990-000,17999-999
zip_range_3668,city_br_284,13140-001,13149-999
zip_range_3669,city_br_3775,65585-000,65589-999
zip_range_3670,city_br_3776,58860-000,58864-999
zip_range_3671,city_br_080,53400-001,53499-999
zip_range_3672,city_br_3777,64750-000,64752-999
zip_range_3673,city_br_3778,17150-000,17159-999
zip_range_3674,city_br_3779,39765-000,39769-999
zip_range_3675,city_br_276,48600-001,48619-999
zip_range_3676,city_br_3780,99718-000,99719-999
zip_range_3677,city_br_3781,15490-000,15494-999
zip_range_3678,city_br_3782,84635-000,84639-999
zip_range_3679,city_br_3783,57740-000,57749-999
zip_range_3680,city_br_3784,88490-000,88494-999
zip_range_3681,city_br_3785,65716-000,65717-999
zip_range_3682,city_br_3786,39814-000,39814-999
zip_range_3683,city_br_3787,95865-000,95869-999
zip_range_3684,city_br_3788,64838-000,64839-999
zip_range_3685,city_br_3789,44655-000,44659-999
zip_range_3686,city_br_3790,87250-000,87259-999
zip_range_3687,city_br_3791,39700-000,39702-999
zip_range_3688,city_br_3792,17280-000,17289-999
zip_range_3689,city_br_3793,55280-000,55289-999
zip_range_3690,city_br_3794,39970-000,39979-999
zip_range_3691,city_br_3795,12990-000,12994-999
zip_range_3692,city_br_3796,35364-000,35364-999
zip_range_3693,city_br_3797,63630-000,63634-999
zip_range_3694,city_br_3798,58790-000,58794-999
zip_range_3695,city_br_3799,68945-000,68947-999
zip_range_3696,city_br_3800,36585-000,36589-999
zip_range_3697,city_br_3801,35565-000,35566-999
zip_range_3698,city_br_3802,36847-000,36849-999
zip_range_3699,city_br_3803,59588-000,59589-999
zip_range_3700,city_br_3804,58180-000,58183-999
zip_range_3701,city_br_3805,49512-000,49513-999
zip_range_3702,city_br_3806,78795-000,78799-999
zip_range_3703,city_br_3807,59547-000,59549-999
zip_range_3704,city_br_3808,37520-000,37523-999
zip_range_3705,city_br_3809,15630-000,15639-999
zip_range_3706,city_br_3810,48140-000,48149-999
zip_range_3707,city_br_3811,96487-000,96489-999
zip_range_3708,city_br_3812,58328-000,58329-999
zip_range_3709,city_br_3813,39492-000,39494-999
zip_range_3710,city_br_3814,88720-000,88729-999
zip_range_3711,city_br_3815,14470-000,14489-999
zip_range_3712,city_br_3816,13920-000,13929-999
zip_range_3713,city_br_3817,65725-000,65726-999
zip_range_3714,city_br_3818,49350-000,49359-999
zip_range_3715,city_br_3819,19865-000,19869-999
zip_range_3716,city_br_3820,38178-000,38179-999
zip_range_3717,city_br_3821,77710-000,77713-999
zip_range_3718,city_br_3822,48580-000,48589-999
zip_range_3719,city_br_3823,59530-000,59534-999
zip_range_3720,city_br_3824,29970-000,29979-999
zip_range_3721,city_br_3825,11790-000,11799-999
zip_range_3722,city_br_3826,65206-000,65207-999
zip_range_3723,city_br_3827,79410-000,79414-999
zip_range_3724,city_br_3828,64255-000,64257-999
zip_range_3725,city_br_3829,64728-000,64729-999
zip_range_3726,city_br_3830,33600-000,33799-999
zip_range_3727,city_br_3831,96360-000,96394-999
zip_range_3728,city_br_3832,58273-000,58274-999
zip_range_3729,city_br_3833,36148-000,36149-999
zip_range_3730,city_br_3834,59196-000,59197-999
zip_range_3731,city_br_3835,77460-000,77462-999
zip_range_3732,city_br_3836,68734-000,68737-999
zip_range_3733,city_br_3837,78530-000,78534-999
zip_range_3734,city_br_3838,98270-000,98279-999
zip_range_3735,city_br_086,96000-001,96147-999
zip_range_3736,city_br_3839,63280-000,63289-999
zip_range_3737,city_br_3840,65213-000,65214-999
zip_range_3738,city_br_3841,16300-001,16309-999
zip_range_3739,city_br_3842,59504-000,59506-999
zip_range_3740,city_br_3843,57200-000,57209-999
zip_range_3741,city_br_3844,88385-000,88389-999
zip_range_3742,city_br_3845,62640-000,62649-999
zip_range_3743,city_br_3846,36610-000,36619-999
zip_range_3744,city_br_3847,35667-000,35667-999
zip_range_3745,city_br_3848,77730-000,77732-999
zip_range_3746,city_br_3849,35545-000,35546-999
zip_range_3747,city_br_3850,38170-000,38174-999
zip_range_3748,city_br_3851,37260-000,37261-999
zip_range_3749,city_br_3852,15370-000,15379-999
zip_range_3750,city_br_3853,18580-000,18589-999
zip_range_3751,city_br_3854,63460-000,63469-999
zip_range_3752,city_br_3855,65245-000,65247-999
zip_range_3753,city_br_3856,35118-000,35119-999
zip_range_3754,city_br_3857,89750-000,89759-999
zip_range_3755,city_br_3858,65418-000,65419-999
zip_range_3756,city_br_3859,87538-000,87539-999
zip_range_3757,city_br_3860,87540-000,87544-999
zip_range_3758,city_br_3861,85740-000,85744-999
zip_range_3759,city_br_3862,75823-000,75824-999
zip_range_3760,city_br_3863,11750-000,11759-999
zip_range_3761,city_br_3864,35114-000,35114-999
zip_range_3762,city_br_3865,88798-000,88799-999
zip_range_3763,city_br_3866,55200-000,55239-999
zip_range_3764,city_br_3867,56460-000,56469-999
zip_range_3765,city_br_3868,88430-000,88439-999
zip_range_3766,city_br_065,56300-001,56354-999
zip_range_3767,city_br_3869,75480-000,75489-999
zip_range_3768,city_br_099,25600-001,25779-999
zip_range_3769,city_br_3870,57210-000,57219-999
zip_range_3770,city_br_3871,16230-000,16239-999
zip_range_3771,city_br_3872,58765-000,58769-999
zip_range_3772,city_br_3873,46765-000,46769-999
zip_range_3773,city_br_3874,36157-000,36159-999
zip_range_3774,city_br_3875,95175-000,95179-999
zip_range_3775,city_br_3876,68575-000,68579-999
zip_range_3776,city_br_3877,64600-001,64609-999
zip_range_3777,city_br_3878,58187-000,58187-999
zip_range_3778,city_br_3879,18170-000,18179-999
zip_range_3779,city_br_3880,35325-000,35325-999
zip_range_3780,city_br_3881,35382-000,35382-999
zip_range_3781,city_br_3882,36227-000,36229-999
zip_range_3782,city_br_3883,35476-000,35477-999
zip_range_3783,city_br_3884,83860-000,83869-999
zip_range_3784,city_br_3885,47240-000,47299-999
zip_range_3785,city_br_3886,57150-000,57159-999
zip_range_3786,city_br_3887,58338-000,58338-999
zip_range_3787,city_br_3888,76372-000,76372-999
zip_range_3788,city_br_3889,18185-000,18189-999
zip_range_3789,city_br_3890,58393-000,58393-999
zip_range_3790,city_br_3891,59960-000,59964-999
zip_range_3791,city_br_3892,58210-000,58212-999
zip_range_3792,city_br_3893,35585-000,35587-999
zip_range_3793,city_br_3894,76970-000,76973-999
zip_range_3794,city_br_3895,64320-000,64324-999
zip_range_3795,city_br_3896,76999-000,76999-999
zip_range_3796,city_br_3897,46360-000,46379-999
zip_range_3797,city_br_179,12400-001,12449-999
zip_range_3798,city_br_3898,65370-000,65377-999
zip_range_3799,city_br_3899,57720-000,57729-999
zip_range_3800,city_br_3900,44770-000,44774-999
zip_range_3801,city_br_3901,15830-000,15839-999
zip_range_3802,city_br_3902,77380-000,77389-999
zip_range_3803,city_br_3903,62860-000,62869-999
zip_range_3804,city_br_3904,35348-000,35349-999
zip_range_3805,city_br_237,83320-001,83349-999
zip_range_3806,city_br_3905,98345-000,98349-999
zip_range_3807,city_br_3906,95390-000,95399-999
zip_range_3808,city_br_3907,85727-000,85729-999
zip_range_3809,city_br_3908,98150-000,98159-999
zip_range_3810,city_br_3909,84925-000,84929-999
zip_range_3811,city_br_3910,89870-000,89870-999
zip_range_3812,city_br_3911,12995-000,12999-999
zip_range_3813,city_br_3912,85170-000,85194-999
zip_range_3814,city_br_3913,49517-000,49519-999
zip_range_3815,city_br_3914,27197-000,27199-999
zip_range_3816,city_br_3915,98435-000,98439-999
zip_range_3817,city_br_3916,65200-000,65203-999
zip_range_3818,city_br_3917,96470-000,96486-999
zip_range_3819,city_br_3918,89570-000,89579-999
zip_range_3820,city_br_3919,29980-000,29999-999
zip_range_3821,city_br_3920,44610-000,44619-999
zip_range_3822,city_br_3921,95717-000,95717-999
zip_range_3823,city_br_3922,39317-000,39317-999
zip_range_3824,city_br_3923,64660-000,64669-999
zip_range_3825,city_br_3924,65707-000,65707-999
zip_range_3826,city_br_3925,19410-000,19429-999
zip_range_3827,city_br_3926,63605-000,63609-999
zip_range_3828,city_br_3927,12620-000,12629-999
zip_range_3829,city_br_3928,12970-000,12979-999
zip_range_3830,city_br_3929,75640-000,75644-999
zip_range_3831,city_br_3930,35536-000,35536-999
zip_range_3832,city_br_054,13400-001,13439-999
zip_range_3833,city_br_3931,64240-000,64242-999
zip_range_3834,city_br_3932,27175-000,27196-999
zip_range_3835,city_br_3933,45436-000,45439-999
zip_range_3836,city_br_3934,84240-000,84249-999
zip_range_3837,city_br_3935,18800-000,18829-999
zip_range_3838,city_br_3936,38210-000,38219-999
zip_range_3839,city_br_3937,16600-000,16639-999
zip_range_3840,city_br_3938,49190-000,49199-999
zip_range_3841,city_br_3939,36480-000,36499-999
zip_range_3842,city_br_3940,15820-000,15822-999
zip_range_3843,city_br_3941,37511-000,37511-999
zip_range_3844,city_br_3942,37508-000,37509-999
zip_range_3845,city_br_3943,57460-000,57469-999
zip_range_3846,city_br_3944,76230-000,76234-999
zip_range_3847,city_br_3945,65460-000,65464-999
zip_range_3848,city_br_3946,36730-000,36739-999
zip_range_3849,city_br_3947,97885-000,97899-999
zip_range_3850,city_br_3948,39270-000,39279-999
zip_range_3851,city_br_3949,06550-000,06599-999
zip_range_3852,city_br_3950,19200-000,19209-999
zip_range_3853,city_br_256,83300-001,83319-999
zip_range_3854,city_br_3951,77888-000,77889-999
zip_range_3855,city_br_3952,13630-001,13649-999
zip_range_3856,city_br_3953,96490-000,96494-999
zip_range_3857,city_br_3954,17490-000,17499-999
zip_range_3858,city_br_3955,89667-000,89668-999
zip_range_3859,city_br_3956,36170-000,36179-999
zip_range_3860,city_br_3957,72980-000,72989-999
zip_range_3861,city_br_3958,75200-000,75209-999
zip_range_3862,city_br_3959,62255-000,62259-999
zip_range_3863,city_br_3960,46270-000,46279-999
zip_range_3864,city_br_3961,64260-000,64264-999
zip_range_3865,city_br_3962,44830-000,44839-999
zip_range_3866,city_br_3963,58213-000,58219-999
zip_range_3867,city_br_3964,85200-000,85224-999
zip_range_3868,city_br_3965,86613-000,86614-999
zip_range_3869,city_br_3966,14750-000,14764-999
zip_range_3870,city_br_3967,35650-000,35654-999
zip_range_3871,city_br_3968,58324-000,58325-999
zip_range_3872,city_br_3969,77570-000,77574-999
zip_range_3873,city_br_3970,29285-000,29289-999
zip_range_3874,city_br_3971,37925-000,37925-999
zip_range_3875,city_br_3972,68138-000,68139-999
zip_range_3876,city_br_3973,69928-000,69929-999
zip_range_3877,city_br_299,73750-001,73759-999
zip_range_3878,city_br_3974,87860-000,87879-999
zip_range_3879,city_br_3975,45375-000,45389-999
zip_range_3880,city_br_3976,45190-000,45199-999
zip_range_3881,city_br_3977,85750-000,85759-999
zip_range_3882,city_br_3978,98470-000,98479-999
zip_range_3883,city_br_3979,15260-000,15264-999
zip_range_3884,city_br_3980,89882-000,89882-999
zip_range_3885,city_br_3981,78855-000,78859-999
zip_range_3886,city_br_3982,38220-000,38229-999
zip_range_3887,city_br_3983,19990-000,19999-999
zip_range_3888,city_br_306,08550-001,08569-999
zip_range_3889,city_br_3984,55240-000,55249-999
zip_range_3890,city_br_3985,65740-000,65749-999
zip_range_3891,city_br_3986,58150-000,58154-999
zip_range_3892,city_br_3987,59560-000,59564-999
zip_range_3893,city_br_3988,58933-000,58934-999
zip_range_3894,city_br_3989,95740-000,95744-999
zip_range_3895,city_br_3990,57510-000,57514-999
zip_range_3896,city_br_3991,58908-000,58909-999
zip_range_3897,city_br_3992,37757-000,37759-999
zip_range_3898,city_br_3993,49810-000,49819-999
zip_range_3899,city_br_3994,49490-000,49499-999
zip_range_3900,city_br_3995,45260-000,45262-999
zip_range_3901,city_br_3996,78175-000,78179-999
zip_range_3902,city_br_183,37700-001,37719-999
zip_range_3903,city_br_3997,36960-000,36969-999
zip_range_3904,city_br_3998,48120-000,48129-999
zip_range_3905,city_br_3999,15160-000,15169-999
zip_range_3906,city_br_4000,58840-000,58852-999
zip_range_3907,city_br_4001,55630-000,55635-999
zip_range_3908,city_br_4002,89107-000,89107-999
zip_range_3909,city_br_4003,17580-000,17589-999
zip_range_3910,city_br_4004,35640-000,35649-999
zip_range_3911,city_br_4005,16660-000,16669-999
zip_range_3912,city_br_4006,68830-000,68839-999
zip_range_3913,city_br_072,84000-001,84129-999
zip_range_3914,city_br_4007,79900-001,79909-999
zip_range_3915,city_br_4008,14180-000,14199-999
zip_range_3916,city_br_4009,78698-000,78699-999
zip_range_3917,city_br_4010,83255-000,83259-999
zip_range_3918,city_br_4011,75620-000,75629-999
zip_range_3919,city_br_4012,15718-000,15719-999
zip_range_3920,city_br_4013,99190-000,99199-999
zip_range_3921,city_br_4014,88550-000,88569-999
zip_range_3922,city_br_4015,77315-000,77317-999
zip_range_3923,city_br_4016,89535-000,89539-999
zip_range_3924,city_br_4017,77590-000,77592-999
zip_range_3925,city_br_4018,78610-000,78612-999
zip_range_3926,city_br_4019,35430-001,35435-999
zip_range_3927,city_br_4020,99735-000,99739-999
zip_range_3928,city_br_4021,89683-000,89686-999
zip_range_3929,city_br_4022,78250-000,78252-999
zip_range_3930,city_br_4023,15560-000,15569-999
zip_range_3931,city_br_4024,29885-000,29889-999
zip_range_3932,city_br_4025,39328-000,39329-999
zip_range_3933,city_br_4026,39615-000,39619-999
zip_range_3934,city_br_4027,44755-000,44769-999
zip_range_3935,city_br_4028,15670-000,15679-999
zip_range_3936,city_br_4029,62220-000,62229-999
zip_range_3937,city_br_4030,18260-000,18264-999
zip_range_3938,city_br_4031,76550-000,76554-999
zip_range_3939,city_br_4032,28390-000,28399-999
zip_range_3940,city_br_4033,86160-000,86164-999
zip_range_3941,city_br_4034,59810-000,59814-999
zip_range_3942,city_br_4035,93180-000,93199-999
zip_range_3943,city_br_4036,75603-000,75609-999
zip_range_3944,city_br_4037,63270-000,63274-999
zip_range_3945,city_br_4038,39520-000,39524-999
zip_range_3946,city_br_4039,68480-000,68484-999
zip_range_3947,city_br_4040,75835-000,75839-999
zip_range_3948,city_br_4041,64145-000,64147-999
zip_range_3949,city_br_4042,69927-000,69927-999
zip_range_3950,city_br_011,90000-001,91999-999
zip_range_3951,city_br_4043,78655-000,78657-999
zip_range_3952,city_br_4044,64858-000,64859-999
zip_range_3953,city_br_4045,77395-000,77399-999
zip_range_3954,city_br_4046,84140-000,84144-999
zip_range_3955,city_br_4047,85345-000,85349-999
zip_range_3956,city_br_4048,88210-000,88214-999
zip_range_3957,city_br_4049,57900-000,57909-999
zip_range_3958,city_br_4050,49800-000,49809-999
zip_range_3959,city_br_4051,68330-000,68359-999
zip_range_3960,city_br_4052,57945-000,57949-999
zip_range_3961,city_br_4053,59668-000,59669-999
zip_range_3962,city_br_4054,78560-000,78562-999
zip_range_3963,city_br_4055,78240-000,78242-999
zip_range_3964,city_br_4056,78398-000,78399-999
zip_range_3965,city_br_4057,18540-000,18549-999
zip_range_3966,city_br_4058,13660-000,13669-999
zip_range_3967,city_br_4059,36576-000,36579-999
zip_range_3968,city_br_4060,65970-000,65972-999
zip_range_3969,city_br_4061,68997-000,68999-999
zip_range_3970,city_br_4062,98980-000,98984-999
zip_range_3971,city_br_4063,98947-000,98949-999
zip_range_3972,city_br_4064,79280-000,79289-999
zip_range_3973,city_br_4065,77500-000,77552-999
zip_range_3974,city_br_4066,27570-000,27579-999
zip_range_3975,city_br_4067,57290-000,57299-999
zip_range_3976,city_br_4068,87950-000,87954-999
zip_range_3977,city_br_4069,65263-000,65264-999
zip_range_3978,city_br_174,45810-000,45819-999
zip_range_3979,city_br_4070,89400-000,89419-999
zip_range_3980,city_br_049,76800-001,76849-999
zip_range_3981,city_br_4071,98985-000,98994-999
zip_range_3982,city_br_4072,84615-000,84619-999
zip_range_3983,city_br_4073,69982-000,69982-999
zip_range_3984,city_br_4074,98995-000,98999-999
zip_range_3985,city_br_4075,73900-000,73909-999
zip_range_3986,city_br_4076,39827-000,39829-999
zip_range_3987,city_br_4077,63160-000,63164-999
zip_range_3988,city_br_4078,12525-000,12529-999
zip_range_3989,city_br_4079,45790-000,45799-999
zip_range_3990,city_br_4080,15105-000,15107-999
zip_range_3991,city_br_4081,62990-000,62999-999
zip_range_3992,city_br_197,37550-001,37562-999
zip_range_3993,city_br_4082,37468-000,37469-999
zip_range_3994,city_br_4083,95945-000,95947-999
zip_range_3995,city_br_4084,89172-000,89175-999
zip_range_3996,city_br_4085,78800-000,78809-999
zip_range_3997,city_br_4086,17790-000,17799-999
zip_range_3998,city_br_4087,68918-000,68919-999
zip_range_3999,city_br_4088,45980-000,45984-999
zip_range_4000,city_br_4089,86618-000,86619-999
zip_range_4001,city_br_4090,14850-000,14859-999
zip_range_4002,city_br_4091,36320-000,36324-999
zip_range_4003,city_br_4092,88990-000,88999-999
zip_range_4004,city_br_077,11700-001,11729-999
zip_range_4005,city_br_4093,77970-000,77979-999
zip_range_4006,city_br_4094,68130-000,68137-999
zip_range_4007,city_br_4095,85730-000,85739-999
zip_range_4008,city_br_4096,38140-000,38149-999
zip_range_4009,city_br_4097,58550-000,58559-999
zip_range_4010,city_br_4098,64370-000,64374-999
zip_range_4011,city_br_4099,18660-000,18669-999
zip_range_4012,city_br_4100,37970-000,37972-999
zip_range_4013,city_br_4101,38960-000,38969-999
zip_range_4014,city_br_4102,16670-000,16679-999
zip_range_4015,city_br_4103,36475-000,36479-999
zip_range_4016,city_br_4104,19300-000,19349-999
zip_range_4017,city_br_4105,89745-000,89749-999
zip_range_4018,city_br_4106,87180-000,87184-999
zip_range_4019,city_br_4107,44930-000,44939-999
zip_range_4020,city_br_4108,65760-000,65761-999
zip_range_4021,city_br_4109,19470-000,19499-999
zip_range_4022,city_br_4110,69735-000,69739-999
zip_range_4023,city_br_4111,89150-000,89154-999
zip_range_4024,city_br_4112,46250-000,46254-999
zip_range_4025,city_br_4113,65140-000,65142-999
zip_range_4026,city_br_4114,39245-000,39247-999
zip_range_4027,city_br_4115,29350-000,29359-999
zip_range_4028,city_br_4116,77745-000,77749-999
zip_range_4029,city_br_4117,39135-000,39139-999
zip_range_4030,city_br_4118,93945-000,93949-999
zip_range_4031,city_br_4119,65279-000,65279-999
zip_range_4032,city_br_4120,76916-000,76918-999
zip_range_4033,city_br_4121,89184-000,89185-999
zip_range_4034,city_br_4122,38750-000,38754-999
zip_range_4035,city_br_138,19000-001,19159-999
zip_range_4036,city_br_4123,65204-000,65205-999
zip_range_4037,city_br_4124,45416-000,45419-999
zip_range_4038,city_br_4125,65455-000,65459-999
zip_range_4039,city_br_4126,19400-000,19409-999
zip_range_4040,city_br_4127,68707-000,68708-999
zip_range_4041,city_br_4128,55510-000,55514-999
zip_range_4042,city_br_4129,76976-000,76976-999
zip_range_4043,city_br_4130,78850-000,78854-999
zip_range_4044,city_br_4131,65190-000,65194-999
zip_range_4045,city_br_4132,86140-000,86149-999
zip_range_4046,city_br_4133,89935-000,89939-999
zip_range_4047,city_br_4134,58755-000,58757-999
zip_range_4048,city_br_4135,75645-000,75649-999
zip_range_4049,city_br_4136,95925-000,95929-999
zip_range_4050,city_br_4137,16370-000,16399-999
zip_range_4051,city_br_4138,49900-000,49909-999
zip_range_4052,city_br_4139,95345-000,95349-999
zip_range_4053,city_br_4140,35738-000,35739-999
zip_range_4054,city_br_4141,84400-000,84429-999
zip_range_4055,city_br_4142,77603-000,77604-999
zip_range_4056,city_br_4143,59582-000,59583-999
zip_range_4057,city_br_4144,95975-000,95979-999
zip_range_4058,city_br_4145,58115-000,58116-999
zip_range_4059,city_br_4146,18255-000,18259-999
zip_range_4060,city_br_4147,97560-000,97569-999
zip_range_4061,city_br_4148,35625-000,35627-999
zip_range_4062,city_br_4149,87365-000,87369-999
zip_range_4063,city_br_4150,19780-000,19799-999
zip_range_4064,city_br_4151,86450-000,86454-999
zip_range_4065,city_br_4152,68709-000,68709-999
zip_range_4066,city_br_4153,27400-001,27459-999
zip_range_4067,city_br_4154,83420-000,83429-999
zip_range_4068,city_br_4155,99720-000,99724-999
zip_range_4069,city_br_4156,85940-000,85944-999
zip_range_4070,city_br_4157,57750-000,57759-999
zip_range_4071,city_br_4158,85460-000,85464-999
zip_range_4072,city_br_4159,64758-000,64759-999
zip_range_4073,city_br_4160,48860-000,48869-999
zip_range_4074,city_br_4161,58470-000,58479-999
zip_range_4075,city_br_213,26300-001,26399-999
zip_range_4076,city_br_4162,17590-000,17599-999
zip_range_4077,city_br_4163,12800-000,12819-999
zip_range_4078,city_br_4164,36424-000,36425-999
zip_range_4079,city_br_4165,78643-000,78644-999
zip_range_4080,city_br_4166,87930-000,87949-999
zip_range_4081,city_br_4167,98140-000,98149-999
zip_range_4082,city_br_4168,48830-000,48839-999
zip_range_4083,city_br_4169,89850-000,89853-999
zip_range_4084,city_br_4170,87265-000,87269-999
zip_range_4085,city_br_4171,17670-000,17679-999
zip_range_4086,city_br_4172,98230-000,98239-999
zip_range_4087,city_br_4173,55415-000,55419-999
zip_range_4088,city_br_4174,75860-000,75864-999
zip_range_4089,city_br_4175,28735-000,28739-999
zip_range_4090,city_br_4176,83840-000,83849-999
zip_range_4091,city_br_4177,63650-000,63659-999
zip_range_4092,city_br_4178,58733-000,58733-999
zip_range_4093,city_br_4179,56828-000,56829-999
zip_range_4094,city_br_4180,44713-000,44714-999
zip_range_4095,city_br_4181,63900-001,63949-999
zip_range_4096,city_br_4182,63515-000,63519-999
zip_range_4097,city_br_4183,63800-000,63859-999
zip_range_4098,city_br_4184,62920-000,62929-999
zip_range_4099,city_br_4185,59990-000,59994-999
zip_range_4100,city_br_4186,59740-000,59759-999
zip_range_4101,city_br_4187,44520-000,44529-999
zip_range_4102,city_br_4188,13370-000,13374-999
zip_range_4103,city_br_4189,85888-000,85889-999
zip_range_4104,city_br_4190,19600-000,19639-999
zip_range_4105,city_br_4191,86290-000,86299-999
zip_range_4106,city_br_4192,87395-000,87399-999
zip_range_4107,city_br_4193,88470-000,88474-999
zip_range_4108,city_br_4194,65138-000,65139-999
zip_range_4109,city_br_4195,34400-000,34499-999
zip_range_4110,city_br_4196,35350-000,35358-999
zip_range_4111,city_br_4197,85770-000,85779-999
zip_range_4112,city_br_4198,84550-000,84559-999
zip_range_4113,city_br_009,50000-001,52999-999
zip_range_4114,city_br_4199,36740-000,36749-999
zip_range_4115,city_br_4200,77733-000,77734-999
zip_range_4116,city_br_4201,62790-000,62794-999
zip_range_4117,city_br_4202,68549-001,68554-999
zip_range_4118,city_br_4203,12170-000,12179-999
zip_range_4119,city_br_4204,64915-000,64919-999
zip_range_4120,city_br_4205,98550-000,98559-999
zip_range_4121,city_br_4206,36920-000,36922-999
zip_range_4122,city_br_4207,64490-000,64494-999
zip_range_4123,city_br_4208,19570-000,19579-999
zip_range_4124,city_br_4209,17190-000,17199-999
zip_range_4125,city_br_4210,11900-000,11909-999
zip_range_4126,city_br_4211,95965-000,95966-999
zip_range_4127,city_br_4212,47200-000,47219-999
zip_range_4128,city_br_4213,58398-000,58398-999
zip_range_4129,city_br_4214,85610-000,85614-999
zip_range_4130,city_br_4215,62260-000,62264-999
zip_range_4131,city_br_233,27500-001,27569-999
zip_range_4132,city_br_4216,36340-000,36344-999
zip_range_4133,city_br_4217,84320-000,84344-999
zip_range_4134,city_br_4218,78265-000,78269-999
zip_range_4135,city_br_4219,85195-000,85199-999
zip_range_4136,city_br_4220,35230-000,35239-999
zip_range_4137,city_br_4221,36270-000,36271-999
zip_range_4138,city_br_4222,14430-000,14439-999
zip_range_4139,city_br_4223,97200-000,97209-999
zip_range_4140,city_br_4224,48750-000,48759-999
zip_range_4141,city_br_4225,65990-000,65994-999
zip_range_4142,city_br_4226,58235-000,58237-999
zip_range_4143,city_br_4227,47970-000,47989-999
zip_range_4144,city_br_4228,58382-000,58384-999
zip_range_4145,city_br_4229,49320-000,49349-999
zip_range_4146,city_br_4230,44640-000,44641-999
zip_range_4147,city_br_4231,58348-000,58349-999
zip_range_4148,city_br_4232,38640-000,38649-999
zip_range_4149,city_br_4233,77893-000,77894-999
zip_range_4150,city_br_4234,59820-000,59829-999
zip_range_4151,city_br_4235,55120-000,55124-999
zip_range_4152,city_br_4236,46470-000,46479-999
zip_range_4153,city_br_4237,59987-000,59989-999
zip_range_4154,city_br_4238,58465-000,58469-999
zip_range_4155,city_br_4239,58870-000,58879-999
zip_range_4156,city_br_4240,39529-000,39529-999
zip_range_4157,city_br_4241,64975-000,64979-999
zip_range_4158,city_br_4242,59470-000,59479-999
zip_range_4159,city_br_4243,49130-000,49139-999
zip_range_4160,city_br_4244,76310-000,76314-999
zip_range_4161,city_br_4245,76315-000,76319-999
zip_range_4162,city_br_4246,65938-000,65938-999
zip_range_4163,city_br_4247,79180-000,79189-999
zip_range_4164,city_br_4248,18380-000,18384-999
zip_range_4165,city_br_4249,48440-000,48444-999
zip_range_4166,city_br_4250,64725-000,64727-999
zip_range_4167,city_br_4251,48400-000,48404-999
zip_range_4168,city_br_4252,55520-000,55524-999
zip_range_4169,city_br_4253,13580-000,13589-999
zip_range_4170,city_br_4254,18430-000,18434-999
zip_range_4171,city_br_4255,78675-000,78677-999
zip_range_4172,city_br_4256,86410-000,86419-999
zip_range_4173,city_br_4257,14445-000,14449-999
zip_range_4174,city_br_084,33800-001,33979-999
zip_range_4175,city_br_4258,45155-000,45156-999
zip_range_4176,city_br_4259,86490-000,86599-999
zip_range_4177,city_br_4260,19930-000,19939-999
zip_range_4178,city_br_4261,19380-000,19399-999
zip_range_4179,city_br_4262,18315-000,18319-999
zip_range_4180,city_br_269,09400-001,09449-999
zip_range_4181,city_br_029,14000-001,14114-999
zip_range_4182,city_br_4263,37264-000,37266-999
zip_range_4183,city_br_4264,78613-000,78614-999
zip_range_4184,city_br_4265,64865-000,64867-999
zip_range_4185,city_br_4266,49530-000,49534-999
zip_range_4186,city_br_4267,14490-000,14499-999
zip_range_4187,city_br_4268,14830-000,14834-999
zip_range_4188,city_br_4269,17740-000,17759-999
zip_range_4189,city_br_4270,34300-000,34399-999
zip_range_4190,city_br_4271,84560-000,84569-999
zip_range_4191,city_br_4272,29920-000,29926-999
zip_range_4192,city_br_4273,86830-000,86839-999
zip_range_4193,city_br_4274,28800-000,28819-999
zip_range_4194,city_br_4275,85340-000,85344-999
zip_range_4195,city_br_070,69900-001,69924-999
zip_range_4196,city_br_4276,78275-000,78277-999
zip_range_4197,city_br_4277,86848-000,86849-999
zip_range_4198,city_br_4278,83540-000,83559-999
zip_range_4199,city_br_4279,79130-000,79139-999
zip_range_4200,city_br_4280,35370-000,35379-999
zip_range_4201,city_br_4281,27460-000,27499-999
zip_range_4202,city_br_151,13500-001,13509-999
zip_range_4203,city_br_4282,76863-000,76863-999
zip_range_4204,city_br_4283,77303-000,77304-999
zip_range_4205,city_br_4284,89550-000,89557-999
zip_range_4206,city_br_4285,27660-000,27699-999
zip_range_4207,city_br_193,28890-001,28899-999
zip_range_4208,city_br_4286,13390-000,13399-999
zip_range_4209,city_br_4287,46170-000,46179-999
zip_range_4210,city_br_002,20000-001,23799-999
zip_range_4211,city_br_4288,46220-000,46249-999
zip_range_4212,city_br_4289,89198-000,89198-999
zip_range_4213,city_br_4290,59578-000,59579-999
zip_range_4214,city_br_4291,89180-000,89181-999
zip_range_4215,city_br_4292,46550-000,46569-999
zip_range_4216,city_br_4293,39940-000,39944-999
zip_range_4217,city_br_4294,89160-001,89169-999
zip_range_4218,city_br_4295,35442-000,35443-999
zip_range_4219,city_br_4296,77655-000,77659-999
zip_range_4220,city_br_4297,89121-000,89123-999
zip_range_4221,city_br_4298,99610-000,99614-999
zip_range_4222,city_br_4299,36460-000,36469-999
zip_range_4223,city_br_4300,55570-000,55577-999
zip_range_4224,city_br_4301,88760-000,88762-999
zip_range_4225,city_br_158,96200-001,96224-999
zip_range_4226,city_br_4302,09450-000,09499-999
zip_range_4227,city_br_4303,64835-000,64837-999
zip_range_4228,city_br_4304,57100-000,57119-999
zip_range_4229,city_br_4305,35485-000,35487-999
zip_range_4230,city_br_4306,68530-000,68532-999
zip_range_4231,city_br_4307,89295-000,89299-999
zip_range_4232,city_br_4308,79470-000,79479-999
zip_range_4233,city_br_4309,83880-000,83899-999
zip_range_4234,city_br_4310,36150-000,36151-999
zip_range_4235,city_br_4311,29290-000,29294-999
zip_range_4236,city_br_4312,38810-000,38819-999
zip_range_4237,city_br_4313,96640-000,96684-999
zip_range_4238,city_br_4314,39530-000,39534-999
zip_range_4239,city_br_4315,35940-000,35949-999
zip_range_4240,city_br_4316,36180-000,36184-999
zip_range_4241,city_br_4317,36130-000,36131-999
zip_range_4242,city_br_4318,69117-000,69119-999
zip_range_4243,city_br_4319,75695-000,75699-999
zip_range_4244,city_br_4320,48330-000,48349-999
zip_range_4245,city_br_4321,88658-000,88679-999
zip_range_4246,city_br_4322,77635-000,77639-999
zip_range_4247,city_br_4323,58297-000,58299-999
zip_range_4248,city_br_136,75900-001,75914-999
zip_range_4249,city_br_4324,79480-000,79489-999
zip_range_4250,city_br_4325,39170-000,39179-999
zip_range_4251,city_br_4326,15495-000,15499-999
zip_range_4252,city_br_4327,95695-000,95699-999
zip_range_4253,city_br_4328,89895-000,89895-999
zip_range_4254,city_br_4329,36335-000,36339-999
zip_range_4255,city_br_4330,18470-000,18474-999
zip_range_4256,city_br_4331,95735-000,95739-999
zip_range_4257,city_br_4332,79450-000,79459-999
zip_range_4258,city_br_4333,36604-000,36605-999
zip_range_4259,city_br_4334,89136-000,89137-999
zip_range_4260,city_br_4335,98360-000,98367-999
zip_range_4261,city_br_4336,36510-000,36511-999
zip_range_4262,city_br_4337,48630-000,48649-999
zip_range_4263,city_br_4338,59830-000,59839-999
zip_range_4264,city_br_4339,69985-000,69989-999
zip_range_4265,city_br_4340,97843-000,97844-999
zip_range_4266,city_br_4341,86600-001,86609-999
zip_range_4267,city_br_4342,95690-000,95694-999
zip_range_4268,city_br_4343,76940-000,76947-999
zip_range_4269,city_br_4344,38520-000,38524-999
zip_range_4270,city_br_4345,89908-000,89908-999
zip_range_4271,city_br_4346,87320-000,87324-999
zip_range_4272,city_br_4347,99670-000,99674-999
zip_range_4273,city_br_4348,99590-000,99599-999
zip_range_4274,city_br_4349,78338-000,78339-999
zip_range_4275,city_br_4350,87800-000,87809-999
zip_range_4276,city_br_4351,68638-000,68638-999
zip_range_4277,city_br_118,78700-001,78759-999
zip_range_4278,city_br_4352,97970-000,97979-999
zip_range_4279,city_br_4353,69373-000,69374-999
zip_range_4280,city_br_4354,19273-000,19274-999
zip_range_4281,city_br_4355,65150-000,65152-999
zip_range_4282,city_br_4356,36878-000,36879-999
zip_range_4283,city_br_4357,49760-000,49769-999
zip_range_4284,city_br_4358,86850-000,86854-999
zip_range_4285,city_br_4359,97590-000,97609-999
zip_range_4286,city_br_4360,78470-000,78479-999
zip_range_4287,city_br_4361,12580-000,12599-999
zip_range_4288,city_br_4362,57257-000,57259-999
zip_range_4289,city_br_4363,39565-000,39567-999
zip_range_4290,city_br_4364,16750-000,16789-999
zip_range_4291,city_br_4365,76350-000,76354-999
zip_range_4292,city_br_4366,39950-000,39959-999
zip_range_4293,city_br_4367,15790-000,15799-999
zip_range_4294,city_br_4368,68165-000,68169-999
zip_range_4295,city_br_4369,62900-000,62909-999
zip_range_4296,city_br_4370,46800-000,46804-999
zip_range_4297,city_br_4371,59420-000,59429-999
zip_range_4298,city_br_234,34500-001,34799-999
zip_range_4299,city_br_4372,86720-000,86729-999
zip_range_4300,city_br_4373,16440-000,16449-999
zip_range_4301,city_br_4374,39750-000,39754-999
zip_range_4302,city_br_4375,63590-000,63594-999
zip_range_4303,city_br_4376,38190-000,38194-999
zip_range_4304,city_br_4377,98330-000,98334-999
zip_range_4305,city_br_4378,17710-000,17719-999
zip_range_4306,city_br_4379,55695-000,55699-999
zip_range_4307,city_br_4380,98250-000,98269-999
zip_range_4308,city_br_4381,14980-000,14989-999
zip_range_4309,city_br_4382,14660-000,14669-999
zip_range_4310,city_br_4383,08970-000,08999-999
zip_range_4311,city_br_4384,89196-000,89197-999
zip_range_4312,city_br_4385,58650-000,58659-999
zip_range_4313,city_br_4386,55675-000,55679-999
zip_range_4314,city_br_4387,49390-000,49399-999
zip_range_4315,city_br_4388,58370-000,58373-999
zip_range_4316,city_br_4389,85620-000,85627-999
zip_range_4317,city_br_4390,56000-000,56119-999
zip_range_4318,city_br_4391,39560-000,39562-999
zip_range_4319,city_br_4392,44450-000,44459-999
zip_range_4320,city_br_4393,68721-000,68721-999
zip_range_4321,city_br_4394,63155-000,63159-999
zip_range_4322,city_br_4395,17720-000,17729-999
zip_range_4323,city_br_4396,55350-000,55354-999
zip_range_4324,city_br_4397,89981-000,89981-999
zip_range_4325,city_br_4398,13440-000,13449-999
zip_range_4326,city_br_220,13320-001,13329-999
zip_range_4327,city_br_4399,39925-000,39927-999
zip_range_4328,city_br_4400,18160-000,18169-999
zip_range_4329,city_br_4401,78270-000,78274-999
zip_range_4330,city_br_4402,84945-000,84949-999
zip_range_4331,city_br_4403,99440-000,99449-999
zip_range_4332,city_br_4404,85670-000,85679-999
zip_range_4333,city_br_4405,19920-000,19929-999
zip_range_4334,city_br_4406,89595-000,89599-999
zip_range_4335,city_br_005,40000-001,42599-999
zip_range_4336,city_br_4407,97940-000,97949-999
zip_range_4337,city_br_4408,95750-000,95754-999
zip_range_4338,city_br_4409,68860-000,68869-999
zip_range_4339,city_br_4410,65830-000,65839-999
zip_range_4340,city_br_4411,77980-000,77984-999
zip_range_4341,city_br_4412,99840-000,99849-999
zip_range_4342,city_br_4413,76160-000,76164-999
zip_range_4343,city_br_4414,77478-000,77479-999
zip_range_4344,city_br_4415,19250-000,19259-999
zip_range_4345,city_br_4416,88717-000,88719-999
zip_range_4346,city_br_4417,55250-000,55259-999
zip_range_4347,city_br_4418,97570-001,97589-999
zip_range_4348,city_br_4419,15950-000,15959-999
zip_range_4349,city_br_4420,15750-000,15754-999
zip_range_4350,city_br_4421,86370-000,86374-999
zip_range_4351,city_br_4422,44150-000,44159-999
zip_range_4352,city_br_4423,35960-000,35968-999
zip_range_4353,city_br_163,13450-001,13464-999
zip_range_4354,city_br_4424,75398-000,75399-999
zip_range_4355,city_br_4425,35328-000,35329-999
zip_range_4356,city_br_4426,36132-000,36134-999
zip_range_4357,city_br_4427,68798-000,68799-999
zip_range_4358,city_br_4428,98240-000,98249-999
zip_range_4359,city_br_4429,36215-000,36219-999
zip_range_4360,city_br_4430,12380-000,12399-999
zip_range_4361,city_br_4431,48570-000,48579-999
zip_range_4362,city_br_4432,78545-000,78547-999
zip_range_4363,city_br_4433,58463-000,58464-999
zip_range_4364,city_br_4434,89540-000,89544-999
zip_range_4365,city_br_4435,86225-000,86229-999
zip_range_4366,city_br_4436,99952-000,99954-999
zip_range_4367,city_br_4437,15785-000,15789-999
zip_range_4368,city_br_4438,95915-000,95917-999
zip_range_4369,city_br_4439,58824-000,58829-999
zip_range_4370,city_br_4440,56215-000,56219-999
zip_range_4371,city_br_4441,59200-000,59209-999
zip_range_4372,city_br_4442,45807-000,45809-999
zip_range_4373,city_br_4443,56895-000,56899-999
zip_range_4374,city_br_4444,13625-000,13629-999
zip_range_4375,city_br_4445,14250-000,14259-999
zip_range_4376,city_br_4446,45725-000,45729-999
zip_range_4377,city_br_4447,13650-000,13659-999
zip_range_4378,city_br_4448,75220-000,75229-999
zip_range_4379,city_br_4449,36328-000,36329-999
zip_range_4380,city_br_4450,87920-000,87929-999
zip_range_4381,city_br_4451,39563-000,39564-999
zip_range_4382,city_br_4452,68850-000,68859-999
zip_range_4383,city_br_4453,55190-001,55199-999
zip_range_4384,city_br_4454,35383-000,35387-999
zip_range_4385,city_br_4455,64545-000,64547-999
zip_range_4386,city_br_4456,18900-000,18934-999
zip_range_4387,city_br_224,96800-001,96874-999
zip_range_4388,city_br_4457,78664-000,78664-999
zip_range_4389,city_br_4458,64315-000,64319-999
zip_range_4390,city_br_4459,39725-000,39727-999
zip_range_4391,city_br_4460,15970-000,15979-999
zip_range_4392,city_br_4461,86770-000,86779-999
zip_range_4393,city_br_4462,76265-000,76269-999
zip_range_4394,city_br_4463,39295-000,39299-999
zip_range_4395,city_br_4464,77848-000,77849-999
zip_range_4396,city_br_4465,15775-000,15779-999
zip_range_4397,city_br_4466,56210-000,56214-999
zip_range_4398,city_br_4467,64945-000,64959-999
zip_range_4399,city_br_4468,65768-000,65769-999
zip_range_4400,city_br_4469,13510-000,13514-999
zip_range_4401,city_br_4470,65208-000,65209-999
zip_range_4402,city_br_4471,58925-000,58927-999
zip_range_4403,city_br_4472,85892-000,85895-999
zip_range_4404,city_br_4473,89915-000,89919-999
zip_range_4405,city_br_4474,75920-000,75924-999
zip_range_4406,city_br_4475,39874-000,39874-999
zip_range_4407,city_br_4476,45320-000,45324-999
zip_range_4408,city_br_4477,65300-001,65309-999
zip_range_4409,city_br_4478,58978-000,58979-999
zip_range_4410,city_br_4479,86660-000,86669-999
zip_range_4411,city_br_4480,76320-000,76329-999
zip_range_4412,city_br_4481,07500-000,07599-999
zip_range_4413,city_br_4482,87910-000,87914-999
zip_range_4414,city_br_4483,69740-000,69749-999
zip_range_4415,city_br_4484,85650-000,85659-999
zip_range_4416,city_br_4485,68790-000,68794-999
zip_range_4417,city_br_4486,38175-000,38177-999
zip_range_4418,city_br_4487,29640-000,29644-999
zip_range_4419,city_br_4488,85795-000,85799-999
zip_range_4420,city_br_4489,14825-000,14829-999
zip_range_4421,city_br_4490,64910-000,64914-999
zip_range_4422,city_br_4491,45865-000,45869-999
zip_range_4423,city_br_4492,65390-000,65392-999
zip_range_4424,city_br_143,33000-001,33199-999
zip_range_4425,city_br_4493,58600-000,58609-999
zip_range_4426,city_br_4494,76950-000,76951-999
zip_range_4427,city_br_4495,49230-000,49249-999
zip_range_4428,city_br_4496,57130-000,57139-999
zip_range_4429,city_br_4497,68644-000,68644-999
zip_range_4430,city_br_4498,65272-000,65273-999
zip_range_4431,city_br_4499,36913-000,36917-999
zip_range_4432,city_br_4500,97335-000,97339-999
zip_range_4433,city_br_4501,59464-000,59469-999
zip_range_4434,city_br_103,97000-001,97179-999
zip_range_4435,city_br_4502,56380-000,56394-999
zip_range_4436,city_br_4503,17370-000,17379-999
zip_range_4437,city_br_4504,47640-000,47649-999
zip_range_4438,city_br_4505,68565-000,68569-999
zip_range_4439,city_br_4506,35910-000,35919-999
zip_range_4440,city_br_4507,29645-000,29649-999
zip_range_4441,city_br_4508,55765-000,55769-999
zip_range_4442,city_br_4509,93995-000,93999-999
zip_range_4443,city_br_4510,85230-000,85239-999
zip_range_4444,city_br_4511,68738-000,68739-999
zip_range_4445,city_br_4512,39928-000,39929-999
zip_range_4446,city_br_4513,39780-000,39783-999
zip_range_4447,city_br_4514,77716-000,77717-999
zip_range_4448,city_br_4515,28770-000,28799-999
zip_range_4449,city_br_4516,86350-000,86359-999
zip_range_4450,city_br_4517,17940-000,17949-999
zip_range_4451,city_br_4518,87915-000,87919-999
zip_range_4452,city_br_4519,62280-000,62296-999
zip_range_4453,city_br_4520,65540-000,65544-999
zip_range_4454,city_br_4521,65145-000,65147-999
zip_range_4455,city_br_201,58300-001,58304-999
zip_range_4456,city_br_4522,15780-000,15784-999
zip_range_4457,city_br_4523,37775-000,37779-999
zip_range_4458,city_br_4524,47150-000,47159-999
zip_range_4459,city_br_4525,36235-000,36239-999
zip_range_4460,city_br_4526,36135-000,36139-999
zip_range_4461,city_br_4527,35326-000,35327-999
zip_range_4462,city_br_4528,75840-000,75844-999
zip_range_4463,city_br_4529,35225-000,35229-999
zip_range_4464,city_br_4530,76395-000,76399-999
zip_range_4465,city_br_4531,79690-000,79699-999
zip_range_4466,city_br_4532,13670-000,13689-999
zip_range_4467,city_br_4533,37540-000,37541-999
zip_range_4468,city_br_4534,77565-000,77569-999
zip_range_4469,city_br_4535,78453-000,78454-999
zip_range_4470,city_br_4536,98780-001,98799-999
zip_range_4471,city_br_4537,38805-000,38809-999
zip_range_4472,city_br_4538,75455-000,75459-999
zip_range_4473,city_br_4539,88763-000,88764-999
zip_range_4474,city_br_4540,49640-000,49649-999
zip_range_4475,city_br_4541,14270-000,14299-999
zip_range_4476,city_br_4542,64518-000,64519-999
zip_range_4477,city_br_4543,69955-000,69959-999
zip_range_4478,city_br_4544,88965-000,88969-999
zip_range_4479,city_br_4545,77375-000,77377-999
zip_range_4480,city_br_4546,15768-000,15769-999
zip_range_4481,city_br_4547,29650-000,29664-999
zip_range_4482,city_br_4548,44590-000,44599-999
zip_range_4483,city_br_4549,58720-000,58722-999
zip_range_4484,city_br_4550,95715-000,95716-999
zip_range_4485,city_br_4551,76480-000,76484-999
zip_range_4486,city_br_4552,85825-000,85825-999
zip_range_4487,city_br_4553,77615-000,77619-999
zip_range_4488,city_br_4554,78650-000,78651-999
zip_range_4489,city_br_4555,56750-000,56759-999
zip_range_4490,city_br_4556,89199-000,89199-999
zip_range_4491,city_br_4557,76500-000,76509-999
zip_range_4492,city_br_4558,85875-000,85876-999
zip_range_4493,city_br_4559,89983-000,89984-999
zip_range_4494,city_br_4560,77885-000,77887-999
zip_range_4495,city_br_4561,38320-000,38349-999
zip_range_4496,city_br_4562,96230-000,96254-999
zip_range_4497,city_br_4563,48880-000,48889-999
zip_range_4498,city_br_289,68925-001,68939-999
zip_range_4499,city_br_4564,47700-000,47729-999
zip_range_4500,city_br_4565,96590-000,96599-999
zip_range_4501,city_br_4566,15765-000,15767-999
zip_range_4502,city_br_4567,37195-000,37199-999
zip_range_4503,city_br_4568,36795-000,36799-999
zip_range_4504,city_br_4569,58985-000,58989-999
zip_range_4505,city_br_195,06500-001,06549-999
zip_range_4506,city_br_4570,35785-000,35788-999
zip_range_4507,city_br_4571,62150-000,62159-999
zip_range_4508,city_br_4572,68560-000,68564-999
zip_range_4509,city_br_4573,63190-000,63194-999
zip_range_4510,city_br_4574,36620-000,36629-999
zip_range_4511,city_br_4575,36146-000,36147-999
zip_range_4512,city_br_4576,57500-000,57509-999
zip_range_4513,city_br_4577,84970-000,84979-999
zip_range_4514,city_br_4578,37278-000,37279-999
zip_range_4515,city_br_4579,36940-000,36946-999
zip_range_4516,city_br_4580,65555-000,65559-999
zip_range_4517,city_br_4581,59520-000,59527-999
zip_range_4518,city_br_4582,57840-000,57859-999
zip_range_4519,city_br_4583,35179-000,35179-999
zip_range_4520,city_br_4584,64615-000,64617-999
zip_range_4521,city_br_4585,35845-000,35849-999
zip_range_4522,city_br_4586,49985-000,49989-999
zip_range_4523,city_br_4587,59350-000,59354-999
zip_range_4524,city_br_4588,58795-000,58797-999
zip_range_4525,city_br_4589,36430-000,36439-999
zip_range_4526,city_br_4590,44260-000,44269-999
zip_range_4527,city_br_082,68000-001,68128-999
zip_range_4528,city_br_4591,68720-000,68720-999
zip_range_4529,city_br_4592,97700-000,97752-999
zip_range_4530,city_br_4593,89854-000,89854-999
zip_range_4531,city_br_4594,78425-000,78429-999
zip_range_4532,city_br_4595,44200-000,44219-999
zip_range_4533,city_br_4596,88140-000,88149-999
zip_range_4534,city_br_4597,49180-000,49189-999
zip_range_4535,city_br_4598,65195-000,65199-999
zip_range_4536,city_br_4599,19360-000,19379-999
zip_range_4537,city_br_4600,58675-000,58679-999
zip_range_4538,city_br_025,09000-001,09299-999
zip_range_4539,city_br_4601,98800-001,98849-999
zip_range_4540,city_br_4602,59255-000,59257-999
zip_range_4541,city_br_4603,14390-000,14399-999
zip_range_4542,city_br_4604,75935-000,75939-999
zip_range_4543,city_br_4605,95500-000,95514-999
zip_range_4544,city_br_4606,86430-000,86449-999
zip_range_4545,city_br_4607,97870-000,97879-999
zip_range_4546,city_br_4608,75375-000,75379-999
zip_range_4547,city_br_313,44570-001,44574-999
zip_range_4548,city_br_4609,64640-000,64644-999
zip_range_4549,city_br_4610,28470-000,28494-999
zip_range_4550,city_br_4611,13830-000,13834-999
zip_range_4551,city_br_4612,37262-000,37263-999
zip_range_4552,city_br_4613,16130-000,16199-999
zip_range_4553,city_br_4614,36670-000,36679-999
zip_range_4554,city_br_4615,87730-000,87739-999
zip_range_4555,city_br_4616,72900-001,72909-999
zip_range_4556,city_br_4617,35388-000,35389-999
zip_range_4557,city_br_4618,69680-000,69684-999
zip_range_4558,city_br_4619,39160-000,39164-999
zip_range_4559,city_br_4620,39935-000,39939-999
zip_range_4560,city_br_4621,13995-000,13999-999
zip_range_4561,city_br_4622,78628-000,78629-999
zip_range_4562,city_br_4623,78180-000,78189-999
zip_range_4563,city_br_4624,35560-000,35564-999
zip_range_4564,city_br_4625,99265-000,99269-999
zip_range_4565,city_br_4626,86315-000,86319-999
zip_range_4566,city_br_4627,12450-000,12459-999
zip_range_4567,city_br_4628,99525-000,99527-999
zip_range_4568,city_br_4629,39538-000,39539-999
zip_range_4569,city_br_4630,35880-000,35893-999
zip_range_4570,city_br_4631,85710-000,85726-999
zip_range_4571,city_br_4632,68786-000,68789-999
zip_range_4572,city_br_4633,65730-000,65734-999
zip_range_4573,city_br_4634,64438-000,64439-999
zip_range_4574,city_br_4635,98590-000,98594-999
zip_range_4575,city_br_4636,98960-000,98969-999
zip_range_4576,city_br_4637,44190-000,44199-999
zip_range_4577,city_br_4638,19190-000,19199-999
zip_range_4578,city_br_4639,99895-000,99899-999
zip_range_4579,city_br_4640,39210-000,39214-999
zip_range_4580,city_br_4641,86650-000,86659-999
zip_range_4581,city_br_4642,64560-000,64562-999
zip_range_4582,city_br_4643,16240-000,16249-999
zip_range_4583,city_br_056,11000-001,11249-999
zip_range_4584,city_br_4644,36240-000,36249-999
zip_range_4585,city_br_4645,62370-000,62374-999
zip_range_4586,city_br_4646,65440-000,65449-999
zip_range_4587,city_br_4647,55410-000,55414-999
zip_range_4588,city_br_4648,58857-000,58859-999
zip_range_4589,city_br_4649,65235-000,65237-999
zip_range_4590,city_br_4650,58865-000,58869-999
zip_range_4591,city_br_4651,37407-000,37407-999
zip_range_4592,city_br_4652,59590-000,59591-999
zip_range_4593,city_br_4653,12490-000,12499-999
zip_range_4594,city_br_4654,89280-001,89293-999
zip_range_4595,city_br_4655,77958-000,77959-999
zip_range_4596,city_br_4656,59210-000,59212-999
zip_range_4597,city_br_4657,55370-000,55374-999
zip_range_4598,city_br_4658,89982-000,89982-999
zip_range_4599,city_br_4659,65550-000,65554-999
zip_range_4600,city_br_021,09600-001,09899-999
zip_range_4601,city_br_4660,88485-000,88489-999
zip_range_4602,city_br_4661,97670-000,97684-999
zip_range_4603,city_br_4662,57380-000,57389-999
zip_range_4604,city_br_4663,35495-000,35496-999
zip_range_4605,city_br_4664,64783-000,64784-999
zip_range_4606,city_br_4665,68775-000,68779-999
zip_range_4607,city_br_178,09500-001,09599-999
zip_range_4608,city_br_4666,55130-000,55139-999
zip_range_4609,city_br_4667,89885-000,89885-999
zip_range_4610,city_br_113,13560-001,13579-999
zip_range_4611,city_br_4668,87770-000,87779-999
zip_range_4612,city_br_4669,49100-000,49119-999
zip_range_4613,city_br_4670,89533-000,89534-999
zip_range_4614,city_br_4671,47820-000,47829-999
zip_range_4615,city_br_4672,48895-000,48899-999
zip_range_4616,city_br_4673,73860-000,73864-999
zip_range_4617,city_br_4674,58853-000,58854-999
zip_range_4618,city_br_4675,89835-000,89836-999
zip_range_4619,city_br_4676,49525-000,49529-999
zip_range_4620,city_br_4677,35335-000,35337-999
zip_range_4621,city_br_4678,68520-000,68520-999
zip_range_4622,city_br_4679,65888-000,65889-999
zip_range_4623,city_br_4680,68635-000,68636-999
zip_range_4624,city_br_4681,58485-000,58486-999
zip_range_4625,city_br_4682,65790-000,65794-999
zip_range_4626,city_br_4683,29745-000,29749-999
zip_range_4627,city_br_4684,35993-000,35999-999
zip_range_4628,city_br_4685,99270-000,99289-999
zip_range_4629,city_br_4686,44550-000,44559-999
zip_range_4630,city_br_4687,76977-000,76978-999
zip_range_4631,city_br_4688,44360-000,44379-999
zip_range_4632,city_br_4689,65890-000,65894-999
zip_range_4633,city_br_4690,35275-000,35276-999
zip_range_4634,city_br_4691,78670-000,78673-999
zip_range_4635,city_br_4692,47665-000,47679-999
zip_range_4636,city_br_4693,64375-000,64377-999
zip_range_4637,city_br_4694,77605-000,77609-999
zip_range_4638,city_br_4695,68380-000,68382-999
zip_range_4639,city_br_4696,59327-000,59329-999
zip_range_4640,city_br_4697,28400-000,28429-999
zip_range_4641,city_br_4698,39300-000,39313-999
zip_range_4642,city_br_4699,58818-000,58818-999
zip_range_4643,city_br_4700,49945-000,49949-999
zip_range_4644,city_br_4701,15710-000,15712-999
zip_range_4645,city_br_4702,97610-000,97639-999
zip_range_4646,city_br_4703,64745-000,64747-999
zip_range_4647,city_br_4704,75490-000,75494-999
zip_range_4648,city_br_4705,28230-000,28249-999
zip_range_4649,city_br_4706,35543-000,35543-999
zip_range_4650,city_br_4707,95400-000,95419-999
zip_range_4651,city_br_4708,38260-000,38269-999
zip_range_4652,city_br_4709,65929-000,65929-999
zip_range_4653,city_br_4710,43900-000,43999-999
zip_range_4654,city_br_4711,36810-000,36814-999
zip_range_4655,city_br_4712,76935-000,76936-999
zip_range_4656,city_br_4713,65650-000,65659-999
zip_range_4657,city_br_4714,59908-000,59909-999
zip_range_4658,city_br_4715,68748-000,68749-999
zip_range_4659,city_br_4716,64550-000,64554-999
zip_range_4660,city_br_4717,89240-000,89244-999
zip_range_4661,city_br_4718,44915-000,44919-999
zip_range_4662,city_br_4719,97300-000,97334-999
zip_range_4663,city_br_4720,69750-000,69799-999
zip_range_4664,city_br_4721,29780-000,29784-999
zip_range_4665,city_br_4722,79490-000,79499-999
zip_range_4666,city_br_4723,36530-000,36539-999
zip_range_4667,city_br_4724,39723-000,39724-999
zip_range_4668,city_br_4725,68570-000,68574-999
zip_range_4669,city_br_4726,35258-000,35259-999
zip_range_4670,city_br_018,24400-001,24799-999
zip_range_4671,city_br_4727,38790-000,38793-999
zip_range_4672,city_br_4728,62670-000,62679-999
zip_range_4673,city_br_265,59290-001,59299-999
zip_range_4674,city_br_4729,64993-000,64994-999
zip_range_4675,city_br_4730,35544-000,35544-999
zip_range_4676,city_br_4731,64435-000,64437-999
zip_range_4677,city_br_4732,35935-000,35937-999
zip_range_4678,city_br_4733,39185-000,39187-999
zip_range_4679,city_br_4734,37490-000,37495-999
zip_range_4680,city_br_4735,44330-000,44339-999
zip_range_4681,city_br_4736,38800-000,38804-999
zip_range_4682,city_br_4737,96700-000,96734-999
zip_range_4683,city_br_4738,86270-000,86279-999
zip_range_4684,city_br_4739,55435-000,55439-999
zip_range_4685,city_br_4740,85570-000,85574-999
zip_range_4686,city_br_4741,65225-000,65229-999
zip_range_4687,city_br_4742,88240-000,88259-999
zip_range_4688,city_br_4743,37920-000,37921-999
zip_range_4689,city_br_4744,73760-000,73769-999
zip_range_4690,city_br_4745,69375-000,69377-999
zip_range_4691,city_br_4746,28200-000,28229-999
zip_range_4692,city_br_4747,13870-001,13879-999
zip_range_4693,city_br_4748,64635-000,64637-999
zip_range_4694,city_br_4749,64243-000,64244-999
zip_range_4695,city_br_4750,39355-000,39359-999
zip_range_4696,city_br_4751,37568-000,37569-999
zip_range_4697,city_br_4752,75985-000,75989-999
zip_range_4698,city_br_4753,68774-000,68774-999
zip_range_4699,city_br_4754,39430-000,39436-999
zip_range_4700,city_br_4755,64350-000,64359-999
zip_range_4701,city_br_4756,99855-000,99859-999
zip_range_4702,city_br_4757,64510-000,64511-999
zip_range_4703,city_br_4758,15640-000,15649-999
zip_range_4704,city_br_4759,39475-000,39477-999
zip_range_4705,city_br_4760,15315-000,15319-999
zip_range_4706,city_br_053,25500-001,25599-999
zip_range_4707,city_br_4761,68719-000,68719-999
zip_range_4708,city_br_4762,36300-001,36319-999
zip_range_4709,city_br_4763,68518-000,68519-999
zip_range_4710,city_br_4764,64155-000,64159-999
zip_range_4711,city_br_4765,87740-000,87749-999
zip_range_4712,city_br_4766,58590-000,58594-999
zip_range_4713,city_br_4767,65385-000,65389-999
zip_range_4714,city_br_4768,88395-000,88399-999
zip_range_4715,city_br_4769,86930-000,86934-999
zip_range_4716,city_br_4770,62965-000,62969-999
zip_range_4717,city_br_4771,36918-000,36919-999
zip_range_4718,city_br_4772,35277-000,35279-999
zip_range_4719,city_br_4773,89897-000,89897-999
zip_range_4720,city_br_4774,35146-000,35146-999
zip_range_4721,city_br_4775,39365-000,39369-999
zip_range_4722,city_br_4776,65973-000,65974-999
zip_range_4723,city_br_4777,39540-000,39546-999
zip_range_4724,city_br_4778,17970-000,17979-999
zip_range_4725,city_br_4779,64760-000,64762-999
zip_range_4726,city_br_4780,97230-000,97249-999
zip_range_4727,city_br_4781,58910-000,58914-999
zip_range_4728,city_br_4782,59310-000,59314-999
zip_range_4729,city_br_4783,65615-000,65619-999
zip_range_4730,city_br_4784,88970-000,88979-999
zip_range_4731,city_br_4785,58520-000,58529-999
zip_range_4732,city_br_4786,84150-000,84159-999
zip_range_4733,city_br_4787,65665-000,65667-999
zip_range_4734,city_br_4788,39704-000,39706-999
zip_range_4735,city_br_4789,36680-000,36689-999
zip_range_4736,city_br_4790,88600-000,88624-999
zip_range_4737,city_br_4791,14600-000,14609-999
zip_range_4738,city_br_4792,32920-000,32999-999
zip_range_4739,city_br_4793,55670-000,55674-999
zip_range_4740,city_br_4794,95365-000,95369-999
zip_range_4741,city_br_4795,85575-000,85579-999
zip_range_4742,city_br_4796,87190-000,87199-999
zip_range_4743,city_br_4797,87555-000,87559-999
zip_range_4744,city_br_104,88100-001,88124-999
zip_range_4745,city_br_4798,37945-000,37947-999
zip_range_4746,city_br_4799,14440-000,14444-999
zip_range_4747,city_br_4800,84980-000,84989-999
zip_range_4748,city_br_4801,55565-000,55569-999
zip_range_4749,city_br_4802,58815-000,58816-999
zip_range_4750,city_br_4803,57860-000,57889-999
zip_range_4751,city_br_4804,33350-000,33399-999
zip_range_4752,city_br_4805,39785-000,39789-000
zip_range_4753,city_br_4806,57445-000,57459-999
zip_range_4754,city_br_4807,35694-000,35694-999
zip_range_4755,city_br_4808,45620-000,45621-999
zip_range_4756,city_br_4809,98325-000,98329-999
zip_range_4757,city_br_4810,85898-000,85899-999
zip_range_4758,city_br_4811,58784-000,58789-999
zip_range_4759,city_br_4812,58723-000,58724-999
zip_range_4760,city_br_4813,59162-000,59163-999
zip_range_4761,city_br_4814,58940-000,58944-999
zip_range_4762,city_br_4815,58758-000,58759-999
zip_range_4763,city_br_119,65110-000,65129-999
zip_range_4764,city_br_4816,28455-000,28459-999
zip_range_4765,city_br_4817,37510-000,37510-999
zip_range_4766,city_br_4818,12830-000,12849-999
zip_range_4767,city_br_4819,56950-000,56979-999
zip_range_4768,city_br_4820,58725-000,58729-999
zip_range_4769,city_br_4821,58893-000,58894-999
zip_range_4770,city_br_4822,29470-000,29479-999
zip_range_4771,city_br_4823,59275-000,59279-999
zip_range_4772,city_br_4824,89930-000,89934-999
zip_range_4773,city_br_4825,88570-000,88579-999
zip_range_4774,city_br_4826,39848-000,39849-999
zip_range_4775,city_br_4827,64245-000,64249-999
zip_range_4776,city_br_4828,56700-000,56719-999
zip_range_4777,city_br_4829,35986-000,35992-999
zip_range_4778,city_br_4830,99380-000,99399-999
zip_range_4779,city_br_4831,95755-000,95757-999
zip_range_4780,city_br_4832,98958-000,98959-999
zip_range_4781,city_br_4833,44698-000,44699-999
zip_range_4782,city_br_4834,39707-000,39707-999
zip_range_4783,city_br_4835,36990-000,36999-999
zip_range_4784,city_br_4836,96225-000,96229-999
zip_range_4785,city_br_4837,99870-000,99877-999
zip_range_4786,city_br_4838,64555-000,64557-999
zip_range_4787,city_br_4839,64625-000,64629-999
zip_range_4788,city_br_4840,78773-000,78774-999
zip_range_4789,city_br_4841,78435-000,78444-999
zip_range_4790,city_br_4842,13720-000,13729-999
zip_range_4791,city_br_045,15000-001,15104-999
zip_range_4792,city_br_4843,58610-000,58619-999
zip_range_4793,city_br_4844,59378-000,59379-999
zip_range_4794,city_br_4845,95748-000,95749-999
zip_range_4795,city_br_4846,25780-000,25799-999
zip_range_4796,city_br_4847,78663-000,78663-999
zip_range_4797,city_br_4848,95280-000,95289-999
zip_range_4798,city_br_4849,65762-000,65762-999
zip_range_4799,city_br_030,12200-001,12249-999
zip_range_4800,city_br_4850,58570-000,58574-999
zip_range_4801,city_br_085,83000-001,83189-999
zip_range_4802,city_br_4851,78285-000,78289-999
zip_range_4803,city_br_4852,58339-000,58339-999
zip_range_4804,city_br_4853,64670-000,64674-999
zip_range_4805,city_br_144,93000-001,93179-999
zip_range_4806,city_br_4854,37470-000,37471-999
zip_range_4807,city_br_282,54700-001,54749-999
zip_range_4808,city_br_4855,06890-000,06899-999
zip_range_4809,city_br_4856,89990-000,89997-999
zip_range_4810,city_br_4857,64778-000,64779-999
zip_range_4811,city_br_4858,96170-000,96177-999
zip_range_4812,city_br_4859,88730-000,88734-999
zip_range_4813,city_br_015,65000-001,65109-999
zip_range_4814,city_br_4860,76100-000,76104-999
zip_range_4815,city_br_4861,62665-000,62669-999
zip_range_4816,city_br_4862,64638-000,64639-999
zip_range_4817,city_br_4863,57920-000,57924-999
zip_range_4818,city_br_4864,65708-000,65708-999
zip_range_4819,city_br_4865,69370-000,69372-999
zip_range_4820,city_br_4866,76365-000,76369-999
zip_range_4821,city_br_4867,12140-000,12169-999
zip_range_4822,city_br_4868,97800-000,97842-999
zip_range_4823,city_br_4869,58625-000,58639-999
zip_range_4824,city_br_4870,87215-000,87219-999
zip_range_4825,city_br_4871,18650-000,18659-999
zip_range_4826,city_br_4872,95190-000,95199-999
zip_range_4827,city_br_4873,98690-000,98699-999
zip_range_4828,city_br_4874,88765-000,88769-999
zip_range_4829,city_br_4875,97190-000,97194-999
zip_range_4830,city_br_246,29930-001,29949-999
zip_range_4831,city_br_4876,65470-000,65479-999
zip_range_4832,city_br_4877,83900-000,83979-999
zip_range_4833,city_br_4878,59920-000,59924-999
zip_range_4834,city_br_4879,18230-000,18239-999
zip_range_4835,city_br_4880,64378-000,64379-999
zip_range_4836,city_br_4881,89879-000,89879-999
zip_range_4837,city_br_4882,44580-000,44589-999
zip_range_4838,city_br_4883,98865-000,98869-999
zip_range_4839,city_br_4884,58334-000,58336-999
zip_range_4840,city_br_4885,49535-000,49539-999
zip_range_4841,city_br_4886,36590-000,36591-999
zip_range_4842,city_br_4887,76590-000,76599-999
zip_range_4843,city_br_4888,64558-000,64559-999
zip_range_4844,city_br_4889,59585-000,59585-999
zip_range_4845,city_br_4890,68660-000,68664-999
zip_range_4846,city_br_4891,76932-000,76933-999
zip_range_4847,city_br_4892,85877-000,85879-999
zip_range_4848,city_br_4893,89900-000,89904-999
zip_range_4849,city_br_4894,75185-000,75189-999
zip_range_4850,city_br_4895,64330-000,64332-999
zip_range_4851,city_br_4896,77925-000,77929-999
zip_range_4852,city_br_4897,57240-001,57249-999
zip_range_4853,city_br_4898,57940-000,57944-999
zip_range_4854,city_br_4899,97880-000,97884-999
zip_range_4855,city_br_4900,76343-000,76344-999
zip_range_4856,city_br_001,01000-001,05999-999
zip_range_4857,city_br_001,08000-000,08499-999
zip_range_4858,city_br_4901,97980-000,97999-999
zip_range_4859,city_br_4902,69600-000,69619-999
zip_range_4860,city_br_4903,59460-000,59463-999
zip_range_4861,city_br_4904,59480-000,59489-999
zip_range_4862,city_br_4905,13520-000,13524-999
zip_range_4863,city_br_4906,65920-000,65920-999
zip_range_4864,city_br_303,28940-001,28949-999
zip_range_4865,city_br_4907,78835-000,78839-999
zip_range_4866,city_br_4908,95758-000,95759-999
zip_range_4867,city_br_4909,37855-000,37859-999
zip_range_4868,city_br_4910,98323-000,98324-999
zip_range_4869,city_br_4911,88125-000,88129-999
zip_range_4870,city_br_4912,97920-000,97929-999
zip_range_4871,city_br_4913,85929-000,85929-999
zip_range_4872,city_br_4914,86945-000,86949-999
zip_range_4873,city_br_4915,87955-000,87959-999
zip_range_4874,city_br_4916,64430-000,64434-999
zip_range_4875,city_br_4917,39784-000,39784-999
zip_range_4876,city_br_4918,97400-000,97409-999
zip_range_4877,city_br_4919,18940-000,18949-999
zip_range_4878,city_br_4920,65978-000,65979-999
zip_range_4879,city_br_4921,35360-000,35363-999
zip_range_4880,city_br_4922,59518-000,59519-999
zip_range_4881,city_br_4923,65840-000,65849-999
zip_range_4882,city_br_4924,65753-000,65754-999
zip_range_4883,city_br_4925,64770-000,64772-999
zip_range_4884,city_br_4926,65758-000,65759-999
zip_range_4885,city_br_4927,39290-000,39294-999
zip_range_4886,city_br_4928,18130-001,18146-999
zip_range_4887,city_br_4929,37927-000,37929-999
zip_range_4888,city_br_4930,29665-000,29669-999
zip_range_4889,city_br_4931,77368-000,77369-999
zip_range_4890,city_br_4932,57275-000,57279-999
zip_range_4891,city_br_4933,11600-001,11629-999
zip_range_4892,city_br_4934,86240-000,86249-999
zip_range_4893,city_br_4935,37567-000,37567-999
zip_range_4894,city_br_4936,68820-000,68824-999
zip_range_4895,city_br_4937,13790-000,13799-999
zip_range_4896,city_br_4938,36793-000,36794-999
zip_range_4897,city_br_4939,58119-000,58119-999
zip_range_4898,city_br_4940,28550-000,28569-999
zip_range_4899,city_br_4941,35334-000,35334-999
zip_range_4900,city_br_4942,95760-000,95764-999
zip_range_4901,city_br_4943,39795-000,39799-999
zip_range_4902,city_br_4944,35567-000,35569-999
zip_range_4903,city_br_4945,37950-000,37959-999
zip_range_4904,city_br_4946,43850-000,43899-999
zip_range_4905,city_br_4947,35815-000,35819-999
zip_range_4906,city_br_4948,37467-000,37467-999
zip_range_4907,city_br_4949,77990-000,77992-999
zip_range_4908,city_br_4950,69135-000,69139-999
zip_range_4909,city_br_4951,58510-000,58514-999
zip_range_4910,city_br_4952,97340-000,97384-999
zip_range_4911,city_br_4953,75890-000,75899-999
zip_range_4912,city_br_4954,14200-000,14209-999
zip_range_4913,city_br_4955,37408-000,37409-999
zip_range_4914,city_br_4956,36350-000,36359-999
zip_range_4915,city_br_4957,37960-000,37964-999
zip_range_4916,city_br_4958,87220-000,87224-999
zip_range_4917,city_br_4959,59400-000,59409-999
zip_range_4918,city_br_4960,99640-000,99644-999
zip_range_4919,city_br_4961,99240-000,99249-999
zip_range_4920,city_br_4962,77390-000,77394-999
zip_range_4921,city_br_4963,98595-000,98599-999
zip_range_4922,city_br_4964,95795-000,95799-999
zip_range_4923,city_br_4965,59340-000,59342-999
zip_range_4924,city_br_083,11300-001,11399-999
zip_range_4925,city_br_4966,37370-000,37399-999
zip_range_4926,city_br_4967,58158-000,58159-999
zip_range_4927,city_br_4968,97420-000,97449-999
zip_range_4928,city_br_4969,65220-000,65222-999
zip_range_4929,city_br_4970,55860-000,55864-999
zip_range_4930,city_br_4971,58340-000,58341-999
zip_range_4931,city_br_4972,44530-000,44539-999
zip_range_4932,city_br_4973,78365-000,78369-999
zip_range_4933,city_br_4974,93800-001,93879-999
zip_range_4934,city_br_4975,84290-000,84299-999
zip_range_4935,city_br_4976,37690-000,37699-999
zip_range_4936,city_br_4977,68548-000,68548-999
zip_range_4937,city_br_4978,25880-000,25899-999
zip_range_4938,city_br_226,93200-001,93249-999
zip_range_4939,city_br_4979,28990-001,28999-999
zip_range_4940,city_br_257,87110-001,87119-999
zip_range_4941,city_br_4980,99560-000,99579-999
zip_range_4942,city_br_4981,18225-000,18229-999
zip_range_4943,city_br_4982,39728-000,39729-999
zip_range_4944,city_br_4983,18840-000,18859-999
zip_range_4945,city_br_4984,32450-000,32469-999
zip_range_4946,city_br_4985,48485-000,48489-999
zip_range_4947,city_br_4986,57120-000,57129-999
zip_range_4948,city_br_4987,65709-000,65709-999
zip_range_4949,city_br_4988,44220-000,44229-999
zip_range_4950,city_br_4989,85568-000,85569-999
zip_range_4951,city_br_4990,89868-000,89869-999
zip_range_4952,city_br_4991,44740-000,44744-999
zip_range_4953,city_br_4992,89275-000,89277-999
zip_range_4954,city_br_4993,46900-000,46929-999
zip_range_4955,city_br_4994,89770-000,89777-999
zip_range_4956,city_br_4995,15180-000,15189-999
zip_range_4957,city_br_4996,64985-000,64989-999
zip_range_4958,city_br_4997,46450-000,46459-999
zip_range_4959,city_br_4998,64873-000,64874-999
zip_range_4960,city_br_4999,98380-000,98384-999
zip_range_4961,city_br_5000,98675-000,98679-999
zip_range_4962,city_br_5001,96910-000,96919-999
zip_range_4963,city_br_5002,99450-000,99456-999
zip_range_4964,city_br_5003,79590-000,79599-999
zip_range_4965,city_br_5004,35441-000,35441-999
zip_range_4966,city_br_5005,69940-000,69944-999
zip_range_4967,city_br_5006,65783-000,65784-999
zip_range_4968,city_br_5007,37615-000,37619-999
zip_range_4969,city_br_194,75250-001,75264-999
zip_range_4970,city_br_5008,36650-000,36659-999
zip_range_4971,city_br_5009,59250-000,59254-999
zip_range_4972,city_br_5010,36540-000,36541-999
zip_range_4973,city_br_5011,59168-000,59169-999
zip_range_4974,city_br_5012,69925-000,69925-999
zip_range_4975,city_br_5013,37586-000,37587-999
zip_range_4976,city_br_5014,68360-000,68364-999
zip_range_4977,city_br_5015,65935-000,65935-499
zip_range_4978,city_br_5016,39190-000,39199-999
zip_range_4979,city_br_5017,63600-000,63604-999
zip_range_4980,city_br_5018,57515-000,57519-999
zip_range_4981,city_br_5019,62470-000,62479-999
zip_range_4982,city_br_5020,98895-000,98897-999
zip_range_4983,city_br_5021,84220-000,84239-999
zip_range_4984,city_br_5022,48970-000,48989-999
zip_range_4985,city_br_5023,36470-000,36474-999
zip_range_4986,city_br_5024,39745-000,39749-999
zip_range_4987,city_br_5025,36275-000,36279-999
zip_range_4988,city_br_5026,96765-000,96769-999
zip_range_4989,city_br_5027,47350-000,47399-999
zip_range_4990,city_br_5028,99250-000,99254-999
zip_range_4991,city_br_5029,35368-000,35369-999
zip_range_4992,city_br_5030,76934-000,76934-999
zip_range_4993,city_br_5031,95918-000,95919-999
zip_range_4994,city_br_5032,37454-000,37455-999
zip_range_4995,city_br_5033,23890-001,23899-999
zip_range_4996,city_br_041,29160-001,29184-999
zip_range_4997,city_br_5034,89871-000,89871-999
zip_range_4998,city_br_5035,14230-000,14239-999
zip_range_4999,city_br_5036,39165-000,39169-999
zip_range_5000,city_br_5037,58580-000,58587-999
zip_range_5001,city_br_5038,59245-000,59246-999
zip_range_5002,city_br_5039,58260-000,58264-999
zip_range_5003,city_br_5040,35617-000,35619-999
zip_range_5004,city_br_5041,59214-000,59214-999
zip_range_5005,city_br_5042,59663-000,59664-999
zip_range_5006,city_br_5043,68948-000,68949-999
zip_range_5007,city_br_5044,47630-000,47639-999
zip_range_5008,city_br_5045,38760-000,38769-999
zip_range_5009,city_br_5046,39868-000,39869-999
zip_range_5010,city_br_5047,47740-000,47749-999
zip_range_5011,city_br_5048,58955-000,58959-999
zip_range_5012,city_br_5049,13930-000,13939-999
zip_range_5013,city_br_5050,59318-000,59319-999
zip_range_5014,city_br_5051,78668-000,78669-999
zip_range_5015,city_br_5052,44660-000,44669-999
zip_range_5016,city_br_5053,58385-000,58386-999
zip_range_5017,city_br_5054,56900-001,56929-999
zip_range_5018,city_br_5055,14150-000,14159-999
zip_range_5019,city_br_5056,37143-000,37143-999
zip_range_5020,city_br_5057,65269-000,65269-999
zip_range_5021,city_br_5058,75820-000,75822-999
zip_range_5022,city_br_5059,39518-000,39519-999
zip_range_5023,city_br_5060,85885-000,85886-999
zip_range_5024,city_br_5061,37452-000,37453-999
zip_range_5025,city_br_5062,58395-000,58395-999
zip_range_5026,city_br_5063,48700-000,48704-999
zip_range_5027,city_br_5064,59258-000,59258-999
zip_range_5028,city_br_5065,59808-000,59809-999
zip_range_5029,city_br_5066,56140-000,56149-999
zip_range_5030,city_br_5067,39150-000,39159-999
zip_range_5031,city_br_5068,44710-000,44712-999
zip_range_5032,city_br_5069,86340-000,86349-999
zip_range_5033,city_br_5070,56600-000,56639-999
zip_range_5034,city_br_5071,86170-000,86179-999
zip_range_5035,city_br_5072,99170-000,99174-999
zip_range_5036,city_br_5073,92850-000,92899-999
zip_range_5037,city_br_5074,58268-000,58269-999
zip_range_5038,city_br_238,14160-001,14179-999
zip_range_5039,city_br_5075,11910-000,11919-999
zip_range_5040,city_br_5076,97960-000,97969-999
zip_range_5041,city_br_135,35700-001,35719-999
zip_range_5042,city_br_5077,79935-000,79939-999
zip_range_5043,city_br_5078,39688-000,39689-999
zip_range_5044,city_br_5079,99810-000,99819-999
zip_range_5045,city_br_5080,59856-000,59864-999
zip_range_5046,city_br_5081,14735-000,14739-999
zip_range_5047,city_br_5082,88860-000,88861-999
zip_range_5048,city_br_5083,79170-000,79179-999
zip_range_5049,city_br_5084,64285-000,64287-999
zip_range_5050,city_br_5085,28820-000,28859-999
zip_range_5051,city_br_5086,75180-000,75183-999
zip_range_5052,city_br_5087,77580-000,77584-999
zip_range_5053,city_br_5088,97195-000,97199-999
zip_range_5054,city_br_5089,36185-000,36189-999
zip_range_5055,city_br_5090,12690-000,12699-999
zip_range_5056,city_br_5091,69114-000,69116-999
zip_range_5057,city_br_5092,37589-000,37589-999
zip_range_5058,city_br_5093,49480-000,49489-999
zip_range_5059,city_br_5094,36123-000,36125-999
zip_range_5060,city_br_5095,64585-000,64589-999
zip_range_5061,city_br_271,43700-000,43799-999
zip_range_5062,city_br_5096,73930-000,73949-999
zip_range_5063,city_br_5097,36930-000,36939-999
zip_range_5064,city_br_5098,64700-000,64704-999
zip_range_5065,city_br_5099,96890-000,96899-999
zip_range_5066,city_br_155,78550-001,78559-999
zip_range_5067,city_br_5100,84940-000,84944-999
zip_range_5068,city_br_5101,55580-000,55589-999
zip_range_5069,city_br_5102,49630-000,49639-999
zip_range_5070,city_br_5103,73990-000,73999-999
zip_range_5071,city_br_5104,47610-000,47629-999
zip_range_5072,city_br_5105,48565-000,48569-999
zip_range_5073,city_br_5106,65925-000,65926-999
zip_range_5074,city_br_5107,59440-000,59459-999
zip_range_5075,city_br_5108,77940-000,77949-999
zip_range_5076,city_br_5109,48925-000,48929-999
zip_range_5077,city_br_5110,96900-000,96907-999
zip_range_5078,city_br_5111,58342-000,58344-999
zip_range_5079,city_br_150,62000-001,62114-999
zip_range_5080,city_br_5112,35144-000,35145-999
zip_range_5081,city_br_5113,13960-000,13969-999
zip_range_5082,city_br_5114,64720-000,64724-999
zip_range_5083,city_br_5115,58225-000,58227-999
zip_range_5084,city_br_5116,58155-000,58157-999
zip_range_5085,city_br_5117,99300-000,99314-999
zip_range_5086,city_br_5118,37478-000,37479-999
zip_range_5087,city_br_5119,56795-000,56799-999
zip_range_5088,city_br_5120,63620-000,63629-999
zip_range_5089,city_br_5121,88960-000,88964-999
zip_range_5090,city_br_5122,79415-000,79419-999
zip_range_5091,city_br_5123,29927-000,29929-999
zip_range_5092,city_br_027,18000-001,18109-999
zip_range_5093,city_br_283,78890-000,78899-999
zip_range_5094,city_br_5124,58177-000,58177-999
zip_range_5095,city_br_5125,68870-000,68879-999
zip_range_5096,city_br_5126,58800-001,58814-999
zip_range_5097,city_br_5127,46990-000,47099-999
zip_range_5098,city_br_5128,77458-000,77459-999
zip_range_5099,city_br_5129,65860-000,65869-999
zip_range_5100,city_br_5130,65668-000,65669-999
zip_range_5101,city_br_5131,15360-000,15369-999
zip_range_5102,city_br_5132,89855-000,89855-999
zip_range_5103,city_br_5133,85565-000,85567-999
zip_range_5104,city_br_098,13170-001,13182-999
zip_range_5105,city_br_5134,58540-000,58547-999
zip_range_5106,city_br_5135,28637-000,28639-999
zip_range_5107,city_br_5136,55750-000,55754-999
zip_range_5108,city_br_5137,64610-000,64611-999
zip_range_5109,city_br_5138,15380-000,15384-999
zip_range_5110,city_br_090,08600-001,08699-999
zip_range_5111,city_br_5139,95863-000,95864-999
zip_range_5112,city_br_5140,78563-000,78564-999
zip_range_5113,city_br_5141,15880-000,15884-999
zip_range_5114,city_br_5142,69640-000,69649-999
zip_range_5115,city_br_5143,14910-000,14919-999
zip_range_5116,city_br_5144,56780-000,56794-999
zip_range_5117,city_br_101,06750-001,06799-999
zip_range_5118,city_br_5145,47760-000,47799-999
zip_range_5119,city_br_5146,59840-000,59854-999
zip_range_5120,city_br_5147,36165-000,36169-999
zip_range_5121,city_br_5148,62960-000,62964-999
zip_range_5122,city_br_5149,55140-000,55149-999
zip_range_5123,city_br_5150,56480-000,56499-999
zip_range_5124,city_br_5151,19590-000,19599-999
zip_range_5125,city_br_5152,58240-000,58249-999
zip_range_5126,city_br_5153,79975-000,79979-999
zip_range_5127,city_br_5154,18890-000,18899-999
zip_range_5128,city_br_5155,77320-000,77324-999
zip_range_5129,city_br_5156,14725-000,14729-999
zip_range_5130,city_br_5157,68695-000,68699-999
zip_range_5131,city_br_5158,89190-000,89193-999
zip_range_5132,city_br_5159,39550-000,39552-999
zip_range_5133,city_br_5160,77308-000,77309-999
zip_range_5134,city_br_5161,59565-000,59569-999
zip_range_5135,city_br_5162,14720-000,14724-999
zip_range_5136,city_br_5163,77483-000,77484-999
zip_range_5137,city_br_5164,55578-000,55579-999
zip_range_5138,city_br_5165,86125-000,86129-999
zip_range_5139,city_br_5166,13710-000,13714-999
zip_range_5140,city_br_5167,87760-000,87769-999
zip_range_5141,city_br_5168,63750-000,63779-999
zip_range_5142,city_br_5169,64893-000,64894-999
zip_range_5143,city_br_5170,15170-000,15179-999
zip_range_5144,city_br_5171,59240-000,59243-999
zip_range_5145,city_br_5172,89642-000,89649-999
zip_range_5146,city_br_293,78300-000,78306-999
zip_range_5147,city_br_5173,24890-000,24899-999
zip_range_5148,city_br_5174,46600-000,46619-999
zip_range_5149,city_br_5175,57635-000,57639-999
zip_range_5150,city_br_5176,64512-000,64513-999
zip_range_5151,city_br_5177,46580-000,46599-999
zip_range_5152,city_br_5178,44160-000,44179-999
zip_range_5153,city_br_5179,36953-000,36954-999
zip_range_5154,city_br_5180,69480-000,69484-999
zip_range_5155,city_br_5181,87430-000,87449-999
zip_range_5156,city_br_5182,99950-000,99951-999
zip_range_5157,city_br_5183,99490-000,99494-999
zip_range_5158,city_br_5184,45430-000,45434-999
zip_range_5159,city_br_5185,58680-000,58684-999
zip_range_5160,city_br_5186,96760-000,96764-999
zip_range_5161,city_br_5187,38185-000,38189-999
zip_range_5162,city_br_5188,87830-000,87839-999
zip_range_5163,city_br_5189,38980-000,38989-999
zip_range_5164,city_br_5190,18180-000,18184-999
zip_range_5165,city_br_5191,44840-000,44849-999
zip_range_5166,city_br_5192,13760-000,13769-999
zip_range_5167,city_br_5193,78573-000,78574-999
zip_range_5168,city_br_5194,95600-001,95624-999
zip_range_5169,city_br_5195,33980-000,33999-999
zip_range_5170,city_br_5196,14765-000,14769-999
zip_range_5171,city_br_5197,76640-000,76649-999
zip_range_5172,city_br_5198,57640-000,57659-999
zip_range_5173,city_br_5199,95860-000,95862-999
zip_range_5174,city_br_5200,15900-000,15909-999
zip_range_5175,city_br_5201,55790-000,55799-999
zip_range_5176,city_br_5202,18740-000,18744-999
zip_range_5177,city_br_5203,18425-000,18429-999
zip_range_5178,city_br_5204,98410-000,98414-999
zip_range_5179,city_br_5205,79765-000,79769-999
zip_range_5180,city_br_5206,19210-000,19219-999
zip_range_5181,city_br_5207,69970-000,69974-999
zip_range_5182,city_br_5208,63145-000,63149-999
zip_range_5183,city_br_5209,68990-000,68996-999
zip_range_5184,city_br_5210,19820-000,19829-999
zip_range_5185,city_br_5211,35140-000,35143-999
zip_range_5186,city_br_5212,65820-000,65829-999
zip_range_5187,city_br_245,18270-001,18284-999
zip_range_5188,city_br_5213,63660-000,63669-999
zip_range_5189,city_br_089,12000-001,12119-999
zip_range_5190,city_br_5214,58753-000,58754-999
zip_range_5191,city_br_5215,96290-000,96299-999
zip_range_5192,city_br_5216,69550-001,69559-999
zip_range_5193,city_br_5217,58735-000,58736-999
zip_range_5194,city_br_207,45985-001,45999-999
zip_range_5195,city_br_5218,84530-000,84534-999
zip_range_5196,city_br_5219,36580-000,36584-999
zip_range_5197,city_br_5220,76928-000,76928-999
zip_range_5198,city_br_5221,62610-000,62619-999
zip_range_5199,city_br_5222,18830-000,18839-999
zip_range_5200,city_br_5223,84260-001,84279-999
zip_range_5201,city_br_5224,49910-000,49919-999
zip_range_5202,city_br_5225,59955-000,59959-999
zip_range_5203,city_br_5226,59338-000,59339-999
zip_range_5204,city_br_5227,98500-000,98527-999
zip_range_5205,city_br_5228,58665-000,58669-999
zip_range_5206,city_br_5229,44280-000,44299-999
zip_range_5207,city_br_5230,19280-000,19299-999
zip_range_5208,city_br_5231,48770-000,48779-999
zip_range_5209,city_br_215,39800-001,39809-999
zip_range_5210,city_br_5232,45465-000,45469-999
zip_range_5211,city_br_5233,57265-000,57269-999
zip_range_5212,city_br_5234,79190-000,79199-999
zip_range_5213,city_br_019,64000-001,64099-999
zip_range_5214,city_br_5235,73795-000,73799-999
zip_range_5215,city_br_181,25950-001,25999-999
zip_range_5216,city_br_5236,55305-000,55309-999
zip_range_5217,city_br_5237,75175-000,75179-999
zip_range_5218,city_br_5238,68773-000,68773-999
zip_range_5219,city_br_5239,87240-000,87249-999
zip_range_5220,city_br_5240,95535-000,95537-999
zip_range_5221,city_br_5241,44270-000,44279-999
zip_range_5222,city_br_5242,56190-000,56199-999
zip_range_5223,city_br_5243,78505-000,78507-999
zip_range_5224,city_br_5244,87890-000,87899-999
zip_range_5225,city_br_5245,85990-000,85997-999
zip_range_5226,city_br_5246,14745-000,14749-999
zip_range_5227,city_br_5247,68285-000,68299-999
zip_range_5228,city_br_5248,78775-000,78779-999
zip_range_5229,city_br_5249,95890-000,95892-999
zip_range_5230,city_br_5250,76866-000,76866-999
zip_range_5231,city_br_5251,62320-000,62339-999
zip_range_5232,city_br_5252,84300-000,84319-999
zip_range_5233,city_br_5253,59678-000,59679-999
zip_range_5234,city_br_5254,59178-000,59179-999
zip_range_5235,city_br_5255,18530-000,18534-999
zip_range_5236,city_br_5256,89875-000,89877-999
zip_range_5237,city_br_5257,88200-000,88209-999
zip_range_5238,city_br_5258,83190-000,83199-999
zip_range_5239,city_br_5259,55870-000,55879-999
zip_range_5240,city_br_5260,59320-000,59323-999
zip_range_5241,city_br_5261,88940-000,88949-999
zip_range_5242,city_br_5262,65420-000,65429-999
zip_range_5243,city_br_5263,89120-000,89120-999
zip_range_5244,city_br_5264,89545-000,89549-999
zip_range_5245,city_br_5265,18860-000,18869-999
zip_range_5246,city_br_169,65630-001,65639-999
zip_range_5247,city_br_5266,35180-001,35184-999
zip_range_5248,city_br_5267,99345-000,99349-999
zip_range_5249,city_br_5268,36325-000,36327-999
zip_range_5250,city_br_5269,98680-000,98689-999
zip_range_5251,city_br_5270,38880-000,38899-999
zip_range_5252,city_br_5271,49300-000,49319-999
zip_range_5253,city_br_5272,77640-000,77644-999
zip_range_5254,city_br_5273,77900-000,77902-999
zip_range_5255,city_br_5274,36512-000,36512-999
zip_range_5256,city_br_5275,37563-000,37563-999
zip_range_5257,city_br_5276,37630-000,37639-999
zip_range_5258,city_br_200,85900-001,85928-999
zip_range_5259,city_br_5277,49280-000,49289-999
zip_range_5260,city_br_5278,84935-000,84939-999
zip_range_5261,city_br_5279,36844-000,36846-999
zip_range_5262,city_br_5280,68680-000,68684-999
zip_range_5263,city_br_5281,69685-000,69699-999
zip_range_5264,city_br_5282,55125-000,55129-999
zip_range_5265,city_br_5283,78695-000,78697-999
zip_range_5266,city_br_5284,97418-000,97419-999
zip_range_5267,city_br_5285,18265-000,18269-999
zip_range_5268,city_br_5286,95560-000,95567-999
zip_range_5269,city_br_5287,17360-000,17369-999
zip_range_5270,city_br_5288,59584-000,59584-999
zip_range_5271,city_br_5289,14935-000,14939-999
zip_range_5272,city_br_5290,68647-000,68649-999
zip_range_5273,city_br_5291,55805-000,55809-999
zip_range_5274,city_br_5292,57370-000,57379-999
zip_range_5275,city_br_5293,68198-000,68199-999
zip_range_5276,city_br_5294,62690-000,62699-999
zip_range_5277,city_br_5295,28750-000,28769-999
zip_range_5278,city_br_5296,95590-000,95594-999
zip_range_5279,city_br_5297,95948-000,95949-999
zip_range_5280,city_br_5298,45170-000,45176-999
zip_range_5281,city_br_5299,12120-000,12129-999
zip_range_5282,city_br_5300,99725-000,99729-999
zip_range_5283,city_br_5301,89490-000,89499-999
zip_range_5284,city_br_5302,85485-000,85499-999
zip_range_5285,city_br_5303,95580-000,95584-999
zip_range_5286,city_br_5304,37410-000,37419-999
zip_range_5287,city_br_5305,95660-000,95669-999
zip_range_5288,city_br_5306,98910-000,98914-999
zip_range_5289,city_br_5307,95575-000,95576-999
zip_range_5290,city_br_5308,15770-000,15772-999
zip_range_5291,city_br_225,79600-001,79669-999
zip_range_5292,city_br_5309,39205-000,39209-999
zip_range_5293,city_br_5310,99675-000,99679-999
zip_range_5294,city_br_5311,98600-000,98634-999
zip_range_5295,city_br_5312,37190-000,37194-999
zip_range_5296,city_br_5313,75720-000,75729-999
zip_range_5297,city_br_5314,25800-001,25844-999
zip_range_5298,city_br_5315,88862-000,88864-999
zip_range_5299,city_br_5316,88710-000,88714-999
zip_range_5300,city_br_5317,89650-000,89651-999
zip_range_5301,city_br_211,75380-001,75394-999
zip_range_5302,city_br_5318,56250-000,56259-999
zip_range_5303,city_br_5319,99615-000,99639-999
zip_range_5304,city_br_5320,58920-000,58921-999
zip_range_5305,city_br_5321,56870-000,56894-999
zip_range_5306,city_br_5322,95840-000,95859-999
zip_range_5307,city_br_5323,59685-000,59689-999
zip_range_5308,city_br_5324,65727-000,65727-999
zip_range_5309,city_br_5325,76460-000,76464-999
zip_range_5310,city_br_5326,89176-000,89177-999
zip_range_5311,city_br_286,88700-001,88709-999
zip_range_5312,city_br_5327,48790-000,48799-999
zip_range_5313,city_br_5328,68385-000,68387-999
zip_range_5314,city_br_5329,98930-000,98939-999
zip_range_5315,city_br_5330,68455-001,68464-999
zip_range_5316,city_br_5331,65378-000,65379-999
zip_range_5317,city_br_5332,12930-000,12934-999
zip_range_5318,city_br_5333,35125-000,35129-999
zip_range_5319,city_br_5334,89898-000,89898-999
zip_range_5320,city_br_5335,99330-000,99339-999
zip_range_5321,city_br_5336,83480-000,83489-999
zip_range_5322,city_br_5337,87450-000,87469-999
zip_range_5323,city_br_5338,65763-000,65764-999
zip_range_5324,city_br_5339,17600-001,17629-999
zip_range_5325,city_br_5340,38480-000,38489-999
zip_range_5326,city_br_5341,56540-000,56549-999
zip_range_5327,city_br_5342,99878-000,99879-999
zip_range_5328,city_br_5343,98170-000,98174-999
zip_range_5329,city_br_5344,95775-000,95777-999
zip_range_5330,city_br_5345,98940-000,98946-999
zip_range_5331,city_br_5346,56760-000,56779-999
zip_range_5332,city_br_5347,85945-000,85947-999
zip_range_5333,city_br_5348,17930-000,17939-999
zip_range_5334,city_br_5349,77704-000,77707-999
zip_range_5335,city_br_5350,77743-000,77744-999
zip_range_5336,city_br_5351,65278-000,65278-999
zip_range_5337,city_br_5352,65276-000,65277-999
zip_range_5338,city_br_5353,15280-000,15284-999
zip_range_5339,city_br_5354,39660-000,39662-999
zip_range_5340,city_br_5355,15755-000,15759-999
zip_range_5341,city_br_5356,96148-000,96149-999
zip_range_5342,city_br_5357,62655-000,62659-999
zip_range_5343,city_br_5358,76110-000,76119-999
zip_range_5344,city_br_5359,75970-000,75979-999
zip_range_5345,city_br_5360,85150-000,85154-999
zip_range_5346,city_br_5361,88930-000,88934-999
zip_range_5347,city_br_5362,37496-000,37497-999
zip_range_5348,city_br_5363,65580-000,65584-999
zip_range_5349,city_br_5364,69530-000,69539-999
zip_range_5350,city_br_5365,48950-000,48959-999
zip_range_5351,city_br_310,36500-000,36509-999
zip_range_5352,city_br_5366,39320-000,39327-999
zip_range_5353,city_br_5367,45310-000,45314-999
zip_range_5354,city_br_5368,45545-000,45549-999
zip_range_5355,city_br_5369,62350-000,62359-999
zip_range_5356,city_br_5370,35338-000,35339-999
zip_range_5357,city_br_5371,15225-000,15229-999
zip_range_5358,city_br_5372,45550-000,45559-999
zip_range_5359,city_br_5373,11680-000,11699-999
zip_range_5360,city_br_081,38000-001,38107-999
zip_range_5361,city_br_028,38400-001,38439-999
zip_range_5362,city_br_5374,17440-000,17449-999
zip_range_5363,city_br_5375,85440-000,85449-999
zip_range_5364,city_br_5376,98898-000,98899-999
zip_range_5365,city_br_5377,15890-000,15894-999
zip_range_5366,city_br_5378,44950-000,44959-999
zip_range_5367,city_br_5379,69358-000,69359-999
zip_range_5368,city_br_5380,76525-000,76529-999
zip_range_5369,city_br_5381,58915-000,58919-999
zip_range_5370,city_br_5382,68632-000,68632-999
zip_range_5371,city_br_5383,63310-000,63319-999
zip_range_5372,city_br_5384,59865-000,59869-999
zip_range_5373,city_br_5385,49260-000,49269-999
zip_range_5374,city_br_5386,44798-000,44799-999
zip_range_5375,city_br_5387,39878-000,39879-999
zip_range_5376,city_br_5388,58497-000,58499-999
zip_range_5377,city_br_5389,62660-000,62664-999
zip_range_5378,city_br_263,87500-001,87524-999
zip_range_5379,city_br_5390,45690-000,45694-999
zip_range_5380,city_br_5391,38610-000,38624-999
zip_range_5381,city_br_5392,64120-000,64125-999
zip_range_5382,city_br_5393,99215-000,99219-999
zip_range_5383,city_br_5394,84600-000,84614-999
zip_range_5384,city_br_5395,38288-000,38289-999
zip_range_5385,city_br_5396,89845-000,89847-999
zip_range_5386,city_br_5397,78543-000,78544-999
zip_range_5387,city_br_5398,57800-000,57819-999
zip_range_5388,city_br_5399,15250-000,15259-999
zip_range_5389,city_br_5400,87640-000,87649-999
zip_range_5390,city_br_5401,97755-000,97759-999
zip_range_5391,city_br_5402,59670-000,59674-999
zip_range_5392,city_br_5403,86280-000,86289-999
zip_range_5393,city_br_5404,46350-000,46359-999
zip_range_5394,city_br_5405,15760-000,15762-999
zip_range_5395,city_br_5406,65530-000,65534-999
zip_range_5396,city_br_5407,16650-000,16659-999
zip_range_5397,city_br_5408,76400-000,76409-999
zip_range_5398,city_br_5409,76335-000,76339-999
zip_range_5399,city_br_5410,38630-000,38639-999
zip_range_5400,city_br_5411,68140-000,68142-999
zip_range_5401,city_br_5412,88650-000,88657-999
zip_range_5402,city_br_5413,62650-000,62654-999
zip_range_5403,city_br_5414,35380-000,35381-999
zip_range_5404,city_br_5415,69130-000,69134-999
zip_range_5405,city_br_5416,45680-000,45689-999
zip_range_5406,city_br_5417,64860-000,64864-999
zip_range_5407,city_br_5418,39315-000,39316-999
zip_range_5408,city_br_5419,69180-000,69189-999
zip_range_5409,city_br_262,97500-001,97537-999
zip_range_5410,city_br_5420,62460-000,62469-999
zip_range_5411,city_br_5421,76929-000,76929-999
zip_range_5412,city_br_5422,88625-000,88639-999
zip_range_5413,city_br_5423,15850-000,15859-999
zip_range_5414,city_br_5424,88840-000,88844-999
zip_range_5415,city_br_5425,75790-000,75794-999
zip_range_5416,city_br_5426,46810-000,46819-999
zip_range_5417,city_br_5427,95200-000,95229-999
zip_range_5418,city_br_5428,78253-000,78253-999
zip_range_5419,city_br_5429,76867-000,76867-999
zip_range_5420,city_br_5430,76923-000,76923-999
zip_range_5421,city_br_5431,96878-000,96879-999
zip_range_5422,city_br_5432,95778-000,95779-999
zip_range_5423,city_br_5433,95833-000,95834-999
zip_range_5424,city_br_5434,45400-000,45415-999
zip_range_5425,city_br_5435,27600-000,27659-999
zip_range_5426,city_br_5436,64300-000,64307-999
zip_range_5427,city_br_5437,48890-000,48894-999
zip_range_5428,city_br_5438,15520-000,15524-999
zip_range_5429,city_br_240,13270-001,13279-999
zip_range_5430,city_br_5439,16880-000,16899-999
zip_range_5431,city_br_153,72870-001,72879-999
zip_range_5432,city_br_5440,99290-000,99299-999
zip_range_5433,city_br_5441,89690-000,89693-999
zip_range_5434,city_br_5442,89638-000,89639-999
zip_range_5435,city_br_5443,12935-000,12939-999
zip_range_5436,city_br_5444,35199-000,35199-999
zip_range_5437,city_br_5445,29295-000,29299-999
zip_range_5438,city_br_5446,37922-000,37924-999
zip_range_5439,city_br_5447,89675-000,89676-999
zip_range_5440,city_br_5448,65430-000,65439-999
zip_range_5441,city_br_5449,39535-000,39535-999
zip_range_5442,city_br_5450,13880-000,13889-999
zip_range_5443,city_br_5451,06730-000,06749-999
zip_range_5444,city_br_216,37000-001,37109-999
zip_range_5445,city_br_5452,75355-000,75359-999
zip_range_5446,city_br_5453,38794-000,38799-999
zip_range_5447,city_br_5454,62265-000,62269-999
zip_range_5448,city_br_5455,28375-000,28379-999
zip_range_5449,city_br_5456,58620-000,58624-999
zip_range_5450,city_br_5457,59185-000,59186-999
zip_range_5451,city_br_5458,63540-000,63559-999
zip_range_5452,city_br_5459,64773-000,64774-999
zip_range_5453,city_br_5460,39260-000,39269-999
zip_range_5454,city_br_5461,44635-000,44639-999
zip_range_5455,city_br_5462,44715-000,44717-999
zip_range_5456,city_br_093,78110-001,78169-999
zip_range_5457,city_br_5463,64525-000,64527-999
zip_range_5458,city_br_5464,44690-000,44694-999
zip_range_5459,city_br_267,13220-001,13229-999
zip_range_5460,city_br_5465,44565-000,44569-999
zip_range_5461,city_br_5466,39450-000,39454-999
zip_range_5462,city_br_5467,27700-000,27899-999
zip_range_5463,city_br_5468,38780-000,38784-999
zip_range_5464,city_br_5469,95800-000,95819-999
zip_range_5465,city_br_5470,29375-000,29379-999
zip_range_5466,city_br_5471,59925-000,59929-999
zip_range_5467,city_br_5472,84345-000,84349-999
zip_range_5468,city_br_5473,55270-000,55279-999
zip_range_5469,city_br_5474,78880-000,78884-999
zip_range_5470,city_br_5475,44470-000,44479-999
zip_range_5471,city_br_5476,59184-000,59184-999
zip_range_5472,city_br_5477,96880-000,96887-999
zip_range_5473,city_br_5478,17560-000,17569-999
zip_range_5474,city_br_5479,85845-000,85849-999
zip_range_5475,city_br_5480,64568-000,64569-999
zip_range_5476,city_br_5481,95330-000,95332-999
zip_range_5477,city_br_5482,56120-000,56129-999
zip_range_5478,city_br_5483,39458-000,39459-999
zip_range_5479,city_br_5484,85585-000,85597-999
zip_range_5480,city_br_5485,45955-000,45959-999
zip_range_5481,city_br_5486,39663-000,39664-999
zip_range_5482,city_br_5487,38150-000,38159-999
zip_range_5483,city_br_5488,35359-000,35359-999
zip_range_5484,city_br_5489,55760-000,55764-999
zip_range_5485,city_br_5490,55770-000,55779-999
zip_range_5486,city_br_235,33200-000,33349-999
zip_range_5487,city_br_5491,95972-000,95974-999
zip_range_5488,city_br_5492,99820-000,99824-999
zip_range_5489,city_br_140,94400-001,94799-999
zip_range_5490,city_br_5493,29130-001,29139-999
zip_range_5491,city_br_5494,65215-000,65217-999
zip_range_5492,city_br_5495,75265-000,75279-999
zip_range_5493,city_br_5496,55850-000,55859-999
zip_range_5494,city_br_5497,98450-000,98459-999
zip_range_5495,city_br_5498,79710-000,79719-999
zip_range_5496,city_br_5499,75555-000,75559-999
zip_range_5497,city_br_5500,57700-000,57719-999
zip_range_5498,city_br_5501,36570-000,36575-999
zip_range_5499,city_br_5502,59815-000,59819-999
zip_range_5500,city_br_5503,62300-000,62319-999
zip_range_5501,city_br_5504,99350-000,99359-999
zip_range_5502,city_br_5505,88443-000,88444-999
zip_range_5503,city_br_5506,89560-000,89569-999
zip_range_5504,city_br_5507,36895-000,36899-999
zip_range_5505,city_br_5508,58822-000,58822-999
zip_range_5506,city_br_5509,68780-000,68784-999
zip_range_5507,city_br_5510,78245-000,78249-999
zip_range_5508,city_br_5511,73825-000,73829-999
zip_range_5509,city_br_5512,59192-000,59193-999
zip_range_5510,city_br_5513,95334-000,95334-999
zip_range_5511,city_br_5514,99955-000,99959-999
zip_range_5512,city_br_5515,99155-000,99159-999
zip_range_5513,city_br_5516,64688-000,64689-999
zip_range_5514,city_br_5517,97385-000,97389-999
zip_range_5515,city_br_5518,65924-000,65924-999
zip_range_5516,city_br_5519,29843-000,29844-999
zip_range_5517,city_br_5520,76393-000,76393-999
zip_range_5518,city_br_5521,78645-000,78649-999
zip_range_5519,city_br_5522,29785-000,29794-999
zip_range_5520,city_br_047,29100-001,29129-999
zip_range_5521,city_br_5523,76980-001,76989-999
zip_range_5522,city_br_5524,13280-001,13289-999
zip_range_5523,city_br_5525,14740-000,14744-999
zip_range_5524,city_br_5526,39630-000,39634-999
zip_range_5525,city_br_5527,37465-000,37465-999
zip_range_5526,city_br_5528,39730-000,39734-999
zip_range_5527,city_br_5529,39715-000,39717-999
zip_range_5528,city_br_5530,85390-000,85399-999
zip_range_5529,city_br_5531,36520-000,36524-999
zip_range_5530,city_br_5532,68620-000,68624-999
zip_range_5531,city_br_5533,98415-000,98429-999
zip_range_5532,city_br_5534,15920-000,15929-999
zip_range_5533,city_br_5535,95325-000,95329-999
zip_range_5534,city_br_5536,98535-000,98539-999
zip_range_5535,city_br_5537,58710-000,58712-999
zip_range_5536,city_br_5538,89148-000,89149-999
zip_range_5537,city_br_087,29000-001,29099-999
zip_range_5538,city_br_5539,15713-000,15714-999
zip_range_5539,city_br_068,45000-001,45119-999
zip_range_5540,city_br_5540,98850-000,98854-999
zip_range_5541,city_br_222,55600-001,55619-999
zip_range_5542,city_br_5541,68924-000,68924-999
zip_range_5543,city_br_5542,65350-000,65359-999
zip_range_5544,city_br_5543,68383-000,68384-999
zip_range_5545,city_br_5544,85520-000,85524-999
zip_range_5546,city_br_5545,65320-000,65334-999
zip_range_5547,city_br_5546,36720-000,36724-999
zip_range_5548,city_br_110,27200-001,27299-999
zip_range_5549,city_br_236,18110-001,18119-999
zip_range_5550,city_br_5547,15500-001,15519-999
zip_range_5551,city_br_5548,46970-000,46979-999
zip_range_5552,city_br_5549,64548-000,64549-999
zip_range_5553,city_br_5550,77860-000,77869-999
zip_range_5554,city_br_5551,47940-000,47949-999
zip_range_5555,city_br_5552,37512-000,37513-999
zip_range_5556,city_br_5553,84950-000,84969-999
zip_range_5557,city_br_5554,45460-000,45464-999
zip_range_5558,city_br_5555,95893-000,95894-999
zip_range_5559,city_br_5556,89157-000,89159-999
zip_range_5560,city_br_5557,77880-000,77884-999
zip_range_5561,city_br_5558,87535-000,87537-999
zip_range_5562,city_br_5559,95588-000,95589-999
zip_range_5563,city_br_5560,89820-000,89823-999
zip_range_5564,city_br_5561,69930-000,69930-999
zip_range_5565,city_br_5562,89780-000,89789-999
zip_range_5566,city_br_5563,89825-000,89827-999
zip_range_5567,city_br_5564,55555-000,55559-999
zip_range_5568,city_br_5565,68555-001,68559-999
zip_range_5569,city_br_5566,47400-000,47439-999
zip_range_5570,city_br_5567,58515-000,58519-999
zip_range_5571,city_br_5568,15265-000,15269-999
zip_range_5572,city_br_5569,65365-000,65367-999
zip_range_5573,city_br_5570,89633-000,89633-999

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

## File: data\res.city.csv

```csv
id,country_id:id,state_id:id,name
city_br_001,base.br,base.state_br_sp,São Paulo
city_br_002,base.br,base.state_br_rj,Rio de Janeiro
city_br_003,base.br,base.state_br_df,Brasília
city_br_004,base.br,base.state_br_ce,Fortaleza
city_br_005,base.br,base.state_br_ba,Salvador
city_br_006,base.br,base.state_br_mg,Belo Horizonte
city_br_007,base.br,base.state_br_am,Manaus
city_br_008,base.br,base.state_br_pr,Curitiba
city_br_009,base.br,base.state_br_pe,Recife
city_br_010,base.br,base.state_br_go,Goiânia
city_br_011,base.br,base.state_br_rs,Porto Alegre
city_br_012,base.br,base.state_br_pa,Belém
city_br_013,base.br,base.state_br_sp,Guarulhos
city_br_014,base.br,base.state_br_sp,Campinas
city_br_015,base.br,base.state_br_ma,São Luís
city_br_016,base.br,base.state_br_al,Maceió
city_br_017,base.br,base.state_br_ms,Campo Grande
city_br_018,base.br,base.state_br_rj,São Gonçalo
city_br_019,base.br,base.state_br_pi,Teresina
city_br_020,base.br,base.state_br_pb,João Pessoa
city_br_021,base.br,base.state_br_sp,São Bernardo do Campo
city_br_022,base.br,base.state_br_rj,Duque de Caxias
city_br_023,base.br,base.state_br_rj,Nova Iguaçu
city_br_024,base.br,base.state_br_rn,Natal
city_br_025,base.br,base.state_br_sp,Santo André
city_br_026,base.br,base.state_br_sp,Osasco
city_br_027,base.br,base.state_br_sp,Sorocaba
city_br_028,base.br,base.state_br_mg,Uberlândia
city_br_029,base.br,base.state_br_sp,Ribeirão Preto
city_br_030,base.br,base.state_br_sp,São José dos Campos
city_br_031,base.br,base.state_br_mt,Cuiabá
city_br_032,base.br,base.state_br_pe,Jaboatão dos Guararapes
city_br_033,base.br,base.state_br_mg,Contagem
city_br_034,base.br,base.state_br_sc,Joinville
city_br_035,base.br,base.state_br_ba,Feira de Santana
city_br_036,base.br,base.state_br_se,Aracaju
city_br_037,base.br,base.state_br_pr,Londrina
city_br_038,base.br,base.state_br_mg,Juiz de Fora
city_br_039,base.br,base.state_br_sc,Florianópolis
city_br_040,base.br,base.state_br_go,Aparecida de Goiânia
city_br_041,base.br,base.state_br_es,Serra
city_br_042,base.br,base.state_br_rj,Campos dos Goytacazes
city_br_043,base.br,base.state_br_rj,Belford Roxo
city_br_044,base.br,base.state_br_rj,Niterói
city_br_045,base.br,base.state_br_sp,São José do Rio Preto
city_br_046,base.br,base.state_br_pa,Ananindeua
city_br_047,base.br,base.state_br_es,Vila Velha
city_br_048,base.br,base.state_br_rs,Caxias do Sul
city_br_049,base.br,base.state_br_ro,Porto Velho
city_br_050,base.br,base.state_br_sp,Mogi das Cruzes
city_br_051,base.br,base.state_br_sp,Jundiaí
city_br_052,base.br,base.state_br_ap,Macapá
city_br_053,base.br,base.state_br_rj,São João de Meriti
city_br_054,base.br,base.state_br_sp,Piracicaba
city_br_055,base.br,base.state_br_pb,Campina Grande
city_br_056,base.br,base.state_br_sp,Santos
city_br_057,base.br,base.state_br_sp,Mauá
city_br_058,base.br,base.state_br_mg,Montes Claros
city_br_059,base.br,base.state_br_rr,Boa Vista
city_br_060,base.br,base.state_br_mg,Betim
city_br_061,base.br,base.state_br_pr,Maringá
city_br_062,base.br,base.state_br_go,Anápolis
city_br_063,base.br,base.state_br_sp,Diadema
city_br_064,base.br,base.state_br_sp,Carapicuíba
city_br_065,base.br,base.state_br_pe,Petrolina
city_br_066,base.br,base.state_br_sp,Bauru
city_br_067,base.br,base.state_br_pe,Caruaru
city_br_068,base.br,base.state_br_ba,Vitória da Conquista
city_br_069,base.br,base.state_br_sp,Itaquaquecetuba
city_br_070,base.br,base.state_br_ac,Rio Branco
city_br_071,base.br,base.state_br_sc,Blumenau
city_br_072,base.br,base.state_br_pr,Ponta Grossa
city_br_073,base.br,base.state_br_ce,Caucaia
city_br_074,base.br,base.state_br_es,Cariacica
city_br_075,base.br,base.state_br_sp,Franca
city_br_076,base.br,base.state_br_pe,Olinda
city_br_077,base.br,base.state_br_sp,Praia Grande
city_br_078,base.br,base.state_br_pr,Cascavel
city_br_079,base.br,base.state_br_rs,Canoas
city_br_080,base.br,base.state_br_pe,Paulista
city_br_081,base.br,base.state_br_mg,Uberaba
city_br_082,base.br,base.state_br_pa,Santarém
city_br_083,base.br,base.state_br_sp,São Vicente
city_br_084,base.br,base.state_br_mg,Ribeirão das Neves
city_br_085,base.br,base.state_br_pr,São José dos Pinhais
city_br_086,base.br,base.state_br_rs,Pelotas
city_br_087,base.br,base.state_br_es,Vitória
city_br_088,base.br,base.state_br_sp,Barueri
city_br_089,base.br,base.state_br_sp,Taubaté
city_br_090,base.br,base.state_br_sp,Suzano
city_br_091,base.br,base.state_br_to,Palmas
city_br_092,base.br,base.state_br_ba,Camaçari
city_br_093,base.br,base.state_br_mt,Várzea Grande
city_br_094,base.br,base.state_br_sp,Limeira
city_br_095,base.br,base.state_br_sp,Guarujá
city_br_096,base.br,base.state_br_ce,Juazeiro do Norte
city_br_097,base.br,base.state_br_pr,Foz do Iguaçu
city_br_098,base.br,base.state_br_sp,Sumaré
city_br_099,base.br,base.state_br_rj,Petrópolis
city_br_100,base.br,base.state_br_sp,Cotia
city_br_101,base.br,base.state_br_sp,Taboão da Serra
city_br_102,base.br,base.state_br_ma,Imperatriz
city_br_103,base.br,base.state_br_rs,Santa Maria
city_br_104,base.br,base.state_br_sc,São José
city_br_105,base.br,base.state_br_pa,Marabá
city_br_106,base.br,base.state_br_pa,Parauapebas
city_br_107,base.br,base.state_br_rs,Gravataí
city_br_108,base.br,base.state_br_rn,Mossoró
city_br_109,base.br,base.state_br_sc,Itajaí
city_br_110,base.br,base.state_br_rj,Volta Redonda
city_br_111,base.br,base.state_br_mg,Governador Valadares
city_br_112,base.br,base.state_br_sp,Indaiatuba
city_br_113,base.br,base.state_br_sp,São Carlos
city_br_114,base.br,base.state_br_sc,Chapecó
city_br_115,base.br,base.state_br_rn,Parnamirim
city_br_116,base.br,base.state_br_sp,Embu das Artes
city_br_117,base.br,base.state_br_rj,Macaé
city_br_118,base.br,base.state_br_mt,Rondonópolis
city_br_119,base.br,base.state_br_ma,São José de Ribamar
city_br_120,base.br,base.state_br_ms,Dourados
city_br_121,base.br,base.state_br_sp,Araraquara
city_br_122,base.br,base.state_br_sp,Jacareí
city_br_123,base.br,base.state_br_sp,Marília
city_br_124,base.br,base.state_br_sp,Americana
city_br_125,base.br,base.state_br_sp,Hortolândia
city_br_126,base.br,base.state_br_ba,Juazeiro
city_br_127,base.br,base.state_br_al,Arapiraca
city_br_128,base.br,base.state_br_ce,Maracanaú
city_br_129,base.br,base.state_br_sp,Itapevi
city_br_130,base.br,base.state_br_pr,Colombo
city_br_131,base.br,base.state_br_mg,Divinópolis
city_br_132,base.br,base.state_br_rj,Magé
city_br_133,base.br,base.state_br_rs,Novo Hamburgo
city_br_134,base.br,base.state_br_mg,Ipatinga
city_br_135,base.br,base.state_br_mg,Sete Lagoas
city_br_136,base.br,base.state_br_go,Rio Verde
city_br_137,base.br,base.state_br_go,Águas Lindas de Goiás
city_br_138,base.br,base.state_br_sp,Presidente Prudente
city_br_139,base.br,base.state_br_rj,Itaboraí
city_br_140,base.br,base.state_br_rs,Viamão
city_br_141,base.br,base.state_br_sc,Palhoça
city_br_142,base.br,base.state_br_rj,Cabo Frio
city_br_143,base.br,base.state_br_mg,Santa Luzia
city_br_144,base.br,base.state_br_rs,São Leopoldo
city_br_145,base.br,base.state_br_sc,Criciúma
city_br_146,base.br,base.state_br_go,Luziânia
city_br_147,base.br,base.state_br_rs,Passo Fundo
city_br_148,base.br,base.state_br_ba,Lauro de Freitas
city_br_149,base.br,base.state_br_pe,Cabo de Santo Agostinho
city_br_150,base.br,base.state_br_ce,Sobral
city_br_151,base.br,base.state_br_sp,Rio Claro
city_br_152,base.br,base.state_br_sp,Araçatuba
city_br_153,base.br,base.state_br_go,Valparaíso de Goiás
city_br_154,base.br,base.state_br_rj,Maricá
city_br_155,base.br,base.state_br_mt,Sinop
city_br_156,base.br,base.state_br_se,Nossa Senhora do Socorro
city_br_157,base.br,base.state_br_pa,Castanhal
city_br_158,base.br,base.state_br_rs,Rio Grande
city_br_159,base.br,base.state_br_rj,Nova Friburgo
city_br_160,base.br,base.state_br_rs,Alvorada
city_br_161,base.br,base.state_br_ba,Itabuna
city_br_162,base.br,base.state_br_es,Cachoeiro de Itapemirim
city_br_163,base.br,base.state_br_sp,Santa Bárbara d'Oeste
city_br_164,base.br,base.state_br_sc,Jaraguá do Sul
city_br_165,base.br,base.state_br_pr,Guarapuava
city_br_166,base.br,base.state_br_sp,Ferraz de Vasconcelos
city_br_167,base.br,base.state_br_ba,Ilhéus
city_br_168,base.br,base.state_br_sp,Bragança Paulista
city_br_169,base.br,base.state_br_ma,Timon
city_br_170,base.br,base.state_br_to,Araguaína
city_br_171,base.br,base.state_br_mg,Ibirité
city_br_172,base.br,base.state_br_rj,Barra Mansa
city_br_173,base.br,base.state_br_sp,Itu
city_br_174,base.br,base.state_br_ba,Porto Seguro
city_br_175,base.br,base.state_br_rj,Angra dos Reis
city_br_176,base.br,base.state_br_rj,Mesquita
city_br_177,base.br,base.state_br_es,Linhares
city_br_178,base.br,base.state_br_sp,São Caetano do Sul
city_br_179,base.br,base.state_br_sp,Pindamonhangaba
city_br_180,base.br,base.state_br_sp,Francisco Morato
city_br_181,base.br,base.state_br_rj,Teresópolis
city_br_182,base.br,base.state_br_sc,Lages
city_br_183,base.br,base.state_br_mg,Poços de Caldas
city_br_184,base.br,base.state_br_pi,Parnaíba
city_br_185,base.br,base.state_br_ba,Barreiras
city_br_186,base.br,base.state_br_mg,Patos de Minas
city_br_187,base.br,base.state_br_ba,Jequié
city_br_188,base.br,base.state_br_sp,Atibaia
city_br_189,base.br,base.state_br_sp,Itapecerica da Serra
city_br_190,base.br,base.state_br_pa,Abaetetuba
city_br_191,base.br,base.state_br_sp,Itapetininga
city_br_192,base.br,base.state_br_ma,Caxias
city_br_193,base.br,base.state_br_rj,Rio das Ostras
city_br_194,base.br,base.state_br_go,Senador Canedo
city_br_195,base.br,base.state_br_sp,Santana de Parnaíba
city_br_196,base.br,base.state_br_sp,Mogi Guaçu
city_br_197,base.br,base.state_br_mg,Pouso Alegre
city_br_198,base.br,base.state_br_pr,Araucária
city_br_199,base.br,base.state_br_ba,Alagoinhas
city_br_200,base.br,base.state_br_pr,Toledo
city_br_201,base.br,base.state_br_pb,Santa Rita
city_br_202,base.br,base.state_br_pr,Fazenda Rio Grande
city_br_203,base.br,base.state_br_pe,Camaragibe
city_br_204,base.br,base.state_br_rj,Nilópolis
city_br_205,base.br,base.state_br_pr,Paranaguá
city_br_206,base.br,base.state_br_ma,Paço do Lumiar
city_br_207,base.br,base.state_br_ba,Teixeira de Freitas
city_br_208,base.br,base.state_br_sp,Botucatu
city_br_209,base.br,base.state_br_sp,Franco da Rocha
city_br_210,base.br,base.state_br_pe,Garanhuns
city_br_211,base.br,base.state_br_go,Trindade
city_br_212,base.br,base.state_br_sc,Brusque
city_br_213,base.br,base.state_br_rj,Queimados
city_br_214,base.br,base.state_br_sc,Balneário Camboriú
city_br_215,base.br,base.state_br_mg,Teófilo Otoni
city_br_216,base.br,base.state_br_mg,Varginha
city_br_217,base.br,base.state_br_pr,Campo Largo
city_br_218,base.br,base.state_br_rs,Cachoeirinha
city_br_219,base.br,base.state_br_sp,Caraguatatuba
city_br_220,base.br,base.state_br_sp,Salto
city_br_221,base.br,base.state_br_pa,Cametá
city_br_222,base.br,base.state_br_pe,Vitória de Santo Antão
city_br_223,base.br,base.state_br_sp,Jaú
city_br_224,base.br,base.state_br_rs,Santa Cruz do Sul
city_br_225,base.br,base.state_br_ms,Três Lagoas
city_br_226,base.br,base.state_br_rs,Sapucaia do Sul
city_br_227,base.br,base.state_br_mg,Conselheiro Lafaiete
city_br_228,base.br,base.state_br_ce,Itapipoca
city_br_229,base.br,base.state_br_ce,Crato
city_br_230,base.br,base.state_br_sp,Araras
city_br_231,base.br,base.state_br_pr,Apucarana
city_br_232,base.br,base.state_br_rj,Araruama
city_br_233,base.br,base.state_br_rj,Resende
city_br_234,base.br,base.state_br_mg,Sabará
city_br_235,base.br,base.state_br_mg,Vespasiano
city_br_236,base.br,base.state_br_sp,Votorantim
city_br_237,base.br,base.state_br_pr,Pinhais
city_br_238,base.br,base.state_br_sp,Sertãozinho
city_br_239,base.br,base.state_br_pa,Barcarena
city_br_240,base.br,base.state_br_sp,Valinhos
city_br_241,base.br,base.state_br_pa,Altamira
city_br_242,base.br,base.state_br_mg,Barbacena
city_br_243,base.br,base.state_br_es,Guarapari
city_br_244,base.br,base.state_br_ro,Ji-Paraná
city_br_245,base.br,base.state_br_sp,Tatuí
city_br_246,base.br,base.state_br_es,São Mateus
city_br_247,base.br,base.state_br_pa,Itaituba
city_br_248,base.br,base.state_br_rs,Bento Gonçalves
city_br_249,base.br,base.state_br_pa,Bragança
city_br_250,base.br,base.state_br_sp,Barretos
city_br_251,base.br,base.state_br_sp,Itatiba
city_br_252,base.br,base.state_br_es,Colatina
city_br_253,base.br,base.state_br_pr,Almirante Tamandaré
city_br_254,base.br,base.state_br_pr,Arapongas
city_br_255,base.br,base.state_br_sp,Birigui
city_br_256,base.br,base.state_br_pr,Piraquara
city_br_257,base.br,base.state_br_pr,Sarandi
city_br_258,base.br,base.state_br_sp,Jandira
city_br_259,base.br,base.state_br_sp,Guaratinguetá
city_br_260,base.br,base.state_br_rs,Bagé
city_br_261,base.br,base.state_br_mg,Araguari
city_br_262,base.br,base.state_br_rs,Uruguaiana
city_br_263,base.br,base.state_br_pr,Umuarama
city_br_264,base.br,base.state_br_rj,Itaguaí
city_br_265,base.br,base.state_br_rn,São Gonçalo do Amarante
city_br_266,base.br,base.state_br_sp,Catanduva
city_br_267,base.br,base.state_br_sp,Várzea Paulista
city_br_268,base.br,base.state_br_go,Formosa
city_br_269,base.br,base.state_br_sp,Ribeirão Pires
city_br_270,base.br,base.state_br_pe,Igarassu
city_br_271,base.br,base.state_br_ba,Simões Filho
city_br_272,base.br,base.state_br_go,Catalão
city_br_273,base.br,base.state_br_ma,Codó
city_br_274,base.br,base.state_br_ba,Eunápolis
city_br_275,base.br,base.state_br_mg,Itabira
city_br_276,base.br,base.state_br_ba,Paulo Afonso
city_br_277,base.br,base.state_br_sp,Itanhaém
city_br_278,base.br,base.state_br_sp,Cubatão
city_br_279,base.br,base.state_br_mg,Passos
city_br_280,base.br,base.state_br_mg,Nova Lima
city_br_281,base.br,base.state_br_mg,Araxá
city_br_282,base.br,base.state_br_pe,São Lourenço da Mata
city_br_283,base.br,base.state_br_mt,Sorriso
city_br_284,base.br,base.state_br_sp,Paulínia
city_br_285,base.br,base.state_br_pa,Marituba
city_br_286,base.br,base.state_br_sc,Tubarão
city_br_287,base.br,base.state_br_go,Itumbiara
city_br_288,base.br,base.state_br_ba,Luís Eduardo Magalhães
city_br_289,base.br,base.state_br_ap,Santana
city_br_290,base.br,base.state_br_pr,Cambé
city_br_291,base.br,base.state_br_pa,Breves
city_br_292,base.br,base.state_br_ma,Açailândia
city_br_293,base.br,base.state_br_mt,Tangará da Serra
city_br_294,base.br,base.state_br_go,Jataí
city_br_295,base.br,base.state_br_rs,Erechim
city_br_296,base.br,base.state_br_mg,Nova Serrana
city_br_297,base.br,base.state_br_pa,Paragominas
city_br_298,base.br,base.state_br_ce,Maranguape
city_br_299,base.br,base.state_br_go,Planaltina
city_br_300,base.br,base.state_br_mg,Lavras
city_br_301,base.br,base.state_br_mg,Coronel Fabriciano
city_br_302,base.br,base.state_br_mg,Muriaé
city_br_303,base.br,base.state_br_rj,São Pedro da Aldeia
city_br_304,base.br,base.state_br_sp,Ourinhos
city_br_305,base.br,base.state_br_go,Novo Gama
city_br_306,base.br,base.state_br_sp,Poá
city_br_307,base.br,base.state_br_ma,Bacabal
city_br_308,base.br,base.state_br_am,Itacoatiara
city_br_309,base.br,base.state_br_se,Itabaiana
city_br_310,base.br,base.state_br_mg,Ubá
city_br_311,base.br,base.state_br_pb,Patos
city_br_312,base.br,base.state_br_sc,Camboriú
city_br_313,base.br,base.state_br_ba,Santo Antônio de Jesus
city_br_314,base.br,base.state_br_mg,Ituiutaba
city_br_315,base.br,base.state_br_am,Manacapuru
city_br_316,base.br,base.state_br_ma,Balsas
city_br_317,base.br,base.state_br_se,Lagarto
city_br_318,base.br,base.state_br_sp,Assis
city_br_319,base.br,base.state_br_rj,Itaperuna
city_br_320,base.br,base.state_br_go,Abadia de Goiás
city_br_321,base.br,base.state_br_mg,Abadia dos Dourados
city_br_322,base.br,base.state_br_go,Abadiânia
city_br_323,base.br,base.state_br_mg,Abaeté
city_br_324,base.br,base.state_br_ce,Abaiara
city_br_325,base.br,base.state_br_ba,Abaíra
city_br_326,base.br,base.state_br_ba,Abaré
city_br_327,base.br,base.state_br_pr,Abatiá
city_br_328,base.br,base.state_br_sc,Abdon Batista
city_br_329,base.br,base.state_br_pa,Abel Figueiredo
city_br_330,base.br,base.state_br_sc,Abelardo Luz
city_br_331,base.br,base.state_br_mg,Abre Campo
city_br_332,base.br,base.state_br_pe,Abreu e Lima
city_br_333,base.br,base.state_br_to,Abreulândia
city_br_334,base.br,base.state_br_mg,Acaiaca
city_br_335,base.br,base.state_br_ba,Acajutiba
city_br_336,base.br,base.state_br_pa,Acará
city_br_337,base.br,base.state_br_ce,Acarape
city_br_338,base.br,base.state_br_ce,Acaraú
city_br_339,base.br,base.state_br_rn,Acari
city_br_340,base.br,base.state_br_pi,Acauã
city_br_341,base.br,base.state_br_rs,Aceguá
city_br_342,base.br,base.state_br_ce,Acopiara
city_br_343,base.br,base.state_br_mt,Acorizal
city_br_344,base.br,base.state_br_ac,Acrelândia
city_br_345,base.br,base.state_br_go,Acreúna
city_br_346,base.br,base.state_br_rn,Açu
city_br_347,base.br,base.state_br_mg,Açucena
city_br_348,base.br,base.state_br_sp,Adamantina
city_br_349,base.br,base.state_br_go,Adelândia
city_br_350,base.br,base.state_br_sp,Adolfo
city_br_351,base.br,base.state_br_pr,Adrianópolis
city_br_352,base.br,base.state_br_ba,Adustina
city_br_353,base.br,base.state_br_pe,Afogados da Ingazeira
city_br_354,base.br,base.state_br_rn,Afonso Bezerra
city_br_355,base.br,base.state_br_es,Afonso Cláudio
city_br_356,base.br,base.state_br_ma,Afonso Cunha
city_br_357,base.br,base.state_br_pe,Afrânio
city_br_358,base.br,base.state_br_pa,Afuá
city_br_359,base.br,base.state_br_pe,Agrestina
city_br_360,base.br,base.state_br_pi,Agricolândia
city_br_361,base.br,base.state_br_sc,Agrolândia
city_br_362,base.br,base.state_br_sc,Agronômica
city_br_363,base.br,base.state_br_pa,Água Azul do Norte
city_br_364,base.br,base.state_br_mg,Água Boa
city_br_365,base.br,base.state_br_mt,Água Boa
city_br_366,base.br,base.state_br_al,Água Branca
city_br_367,base.br,base.state_br_pb,Água Branca
city_br_368,base.br,base.state_br_pi,Água Branca
city_br_369,base.br,base.state_br_ms,Água Clara
city_br_370,base.br,base.state_br_mg,Água Comprida
city_br_371,base.br,base.state_br_sc,Água Doce
city_br_372,base.br,base.state_br_ma,Água Doce do Maranhão
city_br_373,base.br,base.state_br_es,Água Doce do Norte
city_br_374,base.br,base.state_br_ba,Água Fria
city_br_375,base.br,base.state_br_go,Água Fria de Goiás
city_br_376,base.br,base.state_br_go,Água Limpa
city_br_377,base.br,base.state_br_rn,Água Nova
city_br_378,base.br,base.state_br_pe,Água Preta
city_br_379,base.br,base.state_br_rs,Água Santa
city_br_380,base.br,base.state_br_sp,Aguaí
city_br_381,base.br,base.state_br_mg,Aguanil
city_br_382,base.br,base.state_br_pe,Águas Belas
city_br_383,base.br,base.state_br_sp,Águas da Prata
city_br_384,base.br,base.state_br_sc,Águas de Chapecó
city_br_385,base.br,base.state_br_sp,Águas de Lindóia
city_br_386,base.br,base.state_br_sp,Águas de Santa Bárbara
city_br_387,base.br,base.state_br_sp,Águas de São Pedro
city_br_388,base.br,base.state_br_mg,Águas Formosas
city_br_389,base.br,base.state_br_sc,Águas Frias
city_br_390,base.br,base.state_br_sc,Águas Mornas
city_br_391,base.br,base.state_br_mg,Águas Vermelhas
city_br_392,base.br,base.state_br_rs,Agudo
city_br_393,base.br,base.state_br_sp,Agudos
city_br_394,base.br,base.state_br_pr,Agudos do Sul
city_br_395,base.br,base.state_br_es,Águia Branca
city_br_396,base.br,base.state_br_pb,Aguiar
city_br_397,base.br,base.state_br_to,Aguiarnópolis
city_br_398,base.br,base.state_br_mg,Aimorés
city_br_399,base.br,base.state_br_ba,Aiquara
city_br_400,base.br,base.state_br_ce,Aiuaba
city_br_401,base.br,base.state_br_mg,Aiuruoca
city_br_402,base.br,base.state_br_rs,Ajuricaba
city_br_403,base.br,base.state_br_mg,Alagoa
city_br_404,base.br,base.state_br_pb,Alagoa Grande
city_br_405,base.br,base.state_br_pb,Alagoa Nova
city_br_406,base.br,base.state_br_pb,Alagoinha
city_br_407,base.br,base.state_br_pe,Alagoinha
city_br_408,base.br,base.state_br_pi,Alagoinha do Piauí
city_br_409,base.br,base.state_br_sp,Alambari
city_br_410,base.br,base.state_br_mg,Albertina
city_br_411,base.br,base.state_br_ma,Alcântara
city_br_412,base.br,base.state_br_ce,Alcântaras
city_br_413,base.br,base.state_br_pb,Alcantil
city_br_414,base.br,base.state_br_ms,Alcinópolis
city_br_415,base.br,base.state_br_ba,Alcobaça
city_br_416,base.br,base.state_br_ma,Aldeias Altas
city_br_417,base.br,base.state_br_rs,Alecrim
city_br_418,base.br,base.state_br_es,Alegre
city_br_419,base.br,base.state_br_rs,Alegrete
city_br_420,base.br,base.state_br_pi,Alegrete do Piauí
city_br_421,base.br,base.state_br_rs,Alegria
city_br_422,base.br,base.state_br_mg,Além Paraíba
city_br_423,base.br,base.state_br_pa,Alenquer
city_br_424,base.br,base.state_br_rn,Alexandria
city_br_425,base.br,base.state_br_go,Alexânia
city_br_426,base.br,base.state_br_mg,Alfenas
city_br_427,base.br,base.state_br_es,Alfredo Chaves
city_br_428,base.br,base.state_br_sp,Alfredo Marcondes
city_br_429,base.br,base.state_br_mg,Alfredo Vasconcelos
city_br_430,base.br,base.state_br_sc,Alfredo Wagner
city_br_431,base.br,base.state_br_pb,Algodão de Jandaíra
city_br_432,base.br,base.state_br_pb,Alhandra
city_br_433,base.br,base.state_br_pe,Aliança
city_br_434,base.br,base.state_br_to,Aliança do Tocantins
city_br_435,base.br,base.state_br_ba,Almadina
city_br_436,base.br,base.state_br_to,Almas
city_br_437,base.br,base.state_br_pa,Almeirim
city_br_438,base.br,base.state_br_mg,Almenara
city_br_439,base.br,base.state_br_rn,Almino Afonso
city_br_440,base.br,base.state_br_rs,Almirante Tamandaré do Sul
city_br_441,base.br,base.state_br_go,Aloândia
city_br_442,base.br,base.state_br_mg,Alpercata
city_br_443,base.br,base.state_br_rs,Alpestre
city_br_444,base.br,base.state_br_mg,Alpinópolis
city_br_445,base.br,base.state_br_mt,Alta Floresta
city_br_446,base.br,base.state_br_ro,Alta Floresta D'Oeste
city_br_447,base.br,base.state_br_sp,Altair
city_br_448,base.br,base.state_br_ma,Altamira do Maranhão
city_br_449,base.br,base.state_br_pr,Altamira do Paraná
city_br_450,base.br,base.state_br_ce,Altaneira
city_br_451,base.br,base.state_br_mg,Alterosa
city_br_452,base.br,base.state_br_pe,Altinho
city_br_453,base.br,base.state_br_sp,Altinópolis
city_br_454,base.br,base.state_br_rr,Alto Alegre
city_br_455,base.br,base.state_br_rs,Alto Alegre
city_br_456,base.br,base.state_br_sp,Alto Alegre
city_br_457,base.br,base.state_br_ma,Alto Alegre do Maranhão
city_br_458,base.br,base.state_br_ma,Alto Alegre do Pindaré
city_br_459,base.br,base.state_br_ro,Alto Alegre dos Parecis
city_br_460,base.br,base.state_br_mt,Alto Araguaia
city_br_461,base.br,base.state_br_sc,Alto Bela Vista
city_br_462,base.br,base.state_br_mt,Alto Boa Vista
city_br_463,base.br,base.state_br_mg,Alto Caparaó
city_br_464,base.br,base.state_br_rn,Alto do Rodrigues
city_br_465,base.br,base.state_br_rs,Alto Feliz
city_br_466,base.br,base.state_br_mt,Alto Garças
city_br_467,base.br,base.state_br_go,Alto Horizonte
city_br_468,base.br,base.state_br_mg,Alto Jequitibá
city_br_469,base.br,base.state_br_pi,Alto Longá
city_br_470,base.br,base.state_br_mt,Alto Paraguai
city_br_471,base.br,base.state_br_pr,Alto Paraíso
city_br_472,base.br,base.state_br_ro,Alto Paraíso
city_br_473,base.br,base.state_br_go,Alto Paraíso de Goiás
city_br_474,base.br,base.state_br_pr,Alto Paraná
city_br_475,base.br,base.state_br_ma,Alto Parnaíba
city_br_476,base.br,base.state_br_pr,Alto Piquiri
city_br_477,base.br,base.state_br_mg,Alto Rio Doce
city_br_478,base.br,base.state_br_es,Alto Rio Novo
city_br_479,base.br,base.state_br_ce,Alto Santo
city_br_480,base.br,base.state_br_mt,Alto Taquari
city_br_481,base.br,base.state_br_pr,Altônia
city_br_482,base.br,base.state_br_pi,Altos
city_br_483,base.br,base.state_br_sp,Alumínio
city_br_484,base.br,base.state_br_am,Alvarães
city_br_485,base.br,base.state_br_mg,Alvarenga
city_br_486,base.br,base.state_br_sp,Álvares Florence
city_br_487,base.br,base.state_br_sp,Álvares Machado
city_br_488,base.br,base.state_br_sp,Álvaro de Carvalho
city_br_489,base.br,base.state_br_sp,Alvinlândia
city_br_490,base.br,base.state_br_mg,Alvinópolis
city_br_491,base.br,base.state_br_to,Alvorada
city_br_492,base.br,base.state_br_ro,Alvorada D'Oeste
city_br_493,base.br,base.state_br_mg,Alvorada de Minas
city_br_494,base.br,base.state_br_pi,Alvorada do Gurguéia
city_br_495,base.br,base.state_br_go,Alvorada do Norte
city_br_496,base.br,base.state_br_pr,Alvorada do Sul
city_br_497,base.br,base.state_br_rr,Amajari
city_br_498,base.br,base.state_br_ms,Amambai
city_br_499,base.br,base.state_br_ap,Amapá
city_br_500,base.br,base.state_br_ma,Amapá do Maranhão
city_br_501,base.br,base.state_br_pr,Amaporã
city_br_502,base.br,base.state_br_pe,Amaraji
city_br_503,base.br,base.state_br_rs,Amaral Ferrador
city_br_504,base.br,base.state_br_go,Amaralina
city_br_505,base.br,base.state_br_pi,Amarante
city_br_506,base.br,base.state_br_ma,Amarante do Maranhão
city_br_507,base.br,base.state_br_ba,Amargosa
city_br_508,base.br,base.state_br_am,Amaturá
city_br_509,base.br,base.state_br_ba,Amélia Rodrigues
city_br_510,base.br,base.state_br_ba,América Dourada
city_br_511,base.br,base.state_br_go,Americano do Brasil
city_br_512,base.br,base.state_br_sp,Américo Brasiliense
city_br_513,base.br,base.state_br_sp,Américo de Campos
city_br_514,base.br,base.state_br_rs,Ametista do Sul
city_br_515,base.br,base.state_br_ce,Amontada
city_br_516,base.br,base.state_br_go,Amorinópolis
city_br_517,base.br,base.state_br_pb,Amparo
city_br_518,base.br,base.state_br_sp,Amparo
city_br_519,base.br,base.state_br_se,Amparo de São Francisco
city_br_520,base.br,base.state_br_mg,Amparo do Serra
city_br_521,base.br,base.state_br_pr,Ampére
city_br_522,base.br,base.state_br_al,Anadia
city_br_523,base.br,base.state_br_ba,Anagé
city_br_524,base.br,base.state_br_pr,Anahy
city_br_525,base.br,base.state_br_pa,Anajás
city_br_526,base.br,base.state_br_ma,Anajatuba
city_br_527,base.br,base.state_br_sp,Analândia
city_br_528,base.br,base.state_br_am,Anamã
city_br_529,base.br,base.state_br_to,Ananás
city_br_530,base.br,base.state_br_pa,Anapu
city_br_531,base.br,base.state_br_ma,Anapurus
city_br_532,base.br,base.state_br_ms,Anastácio
city_br_533,base.br,base.state_br_ms,Anaurilândia
city_br_534,base.br,base.state_br_es,Anchieta
city_br_535,base.br,base.state_br_sc,Anchieta
city_br_536,base.br,base.state_br_ba,Andaraí
city_br_537,base.br,base.state_br_pr,Andirá
city_br_538,base.br,base.state_br_ba,Andorinha
city_br_539,base.br,base.state_br_mg,Andradas
city_br_540,base.br,base.state_br_sp,Andradina
city_br_541,base.br,base.state_br_rs,André da Rocha
city_br_542,base.br,base.state_br_mg,Andrelândia
city_br_543,base.br,base.state_br_sp,Angatuba
city_br_544,base.br,base.state_br_mg,Angelândia
city_br_545,base.br,base.state_br_ms,Angélica
city_br_546,base.br,base.state_br_pe,Angelim
city_br_547,base.br,base.state_br_sc,Angelina
city_br_548,base.br,base.state_br_ba,Angical
city_br_549,base.br,base.state_br_pi,Angical do Piauí
city_br_550,base.br,base.state_br_to,Angico
city_br_551,base.br,base.state_br_rn,Angicos
city_br_552,base.br,base.state_br_ba,Anguera
city_br_553,base.br,base.state_br_pr,Ângulo
city_br_554,base.br,base.state_br_go,Anhanguera
city_br_555,base.br,base.state_br_sp,Anhembi
city_br_556,base.br,base.state_br_sp,Anhumas
city_br_557,base.br,base.state_br_go,Anicuns
city_br_558,base.br,base.state_br_pi,Anísio de Abreu
city_br_559,base.br,base.state_br_sc,Anita Garibaldi
city_br_560,base.br,base.state_br_sc,Anitápolis
city_br_561,base.br,base.state_br_am,Anori
city_br_562,base.br,base.state_br_rs,Anta Gorda
city_br_563,base.br,base.state_br_ba,Antas
city_br_564,base.br,base.state_br_pr,Antonina
city_br_565,base.br,base.state_br_ce,Antonina do Norte
city_br_566,base.br,base.state_br_pi,Antônio Almeida
city_br_567,base.br,base.state_br_ba,Antônio Cardoso
city_br_568,base.br,base.state_br_mg,Antônio Carlos
city_br_569,base.br,base.state_br_sc,Antônio Carlos
city_br_570,base.br,base.state_br_mg,Antônio Dias
city_br_571,base.br,base.state_br_ba,Antônio Gonçalves
city_br_572,base.br,base.state_br_ms,Antônio João
city_br_573,base.br,base.state_br_rn,Antônio Martins
city_br_574,base.br,base.state_br_pr,Antônio Olinto
city_br_575,base.br,base.state_br_rs,Antônio Prado
city_br_576,base.br,base.state_br_mg,Antônio Prado de Minas
city_br_577,base.br,base.state_br_pb,Aparecida
city_br_578,base.br,base.state_br_sp,Aparecida
city_br_579,base.br,base.state_br_sp,Aparecida d'Oeste
city_br_580,base.br,base.state_br_go,Aparecida do Rio Doce
city_br_581,base.br,base.state_br_to,Aparecida do Rio Negro
city_br_582,base.br,base.state_br_ms,Aparecida do Taboado
city_br_583,base.br,base.state_br_rj,Aperibé
city_br_584,base.br,base.state_br_es,Apiacá
city_br_585,base.br,base.state_br_mt,Apiacás
city_br_586,base.br,base.state_br_sp,Apiaí
city_br_587,base.br,base.state_br_ma,Apicum-Açu
city_br_588,base.br,base.state_br_sc,Apiúna
city_br_589,base.br,base.state_br_rn,Apodi
city_br_590,base.br,base.state_br_ba,Aporá
city_br_591,base.br,base.state_br_go,Aporé
city_br_592,base.br,base.state_br_ba,Apuarema
city_br_593,base.br,base.state_br_am,Apuí
city_br_594,base.br,base.state_br_ce,Apuiarés
city_br_595,base.br,base.state_br_se,Aquidabã
city_br_596,base.br,base.state_br_ms,Aquidauana
city_br_597,base.br,base.state_br_ce,Aquiraz
city_br_598,base.br,base.state_br_sc,Arabutã
city_br_599,base.br,base.state_br_pb,Araçagi
city_br_600,base.br,base.state_br_mg,Araçaí
city_br_601,base.br,base.state_br_sp,Araçariguama
city_br_602,base.br,base.state_br_ba,Araçás
city_br_603,base.br,base.state_br_ce,Aracati
city_br_604,base.br,base.state_br_ba,Aracatu
city_br_605,base.br,base.state_br_ba,Araci
city_br_606,base.br,base.state_br_mg,Aracitaba
city_br_607,base.br,base.state_br_ce,Aracoiaba
city_br_608,base.br,base.state_br_pe,Araçoiaba
city_br_609,base.br,base.state_br_sp,Araçoiaba da Serra
city_br_610,base.br,base.state_br_es,Aracruz
city_br_611,base.br,base.state_br_go,Araçu
city_br_612,base.br,base.state_br_mg,Araçuaí
city_br_613,base.br,base.state_br_go,Aragarças
city_br_614,base.br,base.state_br_go,Aragoiânia
city_br_615,base.br,base.state_br_to,Aragominas
city_br_616,base.br,base.state_br_to,Araguacema
city_br_617,base.br,base.state_br_to,Araguaçu
city_br_618,base.br,base.state_br_mt,Araguaiana
city_br_619,base.br,base.state_br_mt,Araguainha
city_br_620,base.br,base.state_br_ma,Araguanã
city_br_621,base.br,base.state_br_to,Araguanã
city_br_622,base.br,base.state_br_go,Araguapaz
city_br_623,base.br,base.state_br_to,Araguatins
city_br_624,base.br,base.state_br_ma,Araioses
city_br_625,base.br,base.state_br_ms,Aral Moreira
city_br_626,base.br,base.state_br_ba,Aramari
city_br_627,base.br,base.state_br_rs,Arambaré
city_br_628,base.br,base.state_br_ma,Arame
city_br_629,base.br,base.state_br_sp,Aramina
city_br_630,base.br,base.state_br_sp,Arandu
city_br_631,base.br,base.state_br_mg,Arantina
city_br_632,base.br,base.state_br_sp,Arapeí
city_br_633,base.br,base.state_br_to,Arapoema
city_br_634,base.br,base.state_br_mg,Araponga
city_br_635,base.br,base.state_br_mg,Araporã
city_br_636,base.br,base.state_br_pr,Arapoti
city_br_637,base.br,base.state_br_mg,Arapuá
city_br_638,base.br,base.state_br_pr,Arapuã
city_br_639,base.br,base.state_br_mt,Araputanga
city_br_640,base.br,base.state_br_sc,Araquari
city_br_641,base.br,base.state_br_pb,Arara
city_br_642,base.br,base.state_br_sc,Araranguá
city_br_643,base.br,base.state_br_ce,Ararendá
city_br_644,base.br,base.state_br_ma,Arari
city_br_645,base.br,base.state_br_rs,Araricá
city_br_646,base.br,base.state_br_ce,Araripe
city_br_647,base.br,base.state_br_pe,Araripina
city_br_648,base.br,base.state_br_pb,Araruna
city_br_649,base.br,base.state_br_pr,Araruna
city_br_650,base.br,base.state_br_ba,Arataca
city_br_651,base.br,base.state_br_rs,Aratiba
city_br_652,base.br,base.state_br_ce,Aratuba
city_br_653,base.br,base.state_br_ba,Aratuípe
city_br_654,base.br,base.state_br_se,Arauá
city_br_655,base.br,base.state_br_mg,Araújos
city_br_656,base.br,base.state_br_mg,Arceburgo
city_br_657,base.br,base.state_br_sp,Arco-Íris
city_br_658,base.br,base.state_br_mg,Arcos
city_br_659,base.br,base.state_br_pe,Arcoverde
city_br_660,base.br,base.state_br_mg,Areado
city_br_661,base.br,base.state_br_rj,Areal
city_br_662,base.br,base.state_br_sp,Arealva
city_br_663,base.br,base.state_br_pb,Areia
city_br_664,base.br,base.state_br_rn,Areia Branca
city_br_665,base.br,base.state_br_se,Areia Branca
city_br_666,base.br,base.state_br_pb,Areia de Baraúnas
city_br_667,base.br,base.state_br_pb,Areial
city_br_668,base.br,base.state_br_sp,Areias
city_br_669,base.br,base.state_br_sp,Areiópolis
city_br_670,base.br,base.state_br_mt,Arenápolis
city_br_671,base.br,base.state_br_go,Arenópolis
city_br_672,base.br,base.state_br_rn,Arês
city_br_673,base.br,base.state_br_mg,Argirita
city_br_674,base.br,base.state_br_mg,Aricanduva
city_br_675,base.br,base.state_br_mg,Arinos
city_br_676,base.br,base.state_br_mt,Aripuanã
city_br_677,base.br,base.state_br_ro,Ariquemes
city_br_678,base.br,base.state_br_sp,Ariranha
city_br_679,base.br,base.state_br_pr,Ariranha do Ivaí
city_br_680,base.br,base.state_br_rj,Armação dos Búzios
city_br_681,base.br,base.state_br_sc,Armazém
city_br_682,base.br,base.state_br_ce,Arneiroz
city_br_683,base.br,base.state_br_pi,Aroazes
city_br_684,base.br,base.state_br_pb,Aroeiras
city_br_685,base.br,base.state_br_pi,Aroeiras do Itaim
city_br_686,base.br,base.state_br_pi,Arraial
city_br_687,base.br,base.state_br_rj,Arraial do Cabo
city_br_688,base.br,base.state_br_to,Arraias
city_br_689,base.br,base.state_br_rs,Arroio do Meio
city_br_690,base.br,base.state_br_rs,Arroio do Padre
city_br_691,base.br,base.state_br_rs,Arroio do Sal
city_br_692,base.br,base.state_br_rs,Arroio do Tigre
city_br_693,base.br,base.state_br_rs,Arroio dos Ratos
city_br_694,base.br,base.state_br_rs,Arroio Grande
city_br_695,base.br,base.state_br_sc,Arroio Trinta
city_br_696,base.br,base.state_br_sp,Artur Nogueira
city_br_697,base.br,base.state_br_go,Aruanã
city_br_698,base.br,base.state_br_sp,Arujá
city_br_699,base.br,base.state_br_sc,Arvoredo
city_br_700,base.br,base.state_br_rs,Arvorezinha
city_br_701,base.br,base.state_br_sc,Ascurra
city_br_702,base.br,base.state_br_sp,Aspásia
city_br_703,base.br,base.state_br_pr,Assaí
city_br_704,base.br,base.state_br_ce,Assaré
city_br_705,base.br,base.state_br_ac,Assis Brasil
city_br_706,base.br,base.state_br_pr,Assis Chateaubriand
city_br_707,base.br,base.state_br_pb,Assunção
city_br_708,base.br,base.state_br_pi,Assunção do Piauí
city_br_709,base.br,base.state_br_mg,Astolfo Dutra
city_br_710,base.br,base.state_br_pr,Astorga
city_br_711,base.br,base.state_br_al,Atalaia
city_br_712,base.br,base.state_br_pr,Atalaia
city_br_713,base.br,base.state_br_am,Atalaia do Norte
city_br_714,base.br,base.state_br_sc,Atalanta
city_br_715,base.br,base.state_br_mg,Ataléia
city_br_716,base.br,base.state_br_es,Atílio Vivacqua
city_br_717,base.br,base.state_br_to,Augustinópolis
city_br_718,base.br,base.state_br_pa,Augusto Corrêa
city_br_719,base.br,base.state_br_mg,Augusto de Lima
city_br_720,base.br,base.state_br_rs,Augusto Pestana
city_br_721,base.br,base.state_br_rs,Áurea
city_br_722,base.br,base.state_br_ba,Aurelino Leal
city_br_723,base.br,base.state_br_sp,Auriflama
city_br_724,base.br,base.state_br_go,Aurilândia
city_br_725,base.br,base.state_br_ce,Aurora
city_br_726,base.br,base.state_br_sc,Aurora
city_br_727,base.br,base.state_br_pa,Aurora do Pará
city_br_728,base.br,base.state_br_to,Aurora do Tocantins
city_br_729,base.br,base.state_br_am,Autazes
city_br_730,base.br,base.state_br_sp,Avaí
city_br_731,base.br,base.state_br_sp,Avanhandava
city_br_732,base.br,base.state_br_sp,Avaré
city_br_733,base.br,base.state_br_pa,Aveiro
city_br_734,base.br,base.state_br_pi,Avelino Lopes
city_br_735,base.br,base.state_br_go,Avelinópolis
city_br_736,base.br,base.state_br_ma,Axixá
city_br_737,base.br,base.state_br_to,Axixá do Tocantins
city_br_738,base.br,base.state_br_to,Babaçulândia
city_br_739,base.br,base.state_br_ma,Bacabeira
city_br_740,base.br,base.state_br_ma,Bacuri
city_br_741,base.br,base.state_br_ma,Bacurituba
city_br_742,base.br,base.state_br_sp,Bady Bassitt
city_br_743,base.br,base.state_br_mg,Baependi
city_br_744,base.br,base.state_br_pa,Bagre
city_br_745,base.br,base.state_br_pb,Baía da Traição
city_br_746,base.br,base.state_br_rn,Baía Formosa
city_br_747,base.br,base.state_br_ba,Baianópolis
city_br_748,base.br,base.state_br_pa,Baião
city_br_749,base.br,base.state_br_ba,Baixa Grande
city_br_750,base.br,base.state_br_pi,Baixa Grande do Ribeiro
city_br_751,base.br,base.state_br_ce,Baixio
city_br_752,base.br,base.state_br_es,Baixo Guandu
city_br_753,base.br,base.state_br_sp,Balbinos
city_br_754,base.br,base.state_br_mg,Baldim
city_br_755,base.br,base.state_br_go,Baliza
city_br_756,base.br,base.state_br_sc,Balneário Arroio do Silva
city_br_757,base.br,base.state_br_sc,Balneário Barra do Sul
city_br_758,base.br,base.state_br_sc,Balneário Gaivota
city_br_759,base.br,base.state_br_sc,Balneário Piçarras
city_br_760,base.br,base.state_br_rs,Balneário Pinhal
city_br_761,base.br,base.state_br_sc,Balneário Rincão
city_br_762,base.br,base.state_br_pr,Balsa Nova
city_br_763,base.br,base.state_br_sp,Bálsamo
city_br_764,base.br,base.state_br_mg,Bambuí
city_br_765,base.br,base.state_br_ce,Banabuiú
city_br_766,base.br,base.state_br_sp,Bananal
city_br_767,base.br,base.state_br_pb,Bananeiras
city_br_768,base.br,base.state_br_mg,Bandeira
city_br_769,base.br,base.state_br_mg,Bandeira do Sul
city_br_770,base.br,base.state_br_sc,Bandeirante
city_br_771,base.br,base.state_br_ms,Bandeirantes
city_br_772,base.br,base.state_br_pr,Bandeirantes
city_br_773,base.br,base.state_br_to,Bandeirantes do Tocantins
city_br_774,base.br,base.state_br_pa,Bannach
city_br_775,base.br,base.state_br_ba,Banzaê
city_br_776,base.br,base.state_br_rs,Barão
city_br_777,base.br,base.state_br_sp,Barão de Antonina
city_br_778,base.br,base.state_br_mg,Barão de Cocais
city_br_779,base.br,base.state_br_rs,Barão de Cotegipe
city_br_780,base.br,base.state_br_ma,Barão de Grajaú
city_br_781,base.br,base.state_br_mt,Barão de Melgaço
city_br_782,base.br,base.state_br_mg,Barão de Monte Alto
city_br_783,base.br,base.state_br_rs,Barão do Triunfo
city_br_784,base.br,base.state_br_pb,Baraúna
city_br_785,base.br,base.state_br_rn,Baraúna
city_br_786,base.br,base.state_br_ce,Barbalha
city_br_787,base.br,base.state_br_sp,Barbosa
city_br_788,base.br,base.state_br_pr,Barbosa Ferraz
city_br_789,base.br,base.state_br_rn,Barcelona
city_br_790,base.br,base.state_br_am,Barcelos
city_br_791,base.br,base.state_br_sp,Bariri
city_br_792,base.br,base.state_br_ba,Barra
city_br_793,base.br,base.state_br_sc,Barra Bonita
city_br_794,base.br,base.state_br_sp,Barra Bonita
city_br_795,base.br,base.state_br_pi,Barra D'Alcântara
city_br_796,base.br,base.state_br_ba,Barra da Estiva
city_br_797,base.br,base.state_br_pe,Barra de Guabiraba
city_br_798,base.br,base.state_br_pb,Barra de Santa Rosa
city_br_799,base.br,base.state_br_pb,Barra de Santana
city_br_800,base.br,base.state_br_al,Barra de Santo Antônio
city_br_801,base.br,base.state_br_es,Barra de São Francisco
city_br_802,base.br,base.state_br_al,Barra de São Miguel
city_br_803,base.br,base.state_br_pb,Barra de São Miguel
city_br_804,base.br,base.state_br_mt,Barra do Bugres
city_br_805,base.br,base.state_br_sp,Barra do Chapéu
city_br_806,base.br,base.state_br_ba,Barra do Choça
city_br_807,base.br,base.state_br_ma,Barra do Corda
city_br_808,base.br,base.state_br_mt,Barra do Garças
city_br_809,base.br,base.state_br_rs,Barra do Guarita
city_br_810,base.br,base.state_br_pr,Barra do Jacaré
city_br_811,base.br,base.state_br_ba,Barra do Mendes
city_br_812,base.br,base.state_br_to,Barra do Ouro
city_br_813,base.br,base.state_br_rj,Barra do Piraí
city_br_814,base.br,base.state_br_rs,Barra do Quaraí
city_br_815,base.br,base.state_br_rs,Barra do Ribeiro
city_br_816,base.br,base.state_br_rs,Barra do Rio Azul
city_br_817,base.br,base.state_br_ba,Barra do Rocha
city_br_818,base.br,base.state_br_sp,Barra do Turvo
city_br_819,base.br,base.state_br_se,Barra dos Coqueiros
city_br_820,base.br,base.state_br_rs,Barra Funda
city_br_821,base.br,base.state_br_mg,Barra Longa
city_br_822,base.br,base.state_br_sc,Barra Velha
city_br_823,base.br,base.state_br_pr,Barracão
city_br_824,base.br,base.state_br_rs,Barracão
city_br_825,base.br,base.state_br_pi,Barras
city_br_826,base.br,base.state_br_ce,Barreira
city_br_827,base.br,base.state_br_pi,Barreiras do Piauí
city_br_828,base.br,base.state_br_am,Barreirinha
city_br_829,base.br,base.state_br_ma,Barreirinhas
city_br_830,base.br,base.state_br_pe,Barreiros
city_br_831,base.br,base.state_br_sp,Barrinha
city_br_832,base.br,base.state_br_ce,Barro
city_br_833,base.br,base.state_br_ba,Barro Alto
city_br_834,base.br,base.state_br_go,Barro Alto
city_br_835,base.br,base.state_br_pi,Barro Duro
city_br_836,base.br,base.state_br_ba,Barro Preto
city_br_837,base.br,base.state_br_ba,Barrocas
city_br_838,base.br,base.state_br_to,Barrolândia
city_br_839,base.br,base.state_br_ce,Barroquinha
city_br_840,base.br,base.state_br_rs,Barros Cassal
city_br_841,base.br,base.state_br_mg,Barroso
city_br_842,base.br,base.state_br_sp,Bastos
city_br_843,base.br,base.state_br_ms,Bataguassu
city_br_844,base.br,base.state_br_al,Batalha
city_br_845,base.br,base.state_br_pi,Batalha
city_br_846,base.br,base.state_br_sp,Batatais
city_br_847,base.br,base.state_br_ms,Batayporã
city_br_848,base.br,base.state_br_ce,Baturité
city_br_849,base.br,base.state_br_pb,Bayeux
city_br_850,base.br,base.state_br_sp,Bebedouro
city_br_851,base.br,base.state_br_ce,Beberibe
city_br_852,base.br,base.state_br_ce,Bela Cruz
city_br_853,base.br,base.state_br_ms,Bela Vista
city_br_854,base.br,base.state_br_pr,Bela Vista da Caroba
city_br_855,base.br,base.state_br_go,Bela Vista de Goiás
city_br_856,base.br,base.state_br_mg,Bela Vista de Minas
city_br_857,base.br,base.state_br_ma,Bela Vista do Maranhão
city_br_858,base.br,base.state_br_pr,Bela Vista do Paraíso
city_br_859,base.br,base.state_br_pi,Bela Vista do Piauí
city_br_860,base.br,base.state_br_sc,Bela Vista do Toldo
city_br_861,base.br,base.state_br_ma,Belágua
city_br_862,base.br,base.state_br_al,Belém
city_br_863,base.br,base.state_br_pb,Belém
city_br_864,base.br,base.state_br_pe,Belém de Maria
city_br_865,base.br,base.state_br_pb,Belém do Brejo do Cruz
city_br_866,base.br,base.state_br_pi,Belém do Piauí
city_br_867,base.br,base.state_br_pe,Belém do São Francisco
city_br_868,base.br,base.state_br_mg,Belmiro Braga
city_br_869,base.br,base.state_br_ba,Belmonte
city_br_870,base.br,base.state_br_sc,Belmonte
city_br_871,base.br,base.state_br_ba,Belo Campo
city_br_872,base.br,base.state_br_pe,Belo Jardim
city_br_873,base.br,base.state_br_al,Belo Monte
city_br_874,base.br,base.state_br_mg,Belo Oriente
city_br_875,base.br,base.state_br_mg,Belo Vale
city_br_876,base.br,base.state_br_pa,Belterra
city_br_877,base.br,base.state_br_pi,Beneditinos
city_br_878,base.br,base.state_br_ma,Benedito Leite
city_br_879,base.br,base.state_br_sc,Benedito Novo
city_br_880,base.br,base.state_br_pa,Benevides
city_br_881,base.br,base.state_br_am,Benjamin Constant
city_br_882,base.br,base.state_br_rs,Benjamin Constant do Sul
city_br_883,base.br,base.state_br_sp,Bento de Abreu
city_br_884,base.br,base.state_br_rn,Bento Fernandes
city_br_885,base.br,base.state_br_ma,Bequimão
city_br_886,base.br,base.state_br_mg,Berilo
city_br_887,base.br,base.state_br_mg,Berizal
city_br_888,base.br,base.state_br_pb,Bernardino Batista
city_br_889,base.br,base.state_br_sp,Bernardino de Campos
city_br_890,base.br,base.state_br_ma,Bernardo do Mearim
city_br_891,base.br,base.state_br_to,Bernardo Sayão
city_br_892,base.br,base.state_br_sp,Bertioga
city_br_893,base.br,base.state_br_pi,Bertolínia
city_br_894,base.br,base.state_br_mg,Bertópolis
city_br_895,base.br,base.state_br_am,Beruri
city_br_896,base.br,base.state_br_pe,Betânia
city_br_897,base.br,base.state_br_pi,Betânia do Piauí
city_br_898,base.br,base.state_br_pe,Bezerros
city_br_899,base.br,base.state_br_mg,Bias Fortes
city_br_900,base.br,base.state_br_mg,Bicas
city_br_901,base.br,base.state_br_sc,Biguaçu
city_br_902,base.br,base.state_br_sp,Bilac
city_br_903,base.br,base.state_br_mg,Biquinhas
city_br_904,base.br,base.state_br_sp,Biritiba-Mirim
city_br_905,base.br,base.state_br_ba,Biritinga
city_br_906,base.br,base.state_br_pr,Bituruna
city_br_907,base.br,base.state_br_es,Boa Esperança
city_br_908,base.br,base.state_br_mg,Boa Esperança
city_br_909,base.br,base.state_br_pr,Boa Esperança
city_br_910,base.br,base.state_br_pr,Boa Esperança do Iguaçu
city_br_911,base.br,base.state_br_sp,Boa Esperança do Sul
city_br_912,base.br,base.state_br_pi,Boa Hora
city_br_913,base.br,base.state_br_ba,Boa Nova
city_br_914,base.br,base.state_br_rn,Boa Saúde
city_br_915,base.br,base.state_br_pb,Boa Ventura
city_br_916,base.br,base.state_br_pr,Boa Ventura de São Roque
city_br_917,base.br,base.state_br_ce,Boa Viagem
city_br_918,base.br,base.state_br_pb,Boa Vista
city_br_919,base.br,base.state_br_pr,Boa Vista da Aparecida
city_br_920,base.br,base.state_br_rs,Boa Vista das Missões
city_br_921,base.br,base.state_br_rs,Boa Vista do Buricá
city_br_922,base.br,base.state_br_rs,Boa Vista do Cadeado
city_br_923,base.br,base.state_br_ma,Boa Vista do Gurupi
city_br_924,base.br,base.state_br_rs,Boa Vista do Incra
city_br_925,base.br,base.state_br_am,Boa Vista do Ramos
city_br_926,base.br,base.state_br_rs,Boa Vista do Sul
city_br_927,base.br,base.state_br_ba,Boa Vista do Tupim
city_br_928,base.br,base.state_br_al,Boca da Mata
city_br_929,base.br,base.state_br_am,Boca do Acre
city_br_930,base.br,base.state_br_pi,Bocaina
city_br_931,base.br,base.state_br_sp,Bocaina
city_br_932,base.br,base.state_br_mg,Bocaina de Minas
city_br_933,base.br,base.state_br_sc,Bocaina do Sul
city_br_934,base.br,base.state_br_mg,Bocaiúva
city_br_935,base.br,base.state_br_pr,Bocaiúva do Sul
city_br_936,base.br,base.state_br_rn,Bodó
city_br_937,base.br,base.state_br_pe,Bodocó
city_br_938,base.br,base.state_br_ms,Bodoquena
city_br_939,base.br,base.state_br_sp,Bofete
city_br_940,base.br,base.state_br_sp,Boituva
city_br_941,base.br,base.state_br_pe,Bom Conselho
city_br_942,base.br,base.state_br_mg,Bom Despacho
city_br_943,base.br,base.state_br_ma,Bom Jardim
city_br_944,base.br,base.state_br_pe,Bom Jardim
city_br_945,base.br,base.state_br_rj,Bom Jardim
city_br_946,base.br,base.state_br_sc,Bom Jardim da Serra
city_br_947,base.br,base.state_br_go,Bom Jardim de Goiás
city_br_948,base.br,base.state_br_mg,Bom Jardim de Minas
city_br_949,base.br,base.state_br_pb,Bom Jesus
city_br_950,base.br,base.state_br_pi,Bom Jesus
city_br_951,base.br,base.state_br_rn,Bom Jesus
city_br_952,base.br,base.state_br_rs,Bom Jesus
city_br_953,base.br,base.state_br_sc,Bom Jesus
city_br_954,base.br,base.state_br_ba,Bom Jesus da Lapa
city_br_955,base.br,base.state_br_mg,Bom Jesus da Penha
city_br_956,base.br,base.state_br_ba,Bom Jesus da Serra
city_br_957,base.br,base.state_br_ma,Bom Jesus das Selvas
city_br_958,base.br,base.state_br_go,Bom Jesus de Goiás
city_br_959,base.br,base.state_br_mg,Bom Jesus do Amparo
city_br_960,base.br,base.state_br_mt,Bom Jesus do Araguaia
city_br_961,base.br,base.state_br_mg,Bom Jesus do Galho
city_br_962,base.br,base.state_br_rj,Bom Jesus do Itabapoana
city_br_963,base.br,base.state_br_es,Bom Jesus do Norte
city_br_964,base.br,base.state_br_sc,Bom Jesus do Oeste
city_br_965,base.br,base.state_br_pr,Bom Jesus do Sul
city_br_966,base.br,base.state_br_pa,Bom Jesus do Tocantins
city_br_967,base.br,base.state_br_to,Bom Jesus do Tocantins
city_br_968,base.br,base.state_br_sp,Bom Jesus dos Perdões
city_br_969,base.br,base.state_br_ma,Bom Lugar
city_br_970,base.br,base.state_br_rs,Bom Princípio
city_br_971,base.br,base.state_br_pi,Bom Princípio do Piauí
city_br_972,base.br,base.state_br_rs,Bom Progresso
city_br_973,base.br,base.state_br_mg,Bom Repouso
city_br_974,base.br,base.state_br_sc,Bom Retiro
city_br_975,base.br,base.state_br_rs,Bom Retiro do Sul
city_br_976,base.br,base.state_br_mg,Bom Sucesso
city_br_977,base.br,base.state_br_pb,Bom Sucesso
city_br_978,base.br,base.state_br_pr,Bom Sucesso
city_br_979,base.br,base.state_br_sp,Bom Sucesso de Itararé
city_br_980,base.br,base.state_br_pr,Bom Sucesso do Sul
city_br_981,base.br,base.state_br_sc,Bombinhas
city_br_982,base.br,base.state_br_mg,Bonfim
city_br_983,base.br,base.state_br_rr,Bonfim
city_br_984,base.br,base.state_br_pi,Bonfim do Piauí
city_br_985,base.br,base.state_br_go,Bonfinópolis
city_br_986,base.br,base.state_br_mg,Bonfinópolis de Minas
city_br_987,base.br,base.state_br_ba,Boninal
city_br_988,base.br,base.state_br_ba,Bonito
city_br_989,base.br,base.state_br_ms,Bonito
city_br_990,base.br,base.state_br_pa,Bonito
city_br_991,base.br,base.state_br_pe,Bonito
city_br_992,base.br,base.state_br_mg,Bonito de Minas
city_br_993,base.br,base.state_br_pb,Bonito de Santa Fé
city_br_994,base.br,base.state_br_go,Bonópolis
city_br_995,base.br,base.state_br_pb,Boqueirão
city_br_996,base.br,base.state_br_rs,Boqueirão do Leão
city_br_997,base.br,base.state_br_pi,Boqueirão do Piauí
city_br_998,base.br,base.state_br_se,Boquim
city_br_999,base.br,base.state_br_ba,Boquira
city_br_1000,base.br,base.state_br_sp,Borá
city_br_1001,base.br,base.state_br_sp,Boracéia
city_br_1002,base.br,base.state_br_am,Borba
city_br_1003,base.br,base.state_br_pb,Borborema
city_br_1004,base.br,base.state_br_sp,Borborema
city_br_1005,base.br,base.state_br_mg,Borda da Mata
city_br_1006,base.br,base.state_br_sp,Borebi
city_br_1007,base.br,base.state_br_pr,Borrazópolis
city_br_1008,base.br,base.state_br_rs,Bossoroca
city_br_1009,base.br,base.state_br_mg,Botelhos
city_br_1010,base.br,base.state_br_mg,Botumirim
city_br_1011,base.br,base.state_br_ba,Botuporã
city_br_1012,base.br,base.state_br_sc,Botuverá
city_br_1013,base.br,base.state_br_rs,Bozano
city_br_1014,base.br,base.state_br_sc,Braço do Norte
city_br_1015,base.br,base.state_br_sc,Braço do Trombudo
city_br_1016,base.br,base.state_br_rs,Braga
city_br_1017,base.br,base.state_br_pr,Braganey
city_br_1018,base.br,base.state_br_al,Branquinha
city_br_1019,base.br,base.state_br_mg,Brás Pires
city_br_1020,base.br,base.state_br_pa,Brasil Novo
city_br_1021,base.br,base.state_br_ms,Brasilândia
city_br_1022,base.br,base.state_br_mg,Brasilândia de Minas
city_br_1023,base.br,base.state_br_pr,Brasilândia do Sul
city_br_1024,base.br,base.state_br_to,Brasilândia do Tocantins
city_br_1025,base.br,base.state_br_ac,Brasiléia
city_br_1026,base.br,base.state_br_pi,Brasileira
city_br_1027,base.br,base.state_br_mg,Brasília de Minas
city_br_1028,base.br,base.state_br_mt,Brasnorte
city_br_1029,base.br,base.state_br_sp,Braúna
city_br_1030,base.br,base.state_br_mg,Braúnas
city_br_1031,base.br,base.state_br_go,Brazabrantes
city_br_1032,base.br,base.state_br_mg,Brazópolis
city_br_1033,base.br,base.state_br_pe,Brejão
city_br_1034,base.br,base.state_br_es,Brejetuba
city_br_1035,base.br,base.state_br_pe,Brejinho
city_br_1036,base.br,base.state_br_rn,Brejinho
city_br_1037,base.br,base.state_br_to,Brejinho de Nazaré
city_br_1038,base.br,base.state_br_ma,Brejo
city_br_1039,base.br,base.state_br_sp,Brejo Alegre
city_br_1040,base.br,base.state_br_pe,Brejo da Madre de Deus
city_br_1041,base.br,base.state_br_ma,Brejo de Areia
city_br_1042,base.br,base.state_br_pb,Brejo do Cruz
city_br_1043,base.br,base.state_br_pi,Brejo do Piauí
city_br_1044,base.br,base.state_br_pb,Brejo dos Santos
city_br_1045,base.br,base.state_br_se,Brejo Grande
city_br_1046,base.br,base.state_br_pa,Brejo Grande do Araguaia
city_br_1047,base.br,base.state_br_ce,Brejo Santo
city_br_1048,base.br,base.state_br_ba,Brejões
city_br_1049,base.br,base.state_br_ba,Brejolândia
city_br_1050,base.br,base.state_br_pa,Breu Branco
city_br_1051,base.br,base.state_br_go,Britânia
city_br_1052,base.br,base.state_br_rs,Brochier
city_br_1053,base.br,base.state_br_sp,Brodowski
city_br_1054,base.br,base.state_br_sp,Brotas
city_br_1055,base.br,base.state_br_ba,Brotas de Macaúbas
city_br_1056,base.br,base.state_br_mg,Brumadinho
city_br_1057,base.br,base.state_br_ba,Brumado
city_br_1058,base.br,base.state_br_sc,Brunópolis
city_br_1059,base.br,base.state_br_mg,Bueno Brandão
city_br_1060,base.br,base.state_br_mg,Buenópolis
city_br_1061,base.br,base.state_br_pe,Buenos Aires
city_br_1062,base.br,base.state_br_ba,Buerarema
city_br_1063,base.br,base.state_br_mg,Bugre
city_br_1064,base.br,base.state_br_pe,Buíque
city_br_1065,base.br,base.state_br_ac,Bujari
city_br_1066,base.br,base.state_br_pa,Bujaru
city_br_1067,base.br,base.state_br_sp,Buri
city_br_1068,base.br,base.state_br_sp,Buritama
city_br_1069,base.br,base.state_br_ma,Buriti
city_br_1070,base.br,base.state_br_go,Buriti Alegre
city_br_1071,base.br,base.state_br_ma,Buriti Bravo
city_br_1072,base.br,base.state_br_go,Buriti de Goiás
city_br_1073,base.br,base.state_br_to,Buriti do Tocantins
city_br_1074,base.br,base.state_br_pi,Buriti dos Lopes
city_br_1075,base.br,base.state_br_pi,Buriti dos Montes
city_br_1076,base.br,base.state_br_ma,Buriticupu
city_br_1077,base.br,base.state_br_go,Buritinópolis
city_br_1078,base.br,base.state_br_ba,Buritirama
city_br_1079,base.br,base.state_br_ma,Buritirana
city_br_1080,base.br,base.state_br_mg,Buritis
city_br_1081,base.br,base.state_br_ro,Buritis
city_br_1082,base.br,base.state_br_sp,Buritizal
city_br_1083,base.br,base.state_br_mg,Buritizeiro
city_br_1084,base.br,base.state_br_rs,Butiá
city_br_1085,base.br,base.state_br_am,Caapiranga
city_br_1086,base.br,base.state_br_pb,Caaporã
city_br_1087,base.br,base.state_br_ms,Caarapó
city_br_1088,base.br,base.state_br_ba,Caatiba
city_br_1089,base.br,base.state_br_pb,Cabaceiras
city_br_1090,base.br,base.state_br_ba,Cabaceiras do Paraguaçu
city_br_1091,base.br,base.state_br_mg,Cabeceira Grande
city_br_1092,base.br,base.state_br_go,Cabeceiras
city_br_1093,base.br,base.state_br_pi,Cabeceiras do Piauí
city_br_1094,base.br,base.state_br_pb,Cabedelo
city_br_1095,base.br,base.state_br_ro,Cabixi
city_br_1096,base.br,base.state_br_mg,Cabo Verde
city_br_1097,base.br,base.state_br_sp,Cabrália Paulista
city_br_1098,base.br,base.state_br_sp,Cabreúva
city_br_1099,base.br,base.state_br_pe,Cabrobó
city_br_1100,base.br,base.state_br_sc,Caçador
city_br_1101,base.br,base.state_br_sp,Caçapava
city_br_1102,base.br,base.state_br_rs,Caçapava do Sul
city_br_1103,base.br,base.state_br_ro,Cacaulândia
city_br_1104,base.br,base.state_br_rs,Cacequi
city_br_1105,base.br,base.state_br_mt,Cáceres
city_br_1106,base.br,base.state_br_ba,Cachoeira
city_br_1107,base.br,base.state_br_go,Cachoeira Alta
city_br_1108,base.br,base.state_br_mg,Cachoeira da Prata
city_br_1109,base.br,base.state_br_go,Cachoeira de Goiás
city_br_1110,base.br,base.state_br_mg,Cachoeira de Minas
city_br_1111,base.br,base.state_br_mg,Cachoeira de Pajeú
city_br_1112,base.br,base.state_br_pa,Cachoeira do Arari
city_br_1113,base.br,base.state_br_pa,Cachoeira do Piriá
city_br_1114,base.br,base.state_br_rs,Cachoeira do Sul
city_br_1115,base.br,base.state_br_pb,Cachoeira dos Índios
city_br_1116,base.br,base.state_br_go,Cachoeira Dourada
city_br_1117,base.br,base.state_br_mg,Cachoeira Dourada
city_br_1118,base.br,base.state_br_ma,Cachoeira Grande
city_br_1119,base.br,base.state_br_sp,Cachoeira Paulista
city_br_1120,base.br,base.state_br_rj,Cachoeiras de Macacu
city_br_1121,base.br,base.state_br_pe,Cachoeirinha
city_br_1122,base.br,base.state_br_to,Cachoeirinha
city_br_1123,base.br,base.state_br_pb,Cacimba de Areia
city_br_1124,base.br,base.state_br_pb,Cacimba de Dentro
city_br_1125,base.br,base.state_br_pb,Cacimbas
city_br_1126,base.br,base.state_br_al,Cacimbinhas
city_br_1127,base.br,base.state_br_rs,Cacique Doble
city_br_1128,base.br,base.state_br_ro,Cacoal
city_br_1129,base.br,base.state_br_sp,Caconde
city_br_1130,base.br,base.state_br_go,Caçu
city_br_1131,base.br,base.state_br_ba,Caculé
city_br_1132,base.br,base.state_br_ba,Caém
city_br_1133,base.br,base.state_br_mg,Caetanópolis
city_br_1134,base.br,base.state_br_ba,Caetanos
city_br_1135,base.br,base.state_br_mg,Caeté
city_br_1136,base.br,base.state_br_pe,Caetés
city_br_1137,base.br,base.state_br_ba,Caetité
city_br_1138,base.br,base.state_br_ba,Cafarnaum
city_br_1139,base.br,base.state_br_pr,Cafeara
city_br_1140,base.br,base.state_br_pr,Cafelândia
city_br_1141,base.br,base.state_br_sp,Cafelândia
city_br_1142,base.br,base.state_br_pr,Cafezal do Sul
city_br_1143,base.br,base.state_br_sp,Caiabu
city_br_1144,base.br,base.state_br_mg,Caiana
city_br_1145,base.br,base.state_br_go,Caiapônia
city_br_1146,base.br,base.state_br_rs,Caibaté
city_br_1147,base.br,base.state_br_sc,Caibi
city_br_1148,base.br,base.state_br_pb,Caiçara
city_br_1149,base.br,base.state_br_rs,Caiçara
city_br_1150,base.br,base.state_br_rn,Caiçara do Norte
city_br_1151,base.br,base.state_br_rn,Caiçara do Rio do Vento
city_br_1152,base.br,base.state_br_rn,Caicó
city_br_1153,base.br,base.state_br_sp,Caieiras
city_br_1154,base.br,base.state_br_ba,Cairu
city_br_1155,base.br,base.state_br_sp,Caiuá
city_br_1156,base.br,base.state_br_sp,Cajamar
city_br_1157,base.br,base.state_br_ma,Cajapió
city_br_1158,base.br,base.state_br_ma,Cajari
city_br_1159,base.br,base.state_br_sp,Cajati
city_br_1160,base.br,base.state_br_pb,Cajazeiras
city_br_1161,base.br,base.state_br_pi,Cajazeiras do Piauí
city_br_1162,base.br,base.state_br_pb,Cajazeirinhas
city_br_1163,base.br,base.state_br_sp,Cajobi
city_br_1164,base.br,base.state_br_al,Cajueiro
city_br_1165,base.br,base.state_br_pi,Cajueiro da Praia
city_br_1166,base.br,base.state_br_mg,Cajuri
city_br_1167,base.br,base.state_br_sp,Cajuru
city_br_1168,base.br,base.state_br_pe,Calçado
city_br_1169,base.br,base.state_br_ap,Calçoene
city_br_1170,base.br,base.state_br_mg,Caldas
city_br_1171,base.br,base.state_br_pb,Caldas Brandão
city_br_1172,base.br,base.state_br_go,Caldas Novas
city_br_1173,base.br,base.state_br_go,Caldazinha
city_br_1174,base.br,base.state_br_ba,Caldeirão Grande
city_br_1175,base.br,base.state_br_pi,Caldeirão Grande do Piauí
city_br_1176,base.br,base.state_br_pr,Califórnia
city_br_1177,base.br,base.state_br_sc,Calmon
city_br_1178,base.br,base.state_br_pe,Calumbi
city_br_1179,base.br,base.state_br_ba,Camacan
city_br_1180,base.br,base.state_br_mg,Camacho
city_br_1181,base.br,base.state_br_pb,Camalaú
city_br_1182,base.br,base.state_br_ba,Camamu
city_br_1183,base.br,base.state_br_mg,Camanducaia
city_br_1184,base.br,base.state_br_ms,Camapuã
city_br_1185,base.br,base.state_br_rs,Camaquã
city_br_1186,base.br,base.state_br_rs,Camargo
city_br_1187,base.br,base.state_br_pr,Cambará
city_br_1188,base.br,base.state_br_rs,Cambará do Sul
city_br_1189,base.br,base.state_br_pr,Cambira
city_br_1190,base.br,base.state_br_rj,Cambuci
city_br_1191,base.br,base.state_br_mg,Cambuí
city_br_1192,base.br,base.state_br_mg,Cambuquira
city_br_1193,base.br,base.state_br_ce,Camocim
city_br_1194,base.br,base.state_br_pe,Camocim de São Félix
city_br_1195,base.br,base.state_br_mg,Campanário
city_br_1196,base.br,base.state_br_mg,Campanha
city_br_1197,base.br,base.state_br_al,Campestre
city_br_1198,base.br,base.state_br_mg,Campestre
city_br_1199,base.br,base.state_br_rs,Campestre da Serra
city_br_1200,base.br,base.state_br_go,Campestre de Goiás
city_br_1201,base.br,base.state_br_ma,Campestre do Maranhão
city_br_1202,base.br,base.state_br_pr,Campina da Lagoa
city_br_1203,base.br,base.state_br_rs,Campina das Missões
city_br_1204,base.br,base.state_br_sp,Campina do Monte Alegre
city_br_1205,base.br,base.state_br_pr,Campina do Simão
city_br_1206,base.br,base.state_br_pr,Campina Grande do Sul
city_br_1207,base.br,base.state_br_mg,Campina Verde
city_br_1208,base.br,base.state_br_go,Campinaçu
city_br_1209,base.br,base.state_br_mt,Campinápolis
city_br_1210,base.br,base.state_br_pi,Campinas do Piauí
city_br_1211,base.br,base.state_br_rs,Campinas do Sul
city_br_1212,base.br,base.state_br_go,Campinorte
city_br_1213,base.br,base.state_br_al,Campo Alegre
city_br_1214,base.br,base.state_br_sc,Campo Alegre
city_br_1215,base.br,base.state_br_go,Campo Alegre de Goiás
city_br_1216,base.br,base.state_br_ba,Campo Alegre de Lourdes
city_br_1217,base.br,base.state_br_pi,Campo Alegre do Fidalgo
city_br_1218,base.br,base.state_br_mg,Campo Azul
city_br_1219,base.br,base.state_br_mg,Campo Belo
city_br_1220,base.br,base.state_br_sc,Campo Belo do Sul
city_br_1221,base.br,base.state_br_rs,Campo Bom
city_br_1222,base.br,base.state_br_pr,Campo Bonito
city_br_1223,base.br,base.state_br_se,Campo do Brito
city_br_1224,base.br,base.state_br_mg,Campo do Meio
city_br_1225,base.br,base.state_br_pr,Campo do Tenente
city_br_1226,base.br,base.state_br_sc,Campo Erê
city_br_1227,base.br,base.state_br_mg,Campo Florido
city_br_1228,base.br,base.state_br_ba,Campo Formoso
city_br_1229,base.br,base.state_br_al,Campo Grande
city_br_1230,base.br,base.state_br_rn,Campo Grande
city_br_1231,base.br,base.state_br_pi,Campo Grande do Piauí
city_br_1232,base.br,base.state_br_pi,Campo Largo do Piauí
city_br_1233,base.br,base.state_br_go,Campo Limpo de Goiás
city_br_1234,base.br,base.state_br_sp,Campo Limpo Paulista
city_br_1235,base.br,base.state_br_pr,Campo Magro
city_br_1236,base.br,base.state_br_pi,Campo Maior
city_br_1237,base.br,base.state_br_pr,Campo Mourão
city_br_1238,base.br,base.state_br_rs,Campo Novo
city_br_1239,base.br,base.state_br_ro,Campo Novo de Rondônia
city_br_1240,base.br,base.state_br_mt,Campo Novo do Parecis
city_br_1241,base.br,base.state_br_rn,Campo Redondo
city_br_1242,base.br,base.state_br_mt,Campo Verde
city_br_1243,base.br,base.state_br_mg,Campos Altos
city_br_1244,base.br,base.state_br_go,Campos Belos
city_br_1245,base.br,base.state_br_rs,Campos Borges
city_br_1246,base.br,base.state_br_mt,Campos de Júlio
city_br_1247,base.br,base.state_br_sp,Campos do Jordão
city_br_1248,base.br,base.state_br_mg,Campos Gerais
city_br_1249,base.br,base.state_br_to,Campos Lindos
city_br_1250,base.br,base.state_br_sc,Campos Novos
city_br_1251,base.br,base.state_br_sp,Campos Novos Paulista
city_br_1252,base.br,base.state_br_ce,Campos Sales
city_br_1253,base.br,base.state_br_go,Campos Verdes
city_br_1254,base.br,base.state_br_pe,Camutanga
city_br_1255,base.br,base.state_br_mg,Cana Verde
city_br_1256,base.br,base.state_br_mg,Canaã
city_br_1257,base.br,base.state_br_pa,Canaã dos Carajás
city_br_1258,base.br,base.state_br_mt,Canabrava do Norte
city_br_1259,base.br,base.state_br_sp,Cananéia
city_br_1260,base.br,base.state_br_al,Canapi
city_br_1261,base.br,base.state_br_ba,Canápolis
city_br_1262,base.br,base.state_br_mg,Canápolis
city_br_1263,base.br,base.state_br_ba,Canarana
city_br_1264,base.br,base.state_br_mt,Canarana
city_br_1265,base.br,base.state_br_sp,Canas
city_br_1266,base.br,base.state_br_pi,Canavieira
city_br_1267,base.br,base.state_br_ba,Canavieiras
city_br_1268,base.br,base.state_br_ba,Candeal
city_br_1269,base.br,base.state_br_ba,Candeias
city_br_1270,base.br,base.state_br_mg,Candeias
city_br_1271,base.br,base.state_br_ro,Candeias do Jamari
city_br_1272,base.br,base.state_br_rs,Candelária
city_br_1273,base.br,base.state_br_ba,Candiba
city_br_1274,base.br,base.state_br_pr,Cândido de Abreu
city_br_1275,base.br,base.state_br_rs,Cândido Godói
city_br_1276,base.br,base.state_br_ma,Cândido Mendes
city_br_1277,base.br,base.state_br_sp,Cândido Mota
city_br_1278,base.br,base.state_br_sp,Cândido Rodrigues
city_br_1279,base.br,base.state_br_ba,Cândido Sales
city_br_1280,base.br,base.state_br_rs,Candiota
city_br_1281,base.br,base.state_br_pr,Candói
city_br_1282,base.br,base.state_br_rs,Canela
city_br_1283,base.br,base.state_br_sc,Canelinha
city_br_1284,base.br,base.state_br_rn,Canguaretama
city_br_1285,base.br,base.state_br_rs,Canguçu
city_br_1286,base.br,base.state_br_se,Canhoba
city_br_1287,base.br,base.state_br_pe,Canhotinho
city_br_1288,base.br,base.state_br_ce,Canindé
city_br_1289,base.br,base.state_br_se,Canindé de São Francisco
city_br_1290,base.br,base.state_br_sp,Canitar
city_br_1291,base.br,base.state_br_sc,Canoinhas
city_br_1292,base.br,base.state_br_ba,Cansanção
city_br_1293,base.br,base.state_br_rr,Cantá
city_br_1294,base.br,base.state_br_mg,Cantagalo
city_br_1295,base.br,base.state_br_pr,Cantagalo
city_br_1296,base.br,base.state_br_rj,Cantagalo
city_br_1297,base.br,base.state_br_ma,Cantanhede
city_br_1298,base.br,base.state_br_pi,Canto do Buriti
city_br_1299,base.br,base.state_br_ba,Canudos
city_br_1300,base.br,base.state_br_rs,Canudos do Vale
city_br_1301,base.br,base.state_br_am,Canutama
city_br_1302,base.br,base.state_br_pa,Capanema
city_br_1303,base.br,base.state_br_pr,Capanema
city_br_1304,base.br,base.state_br_sc,Capão Alto
city_br_1305,base.br,base.state_br_sp,Capão Bonito
city_br_1306,base.br,base.state_br_rs,Capão Bonito do Sul
city_br_1307,base.br,base.state_br_rs,Capão da Canoa
city_br_1308,base.br,base.state_br_rs,Capão do Cipó
city_br_1309,base.br,base.state_br_rs,Capão do Leão
city_br_1310,base.br,base.state_br_mg,Caparaó
city_br_1311,base.br,base.state_br_al,Capela
city_br_1312,base.br,base.state_br_se,Capela
city_br_1313,base.br,base.state_br_rs,Capela de Santana
city_br_1314,base.br,base.state_br_sp,Capela do Alto
city_br_1315,base.br,base.state_br_ba,Capela do Alto Alegre
city_br_1316,base.br,base.state_br_mg,Capela Nova
city_br_1317,base.br,base.state_br_mg,Capelinha
city_br_1318,base.br,base.state_br_mg,Capetinga
city_br_1319,base.br,base.state_br_pb,Capim
city_br_1320,base.br,base.state_br_mg,Capim Branco
city_br_1321,base.br,base.state_br_ba,Capim Grosso
city_br_1322,base.br,base.state_br_mg,Capinópolis
city_br_1323,base.br,base.state_br_sc,Capinzal
city_br_1324,base.br,base.state_br_ma,Capinzal do Norte
city_br_1325,base.br,base.state_br_ce,Capistrano
city_br_1326,base.br,base.state_br_rs,Capitão
city_br_1327,base.br,base.state_br_mg,Capitão Andrade
city_br_1328,base.br,base.state_br_pi,Capitão de Campos
city_br_1329,base.br,base.state_br_mg,Capitão Enéas
city_br_1330,base.br,base.state_br_pi,Capitão Gervásio Oliveira
city_br_1331,base.br,base.state_br_pr,Capitão Leônidas Marques
city_br_1332,base.br,base.state_br_pa,Capitão Poço
city_br_1333,base.br,base.state_br_mg,Capitólio
city_br_1334,base.br,base.state_br_sp,Capivari
city_br_1335,base.br,base.state_br_sc,Capivari de Baixo
city_br_1336,base.br,base.state_br_rs,Capivari do Sul
city_br_1337,base.br,base.state_br_ac,Capixaba
city_br_1338,base.br,base.state_br_pe,Capoeiras
city_br_1339,base.br,base.state_br_mg,Caputira
city_br_1340,base.br,base.state_br_rs,Caraá
city_br_1341,base.br,base.state_br_rr,Caracaraí
city_br_1342,base.br,base.state_br_ms,Caracol
city_br_1343,base.br,base.state_br_pi,Caracol
city_br_1344,base.br,base.state_br_mg,Caraí
city_br_1345,base.br,base.state_br_ba,Caraíbas
city_br_1346,base.br,base.state_br_pr,Carambeí
city_br_1347,base.br,base.state_br_mg,Caranaíba
city_br_1348,base.br,base.state_br_mg,Carandaí
city_br_1349,base.br,base.state_br_mg,Carangola
city_br_1350,base.br,base.state_br_rj,Carapebus
city_br_1351,base.br,base.state_br_mg,Caratinga
city_br_1352,base.br,base.state_br_am,Carauari
city_br_1353,base.br,base.state_br_pb,Caraúbas
city_br_1354,base.br,base.state_br_rn,Caraúbas
city_br_1355,base.br,base.state_br_pi,Caraúbas do Piauí
city_br_1356,base.br,base.state_br_ba,Caravelas
city_br_1357,base.br,base.state_br_rs,Carazinho
city_br_1358,base.br,base.state_br_mg,Carbonita
city_br_1359,base.br,base.state_br_ba,Cardeal da Silva
city_br_1360,base.br,base.state_br_sp,Cardoso
city_br_1361,base.br,base.state_br_rj,Cardoso Moreira
city_br_1362,base.br,base.state_br_mg,Careaçu
city_br_1363,base.br,base.state_br_am,Careiro
city_br_1364,base.br,base.state_br_am,Careiro da Várzea
city_br_1365,base.br,base.state_br_ce,Caridade
city_br_1366,base.br,base.state_br_pi,Caridade do Piauí
city_br_1367,base.br,base.state_br_ba,Carinhanha
city_br_1368,base.br,base.state_br_se,Carira
city_br_1369,base.br,base.state_br_ce,Cariré
city_br_1370,base.br,base.state_br_to,Cariri do Tocantins
city_br_1371,base.br,base.state_br_ce,Caririaçu
city_br_1372,base.br,base.state_br_ce,Cariús
city_br_1373,base.br,base.state_br_mt,Carlinda
city_br_1374,base.br,base.state_br_pr,Carlópolis
city_br_1375,base.br,base.state_br_rs,Carlos Barbosa
city_br_1376,base.br,base.state_br_mg,Carlos Chagas
city_br_1377,base.br,base.state_br_rs,Carlos Gomes
city_br_1378,base.br,base.state_br_mg,Carmésia
city_br_1379,base.br,base.state_br_rj,Carmo
city_br_1380,base.br,base.state_br_mg,Carmo da Cachoeira
city_br_1381,base.br,base.state_br_mg,Carmo da Mata
city_br_1382,base.br,base.state_br_mg,Carmo de Minas
city_br_1383,base.br,base.state_br_mg,Carmo do Cajuru
city_br_1384,base.br,base.state_br_mg,Carmo do Paranaíba
city_br_1385,base.br,base.state_br_mg,Carmo do Rio Claro
city_br_1386,base.br,base.state_br_go,Carmo do Rio Verde
city_br_1387,base.br,base.state_br_to,Carmolândia
city_br_1388,base.br,base.state_br_se,Carmópolis
city_br_1389,base.br,base.state_br_mg,Carmópolis de Minas
city_br_1390,base.br,base.state_br_pe,Carnaíba
city_br_1391,base.br,base.state_br_rn,Carnaúba dos Dantas
city_br_1392,base.br,base.state_br_rn,Carnaubais
city_br_1393,base.br,base.state_br_ce,Carnaubal
city_br_1394,base.br,base.state_br_pe,Carnaubeira da Penha
city_br_1395,base.br,base.state_br_mg,Carneirinho
city_br_1396,base.br,base.state_br_al,Carneiros
city_br_1397,base.br,base.state_br_rr,Caroebe
city_br_1398,base.br,base.state_br_ma,Carolina
city_br_1399,base.br,base.state_br_pe,Carpina
city_br_1400,base.br,base.state_br_mg,Carrancas
city_br_1401,base.br,base.state_br_pb,Carrapateira
city_br_1402,base.br,base.state_br_to,Carrasco Bonito
city_br_1403,base.br,base.state_br_ma,Carutapera
city_br_1404,base.br,base.state_br_mg,Carvalhópolis
city_br_1405,base.br,base.state_br_mg,Carvalhos
city_br_1406,base.br,base.state_br_sp,Casa Branca
city_br_1407,base.br,base.state_br_mg,Casa Grande
city_br_1408,base.br,base.state_br_ba,Casa Nova
city_br_1409,base.br,base.state_br_rs,Casca
city_br_1410,base.br,base.state_br_mg,Cascalho Rico
city_br_1411,base.br,base.state_br_ce,Cascavel
city_br_1412,base.br,base.state_br_to,Caseara
city_br_1413,base.br,base.state_br_rs,Caseiros
city_br_1414,base.br,base.state_br_rj,Casimiro de Abreu
city_br_1415,base.br,base.state_br_pe,Casinhas
city_br_1416,base.br,base.state_br_pb,Casserengue
city_br_1417,base.br,base.state_br_mg,Cássia
city_br_1418,base.br,base.state_br_sp,Cássia dos Coqueiros
city_br_1419,base.br,base.state_br_ms,Cassilândia
city_br_1420,base.br,base.state_br_mt,Castanheira
city_br_1421,base.br,base.state_br_ro,Castanheiras
city_br_1422,base.br,base.state_br_go,Castelândia
city_br_1423,base.br,base.state_br_es,Castelo
city_br_1424,base.br,base.state_br_pi,Castelo do Piauí
city_br_1425,base.br,base.state_br_sp,Castilho
city_br_1426,base.br,base.state_br_pr,Castro
city_br_1427,base.br,base.state_br_ba,Castro Alves
city_br_1428,base.br,base.state_br_mg,Cataguases
city_br_1429,base.br,base.state_br_pr,Catanduvas
city_br_1430,base.br,base.state_br_sc,Catanduvas
city_br_1431,base.br,base.state_br_ce,Catarina
city_br_1432,base.br,base.state_br_mg,Catas Altas
city_br_1433,base.br,base.state_br_mg,Catas Altas da Noruega
city_br_1434,base.br,base.state_br_pe,Catende
city_br_1435,base.br,base.state_br_sp,Catiguá
city_br_1436,base.br,base.state_br_pb,Catingueira
city_br_1437,base.br,base.state_br_ba,Catolândia
city_br_1438,base.br,base.state_br_pb,Catolé do Rocha
city_br_1439,base.br,base.state_br_ba,Catu
city_br_1440,base.br,base.state_br_rs,Catuípe
city_br_1441,base.br,base.state_br_mg,Catuji
city_br_1442,base.br,base.state_br_ce,Catunda
city_br_1443,base.br,base.state_br_go,Caturaí
city_br_1444,base.br,base.state_br_ba,Caturama
city_br_1445,base.br,base.state_br_pb,Caturité
city_br_1446,base.br,base.state_br_mg,Catuti
city_br_1447,base.br,base.state_br_go,Cavalcante
city_br_1448,base.br,base.state_br_mg,Caxambu
city_br_1449,base.br,base.state_br_sc,Caxambu do Sul
city_br_1450,base.br,base.state_br_pi,Caxingó
city_br_1451,base.br,base.state_br_rn,Ceará-Mirim
city_br_1452,base.br,base.state_br_ma,Cedral
city_br_1453,base.br,base.state_br_sp,Cedral
city_br_1454,base.br,base.state_br_ce,Cedro
city_br_1455,base.br,base.state_br_pe,Cedro
city_br_1456,base.br,base.state_br_se,Cedro de São João
city_br_1457,base.br,base.state_br_mg,Cedro do Abaeté
city_br_1458,base.br,base.state_br_sc,Celso Ramos
city_br_1459,base.br,base.state_br_rs,Centenário
city_br_1460,base.br,base.state_br_to,Centenário
city_br_1461,base.br,base.state_br_pr,Centenário do Sul
city_br_1462,base.br,base.state_br_ba,Central
city_br_1463,base.br,base.state_br_mg,Central de Minas
city_br_1464,base.br,base.state_br_ma,Central do Maranhão
city_br_1465,base.br,base.state_br_mg,Centralina
city_br_1466,base.br,base.state_br_ma,Centro do Guilherme
city_br_1467,base.br,base.state_br_ma,Centro Novo do Maranhão
city_br_1468,base.br,base.state_br_ro,Cerejeiras
city_br_1469,base.br,base.state_br_go,Ceres
city_br_1470,base.br,base.state_br_sp,Cerqueira César
city_br_1471,base.br,base.state_br_sp,Cerquilho
city_br_1472,base.br,base.state_br_rs,Cerrito
city_br_1473,base.br,base.state_br_pr,Cerro Azul
city_br_1474,base.br,base.state_br_rs,Cerro Branco
city_br_1475,base.br,base.state_br_rn,Cerro Corá
city_br_1476,base.br,base.state_br_rs,Cerro Grande
city_br_1477,base.br,base.state_br_rs,Cerro Grande do Sul
city_br_1478,base.br,base.state_br_rs,Cerro Largo
city_br_1479,base.br,base.state_br_sc,Cerro Negro
city_br_1480,base.br,base.state_br_sp,Cesário Lange
city_br_1481,base.br,base.state_br_pr,Céu Azul
city_br_1482,base.br,base.state_br_go,Cezarina
city_br_1483,base.br,base.state_br_pe,Chã de Alegria
city_br_1484,base.br,base.state_br_pe,Chã Grande
city_br_1485,base.br,base.state_br_al,Chã Preta
city_br_1486,base.br,base.state_br_mg,Chácara
city_br_1487,base.br,base.state_br_mg,Chalé
city_br_1488,base.br,base.state_br_rs,Chapada
city_br_1489,base.br,base.state_br_to,Chapada da Natividade
city_br_1490,base.br,base.state_br_to,Chapada de Areia
city_br_1491,base.br,base.state_br_mg,Chapada do Norte
city_br_1492,base.br,base.state_br_mt,Chapada dos Guimarães
city_br_1493,base.br,base.state_br_mg,Chapada Gaúcha
city_br_1494,base.br,base.state_br_go,Chapadão do Céu
city_br_1495,base.br,base.state_br_sc,Chapadão do Lageado
city_br_1496,base.br,base.state_br_ms,Chapadão do Sul
city_br_1497,base.br,base.state_br_ma,Chapadinha
city_br_1498,base.br,base.state_br_sp,Charqueada
city_br_1499,base.br,base.state_br_rs,Charqueadas
city_br_1500,base.br,base.state_br_rs,Charrua
city_br_1501,base.br,base.state_br_ce,Chaval
city_br_1502,base.br,base.state_br_sp,Chavantes
city_br_1503,base.br,base.state_br_pa,Chaves
city_br_1504,base.br,base.state_br_mg,Chiador
city_br_1505,base.br,base.state_br_rs,Chiapetta
city_br_1506,base.br,base.state_br_pr,Chopinzinho
city_br_1507,base.br,base.state_br_ce,Choró
city_br_1508,base.br,base.state_br_ce,Chorozinho
city_br_1509,base.br,base.state_br_ba,Chorrochó
city_br_1510,base.br,base.state_br_rs,Chuí
city_br_1511,base.br,base.state_br_ro,Chupinguaia
city_br_1512,base.br,base.state_br_rs,Chuvisca
city_br_1513,base.br,base.state_br_pr,Cianorte
city_br_1514,base.br,base.state_br_ba,Cícero Dantas
city_br_1515,base.br,base.state_br_pr,Cidade Gaúcha
city_br_1516,base.br,base.state_br_go,Cidade Ocidental
city_br_1517,base.br,base.state_br_ma,Cidelândia
city_br_1518,base.br,base.state_br_rs,Cidreira
city_br_1519,base.br,base.state_br_ba,Cipó
city_br_1520,base.br,base.state_br_mg,Cipotânea
city_br_1521,base.br,base.state_br_rs,Ciríaco
city_br_1522,base.br,base.state_br_mg,Claraval
city_br_1523,base.br,base.state_br_mg,Claro dos Poções
city_br_1524,base.br,base.state_br_mt,Cláudia
city_br_1525,base.br,base.state_br_mg,Cláudio
city_br_1526,base.br,base.state_br_sp,Clementina
city_br_1527,base.br,base.state_br_pr,Clevelândia
city_br_1528,base.br,base.state_br_ba,Coaraci
city_br_1529,base.br,base.state_br_am,Coari
city_br_1530,base.br,base.state_br_pi,Cocal
city_br_1531,base.br,base.state_br_pi,Cocal de Telha
city_br_1532,base.br,base.state_br_sc,Cocal do Sul
city_br_1533,base.br,base.state_br_pi,Cocal dos Alves
city_br_1534,base.br,base.state_br_mt,Cocalinho
city_br_1535,base.br,base.state_br_go,Cocalzinho de Goiás
city_br_1536,base.br,base.state_br_ba,Cocos
city_br_1537,base.br,base.state_br_am,Codajás
city_br_1538,base.br,base.state_br_ma,Coelho Neto
city_br_1539,base.br,base.state_br_mg,Coimbra
city_br_1540,base.br,base.state_br_al,Coité do Nóia
city_br_1541,base.br,base.state_br_pi,Coivaras
city_br_1542,base.br,base.state_br_pa,Colares
city_br_1543,base.br,base.state_br_mt,Colíder
city_br_1544,base.br,base.state_br_sp,Colina
city_br_1545,base.br,base.state_br_ma,Colinas
city_br_1546,base.br,base.state_br_rs,Colinas
city_br_1547,base.br,base.state_br_go,Colinas do Sul
city_br_1548,base.br,base.state_br_to,Colinas do Tocantins
city_br_1549,base.br,base.state_br_to,Colméia
city_br_1550,base.br,base.state_br_mt,Colniza
city_br_1551,base.br,base.state_br_sp,Colômbia
city_br_1552,base.br,base.state_br_pi,Colônia do Gurguéia
city_br_1553,base.br,base.state_br_pi,Colônia do Piauí
city_br_1554,base.br,base.state_br_al,Colônia Leopoldina
city_br_1555,base.br,base.state_br_pr,Colorado
city_br_1556,base.br,base.state_br_rs,Colorado
city_br_1557,base.br,base.state_br_ro,Colorado do Oeste
city_br_1558,base.br,base.state_br_mg,Coluna
city_br_1559,base.br,base.state_br_to,Combinado
city_br_1560,base.br,base.state_br_mg,Comendador Gomes
city_br_1561,base.br,base.state_br_rj,Comendador Levy Gasparian
city_br_1562,base.br,base.state_br_mg,Comercinho
city_br_1563,base.br,base.state_br_mt,Comodoro
city_br_1564,base.br,base.state_br_pb,Conceição
city_br_1565,base.br,base.state_br_mg,Conceição da Aparecida
city_br_1566,base.br,base.state_br_es,Conceição da Barra
city_br_1567,base.br,base.state_br_mg,Conceição da Barra de Minas
city_br_1568,base.br,base.state_br_ba,Conceição da Feira
city_br_1569,base.br,base.state_br_mg,Conceição das Alagoas
city_br_1570,base.br,base.state_br_mg,Conceição das Pedras
city_br_1571,base.br,base.state_br_mg,Conceição de Ipanema
city_br_1572,base.br,base.state_br_rj,Conceição de Macabu
city_br_1573,base.br,base.state_br_ba,Conceição do Almeida
city_br_1574,base.br,base.state_br_pa,Conceição do Araguaia
city_br_1575,base.br,base.state_br_pi,Conceição do Canindé
city_br_1576,base.br,base.state_br_es,Conceição do Castelo
city_br_1577,base.br,base.state_br_ba,Conceição do Coité
city_br_1578,base.br,base.state_br_ba,Conceição do Jacuípe
city_br_1579,base.br,base.state_br_ma,Conceição do Lago-Açu
city_br_1580,base.br,base.state_br_mg,Conceição do Mato Dentro
city_br_1581,base.br,base.state_br_mg,Conceição do Pará
city_br_1582,base.br,base.state_br_mg,Conceição do Rio Verde
city_br_1583,base.br,base.state_br_to,Conceição do Tocantins
city_br_1584,base.br,base.state_br_mg,Conceição dos Ouros
city_br_1585,base.br,base.state_br_sp,Conchal
city_br_1586,base.br,base.state_br_sp,Conchas
city_br_1587,base.br,base.state_br_sc,Concórdia
city_br_1588,base.br,base.state_br_pa,Concórdia do Pará
city_br_1589,base.br,base.state_br_pb,Condado
city_br_1590,base.br,base.state_br_pe,Condado
city_br_1591,base.br,base.state_br_ba,Conde
city_br_1592,base.br,base.state_br_pb,Conde
city_br_1593,base.br,base.state_br_ba,Condeúba
city_br_1594,base.br,base.state_br_rs,Condor
city_br_1595,base.br,base.state_br_mg,Cônego Marinho
city_br_1596,base.br,base.state_br_mg,Confins
city_br_1597,base.br,base.state_br_mt,Confresa
city_br_1598,base.br,base.state_br_pb,Congo
city_br_1599,base.br,base.state_br_mg,Congonhal
city_br_1600,base.br,base.state_br_mg,Congonhas
city_br_1601,base.br,base.state_br_mg,Congonhas do Norte
city_br_1602,base.br,base.state_br_pr,Congonhinhas
city_br_1603,base.br,base.state_br_mg,Conquista
city_br_1604,base.br,base.state_br_mt,Conquista D'Oeste
city_br_1605,base.br,base.state_br_pr,Conselheiro Mairinck
city_br_1606,base.br,base.state_br_mg,Conselheiro Pena
city_br_1607,base.br,base.state_br_mg,Consolação
city_br_1608,base.br,base.state_br_rs,Constantina
city_br_1609,base.br,base.state_br_pr,Contenda
city_br_1610,base.br,base.state_br_ba,Contendas do Sincorá
city_br_1611,base.br,base.state_br_mg,Coqueiral
city_br_1612,base.br,base.state_br_rs,Coqueiro Baixo
city_br_1613,base.br,base.state_br_al,Coqueiro Seco
city_br_1614,base.br,base.state_br_rs,Coqueiros do Sul
city_br_1615,base.br,base.state_br_mg,Coração de Jesus
city_br_1616,base.br,base.state_br_ba,Coração de Maria
city_br_1617,base.br,base.state_br_pr,Corbélia
city_br_1618,base.br,base.state_br_rj,Cordeiro
city_br_1619,base.br,base.state_br_sp,Cordeirópolis
city_br_1620,base.br,base.state_br_ba,Cordeiros
city_br_1621,base.br,base.state_br_sc,Cordilheira Alta
city_br_1622,base.br,base.state_br_mg,Cordisburgo
city_br_1623,base.br,base.state_br_mg,Cordislândia
city_br_1624,base.br,base.state_br_ce,Coreaú
city_br_1625,base.br,base.state_br_pb,Coremas
city_br_1626,base.br,base.state_br_ms,Corguinho
city_br_1627,base.br,base.state_br_ba,Coribe
city_br_1628,base.br,base.state_br_mg,Corinto
city_br_1629,base.br,base.state_br_pr,Cornélio Procópio
city_br_1630,base.br,base.state_br_mg,Coroaci
city_br_1631,base.br,base.state_br_sp,Coroados
city_br_1632,base.br,base.state_br_ma,Coroatá
city_br_1633,base.br,base.state_br_mg,Coromandel
city_br_1634,base.br,base.state_br_rs,Coronel Barros
city_br_1635,base.br,base.state_br_rs,Coronel Bicaco
city_br_1636,base.br,base.state_br_pr,Coronel Domingos Soares
city_br_1637,base.br,base.state_br_rn,Coronel Ezequiel
city_br_1638,base.br,base.state_br_sc,Coronel Freitas
city_br_1639,base.br,base.state_br_rn,Coronel João Pessoa
city_br_1640,base.br,base.state_br_ba,Coronel João Sá
city_br_1641,base.br,base.state_br_pi,Coronel José Dias
city_br_1642,base.br,base.state_br_sp,Coronel Macedo
city_br_1643,base.br,base.state_br_sc,Coronel Martins
city_br_1644,base.br,base.state_br_mg,Coronel Murta
city_br_1645,base.br,base.state_br_mg,Coronel Pacheco
city_br_1646,base.br,base.state_br_rs,Coronel Pilar
city_br_1647,base.br,base.state_br_ms,Coronel Sapucaia
city_br_1648,base.br,base.state_br_pr,Coronel Vivida
city_br_1649,base.br,base.state_br_mg,Coronel Xavier Chaves
city_br_1650,base.br,base.state_br_mg,Córrego Danta
city_br_1651,base.br,base.state_br_mg,Córrego do Bom Jesus
city_br_1652,base.br,base.state_br_go,Córrego do Ouro
city_br_1653,base.br,base.state_br_mg,Córrego Fundo
city_br_1654,base.br,base.state_br_mg,Córrego Novo
city_br_1655,base.br,base.state_br_sc,Correia Pinto
city_br_1656,base.br,base.state_br_pi,Corrente
city_br_1657,base.br,base.state_br_pe,Correntes
city_br_1658,base.br,base.state_br_ba,Correntina
city_br_1659,base.br,base.state_br_pe,Cortês
city_br_1660,base.br,base.state_br_ms,Corumbá
city_br_1661,base.br,base.state_br_go,Corumbá de Goiás
city_br_1662,base.br,base.state_br_go,Corumbaíba
city_br_1663,base.br,base.state_br_sp,Corumbataí
city_br_1664,base.br,base.state_br_pr,Corumbataí do Sul
city_br_1665,base.br,base.state_br_ro,Corumbiara
city_br_1666,base.br,base.state_br_sc,Corupá
city_br_1667,base.br,base.state_br_al,Coruripe
city_br_1668,base.br,base.state_br_sp,Cosmópolis
city_br_1669,base.br,base.state_br_sp,Cosmorama
city_br_1670,base.br,base.state_br_ro,Costa Marques
city_br_1671,base.br,base.state_br_ms,Costa Rica
city_br_1672,base.br,base.state_br_ba,Cotegipe
city_br_1673,base.br,base.state_br_rs,Cotiporã
city_br_1674,base.br,base.state_br_mt,Cotriguaçu
city_br_1675,base.br,base.state_br_mg,Couto de Magalhães de Minas
city_br_1676,base.br,base.state_br_to,Couto Magalhães
city_br_1677,base.br,base.state_br_rs,Coxilha
city_br_1678,base.br,base.state_br_ms,Coxim
city_br_1679,base.br,base.state_br_pb,Coxixola
city_br_1680,base.br,base.state_br_al,Craíbas
city_br_1681,base.br,base.state_br_ce,Crateús
city_br_1682,base.br,base.state_br_sp,Cravinhos
city_br_1683,base.br,base.state_br_ba,Cravolândia
city_br_1684,base.br,base.state_br_mg,Crisólita
city_br_1685,base.br,base.state_br_ba,Crisópolis
city_br_1686,base.br,base.state_br_rs,Crissiumal
city_br_1687,base.br,base.state_br_mg,Cristais
city_br_1688,base.br,base.state_br_sp,Cristais Paulista
city_br_1689,base.br,base.state_br_rs,Cristal
city_br_1690,base.br,base.state_br_rs,Cristal do Sul
city_br_1691,base.br,base.state_br_to,Cristalândia
city_br_1692,base.br,base.state_br_pi,Cristalândia do Piauí
city_br_1693,base.br,base.state_br_mg,Cristália
city_br_1694,base.br,base.state_br_go,Cristalina
city_br_1695,base.br,base.state_br_mg,Cristiano Otoni
city_br_1696,base.br,base.state_br_go,Cristianópolis
city_br_1697,base.br,base.state_br_mg,Cristina
city_br_1698,base.br,base.state_br_se,Cristinápolis
city_br_1699,base.br,base.state_br_pi,Cristino Castro
city_br_1700,base.br,base.state_br_ba,Cristópolis
city_br_1701,base.br,base.state_br_go,Crixás
city_br_1702,base.br,base.state_br_to,Crixás do Tocantins
city_br_1703,base.br,base.state_br_ce,Croatá
city_br_1704,base.br,base.state_br_go,Cromínia
city_br_1705,base.br,base.state_br_mg,Crucilândia
city_br_1706,base.br,base.state_br_ce,Cruz
city_br_1707,base.br,base.state_br_rs,Cruz Alta
city_br_1708,base.br,base.state_br_ba,Cruz das Almas
city_br_1709,base.br,base.state_br_pb,Cruz do Espírito Santo
city_br_1710,base.br,base.state_br_pr,Cruz Machado
city_br_1711,base.br,base.state_br_sp,Cruzália
city_br_1712,base.br,base.state_br_rs,Cruzaltense
city_br_1713,base.br,base.state_br_sp,Cruzeiro
city_br_1714,base.br,base.state_br_mg,Cruzeiro da Fortaleza
city_br_1715,base.br,base.state_br_pr,Cruzeiro do Iguaçu
city_br_1716,base.br,base.state_br_pr,Cruzeiro do Oeste
city_br_1717,base.br,base.state_br_ac,Cruzeiro do Sul
city_br_1718,base.br,base.state_br_pr,Cruzeiro do Sul
city_br_1719,base.br,base.state_br_rs,Cruzeiro do Sul
city_br_1720,base.br,base.state_br_rn,Cruzeta
city_br_1721,base.br,base.state_br_mg,Cruzília
city_br_1722,base.br,base.state_br_pr,Cruzmaltina
city_br_1723,base.br,base.state_br_pb,Cubati
city_br_1724,base.br,base.state_br_pb,Cuité
city_br_1725,base.br,base.state_br_pb,Cuité de Mamanguape
city_br_1726,base.br,base.state_br_pb,Cuitegi
city_br_1727,base.br,base.state_br_ro,Cujubim
city_br_1728,base.br,base.state_br_go,Cumari
city_br_1729,base.br,base.state_br_pe,Cumaru
city_br_1730,base.br,base.state_br_pa,Cumaru do Norte
city_br_1731,base.br,base.state_br_se,Cumbe
city_br_1732,base.br,base.state_br_sp,Cunha
city_br_1733,base.br,base.state_br_sc,Cunha Porã
city_br_1734,base.br,base.state_br_sc,Cunhataí
city_br_1735,base.br,base.state_br_mg,Cuparaque
city_br_1736,base.br,base.state_br_pe,Cupira
city_br_1737,base.br,base.state_br_ba,Curaçá
city_br_1738,base.br,base.state_br_pi,Curimatá
city_br_1739,base.br,base.state_br_pa,Curionópolis
city_br_1740,base.br,base.state_br_sc,Curitibanos
city_br_1741,base.br,base.state_br_pr,Curiúva
city_br_1742,base.br,base.state_br_pi,Currais
city_br_1743,base.br,base.state_br_rn,Currais Novos
city_br_1744,base.br,base.state_br_pb,Curral de Cima
city_br_1745,base.br,base.state_br_mg,Curral de Dentro
city_br_1746,base.br,base.state_br_pi,Curral Novo do Piauí
city_br_1747,base.br,base.state_br_pb,Curral Velho
city_br_1748,base.br,base.state_br_pa,Curralinho
city_br_1749,base.br,base.state_br_pi,Curralinhos
city_br_1750,base.br,base.state_br_pa,Curuá
city_br_1751,base.br,base.state_br_pa,Curuçá
city_br_1752,base.br,base.state_br_ma,Cururupu
city_br_1753,base.br,base.state_br_mt,Curvelândia
city_br_1754,base.br,base.state_br_mg,Curvelo
city_br_1755,base.br,base.state_br_pe,Custódia
city_br_1756,base.br,base.state_br_ap,Cutias
city_br_1757,base.br,base.state_br_go,Damianópolis
city_br_1758,base.br,base.state_br_pb,Damião
city_br_1759,base.br,base.state_br_go,Damolândia
city_br_1760,base.br,base.state_br_to,Darcinópolis
city_br_1761,base.br,base.state_br_ba,Dário Meira
city_br_1762,base.br,base.state_br_mg,Datas
city_br_1763,base.br,base.state_br_rs,David Canabarro
city_br_1764,base.br,base.state_br_go,Davinópolis
city_br_1765,base.br,base.state_br_ma,Davinópolis
city_br_1766,base.br,base.state_br_mg,Delfim Moreira
city_br_1767,base.br,base.state_br_mg,Delfinópolis
city_br_1768,base.br,base.state_br_al,Delmiro Gouveia
city_br_1769,base.br,base.state_br_mg,Delta
city_br_1770,base.br,base.state_br_pi,Demerval Lobão
city_br_1771,base.br,base.state_br_mt,Denise
city_br_1772,base.br,base.state_br_ms,Deodápolis
city_br_1773,base.br,base.state_br_ce,Deputado Irapuan Pinheiro
city_br_1774,base.br,base.state_br_rs,Derrubadas
city_br_1775,base.br,base.state_br_sp,Descalvado
city_br_1776,base.br,base.state_br_sc,Descanso
city_br_1777,base.br,base.state_br_mg,Descoberto
city_br_1778,base.br,base.state_br_pb,Desterro
city_br_1779,base.br,base.state_br_mg,Desterro de Entre Rios
city_br_1780,base.br,base.state_br_mg,Desterro do Melo
city_br_1781,base.br,base.state_br_rs,Dezesseis de Novembro
city_br_1782,base.br,base.state_br_pb,Diamante
city_br_1783,base.br,base.state_br_pr,Diamante D'Oeste
city_br_1784,base.br,base.state_br_pr,Diamante do Norte
city_br_1785,base.br,base.state_br_pr,Diamante do Sul
city_br_1786,base.br,base.state_br_mg,Diamantina
city_br_1787,base.br,base.state_br_mt,Diamantino
city_br_1788,base.br,base.state_br_to,Dianópolis
city_br_1789,base.br,base.state_br_ba,Dias d'Ávila
city_br_1790,base.br,base.state_br_rs,Dilermando de Aguiar
city_br_1791,base.br,base.state_br_mg,Diogo de Vasconcelos
city_br_1792,base.br,base.state_br_mg,Dionísio
city_br_1793,base.br,base.state_br_sc,Dionísio Cerqueira
city_br_1794,base.br,base.state_br_go,Diorama
city_br_1795,base.br,base.state_br_sp,Dirce Reis
city_br_1796,base.br,base.state_br_pi,Dirceu Arcoverde
city_br_1797,base.br,base.state_br_se,Divina Pastora
city_br_1798,base.br,base.state_br_mg,Divinésia
city_br_1799,base.br,base.state_br_mg,Divino
city_br_1800,base.br,base.state_br_mg,Divino das Laranjeiras
city_br_1801,base.br,base.state_br_es,Divino de São Lourenço
city_br_1802,base.br,base.state_br_sp,Divinolândia
city_br_1803,base.br,base.state_br_mg,Divinolândia de Minas
city_br_1804,base.br,base.state_br_go,Divinópolis de Goiás
city_br_1805,base.br,base.state_br_to,Divinópolis do Tocantins
city_br_1806,base.br,base.state_br_mg,Divisa Alegre
city_br_1807,base.br,base.state_br_mg,Divisa Nova
city_br_1808,base.br,base.state_br_mg,Divisópolis
city_br_1809,base.br,base.state_br_sp,Dobrada
city_br_1810,base.br,base.state_br_sp,Dois Córregos
city_br_1811,base.br,base.state_br_rs,Dois Irmãos
city_br_1812,base.br,base.state_br_rs,Dois Irmãos das Missões
city_br_1813,base.br,base.state_br_ms,Dois Irmãos do Buriti
city_br_1814,base.br,base.state_br_to,Dois Irmãos do Tocantins
city_br_1815,base.br,base.state_br_rs,Dois Lajeados
city_br_1816,base.br,base.state_br_al,Dois Riachos
city_br_1817,base.br,base.state_br_pr,Dois Vizinhos
city_br_1818,base.br,base.state_br_sp,Dolcinópolis
city_br_1819,base.br,base.state_br_mt,Dom Aquino
city_br_1820,base.br,base.state_br_ba,Dom Basílio
city_br_1821,base.br,base.state_br_mg,Dom Bosco
city_br_1822,base.br,base.state_br_mg,Dom Cavati
city_br_1823,base.br,base.state_br_pa,Dom Eliseu
city_br_1824,base.br,base.state_br_pi,Dom Expedito Lopes
city_br_1825,base.br,base.state_br_rs,Dom Feliciano
city_br_1826,base.br,base.state_br_pi,Dom Inocêncio
city_br_1827,base.br,base.state_br_mg,Dom Joaquim
city_br_1828,base.br,base.state_br_ba,Dom Macedo Costa
city_br_1829,base.br,base.state_br_rs,Dom Pedrito
city_br_1830,base.br,base.state_br_ma,Dom Pedro
city_br_1831,base.br,base.state_br_rs,Dom Pedro de Alcântara
city_br_1832,base.br,base.state_br_mg,Dom Silvério
city_br_1833,base.br,base.state_br_mg,Dom Viçoso
city_br_1834,base.br,base.state_br_es,Domingos Martins
city_br_1835,base.br,base.state_br_pi,Domingos Mourão
city_br_1836,base.br,base.state_br_sc,Dona Emma
city_br_1837,base.br,base.state_br_mg,Dona Eusébia
city_br_1838,base.br,base.state_br_rs,Dona Francisca
city_br_1839,base.br,base.state_br_pb,Dona Inês
city_br_1840,base.br,base.state_br_mg,Dores de Campos
city_br_1841,base.br,base.state_br_mg,Dores de Guanhães
city_br_1842,base.br,base.state_br_mg,Dores do Indaiá
city_br_1843,base.br,base.state_br_es,Dores do Rio Preto
city_br_1844,base.br,base.state_br_mg,Dores do Turvo
city_br_1845,base.br,base.state_br_mg,Doresópolis
city_br_1846,base.br,base.state_br_pe,Dormentes
city_br_1847,base.br,base.state_br_ms,Douradina
city_br_1848,base.br,base.state_br_pr,Douradina
city_br_1849,base.br,base.state_br_sp,Dourado
city_br_1850,base.br,base.state_br_mg,Douradoquara
city_br_1851,base.br,base.state_br_pr,Doutor Camargo
city_br_1852,base.br,base.state_br_rs,Doutor Maurício Cardoso
city_br_1853,base.br,base.state_br_sc,Doutor Pedrinho
city_br_1854,base.br,base.state_br_rs,Doutor Ricardo
city_br_1855,base.br,base.state_br_rn,Doutor Severiano
city_br_1856,base.br,base.state_br_pr,Doutor Ulysses
city_br_1857,base.br,base.state_br_go,Doverlândia
city_br_1858,base.br,base.state_br_sp,Dracena
city_br_1859,base.br,base.state_br_sp,Duartina
city_br_1860,base.br,base.state_br_rj,Duas Barras
city_br_1861,base.br,base.state_br_pb,Duas Estradas
city_br_1862,base.br,base.state_br_to,Dueré
city_br_1863,base.br,base.state_br_sp,Dumont
city_br_1864,base.br,base.state_br_ma,Duque Bacelar
city_br_1865,base.br,base.state_br_mg,Durandé
city_br_1866,base.br,base.state_br_sp,Echaporã
city_br_1867,base.br,base.state_br_es,Ecoporanga
city_br_1868,base.br,base.state_br_go,Edealina
city_br_1869,base.br,base.state_br_go,Edéia
city_br_1870,base.br,base.state_br_am,Eirunepé
city_br_1871,base.br,base.state_br_ms,Eldorado
city_br_1872,base.br,base.state_br_sp,Eldorado
city_br_1873,base.br,base.state_br_pa,Eldorado do Carajás
city_br_1874,base.br,base.state_br_rs,Eldorado do Sul
city_br_1875,base.br,base.state_br_pi,Elesbão Veloso
city_br_1876,base.br,base.state_br_sp,Elias Fausto
city_br_1877,base.br,base.state_br_pi,Eliseu Martins
city_br_1878,base.br,base.state_br_sp,Elisiário
city_br_1879,base.br,base.state_br_ba,Elísio Medrado
city_br_1880,base.br,base.state_br_mg,Elói Mendes
city_br_1881,base.br,base.state_br_pb,Emas
city_br_1882,base.br,base.state_br_sp,Embaúba
city_br_1883,base.br,base.state_br_sp,Embu-Guaçu
city_br_1884,base.br,base.state_br_sp,Emilianópolis
city_br_1885,base.br,base.state_br_rs,Encantado
city_br_1886,base.br,base.state_br_rn,Encanto
city_br_1887,base.br,base.state_br_ba,Encruzilhada
city_br_1888,base.br,base.state_br_rs,Encruzilhada do Sul
city_br_1889,base.br,base.state_br_pr,Enéas Marques
city_br_1890,base.br,base.state_br_pr,Engenheiro Beltrão
city_br_1891,base.br,base.state_br_mg,Engenheiro Caldas
city_br_1892,base.br,base.state_br_sp,Engenheiro Coelho
city_br_1893,base.br,base.state_br_mg,Engenheiro Navarro
city_br_1894,base.br,base.state_br_rj,Engenheiro Paulo de Frontin
city_br_1895,base.br,base.state_br_rs,Engenho Velho
city_br_1896,base.br,base.state_br_mg,Entre Folhas
city_br_1897,base.br,base.state_br_rs,Entre-Ijuís
city_br_1898,base.br,base.state_br_ba,Entre Rios
city_br_1899,base.br,base.state_br_sc,Entre Rios
city_br_1900,base.br,base.state_br_mg,Entre Rios de Minas
city_br_1901,base.br,base.state_br_pr,Entre Rios do Oeste
city_br_1902,base.br,base.state_br_rs,Entre Rios do Sul
city_br_1903,base.br,base.state_br_am,Envira
city_br_1904,base.br,base.state_br_ac,Epitaciolândia
city_br_1905,base.br,base.state_br_rn,Equador
city_br_1906,base.br,base.state_br_rs,Erebango
city_br_1907,base.br,base.state_br_ce,Ererê
city_br_1908,base.br,base.state_br_ba,Érico Cardoso
city_br_1909,base.br,base.state_br_sc,Ermo
city_br_1910,base.br,base.state_br_rs,Ernestina
city_br_1911,base.br,base.state_br_rs,Erval Grande
city_br_1912,base.br,base.state_br_rs,Erval Seco
city_br_1913,base.br,base.state_br_sc,Erval Velho
city_br_1914,base.br,base.state_br_mg,Ervália
city_br_1915,base.br,base.state_br_pe,Escada
city_br_1916,base.br,base.state_br_rs,Esmeralda
city_br_1917,base.br,base.state_br_mg,Esmeraldas
city_br_1918,base.br,base.state_br_mg,Espera Feliz
city_br_1919,base.br,base.state_br_pb,Esperança
city_br_1920,base.br,base.state_br_rs,Esperança do Sul
city_br_1921,base.br,base.state_br_pr,Esperança Nova
city_br_1922,base.br,base.state_br_pi,Esperantina
city_br_1923,base.br,base.state_br_to,Esperantina
city_br_1924,base.br,base.state_br_ma,Esperantinópolis
city_br_1925,base.br,base.state_br_pr,Espigão Alto do Iguaçu
city_br_1926,base.br,base.state_br_ro,Espigão D'Oeste
city_br_1927,base.br,base.state_br_mg,Espinosa
city_br_1928,base.br,base.state_br_rn,Espírito Santo
city_br_1929,base.br,base.state_br_mg,Espírito Santo do Dourado
city_br_1930,base.br,base.state_br_sp,Espírito Santo do Pinhal
city_br_1931,base.br,base.state_br_sp,Espírito Santo do Turvo
city_br_1932,base.br,base.state_br_ba,Esplanada
city_br_1933,base.br,base.state_br_rs,Espumoso
city_br_1934,base.br,base.state_br_rs,Estação
city_br_1935,base.br,base.state_br_se,Estância
city_br_1936,base.br,base.state_br_rs,Estância Velha
city_br_1937,base.br,base.state_br_rs,Esteio
city_br_1938,base.br,base.state_br_mg,Estiva
city_br_1939,base.br,base.state_br_sp,Estiva Gerbi
city_br_1940,base.br,base.state_br_ma,Estreito
city_br_1941,base.br,base.state_br_rs,Estrela
city_br_1942,base.br,base.state_br_sp,Estrela d'Oeste
city_br_1943,base.br,base.state_br_mg,Estrela Dalva
city_br_1944,base.br,base.state_br_al,Estrela de Alagoas
city_br_1945,base.br,base.state_br_mg,Estrela do Indaiá
city_br_1946,base.br,base.state_br_go,Estrela do Norte
city_br_1947,base.br,base.state_br_sp,Estrela do Norte
city_br_1948,base.br,base.state_br_mg,Estrela do Sul
city_br_1949,base.br,base.state_br_rs,Estrela Velha
city_br_1950,base.br,base.state_br_ba,Euclides da Cunha
city_br_1951,base.br,base.state_br_sp,Euclides da Cunha Paulista
city_br_1952,base.br,base.state_br_rs,Eugênio de Castro
city_br_1953,base.br,base.state_br_mg,Eugenópolis
city_br_1954,base.br,base.state_br_ce,Eusébio
city_br_1955,base.br,base.state_br_mg,Ewbank da Câmara
city_br_1956,base.br,base.state_br_mg,Extrema
city_br_1957,base.br,base.state_br_rn,Extremoz
city_br_1958,base.br,base.state_br_pe,Exu
city_br_1959,base.br,base.state_br_pb,Fagundes
city_br_1960,base.br,base.state_br_rs,Fagundes Varela
city_br_1961,base.br,base.state_br_go,Faina
city_br_1962,base.br,base.state_br_mg,Fama
city_br_1963,base.br,base.state_br_mg,Faria Lemos
city_br_1964,base.br,base.state_br_ce,Farias Brito
city_br_1965,base.br,base.state_br_pa,Faro
city_br_1966,base.br,base.state_br_pr,Farol
city_br_1967,base.br,base.state_br_rs,Farroupilha
city_br_1968,base.br,base.state_br_sp,Fartura
city_br_1969,base.br,base.state_br_pi,Fartura do Piauí
city_br_1970,base.br,base.state_br_ba,Fátima
city_br_1971,base.br,base.state_br_to,Fátima
city_br_1972,base.br,base.state_br_ms,Fátima do Sul
city_br_1973,base.br,base.state_br_pr,Faxinal
city_br_1974,base.br,base.state_br_rs,Faxinal do Soturno
city_br_1975,base.br,base.state_br_sc,Faxinal dos Guedes
city_br_1976,base.br,base.state_br_rs,Faxinalzinho
city_br_1977,base.br,base.state_br_go,Fazenda Nova
city_br_1978,base.br,base.state_br_rs,Fazenda Vilanova
city_br_1979,base.br,base.state_br_ac,Feijó
city_br_1980,base.br,base.state_br_ba,Feira da Mata
city_br_1981,base.br,base.state_br_al,Feira Grande
city_br_1982,base.br,base.state_br_pe,Feira Nova
city_br_1983,base.br,base.state_br_se,Feira Nova
city_br_1984,base.br,base.state_br_ma,Feira Nova do Maranhão
city_br_1985,base.br,base.state_br_mg,Felício dos Santos
city_br_1986,base.br,base.state_br_rn,Felipe Guerra
city_br_1987,base.br,base.state_br_mg,Felisburgo
city_br_1988,base.br,base.state_br_mg,Felixlândia
city_br_1989,base.br,base.state_br_rs,Feliz
city_br_1990,base.br,base.state_br_al,Feliz Deserto
city_br_1991,base.br,base.state_br_mt,Feliz Natal
city_br_1992,base.br,base.state_br_pr,Fênix
city_br_1993,base.br,base.state_br_pr,Fernandes Pinheiro
city_br_1994,base.br,base.state_br_mg,Fernandes Tourinho
city_br_1995,base.br,base.state_br_pe,Fernando de Noronha
city_br_1996,base.br,base.state_br_ma,Fernando Falcão
city_br_1997,base.br,base.state_br_rn,Fernando Pedroza
city_br_1998,base.br,base.state_br_sp,Fernando Prestes
city_br_1999,base.br,base.state_br_sp,Fernandópolis
city_br_2000,base.br,base.state_br_sp,Fernão
city_br_2001,base.br,base.state_br_ap,Ferreira Gomes
city_br_2002,base.br,base.state_br_pe,Ferreiros
city_br_2003,base.br,base.state_br_mg,Ferros
city_br_2004,base.br,base.state_br_mg,Fervedouro
city_br_2005,base.br,base.state_br_pr,Figueira
city_br_2006,base.br,base.state_br_ms,Figueirão
city_br_2007,base.br,base.state_br_to,Figueirópolis
city_br_2008,base.br,base.state_br_mt,Figueirópolis D'Oeste
city_br_2009,base.br,base.state_br_ba,Filadélfia
city_br_2010,base.br,base.state_br_to,Filadélfia
city_br_2011,base.br,base.state_br_ba,Firmino Alves
city_br_2012,base.br,base.state_br_go,Firminópolis
city_br_2013,base.br,base.state_br_al,Flexeiras
city_br_2014,base.br,base.state_br_pr,Flor da Serra do Sul
city_br_2015,base.br,base.state_br_sc,Flor do Sertão
city_br_2016,base.br,base.state_br_sp,Flora Rica
city_br_2017,base.br,base.state_br_pr,Floraí
city_br_2018,base.br,base.state_br_rn,Florânia
city_br_2019,base.br,base.state_br_sp,Floreal
city_br_2020,base.br,base.state_br_pe,Flores
city_br_2021,base.br,base.state_br_rs,Flores da Cunha
city_br_2022,base.br,base.state_br_go,Flores de Goiás
city_br_2023,base.br,base.state_br_pi,Flores do Piauí
city_br_2024,base.br,base.state_br_pe,Floresta
city_br_2025,base.br,base.state_br_pr,Floresta
city_br_2026,base.br,base.state_br_ba,Floresta Azul
city_br_2027,base.br,base.state_br_pa,Floresta do Araguaia
city_br_2028,base.br,base.state_br_pi,Floresta do Piauí
city_br_2029,base.br,base.state_br_mg,Florestal
city_br_2030,base.br,base.state_br_pr,Florestópolis
city_br_2031,base.br,base.state_br_pi,Floriano
city_br_2032,base.br,base.state_br_rs,Floriano Peixoto
city_br_2033,base.br,base.state_br_pr,Flórida
city_br_2034,base.br,base.state_br_sp,Flórida Paulista
city_br_2035,base.br,base.state_br_sp,Florínea
city_br_2036,base.br,base.state_br_am,Fonte Boa
city_br_2037,base.br,base.state_br_rs,Fontoura Xavier
city_br_2038,base.br,base.state_br_mg,Formiga
city_br_2039,base.br,base.state_br_rs,Formigueiro
city_br_2040,base.br,base.state_br_ma,Formosa da Serra Negra
city_br_2041,base.br,base.state_br_pr,Formosa do Oeste
city_br_2042,base.br,base.state_br_ba,Formosa do Rio Preto
city_br_2043,base.br,base.state_br_sc,Formosa do Sul
city_br_2044,base.br,base.state_br_go,Formoso
city_br_2045,base.br,base.state_br_mg,Formoso
city_br_2046,base.br,base.state_br_to,Formoso do Araguaia
city_br_2047,base.br,base.state_br_rs,Forquetinha
city_br_2048,base.br,base.state_br_ce,Forquilha
city_br_2049,base.br,base.state_br_sc,Forquilhinha
city_br_2050,base.br,base.state_br_mg,Fortaleza de Minas
city_br_2051,base.br,base.state_br_to,Fortaleza do Tabocão
city_br_2052,base.br,base.state_br_ma,Fortaleza dos Nogueiras
city_br_2053,base.br,base.state_br_rs,Fortaleza dos Valos
city_br_2054,base.br,base.state_br_ce,Fortim
city_br_2055,base.br,base.state_br_ma,Fortuna
city_br_2056,base.br,base.state_br_mg,Fortuna de Minas
city_br_2057,base.br,base.state_br_pr,Foz do Jordão
city_br_2058,base.br,base.state_br_sc,Fraiburgo
city_br_2059,base.br,base.state_br_pi,Francinópolis
city_br_2060,base.br,base.state_br_pr,Francisco Alves
city_br_2061,base.br,base.state_br_pi,Francisco Ayres
city_br_2062,base.br,base.state_br_mg,Francisco Badaró
city_br_2063,base.br,base.state_br_pr,Francisco Beltrão
city_br_2064,base.br,base.state_br_rn,Francisco Dantas
city_br_2065,base.br,base.state_br_mg,Francisco Dumont
city_br_2066,base.br,base.state_br_pi,Francisco Macedo
city_br_2067,base.br,base.state_br_mg,Francisco Sá
city_br_2068,base.br,base.state_br_pi,Francisco Santos
city_br_2069,base.br,base.state_br_mg,Franciscópolis
city_br_2070,base.br,base.state_br_ce,Frecheirinha
city_br_2071,base.br,base.state_br_rs,Frederico Westphalen
city_br_2072,base.br,base.state_br_mg,Frei Gaspar
city_br_2073,base.br,base.state_br_mg,Frei Inocêncio
city_br_2074,base.br,base.state_br_mg,Frei Lagonegro
city_br_2075,base.br,base.state_br_pb,Frei Martinho
city_br_2076,base.br,base.state_br_pe,Frei Miguelinho
city_br_2077,base.br,base.state_br_se,Frei Paulo
city_br_2078,base.br,base.state_br_sc,Frei Rogério
city_br_2079,base.br,base.state_br_mg,Fronteira
city_br_2080,base.br,base.state_br_mg,Fronteira dos Vales
city_br_2081,base.br,base.state_br_pi,Fronteiras
city_br_2082,base.br,base.state_br_mg,Fruta de Leite
city_br_2083,base.br,base.state_br_mg,Frutal
city_br_2084,base.br,base.state_br_rn,Frutuoso Gomes
city_br_2085,base.br,base.state_br_es,Fundão
city_br_2086,base.br,base.state_br_mg,Funilândia
city_br_2087,base.br,base.state_br_sp,Gabriel Monteiro
city_br_2088,base.br,base.state_br_pb,Gado Bravo
city_br_2089,base.br,base.state_br_sp,Gália
city_br_2090,base.br,base.state_br_mg,Galiléia
city_br_2091,base.br,base.state_br_rn,Galinhos
city_br_2092,base.br,base.state_br_sc,Galvão
city_br_2093,base.br,base.state_br_pe,Gameleira
city_br_2094,base.br,base.state_br_go,Gameleira de Goiás
city_br_2095,base.br,base.state_br_mg,Gameleiras
city_br_2096,base.br,base.state_br_ba,Gandu
city_br_2097,base.br,base.state_br_se,Gararu
city_br_2098,base.br,base.state_br_sp,Garça
city_br_2099,base.br,base.state_br_rs,Garibaldi
city_br_2100,base.br,base.state_br_sc,Garopaba
city_br_2101,base.br,base.state_br_pa,Garrafão do Norte
city_br_2102,base.br,base.state_br_rs,Garruchos
city_br_2103,base.br,base.state_br_sc,Garuva
city_br_2104,base.br,base.state_br_sc,Gaspar
city_br_2105,base.br,base.state_br_sp,Gastão Vidigal
city_br_2106,base.br,base.state_br_mt,Gaúcha do Norte
city_br_2107,base.br,base.state_br_rs,Gaurama
city_br_2108,base.br,base.state_br_ba,Gavião
city_br_2109,base.br,base.state_br_sp,Gavião Peixoto
city_br_2110,base.br,base.state_br_pi,Geminiano
city_br_2111,base.br,base.state_br_rs,General Câmara
city_br_2112,base.br,base.state_br_mt,General Carneiro
city_br_2113,base.br,base.state_br_pr,General Carneiro
city_br_2114,base.br,base.state_br_se,General Maynard
city_br_2115,base.br,base.state_br_sp,General Salgado
city_br_2116,base.br,base.state_br_ce,General Sampaio
city_br_2117,base.br,base.state_br_rs,Gentil
city_br_2118,base.br,base.state_br_ba,Gentio do Ouro
city_br_2119,base.br,base.state_br_sp,Getulina
city_br_2120,base.br,base.state_br_rs,Getúlio Vargas
city_br_2121,base.br,base.state_br_pi,Gilbués
city_br_2122,base.br,base.state_br_al,Girau do Ponciano
city_br_2123,base.br,base.state_br_rs,Giruá
city_br_2124,base.br,base.state_br_mg,Glaucilândia
city_br_2125,base.br,base.state_br_sp,Glicério
city_br_2126,base.br,base.state_br_ba,Glória
city_br_2127,base.br,base.state_br_mt,Glória D'Oeste
city_br_2128,base.br,base.state_br_ms,Glória de Dourados
city_br_2129,base.br,base.state_br_pe,Glória do Goitá
city_br_2130,base.br,base.state_br_rs,Glorinha
city_br_2131,base.br,base.state_br_ma,Godofredo Viana
city_br_2132,base.br,base.state_br_pr,Godoy Moreira
city_br_2133,base.br,base.state_br_mg,Goiabeira
city_br_2134,base.br,base.state_br_pe,Goiana
city_br_2135,base.br,base.state_br_mg,Goianá
city_br_2136,base.br,base.state_br_go,Goianápolis
city_br_2137,base.br,base.state_br_go,Goiandira
city_br_2138,base.br,base.state_br_go,Goianésia
city_br_2139,base.br,base.state_br_pa,Goianésia do Pará
city_br_2140,base.br,base.state_br_rn,Goianinha
city_br_2141,base.br,base.state_br_go,Goianira
city_br_2142,base.br,base.state_br_to,Goianorte
city_br_2143,base.br,base.state_br_go,Goiás
city_br_2144,base.br,base.state_br_to,Goiatins
city_br_2145,base.br,base.state_br_go,Goiatuba
city_br_2146,base.br,base.state_br_pr,Goioerê
city_br_2147,base.br,base.state_br_pr,Goioxim
city_br_2148,base.br,base.state_br_mg,Gonçalves
city_br_2149,base.br,base.state_br_ma,Gonçalves Dias
city_br_2150,base.br,base.state_br_ba,Gongogi
city_br_2151,base.br,base.state_br_mg,Gonzaga
city_br_2152,base.br,base.state_br_mg,Gouveia
city_br_2153,base.br,base.state_br_go,Gouvelândia
city_br_2154,base.br,base.state_br_ma,Governador Archer
city_br_2155,base.br,base.state_br_sc,Governador Celso Ramos
city_br_2156,base.br,base.state_br_rn,Governador Dix-Sept Rosado
city_br_2157,base.br,base.state_br_ma,Governador Edison Lobão
city_br_2158,base.br,base.state_br_ma,Governador Eugênio Barros
city_br_2159,base.br,base.state_br_ro,Governador Jorge Teixeira
city_br_2160,base.br,base.state_br_es,Governador Lindenberg
city_br_2161,base.br,base.state_br_ma,Governador Luiz Rocha
city_br_2162,base.br,base.state_br_ba,Governador Mangabeira
city_br_2163,base.br,base.state_br_ma,Governador Newton Bello
city_br_2164,base.br,base.state_br_ma,Governador Nunes Freire
city_br_2165,base.br,base.state_br_ce,Graça
city_br_2166,base.br,base.state_br_ma,Graça Aranha
city_br_2167,base.br,base.state_br_se,Gracho Cardoso
city_br_2168,base.br,base.state_br_ma,Grajaú
city_br_2169,base.br,base.state_br_rs,Gramado
city_br_2170,base.br,base.state_br_rs,Gramado dos Loureiros
city_br_2171,base.br,base.state_br_rs,Gramado Xavier
city_br_2172,base.br,base.state_br_pr,Grandes Rios
city_br_2173,base.br,base.state_br_pe,Granito
city_br_2174,base.br,base.state_br_ce,Granja
city_br_2175,base.br,base.state_br_ce,Granjeiro
city_br_2176,base.br,base.state_br_mg,Grão Mogol
city_br_2177,base.br,base.state_br_sc,Grão Pará
city_br_2178,base.br,base.state_br_pe,Gravatá
city_br_2179,base.br,base.state_br_sc,Gravatal
city_br_2180,base.br,base.state_br_ce,Groaíras
city_br_2181,base.br,base.state_br_rn,Grossos
city_br_2182,base.br,base.state_br_mg,Grupiara
city_br_2183,base.br,base.state_br_rs,Guabiju
city_br_2184,base.br,base.state_br_sc,Guabiruba
city_br_2185,base.br,base.state_br_es,Guaçuí
city_br_2186,base.br,base.state_br_pi,Guadalupe
city_br_2187,base.br,base.state_br_rs,Guaíba
city_br_2188,base.br,base.state_br_sp,Guaiçara
city_br_2189,base.br,base.state_br_sp,Guaimbê
city_br_2190,base.br,base.state_br_pr,Guaíra
city_br_2191,base.br,base.state_br_sp,Guaíra
city_br_2192,base.br,base.state_br_pr,Guairaçá
city_br_2193,base.br,base.state_br_ce,Guaiúba
city_br_2194,base.br,base.state_br_am,Guajará
city_br_2195,base.br,base.state_br_ro,Guajará-Mirim
city_br_2196,base.br,base.state_br_ba,Guajeru
city_br_2197,base.br,base.state_br_rn,Guamaré
city_br_2198,base.br,base.state_br_pr,Guamiranga
city_br_2199,base.br,base.state_br_ba,Guanambi
city_br_2200,base.br,base.state_br_mg,Guanhães
city_br_2201,base.br,base.state_br_mg,Guapé
city_br_2202,base.br,base.state_br_sp,Guapiaçu
city_br_2203,base.br,base.state_br_sp,Guapiara
city_br_2204,base.br,base.state_br_rj,Guapimirim
city_br_2205,base.br,base.state_br_pr,Guapirama
city_br_2206,base.br,base.state_br_go,Guapó
city_br_2207,base.br,base.state_br_rs,Guaporé
city_br_2208,base.br,base.state_br_pr,Guaporema
city_br_2209,base.br,base.state_br_sp,Guará
city_br_2210,base.br,base.state_br_pb,Guarabira
city_br_2211,base.br,base.state_br_sp,Guaraçaí
city_br_2212,base.br,base.state_br_pr,Guaraci
city_br_2213,base.br,base.state_br_sp,Guaraci
city_br_2214,base.br,base.state_br_mg,Guaraciaba
city_br_2215,base.br,base.state_br_sc,Guaraciaba
city_br_2216,base.br,base.state_br_ce,Guaraciaba do Norte
city_br_2217,base.br,base.state_br_mg,Guaraciama
city_br_2218,base.br,base.state_br_to,Guaraí
city_br_2219,base.br,base.state_br_go,Guaraíta
city_br_2220,base.br,base.state_br_ce,Guaramiranga
city_br_2221,base.br,base.state_br_sc,Guaramirim
city_br_2222,base.br,base.state_br_mg,Guaranésia
city_br_2223,base.br,base.state_br_mg,Guarani
city_br_2224,base.br,base.state_br_sp,Guarani d'Oeste
city_br_2225,base.br,base.state_br_rs,Guarani das Missões
city_br_2226,base.br,base.state_br_go,Guarani de Goiás
city_br_2227,base.br,base.state_br_pr,Guaraniaçu
city_br_2228,base.br,base.state_br_sp,Guarantã
city_br_2229,base.br,base.state_br_mt,Guarantã do Norte
city_br_2230,base.br,base.state_br_pr,Guaraqueçaba
city_br_2231,base.br,base.state_br_mg,Guarará
city_br_2232,base.br,base.state_br_sp,Guararapes
city_br_2233,base.br,base.state_br_sp,Guararema
city_br_2234,base.br,base.state_br_ba,Guaratinga
city_br_2235,base.br,base.state_br_pr,Guaratuba
city_br_2236,base.br,base.state_br_mg,Guarda-Mor
city_br_2237,base.br,base.state_br_sp,Guareí
city_br_2238,base.br,base.state_br_sp,Guariba
city_br_2239,base.br,base.state_br_pi,Guaribas
city_br_2240,base.br,base.state_br_go,Guarinos
city_br_2241,base.br,base.state_br_sc,Guarujá do Sul
city_br_2242,base.br,base.state_br_sc,Guatambú
city_br_2243,base.br,base.state_br_sp,Guatapará
city_br_2244,base.br,base.state_br_mg,Guaxupé
city_br_2245,base.br,base.state_br_ms,Guia Lopes da Laguna
city_br_2246,base.br,base.state_br_mg,Guidoval
city_br_2247,base.br,base.state_br_ma,Guimarães
city_br_2248,base.br,base.state_br_mg,Guimarânia
city_br_2249,base.br,base.state_br_mt,Guiratinga
city_br_2250,base.br,base.state_br_mg,Guiricema
city_br_2251,base.br,base.state_br_mg,Gurinhatã
city_br_2252,base.br,base.state_br_pb,Gurinhém
city_br_2253,base.br,base.state_br_pb,Gurjão
city_br_2254,base.br,base.state_br_pa,Gurupá
city_br_2255,base.br,base.state_br_to,Gurupi
city_br_2256,base.br,base.state_br_sp,Guzolândia
city_br_2257,base.br,base.state_br_rs,Harmonia
city_br_2258,base.br,base.state_br_go,Heitoraí
city_br_2259,base.br,base.state_br_mg,Heliodora
city_br_2260,base.br,base.state_br_ba,Heliópolis
city_br_2261,base.br,base.state_br_sp,Herculândia
city_br_2262,base.br,base.state_br_rs,Herval
city_br_2263,base.br,base.state_br_sc,Herval d'Oeste
city_br_2264,base.br,base.state_br_rs,Herveiras
city_br_2265,base.br,base.state_br_ce,Hidrolândia
city_br_2266,base.br,base.state_br_go,Hidrolândia
city_br_2267,base.br,base.state_br_go,Hidrolina
city_br_2268,base.br,base.state_br_sp,Holambra
city_br_2269,base.br,base.state_br_pr,Honório Serpa
city_br_2270,base.br,base.state_br_ce,Horizonte
city_br_2271,base.br,base.state_br_rs,Horizontina
city_br_2272,base.br,base.state_br_pi,Hugo Napoleão
city_br_2273,base.br,base.state_br_rs,Hulha Negra
city_br_2274,base.br,base.state_br_am,Humaitá
city_br_2275,base.br,base.state_br_rs,Humaitá
city_br_2276,base.br,base.state_br_ma,Humberto de Campos
city_br_2277,base.br,base.state_br_sp,Iacanga
city_br_2278,base.br,base.state_br_go,Iaciara
city_br_2279,base.br,base.state_br_sp,Iacri
city_br_2280,base.br,base.state_br_ba,Iaçu
city_br_2281,base.br,base.state_br_mg,Iapu
city_br_2282,base.br,base.state_br_sp,Iaras
city_br_2283,base.br,base.state_br_pe,Iati
city_br_2284,base.br,base.state_br_pr,Ibaiti
city_br_2285,base.br,base.state_br_rs,Ibarama
city_br_2286,base.br,base.state_br_ce,Ibaretama
city_br_2287,base.br,base.state_br_sp,Ibaté
city_br_2288,base.br,base.state_br_al,Ibateguara
city_br_2289,base.br,base.state_br_es,Ibatiba
city_br_2290,base.br,base.state_br_pr,Ibema
city_br_2291,base.br,base.state_br_mg,Ibertioga
city_br_2292,base.br,base.state_br_mg,Ibiá
city_br_2293,base.br,base.state_br_rs,Ibiaçá
city_br_2294,base.br,base.state_br_mg,Ibiaí
city_br_2295,base.br,base.state_br_sc,Ibiam
city_br_2296,base.br,base.state_br_ce,Ibiapina
city_br_2297,base.br,base.state_br_pb,Ibiara
city_br_2298,base.br,base.state_br_ba,Ibiassucê
city_br_2299,base.br,base.state_br_ba,Ibicaraí
city_br_2300,base.br,base.state_br_sc,Ibicaré
city_br_2301,base.br,base.state_br_ba,Ibicoara
city_br_2302,base.br,base.state_br_ba,Ibicuí
city_br_2303,base.br,base.state_br_ce,Ibicuitinga
city_br_2304,base.br,base.state_br_pe,Ibimirim
city_br_2305,base.br,base.state_br_ba,Ibipeba
city_br_2306,base.br,base.state_br_ba,Ibipitanga
city_br_2307,base.br,base.state_br_pr,Ibiporã
city_br_2308,base.br,base.state_br_ba,Ibiquera
city_br_2309,base.br,base.state_br_sp,Ibirá
city_br_2310,base.br,base.state_br_mg,Ibiracatu
city_br_2311,base.br,base.state_br_mg,Ibiraci
city_br_2312,base.br,base.state_br_es,Ibiraçu
city_br_2313,base.br,base.state_br_rs,Ibiraiaras
city_br_2314,base.br,base.state_br_pe,Ibirajuba
city_br_2315,base.br,base.state_br_sc,Ibirama
city_br_2316,base.br,base.state_br_ba,Ibirapitanga
city_br_2317,base.br,base.state_br_ba,Ibirapuã
city_br_2318,base.br,base.state_br_rs,Ibirapuitã
city_br_2319,base.br,base.state_br_sp,Ibirarema
city_br_2320,base.br,base.state_br_ba,Ibirataia
city_br_2321,base.br,base.state_br_rs,Ibirubá
city_br_2322,base.br,base.state_br_ba,Ibitiara
city_br_2323,base.br,base.state_br_sp,Ibitinga
city_br_2324,base.br,base.state_br_es,Ibitirama
city_br_2325,base.br,base.state_br_ba,Ibititá
city_br_2326,base.br,base.state_br_mg,Ibitiúra de Minas
city_br_2327,base.br,base.state_br_mg,Ibituruna
city_br_2328,base.br,base.state_br_sp,Ibiúna
city_br_2329,base.br,base.state_br_ba,Ibotirama
city_br_2330,base.br,base.state_br_ce,Icapuí
city_br_2331,base.br,base.state_br_sc,Içara
city_br_2332,base.br,base.state_br_mg,Icaraí de Minas
city_br_2333,base.br,base.state_br_pr,Icaraíma
city_br_2334,base.br,base.state_br_ma,Icatu
city_br_2335,base.br,base.state_br_sp,Icém
city_br_2336,base.br,base.state_br_ba,Ichu
city_br_2337,base.br,base.state_br_ce,Icó
city_br_2338,base.br,base.state_br_es,Iconha
city_br_2339,base.br,base.state_br_rn,Ielmo Marinho
city_br_2340,base.br,base.state_br_sp,Iepê
city_br_2341,base.br,base.state_br_al,Igaci
city_br_2342,base.br,base.state_br_ba,Igaporã
city_br_2343,base.br,base.state_br_sp,Igaraçu do Tietê
city_br_2344,base.br,base.state_br_pb,Igaracy
city_br_2345,base.br,base.state_br_sp,Igarapava
city_br_2346,base.br,base.state_br_mg,Igarapé
city_br_2347,base.br,base.state_br_pa,Igarapé-Açu
city_br_2348,base.br,base.state_br_ma,Igarapé do Meio
city_br_2349,base.br,base.state_br_ma,Igarapé Grande
city_br_2350,base.br,base.state_br_pa,Igarapé-Miri
city_br_2351,base.br,base.state_br_sp,Igaratá
city_br_2352,base.br,base.state_br_mg,Igaratinga
city_br_2353,base.br,base.state_br_ba,Igrapiúna
city_br_2354,base.br,base.state_br_al,Igreja Nova
city_br_2355,base.br,base.state_br_rs,Igrejinha
city_br_2356,base.br,base.state_br_rj,Iguaba Grande
city_br_2357,base.br,base.state_br_ba,Iguaí
city_br_2358,base.br,base.state_br_sp,Iguape
city_br_2359,base.br,base.state_br_pr,Iguaraçu
city_br_2360,base.br,base.state_br_pe,Iguaracy
city_br_2361,base.br,base.state_br_mg,Iguatama
city_br_2362,base.br,base.state_br_ms,Iguatemi
city_br_2363,base.br,base.state_br_ce,Iguatu
city_br_2364,base.br,base.state_br_pr,Iguatu
city_br_2365,base.br,base.state_br_mg,Ijaci
city_br_2366,base.br,base.state_br_rs,Ijuí
city_br_2367,base.br,base.state_br_sp,Ilha Comprida
city_br_2368,base.br,base.state_br_se,Ilha das Flores
city_br_2369,base.br,base.state_br_pe,Ilha de Itamaracá
city_br_2370,base.br,base.state_br_pi,Ilha Grande
city_br_2371,base.br,base.state_br_sp,Ilha Solteira
city_br_2372,base.br,base.state_br_sp,Ilhabela
city_br_2373,base.br,base.state_br_sc,Ilhota
city_br_2374,base.br,base.state_br_mg,Ilicínea
city_br_2375,base.br,base.state_br_rs,Ilópolis
city_br_2376,base.br,base.state_br_pb,Imaculada
city_br_2377,base.br,base.state_br_sc,Imaruí
city_br_2378,base.br,base.state_br_pr,Imbaú
city_br_2379,base.br,base.state_br_rs,Imbé
city_br_2380,base.br,base.state_br_mg,Imbé de Minas
city_br_2381,base.br,base.state_br_sc,Imbituba
city_br_2382,base.br,base.state_br_pr,Imbituva
city_br_2383,base.br,base.state_br_sc,Imbuia
city_br_2384,base.br,base.state_br_rs,Imigrante
city_br_2385,base.br,base.state_br_pr,Inácio Martins
city_br_2386,base.br,base.state_br_go,Inaciolândia
city_br_2387,base.br,base.state_br_pe,Inajá
city_br_2388,base.br,base.state_br_pr,Inajá
city_br_2389,base.br,base.state_br_mg,Inconfidentes
city_br_2390,base.br,base.state_br_mg,Indaiabira
city_br_2391,base.br,base.state_br_sc,Indaial
city_br_2392,base.br,base.state_br_ce,Independência
city_br_2393,base.br,base.state_br_rs,Independência
city_br_2394,base.br,base.state_br_sp,Indiana
city_br_2395,base.br,base.state_br_mg,Indianópolis
city_br_2396,base.br,base.state_br_pr,Indianópolis
city_br_2397,base.br,base.state_br_sp,Indiaporã
city_br_2398,base.br,base.state_br_go,Indiara
city_br_2399,base.br,base.state_br_se,Indiaroba
city_br_2400,base.br,base.state_br_mt,Indiavaí
city_br_2401,base.br,base.state_br_pb,Ingá
city_br_2402,base.br,base.state_br_mg,Ingaí
city_br_2403,base.br,base.state_br_pe,Ingazeira
city_br_2404,base.br,base.state_br_rs,Inhacorá
city_br_2405,base.br,base.state_br_ba,Inhambupe
city_br_2406,base.br,base.state_br_pa,Inhangapi
city_br_2407,base.br,base.state_br_al,Inhapi
city_br_2408,base.br,base.state_br_mg,Inhapim
city_br_2409,base.br,base.state_br_mg,Inhaúma
city_br_2410,base.br,base.state_br_pi,Inhuma
city_br_2411,base.br,base.state_br_go,Inhumas
city_br_2412,base.br,base.state_br_mg,Inimutaba
city_br_2413,base.br,base.state_br_ms,Inocência
city_br_2414,base.br,base.state_br_sp,Inúbia Paulista
city_br_2415,base.br,base.state_br_sc,Iomerê
city_br_2416,base.br,base.state_br_mg,Ipaba
city_br_2417,base.br,base.state_br_go,Ipameri
city_br_2418,base.br,base.state_br_mg,Ipanema
city_br_2419,base.br,base.state_br_rn,Ipanguaçu
city_br_2420,base.br,base.state_br_ce,Ipaporanga
city_br_2421,base.br,base.state_br_ce,Ipaumirim
city_br_2422,base.br,base.state_br_sp,Ipaussu
city_br_2423,base.br,base.state_br_rs,Ipê
city_br_2424,base.br,base.state_br_ba,Ipecaetá
city_br_2425,base.br,base.state_br_sp,Iperó
city_br_2426,base.br,base.state_br_sp,Ipeúna
city_br_2427,base.br,base.state_br_mg,Ipiaçu
city_br_2428,base.br,base.state_br_ba,Ipiaú
city_br_2429,base.br,base.state_br_sp,Ipiguá
city_br_2430,base.br,base.state_br_sc,Ipira
city_br_2431,base.br,base.state_br_ba,Ipirá
city_br_2432,base.br,base.state_br_pr,Ipiranga
city_br_2433,base.br,base.state_br_go,Ipiranga de Goiás
city_br_2434,base.br,base.state_br_mt,Ipiranga do Norte
city_br_2435,base.br,base.state_br_pi,Ipiranga do Piauí
city_br_2436,base.br,base.state_br_rs,Ipiranga do Sul
city_br_2437,base.br,base.state_br_am,Ipixuna
city_br_2438,base.br,base.state_br_pa,Ipixuna do Pará
city_br_2439,base.br,base.state_br_pe,Ipojuca
city_br_2440,base.br,base.state_br_go,Iporá
city_br_2441,base.br,base.state_br_pr,Iporã
city_br_2442,base.br,base.state_br_sc,Iporã do Oeste
city_br_2443,base.br,base.state_br_sp,Iporanga
city_br_2444,base.br,base.state_br_ce,Ipu
city_br_2445,base.br,base.state_br_sp,Ipuã
city_br_2446,base.br,base.state_br_sc,Ipuaçu
city_br_2447,base.br,base.state_br_pe,Ipubi
city_br_2448,base.br,base.state_br_rn,Ipueira
city_br_2449,base.br,base.state_br_ce,Ipueiras
city_br_2450,base.br,base.state_br_to,Ipueiras
city_br_2451,base.br,base.state_br_mg,Ipuiúna
city_br_2452,base.br,base.state_br_sc,Ipumirim
city_br_2453,base.br,base.state_br_ba,Ipupiara
city_br_2454,base.br,base.state_br_ce,Iracema
city_br_2455,base.br,base.state_br_rr,Iracema
city_br_2456,base.br,base.state_br_pr,Iracema do Oeste
city_br_2457,base.br,base.state_br_sp,Iracemápolis
city_br_2458,base.br,base.state_br_sc,Iraceminha
city_br_2459,base.br,base.state_br_rs,Iraí
city_br_2460,base.br,base.state_br_mg,Iraí de Minas
city_br_2461,base.br,base.state_br_ba,Irajuba
city_br_2462,base.br,base.state_br_ba,Iramaia
city_br_2463,base.br,base.state_br_am,Iranduba
city_br_2464,base.br,base.state_br_sc,Irani
city_br_2465,base.br,base.state_br_sp,Irapuã
city_br_2466,base.br,base.state_br_sp,Irapuru
city_br_2467,base.br,base.state_br_ba,Iraquara
city_br_2468,base.br,base.state_br_ba,Irará
city_br_2469,base.br,base.state_br_pr,Irati
city_br_2470,base.br,base.state_br_sc,Irati
city_br_2471,base.br,base.state_br_ce,Irauçuba
city_br_2472,base.br,base.state_br_ba,Irecê
city_br_2473,base.br,base.state_br_pr,Iretama
city_br_2474,base.br,base.state_br_sc,Irineópolis
city_br_2475,base.br,base.state_br_pa,Irituia
city_br_2476,base.br,base.state_br_es,Irupi
city_br_2477,base.br,base.state_br_pi,Isaías Coelho
city_br_2478,base.br,base.state_br_go,Israelândia
city_br_2479,base.br,base.state_br_sc,Itá
city_br_2480,base.br,base.state_br_rs,Itaara
city_br_2481,base.br,base.state_br_pb,Itabaiana
city_br_2482,base.br,base.state_br_se,Itabaianinha
city_br_2483,base.br,base.state_br_ba,Itabela
city_br_2484,base.br,base.state_br_sp,Itaberá
city_br_2485,base.br,base.state_br_ba,Itaberaba
city_br_2486,base.br,base.state_br_go,Itaberaí
city_br_2487,base.br,base.state_br_se,Itabi
city_br_2488,base.br,base.state_br_mg,Itabirinha
city_br_2489,base.br,base.state_br_mg,Itabirito
city_br_2490,base.br,base.state_br_to,Itacajá
city_br_2491,base.br,base.state_br_mg,Itacambira
city_br_2492,base.br,base.state_br_mg,Itacarambi
city_br_2493,base.br,base.state_br_ba,Itacaré
city_br_2494,base.br,base.state_br_pe,Itacuruba
city_br_2495,base.br,base.state_br_rs,Itacurubi
city_br_2496,base.br,base.state_br_ba,Itaeté
city_br_2497,base.br,base.state_br_ba,Itagi
city_br_2498,base.br,base.state_br_ba,Itagibá
city_br_2499,base.br,base.state_br_ba,Itagimirim
city_br_2500,base.br,base.state_br_es,Itaguaçu
city_br_2501,base.br,base.state_br_ba,Itaguaçu da Bahia
city_br_2502,base.br,base.state_br_pr,Itaguajé
city_br_2503,base.br,base.state_br_mg,Itaguara
city_br_2504,base.br,base.state_br_go,Itaguari
city_br_2505,base.br,base.state_br_go,Itaguaru
city_br_2506,base.br,base.state_br_to,Itaguatins
city_br_2507,base.br,base.state_br_sp,Itaí
city_br_2508,base.br,base.state_br_pe,Itaíba
city_br_2509,base.br,base.state_br_ce,Itaiçaba
city_br_2510,base.br,base.state_br_pi,Itainópolis
city_br_2511,base.br,base.state_br_sc,Itaiópolis
city_br_2512,base.br,base.state_br_ma,Itaipava do Grajaú
city_br_2513,base.br,base.state_br_mg,Itaipé
city_br_2514,base.br,base.state_br_pr,Itaipulândia
city_br_2515,base.br,base.state_br_ce,Itaitinga
city_br_2516,base.br,base.state_br_go,Itajá
city_br_2517,base.br,base.state_br_rn,Itajá
city_br_2518,base.br,base.state_br_sp,Itajobi
city_br_2519,base.br,base.state_br_sp,Itaju
city_br_2520,base.br,base.state_br_ba,Itaju do Colônia
city_br_2521,base.br,base.state_br_mg,Itajubá
city_br_2522,base.br,base.state_br_ba,Itajuípe
city_br_2523,base.br,base.state_br_rj,Italva
city_br_2524,base.br,base.state_br_ba,Itamaraju
city_br_2525,base.br,base.state_br_mg,Itamarandiba
city_br_2526,base.br,base.state_br_am,Itamarati
city_br_2527,base.br,base.state_br_mg,Itamarati de Minas
city_br_2528,base.br,base.state_br_ba,Itamari
city_br_2529,base.br,base.state_br_mg,Itambacuri
city_br_2530,base.br,base.state_br_pr,Itambaracá
city_br_2531,base.br,base.state_br_ba,Itambé
city_br_2532,base.br,base.state_br_pe,Itambé
city_br_2533,base.br,base.state_br_pr,Itambé
city_br_2534,base.br,base.state_br_mg,Itambé do Mato Dentro
city_br_2535,base.br,base.state_br_mg,Itamogi
city_br_2536,base.br,base.state_br_mg,Itamonte
city_br_2537,base.br,base.state_br_ba,Itanagra
city_br_2538,base.br,base.state_br_mg,Itanhandu
city_br_2539,base.br,base.state_br_mt,Itanhangá
city_br_2540,base.br,base.state_br_ba,Itanhém
city_br_2541,base.br,base.state_br_mg,Itanhomi
city_br_2542,base.br,base.state_br_mg,Itaobim
city_br_2543,base.br,base.state_br_sp,Itaóca
city_br_2544,base.br,base.state_br_rj,Itaocara
city_br_2545,base.br,base.state_br_go,Itapaci
city_br_2546,base.br,base.state_br_mg,Itapagipe
city_br_2547,base.br,base.state_br_ce,Itapajé
city_br_2548,base.br,base.state_br_ba,Itaparica
city_br_2549,base.br,base.state_br_ba,Itapé
city_br_2550,base.br,base.state_br_ba,Itapebi
city_br_2551,base.br,base.state_br_mg,Itapecerica
city_br_2552,base.br,base.state_br_ma,Itapecuru Mirim
city_br_2553,base.br,base.state_br_pr,Itapejara d'Oeste
city_br_2554,base.br,base.state_br_sc,Itapema
city_br_2555,base.br,base.state_br_es,Itapemirim
city_br_2556,base.br,base.state_br_pr,Itaperuçu
city_br_2557,base.br,base.state_br_pe,Itapetim
city_br_2558,base.br,base.state_br_ba,Itapetinga
city_br_2559,base.br,base.state_br_mg,Itapeva
city_br_2560,base.br,base.state_br_sp,Itapeva
city_br_2561,base.br,base.state_br_ba,Itapicuru
city_br_2562,base.br,base.state_br_sp,Itapira
city_br_2563,base.br,base.state_br_am,Itapiranga
city_br_2564,base.br,base.state_br_sc,Itapiranga
city_br_2565,base.br,base.state_br_go,Itapirapuã
city_br_2566,base.br,base.state_br_sp,Itapirapuã Paulista
city_br_2567,base.br,base.state_br_to,Itapiratins
city_br_2568,base.br,base.state_br_pe,Itapissuma
city_br_2569,base.br,base.state_br_ba,Itapitanga
city_br_2570,base.br,base.state_br_ce,Itapiúna
city_br_2571,base.br,base.state_br_sc,Itapoá
city_br_2572,base.br,base.state_br_sp,Itápolis
city_br_2573,base.br,base.state_br_ms,Itaporã
city_br_2574,base.br,base.state_br_to,Itaporã do Tocantins
city_br_2575,base.br,base.state_br_pb,Itaporanga
city_br_2576,base.br,base.state_br_sp,Itaporanga
city_br_2577,base.br,base.state_br_se,Itaporanga d'Ajuda
city_br_2578,base.br,base.state_br_pb,Itapororoca
city_br_2579,base.br,base.state_br_ro,Itapuã do Oeste
city_br_2580,base.br,base.state_br_rs,Itapuca
city_br_2581,base.br,base.state_br_sp,Itapuí
city_br_2582,base.br,base.state_br_sp,Itapura
city_br_2583,base.br,base.state_br_go,Itapuranga
city_br_2584,base.br,base.state_br_ba,Itaquara
city_br_2585,base.br,base.state_br_rs,Itaqui
city_br_2586,base.br,base.state_br_ms,Itaquiraí
city_br_2587,base.br,base.state_br_pe,Itaquitinga
city_br_2588,base.br,base.state_br_es,Itarana
city_br_2589,base.br,base.state_br_ba,Itarantim
city_br_2590,base.br,base.state_br_sp,Itararé
city_br_2591,base.br,base.state_br_ce,Itarema
city_br_2592,base.br,base.state_br_sp,Itariri
city_br_2593,base.br,base.state_br_go,Itarumã
city_br_2594,base.br,base.state_br_rs,Itati
city_br_2595,base.br,base.state_br_rj,Itatiaia
city_br_2596,base.br,base.state_br_mg,Itatiaiuçu
city_br_2597,base.br,base.state_br_rs,Itatiba do Sul
city_br_2598,base.br,base.state_br_ba,Itatim
city_br_2599,base.br,base.state_br_sp,Itatinga
city_br_2600,base.br,base.state_br_ce,Itatira
city_br_2601,base.br,base.state_br_pb,Itatuba
city_br_2602,base.br,base.state_br_rn,Itaú
city_br_2603,base.br,base.state_br_mg,Itaú de Minas
city_br_2604,base.br,base.state_br_mt,Itaúba
city_br_2605,base.br,base.state_br_ap,Itaubal
city_br_2606,base.br,base.state_br_go,Itauçu
city_br_2607,base.br,base.state_br_pi,Itaueira
city_br_2608,base.br,base.state_br_mg,Itaúna
city_br_2609,base.br,base.state_br_pr,Itaúna do Sul
city_br_2610,base.br,base.state_br_mg,Itaverava
city_br_2611,base.br,base.state_br_mg,Itinga
city_br_2612,base.br,base.state_br_ma,Itinga do Maranhão
city_br_2613,base.br,base.state_br_mt,Itiquira
city_br_2614,base.br,base.state_br_sp,Itirapina
city_br_2615,base.br,base.state_br_sp,Itirapuã
city_br_2616,base.br,base.state_br_ba,Itiruçu
city_br_2617,base.br,base.state_br_ba,Itiúba
city_br_2618,base.br,base.state_br_sp,Itobi
city_br_2619,base.br,base.state_br_ba,Itororó
city_br_2620,base.br,base.state_br_ba,Ituaçu
city_br_2621,base.br,base.state_br_ba,Ituberá
city_br_2622,base.br,base.state_br_mg,Itueta
city_br_2623,base.br,base.state_br_mg,Itumirim
city_br_2624,base.br,base.state_br_sp,Itupeva
city_br_2625,base.br,base.state_br_pa,Itupiranga
city_br_2626,base.br,base.state_br_sc,Ituporanga
city_br_2627,base.br,base.state_br_mg,Iturama
city_br_2628,base.br,base.state_br_mg,Itutinga
city_br_2629,base.br,base.state_br_sp,Ituverava
city_br_2630,base.br,base.state_br_ba,Iuiú
city_br_2631,base.br,base.state_br_es,Iúna
city_br_2632,base.br,base.state_br_pr,Ivaí
city_br_2633,base.br,base.state_br_pr,Ivaiporã
city_br_2634,base.br,base.state_br_pr,Ivaté
city_br_2635,base.br,base.state_br_pr,Ivatuba
city_br_2636,base.br,base.state_br_ms,Ivinhema
city_br_2637,base.br,base.state_br_go,Ivolândia
city_br_2638,base.br,base.state_br_rs,Ivorá
city_br_2639,base.br,base.state_br_rs,Ivoti
city_br_2640,base.br,base.state_br_sc,Jaborá
city_br_2641,base.br,base.state_br_ba,Jaborandi
city_br_2642,base.br,base.state_br_sp,Jaborandi
city_br_2643,base.br,base.state_br_pr,Jaboti
city_br_2644,base.br,base.state_br_rs,Jaboticaba
city_br_2645,base.br,base.state_br_sp,Jaboticabal
city_br_2646,base.br,base.state_br_mg,Jaboticatubas
city_br_2647,base.br,base.state_br_rn,Jaçanã
city_br_2648,base.br,base.state_br_ba,Jacaraci
city_br_2649,base.br,base.state_br_pb,Jacaraú
city_br_2650,base.br,base.state_br_al,Jacaré dos Homens
city_br_2651,base.br,base.state_br_pa,Jacareacanga
city_br_2652,base.br,base.state_br_pr,Jacarezinho
city_br_2653,base.br,base.state_br_sp,Jaci
city_br_2654,base.br,base.state_br_mt,Jaciara
city_br_2655,base.br,base.state_br_mg,Jacinto
city_br_2656,base.br,base.state_br_sc,Jacinto Machado
city_br_2657,base.br,base.state_br_ba,Jacobina
city_br_2658,base.br,base.state_br_pi,Jacobina do Piauí
city_br_2659,base.br,base.state_br_mg,Jacuí
city_br_2660,base.br,base.state_br_al,Jacuípe
city_br_2661,base.br,base.state_br_rs,Jacuizinho
city_br_2662,base.br,base.state_br_pa,Jacundá
city_br_2663,base.br,base.state_br_sp,Jacupiranga
city_br_2664,base.br,base.state_br_mg,Jacutinga
city_br_2665,base.br,base.state_br_rs,Jacutinga
city_br_2666,base.br,base.state_br_pr,Jaguapitã
city_br_2667,base.br,base.state_br_ba,Jaguaquara
city_br_2668,base.br,base.state_br_mg,Jaguaraçu
city_br_2669,base.br,base.state_br_rs,Jaguarão
city_br_2670,base.br,base.state_br_ba,Jaguarari
city_br_2671,base.br,base.state_br_es,Jaguaré
city_br_2672,base.br,base.state_br_ce,Jaguaretama
city_br_2673,base.br,base.state_br_rs,Jaguari
city_br_2674,base.br,base.state_br_pr,Jaguariaíva
city_br_2675,base.br,base.state_br_ce,Jaguaribara
city_br_2676,base.br,base.state_br_ce,Jaguaribe
city_br_2677,base.br,base.state_br_ba,Jaguaripe
city_br_2678,base.br,base.state_br_sp,Jaguariúna
city_br_2679,base.br,base.state_br_ce,Jaguaruana
city_br_2680,base.br,base.state_br_sc,Jaguaruna
city_br_2681,base.br,base.state_br_mg,Jaíba
city_br_2682,base.br,base.state_br_pi,Jaicós
city_br_2683,base.br,base.state_br_sp,Jales
city_br_2684,base.br,base.state_br_sp,Jambeiro
city_br_2685,base.br,base.state_br_mg,Jampruca
city_br_2686,base.br,base.state_br_mg,Janaúba
city_br_2687,base.br,base.state_br_go,Jandaia
city_br_2688,base.br,base.state_br_pr,Jandaia do Sul
city_br_2689,base.br,base.state_br_ba,Jandaíra
city_br_2690,base.br,base.state_br_rn,Jandaíra
city_br_2691,base.br,base.state_br_rn,Janduís
city_br_2692,base.br,base.state_br_mt,Jangada
city_br_2693,base.br,base.state_br_pr,Janiópolis
city_br_2694,base.br,base.state_br_mg,Januária
city_br_2695,base.br,base.state_br_mg,Japaraíba
city_br_2696,base.br,base.state_br_al,Japaratinga
city_br_2697,base.br,base.state_br_se,Japaratuba
city_br_2698,base.br,base.state_br_rj,Japeri
city_br_2699,base.br,base.state_br_rn,Japi
city_br_2700,base.br,base.state_br_pr,Japira
city_br_2701,base.br,base.state_br_se,Japoatã
city_br_2702,base.br,base.state_br_mg,Japonvar
city_br_2703,base.br,base.state_br_ms,Japorã
city_br_2704,base.br,base.state_br_am,Japurá
city_br_2705,base.br,base.state_br_pr,Japurá
city_br_2706,base.br,base.state_br_pe,Jaqueira
city_br_2707,base.br,base.state_br_rs,Jaquirana
city_br_2708,base.br,base.state_br_go,Jaraguá
city_br_2709,base.br,base.state_br_ms,Jaraguari
city_br_2710,base.br,base.state_br_al,Jaramataia
city_br_2711,base.br,base.state_br_ce,Jardim
city_br_2712,base.br,base.state_br_ms,Jardim
city_br_2713,base.br,base.state_br_pr,Jardim Alegre
city_br_2714,base.br,base.state_br_rn,Jardim de Angicos
city_br_2715,base.br,base.state_br_rn,Jardim de Piranhas
city_br_2716,base.br,base.state_br_pi,Jardim do Mulato
city_br_2717,base.br,base.state_br_rn,Jardim do Seridó
city_br_2718,base.br,base.state_br_pr,Jardim Olinda
city_br_2719,base.br,base.state_br_sc,Jardinópolis
city_br_2720,base.br,base.state_br_sp,Jardinópolis
city_br_2721,base.br,base.state_br_rs,Jari
city_br_2722,base.br,base.state_br_sp,Jarinu
city_br_2723,base.br,base.state_br_ro,Jaru
city_br_2724,base.br,base.state_br_pr,Jataizinho
city_br_2725,base.br,base.state_br_pe,Jataúba
city_br_2726,base.br,base.state_br_ms,Jateí
city_br_2727,base.br,base.state_br_ce,Jati
city_br_2728,base.br,base.state_br_ma,Jatobá
city_br_2729,base.br,base.state_br_pe,Jatobá
city_br_2730,base.br,base.state_br_pi,Jatobá do Piauí
city_br_2731,base.br,base.state_br_to,Jaú do Tocantins
city_br_2732,base.br,base.state_br_go,Jaupaci
city_br_2733,base.br,base.state_br_mt,Jauru
city_br_2734,base.br,base.state_br_mg,Jeceaba
city_br_2735,base.br,base.state_br_mg,Jenipapo de Minas
city_br_2736,base.br,base.state_br_ma,Jenipapo dos Vieiras
city_br_2737,base.br,base.state_br_mg,Jequeri
city_br_2738,base.br,base.state_br_al,Jequiá da Praia
city_br_2739,base.br,base.state_br_mg,Jequitaí
city_br_2740,base.br,base.state_br_mg,Jequitibá
city_br_2741,base.br,base.state_br_mg,Jequitinhonha
city_br_2742,base.br,base.state_br_ba,Jeremoabo
city_br_2743,base.br,base.state_br_pb,Jericó
city_br_2744,base.br,base.state_br_sp,Jeriquara
city_br_2745,base.br,base.state_br_es,Jerônimo Monteiro
city_br_2746,base.br,base.state_br_pi,Jerumenha
city_br_2747,base.br,base.state_br_mg,Jesuânia
city_br_2748,base.br,base.state_br_pr,Jesuítas
city_br_2749,base.br,base.state_br_go,Jesúpolis
city_br_2750,base.br,base.state_br_ce,Jijoca de Jericoacoara
city_br_2751,base.br,base.state_br_ba,Jiquiriçá
city_br_2752,base.br,base.state_br_ba,Jitaúna
city_br_2753,base.br,base.state_br_sc,Joaçaba
city_br_2754,base.br,base.state_br_mg,Joaíma
city_br_2755,base.br,base.state_br_mg,Joanésia
city_br_2756,base.br,base.state_br_sp,Joanópolis
city_br_2757,base.br,base.state_br_pe,João Alfredo
city_br_2758,base.br,base.state_br_rn,João Câmara
city_br_2759,base.br,base.state_br_pi,João Costa
city_br_2760,base.br,base.state_br_rn,João Dias
city_br_2761,base.br,base.state_br_ba,João Dourado
city_br_2762,base.br,base.state_br_ma,João Lisboa
city_br_2763,base.br,base.state_br_mg,João Monlevade
city_br_2764,base.br,base.state_br_es,João Neiva
city_br_2765,base.br,base.state_br_mg,João Pinheiro
city_br_2766,base.br,base.state_br_sp,João Ramalho
city_br_2767,base.br,base.state_br_mg,Joaquim Felício
city_br_2768,base.br,base.state_br_al,Joaquim Gomes
city_br_2769,base.br,base.state_br_pe,Joaquim Nabuco
city_br_2770,base.br,base.state_br_pi,Joaquim Pires
city_br_2771,base.br,base.state_br_pr,Joaquim Távora
city_br_2772,base.br,base.state_br_pb,Joca Claudino
city_br_2773,base.br,base.state_br_pi,Joca Marques
city_br_2774,base.br,base.state_br_rs,Jóia
city_br_2775,base.br,base.state_br_mg,Jordânia
city_br_2776,base.br,base.state_br_ac,Jordão
city_br_2777,base.br,base.state_br_sc,José Boiteux
city_br_2778,base.br,base.state_br_sp,José Bonifácio
city_br_2779,base.br,base.state_br_rn,José da Penha
city_br_2780,base.br,base.state_br_pi,José de Freitas
city_br_2781,base.br,base.state_br_mg,José Gonçalves de Minas
city_br_2782,base.br,base.state_br_mg,José Raydan
city_br_2783,base.br,base.state_br_ma,Joselândia
city_br_2784,base.br,base.state_br_mg,Josenópolis
city_br_2785,base.br,base.state_br_go,Joviânia
city_br_2786,base.br,base.state_br_mt,Juara
city_br_2787,base.br,base.state_br_pb,Juarez Távora
city_br_2788,base.br,base.state_br_to,Juarina
city_br_2789,base.br,base.state_br_mg,Juatuba
city_br_2790,base.br,base.state_br_pb,Juazeirinho
city_br_2791,base.br,base.state_br_pi,Juazeiro do Piauí
city_br_2792,base.br,base.state_br_ce,Jucás
city_br_2793,base.br,base.state_br_pe,Jucati
city_br_2794,base.br,base.state_br_ba,Jucuruçu
city_br_2795,base.br,base.state_br_rn,Jucurutu
city_br_2796,base.br,base.state_br_mt,Juína
city_br_2797,base.br,base.state_br_pi,Júlio Borges
city_br_2798,base.br,base.state_br_rs,Júlio de Castilhos
city_br_2799,base.br,base.state_br_sp,Júlio Mesquita
city_br_2800,base.br,base.state_br_sp,Jumirim
city_br_2801,base.br,base.state_br_ma,Junco do Maranhão
city_br_2802,base.br,base.state_br_pb,Junco do Seridó
city_br_2803,base.br,base.state_br_al,Jundiá
city_br_2804,base.br,base.state_br_rn,Jundiá
city_br_2805,base.br,base.state_br_pr,Jundiaí do Sul
city_br_2806,base.br,base.state_br_al,Junqueiro
city_br_2807,base.br,base.state_br_sp,Junqueirópolis
city_br_2808,base.br,base.state_br_pe,Jupi
city_br_2809,base.br,base.state_br_sc,Jupiá
city_br_2810,base.br,base.state_br_sp,Juquiá
city_br_2811,base.br,base.state_br_sp,Juquitiba
city_br_2812,base.br,base.state_br_mg,Juramento
city_br_2813,base.br,base.state_br_pr,Juranda
city_br_2814,base.br,base.state_br_pe,Jurema
city_br_2815,base.br,base.state_br_pi,Jurema
city_br_2816,base.br,base.state_br_pb,Juripiranga
city_br_2817,base.br,base.state_br_pb,Juru
city_br_2818,base.br,base.state_br_am,Juruá
city_br_2819,base.br,base.state_br_mg,Juruaia
city_br_2820,base.br,base.state_br_mt,Juruena
city_br_2821,base.br,base.state_br_pa,Juruti
city_br_2822,base.br,base.state_br_mt,Juscimeira
city_br_2823,base.br,base.state_br_ba,Jussara
city_br_2824,base.br,base.state_br_go,Jussara
city_br_2825,base.br,base.state_br_pr,Jussara
city_br_2826,base.br,base.state_br_ba,Jussari
city_br_2827,base.br,base.state_br_ba,Jussiape
city_br_2828,base.br,base.state_br_am,Jutaí
city_br_2829,base.br,base.state_br_ms,Juti
city_br_2830,base.br,base.state_br_mg,Juvenília
city_br_2831,base.br,base.state_br_pr,Kaloré
city_br_2832,base.br,base.state_br_am,Lábrea
city_br_2833,base.br,base.state_br_sc,Lacerdópolis
city_br_2834,base.br,base.state_br_mg,Ladainha
city_br_2835,base.br,base.state_br_ms,Ladário
city_br_2836,base.br,base.state_br_ba,Lafaiete Coutinho
city_br_2837,base.br,base.state_br_mg,Lagamar
city_br_2838,base.br,base.state_br_ma,Lago da Pedra
city_br_2839,base.br,base.state_br_ma,Lago do Junco
city_br_2840,base.br,base.state_br_ma,Lago dos Rodrigues
city_br_2841,base.br,base.state_br_ma,Lago Verde
city_br_2842,base.br,base.state_br_pb,Lagoa
city_br_2843,base.br,base.state_br_pi,Lagoa Alegre
city_br_2844,base.br,base.state_br_rs,Lagoa Bonita do Sul
city_br_2845,base.br,base.state_br_rn,Lagoa d'Anta
city_br_2846,base.br,base.state_br_al,Lagoa da Canoa
city_br_2847,base.br,base.state_br_to,Lagoa da Confusão
city_br_2848,base.br,base.state_br_mg,Lagoa da Prata
city_br_2849,base.br,base.state_br_pb,Lagoa de Dentro
city_br_2850,base.br,base.state_br_pe,Lagoa de Itaenga
city_br_2851,base.br,base.state_br_rn,Lagoa de Pedras
city_br_2852,base.br,base.state_br_pi,Lagoa de São Francisco
city_br_2853,base.br,base.state_br_rn,Lagoa de Velhos
city_br_2854,base.br,base.state_br_pi,Lagoa do Barro do Piauí
city_br_2855,base.br,base.state_br_pe,Lagoa do Carro
city_br_2856,base.br,base.state_br_ma,Lagoa do Mato
city_br_2857,base.br,base.state_br_pe,Lagoa do Ouro
city_br_2858,base.br,base.state_br_pi,Lagoa do Piauí
city_br_2859,base.br,base.state_br_pi,Lagoa do Sítio
city_br_2860,base.br,base.state_br_to,Lagoa do Tocantins
city_br_2861,base.br,base.state_br_pe,Lagoa dos Gatos
city_br_2862,base.br,base.state_br_mg,Lagoa dos Patos
city_br_2863,base.br,base.state_br_rs,Lagoa dos Três Cantos
city_br_2864,base.br,base.state_br_mg,Lagoa Dourada
city_br_2865,base.br,base.state_br_mg,Lagoa Formosa
city_br_2866,base.br,base.state_br_mg,Lagoa Grande
city_br_2867,base.br,base.state_br_pe,Lagoa Grande
city_br_2868,base.br,base.state_br_ma,Lagoa Grande do Maranhão
city_br_2869,base.br,base.state_br_rn,Lagoa Nova
city_br_2870,base.br,base.state_br_ba,Lagoa Real
city_br_2871,base.br,base.state_br_rn,Lagoa Salgada
city_br_2872,base.br,base.state_br_go,Lagoa Santa
city_br_2873,base.br,base.state_br_mg,Lagoa Santa
city_br_2874,base.br,base.state_br_pb,Lagoa Seca
city_br_2875,base.br,base.state_br_rs,Lagoa Vermelha
city_br_2876,base.br,base.state_br_rs,Lagoão
city_br_2877,base.br,base.state_br_sp,Lagoinha
city_br_2878,base.br,base.state_br_pi,Lagoinha do Piauí
city_br_2879,base.br,base.state_br_sc,Laguna
city_br_2880,base.br,base.state_br_ms,Laguna Carapã
city_br_2881,base.br,base.state_br_ba,Laje
city_br_2882,base.br,base.state_br_rj,Laje do Muriaé
city_br_2883,base.br,base.state_br_rs,Lajeado
city_br_2884,base.br,base.state_br_to,Lajeado
city_br_2885,base.br,base.state_br_rs,Lajeado do Bugre
city_br_2886,base.br,base.state_br_sc,Lajeado Grande
city_br_2887,base.br,base.state_br_ma,Lajeado Novo
city_br_2888,base.br,base.state_br_ba,Lajedão
city_br_2889,base.br,base.state_br_ba,Lajedinho
city_br_2890,base.br,base.state_br_pe,Lajedo
city_br_2891,base.br,base.state_br_ba,Lajedo do Tabocal
city_br_2892,base.br,base.state_br_rn,Lajes
city_br_2893,base.br,base.state_br_rn,Lajes Pintadas
city_br_2894,base.br,base.state_br_mg,Lajinha
city_br_2895,base.br,base.state_br_ba,Lamarão
city_br_2896,base.br,base.state_br_mg,Lambari
city_br_2897,base.br,base.state_br_mt,Lambari D'Oeste
city_br_2898,base.br,base.state_br_mg,Lamim
city_br_2899,base.br,base.state_br_pi,Landri Sales
city_br_2900,base.br,base.state_br_pr,Lapa
city_br_2901,base.br,base.state_br_ba,Lapão
city_br_2902,base.br,base.state_br_es,Laranja da Terra
city_br_2903,base.br,base.state_br_mg,Laranjal
city_br_2904,base.br,base.state_br_pr,Laranjal
city_br_2905,base.br,base.state_br_ap,Laranjal do Jari
city_br_2906,base.br,base.state_br_sp,Laranjal Paulista
city_br_2907,base.br,base.state_br_se,Laranjeiras
city_br_2908,base.br,base.state_br_pr,Laranjeiras do Sul
city_br_2909,base.br,base.state_br_mg,Lassance
city_br_2910,base.br,base.state_br_pb,Lastro
city_br_2911,base.br,base.state_br_sc,Laurentino
city_br_2912,base.br,base.state_br_sc,Lauro Müller
city_br_2913,base.br,base.state_br_to,Lavandeira
city_br_2914,base.br,base.state_br_sp,Lavínia
city_br_2915,base.br,base.state_br_ce,Lavras da Mangabeira
city_br_2916,base.br,base.state_br_rs,Lavras do Sul
city_br_2917,base.br,base.state_br_sp,Lavrinhas
city_br_2918,base.br,base.state_br_mg,Leandro Ferreira
city_br_2919,base.br,base.state_br_sc,Lebon Régis
city_br_2920,base.br,base.state_br_sp,Leme
city_br_2921,base.br,base.state_br_mg,Leme do Prado
city_br_2922,base.br,base.state_br_ba,Lençóis
city_br_2923,base.br,base.state_br_sp,Lençóis Paulista
city_br_2924,base.br,base.state_br_sc,Leoberto Leal
city_br_2925,base.br,base.state_br_mg,Leopoldina
city_br_2926,base.br,base.state_br_go,Leopoldo de Bulhões
city_br_2927,base.br,base.state_br_pr,Leópolis
city_br_2928,base.br,base.state_br_rs,Liberato Salzano
city_br_2929,base.br,base.state_br_mg,Liberdade
city_br_2930,base.br,base.state_br_ba,Licínio de Almeida
city_br_2931,base.br,base.state_br_pr,Lidianópolis
city_br_2932,base.br,base.state_br_ma,Lima Campos
city_br_2933,base.br,base.state_br_mg,Lima Duarte
city_br_2934,base.br,base.state_br_mg,Limeira do Oeste
city_br_2935,base.br,base.state_br_pe,Limoeiro
city_br_2936,base.br,base.state_br_al,Limoeiro de Anadia
city_br_2937,base.br,base.state_br_pa,Limoeiro do Ajuru
city_br_2938,base.br,base.state_br_ce,Limoeiro do Norte
city_br_2939,base.br,base.state_br_pr,Lindoeste
city_br_2940,base.br,base.state_br_sp,Lindóia
city_br_2941,base.br,base.state_br_sc,Lindóia do Sul
city_br_2942,base.br,base.state_br_rs,Lindolfo Collor
city_br_2943,base.br,base.state_br_rs,Linha Nova
city_br_2944,base.br,base.state_br_sp,Lins
city_br_2945,base.br,base.state_br_pb,Livramento
city_br_2946,base.br,base.state_br_ba,Livramento de Nossa Senhora
city_br_2947,base.br,base.state_br_to,Lizarda
city_br_2948,base.br,base.state_br_pr,Loanda
city_br_2949,base.br,base.state_br_pr,Lobato
city_br_2950,base.br,base.state_br_pb,Logradouro
city_br_2951,base.br,base.state_br_mg,Lontra
city_br_2952,base.br,base.state_br_sc,Lontras
city_br_2953,base.br,base.state_br_sp,Lorena
city_br_2954,base.br,base.state_br_ma,Loreto
city_br_2955,base.br,base.state_br_sp,Lourdes
city_br_2956,base.br,base.state_br_sp,Louveira
city_br_2957,base.br,base.state_br_mt,Lucas do Rio Verde
city_br_2958,base.br,base.state_br_sp,Lucélia
city_br_2959,base.br,base.state_br_pb,Lucena
city_br_2960,base.br,base.state_br_sp,Lucianópolis
city_br_2961,base.br,base.state_br_mt,Luciara
city_br_2962,base.br,base.state_br_rn,Lucrécia
city_br_2963,base.br,base.state_br_sp,Luís Antônio
city_br_2964,base.br,base.state_br_pi,Luís Correia
city_br_2965,base.br,base.state_br_ma,Luís Domingues
city_br_2966,base.br,base.state_br_rn,Luís Gomes
city_br_2967,base.br,base.state_br_mg,Luisburgo
city_br_2968,base.br,base.state_br_mg,Luislândia
city_br_2969,base.br,base.state_br_sc,Luiz Alves
city_br_2970,base.br,base.state_br_pr,Luiziana
city_br_2971,base.br,base.state_br_sp,Luiziânia
city_br_2972,base.br,base.state_br_mg,Luminárias
city_br_2973,base.br,base.state_br_pr,Lunardelli
city_br_2974,base.br,base.state_br_sp,Lupércio
city_br_2975,base.br,base.state_br_pr,Lupionópolis
city_br_2976,base.br,base.state_br_sp,Lutécia
city_br_2977,base.br,base.state_br_mg,Luz
city_br_2978,base.br,base.state_br_sc,Luzerna
city_br_2979,base.br,base.state_br_pi,Luzilândia
city_br_2980,base.br,base.state_br_to,Luzinópolis
city_br_2981,base.br,base.state_br_rn,Macaíba
city_br_2982,base.br,base.state_br_ba,Macajuba
city_br_2983,base.br,base.state_br_rs,Maçambará
city_br_2984,base.br,base.state_br_se,Macambira
city_br_2985,base.br,base.state_br_pe,Macaparana
city_br_2986,base.br,base.state_br_ba,Macarani
city_br_2987,base.br,base.state_br_sp,Macatuba
city_br_2988,base.br,base.state_br_rn,Macau
city_br_2989,base.br,base.state_br_sp,Macaubal
city_br_2990,base.br,base.state_br_ba,Macaúbas
city_br_2991,base.br,base.state_br_sp,Macedônia
city_br_2992,base.br,base.state_br_mg,Machacalis
city_br_2993,base.br,base.state_br_rs,Machadinho
city_br_2994,base.br,base.state_br_ro,Machadinho D'Oeste
city_br_2995,base.br,base.state_br_mg,Machado
city_br_2996,base.br,base.state_br_pe,Machados
city_br_2997,base.br,base.state_br_sc,Macieira
city_br_2998,base.br,base.state_br_rj,Macuco
city_br_2999,base.br,base.state_br_ba,Macururé
city_br_3000,base.br,base.state_br_ce,Madalena
city_br_3001,base.br,base.state_br_pi,Madeiro
city_br_3002,base.br,base.state_br_ba,Madre de Deus
city_br_3003,base.br,base.state_br_mg,Madre de Deus de Minas
city_br_3004,base.br,base.state_br_pb,Mãe d'Água
city_br_3005,base.br,base.state_br_pa,Mãe do Rio
city_br_3006,base.br,base.state_br_ba,Maetinga
city_br_3007,base.br,base.state_br_sc,Mafra
city_br_3008,base.br,base.state_br_pa,Magalhães Barata
city_br_3009,base.br,base.state_br_ma,Magalhães de Almeida
city_br_3010,base.br,base.state_br_sp,Magda
city_br_3011,base.br,base.state_br_ba,Maiquinique
city_br_3012,base.br,base.state_br_ba,Mairi
city_br_3013,base.br,base.state_br_sp,Mairinque
city_br_3014,base.br,base.state_br_sp,Mairiporã
city_br_3015,base.br,base.state_br_go,Mairipotaba
city_br_3016,base.br,base.state_br_sc,Major Gercino
city_br_3017,base.br,base.state_br_al,Major Isidoro
city_br_3018,base.br,base.state_br_rn,Major Sales
city_br_3019,base.br,base.state_br_sc,Major Vieira
city_br_3020,base.br,base.state_br_mg,Malacacheta
city_br_3021,base.br,base.state_br_ba,Malhada
city_br_3022,base.br,base.state_br_ba,Malhada de Pedras
city_br_3023,base.br,base.state_br_se,Malhada dos Bois
city_br_3024,base.br,base.state_br_se,Malhador
city_br_3025,base.br,base.state_br_pr,Mallet
city_br_3026,base.br,base.state_br_pb,Malta
city_br_3027,base.br,base.state_br_pb,Mamanguape
city_br_3028,base.br,base.state_br_go,Mambaí
city_br_3029,base.br,base.state_br_pr,Mamborê
city_br_3030,base.br,base.state_br_mg,Mamonas
city_br_3031,base.br,base.state_br_rs,Mampituba
city_br_3032,base.br,base.state_br_pb,Manaíra
city_br_3033,base.br,base.state_br_am,Manaquiri
city_br_3034,base.br,base.state_br_pe,Manari
city_br_3035,base.br,base.state_br_ac,Mâncio Lima
city_br_3036,base.br,base.state_br_pr,Mandaguaçu
city_br_3037,base.br,base.state_br_pr,Mandaguari
city_br_3038,base.br,base.state_br_pr,Mandirituba
city_br_3039,base.br,base.state_br_sp,Manduri
city_br_3040,base.br,base.state_br_pr,Manfrinópolis
city_br_3041,base.br,base.state_br_mg,Manga
city_br_3042,base.br,base.state_br_rj,Mangaratiba
city_br_3043,base.br,base.state_br_pr,Mangueirinha
city_br_3044,base.br,base.state_br_mg,Manhuaçu
city_br_3045,base.br,base.state_br_mg,Manhumirim
city_br_3046,base.br,base.state_br_am,Manicoré
city_br_3047,base.br,base.state_br_pi,Manoel Emídio
city_br_3048,base.br,base.state_br_pr,Manoel Ribas
city_br_3049,base.br,base.state_br_ac,Manoel Urbano
city_br_3050,base.br,base.state_br_rs,Manoel Viana
city_br_3051,base.br,base.state_br_ba,Manoel Vitorino
city_br_3052,base.br,base.state_br_ba,Mansidão
city_br_3053,base.br,base.state_br_mg,Mantena
city_br_3054,base.br,base.state_br_es,Mantenópolis
city_br_3055,base.br,base.state_br_rs,Maquiné
city_br_3056,base.br,base.state_br_mg,Mar de Espanha
city_br_3057,base.br,base.state_br_al,Mar Vermelho
city_br_3058,base.br,base.state_br_go,Mara Rosa
city_br_3059,base.br,base.state_br_am,Maraã
city_br_3060,base.br,base.state_br_sp,Marabá Paulista
city_br_3061,base.br,base.state_br_ma,Maracaçumé
city_br_3062,base.br,base.state_br_sp,Maracaí
city_br_3063,base.br,base.state_br_sc,Maracajá
city_br_3064,base.br,base.state_br_ms,Maracaju
city_br_3065,base.br,base.state_br_pa,Maracanã
city_br_3066,base.br,base.state_br_ba,Maracás
city_br_3067,base.br,base.state_br_al,Maragogi
city_br_3068,base.br,base.state_br_ba,Maragogipe
city_br_3069,base.br,base.state_br_pe,Maraial
city_br_3070,base.br,base.state_br_ma,Marajá do Sena
city_br_3071,base.br,base.state_br_ma,Maranhãozinho
city_br_3072,base.br,base.state_br_pa,Marapanim
city_br_3073,base.br,base.state_br_sp,Marapoama
city_br_3074,base.br,base.state_br_rs,Maratá
city_br_3075,base.br,base.state_br_es,Marataízes
city_br_3076,base.br,base.state_br_rs,Marau
city_br_3077,base.br,base.state_br_ba,Maraú
city_br_3078,base.br,base.state_br_al,Maravilha
city_br_3079,base.br,base.state_br_sc,Maravilha
city_br_3080,base.br,base.state_br_mg,Maravilhas
city_br_3081,base.br,base.state_br_pb,Marcação
city_br_3082,base.br,base.state_br_mt,Marcelândia
city_br_3083,base.br,base.state_br_rs,Marcelino Ramos
city_br_3084,base.br,base.state_br_rn,Marcelino Vieira
city_br_3085,base.br,base.state_br_ba,Marcionílio Souza
city_br_3086,base.br,base.state_br_ce,Marco
city_br_3087,base.br,base.state_br_pi,Marcolândia
city_br_3088,base.br,base.state_br_pi,Marcos Parente
city_br_3089,base.br,base.state_br_pr,Marechal Cândido Rondon
city_br_3090,base.br,base.state_br_al,Marechal Deodoro
city_br_3091,base.br,base.state_br_es,Marechal Floriano
city_br_3092,base.br,base.state_br_ac,Marechal Thaumaturgo
city_br_3093,base.br,base.state_br_sc,Marema
city_br_3094,base.br,base.state_br_pb,Mari
city_br_3095,base.br,base.state_br_mg,Maria da Fé
city_br_3096,base.br,base.state_br_pr,Maria Helena
city_br_3097,base.br,base.state_br_pr,Marialva
city_br_3098,base.br,base.state_br_mg,Mariana
city_br_3099,base.br,base.state_br_rs,Mariana Pimentel
city_br_3100,base.br,base.state_br_rs,Mariano Moro
city_br_3101,base.br,base.state_br_to,Marianópolis do Tocantins
city_br_3102,base.br,base.state_br_sp,Mariápolis
city_br_3103,base.br,base.state_br_al,Maribondo
city_br_3104,base.br,base.state_br_mg,Marilac
city_br_3105,base.br,base.state_br_es,Marilândia
city_br_3106,base.br,base.state_br_pr,Marilândia do Sul
city_br_3107,base.br,base.state_br_pr,Marilena
city_br_3108,base.br,base.state_br_pr,Mariluz
city_br_3109,base.br,base.state_br_sp,Marinópolis
city_br_3110,base.br,base.state_br_mg,Mário Campos
city_br_3111,base.br,base.state_br_pr,Mariópolis
city_br_3112,base.br,base.state_br_pr,Maripá
city_br_3113,base.br,base.state_br_mg,Maripá de Minas
city_br_3114,base.br,base.state_br_pb,Marizópolis
city_br_3115,base.br,base.state_br_mg,Marliéria
city_br_3116,base.br,base.state_br_pr,Marmeleiro
city_br_3117,base.br,base.state_br_mg,Marmelópolis
city_br_3118,base.br,base.state_br_rs,Marques de Souza
city_br_3119,base.br,base.state_br_pr,Marquinho
city_br_3120,base.br,base.state_br_mg,Martinho Campos
city_br_3121,base.br,base.state_br_ce,Martinópole
city_br_3122,base.br,base.state_br_sp,Martinópolis
city_br_3123,base.br,base.state_br_rn,Martins
city_br_3124,base.br,base.state_br_mg,Martins Soares
city_br_3125,base.br,base.state_br_se,Maruim
city_br_3126,base.br,base.state_br_pr,Marumbi
city_br_3127,base.br,base.state_br_go,Marzagão
city_br_3128,base.br,base.state_br_ba,Mascote
city_br_3129,base.br,base.state_br_ce,Massapê
city_br_3130,base.br,base.state_br_pi,Massapê do Piauí
city_br_3131,base.br,base.state_br_pb,Massaranduba
city_br_3132,base.br,base.state_br_sc,Massaranduba
city_br_3133,base.br,base.state_br_rs,Mata
city_br_3134,base.br,base.state_br_ba,Mata de São João
city_br_3135,base.br,base.state_br_al,Mata Grande
city_br_3136,base.br,base.state_br_ma,Mata Roma
city_br_3137,base.br,base.state_br_mg,Mata Verde
city_br_3138,base.br,base.state_br_sp,Matão
city_br_3139,base.br,base.state_br_pb,Mataraca
city_br_3140,base.br,base.state_br_to,Mateiros
city_br_3141,base.br,base.state_br_pr,Matelândia
city_br_3142,base.br,base.state_br_mg,Materlândia
city_br_3143,base.br,base.state_br_mg,Mateus Leme
city_br_3144,base.br,base.state_br_mg,Mathias Lobato
city_br_3145,base.br,base.state_br_mg,Matias Barbosa
city_br_3146,base.br,base.state_br_mg,Matias Cardoso
city_br_3147,base.br,base.state_br_pi,Matias Olímpio
city_br_3148,base.br,base.state_br_ba,Matina
city_br_3149,base.br,base.state_br_ma,Matinha
city_br_3150,base.br,base.state_br_pb,Matinhas
city_br_3151,base.br,base.state_br_pr,Matinhos
city_br_3152,base.br,base.state_br_mg,Matipó
city_br_3153,base.br,base.state_br_rs,Mato Castelhano
city_br_3154,base.br,base.state_br_pb,Mato Grosso
city_br_3155,base.br,base.state_br_rs,Mato Leitão
city_br_3156,base.br,base.state_br_rs,Mato Queimado
city_br_3157,base.br,base.state_br_pr,Mato Rico
city_br_3158,base.br,base.state_br_mg,Mato Verde
city_br_3159,base.br,base.state_br_ma,Matões
city_br_3160,base.br,base.state_br_ma,Matões do Norte
city_br_3161,base.br,base.state_br_sc,Matos Costa
city_br_3162,base.br,base.state_br_mg,Matozinhos
city_br_3163,base.br,base.state_br_go,Matrinchã
city_br_3164,base.br,base.state_br_al,Matriz de Camaragibe
city_br_3165,base.br,base.state_br_mt,Matupá
city_br_3166,base.br,base.state_br_pb,Maturéia
city_br_3167,base.br,base.state_br_mg,Matutina
city_br_3168,base.br,base.state_br_pr,Mauá da Serra
city_br_3169,base.br,base.state_br_am,Maués
city_br_3170,base.br,base.state_br_go,Maurilândia
city_br_3171,base.br,base.state_br_to,Maurilândia do Tocantins
city_br_3172,base.br,base.state_br_ce,Mauriti
city_br_3173,base.br,base.state_br_rn,Maxaranguape
city_br_3174,base.br,base.state_br_rs,Maximiliano de Almeida
city_br_3175,base.br,base.state_br_ap,Mazagão
city_br_3176,base.br,base.state_br_mg,Medeiros
city_br_3177,base.br,base.state_br_ba,Medeiros Neto
city_br_3178,base.br,base.state_br_pr,Medianeira
city_br_3179,base.br,base.state_br_pa,Medicilândia
city_br_3180,base.br,base.state_br_mg,Medina
city_br_3181,base.br,base.state_br_sc,Meleiro
city_br_3182,base.br,base.state_br_pa,Melgaço
city_br_3183,base.br,base.state_br_rj,Mendes
city_br_3184,base.br,base.state_br_mg,Mendes Pimentel
city_br_3185,base.br,base.state_br_sp,Mendonça
city_br_3186,base.br,base.state_br_pr,Mercedes
city_br_3187,base.br,base.state_br_mg,Mercês
city_br_3188,base.br,base.state_br_sp,Meridiano
city_br_3189,base.br,base.state_br_ce,Meruoca
city_br_3190,base.br,base.state_br_sp,Mesópolis
city_br_3191,base.br,base.state_br_mg,Mesquita
city_br_3192,base.br,base.state_br_al,Messias
city_br_3193,base.br,base.state_br_rn,Messias Targino
city_br_3194,base.br,base.state_br_pi,Miguel Alves
city_br_3195,base.br,base.state_br_ba,Miguel Calmon
city_br_3196,base.br,base.state_br_pi,Miguel Leão
city_br_3197,base.br,base.state_br_rj,Miguel Pereira
city_br_3198,base.br,base.state_br_sp,Miguelópolis
city_br_3199,base.br,base.state_br_ba,Milagres
city_br_3200,base.br,base.state_br_ce,Milagres
city_br_3201,base.br,base.state_br_ma,Milagres do Maranhão
city_br_3202,base.br,base.state_br_ce,Milhã
city_br_3203,base.br,base.state_br_pi,Milton Brandão
city_br_3204,base.br,base.state_br_go,Mimoso de Goiás
city_br_3205,base.br,base.state_br_es,Mimoso do Sul
city_br_3206,base.br,base.state_br_go,Minaçu
city_br_3207,base.br,base.state_br_al,Minador do Negrão
city_br_3208,base.br,base.state_br_rs,Minas do Leão
city_br_3209,base.br,base.state_br_mg,Minas Novas
city_br_3210,base.br,base.state_br_mg,Minduri
city_br_3211,base.br,base.state_br_go,Mineiros
city_br_3212,base.br,base.state_br_sp,Mineiros do Tietê
city_br_3213,base.br,base.state_br_ro,Ministro Andreazza
city_br_3214,base.br,base.state_br_sp,Mira Estrela
city_br_3215,base.br,base.state_br_mg,Mirabela
city_br_3216,base.br,base.state_br_sp,Miracatu
city_br_3217,base.br,base.state_br_rj,Miracema
city_br_3218,base.br,base.state_br_to,Miracema do Tocantins
city_br_3219,base.br,base.state_br_ma,Mirador
city_br_3220,base.br,base.state_br_pr,Mirador
city_br_3221,base.br,base.state_br_mg,Miradouro
city_br_3222,base.br,base.state_br_rs,Miraguaí
city_br_3223,base.br,base.state_br_mg,Miraí
city_br_3224,base.br,base.state_br_ce,Miraíma
city_br_3225,base.br,base.state_br_ms,Miranda
city_br_3226,base.br,base.state_br_ma,Miranda do Norte
city_br_3227,base.br,base.state_br_pe,Mirandiba
city_br_3228,base.br,base.state_br_sp,Mirandópolis
city_br_3229,base.br,base.state_br_ba,Mirangaba
city_br_3230,base.br,base.state_br_to,Miranorte
city_br_3231,base.br,base.state_br_ba,Mirante
city_br_3232,base.br,base.state_br_ro,Mirante da Serra
city_br_3233,base.br,base.state_br_sp,Mirante do Paranapanema
city_br_3234,base.br,base.state_br_pr,Miraselva
city_br_3235,base.br,base.state_br_sp,Mirassol
city_br_3236,base.br,base.state_br_mt,Mirassol d'Oeste
city_br_3237,base.br,base.state_br_sp,Mirassolândia
city_br_3238,base.br,base.state_br_mg,Miravânia
city_br_3239,base.br,base.state_br_sc,Mirim Doce
city_br_3240,base.br,base.state_br_ma,Mirinzal
city_br_3241,base.br,base.state_br_pr,Missal
city_br_3242,base.br,base.state_br_ce,Missão Velha
city_br_3243,base.br,base.state_br_pa,Mocajuba
city_br_3244,base.br,base.state_br_sp,Mococa
city_br_3245,base.br,base.state_br_sc,Modelo
city_br_3246,base.br,base.state_br_mg,Moeda
city_br_3247,base.br,base.state_br_mg,Moema
city_br_3248,base.br,base.state_br_pb,Mogeiro
city_br_3249,base.br,base.state_br_sp,Mogi Mirim
city_br_3250,base.br,base.state_br_go,Moiporá
city_br_3251,base.br,base.state_br_se,Moita Bonita
city_br_3252,base.br,base.state_br_pa,Moju
city_br_3253,base.br,base.state_br_pa,Mojuí dos Campos
city_br_3254,base.br,base.state_br_ce,Mombaça
city_br_3255,base.br,base.state_br_sp,Mombuca
city_br_3256,base.br,base.state_br_ma,Monção
city_br_3257,base.br,base.state_br_sp,Monções
city_br_3258,base.br,base.state_br_sc,Mondaí
city_br_3259,base.br,base.state_br_sp,Mongaguá
city_br_3260,base.br,base.state_br_mg,Monjolos
city_br_3261,base.br,base.state_br_pi,Monsenhor Gil
city_br_3262,base.br,base.state_br_pi,Monsenhor Hipólito
city_br_3263,base.br,base.state_br_mg,Monsenhor Paulo
city_br_3264,base.br,base.state_br_ce,Monsenhor Tabosa
city_br_3265,base.br,base.state_br_pb,Montadas
city_br_3266,base.br,base.state_br_mg,Montalvânia
city_br_3267,base.br,base.state_br_es,Montanha
city_br_3268,base.br,base.state_br_rn,Montanhas
city_br_3269,base.br,base.state_br_rs,Montauri
city_br_3270,base.br,base.state_br_pa,Monte Alegre
city_br_3271,base.br,base.state_br_rn,Monte Alegre
city_br_3272,base.br,base.state_br_go,Monte Alegre de Goiás
city_br_3273,base.br,base.state_br_mg,Monte Alegre de Minas
city_br_3274,base.br,base.state_br_se,Monte Alegre de Sergipe
city_br_3275,base.br,base.state_br_pi,Monte Alegre do Piauí
city_br_3276,base.br,base.state_br_sp,Monte Alegre do Sul
city_br_3277,base.br,base.state_br_rs,Monte Alegre dos Campos
city_br_3278,base.br,base.state_br_sp,Monte Alto
city_br_3279,base.br,base.state_br_sp,Monte Aprazível
city_br_3280,base.br,base.state_br_mg,Monte Azul
city_br_3281,base.br,base.state_br_sp,Monte Azul Paulista
city_br_3282,base.br,base.state_br_mg,Monte Belo
city_br_3283,base.br,base.state_br_rs,Monte Belo do Sul
city_br_3284,base.br,base.state_br_sc,Monte Carlo
city_br_3285,base.br,base.state_br_mg,Monte Carmelo
city_br_3286,base.br,base.state_br_sc,Monte Castelo
city_br_3287,base.br,base.state_br_sp,Monte Castelo
city_br_3288,base.br,base.state_br_rn,Monte das Gameleiras
city_br_3289,base.br,base.state_br_to,Monte do Carmo
city_br_3290,base.br,base.state_br_mg,Monte Formoso
city_br_3291,base.br,base.state_br_pb,Monte Horebe
city_br_3292,base.br,base.state_br_sp,Monte Mor
city_br_3293,base.br,base.state_br_ro,Monte Negro
city_br_3294,base.br,base.state_br_ba,Monte Santo
city_br_3295,base.br,base.state_br_mg,Monte Santo de Minas
city_br_3296,base.br,base.state_br_to,Monte Santo do Tocantins
city_br_3297,base.br,base.state_br_mg,Monte Sião
city_br_3298,base.br,base.state_br_pb,Monteiro
city_br_3299,base.br,base.state_br_sp,Monteiro Lobato
city_br_3300,base.br,base.state_br_al,Monteirópolis
city_br_3301,base.br,base.state_br_rs,Montenegro
city_br_3302,base.br,base.state_br_ma,Montes Altos
city_br_3303,base.br,base.state_br_go,Montes Claros de Goiás
city_br_3304,base.br,base.state_br_mg,Montezuma
city_br_3305,base.br,base.state_br_go,Montividiu
city_br_3306,base.br,base.state_br_go,Montividiu do Norte
city_br_3307,base.br,base.state_br_ce,Morada Nova
city_br_3308,base.br,base.state_br_mg,Morada Nova de Minas
city_br_3309,base.br,base.state_br_ce,Moraújo
city_br_3310,base.br,base.state_br_pe,Moreilândia
city_br_3311,base.br,base.state_br_pr,Moreira Sales
city_br_3312,base.br,base.state_br_pe,Moreno
city_br_3313,base.br,base.state_br_rs,Mormaço
city_br_3314,base.br,base.state_br_ba,Morpará
city_br_3315,base.br,base.state_br_pr,Morretes
city_br_3316,base.br,base.state_br_ce,Morrinhos
city_br_3317,base.br,base.state_br_go,Morrinhos
city_br_3318,base.br,base.state_br_rs,Morrinhos do Sul
city_br_3319,base.br,base.state_br_sp,Morro Agudo
city_br_3320,base.br,base.state_br_go,Morro Agudo de Goiás
city_br_3321,base.br,base.state_br_pi,Morro Cabeça no Tempo
city_br_3322,base.br,base.state_br_sc,Morro da Fumaça
city_br_3323,base.br,base.state_br_mg,Morro da Garça
city_br_3324,base.br,base.state_br_ba,Morro do Chapéu
city_br_3325,base.br,base.state_br_pi,Morro do Chapéu do Piauí
city_br_3326,base.br,base.state_br_mg,Morro do Pilar
city_br_3327,base.br,base.state_br_sc,Morro Grande
city_br_3328,base.br,base.state_br_rs,Morro Redondo
city_br_3329,base.br,base.state_br_rs,Morro Reuter
city_br_3330,base.br,base.state_br_ma,Morros
city_br_3331,base.br,base.state_br_ba,Mortugaba
city_br_3332,base.br,base.state_br_sp,Morungaba
city_br_3333,base.br,base.state_br_go,Mossâmedes
city_br_3334,base.br,base.state_br_rs,Mostardas
city_br_3335,base.br,base.state_br_sp,Motuca
city_br_3336,base.br,base.state_br_go,Mozarlândia
city_br_3337,base.br,base.state_br_pa,Muaná
city_br_3338,base.br,base.state_br_rr,Mucajaí
city_br_3339,base.br,base.state_br_ce,Mucambo
city_br_3340,base.br,base.state_br_ba,Mucugê
city_br_3341,base.br,base.state_br_rs,Muçum
city_br_3342,base.br,base.state_br_ba,Mucuri
city_br_3343,base.br,base.state_br_es,Mucurici
city_br_3344,base.br,base.state_br_rs,Muitos Capões
city_br_3345,base.br,base.state_br_rs,Muliterno
city_br_3346,base.br,base.state_br_ce,Mulungu
city_br_3347,base.br,base.state_br_pb,Mulungu
city_br_3348,base.br,base.state_br_ba,Mulungu do Morro
city_br_3349,base.br,base.state_br_ba,Mundo Novo
city_br_3350,base.br,base.state_br_go,Mundo Novo
city_br_3351,base.br,base.state_br_ms,Mundo Novo
city_br_3352,base.br,base.state_br_mg,Munhoz
city_br_3353,base.br,base.state_br_pr,Munhoz de Melo
city_br_3354,base.br,base.state_br_ba,Muniz Ferreira
city_br_3355,base.br,base.state_br_es,Muniz Freire
city_br_3356,base.br,base.state_br_ba,Muquém de São Francisco
city_br_3357,base.br,base.state_br_es,Muqui
city_br_3358,base.br,base.state_br_se,Muribeca
city_br_3359,base.br,base.state_br_al,Murici
city_br_3360,base.br,base.state_br_pi,Murici dos Portelas
city_br_3361,base.br,base.state_br_to,Muricilândia
city_br_3362,base.br,base.state_br_ba,Muritiba
city_br_3363,base.br,base.state_br_sp,Murutinga do Sul
city_br_3364,base.br,base.state_br_ba,Mutuípe
city_br_3365,base.br,base.state_br_mg,Mutum
city_br_3366,base.br,base.state_br_go,Mutunópolis
city_br_3367,base.br,base.state_br_mg,Muzambinho
city_br_3368,base.br,base.state_br_mg,Nacip Raydan
city_br_3369,base.br,base.state_br_sp,Nantes
city_br_3370,base.br,base.state_br_mg,Nanuque
city_br_3371,base.br,base.state_br_rs,Não-Me-Toque
city_br_3372,base.br,base.state_br_mg,Naque
city_br_3373,base.br,base.state_br_sp,Narandiba
city_br_3374,base.br,base.state_br_mg,Natalândia
city_br_3375,base.br,base.state_br_mg,Natércia
city_br_3376,base.br,base.state_br_rj,Natividade
city_br_3377,base.br,base.state_br_to,Natividade
city_br_3378,base.br,base.state_br_sp,Natividade da Serra
city_br_3379,base.br,base.state_br_pb,Natuba
city_br_3380,base.br,base.state_br_sc,Navegantes
city_br_3381,base.br,base.state_br_ms,Naviraí
city_br_3382,base.br,base.state_br_ba,Nazaré
city_br_3383,base.br,base.state_br_to,Nazaré
city_br_3384,base.br,base.state_br_pe,Nazaré da Mata
city_br_3385,base.br,base.state_br_pi,Nazaré do Piauí
city_br_3386,base.br,base.state_br_sp,Nazaré Paulista
city_br_3387,base.br,base.state_br_mg,Nazareno
city_br_3388,base.br,base.state_br_pb,Nazarezinho
city_br_3389,base.br,base.state_br_pi,Nazária
city_br_3390,base.br,base.state_br_go,Nazário
city_br_3391,base.br,base.state_br_se,Neópolis
city_br_3392,base.br,base.state_br_mg,Nepomuceno
city_br_3393,base.br,base.state_br_go,Nerópolis
city_br_3394,base.br,base.state_br_sp,Neves Paulista
city_br_3395,base.br,base.state_br_am,Nhamundá
city_br_3396,base.br,base.state_br_sp,Nhandeara
city_br_3397,base.br,base.state_br_rs,Nicolau Vergueiro
city_br_3398,base.br,base.state_br_ba,Nilo Peçanha
city_br_3399,base.br,base.state_br_ma,Nina Rodrigues
city_br_3400,base.br,base.state_br_mg,Ninheira
city_br_3401,base.br,base.state_br_ms,Nioaque
city_br_3402,base.br,base.state_br_sp,Nipoã
city_br_3403,base.br,base.state_br_go,Niquelândia
city_br_3404,base.br,base.state_br_rn,Nísia Floresta
city_br_3405,base.br,base.state_br_mt,Nobres
city_br_3406,base.br,base.state_br_rs,Nonoai
city_br_3407,base.br,base.state_br_ba,Nordestina
city_br_3408,base.br,base.state_br_rr,Normandia
city_br_3409,base.br,base.state_br_mt,Nortelândia
city_br_3410,base.br,base.state_br_se,Nossa Senhora Aparecida
city_br_3411,base.br,base.state_br_se,Nossa Senhora da Glória
city_br_3412,base.br,base.state_br_se,Nossa Senhora das Dores
city_br_3413,base.br,base.state_br_pr,Nossa Senhora das Graças
city_br_3414,base.br,base.state_br_se,Nossa Senhora de Lourdes
city_br_3415,base.br,base.state_br_pi,Nossa Senhora de Nazaré
city_br_3416,base.br,base.state_br_mt,Nossa Senhora do Livramento
city_br_3417,base.br,base.state_br_pi,Nossa Senhora dos Remédios
city_br_3418,base.br,base.state_br_sp,Nova Aliança
city_br_3419,base.br,base.state_br_pr,Nova Aliança do Ivaí
city_br_3420,base.br,base.state_br_rs,Nova Alvorada
city_br_3421,base.br,base.state_br_ms,Nova Alvorada do Sul
city_br_3422,base.br,base.state_br_go,Nova América
city_br_3423,base.br,base.state_br_pr,Nova América da Colina
city_br_3424,base.br,base.state_br_ms,Nova Andradina
city_br_3425,base.br,base.state_br_rs,Nova Araçá
city_br_3426,base.br,base.state_br_go,Nova Aurora
city_br_3427,base.br,base.state_br_pr,Nova Aurora
city_br_3428,base.br,base.state_br_mt,Nova Bandeirantes
city_br_3429,base.br,base.state_br_rs,Nova Bassano
city_br_3430,base.br,base.state_br_mg,Nova Belém
city_br_3431,base.br,base.state_br_rs,Nova Boa Vista
city_br_3432,base.br,base.state_br_mt,Nova Brasilândia
city_br_3433,base.br,base.state_br_ro,Nova Brasilândia D'Oeste
city_br_3434,base.br,base.state_br_rs,Nova Bréscia
city_br_3435,base.br,base.state_br_sp,Nova Campina
city_br_3436,base.br,base.state_br_ba,Nova Canaã
city_br_3437,base.br,base.state_br_mt,Nova Canaã do Norte
city_br_3438,base.br,base.state_br_sp,Nova Canaã Paulista
city_br_3439,base.br,base.state_br_rs,Nova Candelária
city_br_3440,base.br,base.state_br_pr,Nova Cantu
city_br_3441,base.br,base.state_br_sp,Nova Castilho
city_br_3442,base.br,base.state_br_ma,Nova Colinas
city_br_3443,base.br,base.state_br_go,Nova Crixás
city_br_3444,base.br,base.state_br_rn,Nova Cruz
city_br_3445,base.br,base.state_br_mg,Nova Era
city_br_3446,base.br,base.state_br_sc,Nova Erechim
city_br_3447,base.br,base.state_br_pr,Nova Esperança
city_br_3448,base.br,base.state_br_pa,Nova Esperança do Piriá
city_br_3449,base.br,base.state_br_pr,Nova Esperança do Sudoeste
city_br_3450,base.br,base.state_br_rs,Nova Esperança do Sul
city_br_3451,base.br,base.state_br_sp,Nova Europa
city_br_3452,base.br,base.state_br_ba,Nova Fátima
city_br_3453,base.br,base.state_br_pr,Nova Fátima
city_br_3454,base.br,base.state_br_pb,Nova Floresta
city_br_3455,base.br,base.state_br_go,Nova Glória
city_br_3456,base.br,base.state_br_sp,Nova Granada
city_br_3457,base.br,base.state_br_mt,Nova Guarita
city_br_3458,base.br,base.state_br_sp,Nova Guataporanga
city_br_3459,base.br,base.state_br_rs,Nova Hartz
city_br_3460,base.br,base.state_br_ba,Nova Ibiá
city_br_3461,base.br,base.state_br_go,Nova Iguaçu de Goiás
city_br_3462,base.br,base.state_br_sp,Nova Independência
city_br_3463,base.br,base.state_br_ma,Nova Iorque
city_br_3464,base.br,base.state_br_pa,Nova Ipixuna
city_br_3465,base.br,base.state_br_sc,Nova Itaberaba
city_br_3466,base.br,base.state_br_ba,Nova Itarana
city_br_3467,base.br,base.state_br_mt,Nova Lacerda
city_br_3468,base.br,base.state_br_pr,Nova Laranjeiras
city_br_3469,base.br,base.state_br_pr,Nova Londrina
city_br_3470,base.br,base.state_br_sp,Nova Luzitânia
city_br_3471,base.br,base.state_br_ro,Nova Mamoré
city_br_3472,base.br,base.state_br_mt,Nova Marilândia
city_br_3473,base.br,base.state_br_mt,Nova Maringá
city_br_3474,base.br,base.state_br_mg,Nova Módica
city_br_3475,base.br,base.state_br_mt,Nova Monte Verde
city_br_3476,base.br,base.state_br_mt,Nova Mutum
city_br_3477,base.br,base.state_br_mt,Nova Nazaré
city_br_3478,base.br,base.state_br_sp,Nova Odessa
city_br_3479,base.br,base.state_br_mt,Nova Olímpia
city_br_3480,base.br,base.state_br_pr,Nova Olímpia
city_br_3481,base.br,base.state_br_ce,Nova Olinda
city_br_3482,base.br,base.state_br_pb,Nova Olinda
city_br_3483,base.br,base.state_br_to,Nova Olinda
city_br_3484,base.br,base.state_br_ma,Nova Olinda do Maranhão
city_br_3485,base.br,base.state_br_am,Nova Olinda do Norte
city_br_3486,base.br,base.state_br_rs,Nova Pádua
city_br_3487,base.br,base.state_br_rs,Nova Palma
city_br_3488,base.br,base.state_br_pb,Nova Palmeira
city_br_3489,base.br,base.state_br_rs,Nova Petrópolis
city_br_3490,base.br,base.state_br_mg,Nova Ponte
city_br_3491,base.br,base.state_br_mg,Nova Porteirinha
city_br_3492,base.br,base.state_br_rs,Nova Prata
city_br_3493,base.br,base.state_br_pr,Nova Prata do Iguaçu
city_br_3494,base.br,base.state_br_rs,Nova Ramada
city_br_3495,base.br,base.state_br_ba,Nova Redenção
city_br_3496,base.br,base.state_br_mg,Nova Resende
city_br_3497,base.br,base.state_br_go,Nova Roma
city_br_3498,base.br,base.state_br_rs,Nova Roma do Sul
city_br_3499,base.br,base.state_br_to,Nova Rosalândia
city_br_3500,base.br,base.state_br_ce,Nova Russas
city_br_3501,base.br,base.state_br_pr,Nova Santa Bárbara
city_br_3502,base.br,base.state_br_mt,Nova Santa Helena
city_br_3503,base.br,base.state_br_pi,Nova Santa Rita
city_br_3504,base.br,base.state_br_rs,Nova Santa Rita
city_br_3505,base.br,base.state_br_pr,Nova Santa Rosa
city_br_3506,base.br,base.state_br_ba,Nova Soure
city_br_3507,base.br,base.state_br_pr,Nova Tebas
city_br_3508,base.br,base.state_br_pa,Nova Timboteua
city_br_3509,base.br,base.state_br_sc,Nova Trento
city_br_3510,base.br,base.state_br_mt,Nova Ubiratã
city_br_3511,base.br,base.state_br_mg,Nova União
city_br_3512,base.br,base.state_br_ro,Nova União
city_br_3513,base.br,base.state_br_es,Nova Venécia
city_br_3514,base.br,base.state_br_go,Nova Veneza
city_br_3515,base.br,base.state_br_sc,Nova Veneza
city_br_3516,base.br,base.state_br_ba,Nova Viçosa
city_br_3517,base.br,base.state_br_mt,Nova Xavantina
city_br_3518,base.br,base.state_br_sp,Novais
city_br_3519,base.br,base.state_br_to,Novo Acordo
city_br_3520,base.br,base.state_br_am,Novo Airão
city_br_3521,base.br,base.state_br_to,Novo Alegre
city_br_3522,base.br,base.state_br_am,Novo Aripuanã
city_br_3523,base.br,base.state_br_rs,Novo Barreiro
city_br_3524,base.br,base.state_br_go,Novo Brasil
city_br_3525,base.br,base.state_br_rs,Novo Cabrais
city_br_3526,base.br,base.state_br_mg,Novo Cruzeiro
city_br_3527,base.br,base.state_br_ba,Novo Horizonte
city_br_3528,base.br,base.state_br_sc,Novo Horizonte
city_br_3529,base.br,base.state_br_sp,Novo Horizonte
city_br_3530,base.br,base.state_br_mt,Novo Horizonte do Norte
city_br_3531,base.br,base.state_br_ro,Novo Horizonte do Oeste
city_br_3532,base.br,base.state_br_ms,Novo Horizonte do Sul
city_br_3533,base.br,base.state_br_pr,Novo Itacolomi
city_br_3534,base.br,base.state_br_to,Novo Jardim
city_br_3535,base.br,base.state_br_al,Novo Lino
city_br_3536,base.br,base.state_br_rs,Novo Machado
city_br_3537,base.br,base.state_br_mt,Novo Mundo
city_br_3538,base.br,base.state_br_ce,Novo Oriente
city_br_3539,base.br,base.state_br_mg,Novo Oriente de Minas
city_br_3540,base.br,base.state_br_pi,Novo Oriente do Piauí
city_br_3541,base.br,base.state_br_go,Novo Planalto
city_br_3542,base.br,base.state_br_pa,Novo Progresso
city_br_3543,base.br,base.state_br_pa,Novo Repartimento
city_br_3544,base.br,base.state_br_mt,Novo Santo Antônio
city_br_3545,base.br,base.state_br_pi,Novo Santo Antônio
city_br_3546,base.br,base.state_br_mt,Novo São Joaquim
city_br_3547,base.br,base.state_br_rs,Novo Tiradentes
city_br_3548,base.br,base.state_br_ba,Novo Triunfo
city_br_3549,base.br,base.state_br_rs,Novo Xingu
city_br_3550,base.br,base.state_br_mg,Novorizonte
city_br_3551,base.br,base.state_br_sp,Nuporanga
city_br_3552,base.br,base.state_br_pa,Óbidos
city_br_3553,base.br,base.state_br_ce,Ocara
city_br_3554,base.br,base.state_br_sp,Ocauçu
city_br_3555,base.br,base.state_br_pi,Oeiras
city_br_3556,base.br,base.state_br_pa,Oeiras do Pará
city_br_3557,base.br,base.state_br_ap,Oiapoque
city_br_3558,base.br,base.state_br_mg,Olaria
city_br_3559,base.br,base.state_br_sp,Óleo
city_br_3560,base.br,base.state_br_pb,Olho d'Água
city_br_3561,base.br,base.state_br_ma,Olho d'Água das Cunhãs
city_br_3562,base.br,base.state_br_al,Olho d'Água das Flores
city_br_3563,base.br,base.state_br_rn,Olho-d'Água do Borges
city_br_3564,base.br,base.state_br_al,Olho d'Água do Casado
city_br_3565,base.br,base.state_br_pi,Olho D'Água do Piauí
city_br_3566,base.br,base.state_br_al,Olho d'Água Grande
city_br_3567,base.br,base.state_br_mg,Olhos-d'Água
city_br_3568,base.br,base.state_br_sp,Olímpia
city_br_3569,base.br,base.state_br_mg,Olímpio Noronha
city_br_3570,base.br,base.state_br_ma,Olinda Nova do Maranhão
city_br_3571,base.br,base.state_br_ba,Olindina
city_br_3572,base.br,base.state_br_pb,Olivedos
city_br_3573,base.br,base.state_br_mg,Oliveira
city_br_3574,base.br,base.state_br_to,Oliveira de Fátima
city_br_3575,base.br,base.state_br_ba,Oliveira dos Brejinhos
city_br_3576,base.br,base.state_br_mg,Oliveira Fortes
city_br_3577,base.br,base.state_br_al,Olivença
city_br_3578,base.br,base.state_br_mg,Onça de Pitangui
city_br_3579,base.br,base.state_br_sp,Onda Verde
city_br_3580,base.br,base.state_br_mg,Oratórios
city_br_3581,base.br,base.state_br_sp,Oriente
city_br_3582,base.br,base.state_br_sp,Orindiúva
city_br_3583,base.br,base.state_br_pa,Oriximiná
city_br_3584,base.br,base.state_br_mg,Orizânia
city_br_3585,base.br,base.state_br_go,Orizona
city_br_3586,base.br,base.state_br_sp,Orlândia
city_br_3587,base.br,base.state_br_sc,Orleans
city_br_3588,base.br,base.state_br_pe,Orobó
city_br_3589,base.br,base.state_br_pe,Orocó
city_br_3590,base.br,base.state_br_ce,Orós
city_br_3591,base.br,base.state_br_pr,Ortigueira
city_br_3592,base.br,base.state_br_sp,Oscar Bressane
city_br_3593,base.br,base.state_br_rs,Osório
city_br_3594,base.br,base.state_br_sp,Osvaldo Cruz
city_br_3595,base.br,base.state_br_sc,Otacílio Costa
city_br_3596,base.br,base.state_br_pa,Ourém
city_br_3597,base.br,base.state_br_ba,Ouriçangas
city_br_3598,base.br,base.state_br_pe,Ouricuri
city_br_3599,base.br,base.state_br_pa,Ourilândia do Norte
city_br_3600,base.br,base.state_br_pr,Ourizona
city_br_3601,base.br,base.state_br_sc,Ouro
city_br_3602,base.br,base.state_br_al,Ouro Branco
city_br_3603,base.br,base.state_br_mg,Ouro Branco
city_br_3604,base.br,base.state_br_rn,Ouro Branco
city_br_3605,base.br,base.state_br_mg,Ouro Fino
city_br_3606,base.br,base.state_br_mg,Ouro Preto
city_br_3607,base.br,base.state_br_ro,Ouro Preto do Oeste
city_br_3608,base.br,base.state_br_pb,Ouro Velho
city_br_3609,base.br,base.state_br_sc,Ouro Verde
city_br_3610,base.br,base.state_br_sp,Ouro Verde
city_br_3611,base.br,base.state_br_go,Ouro Verde de Goiás
city_br_3612,base.br,base.state_br_mg,Ouro Verde de Minas
city_br_3613,base.br,base.state_br_pr,Ouro Verde do Oeste
city_br_3614,base.br,base.state_br_sp,Ouroeste
city_br_3615,base.br,base.state_br_ba,Ourolândia
city_br_3616,base.br,base.state_br_go,Ouvidor
city_br_3617,base.br,base.state_br_sp,Pacaembu
city_br_3618,base.br,base.state_br_pa,Pacajá
city_br_3619,base.br,base.state_br_ce,Pacajus
city_br_3620,base.br,base.state_br_rr,Pacaraima
city_br_3621,base.br,base.state_br_ce,Pacatuba
city_br_3622,base.br,base.state_br_se,Pacatuba
city_br_3623,base.br,base.state_br_ce,Pacoti
city_br_3624,base.br,base.state_br_ce,Pacujá
city_br_3625,base.br,base.state_br_go,Padre Bernardo
city_br_3626,base.br,base.state_br_mg,Padre Carvalho
city_br_3627,base.br,base.state_br_pi,Padre Marcos
city_br_3628,base.br,base.state_br_mg,Padre Paraíso
city_br_3629,base.br,base.state_br_pi,Paes Landim
city_br_3630,base.br,base.state_br_mg,Pai Pedro
city_br_3631,base.br,base.state_br_sc,Paial
city_br_3632,base.br,base.state_br_pr,Paiçandu
city_br_3633,base.br,base.state_br_rs,Paim Filho
city_br_3634,base.br,base.state_br_mg,Paineiras
city_br_3635,base.br,base.state_br_sc,Painel
city_br_3636,base.br,base.state_br_mg,Pains
city_br_3637,base.br,base.state_br_mg,Paiva
city_br_3638,base.br,base.state_br_pi,Pajeú do Piauí
city_br_3639,base.br,base.state_br_al,Palestina
city_br_3640,base.br,base.state_br_sp,Palestina
city_br_3641,base.br,base.state_br_go,Palestina de Goiás
city_br_3642,base.br,base.state_br_pa,Palestina do Pará
city_br_3643,base.br,base.state_br_ce,Palhano
city_br_3644,base.br,base.state_br_mg,Palma
city_br_3645,base.br,base.state_br_sc,Palma Sola
city_br_3646,base.br,base.state_br_ce,Palmácia
city_br_3647,base.br,base.state_br_pe,Palmares
city_br_3648,base.br,base.state_br_rs,Palmares do Sul
city_br_3649,base.br,base.state_br_sp,Palmares Paulista
city_br_3650,base.br,base.state_br_pr,Palmas
city_br_3651,base.br,base.state_br_ba,Palmas de Monte Alto
city_br_3652,base.br,base.state_br_pr,Palmeira
city_br_3653,base.br,base.state_br_sc,Palmeira
city_br_3654,base.br,base.state_br_sp,Palmeira d'Oeste
city_br_3655,base.br,base.state_br_rs,Palmeira das Missões
city_br_3656,base.br,base.state_br_pi,Palmeira do Piauí
city_br_3657,base.br,base.state_br_al,Palmeira dos Índios
city_br_3658,base.br,base.state_br_pi,Palmeirais
city_br_3659,base.br,base.state_br_ma,Palmeirândia
city_br_3660,base.br,base.state_br_to,Palmeirante
city_br_3661,base.br,base.state_br_ba,Palmeiras
city_br_3662,base.br,base.state_br_go,Palmeiras de Goiás
city_br_3663,base.br,base.state_br_to,Palmeiras do Tocantins
city_br_3664,base.br,base.state_br_pe,Palmeirina
city_br_3665,base.br,base.state_br_to,Palmeirópolis
city_br_3666,base.br,base.state_br_go,Palmelo
city_br_3667,base.br,base.state_br_go,Palminópolis
city_br_3668,base.br,base.state_br_pr,Palmital
city_br_3669,base.br,base.state_br_sp,Palmital
city_br_3670,base.br,base.state_br_rs,Palmitinho
city_br_3671,base.br,base.state_br_sc,Palmitos
city_br_3672,base.br,base.state_br_mg,Palmópolis
city_br_3673,base.br,base.state_br_pr,Palotina
city_br_3674,base.br,base.state_br_go,Panamá
city_br_3675,base.br,base.state_br_rs,Panambi
city_br_3676,base.br,base.state_br_es,Pancas
city_br_3677,base.br,base.state_br_pe,Panelas
city_br_3678,base.br,base.state_br_sp,Panorama
city_br_3679,base.br,base.state_br_rs,Pantano Grande
city_br_3680,base.br,base.state_br_al,Pão de Açúcar
city_br_3681,base.br,base.state_br_mg,Papagaios
city_br_3682,base.br,base.state_br_sc,Papanduva
city_br_3683,base.br,base.state_br_pi,Paquetá
city_br_3684,base.br,base.state_br_mg,Pará de Minas
city_br_3685,base.br,base.state_br_rj,Paracambi
city_br_3686,base.br,base.state_br_mg,Paracatu
city_br_3687,base.br,base.state_br_ce,Paracuru
city_br_3688,base.br,base.state_br_mg,Paraguaçu
city_br_3689,base.br,base.state_br_sp,Paraguaçu Paulista
city_br_3690,base.br,base.state_br_rs,Paraí
city_br_3691,base.br,base.state_br_rj,Paraíba do Sul
city_br_3692,base.br,base.state_br_ma,Paraibano
city_br_3693,base.br,base.state_br_sp,Paraibuna
city_br_3694,base.br,base.state_br_ce,Paraipaba
city_br_3695,base.br,base.state_br_sc,Paraíso
city_br_3696,base.br,base.state_br_sp,Paraíso
city_br_3697,base.br,base.state_br_ms,Paraíso das Águas
city_br_3698,base.br,base.state_br_pr,Paraíso do Norte
city_br_3699,base.br,base.state_br_rs,Paraíso do Sul
city_br_3700,base.br,base.state_br_to,Paraíso do Tocantins
city_br_3701,base.br,base.state_br_mg,Paraisópolis
city_br_3702,base.br,base.state_br_ce,Parambu
city_br_3703,base.br,base.state_br_ba,Paramirim
city_br_3704,base.br,base.state_br_ce,Paramoti
city_br_3705,base.br,base.state_br_rn,Paraná
city_br_3706,base.br,base.state_br_to,Paranã
city_br_3707,base.br,base.state_br_pr,Paranacity
city_br_3708,base.br,base.state_br_ms,Paranaíba
city_br_3709,base.br,base.state_br_go,Paranaiguara
city_br_3710,base.br,base.state_br_mt,Paranaíta
city_br_3711,base.br,base.state_br_sp,Paranapanema
city_br_3712,base.br,base.state_br_pr,Paranapoema
city_br_3713,base.br,base.state_br_sp,Paranapuã
city_br_3714,base.br,base.state_br_pe,Paranatama
city_br_3715,base.br,base.state_br_mt,Paranatinga
city_br_3716,base.br,base.state_br_pr,Paranavaí
city_br_3717,base.br,base.state_br_ms,Paranhos
city_br_3718,base.br,base.state_br_mg,Paraopeba
city_br_3719,base.br,base.state_br_sp,Parapuã
city_br_3720,base.br,base.state_br_pb,Parari
city_br_3721,base.br,base.state_br_ba,Paratinga
city_br_3722,base.br,base.state_br_rj,Paraty
city_br_3723,base.br,base.state_br_rn,Paraú
city_br_3724,base.br,base.state_br_go,Paraúna
city_br_3725,base.br,base.state_br_rn,Parazinho
city_br_3726,base.br,base.state_br_sp,Pardinho
city_br_3727,base.br,base.state_br_rs,Pareci Novo
city_br_3728,base.br,base.state_br_ro,Parecis
city_br_3729,base.br,base.state_br_rn,Parelhas
city_br_3730,base.br,base.state_br_al,Pariconha
city_br_3731,base.br,base.state_br_am,Parintins
city_br_3732,base.br,base.state_br_ba,Paripiranga
city_br_3733,base.br,base.state_br_al,Paripueira
city_br_3734,base.br,base.state_br_sp,Pariquera-Açu
city_br_3735,base.br,base.state_br_sp,Parisi
city_br_3736,base.br,base.state_br_pi,Parnaguá
city_br_3737,base.br,base.state_br_pe,Parnamirim
city_br_3738,base.br,base.state_br_ma,Parnarama
city_br_3739,base.br,base.state_br_rs,Parobé
city_br_3740,base.br,base.state_br_rn,Passa e Fica
city_br_3741,base.br,base.state_br_mg,Passa Quatro
city_br_3742,base.br,base.state_br_rs,Passa Sete
city_br_3743,base.br,base.state_br_mg,Passa Tempo
city_br_3744,base.br,base.state_br_mg,Passa-Vinte
city_br_3745,base.br,base.state_br_mg,Passabém
city_br_3746,base.br,base.state_br_pb,Passagem
city_br_3747,base.br,base.state_br_rn,Passagem
city_br_3748,base.br,base.state_br_ma,Passagem Franca
city_br_3749,base.br,base.state_br_pi,Passagem Franca do Piauí
city_br_3750,base.br,base.state_br_pe,Passira
city_br_3751,base.br,base.state_br_al,Passo de Camaragibe
city_br_3752,base.br,base.state_br_sc,Passo de Torres
city_br_3753,base.br,base.state_br_rs,Passo do Sobrado
city_br_3754,base.br,base.state_br_sc,Passos Maia
city_br_3755,base.br,base.state_br_ma,Pastos Bons
city_br_3756,base.br,base.state_br_mg,Patis
city_br_3757,base.br,base.state_br_pr,Pato Bragado
city_br_3758,base.br,base.state_br_pr,Pato Branco
city_br_3759,base.br,base.state_br_pi,Patos do Piauí
city_br_3760,base.br,base.state_br_mg,Patrocínio
city_br_3761,base.br,base.state_br_mg,Patrocínio do Muriaé
city_br_3762,base.br,base.state_br_sp,Patrocínio Paulista
city_br_3763,base.br,base.state_br_rn,Patu
city_br_3764,base.br,base.state_br_rj,Paty do Alferes
city_br_3765,base.br,base.state_br_ba,Pau Brasil
city_br_3766,base.br,base.state_br_pa,Pau D'Arco
city_br_3767,base.br,base.state_br_to,Pau D'Arco
city_br_3768,base.br,base.state_br_pi,Pau D'Arco do Piauí
city_br_3769,base.br,base.state_br_rn,Pau dos Ferros
city_br_3770,base.br,base.state_br_pe,Paudalho
city_br_3771,base.br,base.state_br_am,Pauini
city_br_3772,base.br,base.state_br_mg,Paula Cândido
city_br_3773,base.br,base.state_br_pr,Paula Freitas
city_br_3774,base.br,base.state_br_sp,Paulicéia
city_br_3775,base.br,base.state_br_ma,Paulino Neves
city_br_3776,base.br,base.state_br_pb,Paulista
city_br_3777,base.br,base.state_br_pi,Paulistana
city_br_3778,base.br,base.state_br_sp,Paulistânia
city_br_3779,base.br,base.state_br_mg,Paulistas
city_br_3780,base.br,base.state_br_rs,Paulo Bento
city_br_3781,base.br,base.state_br_sp,Paulo de Faria
city_br_3782,base.br,base.state_br_pr,Paulo Frontin
city_br_3783,base.br,base.state_br_al,Paulo Jacinto
city_br_3784,base.br,base.state_br_sc,Paulo Lopes
city_br_3785,base.br,base.state_br_ma,Paulo Ramos
city_br_3786,base.br,base.state_br_mg,Pavão
city_br_3787,base.br,base.state_br_rs,Paverama
city_br_3788,base.br,base.state_br_pi,Pavussu
city_br_3789,base.br,base.state_br_ba,Pé de Serra
city_br_3790,base.br,base.state_br_pr,Peabiru
city_br_3791,base.br,base.state_br_mg,Peçanha
city_br_3792,base.br,base.state_br_sp,Pederneiras
city_br_3793,base.br,base.state_br_pe,Pedra
city_br_3794,base.br,base.state_br_mg,Pedra Azul
city_br_3795,base.br,base.state_br_sp,Pedra Bela
city_br_3796,base.br,base.state_br_mg,Pedra Bonita
city_br_3797,base.br,base.state_br_ce,Pedra Branca
city_br_3798,base.br,base.state_br_pb,Pedra Branca
city_br_3799,base.br,base.state_br_ap,Pedra Branca do Amapari
city_br_3800,base.br,base.state_br_mg,Pedra do Anta
city_br_3801,base.br,base.state_br_mg,Pedra do Indaiá
city_br_3802,base.br,base.state_br_mg,Pedra Dourada
city_br_3803,base.br,base.state_br_rn,Pedra Grande
city_br_3804,base.br,base.state_br_pb,Pedra Lavrada
city_br_3805,base.br,base.state_br_se,Pedra Mole
city_br_3806,base.br,base.state_br_mt,Pedra Preta
city_br_3807,base.br,base.state_br_rn,Pedra Preta
city_br_3808,base.br,base.state_br_mg,Pedralva
city_br_3809,base.br,base.state_br_sp,Pedranópolis
city_br_3810,base.br,base.state_br_ba,Pedrão
city_br_3811,base.br,base.state_br_rs,Pedras Altas
city_br_3812,base.br,base.state_br_pb,Pedras de Fogo
city_br_3813,base.br,base.state_br_mg,Pedras de Maria da Cruz
city_br_3814,base.br,base.state_br_sc,Pedras Grandes
city_br_3815,base.br,base.state_br_sp,Pedregulho
city_br_3816,base.br,base.state_br_sp,Pedreira
city_br_3817,base.br,base.state_br_ma,Pedreiras
city_br_3818,base.br,base.state_br_se,Pedrinhas
city_br_3819,base.br,base.state_br_sp,Pedrinhas Paulista
city_br_3820,base.br,base.state_br_mg,Pedrinópolis
city_br_3821,base.br,base.state_br_to,Pedro Afonso
city_br_3822,base.br,base.state_br_ba,Pedro Alexandre
city_br_3823,base.br,base.state_br_rn,Pedro Avelino
city_br_3824,base.br,base.state_br_es,Pedro Canário
city_br_3825,base.br,base.state_br_sp,Pedro de Toledo
city_br_3826,base.br,base.state_br_ma,Pedro do Rosário
city_br_3827,base.br,base.state_br_ms,Pedro Gomes
city_br_3828,base.br,base.state_br_pi,Pedro II
city_br_3829,base.br,base.state_br_pi,Pedro Laurentino
city_br_3830,base.br,base.state_br_mg,Pedro Leopoldo
city_br_3831,base.br,base.state_br_rs,Pedro Osório
city_br_3832,base.br,base.state_br_pb,Pedro Régis
city_br_3833,base.br,base.state_br_mg,Pedro Teixeira
city_br_3834,base.br,base.state_br_rn,Pedro Velho
city_br_3835,base.br,base.state_br_to,Peixe
city_br_3836,base.br,base.state_br_pa,Peixe-Boi
city_br_3837,base.br,base.state_br_mt,Peixoto de Azevedo
city_br_3838,base.br,base.state_br_rs,Pejuçara
city_br_3839,base.br,base.state_br_ce,Penaforte
city_br_3840,base.br,base.state_br_ma,Penalva
city_br_3841,base.br,base.state_br_sp,Penápolis
city_br_3842,base.br,base.state_br_rn,Pendências
city_br_3843,base.br,base.state_br_al,Penedo
city_br_3844,base.br,base.state_br_sc,Penha
city_br_3845,base.br,base.state_br_ce,Pentecoste
city_br_3846,base.br,base.state_br_mg,Pequeri
city_br_3847,base.br,base.state_br_mg,Pequi
city_br_3848,base.br,base.state_br_to,Pequizeiro
city_br_3849,base.br,base.state_br_mg,Perdigão
city_br_3850,base.br,base.state_br_mg,Perdizes
city_br_3851,base.br,base.state_br_mg,Perdões
city_br_3852,base.br,base.state_br_sp,Pereira Barreto
city_br_3853,base.br,base.state_br_sp,Pereiras
city_br_3854,base.br,base.state_br_ce,Pereiro
city_br_3855,base.br,base.state_br_ma,Peri Mirim
city_br_3856,base.br,base.state_br_mg,Periquito
city_br_3857,base.br,base.state_br_sc,Peritiba
city_br_3858,base.br,base.state_br_ma,Peritoró
city_br_3859,base.br,base.state_br_pr,Perobal
city_br_3860,base.br,base.state_br_pr,Pérola
city_br_3861,base.br,base.state_br_pr,Pérola d'Oeste
city_br_3862,base.br,base.state_br_go,Perolândia
city_br_3863,base.br,base.state_br_sp,Peruíbe
city_br_3864,base.br,base.state_br_mg,Pescador
city_br_3865,base.br,base.state_br_sc,Pescaria Brava
city_br_3866,base.br,base.state_br_pe,Pesqueira
city_br_3867,base.br,base.state_br_pe,Petrolândia
city_br_3868,base.br,base.state_br_sc,Petrolândia
city_br_3869,base.br,base.state_br_go,Petrolina de Goiás
city_br_3870,base.br,base.state_br_al,Piaçabuçu
city_br_3871,base.br,base.state_br_sp,Piacatu
city_br_3872,base.br,base.state_br_pb,Piancó
city_br_3873,base.br,base.state_br_ba,Piatã
city_br_3874,base.br,base.state_br_mg,Piau
city_br_3875,base.br,base.state_br_rs,Picada Café
city_br_3876,base.br,base.state_br_pa,Piçarra
city_br_3877,base.br,base.state_br_pi,Picos
city_br_3878,base.br,base.state_br_pb,Picuí
city_br_3879,base.br,base.state_br_sp,Piedade
city_br_3880,base.br,base.state_br_mg,Piedade de Caratinga
city_br_3881,base.br,base.state_br_mg,Piedade de Ponte Nova
city_br_3882,base.br,base.state_br_mg,Piedade do Rio Grande
city_br_3883,base.br,base.state_br_mg,Piedade dos Gerais
city_br_3884,base.br,base.state_br_pr,Piên
city_br_3885,base.br,base.state_br_ba,Pilão Arcado
city_br_3886,base.br,base.state_br_al,Pilar
city_br_3887,base.br,base.state_br_pb,Pilar
city_br_3888,base.br,base.state_br_go,Pilar de Goiás
city_br_3889,base.br,base.state_br_sp,Pilar do Sul
city_br_3890,base.br,base.state_br_pb,Pilões
city_br_3891,base.br,base.state_br_rn,Pilões
city_br_3892,base.br,base.state_br_pb,Pilõezinhos
city_br_3893,base.br,base.state_br_mg,Pimenta
city_br_3894,base.br,base.state_br_ro,Pimenta Bueno
city_br_3895,base.br,base.state_br_pi,Pimenteiras
city_br_3896,base.br,base.state_br_ro,Pimenteiras do Oeste
city_br_3897,base.br,base.state_br_ba,Pindaí
city_br_3898,base.br,base.state_br_ma,Pindaré-Mirim
city_br_3899,base.br,base.state_br_al,Pindoba
city_br_3900,base.br,base.state_br_ba,Pindobaçu
city_br_3901,base.br,base.state_br_sp,Pindorama
city_br_3902,base.br,base.state_br_to,Pindorama do Tocantins
city_br_3903,base.br,base.state_br_ce,Pindoretama
city_br_3904,base.br,base.state_br_mg,Pingo-d'Água
city_br_3905,base.br,base.state_br_rs,Pinhal
city_br_3906,base.br,base.state_br_rs,Pinhal da Serra
city_br_3907,base.br,base.state_br_pr,Pinhal de São Bento
city_br_3908,base.br,base.state_br_rs,Pinhal Grande
city_br_3909,base.br,base.state_br_pr,Pinhalão
city_br_3910,base.br,base.state_br_sc,Pinhalzinho
city_br_3911,base.br,base.state_br_sp,Pinhalzinho
city_br_3912,base.br,base.state_br_pr,Pinhão
city_br_3913,base.br,base.state_br_se,Pinhão
city_br_3914,base.br,base.state_br_rj,Pinheiral
city_br_3915,base.br,base.state_br_rs,Pinheirinho do Vale
city_br_3916,base.br,base.state_br_ma,Pinheiro
city_br_3917,base.br,base.state_br_rs,Pinheiro Machado
city_br_3918,base.br,base.state_br_sc,Pinheiro Preto
city_br_3919,base.br,base.state_br_es,Pinheiros
city_br_3920,base.br,base.state_br_ba,Pintadas
city_br_3921,base.br,base.state_br_rs,Pinto Bandeira
city_br_3922,base.br,base.state_br_mg,Pintópolis
city_br_3923,base.br,base.state_br_pi,Pio IX
city_br_3924,base.br,base.state_br_ma,Pio XII
city_br_3925,base.br,base.state_br_sp,Piquerobi
city_br_3926,base.br,base.state_br_ce,Piquet Carneiro
city_br_3927,base.br,base.state_br_sp,Piquete
city_br_3928,base.br,base.state_br_sp,Piracaia
city_br_3929,base.br,base.state_br_go,Piracanjuba
city_br_3930,base.br,base.state_br_mg,Piracema
city_br_3931,base.br,base.state_br_pi,Piracuruca
city_br_3932,base.br,base.state_br_rj,Piraí
city_br_3933,base.br,base.state_br_ba,Piraí do Norte
city_br_3934,base.br,base.state_br_pr,Piraí do Sul
city_br_3935,base.br,base.state_br_sp,Piraju
city_br_3936,base.br,base.state_br_mg,Pirajuba
city_br_3937,base.br,base.state_br_sp,Pirajuí
city_br_3938,base.br,base.state_br_se,Pirambu
city_br_3939,base.br,base.state_br_mg,Piranga
city_br_3940,base.br,base.state_br_sp,Pirangi
city_br_3941,base.br,base.state_br_mg,Piranguçu
city_br_3942,base.br,base.state_br_mg,Piranguinho
city_br_3943,base.br,base.state_br_al,Piranhas
city_br_3944,base.br,base.state_br_go,Piranhas
city_br_3945,base.br,base.state_br_ma,Pirapemas
city_br_3946,base.br,base.state_br_mg,Pirapetinga
city_br_3947,base.br,base.state_br_rs,Pirapó
city_br_3948,base.br,base.state_br_mg,Pirapora
city_br_3949,base.br,base.state_br_sp,Pirapora do Bom Jesus
city_br_3950,base.br,base.state_br_sp,Pirapozinho
city_br_3951,base.br,base.state_br_to,Piraquê
city_br_3952,base.br,base.state_br_sp,Pirassununga
city_br_3953,base.br,base.state_br_rs,Piratini
city_br_3954,base.br,base.state_br_sp,Piratininga
city_br_3955,base.br,base.state_br_sc,Piratuba
city_br_3956,base.br,base.state_br_mg,Piraúba
city_br_3957,base.br,base.state_br_go,Pirenópolis
city_br_3958,base.br,base.state_br_go,Pires do Rio
city_br_3959,base.br,base.state_br_ce,Pires Ferreira
city_br_3960,base.br,base.state_br_ba,Piripá
city_br_3961,base.br,base.state_br_pi,Piripiri
city_br_3962,base.br,base.state_br_ba,Piritiba
city_br_3963,base.br,base.state_br_pb,Pirpirituba
city_br_3964,base.br,base.state_br_pr,Pitanga
city_br_3965,base.br,base.state_br_pr,Pitangueiras
city_br_3966,base.br,base.state_br_sp,Pitangueiras
city_br_3967,base.br,base.state_br_mg,Pitangui
city_br_3968,base.br,base.state_br_pb,Pitimbu
city_br_3969,base.br,base.state_br_to,Pium
city_br_3970,base.br,base.state_br_es,Piúma
city_br_3971,base.br,base.state_br_mg,Piumhi
city_br_3972,base.br,base.state_br_pa,Placas
city_br_3973,base.br,base.state_br_ac,Plácido de Castro
city_br_3974,base.br,base.state_br_pr,Planaltina do Paraná
city_br_3975,base.br,base.state_br_ba,Planaltino
city_br_3976,base.br,base.state_br_ba,Planalto
city_br_3977,base.br,base.state_br_pr,Planalto
city_br_3978,base.br,base.state_br_rs,Planalto
city_br_3979,base.br,base.state_br_sp,Planalto
city_br_3980,base.br,base.state_br_sc,Planalto Alegre
city_br_3981,base.br,base.state_br_mt,Planalto da Serra
city_br_3982,base.br,base.state_br_mg,Planura
city_br_3983,base.br,base.state_br_sp,Platina
city_br_3984,base.br,base.state_br_pe,Poção
city_br_3985,base.br,base.state_br_ma,Poção de Pedras
city_br_3986,base.br,base.state_br_pb,Pocinhos
city_br_3987,base.br,base.state_br_rn,Poço Branco
city_br_3988,base.br,base.state_br_pb,Poço Dantas
city_br_3989,base.br,base.state_br_rs,Poço das Antas
city_br_3990,base.br,base.state_br_al,Poço das Trincheiras
city_br_3991,base.br,base.state_br_pb,Poço de José de Moura
city_br_3992,base.br,base.state_br_mg,Poço Fundo
city_br_3993,base.br,base.state_br_se,Poço Redondo
city_br_3994,base.br,base.state_br_se,Poço Verde
city_br_3995,base.br,base.state_br_ba,Poções
city_br_3996,base.br,base.state_br_mt,Poconé
city_br_3997,base.br,base.state_br_mg,Pocrane
city_br_3998,base.br,base.state_br_ba,Pojuca
city_br_3999,base.br,base.state_br_sp,Poloni
city_br_4000,base.br,base.state_br_pb,Pombal
city_br_4001,base.br,base.state_br_pe,Pombos
city_br_4002,base.br,base.state_br_sc,Pomerode
city_br_4003,base.br,base.state_br_sp,Pompéia
city_br_4004,base.br,base.state_br_mg,Pompéu
city_br_4005,base.br,base.state_br_sp,Pongaí
city_br_4006,base.br,base.state_br_pa,Ponta de Pedras
city_br_4007,base.br,base.state_br_ms,Ponta Porã
city_br_4008,base.br,base.state_br_sp,Pontal
city_br_4009,base.br,base.state_br_mt,Pontal do Araguaia
city_br_4010,base.br,base.state_br_pr,Pontal do Paraná
city_br_4011,base.br,base.state_br_go,Pontalina
city_br_4012,base.br,base.state_br_sp,Pontalinda
city_br_4013,base.br,base.state_br_rs,Pontão
city_br_4014,base.br,base.state_br_sc,Ponte Alta
city_br_4015,base.br,base.state_br_to,Ponte Alta do Bom Jesus
city_br_4016,base.br,base.state_br_sc,Ponte Alta do Norte
city_br_4017,base.br,base.state_br_to,Ponte Alta do Tocantins
city_br_4018,base.br,base.state_br_mt,Ponte Branca
city_br_4019,base.br,base.state_br_mg,Ponte Nova
city_br_4020,base.br,base.state_br_rs,Ponte Preta
city_br_4021,base.br,base.state_br_sc,Ponte Serrada
city_br_4022,base.br,base.state_br_mt,Pontes e Lacerda
city_br_4023,base.br,base.state_br_sp,Pontes Gestal
city_br_4024,base.br,base.state_br_es,Ponto Belo
city_br_4025,base.br,base.state_br_mg,Ponto Chique
city_br_4026,base.br,base.state_br_mg,Ponto dos Volantes
city_br_4027,base.br,base.state_br_ba,Ponto Novo
city_br_4028,base.br,base.state_br_sp,Populina
city_br_4029,base.br,base.state_br_ce,Poranga
city_br_4030,base.br,base.state_br_sp,Porangaba
city_br_4031,base.br,base.state_br_go,Porangatu
city_br_4032,base.br,base.state_br_rj,Porciúncula
city_br_4033,base.br,base.state_br_pr,Porecatu
city_br_4034,base.br,base.state_br_rn,Portalegre
city_br_4035,base.br,base.state_br_rs,Portão
city_br_4036,base.br,base.state_br_go,Porteirão
city_br_4037,base.br,base.state_br_ce,Porteiras
city_br_4038,base.br,base.state_br_mg,Porteirinha
city_br_4039,base.br,base.state_br_pa,Portel
city_br_4040,base.br,base.state_br_go,Portelândia
city_br_4041,base.br,base.state_br_pi,Porto
city_br_4042,base.br,base.state_br_ac,Porto Acre
city_br_4043,base.br,base.state_br_mt,Porto Alegre do Norte
city_br_4044,base.br,base.state_br_pi,Porto Alegre do Piauí
city_br_4045,base.br,base.state_br_to,Porto Alegre do Tocantins
city_br_4046,base.br,base.state_br_pr,Porto Amazonas
city_br_4047,base.br,base.state_br_pr,Porto Barreiro
city_br_4048,base.br,base.state_br_sc,Porto Belo
city_br_4049,base.br,base.state_br_al,Porto Calvo
city_br_4050,base.br,base.state_br_se,Porto da Folha
city_br_4051,base.br,base.state_br_pa,Porto de Moz
city_br_4052,base.br,base.state_br_al,Porto de Pedras
city_br_4053,base.br,base.state_br_rn,Porto do Mangue
city_br_4054,base.br,base.state_br_mt,Porto dos Gaúchos
city_br_4055,base.br,base.state_br_mt,Porto Esperidião
city_br_4056,base.br,base.state_br_mt,Porto Estrela
city_br_4057,base.br,base.state_br_sp,Porto Feliz
city_br_4058,base.br,base.state_br_sp,Porto Ferreira
city_br_4059,base.br,base.state_br_mg,Porto Firme
city_br_4060,base.br,base.state_br_ma,Porto Franco
city_br_4061,base.br,base.state_br_ap,Porto Grande
city_br_4062,base.br,base.state_br_rs,Porto Lucena
city_br_4063,base.br,base.state_br_rs,Porto Mauá
city_br_4064,base.br,base.state_br_ms,Porto Murtinho
city_br_4065,base.br,base.state_br_to,Porto Nacional
city_br_4066,base.br,base.state_br_rj,Porto Real
city_br_4067,base.br,base.state_br_al,Porto Real do Colégio
city_br_4068,base.br,base.state_br_pr,Porto Rico
city_br_4069,base.br,base.state_br_ma,Porto Rico do Maranhão
city_br_4070,base.br,base.state_br_sc,Porto União
city_br_4071,base.br,base.state_br_rs,Porto Vera Cruz
city_br_4072,base.br,base.state_br_pr,Porto Vitória
city_br_4073,base.br,base.state_br_ac,Porto Walter
city_br_4074,base.br,base.state_br_rs,Porto Xavier
city_br_4075,base.br,base.state_br_go,Posse
city_br_4076,base.br,base.state_br_mg,Poté
city_br_4077,base.br,base.state_br_ce,Potengi
city_br_4078,base.br,base.state_br_sp,Potim
city_br_4079,base.br,base.state_br_ba,Potiraguá
city_br_4080,base.br,base.state_br_sp,Potirendaba
city_br_4081,base.br,base.state_br_ce,Potiretama
city_br_4082,base.br,base.state_br_mg,Pouso Alto
city_br_4083,base.br,base.state_br_rs,Pouso Novo
city_br_4084,base.br,base.state_br_sc,Pouso Redondo
city_br_4085,base.br,base.state_br_mt,Poxoréu
city_br_4086,base.br,base.state_br_sp,Pracinha
city_br_4087,base.br,base.state_br_ap,Pracuúba
city_br_4088,base.br,base.state_br_ba,Prado
city_br_4089,base.br,base.state_br_pr,Prado Ferreira
city_br_4090,base.br,base.state_br_sp,Pradópolis
city_br_4091,base.br,base.state_br_mg,Prados
city_br_4092,base.br,base.state_br_sc,Praia Grande
city_br_4093,base.br,base.state_br_to,Praia Norte
city_br_4094,base.br,base.state_br_pa,Prainha
city_br_4095,base.br,base.state_br_pr,Pranchita
city_br_4096,base.br,base.state_br_mg,Prata
city_br_4097,base.br,base.state_br_pb,Prata
city_br_4098,base.br,base.state_br_pi,Prata do Piauí
city_br_4099,base.br,base.state_br_sp,Pratânia
city_br_4100,base.br,base.state_br_mg,Pratápolis
city_br_4101,base.br,base.state_br_mg,Pratinha
city_br_4102,base.br,base.state_br_sp,Presidente Alves
city_br_4103,base.br,base.state_br_mg,Presidente Bernardes
city_br_4104,base.br,base.state_br_sp,Presidente Bernardes
city_br_4105,base.br,base.state_br_sc,Presidente Castello Branco
city_br_4106,base.br,base.state_br_pr,Presidente Castelo Branco
city_br_4107,base.br,base.state_br_ba,Presidente Dutra
city_br_4108,base.br,base.state_br_ma,Presidente Dutra
city_br_4109,base.br,base.state_br_sp,Presidente Epitácio
city_br_4110,base.br,base.state_br_am,Presidente Figueiredo
city_br_4111,base.br,base.state_br_sc,Presidente Getúlio
city_br_4112,base.br,base.state_br_ba,Presidente Jânio Quadros
city_br_4113,base.br,base.state_br_ma,Presidente Juscelino
city_br_4114,base.br,base.state_br_mg,Presidente Juscelino
city_br_4115,base.br,base.state_br_es,Presidente Kennedy
city_br_4116,base.br,base.state_br_to,Presidente Kennedy
city_br_4117,base.br,base.state_br_mg,Presidente Kubitschek
city_br_4118,base.br,base.state_br_rs,Presidente Lucena
city_br_4119,base.br,base.state_br_ma,Presidente Médici
city_br_4120,base.br,base.state_br_ro,Presidente Médici
city_br_4121,base.br,base.state_br_sc,Presidente Nereu
city_br_4122,base.br,base.state_br_mg,Presidente Olegário
city_br_4123,base.br,base.state_br_ma,Presidente Sarney
city_br_4124,base.br,base.state_br_ba,Presidente Tancredo Neves
city_br_4125,base.br,base.state_br_ma,Presidente Vargas
city_br_4126,base.br,base.state_br_sp,Presidente Venceslau
city_br_4127,base.br,base.state_br_pa,Primavera
city_br_4128,base.br,base.state_br_pe,Primavera
city_br_4129,base.br,base.state_br_ro,Primavera de Rondônia
city_br_4130,base.br,base.state_br_mt,Primavera do Leste
city_br_4131,base.br,base.state_br_ma,Primeira Cruz
city_br_4132,base.br,base.state_br_pr,Primeiro de Maio
city_br_4133,base.br,base.state_br_sc,Princesa
city_br_4134,base.br,base.state_br_pb,Princesa Isabel
city_br_4135,base.br,base.state_br_go,Professor Jamil
city_br_4136,base.br,base.state_br_rs,Progresso
city_br_4137,base.br,base.state_br_sp,Promissão
city_br_4138,base.br,base.state_br_se,Propriá
city_br_4139,base.br,base.state_br_rs,Protásio Alves
city_br_4140,base.br,base.state_br_mg,Prudente de Morais
city_br_4141,base.br,base.state_br_pr,Prudentópolis
city_br_4142,base.br,base.state_br_to,Pugmil
city_br_4143,base.br,base.state_br_rn,Pureza
city_br_4144,base.br,base.state_br_rs,Putinga
city_br_4145,base.br,base.state_br_pb,Puxinanã
city_br_4146,base.br,base.state_br_sp,Quadra
city_br_4147,base.br,base.state_br_rs,Quaraí
city_br_4148,base.br,base.state_br_mg,Quartel Geral
city_br_4149,base.br,base.state_br_pr,Quarto Centenário
city_br_4150,base.br,base.state_br_sp,Quatá
city_br_4151,base.br,base.state_br_pr,Quatiguá
city_br_4152,base.br,base.state_br_pa,Quatipuru
city_br_4153,base.br,base.state_br_rj,Quatis
city_br_4154,base.br,base.state_br_pr,Quatro Barras
city_br_4155,base.br,base.state_br_rs,Quatro Irmãos
city_br_4156,base.br,base.state_br_pr,Quatro Pontes
city_br_4157,base.br,base.state_br_al,Quebrangulo
city_br_4158,base.br,base.state_br_pr,Quedas do Iguaçu
city_br_4159,base.br,base.state_br_pi,Queimada Nova
city_br_4160,base.br,base.state_br_ba,Queimadas
city_br_4161,base.br,base.state_br_pb,Queimadas
city_br_4162,base.br,base.state_br_sp,Queiroz
city_br_4163,base.br,base.state_br_sp,Queluz
city_br_4164,base.br,base.state_br_mg,Queluzito
city_br_4165,base.br,base.state_br_mt,Querência
city_br_4166,base.br,base.state_br_pr,Querência do Norte
city_br_4167,base.br,base.state_br_rs,Quevedos
city_br_4168,base.br,base.state_br_ba,Quijingue
city_br_4169,base.br,base.state_br_sc,Quilombo
city_br_4170,base.br,base.state_br_pr,Quinta do Sol
city_br_4171,base.br,base.state_br_sp,Quintana
city_br_4172,base.br,base.state_br_rs,Quinze de Novembro
city_br_4173,base.br,base.state_br_pe,Quipapá
city_br_4174,base.br,base.state_br_go,Quirinópolis
city_br_4175,base.br,base.state_br_rj,Quissamã
city_br_4176,base.br,base.state_br_pr,Quitandinha
city_br_4177,base.br,base.state_br_ce,Quiterianópolis
city_br_4178,base.br,base.state_br_pb,Quixaba
city_br_4179,base.br,base.state_br_pe,Quixaba
city_br_4180,base.br,base.state_br_ba,Quixabeira
city_br_4181,base.br,base.state_br_ce,Quixadá
city_br_4182,base.br,base.state_br_ce,Quixelô
city_br_4183,base.br,base.state_br_ce,Quixeramobim
city_br_4184,base.br,base.state_br_ce,Quixeré
city_br_4185,base.br,base.state_br_rn,Rafael Fernandes
city_br_4186,base.br,base.state_br_rn,Rafael Godeiro
city_br_4187,base.br,base.state_br_ba,Rafael Jambeiro
city_br_4188,base.br,base.state_br_sp,Rafard
city_br_4189,base.br,base.state_br_pr,Ramilândia
city_br_4190,base.br,base.state_br_sp,Rancharia
city_br_4191,base.br,base.state_br_pr,Rancho Alegre
city_br_4192,base.br,base.state_br_pr,Rancho Alegre D'Oeste
city_br_4193,base.br,base.state_br_sc,Rancho Queimado
city_br_4194,base.br,base.state_br_ma,Raposa
city_br_4195,base.br,base.state_br_mg,Raposos
city_br_4196,base.br,base.state_br_mg,Raul Soares
city_br_4197,base.br,base.state_br_pr,Realeza
city_br_4198,base.br,base.state_br_pr,Rebouças
city_br_4199,base.br,base.state_br_mg,Recreio
city_br_4200,base.br,base.state_br_to,Recursolândia
city_br_4201,base.br,base.state_br_ce,Redenção
city_br_4202,base.br,base.state_br_pa,Redenção
city_br_4203,base.br,base.state_br_sp,Redenção da Serra
city_br_4204,base.br,base.state_br_pi,Redenção do Gurguéia
city_br_4205,base.br,base.state_br_rs,Redentora
city_br_4206,base.br,base.state_br_mg,Reduto
city_br_4207,base.br,base.state_br_pi,Regeneração
city_br_4208,base.br,base.state_br_sp,Regente Feijó
city_br_4209,base.br,base.state_br_sp,Reginópolis
city_br_4210,base.br,base.state_br_sp,Registro
city_br_4211,base.br,base.state_br_rs,Relvado
city_br_4212,base.br,base.state_br_ba,Remanso
city_br_4213,base.br,base.state_br_pb,Remígio
city_br_4214,base.br,base.state_br_pr,Renascença
city_br_4215,base.br,base.state_br_ce,Reriutaba
city_br_4216,base.br,base.state_br_mg,Resende Costa
city_br_4217,base.br,base.state_br_pr,Reserva
city_br_4218,base.br,base.state_br_mt,Reserva do Cabaçal
city_br_4219,base.br,base.state_br_pr,Reserva do Iguaçu
city_br_4220,base.br,base.state_br_mg,Resplendor
city_br_4221,base.br,base.state_br_mg,Ressaquinha
city_br_4222,base.br,base.state_br_sp,Restinga
city_br_4223,base.br,base.state_br_rs,Restinga Sêca
city_br_4224,base.br,base.state_br_ba,Retirolândia
city_br_4225,base.br,base.state_br_ma,Riachão
city_br_4226,base.br,base.state_br_pb,Riachão
city_br_4227,base.br,base.state_br_ba,Riachão das Neves
city_br_4228,base.br,base.state_br_pb,Riachão do Bacamarte
city_br_4229,base.br,base.state_br_se,Riachão do Dantas
city_br_4230,base.br,base.state_br_ba,Riachão do Jacuípe
city_br_4231,base.br,base.state_br_pb,Riachão do Poço
city_br_4232,base.br,base.state_br_mg,Riachinho
city_br_4233,base.br,base.state_br_to,Riachinho
city_br_4234,base.br,base.state_br_rn,Riacho da Cruz
city_br_4235,base.br,base.state_br_pe,Riacho das Almas
city_br_4236,base.br,base.state_br_ba,Riacho de Santana
city_br_4237,base.br,base.state_br_rn,Riacho de Santana
city_br_4238,base.br,base.state_br_pb,Riacho de Santo Antônio
city_br_4239,base.br,base.state_br_pb,Riacho dos Cavalos
city_br_4240,base.br,base.state_br_mg,Riacho dos Machados
city_br_4241,base.br,base.state_br_pi,Riacho Frio
city_br_4242,base.br,base.state_br_rn,Riachuelo
city_br_4243,base.br,base.state_br_se,Riachuelo
city_br_4244,base.br,base.state_br_go,Rialma
city_br_4245,base.br,base.state_br_go,Rianápolis
city_br_4246,base.br,base.state_br_ma,Ribamar Fiquene
city_br_4247,base.br,base.state_br_ms,Ribas do Rio Pardo
city_br_4248,base.br,base.state_br_sp,Ribeira
city_br_4249,base.br,base.state_br_ba,Ribeira do Amparo
city_br_4250,base.br,base.state_br_pi,Ribeira do Piauí
city_br_4251,base.br,base.state_br_ba,Ribeira do Pombal
city_br_4252,base.br,base.state_br_pe,Ribeirão
city_br_4253,base.br,base.state_br_sp,Ribeirão Bonito
city_br_4254,base.br,base.state_br_sp,Ribeirão Branco
city_br_4255,base.br,base.state_br_mt,Ribeirão Cascalheira
city_br_4256,base.br,base.state_br_pr,Ribeirão Claro
city_br_4257,base.br,base.state_br_sp,Ribeirão Corrente
city_br_4258,base.br,base.state_br_ba,Ribeirão do Largo
city_br_4259,base.br,base.state_br_pr,Ribeirão do Pinhal
city_br_4260,base.br,base.state_br_sp,Ribeirão do Sul
city_br_4261,base.br,base.state_br_sp,Ribeirão dos Índios
city_br_4262,base.br,base.state_br_sp,Ribeirão Grande
city_br_4263,base.br,base.state_br_mg,Ribeirão Vermelho
city_br_4264,base.br,base.state_br_mt,Ribeirãozinho
city_br_4265,base.br,base.state_br_pi,Ribeiro Gonçalves
city_br_4266,base.br,base.state_br_se,Ribeirópolis
city_br_4267,base.br,base.state_br_sp,Rifaina
city_br_4268,base.br,base.state_br_sp,Rincão
city_br_4269,base.br,base.state_br_sp,Rinópolis
city_br_4270,base.br,base.state_br_mg,Rio Acima
city_br_4271,base.br,base.state_br_pr,Rio Azul
city_br_4272,base.br,base.state_br_es,Rio Bananal
city_br_4273,base.br,base.state_br_pr,Rio Bom
city_br_4274,base.br,base.state_br_rj,Rio Bonito
city_br_4275,base.br,base.state_br_pr,Rio Bonito do Iguaçu
city_br_4276,base.br,base.state_br_mt,Rio Branco
city_br_4277,base.br,base.state_br_pr,Rio Branco do Ivaí
city_br_4278,base.br,base.state_br_pr,Rio Branco do Sul
city_br_4279,base.br,base.state_br_ms,Rio Brilhante
city_br_4280,base.br,base.state_br_mg,Rio Casca
city_br_4281,base.br,base.state_br_rj,Rio Claro
city_br_4282,base.br,base.state_br_ro,Rio Crespo
city_br_4283,base.br,base.state_br_to,Rio da Conceição
city_br_4284,base.br,base.state_br_sc,Rio das Antas
city_br_4285,base.br,base.state_br_rj,Rio das Flores
city_br_4286,base.br,base.state_br_sp,Rio das Pedras
city_br_4287,base.br,base.state_br_ba,Rio de Contas
city_br_4288,base.br,base.state_br_ba,Rio do Antônio
city_br_4289,base.br,base.state_br_sc,Rio do Campo
city_br_4290,base.br,base.state_br_rn,Rio do Fogo
city_br_4291,base.br,base.state_br_sc,Rio do Oeste
city_br_4292,base.br,base.state_br_ba,Rio do Pires
city_br_4293,base.br,base.state_br_mg,Rio do Prado
city_br_4294,base.br,base.state_br_sc,Rio do Sul
city_br_4295,base.br,base.state_br_mg,Rio Doce
city_br_4296,base.br,base.state_br_to,Rio dos Bois
city_br_4297,base.br,base.state_br_sc,Rio dos Cedros
city_br_4298,base.br,base.state_br_rs,Rio dos Índios
city_br_4299,base.br,base.state_br_mg,Rio Espera
city_br_4300,base.br,base.state_br_pe,Rio Formoso
city_br_4301,base.br,base.state_br_sc,Rio Fortuna
city_br_4302,base.br,base.state_br_sp,Rio Grande da Serra
city_br_4303,base.br,base.state_br_pi,Rio Grande do Piauí
city_br_4304,base.br,base.state_br_al,Rio Largo
city_br_4305,base.br,base.state_br_mg,Rio Manso
city_br_4306,base.br,base.state_br_pa,Rio Maria
city_br_4307,base.br,base.state_br_sc,Rio Negrinho
city_br_4308,base.br,base.state_br_ms,Rio Negro
city_br_4309,base.br,base.state_br_pr,Rio Negro
city_br_4310,base.br,base.state_br_mg,Rio Novo
city_br_4311,base.br,base.state_br_es,Rio Novo do Sul
city_br_4312,base.br,base.state_br_mg,Rio Paranaíba
city_br_4313,base.br,base.state_br_rs,Rio Pardo
city_br_4314,base.br,base.state_br_mg,Rio Pardo de Minas
city_br_4315,base.br,base.state_br_mg,Rio Piracicaba
city_br_4316,base.br,base.state_br_mg,Rio Pomba
city_br_4317,base.br,base.state_br_mg,Rio Preto
city_br_4318,base.br,base.state_br_am,Rio Preto da Eva
city_br_4319,base.br,base.state_br_go,Rio Quente
city_br_4320,base.br,base.state_br_ba,Rio Real
city_br_4321,base.br,base.state_br_sc,Rio Rufino
city_br_4322,base.br,base.state_br_to,Rio Sono
city_br_4323,base.br,base.state_br_pb,Rio Tinto
city_br_4324,base.br,base.state_br_ms,Rio Verde de Mato Grosso
city_br_4325,base.br,base.state_br_mg,Rio Vermelho
city_br_4326,base.br,base.state_br_sp,Riolândia
city_br_4327,base.br,base.state_br_rs,Riozinho
city_br_4328,base.br,base.state_br_sc,Riqueza
city_br_4329,base.br,base.state_br_mg,Ritápolis
city_br_4330,base.br,base.state_br_sp,Riversul
city_br_4331,base.br,base.state_br_rs,Roca Sales
city_br_4332,base.br,base.state_br_ms,Rochedo
city_br_4333,base.br,base.state_br_mg,Rochedo de Minas
city_br_4334,base.br,base.state_br_sc,Rodeio
city_br_4335,base.br,base.state_br_rs,Rodeio Bonito
city_br_4336,base.br,base.state_br_mg,Rodeiro
city_br_4337,base.br,base.state_br_ba,Rodelas
city_br_4338,base.br,base.state_br_rn,Rodolfo Fernandes
city_br_4339,base.br,base.state_br_ac,Rodrigues Alves
city_br_4340,base.br,base.state_br_rs,Rolador
city_br_4341,base.br,base.state_br_pr,Rolândia
city_br_4342,base.br,base.state_br_rs,Rolante
city_br_4343,base.br,base.state_br_ro,Rolim de Moura
city_br_4344,base.br,base.state_br_mg,Romaria
city_br_4345,base.br,base.state_br_sc,Romelândia
city_br_4346,base.br,base.state_br_pr,Roncador
city_br_4347,base.br,base.state_br_rs,Ronda Alta
city_br_4348,base.br,base.state_br_rs,Rondinha
city_br_4349,base.br,base.state_br_mt,Rondolândia
city_br_4350,base.br,base.state_br_pr,Rondon
city_br_4351,base.br,base.state_br_pa,Rondon do Pará
city_br_4352,base.br,base.state_br_rs,Roque Gonzales
city_br_4353,base.br,base.state_br_rr,Rorainópolis
city_br_4354,base.br,base.state_br_sp,Rosana
city_br_4355,base.br,base.state_br_ma,Rosário
city_br_4356,base.br,base.state_br_mg,Rosário da Limeira
city_br_4357,base.br,base.state_br_se,Rosário do Catete
city_br_4358,base.br,base.state_br_pr,Rosário do Ivaí
city_br_4359,base.br,base.state_br_rs,Rosário do Sul
city_br_4360,base.br,base.state_br_mt,Rosário Oeste
city_br_4361,base.br,base.state_br_sp,Roseira
city_br_4362,base.br,base.state_br_al,Roteiro
city_br_4363,base.br,base.state_br_mg,Rubelita
city_br_4364,base.br,base.state_br_sp,Rubiácea
city_br_4365,base.br,base.state_br_go,Rubiataba
city_br_4366,base.br,base.state_br_mg,Rubim
city_br_4367,base.br,base.state_br_sp,Rubinéia
city_br_4368,base.br,base.state_br_pa,Rurópolis
city_br_4369,base.br,base.state_br_ce,Russas
city_br_4370,base.br,base.state_br_ba,Ruy Barbosa
city_br_4371,base.br,base.state_br_rn,Ruy Barbosa
city_br_4372,base.br,base.state_br_pr,Sabáudia
city_br_4373,base.br,base.state_br_sp,Sabino
city_br_4374,base.br,base.state_br_mg,Sabinópolis
city_br_4375,base.br,base.state_br_ce,Saboeiro
city_br_4376,base.br,base.state_br_mg,Sacramento
city_br_4377,base.br,base.state_br_rs,Sagrada Família
city_br_4378,base.br,base.state_br_sp,Sagres
city_br_4379,base.br,base.state_br_pe,Sairé
city_br_4380,base.br,base.state_br_rs,Saldanha Marinho
city_br_4381,base.br,base.state_br_sp,Sales
city_br_4382,base.br,base.state_br_sp,Sales Oliveira
city_br_4383,base.br,base.state_br_sp,Salesópolis
city_br_4384,base.br,base.state_br_sc,Salete
city_br_4385,base.br,base.state_br_pb,Salgadinho
city_br_4386,base.br,base.state_br_pe,Salgadinho
city_br_4387,base.br,base.state_br_se,Salgado
city_br_4388,base.br,base.state_br_pb,Salgado de São Félix
city_br_4389,base.br,base.state_br_pr,Salgado Filho
city_br_4390,base.br,base.state_br_pe,Salgueiro
city_br_4391,base.br,base.state_br_mg,Salinas
city_br_4392,base.br,base.state_br_ba,Salinas da Margarida
city_br_4393,base.br,base.state_br_pa,Salinópolis
city_br_4394,base.br,base.state_br_ce,Salitre
city_br_4395,base.br,base.state_br_sp,Salmourão
city_br_4396,base.br,base.state_br_pe,Saloá
city_br_4397,base.br,base.state_br_sc,Saltinho
city_br_4398,base.br,base.state_br_sp,Saltinho
city_br_4399,base.br,base.state_br_mg,Salto da Divisa
city_br_4400,base.br,base.state_br_sp,Salto de Pirapora
city_br_4401,base.br,base.state_br_mt,Salto do Céu
city_br_4402,base.br,base.state_br_pr,Salto do Itararé
city_br_4403,base.br,base.state_br_rs,Salto do Jacuí
city_br_4404,base.br,base.state_br_pr,Salto do Lontra
city_br_4405,base.br,base.state_br_sp,Salto Grande
city_br_4406,base.br,base.state_br_sc,Salto Veloso
city_br_4407,base.br,base.state_br_rs,Salvador das Missões
city_br_4408,base.br,base.state_br_rs,Salvador do Sul
city_br_4409,base.br,base.state_br_pa,Salvaterra
city_br_4410,base.br,base.state_br_ma,Sambaíba
city_br_4411,base.br,base.state_br_to,Sampaio
city_br_4412,base.br,base.state_br_rs,Sananduva
city_br_4413,base.br,base.state_br_go,Sanclerlândia
city_br_4414,base.br,base.state_br_to,Sandolândia
city_br_4415,base.br,base.state_br_sp,Sandovalina
city_br_4416,base.br,base.state_br_sc,Sangão
city_br_4417,base.br,base.state_br_pe,Sanharó
city_br_4418,base.br,base.state_br_rs,Sant'Ana do Livramento
city_br_4419,base.br,base.state_br_sp,Santa Adélia
city_br_4420,base.br,base.state_br_sp,Santa Albertina
city_br_4421,base.br,base.state_br_pr,Santa Amélia
city_br_4422,base.br,base.state_br_ba,Santa Bárbara
city_br_4423,base.br,base.state_br_mg,Santa Bárbara
city_br_4424,base.br,base.state_br_go,Santa Bárbara de Goiás
city_br_4425,base.br,base.state_br_mg,Santa Bárbara do Leste
city_br_4426,base.br,base.state_br_mg,Santa Bárbara do Monte Verde
city_br_4427,base.br,base.state_br_pa,Santa Bárbara do Pará
city_br_4428,base.br,base.state_br_rs,Santa Bárbara do Sul
city_br_4429,base.br,base.state_br_mg,Santa Bárbara do Tugúrio
city_br_4430,base.br,base.state_br_sp,Santa Branca
city_br_4431,base.br,base.state_br_ba,Santa Brígida
city_br_4432,base.br,base.state_br_mt,Santa Carmem
city_br_4433,base.br,base.state_br_pb,Santa Cecília
city_br_4434,base.br,base.state_br_sc,Santa Cecília
city_br_4435,base.br,base.state_br_pr,Santa Cecília do Pavão
city_br_4436,base.br,base.state_br_rs,Santa Cecília do Sul
city_br_4437,base.br,base.state_br_sp,Santa Clara d'Oeste
city_br_4438,base.br,base.state_br_rs,Santa Clara do Sul
city_br_4439,base.br,base.state_br_pb,Santa Cruz
city_br_4440,base.br,base.state_br_pe,Santa Cruz
city_br_4441,base.br,base.state_br_rn,Santa Cruz
city_br_4442,base.br,base.state_br_ba,Santa Cruz Cabrália
city_br_4443,base.br,base.state_br_pe,Santa Cruz da Baixa Verde
city_br_4444,base.br,base.state_br_sp,Santa Cruz da Conceição
city_br_4445,base.br,base.state_br_sp,Santa Cruz da Esperança
city_br_4446,base.br,base.state_br_ba,Santa Cruz da Vitória
city_br_4447,base.br,base.state_br_sp,Santa Cruz das Palmeiras
city_br_4448,base.br,base.state_br_go,Santa Cruz de Goiás
city_br_4449,base.br,base.state_br_mg,Santa Cruz de Minas
city_br_4450,base.br,base.state_br_pr,Santa Cruz de Monte Castelo
city_br_4451,base.br,base.state_br_mg,Santa Cruz de Salinas
city_br_4452,base.br,base.state_br_pa,Santa Cruz do Arari
city_br_4453,base.br,base.state_br_pe,Santa Cruz do Capibaribe
city_br_4454,base.br,base.state_br_mg,Santa Cruz do Escalvado
city_br_4455,base.br,base.state_br_pi,Santa Cruz do Piauí
city_br_4456,base.br,base.state_br_sp,Santa Cruz do Rio Pardo
city_br_4457,base.br,base.state_br_mt,Santa Cruz do Xingu
city_br_4458,base.br,base.state_br_pi,Santa Cruz dos Milagres
city_br_4459,base.br,base.state_br_mg,Santa Efigênia de Minas
city_br_4460,base.br,base.state_br_sp,Santa Ernestina
city_br_4461,base.br,base.state_br_pr,Santa Fé
city_br_4462,base.br,base.state_br_go,Santa Fé de Goiás
city_br_4463,base.br,base.state_br_mg,Santa Fé de Minas
city_br_4464,base.br,base.state_br_to,Santa Fé do Araguaia
city_br_4465,base.br,base.state_br_sp,Santa Fé do Sul
city_br_4466,base.br,base.state_br_pe,Santa Filomena
city_br_4467,base.br,base.state_br_pi,Santa Filomena
city_br_4468,base.br,base.state_br_ma,Santa Filomena do Maranhão
city_br_4469,base.br,base.state_br_sp,Santa Gertrudes
city_br_4470,base.br,base.state_br_ma,Santa Helena
city_br_4471,base.br,base.state_br_pb,Santa Helena
city_br_4472,base.br,base.state_br_pr,Santa Helena
city_br_4473,base.br,base.state_br_sc,Santa Helena
city_br_4474,base.br,base.state_br_go,Santa Helena de Goiás
city_br_4475,base.br,base.state_br_mg,Santa Helena de Minas
city_br_4476,base.br,base.state_br_ba,Santa Inês
city_br_4477,base.br,base.state_br_ma,Santa Inês
city_br_4478,base.br,base.state_br_pb,Santa Inês
city_br_4479,base.br,base.state_br_pr,Santa Inês
city_br_4480,base.br,base.state_br_go,Santa Isabel
city_br_4481,base.br,base.state_br_sp,Santa Isabel
city_br_4482,base.br,base.state_br_pr,Santa Isabel do Ivaí
city_br_4483,base.br,base.state_br_am,Santa Isabel do Rio Negro
city_br_4484,base.br,base.state_br_pr,Santa Izabel do Oeste
city_br_4485,base.br,base.state_br_pa,Santa Izabel do Pará
city_br_4486,base.br,base.state_br_mg,Santa Juliana
city_br_4487,base.br,base.state_br_es,Santa Leopoldina
city_br_4488,base.br,base.state_br_pr,Santa Lúcia
city_br_4489,base.br,base.state_br_sp,Santa Lúcia
city_br_4490,base.br,base.state_br_pi,Santa Luz
city_br_4491,base.br,base.state_br_ba,Santa Luzia
city_br_4492,base.br,base.state_br_ma,Santa Luzia
city_br_4493,base.br,base.state_br_pb,Santa Luzia
city_br_4494,base.br,base.state_br_ro,Santa Luzia D'Oeste
city_br_4495,base.br,base.state_br_se,Santa Luzia do Itanhy
city_br_4496,base.br,base.state_br_al,Santa Luzia do Norte
city_br_4497,base.br,base.state_br_pa,Santa Luzia do Pará
city_br_4498,base.br,base.state_br_ma,Santa Luzia do Paruá
city_br_4499,base.br,base.state_br_mg,Santa Margarida
city_br_4500,base.br,base.state_br_rs,Santa Margarida do Sul
city_br_4501,base.br,base.state_br_rn,Santa Maria
city_br_4502,base.br,base.state_br_pe,Santa Maria da Boa Vista
city_br_4503,base.br,base.state_br_sp,Santa Maria da Serra
city_br_4504,base.br,base.state_br_ba,Santa Maria da Vitória
city_br_4505,base.br,base.state_br_pa,Santa Maria das Barreiras
city_br_4506,base.br,base.state_br_mg,Santa Maria de Itabira
city_br_4507,base.br,base.state_br_es,Santa Maria de Jetibá
city_br_4508,base.br,base.state_br_pe,Santa Maria do Cambucá
city_br_4509,base.br,base.state_br_rs,Santa Maria do Herval
city_br_4510,base.br,base.state_br_pr,Santa Maria do Oeste
city_br_4511,base.br,base.state_br_pa,Santa Maria do Pará
city_br_4512,base.br,base.state_br_mg,Santa Maria do Salto
city_br_4513,base.br,base.state_br_mg,Santa Maria do Suaçuí
city_br_4514,base.br,base.state_br_to,Santa Maria do Tocantins
city_br_4515,base.br,base.state_br_rj,Santa Maria Madalena
city_br_4516,base.br,base.state_br_pr,Santa Mariana
city_br_4517,base.br,base.state_br_sp,Santa Mercedes
city_br_4518,base.br,base.state_br_pr,Santa Mônica
city_br_4519,base.br,base.state_br_ce,Santa Quitéria
city_br_4520,base.br,base.state_br_ma,Santa Quitéria do Maranhão
city_br_4521,base.br,base.state_br_ma,Santa Rita
city_br_4522,base.br,base.state_br_sp,Santa Rita d'Oeste
city_br_4523,base.br,base.state_br_mg,Santa Rita de Caldas
city_br_4524,base.br,base.state_br_ba,Santa Rita de Cássia
city_br_4525,base.br,base.state_br_mg,Santa Rita de Ibitipoca
city_br_4526,base.br,base.state_br_mg,Santa Rita de Jacutinga
city_br_4527,base.br,base.state_br_mg,Santa Rita de Minas
city_br_4528,base.br,base.state_br_go,Santa Rita do Araguaia
city_br_4529,base.br,base.state_br_mg,Santa Rita do Itueto
city_br_4530,base.br,base.state_br_go,Santa Rita do Novo Destino
city_br_4531,base.br,base.state_br_ms,Santa Rita do Pardo
city_br_4532,base.br,base.state_br_sp,Santa Rita do Passa Quatro
city_br_4533,base.br,base.state_br_mg,Santa Rita do Sapucaí
city_br_4534,base.br,base.state_br_to,Santa Rita do Tocantins
city_br_4535,base.br,base.state_br_mt,Santa Rita do Trivelato
city_br_4536,base.br,base.state_br_rs,Santa Rosa
city_br_4537,base.br,base.state_br_mg,Santa Rosa da Serra
city_br_4538,base.br,base.state_br_go,Santa Rosa de Goiás
city_br_4539,base.br,base.state_br_sc,Santa Rosa de Lima
city_br_4540,base.br,base.state_br_se,Santa Rosa de Lima
city_br_4541,base.br,base.state_br_sp,Santa Rosa de Viterbo
city_br_4542,base.br,base.state_br_pi,Santa Rosa do Piauí
city_br_4543,base.br,base.state_br_ac,Santa Rosa do Purus
city_br_4544,base.br,base.state_br_sc,Santa Rosa do Sul
city_br_4545,base.br,base.state_br_to,Santa Rosa do Tocantins
city_br_4546,base.br,base.state_br_sp,Santa Salete
city_br_4547,base.br,base.state_br_es,Santa Teresa
city_br_4548,base.br,base.state_br_ba,Santa Teresinha
city_br_4549,base.br,base.state_br_pb,Santa Teresinha
city_br_4550,base.br,base.state_br_rs,Santa Tereza
city_br_4551,base.br,base.state_br_go,Santa Tereza de Goiás
city_br_4552,base.br,base.state_br_pr,Santa Tereza do Oeste
city_br_4553,base.br,base.state_br_to,Santa Tereza do Tocantins
city_br_4554,base.br,base.state_br_mt,Santa Terezinha
city_br_4555,base.br,base.state_br_pe,Santa Terezinha
city_br_4556,base.br,base.state_br_sc,Santa Terezinha
city_br_4557,base.br,base.state_br_go,Santa Terezinha de Goiás
city_br_4558,base.br,base.state_br_pr,Santa Terezinha de Itaipu
city_br_4559,base.br,base.state_br_sc,Santa Terezinha do Progresso
city_br_4560,base.br,base.state_br_to,Santa Terezinha do Tocantins
city_br_4561,base.br,base.state_br_mg,Santa Vitória
city_br_4562,base.br,base.state_br_rs,Santa Vitória do Palmar
city_br_4563,base.br,base.state_br_ba,Santaluz
city_br_4564,base.br,base.state_br_ba,Santana
city_br_4565,base.br,base.state_br_rs,Santana da Boa Vista
city_br_4566,base.br,base.state_br_sp,Santana da Ponte Pensa
city_br_4567,base.br,base.state_br_mg,Santana da Vargem
city_br_4568,base.br,base.state_br_mg,Santana de Cataguases
city_br_4569,base.br,base.state_br_pb,Santana de Mangueira
city_br_4570,base.br,base.state_br_mg,Santana de Pirapama
city_br_4571,base.br,base.state_br_ce,Santana do Acaraú
city_br_4572,base.br,base.state_br_pa,Santana do Araguaia
city_br_4573,base.br,base.state_br_ce,Santana do Cariri
city_br_4574,base.br,base.state_br_mg,Santana do Deserto
city_br_4575,base.br,base.state_br_mg,Santana do Garambéu
city_br_4576,base.br,base.state_br_al,Santana do Ipanema
city_br_4577,base.br,base.state_br_pr,Santana do Itararé
city_br_4578,base.br,base.state_br_mg,Santana do Jacaré
city_br_4579,base.br,base.state_br_mg,Santana do Manhuaçu
city_br_4580,base.br,base.state_br_ma,Santana do Maranhão
city_br_4581,base.br,base.state_br_rn,Santana do Matos
city_br_4582,base.br,base.state_br_al,Santana do Mundaú
city_br_4583,base.br,base.state_br_mg,Santana do Paraíso
city_br_4584,base.br,base.state_br_pi,Santana do Piauí
city_br_4585,base.br,base.state_br_mg,Santana do Riacho
city_br_4586,base.br,base.state_br_se,Santana do São Francisco
city_br_4587,base.br,base.state_br_rn,Santana do Seridó
city_br_4588,base.br,base.state_br_pb,Santana dos Garrotes
city_br_4589,base.br,base.state_br_mg,Santana dos Montes
city_br_4590,base.br,base.state_br_ba,Santanópolis
city_br_4591,base.br,base.state_br_pa,Santarém Novo
city_br_4592,base.br,base.state_br_rs,Santiago
city_br_4593,base.br,base.state_br_sc,Santiago do Sul
city_br_4594,base.br,base.state_br_mt,Santo Afonso
city_br_4595,base.br,base.state_br_ba,Santo Amaro
city_br_4596,base.br,base.state_br_sc,Santo Amaro da Imperatriz
city_br_4597,base.br,base.state_br_se,Santo Amaro das Brotas
city_br_4598,base.br,base.state_br_ma,Santo Amaro do Maranhão
city_br_4599,base.br,base.state_br_sp,Santo Anastácio
city_br_4600,base.br,base.state_br_pb,Santo André
city_br_4601,base.br,base.state_br_rs,Santo Ângelo
city_br_4602,base.br,base.state_br_rn,Santo Antônio
city_br_4603,base.br,base.state_br_sp,Santo Antônio da Alegria
city_br_4604,base.br,base.state_br_go,Santo Antônio da Barra
city_br_4605,base.br,base.state_br_rs,Santo Antônio da Patrulha
city_br_4606,base.br,base.state_br_pr,Santo Antônio da Platina
city_br_4607,base.br,base.state_br_rs,Santo Antônio das Missões
city_br_4608,base.br,base.state_br_go,Santo Antônio de Goiás
city_br_4609,base.br,base.state_br_pi,Santo Antônio de Lisboa
city_br_4610,base.br,base.state_br_rj,Santo Antônio de Pádua
city_br_4611,base.br,base.state_br_sp,Santo Antônio de Posse
city_br_4612,base.br,base.state_br_mg,Santo Antônio do Amparo
city_br_4613,base.br,base.state_br_sp,Santo Antônio do Aracanguá
city_br_4614,base.br,base.state_br_mg,Santo Antônio do Aventureiro
city_br_4615,base.br,base.state_br_pr,Santo Antônio do Caiuá
city_br_4616,base.br,base.state_br_go,Santo Antônio do Descoberto
city_br_4617,base.br,base.state_br_mg,Santo Antônio do Grama
city_br_4618,base.br,base.state_br_am,Santo Antônio do Içá
city_br_4619,base.br,base.state_br_mg,Santo Antônio do Itambé
city_br_4620,base.br,base.state_br_mg,Santo Antônio do Jacinto
city_br_4621,base.br,base.state_br_sp,Santo Antônio do Jardim
city_br_4622,base.br,base.state_br_mt,Santo Antônio do Leste
city_br_4623,base.br,base.state_br_mt,Santo Antônio do Leverger
city_br_4624,base.br,base.state_br_mg,Santo Antônio do Monte
city_br_4625,base.br,base.state_br_rs,Santo Antônio do Palma
city_br_4626,base.br,base.state_br_pr,Santo Antônio do Paraíso
city_br_4627,base.br,base.state_br_sp,Santo Antônio do Pinhal
city_br_4628,base.br,base.state_br_rs,Santo Antônio do Planalto
city_br_4629,base.br,base.state_br_mg,Santo Antônio do Retiro
city_br_4630,base.br,base.state_br_mg,Santo Antônio do Rio Abaixo
city_br_4631,base.br,base.state_br_pr,Santo Antônio do Sudoeste
city_br_4632,base.br,base.state_br_pa,Santo Antônio do Tauá
city_br_4633,base.br,base.state_br_ma,Santo Antônio dos Lopes
city_br_4634,base.br,base.state_br_pi,Santo Antônio dos Milagres
city_br_4635,base.br,base.state_br_rs,Santo Augusto
city_br_4636,base.br,base.state_br_rs,Santo Cristo
city_br_4637,base.br,base.state_br_ba,Santo Estêvão
city_br_4638,base.br,base.state_br_sp,Santo Expedito
city_br_4639,base.br,base.state_br_rs,Santo Expedito do Sul
city_br_4640,base.br,base.state_br_mg,Santo Hipólito
city_br_4641,base.br,base.state_br_pr,Santo Inácio
city_br_4642,base.br,base.state_br_pi,Santo Inácio do Piauí
city_br_4643,base.br,base.state_br_sp,Santópolis do Aguapeí
city_br_4644,base.br,base.state_br_mg,Santos Dumont
city_br_4645,base.br,base.state_br_ce,São Benedito
city_br_4646,base.br,base.state_br_ma,São Benedito do Rio Preto
city_br_4647,base.br,base.state_br_pe,São Benedito do Sul
city_br_4648,base.br,base.state_br_pb,São Bentinho
city_br_4649,base.br,base.state_br_ma,São Bento
city_br_4650,base.br,base.state_br_pb,São Bento
city_br_4651,base.br,base.state_br_mg,São Bento Abade
city_br_4652,base.br,base.state_br_rn,São Bento do Norte
city_br_4653,base.br,base.state_br_sp,São Bento do Sapucaí
city_br_4654,base.br,base.state_br_sc,São Bento do Sul
city_br_4655,base.br,base.state_br_to,São Bento do Tocantins
city_br_4656,base.br,base.state_br_rn,São Bento do Trairí
city_br_4657,base.br,base.state_br_pe,São Bento do Una
city_br_4658,base.br,base.state_br_sc,São Bernardino
city_br_4659,base.br,base.state_br_ma,São Bernardo
city_br_4660,base.br,base.state_br_sc,São Bonifácio
city_br_4661,base.br,base.state_br_rs,São Borja
city_br_4662,base.br,base.state_br_al,São Brás
city_br_4663,base.br,base.state_br_mg,São Brás do Suaçuí
city_br_4664,base.br,base.state_br_pi,São Braz do Piauí
city_br_4665,base.br,base.state_br_pa,São Caetano de Odivelas
city_br_4666,base.br,base.state_br_pe,São Caitano
city_br_4667,base.br,base.state_br_sc,São Carlos
city_br_4668,base.br,base.state_br_pr,São Carlos do Ivaí
city_br_4669,base.br,base.state_br_se,São Cristóvão
city_br_4670,base.br,base.state_br_sc,São Cristóvão do Sul
city_br_4671,base.br,base.state_br_ba,São Desidério
city_br_4672,base.br,base.state_br_ba,São Domingos
city_br_4673,base.br,base.state_br_go,São Domingos
city_br_4674,base.br,base.state_br_pb,São Domingos
city_br_4675,base.br,base.state_br_sc,São Domingos
city_br_4676,base.br,base.state_br_se,São Domingos
city_br_4677,base.br,base.state_br_mg,São Domingos das Dores
city_br_4678,base.br,base.state_br_pa,São Domingos do Araguaia
city_br_4679,base.br,base.state_br_ma,São Domingos do Azeitão
city_br_4680,base.br,base.state_br_pa,São Domingos do Capim
city_br_4681,base.br,base.state_br_pb,São Domingos do Cariri
city_br_4682,base.br,base.state_br_ma,São Domingos do Maranhão
city_br_4683,base.br,base.state_br_es,São Domingos do Norte
city_br_4684,base.br,base.state_br_mg,São Domingos do Prata
city_br_4685,base.br,base.state_br_rs,São Domingos do Sul
city_br_4686,base.br,base.state_br_ba,São Felipe
city_br_4687,base.br,base.state_br_ro,São Felipe D'Oeste
city_br_4688,base.br,base.state_br_ba,São Félix
city_br_4689,base.br,base.state_br_ma,São Félix de Balsas
city_br_4690,base.br,base.state_br_mg,São Félix de Minas
city_br_4691,base.br,base.state_br_mt,São Félix do Araguaia
city_br_4692,base.br,base.state_br_ba,São Félix do Coribe
city_br_4693,base.br,base.state_br_pi,São Félix do Piauí
city_br_4694,base.br,base.state_br_to,São Félix do Tocantins
city_br_4695,base.br,base.state_br_pa,São Félix do Xingu
city_br_4696,base.br,base.state_br_rn,São Fernando
city_br_4697,base.br,base.state_br_rj,São Fidélis
city_br_4698,base.br,base.state_br_mg,São Francisco
city_br_4699,base.br,base.state_br_pb,São Francisco
city_br_4700,base.br,base.state_br_se,São Francisco
city_br_4701,base.br,base.state_br_sp,São Francisco
city_br_4702,base.br,base.state_br_rs,São Francisco de Assis
city_br_4703,base.br,base.state_br_pi,São Francisco de Assis do Piauí
city_br_4704,base.br,base.state_br_go,São Francisco de Goiás
city_br_4705,base.br,base.state_br_rj,São Francisco de Itabapoana
city_br_4706,base.br,base.state_br_mg,São Francisco de Paula
city_br_4707,base.br,base.state_br_rs,São Francisco de Paula
city_br_4708,base.br,base.state_br_mg,São Francisco de Sales
city_br_4709,base.br,base.state_br_ma,São Francisco do Brejão
city_br_4710,base.br,base.state_br_ba,São Francisco do Conde
city_br_4711,base.br,base.state_br_mg,São Francisco do Glória
city_br_4712,base.br,base.state_br_ro,São Francisco do Guaporé
city_br_4713,base.br,base.state_br_ma,São Francisco do Maranhão
city_br_4714,base.br,base.state_br_rn,São Francisco do Oeste
city_br_4715,base.br,base.state_br_pa,São Francisco do Pará
city_br_4716,base.br,base.state_br_pi,São Francisco do Piauí
city_br_4717,base.br,base.state_br_sc,São Francisco do Sul
city_br_4718,base.br,base.state_br_ba,São Gabriel
city_br_4719,base.br,base.state_br_rs,São Gabriel
city_br_4720,base.br,base.state_br_am,São Gabriel da Cachoeira
city_br_4721,base.br,base.state_br_es,São Gabriel da Palha
city_br_4722,base.br,base.state_br_ms,São Gabriel do Oeste
city_br_4723,base.br,base.state_br_mg,São Geraldo
city_br_4724,base.br,base.state_br_mg,São Geraldo da Piedade
city_br_4725,base.br,base.state_br_pa,São Geraldo do Araguaia
city_br_4726,base.br,base.state_br_mg,São Geraldo do Baixio
city_br_4727,base.br,base.state_br_mg,São Gonçalo do Abaeté
city_br_4728,base.br,base.state_br_ce,São Gonçalo do Amarante
city_br_4729,base.br,base.state_br_pi,São Gonçalo do Gurguéia
city_br_4730,base.br,base.state_br_mg,São Gonçalo do Pará
city_br_4731,base.br,base.state_br_pi,São Gonçalo do Piauí
city_br_4732,base.br,base.state_br_mg,São Gonçalo do Rio Abaixo
city_br_4733,base.br,base.state_br_mg,São Gonçalo do Rio Preto
city_br_4734,base.br,base.state_br_mg,São Gonçalo do Sapucaí
city_br_4735,base.br,base.state_br_ba,São Gonçalo dos Campos
city_br_4736,base.br,base.state_br_mg,São Gotardo
city_br_4737,base.br,base.state_br_rs,São Jerônimo
city_br_4738,base.br,base.state_br_pr,São Jerônimo da Serra
city_br_4739,base.br,base.state_br_pe,São João
city_br_4740,base.br,base.state_br_pr,São João
city_br_4741,base.br,base.state_br_ma,São João Batista
city_br_4742,base.br,base.state_br_sc,São João Batista
city_br_4743,base.br,base.state_br_mg,São João Batista do Glória
city_br_4744,base.br,base.state_br_go,São João d'Aliança
city_br_4745,base.br,base.state_br_rr,São João da Baliza
city_br_4746,base.br,base.state_br_rj,São João da Barra
city_br_4747,base.br,base.state_br_sp,São João da Boa Vista
city_br_4748,base.br,base.state_br_pi,São João da Canabrava
city_br_4749,base.br,base.state_br_pi,São João da Fronteira
city_br_4750,base.br,base.state_br_mg,São João da Lagoa
city_br_4751,base.br,base.state_br_mg,São João da Mata
city_br_4752,base.br,base.state_br_go,São João da Paraúna
city_br_4753,base.br,base.state_br_pa,São João da Ponta
city_br_4754,base.br,base.state_br_mg,São João da Ponte
city_br_4755,base.br,base.state_br_pi,São João da Serra
city_br_4756,base.br,base.state_br_rs,São João da Urtiga
city_br_4757,base.br,base.state_br_pi,São João da Varjota
city_br_4758,base.br,base.state_br_sp,São João das Duas Pontes
city_br_4759,base.br,base.state_br_mg,São João das Missões
city_br_4760,base.br,base.state_br_sp,São João de Iracema
city_br_4761,base.br,base.state_br_pa,São João de Pirabas
city_br_4762,base.br,base.state_br_mg,São João del Rei
city_br_4763,base.br,base.state_br_pa,São João do Araguaia
city_br_4764,base.br,base.state_br_pi,São João do Arraial
city_br_4765,base.br,base.state_br_pr,São João do Caiuá
city_br_4766,base.br,base.state_br_pb,São João do Cariri
city_br_4767,base.br,base.state_br_ma,São João do Carú
city_br_4768,base.br,base.state_br_sc,São João do Itaperiú
city_br_4769,base.br,base.state_br_pr,São João do Ivaí
city_br_4770,base.br,base.state_br_ce,São João do Jaguaribe
city_br_4771,base.br,base.state_br_mg,São João do Manhuaçu
city_br_4772,base.br,base.state_br_mg,São João do Manteninha
city_br_4773,base.br,base.state_br_sc,São João do Oeste
city_br_4774,base.br,base.state_br_mg,São João do Oriente
city_br_4775,base.br,base.state_br_mg,São João do Pacuí
city_br_4776,base.br,base.state_br_ma,São João do Paraíso
city_br_4777,base.br,base.state_br_mg,São João do Paraíso
city_br_4778,base.br,base.state_br_sp,São João do Pau d'Alho
city_br_4779,base.br,base.state_br_pi,São João do Piauí
city_br_4780,base.br,base.state_br_rs,São João do Polêsine
city_br_4781,base.br,base.state_br_pb,São João do Rio do Peixe
city_br_4782,base.br,base.state_br_rn,São João do Sabugi
city_br_4783,base.br,base.state_br_ma,São João do Soter
city_br_4784,base.br,base.state_br_sc,São João do Sul
city_br_4785,base.br,base.state_br_pb,São João do Tigre
city_br_4786,base.br,base.state_br_pr,São João do Triunfo
city_br_4787,base.br,base.state_br_ma,São João dos Patos
city_br_4788,base.br,base.state_br_mg,São João Evangelista
city_br_4789,base.br,base.state_br_mg,São João Nepomuceno
city_br_4790,base.br,base.state_br_sc,São Joaquim
city_br_4791,base.br,base.state_br_sp,São Joaquim da Barra
city_br_4792,base.br,base.state_br_mg,São Joaquim de Bicas
city_br_4793,base.br,base.state_br_pe,São Joaquim do Monte
city_br_4794,base.br,base.state_br_rs,São Jorge
city_br_4795,base.br,base.state_br_pr,São Jorge d'Oeste
city_br_4796,base.br,base.state_br_pr,São Jorge do Ivaí
city_br_4797,base.br,base.state_br_pr,São Jorge do Patrocínio
city_br_4798,base.br,base.state_br_mg,São José da Barra
city_br_4799,base.br,base.state_br_sp,São José da Bela Vista
city_br_4800,base.br,base.state_br_pr,São José da Boa Vista
city_br_4801,base.br,base.state_br_pe,São José da Coroa Grande
city_br_4802,base.br,base.state_br_pb,São José da Lagoa Tapada
city_br_4803,base.br,base.state_br_al,São José da Laje
city_br_4804,base.br,base.state_br_mg,São José da Lapa
city_br_4805,base.br,base.state_br_mg,São José da Safira
city_br_4806,base.br,base.state_br_al,São José da Tapera
city_br_4807,base.br,base.state_br_mg,São José da Varginha
city_br_4808,base.br,base.state_br_ba,São José da Vitória
city_br_4809,base.br,base.state_br_rs,São José das Missões
city_br_4810,base.br,base.state_br_pr,São José das Palmeiras
city_br_4811,base.br,base.state_br_pb,São José de Caiana
city_br_4812,base.br,base.state_br_pb,São José de Espinharas
city_br_4813,base.br,base.state_br_rn,São José de Mipibu
city_br_4814,base.br,base.state_br_pb,São José de Piranhas
city_br_4815,base.br,base.state_br_pb,São José de Princesa
city_br_4816,base.br,base.state_br_rj,São José de Ubá
city_br_4817,base.br,base.state_br_mg,São José do Alegre
city_br_4818,base.br,base.state_br_sp,São José do Barreiro
city_br_4819,base.br,base.state_br_pe,São José do Belmonte
city_br_4820,base.br,base.state_br_pb,São José do Bonfim
city_br_4821,base.br,base.state_br_pb,São José do Brejo do Cruz
city_br_4822,base.br,base.state_br_es,São José do Calçado
city_br_4823,base.br,base.state_br_rn,São José do Campestre
city_br_4824,base.br,base.state_br_sc,São José do Cedro
city_br_4825,base.br,base.state_br_sc,São José do Cerrito
city_br_4826,base.br,base.state_br_mg,São José do Divino
city_br_4827,base.br,base.state_br_pi,São José do Divino
city_br_4828,base.br,base.state_br_pe,São José do Egito
city_br_4829,base.br,base.state_br_mg,São José do Goiabal
city_br_4830,base.br,base.state_br_rs,São José do Herval
city_br_4831,base.br,base.state_br_rs,São José do Hortêncio
city_br_4832,base.br,base.state_br_rs,São José do Inhacorá
city_br_4833,base.br,base.state_br_ba,São José do Jacuípe
city_br_4834,base.br,base.state_br_mg,São José do Jacuri
city_br_4835,base.br,base.state_br_mg,São José do Mantimento
city_br_4836,base.br,base.state_br_rs,São José do Norte
city_br_4837,base.br,base.state_br_rs,São José do Ouro
city_br_4838,base.br,base.state_br_pi,São José do Peixe
city_br_4839,base.br,base.state_br_pi,São José do Piauí
city_br_4840,base.br,base.state_br_mt,São José do Povo
city_br_4841,base.br,base.state_br_mt,São José do Rio Claro
city_br_4842,base.br,base.state_br_sp,São José do Rio Pardo
city_br_4843,base.br,base.state_br_pb,São José do Sabugi
city_br_4844,base.br,base.state_br_rn,São José do Seridó
city_br_4845,base.br,base.state_br_rs,São José do Sul
city_br_4846,base.br,base.state_br_rj,São José do Vale do Rio Preto
city_br_4847,base.br,base.state_br_mt,São José do Xingu
city_br_4848,base.br,base.state_br_rs,São José dos Ausentes
city_br_4849,base.br,base.state_br_ma,São José dos Basílios
city_br_4850,base.br,base.state_br_pb,São José dos Cordeiros
city_br_4851,base.br,base.state_br_mt,São José dos Quatro Marcos
city_br_4852,base.br,base.state_br_pb,São José dos Ramos
city_br_4853,base.br,base.state_br_pi,São Julião
city_br_4854,base.br,base.state_br_mg,São Lourenço
city_br_4855,base.br,base.state_br_sp,São Lourenço da Serra
city_br_4856,base.br,base.state_br_sc,São Lourenço do Oeste
city_br_4857,base.br,base.state_br_pi,São Lourenço do Piauí
city_br_4858,base.br,base.state_br_rs,São Lourenço do Sul
city_br_4859,base.br,base.state_br_sc,São Ludgero
city_br_4860,base.br,base.state_br_go,São Luís de Montes Belos
city_br_4861,base.br,base.state_br_ce,São Luís do Curu
city_br_4862,base.br,base.state_br_pi,São Luis do Piauí
city_br_4863,base.br,base.state_br_al,São Luís do Quitunde
city_br_4864,base.br,base.state_br_ma,São Luís Gonzaga do Maranhão
city_br_4865,base.br,base.state_br_rr,São Luiz
city_br_4866,base.br,base.state_br_go,São Luiz do Norte
city_br_4867,base.br,base.state_br_sp,São Luiz do Paraitinga
city_br_4868,base.br,base.state_br_rs,São Luiz Gonzaga
city_br_4869,base.br,base.state_br_pb,São Mamede
city_br_4870,base.br,base.state_br_pr,São Manoel do Paraná
city_br_4871,base.br,base.state_br_sp,São Manuel
city_br_4872,base.br,base.state_br_rs,São Marcos
city_br_4873,base.br,base.state_br_rs,São Martinho
city_br_4874,base.br,base.state_br_sc,São Martinho
city_br_4875,base.br,base.state_br_rs,São Martinho da Serra
city_br_4876,base.br,base.state_br_ma,São Mateus do Maranhão
city_br_4877,base.br,base.state_br_pr,São Mateus do Sul
city_br_4878,base.br,base.state_br_rn,São Miguel
city_br_4879,base.br,base.state_br_sp,São Miguel Arcanjo
city_br_4880,base.br,base.state_br_pi,São Miguel da Baixa Grande
city_br_4881,base.br,base.state_br_sc,São Miguel da Boa Vista
city_br_4882,base.br,base.state_br_ba,São Miguel das Matas
city_br_4883,base.br,base.state_br_rs,São Miguel das Missões
city_br_4884,base.br,base.state_br_pb,São Miguel de Taipu
city_br_4885,base.br,base.state_br_se,São Miguel do Aleixo
city_br_4886,base.br,base.state_br_mg,São Miguel do Anta
city_br_4887,base.br,base.state_br_go,São Miguel do Araguaia
city_br_4888,base.br,base.state_br_pi,São Miguel do Fidalgo
city_br_4889,base.br,base.state_br_rn,São Miguel do Gostoso
city_br_4890,base.br,base.state_br_pa,São Miguel do Guamá
city_br_4891,base.br,base.state_br_ro,São Miguel do Guaporé
city_br_4892,base.br,base.state_br_pr,São Miguel do Iguaçu
city_br_4893,base.br,base.state_br_sc,São Miguel do Oeste
city_br_4894,base.br,base.state_br_go,São Miguel do Passa Quatro
city_br_4895,base.br,base.state_br_pi,São Miguel do Tapuio
city_br_4896,base.br,base.state_br_to,São Miguel do Tocantins
city_br_4897,base.br,base.state_br_al,São Miguel dos Campos
city_br_4898,base.br,base.state_br_al,São Miguel dos Milagres
city_br_4899,base.br,base.state_br_rs,São Nicolau
city_br_4900,base.br,base.state_br_go,São Patrício
city_br_4901,base.br,base.state_br_rs,São Paulo das Missões
city_br_4902,base.br,base.state_br_am,São Paulo de Olivença
city_br_4903,base.br,base.state_br_rn,São Paulo do Potengi
city_br_4904,base.br,base.state_br_rn,São Pedro
city_br_4905,base.br,base.state_br_sp,São Pedro
city_br_4906,base.br,base.state_br_ma,São Pedro da Água Branca
city_br_4907,base.br,base.state_br_mt,São Pedro da Cipa
city_br_4908,base.br,base.state_br_rs,São Pedro da Serra
city_br_4909,base.br,base.state_br_mg,São Pedro da União
city_br_4910,base.br,base.state_br_rs,São Pedro das Missões
city_br_4911,base.br,base.state_br_sc,São Pedro de Alcântara
city_br_4912,base.br,base.state_br_rs,São Pedro do Butiá
city_br_4913,base.br,base.state_br_pr,São Pedro do Iguaçu
city_br_4914,base.br,base.state_br_pr,São Pedro do Ivaí
city_br_4915,base.br,base.state_br_pr,São Pedro do Paraná
city_br_4916,base.br,base.state_br_pi,São Pedro do Piauí
city_br_4917,base.br,base.state_br_mg,São Pedro do Suaçuí
city_br_4918,base.br,base.state_br_rs,São Pedro do Sul
city_br_4919,base.br,base.state_br_sp,São Pedro do Turvo
city_br_4920,base.br,base.state_br_ma,São Pedro dos Crentes
city_br_4921,base.br,base.state_br_mg,São Pedro dos Ferros
city_br_4922,base.br,base.state_br_rn,São Rafael
city_br_4923,base.br,base.state_br_ma,São Raimundo das Mangabeiras
city_br_4924,base.br,base.state_br_ma,São Raimundo do Doca Bezerra
city_br_4925,base.br,base.state_br_pi,São Raimundo Nonato
city_br_4926,base.br,base.state_br_ma,São Roberto
city_br_4927,base.br,base.state_br_mg,São Romão
city_br_4928,base.br,base.state_br_sp,São Roque
city_br_4929,base.br,base.state_br_mg,São Roque de Minas
city_br_4930,base.br,base.state_br_es,São Roque do Canaã
city_br_4931,base.br,base.state_br_to,São Salvador do Tocantins
city_br_4932,base.br,base.state_br_al,São Sebastião
city_br_4933,base.br,base.state_br_sp,São Sebastião
city_br_4934,base.br,base.state_br_pr,São Sebastião da Amoreira
city_br_4935,base.br,base.state_br_mg,São Sebastião da Bela Vista
city_br_4936,base.br,base.state_br_pa,São Sebastião da Boa Vista
city_br_4937,base.br,base.state_br_sp,São Sebastião da Grama
city_br_4938,base.br,base.state_br_mg,São Sebastião da Vargem Alegre
city_br_4939,base.br,base.state_br_pb,São Sebastião de Lagoa de Roça
city_br_4940,base.br,base.state_br_rj,São Sebastião do Alto
city_br_4941,base.br,base.state_br_mg,São Sebastião do Anta
city_br_4942,base.br,base.state_br_rs,São Sebastião do Caí
city_br_4943,base.br,base.state_br_mg,São Sebastião do Maranhão
city_br_4944,base.br,base.state_br_mg,São Sebastião do Oeste
city_br_4945,base.br,base.state_br_mg,São Sebastião do Paraíso
city_br_4946,base.br,base.state_br_ba,São Sebastião do Passé
city_br_4947,base.br,base.state_br_mg,São Sebastião do Rio Preto
city_br_4948,base.br,base.state_br_mg,São Sebastião do Rio Verde
city_br_4949,base.br,base.state_br_to,São Sebastião do Tocantins
city_br_4950,base.br,base.state_br_am,São Sebastião do Uatumã
city_br_4951,base.br,base.state_br_pb,São Sebastião do Umbuzeiro
city_br_4952,base.br,base.state_br_rs,São Sepé
city_br_4953,base.br,base.state_br_go,São Simão
city_br_4954,base.br,base.state_br_sp,São Simão
city_br_4955,base.br,base.state_br_mg,São Thomé das Letras
city_br_4956,base.br,base.state_br_mg,São Tiago
city_br_4957,base.br,base.state_br_mg,São Tomás de Aquino
city_br_4958,base.br,base.state_br_pr,São Tomé
city_br_4959,base.br,base.state_br_rn,São Tomé
city_br_4960,base.br,base.state_br_rs,São Valentim
city_br_4961,base.br,base.state_br_rs,São Valentim do Sul
city_br_4962,base.br,base.state_br_to,São Valério
city_br_4963,base.br,base.state_br_rs,São Valério do Sul
city_br_4964,base.br,base.state_br_rs,São Vendelino
city_br_4965,base.br,base.state_br_rn,São Vicente
city_br_4966,base.br,base.state_br_mg,São Vicente de Minas
city_br_4967,base.br,base.state_br_pb,São Vicente do Seridó
city_br_4968,base.br,base.state_br_rs,São Vicente do Sul
city_br_4969,base.br,base.state_br_ma,São Vicente Ferrer
city_br_4970,base.br,base.state_br_pe,São Vicente Férrer
city_br_4971,base.br,base.state_br_pb,Sapé
city_br_4972,base.br,base.state_br_ba,Sapeaçu
city_br_4973,base.br,base.state_br_mt,Sapezal
city_br_4974,base.br,base.state_br_rs,Sapiranga
city_br_4975,base.br,base.state_br_pr,Sapopema
city_br_4976,base.br,base.state_br_mg,Sapucaí-Mirim
city_br_4977,base.br,base.state_br_pa,Sapucaia
city_br_4978,base.br,base.state_br_rj,Sapucaia
city_br_4979,base.br,base.state_br_rj,Saquarema
city_br_4980,base.br,base.state_br_rs,Sarandi
city_br_4981,base.br,base.state_br_sp,Sarapuí
city_br_4982,base.br,base.state_br_mg,Sardoá
city_br_4983,base.br,base.state_br_sp,Sarutaiá
city_br_4984,base.br,base.state_br_mg,Sarzedo
city_br_4985,base.br,base.state_br_ba,Sátiro Dias
city_br_4986,base.br,base.state_br_al,Satuba
city_br_4987,base.br,base.state_br_ma,Satubinha
city_br_4988,base.br,base.state_br_ba,Saubara
city_br_4989,base.br,base.state_br_pr,Saudade do Iguaçu
city_br_4990,base.br,base.state_br_sc,Saudades
city_br_4991,base.br,base.state_br_ba,Saúde
city_br_4992,base.br,base.state_br_sc,Schroeder
city_br_4993,base.br,base.state_br_ba,Seabra
city_br_4994,base.br,base.state_br_sc,Seara
city_br_4995,base.br,base.state_br_sp,Sebastianópolis do Sul
city_br_4996,base.br,base.state_br_pi,Sebastião Barros
city_br_4997,base.br,base.state_br_ba,Sebastião Laranjeiras
city_br_4998,base.br,base.state_br_pi,Sebastião Leal
city_br_4999,base.br,base.state_br_rs,Seberi
city_br_5000,base.br,base.state_br_rs,Sede Nova
city_br_5001,base.br,base.state_br_rs,Segredo
city_br_5002,base.br,base.state_br_rs,Selbach
city_br_5003,base.br,base.state_br_ms,Selvíria
city_br_5004,base.br,base.state_br_mg,Sem-Peixe
city_br_5005,base.br,base.state_br_ac,Sena Madureira
city_br_5006,base.br,base.state_br_ma,Senador Alexandre Costa
city_br_5007,base.br,base.state_br_mg,Senador Amaral
city_br_5008,base.br,base.state_br_mg,Senador Cortes
city_br_5009,base.br,base.state_br_rn,Senador Elói de Souza
city_br_5010,base.br,base.state_br_mg,Senador Firmino
city_br_5011,base.br,base.state_br_rn,Senador Georgino Avelino
city_br_5012,base.br,base.state_br_ac,Senador Guiomard
city_br_5013,base.br,base.state_br_mg,Senador José Bento
city_br_5014,base.br,base.state_br_pa,Senador José Porfírio
city_br_5015,base.br,base.state_br_ma,Senador La Rocque
city_br_5016,base.br,base.state_br_mg,Senador Modestino Gonçalves
city_br_5017,base.br,base.state_br_ce,Senador Pompeu
city_br_5018,base.br,base.state_br_al,Senador Rui Palmeira
city_br_5019,base.br,base.state_br_ce,Senador Sá
city_br_5020,base.br,base.state_br_rs,Senador Salgado Filho
city_br_5021,base.br,base.state_br_pr,Sengés
city_br_5022,base.br,base.state_br_ba,Senhor do Bonfim
city_br_5023,base.br,base.state_br_mg,Senhora de Oliveira
city_br_5024,base.br,base.state_br_mg,Senhora do Porto
city_br_5025,base.br,base.state_br_mg,Senhora dos Remédios
city_br_5026,base.br,base.state_br_rs,Sentinela do Sul
city_br_5027,base.br,base.state_br_ba,Sento Sé
city_br_5028,base.br,base.state_br_rs,Serafina Corrêa
city_br_5029,base.br,base.state_br_mg,Sericita
city_br_5030,base.br,base.state_br_ro,Seringueiras
city_br_5031,base.br,base.state_br_rs,Sério
city_br_5032,base.br,base.state_br_mg,Seritinga
city_br_5033,base.br,base.state_br_rj,Seropédica
city_br_5034,base.br,base.state_br_sc,Serra Alta
city_br_5035,base.br,base.state_br_sp,Serra Azul
city_br_5036,base.br,base.state_br_mg,Serra Azul de Minas
city_br_5037,base.br,base.state_br_pb,Serra Branca
city_br_5038,base.br,base.state_br_rn,Serra Caiada
city_br_5039,base.br,base.state_br_pb,Serra da Raiz
city_br_5040,base.br,base.state_br_mg,Serra da Saudade
city_br_5041,base.br,base.state_br_rn,Serra de São Bento
city_br_5042,base.br,base.state_br_rn,Serra do Mel
city_br_5043,base.br,base.state_br_ap,Serra do Navio
city_br_5044,base.br,base.state_br_ba,Serra do Ramalho
city_br_5045,base.br,base.state_br_mg,Serra do Salitre
city_br_5046,base.br,base.state_br_mg,Serra dos Aimorés
city_br_5047,base.br,base.state_br_ba,Serra Dourada
city_br_5048,base.br,base.state_br_pb,Serra Grande
city_br_5049,base.br,base.state_br_sp,Serra Negra
city_br_5050,base.br,base.state_br_rn,Serra Negra do Norte
city_br_5051,base.br,base.state_br_mt,Serra Nova Dourada
city_br_5052,base.br,base.state_br_ba,Serra Preta
city_br_5053,base.br,base.state_br_pb,Serra Redonda
city_br_5054,base.br,base.state_br_pe,Serra Talhada
city_br_5055,base.br,base.state_br_sp,Serrana
city_br_5056,base.br,base.state_br_mg,Serrania
city_br_5057,base.br,base.state_br_ma,Serrano do Maranhão
city_br_5058,base.br,base.state_br_go,Serranópolis
city_br_5059,base.br,base.state_br_mg,Serranópolis de Minas
city_br_5060,base.br,base.state_br_pr,Serranópolis do Iguaçu
city_br_5061,base.br,base.state_br_mg,Serranos
city_br_5062,base.br,base.state_br_pb,Serraria
city_br_5063,base.br,base.state_br_ba,Serrinha
city_br_5064,base.br,base.state_br_rn,Serrinha
city_br_5065,base.br,base.state_br_rn,Serrinha dos Pintos
city_br_5066,base.br,base.state_br_pe,Serrita
city_br_5067,base.br,base.state_br_mg,Serro
city_br_5068,base.br,base.state_br_ba,Serrolândia
city_br_5069,base.br,base.state_br_pr,Sertaneja
city_br_5070,base.br,base.state_br_pe,Sertânia
city_br_5071,base.br,base.state_br_pr,Sertanópolis
city_br_5072,base.br,base.state_br_rs,Sertão
city_br_5073,base.br,base.state_br_rs,Sertão Santana
city_br_5074,base.br,base.state_br_pb,Sertãozinho
city_br_5075,base.br,base.state_br_sp,Sete Barras
city_br_5076,base.br,base.state_br_rs,Sete de Setembro
city_br_5077,base.br,base.state_br_ms,Sete Quedas
city_br_5078,base.br,base.state_br_mg,Setubinha
city_br_5079,base.br,base.state_br_rs,Severiano de Almeida
city_br_5080,base.br,base.state_br_rn,Severiano Melo
city_br_5081,base.br,base.state_br_sp,Severínia
city_br_5082,base.br,base.state_br_sc,Siderópolis
city_br_5083,base.br,base.state_br_ms,Sidrolândia
city_br_5084,base.br,base.state_br_pi,Sigefredo Pacheco
city_br_5085,base.br,base.state_br_rj,Silva Jardim
city_br_5086,base.br,base.state_br_go,Silvânia
city_br_5087,base.br,base.state_br_to,Silvanópolis
city_br_5088,base.br,base.state_br_rs,Silveira Martins
city_br_5089,base.br,base.state_br_mg,Silveirânia
city_br_5090,base.br,base.state_br_sp,Silveiras
city_br_5091,base.br,base.state_br_am,Silves
city_br_5092,base.br,base.state_br_mg,Silvianópolis
city_br_5093,base.br,base.state_br_se,Simão Dias
city_br_5094,base.br,base.state_br_mg,Simão Pereira
city_br_5095,base.br,base.state_br_pi,Simões
city_br_5096,base.br,base.state_br_go,Simolândia
city_br_5097,base.br,base.state_br_mg,Simonésia
city_br_5098,base.br,base.state_br_pi,Simplício Mendes
city_br_5099,base.br,base.state_br_rs,Sinimbu
city_br_5100,base.br,base.state_br_pr,Siqueira Campos
city_br_5101,base.br,base.state_br_pe,Sirinhaém
city_br_5102,base.br,base.state_br_se,Siriri
city_br_5103,base.br,base.state_br_go,Sítio d'Abadia
city_br_5104,base.br,base.state_br_ba,Sítio do Mato
city_br_5105,base.br,base.state_br_ba,Sítio do Quinto
city_br_5106,base.br,base.state_br_ma,Sítio Novo
city_br_5107,base.br,base.state_br_rn,Sítio Novo
city_br_5108,base.br,base.state_br_to,Sítio Novo do Tocantins
city_br_5109,base.br,base.state_br_ba,Sobradinho
city_br_5110,base.br,base.state_br_rs,Sobradinho
city_br_5111,base.br,base.state_br_pb,Sobrado
city_br_5112,base.br,base.state_br_mg,Sobrália
city_br_5113,base.br,base.state_br_sp,Socorro
city_br_5114,base.br,base.state_br_pi,Socorro do Piauí
city_br_5115,base.br,base.state_br_pb,Solânea
city_br_5116,base.br,base.state_br_pb,Soledade
city_br_5117,base.br,base.state_br_rs,Soledade
city_br_5118,base.br,base.state_br_mg,Soledade de Minas
city_br_5119,base.br,base.state_br_pe,Solidão
city_br_5120,base.br,base.state_br_ce,Solonópole
city_br_5121,base.br,base.state_br_sc,Sombrio
city_br_5122,base.br,base.state_br_ms,Sonora
city_br_5123,base.br,base.state_br_es,Sooretama
city_br_5124,base.br,base.state_br_pb,Sossêgo
city_br_5125,base.br,base.state_br_pa,Soure
city_br_5126,base.br,base.state_br_pb,Sousa
city_br_5127,base.br,base.state_br_ba,Souto Soares
city_br_5128,base.br,base.state_br_to,Sucupira
city_br_5129,base.br,base.state_br_ma,Sucupira do Norte
city_br_5130,base.br,base.state_br_ma,Sucupira do Riachão
city_br_5131,base.br,base.state_br_sp,Sud Mennucci
city_br_5132,base.br,base.state_br_sc,Sul Brasil
city_br_5133,base.br,base.state_br_pr,Sulina
city_br_5134,base.br,base.state_br_pb,Sumé
city_br_5135,base.br,base.state_br_rj,Sumidouro
city_br_5136,base.br,base.state_br_pe,Surubim
city_br_5137,base.br,base.state_br_pi,Sussuapara
city_br_5138,base.br,base.state_br_sp,Suzanápolis
city_br_5139,base.br,base.state_br_rs,Tabaí
city_br_5140,base.br,base.state_br_mt,Tabaporã
city_br_5141,base.br,base.state_br_sp,Tabapuã
city_br_5142,base.br,base.state_br_am,Tabatinga
city_br_5143,base.br,base.state_br_sp,Tabatinga
city_br_5144,base.br,base.state_br_pe,Tabira
city_br_5145,base.br,base.state_br_ba,Tabocas do Brejo Velho
city_br_5146,base.br,base.state_br_rn,Taboleiro Grande
city_br_5147,base.br,base.state_br_mg,Tabuleiro
city_br_5148,base.br,base.state_br_ce,Tabuleiro do Norte
city_br_5149,base.br,base.state_br_pe,Tacaimbó
city_br_5150,base.br,base.state_br_pe,Tacaratu
city_br_5151,base.br,base.state_br_sp,Taciba
city_br_5152,base.br,base.state_br_pb,Tacima
city_br_5153,base.br,base.state_br_ms,Tacuru
city_br_5154,base.br,base.state_br_sp,Taguaí
city_br_5155,base.br,base.state_br_to,Taguatinga
city_br_5156,base.br,base.state_br_sp,Taiaçu
city_br_5157,base.br,base.state_br_pa,Tailândia
city_br_5158,base.br,base.state_br_sc,Taió
city_br_5159,base.br,base.state_br_mg,Taiobeiras
city_br_5160,base.br,base.state_br_to,Taipas do Tocantins
city_br_5161,base.br,base.state_br_rn,Taipu
city_br_5162,base.br,base.state_br_sp,Taiúva
city_br_5163,base.br,base.state_br_to,Talismã
city_br_5164,base.br,base.state_br_pe,Tamandaré
city_br_5165,base.br,base.state_br_pr,Tamarana
city_br_5166,base.br,base.state_br_sp,Tambaú
city_br_5167,base.br,base.state_br_pr,Tamboara
city_br_5168,base.br,base.state_br_ce,Tamboril
city_br_5169,base.br,base.state_br_pi,Tamboril do Piauí
city_br_5170,base.br,base.state_br_sp,Tanabi
city_br_5171,base.br,base.state_br_rn,Tangará
city_br_5172,base.br,base.state_br_sc,Tangará
city_br_5173,base.br,base.state_br_rj,Tanguá
city_br_5174,base.br,base.state_br_ba,Tanhaçu
city_br_5175,base.br,base.state_br_al,Tanque d'Arca
city_br_5176,base.br,base.state_br_pi,Tanque do Piauí
city_br_5177,base.br,base.state_br_ba,Tanque Novo
city_br_5178,base.br,base.state_br_ba,Tanquinho
city_br_5179,base.br,base.state_br_mg,Taparuba
city_br_5180,base.br,base.state_br_am,Tapauá
city_br_5181,base.br,base.state_br_pr,Tapejara
city_br_5182,base.br,base.state_br_rs,Tapejara
city_br_5183,base.br,base.state_br_rs,Tapera
city_br_5184,base.br,base.state_br_ba,Taperoá
city_br_5185,base.br,base.state_br_pb,Taperoá
city_br_5186,base.br,base.state_br_rs,Tapes
city_br_5187,base.br,base.state_br_mg,Tapira
city_br_5188,base.br,base.state_br_pr,Tapira
city_br_5189,base.br,base.state_br_mg,Tapiraí
city_br_5190,base.br,base.state_br_sp,Tapiraí
city_br_5191,base.br,base.state_br_ba,Tapiramutá
city_br_5192,base.br,base.state_br_sp,Tapiratiba
city_br_5193,base.br,base.state_br_mt,Tapurah
city_br_5194,base.br,base.state_br_rs,Taquara
city_br_5195,base.br,base.state_br_mg,Taquaraçu de Minas
city_br_5196,base.br,base.state_br_sp,Taquaral
city_br_5197,base.br,base.state_br_go,Taquaral de Goiás
city_br_5198,base.br,base.state_br_al,Taquarana
city_br_5199,base.br,base.state_br_rs,Taquari
city_br_5200,base.br,base.state_br_sp,Taquaritinga
city_br_5201,base.br,base.state_br_pe,Taquaritinga do Norte
city_br_5202,base.br,base.state_br_sp,Taquarituba
city_br_5203,base.br,base.state_br_sp,Taquarivaí
city_br_5204,base.br,base.state_br_rs,Taquaruçu do Sul
city_br_5205,base.br,base.state_br_ms,Taquarussu
city_br_5206,base.br,base.state_br_sp,Tarabai
city_br_5207,base.br,base.state_br_ac,Tarauacá
city_br_5208,base.br,base.state_br_ce,Tarrafas
city_br_5209,base.br,base.state_br_ap,Tartarugalzinho
city_br_5210,base.br,base.state_br_sp,Tarumã
city_br_5211,base.br,base.state_br_mg,Tarumirim
city_br_5212,base.br,base.state_br_ma,Tasso Fragoso
city_br_5213,base.br,base.state_br_ce,Tauá
city_br_5214,base.br,base.state_br_pb,Tavares
city_br_5215,base.br,base.state_br_rs,Tavares
city_br_5216,base.br,base.state_br_am,Tefé
city_br_5217,base.br,base.state_br_pb,Teixeira
city_br_5218,base.br,base.state_br_pr,Teixeira Soares
city_br_5219,base.br,base.state_br_mg,Teixeiras
city_br_5220,base.br,base.state_br_ro,Teixeirópolis
city_br_5221,base.br,base.state_br_ce,Tejuçuoca
city_br_5222,base.br,base.state_br_sp,Tejupá
city_br_5223,base.br,base.state_br_pr,Telêmaco Borba
city_br_5224,base.br,base.state_br_se,Telha
city_br_5225,base.br,base.state_br_rn,Tenente Ananias
city_br_5226,base.br,base.state_br_rn,Tenente Laurentino Cruz
city_br_5227,base.br,base.state_br_rs,Tenente Portela
city_br_5228,base.br,base.state_br_pb,Tenório
city_br_5229,base.br,base.state_br_ba,Teodoro Sampaio
city_br_5230,base.br,base.state_br_sp,Teodoro Sampaio
city_br_5231,base.br,base.state_br_ba,Teofilândia
city_br_5232,base.br,base.state_br_ba,Teolândia
city_br_5233,base.br,base.state_br_al,Teotônio Vilela
city_br_5234,base.br,base.state_br_ms,Terenos
city_br_5235,base.br,base.state_br_go,Teresina de Goiás
city_br_5236,base.br,base.state_br_pe,Terezinha
city_br_5237,base.br,base.state_br_go,Terezópolis de Goiás
city_br_5238,base.br,base.state_br_pa,Terra Alta
city_br_5239,base.br,base.state_br_pr,Terra Boa
city_br_5240,base.br,base.state_br_rs,Terra de Areia
city_br_5241,base.br,base.state_br_ba,Terra Nova
city_br_5242,base.br,base.state_br_pe,Terra Nova
city_br_5243,base.br,base.state_br_mt,Terra Nova do Norte
city_br_5244,base.br,base.state_br_pr,Terra Rica
city_br_5245,base.br,base.state_br_pr,Terra Roxa
city_br_5246,base.br,base.state_br_sp,Terra Roxa
city_br_5247,base.br,base.state_br_pa,Terra Santa
city_br_5248,base.br,base.state_br_mt,Tesouro
city_br_5249,base.br,base.state_br_rs,Teutônia
city_br_5250,base.br,base.state_br_ro,Theobroma
city_br_5251,base.br,base.state_br_ce,Tianguá
city_br_5252,base.br,base.state_br_pr,Tibagi
city_br_5253,base.br,base.state_br_rn,Tibau
city_br_5254,base.br,base.state_br_rn,Tibau do Sul
city_br_5255,base.br,base.state_br_sp,Tietê
city_br_5256,base.br,base.state_br_sc,Tigrinhos
city_br_5257,base.br,base.state_br_sc,Tijucas
city_br_5258,base.br,base.state_br_pr,Tijucas do Sul
city_br_5259,base.br,base.state_br_pe,Timbaúba
city_br_5260,base.br,base.state_br_rn,Timbaúba dos Batistas
city_br_5261,base.br,base.state_br_sc,Timbé do Sul
city_br_5262,base.br,base.state_br_ma,Timbiras
city_br_5263,base.br,base.state_br_sc,Timbó
city_br_5264,base.br,base.state_br_sc,Timbó Grande
city_br_5265,base.br,base.state_br_sp,Timburi
city_br_5266,base.br,base.state_br_mg,Timóteo
city_br_5267,base.br,base.state_br_rs,Tio Hugo
city_br_5268,base.br,base.state_br_mg,Tiradentes
city_br_5269,base.br,base.state_br_rs,Tiradentes do Sul
city_br_5270,base.br,base.state_br_mg,Tiros
city_br_5271,base.br,base.state_br_se,Tobias Barreto
city_br_5272,base.br,base.state_br_to,Tocantínia
city_br_5273,base.br,base.state_br_to,Tocantinópolis
city_br_5274,base.br,base.state_br_mg,Tocantins
city_br_5275,base.br,base.state_br_mg,Tocos do Moji
city_br_5276,base.br,base.state_br_mg,Toledo
city_br_5277,base.br,base.state_br_se,Tomar do Geru
city_br_5278,base.br,base.state_br_pr,Tomazina
city_br_5279,base.br,base.state_br_mg,Tombos
city_br_5280,base.br,base.state_br_pa,Tomé-Açu
city_br_5281,base.br,base.state_br_am,Tonantins
city_br_5282,base.br,base.state_br_pe,Toritama
city_br_5283,base.br,base.state_br_mt,Torixoréu
city_br_5284,base.br,base.state_br_rs,Toropi
city_br_5285,base.br,base.state_br_sp,Torre de Pedra
city_br_5286,base.br,base.state_br_rs,Torres
city_br_5287,base.br,base.state_br_sp,Torrinha
city_br_5288,base.br,base.state_br_rn,Touros
city_br_5289,base.br,base.state_br_sp,Trabiju
city_br_5290,base.br,base.state_br_pa,Tracuateua
city_br_5291,base.br,base.state_br_pe,Tracunhaém
city_br_5292,base.br,base.state_br_al,Traipu
city_br_5293,base.br,base.state_br_pa,Trairão
city_br_5294,base.br,base.state_br_ce,Trairi
city_br_5295,base.br,base.state_br_rj,Trajano de Moraes
city_br_5296,base.br,base.state_br_rs,Tramandaí
city_br_5297,base.br,base.state_br_rs,Travesseiro
city_br_5298,base.br,base.state_br_ba,Tremedal
city_br_5299,base.br,base.state_br_sp,Tremembé
city_br_5300,base.br,base.state_br_rs,Três Arroios
city_br_5301,base.br,base.state_br_sc,Três Barras
city_br_5302,base.br,base.state_br_pr,Três Barras do Paraná
city_br_5303,base.br,base.state_br_rs,Três Cachoeiras
city_br_5304,base.br,base.state_br_mg,Três Corações
city_br_5305,base.br,base.state_br_rs,Três Coroas
city_br_5306,base.br,base.state_br_rs,Três de Maio
city_br_5307,base.br,base.state_br_rs,Três Forquilhas
city_br_5308,base.br,base.state_br_sp,Três Fronteiras
city_br_5309,base.br,base.state_br_mg,Três Marias
city_br_5310,base.br,base.state_br_rs,Três Palmeiras
city_br_5311,base.br,base.state_br_rs,Três Passos
city_br_5312,base.br,base.state_br_mg,Três Pontas
city_br_5313,base.br,base.state_br_go,Três Ranchos
city_br_5314,base.br,base.state_br_rj,Três Rios
city_br_5315,base.br,base.state_br_sc,Treviso
city_br_5316,base.br,base.state_br_sc,Treze de Maio
city_br_5317,base.br,base.state_br_sc,Treze Tílias
city_br_5318,base.br,base.state_br_pe,Trindade
city_br_5319,base.br,base.state_br_rs,Trindade do Sul
city_br_5320,base.br,base.state_br_pb,Triunfo
city_br_5321,base.br,base.state_br_pe,Triunfo
city_br_5322,base.br,base.state_br_rs,Triunfo
city_br_5323,base.br,base.state_br_rn,Triunfo Potiguar
city_br_5324,base.br,base.state_br_ma,Trizidela do Vale
city_br_5325,base.br,base.state_br_go,Trombas
city_br_5326,base.br,base.state_br_sc,Trombudo Central
city_br_5327,base.br,base.state_br_ba,Tucano
city_br_5328,base.br,base.state_br_pa,Tucumã
city_br_5329,base.br,base.state_br_rs,Tucunduva
city_br_5330,base.br,base.state_br_pa,Tucuruí
city_br_5331,base.br,base.state_br_ma,Tufilândia
city_br_5332,base.br,base.state_br_sp,Tuiuti
city_br_5333,base.br,base.state_br_mg,Tumiritinga
city_br_5334,base.br,base.state_br_sc,Tunápolis
city_br_5335,base.br,base.state_br_rs,Tunas
city_br_5336,base.br,base.state_br_pr,Tunas do Paraná
city_br_5337,base.br,base.state_br_pr,Tuneiras do Oeste
city_br_5338,base.br,base.state_br_ma,Tuntum
city_br_5339,base.br,base.state_br_sp,Tupã
city_br_5340,base.br,base.state_br_mg,Tupaciguara
city_br_5341,base.br,base.state_br_pe,Tupanatinga
city_br_5342,base.br,base.state_br_rs,Tupanci do Sul
city_br_5343,base.br,base.state_br_rs,Tupanciretã
city_br_5344,base.br,base.state_br_rs,Tupandi
city_br_5345,base.br,base.state_br_rs,Tuparendi
city_br_5346,base.br,base.state_br_pe,Tuparetama
city_br_5347,base.br,base.state_br_pr,Tupãssi
city_br_5348,base.br,base.state_br_sp,Tupi Paulista
city_br_5349,base.br,base.state_br_to,Tupirama
city_br_5350,base.br,base.state_br_to,Tupiratins
city_br_5351,base.br,base.state_br_ma,Turiaçu
city_br_5352,base.br,base.state_br_ma,Turilândia
city_br_5353,base.br,base.state_br_sp,Turiúba
city_br_5354,base.br,base.state_br_mg,Turmalina
city_br_5355,base.br,base.state_br_sp,Turmalina
city_br_5356,base.br,base.state_br_rs,Turuçu
city_br_5357,base.br,base.state_br_ce,Tururu
city_br_5358,base.br,base.state_br_go,Turvânia
city_br_5359,base.br,base.state_br_go,Turvelândia
city_br_5360,base.br,base.state_br_pr,Turvo
city_br_5361,base.br,base.state_br_sc,Turvo
city_br_5362,base.br,base.state_br_mg,Turvolândia
city_br_5363,base.br,base.state_br_ma,Tutóia
city_br_5364,base.br,base.state_br_am,Uarini
city_br_5365,base.br,base.state_br_ba,Uauá
city_br_5366,base.br,base.state_br_mg,Ubaí
city_br_5367,base.br,base.state_br_ba,Ubaíra
city_br_5368,base.br,base.state_br_ba,Ubaitaba
city_br_5369,base.br,base.state_br_ce,Ubajara
city_br_5370,base.br,base.state_br_mg,Ubaporanga
city_br_5371,base.br,base.state_br_sp,Ubarana
city_br_5372,base.br,base.state_br_ba,Ubatã
city_br_5373,base.br,base.state_br_sp,Ubatuba
city_br_5374,base.br,base.state_br_sp,Ubirajara
city_br_5375,base.br,base.state_br_pr,Ubiratã
city_br_5376,base.br,base.state_br_rs,Ubiretama
city_br_5377,base.br,base.state_br_sp,Uchoa
city_br_5378,base.br,base.state_br_ba,Uibaí
city_br_5379,base.br,base.state_br_rr,Uiramutã
city_br_5380,base.br,base.state_br_go,Uirapuru
city_br_5381,base.br,base.state_br_pb,Uiraúna
city_br_5382,base.br,base.state_br_pa,Ulianópolis
city_br_5383,base.br,base.state_br_ce,Umari
city_br_5384,base.br,base.state_br_rn,Umarizal
city_br_5385,base.br,base.state_br_se,Umbaúba
city_br_5386,base.br,base.state_br_ba,Umburanas
city_br_5387,base.br,base.state_br_mg,Umburatiba
city_br_5388,base.br,base.state_br_pb,Umbuzeiro
city_br_5389,base.br,base.state_br_ce,Umirim
city_br_5390,base.br,base.state_br_ba,Una
city_br_5391,base.br,base.state_br_mg,Unaí
city_br_5392,base.br,base.state_br_pi,União
city_br_5393,base.br,base.state_br_rs,União da Serra
city_br_5394,base.br,base.state_br_pr,União da Vitória
city_br_5395,base.br,base.state_br_mg,União de Minas
city_br_5396,base.br,base.state_br_sc,União do Oeste
city_br_5397,base.br,base.state_br_mt,União do Sul
city_br_5398,base.br,base.state_br_al,União dos Palmares
city_br_5399,base.br,base.state_br_sp,União Paulista
city_br_5400,base.br,base.state_br_pr,Uniflor
city_br_5401,base.br,base.state_br_rs,Unistalda
city_br_5402,base.br,base.state_br_rn,Upanema
city_br_5403,base.br,base.state_br_pr,Uraí
city_br_5404,base.br,base.state_br_ba,Urandi
city_br_5405,base.br,base.state_br_sp,Urânia
city_br_5406,base.br,base.state_br_ma,Urbano Santos
city_br_5407,base.br,base.state_br_sp,Uru
city_br_5408,base.br,base.state_br_go,Uruaçu
city_br_5409,base.br,base.state_br_go,Uruana
city_br_5410,base.br,base.state_br_mg,Uruana de Minas
city_br_5411,base.br,base.state_br_pa,Uruará
city_br_5412,base.br,base.state_br_sc,Urubici
city_br_5413,base.br,base.state_br_ce,Uruburetama
city_br_5414,base.br,base.state_br_mg,Urucânia
city_br_5415,base.br,base.state_br_am,Urucará
city_br_5416,base.br,base.state_br_ba,Uruçuca
city_br_5417,base.br,base.state_br_pi,Uruçuí
city_br_5418,base.br,base.state_br_mg,Urucuia
city_br_5419,base.br,base.state_br_am,Urucurituba
city_br_5420,base.br,base.state_br_ce,Uruoca
city_br_5421,base.br,base.state_br_ro,Urupá
city_br_5422,base.br,base.state_br_sc,Urupema
city_br_5423,base.br,base.state_br_sp,Urupês
city_br_5424,base.br,base.state_br_sc,Urussanga
city_br_5425,base.br,base.state_br_go,Urutaí
city_br_5426,base.br,base.state_br_ba,Utinga
city_br_5427,base.br,base.state_br_rs,Vacaria
city_br_5428,base.br,base.state_br_mt,Vale de São Domingos
city_br_5429,base.br,base.state_br_ro,Vale do Anari
city_br_5430,base.br,base.state_br_ro,Vale do Paraíso
city_br_5431,base.br,base.state_br_rs,Vale do Sol
city_br_5432,base.br,base.state_br_rs,Vale Real
city_br_5433,base.br,base.state_br_rs,Vale Verde
city_br_5434,base.br,base.state_br_ba,Valença
city_br_5435,base.br,base.state_br_rj,Valença
city_br_5436,base.br,base.state_br_pi,Valença do Piauí
city_br_5437,base.br,base.state_br_ba,Valente
city_br_5438,base.br,base.state_br_sp,Valentim Gentil
city_br_5439,base.br,base.state_br_sp,Valparaíso
city_br_5440,base.br,base.state_br_rs,Vanini
city_br_5441,base.br,base.state_br_sc,Vargeão
city_br_5442,base.br,base.state_br_sc,Vargem
city_br_5443,base.br,base.state_br_sp,Vargem
city_br_5444,base.br,base.state_br_mg,Vargem Alegre
city_br_5445,base.br,base.state_br_es,Vargem Alta
city_br_5446,base.br,base.state_br_mg,Vargem Bonita
city_br_5447,base.br,base.state_br_sc,Vargem Bonita
city_br_5448,base.br,base.state_br_ma,Vargem Grande
city_br_5449,base.br,base.state_br_mg,Vargem Grande do Rio Pardo
city_br_5450,base.br,base.state_br_sp,Vargem Grande do Sul
city_br_5451,base.br,base.state_br_sp,Vargem Grande Paulista
city_br_5452,base.br,base.state_br_go,Varjão
city_br_5453,base.br,base.state_br_mg,Varjão de Minas
city_br_5454,base.br,base.state_br_ce,Varjota
city_br_5455,base.br,base.state_br_rj,Varre-Sai
city_br_5456,base.br,base.state_br_pb,Várzea
city_br_5457,base.br,base.state_br_rn,Várzea
city_br_5458,base.br,base.state_br_ce,Várzea Alegre
city_br_5459,base.br,base.state_br_pi,Várzea Branca
city_br_5460,base.br,base.state_br_mg,Várzea da Palma
city_br_5461,base.br,base.state_br_ba,Várzea da Roça
city_br_5462,base.br,base.state_br_ba,Várzea do Poço
city_br_5463,base.br,base.state_br_pi,Várzea Grande
city_br_5464,base.br,base.state_br_ba,Várzea Nova
city_br_5465,base.br,base.state_br_ba,Varzedo
city_br_5466,base.br,base.state_br_mg,Varzelândia
city_br_5467,base.br,base.state_br_rj,Vassouras
city_br_5468,base.br,base.state_br_mg,Vazante
city_br_5469,base.br,base.state_br_rs,Venâncio Aires
city_br_5470,base.br,base.state_br_es,Venda Nova do Imigrante
city_br_5471,base.br,base.state_br_rn,Venha-Ver
city_br_5472,base.br,base.state_br_pr,Ventania
city_br_5473,base.br,base.state_br_pe,Venturosa
city_br_5474,base.br,base.state_br_mt,Vera
city_br_5475,base.br,base.state_br_ba,Vera Cruz
city_br_5476,base.br,base.state_br_rn,Vera Cruz
city_br_5477,base.br,base.state_br_rs,Vera Cruz
city_br_5478,base.br,base.state_br_sp,Vera Cruz
city_br_5479,base.br,base.state_br_pr,Vera Cruz do Oeste
city_br_5480,base.br,base.state_br_pi,Vera Mendes
city_br_5481,base.br,base.state_br_rs,Veranópolis
city_br_5482,base.br,base.state_br_pe,Verdejante
city_br_5483,base.br,base.state_br_mg,Verdelândia
city_br_5484,base.br,base.state_br_pr,Verê
city_br_5485,base.br,base.state_br_ba,Vereda
city_br_5486,base.br,base.state_br_mg,Veredinha
city_br_5487,base.br,base.state_br_mg,Veríssimo
city_br_5488,base.br,base.state_br_mg,Vermelho Novo
city_br_5489,base.br,base.state_br_pe,Vertente do Lério
city_br_5490,base.br,base.state_br_pe,Vertentes
city_br_5491,base.br,base.state_br_rs,Vespasiano Corrêa
city_br_5492,base.br,base.state_br_rs,Viadutos
city_br_5493,base.br,base.state_br_es,Viana
city_br_5494,base.br,base.state_br_ma,Viana
city_br_5495,base.br,base.state_br_go,Vianópolis
city_br_5496,base.br,base.state_br_pe,Vicência
city_br_5497,base.br,base.state_br_rs,Vicente Dutra
city_br_5498,base.br,base.state_br_ms,Vicentina
city_br_5499,base.br,base.state_br_go,Vicentinópolis
city_br_5500,base.br,base.state_br_al,Viçosa
city_br_5501,base.br,base.state_br_mg,Viçosa
city_br_5502,base.br,base.state_br_rn,Viçosa
city_br_5503,base.br,base.state_br_ce,Viçosa do Ceará
city_br_5504,base.br,base.state_br_rs,Victor Graeff
city_br_5505,base.br,base.state_br_sc,Vidal Ramos
city_br_5506,base.br,base.state_br_sc,Videira
city_br_5507,base.br,base.state_br_mg,Vieiras
city_br_5508,base.br,base.state_br_pb,Vieirópolis
city_br_5509,base.br,base.state_br_pa,Vigia
city_br_5510,base.br,base.state_br_mt,Vila Bela da Santíssima Trindade
city_br_5511,base.br,base.state_br_go,Vila Boa
city_br_5512,base.br,base.state_br_rn,Vila Flor
city_br_5513,base.br,base.state_br_rs,Vila Flores
city_br_5514,base.br,base.state_br_rs,Vila Lângaro
city_br_5515,base.br,base.state_br_rs,Vila Maria
city_br_5516,base.br,base.state_br_pi,Vila Nova do Piauí
city_br_5517,base.br,base.state_br_rs,Vila Nova do Sul
city_br_5518,base.br,base.state_br_ma,Vila Nova dos Martírios
city_br_5519,base.br,base.state_br_es,Vila Pavão
city_br_5520,base.br,base.state_br_go,Vila Propício
city_br_5521,base.br,base.state_br_mt,Vila Rica
city_br_5522,base.br,base.state_br_es,Vila Valério
city_br_5523,base.br,base.state_br_ro,Vilhena
city_br_5524,base.br,base.state_br_sp,Vinhedo
city_br_5525,base.br,base.state_br_sp,Viradouro
city_br_5526,base.br,base.state_br_mg,Virgem da Lapa
city_br_5527,base.br,base.state_br_mg,Virgínia
city_br_5528,base.br,base.state_br_mg,Virginópolis
city_br_5529,base.br,base.state_br_mg,Virgolândia
city_br_5530,base.br,base.state_br_pr,Virmond
city_br_5531,base.br,base.state_br_mg,Visconde do Rio Branco
city_br_5532,base.br,base.state_br_pa,Viseu
city_br_5533,base.br,base.state_br_rs,Vista Alegre
city_br_5534,base.br,base.state_br_sp,Vista Alegre do Alto
city_br_5535,base.br,base.state_br_rs,Vista Alegre do Prata
city_br_5536,base.br,base.state_br_rs,Vista Gaúcha
city_br_5537,base.br,base.state_br_pb,Vista Serrana
city_br_5538,base.br,base.state_br_sc,Vitor Meireles
city_br_5539,base.br,base.state_br_sp,Vitória Brasil
city_br_5540,base.br,base.state_br_rs,Vitória das Missões
city_br_5541,base.br,base.state_br_ap,Vitória do Jari
city_br_5542,base.br,base.state_br_ma,Vitória do Mearim
city_br_5543,base.br,base.state_br_pa,Vitória do Xingu
city_br_5544,base.br,base.state_br_pr,Vitorino
city_br_5545,base.br,base.state_br_ma,Vitorino Freire
city_br_5546,base.br,base.state_br_mg,Volta Grande
city_br_5547,base.br,base.state_br_sp,Votuporanga
city_br_5548,base.br,base.state_br_ba,Wagner
city_br_5549,base.br,base.state_br_pi,Wall Ferraz
city_br_5550,base.br,base.state_br_to,Wanderlândia
city_br_5551,base.br,base.state_br_ba,Wanderley
city_br_5552,base.br,base.state_br_mg,Wenceslau Braz
city_br_5553,base.br,base.state_br_pr,Wenceslau Braz
city_br_5554,base.br,base.state_br_ba,Wenceslau Guimarães
city_br_5555,base.br,base.state_br_rs,Westfália
city_br_5556,base.br,base.state_br_sc,Witmarsum
city_br_5557,base.br,base.state_br_to,Xambioá
city_br_5558,base.br,base.state_br_pr,Xambrê
city_br_5559,base.br,base.state_br_rs,Xangri-lá
city_br_5560,base.br,base.state_br_sc,Xanxerê
city_br_5561,base.br,base.state_br_ac,Xapuri
city_br_5562,base.br,base.state_br_sc,Xavantina
city_br_5563,base.br,base.state_br_sc,Xaxim
city_br_5564,base.br,base.state_br_pe,Xexéu
city_br_5565,base.br,base.state_br_pa,Xinguara
city_br_5566,base.br,base.state_br_ba,Xique-Xique
city_br_5567,base.br,base.state_br_pb,Zabelê
city_br_5568,base.br,base.state_br_sp,Zacarias
city_br_5569,base.br,base.state_br_ma,Zé Doca
city_br_5570,base.br,base.state_br_sc,Zortéa
```

## File: data\res_country_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="base.br" model="res.country">
            <field name="address_view_id" ref="br_partner_address_form" />
        </record>
    </data>
</odoo>

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
"id","active","description","invoice_label","name","amount","type_tax_use","price_include_override","tax_discount","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@pt"
"tax_template_out_icms_interno17","True","ICMS Internal Sale 17%","ICMS Internal 17%","17% ICMS I","17.0","sale","tax_excluded","1","tax_group_icms_17","base","invoice","+ICMS_1","","ICMS Saída Interno 17%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_out_icms_externo17","True","ICMS External Sale 17%","ICMS External 17%","17% ICMS E","17.0","sale","tax_excluded","1","tax_group_icms_17","base","invoice","+ICMS_1","","ICMS Saída Externo 17%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_in_icms_interno17","True","ICMS Internal Purchase 17%","ICMS Internal 17%","17% ICMS I","17.0","purchase","tax_excluded","1","tax_group_icms_17","base","invoice","-ICMS_1","","ICMS Entrada Interno 17%"
"","","","","","","","","","","tax","invoice","-ICMS_2","account_template_101020302",""
"","","","","","","","","","","base","refund","+ICMS_1","",""
"","","","","","","","","","","tax","refund","+ICMS_2","account_template_201010903",""
"tax_template_in_icms_externo17","True","ICMS External Purchase 17%","ICMS External 17%","17% ICMS E","17.0","purchase","tax_excluded","1","tax_group_icms_17","base","invoice","-ICMS_1","","ICMS Entrada Externo 17%"
"","","","","","","","","","","tax","invoice","-ICMS_2","account_template_101020302",""
"","","","","","","","","","","base","refund","+ICMS_1","",""
"","","","","","","","","","","tax","refund","+ICMS_2","account_template_201010903",""
"tax_template_out_icms_interno","True","ICMS Internal Sale 0%","ICMS Internal market","0% ICMS I","0.0","sale","tax_excluded","1","tax_group_icms_0","base","invoice","+ICMS_2","","ICMS Saída Interno 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-ICMS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_icms_externo","True","ICMS External Sale 0%","ICMS External","0% ICMS E","0.0","sale","tax_excluded","1","tax_group_icms_0","base","invoice","+ICMS_2","","ICMS Saída Externo 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-ICMS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_icms_interno","True","ICMS Internal Purchase 0%","ICMS Internal market","0% ICMS I","0.0","purchase","tax_excluded","1","tax_group_icms_0","base","invoice","-ICMS_2","","ICMS Entrada Interno 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+ICMS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_icms_externo","True","ICMS External Purchase 0%","ICMS External","0% ICMS E","0.0","purchase","tax_excluded","1","tax_group_icms_0","base","invoice","-ICMS_2","","ICMS Entrada Externo 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+ICMS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_icms_externo7","True","ICMS External Sale 7%","ICMS External 7%","7% ICMS E","7.0","sale","tax_excluded","1","tax_group_icms_7","base","invoice","+ICMS_1","","ICMS Saída Externo 7%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_in_icms_externo7","True","ICMS External Purchase 7%","ICMS External 7%","7% ICMS E","7.0","purchase","","1","tax_group_icms_7","base","invoice","+ICMS_1","","ICMS Entrada Externo 7%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_out_icms_externo12","True","ICMS External Sale 12%","ICMS External 12%","12% ICMS E","12.0","sale","tax_excluded","1","tax_group_icms_12","base","invoice","+ICMS_1","","ICMS Saída Externo 12%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_in_icms_externo12","True","ICMS External Purchase 12%","ICMS External 12%","12% ICMS E","12.0","purchase","","1","tax_group_icms_12","base","invoice","+ICMS_1","","ICMS Entrada Externo 12%"
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_101020302",""
"tax_template_out_icms_subist","True","ICMS Subist Sale 0%","ICMS Subist","0% ICMS S","0.0","sale","tax_excluded","0","tax_group_icms_0","base","invoice","+ICMSST_1","","ICMS Saída Subist 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-ICMSST_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_icms_subist","True","ICMS Subist Purchase 0%","ICMS Subist","0% ICMS S","0.0","purchase","tax_excluded","0","tax_group_icms_0","base","invoice","-ICMSST_1","","ICMS Entrada Subist 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+ICMSST_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_ipi10","True","IPI Sale 10%","IPI 10%","10% IPI","10.0","sale","tax_excluded","0","tax_group_ipi_10","base","invoice","+IPI_1","","IPI Saída 10%"
"","","","","","","","","","","tax","invoice","+IPI_2","account_template_201010902",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","-IPI_2","account_template_101020301",""
"tax_template_in_ipi10","True","IPI Purchase 10%","IPI 10%","10% IPI","10.0","purchase","tax_excluded","0","tax_group_ipi_10","base","invoice","-IPI_1","","IPI Entrada 10%"
"","","","","","","","","","","tax","invoice","-IPI_2","account_template_101020301",""
"","","","","","","","","","","base","refund","+IPI_1","",""
"","","","","","","","","","","tax","refund","+IPI_2","account_template_201010902",""
"tax_template_out_ipi","True","IPI Sale 0%","IPI","0% IPI","0.0","sale","tax_excluded","0","tax_group_ipi_0","base","invoice","+IPI_1","","IPI Saída 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_ipi","True","IPI Purchase 0%","IPI","0% IPI","0.0","purchase","tax_excluded","0","tax_group_ipi_0","base","invoice","-IPI_1","","IPI Entrada 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+IPI_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_pis","True","PIS Sale 0%","PIS","0% PIS","0.0","sale","tax_excluded","1","tax_group_pis_0","base","invoice","+PIS_2","","PIS Saída 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-PIS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_pis065","True","PIS Sale 0.65%","PIS 0,65%","0.65% PIS","0.65","sale","tax_excluded","1","tax_group_pis_065","base","invoice","+PIS_1","","PIS Saída 0,65%"
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_101020303",""
"tax_template_in_pis","True","PIS Purchase 0%","PIS","0% PIS","0.0","purchase","tax_excluded","1","tax_group_pis_0","base","invoice","-PIS_2","","PIS Entrada 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+PIS_2","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_pis065","True","PIS Purchase 0.65%","PIS 0,65%","0.65% PIS","0.65","purchase","tax_excluded","1","tax_group_pis_065","base","invoice","-PIS_1","","PIS Entrada 0,65%"
"","","","","","","","","","","tax","invoice","-PIS_2","account_template_101020303",""
"","","","","","","","","","","base","refund","+PIS_1","",""
"","","","","","","","","","","tax","refund","+PIS_2","account_template_201010904",""
"tax_template_out_cofins","True","COFINS Sale 0%","COFINS","0% COFINS","0.0","sale","tax_excluded","1","tax_group_cofins_0","base","invoice","+COFINS_1","","COFINS Saída 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_cofins3","True","COFINS Sale 3%","COFINS 3%","3% COFINS","3.0","sale","tax_excluded","1","tax_group_cofins_3","base","invoice","+COFINS_1","","COFINS Saída 3%"
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_101020305",""
"tax_template_in_cofins","True","COFINS Purchase 0%","COFINS","0% COFINS","0.0","purchase","tax_excluded","1","tax_group_cofins_0","base","invoice","-COFINS_1","","COFINS Entrada 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","+COFINS_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_cofins3","True","COFINS Purchase 3%","COFINS 3%","3% COFINS","3.0","purchase","tax_excluded","1","tax_group_cofins_3","base","invoice","-COFINS_1","","COFINS Entrada 3%"
"","","","","","","","","","","tax","invoice","-COFINS_2","account_template_101020305",""
"","","","","","","","","","","base","refund","+COFINS_1","",""
"","","","","","","","","","","tax","refund","+COFINS_2","account_template_201010905",""
"tax_template_out_irpj","True","IRPJ 0%","IRPJ","0% IRPJ","0.0","sale","tax_excluded","1","tax_group_irpj_0","base","invoice","+IRPJ_1","","IRPJ 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-IRPJ_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_ir","True","IR 0%","IR","0% IR","0.0","sale","tax_excluded","1","tax_group_ir_0","base","invoice","+IR_1","","IR 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-IR_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_issqn2","True","ISSQN Sale 2%","ISSQN 2%","2% ISSQN","2.0","sale","tax_excluded","1","tax_group_issqn_2","base","invoice","+ISSQN_1","","ISSQN Saída 2%"
"","","","","","","","","","","tax","invoice","+ISSQN_2","account_template_101020340",""
"","","","","","","","","","","base","refund","-ISSQN_1","",""
"","","","","","","","","","","tax","refund","-ISSQN_2","account_template_201010928",""
"tax_template_in_issqn2","True","ISSQN Purchase 2%","ISSQN 2%","2% ISSQN","2.0","purchase","tax_excluded","1","tax_group_issqn_2","base","invoice","-ISSQN_1","","ISSQN Entrada 2%"
"","","","","","","","","","","tax","invoice","-ISSQN_2","account_template_101020340",""
"","","","","","","","","","","base","refund","+ISSQN_1","",""
"","","","","","","","","","","tax","refund","+ISSQN_2","account_template_201010928",""
"tax_template_out_csll","True","CSLL 0%","CSLL","0% CSLL","0.0","sale","tax_excluded","1","tax_group_csll_0","base","invoice","+CSLL_1","","CSLL 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-CSLL_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_ii0","True","II Sale 0%","II","0% II","0.0","sale","tax_excluded","0","tax_group_ii_0","base","invoice","+II_1","","II Saída 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-II_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_ii0","True","II Purchase 0%","II","0% II","0.0","purchase","tax_excluded","0","tax_group_ii_0","base","invoice","+II_1","","II Entrada 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-II_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_inss0","True","INSS Sale 0%","INSS","0% INSS","0.0","sale","tax_excluded","0","tax_group_inss_0","base","invoice","+INSS_1","","INSS Saída 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_in_inss0","True","INSS Purchase 0%","INSS","0% INSS","0.0","purchase","tax_excluded","0","tax_group_inss_0","base","invoice","+INSS_1","","INSS Entrada 0%"
"","","","","","","","","","","tax","invoice","","",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","","",""
"tax_template_out_aproxtrib_fed_incl_goods","False","","Approximated Federal Taxation Incl.","Approximated Federal Taxation Incl.","1.00","sale","tax_included","","tax_group_aproxtrib_fed_incl_goods","base","invoice","","","Tributação Federal Aproximada Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_aproxtrib_fed_excl_goods","False","","Approximated Federal Taxation Excl.","Approximated Federal Taxation Excl.","1.00","sale","tax_excluded","","tax_group_aproxtrib_fed_excl_goods","base","invoice","","","Tributação Federal Aproximada Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_aproxtrib_state_incl_goods","False","","Approximated State Taxation Incl.","Approximated State Taxation Incl.","1.00","sale","tax_included","","tax_group_aproxtrib_state_incl_goods","base","invoice","","","Tributação Estadual Aproximada Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_aproxtrib_state_excl_goods","False","","Approximated State Taxation Excl.","Approximated State Taxation Excl.","1.00","sale","tax_excluded","","tax_group_aproxtrib_state_excl_goods","base","invoice","","","Tributação Estadual Aproximada Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_cofins_incl_goods","False","","COFINS Incl.","COFINS Incl.","1.00","sale","tax_included","","tax_group_cofins_incl_goods","base","invoice","+COFINS_1","","COFINS Incl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_excl_goods","False","","COFINS Excl.","COFINS Excl.","1.00","sale","tax_excluded","","tax_group_cofins_excl_goods","base","invoice","+COFINS_1","","COFINS Excl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_deson_incl_goods","False","","COFINS Exemption Incl.","COFINS Exemption Incl.","1.00","sale","tax_included","","tax_group_cofins_deson_incl_goods","base","invoice","+COFINS_1","","COFINS Desoneração Incl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_deson_excl_goods","False","","COFINS Exemption Excl.","COFINS Exemption Excl.","1.00","sale","tax_excluded","","tax_group_cofins_deson_excl_goods","base","invoice","+COFINS_1","","COFINS Desoneração Excl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_st_incl_goods","False","","COFINS ST Incl.","COFINS ST Incl.","1.00","sale","tax_included","","tax_group_cofins_st_incl_goods","base","invoice","+COFINS_1","","COFINS ST Incl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_st_excl_goods","False","","COFINS ST Excl.","COFINS ST Excl.","1.00","sale","tax_excluded","","tax_group_cofins_st_excl_goods","base","invoice","+COFINS_1","","COFINS ST Excl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_icms_incl_goods","False","","ICMS Incl.","ICMS Incl.","1.00","sale","tax_included","","tax_group_icms_incl_goods","base","invoice","+ICMS_1","","ICMS Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_excl_goods","False","","ICMS Excl.","ICMS Excl.","1.00","sale","tax_excluded","","tax_group_icms_excl_goods","base","invoice","+ICMS_1","","ICMS Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_credsn_incl_goods","False","","ICMS CredSN Incl.","ICMS CredSN Incl.","1.00","sale","tax_included","","tax_group_icms_credsn_incl_goods","base","invoice","+ICMS_1","","ICMS CredSN Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_credsn_excl_goods","False","","ICMS CredSN Excl.","ICMS CredSN Excl.","1.00","sale","tax_excluded","","tax_group_icms_credsn_excl_goods","base","invoice","+ICMS_1","","ICMS CredSN Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_deson_incl_goods","False","","ICMS Exemption Incl.","ICMS Exemption Incl.","1.00","sale","tax_included","","tax_group_icms_deson_incl_goods","base","invoice","+ICMS_1","","ICMS Desoneração Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_deson_excl_goods","False","","ICMS Exemption Excl.","ICMS Exemption Excl.","1.00","sale","tax_excluded","","tax_group_icms_deson_excl_goods","base","invoice","+ICMS_1","","ICMS Desoneração Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_dest_incl_goods","False","","ICMS DIFA Recipient Incl.","ICMS DIFA Recipient Incl.","1.00","sale","tax_included","","tax_group_icms_difa_dest_incl_goods","base","invoice","+ICMS_1","","ICMS DIFA Destinatário Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_dest_excl_goods","False","","ICMS DIFA Recipient Excl.","ICMS DIFA Recipient Excl.","1.00","sale","tax_excluded","","tax_group_icms_difa_dest_excl_goods","base","invoice","+ICMS_1","","ICMS DIFA Destinatário Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_fcp_incl_goods","False","","ICMS DIFA FCP Incl.","ICMS DIFA FCP Incl.","1.00","sale","tax_included","","tax_group_icms_difa_fcp_incl_goods","base","invoice","+ICMS_1","","ICMS DIFA FCP Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_fcp_excl_goods","False","","ICMS DIFA FCP Excl.","ICMS DIFA FCP Excl.","1.00","sale","tax_excluded","","tax_group_icms_difa_fcp_excl_goods","base","invoice","+ICMS_1","","ICMS DIFA FCP Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_remet_incl_goods","False","","ICMS DIFA Sender Incl.","ICMS DIFA Sender Incl.","1.00","sale","tax_included","","tax_group_icms_difa_remet_incl_goods","base","invoice","+ICMS_1","","ICMS DIFA Remetente Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_difa_remet_excl_goods","False","","ICMS DIFA Sender Excl.","ICMS DIFA Sender Excl.","1.00","sale","tax_excluded","","tax_group_icms_difa_remet_excl_goods","base","invoice","+ICMS_1","","ICMS DIFA Remetente Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_eff_incl_goods","False","","ICMS EFF Incl.","ICMS EFF Incl.","1.00","sale","tax_included","","tax_group_icms_eff_incl_goods","base","invoice","+ICMS_1","","ICMS EFF Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_eff_excl_goods","False","","ICMS EFF Excl.","ICMS EFF Excl.","1.00","sale","tax_excluded","","tax_group_icms_eff_excl_goods","base","invoice","+ICMS_1","","ICMS EFF Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_fcp_incl_goods","False","","ICMS FCP Incl.","ICMS FCP Incl.","1.00","sale","tax_included","","tax_group_icms_fcp_incl_goods","base","invoice","+ICMS_1","","ICMS FCP Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_fcp_excl_goods","False","","ICMS FCP Excl.","ICMS FCP Excl.","1.00","sale","tax_excluded","","tax_group_icms_fcp_excl_goods","base","invoice","+ICMS_1","","ICMS FCP Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_own_payer_incl_goods","False","","ICMS Own Issuer Incl.","ICMS Own Issuer Incl.","1.00","sale","tax_included","","tax_group_icms_own_payer_incl_goods","base","invoice","+ICMS_1","","ICMS Próprio Emitente Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_own_payer_excl_goods","False","","ICMS Own Issuer Excl.","ICMS Own Issuer Excl.","1.00","sale","tax_excluded","","tax_group_icms_own_payer_excl_goods","base","invoice","+ICMS_1","","ICMS Próprio Emitente Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_part_incl_goods","False","","ICMS Sharing Incl.","ICMS Sharing Incl.","1.00","sale","tax_included","","tax_group_icms_part_incl_goods","base","invoice","+ICMS_1","","ICMS Partilha Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_part_excl_goods","False","","ICMS Sharing Excl.","ICMS Sharing Excl.","1.00","sale","tax_excluded","","tax_group_icms_part_excl_goods","base","invoice","+ICMS_1","","ICMS Partilha Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_rf_incl_goods","False","","ICMS RF Incl.","ICMS RF Incl.","1.00","sale","tax_included","","tax_group_icms_rf_incl_goods","base","invoice","+ICMS_1","","ICMS RF Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_rf_excl_goods","False","","ICMS RF Excl.","ICMS RF Excl.","1.00","sale","tax_excluded","","tax_group_icms_rf_excl_goods","base","invoice","+ICMS_1","","ICMS RF Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_incl_goods","False","","ICMS ST Incl.","ICMS ST Incl.","1.00","sale","tax_included","","tax_group_icms_st_incl_goods","base","invoice","+ICMS_1","","ICMS ST Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_excl_goods","False","","ICMS ST Excl.","ICMS ST Excl.","1.00","sale","tax_excluded","","tax_group_icms_st_excl_goods","base","invoice","+ICMS_1","","ICMS ST Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_fcp_incl_goods","False","","ICMS ST FCP Incl.","ICMS ST FCP Incl.","1.00","sale","tax_included","","tax_group_icms_st_fcp_incl_goods","base","invoice","+ICMS_1","","ICMS ST FCP Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_fcp_excl_goods","False","","ICMS ST FCP Excl.","ICMS ST FCP Excl.","1.00","sale","tax_excluded","","tax_group_icms_st_fcp_excl_goods","base","invoice","+ICMS_1","","ICMS ST FCP Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_fcppart_incl_goods","False","","ICMS ST FCP Sharing Incl.","ICMS ST FCP Sharing Incl.","1.00","sale","tax_included","","tax_group_icms_st_fcppart_incl_goods","base","invoice","+ICMS_1","","ICMS ST FCP Partilha Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_fcppart_excl_goods","False","","ICMS ST FCP Sharing Excl.","ICMS ST FCP Sharing Excl.","1.00","sale","tax_excluded","","tax_group_icms_st_fcppart_excl_goods","base","invoice","+ICMS_1","","ICMS ST FCP Partilha Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_part_incl_goods","False","","ICMS ST Sharing Incl.","ICMS ST Sharing Incl.","1.00","sale","tax_included","","tax_group_icms_st_part_incl_goods","base","invoice","+ICMS_1","","ICMS ST Partilha Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_part_excl_goods","False","","ICMS ST Sharing Excl.","ICMS ST Sharing Excl.","1.00","sale","tax_excluded","","tax_group_icms_st_part_excl_goods","base","invoice","+ICMS_1","","ICMS ST Partilha Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_sd_incl_goods","False","","ICMS ST SD Incl.","ICMS ST SD Incl.","1.00","sale","tax_included","","tax_group_icms_st_sd_incl_goods","base","invoice","+ICMS_1","","ICMS ST SD Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_sd_excl_goods","False","","ICMS ST SD Excl.","ICMS ST SD Excl.","1.00","sale","tax_excluded","","tax_group_icms_st_sd_excl_goods","base","invoice","+ICMS_1","","ICMS ST SD Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_sd_fcp_incl_goods","False","","ICMS ST SD FCP Incl.","ICMS ST SD FCP Incl.","1.00","sale","tax_included","","tax_group_icms_st_sd_fcp_incl_goods","base","invoice","+ICMS_1","","ICMS ST SD FCP Incl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_icms_st_sd_fcp_excl_goods","False","","ICMS ST SD FCP Excl.","ICMS ST SD FCP Excl.","1.00","sale","tax_excluded","","tax_group_icms_st_sd_fcp_excl_goods","base","invoice","+ICMS_1","","ICMS ST SD FCP Excl."
"","","","","","","","","","","tax","invoice","+ICMS_2","account_template_201010903",""
"","","","","","","","","","","base","refund","-ICMS_1","",""
"","","","","","","","","","","tax","refund","-ICMS_2","account_template_201010903",""
"tax_template_out_ii_incl_goods","False","","II - Import Tax Incl.","II - Import Tax Incl.","1.00","sale","tax_included","","tax_group_ii_incl_goods","base","invoice","+II_1","","II - Imposto de Importação Incl."
"","","","","","","","","","","tax","invoice","+II_2","account_template_201010928",""
"","","","","","","","","","","base","refund","-II_1","",""
"","","","","","","","","","","tax","refund","-II_2","account_template_201010928",""
"tax_template_out_ii_excl_goods","False","","II - Import Tax Excl.","II - Import Tax Excl.","1.00","sale","tax_excluded","","tax_group_ii_excl_goods","base","invoice","+II_1","","II - Imposto de Importação Excl."
"","","","","","","","","","","tax","invoice","+II_2","account_template_201010928",""
"","","","","","","","","","","base","refund","-II_1","",""
"","","","","","","","","","","tax","refund","-II_2","account_template_201010928",""
"tax_template_out_iof_incl_goods","False","","IOF - Tax on Financial Operations Incl.","IOF - Tax on Financial Operations Incl.","1.00","sale","tax_included","","tax_group_iof_incl_goods","base","invoice","","","IOF - Imposto sobre Operações Financeiras Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010906",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010906",""
"tax_template_out_iof_excl_goods","False","","IOF - Tax on Financial Operations Excl.","IOF - Tax on Financial Operations Excl.","1.00","sale","tax_excluded","","tax_group_iof_excl_goods","base","invoice","","","IOF - Imposto sobre Operações Financeiras Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010906",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010906",""
"tax_template_out_ipi_incl_goods","False","","IPI Incl.","IPI Incl.","1.00","sale","tax_included","","tax_group_ipi_incl_goods","base","invoice","+IPI_1","","IPI Incl."
"","","","","","","","","","","tax","invoice","+IPI_2","account_template_201010902",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","-IPI_2","account_template_201010902",""
"tax_template_out_ipi_excl_goods","False","","IPI Excl.","IPI Excl.","1.00","sale","tax_excluded","","tax_group_ipi_excl_goods","base","invoice","+IPI_1","","IPI Excl."
"","","","","","","","","","","tax","invoice","+IPI_2","account_template_201010902",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","-IPI_2","account_template_201010902",""
"tax_template_out_ipi_returned_incl_goods","False","","IPI Returned Incl.","IPI Returned Incl.","1.00","sale","tax_included","","tax_group_ipi_returned_incl_goods","base","invoice","+IPI_1","","IPI Retornado Incl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010902",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","-IPI_2","account_template_201010902",""
"tax_template_out_ipi_returned_excl_goods","False","","IPI Returned Excl.","IPI Returned Excl.","1.00","sale","tax_excluded","","tax_group_ipi_returned_excl_goods","base","invoice","+IPI_1","","IPI Retornado Excl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010902",""
"","","","","","","","","","","base","refund","-IPI_1","",""
"","","","","","","","","","","tax","refund","-IPI_2","account_template_201010902",""
"tax_template_out_pis_incl_goods","False","","PIS Incl.","PIS Incl.","1.00","sale","tax_included","","tax_group_pis_incl_goods","base","invoice","+PIS_1","","PIS Incl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_excl_goods","False","","PIS Excl.","PIS Excl.","1.00","sale","tax_excluded","","tax_group_pis_excl_goods","base","invoice","+PIS_1","","PIS Excl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_deson_incl_goods","False","","PIS Exemption Incl.","PIS Exemption Incl.","1.00","sale","tax_included","","tax_group_pis_deson_incl_goods","base","invoice","+PIS_1","","PIS Desoneração Incl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_deson_excl_goods","False","","PIS Exemption Excl.","PIS Exemption Excl.","1.00","sale","tax_excluded","","tax_group_pis_deson_excl_goods","base","invoice","+PIS_1","","PIS Desoneração Excl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_st_incl_goods","False","","PIS ST Incl.","PIS ST Incl.","1.00","sale","tax_included","","tax_group_pis_st_incl_goods","base","invoice","+PIS_1","","PIS ST Incl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_st_excl_goods","False","","PIS ST Excl.","PIS ST Excl.","1.00","sale","tax_excluded","","tax_group_pis_st_excl_goods","base","invoice","+PIS_1","","PIS ST Excl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_aproxtrib_city_incl_service","False","","Approximated City Taxation Incl.","Approximated City Taxation Incl.","1","sale","tax_included","","tax_group_aproxtrib_city_incl_services","base","invoice","","","Tributação Municipal Aproximada Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010908",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010908",""
"tax_template_out_aproxtrib_city_excl_service","False","","Approximated City Taxation Excl.","Approximated City Taxation Excl.","1","sale","tax_excluded","","tax_group_aproxtrib_city_excl_services","base","invoice","","","Tributação Municipal Aproximada Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010908",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010908",""
"tax_template_out_pis_rf_incl_service","False","","PIS RF Incl.","PIS RF Incl.","1","sale","tax_included","","tax_group_pis_rf_incl_services","base","invoice","+PIS_1","","PIS RF Incl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_pis_rf_excl_service","False","","PIS RF Excl.","PIS RF Excl.","1","sale","tax_excluded","","tax_group_pis_rf_excl_services","base","invoice","+PIS_1","","PIS RF Excl."
"","","","","","","","","","","tax","invoice","+PIS_2","account_template_201010904",""
"","","","","","","","","","","base","refund","-PIS_1","",""
"","","","","","","","","","","tax","refund","-PIS_2","account_template_201010904",""
"tax_template_out_cofins_rf_incl_service","False","","COFINS RF Incl.","COFINS RF Incl.","1","sale","tax_included","","tax_group_cofins_rf_incl_services","base","invoice","+COFINS_1","","COFINS RF Incl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_cofins_rf_excl_service","False","","COFINS RF Excl.","COFINS RF Excl.","1","sale","tax_excluded","","tax_group_cofins_rf_excl_services","base","invoice","+COFINS_1","","COFINS RF Excl."
"","","","","","","","","","","tax","invoice","+COFINS_2","account_template_201010905",""
"","","","","","","","","","","base","refund","-COFINS_1","",""
"","","","","","","","","","","tax","refund","-COFINS_2","account_template_201010905",""
"tax_template_out_csll_incl_service","False","","CSLL Incl.","CSLL Incl.","1","sale","tax_included","","tax_group_csll_incl_services","base","invoice","+CSLL_1","","CSLL Incl."
"","","","","","","","","","","tax","invoice","+CSLL_2","account_template_201010914",""
"","","","","","","","","","","base","refund","-CSLL_1","",""
"","","","","","","","","","","tax","refund","-CSLL_2","account_template_201010914",""
"tax_template_out_csll_excl_service","False","","CSLL Excl.","CSLL Excl.","1","sale","tax_excluded","","tax_group_csll_excl_services","base","invoice","+CSLL_1","","CSLL Excl."
"","","","","","","","","","","tax","invoice","+CSLL_2","account_template_201010914",""
"","","","","","","","","","","base","refund","-CSLL_1","",""
"","","","","","","","","","","tax","refund","-CSLL_2","account_template_201010914",""
"tax_template_out_csll_rf_incl_service","False","","CSLL RF Incl.","CSLL RF Incl.","1","sale","tax_included","","tax_group_csll_rf_incl_services","base","invoice","+CSLL_1","","CSLL RF Incl."
"","","","","","","","","","","tax","invoice","+CSLL_2","account_template_201010914",""
"","","","","","","","","","","base","refund","-CSLL_1","",""
"","","","","","","","","","","tax","refund","-CSLL_2","account_template_201010914",""
"tax_template_out_csll_rf_excl_service","False","","CSLL RF Excl.","CSLL RF Excl.","1","sale","tax_excluded","","tax_group_csll_rf_excl_services","base","invoice","+CSLL_1","","CSLL RF Excl."
"","","","","","","","","","","tax","invoice","+CSLL_2","account_template_201010914",""
"","","","","","","","","","","base","refund","-CSLL_1","",""
"","","","","","","","","","","tax","refund","-CSLL_2","account_template_201010914",""
"tax_template_out_iss_incl_service","False","","ISS Incl.","ISS Incl.","1","sale","tax_included","","tax_group_iss_incl_services","base","invoice","+ISSQN_1","","ISS Incl."
"","","","","","","","","","","tax","invoice","+ISSQN_2","account_template_201010908",""
"","","","","","","","","","","base","refund","-ISSQN_1","",""
"","","","","","","","","","","tax","refund","-ISSQN_2","account_template_201010908",""
"tax_template_out_iss_excl_service","False","","ISS Excl.","ISS Excl.","1","sale","tax_excluded","","tax_group_iss_excl_services","base","invoice","+ISSQN_1","","ISS Excl."
"","","","","","","","","","","tax","invoice","+ISSQN_2","account_template_201010908",""
"","","","","","","","","","","base","refund","-ISSQN_1","",""
"","","","","","","","","","","tax","refund","-ISSQN_2","account_template_201010908",""
"tax_template_out_iss_rf_incl_service","False","","ISS RF Incl.","ISS RF Incl.","1","sale","tax_included","","tax_group_iss_rf_incl_services","base","invoice","+ISSQN_1","","ISS RF Incl."
"","","","","","","","","","","tax","invoice","+ISSQN_2","account_template_201010908",""
"","","","","","","","","","","base","refund","-ISSQN_1","",""
"","","","","","","","","","","tax","refund","-ISSQN_2","account_template_201010908",""
"tax_template_out_iss_rf_excl_service","False","","ISS RF Excl.","ISS RF Excl.","1","sale","tax_excluded","","tax_group_iss_rf_excl_services","base","invoice","+ISSQN_1","","ISS RF Excl."
"","","","","","","","","","","tax","invoice","+ISSQN_2","account_template_201010908",""
"","","","","","","","","","","base","refund","-ISSQN_1","",""
"","","","","","","","","","","tax","refund","-ISSQN_2","account_template_201010908",""
"tax_template_out_ir_pj_incl_service","False","","IR PJ Incl.","IR PJ Incl.","1","sale","tax_included","","tax_group_ir_pj_incl_services","base","invoice","+IRPJ_1","","IR PJ Incl."
"","","","","","","","","","","tax","invoice","+IRPJ_2","account_template_201010901",""
"","","","","","","","","","","base","refund","-IRPJ_1","",""
"","","","","","","","","","","tax","refund","-IRPJ_2","account_template_201010901",""
"tax_template_out_ir_pj_excl_service","False","","IR PJ Excl.","IR PJ Excl.","1","sale","tax_excluded","","tax_group_ir_pj_excl_services","base","invoice","+IRPJ_1","","IR PJ Excl."
"","","","","","","","","","","tax","invoice","+IRPJ_2","account_template_201010901",""
"","","","","","","","","","","base","refund","-IRPJ_1","",""
"","","","","","","","","","","tax","refund","-IRPJ_2","account_template_201010901",""
"tax_template_out_ir_rf_incl_service","False","","IR RF Incl.","IR RF Incl.","1","sale","tax_included","","tax_group_ir_rf_incl_services","base","invoice","+IR_1","","IR RF Incl."
"","","","","","","","","","","tax","invoice","+IR_2","account_template_201010901",""
"","","","","","","","","","","base","refund","-IR_1","",""
"","","","","","","","","","","tax","refund","-IR_2","account_template_201010901",""
"tax_template_out_ir_rf_excl_service","False","","IR RF Excl.","IR RF Excl.","1","sale","tax_excluded","","tax_group_ir_rf_excl_services","base","invoice","+IR_1","","IR RF Excl."
"","","","","","","","","","","tax","invoice","+IR_2","account_template_201010901",""
"","","","","","","","","","","base","refund","-IR_1","",""
"","","","","","","","","","","tax","refund","-IR_2","account_template_201010901",""
"tax_template_out_cprb_incl_service","False","","CPRB Incl.","CPRB Incl.","1","sale","tax_included","","tax_group_cprb_incl_services","base","invoice","","","CPRB Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_cprb_excl_service","False","","CPRB Excl.","CPRB Excl.","1","sale","tax_excluded","","tax_group_cprb_excl_services","base","invoice","","","CPRB Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_cprb_rf_incl_service","False","","CPRB RF Incl.","CPRB RF Incl.","1","sale","tax_included","","tax_group_cprb_rf_incl_services","base","invoice","","","CPRB RF Incl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_cprb_rf_excl_service","False","","CPRB RF Excl.","CPRB RF Excl.","1","sale","tax_excluded","","tax_group_cprb_rf_excl_services","base","invoice","","","CPRB RF Excl."
"","","","","","","","","","","tax","invoice","","account_template_201010928",""
"","","","","","","","","","","base","refund","","",""
"","","","","","","","","","","tax","refund","","account_template_201010928",""
"tax_template_out_inss_ar_incl_service","False","","INSS AR Incl.","INSS AR Incl.","1","sale","tax_included","","tax_group_inss_ar_incl_services","base","invoice","+INSS_1","","INSS AR Incl."
"","","","","","","","","","","tax","invoice","+INSS_2","account_template_201010103",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","-INSS_2","account_template_201010103",""
"tax_template_out_inss_ar_excl_service","False","","INSS AR Excl.","INSS AR Excl.","1","sale","tax_excluded","","tax_group_inss_ar_excl_services","base","invoice","+INSS_1","","INSS AR Excl."
"","","","","","","","","","","tax","invoice","+INSS_2","account_template_201010103",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","-INSS_2","account_template_201010103",""
"tax_template_out_inss_rf_incl_service","False","","INSS RF Incl.","INSS RF Incl.","1","sale","tax_included","","tax_group_inss_rf_incl_services","base","invoice","+INSS_1","","INSS RF Incl."
"","","","","","","","","","","tax","invoice","+INSS_2","account_template_201010103",""
"","","","","","","","","","","base","refund","-INSS_1","",""
"","","","","","","","","","","tax","refund","-INSS_2","account_template_201010103",""
"tax_template_out_inss_rf_excl_service","False","","INSS RF Excl.","INSS RF Excl.","1","sale","tax_excluded","","tax_group_inss_rf_excl_services","base","invoice","+INSS_1","","INSS RF Excl."
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

## File: models\l10n_br_zip_range.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re

from odoo import models, fields, api, _
from odoo.exceptions import ValidationError


class L10nBrZipRange(models.Model):
    _name = "l10n_br.zip.range"
    _description = "Brazilian city zip range"

    city_id = fields.Many2one("res.city", string="City", required=True)
    start = fields.Char(string="From", required=True)
    end = fields.Char(string="To", required=True)

    _sql_constraints = [
        ("uniq_start", "unique(start)", 'The "from" zip must be unique'),
        ("uniq_end", 'unique("end")', 'The "to" zip must be unique.'),
    ]

    @api.constrains("start", "end")
    def _check_range(self):
        zip_format = re.compile(r"\d{5}-\d{3}")
        for zip_range in self:
            if not zip_format.fullmatch(zip_range.start) or not zip_format.fullmatch(zip_range.end):
                raise ValidationError(
                    _(
                        "Invalid zip range format: %(start)s %(end)s. It should follow this format: 01000-001",
                        start=zip_range.start,
                        end=zip_range.end,
                    )
                )

            if zip_range.start >= zip_range.end:
                raise ValidationError(
                    _("Start should be less than end: %(start)s %(end)s", start=zip_range.start, end=zip_range.end)
                )

```

## File: models\res_city.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api


class ResCity(models.Model):
    _inherit = "res.city"

    l10n_br_zip_range_ids = fields.One2many(
        string="Zip Ranges",
        comodel_name="l10n_br.zip.range",
        inverse_name="city_id",
        help="Brazil: technical field that maps a city to one or more zip code ranges.",
    )

    l10n_br_zip_ranges = fields.Char(
        string="Frontend Zip Ranges",
        compute="_compute_l10n_br_zip_ranges",
        help="Brazil: technical field that maps a city to one or more zip code ranges for the frontend.",
    )

    @api.depends("l10n_br_zip_range_ids")
    def _compute_l10n_br_zip_ranges(self):
        for city in self:
            city.l10n_br_zip_ranges = " ".join(
                city.l10n_br_zip_range_ids.mapped(lambda zip_range: f"[{zip_range.start} {zip_range.end}]")
            )

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

## File: models\res_partner_bank.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re

from odoo import models, fields, api, _
from odoo.addons.mail.tools.mail_validation import mail_validate
from odoo.exceptions import ValidationError
from odoo.tools import float_repr


class ResPartnerBank(models.Model):
    _inherit = "res.partner.bank"

    proxy_type = fields.Selection(
        selection_add=[
            ("email", "Email Address"),
            ("mobile", "Mobile Number"),
            ("br_cpf_cnpj", "CPF/CNPJ (BR)"),
            ("br_random", "Random Key (BR)"),
        ],
        ondelete={
            "email": "set default",
            "mobile": "set default",
            "br_cpf_cnpj": "set default",
            "br_random": "set default",
        },
    )

    @api.constrains("proxy_type", "proxy_value", "partner_id")
    def _check_br_proxy(self):
        for bank in self.filtered(lambda bank: bank.country_code == "BR" and bank.proxy_type != "none"):
            if bank.proxy_type not in ("email", "mobile", "br_cpf_cnpj", "br_random"):
                raise ValidationError(
                    _(
                        "The proxy type must be Email Address, Mobile Number, CPF/CNPJ (BR) or Random Key (BR) for Pix code generation."
                    )
                )

            value = bank.proxy_value
            if bank.proxy_type == "email" and not mail_validate(value):
                raise ValidationError(_("%s is not a valid email.", value))

            if bank.proxy_type == "br_cpf_cnpj" and (
                not self.partner_id.check_vat_br(value) or any(not char.isdecimal() for char in value)
            ):
                raise ValidationError(_("%s is not a valid CPF or CNPJ (don't include periods or dashes).", value))

            if bank.proxy_type == "mobile" and (not value or not value.startswith("+55") or len(value) != 14):
                raise ValidationError(
                    _(
                        "The mobile number %s is invalid. It must start with +55, contain a 2 digit territory or state code followed by a 9 digit number.",
                        value,
                    )
                )

            regex = r"%(char)s{8}-%(char)s{4}-%(char)s{4}-%(char)s{4}-%(char)s{12}" % {"char": "[a-fA-F0-9]"}
            if bank.proxy_type == "br_random" and not re.fullmatch(regex, bank.proxy_value):
                raise ValidationError(
                    _(
                        "The random key %s is invalid, the format looks like this: 71d6c6e1-64ea-4a11-9560-a10870c40ca2",
                        value,
                    )
                )

    @api.depends("country_code")
    def _compute_display_qr_setting(self):
        """Override."""
        bank_br = self.filtered(lambda b: b.country_code == "BR")
        bank_br.display_qr_setting = True
        super(ResPartnerBank, self - bank_br)._compute_display_qr_setting()

    def _get_additional_data_field(self, comment):
        """Override."""
        if self.country_code == "BR":
            # Only include characters allowed by the Pix spec.
            return self._serialize(5, re.sub(r"[^a-zA-Z0-9*]", "", comment))
        return super()._get_additional_data_field(comment)

    def _get_qr_code_vals_list(self, *args, **kwargs):
        """Override. Force the amount field to always have two decimals. Uppercase the merchant name and merchant city.
        Although not specified explicitly in the spec, not uppercasing causes errors when scanning the code. Also ensure
        there is always some comment set."""
        res = super()._get_qr_code_vals_list(*args, **kwargs)
        if self.country_code == "BR":
            res[5] = (res[5][0], float_repr(res[5][1], 2) if res[5][1] else None)  # amount
            res[7] = (res[7][0], res[7][1].upper())  # merchant_name
            res[8] = (res[8][0], res[8][1].upper())  # merchant_city
            if not res[9][1]:
                res[9] = (res[9][0], self._get_additional_data_field("***"))  # default comment if none is set
        return res

    def _get_merchant_account_info(self):
        """Override."""
        if self.country_code == "BR":
            merchant_account_info_data = (
                (0, "br.gov.bcb.pix"),  # GUI
                (1, self.proxy_value),  # key
            )
            return 26, "".join(self._serialize(*val) for val in merchant_account_info_data)

        return super()._get_merchant_account_info()

    def _get_error_messages_for_qr(self, qr_method, debtor_partner, currency):
        """Override."""
        if qr_method == "emv_qr" and self.country_code == "BR":
            if currency.name != "BRL":
                return _("Can't generate a Pix QR code with a currency other than BRL.")
            return None

        return super()._get_error_messages_for_qr(qr_method, debtor_partner, currency)

    def _check_for_qr_code_errors(
        self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication
    ):
        """Override."""
        if (
            qr_method == "emv_qr"
            and self.country_code == "BR"
            and self.proxy_type not in ("email", "mobile", "br_cpf_cnpj", "br_random")
        ):
            return _(
                "To generate a Pix code the proxy type for %s must be Email Address, Mobile Number, CPF/CNPJ (BR) or Random Key (BR).",
                self.display_name,
            )

        return super()._check_for_qr_code_errors(
            qr_method, amount, currency, debtor_partner, free_communication, structured_communication
        )

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
from . import l10n_br_zip_range
from . import res_partner
from . import res_city
from . import res_company
from . import res_partner_bank

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
l10n_br.access_l10n_br_zip_range_group_manager,access_l10n_br_zip_range,l10n_br.model_l10n_br_zip_range,base.group_partner_manager,1,1,1,1
l10n_br.access_l10n_br_zip_range_group_user,access_l10n_br_zip_range,l10n_br.model_l10n_br_zip_range,base.group_user,1,0,0,0

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

## File: views\ir_ui_menu_brazil.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem
        id="brazilian_accounting_menu"
        name="Brazil"
        parent="account.menu_finance_configuration"
        sequence="25"/>
</odoo>

```

## File: views\res_bank_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_bank_form_inherit_account" model="ir.ui.view">
        <field name="name">res.partner.bank.form.inherit</field>
        <field name="model">res.partner.bank</field>
        <field name="inherit_id" ref="base.view_partner_bank_form"/>
        <field name="arch" type="xml">
            <field name="include_reference" position="after">
                <p invisible="country_code != 'BR'">
                    <widget name="documentation_link" path="/applications/finance/fiscal_localizations/brazil.html" label="Documentation"/>
                </p>
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
        <record id="br_partner_address_form" model="ir.ui.view">
            <field name="name">partner.form.address.extended</field>
            <field name="model">res.partner</field>
            <field name="priority" eval="900"/>
            <field name="arch" type="xml">
                <form>
                    <div class="o_address_format">
                        <field name="country_enforce_cities" invisible="1"/>
                        <field name="parent_id" invisible="1"/>
                        <field name="type" invisible="1"/>
                        <field name="street" placeholder="Street..." class="o_address_street oe_read_only"
                               readonly="type == 'contact' and parent_id"/>
                        <div class="oe_edit_only o_row">
                            <field name="street_name" placeholder="Street" style="flex: 3 1 auto"
                                   readonly="type == 'contact' and parent_id"/>
                            <span> </span>
                            <field name="street_number" placeholder="Street #" style="flex: 1 1 auto"
                                   readonly="type == 'contact' and parent_id"/>
                            <span> - </span>
                            <field name="street_number2" placeholder="Complement" style="flex: 1 1 auto"
                                   readonly="type == 'contact' and parent_id"/>
                        </div>
                        <field name="street2" placeholder="Neighborhood" class="o_address_street"
                               readonly="type == 'contact' and parent_id"/>
                        <field name="city_id"
                               placeholder="City"
                               class="o_address_city"
                               domain="[('country_id', '=', country_id)]"
                               invisible="not country_enforce_cities"
                               readonly="type == 'contact' and parent_id"
                               context="{'default_country_id': country_id, 'default_state_id': state_id, 'default_zipcode': zip}"/>
                        <field name="city"
                               placeholder="City"
                               class="o_address_city"
                               invisible="country_enforce_cities and (city_id or city in ('', False))"
                               readonly="type == 'contact' and parent_id"/>
                        <field name="state_id"
                               class="o_address_state"
                               placeholder="State"
                               readonly="type == 'contact' and parent_id"
                               options="{'no_open': True, 'no_quick_create': True}"
                               context="{'default_country_id': country_id}"/>
                        <field name="zip" placeholder="ZIP" class="o_address_zip"
                               readonly="type == 'contact' and parent_id"/>
                        <field name="country_id"
                               placeholder="Country"
                               class="o_address_country"
                               readonly="type == 'contact' and parent_id"
                               options="{'no_open': True, 'no_create': True}"/>
                    </div>
                </form>
            </field>
        </record>

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

