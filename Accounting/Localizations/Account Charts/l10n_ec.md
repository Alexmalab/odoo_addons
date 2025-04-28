# Odoo Module: l10n_ec

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
    'name': 'Ecuadorian Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ec'],
    'version': '3.9',
    'description': """
Functional
----------

This module adds accounting features for Ecuadorian localization, which
represent the minimum requirements to operate a business in Ecuador in compliance
with local regulation bodies such as the ecuadorian tax authority -SRI- and the
Superintendency of Companies -Super Intendencia de Compañías-

Follow the next configuration steps:
1. Go to your company and configure your country as Ecuador
2. Install the invoicing or accounting module, everything will be handled automatically

Highlights:
* Ecuadorian chart of accounts will be automatically installed, based on example provided by Super Intendencia de Compañías
* List of taxes (including withholds) will also be installed, you can switch off the ones your company doesn't use
* Fiscal position, document types, list of local banks, list of local states, etc, will also be installed

Technical
---------
Master Data:
* Chart of Accounts, based on recomendation by Super Cías
* Ecuadorian Taxes, Tax Tags, and Tax Groups
* Ecuadorian Fiscal Positions
* Document types (there are about 41 purchase documents types in Ecuador)
* Identification types
* Ecuador banks
* Partners: Consumidor Final, SRI, IESS, and also basic VAT validation
""",
    'author': 'TRESCLOUD, OPA CONSULTING (https://opa-consulting.com)',
    'category': 'Accounting/Localizations/Account Charts',
    'maintainer': 'TRESCLOUD',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/ecuador.html',
    'license': 'LGPL-3',
    'depends': [
        'base',
        'base_iban',
        'account_debit_note',
        'l10n_latam_invoice_document',
        'l10n_latam_base',
        'account',
    ],
    'data': [
        'security/ir.model.access.csv',
        'data/account_tax_report_data.xml',
        'data/res.bank.csv',
        'data/l10n_latam_identification_type_data.xml',
        'data/res_partner_data.xml',
        'data/l10n_latam.document.type.csv',
        'data/l10n_ec.sri.payment.csv',
        'views/root_sri_menu.xml',
        'views/account_tax_view.xml',
        'views/l10n_latam_document_type_view.xml',
        'views/l10n_ec_sri_payment.xml',
        'views/account_journal_view.xml',
        "views/res_partner_view.xml",
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'installable': True,
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report_104" model="account.report">
        <field name="name">104</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ec"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_104_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_parent_line_report_1" model="account.report.line">
                <field name="name">SUMMARY OF SALES AND OTHER TRANSACTIONS FOR THE REPORTING PERIOD</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_parent_line_report_2" model="account.report.line">
                        <field name="name">Local sales (excluding fixed assets) taxed at a rate different from zero</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_401" model="account.report.line">
                                <field name="name">Gross value(401)</field>
                                <field name="code">c401</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_401_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">401 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_411" model="account.report.line">
                                <field name="name">Net value(411)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_411_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">411 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_421" model="account.report.line">
                                <field name="name">Tax generated(421)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_421_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">421 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_6" model="account.report.line">
                        <field name="name">Sales of fixed assets taxed at a rate different from zero</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_402" model="account.report.line">
                                <field name="name">Gross value(402)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_402_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">402 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_412" model="account.report.line">
                                <field name="name">Net value(412)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_412_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">412 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_422" model="account.report.line">
                                <field name="name">Tax generated(422)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_422_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">422 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_7" model="account.report.line">
                        <field name="name">Local sales (excluding fixed assets) taxed at 5%</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_425" model="account.report.line">
                                <field name="name">Gross value(425)</field>
                                <field name="code">c425_104</field>
                            <field name="expression_ids">
                                    <record id="tax_report_line_104_425_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">425 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_435" model="account.report.line">
                                <field name="name">Net value(435)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_435_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">435 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_445" model="account.report.line">
                                <field name="name">Tax generated(445)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_445_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">445 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_10" model="account.report.line">
                        <field name="name">VAT generated on the difference between sales and credit notes at different rates (adjustment payable)</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_423" model="account.report.line">
                                <field name="name">Tax generated(423)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_423_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">423 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_12" model="account.report.line">
                        <field name="name">VAT generated on the difference between sales and credit notes at different rates (credit adjustment)</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_424" model="account.report.line">
                                <field name="name">Tax generated(424)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_424_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">424 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_14" model="account.report.line">
                        <field name="name">Local sales (excluding fixed assets) taxed at 0% rate that do not entitle to tax credits</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_403" model="account.report.line">
                                <field name="name">Gross value(403)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_403_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">403 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_413" model="account.report.line">
                                <field name="name">Net value(413)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_413_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">413 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_17" model="account.report.line">
                        <field name="name">Sales of fixed assets taxed at 0% rate that do not entitle to tax credits</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_404" model="account.report.line">
                                <field name="name">Gross value(404)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_404_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">404 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_414" model="account.report.line">
                                <field name="name">Net value(414)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_414_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">414 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_20" model="account.report.line">
                        <field name="name">Local sales (excluding fixed assets) taxed at 0% rate that entitle to tax credits</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_405" model="account.report.line">
                                <field name="name">Gross value(405)</field>
                                <field name="code">c405</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_405_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">405 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_415" model="account.report.line">
                                <field name="name">Net value(415)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_415_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">415 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_23" model="account.report.line">
                        <field name="name">Sales of fixed assets taxed at 0% rate that qualify for tax credits</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_406" model="account.report.line">
                                <field name="name">Gross value(406)</field>
                                <field name="code">c406</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_406_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">406 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_416" model="account.report.line">
                                <field name="name">Net value(416)</field>
                                <field name="code">c416</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_416_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">416 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_26" model="account.report.line">
                        <field name="name">Exports of goods</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_407" model="account.report.line">
                                <field name="name">Gross value(407)</field>
                                <field name="code">c407</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_407_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">407 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_417" model="account.report.line">
                                <field name="name">Net value(417)</field>
                                <field name="code">c417</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_417_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">417 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_29" model="account.report.line">
                        <field name="name">Exports of services and/or rights</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_408" model="account.report.line">
                                <field name="name">Gross value(408)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_408_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">408 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_418" model="account.report.line">
                                <field name="name">Net value(418)</field>
                                <field name="code">c418</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_418_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">418 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_32" model="account.report.line">
                        <field name="name">TOTAL SALES AND OTHER OPERATIONS</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_409" model="account.report.line">
                                <field name="name">Gross value(409)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_409_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">409 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_419" model="account.report.line">
                                <field name="name">Net value(419)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_419_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">419 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_429" model="account.report.line">
                                <field name="name">Tax generated(429)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_429_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">429 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_36" model="account.report.line">
                        <field name="name">Transfers not subject to or exempt from VAT</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_431" model="account.report.line">
                                <field name="name">Gross value(431)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_431_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">431 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_441" model="account.report.line">
                                <field name="name">Net value(441)</field>
                                <field name="code">c441</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_441_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">441 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_39" model="account.report.line">
                        <field name="name">Credit notes 0% rate to compensate next month</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_442" model="account.report.line">
                                <field name="name">Net value(442)</field>
                                <field name="code">c442</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_442_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">442 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_41" model="account.report.line">
                        <field name="name">Credit notes at non-zero rate to compensate next month</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_443" model="account.report.line">
                                <field name="name">Net value(443)</field>
                                <field name="code">c443</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_443_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">443 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_453" model="account.report.line">
                                <field name="name">Tax generated(453)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_453_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">453 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_44" model="account.report.line">
                        <field name="name">Reimbursement income as intermediary / values invoiced by transport operators (informative)</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_434" model="account.report.line">
                                <field name="name">Gross value(434)</field>
                                <field name="code">c434</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_434_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">434 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_444" model="account.report.line">
                                <field name="name">Net value(444)</field>
                                <field name="code">c444</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_444_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">444 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_454" model="account.report.line">
                                <field name="name">Tax generated(454)</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_454_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">454 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_parent_line_report_48" model="account.report.line">
                <field name="name">VAT LIQUIDATION IN THE MONTH</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_104_480" model="account.report.line">
                        <field name="name">Total transfers taxed at non-zero cash rate this month(480)</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_480_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">480 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_481" model="account.report.line">
                        <field name="name">Total transfers taxed at a rate different from zero to credit this month(481)</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_481_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">481 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_482" model="account.report.line">
                        <field name="name">Total Tax generated(482)</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_482_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">482 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_483" model="account.report.line">
                        <field name="name">Tax to be settled for the previous month(483)</field>
                        <field name="code">c483</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_483_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">483 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_484" model="account.report.line">
                        <field name="name">Tax to be paid this month(484)</field>
                        <field name="code">c484</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_484_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">484 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_495" model="account.report.line">
                        <field name="name">Tax to be settled in the next month(495)</field>
                        <field name="code">c495</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_495_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">495 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_499" model="account.report.line">
                        <field name="name">TOTAL TAX TO BE PAID THIS MONTH(499)</field>
                        <field name="code">c499</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_499_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">499 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_parent_line_report_56" model="account.report.line">
                <field name="name">SUMMARY OF PURCHASES AND PAYMENTS FOR THE REPORTING PERIOD</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_parent_line_report_57" model="account.report.line">
                        <field name="name">Acquisitions and payments (excluding fixed assets) taxed at a rate different from zero (with the right to tax credits)</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_500" model="account.report.line">
                                <field name="name">Gross value(500)</field>
                                <field name="code">c500</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_500_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">500 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_510" model="account.report.line">
                                <field name="name">Net value(510)</field>
                                <field name="code">c510</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_510_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">510 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_520" model="account.report.line">
                                <field name="name">Tax generated(520)</field>
                                <field name="code">c520</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_520_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">520 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_61" model="account.report.line">
                        <field name="name">Local acquisitions of fixed assets taxed at a rate different from zero (with the right to tax credits)</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_501" model="account.report.line">
                                <field name="name">Gross value(501)</field>
                                <field name="code">c501</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_501_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">501 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_511" model="account.report.line">
                                <field name="name">Net value(511)</field>
                                <field name="code">c511</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_511_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">511 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_521" model="account.report.line">
                                <field name="name">Tax generated(521)</field>
                                <field name="code">c521</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_521_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">521 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_62" model="account.report.line">
                        <field name="name">Acquisitions and local payments (excludes fixed assets) taxed at 5% (with the right to tax credits)</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_540" model="account.report.line">
                                <field name="name">Gross value(540)</field>
                                <field name="code">c540</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_540_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">540 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_550" model="account.report.line">
                                <field name="name">Net value(550)</field>
                                <field name="code">c550</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_550_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">550 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_560" model="account.report.line">
                                <field name="name">Tax generated(560)</field>
                                <field name="code">c560</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_560_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">560 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_65" model="account.report.line">
                        <field name="name">Other acquisitions and payments taxed at a rate different from zero (not entitled to tax credit)</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_502" model="account.report.line">
                                <field name="name">Gross value(502)</field>
                                <field name="code">c502</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_502_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">502 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_512" model="account.report.line">
                                <field name="name">Net value(512)</field>
                                <field name="code">c512</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_512_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">512 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_522" model="account.report.line">
                                <field name="name">Tax generated(522)</field>
                                <field name="code">c522</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_522_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">522 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_69" model="account.report.line">
                        <field name="name">Imports of services and/or duties taxed at rates different from zero</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_503" model="account.report.line">
                                <field name="name">Gross value(503)</field>
                                <field name="code">c503</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_503_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">503 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_513" model="account.report.line">
                                <field name="name">Net value(513)</field>
                                <field name="code">c513</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_513_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">513 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_523" model="account.report.line">
                                <field name="name">Tax generated(523)</field>
                                <field name="code">c523</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_523_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">523 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_73" model="account.report.line">
                        <field name="name">Imports of goods (excludes fixed assets) taxed at a rate different from zero</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_504" model="account.report.line">
                                <field name="name">Gross value(504)</field>
                                <field name="code">c504</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_504_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">504 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_514" model="account.report.line">
                                <field name="name">Net value(514)</field>
                                <field name="code">c514</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_514_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">514 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_524" model="account.report.line">
                                <field name="name">Tax generated(524)</field>
                                <field name="code">c524</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_524_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">524 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_77" model="account.report.line">
                        <field name="name">Imports of fixed assets taxed at rates different from zero</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_505" model="account.report.line">
                                <field name="name">Gross value(505)</field>
                                <field name="code">c505</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_505_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">505 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_515" model="account.report.line">
                                <field name="name">Net value(515)</field>
                                <field name="code">c515</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_515_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">515 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_525" model="account.report.line">
                                <field name="name">Tax generated(525)</field>
                                <field name="code">c525</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_525_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">525 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_81" model="account.report.line">
                        <field name="name">VAT generated on the difference between acquisitions and credit notes at different rates (positive adjustment to tax credit)</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_526" model="account.report.line">
                                <field name="name">Tax generated(526)</field>
                                <field name="code">c526</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_526_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">526 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_83" model="account.report.line">
                        <field name="name">VAT generated on the difference between acquisitions and credit notes at different rates (negative adjustment to tax credit)</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_527" model="account.report.line">
                                <field name="name">Tax generated(527)</field>
                                <field name="code">c527</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_527_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">527 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_85" model="account.report.line">
                        <field name="name">Imports of goods (including fixed assets) taxed at 0% rate</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_506" model="account.report.line">
                                <field name="name">Gross value(506)</field>
                                <field name="code">c506</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_506_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">506 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_516" model="account.report.line">
                                <field name="name">Net value(516)</field>
                                <field name="code">c516</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_516_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">516 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_88" model="account.report.line">
                        <field name="name">Acquisitions and payments (including fixed assets) taxed at 0% rate</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_507" model="account.report.line">
                                <field name="name">Gross value(507)</field>
                                <field name="code">c507</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_507_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">507 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_517" model="account.report.line">
                                <field name="name">Net value(517)</field>
                                <field name="code">c517</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_517_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">517 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_91" model="account.report.line">
                        <field name="name">Acquisitions from RISE taxpayers</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_508" model="account.report.line">
                                <field name="name">Gross value(508)</field>
                                <field name="code">c508</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_508_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">508 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_518" model="account.report.line">
                                <field name="name">Net value(518)</field>
                                <field name="code">c518</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_518_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">518 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_94" model="account.report.line">
                        <field name="name">TOTAL ACQUISITIONS AND PAYMENTS</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_509" model="account.report.line">
                                <field name="name">Gross value(509)</field>
                                <field name="code">c509</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_509_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">509 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_519" model="account.report.line">
                                <field name="name">Net value(519)</field>
                                <field name="code">c519</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_519_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">519 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_529" model="account.report.line">
                                <field name="name">Tax generated(529)</field>
                                <field name="code">c529</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_529_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">529 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_98" model="account.report.line">
                        <field name="name">Acquisitions not subject to VAT</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_531" model="account.report.line">
                                <field name="name">Gross value(531)</field>
                                <field name="code">c531</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_531_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">531 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_541" model="account.report.line">
                                <field name="name">Net value(541)</field>
                                <field name="code">c541</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_541_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">541 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_101" model="account.report.line">
                        <field name="name">Acquisitions exempt from VAT</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_532" model="account.report.line">
                                <field name="name">Gross value(532)</field>
                                <field name="code">c532</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_532_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">532 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_542" model="account.report.line">
                                <field name="name">Net value(542)</field>
                                <field name="code">c542</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_542_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">542 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_104" model="account.report.line">
                        <field name="name">Credit notes 0% rate to compensate next month</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_543" model="account.report.line">
                                <field name="name">Net value(543)</field>
                                <field name="code">c543</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_543_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">543 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_106" model="account.report.line">
                        <field name="name">Credit notes at non-zero rate to compensate next month</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_544" model="account.report.line">
                                <field name="name">Net value(544)</field>
                                <field name="code">c544</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_544_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">544 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_554" model="account.report.line">
                                <field name="name">Tax generated(554)</field>
                                <field name="code">c554</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_554_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">554 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_109" model="account.report.line">
                        <field name="name">Net payments for reimbursement as intermediary / values invoiced by partners to transport operators (informative)</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_535" model="account.report.line">
                                <field name="name">Gross value(535)</field>
                                <field name="code">c535</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_535_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">535 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_545" model="account.report.line">
                                <field name="name">Net value(545)</field>
                                <field name="code">c545</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_545_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">545 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_555" model="account.report.line">
                                <field name="name">Tax generated(555)</field>
                                <field name="code">c555</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_555_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">555 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_parent_line_report_113" model="account.report.line">
                <field name="name">TAX SUMMARY: AGENT FOR THE COLLECTION OF VALUE ADDED TAX</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_104_601" model="account.report.line">
                        <field name="name">Tax incurred (if the difference in fields 499-564 is greater than zero)(601)</field>
                        <field name="code">c601</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_601_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">601 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_602" model="account.report.line">
                        <field name="name">Tax credit applicable in this period (if the difference in fields 499-564 is less than zero)(602)</field>
                        <field name="code">c602</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_602_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">602 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_604" model="account.report.line">
                        <field name="name">(-) VAT offset for sales made in affected areas - Solidarity Law, restitution of tax credit in resolutions(604)</field>
                        <field name="code">c604</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_604_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">604 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_117" model="account.report.line">
                        <field name="name">(-) Tax credit balance from previous month</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_605" model="account.report.line">
                                <field name="name">For acquisitions and imports (transfer field 615 from the previous period's declaration)(605)</field>
                                <field name="code">c605</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_605_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">605 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_606" model="account.report.line">
                                <field name="name">For VAT withholdings at source that have been made (transfer the field 617 of the previous period's return)(606)</field>
                                <field name="code">c606</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_606_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">606 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_608" model="account.report.line">
                                <field name="name">VAT offset for sales made in affected areas - Solidarity Law (transfer field 619 of the previous period's return)(608)</field>
                                <field name="code">c608</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_608_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">608 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_609" model="account.report.line">
                        <field name="name">(-) VAT withholdings at source made during this period(609)</field>
                        <field name="code">c609</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_609_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">609 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_610" model="account.report.line">
                        <field name="name">(+) Adjustment for VAT refunded or deducted for purchases made by electronic means(610)</field>
                        <field name="code">c610</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_610_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">610 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_612" model="account.report.line">
                        <field name="name">(+) Adjustment for VAT refunds and VAT rejections (VAT refunds), VAT adjustment for control processes and others (import acquisitions), attributable to tax credit(612)</field>
                        <field name="code">c612</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_612_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">612 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_613" model="account.report.line">
                        <field name="name">(+) Adjustment for VAT refunded and rejected VAT, VAT adjustment for control processes and others (for VAT withholdings at source), attributable to tax credit(613)</field>
                        <field name="code">c613</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_613_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">613 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_614" model="account.report.line">
                        <field name="name">(+) Adjustment for VAT refunded by other public sector institutions attributable to tax credit in the month(614)</field>
                        <field name="code">c614</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_614_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">614 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_126" model="account.report.line">
                        <field name="name">Tax credit balance for next month</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_615" model="account.report.line">
                                <field name="name">Acquisitions and Imports(615)</field>
                                <field name="code">c615</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_615_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">615 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_617" model="account.report.line">
                                <field name="name">For VAT withholdings at the source that have been made(617)</field>
                                <field name="code">c617</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_617_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">617 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_619" model="account.report.line">
                                <field name="name">VAT offset for sales made in affected areas - Solidarity Law, restitution of tax credit in resolutions(619)</field>
                                <field name="code">c619</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_619_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">619 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_620" model="account.report.line">
                        <field name="name">SUBTOTAL PAYABLE If (601-602-603-604-605-606-607-608-609+610+611+612+613+614) &gt; 0(620)</field>
                        <field name="code">c620</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_620_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">620 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_621" model="account.report.line">
                        <field name="name">Presumptive VAT on gaming halls (mechanical bingo) and other games of chance (applicable for years prior to 2013)(621)</field>
                        <field name="code">c621</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_621_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">621 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_699" model="account.report.line">
                        <field name="name">TOTAL TAX TO BE PAID FOR COLLECTION(699)</field>
                        <field name="code">c699</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_699_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">699 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_parent_line_report_133" model="account.report.line">
                <field name="name">TAX ON FOREIGN EXCHANGE OUTFLOWS FOR PURPOSES OF REFUND TO REGULAR EXPORTERS OF GOODS</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_parent_line_report_134" model="account.report.line">
                        <field name="name">Imports of raw materials, inputs and capital goods that are incorporated in the production processes of goods to be exported</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_700" model="account.report.line">
                                <field name="name">Value(700)</field>
                                <field name="code">c700</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_700_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">700 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104_701" model="account.report.line">
                                <field name="name">ISD paid(701)</field>
                                <field name="code">c701</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_701_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">701 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_parent_line_report_137" model="account.report.line">
                <field name="name">VALUE ADDED TAX WITHHOLDING AGENT</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_104_721" model="account.report.line">
                        <field name="name">10% withholding(721)</field>
                        <field name="code">c721</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_721_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">721 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_723" model="account.report.line">
                        <field name="name">20% withholding(723)</field>
                        <field name="code">c723</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_723_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">723 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_725" model="account.report.line">
                        <field name="name">30% withholding(725)</field>
                        <field name="code">c725</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_725_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">725 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_727" model="account.report.line">
                        <field name="name">50% withholding(727)</field>
                        <field name="code">c727</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_727_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">727 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_729" model="account.report.line">
                        <field name="name">70% withholding(729)</field>
                        <field name="code">c729</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_729_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">729 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_104_731" model="account.report.line">
                        <field name="name">100% withholding(731)</field>
                        <field name="code">c731</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_104_731_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">731 (Reporte 104)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_144" model="account.report.line">
                        <field name="name">TOTAL TAX WITHHELD</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_799" model="account.report.line">
                                <field name="name">(799)</field>
                                <field name="code">c799</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_799_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">799 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_146" model="account.report.line">
                        <field name="name">Provisional VAT refund by offsetting with withholdings made</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_800" model="account.report.line">
                                <field name="name">(800)</field>
                                <field name="code">c800</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_800_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">800 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_148" model="account.report.line">
                        <field name="name">TOTAL WITHHOLDING TAX PAYABLE</field>
                        <field name="children_ids">
                            <record id="tax_report_line_104_801" model="account.report.line">
                                <field name="name">(801)</field>
                                <field name="code">c801</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_801_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">801 (Reporte 104)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
    <record id="tax_report_103" model="account.report">
        <field name="name">103</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ec"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_103_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_parent_line_report_150" model="account.report.line">
                <field name="name">FOR PAYMENTS MADE TO RESIDENTS AND PERMANENT ESTABLISHMENTS</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_parent_line_report_151" model="account.report.line">
                        <field name="name">In a dependency relationship that exceeds or does not exceed the tax-deductible basis</field>
                        <field name="children_ids">
                            <record id="tax_report_line_103_302" model="account.report.line">
                                <field name="name">Taxable income(302)</field>
                                <field name="code">c302</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_302_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">302 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_103_352" model="account.report.line">
                                <field name="name">Value withheld(352)</field>
                                <field name="code">c352</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_352_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">352 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_154" model="account.report.line">
                                <field name="name">Services</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_parent_line_report_155" model="account.report.line">
                                        <field name="name">Professional fees</field>
                                        <field name="children_ids">
                                            <record id="tax_report_line_103_303" model="account.report.line">
                                                <field name="name">Taxable income(303)</field>
                                                <field name="code">c303</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_303_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">303 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_line_103_353" model="account.report.line">
                                                <field name="name">Value withheld(353)</field>
                                                <field name="code">c353</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_353_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">353 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                     <record id="tax_report_line_parent_line_report_156" model="account.report.line">
                                        <field name="name">Servicios profesionales prestados por sociedades residentes</field>
                                        <field name="children_ids">
                                            <record id="tax_report_line_103_3030" model="account.report.line">
                                                <field name="name">Base imponible(3030)</field>
                                                <field name="code">c3030</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_3030_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">3030 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_line_103_3530" model="account.report.line">
                                                <field name="name">Valor retenido(3530)</field>
                                                <field name="code">c3530</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_3530_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">3530 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_parent_line_report_158" model="account.report.line">
                                        <field name="name">Intellect predominates</field>
                                        <field name="children_ids">
                                            <record id="tax_report_line_103_304" model="account.report.line">
                                                <field name="name">Taxable income(304)</field>
                                                <field name="code">c304</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_304_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">304 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_line_103_354" model="account.report.line">
                                                <field name="name">Value withheld(354)</field>
                                                <field name="code">c354</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_354_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">354 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_parent_line_report_161" model="account.report.line">
                                        <field name="name">Labor predominates</field>
                                        <field name="children_ids">
                                            <record id="tax_report_line_103_307" model="account.report.line">
                                                <field name="name">Taxable income(307)</field>
                                                <field name="code">c307</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_307_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">307 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_line_103_357" model="account.report.line">
                                                <field name="name">Value withheld(357)</field>
                                                <field name="code">c357</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_357_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">357 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_parent_line_report_164" model="account.report.line">
                                        <field name="name">Use or exploitation of image or reputation</field>
                                        <field name="children_ids">
                                            <record id="tax_report_line_103_308" model="account.report.line">
                                                <field name="name">Taxable income(308)</field>
                                                <field name="code">c308</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_308_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">308 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_line_103_358" model="account.report.line">
                                                <field name="name">Value withheld(358)</field>
                                                <field name="code">c358</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_358_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">358 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_parent_line_report_167" model="account.report.line">
                                        <field name="name">Advertising and communication</field>
                                        <field name="children_ids">
                                            <record id="tax_report_line_103_309" model="account.report.line">
                                                <field name="name">Taxable income(309)</field>
                                                <field name="code">c309</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_309_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">309 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_line_103_359" model="account.report.line">
                                                <field name="name">Value withheld(359)</field>
                                                <field name="code">c359</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_359_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">359 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_parent_line_report_170" model="account.report.line">
                                        <field name="name">Private passenger transportation or public or private freight service</field>
                                        <field name="children_ids">
                                            <record id="tax_report_line_103_310" model="account.report.line">
                                                <field name="name">Taxable income(310)</field>
                                                <field name="code">c310</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_310_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">310 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_line_103_360" model="account.report.line">
                                                <field name="name">Value withheld(360)</field>
                                                <field name="code">c360</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_360_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">360 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_173" model="account.report.line">
                                <field name="name">Through purchase settlements (cultural level or rusticity)</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_311" model="account.report.line">
                                        <field name="name">Taxable income(311)</field>
                                        <field name="code">c311</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_311_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">311 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_361" model="account.report.line">
                                        <field name="name">Value withheld(361)</field>
                                        <field name="code">c361</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_361_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">361 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_176" model="account.report.line">
                                <field name="name">Transfer of tangible personal property</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_312" model="account.report.line">
                                        <field name="name">Taxable income(312)</field>
                                        <field name="code">c312</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_312_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">312 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_362" model="account.report.line">
                                        <field name="name">Value withheld(362)</field>
                                        <field name="code">c362</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_362_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">362 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_177" model="account.report.line">
                                <field name="name">Compras al productor: de bienes de origen agrícola, avícola, pecuario, apícola, cunícola, bioacuático, forestal y carnes en estado natural y los descritos en el art.27.1 de LRTI.</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_3120" model="account.report.line">
                                        <field name="name">Base imponible(3120)</field>
                                        <field name="code">c3120</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_3120_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">3120 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_3620" model="account.report.line">
                                        <field name="name">Value withheld(3620)</field>
                                        <field name="code">c3620</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_3620_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">3620 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_178" model="account.report.line">
                                <field name="name">Compras al Comercializador: de Bienes de Origen Bioacuático, Forestal y los Descritos el Art.27.1 de LRTI</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_3121" model="account.report.line">
                                        <field name="name">Base imponible(3121)</field>
                                        <field name="code">c3121</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_3121_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">3121 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_3621" model="account.report.line">
                                        <field name="name">Valor retenido(3621)</field>
                                        <field name="code">c3621</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_3621_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">3621 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_182" model="account.report.line">
                                <field name="name">For royalties, copyrights, trademarks, patents and similar rights</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_314" model="account.report.line">
                                        <field name="name">Taxable income(314)</field>
                                        <field name="code">c314</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_314_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">314 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_364" model="account.report.line">
                                        <field name="name">Value withheld(364)</field>
                                        <field name="code">c364</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_364_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">364 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_183" model="account.report.line">
                                <field name="name">Comisiones pagadas a sociedades, nacionales o extranjeras residentes en el Ecuador y establecimientos permanentes domiciliados en el país</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_3140" model="account.report.line">
                                        <field name="name">Base imponible(3140)</field>
                                        <field name="code">c3140</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_3140_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">3140 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_3640" model="account.report.line">
                                        <field name="name">Valor retenido(3640)</field>
                                        <field name="code">c3640</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_3640_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">3640 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_185" model="account.report.line">
                                <field name="name">Leasing</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_parent_line_report_186" model="account.report.line">
                                        <field name="name">Commercial</field>
                                        <field name="children_ids">
                                            <record id="tax_report_line_103_319" model="account.report.line">
                                                <field name="name">Taxable income(319)</field>
                                                <field name="code">c319</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_319_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">319 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_line_103_369" model="account.report.line">
                                                <field name="name">Value withheld(369)</field>
                                                <field name="code">c369</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_369_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">369 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_parent_line_report_189" model="account.report.line">
                                        <field name="name">Real estate</field>
                                        <field name="children_ids">
                                            <record id="tax_report_line_103_320" model="account.report.line">
                                                <field name="name">Taxable income(320)</field>
                                                <field name="code">c320</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_320_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">320 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_line_103_370" model="account.report.line">
                                                <field name="name">Value withheld(370)</field>
                                                <field name="code">c370</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_line_103_370_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">370 (Reporte 103)</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_192" model="account.report.line">
                                <field name="name">Insurance and reinsurance (premiums and cessions)</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_322" model="account.report.line">
                                        <field name="name">Taxable income(322)</field>
                                        <field name="code">c322</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_322_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">322 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_372" model="account.report.line">
                                        <field name="name">Value withheld(372)</field>
                                        <field name="code">c372</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_372_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">372 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_195" model="account.report.line">
                                <field name="name">Financial returns</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_323" model="account.report.line">
                                        <field name="name">Taxable income(323)</field>
                                        <field name="code">c323</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_323_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">323 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_373" model="account.report.line">
                                        <field name="name">Value withheld(373)</field>
                                        <field name="code">c373</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_373_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">373 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_198" model="account.report.line">
                                <field name="name">Financial returns between institutions of the financial system and popular and solidarity economy entities</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_324" model="account.report.line">
                                        <field name="name">Taxable income(324)</field>
                                        <field name="code">c324</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_324_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">324 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_374" model="account.report.line">
                                        <field name="name">Value withheld(374)</field>
                                        <field name="code">c374</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_374_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">374 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_201" model="account.report.line">
                                <field name="name">Dividend advance</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_325" model="account.report.line">
                                        <field name="name">Taxable income(325)</field>
                                        <field name="code">c325</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_325_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">325 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_375" model="account.report.line">
                                        <field name="name">Value withheld(375)</field>
                                        <field name="code">c375</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_375_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">375 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_204" model="account.report.line">
                                <field name="name">Dividends distributed that correspond to the single income tax established in art. 27 of the LRTI</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_326" model="account.report.line">
                                        <field name="name">Taxable income(326)</field>
                                        <field name="code">c326</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_326_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">326 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_376" model="account.report.line">
                                        <field name="name">Value withheld(376)</field>
                                        <field name="code">c376</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_376_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">376 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_207" model="account.report.line">
                                <field name="name">Dividends distributed to resident individuals</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_327" model="account.report.line">
                                        <field name="name">Taxable income(327)</field>
                                        <field name="code">c327</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_327_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">327 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_377" model="account.report.line">
                                        <field name="name">Value withheld(377)</field>
                                        <field name="code">c377</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_377_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">377 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_210" model="account.report.line">
                                <field name="name">Dividends distributed to resident companies</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_328" model="account.report.line">
                                        <field name="name">Taxable income(328)</field>
                                        <field name="code">c328</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_328_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">328 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_378" model="account.report.line">
                                        <field name="name">Value withheld(378)</field>
                                        <field name="code">c378</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_378_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">378 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_213" model="account.report.line">
                                <field name="name">Dividends distributed to resident trusts</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_329" model="account.report.line">
                                        <field name="name">Taxable income(329)</field>
                                        <field name="code">c329</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_329_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">329 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_379" model="account.report.line">
                                        <field name="name">Value withheld(379)</field>
                                        <field name="code">c379</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_379_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">379 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_216" model="account.report.line">
                                <field name="name">Stock dividends (capitalization of earnings)</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_331" model="account.report.line">
                                        <field name="name">Taxable income(331)</field>
                                        <field name="code">c331</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_331_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">331 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_218" model="account.report.line">
                                <field name="name">Payments for goods and services not subject to withholding tax</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_332" model="account.report.line">
                                        <field name="name">Taxable income(332)</field>
                                        <field name="code">c332</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_332_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">332 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_220" model="account.report.line">
                                <field name="name">Disposal of capital rights and other rights listed on the Ecuadorian stock exchange</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_333" model="account.report.line">
                                        <field name="name">Taxable income(333)</field>
                                        <field name="code">c333</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_333_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">333 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_383" model="account.report.line">
                                        <field name="name">Value withheld(383)</field>
                                        <field name="code">c383</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_383_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">383 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_223" model="account.report.line">
                                <field name="name">Disposal of rights representing capital and other rights not listed on the Ecuadorian stock exchange</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_334" model="account.report.line">
                                        <field name="name">Taxable income(334)</field>
                                        <field name="code">c334</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_334_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">334 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_384" model="account.report.line">
                                        <field name="name">Value withheld(384)</field>
                                        <field name="code">c384</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_384_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">384 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_226" model="account.report.line">
                                <field name="name">Lotteries, raffles, bets and the like</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_335" model="account.report.line">
                                        <field name="name">Taxable income(335)</field>
                                        <field name="code">c335</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_335_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">335 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_385" model="account.report.line">
                                        <field name="name">Value withheld(385)</field>
                                        <field name="code">c385</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_385_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">385 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_229" model="account.report.line">
                        <field name="name">Fuel sales</field>
                        <field name="children_ids">
                            <record id="tax_report_line_parent_line_report_230" model="account.report.line">
                                <field name="name">To marketers</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_336" model="account.report.line">
                                        <field name="name">Taxable income(336)</field>
                                        <field name="code">c336</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_336_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">336 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_386" model="account.report.line">
                                        <field name="name">Value withheld(386)</field>
                                        <field name="code">c386</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_386_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">386 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_233" model="account.report.line">
                                <field name="name">To distributors</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_337" model="account.report.line">
                                        <field name="name">Taxable income(337)</field>
                                        <field name="code">c337</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_337_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">337 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_387" model="account.report.line">
                                        <field name="name">Value withheld(387)</field>
                                        <field name="code">c387</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_387_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">387 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_236" model="account.report.line">
                        <field name="name">Local production and sale of bananas, whether or not produced by the same taxpayer</field>
                        <field name="children_ids">
                            <record id="tax_report_line_103_3380" model="account.report.line">
                                <field name="name">Taxable income(3380)</field>
                                <field name="code">c3380</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_3380_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3380 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_103_3880" model="account.report.line">
                                <field name="name">Value withheld(3880)</field>
                                <field name="code">c3880</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_3880_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3880 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_239" model="account.report.line">
                        <field name="name">Single banana export tax</field>
                        <field name="children_ids">
                            <record id="tax_report_line_103_3400" model="account.report.line">
                                <field name="name">Taxable income(3400)</field>
                                <field name="code">c3400</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_3400_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3400 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_103_3900" model="account.report.line">
                                <field name="name">Value withheld(3900)</field>
                                <field name="code">c3900</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_3900_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3900 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_242" model="account.report.line">
                        <field name="name">Other withholdings</field>
                        <field name="children_ids">
                            <record id="tax_report_line_parent_line_report_243" model="account.report.line">
                                <field name="name">Applicable 1%</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_343" model="account.report.line">
                                        <field name="name">Taxable income(343)</field>
                                        <field name="code">c343</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_343_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">343 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_393" model="account.report.line">
                                        <field name="name">Value withheld(393)</field>
                                        <field name="code">c393</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_393_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">393 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_246" model="account.report.line">
                                <field name="name">Applicable at 2%</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_344" model="account.report.line">
                                        <field name="name">Taxable income(344)</field>
                                        <field name="code">c344</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_344_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">344 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_394" model="account.report.line">
                                        <field name="name">Value withheld(394)</field>
                                        <field name="code">c394</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_394_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">394 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_249" model="account.report.line">
                                <field name="name">Applicable at 2,75%</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_3440" model="account.report.line">
                                        <field name="name">Taxable income(3440)</field>
                                        <field name="code">c3440</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_3440_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">3440 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_3940" model="account.report.line">
                                        <field name="name">Value withheld(3940)</field>
                                        <field name="code">c3940</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_3940_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">3940 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_252" model="account.report.line">
                                <field name="name">Applicable at 8%</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_345" model="account.report.line">
                                        <field name="name">Taxable income(345)</field>
                                        <field name="code">c345</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_345_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">345 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_395" model="account.report.line">
                                        <field name="name">Value withheld(395)</field>
                                        <field name="code">c395</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_395_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">395 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_255" model="account.report.line">
                                <field name="name">Applicable to other percentages (including Micro-enterprise regime)</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_346" model="account.report.line">
                                        <field name="name">Taxable income(346)</field>
                                        <field name="code">c346</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_346_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">346 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_396" model="account.report.line">
                                        <field name="name">Value withheld(396)</field>
                                        <field name="code">c396</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_396_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">396 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_258" model="account.report.line">
                        <field name="name">Single tax on income from agricultural activities at the production / local marketing or export stage</field>
                        <field name="children_ids">
                            <record id="tax_report_line_103_348" model="account.report.line">
                                <field name="name">Taxable income(348)</field>
                                <field name="code">c348</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_348_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">348 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_103_398" model="account.report.line">
                                <field name="name">Value withheld(398)</field>
                                <field name="code">c398</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_398_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">398 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_261" model="account.report.line">
                        <field name="name">Other self-remittances</field>
                        <field name="children_ids">
                            <record id="tax_report_line_103_350" model="account.report.line">
                                <field name="name">Taxable income(350)</field>
                                <field name="code">c350</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_350_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">350 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_103_400" model="account.report.line">
                                <field name="name">Value withheld(400)</field>
                                <field name="code">c400</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_400_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">400 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_264" model="account.report.line">
                        <field name="name">SUBTOTAL OPERATIONS CARRIED OUT IN THE COUNTRY</field>
                        <field name="children_ids">
                            <record id="tax_report_line_103_349" model="account.report.line">
                                <field name="name">Taxable income(349)</field>
                                <field name="code">c349</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_349_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">349 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_103_399" model="account.report.line">
                                <field name="name">Value withheld(399)</field>
                                <field name="code">c399</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_399_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">399 (Reporte 103)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_parent_line_report_267" model="account.report.line">
                <field name="name">FOR PAYMENTS TO NON-RESIDENTS</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_parent_line_report_268" model="account.report.line">
                        <field name="name">With double taxation agreement</field>
                        <field name="children_ids">
                            <record id="tax_report_line_parent_line_report_269" model="account.report.line">
                                <field name="name">Interest on supplier financing</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_402" model="account.report.line">
                                        <field name="name">Taxable income(402)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_402_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">402 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_452" model="account.report.line">
                                        <field name="name">Value withheld(452)</field>
                                        <field name="code">c452</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_452_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">452 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_272" model="account.report.line">
                                <field name="name">Interest on loans</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_403" model="account.report.line">
                                        <field name="name">Taxable income(403)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_403_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">403 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_453" model="account.report.line">
                                        <field name="name">Value withheld(453)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_453_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">453 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_275" model="account.report.line">
                                <field name="name">Dividend advance</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_404" model="account.report.line">
                                        <field name="name">Taxable income(404)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_404_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">404 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_454" model="account.report.line">
                                        <field name="name">Value withheld(454)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_454_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">454 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_278" model="account.report.line">
                                <field name="name">Dividends with no beneficial owner, natural person resident in Ecuador</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_4050" model="account.report.line">
                                        <field name="name">Taxable income(4050)</field>
                                        <field name="code">c4050</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4050_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4050 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_4550" model="account.report.line">
                                        <field name="name">Value withheld(4550)</field>
                                        <field name="code">c4550</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4550_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4550 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_281" model="account.report.line">
                                <field name="name">Dividends with beneficial ownership by an individual resident in Ecuador</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_4060" model="account.report.line">
                                        <field name="name">Taxable income(4060)</field>
                                        <field name="code">c4060</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4060_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4060 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_4560" model="account.report.line">
                                        <field name="name">Value withheld(4560)</field>
                                        <field name="code">c4560</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4560_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4560 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_284" model="account.report.line">
                                <field name="name">Dividends in breach of the duty to disclose the company's shareholder structure</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_4070" model="account.report.line">
                                        <field name="name">Taxable income(4070)</field>
                                        <field name="code">c4070</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4070_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4070 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_4570" model="account.report.line">
                                        <field name="name">Value withheld(4570)</field>
                                        <field name="code">c4570</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4570_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4570 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_287" model="account.report.line">
                                <field name="name">Disposal of rights representing capital and other rights</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_408" model="account.report.line">
                                        <field name="name">Taxable income(408)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_408_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">408 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_458" model="account.report.line">
                                        <field name="name">Value withheld(458)</field>
                                        <field name="code">c458</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_458_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">458 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_290" model="account.report.line">
                                <field name="name">Insurance and reinsurance (premiums and cessions)</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_409" model="account.report.line">
                                        <field name="name">Taxable income(409)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_409_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">409 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_459" model="account.report.line">
                                        <field name="name">Value withheld(459)</field>
                                        <field name="code">c459</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_459_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">459 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_293" model="account.report.line">
                                <field name="name">Technical, administrative or consulting services and royalties</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_410" model="account.report.line">
                                        <field name="name">Taxable income(410)</field>
                                        <field name="code">c410</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_410_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">410 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_460" model="account.report.line">
                                        <field name="name">Value withheld(460)</field>
                                        <field name="code">c460</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_460_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">460 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_296" model="account.report.line">
                                <field name="name">Other items of taxable income</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_411" model="account.report.line">
                                        <field name="name">Taxable income(411)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_411_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">411 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_461" model="account.report.line">
                                        <field name="name">Value withheld(461)</field>
                                        <field name="code">c461</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_461_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">461 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_299" model="account.report.line">
                                <field name="name">Other foreign payments not subject to withholding</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_412" model="account.report.line">
                                        <field name="name">Taxable income(412)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_412_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">412 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_301" model="account.report.line">
                        <field name="name">No double taxation agreement</field>
                        <field name="children_ids">
                            <record id="tax_report_line_parent_line_report_302" model="account.report.line">
                                <field name="name">Interest on supplier financing</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_413" model="account.report.line">
                                        <field name="name">Taxable income(413)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_413_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">413 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_463" model="account.report.line">
                                        <field name="name">Value withheld(463)</field>
                                        <field name="code">c463</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_463_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">463 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_305" model="account.report.line">
                                <field name="name">Interest on loans</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_414" model="account.report.line">
                                        <field name="name">Taxable income(414)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_414_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">414 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_464" model="account.report.line">
                                        <field name="name">Value withheld(464)</field>
                                        <field name="code">c464</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_464_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">464 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_308" model="account.report.line">
                                <field name="name">Dividend advance</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_415" model="account.report.line">
                                        <field name="name">Taxable income(415)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_415_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">415 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_465" model="account.report.line">
                                        <field name="name">Value withheld(465)</field>
                                        <field name="code">c465</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_465_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">465 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_311" model="account.report.line">
                                <field name="name">Dividends with no beneficial owner, natural person resident in Ecuador</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_4160" model="account.report.line">
                                        <field name="name">Taxable income(4160)</field>
                                        <field name="code">c4160</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4160_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4160 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_4660" model="account.report.line">
                                        <field name="name">Value withheld(4660)</field>
                                        <field name="code">c4660</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4660_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4660 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_314" model="account.report.line">
                                <field name="name">Dividends with beneficial ownership by an individual resident in Ecuador</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_4170" model="account.report.line">
                                        <field name="name">Taxable income(4170)</field>
                                        <field name="code">c4170</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4170_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4170 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_4670" model="account.report.line">
                                        <field name="name">Value withheld(4670)</field>
                                        <field name="code">c4670</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4670_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4670 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_317" model="account.report.line">
                                <field name="name">Dividends in breach of the duty to disclose the company's shareholder structure</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_4180" model="account.report.line">
                                        <field name="name">Taxable income(4180)</field>
                                        <field name="code">c4180</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4180_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4180 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_4680" model="account.report.line">
                                        <field name="name">Value withheld(4680)</field>
                                        <field name="code">c4680</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4680_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4680 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_320" model="account.report.line">
                                <field name="name">Disposal of rights representing capital and other rights</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_419" model="account.report.line">
                                        <field name="name">Taxable income(419)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_419_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">419 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_469" model="account.report.line">
                                        <field name="name">Value withheld(469)</field>
                                        <field name="code">c469</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_469_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">469 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_323" model="account.report.line">
                                <field name="name">Insurance and reinsurance (premiums and cessions)</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_420" model="account.report.line">
                                        <field name="name">Taxable income(420)</field>
                                        <field name="code">c420</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_420_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">420 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_470" model="account.report.line">
                                        <field name="name">Value withheld(470)</field>
                                        <field name="code">c470</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_470_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">470 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_326" model="account.report.line">
                                <field name="name">Technical, administrative or consulting services and royalties</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_421" model="account.report.line">
                                        <field name="name">Taxable income(421)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_421_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">421 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_471" model="account.report.line">
                                        <field name="name">Value withheld(471)</field>
                                        <field name="code">c471</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_471_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">471 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_329" model="account.report.line">
                                <field name="name">Other items of taxable income</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_422" model="account.report.line">
                                        <field name="name">Taxable income(422)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_422_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">422 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_472" model="account.report.line">
                                        <field name="name">Value withheld(472)</field>
                                        <field name="code">c472</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_472_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">472 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_332" model="account.report.line">
                                <field name="name">Other foreign payments not subject to withholding</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_423" model="account.report.line">
                                        <field name="name">Taxable income(423)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_423_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">423 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_parent_line_report_334" model="account.report.line">
                        <field name="name">In tax havens or preferential tax regimes</field>
                        <field name="children_ids">
                            <record id="tax_report_line_parent_line_report_335" model="account.report.line">
                                <field name="name">Interests</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_424" model="account.report.line">
                                        <field name="name">Taxable income(424)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_424_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">424 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_474" model="account.report.line">
                                        <field name="name">Value withheld(474)</field>
                                        <field name="code">c474</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_474_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">474 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_338" model="account.report.line">
                                <field name="name">Dividend advance</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_425" model="account.report.line">
                                        <field name="name">Taxable income(425)</field>
                                        <field name="code">c425</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_425_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">425 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_475" model="account.report.line">
                                        <field name="name">Value withheld(475)</field>
                                        <field name="code">c475</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_475_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">475 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_341" model="account.report.line">
                                <field name="name">Dividends with no beneficial owner, natural person resident in Ecuador</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_4260" model="account.report.line">
                                        <field name="name">Taxable income(4260)</field>
                                        <field name="code">c4260</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4260_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4260 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_4760" model="account.report.line">
                                        <field name="name">Value withheld(4760)</field>
                                        <field name="code">c4760</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4760_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4760 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_344" model="account.report.line">
                                <field name="name">Dividends with beneficial ownership by an individual resident in Ecuador</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_4270" model="account.report.line">
                                        <field name="name">Taxable income(4270)</field>
                                        <field name="code">c4270</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4270_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4270 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_4770" model="account.report.line">
                                        <field name="name">Value withheld(4770)</field>
                                        <field name="code">c4770</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4770_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4770 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_347" model="account.report.line">
                                <field name="name">Dividends in breach of the duty to disclose the company's shareholder structure</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_4280" model="account.report.line">
                                        <field name="name">Taxable income(4280)</field>
                                        <field name="code">c4280</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4280_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4280 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_4780" model="account.report.line">
                                        <field name="name">Value withheld(4780)</field>
                                        <field name="code">c4780</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_4780_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">4780 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_350" model="account.report.line">
                                <field name="name">Disposal of rights representing capital and other rights</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_429" model="account.report.line">
                                        <field name="name">Taxable income(429)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_429_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">429 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_479" model="account.report.line">
                                        <field name="name">Value withheld(479)</field>
                                        <field name="code">c479</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_479_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">479 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_353" model="account.report.line">
                                <field name="name">Insurance and reinsurance (premiums and cessions)</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_430" model="account.report.line">
                                        <field name="name">Taxable income(430)</field>
                                        <field name="code">c430</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_430_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">430 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_480" model="account.report.line">
                                        <field name="name">Value withheld(480)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_480_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">480 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_356" model="account.report.line">
                                <field name="name">Technical, administrative or consulting services and royalties</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_431" model="account.report.line">
                                        <field name="name">Taxable income(431)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_431_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">431 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_481" model="account.report.line">
                                        <field name="name">Value withheld(481)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_481_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">481 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_359" model="account.report.line">
                                <field name="name">Other items of taxable income</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_432" model="account.report.line">
                                        <field name="name">Taxable income(432)</field>
                                        <field name="code">c432</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_432_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">432 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_103_482" model="account.report.line">
                                        <field name="name">Value withheld(482)</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_482_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">482 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_parent_line_report_362" model="account.report.line">
                                <field name="name">Other foreign payments not subject to withholding</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_103_433" model="account.report.line">
                                        <field name="name">Taxable income(433)</field>
                                        <field name="code">c433</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_103_433_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">433 (Reporte 103)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_parent_line_report_364" model="account.report.line">
                <field name="name">SUBTOTAL FOREIGN OPERATIONS</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_103_487" model="account.report.line">
                        <field name="name">Taxable income(487)</field>
                        <field name="code">c487</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_103_487_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">487 (Reporte 103)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_103_498" model="account.report.line">
                        <field name="name">Value withheld(498)</field>
                        <field name="code">c498</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_103_498_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">498 (Reporte 103)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_ec.sri.payment.csv

```csv
id,code,name,active
P1,01,No use of the financial system,1
P15,15,Offset of Debts,1
P16,16,Debit Card,1
P17,17,Electronic Cash,0
P18,18,Prepaid Card,0
P19,19,Credit Card,1
P20,20,Others with use of the financial system,1
P21,21,Endorsement of Securities,0

```

## File: data\l10n_latam.document.type.csv

```csv
"id","sequence","code","name","report_name","internal_type","doc_code_prefix","country_id/id","active","l10n_ec_check_format"
"ec_dt_01",1,"01","Invoice","Invoice","invoice","Fact","base.ec",1,1
"ec_dt_02",2,"02","Note or bill of sale","Note or bill of sale","invoice","Not","base.ec",1,1
"ec_dt_03",3,"03","Settlement of purchase of goods or provision of services","Settlement of purchase of goods or provision of services","purchase_liquidation","LiqCo","base.ec",1,1
"ec_dt_04",4,"04","Credit Note","Credit Note","credit_note","NotCr","base.ec",1,1
"ec_dt_05",5,"05","Debit Note","Debit Note","debit_note","NotDb","base.ec",1,1
"ec_dt_08",6,"08","Tickets or tickets to public shows","Tickets or tickets to public shows","invoice","Ent","base.ec",1,0
"ec_dt_09",7,"09","Tickets or vouchers issued by cash register machines","Tickets or vouchers issued by cash register machines","invoice","Tiq","base.ec",1,0
"ec_dt_11",8,"11","Tickets issued by airline companies","Tickets issued by airline companies","invoice","Pas","base.ec",1,0
"ec_dt_12",9,"12","Documents issued by financial institutions","Documents issued by financial institutions","invoice","Fin","base.ec",1,0
"ec_dt_15",10,"15","Sales receipts issued abroad","Sales receipts issued abroad","invoice","FacExt","base.ec",1,0
"ec_dt_16",11,"16","Single Export Form (FUE) or Single Customs Declaration (DAU) or Andean Value Declaration (DAV)","Single Export Form (FUE) or Single Customs Declaration (DAU) or Andean Value Declaration (DAV)","invoice","DAU","base.ec",1,0
"ec_dt_18",12,"18","Sales invoice","Sales invoice","invoice","Fact","base.ec",0,1
"ec_dt_19",13,"19","Dues or Contribution Payment Receipts","Dues or Contribution Payment Receipts","invoice","Aprt","base.ec",1,0
"ec_dt_20",14,"20","Documents for Administrative Services issued by State institutions","Documents for Administrative Services issued by State institutions","invoice","Gov","base.ec",1,0
"ec_dt_21",15,"21","Airway Bill","Airway Bill","invoice","Aér","base.ec",1,0
"ec_dt_22",16,"22","RECAP","RECAP","invoice","RECAP","base.ec",0,0
"ec_dt_23",17,"23","TC Credit Note","TC Credit Note","credit_note","NotCrTC","base.ec",0,1
"ec_dt_24",18,"24","TC Debit Note","TC Debit Note","debit_note","NotDbTC","base.ec",0,1
"ec_dt_41",19,"41","Sales receipt issued for reimbursement","Sales receipt issued for reimbursement","invoice","Fact","base.ec",0,0
"ec_dt_42",20,"42","Presumptive withholding tax document and withholding tax issued by the seller or by an intermediary","Presumptive withholding tax document and withholding tax issued by the seller or by an intermediary","invoice","Doc","base.ec",0,0
"ec_dt_43",21,"43","Liquidation for Hydrocarbon Exploitation and Exploration","Liquidation for Hydrocarbon Exploitation and Exploration","invoice","Doc","base.ec",0,0
"ec_dt_44",22,"44","Contributions and Contribution Voucher","Contributions and Contribution Voucher","invoice","Contrib","base.ec",0,0
"ec_dt_45",23,"45","Settlement of prepaid medicine","Settlement of prepaid medicine","purchase_liquidation","LiqAseg","base.ec",0,0
"ec_dt_47",24,"47","Credit note for Refund Issued by Intermediary","Credit note for Refund Issued by Intermediary","credit_note","NotCr","base.ec",0,1
"ec_dt_48",25,"48","Debit note for Refund Issued by Intermediary","Debit note for Refund Issued by Intermediary","debit_note","NotDb","base.ec",0,1
"ec_dt_49",26,"49","Direct Supplier of Exporter under Special Regime","Direct Supplier of Exporter under Special Regime","invoice","Doc","base.ec",0,0
"ec_dt_50",27,"50","To State and Public Institutions that receive income exempt from income tax","To State and Public Institutions that receive income exempt from income tax","invoice","Doc","base.ec",0,0
"ec_dt_51",28,"51","Credit note to State and Public Institutions receiving income exempt from income taxes","Credit note to State and Public Institutions receiving income exempt from income taxes","credit_note","Doc","base.ec",0,1
"ec_dt_52",29,"52","Debit note to government and public companies receiving income exempt from income taxes","Debit note to government and public companies receiving income exempt from income taxes","debit_note","Doc","base.ec",0,1
"ec_dt_294",30,"294","Used Personal Property Purchase Settlement","Used Personal Property Purchase Settlement","purchase_liquidation","LiqCo","base.ec",0,0
"ec_dt_344",31,"344","Used vehicle purchase liquidation","Used vehicle purchase liquidation","purchase_liquidation","LiqCo","base.ec",0,0
"ec_dt_364",32,"364","Delivery-Reception Act PET","Delivery-Reception Act PET","invoice","Rpet","base.ec",0,0
"ec_dt_370",33,"370","Invoice transport operator / partner","Invoice transport operator / partner","invoice","Fact","base.ec",0,0
"ec_dt_371",34,"371","Proof of membership in a transport operator","Proof of membership in a transport operator","invoice","Fact","base.ec",0,0
"ec_dt_372",35,"372","Transport operator / partner credit note","Transport operator / partner credit note","credit_note","NotCr","base.ec",0,0
"ec_dt_373",36,"373","Transport operator / partner debit note","Transport operator / partner debit note","debit_note","NotDb","base.ec",0,0
"ec_dt_07",37,"07","Withhold","Receipt of Withhold","withhold","Ret","base.ec",1,1

```

## File: data\l10n_latam_identification_type_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id='ec_ruc' model='l10n_latam.identification.type'>
            <field name='name'>RUC</field>
            <field name='description'>Single Taxpayer Registration</field>
            <field name='country_id' ref='base.ec'/>
            <field name='is_vat' eval='True'/>
            <field name='sequence'>10</field>
        </record>
        <record id='ec_dni' model='l10n_latam.identification.type'>
            <field name='name'>Citizenship</field>
            <field name='description'>Citizenship card or Identity Card</field>
            <field name='country_id' ref='base.ec'/>
            <field name='sequence'>20</field>
        </record>
        <!-- TODO: Remove in master -->
        <record id='ec_passport' model='l10n_latam.identification.type'>
            <field name='name'>Passport</field>
            <field name='description'>Passport for foreigners with domicile in the country (Deprecated)</field>
            <field name='country_id' ref='base.ec'/>
            <field name='active' eval="False"/>
            <field name='sequence'>20</field>
        </record>
        <!-- TODO: Remove in master -->
        <record id='ec_unknown' model='l10n_latam.identification.type'>
            <field name='name'>Unknown</field>
            <field name='description'>To identify, useful for quick sales registration (Deprecated)</field>
            <field name='country_id' ref='base.ec'/>
            <field name='active' eval="False"/>
            <field name='sequence'>110</field>
        </record>
    </data>
</odoo>

```

## File: data\res.bank.csv

```csv
id,bic,name,country
bank_1,1,Banco Central Del Ecuador,Ecuador
bank_2,10,Banco Pichincha C.A.,Ecuador
bank_3,17,Banco De Guayaquil S.A,Ecuador
bank_4,24,Banco City Bank,Ecuador
bank_5,25,Banco Machala,Ecuador
bank_6,29,Banco De Loja,Ecuador
bank_7,30,Banco Del Pacifico,Ecuador
bank_8,32,Banco Internacional,Ecuador
bank_9,34,Banco Amazonas,Ecuador
bank_10,35,Banco Del Austro,Ecuador
bank_11,36,Produbanco / Promerica,Ecuador
bank_12,37,Banco Bolivariano,Ecuador
bank_13,39,Banco Comercial De Manabi,Ecuador
bank_14,42,Banco General Ruminahui S.A.,Ecuador
bank_15,43,Banco Del Litoral S.A.,Ecuador
bank_16,59,Banco Solidario,Ecuador
bank_17,60,Banco Procredit S.A.,Ecuador
bank_18,61,Banco Capital,Ecuador
bank_19,65,Banco Desarrollo De Los Pueblos S.A.,Ecuador
bank_20,66,Banecuador B.P.,Ecuador
bank_21,70,Coop. Ahorro Y Credito 15 De Abril Ltda,Ecuador
bank_22,76,Coop. Jardin Azuayo,Ecuador
bank_23,79,Coop.De Ahorro Y Credito Policia Nacional,Ecuador
bank_24,82,Financoop,Ecuador
bank_25,201,Banco Delbank S.A.,Ecuador
bank_26,202,Banco Ecuatoriano De La Vivienda,Ecuador
bank_27,203,Cacpeco Ltda,Ecuador
bank_28,204,C.Peq.Empresa De Pastaza,Ecuador
bank_29,205,Coop. Ahorro Y Credito 23 De Julio,Ecuador
bank_30,206,Coop. Ahorro Y Credito 29 De Octubre,Ecuador
bank_31,207,Coop. Ahorro Y Credito Andalucia,Ecuador
bank_32,208,Coop. Ahorro Y Credito Cotocollao,Ecuador
bank_33,210,Coop. Ahorro Y Credito El Sagrario,Ecuador
bank_34,211,Coop. Ahorro Y Credito Guaranda Ltda,Ecuador
bank_35,213,Coop. Juventud Ecuatoriana Progresista Ltda.,Ecuador
bank_36,214,Coop. Ahorro Y Credito Manuel,Ecuador
bank_37,216,Coop. Ahorro Y Credito Oscus,Ecuador
bank_38,217,Coop. Ahorro Y Credito Pablo Munoz Vega,Ecuador
bank_39,218,Coop. Ahorro Y Credito Progreso,Ecuador
bank_40,219,Coop. Ahorro Y Credito Riobamba,Ecuador
bank_41,220,Coop. Ahorro Y Credito San Francisco,Ecuador
bank_42,222,Coop. Ahorro Y Credito Tulcan,Ecuador
bank_43,223,Coop. De Ahorro Y Credito Atuntaqui Ltda.,Ecuador
bank_44,224,Coop. De Ahorro Y Credito Comercio Ltda Portoviejo,Ecuador
bank_45,225,Coop. De Ahorro Y Credito La Dolorosa Ltda,Ecuador
bank_46,226,Coop. Prevision Ahorro Y Desarrollo,Ecuador
bank_47,227,Coop.Ahorro Y Credito Alianza Del Valle Ltda,Ecuador
bank_48,228,Coop Ahorro Y Credito Constrc Comercio Y Produc,Ecuador
bank_49,229,Coop.Ahorro Y Credito Chone Ltda,Ecuador
bank_50,231,Cooperativa De Ahorro Y Credito Santa Ana Ltda,Ecuador
bank_51,232,Financiera - Diners Club Del Ecuador,Ecuador
bank_52,233,Mutualista Ambato,Ecuador
bank_53,234,Mutualista Azuay,Ecuador
bank_54,236,Mutualista Imbabura,Ecuador
bank_55,238,Mutualista Pichincha,Ecuador
bank_56,240,Coop De A. Y C. Corporacion Centro Ltda.,Ecuador
bank_57,242,Coop. De A. Y C. Cordillera De Los Andes Ltda.,Ecuador
bank_58,243,Coop. De A. Y C. Puerto Francisco De Orellana,Ecuador
bank_59,244,Coop. De A Y C. Luz Del Valle,Ecuador
bank_60,245,Coop. De A Y C. Esperanza Y Progreso Del Valle,Ecuador
bank_61,246,Coop. De A. Y C. Choco Tungurahua Runa Ltda,Ecuador
bank_62,247,Coop. De A. Y C. Coopartamos Ltda,Ecuador
bank_63,249,Coop. De A. Y C. Emprendedores Coopemprender Ltda.,Ecuador
bank_64,250,Coop. De A. Y C. Nueva Loja Ltda.,Ecuador
bank_65,251,Coop. De A. Y C. Pichncha Ltda.,Ecuador
bank_66,252,Coop. Credito Y Ahorro San Francisco De Asis,Ecuador
bank_67,253,Coop.De Ahorro Y Credito Cacec Ltda. (Cotopaxi),Ecuador
bank_68,254,Coop.De A. Y C. Vencedores De Pichincha Ltda.,Ecuador
bank_69,255,Cooperativa De Ahorro Y Credito San Bartolo Ltda,Ecuador
bank_70,257,Coop. De A. Y C. Del Emigrante Ecuatoriano Y Su Fa,Ecuador
bank_71,258,Coop. Ahorro Y Credito La Libertad Ltda,Ecuador
bank_72,259,Coac Cuna De La Nacionalidad Ltda.,Ecuador
bank_73,260,Cooperativa De Ahorro Y Credito Macodes Ltda,Ecuador
bank_74,262,Cooperativa De Ahorro Y Credito Santa Isabel Ltda.,Ecuador
bank_75,263,Cooperativa De Ahorro Y Credito Ganansol Ltda,Ecuador
bank_76,264,Cooperativa De Ahorro Y Credito Del Azuay,Ecuador
bank_77,265,Cooperativa De Ahorro Y Credito Del Colegio De Arq,Ecuador
bank_78,266,Cooperativa De Ahorro Y Credito Nukanchik,Ecuador
bank_79,267,Coop De Ahorro Y Credito Santa Ana Cuenca,Ecuador
bank_80,268,Cooperativa De Ahorro Y Credito Multiempresarial L,Ecuador
bank_81,269,Coac Del Sindicato De Choferes Profesionales Del A,Ecuador
bank_82,270,Coop. Ahorro Y Credito Juan De Salinas Ltda.,Ecuador
bank_83,271,Coop. Ahorro Y Credito Puellaro Ltda,Ecuador
bank_84,272,Coop. Ahorro Y Credito Nueva Jerusalen,Ecuador
bank_85,273,Coop. Ahorro Y Credito Malchingui Ltda.,Ecuador
bank_86,275,Coop. De Ahorro Y Credito Chola Cuencana Ltda.,Ecuador
bank_87,276,Coop. Ahorro Y Credito Tena Ltda.,Ecuador
bank_88,277,Coop. Ahorro Y Credito De La Pequena Empresa Guala,Ecuador
bank_89,278,Coop. Ahorro Y Credito Alianza Minas Ltda.,Ecuador
bank_90,279,Coop. De Ahorro Y Credito Pedro Moncayo Ltda.,Ecuador
bank_91,281,Cooperativa De Ahorro Y Credito Provida,Ecuador
bank_92,283,Cooperativa De Ahorro Y Credito San Jose S.J.,Ecuador
bank_93,285,Coop. Ahorro Y Credito Senor De Giron,Ecuador
bank_94,287,Coop. De Ahorro Y Credito Educ.Del Tungurahua Ltda,Ecuador
bank_95,290,Cooperativa 15 De Agosto Pilacoto,Ecuador
bank_96,291,Coop. De Ahorro Y Credito Cristo Rey,Ecuador
bank_97,292,Coop. De Ahorro Y Credito Educadores De Chimborazo,Ecuador
bank_98,293,Cooperativa De Ahorro Y Credito Minga Ltda.,Ecuador
bank_99,294,Coop. De Ahorro Y Credito 4 De Octubre Ltda.,Ecuador
bank_100,295,Coop. De Ahorro Y Credito Credi Facil Ltda.,Ecuador
bank_101,296,Coop. De Ahorro Y Credito Alfonso Jaramillo C.C.C.,Ecuador
bank_102,297,Coop. De A. Y C. Indigena Sac Pelileo,Ecuador
bank_103,298,Coop. De A. Y C. Intercultural Tawantinsuyu Ltda.,Ecuador
bank_104,299,Coop. De A. Y C. Ocipsa,Ecuador
bank_105,300,Coop. De A. Y C. 21 De Noviembre Ltda.,Ecuador
bank_106,301,Coop. De A. Y C. La Floresta Ltda.,Ecuador
bank_107,302,Coop. De A. Y C. Corp. Org. Campesinas De Quisapin,Ecuador
bank_108,303,Coop. De A. Y C. Multicultural Banco Indigena Ltda,Ecuador
bank_109,304,Coop De A. Y C. Crecer Winari Ltda.,Ecuador
bank_110,305,Coop. De A. Y C. Banos De Agua Santa Ltda.,Ecuador
bank_111,306,Coop. De Ahorro Y Credito 1 De Julio,Ecuador
bank_112,307,Coop. De A. Y C. Sumak Nan Ltda.,Ecuador
bank_113,308,Cooperativa De Ahorro Y Credito Canar Ltda.,Ecuador
bank_114,309,Cooperativa De Ahorro Y Credito San Antonio Ltda.,Ecuador
bank_115,310,Coop. De Ahorro Y Credito Fundar,Ecuador
bank_116,312,Coop. Ahorro Y Credito Metropolis Ltda.,Ecuador
bank_117,313,Coop. De Ahorro Y Credito El Cafetal,Ecuador
bank_118,314,Coop.De Ahorro Y Credito Microempresarial Sucre,Ecuador
bank_119,315,Coop. De A. Y C. Afroecuatoriana De La Peq. Emp. L,Ecuador
bank_120,316,Cooperativa De Ahorro Y Credito Joyocoto Ltda.,Ecuador
bank_121,317,Coop. Ahorro Y Credito San Antonio Ltda.,Ecuador
bank_122,318,Coop. De A. Y C. Uniotavalo Ltda.,Ecuador
bank_123,319,Coop. De A. Y C. Union El Ejido,Ecuador
bank_124,320,Coop. De A. Y C. Genesis Ltda.,Ecuador
bank_125,321,Coop. De A Y C. Maria Auxiliadora De Quiroga Ltda,Ecuador
bank_126,322,Coop. De A. Y C. Fortaleza,Ecuador
bank_127,323,Banco Para La Asistencia Comunitaria Finca S.A.,Ecuador
bank_128,324,Coop. Calceta Ltda,Ecuador
bank_129,325,Coop. Ahorro Y Credito 9 De Octubre Ltda,Ecuador
bank_130,327,Coop. Ahorro Y Credito D La Peq Empr Cacpe Biblian,Ecuador
bank_131,328,Coop. Ahorro Y Credito San Gabriel Ltda.,Ecuador
bank_132,329,Coop. Ahorro Y Credito San Jose Ltda,Ecuador
bank_133,330,Coop. Ahorro Y Credito San Miguel De Los Bancos,Ecuador
bank_134,332,Coop. Ahorro Y Credito Semilla Del Progreso Ltda,Ecuador
bank_135,333,Coop. Ahorro. Y Credi. Mujeres Unidas Tantanakushk,Ecuador
bank_136,334,Coop. De Ahorro Y Cred. Santa Rosa Ltda,Ecuador
bank_137,335,Coop. De Ahorro Y Credito Once De Junio,Ecuador
bank_138,336,Coop. De A. Y C. Pujili Ltda,Ecuador
bank_139,337,Coop. De A. Y C. Credil Ltda.,Ecuador
bank_140,339,Coop. De A. Y C. Mushuk Pakari Ltda.,Ecuador
bank_141,340,Coop. De A. Y C. Uniblock Y Servicios Ltda.,Ecuador
bank_142,341,Coop. De A. Y C. San Fernando Limitada,Ecuador
bank_143,342,Coop. De A. Y C. Futuro Lamanense,Ecuador
bank_144,343,Coop. De A. Y C. Saquisili Ltda.,Ecuador
bank_145,344,Coop. De A. Y C. Innovacion Andina Ltda.,Ecuador
bank_146,345,Coop.De A.Y C. El Comerciante Ltda (Mies),Ecuador
bank_147,346,Coop. De A. Y C. Mushuk Wasi Ltda ( Mies ),Ecuador
bank_148,347,Cooperativa De Ahorro Y Credito Solidaria Ltda,Ecuador
bank_149,348,Cooperativa De Ahorro Y Credito Llanganates,Ecuador
bank_150,349,Interdin S.A.,Ecuador
bank_151,350,Coop.Aho Y Credito De La Peq. Emp. De Loja Cacpe,Ecuador
bank_152,351,Cooperativa De Ahorro Y Credito Economia Del Sur C,Ecuador
bank_153,352,Cooperativa De Ahorro Y Credito Migrantes Y Empren,Ecuador
bank_154,353,Cooperativa De Ahorro Y Credito San Sebastian,Ecuador
bank_155,354,Coop. De A. Y C. Del Sindicato De Choferes Profesi,Ecuador
bank_156,355,Coop.De Ahorro Y Credito 29 De Enero,Ecuador
bank_157,356,Coop. Ahorro Y Credito Agraria Mushuk Kawsay Ltda.,Ecuador
bank_158,357,Cooperativa De Ahorro Y Credito Runa Shungo Ltda.,Ecuador
bank_159,358,Coop. De A. Y C. San Marcos,Ecuador
bank_160,359,Coop. De A. Y C. Antorcha Ltda.,Ecuador
bank_161,360,Cooperativa De Ahorro Y Credito Socio Amigo,Ecuador
bank_162,361,Coop. De A. Y C. Indigenas Galapagos Ltda.,Ecuador
bank_163,362,Coop. Ahorro Y Credito Camara De Comercio Del Cant,Ecuador
bank_164,363,Cooperativa De Ahorro Y Credito La Benefica Ltda.,Ecuador
bank_165,364,Coop.De Ahorro Y Credito Abdon Calderon Ltda.,Ecuador
bank_166,365,Coop. A Y C Camara De Comercio Canton -El Carmen L,Ecuador
bank_167,366,Coop.De Ahorro Y Credito Agroproductiva Manabi Ltd,Ecuador
bank_168,367,Coop. De Ahorro Y Cred. La Inmaculada De San Placi,Ecuador
bank_169,368,Coop. Ahorro Y Credito Cacpe Manabi,Ecuador
bank_170,369,Coop.Ahorro Y Credito Magisterio Manabita Limitada,Ecuador
bank_171,370,Coac Tienda De Dinero Ltda.,Ecuador
bank_172,371,Coop.A.Y C. Santa Maria De La Manga Del Cura Ltda.,Ecuador
bank_173,375,Coop. Ahorro Y Credito Camara De Comercio Indigena,Ecuador
bank_174,379,Coop. De A. Y C. Guamote Ltda.,Ecuador
bank_175,382,Coop. De A.Y C. Produc. Ahorro Invers. Servicio P.,Ecuador
bank_176,383,Coop. De A. Y C. Frandesc Ltda.,Ecuador
bank_177,384,Coop. De A. Y C. Nuka Llakta Ltda.,Ecuador
bank_178,385,Coop. De A Y C. Camara De Comercio Riobamba,Ecuador
bank_179,386,Coop. De A. Y C. Bashalan Ltda.,Ecuador
bank_180,387,Coop. De A. Y C. Camara De Comercio Santo Domingo,Ecuador
bank_181,388,Coop. De A. Y C. Educadores Tulcan Ltda.,Ecuador
bank_182,400,Coop. Warmikunapak Rikchari,Ecuador
bank_183,403,Coop Ahorro Y Credito Mi Tierra,Ecuador
bank_184,404,Coop Ahorro Y Credito De La Peq Emp Cacpe Yanzatza,Ecuador
bank_185,405,Coop Ahorro Y Credito Fundesarrollo,Ecuador
bank_186,406,Caja Ecuaespana,Ecuador
bank_187,408,Coop De Ahorro Y Credito Nueva Huancavilca,Ecuador
bank_188,410,Coop De Ahorro Y Credito Erco Ltda,Ecuador
bank_189,411,Coop Ahorro Y Credito Camara Comercio De Ambato,Ecuador
bank_190,412,Coop Ahorro Y Credito Mushuc Runa Ltda,Ecuador
bank_191,414,Coop De Ahorro Y Credito Ambato Ltda,Ecuador
bank_192,415,Coop De Ahorro Y Credito Dorado Ltda,Ecuador
bank_193,416,Coop De Ahorro Y Credito Nuestros Abuelos Ltda,Ecuador
bank_194,417,Coop De Ahorro Y Credito Artesanos Ltda,Ecuador
bank_195,418,Coop De Ahorro Y Credito Santa Anita Ltda,Ecuador
bank_196,419,Coop De Ahorro Y Credito Huayco Pungo Ltda,Ecuador
bank_197,420,Coop Manuel Esteban Godoy Ortega Ltda Coopmego,Ecuador
bank_198,421,Coop Ahorro Y Credito Padre Julian Lorente Ltda,Ecuador
bank_199,422,Coop Ahorro Y Credito Cariamanga Ltda,Ecuador
bank_200,423,Coop De Ahorro Y Credito Marcabeli Ltda,Ecuador
bank_201,424,Coop De Ahorro Y Credito Agricola Junin Ltda,Ecuador
bank_202,425,Coop De Ahorro Y Credito Puerto Lopez Ltda,Ecuador
bank_203,427,Coop De Ahorro Y Credito Nueva Esperanza,Ecuador
bank_204,428,Coop De Ahorro Y Credito San Jorge Ltda,Ecuador
bank_205,429,Coop De Ahorro Y Credito Fernando Daquilema,Ecuador
bank_206,435,Coop De A Y C Educadores De Pastaza Ltda,Ecuador
bank_207,450,Coop. A. Y C. De La Peq. Emp. Cacpe Zamora Ltda,Ecuador
bank_208,451,Coop. De A. Y C. Desarrollo Integral Ltda,Ecuador
bank_209,452,Coop. De Ahorro Y Credito El Calvario Ltda,Ecuador
bank_210,453,Coop. De A. Y C. Juan Pio De Mora Ltda,Ecuador
bank_211,454,Coop. De Ahorro Y Credito Pilahuin,Ecuador
bank_212,455,Coop. De Ahorro Y Credito Pucara Ltda,Ecuador
bank_213,456,Coop. A. Y C. Camara De Comercio De Pindal Cadecop,Ecuador
bank_214,461,Banco Del Instituto Ecuatoriano De Seguridad Social,Ecuador
bank_215,462,Coop De A Y C De Serv Publ Del Min Educacion Y Cul,Ecuador
bank_216,463,Coop De A Y C 23 De Mayo Ltda,Ecuador
bank_217,464,Coop De A Y C Coca Ltda,Ecuador
bank_218,465,Coop De Ahorro Y Credito Pilahuin Tio Ltda,Ecuador
bank_219,467,Coop De Ahorro Y Credito Huaicana Ltda,Ecuador
bank_220,468,Coop De A Y C Credisur Ltda,Ecuador
bank_221,469,Fondo De Cesantia Del Magisterio Ecuat Fcme-Fcpc,Ecuador
bank_222,471,Coop De Ahorro Y Credito Huaquillas Ltda,Ecuador
bank_223,472,Coop De A Y C Banos Ltda,Ecuador
bank_224,473,Coop De Ahorro Y Credito Cac-Cica Mies,Ecuador
bank_225,475,Coop De A Y C Vencedores De Tungurahua,Ecuador
bank_226,476,Coop A Y C Ecuafuturo Ltda,Ecuador
bank_227,477,Coop De A Y C Maquita Cushun Ltda,Ecuador
bank_228,478,Coop De A Y C Valles Del Lirio,Ecuador
bank_229,479,Coop Esfuerzo Unido Para El Desarr Del Chilco,Ecuador
bank_230,480,Coop A Y C Carroceros De Tungurahua,Ecuador
bank_231,481,Coop De Ahorro Y Credito Simiatug Ltda,Ecuador
bank_232,482,Coop De A Y C Sinchi Runa Ltda,Ecuador
bank_233,483,Coop De A Y C Santa Rosa De Pautan Ltda,Ecuador
bank_234,484,Coop De Ahorro Y Credito San Miguel De Sigchos,Ecuador
bank_235,485,Coop De A Y C Sierra Centro Ltda,Ecuador
bank_236,486,Coop De A Y C Gonzanama Mies,Ecuador
bank_237,487,Coop De Ahorro Y Credito Quilanga Ltda,Ecuador
bank_238,488,Coop De Ahorro Y Credito 27 De Abril Loja,Ecuador
bank_239,489,Coop De A Y C Crediamigo Ltda Loja Mies,Ecuador
bank_240,490,Coop De Ahorro Y Credito Fortuna Mies,Ecuador
bank_241,491,Coop A Y C Profesionales Del Volante Union Ltda,Ecuador
bank_242,492,Coop De Ahorro Y Credito Cacpe Celica,Ecuador
bank_243,493,Coop De A Y C Futuro Y Progreso De Galapagos Ltda,Ecuador
bank_244,494,Coop De A Y C Sumac Llacta Ltda,Ecuador
bank_245,495,Coop De A Y C Lucha Campesina Ltda,Ecuador
bank_246,497,Coop De A Y C Maquita Cushunchic Ltda,Ecuador
bank_247,498,Coop A Y C Focla,Ecuador
bank_248,499,Coop De A Y C Casag Ltda,Ecuador
bank_249,500,Banco D Miro Sa,Ecuador
bank_250,501,Coop De A Y C General Ruminahui,Ecuador
bank_251,502,Coop De A Y C Financiera Indigena Ltda,Ecuador
bank_252,503,Coop A Y C 20 Febrero Ltda,Ecuador
bank_253,504,Coop De A Y C Del Col. Fisc. Vicente Rocafuerte,Ecuador
bank_254,505,Coop De A Y C Jadan Ltda (Mies),Ecuador
bank_255,509,Coop De A Y C Inka Kipu Ltda,Ecuador
bank_256,510,Coop De A Y C Accion Tungurahua Ltda,Ecuador
bank_257,511,Coop De A Y C Pijal,Ecuador
bank_258,512,Coop De A Y C Escencia Indigena Ltda,Ecuador
bank_259,513,Coop De A Y C Union Mercedaria Ltda,Ecuador
bank_260,514,Coop De A Y C Catamayo Ltda,Ecuador
bank_261,515,Coop De A Y C De La Peq Empresa Cacpe Macara,Ecuador
bank_262,516,Coop A Y C Nuevos Horizontes El Oro Ltda,Ecuador
bank_263,517,Banco Coopnacional Sa,Ecuador
bank_264,518,Coop De A Y C 16 De Junio,Ecuador
bank_265,519,Coop A Y C Esc Sup Politec Agrop De Manabi Manuel,Ecuador
bank_266,526,Coop De Ahorro Y Credito Fasaynan Ltda,Ecuador
bank_267,546,Coop Cacique Guritave,Ecuador
bank_268,547,Coop Solidaridad Y Progreso Oriental,Ecuador
bank_269,567,Coop. De Aho Y Cred Los Andes Latinos Ltda.,Ecuador
bank_270,569,C. De A. Y C Coopac Austro Ltda (Miess),Ecuador
bank_271,570,Coop. De A. Y C. Salasaca,Ecuador
bank_272,571,Coop. De A. Y C. Sumak Samy Ltda.,Ecuador
bank_273,572,C. A. Y C. Intercult. Tarpuk Runa Ltda.,Ecuador
bank_274,573,Coop. De A. Y C. Virgen Del Cisne,Ecuador
bank_275,574,C. De A Y C Educad. De Zamora Chinchipe,Ecuador
bank_276,575,C. De Ah. Y Credito Las Lagunas (Miess),Ecuador
bank_277,577,C. De A Y C. Educadores De El Oro Ltd,Ecuador
bank_278,578,C De A Y C Emplead Bancar Del Oro Ltda,Ecuador
bank_279,579,Cooperativa De Ah. Y Credito Riochico,Ecuador
bank_280,580,C. De A. Y C. San Martin De Tisaleo Ltda.,Ecuador
bank_281,581,Coop. De A. Y C. Alli Tarpuc Ltda.,Ecuador
bank_282,582,C. De A Y C. San Miguel De Pallatanga,Ecuador
bank_283,590,Coop De A Y C Fenix,Ecuador
bank_284,591,Coop.De Ahorro Y Credito Camara De Comercio De Loj,Ecuador
bank_285,592,Coop.De A. Y C. De Crecimiento Economico Rentable,Ecuador
bank_286,593,Cooperativa De Ahorro Y Credito Santiago Ltda,Ecuador
bank_287,594,Coop.De A. Y C. Popular Y Solidaria,Ecuador
bank_288,595,Coop.De Ahorro Y Credito San Placido Ltda.,Ecuador
bank_289,596,Coop.De Ahorro Y Credito Ciudad De Zamora,Ecuador
bank_290,597,Coop. De A Y C De La Pequena Empresa De Palora,Ecuador
bank_291,600,Coop. De A. Y C. Indigena Alfa Y Omega Ltda,Ecuador
bank_292,603,Cooperativa De Ahorro Y Credito Crea Ltda ( Mies),Ecuador
bank_293,604,Coop. De A. Y C. Chibuleo Ltda.,Ecuador
bank_294,605,Coop. De A. Y C. El Tesoro Pillareno,Ecuador
bank_295,606,Coop. De A. Y C. Kisapincha Ltda.,Ecuador
bank_296,607,Coop. De A. Y C. Juventud Unida Ltda.,Ecuador
bank_297,608,Coop. De A. Y C. Union Quisapincha Ltda.,Ecuador
bank_298,609,Coop. De A. Y C. 13 De Abril Ltda,Ecuador
bank_299,610,Coop. De A. Y C. Salinas Ltda.,Ecuador
bank_300,611,Coop. De A. Y C. San Pedro Ltda.,Ecuador
bank_301,612,Coop. De A. Y C. Los Chasquis Pastocalle Ltda.,Ecuador
bank_302,613,Coop. De A. Y C. Coopindigena Ltda.,Ecuador
bank_303,614,Coop. De A. Y C. La Union Ltda.,Ecuador
bank_304,615,Coop. De A. Y C. Padre Vicente Ponce Rubio,Ecuador
bank_305,633,Coop. De A. Y C. Mushug Causay Ltda.,Ecuador
bank_306,634,Coop. De A. Y C. 29 De Agosto,Ecuador
bank_307,641,Coop. De A. Y C. Credisocio,Ecuador
bank_308,673,Coop. De A. Y C. Fondo Para El Desarrollo Y La Vid,Ecuador
bank_309,674,Cooperativa De A Y C Nueva Vision,Ecuador
bank_310,675,Coop. De Ahorro Y Credito Focash Ltda.,Ecuador
bank_311,676,Coop. De A. Y C. San Vicente Del Sur Ltda.,Ecuador
bank_312,677,Coop. De A. Y C. Para La Vivienda Orden Y Segurida,Ecuador
bank_313,678,Coop De A. Y C. Camara De Comercio Joya De Los Sac,Ecuador
bank_314,679,Coop. De A. Y C. Focap,Ecuador
bank_315,680,Coop. De A. Y C. 18 De Noviembre,Ecuador
bank_316,681,Cooperativa De Ahorro Y Credito Dr. Cornelio Saenz,Ecuador
bank_317,682,Coop De A. Y C. Educadores Del Azuay,Ecuador
bank_318,683,Coop. De A. Y C. Indigena Sac Ltda,Ecuador
bank_319,684,Coop. De A. Y C. Credi Ya Ltda,Ecuador
bank_320,685,Coop. De A. Y C. San Bartolome Ltda,Ecuador
bank_321,686,Coop. De Ahorro Y Credito Mushuk Yuyay,Ecuador
bank_322,687,Coop De A. Y C. Sisay Kanari,Ecuador
bank_323,688,Cooperativa De A Y C Accion Imbaburapak Ltda.,Ecuador
bank_324,689,Coop De A. Y C. Hermes Gaibor Verdesoto,Ecuador
bank_325,690,Coop. De A. Y C. Semillas De Pangua,Ecuador
bank_326,691,Cooperativa De A Y C Mushuk Solidaria,Ecuador
bank_327,692,Coop De A. Y C. Panamericana Ltda,Ecuador
bank_328,693,Coop. De A. Y C. Vilcabamba Cacvil,Ecuador
bank_329,694,Coop. De A. Y C. Familia Solidaria,Ecuador
bank_330,695,Cooperativa De Ahorro Y Credito El Paraiso Manga D,Ecuador
bank_331,696,Coac De Los Profesores Empleados Y Trabajadores De,Ecuador
bank_332,697,Coac. Santa Rosa De San Carlos Ltda.,Ecuador
bank_333,698,Coop. De A. Y C. Constructor Del Desarrollo Solida,Ecuador
bank_334,699,Coop De A. Y C. Mushuk Yuyay Ltda,Ecuador
bank_335,700,Corporacion Financiera,Ecuador
bank_336,701,Coop. De A. Y C. Alianza Social Ecu. Alsec,Ecuador
bank_337,702,Coop. De A. Y C. Renovadora Ecu,Ecuador
bank_338,703,Coop. De A. Y C. Mushuk Yuyay - Napo,Ecuador
bank_339,704,Coop. De A. Y C. Santa Ana De Nayon,Ecuador
bank_340,705,Coop. De A. Y C. Educadores Del Napo,Ecuador
bank_341,706,Cooperativa De A. Y C. Textil 14 De Marzo,Ecuador
bank_342,707,Coop. De A Y C San Carlos Ltda.,Ecuador
bank_343,708,Coac A Y C Esperanza De Valle De La Virgen Ltda.,Ecuador
bank_344,709,Coac A Y C Zona De Capital Corcimol,Ecuador
bank_345,710,Coop.De A. Y C. Artesanal Del Azuay,Ecuador
bank_346,711,Cooperativa De Ahorro Y Credito Sumak Sisa,Ecuador
bank_347,712,Coop. De A. Y C. Rey David Ltda,Ecuador
bank_348,713,Coop. De A. Y C. Kullki Wasi Ltda.,Ecuador
bank_349,714,Coop De A. Y C. Sinchi Codefis,Ecuador
bank_350,715,Coop. De A. Y C. Fuerza De Los Andes,Ecuador
bank_351,716,Coop De A. Y C. Camino De Luz Ltda.,Ecuador
bank_352,717,Coac A Y C Educadores De Bolivar,Ecuador
bank_353,718,Coac San Miguel Ltda.,Ecuador
bank_354,719,Coop. De A. Y C. Salinerita,Ecuador
bank_355,720,Coop. De A. Y C. Bola Amarilla,Ecuador
bank_356,721,Coop A Y C Vision De Los Andes Visandes,Ecuador
bank_357,722,Coop. De A. Y C. Senor Del Arbol,Ecuador
bank_358,724,Coop. De A. Y C. Camara De Comercio De La Mana,Ecuador
bank_359,725,Coop De A. Y C. Educadores De Loja,Ecuador
bank_360,726,Coop De A. Y C. Indigena Sac Latacunga Ltda,Ecuador
bank_361,727,Coop De A.Y C. Cadecog Gonzanama,Ecuador
bank_362,728,Coac Coopymec Macara,Ecuador
bank_363,729,Coac Cadecom Macara,Ecuador
bank_364,730,Coop De A. Y C. 22 De Junio-Orianga,Ecuador
bank_365,732,Coop. A.Yc.Sagrada Familia Solidaridad,Ecuador
bank_366,733,Coop. De A. Y C. Naupa Kausay,Ecuador
bank_367,734,Ccop A Y C De Apecap Cac Apecap,Ecuador
bank_368,248,Coop De A. Y C. San Juan De Cotogchoa,Ecuador
bank_369,2557,Coop De Ahorro Y Credito La Merced,Ecuador
bank_370,338,Coop. De A. Y C. Cooptopaxi Ltda.,Ecuador
bank_371,620,Coop. De A. Y C. Grameen Amazonas,Ecuador
bank_372,621,Coop. De A. Y C. El Transportista Cacet,Ecuador
bank_373,626,Coop. De A Y C. Servidores Municipales De Cuenca,Ecuador
bank_374,628,Coop. De Ahorro Y Credito Profuturo Ltda.,Ecuador
bank_375,640,Coop. De A. Y C. Iliniza Ltda.,Ecuador
bank_376,642,Coop. De Ahorro Y Credito Saraguros,Ecuador
bank_377,643,Cooperativa De Ahorro Y Credito Inti Wasi Ltda.,Ecuador
bank_378,647,Cooperativa De Ahorro Y Credito San Isidro Ltda.,Ecuador
bank_379,722,Coop. De A. Y C. Empresas Comunitarias Coocredito,Ecuador
bank_380,723,Silvercross S.A Casa De Valores Sccv,Ecuador
bank_381,724,Real Casa De Valores De Guayaquil S.A.,Ecuador
bank_382,725,Cooperativa De Ahorro Y Credito Etapa,Ecuador
bank_383,726,Coop. Ahorro Y Credito Mascoop,Ecuador
bank_384,727,Coop. De A. Y C. Saint Michel Ltda.,Ecuador
bank_385,8000,Coop. Ahorro Y Credito Manantial De Oro Ltda.,Ecuador
bank_386,8001,Coop. De Ahorro Y Credito Andina Ltda.,Ecuador
bank_387,8002,Coop. De Ahorro Y Credito Accion Y Desarrollo Ltda,Ecuador
bank_388,9901,Coop. De A. Y C. Union Y Desarrollo,Ecuador
bank_389,9902,Coop. De A. Y C. Esperanza Del Futuro Ltda.,Ecuador
bank_390,9903,Coop. De A. Y C. 17 De Marzo Ltda,Ecuador
bank_391,9904,Coop. De A. Y C. Catar Ltda,Ecuador
bank_392,9905,Coop. De A. Y C. De Accion Popular,Ecuador
bank_393,9906,Coop. De A. Y C. Alli Tarpuk Ltda,Ecuador
bank_394,9907,Coop. De A. Y C. 16 De Julio Ltda,Ecuador
bank_395,9908,Coop. De A. Y C. Suboficiales De La Policia Nacion,Ecuador
bank_396,9909,Coop. De A. Y C. Politecnica Ltda.,Ecuador
bank_397,9910,Coop. De A. Y C. Nacional Llano Grande Ltda.,Ecuador
bank_398,9911,Coop. De A. Y C. San Valentin,Ecuador
bank_399,9912,Coop. De A. Y C. Totalife Ltda,Ecuador
bank_400,9913,Acciones Valores Casa De Valores S.A. Accival,Ecuador
bank_401,9914,Santa Fe Casa De Valores S.A. Santafevalores,Ecuador
bank_402,9915,Picaval Casa De Valores S.A,Ecuador
bank_403,9916,Analytica Securties C.A. Casa De Valores,Ecuador
bank_404,9917,Orion Casa De Valores S.A,Ecuador
bank_405,9918,Stratega Casa De Valores S.A.,Ecuador
bank_406,9919,Ecofsa Casa De Valores S.A.,Ecuador
bank_407,9920,Merchantvalores Casa De Valores S.A.,Ecuador
bank_408,9921,Casa De Valores Value S.A.,Ecuador
bank_409,9922,Inmovalor Casa De Valores S.A.,Ecuador
bank_410,9923,Ecuabursatil Casa De Valores S.A.,Ecuador
bank_411,9924,Plus Valores Casa De Valores S.A.,Ecuador
bank_412,9925,Mercapital Casa De Valores S.A.,Ecuador
bank_413,9926,Portafolio Casa De Valores S.A. Portavalor,Ecuador
bank_414,9927,Metrovalores Casa De Valores S.A.,Ecuador
bank_415,9928,Vectorglobal Wmg Casa De Valores S.A.,Ecuador
bank_416,9929,Holdunpartners Casa De Valores S.A.,Ecuador
bank_417,9930,Coop. De Ahorro Y Credito Base De Taura,Ecuador
bank_418,9931,Casa De Valores Banrio S.A.,Ecuador
bank_419,9932,Albion Casa De Valores S.A.,Ecuador
bank_420,9933,Masvalores Casa De Valores S.A Cavamasa,Ecuador
bank_421,9934,Casa De Valores Del Pacifico (Valpacifico) S.A.,Ecuador
bank_422,9935,Casa De Valores Advfin S.A.,Ecuador
bank_423,9936,Kapital One Casa De Valores S.A. Kaovalsa,Ecuador
bank_424,9937,Plusbursatil Casa De Valores S.A,Ecuador
bank_425,9938,Activa Ases.E Intermed.Valores Activalores Casa De,Ecuador
bank_426,9939,Citadel Casa De Valores S.A.,Ecuador
bank_427,9940,Decevale S.A.,Ecuador
bank_428,9941,Coop. De A. Y C. Santa Lucia Ltda,Ecuador
bank_429,9942,Coop. De A. Y C. Produactiva Ltda,Ecuador
bank_430,9943,Coop. De A. Y C. Tamboloma Ltda.,Ecuador
bank_431,9944,Coop De A. Y C. Pushak Runa Hombre Lider,Ecuador
bank_432,9945,Coop. De A. Y C. 15 De Agosto Ltda.,Ecuador
bank_433,9946,Coop. De A. Y C. Migrantes Del Ecuador Ltda,Ecuador
bank_434,9947,Cooperativa De Ahorro Y Credito Wuamanloma Ltda.,Ecuador
bank_435,9948,Coop. De A. Y C. Salate Ltda.,Ecuador
bank_436,9949,Coop. De A. Y C. Union Popular Ltda.,Ecuador
bank_437,9950,Coac Achik Inti Ltda,Ecuador
bank_438,9951,Coac El Migrante Solidario,Ecuador
bank_439,9952,Coop Ac Las Naves,Ecuador
bank_440,9953,Coop. De A. Y C. De Imbabura Amazonas,Ecuador
bank_441,9954,Coop. A. Y C. De Indigenas Chuchuqui Ltda,Ecuador
bank_442,9955,Coop. Ahorro Y Credito Focazsum Ltda.,Ecuador
bank_443,9956,Coop. De A. Y C. Solidaria Ltda.- Cotopaxi,Ecuador
bank_444,9957,Coop. De A. Y C. Unidad Y Progreso,Ecuador
bank_445,9958,Coac Loja Internacional Ltda.,Ecuador
bank_446,9959,Coac 23 De Enero,Ecuador
bank_447,9960,Coac Obras Publicas Fiscales De Loja Y Zamora,Ecuador
bank_448,9961,Coac Del Sind Chof Prof Virgen Del Cisne,Ecuador
bank_449,9962,Coac San Miguel De Chirijos Ltda,Ecuador
bank_450,9963,Coop. Ahorro Y Credito Quevedo Ltda.,Ecuador
bank_451,9964,Coop. De A. Y C. El Altar Ltda.,Ecuador
bank_452,9965,Coop. De A. Y C. Nueva Alianza De Chimborazo Ltda,Ecuador
bank_453,9966,Coop. De A. Y C. Luis Felipe Duchicela Xxvii,Ecuador
bank_454,9967,Coop. De A. Y C. Nizag Ltda.,Ecuador
bank_455,9968,Coop. De A. Y C. Makita Kunchik,Ecuador
bank_456,9969,Coop. De A. Y C. Cerrada Manuela Leon,Ecuador
bank_457,9970,Coop. De A. Y C. El Buen Sembrador Ltda.,Ecuador
bank_458,9971,Coac Sindicato De Choferes Profesionales De Yantza,Ecuador
bank_459,9972,Coop. De A. Y C. 5 De Mayo De Santa Martha De Cuba,Ecuador

```

## File: data\res_partner_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="ec_final_consumer" model="res.partner">
            <field name="name">Consumidor final</field>
            <field name="country_id" ref="base.ec"/>
            <field name="l10n_latam_identification_type_id" ref="l10n_ec.ec_ruc"/>
            <field name="vat">9999999999999</field>
        </record>
        <record id="ec_social_security" model="res.partner">
            <field name="image_1920" type="base64" file="l10n_ec/static/img/logo_iess.png"/>
            <field name="name">Instituto Ecuatoriano de Seguridad Social</field>
            <field name="state_id" ref="base.state_ec_17"/>
            <field name="country_id" ref="base.ec"/>
            <field name="website">https://www.iess.gob.ec</field>
            <field name="is_company">1</field>
            <field name="l10n_latam_identification_type_id" ref="l10n_ec.ec_ruc"/>
            <field name="vat">1760004650001</field>
        </record>
        <record id="ec_tax_authority" model="res.partner">
            <field name="image_1920" type="base64" file="l10n_ec/static/img/logo_sri.png"/>
            <field name="name">Servicio de Rentas Internas</field>
            <field name="state_id" ref="base.state_ec_17"/>
            <field name="country_id" ref="base.ec"/>
            <field name="website">http://www.sri.gob.ec</field>
            <field name="is_company">1</field>
            <field name="l10n_latam_identification_type_id" ref="l10n_ec.ec_ruc"/>
            <field name="vat">1760013210001</field>
        </record>
    </data>
</odoo>

```

## File: data\template\account.account-ec.csv

```csv
"id","code","name","reconcile","account_type","name@es"
"l10n_ec_ifrs_liquidity_transfer","11010301","Interbank transfers","True","asset_current","Transferencias interbancarias"
"ec11010401","11010401","Securities in custody","False","asset_current","Valores en custodia"
"ec110201","110201","Financial assets at fair value through profit or loss","False","asset_current","Activos financieros a valor razonable con cambios en resultados"
"ec110202","110202","Available-for-sale financial assets","False","asset_current","Activos financieros disponibles para la venta"
"ec110203","110203","Financial assets held to maturity","False","asset_current","Activos financieros mantenidos hasta el vencimiento"
"ec110204","110204","Provision for impairment","False","asset_current","Provision por deterioro"
"ec1102050101","1102050101","Accounts receivable from local unrelated customers","True","asset_receivable","Cuentas por cobrar clientes no relacionados locales"
"ec1102050102","1102050102","Accounts receivable from unrelated foreign customers","True","asset_receivable","Cuentas por cobrar clientes no relacionados extranjeros"
"ec11020601","11020601","Local customer related notes and accounts receivable","True","asset_receivable","Documentos y cuentas por cobrar cliente relacionados locales"
"ec11020602","11020602","Foreign customer related notes and accounts receivable","True","asset_receivable","Documentos y cuentas por cobrar cliente relacionados extranjeros"
"ec11020701","11020701","Salary advances","True","asset_current","Anticipo sueldos"
"ec11020702","11020702","Thirteenth salary advance","True","asset_current","Anticipo décimo tercer sueldo"
"ec11020703","11020703","Fourteenth salary advance","True","asset_current","Anticipo décimo cuarto sueldo"
"ec11020704","11020704","Profit sharing advance","True","asset_current","Anticipo utilidades"
"ec110208","110208","Other related accounts receivable","True","asset_receivable","Otras cuentas por cobrar relacionadas"
"ec110209","110209","Other accounts receivable","True","asset_receivable","Otras cuentas por cobrar"
"ec110210","110210","Allowance for doubtful accounts","False","asset_current","Provision por cuentas incobrables"
"ec110301","110301","Raw material inventories","False","asset_current","Inventarios de materia prima"
"ec110302","110302","Work in process inventories","False","asset_current","Inventarios de productos en proceso"
"ec110303","110303","Inventories of supplies or materials to be consumed in the production process","False","asset_current","Inventarios de suministros o materiales a ser consumidos en el proceso de produccion"
"ec110304","110304","Inventories of supplies or materials to be consumed in the provision of services","False","asset_current","Inventarios de suministros o materiales a ser consumidos en la prestacion de servicios"
"ec110305","110305","Inventories of finished goods and merchandise in stock - produced by the company","False","asset_current","Inventarios de prod. term. y mercad. en almacen - producido por la compañia"
"ec110306","110306","Inventories of finished goods and merchandise in stock - purchased from third parties","False","asset_current","Inventarios de prod. term. y mercad. en almacen - comprado a terceros"
"ec110307","110307","Goods in transit","False","asset_current","Mercaderias en transito"
"ec110308","110308","Provision for inventories at net realizable value","False","asset_current","Provision de inventarios por valor neto de realizacion"
"ec110309","110309","Provision for physical deterioration of inventories","False","asset_current","Provision de inventarios por deterioro fisico"
"ec110401","110401","Prepaid insurance","True","asset_prepayments","Seguros pagados por anticipado"
"ec110402","110402","Prepaid leases","True","asset_prepayments","Arriendos pagados por anticipado"
"ec110403","110403","Advances to suppliers","True","asset_prepayments","Anticipos a proveedores"
"ec_other_downpayments","11040401","Other advances delivered","True","asset_prepayments","Otros anticipos entregados"
"ec_purchase_vat","11050101","VAT paid on local purchases","False","asset_current","IVA pagado en compras locales"
"ec_purchase_vat_assets","11050102","VAT paid on local purchases of fixed assets","False","asset_current","IVA pagado en compras locales de activos fijos"
"ec_purchase_vat_service_imports","11050103","VAT paid on import of services","False","asset_current","IVA pagado en importacion de servicios"
"ec_purchase_vat_goods_imports","11050104","VAT paid on imported goods","False","asset_current","IVA pagado en importacion de bienes"
"ec_purchase_vat_assets_imports","11050105","VAT paid on import of fixed assets","False","asset_current","IVA pagado en importacion de activos fijos"
"ec_sale_vat_outstanding_withholds","11050106","VAT paid on withholdings at source","False","asset_current","IVA pagado en retenciones de la fuente"
"ec_purchase_vat_zero","11050107","VAT on purchases 0%","False","asset_current","IVA en compras 0%"
"ec_profit_tax_credit","11050201","Tributary credit for income tax","False","asset_current","Crédito tributario por Impuesto a la Renta"
"ec_vat_tax_credit","11050202","VAT income tax","False","asset_current","Crédito tributario de IVA"
"ec_withhold_tax_credit","11050203","Income tax for withhold VAT","False","asset_current","Crédito tributario por Retenciones IVA"
"ec_isd_tax_credit","11050204","Income tax for ISD VAT","False","asset_current","Crédito tributario por impuesto ISD"
"ec_others_tax_credit","11050205","Income tax for other concepts","False","asset_current","Credito Tributario por otros conceptos"
"ec110503","110503","Income tax advance","False","asset_current","Anticipo de impuesto a la renta"
"ec_sale_profit_withhold","110504","Withholding taxes paid","False","asset_current","Retenciones en la fuente pagadas"
"ec1106","1106","Non-current assets held for sale and discontinued operations","False","asset_current","Activos no corrientes mantenidos para la venta y operaciones discontinuadas"
"ec1107","1107","Construction in progress (nic 11 and sec.23 SMEs)","False","asset_current","Construcciones en proceso (nic 11 y secc.23 pymes)"
"ec1108","1108","Other current assets","False","asset_current","Otros activos corrientes"
"ec120101","120101","Land","False","asset_fixed","Terrenos"
"ec120102","120102","Buildings","False","asset_fixed","Edificios"
"ec120103","120103","Construction in progress","False","asset_fixed","Contrucciones en curso"
"ec120104","120104","Facilities","False","asset_fixed","Instalaciones"
"ec120105","120105","Furniture and fixtures","False","asset_fixed","Muebles y enseres"
"ec120106","120106","Machinery and equipment","False","asset_fixed","Maquinaria y equipo"
"ec120107","120107","Vessels, aircraft, barges and the like","False","asset_fixed","Naves, aeronaves, barcazas y similares"
"ec120108","120108","Computer equipment","False","asset_fixed","Equipo de computacion"
"ec120109","120109","Vehicles, transport equipment and mobile road equipment","False","asset_fixed","Vehiculos, equipos de transporte y equipo caminero movil"
"ec120110","120110","Other property, plant and equipment","False","asset_fixed","Otros propiedades, planta y equipo"
"ec120111","120111","Spare parts and tools","False","asset_fixed","Repuestos y herramientas"
"ec12011201","12011201","Accumulated depreciation of buildings","False","asset_fixed","Depreciacion acumulada edificios"
"ec12011202","12011202","Accumulated depreciation of construction in progress","False","asset_fixed","Depreciacion acumulada contrucciones en curso"
"ec12011203","12011203","Accumulated depreciation of facilities","False","asset_fixed","Depreciacion acumulada instalaciones"
"ec12011204","12011204","Accumulated depreciation of furniture and fixtures","False","asset_fixed","Depreciacion acumulada muebles y enseres"
"ec12011205","12011205","Accumulated depreciation of machinery and equipment","False","asset_fixed","Depreciacion acumulada maquinaria y equipo"
"ec12011206","12011206","Accumulated depreciation of vessels, aircraft, barges and similar items","False","asset_fixed","Depreciacion acumulada naves, aeronaves, barcazas y similares"
"ec12011207","12011207","Accumulated depreciation computer equipment","False","asset_fixed","Depreciacion acumulada equipo de computacion"
"ec12011208","12011208","Accumulated depreciation vehicles, transportation equipment and mobile road equipment","False","asset_fixed","Depreciacion acumulada vehiculos, equipos de transporte y equipo caminero movil"
"ec12011209","12011209","Accumulated depreciation other property, plant and equipment","False","asset_fixed","Depreciacion acumulada otros propiedades, planta y equipo"
"ec12011210","12011210","Accumulated depreciation parts and tools","False","asset_fixed","Depreciacion acumulada repuestos y herramientas"
"ec12011301","12011301","Accumulated impairment of buildings","False","asset_fixed","Deterioro acumulado edificios"
"ec12011302","12011302","Accumulated impairment of construction in progress","False","asset_fixed","Deterioro acumulado contrucciones en curso"
"ec12011303","12011303","Accumulated impairment of facilities","False","asset_fixed","Deterioro acumulado instalaciones"
"ec12011304","12011304","Accumulated impairment of furniture and fixtures","False","asset_fixed","Deterioro acumulado muebles y enseres"
"ec12011305","12011305","Accumulated impairment of machinery and equipment","False","asset_fixed","Deterioro acumulado maquinaria y equipo"
"ec12011306","12011306","Accumulated impairment vessels, aircraft, barges and similar","False","asset_fixed","Deterioro acumulado naves, aeronaves, barcazas y similares"
"ec12011307","12011307","Accumulated impairment of computer equipment","False","asset_fixed","Deterioro acumulado equipo de computacion"
"ec12011308","12011308","Accumulated impairment of vehicles, transportation equipment and mobile road equipment","False","asset_fixed","Deterioro acumulado vehiculos, equipos de transporte y equipo caminero movil"
"ec12011309","12011309","Accumulated impairment of other property, plant and equipment","False","asset_fixed","Deterioro acumulado otros propiedades, planta y equipo"
"ec12011310","12011310","Accumulated impairment of spare parts and tools","False","asset_fixed","Deterioro acumulado repuestos y herramientas"
"ec120201","120201","Land","False","asset_fixed","Terrenos"
"ec120202","120202","Buildings","False","asset_fixed","Edificios"
"ec120203","120203","Accumulated depreciation of investment property","False","asset_fixed","Depreciacion acumulada de propiedades de inversion"
"ec120204","120204","Accumulated impairment of investment properties","False","asset_fixed","Deterioro acumulado de propiedades de inversion"
"ec120301","120301","Live animals growing","False","asset_current","Animales vivos en crecimiento"
"ec120302","120302","Live animals in production","False","asset_current","Animales vivos en produccion"
"ec120303","120303","Growing plants","False","asset_current","Plantas en crecimiento"
"ec120304","120304","Plants in production","False","asset_current","Plantas en produccion"
"ec120305","120305","(-) accumulated depreciation of biological assets","False","asset_current","(-) depreciacion acumulada de activos biológicos"
"ec120306","120306","(-) accumulated impairment of biological assets","False","asset_current","(-) deterioro acumulado de activos biologícos"
"ec120401","120401","Capital gains","False","asset_current","Plusvalias"
"ec120402","120402","Trademarks, patents, key rights, patrimonial quotas and other similar rights","False","asset_current","Marcas, patentes, derechos de llave , cuotas patrimoniales y otros similares"
"ec120403","120403","Exploration and development assets","False","asset_current","Activos de exploracion y explotacion"
"ec120404","120404","Accumulated amortization of intangible assets","False","asset_current","Amortizacion acumulada de activos intangible"
"ec120405","120405","Accumulated impairment of intangible assets","False","asset_current","Deterioro acumulado de activo intangible"
"ec120406","120406","Other intangibles","False","asset_current","Otros intangibles"
"ec120501","120501","Deferred tax assets","False","asset_non_current","Activos por impuestos diferidos"
"ec120601","120601","Financial assets held to maturity","False","asset_non_current","Activos financieros mantenidos hasta el vencimiento"
"ec120602","120602","Provision for impairment of financial assets held to maturity","False","asset_non_current","Provision por deterioro de activos financieros mantenidos hasta el vencimiento"
"ec120603","120603","Notes and accounts receivable","True","asset_receivable","Documentos y cuentas por cobrar"
"ec120604","120604","Provision for uncollectible accounts of non-current financial assets","False","asset_non_current","Provision cuentas incobrables de activos financieros no corrientes"
"ec120701","120701","Subsidiary investments","False","asset_non_current","Inversiones subsidiarias"
"ec120702","120702","Associated investments","False","asset_non_current","Inversiones asociadas"
"ec120703","120703","Investments in joint ventures","False","asset_non_current","Inversiones negocios conjuntos"
"ec120704","120704","Other investments","False","asset_non_current","Otras inversiones"
"ec120705","120705","Provision for valuation of investments","False","asset_non_current","Provision valuacion de inversiones"
"ec120706","120706","Other non-current assets","False","asset_non_current","Otros activos no corrientes"
"ec2101","2101","Financial liabilities at fair value through profit or loss","False","liability_current","Pasivos financieros a valor razonable con cambios en resultados"
"ec2102","2102","Liabilities under finance leases","False","liability_current","Pasivos por contratos de arrendamiento financiero"
"ec210301","210301","Local accounts and notes payable","True","liability_payable","Cuentas y documentos por pagar locales"
"ec210302","210302","Foreign accounts and notes payable","True","liability_payable","Cuentas y documentos por pagar del exterior"
"ec210401","210401","Obligations with local financial institutions","True","liability_payable","Obligaciones con instituciones financieras locales"
"ec210402","210402","Obligations with foreign financial institutions","True","liability_payable","Obligaciones con instituciones financieras del exterior"
"ec2105010101","2105010101","Vacation Provision","True","liability_current","Provision vacaciones"
"ec2105010102","2105010102","Provision Thirteen","True","liability_current","Provision decimo tercero"
"ec2105010103","2105010103","Provision fourteenth","True","liability_current","Provision decimo cuarto"
"ec2105010104","2105010104","Other provisions in favor of employees","True","liability_current","Otras provisiones a favor de los empleados"
"ec21050102","21050102","Other local provisions","True","liability_current","Otras provisiones locales"
"ec210502","210502","Provisions from abroad","True","liability_current","Provisiones del exterior"
"ec2106","2106","Current portion of bonds issued","False","liability_current","Porcion corriente de obligaciones emitidas"
"ec21070101","21070101","Income tax withheld from employees","True","liability_current","Impuesto a la renta retenido a empleados"
"ec_vat_tax_deduction","21070102","Tax payable VAT","True","liability_current","Impuesto por pagar IVA"
"ec_profit_tax_deduction","21070103","Withholding payable taxes","True","liability_current","Retenciones en la fuente por pagar"
"ec_ice_tax_deduction","21070104","Tax payable ICE","True","liability_current","Impuesto por pagar ICE"
"ec_irbpnr_tax_deduction","21070105","IRBPNR tax payable","True","liability_current","Impuesto IRBPNR por pagar"
"ec_others_tax_deduction","21070106","Other taxes payable","True","liability_current","Otros impuestos por pagar"
"ec210702","210702","Income tax payable for the year","True","liability_payable","Impuesto a la renta por pagar del ejercicio"
"ec21070301","21070301","Unsecured loans","True","liability_current","Prestamos quirografarios"
"ec21070302","21070302","Mortgage loans","True","liability_current","Prestamos hipotecarios"
"ec21070303","21070303","Employer's contribution to the iess","True","liability_current","Aportacion patronal al iess"
"ec21070304","21070304","Personal contribution to iess","True","liability_current","Aportacion personal al iess"
"ec21070305","21070305","Reserve funds withheld from employees","True","liability_current","Fondos de reserva retenidos a empleados"
"ec21070306","21070306","Other current obligations with the iess","True","liability_current","Otras obligaciones corrientes con el iess"
"ec21070307","21070307","Solidarity benefits to the IESS",True,liability_current,"Prestaciones solidarias al IESS"
"ec21070401","21070401","Wages and salaries payable","True","liability_payable","Sueldos y salarios por pagar"
"ec21070402","21070402","Other current employee benefit obligations","True","liability_payable","Otras obligaciones corrientes por beneficios de ley a empleados"
"ec21070403","21070403","Tenths due","True","liability_payable","Décimos por pagar"
"ec210705","210705","Employee profit sharing payable for the year","True","liability_current","Participacion trabajadores por pagar del ejercicio"
"ec210706","210706","Dividends payable","True","liability_current","Dividendos por pagar"
"ec2108","2108","Sundry/related accounts payable","True","liability_payable","Cuentas por pagar diversas/relacionadas"
"ec2109","2109","Other financial liabilities","True","liability_payable","Otros pasivos financieros"
"ec2110","2110","Customer advances","True","liability_payable","Anticipos de clientes"
"ec2111","2111","Liabilities directly associated with non-current assets and discontinued operations","False","liability_current","Pasivos directamente asociados con los activos no corrientes y operaciones discontinuadas"
"ec211201","211201","Other long-term employee benefits","True","liability_payable","Otros beneficios a largo plazo para los empleados"
"ec211301","211301","Judicial withholdings","False","liability_current","Retenciones judiciales"
"ec_sale_vat","2114010101","VAT charged on local sales locales","False","liability_current","Iva cobrado en ventas locales"
"ec_sale_vat_assets","2114010102","VAT collected on sales of fixed assets","False","liability_current","Iva cobrado en ventas de activos fijos"
"ec_sale_vat_goods_exports","2114010103","VAT collected on exportation of goods","False","liability_current","Iva cobrado en exportacion de bienes"
"ec_sale_vat_services_exports","2114010104","VAT charged on exportation of services","False","liability_current","Iva cobrado en exportacion de servicios"
"ec_sale_vat_zero","2114010105","VAT on sales 0%","False","liability_current","Iva en ventas 0%"
"ret_ir_1x100","2114010201","Withholding tax 1%","False","liability_current","Retenciones de la fuente 1%"
"ret_ir_1_75x100","2114010202","Withholding tax 1.75%","False","liability_current","Retenciones de la fuente 1.75%"
"ret_ir_2x100","2114010203","Withholding tax 2%","False","liability_current","Retenciones de la fuente 2%"
"ret_ir_2_75x100","2114010204","Withholding tax 2.75%","False","liability_current","Retenciones de la fuente 2.75%"
"ret_ir_5x100","2114010205","Withholding tax 5%","False","liability_current","Retenciones de la fuente 5%"
"ret_ir_8x100","2114010206","Withholding tax 8%","False","liability_current","Retenciones de la fuente 8%"
"ret_ir_10x100","2114010207","Withholding tax 10%","False","liability_current","Retenciones de la fuente 10%"
"ret_ir_15x100","2114010208","Withholding tax 15%","False","liability_current","Retenciones de la fuente 15%"
"ret_ir_22x100","2114010209","Withholding tax 22%","False","liability_current","Retenciones de la fuente 22%"
"ret_ir_others","2114010210","Other withholdings","False","liability_current","Otras retenciones"
"ret_ir_3x100","2114010211","Withholding tax 3%","False","liability_current","Retenciones de la fuente 3%"
"ec_vat_withhold_10","2114010301","VAT withholdings 10%","False","liability_current","Retenciones de iva 10%"
"ec_vat_withhold_20","2114010302","VAT withholdings 20%","False","liability_current","Retenciones de iva 20%"
"ec_vat_withhold_30","2114010303","VAT withholdings 30%","False","liability_current","Retenciones de iva 30%"
"ec_vat_withhold_50","2114010304","VAT withholdings 50%","False","liability_current","Retenciones de iva 50%"
"ec_vat_withhold_70","2114010305","VAT withholdings 70%","False","liability_current","Retenciones de iva 70%"
"ec_vat_withhold_100","2114010306","VAT withholdings 100%","False","liability_current","Retenciones de iva 100%"
"ec_vat_withhold_others","2114010307","Other withholdings","False","liability_current","Otras retenciones"
"ec2201","2201","Liabilities under finance leases","True","liability_current","Pasivos por contratos de arrendamiento financiero"
"ec220201","220201","Local accounts and notes payable","True","liability_payable","Cuentas y documentos por pagar locales"
"ec220202","220202","Foreign accounts and notes payable","True","liability_payable","Cuentas y documentos por pagar del exterior"
"ec220301","220301","Obligations with local financial institutions","True","liability_payable","Obligaciones con instituciones financieras locales"
"ec220302","220302","Obligations with foreign financial institutions","True","liability_payable","Obligaciones con instituciones financieras del exterior"
"ec220401","220401","Local miscellaneous/related accounts payable","True","liability_payable","Cuentas por pagar diversas/relacionadas locales"
"ec220402","220402","Foreign sundry/related accounts payable","True","liability_payable","Cuentas por pagar diversas/relacionadas del exterior"
"ec2205","2205","Bonds issued","True","liability_payable","Obligaciones emitidas"
"ec2206","2206","Customer advances","True","liability_payable","Anticipos de clientes"
"ec220701","220701","Other non-current employee benefits","True","liability_payable","Otros beneficios no corrientes para los empleados"
"ec2208","2208","Other provisions","True","liability_non_current","Otras provisiones"
"ec220901","220901","Deferred income","True","liability_non_current","Ingresos diferidos"
"ec220902","220902","Deferred tax liabilities","True","liability_non_current","Pasivos por impuestos diferidos"
"ec2210","2210","Other non-current liabilities","False","liability_non_current","Otros pasivos no corrientes"
"ec3101","3101","Subscribed or assigned capital","False","equity","Capital suscrito o asignado"
"ec3102","3102","Unpaid subscribed capital, treasury stock","False","equity","Capital suscrito no pagado, acciones en tesoreria"
"ec32","32","Contributions from partners or stockholders for future capitalization","False","equity","Aportes de socios o accionistas para futura capitalizacion"
"ec33","33","Premium on primary issuance of shares","False","equity","Prima por emision primaria de acciones"
"ec3401","3401","Legal reserve","False","equity","Reserva legal"
"ec3402","3402","Optional and statutory reserves","False","equity","Reservas facultativa y estatutaria"
"ec3403","3403","Capital reserve","False","equity","Reserva de capital"
"ec3404","3404","Other reserves","False","equity","Otras reservas"
"ec3501","3501","Surplus of financial assets available for sale","False","equity","Superavit de activos financieros disponibles para la venta"
"ec3502","3502","Property, plant and equipment revaluation surplus","False","equity","Superavit por revaluacion de propiedades, planta y equipo"
"ec3503","3503","Revaluation surplus on intangible assets","False","equity","Superavit por revaluacion de activos intangibles"
"ec3504","3504","Other revaluation surplus","False","equity","Otros superavit por revaluacion"
"ec3601","3601","Retained earnings","False","equity","Ganancias acumuladas"
"ec3602","3602","Accumulated losses","False","equity","Perdidas acumuladas"
"ec3603","3603","Cumulative results from the first-time adoption of niif","False","equity","Resultados acumulados provenientes de la adopcion por primera vez de las niif"
"ec3701","3701","Net income for the period","False","equity_unaffected","Resultado neto del periodo"
"ec410101","410101","Sale of goods","False","income","Venta de bienes"
"ec410201","410201","Provision of services","False","income","Prestacion de servicios"
"ec4103","4103","Construction contracts","False","income","Contratos de construccion"
"ec4104","4104","Government grants","False","income","Subvenciones del gobierno"
"ec4105","4105","Royalties","False","income","Regalias"
"ec4106","4106","Interests","False","income","Intereses"
"ec4107","4107","Dividends","False","income","Dividendos"
"ec4108","4108","Gain on fair value measurement of biological assets","False","income","Ganancia por medición a valor razonable de activos biológicos"
"ec4109","4109","Other income from ordinary activities","False","income","Otros ingresos de actividades ordinarias"
"ec_early_pay_discount_loss","4110","(-) Early payment discount on sales","False","income","(-) Descuento pronto pago en ventas"
"ec4111","4111","(-) sales returns","False","income","(-) devoluciones en ventas"
"ec42","42","Gross profit","False","income","Ganancia bruta"
"ec4301","4301","Dividends","False","income","Dividendos"
"ec4302","4302","Financial interest","False","income","Intereses financieros"
"ec4303","4303","Gain on investments in associates / subsidiaries and others","False","income","Ganancia en inversiones en asociadas / subsidiarias y otras"
"ec4304","4304","Valuation of financial instruments at fair value through profit or loss","False","income","Valuacion de instrumentos financieros a valor razonable con cambio en resultados"
"ec430501","430501","Income from foreign exchange differences","False","income","Ingresos por diferecias de cambio"
"ec_income_cash_difference","430502","Cash difference account","False","income","Ingresos por diferencias en efectivo"
"ec430503","430503","Otras rentas","False","income","Other Income"
"ec510101","510101","Beginning inventory of goods not produced by the company","False","expense_direct_cost","Inventario inicial de bienes no producidos por la compañia"
"ec510102","510102","Net local purchases of goods not produced by the company","False","expense_direct_cost","Compras netas locales de bienes no producidos por la compañia"
"ec510103","510103","Imports of goods not produced by the company","False","expense_direct_cost","Importaciones de bienes no producidos por la compañia"
"ec510104","510104","Ending inventory of goods not produced by the company","False","expense_direct_cost","Inventario final de bienes no producidos por la compañia"
"ec510105","510105","Initial raw material inventory","False","expense_direct_cost","Inventario inicial de materia prima"
"ec510106","510106","Net local purchases of raw materials","False","expense_direct_cost","Compras netas locales de materia prima"
"ec510107","510107","Raw material imports","False","expense_direct_cost","Importaciones de materia prima"
"ec510108","510108","Final raw material inventory","False","expense_direct_cost","Inventario final de materia prima"
"ec510109","510109","Initial in-process inventory","False","expense_direct_cost","Inventario inicial de productos en proceso"
"ec510110","510110","Final in-process inventory","False","expense_direct_cost","Inventario final de productos en proceso"
"ec510111","510111","Beginning inventory of finished products","False","expense_direct_cost","Inventario inicial productos terminados"
"ec510112","510112","Final inventory of finished products","False","expense_direct_cost","Inventario final de productos terminados"
"ec51020101","51020101","Wages, salaries and other remuneration","False","expense_direct_cost","Sueldos, salarios y demas remuneraciones"
"ec51020102","51020102","Overtime","False","expense_direct_cost","Horas extra"
"ec51020103","51020103","Performance bonuses","False","expense_direct_cost","Bonificaciones por desempeño"
"ec51020201","51020201","Medical insurance","False","expense_direct_cost","Seguro medico"
"ec51020202","51020202","Training","False","expense_direct_cost","Capacitacion"
"ec51020203","51020203","Uniforms","False","expense_direct_cost","Uniformes"
"ec51020204","51020204","Feeding","False","expense_direct_cost","Alimentacion"
"ec5102020501","5102020501","Employee income tax borne by the company","False","expense_direct_cost","Impuesto a la renta empleados asumido por la empresa"
"ec5102020502","5102020502","Transportation","False","expense_direct_cost","Transporte"
"ec51020301","51020301","Employer's contribution","False","expense_direct_cost","Aporte patronal"
"ec51020302","51020302","Reserve funds","False","expense_direct_cost","Fondos de reserva"
"ec51020401","51020401","Thirteenth salary","False","expense_direct_cost","Decimo tercer sueldo"
"ec51020402","51020402","Fourteenth salary","False","expense_direct_cost","Decimo cuarto sueldo"
"ec51020403","51020403","Vacations","False","expense_direct_cost","Vacaciones"
"ec51020404","51020404","Untimely dismissal","False","expense_direct_cost","Despido intempestivo"
"ec51020405","51020405","Eviction","False","expense_direct_cost","Desahucio"
"ec51020406","51020406","Other benefits and indemnities","False","expense_direct_cost","Otros beneficios e indemnizaciones"
"ec51030101","51030101","Wages, salaries and other remuneration","False","expense_direct_cost","Sueldos, salarios y demas remuneraciones"
"ec51030102","51030102","Overtime","False","expense_direct_cost","Horas extra"
"ec51030103","51030103","Performance bonuses","False","expense_direct_cost","Bonificaciones por desempeño"
"ec51030201","51030201","Medical insurance","False","expense_direct_cost","Seguro medico"
"ec51030202","51030202","Training","False","expense_direct_cost","Capacitacion"
"ec51030203","51030203","Uniforms","False","expense_direct_cost","Uniformes"
"ec51030204","51030204","Food","False","expense_direct_cost","Alimentacion"
"ec5103020501","5103020501","Employee income tax borne by the company","False","expense_direct_cost","Impuesto a la renta empleados asumido por la empresa"
"ec5103020502","5103020502","Transportation","False","expense_direct_cost","Transporte"
"ec51030301","51030301","Employer's contribution","False","expense_direct_cost","Aporte patronal"
"ec51030302","51030302","Reserve funds","False","expense_direct_cost","Fondos de reserva"
"ec51030401","51030401","Thirteenth salary","False","expense_direct_cost","Decimo tercer sueldo"
"ec51030402","51030402","Fourteenth salary","False","expense_direct_cost","Decimo cuarto sueldo"
"ec51030403","51030403","Vacations","False","expense_direct_cost","Vacaciones"
"ec51030404","51030404","Untimely dismissal","False","expense_direct_cost","Despido intempestivo"
"ec51030405","51030405","Eviction","False","expense_direct_cost","Desahucio"
"ec51030406","51030406","Other benefits and indemnities","False","expense_direct_cost","Otros beneficios e indemnizaciones"
"ec51040101","51040101","Buildings","False","expense_depreciation","Edificios"
"ec51040102","51040102","Construction in progress","False","expense_depreciation","Contrucciones en curso"
"ec51040103","51040103","Facilities","False","expense_depreciation","Instalaciones"
"ec51040104","51040104","Furniture and fixtures","False","expense_depreciation","Muebles y enseres"
"ec51040105","51040105","Machinery and equipment","False","expense_depreciation","Maquinaria y equipo"
"ec51040106","51040106","Vessels, aircraft, barges and similar","False","expense_depreciation","Naves, aeronaves, barcazas y similares"
"ec51040107","51040107","Computer equipment","False","expense_depreciation","Equipo de computacion"
"ec51040108","51040108","Vehicles, transport equipment and mobile road equipment","False","expense_depreciation","Vehiculos, equipos de transporte y equipo caminero movil"
"ec51040109","51040109","Other property, plant and equipment","False","expense_depreciation","Otros propiedades, planta y equipo"
"ec51040110","51040110","Spare parts and tools","False","expense_depreciation","Repuestos y herramientas"
"ec510402","510402","Impairment or loss of biological assets","False","expense_direct_cost","Deterioro o pérdidas de activos biológicos"
"ec51040301","51040301","Buildings","False","expense_direct_cost","Edificios"
"ec51040302","51040302","Construction in progress","False","expense_direct_cost","Contrucciones en curso"
"ec51040303","51040303","Facilities","False","expense_direct_cost","Instalaciones"
"ec51040304","51040304","Furniture and fixtures","False","expense_direct_cost","Muebles y enseres"
"ec51040305","51040305","Machinery and equipment","False","expense_direct_cost","Maquinaria y equipo"
"ec51040306","51040306","Vessels, aircraft, barges and similar","False","expense_direct_cost","Naves, aeronaves, barcazas y similares"
"ec51040307","51040307","Computer equipment","False","expense_direct_cost","Equipo de computacion"
"ec51040308","51040308","Vehicles, transport equipment and mobile road equipment","False","expense_direct_cost","Vehiculos, equipos de transporte y equipo caminero movil"
"ec51040309","51040309","Other property, plant and equipment","False","expense_direct_cost","Otros propiedades, planta y equipo"
"ec51040310","51040310","Spare parts and tools","False","expense_direct_cost","Repuestos y herramientas"
"ec510404","510404","Net realizable value effect of inventories","False","expense_direct_cost","Efecto valor neto de realizacion de inventarios"
"ec510405","510405","Warranty expenses on sale of products or services","False","expense_direct_cost","Gasto por garantias en venta de productos o servicios"
"ec510406","510406","Maintenance and repairs","False","expense_direct_cost","Mantenimiento y reparaciones"
"ec510407","510407","Material supplies and spare parts","False","expense_direct_cost","Suministros materiales y repuestos"
"ec51040801","51040801","Other production costs","False","expense_direct_cost","Otros costos de produccion"
"ec52010101","52010101","Wages, salaries and other remuneration","False","expense","Sueldos, salarios y demas remuneraciones"
"ec51040802","51040802","Faltantes y sobrantes de ajustes de inventario","False","expense_direct_cost",""
"ec51040803","51040803","Desperdicios en el proceso productivo","False","expense_direct_cost",""
"ec52010102","52010102","Overtime","False","expense","Horas extra"
"ec52010103","52010103","Performance bonuses","False","expense","Bonificaciones por desempeño"
"ec52010201","52010201","Employer's contribution","False","expense","Aporte patronal"
"ec52010202","52010202","Reserve funds","False","expense","Fondos de reserva"
"ec52010301","52010301","Thirteenth salary","False","expense","Decimo tercer sueldo"
"ec52010302","52010302","Fourteenth salary","False","expense","Decimo cuarto sueldo"
"ec52010303","52010303","Vacations","False","expense","Vacaciones"
"ec52010304","52010304","Untimely dismissal","False","expense","Despido intempestivo"
"ec52010305","52010305","Eviction","False","expense","Desahucio"
"ec52010306","52010306","Other benefits and indemnities","False","expense","Otros beneficios e indemnizaciones"
"ec52010401","52010401","Medical insurance","False","expense","Seguro medico"
"ec52010402","52010402","Training","False","expense","Capacitacion"
"ec52010403","52010403","Uniforms","False","expense","Uniformes"
"ec52010404","52010404","Food","False","expense","Alimentacion"
"ec5201040501","5201040501","Employee income tax borne by the company","False","expense","Impuesto a la renta empleados asumido por la empresa"
"ec5201040502","5201040502","Transportation","False","expense","Transporte"
"ec520105","520105","Fees, commissions and per diems to natural persons","False","expense","Honorarios, comisiones y dietas a personas naturales"
"ec520106","520106","Remuneration to other self-employed workers","False","expense","Remuneraciones a otros trabajadores autonomos"
"ec520107","520107","Fees to foreigners for occasional services","False","expense","Honorarios a extranjeros por servicios ocasionales"
"ec520108","520108","Maintenance and repairs","False","expense","Mantenimiento y reparaciones"
"ec520109","520109","Operating leases","False","expense","Arrendamiento operativo"
"ec520110","520110","Commissions","False","expense","Comisiones"
"ec520111","520111","Promotion and advertising","False","expense","Promocion y publicidad"
"ec520112","520112","Fuels","False","expense","Combustibles"
"ec520113","520113","Lubricants","False","expense","Lubricantes"
"ec520114","520114","Insurance and reinsurance (premiums and cessions)","False","expense","Seguros y reaseguros (primas y cesiones)"
"ec520115","520115","Transportation","False","expense","Transporte"
"ec520116","520116","Operating expenses (entertainment for shareholders, employees and customers)","False","expense","Gastos de gestion (agasajos a accionistas, trabajadores y clientes)"
"ec520117","520117","Travel expenses","False","expense","Gastos de viaje"
"ec52011801","52011801","Drinking water","False","expense","Agua potable"
"ec52011802","52011802","Electric energy","False","expense","Energia electrica"
"ec52011803","52011803","Fixed telephony","False","expense","Telefonia fija"
"ec52011804","52011804","Mobile Telephony","False","expense","Telefonia movil"
"ec520119","520119","Notaries and registrars of property or mercantile registries","False","expense","Notarios y registradores de la propiedad o mercantiles"
"ec520120","520120","Taxes, contributions and other","False","expense","Impuestos, contribuciones y otros"
"ec5201210101","5201210101","Buildings","False","expense_depreciation","Edificios"
"ec5201210102","5201210102","Construction in progress","False","expense_depreciation","Contrucciones en curso"
"ec5201210103","5201210103","Facilities","False","expense_depreciation","Instalaciones"
"ec5201210104","5201210104","Furniture and fixtures","False","expense_depreciation","Muebles y enseres"
"ec5201210105","5201210105","Machinery and equipment","False","expense_depreciation","Maquinaria y equipo"
"ec5201210106","5201210106","Vessels, aircraft, barges and similar","False","expense_depreciation","Naves, aeronaves, barcazas y similares"
"ec5201210107","5201210107","Computer equipment","False","expense_depreciation","Equipo de computacion"
"ec5201210108","5201210108","Vehicles, transport equipment and mobile road equipment","False","expense_depreciation","Vehiculos, equipos de transporte y equipo caminero movil"
"ec5201210109","5201210109","Other property, plant and equipment","False","expense_depreciation","Otros propiedades, planta y equipo"
"ec5201210110","5201210110","Spare parts and tools","False","expense_depreciation","Repuestos y herramientas"
"ec52012102","52012102","Investment properties","False","expense_depreciation","Propiedades de inversion"
"ec52012201","52012201","Intangibles","False","expense","Intangibles"
"ec52012202","52012202","Other assets","False","expense","Otros activos"
"ec5201230101","5201230101","Buildings","False","expense","Edificios"
"ec5201230102","5201230102","Construction in progress","False","expense","Contrucciones en curso"
"ec5201230103","5201230103","Facilities","False","expense","Instalaciones"
"ec5201230104","5201230104","Furniture and fixtures","False","expense","Muebles y enseres"
"ec5201230105","5201230105","Machinery and equipment","False","expense","Maquinaria y equipo"
"ec5201230106","5201230106","Vessels, aircraft, barges and similar","False","expense","Naves, aeronaves, barcazas y similares"
"ec5201230107","5201230107","Computer equipment","False","expense","Equipo de computacion"
"ec5201230108","5201230108","Vehicles, transport equipment and mobile road equipment","False","expense","Vehiculos, equipos de transporte y equipo caminero movil"
"ec5201230109","5201230109","Other property, plant and equipment","False","expense","Otros propiedades, planta y equipo"
"ec5201230110","5201230110","Spare parts and tools","False","expense","Repuestos y herramientas"
"ec52012302","52012302","Financial instruments","False","expense","Instrumentos financieros"
"ec52012303","52012303","Intangibles","False","expense","Intangibles"
"ec52012304","52012304","Accounts receivable","False","expense","Cuentas por cobrar"
"ec52012305","52012305","Other assets","False","expense","Otros activos"
"ec520124","520124","Expenses for abnormal amounts of utilization in the production process","False","expense","Gastos por cantidades anormales de utilización en el proceso de producción"
"ec520125","520125","Restructuring expense","False","expense","Gasto por reestructuracion"
"ec520126","520126","Net realizable value of inventories","False","expense","Valor neto de realizacion de inventarios"
"ec52012701","52012701","Printing service","False","expense","Servicio de imprenta"
"ec52012702","52012702","Office supplies","False","expense","Suministros de oficina"
"ec52012703","52012703","Cleaning and janitorial supplies and supplies","False","expense","Suministros y articulos de aseo y limpieza"
"ec52012704","52012704","Assessments and condominium","False","expense","Alícuotas y condominio"
"ec52012705","52012705","Guides, couriers, couriers","False","expense","Guias, couriers, correos"
"ec52012706","52012706","Tolls, parking and others","False","expense","Peajes, parqueaderos y otros"
"ec52012707","52012707","Training, courses and seminars","False","expense","Capacitación, cursos y seminarios"
"ec52012708","52012708","Food and refreshments","False","expense","Alimentación y refrigerios"
"ec52012709","52012709","Security and surveillance","False","expense","Seguridad y vigilancia"
"ec52012710","52012710","Cleaning services","False","expense","Servicios de limpieza"
"ec52012711","52012711","Other taxes","False","expense","Mantenimiento equipo computo"
"ec52012712","52012712","Maintenance of computer equipment","False","expense","Mantenimiento equipo computo"
"ec52012713","52012713","Legal expenses","False","expense","Gastos legales"
"ec52012714","52012714","Services contracted to third parties","False","expense","Servicios contratados a terceros"
"ec52012715","52012715","Other expenses","False","expense","Otros gastos"
"ec52020101","52020101","Wages, salaries and other remuneration","False","expense","Sueldos, salarios y demas remuneraciones"
"ec52020102","52020102","Overtime","False","expense","Horas extra"
"ec52020103","52020103","Performance bonuses","False","expense","Bonificaciones por desempeño"
"ec52020201","52020201","Employer's contribution","False","expense","Aporte patronal"
"ec52020202","52020202","Reserve funds","False","expense","Fondos de reserva"
"ec52020301","52020301","Thirteenth salary","False","expense","Decimo tercer sueldo"
"ec52020302","52020302","Fourteenth salary","False","expense","Decimo cuarto sueldo"
"ec52020303","52020303","Vacations","False","expense","Vacaciones"
"ec52020304","52020304","Untimely dismissal","False","expense","Despido intempestivo"
"ec52020305","52020305","Eviction","False","expense","Desahucio"
"ec52020306","52020306","Other benefits and indemnities","False","expense","Otros beneficios e indemnizaciones"
"ec52020401","52020401","Medical insurance","False","expense","Seguro medico"
"ec52020402","52020402","Training","False","expense","Capacitacion"
"ec52020403","52020403","Uniforms","False","expense","Uniformes"
"ec52020404","52020404","Food","False","expense","Alimentacion"
"ec52020405","52020405","Other employee benefit plan expenses","False","expense","Otros gastos de planes de beneficios a empleados"
"ec5202040501","5202040501","Employee income tax borne by the company","False","expense","Impuesto a la renta empleados asumido por la empresa"
"ec5202040502","5202040502","Transportation","False","expense","Transporte"
"ec520205","520205","Fees, commissions and per diems to natural persons","False","expense","Honorarios, comisiones y dietas a personas naturales"
"ec520206","520206","Remuneration to other self-employed workers","False","expense","Remuneraciones a otros trabajadores autonomos"
"ec520207","520207","Fees to foreigners for occasional services","False","expense","Honorarios a extranjeros por servicios ocasionales"
"ec520208","520208","Maintenance and repairs","False","expense","Mantenimiento y reparaciones"
"ec520209","520209","Operating leases","False","expense","Arrendamiento operativo"
"ec520210","520210","Commissions","False","expense","Comisiones"
"ec520211","520211","Promotion and advertising","False","expense","Promocion y publicidad"
"ec520212","520212","Fuels","False","expense","Combustibles"
"ec520213","520213","Lubricants","False","expense","Lubricantes"
"ec520214","520214","Insurance and reinsurance (premiums and cessions)","False","expense","Seguros y reaseguros (primas y cesiones)"
"ec520215","520215","Transportation","False","expense","Transporte"
"ec520216","520216","Operating expenses (entertainment for shareholders, employees and customers)","False","expense","Gastos de gestion (agasajos a accionistas, trabajadores y clientes)"
"ec520217","520217","Travel expenses","False","expense","Gastos de viaje"
"ec52021801","52021801","Drinking water","False","expense","Agua potable"
"ec52021802","52021802","Electric energy","False","expense","Energia electrica"
"ec52021803","52021803","Fixed telephony","False","expense","Telefonia fija"
"ec52021804","52021804","Mobile Telephony","False","expense","Telefonia movil"
"ec520219","520219","Notaries and registrars of property or mercantile records","False","expense","Notarios y registradores de la propiedad o mercantiles"
"ec52022001","52022001","Municipal license","False","expense","Patente municipal"
"ec52022002","52022002","1.5x thousand total assets","False","expense","1.5x mil activos totales"
"ec52022003","52022003","Contributions","False","expense","Contribuciones super cias"
"ec52022004","52022004","Chamber of Commerce","False","expense","Camara de comercio"
"ec52022005","52022005","Currency Outflow Tax (ISD)","False","expense","Impuesto Salida de Divisas (ISD)"
"ec52022006","52022006","Other taxes","False","expense","Otros impuestos"
"ec5202210101","5202210101","Buildings","False","expense_depreciation","Edificios"
"ec5202210102","5202210102","Construction in progress","False","expense_depreciation","Contrucciones en curso"
"ec5202210103","5202210103","Facilities","False","expense_depreciation","Instalaciones"
"ec5202210104","5202210104","Furniture and fixtures","False","expense_depreciation","Muebles y enseres"
"ec5202210105","5202210105","Machinery and equipment","False","expense_depreciation","Maquinaria y equipo"
"ec5202210106","5202210106","Vessels, aircraft, barges and similar","False","expense_depreciation","Naves, aeronaves, barcazas y similares"
"ec5202210107","5202210107","Computer equipment","False","expense_depreciation","Equipo de computacion"
"ec5202210108","5202210108","Vehicles, transport equipment and mobile road equipment","False","expense_depreciation","Vehiculos, equipos de transporte y equipo caminero movil"
"ec5202210109","5202210109","Other property, plant and equipment","False","expense_depreciation","Otros propiedades, planta y equipo"
"ec5202210110","5202210110","Spare parts and tools","False","expense_depreciation","Repuestos y herramientas"
"ec52022102","52022102","Investment properties","False","expense_depreciation","Propiedades de inversion"
"ec52022201","52022201","Intangibles","False","expense","Intangibles"
"ec52022202","52022202","Other assets","False","expense","Otros activos"
"ec5202230101","5202230101","Buildings","False","expense","Edificios"
"ec5202230102","5202230102","Construction in progress","False","expense","Contrucciones en curso"
"ec5202230103","5202230103","Facilities","False","expense","Instalaciones"
"ec5202230104","5202230104","Furniture and fixtures","False","expense","Muebles y enseres"
"ec5202230105","5202230105","Machinery and equipment","False","expense","Maquinaria y equipo"
"ec5202230106","5202230106","Vessels, aircraft, barges and similar","False","expense","Naves, aeronaves, barcazas y similares"
"ec5202230107","5202230107","Computer equipment","False","expense","Equipo de computacion"
"ec5202230108","5202230108","Vehicles, transport equipment and mobile road equipment","False","expense","Vehiculos, equipos de transporte y equipo caminero movil"
"ec5202230109","5202230109","Other property, plant and equipment","False","expense","Otros propiedades, planta y equipo"
"ec5202230110","5202230110","Spare parts and tools","False","expense","Repuestos y herramientas"
"ec52022302","52022302","Inventories","False","expense","Inventarios"
"ec52022303","52022303","Financial instruments","False","expense","Instrumentos financieros"
"ec52022304","52022304","Intangibles","False","expense","Intangibles"
"ec52022305","52022305","Accounts receivable","False","expense","Cuentas por cobrar"
"ec52022306","52022306","Other assets","False","expense","Otros activos"
"ec520224","520224","Expenses for abnormal amounts of utilization in the production process","False","expense","Gastos por cantidades anormales de utilización en el proceso de producción"
"ec520225","520225","Restructuring expense","False","expense","Gasto por reestructuracion"
"ec520226","520226","Net realizable value of inventories","False","expense","Valor neto de realizacion de inventarios"
"ec520227","520227","Income tax expense (deferred income tax assets and liabilities)","False","expense","Gasto impuesto a la renta (activos y pasivos diferidos)"
"ec52022801","52022801","Office supplies","False","expense","Suministros de oficina"
"ec52022802","52022802","Cleaning and sanitation supplies and supplies","False","expense","Suministros y artículos de aseo y limpieza"
"ec52022803","52022803","Assessments and condominium","False","expense","Alícuotas y condominio"
"ec52022804","52022804","Guides, couriers, couriers","False","expense","Guias, couriers, correos"
"ec52022805","52022805","Tolls, parking and others","False","expense","Peajes, parqueaderos y otros"
"ec52022806","52022806","Training, courses and seminars","False","expense","Capacitación, cursos y seminarios"
"ec52022807","52022807","Food and refreshments","False","expense","Alimentación y refrigerios"
"ec52022808","52022808","Security and surveillance","False","expense","Seguridad y vigilancia"
"ec52022809","52022809","Cleaning services","False","expense","Servicios de limpieza"
"ec52022810","52022810","Other taxes","False","expense","Mantenimiento equipo computo"
"ec52022811","52022811","Other taxes","False","expense","Mantenimiento equipo computo"
"ec52022812","52022812","Osteosynthesis material","False","expense","Material de osteosintesis"
"ec52022813","52022813","Construction costs","False","expense","Gastos de construcción"
"ec52022814","52022814","Legal expenses","False","expense","Gastos legales"
"ec52022815","52022815","Services contracted to third parties","False","expense","Servicios contratados a terceros"
"ec52022816","52022816","Other expenses","False","expense","Otros gastos"
"ec52022817","52022817","Waste collection","False","expense","Recolección residuos"
"ec5202281801","5202281801","Supplies","False","expense","Gnd suministros"
"ec5202281802","5202281802","Expenses for management and entertainment of customers and employees","False","expense","Gnd gastos de gestion y agasajos a clientes y empleados"
"ec5202281803","5202281803","Retentions assumed","False","expense","Gnd retenciones asumidas"
"ec5202281804","5202281804","Technical support","False","expense","Gnd soporte tecnico"
"ec5202281805","5202281805","Non-deductible mobilization","False","expense","Gnd movilización no deducible"
"ec5202281806","5202281806","Taxes, interest and penalties","False","expense","Gnd impuestos, intereses y multas"
"ec5202281807","5202281807","Gnd Foreign Currency Outflow Tax (ISD)","False","expense","Gnd Impuesto Salida de Divisas (ISD)"
"ec5202281808","5202281808","Other non-deductible expenses","False","expense","Gnd otros gastos no deducibles"
"ec520301","520301","Interests","False","expense","Intereses"
"ec520302","520302","Commissions","False","expense","Comisiones"
"ec520303","520303","Asset financing expenses","False","expense","Gastos de financiamiento de activos"
"ec520304","520304","Difference in exchange","False","expense","Diferencia en cambio"
"ec520305","520305","Other financial expenses","False","expense","Otros gastos financieros"
"ec520401","520401","Loss on investments in associates / subsidiaries and others","False","expense","Perdida en inversiones en asociadas / subsidiarias y otras"
"ec_early_pay_discount_gain","52040201","(-) Early Cash Discount Gain from purchases","False","expense","(-) Descuento pronto pago en compras de servicios"
"ec_expense_cash_difference","52040202","(-) Expenses for Cash Differences","False","expense","(-) Egresos por diferencias en efectivo"
"ec_expense_others","52040203","Otros gastos","False",expense,"Other expenses"

```

## File: data\template\account.fiscal.position-ec.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@es"
"fp_local","10","National regime","1","1","base.ec","","","Régimen nacional"
"fp_foreign","20","Foreign regime","1","1","","tax_vat_15_411_goods","tax_vat_417","Régimen extranjero"
"","","","","","","tax_vat_15_411_services","tax_vat_417"
"","","","","","","tax_vat_411_goods","tax_vat_417"
"","","","","","","tax_vat_411_services","tax_vat_417"
"","","","","","","tax_vat_415_goods","tax_vat_417"
"","","","","","","tax_vat_415_services","tax_vat_417"

```

## File: data\template\account.group-ec.csv

```csv
"id","code_prefix_start","name","parent_id","name@es"
"ec1","1","Active","","Activo"
"ec11","11","Current assets","ec1","Activo corriente"
"ec1101","1101","Cash and cash equivalents","ec11","Efectivo y equivalentes al efectivo"
"ec110101","110101","Cash","ec1101","Efectivo"
"ec110102","110102","Banks","ec1101","Bancos"
"ec110103","110103","Transfers","ec1101","Transferencias"
"ec110104","110104","Securities in custody","ec1101","Valores en custodia"
"ec1102","1102","Financial assets","ec11","Activos financieros"
"ec110205","110205","Unrelated notes and accounts receivable","ec1102","Documentos y cuentas por cobrar cliente no relacionados"
"ec11020501","11020501","Non-interest bearing notes and accounts receivable from unrelated ordinary activities interest-bearing","ec110205","Documentos y cuentas por cobrar cliente no relacionados de actividades ordinarias que no generen intereses"
"ec110206","110206","Related notes and accounts receivable","ec1102","Documentos y cuentas por cobrar cliente relacionados"
"ec110207","110207","Accounts receivable from employees","ec1102","Cuentas por cobrar empleados"
"ec1103","1103","Inventories","ec11","Inventarios"
"ec1104","1104","Services and other prepayments","ec11","Servicios y otros pagos anticipados"
"ec110404","110404","Other advances delivered","ec1104","Otros anticipos entregados"
"ec1105","1105","Current tax assets","ec11","Activos por impuestos corrientes"
"ec110501","110501","Tax credit in favor of the company (vat)","ec1105","Credito tributario a favor de la empresa(IVA)"
"ec110502","110502","Tax credit of the company","ec1105","Crédito Tributario de la Empresa"
"ec12","12","Non-current assets","ec1","Activo no corriente"
"ec1201","1201","Property, plant and equipment","ec12","Propiedades, planta y equipo"
"ec120112","120112","Accumulated depreciation property, plant and equipment","ec1201","Depreciacion acumulada propiedades, planta y equipo"
"ec120113","120113","Accumulated impairment of property, plant and equipment","ec1201","Deterioro acumulado de propiedades, planta y equipo"
"ec1202","1202","Investment properties","ec12","Propiedades de inversion"
"ec1203","1203","Biological assets","ec12","Activos biologicos"
"ec1204","1204","Intangible assets","ec12","Activo intangible"
"ec1205","1205","Deferred tax assets","ec12","Activos por impuestos diferidos"
"ec1206","1206","Non-current financial assets","ec12","Activos financieros no corrientes"
"ec1207","1207","Other non-current assets","ec12","Otros activos no corrientes"
"ec2","2","Passive","","Pasivo"
"ec21","21","Current liabilities","ec2","Pasivo corriente"
"ec2103","2103","Accounts and notes payable","ec21","Cuentas y documentos por pagar"
"ec2104","2104","Obligations with financial institutions","ec21","Obligaciones con instituciones financieras"
"ec2105","2105","Provisions","ec21","Provisiones"
"ec210501","210501","Local provisions","ec2105","Provisiones locales"
"ec21050101","21050101","Provisions in favor of employees","ec210501","Provisiones a favor de los empleados"
"ec2107","2107","Other current liabilities","ec21","Otras obligaciones corrientes"
"ec210701","210701","Other current obligations with the tax authorities","ec2107","Otras obligaciones corrientes con la administracion tributaria"
"ec210703","210703","Other current obligations with the iess","ec2107","Otras obligaciones corrientes con el iess"
"ec210704","210704","Other current employee benefit obligations","ec2107","Otras obligaciones corrientes por beneficios de ley a empleados"
"ec2112","2112","Current portion of employee benefit provisions","ec21","Porcion corriente de provisiones por beneficios a empleados"
"ec2113","2113","Other current liabilities","ec21","Otros pasivos corrientes"
"ec2114","2114","Tax obligations","ec21","Obligaciones tributarias"
"ec211401","211401","Taxes","ec2114","Impuestos"
"ec21140101","21140101","VAT collected","ec211401","IVA cobrado"
"ec21140102","21140102","Withholding taxes","ec211401","Retenciones de la fuente"
"ec21140103","21140103","VAT withholdings","ec211401","Retenciones de IVA"
"ec22","22","Non-current liabilities","ec2","Pasivo no corriente"
"ec2202","2202","Accounts and notes payable","ec22","Cuentas y documentos por pagar"
"ec2203","2203","Obligations with financial institutions","ec22","Obligaciones con instituciones financieras"
"ec2204","2204","Sundry/related accounts payable","ec22","Cuentas por pagar diversas/relacionadas"
"ec2207","2207","Provisions for employee benefits","ec22","Provisiones por beneficios a empleados"
"ec2209","2209","Deferred liabilities","ec22","Pasivo diferido"
"ec3","3","Net worth","","Patrimonio neto"
"ec31","31","Capital","ec3","Capital"
"ec34","34","Reservations","ec3","Reservas"
"ec35","35","Other comprehensive income","ec3","Otros resultados integrales"
"ec36","36","Accumulated results","ec3","Resultados acumulados"
"ec37","37","Results for the year","ec3","Resultados del ejercicio"
"ec4","4","Revenues","","Ingresos"
"ec41","41","Income from ordinary activities","ec4","Ingresos de actividades ordinarias"
"ec4101","4101","Sale of goods","ec41","Venta de bienes"
"ec4102","4102","Provision of services","ec41","Prestacion de servicios"
"ec43","43","Other income","ec4","Otros ingresos"
"ec4305","4305","Other rentals","ec43","Otras rentas"
"ec5","5","Expenses","","Egresos"
"ec51","51","Cost of sales and production","ec5","Costo de ventas y produccion"
"ec5101","5101","Materials used or products sold","ec51","Materiales utilizados o productos vendidos"
"ec5102","5102","Direct labor","ec51","Mano de obra directa"
"ec510201","510201","Wages, salaries and other remuneration","ec5102","Sueldos, salarios y demas remuneraciones"
"ec510202","510202","Employee benefit plan expense","ec5102","Gasto planes de beneficios a empleados"
"ec51020205","51020205","Other employee benefit plan expenses","ec510202","Otros gastos de planes de beneficios a empleados"
"ec510203","510203","Social security contributions","ec5102","Aportes a la seguridad social"
"ec510204","510204","Social benefits and indemnities","ec5102","Beneficios sociales e indemnizaciones"
"ec5103","5103","Indirect labor","ec51","Mano de obra indirecta"
"ec510301","510301","Wages, salaries and other remuneration","ec5103","Sueldos, salarios y demas remuneraciones"
"ec510302","510302","Employee benefit plan expense","ec5103","Gasto planes de beneficios a empleados"
"ec51030205","51030205","Other employee benefit plan expenses","ec510302","Otros gastos de planes de beneficios a empleados"
"ec510303","510303","Social security contributions (including reserve fund)","ec5103","Aportes a la seguridad social (incluido fondo de reserva)"
"ec510304","510304","Social benefits and indemnities","ec5103","Beneficios sociales e indemnizaciones"
"ec5104","5104","Other indirect manufacturing costs","ec51","Otros costos indirectos de fabricacion"
"ec510401","510401","Depreciation of property, plant and equipment","ec5104","Depreciacion propiedades, planta y equipo"
"ec510403","510403","Impairment of property, plant and equipment","ec5104","Deterioro de propiedad, planta y equipo"
"ec510408","510408","Other production costs","ec5104","Otros costos de produccion"
"ec52","52","Expenses","ec5","Gastos"
"ec5201","5201","Cost of sales","ec52","Gastos de ventas"
"ec520101","520101","Wages, salaries and other remuneration","ec5201","Sueldos, salarios y demas remuneraciones"
"ec520102","520102","Social security contributions (including reserve fund)","ec5201","Aportes a la seguridad social (incluido fondo de reserva)"
"ec520103","520103","Social benefits and indemnities","ec5201","Beneficios sociales e indemnizaciones"
"ec520104","520104","Employee benefit plan expense","ec5201","Gasto planes de beneficios a empleados"
"ec52010405","52010405","Other employee benefit plan expenses","ec520104","Otros gastos de planes de beneficios a empleados"
"ec520118","520118","Water, power, electricity and telecommunications","ec5201","Agua, energia, luz, y telecomunicaciones"
"ec520121","520121","Depreciation","ec5201","Depreciaciones"
"ec52012101","52012101","Property, plant and equipment","ec520121","Propiedades, planta y equipo"
"ec520122","520122","Amortizations","ec5201","Amortizaciones"
"ec520123","520123","Impairment expense","ec5201","Gasto deterioro"
"ec52012301","52012301","Property, plant and equipment","ec520123","Propiedades, planta y equipo"
"ec520127","520127","Other expenses","ec5201","Otros gastos"
"ec5202","5202","Administrative expenses","ec52","Gastos administrativos"
"ec520201","520201","Wages, salaries and other remuneration","ec5202","Sueldos, salarios y demas remuneraciones"
"ec520202","520202","Social security contributions","ec5202","Aportes a la seguridad social"
"ec520203","520203","Social benefits and indemnities","ec5202","Beneficios sociales e indemnizaciones"
"ec520204","520204","Employee benefit plan expense","ec5202","Gasto planes de beneficios a empleados"
"ec520218","520218","Water, power, electricity and telecommunications","ec5202","Agua, energia, luz, y telecomunicaciones"
"ec520220","520220","Taxes, contributions and other","ec5202","Impuestos, contribuciones y otros"
"ec520221","520221","Depreciation","ec5202","Depreciaciones"
"ec52022101","52022101","Property, plant and equipment","ec520221","Propiedades, planta y equipo"
"ec520222","520222","Amortizations","ec5202","Amortizaciones"
"ec520223","520223","Impairment expense","ec5202","Gasto deterioro"
"ec52022301","52022301","Property, plant and equipment","ec520223","Propiedades, planta y equipo"
"ec520228","520228","Other expenses","ec5202","Otros gastos"
"ec52022818","52022818","Non-deductible expenses","ec520228","Gnd gastos no deducibles"
"ec5203","5203","Financial expenses","ec52","Gastos financieros"
"ec5204","5204","Other expenses","ec52","Otros gastos"
"ec520402","520402","Others","ec5204","Otros"

```

## File: data\template\account.tax-ec.csv

```csv
"id","name","type_tax_use","amount_type","sequence","amount","description","l10n_ec_code_base","l10n_ec_code_applied","tax_group_id","l10n_ec_code_ats","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","name@es","description@es","invoice_label@es","invoice_label","active"
"tax_vat_15_411_goods","VAT 15% G","sale","percent","5","15","VAT 15% (411, Goods)","411","421","tax_group_vat_15","","100","base","invoice","+401 (Reporte 104)||+411 (Reporte 104)","","IVA 15% (411, B)","IVA 15% (411, Bienes)","IVA 15%","VAT 15% G","True"
"","","","","","","","","","","","100","tax","invoice","+421 (Reporte 104)","ec_sale_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-411 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-421 (Reporte 104)","ec_sale_vat","","","","",""
"tax_vat_15_411_services","VAT 15% S","sale","percent","5","15","VAT 15% (411, Services)","411","421","tax_group_vat_15","","","base","invoice","+401 (Reporte 104)||+411 (Reporte 104)","","IVA 15% (411, S)","IVA 15% (411, Servicios)","IVA 15%","VAT 15% S","True"
"","","","","","","","","","","","","tax","invoice","+421 (Reporte 104)","ec_sale_vat","","","","",""
"","","","","","","","","","","","","base","refund","-411 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-421 (Reporte 104)","ec_sale_vat","","","","",""
"tax_vat_15_412","VAT 15% assets","sale","percent","5","15","VAT 15% (412, Assets)","412","422","tax_group_vat_15","","","base","invoice","+402 (Reporte 104)||+412 (Reporte 104)","","IVA 15% (412, A)","IVA 15% (412, Activos)","IVA 15%","VAT 15% assets","True"
"","","","","","","","","","","","","tax","invoice","+422 (Reporte 104)","ec_sale_vat","","","","",""
"","","","","","","","","","","","","base","refund","-412 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-422 (Reporte 104)","ec_sale_vat","","","","",""
"tax_vat_411_goods","VAT 12% G","sale","percent","20","12","VAT 12% (411, Goods)","411","421","tax_group_vat_12","","100","base","invoice","+401 (Reporte 104)||+411 (Reporte 104)","","IVA 12% (411, B)","IVA 12% (411, Bienes)","IVA 12%","VAT 12% G","False"
"","","","","","","","","","","","100","tax","invoice","+421 (Reporte 104)","ec_sale_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-411 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-421 (Reporte 104)","ec_sale_vat","","","","",""
"tax_vat_411_services","VAT 12% S","sale","percent","20","12","VAT 12% (411, Services)","411","421","tax_group_vat_12","","","base","invoice","+401 (Reporte 104)||+411 (Reporte 104)","","IVA 12% (411, S)","IVA 12% (411, Servicios)","IVA 12%","VAT 12% S","False"
"","","","","","","","","","","","","tax","invoice","+421 (Reporte 104)","ec_sale_vat","","","","",""
"","","","","","","","","","","","","base","refund","-411 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-421 (Reporte 104)","ec_sale_vat","","","","",""
"tax_vat_412","VAT 12% assets","sale","percent","20","12","VAT 12% (412, Assets)","412","422","tax_group_vat_12","","","base","invoice","+402 (Reporte 104)||+412 (Reporte 104)","","IVA 12% (412, A)","IVA 12% (412, Activos)","IVA 12%","VAT 12% assets","False"
"","","","","","","","","","","","","tax","invoice","+422 (Reporte 104)","ec_sale_vat","","","","",""
"","","","","","","","","","","","","base","refund","-412 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-422 (Reporte 104)","ec_sale_vat","","","","",""
"tax_vat_05_411_goods","VAT 5% G","sale","percent","25","5","VAT 5% (435, Goods)","435","445","tax_group_vat_05","","100","base","invoice","+425 (Reporte 104)||+435 (Reporte 104)","","IVA 5% (435, B)","IVA 5% (435, Bienes)","IVA 5%","VAT 5% G","False"
"","","","","","","","","","","","100","tax","invoice","+445 (Reporte 104)","ec_sale_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-435 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-445 (Reporte 104)","ec_sale_vat","","","","",""
"tax_vat_05_411_services","VAT 5% S","sale","percent","25","5","VAT 5% (411, Services)","411","421","tax_group_vat_05","","","base","invoice","+401 (Reporte 104)||+411 (Reporte 104)","","IVA 5% (411, S)","IVA 5% (411, Servicios)","IVA 5%","VAT 5% S","False"
"","","","","","","","","","","","","tax","invoice","+421 (Reporte 104)","ec_sale_vat","","","","",""
"","","","","","","","","","","","","base","refund","-411 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-421 (Reporte 104)","ec_sale_vat","","","","",""
"tax_vat_05_412","VAT 5% assets","sale","percent","25","5","VAT 5% (412, Assets)","412","422","tax_group_vat_05","","","base","invoice","+402 (Reporte 104)||+412 (Reporte 104)","","IVA 5% (412, A)","IVA 5% (412, Activos)","IVA 5%","VAT 5% assets","False"
"","","","","","","","","","","","","tax","invoice","+422 (Reporte 104)","ec_sale_vat","","","","",""
"","","","","","","","","","","","","base","refund","-412 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-422 (Reporte 104)","ec_sale_vat","","","","",""
"tax_vat_415_goods","VAT 0% G","sale","percent","40","0","VAT 0% (415, Goods)","415","","tax_group_vat0","","100","base","invoice","+405 (Reporte 104)||+415 (Reporte 104)","","IVA 0% (415, B)","IVA 0% (415, Bienes)","IVA 0%","VAT 0% G","True"
"","","","","","","","","","","","100","tax","invoice","","ec_sale_vat_zero","","","","",""
"","","","","","","","","","","","100","base","refund","-415 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","","ec_sale_vat_zero","","","","",""
"tax_vat_415_services","VAT 0% S","sale","percent","50","0","VAT 0% (415, Services)","415","","tax_group_vat0","","","base","invoice","+405 (Reporte 104)||+415 (Reporte 104)","","IVA 0% (415, S)","IVA 0% (415, Servicios)","IVA 0%","VAT 0% S","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-415 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_vat_zero","","","","",""
"tax_vat_416","VAT 0% assets","sale","percent","60","0","VAT 0% (416, Assets)","416","","tax_group_vat0","","","base","invoice","+406 (Reporte 104)||+416 (Reporte 104)","","IVA 0% (416, A)","IVA 0% (416, Activos)","IVA 0%","VAT 0% assets","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-416 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_vat_zero","","","","",""
"tax_vat_413","VAT 0% NTC","sale","percent","70","0","VAT 0% (413, No Tax Credit)","413","","tax_group_vat0","","","base","invoice","+403 (Reporte 104)||+413 (Reporte 104)","","IVA 0% (413, Sin Créd Trib)","IVA 0% (413, Sin Crédito Tributario)","IVA 0%","VAT 0% NTC","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-413 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_vat_zero","","","","",""
"tax_vat_414","VAT 0% assets NTC","sale","percent","80","0","VAT 0% (414, Assets Without Tax Credit)","414","","tax_group_vat0","","","base","invoice","+404 (Reporte 104)||+414 (Reporte 104)","","IVA 0% (414, A Sin Créd Trib)","IVA 0% (414, Activos Sin Crédito Tributario)","IVA 0%","VAT 0% assets NTC","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-414 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_vat_zero","","","","",""
"tax_vat_417","VAT 0% EX G","sale","percent","90","0","VAT 0% (417, Export Goods)","417","","tax_group_vat0","","","base","invoice","+407 (Reporte 104)||+417 (Reporte 104)","","IVA 0% (417, Expo B)","IVA 0% (417, Exportación Bienes)","IVA 0%","VAT 0% EX G","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_vat_goods_exports","","","","",""
"","","","","","","","","","","","","base","refund","-417 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_vat_goods_exports","","","","",""
"tax_vat_418","VAT 0% EX S","sale","percent","100","0","VAT 0% (418, Export Services)","418","","tax_group_vat0","","","base","invoice","+408 (Reporte 104)||+418 (Reporte 104)","","IVA 0% (418, Expo S)","IVA 0% (418, Exportación Servicios)","IVA 0%","VAT 0% EX S","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_vat_services_exports","","","","",""
"","","","","","","","","","","","","base","refund","-418 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_vat_services_exports","","","","",""
"tax_vat_441","VAT 0% EX","sale","percent","110","0","VAT 0% (441, Exempt/Non-Object)","441","","tax_group_vat_exempt","","","base","invoice","+431 (Reporte 104)||+441 (Reporte 104)","","IVA 0% (441, No Obj/Exen)","IVA 0% (441, No Objeto/Exentas)","IVA EXENTO","VAT 0% EX","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-441 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_vat_zero","","","","",""
"tax_vat_15_444","VAT 15% REIMB","sale","percent","115","15","VAT 15% (444, Reimbursement)","444","454","tax_group_vat_15","","","base","invoice","+434 (Reporte 104)||+444 (Reporte 104)","","IVA 15% (444, Reemb)","IVA 15% (444, Reembolsos)","IVA 15%","VAT 15% REIMB","True"
"","","","","","","","","","","","","tax","invoice","+454 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-444 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-454 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_444","VAT 12% REIMB","sale","percent","125","12","VAT 12% (444, Reimbursement)","444","454","tax_group_vat_12","","","base","invoice","+434 (Reporte 104)||+444 (Reporte 104)","","IVA 12% (444, Reemb)","IVA 12% (444, Reembolsos)","IVA 12%","VAT 12% REIMB","False"
"","","","","","","","","","","","","tax","invoice","+454 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-444 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-454 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_05_444","VAT 5% REIMB","sale","percent","128","5","VAT 5% (444, Reimbursement)","444","454","tax_group_vat_05","","","base","invoice","+434 (Reporte 104)||+444 (Reporte 104)","","IVA 5% (444, Reemb)","IVA 5% (444, Reembolsos)","IVA 5%","VAT 5% REIMB","False"
"","","","","","","","","","","","","tax","invoice","+454 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-444 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-454 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_444_zero_vat","VAT 0% REIMB","sale","percent","130","0","VAT 0% (444, Reimbursement)","444","454","tax_group_vat0","","","base","invoice","+434 (Reporte 104)||+444 (Reporte 104)","","IVA 0% (444, Reemb)","IVA 0% (444, Reembolsos)","IVA 0%","VAT 0% REIMB","True"
"","","","","","","","","","","","","tax","invoice","+454 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-444 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-454 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_444_not_charged_vat","VAT 0% N S REIMB","sale","percent","140","0","VAT 0% Non-Subject (444, Reimbursement)","444","454","tax_group_vat_not_charged","","","base","invoice","+434 (Reporte 104)||+444 (Reporte 104)","","No Obj IVA 0% (444, Reemb)","No Objeto IVA 0% (444, Reembolsos)","IVA 0%","VAT 0% N S REIMB","True"
"","","","","","","","","","","","","tax","invoice","+454 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-444 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-454 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_444_not_exempt_vat","VAT 0% EXEMPT REIMB","sale","percent","150","0","VAT 0% Exempt (444, Reimbursement)","444","454","tax_group_vat_exempt","","","base","invoice","+434 (Reporte 104)||+444 (Reporte 104)","","Exen IVA 0% (444, Reemb)","Exento IVA 0% (444, Reembolsos)","IVA EXENTO","VAT 0% EXEMPT REIMB","True"
"","","","","","","","","","","","","tax","invoice","+454 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-444 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-454 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_15_510_sup_01","VAT 15% 510 01","purchase","percent","5","15","VAT 15% (510, 01 VAT Credit)","510","520","tax_group_vat_15","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 15% (510, 01 Créd)","IVA 15% (510, 01 Crédito IVA)","IVA 15%","VAT 15% 510 01","True"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_15_510_sup_05","VAT 15% 510 05 S T E IR","purchase","percent","5","15","VAT 15% (510, 05 Settlement Trip Expense IR)","510","520","tax_group_vat_15","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 15% (510, 05 Liq. V G IR)","IVA 15% (510, 05 Liq. Viaje Gasto IR)","IVA 15%","VAT 15% 510 05 S T E IR","True"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_15_510_sup_06","VAT 15% 510 06 I","purchase","percent","5","15","VAT 15% (510, 06 Inventory VAT Credit)","510","520","tax_group_vat_15","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 15% (510, 06 Inv Créd)","IVA 15% (510, 06 Inventario Crédito IVA)","IVA 15%","VAT 15% 510 06 I","True"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_15_510_sup_15","VAT 15% 511 15 D S","purchase","percent","5","15","VAT 15% (510, 15 Digital Services)","510","520","tax_group_vat_15","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 15% (510, 15 Serv Dig)","IVA 15% (510, 15 Servicios Digitales)","IVA 15%","VAT 15% 511 15 D S","True"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_15_511_sup_03","VAT 15% 511 03 assets","purchase","percent","6","15","VAT 15% (511, 03 VAT Credit Assets)","511","521","tax_group_vat_15","","","base","invoice","+501 (Reporte 104)||+511 (Reporte 104)","","IVA 15% (511, 03 A Créd)","IVA 15% (511, 03 Activos Crédito IVA)","IVA 15%","VAT 15% 511 03 assets","True"
"","","","","","","","","","","","","tax","invoice","+521 (Reporte 104)","ec_purchase_vat_assets","","","","",""
"","","","","","","","","","","","","base","refund","-511 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-521 (Reporte 104)","ec_purchase_vat_assets","","","","",""
"tax_vat_15_512_sup_04","VAT 15% 512 04 Assets IR C","purchase","percent","6","15","VAT 15% (512, 04 Assets IR Cost)","512","522","tax_group_vat_15","","","base","invoice","+502 (Reporte 104)||+512 (Reporte 104)","","IVA 15% (512, 04 Act. Costo IR)","IVA 15% (512, 04 Activos Costo IR)","IVA 15%","VAT 15% 512 04 Assets IR C","True"
"","","","","","","","","","","","","tax","invoice","+522 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","base","refund","-512 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-522 (Reporte 104)","","","","","",""
"tax_vat_15_512_sup_05","VAT 15% 512 05 S T E IR","purchase","percent","6","15","VAT 15% (512, 05 Settlement Trip Expense IR)","512","522","tax_group_vat_15","","","base","invoice","+502 (Reporte 104)||+512 (Reporte 104)","","IVA 15% (512, 05 Liq. Viaje G. IR)","IVA 15% (512, 05 Liq. Viaje Gasto IR)","IVA 15%","VAT 15% 512 05 S T E IR","True"
"","","","","","","","","","","","","tax","invoice","+522 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","base","refund","-512 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-522 (Reporte 104)","","","","","",""
"tax_vat_15_512_sup_07","VAT 15% 512 07 I C IR","purchase","percent","6","15","VAT 15% (512, 07 Inventory Cost IR)","512","522","tax_group_vat_15","","100","base","invoice","+502 (Reporte 104)||+512 (Reporte 104)","","IVA 15% (512, 07 Inv. Costo IR)","IVA 15% (512, 07 Inventario Costo IR)","IVA 15%","VAT 15% 512 07 I C IR","True"
"","","","","","","","","","","","100","tax","invoice","+522 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","base","refund","-512 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-522 (Reporte 104)","","","","","",""
"tax_vat_15_513_sup_01","VAT 15% 513 01","purchase","percent","6","15","VAT 15% (513, 01 VAT Credit)","513","523","tax_group_vat_15","","","base","invoice","+503 (Reporte 104)||+513 (Reporte 104)","","IVA 15% (513, 01 Créd)","IVA 15% (513, 01 Crédito IVA)","IVA 15%","VAT 15% 513 01","True"
"","","","","","","","","","","","","tax","invoice","+523 (Reporte 104)","ec_purchase_vat_service_imports","","","","",""
"","","","","","","","","","","","","base","refund","-513 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-523 (Reporte 104)","ec_purchase_vat_service_imports","","","","",""
"tax_vat_15_514_sup_06","VAT 15% 514 06 I","purchase","percent","6","15","VAT 15% (514, 06 Inventory VAT Credit)","514","524","tax_group_vat_15","","","base","invoice","+504 (Reporte 104)||+514 (Reporte 104)","","IVA 15% (514, 06 Inv. Créd)","IVA 15% (514, 06 Inventario Crédito IVA)","IVA 15%","VAT 15% 514 06 I","True"
"","","","","","","","","","","","","tax","invoice","+524 (Reporte 104)","ec_purchase_vat_goods_imports","","","","",""
"","","","","","","","","","","","","base","refund","-514 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-524 (Reporte 104)","ec_purchase_vat_goods_imports","","","","",""
"tax_vat_15_515_sup_03","VAT 15% 515 03 assets","purchase","percent","6","15","VAT 15% (515, 03 VAT Credit Assets)","515","525","tax_group_vat_15","","","base","invoice","+505 (Reporte 104)||+515 (Reporte 104)","","IVA 15% (515, 03 A. Créd)","IVA 15% (515, 03 Activos Crédito IVA)","IVA 15%","VAT 15% 515 03 assets","True"
"","","","","","","","","","","","","tax","invoice","+525 (Reporte 104)","ec_purchase_vat_assets_imports","","","","",""
"","","","","","","","","","","","","base","refund","-515 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-525 (Reporte 104)","ec_purchase_vat_assets_imports","","","","",""
"tax_vat_510_sup_01","VAT 12% 510 01","purchase","percent","9","12","VAT 12% (510, 01 VAT Credit)","510","520","tax_group_vat_12","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 12% (510, 01 Créd)","IVA 12% (510, 01 Crédito IVA)","IVA 12%","VAT 12% 510 01","False"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_510_sup_05","VAT 12% 510 05 S T E IR","purchase","percent","9","12","VAT 12% (510, 05 Settlement Trip Expense IR)","510","520","tax_group_vat_12","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 12% (510, 05 Liq. V G IR)","IVA 12% (510, 05 Liq. Viaje Gasto IR)","IVA 12%","VAT 12% 510 05 S T E IR","False"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_510_sup_06","VAT 12% 510 06 I","purchase","percent","9","12","VAT 12% (510, 06 Inventory VAT Credit)","510","520","tax_group_vat_12","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 12% (510, 06 Inv Créd)","IVA 12% (510, 06 Inventario Crédito IVA)","IVA 12%","VAT 12% 510 06 I","False"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_510_sup_15","VAT 12% 511 15 D S","purchase","percent","9","12","VAT 12% (510, 15 Digital Services)","510","520","tax_group_vat_12","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 12% (510, 15 Serv Dig)","IVA 12% (510, 15 Servicios Digitales)","IVA 12%","VAT 12% 511 15 D S","False"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_511_sup_03","VAT 12% 511 03 assets","purchase","percent","10","12","VAT 12% (511, 03 VAT Credit Assets)","511","521","tax_group_vat_12","","","base","invoice","+501 (Reporte 104)||+511 (Reporte 104)","","IVA 12% (511, 03 A Créd)","IVA 12% (511, 03 Activos Crédito IVA)","IVA 12%","VAT 12% 511 03 assets","False"
"","","","","","","","","","","","","tax","invoice","+521 (Reporte 104)","ec_purchase_vat_assets","","","","",""
"","","","","","","","","","","","","base","refund","-511 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-521 (Reporte 104)","ec_purchase_vat_assets","","","","",""
"tax_vat_512_sup_04","VAT 12% 512 04 Assets IR C","purchase","percent","10","12","VAT 12% (512, 04 Assets IR Cost)","512","522","tax_group_vat_12","","","base","invoice","+502 (Reporte 104)||+512 (Reporte 104)","","IVA 12% (512, 04 Act. Costo IR)","IVA 12% (512, 04 Activos Costo IR)","IVA 12%","VAT 12% 512 04 Assets IR C","False"
"","","","","","","","","","","","","tax","invoice","+522 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","base","refund","-512 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-522 (Reporte 104)","","","","","",""
"tax_vat_512_sup_05","VAT 12% 512 05 S T E IR","purchase","percent","10","12","VAT 12% (512, 05 Settlement Trip Expense IR)","512","522","tax_group_vat_12","","","base","invoice","+502 (Reporte 104)||+512 (Reporte 104)","","IVA 12% (512, 05 Liq. Viaje G. IR)","IVA 12% (512, 05 Liq. Viaje Gasto IR)","IVA 12%","VAT 12% 512 05 S T E IR","False"
"","","","","","","","","","","","","tax","invoice","+522 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","base","refund","-512 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-522 (Reporte 104)","","","","","",""
"tax_vat_512_sup_07","VAT 12% 512 07 I C IR","purchase","percent","10","12","VAT 12% (512, 07 Inventory Cost IR)","512","522","tax_group_vat_12","","100","base","invoice","+502 (Reporte 104)||+512 (Reporte 104)","","IVA 12% (512, 07 Inv. Costo IR)","IVA 12% (512, 07 Inventario Costo IR)","IVA 12%","VAT 12% 512 07 I C IR","False"
"","","","","","","","","","","","100","tax","invoice","+522 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","base","refund","-512 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-522 (Reporte 104)","","","","","",""
"tax_vat_513_sup_01","VAT 12% 513 01","purchase","percent","10","12","VAT 12% (513, 01 VAT Credit)","513","523","tax_group_vat_12","","","base","invoice","+503 (Reporte 104)||+513 (Reporte 104)","","IVA 12% (513, 01 Créd)","IVA 12% (513, 01 Crédito IVA)","IVA 12%","VAT 12% 513 01","False"
"","","","","","","","","","","","","tax","invoice","+523 (Reporte 104)","ec_purchase_vat_service_imports","","","","",""
"","","","","","","","","","","","","base","refund","-513 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-523 (Reporte 104)","ec_purchase_vat_service_imports","","","","",""
"tax_vat_514_sup_06","VAT 12% 514 06 I","purchase","percent","10","12","VAT 12% (514, 06 Inventory VAT Credit)","514","524","tax_group_vat_12","","","base","invoice","+504 (Reporte 104)||+514 (Reporte 104)","","IVA 12% (514, 06 Inv. Créd)","IVA 12% (514, 06 Inventario Crédito IVA)","IVA 12%","VAT 12% 514 06 I","False"
"","","","","","","","","","","","","tax","invoice","+524 (Reporte 104)","ec_purchase_vat_goods_imports","","","","",""
"","","","","","","","","","","","","base","refund","-514 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-524 (Reporte 104)","ec_purchase_vat_goods_imports","","","","",""
"tax_vat_515_sup_03","VAT 12% 515 03 assets","purchase","percent","10","12","VAT 12% (515, 03 VAT Credit Assets)","515","525","tax_group_vat_12","","","base","invoice","+505 (Reporte 104)||+515 (Reporte 104)","","IVA 12% (515, 03 A. Créd)","IVA 12% (515, 03 Activos Crédito IVA)","IVA 12%","VAT 12% 515 03 assets","False"
"","","","","","","","","","","","","tax","invoice","+525 (Reporte 104)","ec_purchase_vat_assets_imports","","","","",""
"","","","","","","","","","","","","base","refund","-515 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-525 (Reporte 104)","ec_purchase_vat_assets_imports","","","","",""
"tax_vat_05_510_sup_01","VAT 5% 550 01","purchase","percent","13","5","VAT 5% (550, 01 VAT Credit)","550","560","tax_group_vat_05","","100","base","invoice","+540 (Reporte 104)||+550 (Reporte 104)","","IVA 5% (550, 01 Créd)","IVA 5% (550, 01 Crédito IVA)","IVA 5%","VAT 5% 550 01","True"
"","","","","","","","","","","","100","tax","invoice","+560 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-550 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-560 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_05_510_sup_05","VAT 5% 510 05 S T E IR","purchase","percent","13","5","VAT 5% (510, 05 Settlement Trip Expense IR)","510","520","tax_group_vat_05","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 5% (510, 05 Liq. V G IR)","IVA 5% (510, 05 Liq. Viaje Gasto IR)","IVA 5%","VAT 5% 510 05 S T E IR","True"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_05_510_sup_06","VAT 5% 510 06 I","purchase","percent","13","5","VAT 5% (510, 06 Inventory VAT Credit)","510","520","tax_group_vat_05","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 5% (510, 06 Inv Créd)","IVA 5% (510, 06 Inventario Crédito IVA)","IVA 5%","VAT 5% 510 06 I","True"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_05_510_sup_15","VAT 5% 511 15 D S","purchase","percent","13","5","VAT 5% (510, 15 Digital Services)","510","520","tax_group_vat_05","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 5% (510, 15 Serv Dig)","IVA 5% (510, 15 Servicios Digitales)","IVA 5%","VAT 5% 511 15 D S","True"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_05_511_sup_03","VAT 5% 511 03 assets","purchase","percent","14","5","VAT 5% (511, 03 VAT Credit Assets)","511","521","tax_group_vat_05","","","base","invoice","+501 (Reporte 104)||+511 (Reporte 104)","","IVA 5% (511, 03 A Créd)","IVA 5% (511, 03 Activos Crédito IVA)","IVA 5%","VAT 5% 511 03 assets","True"
"","","","","","","","","","","","","tax","invoice","+521 (Reporte 104)","ec_purchase_vat_assets","","","","",""
"","","","","","","","","","","","","base","refund","-511 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-521 (Reporte 104)","ec_purchase_vat_assets","","","","",""
"tax_vat_05_512_sup_04","VAT 5% 512 04 Assets IR C","purchase","percent","14","5","VAT 5% (512, 04 Assets IR Cost)","512","522","tax_group_vat_05","","","base","invoice","+502 (Reporte 104)||+512 (Reporte 104)","","IVA 5% (512, 04 Act. Costo IR)","IVA 5% (512, 04 Activos Costo IR)","IVA 5%","VAT 5% 512 04 Assets IR C","True"
"","","","","","","","","","","","","tax","invoice","+522 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","base","refund","-512 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-522 (Reporte 104)","","","","","",""
"tax_vat_05_512_sup_05","VAT 5% 512 05 S T E IR","purchase","percent","14","5","VAT 5% (512, 05 Settlement Trip Expense IR)","512","522","tax_group_vat_05","","","base","invoice","+502 (Reporte 104)||+512 (Reporte 104)","","IVA 5% (512, 05 Liq. Viaje G. IR)","IVA 5% (512, 05 Liq. Viaje Gasto IR)","IVA 5%","VAT 5% 512 05 S T E IR","True"
"","","","","","","","","","","","","tax","invoice","+522 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","base","refund","-512 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-522 (Reporte 104)","","","","","",""
"tax_vat_05_512_sup_07","VAT 5% 512 07 I C IR","purchase","percent","14","5","VAT 5% (512, 07 Inventory Cost IR)","512","522","tax_group_vat_05","","100","base","invoice","+502 (Reporte 104)||+512 (Reporte 104)","","IVA 5% (512, 07 Inv. Costo IR)","IVA 5% (512, 07 Inventario Costo IR)","IVA 5%","VAT 5% 512 07 I C IR","True"
"","","","","","","","","","","","100","tax","invoice","+522 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","base","refund","-512 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-522 (Reporte 104)","","","","","",""
"tax_vat_05_513_sup_01","VAT 5% 513 01","purchase","percent","14","5","VAT 5% (513, 01 VAT Credit)","513","523","tax_group_vat_05","","","base","invoice","+503 (Reporte 104)||+513 (Reporte 104)","","IVA 5% (513, 01 Créd)","IVA 5% (513, 01 Crédito IVA)","IVA 5%","VAT 5% 513 01","True"
"","","","","","","","","","","","","tax","invoice","+523 (Reporte 104)","ec_purchase_vat_service_imports","","","","",""
"","","","","","","","","","","","","base","refund","-513 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-523 (Reporte 104)","ec_purchase_vat_service_imports","","","","",""
"tax_vat_05_514_sup_06","VAT 5% 514 06 I","purchase","percent","14","5","VAT 5% (514, 06 Inventory VAT Credit)","514","524","tax_group_vat_05","","","base","invoice","+504 (Reporte 104)||+514 (Reporte 104)","","IVA 5% (514, 06 Inv. Créd)","IVA 5% (514, 06 Inventario Crédito IVA)","IVA 5%","VAT 5% 514 06 I","True"
"","","","","","","","","","","","","tax","invoice","+524 (Reporte 104)","ec_purchase_vat_goods_imports","","","","",""
"","","","","","","","","","","","","base","refund","-514 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-524 (Reporte 104)","ec_purchase_vat_goods_imports","","","","",""
"tax_vat_05_515_sup_03","VAT 5% 515 03 assets","purchase","percent","14","5","VAT 5% (515, 03 VAT Credit Assets)","515","525","tax_group_vat_05","","","base","invoice","+505 (Reporte 104)||+515 (Reporte 104)","","IVA 5% (515, 03 A. Créd)","IVA 5% (515, 03 Activos Crédito IVA)","IVA 5%","VAT 5% 515 03 assets","True"
"","","","","","","","","","","","","tax","invoice","+525 (Reporte 104)","ec_purchase_vat_assets_imports","","","","",""
"","","","","","","","","","","","","base","refund","-515 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-525 (Reporte 104)","ec_purchase_vat_assets_imports","","","","",""
"tax_vat_516_sup_07","VAT 0% 516 07 I C IR","purchase","percent","20","0","VAT 0% (516, 07 Inventory Cost IR)","516","","tax_group_vat0","","","base","invoice","+506 (Reporte 104)||+516 (Reporte 104)","","IVA 0% (516, 07 Inv. Costo IR)","IVA 0% (516, 07 Inventario Costo IR)","IVA 0%","VAT 0% 516 07 I C IR","True"
"","","","","","","","","","","","","tax","invoice","","ec_purchase_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-516 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_purchase_vat_zero","","","","",""
"tax_vat_517_sup_02","VAT 0% 517 02 IR C","purchase","percent","20","0","VAT 0% (517, 02 IR Cost)","517","","tax_group_vat0","","","base","invoice","+507 (Reporte 104)||+517 (Reporte 104)","","IVA 0% (517, 02 Costo IR)","IVA 0% (517, 02 Costo IR)","IVA 0%","VAT 0% 517 02 IR C","True"
"","","","","","","","","","","","","tax","invoice","","ec_purchase_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-517 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_purchase_vat_zero","","","","",""
"tax_vat_517_sup_04","VAT 0% 517 04 A IR C","purchase","percent","20","0","VAT 0% (517, 04 Assets IR Cost)","517","","tax_group_vat0","","","base","invoice","+507 (Reporte 104)||+517 (Reporte 104)","","IVA 0% (517, 04 A. Costo IR)","IVA 0% (517, 04 Activos Costo IR)","IVA 0%","VAT 0% 517 04 A IR C","True"
"","","","","","","","","","","","","tax","invoice","","ec_purchase_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-517 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_purchase_vat_zero","","","","",""
"tax_vat_517_sup_05","VAT 0% 517 05 S T E IR","purchase","percent","20","0","VAT 0% (517, 05 Settlement Trip Expense IR)","517","","tax_group_vat0","","","base","invoice","+507 (Reporte 104)||+517 (Reporte 104)","","IVA 0% (517, 05 Liq. Viaje G. IR)","IVA 0% (517, 05 Liq. Viaje Gasto IR)","IVA 0%","VAT 0% 517 05 S T E IR","True"
"","","","","","","","","","","","","tax","invoice","","ec_purchase_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-517 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_purchase_vat_zero","","","","",""
"tax_vat_517_sup_07","VAT 0% 517 07 I C IR","purchase","percent","20","0","VAT 0% (517, 07 Inventory Cost IR)","517","","tax_group_vat0","","100","base","invoice","+507 (Reporte 104)||+517 (Reporte 104)","","IVA 0% (517, 07 Inv. Costo IR)","IVA 0% (517, 07 Inventario Costo IR)","IVA 0%","VAT 0% 517 07 I C IR","True"
"","","","","","","","","","","","100","tax","invoice","","ec_purchase_vat_zero","","","","",""
"","","","","","","","","","","","100","base","refund","-517 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","","ec_purchase_vat_zero","","","","",""
"tax_vat_517_sup_15","VAT 0% 517 15 D S","purchase","percent","20","0","VAT 0% (517, 15 Digital Services)","517","","tax_group_vat0","","","base","invoice","+507 (Reporte 104)||+517 (Reporte 104)","","IVA 0% (517, 15 Serv. Dig)","IVA 0% (517, 15 Servicios Digitales)","IVA 0%","VAT 0% 517 15 D S","True"
"","","","","","","","","","","","","tax","invoice","","ec_purchase_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-517 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","","ec_purchase_vat_zero","","","","",""
"tax_vat_518_sup_02","VAT 0% 518 02 IR C","purchase","percent","20","0","VAT 0% (518, 02 IR Cost)","518","","tax_group_vat0","","","base","invoice","+508 (Reporte 104)||+518 (Reporte 104)","","IVA 0% (518, 02 Costo IR)","IVA 0% (518, 02 Costo IR)","IVA 0%","VAT 0% 518 02 IR C","True"
"","","","","","","","","","","","","tax","invoice","","ec_purchase_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-518 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","","ec_purchase_vat_zero","","","","",""
"tax_vat_541_sup_02","VAT 0% 541 02 IR C N O","purchase","percent","30","0","VAT 0% (541, 02 IR Cost, No Object)","541","","tax_group_vat_not_charged","","100","base","invoice","+531 (Reporte 104)||+541 (Reporte 104)","","No Obj. IVA 0% (541, 02 Costo IR)","No Objeto IVA 0% (541, 02 Costo IR)","IVA 0%","VAT 0% 541 02 IR C N O","True"
"","","","","","","","","","","","","tax","invoice","","ec_purchase_vat_zero","","","","",""
"","","","","","","","","","","","100","base","refund","-541 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","","ec_purchase_vat_zero","","","","",""
"tax_vat_510_08_sup_01","VAT 8% 510 01","purchase","percent","50","8","VAT 8% (510, 01 VAT credit)","510","520","tax_group_vat_08","","100","base","invoice","+500 (Reporte 104)||+510 (Reporte 104)","","IVA 8% (510, 01 Créd)","IVA 8% (510, 01 Crédito IVA)","IVA 8%","VAT 8% 510 01","True"
"","","","","","","","","","","","100","tax","invoice","+520 (Reporte 104)","ec_purchase_vat","","","","",""
"","","","","","","","","","","","100","base","refund","-510 (Reporte 104)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-520 (Reporte 104)","ec_purchase_vat","","","","",""
"tax_vat_542_sup_02","VAT 0% EXEMPT 542 02 IR C","purchase","percent","40","0","VAT exempt 0% (542, 02 IR Cost)","542","","tax_group_vat_exempt","","","base","invoice","+532 (Reporte 104)||+542 (Reporte 104)","","Exento IVA 0% (542, 02 Costo IR)","Exento IVA 0% (542, 02 Costo IR)","IVA EXENTO","VAT 0% EXEMPT 542 02 IR C","True"
"","","","","","","","","","","","","tax","invoice","","ec_purchase_vat_zero","","","","",""
"","","","","","","","","","","","","base","refund","-542 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_purchase_vat_zero","","","","",""
"tax_vat_15_545_sup_08","VAT 15% 545 08 REIMB","purchase","percent","6","15","VAT 15% (545, 08 Reimbursement)","545","555","tax_group_vat_15","","","base","invoice","+535 (Reporte 104)||+545 (Reporte 104)","","IVA 15% (545, 08 Reemb)","IVA 15% (545, 08 Reembolso)","IVA 15%","VAT 15% 545 08 REIMB","True"
"","","","","","","","","","","","","tax","invoice","+555 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-545 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-555 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_545_sup_08","VAT 12% 545 08 REIMB","purchase","percent","12","12","VAT 12% (545, 08 Reimbursement)","545","555","tax_group_vat_12","","","base","invoice","+535 (Reporte 104)||+545 (Reporte 104)","","IVA 12% (545, 08 Reemb)","IVA 12% (545, 08 Reembolso)","IVA 12%","VAT 12% 545 08 REIMB","False"
"","","","","","","","","","","","","tax","invoice","+555 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-545 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-555 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_05_545_sup_08","VAT 5% 545 08 REIMB","purchase","percent","14","5","VAT 5% (545, 08 Reimbursement)","545","555","tax_group_vat_05","","","base","invoice","+535 (Reporte 104)||+545 (Reporte 104)","","IVA 5% (545, 08 Reemb)","IVA 5% (545, 08 Reembolso)","IVA 5%","VAT 5% 545 08 REIMB","True"
"","","","","","","","","","","","","tax","invoice","+555 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-545 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-555 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_545_sup_08_vat0","VAT 0% 545 08 REIMB","purchase","percent","20","0","VAT 0% (545, 08 Reimbursement)","545","555","tax_group_vat0","","","base","invoice","+535 (Reporte 104)||+545 (Reporte 104)","","IVA 0% (545, 08 Reemb)","IVA 0% (545, 08 Reembolso)","IVA 0%","VAT 0% 545 08 REIMB","True"
"","","","","","","","","","","","","tax","invoice","+555 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-545 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-555 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_545_sup_08_vat_exempt","VAT 0% Exempt 545 08 REIMB","purchase","percent","30","0","VAT 0% Exempt (545, 08 Reimbursement)","545","555","tax_group_vat_exempt","","","base","invoice","+535 (Reporte 104)||+545 (Reporte 104)","","Exento IVA 0% (545, 08 Reemb)","Exento IVA 0% (545, 08 Reembolso)","IVA EXENTO","VAT 0% Exempt 545 08 REIMB","True"
"","","","","","","","","","","","","tax","invoice","+555 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-545 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-555 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_545_sup_08_vat_not_charged","VAT 0% N S 545 08 REIMB","purchase","percent","40","0","VAT 0% Non-Subject (545, 08 Reimbursement)","545","555","tax_group_vat_not_charged","","","base","invoice","+535 (Reporte 104)||+545 (Reporte 104)","","No Obj. IVA 0% (545, 08 Reemb)","No Objeto IVA 0% (545, 08 Reembolso)","IVA 0%","VAT 0% N S 545 08 REIMB","True"
"","","","","","","","","","","","","tax","invoice","+555 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-545 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-555 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_15_545_sup_09","VAT 15% 545 09 REIMB S","purchase","percent","6","15","VAT 15% (545, 09 Reimbursement Sinister)","545","555","tax_group_vat_15","","","base","invoice","+535 (Reporte 104)||+545 (Reporte 104)","","IVA 15% (545, 09 Reemb Sinies)","IVA 15% (545, 09 Reembolso Siniestro)","IVA 15%","VAT 15% 545 09 REIMB S","True"
"","","","","","","","","","","","","tax","invoice","+555 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-545 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-555 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_545_sup_09","VAT 12% 545 09 REIMB S","purchase","percent","12","12","VAT 12% (545, 09 Reimbursement Sinister)","545","555","tax_group_vat_12","","","base","invoice","+535 (Reporte 104)||+545 (Reporte 104)","","IVA 12% (545, 09 Reemb Sinies)","IVA 12% (545, 09 Reembolso Siniestro)","IVA 12%","VAT 12% 545 09 REIMB S","False"
"","","","","","","","","","","","","tax","invoice","+555 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-545 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-555 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_vat_05_545_sup_09","VAT 5% 545 09 REIMB S","purchase","percent","14","5","VAT 5% (545, 09 Reimbursement Sinister)","545","555","tax_group_vat_05","","","base","invoice","+535 (Reporte 104)||+545 (Reporte 104)","","IVA 5% (545, 09 Reemb Sinies)","IVA 5% (545, 09 Reembolso Siniestro)","IVA 5%","VAT 5% 545 09 REIMB S","True"
"","","","","","","","","","","","","tax","invoice","+555 (Reporte 104)","ec_other_downpayments","","","","",""
"","","","","","","","","","","","","base","refund","-545 (Reporte 104)","","","","","",""
"","","","","","","","","","","","","tax","refund","-555 (Reporte 104)","ec_other_downpayments","","","","",""
"tax_withhold_profit_303","10% 303","none","percent","70","-10","303 10% Professional Fees and Other Payments for Services Related to the Professional Degree","303","353","tax_group_withhold_income_purchase","303","","base","invoice","+303 (Reporte 103)","","303 10% Honorarios Profesionales","303 10% Honorarios Profesionales y Demás Pagos por Servicios Relacionados con el Titulo Profesional","303","10% 303","True"
"","","","","","","","","","","","","tax","invoice","+353 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-303 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-353 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_304","8% 304","none","percent","70","-8","304 8% Services Predominantly Intellectual Services not Related to Professional Degree","304","354","tax_group_withhold_income_purchase","304","","base","invoice","+304 (Reporte 103)","","304 8% Servicios Intelecto","304 8% Servicios Predomina el Intelecto No Relacionados con el Titulo Profesional","304","8% 304","False"
"","","","","","","","","","","","","tax","invoice","+354 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","-304 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-354 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_304A","8% 304a","none","percent","70","-8","304a 8% Commissions and Other Payments for Services Predominantly Intellectually Unrelated to Professional Title","304","354","tax_group_withhold_income_purchase","304A","","base","invoice","+304 (Reporte 103)","","304A 8% Comisiones","304A 8% Comisiones y Demas Pagos por Servicios Predomina Intelecto No Relacionados con el Titulo Profesional","304","8% 304a","False"
"","","","","","","","","","","","","tax","invoice","+354 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","-304 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-354 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_304B","8% 304b","none","percent","70","-8","304b 8% Payments to Notaries and Property and Commercial Registrars for their Activities as Such","304","354","tax_group_withhold_income_purchase","304B","","base","invoice","+304 (Reporte 103)","","304B 8% Notarios","304B 8% Pagos a Notarios y Registradores de la Propiedad y Mercantil por sus Actividades Ejercidas Como Tales","304","8% 304b","False"
"","","","","","","","","","","","","tax","invoice","+354 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","-304 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-354 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_304C","8% 304c","none","percent","70","-8","304c 8% Payments to Athletes, Coaches, Referees, Members of the Coaching Staff for their Activities as Such","304","354","tax_group_withhold_income_purchase","304C","","base","invoice","+304 (Reporte 103)","","304C 8% Deportistas","304C 8% Pagos a Deportistas, Entrenadores, Arbitros, Miembros del Cuerpo Tecnico por sus Actividades Ejercidas Como Tales","304","8% 304c","True"
"","","","","","","","","","","","","tax","invoice","+354 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","-304 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-354 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_304D","8% 304d","none","percent","70","-8","304d 8% Payments to Artists for their Activities as Performers","304","354","tax_group_withhold_income_purchase","304D","","base","invoice","+304 (Reporte 103)","","304D 8% Artistas","304D 8% Pagos a Artistas por sus Actividades Ejercidas Como Tales","304","8% 304d","True"
"","","","","","","","","","","","","tax","invoice","+354 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","+304 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","+354 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_304E","8% 304e","none","percent","70","-8","304e 8% Fees and Other Payments for Teaching Services","304","354","tax_group_withhold_income_purchase","304E","","base","invoice","+304 (Reporte 103)","","304E 8% Docencia","304E 8% Honorarios y Demas Pagos por Servicios de Docencia","304","8% 304e","True"
"","","","","","","","","","","","","tax","invoice","+354 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","-304 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-354 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_307","2% 307","none","percent","70","-2","307 2% Services Predominantly Labor Force","307","357","tax_group_withhold_income_purchase","307","","base","invoice","+307 (Reporte 103)","","307 2% Servicios Mano de Obra","307 2% Servicios Predomina Mano de Obra","307","2% 307","True"
"","","","","","","","","","","","","tax","invoice","+357 (Reporte 103)","ret_ir_2x100","","","","",""
"","","","","","","","","","","","","base","refund","-307 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-357 (Reporte 103)","ret_ir_2x100","","","","",""
"tax_withhold_profit_308","10% 308","none","percent","70","-10","308 10% Use or Exploitation of Image or Reputation","308","358","tax_group_withhold_income_purchase","308","","base","invoice","+308 (Reporte 103)","","308 10% Imagen/Renombre","308 10% Utilizacion o Aprovechamiento de la Imagen o Renombre","308","10% 308","True"
"","","","","","","","","","","","","tax","invoice","+358 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-308 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-358 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_309","1.75% 309","none","percent","70","-1.75","309 1.75% Services Rendered by Media and Advertising Agencies","309","359","tax_group_withhold_income_purchase","309","","base","invoice","+309 (Reporte 103)","","309 1.75% Medios/Publicidad","309 1.75% Servicios Prestados por Medios de Comunicación y Agencias de Publicidad","309","1.75% 309","False"
"","","","","","","","","","","","","tax","invoice","+359 (Reporte 103)","ret_ir_1_75x100","","","","",""
"","","","","","","","","","","","","base","refund","-309 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-359 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_310","1% 310","none","percent","70","-1","310 1% Private Passenger Transportation Service or Public or Private Transportation of Cargo","310","360","tax_group_withhold_income_purchase","310","","base","invoice","+310 (Reporte 103)","","310 1% Transporte","310 1% Servicio de Transporte Privado de Pasajeros o Transporte Publico o Privado de Carga","310","1% 310","True"
"","","","","","","","","","","","","tax","invoice","+360 (Reporte 103)","ret_ir_1x100","","","","",""
"","","","","","","","","","","","","base","refund","-310 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-360 (Reporte 103)","ret_ir_1x100","","","","",""
"tax_withhold_profit_311","2% 311","none","percent","70","-2","311 2% For Payments Through Purchase Settlement (Cultural or Rustic Level)","311","361","tax_group_withhold_income_purchase","311","","base","invoice","+311 (Reporte 103)","","311 2% Compra","311 2% Por Pagos a Traves de Liquidacion de Compra (nivel cultural o rusticidad)","311","2% 311","True"
"","","","","","","","","","","","","tax","invoice","+361 (Reporte 103)","ret_ir_2x100","","","","",""
"","","","","","","","","","","","","base","refund","-311 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-361 (Reporte 103)","ret_ir_2x100","","","","",""
"tax_withhold_profit_312","1.75% 312","none","percent","70","-1.75","312 1.75% Transfer of Tangible Movable Property of a Tangible Nature","312","362","tax_group_withhold_income_purchase","312","","base","invoice","+312 (Reporte 103)","","312 1.75% Transferencia Bienes","312 1.75% Transferencia de Bienes Muebles de Naturaleza Corporal","312","1.75% 312","True"
"","","","","","","","","","","","","tax","invoice","+362 (Reporte 103)","ret_ir_1_75x100","","","","",""
"","","","","","","","","","","","","base","refund","-312 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-362 (Reporte 103)","ret_ir_1_75x100","","","","",""
"tax_withhold_profit_312A","1% 312a","none","percent","70","-1","312a 1% Purchases from the Producer: of Goods of Bio-aquatic, Forestry Origin and Those Described in Art.27.1 of LRTI","3120","3620","tax_group_withhold_income_purchase","312A","","base","invoice","+3120 (Reporte 103)","","312A 1% Compras al Productor","312A 1% Compras al Productor: de Bienes de Origen Bioacuático, Forestal y los Descritos el Art.27.1 de LRTI","3120","1% 312a","True"
"","","","","","","","","","","","","tax","invoice","+3620 (Reporte 103)","ret_ir_1x100","","","","",""
"","","","","","","","","","","","","base","refund","-3120 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-3620 (Reporte 103)","ret_ir_1x100","","","","",""
"tax_withhold_profit_314A","8% 314a","none","percent","70","-8","314a 8% Royalties From Franchises Under The Intellectual Property Law - Payment to Individuals","314","364","tax_group_withhold_income_purchase","314A","","base","invoice","+314 (Reporte 103)","","314A 8% Regalias Personas","314A 8% Regalias por Concepto de Franquicias de Acuerdo a Ley de Propiedad Intelectual - Pago a Personas Naturales","314","8% 314a","False"
"","","","","","","","","","","","","tax","invoice","+364 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","-314 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-364 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_314B","8% 314b","none","percent","70","-8","314b 8% Royalties, Copyrights, Trademarks, Patents and Similar in Accordance with the Intellectual Property Law - Payment to Natural Persons","314","364","tax_group_withhold_income_purchase","314B","","base","invoice","+314 (Reporte 103)","","314B 8% Derechos Autor","314B 8% Casales, Derechos de Autor, Marcas, Patentes y Similares de Acuerdo a Ley de Propiedad Intelectual – Pago a Personas Naturales","314","8% 314b","False"
"","","","","","","","","","","","","tax","invoice","+364 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","-314 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-364 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_314C","8% 314c","none","percent","70","-8","314c 8% Royalties from Franchises in Accordance with Intellectual Property Law - Payment to Corporations","314","364","tax_group_withhold_income_purchase","314C","","base","invoice","+314 (Reporte 103)","","314C 8% Regalias Sociedades","314C 8% Regalias por Concepto de Franquicias de Acuerdo a Ley de Propiedad Intelectual - Pago a Sociedades","314","8% 314c","False"
"","","","","","","","","","","","","tax","invoice","+364 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","-314 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-364 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_314D","8% 314d","none","percent","70","-8","314d 8% Royalties, Copyrights, Trademarks, Patents and Similar Under Intellectual Property law - Payment to Companies","314","364","tax_group_withhold_income_purchase","314D","","base","invoice","+314 (Reporte 103)","","314D 8% Derechos Sociedades","314D 8% Casales, Derechos de Autor, Marcas, Patentes y Similares de Acuerdo a Ley de Propiedad Intelectual – Pago a Sociedades","314","8% 314d","False"
"","","","","","","","","","","","","tax","invoice","+364 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","-314 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-364 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_319","1.75% 319","none","percent","70","-1.75","319 1.75% Lease Payments (Provided by Companies), Including the Option to Purchase","319","369","tax_group_withhold_income_purchase","319","","base","invoice","+319 (Reporte 103)","","319 1.75% Cuotas Arrendamiento","319 1.75% Cuotas de Arrendamiento Mercantil (Prestado por Sociedades), Inclusive la de Opción de Compra","319","1.75% 319","False"
"","","","","","","","","","","","","tax","invoice","+369 (Reporte 103)","ret_ir_1_75x100","","","","",""
"","","","","","","","","","","","","base","refund","-319 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-369 (Reporte 103)","ret_ir_1_75x100","","","","",""
"tax_withhold_profit_319_2","2% 319","none","percent","70","-2","319 2% Commercial Lease Fees (Loaned by Companies), Including the Purchase Option","319","369","tax_group_withhold_income_purchase","319","","base","invoice","+319 (Reporte 103)","","319 2% Cuotas de Arrendamiento Mercantil","319 2% Cuotas de Arrendamiento Mercantil (Prestado por Sociedades), Inclusive la de Opción de Compra","319","2% 319","True"
"","","","","","","","","","","","","tax","invoice","+369 (Reporte 103)","ret_ir_2x100","","","","",""
"","","","","","","","","","","","","base","refund","-319 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-369 (Reporte 103)","ret_ir_2x100","","","","",""
"tax_withhold_profit_320","8% 320","none","percent","70","-8","320 8% For Lease of Real Estate","320","370","tax_group_withhold_income_purchase","320","","base","invoice","+320 (Reporte 103)","","320 8% Arrendamiento Inmuebles","320 8% Por Arrendamiento Bienes Inmuebles","320","8% 320","False"
"","","","","","","","","","","","","tax","invoice","+370 (Reporte 103)","ret_ir_8x100","","","","",""
"","","","","","","","","","","","","base","refund","-320 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-370 (Reporte 103)","ret_ir_8x100","","","","",""
"tax_withhold_profit_322","1.75% 322","none","percent","70","-1.75","322 1.75% Insurance and Reinsurance (Premiums and Cessions)","322","372","tax_group_withhold_income_purchase","322","","base","invoice","+322 (Reporte 103)","","322 1.75% Seguros","322 1.75% Seguros y Reaseguros (Primas y Cesiones) 1.75%","322","1.75% 322","False"
"","","","","","","","","","","","","tax","invoice","+372 (Reporte 103)","ret_ir_1_75x100","","","","",""
"","","","","","","","","","","","","base","refund","-322 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-372 (Reporte 103)","ret_ir_1_75x100","","","","",""
"tax_withhold_profit_332","0% 332","none","percent","70","0","332 0% Other Purchases of Goods and Services not Subject to Withholding Tax","332","","tax_group_withhold_income_purchase","332","","base","invoice","+332 (Reporte 103)","","332 0% Otras Compras","332 0% Otras Compras de Bienes y Servicios No Sujetas a Retencion","332","0% 332","True"
"","","","","","","","","","","","","tax","invoice","","ret_ir_others","","","","",""
"","","","","","","","","","","","","base","refund","-332 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ret_ir_others","","","","",""
"tax_withhold_profit_332A","0% 332a","none","percent","70","0","332a 0% Disposal of Rights Representing Capital and Other Exempted Rights (may 2016)","332","","tax_group_withhold_income_purchase","332A","","base","invoice","+332 (Reporte 103)","","332A 0% Enajenacion","332A 0% Enajenacion de Derechos Representativos de Capital y Otros Derechos Exentos (mayo 2016)","332","0% 332a","True"
"","","","","","","","","","","","","tax","invoice","","ret_ir_others","","","","",""
"","","","","","","","","","","","","base","refund","-332 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ret_ir_others","","","","",""
"tax_withhold_profit_332B","0% 332b","none","percent","70","0","332b 0% Purchase of Real Estate","332","","tax_group_withhold_income_purchase","332B","","base","invoice","+332 (Reporte 103)","","332B 0% Compra Inmuebles","332B 0% Compra de Bienes Inmuebles","332","0% 332b","True"
"","","","","","","","","","","","","tax","invoice","","ret_ir_others","","","","",""
"","","","","","","","","","","","","base","refund","-332 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ret_ir_others","","","","",""
"tax_withhold_profit_332C","0% 332c","none","percent","70","0","332c 0% Public Passenger Transportation","332","","tax_group_withhold_income_purchase","332C","","base","invoice","+332 (Reporte 103)","","332C 0% Transporte Pasajeros","332C 0% Transporte Publico de Pasajeros","332","0% 332c","True"
"","","","","","","","","","","","","tax","invoice","","ret_ir_others","","","","",""
"","","","","","","","","","","","","base","refund","-332 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ret_ir_others","","","","",""
"tax_withhold_profit_332D","0% 332d","none","percent","70","0","332d 0% Payments in the Country for the Transportation of Passengers or International Transportation of Cargo, to National or Foreign Aviation or Maritime Companies","332","","tax_group_withhold_income_purchase","332D","","base","invoice","+332 (Reporte 103)","","332D 0% Pagos Transporte","332D 0% Pagos en el Pais por Transporte de Pasajeros o Transporte Internacional de Carga, a Compañias Nacionales o Extranjeras de Aviacion o Maritimas","332","0% 332d","True"
"","","","","","","","","","","","","tax","invoice","","ret_ir_others","","","","",""
"","","","","","","","","","","","","base","refund","-332 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ret_ir_others","","","","",""
"tax_withhold_profit_332G","0% 332g","none","percent","70","0","332g 0% Credit Card Payments","332","","tax_group_withhold_income_purchase","332G","","base","invoice","+332 (Reporte 103)","","332G 0% Pagos Tarjeta","332G 0% Pagos con Tarjeta de Credito","332","0% 332g","True"
"","","","","","","","","","","","","tax","invoice","","ret_ir_others","","","","",""
"","","","","","","","","","","","","base","refund","-332 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ret_ir_others","","","","",""
"tax_withhold_profit_332I","0% 332i","none","percent","70","0","332i 0% Payments Through Debit Agreements (ifi`s Customers)","332","","tax_group_withhold_income_purchase","332I","","base","invoice","+332 (Reporte 103)","","332I 0% Pagos Débito","332I 0% Pagos a Través de Convenios de Débito (clientes ifi`s)","332","0% 332i","True"
"","","","","","","","","","","","","tax","invoice","","ret_ir_others","","","","",""
"","","","","","","","","","","","","base","refund","-332 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","","ret_ir_others","","","","",""
"tax_withhold_profit_343","1% 343","none","percent","70","-1","343 1% Other Applicable Withholdings 1% (Includes RIMPE regimen)","343","393","tax_group_withhold_income_purchase","343","100","base","invoice","+343 (Reporte 103)","","343 1% Otras Retenciones","343 1% Otras Retenciones Aplicables el 1% (Incluye régimen RIMPE)","Otras 1%","1% 343","True"
"","","","","","","","","","","","100","tax","invoice","+393 (Reporte 103)","ret_ir_1x100","","","","",""
"","","","","","","","","","","","100","base","refund","-343 (Reporte 103)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-393 (Reporte 103)","ret_ir_1x100","","","","",""
"tax_withhold_profit_343A","1% 343a","none","percent","70","-1","343a 1% for electric energy","343","393","tax_group_withhold_income_purchase","343A","","base","invoice","+343 (Reporte 103)","","343A 1% Energia Electrica","343A 1% Por Energia Electrica","343","1% 343a","True"
"","","","","","","","","","","","","tax","invoice","+393 (Reporte 103)","ret_ir_1x100","","","","",""
"","","","","","","","","","","","","base","refund","-343 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-393 (Reporte 103)","ret_ir_1x100","","","","",""
"tax_withhold_profit_343B","1% 343b","none","percent","70","-1","343b 1% For Construction Activities of Real Estate"," Urbanization"," Land Development or Similar Activities","tax_group_withhold_income_purchase","343B","","base","invoice","+343 (Reporte 103)","","343B 1%,"True" Construccion","343B 1% Por Actividades de Construccion de Obra Material Inmueble, Urbanizacion, Lotizacion o Actividades Similares","343","1% 343b"
"","","","","","","","","","","","","tax","invoice","+393 (Reporte 103)","ret_ir_1x100","","","","",""
"","","","","","","","","","","","","base","refund","-343 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-393 (Reporte 103)","ret_ir_1x100","","","","",""
"tax_withhold_profit_3440","2.75% 3440","none","percent","70","-2.75","3440-344 2.75% Other Withholdings 2.75% Applicable","3440","3940","tax_group_withhold_income_purchase","3440","","base","invoice","+3440 (Reporte 103)","","3440-344 2.75% Otras Retenciones","3440-344 2.75% Otras Retenciones Aplicables el 2,75%","3440","2.75% 3440","True"
"","","","","","","","","","","","","tax","invoice","+3940 (Reporte 103)","ret_ir_2_75x100","","","","",""
"","","","","","","","","","","","","base","refund","-3440 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-3940 (Reporte 103)","ret_ir_2_75x100","","","","",""
"tax_withhold_profit_346","1.75% 346","none","percent","70","-1.75","346 1.75% Microenterprises (Other Withholdings Applicable to Other Percentages)","346","396","tax_group_withhold_income_purchase","346","","base","invoice","+346 (Reporte 103)","","346 1.75% Microempresas","346 1.75% Microempresas (Otras retenciones aplicables a otros porcentajes)","346","1.75% 346","True"
"","","","","","","","","","","","","tax","invoice","+396 (Reporte 103)","ret_ir_1_75x100","","","","",""
"","","","","","","","","","","","","base","refund","-346 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-396 (Reporte 103)","ret_ir_1_75x100","","","","",""
"tax_withhold_profit_347_346","2% 347-346","none","percent","70","-2","347-346 2% Cash Donations - Donation Tax","346","396","tax_group_withhold_income_purchase","347","","base","invoice","+346 (Reporte 103)","","347-346 2% Donaciones","347-346 2% Donaciones en Dinero -Impuesto a las Donaciones","347","2% 347-346","True"
"","","","","","","","","","","","","tax","invoice","+396 (Reporte 103)","ret_ir_2x100","","","","",""
"","","","","","","","","","","","","base","refund","-346 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-396 (Reporte 103)","ret_ir_2x100","","","","",""
"tax_withhold_profit_501_411","22% 501-411","none","percent","70","-22","501-411 22% Payment Abroad - Corporate Profits (With Double Taxation Agreement)","411","461","tax_group_withhold_income_purchase","501","","base","invoice","+411 (Reporte 103)","","501-411 22% Pago al Ext. Benef. Emp. (C/ Conv. D. T.)","501-411 22% Pago al Exterior - Beneficios Empresariales (Con Convenio de Doble Tributación)","411","22% 501-411","True"
"","","","","","","","","","","","","tax","invoice","+461 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-411 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-461 (Reporte 103)","ret_ir_22x100","","","","",""
"tax_withhold_profit_501_422","22% 501-422","none","percent","70","-22","501-422 22% Payment Abroad - Corporate Profits (Without Double Taxation Agreement)","422","472","tax_group_withhold_income_purchase","501","","base","invoice","+422 (Reporte 103)","","501-422 22% Pago al Ext. Benef. Emp. (S/ Conv. D. T.)","501-422 22% Pago al Exterior - Beneficios Empresariales (Sin Convenio de Doble Tributación)","422","22% 501-422","True"
"","","","","","","","","","","","","tax","invoice","+472 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-422 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-472 (Reporte 103)","ret_ir_22x100","","","","",""
"tax_withhold_profit_502_411","22% 502-411","none","percent","70","-22","502-411 22% Payment Abroad - Business Services (Under Double Taxation Agreement)","411","461","tax_group_withhold_income_purchase","502","","base","invoice","+411 (Reporte 103)","","502-411 22% Pago al Ext. Serv. Emp. (C/ Conv. D. T.)","502-411 22% Pago al Exterior - Servicios Empresariales (Con Convenio de Doble Tributación)","411","22% 502-411","True"
"","","","","","","","","","","","","tax","invoice","+461 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-411 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-461 (Reporte 103)","ret_ir_22x100","","","","",""
"tax_withhold_profit_502_422","22% 502-422","none","percent","70","-22","502-422 22% Payment Abroad - Business Services (No Double Taxation Agreement)","422","472","tax_group_withhold_income_purchase","502","","base","invoice","+422 (Reporte 103)","","502-422 22% Pago al Ext. Serv. Emp. (S/ Conv. D. T.)","502-422 22% Pago al Exterior - Servicios Empresariales (Sin Convenio de Doble Tributación)","422","22% 502-422","True"
"","","","","","","","","","","","","tax","invoice","+472 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-422 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-472 (Reporte 103)","ret_ir_22x100","","","","",""
"tax_withhold_profit_509_411","22% 509-411","none","percent","70","-22","509-411 22% Payment Abroad - Royalties, Copyrights, Trademarks, Patents and the Like (With Double Taxation Agreement)","411","461","tax_group_withhold_income_purchase","509","","base","invoice","+411 (Reporte 103)","","509-411 22% Pago al Ext. Casales (C/ Conv. D. T.)","509-411 22% Pago al Exterior - Casales, Derechos de Autor, Marcas, Patentes y Similares (Con Convenio de Doble Tributación)","411","22% 509-411","True"
"","","","","","","","","","","","","tax","invoice","+461 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-411 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-461 (Reporte 103)","ret_ir_22x100","","","","",""
"tax_withhold_profit_509_422","22% 509-422","none","percent","70","-22","509-422 Payment Abroad - Royalties, Copyrights, Trademarks, Patents and Similar (Without Double Taxation Agreement)","422","472","tax_group_withhold_income_purchase","509","","base","invoice","+422 (Reporte 103)","","509-422 Pago al Ext. Casales (S/ Conv. D. T.)","509-422 Pago al Exterior - Casales, Derechos de Autor, Marcas, Patentes y Similares (Sin Convenio de Doble Tributación)","422","22% 509-422","True"
"","","","","","","","","","","","","tax","invoice","+472 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-422 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-472 (Reporte 103)","ret_ir_22x100","","","","",""
"tax_withhold_profit_511_411","22% 511-411","none","percent","70","-22","511-411 22% Payment Abroad - Independent Professional Services (With Double Taxation Agreement)","411","461","tax_group_withhold_income_purchase","511","","base","invoice","+411 (Reporte 103)","","511-411 22% Pago al Ext. Serv. Prof. Ind. (C/ Conv. D. T.)","511-411 22% Pago al Exterior - Servicios Profesionales Independientes (Con Convenio de Doble Tributación)","411","22% 511-411","True"
"","","","","","","","","","","","","tax","invoice","+461 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-411 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-461 (Reporte 103)","ret_ir_22x100","","","","",""
"tax_withhold_profit_512_411","22% 512-411","none","percent","70","-22","512-411 22% Payment Abroad - Dependent Professional Services (With Double Taxation Agreement)","411","461","tax_group_withhold_income_purchase","512","","base","invoice","+411 (Reporte 103)","","512-411 22% Pago al Ext. Serv. Prof. Dep. (C/ Conv. D. T.)","512-411 22% Pago al Exterior - Servicios Profesionales Dependientes (Con Convenio de Doble Tributación)","411","22% 512-411","True"
"","","","","","","","","","","","","tax","invoice","+461 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-411 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-461 (Reporte 103)","ret_ir_22x100","","","","",""
"tax_withhold_profit_517_411","22% 517-411","none","percent","70","-22","517-411 22% Payment Abroad - Reimbursement of Expenses (With Double Taxation Agreement)","411","461","tax_group_withhold_income_purchase","517","","base","invoice","+411 (Reporte 103)","","517-411 22% Pago al Ext. Reem. Gastos (C/ Conv. D. T.)","517-411 22% Pago al Exterior - Reembolso de Gastos (Con Convenio de Doble Tributación)","411","22% 517-411","True"
"","","","","","","","","","","","","tax","invoice","+461 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-411 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-461 (Reporte 103)","ret_ir_22x100","","","","",""
"tax_withhold_profit_520D_411","22% 520d-411","none","percent","70","-22","520d-411 22% Payment Abroad - Commissions on Exports and Inbound Tourism Promotion (With Double Taxation Agreement)","411","461","tax_group_withhold_income_purchase","520D","","base","invoice","+411 (Reporte 103)","","520D-411 22% Pago al Ext. Comis. Export. y Promoc. Tur. Rec. (C/ Conv. D. T.)","520D-411 22% Pago al Exterior - Comisiones por Exportaciones y Por Promocion de Turismo Receptivo (Con Convenio de Doble Tributación)","411","22% 520d-411","True"
"","","","","","","","","","","","","tax","invoice","+461 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-411 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-461 (Reporte 103)","ret_ir_22x100","","","","",""
"tax_withhold_profit_522A_410","22% 522a-410","none","percent","70","-22","522a-410 22% Foreign Payment - Technical, Administrative or Consulting Services and Royalties with Double Taxation Agreement (With Double Taxation Agreement)","410","460","tax_group_withhold_income_purchase","522A","","base","invoice","+410 (Reporte 103)","","522A-410 22% Pago al Ext. Serv. Téc., Admin. o Consult. y Regalias (C/ Conv. D. T.)","522A-410 22% Pago al Exterior - Servicios Tecnicos, Administrativos o de Consultoria y Regalias (Con Convenio de Doble Tributación)","410","22% 522a-410","True"
"","","","","","","","","","","","","tax","invoice","+460 (Reporte 103)","ret_ir_22x100","","","","",""
"","","","","","","","","","","","","base","refund","-410 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-460 (Reporte 103)","ret_ir_22x100","","","","",""
"income_tax_withholding_302","22% 302 WTH","none","percent","70","-22","302 In a Dependency Relationship that Exceeds or Not the Deductible Base","302","352","tax_group_withhold_income_purchase","352","100","base","invoice","+302 (Reporte 103)","","302 En rel. de Dep. que Supera o no la Base Desg.","302 En relación de Dependencia que Supera o no la Base Desgravada","302","22% 302 WTH","True"
"","","","","","","","","","","","100","tax","invoice","+352 (Reporte 103)","ec21070101","","","","",""
"","","","","","","","","","","","100","base","refund","-302 (Reporte 103)","","","","","",""
"","","","","","","","","","","","100","tax","refund","-352 (Reporte 103)","ec21070101","","","","",""
"tax_withhold_profit_304_10","10% 304","none","percent","70","-10","304 10% Services Predominantly Intellectual Services not Related to Professional Degree","304","354","tax_group_withhold_income_purchase","304","","base","invoice","+304 (Reporte 103)","","304 10% Servicios Predomina el Intelecto No Relacionados con el Titulo Profesional","304 10% Servicios Predomina el Intelecto No Relacionados con el Titulo Profesional","304","10% 304","True"
"","","","","","","","","","","","","tax","invoice","+354 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-304 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-354 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_304A_10","10% 304a","none","percent","70","-10","304a 10% Commissions and Other Payments for Services Predominantly Intellectually Unrelated to Professional Title","304","354","tax_group_withhold_income_purchase","304A","","base","invoice","+304 (Reporte 103)","","304A 10% Comisiones y Demas Pagos por Servicios Predomina Intelecto No Relacionados con el Titulo Profesional","304A 10% Comisiones y Demas Pagos por Servicios Predomina Intelecto No Relacionados con el Titulo Profesional","304","10% 304a","True"
"","","","","","","","","","","","","tax","invoice","+354 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-304 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-354 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_304B_10","10% 304b","none","percent","70","-10","304b 10% Payments to Notaries and Property and Commercial Registrars for their Activities as Such","304","354","tax_group_withhold_income_purchase","304B","","base","invoice","+304 (Reporte 103)","","304B 10% Pagos a Notarios y Registradores de la Propiedad y Mercantil por sus Actividades Ejercidas Como Tales","304B 10% Pagos a Notarios y Registradores de la Propiedad y Mercantil por sus Actividades Ejercidas Como Tales","304","10% 304b","True"
"","","","","","","","","","","","","tax","invoice","+354 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-304 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-354 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_304E_10","10% 304e","none","percent","70","-10","304e 10% Fees and Other Payments for Teaching Services","304","354","tax_group_withhold_income_purchase","304E","","base","invoice","+304 (Reporte 103)","","304E 10% Honorarios y Demas Pagos por Servicios de Docencia","304E 10% Honorarios y Demas Pagos por Servicios de Docencia","304","10% 304e","False"
"","","","","","","","","","","","","tax","invoice","+354 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-304 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-354 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_309_2_75","2.75% 309","none","percent","70",-2.75,"309 2.75% Services Rendered by Media and Advertising Agencies","309","359","tax_group_withhold_income_purchase","309","","base","invoice","+309 (Reporte 103)","","309 2.75% Servicios Prestados por Medios de Comunicación y Agencias de Publicidad","309 2.75% Servicios Prestados por Medios de Comunicación y Agencias de Publicidad","309","2.75% 309","True"
"","","","","","","","","","","","","tax","invoice","+359 (Reporte 103)","ret_ir_2_75x100","","","","",""
"","","","","","","","","","","","","base","refund","-309 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-359 (Reporte 103)","ret_ir_2_75x100","","","","",""
"tax_withhold_profit_303A","3% 303a","none","percent","70","-3","303a 3% Professional Services Provided by Resident Companies","3030","3530","tax_group_withhold_income_purchase","303A","","base","invoice","+3030 (Reporte 103)","","303A 3% Servicios Profesionales Prestados por Sociedades Residentes","303A 3% Servicios Profesionales Prestados por Sociedades Residentes","3030","3% 303a","True"
"","","","","","","","","","","","","tax","invoice","+3530 (Reporte 103)","ret_ir_3x100","","","","",""
"","","","","","","","","","","","","base","refund","-3030 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-3530 (Reporte 103)","ret_ir_3x100","","","","",""
"tax_withhold_profit_312C","1.75% 312c","none","percent","70","-1.75","312c 1.75% Purchases from the Marketer: of Goods of Bio-aquatic, Forest and those described in Art.27.1 of LRTI","3121","3621","tax_group_withhold_income_purchase","312C","","base","invoice","+3121 (Reporte 103)","","312C 1.75% Compras al Comercializador: de Bienes de Origen Bioacuático, Forestal y los Descritos el Art.27.1 de LRTI","312C 1.75% Compras al Comercializador: de Bienes de Origen Bioacuático, Forestal y los Descritos el Art.27.1 de LRTI","3121","1.75% 312c","True"
"","","","","","","","","","","","","tax","invoice","+3621 (Reporte 103)","ret_ir_1_75x100","","","","",""
"","","","","","","","","","","","","base","refund","-3121 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-3621 (Reporte 103)","ret_ir_1_75x100","","","","",""
"tax_withhold_profit_3482","3% 3482","none","percent","70","-3.0","3482 3% Comisiones a sociedades, nacionales o extranjeras residentes y establecimientos permanentes domiciliados en el país","3140","3640","tax_group_withhold_income_purchase","3482","","base","invoice","+3140 (Reporte 103)","","3482 3% Comisiones a sociedades, nacionales o extranjeras residentes y establecimientos permanentes domiciliados en el país","3482 3% Comisiones a sociedades, nacionales o extranjeras residentes y establecimientos permanentes domiciliados en el país","3140","3% 3482","True"
"","","","","","","","","","","","","tax","invoice","+3640 (Reporte 103)","ret_ir_3x100","","","","",""
"","","","","","","","","","","","","base","refund","-3140 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-3640 (Reporte 103)","ret_ir_3x100","","","","",""

"tax_withhold_profit_314A_10","10% 314a","none","percent","70","-10","314a 10% Royalties for Franchises under the INGENIOS Code (COESCCI) - Payment to Natural Persons","314","364","tax_group_withhold_income_purchase","314A","","base","invoice","+314 (Reporte 103)","","314A 10% Regalías por Concepto de Franquicias de Acuerdo al Código INGENIOS (COESCCI) - Pago a Personas Naturales","314A 10% Regalías por Concepto de Franquicias de Acuerdo al Código INGENIOS (COESCCI) - Pago a Personas Naturales","314","10% 314a","True"
"","","","","","","","","","","","","tax","invoice","+364 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-314 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-364 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_314B_10","10% 314b","none","percent","70","-10","314b 10% Royalties, Copyrights, Brands, Patents and Similar according to the INGENIOS Code (COESCCI) - Payment to Natural Persons","314","364","tax_group_withhold_income_purchase","314B","","base","invoice","+314 (Reporte 103)","","314B 10% Cánones, Derechos de Autor, Marcas, Patentes y Similares de Acuerdo al Código INGENIOS (COESCCI) – Pago a Personas Naturales","314B 10% Cánones, Derechos de Autor, Marcas, Patentes y Similares de Acuerdo al Código INGENIOS (COESCCI) – Pago a Personas Naturales","314","10% 314b","True"
"","","","","","","","","","","","","tax","invoice","+364 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-314 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-364 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_314C_10","10% 314c","none","percent","70","-10","314c 10% Royalties on account of Franchises under the INGENIOS Code (COESCCI) - Payment to Companies","314","364","tax_group_withhold_income_purchase",314C,"","base","invoice","+314 (Reporte 103)","","314C 10% Regalias por Concepto de Franquicias de Acuerdo al Código INGENIOS (COESCCI) - Pago a Sociedades","314C 10% Regalias por Concepto de Franquicias de Acuerdo al Código INGENIOS (COESCCI) - Pago a Sociedades","314","10% 314c","True"
"","","","","","","","","","","","","tax","invoice","+364 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-314 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-364 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_314D_10","10% 314d","none","percent","70","-10","314d 10% Royalties, Copyrights, Trademarks, Patents and Similar according to the INGENIOS Code (COESCCI)","314","364","tax_group_withhold_income_purchase",314D,"","base","invoice","+314 (Reporte 103)","","314D 10% Cánones, Derechos de Autor, Marcas, Patentes y Similares de Acuerdo al Código INGENIOS (COESCCI)","314D 10% Cánones, Derechos de Autor, Marcas, Patentes y Similares de Acuerdo al Código INGENIOS (COESCCI)","314","10% 314d","True"
"","","","","","","","","","","","","tax","invoice","+364 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-314 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-364 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_320_10","10% 320","none","percent","70","-10","320 10% For Lease of Real Estate","320","370","tax_group_withhold_income_purchase","320","","base","invoice","+320 (Reporte 103)","","320 10% Por Arrendamiento Bienes Inmuebles","320 10% Por Arrendamiento Bienes Inmuebles","320","10% 320","True"
"","","","","","","","","","","","","tax","invoice","+370 (Reporte 103)","ret_ir_10x100","","","","",""
"","","","","","","","","","","","","base","refund","-320 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-370 (Reporte 103)","ret_ir_10x100","","","","",""
"tax_withhold_profit_322_1","1% 322","none","percent","70","-1","322 1% Insurance and Reinsurance (Premiums and Cessions)","322","372","tax_group_withhold_income_purchase","322","","base","invoice","+322 (Reporte 103)","","322 1% Seguros y Reaseguros (Primas y Cesiones)","322 1% Seguros y Reaseguros (Primas y Cesiones)","322","1% 322","True"
"","","","","","","","","","","","","tax","invoice","+372 (Reporte 103)","ret_ir_1x100","","","","",""
"","","","","","","","","","","","","base","refund","-322 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-372 (Reporte 103)","ret_ir_1x100","","","","",""
"tax_withhold_profit_323O","1% 323o","none","percent","70","-1","323o 1% Interest and Other Financial Income Paid to Banks and Other Entities Subject to the Control of the Superintendency of Banks and the Popular and Solidarity Economy","323","373","tax_group_withhold_income_purchase","323O","","base","invoice","+323 (Reporte 103)","","323O 1% Intereses y Demás Rendimientos Financieros Pagados a Bancos y Otras Entidades Sometidas al Control de la Superintendencia de Bancos y de la Economía Popular y Solidaria","323O 1% Intereses y Demás Rendimientos Financieros Pagados a Bancos y Otras Entidades Sometidas al Control de la Superintendencia de Bancos y de la Economía Popular y Solidaria","323","1% 323o","True"
"","","","","","","","","","","","","tax","invoice","+372 (Reporte 103)","ret_ir_1x100","","","","",""
"","","","","","","","","","","","","base","refund","-322 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-372 (Reporte 103)","ret_ir_1x100","","","","",""
"tax_withhold_profit_343C","2% 343c","none","percent","70","-2","343c 2% PET Non-Returnable Plastic Bottles Reception","344","394","tax_group_withhold_income_purchase","343C","","base","invoice","+344 (Reporte 103)","","343C 2% Recepción de Botellas Plásticas no Retornables de PET","343C 2% Recepción de Botellas Plásticas no Retornables de PET","344","2% 343c","True"
"","","","","","","","","","","","","tax","invoice","+394 (Reporte 103)","ret_ir_2x100","","","","",""
"","","","","","","","","","","","","base","refund","-344 (Reporte 103)","","","","","",""
"","","","","","","","","","","","","tax","refund","-394 (Reporte 103)","ret_ir_2x100","","","","",""
"tax_withhold_profit_sale_1x100","1% WTH","none","percent","70","-1","1% Withholding Taxes","","","tax_group_withhold_income_sale","","","base","invoice","","","1% Ret. de la Fuente","1% Retenciones de la Fuente","1%","1% WTH","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_profit_withhold","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_profit_withhold","","","","",""
"tax_withhold_profit_sale_1_75x100","1.75% WTH","none","percent","70","-1.75","1.75% Withholding Taxes","","","tax_group_withhold_income_sale","","","base","invoice","","","1.75% Ret. de la Fuente","1.75% Retenciones de la Fuente","1.75%","1.75% WTH","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_profit_withhold","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_profit_withhold","","","","",""
"tax_withhold_profit_sale_2x100","2% WTH","none","percent","70","-2","2% Withholding Taxes","","","tax_group_withhold_income_sale","","","base","invoice","","","2% Ret. de la Fuente","2% Retenciones de la Fuente","2%","2% WTH","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_profit_withhold","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_profit_withhold","","","","",""
"tax_withhold_profit_sale_2_75x100","2.75% WTH","none","percent","70","-2.75","2.75% Withholding Taxes","","","tax_group_withhold_income_sale","","","base","invoice","","","2.75% Ret. de la Fuente","2.75% Retenciones de la Fuente","2.75%","2.75% WTH","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_profit_withhold","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_profit_withhold","","","","",""
"tax_withhold_profit_sale_3x100","3% WH","none","percent","70","-3","3% Withholding Taxes","","","tax_group_withhold_income_sale","","","base","invoice","","","3% Ret. de la Fuente","3% Retenciones de la Fuente","3%","3% WTH","True"
"","","","","","","","","","","","",tax,invoice,"",ec_sale_profit_withhold,"","","","",""
"","","","","","","","","","","","",base,refund,"","","","","","",""
"","","","","","","","","","","","",tax,refund,"",ec_sale_profit_withhold,"","","","",""
"tax_withhold_profit_sale_5x100","5% WTH","none","percent","70","-5","5% Withholding Taxes","","","tax_group_withhold_income_sale","","","base","invoice","","","5% Ret. de la Fuente","5% Retenciones de la Fuente","5%","5% WTH","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_profit_withhold","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_profit_withhold","","","","",""
"tax_withhold_profit_sale_8x100","8% WTH","none","percent","70","-8","8% Withholding Taxes","","","tax_group_withhold_income_sale","","","base","invoice","","","8% Ret. de la Fuente","8% Retenciones de la Fuente","8%","8% WTH","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_profit_withhold","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_profit_withhold","","","","",""
"tax_withhold_profit_sale_10x100","10% WTH","none","percent","70","-10","10% Withholding Taxes","","","tax_group_withhold_income_sale","","","base","invoice","","","10% Ret. de la Fuente","10% Retenciones de la Fuente","10%","10% WTH","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_profit_withhold","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_profit_withhold","","","","",""
"tax_withhold_profit_sale_15x100","15% WTH","none","percent","70","-15","15% Withholding Taxes","","","tax_group_withhold_income_sale","","","base","invoice","","","15% Ret. de la Fuente","15% Retenciones de la Fuente","15%","15% WTH","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_profit_withhold","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_profit_withhold","","","","",""
"tax_withhold_profit_sale_22x100","22% WTH","none","percent","70","-22","22% Withholding Taxes","","","tax_group_withhold_income_sale","","","base","invoice","","","22% Ret. de la Fuente","22% Retenciones de la Fuente","22%","22% WTH","True"
"","","","","","","","","","","","","tax","invoice","","ec_sale_profit_withhold","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","","tax","refund","","ec_sale_profit_withhold","","","","",""
"tax_withhold_vat_10","10% WTH","none","percent","50","-10","VAT Withholding 10%","","721","tax_group_withhold_vat_purchase","","","base","invoice","","","10% Ret. IVA","10% Retención IVA","RET IVA 10%","10% WTH","True"
"","","","","","","","","","","","100","tax","invoice","+721 (Reporte 104)","ec_vat_withhold_10","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-721 (Reporte 104)","ec_vat_withhold_10","","","","",""
"tax_withhold_vat_20","20% WTH","none","percent","50","-20","VAT Withholding 20%","","723","tax_group_withhold_vat_purchase","","","base","invoice","","","20% Ret. IVA","20% Retención IVA","RET IVA 20%","20% WTH","True"
"","","","","","","","","","","","100","tax","invoice","+723 (Reporte 104)","ec_vat_withhold_20","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-723 (Reporte 104)","ec_vat_withhold_20","","","","",""
"tax_withhold_vat_30","30% WTH","none","percent","50","-30","VAT Withholding 30%","","725","tax_group_withhold_vat_purchase","","","base","invoice","","","30% Ret. IVA","30% Retención IVA","RET IVA 30%","30% WTH","True"
"","","","","","","","","","","","100","tax","invoice","+725 (Reporte 104)","ec_vat_withhold_30","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-725 (Reporte 104)","ec_vat_withhold_30","","","","",""
"tax_withhold_vat_50","50% WTH","none","percent","50","-50","VAT Withholding 50%","","727","tax_group_withhold_vat_purchase","","","base","invoice","","","50% Ret. IVA","50% Retención IVA","RET IVA 50%","50% WTH","True"
"","","","","","","","","","","","100","tax","invoice","+727 (Reporte 104)","ec_vat_withhold_50","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-727 (Reporte 104)","ec_vat_withhold_50","","","","",""
"tax_withhold_vat_70","70% WTH","none","percent","50","-70","VAT Withholding 70%","","729","tax_group_withhold_vat_purchase","","","base","invoice","","","70% Ret. IVA","70% Retención IVA","RET IVA 70%","70% WTH","True"
"","","","","","","","","","","","100","tax","invoice","+729 (Reporte 104)","ec_vat_withhold_70","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-729 (Reporte 104)","ec_vat_withhold_70","","","","",""
"tax_withhold_vat_100","100% WTH","none","percent","50","-100","VAT Withholding 100%","","731","tax_group_withhold_vat_purchase","","","base","invoice","","","100% Ret. IVA","100% Retención IVA","RET IVA 100%","100% WTH","True"
"","","","","","","","","","","","100","tax","invoice","+731 (Reporte 104)","ec_vat_withhold_100","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-731 (Reporte 104)","ec_vat_withhold_100","","","","",""
"tax_sale_withhold_vat_10","10% WTH","none","percent","60","-10","VAT Withholding 10%","","609","tax_group_withhold_vat_sale","","","base","invoice","","","10% Ret. IVA","10% Retención IVA","RET IVA 10%","10% WTH","True"
"","","","","","","","","","","","100","tax","invoice","+609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"tax_sale_withhold_vat_20","VAT WTH 20%","none","percent","60","-20","VAT Withholding 20%","","609","tax_group_withhold_vat_sale","","","base","invoice","","","20% Ret. IVA","20% Retención IVA","RET IVA 20%","VAT WTH 20%","True"
"","","","","","","","","","","","100","tax","invoice","+609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"tax_sale_withhold_vat_30","VAT WTH 30%","none","percent","60","-30","VAT Withholding 30%","","609","tax_group_withhold_vat_sale","","","base","invoice","","","30% Ret. IVA","30% Retención IVA","RET IVA 30%","VAT WTH 30%","True"
"","","","","","","","","","","","100","tax","invoice","+609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"tax_sale_withhold_vat_50","VAT WTH 50%","none","percent","60","-50","VAT Withholding 50%","","609","tax_group_withhold_vat_sale","","","base","invoice","","","50% Ret. IVA","50% Retención IVA","RET IVA 50%","VAT WTH 50%","True"
"","","","","","","","","","","","100","tax","invoice","+609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"tax_sale_withhold_vat_70","VAT WTH 70%","none","percent","60","-70","VAT Withholding 70%","","609","tax_group_withhold_vat_sale","","","base","invoice","","","70% Ret. IVA","70% Retención IVA","RET IVA 70%","VAT WTH 70%","True"
"","","","","","","","","","","","100","tax","invoice","+609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"tax_sale_withhold_vat_100","VAT WTH 100%","none","percent","60","-100","VAT Withholding 100%","","609","tax_group_withhold_vat_sale","","","base","invoice","","","100% Ret. IVA","100% Retención IVA","RET IVA 100%","VAT WTH 100%","True"
"","","","","","","","","","","","100","tax","invoice","+609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
"","","","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","","100","tax","refund","-609 (Reporte 104)","ec_sale_vat_outstanding_withholds","","","","",""
```

## File: data\template\account.tax.group-ec.csv

```csv
"id","name","l10n_ec_type","sequence","country_id","tax_payable_account_id","tax_receivable_account_id","name@es"
"tax_group_ice","Special Consumptions (ICE)","ice","60","base.ec","ec_ice_tax_deduction","ec_ice_tax_deduction","Consumos Especiales (ICE)"
"tax_group_vat_05","VAT 5%","vat05","3","base.ec","ec_vat_tax_deduction","ec_vat_tax_credit","IVA 5%"
"tax_group_vat_08","VAT 8%","vat08","5","base.ec","ec_vat_tax_deduction","ec_vat_tax_credit","IVA 8%"
"tax_group_vat_12","VAT 12%","vat12","10","base.ec","ec_vat_tax_deduction","ec_vat_tax_credit","IVA 12%"
"tax_group_vat_13","VAT 13%","vat13","15","base.ec","ec_vat_tax_deduction","ec_vat_tax_credit","IVA 13%"
"tax_group_vat14","VAT 14%","vat14","20","base.ec","ec_vat_tax_deduction","ec_vat_tax_credit","IVA 14%"
"tax_group_vat_15","VAT 15%","vat15","25","base.ec","ec_vat_tax_deduction","ec_vat_tax_credit","IVA 15%"
"tax_group_vat0","VAT 0%","zero_vat","30","base.ec","ec_vat_tax_deduction","ec_vat_tax_credit","IVA 0%"
"tax_group_vat_not_charged","Non-Subject VAT","not_charged_vat","40","base.ec","ec_vat_tax_deduction","ec_vat_tax_credit","IVA no cobrado"
"tax_group_vat_exempt","Exempt VAT","exempt_vat","50","base.ec","ec_vat_tax_deduction","ec_vat_tax_credit","Exento de IVA"
"tax_group_irbpnr","Plastic Bottles (IRBPNR)","irbpnr","70","base.ec","ec_irbpnr_tax_deduction","ec_irbpnr_tax_deduction","Botellas de plástico (IRBPNR)"
"tax_group_withhold_vat_sale","Sales Withholding VAT","withhold_vat_sale","80","base.ec","ec_vat_tax_deduction","ec_withhold_tax_credit","Retención IVA en Ventas"
"tax_group_withhold_vat_purchase","Purchase Withholding VAT","withhold_vat_purchase","80","base.ec","ec_vat_tax_deduction","ec_withhold_tax_credit","Retención IVA en Compras"
"tax_group_withhold_income_sale","Sale Profit Withhold","withhold_income_sale","90","base.ec","ec_profit_tax_deduction","ec_profit_tax_credit","Retención Renta en Ventas"
"tax_group_withhold_income_purchase","Purchase Profit Withhold","withhold_income_purchase","95","base.ec","ec_profit_tax_deduction","ec_profit_tax_credit"
"tax_group_outflows","Exchange Outflows","outflows_tax","100","base.ec","ec_others_tax_deduction","ec_others_tax_credit","Salidas por cambio"
"tax_group_others","Others","other","110","base.ec","ec_others_tax_deduction","ec_others_tax_credit","Otros"
```

## File: models\account_journal.py

```python
from odoo import api, fields, models


class AccountJournal(models.Model):
    _inherit = "account.journal"

    l10n_ec_require_emission = fields.Boolean(
        string='Require Emission',
        compute='_compute_l10n_ec_require_emission',
        help='True if an entity and emission point must be set on the journal'
    )
    l10n_ec_entity = fields.Char(
        string="Emission Entity",
        size=3,
        copy=False,
        help="Ecuador: Emission entity number that is given by the SRI."
    )
    l10n_ec_emission = fields.Char(
        string="Emission Point",
        size=3, copy=False,
        help="Ecuador: Emission point number that is given by the SRI."
    )
    l10n_ec_emission_address_id = fields.Many2one(
        comodel_name="res.partner",
        string="Emission address",
        domain="['|', ('id', '=', company_partner_id), '&', ('id', 'child_of', company_partner_id), ('type', '!=', 'contact')]",
        help="Ecuador: Address for electronic invoicing.",
    )

    @api.depends('type', 'country_code', 'l10n_latam_use_documents')
    def _compute_l10n_ec_require_emission(self):
        for journal in self:
            journal.l10n_ec_require_emission = journal.type == 'sale' and journal.country_code == 'EC' and journal.l10n_latam_use_documents


```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.l10n_ec.models.res_partner import PartnerIdTypeEc
from odoo import fields, models, api

_DOCUMENTS_MAPPING = {
    "01": [
        'ec_dt_01',
        'ec_dt_02',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_08',
        'ec_dt_09',
        'ec_dt_11',
        'ec_dt_12',
        'ec_dt_16',
        'ec_dt_20',
        'ec_dt_21',
        'ec_dt_41',
        'ec_dt_42',
        'ec_dt_43',
        'ec_dt_45',
        'ec_dt_47',
        'ec_dt_48'
    ],
    "02": [
        'ec_dt_03',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_09',
        'ec_dt_19',
        'ec_dt_41',
        'ec_dt_294',
        'ec_dt_344'
    ],
    "03": [
        'ec_dt_03',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_09',
        'ec_dt_15',
        'ec_dt_19',
        'ec_dt_41',
        'ec_dt_45',
        'ec_dt_294',
        'ec_dt_344'
    ],
    "04": [
        'ec_dt_01',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_41',
        'ec_dt_44',
        'ec_dt_47',
        'ec_dt_48',
        'ec_dt_49',
        'ec_dt_50',
        'ec_dt_51',
        'ec_dt_52',
        'ec_dt_370',
        'ec_dt_371',
        'ec_dt_372',
        'ec_dt_373'
    ],
    "05": [
        'ec_dt_01',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_41',
        'ec_dt_44',
        'ec_dt_47',
        'ec_dt_48',
        'ec_dt_370',
        'ec_dt_371',
        'ec_dt_372',
        'ec_dt_373'
    ],
    "06": [
        'ec_dt_01',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_41',
        'ec_dt_44',
        'ec_dt_47',
        'ec_dt_48',
        'ec_dt_370',
        'ec_dt_371',
        'ec_dt_372',
        'ec_dt_373'
    ],
    "07": [
        'ec_dt_01',
        'ec_dt_04',
        'ec_dt_05',
    ],
    "09": [
        'ec_dt_01',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_15',
        'ec_dt_16',
        'ec_dt_41',
        'ec_dt_47',
        'ec_dt_48',
    ],
    "20": [
        'ec_dt_01',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_15',
        'ec_dt_16',
        'ec_dt_41',
        'ec_dt_47',
        'ec_dt_48'
    ],
    "21": [
        'ec_dt_01',
        'ec_dt_04',
        'ec_dt_05',
        'ec_dt_15',
        'ec_dt_16',
        'ec_dt_41',
        'ec_dt_47',
        'ec_dt_48'
    ],
}


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_ec_sri_payment_id = fields.Many2one(
        comodel_name="l10n_ec.sri.payment",
        string="Payment Method (SRI)",
        help="Ecuador: Payment Methods Defined by the SRI.",
    )

    @api.model
    def _get_l10n_ec_documents_allowed(self, identification_code):
        documents_allowed = self.env['l10n_latam.document.type']
        for document_ref in _DOCUMENTS_MAPPING.get(identification_code.value, []):
            document_allowed = self.env.ref('l10n_ec.%s' % document_ref, False)
            if document_allowed:
                documents_allowed |= document_allowed
        return documents_allowed

    def _get_l10n_latam_documents_domain(self):
        self.ensure_one()
        domain = super()._get_l10n_latam_documents_domain()
        if self.country_code == 'EC' and self.journal_id.l10n_latam_use_documents:
            if self.debit_origin_id:  # show/hide the debit note document type
                domain.extend([('internal_type', '=', 'debit_note')])
            elif self.move_type in ('out_invoice', 'in_invoice'):
                domain.extend([('internal_type', '=', 'invoice')])
            allowed_documents = self._get_l10n_ec_documents_allowed(PartnerIdTypeEc.get_ats_code_for_partner(self.partner_id, self.move_type))
            domain.extend([('id', 'in', allowed_documents.ids)])
        return domain

    def _get_ec_formatted_sequence(self, number=0):
        return "%s %s-%s-%09d" % (
            self.l10n_latam_document_type_id.doc_code_prefix,
            self.journal_id.l10n_ec_entity,
            self.journal_id.l10n_ec_emission,
            number,
        )

    def _get_starting_sequence(self):
        """If use documents then will create a new starting sequence using the document type code prefix and the
        journal document number with a 8 padding number"""
        if (
            self.journal_id.l10n_latam_use_documents
            and self.company_id.country_id.code == "EC"
        ):
            if self.l10n_latam_document_type_id:
                return self._get_ec_formatted_sequence()
        return super()._get_starting_sequence()

    def _get_last_sequence_domain(self, relaxed=False):
        where_string, param = super(AccountMove, self)._get_last_sequence_domain(relaxed)
        if self.country_code == "EC" and self.l10n_latam_use_documents:
            internal_type = self.l10n_latam_document_type_id.internal_type
            document_types = self.env['l10n_latam.document.type'].search([
                ('internal_type', '=', internal_type),
                ('country_id.code', '=', 'EC'),
            ])
            if document_types:
                where_string += """
                AND l10n_latam_document_type_id in %(l10n_latam_document_type_id)s
                """
                param["l10n_latam_document_type_id"] = tuple(document_types.ids)
        return where_string, param

    def _skip_format_document_number(self):
        """
        If a Credit Note is created from a Vendor Bill and the partner_id != "EC",
        we want to allow the user to allocate any number without following the EC format.
        """
        self.ensure_one()
        if self.country_code == 'EC':
            return (
                    self.l10n_latam_document_type_id.internal_type in ('credit_note', 'debit_note')
                    and self.partner_id.country_code != "EC"
                    and self.move_type == 'in_refund'
                    and self.journal_id.type == 'purchase'
            )
        super()._skip_format_document_number()

```

## File: models\account_tax.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountTax(models.Model):

    _inherit = "account.tax"

    l10n_ec_code_base = fields.Char(
        string="Code base",
        help="Ecuador: Tax declaration code of the base amount prior to the calculation of the tax.",
    )
    l10n_ec_code_applied = fields.Char(
        string="Code applied",
        help="Ecuador: Tax declaration code of the resulting amount after the calculation of the tax.",
    )
    l10n_ec_code_ats = fields.Char(
        string="Code ATS",
        help="Ecuador: Indicates if the purchase invoice supports tax credit or cost or expenses, conforming table 5 of ATS.",
    )

```

## File: models\account_tax_group.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

_TYPE_EC = [
    ("vat05", "VAT 5%"),
    ("vat08", "VAT 8%"),
    ("vat12", "VAT 12%"),
    ("vat13", "VAT 13%"),
    ("vat14", "VAT 14%"),
    ("vat15", "VAT 15%"),
    ("zero_vat", "VAT 0%"),
    ("not_charged_vat", "VAT Not Charged"),
    ("exempt_vat", "VAT Exempt"),
    ("ice", "Special Consumptions Tax (ICE)"),
    ("irbpnr", "Plastic Bottles (IRBPNR)"),
    ("withhold_vat_sale", "VAT Withhold on Sales"),
    ("withhold_vat_purchase", "VAT Withhold on Purchases"),
    ("withhold_income_sale", "Profit Withhold on Sales"),
    ("withhold_income_purchase", "Profit Withhold on Purchases"),
    ("outflows_tax", "Exchange Outflows"),
    ("other", "Others"),
]


class AccountTaxGroup(models.Model):
    _inherit = "account.tax.group"

    l10n_ec_type = fields.Selection(
        _TYPE_EC, string="Type Ecuadorian Tax", help="Ecuadorian taxes subtype"
    )

```

## File: models\l10n_ec_sri_payment.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SriPayment(models.Model):

    _name = "l10n_ec.sri.payment"
    _description = "SRI Payment Method"

    name = fields.Char("Name", translate=True)
    code = fields.Char("Code")
    active = fields.Boolean("Active", default=True)

```

## File: models\l10n_latam_document_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _
from odoo.exceptions import UserError
import re


class L10nLatamDocumentType(models.Model):
    _inherit = "l10n_latam.document.type"

    internal_type = fields.Selection(
        selection_add=[
            ("purchase_liquidation", "Purchase Liquidation"),
            ("withhold", "Withhold"),
        ]
    )

    l10n_ec_check_format = fields.Boolean(
        string="Check Number Format EC", default=False
    )

    def _format_document_number(self, document_number):
        self.ensure_one()
        if self.country_id != self.env.ref("base.ec"):
            return super()._format_document_number(document_number)
        if not document_number:
            return False
        if self.l10n_ec_check_format:
            document_number = re.sub(r'\s+', "", document_number)  # remove any whitespace
            num_match = re.match(r'(\d{1,3})-(\d{1,3})-(\d{1,9})', document_number)
            if num_match:
                # Fill each number group with zeroes (3, 3 and 9 respectively)
                document_number = "-".join([n.zfill(3 if i < 2 else 9) for i, n in enumerate(num_match.groups())])
            else:
                raise UserError(_(
                    "Ecuadorian Document %s must be like 001-001-123456789",
                    self.display_name
                ))

        return document_number

```

## File: models\res_company.py

```python
from odoo import models


class ResCompany(models.Model):

    _inherit = "res.company"

    def _localization_use_documents(self):
        self.ensure_one()
        return self.account_fiscal_country_id.code == "EC" or super(ResCompany, self)._localization_use_documents()

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import enum
import stdnum
from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


def verify_final_consumer(vat):
    return vat == '9' * 13  # final consumer is identified with 9999999999999


class PartnerIdTypeEc(enum.Enum):
    """
    Ecuadorian partner identification type/code for ATS and SRI.
    """

    IN_RUC = '01'
    IN_CEDULA = '02'
    IN_PASSPORT = '03'
    OUT_RUC = '04'
    OUT_CEDULA = '05'
    OUT_PASSPORT = '06'
    FINAL_CONSUMER = '07'
    FOREIGN = '08'

    @classmethod
    def get_ats_code_for_partner(cls, partner, move_type):
        """
        Returns ID code for move and partner based on subset of Table 2 of SRI's ATS specification
        """
        partner_id_type = partner._l10n_ec_get_identification_type()
        if partner.vat and verify_final_consumer(partner.vat):
            return cls.FINAL_CONSUMER
        elif move_type.startswith('in_'):
            if partner_id_type == 'ruc':  # includes final consumer
                return cls.IN_RUC
            elif partner_id_type == 'cedula':
                return cls.IN_CEDULA
            elif partner_id_type in ['foreign', 'passport']:
                return cls.IN_PASSPORT
        elif move_type.startswith('out_'):
            if partner_id_type == 'ruc':  # includes final consumer
                return cls.OUT_RUC
            elif partner_id_type == 'cedula':
                return cls.OUT_CEDULA
            elif partner_id_type in ['foreign', 'passport']:
                return cls.OUT_PASSPORT


class ResPartner(models.Model):

    _inherit = "res.partner"

    l10n_ec_vat_validation = fields.Char(
        string="VAT Error message validation",
        compute="_compute_l10n_ec_vat_validation",
        help="Error message when validating the Ecuadorian VAT",
    )

    @api.constrains("vat", "country_id", "l10n_latam_identification_type_id")
    def check_vat(self):
        it_ruc = self.env.ref("l10n_ec.ec_ruc", False)
        it_dni = self.env.ref("l10n_ec.ec_dni", False)
        ecuadorian_partners = self.filtered(
            lambda x: x.country_id == self.env.ref("base.ec")
        )
        for partner in ecuadorian_partners:
            if partner.vat:
                if partner.l10n_latam_identification_type_id.id in (
                    it_ruc.id,
                    it_dni.id,
                ):
                    if partner.l10n_latam_identification_type_id.id == it_dni.id and len(partner.vat) != 10:
                        raise ValidationError(_('If your identification type is %s, it must be 10 digits',
                                                it_dni.display_name))
                    if partner.l10n_latam_identification_type_id.id == it_ruc.id and len(partner.vat) != 13:
                        raise ValidationError(_('If your identification type is %s, it must be 13 digits',
                                                it_ruc.display_name))
        return super(ResPartner, self - ecuadorian_partners).check_vat()

    @api.depends("vat", "country_id", "l10n_latam_identification_type_id")
    def _compute_l10n_ec_vat_validation(self):
        it_ruc = self.env.ref("l10n_ec.ec_ruc", False)
        it_dni = self.env.ref("l10n_ec.ec_dni", False)
        ruc = stdnum.util.get_cc_module("ec", "ruc")
        ci = stdnum.util.get_cc_module("ec", "ci")
        for partner in self:
            partner.l10n_ec_vat_validation = False
            if partner and partner.l10n_latam_identification_type_id in (it_ruc, it_dni) and partner.vat:
                final_consumer = verify_final_consumer(partner.vat)
                if not final_consumer:
                    if partner.l10n_latam_identification_type_id.id == it_dni.id and not ci.is_valid(partner.vat):
                        partner.l10n_ec_vat_validation = _("The VAT %s seems to be invalid as the tenth digit doesn't comply with the validation algorithm "
                                                           "(could be an old VAT number)", partner.vat)
                    if partner.l10n_latam_identification_type_id.id == it_ruc.id and not ruc.is_valid(partner.vat):
                        partner.l10n_ec_vat_validation = _("The VAT %s seems to be invalid as the tenth digit doesn't comply with the validation algorithm "
                                                           "(SRI has stated that this validation is not required anymore for some VAT numbers)", partner.vat)

    def _l10n_ec_get_identification_type(self):
        """Maps Odoo identification types to Ecuadorian ones.
        Useful for document type domains, electronic documents, ats, others.
        """
        self.ensure_one()

        id_types_by_xmlid = {
            'l10n_ec.ec_dni': 'cedula',  # DNI
            'l10n_ec.ec_ruc': 'ruc',  # RUC
            'l10n_ec.ec_passport': 'ec_passport',  # EC passport
            'l10n_latam_base.it_pass': 'passport',  # Passport
            'l10n_latam_base.it_fid': 'foreign',  # Foreign ID
            'l10n_latam_base.it_vat': 'foreign',
        }

        # This method is orm-cached, which makes it more efficient in loops than get_external_id()
        xmlid_by_res_id = {
            self.env['ir.model.data']._xmlid_to_res_model_res_id(xmlid, raise_if_not_found=True)[1]: xmlid
            for xmlid in id_types_by_xmlid
        }

        id_type_xmlid = xmlid_by_res_id.get(self.l10n_latam_identification_type_id.id)
        if id_type_xmlid in id_types_by_xmlid:
            return id_types_by_xmlid[id_type_xmlid]

        if self.l10n_latam_identification_type_id.country_id.code != 'EC':
            return 'foreign'

```

## File: models\template_ec.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ec')
    def _get_ec_template_data(self):
        return {
            'property_account_receivable_id': 'ec1102050101',
            'property_account_payable_id': 'ec210301',
            'property_account_expense_categ_id': 'ec110307',
            'journal_account_expense_categ_id': 'ec52022816',
            'property_account_income_categ_id': 'ec410101',
            'property_stock_account_input_categ_id': 'ec110307',
            'property_stock_account_output_categ_id': 'ec510102',
            'property_stock_valuation_account_id': 'ec110306',
            'loss_stock_valuation_account': 'ec510112',
            'production_stock_valuation_account': 'ec110302',
            'code_digits': '4',
        }

    @template('ec', 'res.company')
    def _get_ec_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.ec',
                'bank_account_code_prefix': '1101020',
                'cash_account_code_prefix': '1101010',
                'transfer_account_code_prefix': '1101030',
                'account_default_pos_receivable_account_id': 'ec1102050101',
                'income_currency_exchange_account_id': 'ec430501',
                'expense_currency_exchange_account_id': 'ec520304',
                'account_journal_early_pay_discount_loss_account_id': 'ec_early_pay_discount_loss',
                'account_journal_early_pay_discount_gain_account_id': 'ec_early_pay_discount_gain',
                'default_cash_difference_income_account_id': 'ec_income_cash_difference',
                'default_cash_difference_expense_account_id': 'ec_expense_cash_difference',
                'account_sale_tax_id': 'tax_vat_15_411_goods',
                'account_purchase_tax_id': 'tax_vat_15_510_sup_01',
            },
        }

    @template('ec', 'account.journal')
    def _get_ec_account_journal(self):
        """ In case of an Ecuador, we modified the sales journal"""
        return {
            'sale': {
                'name': "001-001 Facturas de cliente",
                'l10n_ec_entity': '001',
                'l10n_ec_emission': '001',
                'l10n_ec_emission_address_id': self.env.company.partner_id.id,
            },
        }

    def _post_load_data(self, template_code, company, template_data):
        super()._post_load_data(template_code, company, template_data)
        # Setup default Income/Expense Accounts on Sale/Purchase journals
        if (purchase_journal := self.ref("purchase", raise_if_not_found=False)) and (expense_account_ref := template_data.get('journal_account_expense_categ_id')):
            purchase_journal.default_account_id = self.ref(expense_account_ref, raise_if_not_found=False)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ec
from . import res_partner
from . import l10n_ec_sri_payment
from . import l10n_latam_document_type
from . import account_move
from . import account_tax
from . import account_tax_group
from . import res_company
from . import account_journal

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_ec_sri_payment_readonly,l10n_ec_sri_payment_readonly,model_l10n_ec_sri_payment,account.group_account_readonly,1,0,0,0
access_l10n_ec_sri_payment_invoice,l10n_ec_sri_payment_invoice,model_l10n_ec_sri_payment,account.group_account_invoice,1,0,0,0
access_l10n_ec_sri_payment_admin,l10n_ec_sri_payment_public,model_l10n_ec_sri_payment,account.group_account_manager,1,1,1,1

```

## File: views\account_journal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_journal_form" model="ir.ui.view">
            <field name="model">account.journal</field>
            <field name="name">account.journal.form</field>
            <field name="inherit_id" ref="l10n_latam_invoice_document.view_account_journal_form"/>
            <field name="arch" type="xml">
                <field name="l10n_latam_use_documents" position="after">
                    <field name="l10n_ec_require_emission" invisible="1"/>
                    <field name="l10n_ec_entity"
                           placeholder="001"
                           invisible="not l10n_ec_require_emission"
                           required="l10n_ec_require_emission"/>
                    <field name="l10n_ec_emission"
                           placeholder="001"
                           invisible="not l10n_ec_require_emission"
                           required="l10n_ec_require_emission"/>
                    <field name="l10n_ec_emission_address_id"
                           invisible="not l10n_ec_require_emission"
                           required="l10n_ec_require_emission"
                           context="{'default_parent_id': company_partner_id, 
                                     'default_type': 'invoice',
                                     'form_view_ref': 'base.view_partner_address_form'
                                     }"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\account_tax_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="account_tax_form_view" model="ir.ui.view">
        <field name="name">account.tax.form</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='type_tax_use']" position="after">
                <field name="l10n_ec_code_base" invisible="country_code != 'EC'"
                       groups="base.group_no_one"/>
                <field name="l10n_ec_code_applied" invisible="country_code != 'EC'"
                       groups="base.group_no_one"/>
                <field name="l10n_ec_code_ats" invisible="country_code != 'EC'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\l10n_ec_sri_payment.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="view_payment_method_form" model="ir.ui.view">
        <field name="name">l10n_ec.sri.payment.form</field>
        <field name="model">l10n_ec.sri.payment</field>
        <field name="type">form</field>
        <field name="arch" type="xml">
            <form string="Payment Method" create="0" edit="0">
                <sheet>
                    <group>
                          <field name="code"/>
                          <field name="name"/>
                          <field name="active"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    
    <record id="view_payment_method_tree" model="ir.ui.view" >
        <field name="name">l10n_ec.sri.payment.tree</field>
        <field name="model">l10n_ec.sri.payment</field>
        <field name="type">tree</field>
        <field name="arch" type="xml">
            <tree string="Payment Method" create="0" edit="0">
                <field name="code"/>
                <field name="name"/>
                <field name="active" widget="boolean_toggle"/>
            </tree>
        </field>
    </record>

    <record id="action_account_l10n_ec_sri_payment_tree" model="ir.actions.act_window">
        <field name="name">Payment Methods SRI</field>
        <field name="res_model">l10n_ec.sri.payment</field>
        <field name="view_mode">tree,form</field>
        <field name="view_id" ref="view_payment_method_tree"/>
        <field name="context">{'active_test': False}</field>
    </record>

    <menuitem id="menu_action_account_l10n_ec_sri_payment" action="action_account_l10n_ec_sri_payment_tree"
              groups="account.group_account_manager" parent="l10n_ec.sri_menu" sequence="3"/>
</odoo>

```

## File: views\l10n_latam_document_type_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_document_type_conf_form" model="ir.ui.view">
        <field name="name">view.document.type.conf.form</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_document_type_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group" position="after">
                <group string="Ecuadorian Configuration" col="2" colspan="2">
                    <field name="l10n_ec_check_format"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="view_document_type_conf_tree" model="ir.ui.view">
        <field name="name">view.document.type.conf.tree</field>
        <field name="model">l10n_latam.document.type</field>
        <field name="inherit_id" ref="l10n_latam_invoice_document.view_document_type_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='internal_type']" position="after">
                <field name="l10n_ec_check_format"/>
            </xpath>
        </field>
    </record>

</odoo>

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_form" model="ir.ui.view">
        <field name="name">res.partner.form</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <div name="warning_tax" position="after">
                <div class="alert alert-warning" role="alert" invisible="not l10n_ec_vat_validation">
                    <field name="l10n_ec_vat_validation"/>
                </div>
            </div>
        </field>
    </record>
</odoo>
```

## File: views\root_sri_menu.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem
        id="sri_menu"
        name="Ecuadorian SRI"
        parent="account.menu_finance_configuration"
        sequence="25"/>
</odoo>

```

