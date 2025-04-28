# Odoo Module: l10n_gr

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Greece - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['gr'],
    'author': 'P. Christeas, Odoo S.A.',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Greece.
==================================================================

Greek accounting chart and localization.
    """,
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'data': [
        'data/account_tax_report_data.xml',
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
        <field name="country_id" ref="base.gr"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_gr_tr_table_B" model="account.report.line">
                <field name="name">B. TABLE OF OUTPUTS – INPUTS after the reduction (according to the VAT rates) of refunds - deductions</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_gr_tr_section_a" model="account.report.line">
                        <field name="name">a. Taxable outputs - output tax</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="l10n_gr_tr_301" model="account.report.line">
                                <field name="name">301 - Sales 13%</field>
                                <field name="code">l10n_gr_301</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_301_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">301</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_302" model="account.report.line">
                                <field name="name">302 - Sales 6%</field>
                                <field name="code">l10n_gr_302</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_302_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">302</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_303" model="account.report.line">
                                <field name="name">303 - Sales 24%</field>
                                <field name="code">l10n_gr_303</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_303_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">303</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_304" model="account.report.line">
                                <field name="name">304 - Sales 9%</field>
                                <field name="code">l10n_gr_304</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_304_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">304</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_305" model="account.report.line">
                                <field name="name">305 - Sales 4%</field>
                                <field name="code">l10n_gr_305</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_305_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">305</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_306" model="account.report.line">
                                <field name="name">306 - Sales 17%</field>
                                <field name="code">l10n_gr_306</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_306_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">306</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_307" model="account.report.line">
                                <field name="name">307 - Taxable outputs</field>
                                <field name="code">l10n_gr_307</field>
                                <field name="hierarchy_level">2</field>
                                <field name="aggregation_formula">
                                    l10n_gr_301.balance +
                                    l10n_gr_302.balance +
                                    l10n_gr_303.balance +
                                    l10n_gr_304.balance +
                                    l10n_gr_305.balance +
                                    l10n_gr_306.balance
                                </field>
                            </record>
                            <record id="l10n_gr_tr_331" model="account.report.line">
                                <field name="name">331 - VAT sales 13%</field>
                                <field name="code">l10n_gr_331</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_331_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">331</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_332" model="account.report.line">
                                <field name="name">332 - VAT sales 6%</field>
                                <field name="code">l10n_gr_332</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_332_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">332</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_333" model="account.report.line">
                                <field name="name">333 - VAT sales 24%</field>
                                <field name="code">l10n_gr_333</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_333_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">333</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_334" model="account.report.line">
                                <field name="name">334 - VAT Sales 9%</field>
                                <field name="code">l10n_gr_334</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_334_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">334</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_335" model="account.report.line">
                                <field name="name">335 - VAT sales 4%</field>
                                <field name="code">l10n_gr_335</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_335_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">335</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_336" model="account.report.line">
                                <field name="name">336 - VAT sales 17%</field>
                                <field name="code">l10n_gr_336</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_336_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">336</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_337" model="account.report.line">
                                <field name="name">337 - VAT taxable outputs</field>
                                <field name="code">l10n_gr_337</field>
                                <field name="hierarchy_level">2</field>
                                <field name="aggregation_formula">
                                    l10n_gr_331.balance +
                                    l10n_gr_332.balance +
                                    l10n_gr_333.balance +
                                    l10n_gr_334.balance +
                                    l10n_gr_335.balance +
                                    l10n_gr_336.balance
                                </field>
                            </record>
                            <record id="l10n_gr_tr_342" model="account.report.line">
                                <field name="name">342 - Intra-community supplies</field>
                                <field name="code">l10n_gr_342</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_342_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">342</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_345" model="account.report.line">
                                <field name="name">345 - Intra-community supplies of services</field>
                                <field name="code">l10n_gr_345</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_345_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">345</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_348" model="account.report.line">
                                <field name="name">348 - Exports and exemptions of ships and aircrafts</field>
                                <field name="code">l10n_gr_348</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_348_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">348</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_349" model="account.report.line">
                                <field name="name">349 - Other outputs without VAT with right to deduct</field>
                                <field name="code">l10n_gr_349</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_349_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">349</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_310" model="account.report.line">
                                <field name="name">310 - Outputs exempted or excepted, without right to deduct</field>
                                <field name="code">l10n_gr_310</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_310_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">310</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_311" model="account.report.line">
                                <field name="name">311 - Total outputs</field>
                                <field name="code">l10n_gr_311</field>
                                <field name="hierarchy_level">2</field>
                                <field name="aggregation_formula">
                                    l10n_gr_307.balance +
                                    l10n_gr_342.balance +
                                    l10n_gr_345.balance +
                                    l10n_gr_348.balance +
                                    l10n_gr_349.balance +
                                    l10n_gr_310.balance
                                </field>
                            </record>
                            <record id="l10n_gr_tr_312" model="account.report.line">
                                <field name="name">312 - VAT turnover</field>
                                <field name="code">l10n_gr_312</field>
                                <field name="hierarchy_level">2</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_312_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_section_b" model="account.report.line">
                        <field name="name">b. Taxable inputs - input tax</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="l10n_gr_tr_361" model="account.report.line">
                                <field name="name">361 - Purchases and expenditures within the country</field>
                                <field name="code">l10n_gr_361</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_361_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">361</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_362" model="account.report.line">
                                <field name="name">362 - Purchases and imports of investment goods</field>
                                <field name="code">l10n_gr_362</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_362_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">362</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_363" model="account.report.line">
                                <field name="name">363 - Other imports</field>
                                <field name="code">l10n_gr_363</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_363_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">363</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_364" model="account.report.line">
                                <field name="name">364 - Intra-community acquisitions of goods</field>
                                <field name="code">l10n_gr_364</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_364_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">364</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_365" model="account.report.line">
                                <field name="name">365 - Intra-community acquisitions of services art. 14.2.a</field>
                                <field name="code">l10n_gr_365</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_365_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">365</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_366" model="account.report.line">
                                <field name="name">366 - Other reverse charge transactions</field>
                                <field name="code">l10n_gr_366</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_366_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">366</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_367" model="account.report.line">
                                <field name="name">367 - Taxable inputs total</field>
                                <field name="code">l10n_gr_367</field>
                                <field name="hierarchy_level">2</field>
                                <field name="aggregation_formula">
                                    l10n_gr_361.balance +
                                    l10n_gr_362.balance +
                                    l10n_gr_363.balance +
                                    l10n_gr_364.balance +
                                    l10n_gr_365.balance +
                                    l10n_gr_366.balance
                                </field>
                            </record>
                            <record id="l10n_gr_tr_381" model="account.report.line">
                                <field name="name">381 - VAT purchases and expenditures within the country</field>
                                <field name="code">l10n_gr_381</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_381_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">381</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_382" model="account.report.line">
                                <field name="name">382 - VAT purchases and imports of investment goods</field>
                                <field name="code">l10n_gr_382</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_382_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">382</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_383" model="account.report.line">
                                <field name="name">383 - VAT other imports</field>
                                <field name="code">l10n_gr_383</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_383_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">383</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_384" model="account.report.line">
                                <field name="name">384 - VAT intra-community acquisitions of goods</field>
                                <field name="code">l10n_gr_384</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_384_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">384</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_385" model="account.report.line">
                                <field name="name">385 - VAT Intra-Community Acquisitions of services art. 14.2.a</field>
                                <field name="code">l10n_gr_385</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_385_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">385</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_386" model="account.report.line">
                                <field name="name">386 - VAT other reverse charge transactions</field>
                                <field name="code">l10n_gr_386</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_386_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">386</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_387" model="account.report.line">
                                <field name="name">387 - VAT taxable inputs total</field>
                                <field name="code">l10n_gr_387</field>
                                <field name="hierarchy_level">2</field>
                                <field name="aggregation_formula">
                                    l10n_gr_381.balance +
                                    l10n_gr_382.balance +
                                    l10n_gr_383.balance +
                                    l10n_gr_384.balance +
                                    l10n_gr_385.balance +
                                    l10n_gr_386.balance
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_section_d" model="account.report.line">
                        <field name="name">d. Amounts added to the input tax total</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="l10n_gr_tr_400" model="account.report.line">
                                <field name="name">400 - Tax refund (sales of agricultural products x 3%)</field>
                                <field name="code">l10n_gr_400</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_400_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_402" model="account.report.line">
                                <field name="name">402 - Other added amounts</field>
                                <field name="code">l10n_gr_402</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_402_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_407" model="account.report.line">
                                <field name="name">407 - Amounts of adjustments of previous tax year for deduction</field>
                                <field name="code">l10n_gr_407</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_407_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_410" model="account.report.line">
                                <field name="name">410 - Added amounts to total inputs tax</field>
                                <field name="code">l10n_gr_410</field>
                                <field name="hierarchy_level">2</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_410_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">l10n_gr_400.balance + l10n_gr_402.balance + l10n_gr_407.balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_section_e" model="account.report.line">
                        <field name="name">e. Deductible amounts from the total input tax</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="l10n_gr_tr_411" model="account.report.line">
                                <field name="name">411 - Inputs VAT reduced under prorata scheme</field>
                                <field name="code">l10n_gr_411</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_411_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_422" model="account.report.line">
                                <field name="name">422 - Other removable amounts</field>
                                <field name="code">l10n_gr_422</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_422_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_423" model="account.report.line">
                                <field name="name">423 - Amount of adjustments of previous tax year to be paid</field>
                                <field name="code">l10n_gr_423</field>
                                <field name="hierarchy_level">3</field>
                                <field name="expression_ids">
                                    <record id="l10n_gr_tr_423_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_gr_tr_428" model="account.report.line">
                                <field name="name">428 - Amounts removable from total inputs tax</field>
                                <field name="code">l10n_gr_428</field>
                                <field name="hierarchy_level">2</field>
                                <field name="aggregation_formula">
                                    l10n_gr_411.balance +
                                    l10n_gr_422.balance +
                                    l10n_gr_423.balance
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_gr_tr_430" model="account.report.line">
                <field name="name">430 - Remaining inputs VAT</field>
                <field name="code">l10n_gr_430</field>
                <field name="hierarchy_level">1</field>
                <field name="aggregation_formula">
                    l10n_gr_387.balance +
                    l10n_gr_410.balance -
                    l10n_gr_428.balance
                </field>
            </record>
            <record id="l10n_gr_tr_table_C" model="account.report.line">
                <field name="name">C. TABLE OF TAX SETTLEMENT to be paid, deducted or refunded</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_gr_tr_470" model="account.report.line">
                        <field name="name">470 - Credit balance</field>
                        <field name="code">l10n_gr_470</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_gr_tr_470_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_gr_430.balance - l10n_gr_337.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_480" model="account.report.line">
                        <field name="name">480 - Debit balance</field>
                        <field name="code">l10n_gr_480</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_gr_tr_480_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_gr_337.balance - l10n_gr_430.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_401" model="account.report.line">
                        <field name="name">401 - Credit balance of previous taxable period</field>
                        <field name="code">l10n_gr_401</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_gr_tr_401_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="l10n_gr_tr_401_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">401</field>
                            </record>
                            <record id="l10n_gr_tr_401_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_gr_401._applied_carryover_balance + l10n_gr_401.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_403" model="account.report.line">
                        <field name="name">403 - Assessed amount of previous return of this taxable period (= code 511)</field>
                        <field name="code">l10n_gr_403</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_gr_tr_403_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_404" model="account.report.line">
                        <field name="name">404 - Tax that is committed through banks</field>
                        <field name="code">l10n_gr_404</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_gr_tr_404_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_483" model="account.report.line">
                        <field name="name">483 - Debit amount less than 30€ of previous taxable period</field>
                        <field name="code">l10n_gr_483</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_gr_tr_483_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="l10n_gr_tr_483_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">483</field>
                            </record>
                            <record id="l10n_gr_tr_483_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_gr_483._applied_carryover_balance + l10n_gr_483.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_505" model="account.report.line">
                        <field name="name">505 - Amount which has been refunded or requested to be refund</field>
                        <field name="code">l10n_gr_505</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_gr_tr_505_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_502" model="account.report.line">
                        <field name="name">502 - AMOUNT for deduction</field>
                        <field name="code">l10n_gr_502</field>
                        <field name="hierarchy_level">2</field>
                        <field name="expression_ids">
                            <record id="l10n_gr_tr_502_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    l10n_gr_470.balance +
                                    l10n_gr_401.balance +
                                    l10n_gr_403.balance +
                                    l10n_gr_404.balance -
                                    l10n_gr_480.balance -
                                    l10n_gr_483.balance -
                                    l10n_gr_505.balance
                                </field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                            <record id="l10n_gr_tr_502_carryover" model="account.report.expression">
                                <field name="label">_carryover_balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_gr_502.balance</field>
                                <field name="carryover_target">l10n_gr_401._applied_carryover_balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_503" model="account.report.line">
                        <field name="name">503 - AMOUNT REQUESTED for refund</field>
                        <field name="code">l10n_gr_503</field>
                        <field name="hierarchy_level">2</field>
                        <field name="expression_ids">
                            <record id="l10n_gr_tr_503_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_gr_tr_511" model="account.report.line">
                        <field name="name">511 - AMOUNT TO BE PAID</field>
                        <field name="code">l10n_gr_511</field>
                        <field name="hierarchy_level">2</field>
                        <field name="expression_ids">
                            <record id="l10n_gr_tr_511_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">
                                    l10n_gr_480.balance +
                                    l10n_gr_483.balance +
                                    l10n_gr_505.balance -
                                    l10n_gr_470.balance -
                                    l10n_gr_401.balance -
                                    l10n_gr_403.balance -
                                    l10n_gr_404.balance
                                </field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                            <record id="l10n_gr_tr_511_carryover_amount" model="account.report.expression">
                                <field name="label">_carryover_balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_gr_511.balance</field>
                                <field name="subformula">if_below(EUR(30))</field>
                                <field name="carryover_target">l10n_gr_483._applied_carryover_balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-gr.csv

```csv
"id","name","code","account_type","reconcile","tag_ids","name@gr"
"l10n_gr_10_01","Land - Gross amount (cost or revaluation) of land","10.01.00","asset_fixed","False","","Μικτή αξία (κόστος ή αναπροσαρμοσμένη) γης"
"l10n_gr_10_02","Land - Accumulated impairment","10.02.00","asset_fixed","False","","Σωρευμένες απομειώσεις γης"
"l10n_gr_11_00","Depreciable land improvements","11.00.00","asset_fixed","False","","Διαμορφώσεις γης υποκείμενες σε απόσβεση"
"l10n_gr_11_01","Depreciable land improvements - Gross amount (cost or revaluation) of depreciable land improvements","11.01.00","asset_fixed","False","","Μικτή αξία (κόστος ή αναπροσαρμοσμένη) διαμορφώσεων γης"
"l10n_gr_11_02","Depreciable land improvements - Accumulated depreciation","11.02.00","asset_fixed","False","","Σωρευμένες αποσβέσεις διαμορφώσεων γης"
"l10n_gr_11_03","Depreciable land improvements - Accumulated impairment","11.03.00","asset_fixed","False","","Σωρευμένες απομειώσεις διαμορφώσεων γης"
"l10n_gr_12_01","Buildings and physical infrastructure - Gross amount (cost or revaluation)","12.01.00","asset_fixed","False","","Μικτή αξία (κόστος ή αναπροσαρμοσμένη) κτηρίων - τεχνικών έργων"
"l10n_gr_12_02","Buildings and physical infrastructure - Accumulated depreciation","12.02.00","asset_fixed","False","","Σωρευμένες αποσβέσεις κτηρίων - τεχνικών έργων"
"l10n_gr_12_03","Buildings and physical infrastructure - Accumulated impairment","12.03.00","asset_fixed","False","","Σωρευμένες απομειώσεις κτηρίων - τεχνικών έργων"
"l10n_gr_13_01","Machinery - Acquisition cost","13.01.00","asset_fixed","False","","Αξία κτήσης μηχανολογικού εξοπλισμού"
"l10n_gr_13_02","Machinery - Accumulated depreciation","13.02.00","asset_fixed","False","","Σωρευμένες αποσβέσεις μηχανολογικού εξοπλισμού"
"l10n_gr_13_03","Machinery - Accumulated impairment","13.03.00","asset_fixed","False","","Σωρευμένες απομειώσεις μηχανολογικού εξοπλισμού"
"l10n_gr_14_01","Transportation equipment - Acquisition cost","14.01.00","asset_fixed","False","","Μικτή αξία κτήσης μεταφορικών μέσων"
"l10n_gr_14_02","Transportation equipment - Accumulated depreciation","14.02.00","asset_fixed","False","","Σωρευμένες αποσβέσεις μεταφορικών μέσων"
"l10n_gr_14_03","Transportation equipment - Accumulated impairment","14.03.00","asset_fixed","False","","Σωρευμένες απομειώσεις μεταφορικών μέσων"
"l10n_gr_15_01","Other equipment - Acquisition cost","15.01.00","asset_fixed","False","","Μικτή αξία κτήσης εξοπλισμού"
"l10n_gr_15_02","Other equipment - Accumulated depreciation","15.02.00","asset_fixed","False","","Σωρευμένες αποσβέσεις εξοπλισμού"
"l10n_gr_15_03","Other equipment - Accumulated impairment","15.03.00","asset_fixed","False","","Σωρευμένες απομειώσεις εξοπλισμού"
"l10n_gr_16_01","Investment property - Gross amount (cost or revaluation)","16.01.00","asset_fixed","False","","Μικτή αξία (κόστος ή αναπροσαρμοσμένη) επενδύσεων σε ακίνητα"
"l10n_gr_16_02","Investment property - Accumulated depreciation","16.02.00","asset_fixed","False","","Σωρευμένες αποσβέσεις επενδύσεων σε ακίνητα"
"l10n_gr_16_03","Investment property - Accumulated impairment","16.03.00","asset_fixed","False","","Σωρευμένες απομειώσεις επενδύσεων σε ακίνητα"
"l10n_gr_17_01_01","Biological fixed assets - Living animals - Gross amount (cost or revaluation)","17.01.01","asset_fixed","False","","Μικτή αξία (κόστος ή αναπροσαρμοσμένη) ζώντων ζώων"
"l10n_gr_17_01_02","Biological fixed assets - Living animals - Accumulated depreciation","17.01.02","asset_fixed","False","","Σωρευμένες αποσβέσεις ζώντων ζώων"
"l10n_gr_17_01_03","Biological fixed assets - Living animals - Accumulated impairment","17.01.03","asset_fixed","False","","Σωρευμένες απομειώσεις ζώντων ζώων"
"l10n_gr_17_02_01","Biological fixed assets - Trees and plants - Gross amount (cost or revaluation)","17.02.01","asset_fixed","False","","Μικτή αξία (κόστος ή αναπροσαρμοσμένη) δένδρων και φυτών"
"l10n_gr_17_02_02","Biological fixed assets - Trees and plants - Accumulated depreciation","17.02.02","asset_fixed","False","","Σωρευμένες αποσβέσεις δένδρων και φυτών"
"l10n_gr_17_02_03","Biological fixed assets - Trees and plants - Accumulated impairment","17.02.03","asset_fixed","False","","Σωρευμένες απομειώσεις δένδρων και φυτών"
"l10n_gr_18_01_01_01","Acquisition cost of development expenditure - Ongoing expenditure","18.01.01.01","asset_non_current","False","","Μικτή αξία κτήσης δαπανών ανάπτυξης - τρέχουσες δαπάνες"
"l10n_gr_18_01_01_02","Acquisition cost of development expenditure - Except ongoing expenditure","18.01.01.02","asset_non_current","False","","Μικτή αξία κτήσης δαπανών ανάπτυξης - εκτός των συνεχιζόμενων δαπανών"
"l10n_gr_18_01_02_01","Accumulated depreciation of development expenditure - Ongoing expenditure","18.01.02.01","asset_non_current","False","","Σωρευμένες αποσβέσεις δαπανών ανάπτυξης - τρέχουσες δαπάνες"
"l10n_gr_18_01_02_02","Accumulated depreciation of development expenditure - Except ongoing expenditure","18.01.02.02","asset_non_current","False","","Σωρευμένες αποσβέσεις δαπανών ανάπτυξης - εκτός των συνεχιζόμενων δαπανών"
"l10n_gr_18_01_03_01","Accumulated impairment of development expenditure - Ongoing expenditure","18.01.03.01","asset_non_current","False","","Σωρευμένες απομειώσεις δαπανών ανάπτυξης - τρέχουσες δαπάνες"
"l10n_gr_18_01_03_02","Accumulated impairment of development expenditure - Except ongoing expenditure","18.01.03.02","asset_non_current","False","","Σωρευμένες απομειώσεις δαπανών ανάπτυξης - εκτός των συνεχιζόμενων δαπανών"
"l10n_gr_18_02_01","Acquisition cost of goodwill","18.02.01","asset_non_current","False","","Μικτή αξία κτήσης υπεραξίας"
"l10n_gr_18_02_02","Accumulated depreciation of goodwill","18.02.02","asset_non_current","False","","Σωρευμένες αποσβέσεις υπεραξίας"
"l10n_gr_18_02_03","Accumulated impairment of goodwill","18.02.03","asset_non_current","False","","Σωρευμένες απομειώσεις υπεραξίας"
"l10n_gr_18_03_01","Acquisition cost of other intangibles","18.03.01","asset_non_current","False","","Μικτή αξία κτήσης λοιπών άυλων"
"l10n_gr_18_03_02","Accumulated depreciation of other intangibles","18.03.02","asset_non_current","False","","Σωρευμένες αποσβέσεις λοιπών άυλων"
"l10n_gr_18_03_03","Accumulated impairment of other intangibles","18.03.03","asset_non_current","False","","Σωρευμένες απομειώσεις λοιπών άυλων"
"l10n_gr_20_01","Merchandise - Opening balance","20.01.00","asset_current","False","","Εμπορεύματα έναρξης"
"l10n_gr_20_02","Purchases of merchandise","20.02.00","asset_current","False","","Αγορές εμπορευμάτων χρήσης"
"l10n_gr_20_03","Discounts and allowances of merchandise","20.03.00","asset_current","False","","Εκπτώσεις αγορών εμπορευμάτων"
"l10n_gr_20_04","Returns of merchandise","20.04.00","asset_current","False","","Επιστροφές αγορών εμπορευμάτων"
"l10n_gr_20_05","Write-downs of merchandise","20.05.00","asset_current","False","","Απομείωση εμπορευμάτων"
"l10n_gr_20_06","Merchandise - Closing balance","20.06.00","asset_current","False","","Εμπορεύματα λήξης"
"l10n_gr_21_01","Finished products - Opening balance","21.01.00","asset_current","False","","Προϊόντα έναρξης"
"l10n_gr_21_02","Production of goods","21.02.00","asset_current","False","","Παραγωγή χρήσης"
"l10n_gr_21_03","Write-downs of goods","21.03.00","asset_current","False","","Απομείωση προϊόντων"
"l10n_gr_21_04","Finished products - Closing balance","21.04.00","asset_current","False","","Προϊόντα λήξης"
"l10n_gr_22_01_01","Living Animals - Opening balance","22.01.01","asset_current","False","","Ζώντα ζώα έναρξης"
"l10n_gr_22_01_02","Purchases of living animals","22.01.02","asset_current","False","","Αγορές ζώντων ζώων"
"l10n_gr_22_01_03","Discounts and allowances on living animals","22.01.03","asset_current","False","","Εκπτώσεις αγορών ζώντων ζώων"
"l10n_gr_22_01_04","Returns of living animals","22.01.04","asset_current","False","","Επιστροφές αγορών ζώντων ζώων"
"l10n_gr_22_01_05","Write-downs of living animals","22.01.05","asset_current","False","","Απομείωση ζώντων ζώων"
"l10n_gr_22_01_06","Fair value differences - Living animals","22.01.06","asset_current","False","","Διαφορές επιμέτρησης εύλογης αξίας ζώντων ζώων"
"l10n_gr_22_01_07","Living Animals - Closing balance","22.01.07","asset_current","False","","Ζώντα ζώα λήξης"
"l10n_gr_22_02_01","Trees and plants - Opening balance","22.02.01","asset_current","False","","Δένδρα και φυτά έναρξης"
"l10n_gr_22_02_02","Purchases of trees and plants","22.02.02","asset_current","False","","Αγορές δένδρων και φυτών"
"l10n_gr_22_02_03","Discounts and allowances on trees and plants","22.02.03","asset_current","False","","Εκπτώσεις αγορών δένδρων και φυτών"
"l10n_gr_22_02_04","Returns of trees and plants","22.02.04","asset_current","False","","Επιστροφές αγορών δένδρων και φυτών"
"l10n_gr_22_02_05","Write-downs of trees and plants","22.02.05","asset_current","False","","Απομείωση δένδρων και φυτών"
"l10n_gr_22_02_06","Fair value differences of trees and plants","22.02.06","asset_current","False","","Διαφορές επιμέτρησης εύλογης αξίας δένδρων και φυτών"
"l10n_gr_22_02_07","Trees and plants - Closing balance","22.02.07","asset_current","False","","Δένδρα και φυτά λήξης"
"l10n_gr_23_01","Opening balance of work in progress","23.01.00","asset_current","False","","Παραγωγή σε εξέλιξη έναρξης"
"l10n_gr_23_02","Closing balance of work in progress","23.02.00","asset_current","False","","Παραγωγή σε εξέλιξη λήξης"
"l10n_gr_24_01","Materials - Opening balance of materials","24.01.00","asset_current","False","","Πρώτες ύλες και υλικά έναρξης"
"l10n_gr_24_02","Purchases of materials","24.02.00","asset_current","False","","Αγορές πρώτων υλών και υλικών χρήσης"
"l10n_gr_24_03","Discounts and allowances of materials","24.03.00","asset_current","False","","Εκπτώσεις αγορών πρώτων υλών και υλικών"
"l10n_gr_24_04","Returns of materials","24.04.00","asset_current","False","","Επιστροφές αγορών πρώτων υλών και υλικών"
"l10n_gr_24_05","Write-downs of materials","24.05.00","asset_current","False","","Απομείωση πρώτων υλών και υλικών"
"l10n_gr_24_06","Materials - Closing balance","24.06.00","asset_current","False","","Αποθέματα λήξης πρώτων υλών και υλικών"
"l10n_gr_25_01","Packing materials - Opening balance","25.01.00","asset_current","False","","Υλικά συσκευασίας έναρξης"
"l10n_gr_25_02","Purchases of packing materials","25.02.00","asset_current","False","","Αγορές υλικών συσκευασίας"
"l10n_gr_25_03","Discounts and allowances of packing materials","25.03.00","asset_current","False","","Εκπτώσεις αγορών υλικών συσκευασίας"
"l10n_gr_25_04","Returns of packing materials","25.04.00","asset_current","False","","Επιστροφές αγορών υλικών συσκευασίας"
"l10n_gr_25_05","Write-downs of packing materials","25.05.00","asset_current","False","","Απομείωση υλικών συσκευασίας"
"l10n_gr_25_06","Packing materials - Closing balance","25.06.00","asset_current","False","","Υλικά συσκευασίας λήξης"
"l10n_gr_26_01","Spare parts - Opening balance","26.01.00","asset_current","False","","Ανταλλακτικά παγίων έναρξης"
"l10n_gr_26_02","Purchases of spare parts","26.02.00","asset_current","False","","Αγορές ανταλλακτικών παγίων"
"l10n_gr_26_03","Discounts and allowances of spare parts","26.03.00","asset_current","False","","Εκπτώσεις αγορών ανταλλακτικών παγίων"
"l10n_gr_26_04","Returns of spare parts","26.04.00","asset_current","False","","Επιστροφές αγορών ανταλλακτικών παγίων"
"l10n_gr_26_05","Write-downs of spare parts","26.05.00","asset_current","False","","Απομείωση ανταλλακτικών"
"l10n_gr_26_06","Spare parts - Closing balance","26.06.00","asset_current","False","","Ανταλλακτικά παγίων λήξης"
"l10n_gr_27_01","Other inventory - Opening balance","27.01.00","asset_current","False","","Λοιπά αποθέματα έναρξης"
"l10n_gr_27_02","Purchases of other inventory","27.02.00","asset_current","False","","Αγορές λοιπών αποθεμάτων"
"l10n_gr_27_03","Discounts and allowances of other inventory","27.03.00","asset_current","False","","Εκπτώσεις αγορών λοιπών αποθεμάτων"
"l10n_gr_27_04","Returns of other inventory","27.04.00","asset_current","False","","Επιστροφές αγορών λοιπών αποθεμάτων"
"l10n_gr_27_05","Write-downs of other inventory","27.05.00","asset_current","False","","Απομείωση λοιπών αποθεμάτων"
"l10n_gr_27_06","Other inventory - Closing balance","27.06.00","asset_current","False","","Λοιπά αποθέματα λήξης"
"l10n_gr_30_01_01_01","Accounts receivables from non-related entities - Nominal amount - Short term portion","30.01.01.01","asset_receivable","True","","Πελάτες μη συνδεδεμένες οντότητες - ονομαστικό ποσό - βραχυπρόθεσμο μέρος"
"l10n_gr_30_01_01_02","Accounts receivables from non-related entities - Nominal amount - Long term portion","30.01.01.02","asset_receivable","True","","Πελάτες μη συνδεδεμένες οντότητες - ονομαστικό ποσό - μακροπρόθεσμα"
"l10n_gr_30_01_02_01","Interest not accrued on receivables from non-related entities - Short term portion","30.01.02.01","asset_current","False","","Μη δουλευμένοι τόκοι μη συνδεδεμένων πελατών - βραχυπρόθεσμο μέρος"
"l10n_gr_30_01_02_02","Interest not accrued on receivables from non-related entities - Long term portion","30.01.02.02","asset_non_current","False","","Μη δουλευμένοι τόκοι μη συνδεδεμένων πελατών - μακροπρόθεσμα"
"l10n_gr_30_01_03","Advances from customers - Non-related entities","30.01.03","liability_current","False","","Προκαταβολές μη συνδεδεμένων πελατών"
"l10n_gr_30_01_04_01","Impairment of receivables from non-related entities - Short term portion","30.01.04.01","asset_current","False","","Απομείωση μη συνδεδεμένων πελατών - βραχυπρόθεσμο μέρος"
"l10n_gr_30_01_04_02","Impairment of receivables from non-related entities - Long term portion","30.01.04.02","asset_current","False","","Απομείωση μη συνδεδεμένων πελατών - μακροπρόθεσμα"
"l10n_gr_30_02_01_01","Accounts receivables from related entities - Nominal amount - Short term portion","30.02.01.01","asset_receivable","True","","Συνδεδεμένοι πελάτες - ονομαστικό ποσό - βραχυπρόθεσμο μέρος"
"l10n_gr_30_02_01_02","Accounts receivables from related entities - Nominal amount - Long term portion","30.02.01.02","asset_receivable","True","","Συνδεδεμένοι πελάτες - ονομαστικό ποσό - μακροπρόθεσμα"
"l10n_gr_30_02_02_01","Interest not-accrued on receivables from related entities - Short term portion","30.02.02.01","asset_current","False","","Μη δουλευμένοι τόκοι συνδεδεμένων πελατών - βραχυπρόθεσμο μέρος"
"l10n_gr_30_02_02_02","Interest not-accrued on receivables from related entities - Long term portion","30.02.02.02","asset_non_current","False","","Μη δουλευμένοι τόκοι συνδεδεμένων πελατών - μακροπρόθεσμα"
"l10n_gr_30_02_03","Advances from customers - Related entities","30.02.03","liability_current","False","","Προκαταβολές συνδεδεμένων πελατών"
"l10n_gr_30_02_04_01","Impairment of receivables from related entities - Short term portion","30.02.04.01","asset_current","False","","Απομείωση συνδεδεμένων πελατών - βραχυπρόθεσμο μέρος"
"l10n_gr_30_02_04_02","Impairment of receivables from related entities - Long term portion","30.02.04.02","asset_current","False","","Απομείωση συνδεδεμένων πελατών - μακροπρόθεσμα"
"l10n_gr_31_01_01_01","Notes receivables from non-related entities - Nominal amount - Short term portion","31.01.01.01","asset_receivable","True","","Αξιόγραφα εμπορικών απαιτήσεων μη συνδεδεμένων οντοτήτων - ονομαστικό ποσό - βραχυπρόθεσμο μέρος"
"l10n_gr_31_01_01_02","Notes receivables from non-related entities - Nominal amount - Long term portion","31.01.01.02","asset_receivable","True","","Αξιόγραφα εμπορικών απαιτήσεων μη συνδεδεμένων οντοτήτων - ονομαστικό ποσό - μακροπρόθεσμα"
"l10n_gr_31_01_02_01","Interest not-accrued on receivables from non-related entities - Short term portion","31.01.02.01","asset_current","False","","Μη δουλευμένοι τόκοι αξιογράφων εμπορικών απαιτήσεων μη συνδεδεμένων οντοτήτων - βραχυπρόθεσμο μέρος"
"l10n_gr_31_01_02_02","Interest not-accrued on receivables from non-related entities - Long term portion","31.01.02.02","asset_non_current","False","","Μη δουλευμένοι τόκοι αξιογράφων εμπορικών απαιτήσεων μη συνδεδεμένων οντοτήτων - μακροπρόθεσμα"
"l10n_gr_31_01_03_01","Impairment of receivables from non-related entities - Short term portion","31.01.03.01","asset_current","False","","Απομείωση αξιογράφων εμπορικών απαιτήσεων μη συνδεδεμένων οντοτήτων - βραχυπρόθεσμο μέρος"
"l10n_gr_31_01_03_02","Impairment of receivables from non-related entities - Long term portion","31.01.03.02","asset_non_current","False","","Απομείωση αξιογράφων εμπορικών απαιτήσεων μη συνδεδεμένων οντοτήτων - μακροπρόθεσμα"
"l10n_gr_31_02_01_01","Notes receivables from related entities - Nominal amount - Short term portion","31.02.01.01","asset_receivable","True","","Αξιόγραφα εμπορικών απαιτήσεων συνδεδεμένων οντοτήτων - ονομαστικό ποσό - βραχυπρόθεσμο μέρος"
"l10n_gr_31_02_01_02","Notes receivables from related entities - Nominal amount - Long term portion","31.02.01.02","asset_receivable","True","","Αξιόγραφα εμπορικών απαιτήσεων συνδεδεμένων οντοτήτων - ονομαστικό ποσό - μακροπρόθεσμα"
"l10n_gr_31_02_02_01","Interest not-accrued on receivables from related entities - Short term portion","31.02.02.01","asset_current","False","","Μη δουλευμένοι τόκοι αξιογράφων εμπορικών απαιτήσεων συνδεδεμένων οντοτήτων - βραχυπρόθεσμο μέρος"
"l10n_gr_31_02_02_02","Interest not-accrued on receivables from related entities - Long term portion","31.02.02.02","asset_non_current","False","","Μη δουλευμένοι τόκοι αξιογράφων εμπορικών απαιτήσεων συνδεδεμένων οντοτήτων - μακροπρόθεσμα"
"l10n_gr_31_02_03_01","Impairment of receivables from related entities - Short term portion","31.02.03.01","asset_current","False","","Απομείωση αξιογράφων εμπορικών απαιτήσεων συνδεδεμένων οντοτήτων - βραχυπρόθεσμο μέρος"
"l10n_gr_31_02_03_02","Impairment of receivables from related entities - Long term portion","31.02.03.02","asset_non_current","False","","Απομείωση αξιογράφων εμπορικών απαιτήσεων συνδεδεμένων οντοτήτων - μακροπρόθεσμα"
"l10n_gr_32_01_01","Loans given to related parties - Short term portion","32.01.01","asset_current","False","","Δάνεια χορηγηθέντα σε συνδεδεμένες οντότητες - βραχυπρόθεσμο μέρος"
"l10n_gr_32_01_02","Loans given to related parties - Long term portion","32.01.02","asset_non_current","False","","Δάνεια χορηγηθέντα σε συνδεδεμένες οντότητες - μακροπρόθεσμα"
"l10n_gr_32_02_01","Loans given to personnel and management - Short term portion","32.02.01","asset_current","False","","Δάνεια χορηγηθέντα στο προσωπικό και στη διοίκηση - βραχυπρόθεσμο μέρος"
"l10n_gr_32_02_02","Loans given to personnel and management - Long term portion","32.02.02","asset_non_current","False","","Δάνεια χορηγηθέντα στο προσωπικό και στη διοίκηση - μακροπρόθεσμα"
"l10n_gr_32_03_01","Other loans given - Short term portion","32.03.01","asset_current","False","","Λοιπά χορηγηθέντα δάνεια - βραχυπρόθεσμο μέρος"
"l10n_gr_32_03_02","Other loans given - Long term portion","32.03.02","asset_non_current","False","","Λοιπά χορηγηθέντα δάνεια - μακροπρόθεσμα"
"l10n_gr_32_04_01","Impairment of loans given - Short term portion","32.04.01","asset_current","False","","Απομείωση χορηγηθέντων δανείων - βραχυπρόθεσμο μέρος"
"l10n_gr_32_04_02","Impairment of loans given - Long term portion","32.04.02","asset_non_current","False","","Απομείωση χορηγηθέντων δανείων - μακροπρόθεσμα"
"l10n_gr_33_01_01_01","Revenue from equity investments receivable - Nominal amount - Short term portion","33.01.01.01","asset_current","False","","Έσοδα από πάσης φύσεως συμμετοχές εισπρακτέα - ονομαστικό ποσό - βραχυπρόθεσμο μέρος"
"l10n_gr_33_01_01_02","Revenue from equity investments receivable - Nominal amount - Long term portion","33.01.01.02","asset_non_current","False","","Έσοδα από πάσης φύσεως συμμετοχές εισπρακτέα - ονομαστικό ποσό - μακροπρόθεσμα"
"l10n_gr_33_01_02_01","Impairment of revenue from equity investments receivable - Short term portion","33.01.02.01","asset_current","False","","Απομείωση - έσοδα από πάσης φύσεως συμμετοχές εισπρακτέα - βραχυπρόθεσμο μέρος"
"l10n_gr_33_01_02_02","Impairment of revenue from equity investments receivable - Long term portion","33.01.02.02","asset_non_current","False","","Απομείωση - έσοδα από πάσης φύσεως συμμετοχές εισπρακτέα - μακροπρόθεσμα"
"l10n_gr_33_02_01_01","Other receivables from related entities - Nominal amount - Short term portion","33.02.01.01","asset_receivable","True","","Άλλες απαιτήσεις από συνδεδεμένες οντότητες - ονομαστικό ποσό - βραχυπρόθεσμο μέρος"
"l10n_gr_33_02_01_02","Other receivables from related entities - Nominal amount - Long term portion","33.02.01.02","asset_receivable","True","","Άλλες απαιτήσεις από συνδεδεμένες οντότητες - ονομαστικό ποσό - μακροπρόθεσμα"
"l10n_gr_33_02_02_01","Impairment of other receivables from related entities - Short term portion","33.02.02.01","asset_current","False","","Απομείωση - άλλες απαιτήσεις από συνδεδεμένες οντότητες - βραχυπρόθεσμο μέρος"
"l10n_gr_33_02_02_02","Impairment of other receivables from related entities - Long term portion","33.02.02.02","asset_non_current","False","","Απομείωση - άλλες απαιτήσεις από συνδεδεμένες οντότητες - μακροπρόθεσμα"
"l10n_gr_33_03_01_01","Other receivables from non-related entities, nominal amount - Short term portion","33.03.01.01","asset_receivable","True","","Άλλες απαιτήσεις από μη συνδεδεμένες οντότητες - ονομαστικό ποσό - βραχυπρόθεσμο μέρος"
"l10n_gr_33_03_01_02","Other receivables from non-related entities, nominal amount - Long term portion","33.03.01.02","asset_receivable","True","","Άλλες απαιτήσεις από μη συνδεδεμένες οντότητες - ονομαστικό ποσό - μακροπρόθεσμα"
"l10n_gr_33_03_02_01","Impairment of other receivables from non-related entities - Short term portion","33.03.02.01","asset_current","False","","Απομείωση - άλλες απαιτήσεις από μη συνδεδεμένες οντότητες - βραχυπρόθεσμο μέρος"
"l10n_gr_33_03_02_02","Impairment of other receivables from non-related entities - Long term portion","33.03.02.02","asset_non_current","False","","Απομείωση - άλλες απαιτήσεις από μη συνδεδεμένες οντότητες - μακροπρόθεσμα"
"l10n_gr_33_04_01","Guarantees - Short term portion","33.04.01","asset_current","False","","Εγγυήσεις - βραχυπρόθεσμο μέρος"
"l10n_gr_33_04_02","Guarantees - Long term portion","33.04.02","asset_non_current","False","","Εγγυήσεις - μακροπρόθεσμα"
"l10n_gr_34_01","Debt instruments","34.01.01","asset_current","False","","Χρεωστικοί τίτλοι"
"l10n_gr_34_02","Other equity instruments","34.02.00","asset_current","False","","Λοιποί συμμετοχικοί τίτλοι"
"l10n_gr_34_03_01","Other current financial assets","34.03.01","asset_current","False","","Λοιπά κυκλοφορούντα χρηματοοικονομικά περιουσιακά στοιχεία"
"l10n_gr_34_03_02","Other non current financial assets","34.03.02","asset_current","False","","Λοιπά μη κυκλοφορούντα χρηματοοικονομικά περιουσιακά στοιχεία"
"l10n_gr_35_01","Financial instruments held for fair value hedging","35.01.00","asset_current","False","","Χρηματοοικονομικά στοιχεία για αντιστάθμιση εύλογης αξίας"
"l10n_gr_35_02","Financial instruments held for cash flow hedging","35.02.00","asset_current","False","","Χρηματοοικονομικά στοιχεία για αντιστάθμιση ταμειακών ροών"
"l10n_gr_36_01_01","Investment in subsidiaries","36.01.01","asset_current","False","","Συμμετοχές σε θυγατρικές"
"l10n_gr_36_01_02","Impairment of investment in subsidiaries","36.01.02","asset_current","False","","Απομείωση συμμετοχών σε θυγατρικές"
"l10n_gr_36_02_01","Investment in associates","36.02.01","asset_current","False","","Συμμετοχές σε συγγενείς"
"l10n_gr_36_02_02","Impairment of investment in associates","36.02.02","asset_current","False","","Απομείωση συμμετοχών σε συγγενείς"
"l10n_gr_36_03_01","Investment in joint ventures","36.03.01","asset_current","False","","Συμμετοχές σε κοινοπραξίες"
"l10n_gr_36_03_02","Impairment of investment in joint ventures","36.03.02","asset_current","False","","Απομείωση συμμετοχών σε κοινοπραξίες"
"l10n_gr_37_01_01","Prepaid expenses to non-related entities","37.01.01","asset_prepayments","False","","Προπληρωμένα έξοδα σε μη συνδεδεμένες οντότητες"
"l10n_gr_37_01_02","Prepaid expenses to related entities","37.01.02","asset_prepayments","False","","Προπληρωμένα έξοδα σε συνδεδεμένες οντότητες"
"l10n_gr_37_02_01","Accrued revenue from non-related entities","37.02.01","asset_current","False","","Δουλευμένα έσοδα περιόδου από μη συνδεδεμένες οντότητες"
"l10n_gr_37_02_02","Accrued revenue from related entities","37.02.02","asset_current","False","","Δουλευμένα έσοδα περιόδου από συνδεδεμένες οντότητες"
"l10n_gr_38_01","Cash in hand","38.01.00","asset_cash","False","","Ταμείο"
"l10n_gr_38_02","Sight deposits","38.02.00","asset_cash","False","","Καταθέσεις όψεως"
"l10n_gr_38_03","Time deposits","38.03.00","asset_cash","False","","Καταθέσεις προθεσμίας"
"l10n_gr_38_04","Other cash equivalents","38.04.00","asset_cash","False","","Λοιπά ταμειακά ισοδύναμα"
"l10n_gr_39_00","Deferred tax asset","39.00.00","asset_current","False","","Αναβαλλόμενοι φόροι ενεργητικού"
"l10n_gr_40_00","Capital (paid-in)","40.00.00","equity","False","","Κεφάλαιο"
"l10n_gr_41_00","Share premium","41.00.00","equity","False","","Υπέρ το άρτιο"
"l10n_gr_42_00","Owners' deposits","42.00.00","equity","False","","Καταθέσεις ιδιοκτητών"
"l10n_gr_43_01","Acquisition cost of treasury titles","43.01.00","equity","False","","Αξία κτήσης ίδιων τίτλων"
"l10n_gr_43_02","Gain/Loss from the sale of treasury titles","43.02.00","equity","False","","Αποτέλεσμα (κέρδος/ζημία) από τη διάθεση ίδιων τίτλων"
"l10n_gr_44_01","Fair value reserves - Tangible assets","44.01.00","equity","False","","Διαφορές εύλογης αξίας ενσώματων παγίων"
"l10n_gr_44_02","Fair value reserves - Available for sale","44.02.00","equity","False","","Διαφορές εύλογης αξίας διαθέσιμων για πώληση"
"l10n_gr_44_03","Fair value reserves - Cash flow hedges","44.03.00","equity","False","","Διαφορές εύλογης αξίας στοιχείων αντιστάθμισης ταμειακών ροών"
"l10n_gr_45_00","Exchange differences","45.00.00","equity","False","","Συναλλαγματικές διαφορές"
"l10n_gr_46_00","Legal reserves","46.00.00","equity","False","","Αποθεματικά νόμων"
"l10n_gr_47_00","Tax reserves","47.00.00","equity","False","","Αφορολόγητα αποθεματικά"
"l10n_gr_48_01","Articles of association reserves","48.01.00","equity","False","","Αποθεματικά καταστατικού"
"l10n_gr_48_02","Optional reserves formed by a decision of the owners’ general assembly","48.02.00","equity","False","","Προαιρετικά αποθεματικά αποφάσεων γενικής συνέλευσης ιδιοκτητών"
"l10n_gr_49_00","Retained earnings","49.00.00","equity_unaffected","False","","Αποτελέσματα εις νέο"
"l10n_gr_50_01_01","Accounts payable - Non-related entities - Short term part","50.01.01","liability_payable","True","","Πληρωτέοι λογαριασμοί - μη συνδεδεμένες οντότητες - βραχυπρόθεσμο μέρος"
"l10n_gr_50_01_02","Accounts payable - Non-related entities - Long term portion","50.01.02","liability_payable","True","","Πληρωτέοι λογαριασμοί - μη συνδεδεμένες οντότητες - μακροπρόθεσμο μέρος"
"l10n_gr_50_02_01","Accounts payable - Related entities - Short term part","50.02.01","liability_payable","True","","Πληρωτέοι λογαριασμοί - συνδεδεμένες οντότητες - βραχυπρόθεσμο μέρος"
"l10n_gr_50_02_02","Accounts payable - Related entities - Long term portion","50.02.02","liability_payable","True","","Πληρωτέοι λογαριασμοί - συνδεδεμένες οντότητες - μακροπρόθεσμο τμήμα"
"l10n_gr_50_03_01","Advances for non-current assets, non-related entities","50.03.01","asset_non_current","False","","Προκαταβολές σε προμηθευτές για μη κυκλοφορούντο στοιχεία - μη συνδεδεμένες οντότητες"
"l10n_gr_50_03_02","Advances for inventory, non-related entities","50.03.02","asset_current","False","","Προκαταβολές σε προμηθευτές για αποθέματα - μη συνδεδεμένες οντότητες"
"l10n_gr_50_03_03","Other advances to suppliers, non-related entities","50.03.03","asset_current","False","","Λοιπές προκαταβολές σε προμηθευτές - μη συνδεδεμένες οντότητες"
"l10n_gr_50_04_01","Advances for non-current assets, related entities","50.04.01","asset_non_current","False","","Προκαταβολές σε προμηθευτές για μη κυκλοφορούντα στοιχεία - συνδεδεμένες οντότητες"
"l10n_gr_50_04_02","Advances for inventory, related entities","50.04.02","asset_current","False","","Προκαταβολές σε προμηθευτές για αποθέματα - συνδεδεμένες οντότητες"
"l10n_gr_50_04_03","Other advances to suppliers, related entities","50.04.03","asset_current","False","","Λοιπές προκαταβολές σε προμηθευτές - συνδεδεμένες οντότητες"
"l10n_gr_51_01_01","Notes payable to non-related entities - short term part","51.01.01","liability_payable","True","","Αξιόγραφα εμπορικών υποχρεώσεων — μη συνδεδεμένες οντότητες - βραχυπρόθεσμο τμήμα"
"l10n_gr_51_01_02","Notes payable to non-related entities - Long term portion","51.01.02","liability_payable","True","","Αξιόγραφα εμπορικών υποχρεώσεων — μη συνδεδεμένες οντότητες - μη βραχυπρόθεσμο μέρος"
"l10n_gr_51_02_01","Notes payable to related entities - Short term part","51.02.01","liability_payable","True","","Αξιόγραφα εμπορικών υποχρεώσεων - συνδεδεμένες οντότητες - βραχυπρόθεσμο τμήμα"
"l10n_gr_51_02_02","Notes payable to related entities - Long term portion","51.02.02","liability_payable","True","","Αξιόγραφα εμπορικών υποχρεώσεων - συνδεδεμένες οντότητες - μη βραχυπρόθεσμο μέρος"
"l10n_gr_52_01_01","Bank loans from non-related entities - Short term","52.01.01","liability_current","False","","Τραπεζικά δάνεια από μη συνδεδεμένες οντότητες - βραχυπρόθεσμα"
"l10n_gr_52_01_02","Bank loans from non-related entities - Long term","52.01.02","liability_non_current","False","","Τραπεζικά δάνεια από μη συνδεδεμένες οντότητες - μακροπρόθεσμα"
"l10n_gr_52_01_03","Bank loans from non-related entities - Short term part of long term loans","52.01.03","liability_current","False","","Τραπεζικά δάνεια από μη συνδεδεμένες οντότητες - βραχυπρόθεσμο μέρος μακροπρόθεσμων δανείων"
"l10n_gr_52_02_01","Bank loans from related entities - Short term","52.02.01","liability_current","False","","Τραπεζικά δάνεια από συνδεδεμένες οντότητες - βραχυπρόθεσμα"
"l10n_gr_52_02_02","Bank loans from related entities - Long term","52.02.02","liability_non_current","False","","Τραπεζικά δάνεια από συνδεδεμένες οντότητες - μακροπρόθεσμα"
"l10n_gr_52_02_03","Bank loans from related entities - Short term part of long term loans","52.02.03","liability_current","False","","Τραπεζικά δάνεια από συνδεδεμένες οντότητες - βραχυπρόθεσμο μέρος μακροπρόθεσμων δανείων"
"l10n_gr_53_01_01","Loans received from non-related parties - Short term portion","53.01.01","liability_current","False","","Δάνεια από μη συνδεδεμένες οντότητες - βραχυπρόθεσμα"
"l10n_gr_53_01_02","Loans received from non-related parties - Long term portion","53.01.02","liability_non_current","False","","Δάνεια από μη συνδεδεμένες οντότητες - μακροπρόθεσμα"
"l10n_gr_53_02_01","Loans received from related parties - Short term portion","53.02.01","liability_current","False","","Δάνεια από συνδεδεμένες οντότητες - βραχυπρόθεσμα"
"l10n_gr_53_02_02","Loans received from related parties - Long term portion","53.02.02","liability_non_current","False","","Δάνεια από συνδεδεμένες οντότητες - μακροπρόθεσμα"
"l10n_gr_53_03_01","Employee compensation payable - Short term portion","53.03.01","liability_payable","True","","Αποδοχές προσωπικού πληρωτέες - βραχυπρόθεσμα"
"l10n_gr_53_03_02","Employee compensation payable - Long term portion","53.03.02","liability_payable","True","","Αποδοχές προσωπικού πληρωτέες - μακροπρόθεσμα"
"l10n_gr_53_04_01","Liabilities to owners and management - Short term portion","53.04.01","liability_current","False","","Υποχρεώσεις προς ιδιοκτήτες και διευθυντικό προσωπικό - βραχυπρόθεσμα"
"l10n_gr_53_04_02","Liabilities to owners and management - Long term portion","53.04.02","liability_non_current","False","","Υποχρεώσεις προς ιδιοκτήτες και διευθυντικό προσωπικό - μακροπρόθεσμα"
"l10n_gr_53_05_01","Dividends and amounts of a similar nature payable - Short term portion","53.05.01","liability_payable","True","","Μερίσματα, προμερίσματα και άλλα ποσά συναφούς φύσης πληρωτέα - βραχυπρόθεσμα"
"l10n_gr_53_05_02","Dividends and amounts of a similar nature payable - Long term portion","53.05.02","liability_payable","True","","Μερίσματα, προμερίσματα και άλλα ποσά συναφούς φύσης πληρωτέα - μακροπρόθεσμα"
"l10n_gr_53_06_01","Other liabilities to non-related entities - Short term portion","53.06.01","liability_current","False","","Άλλες υποχρεώσεις προς μη συνδεδεμένες οντότητες - βραχυπρόθεσμα"
"l10n_gr_53_06_02","Other liabilities to non-related entities - Long term portion","53.06.02","liability_non_current","False","","Άλλες υποχρεώσεις προς μη συνδεδεμένες οντότητες - μακροπρόθεσμα"
"l10n_gr_53_07_01","Other liabilities to related entities - Short term portion","53.07.01","liability_current","False","","Άλλες υποχρεώσεις προς συνδεδεμένες οντότητες - βραχυπρόθεσμα"
"l10n_gr_53_07_02","Other liabilities to related entities - Long term portion","53.07.02","liability_non_current","False","","Άλλες υποχρεώσεις προς συνδεδεμένες οντότητες - μακροπρόθεσμα"
"l10n_gr_54_01_01","Income tax due as per annual tax return","54.01.01","liability_current","False","","Φόρος εισοδήματος ετήσιας δήλωσης"
"l10n_gr_54_01_02","Income tax withheld (contra account)","54.01.02","liability_current","False","","Παρακρατούμενος φόρος εισοδήματος της οντότητας (αντίθετος)"
"l10n_gr_54_01_03","Advance payment of income tax (contra account)","54.01.03","liability_current","False","","Προκαταβολή φόρου εισοδήματος (αντίθετος)"
"l10n_gr_54_02_01","Value added tax on sales revenue","54.02.01","liability_current","False","","ΦΠΑ εκροών"
"l10n_gr_54_02_02","Value added tax on purchases","54.02.02","liability_current","False","","ΦΠΑ εισροών"
"l10n_gr_54_02_03","Value added tax paid","54.02.03","liability_current","False","","Καταβληθείς ΦΠΑ"
"l10n_gr_54_03_01","Income tax of salaries and pensions","54.03.01","liability_current","False","","Παρακρατούμενος φόρος από μισθωτή εργασία και συντάξεις"
"l10n_gr_54_03_02","Income tax of business income","54.03.02","liability_current","False","","Παρακρατούμενος φόρος από επιχειρηματική δραστηριότητα"
"l10n_gr_54_03_03","Income tax on dividends","54.03.03","liability_current","False","","Παρακρατούμενος φόρος διανεμομένων μερισμάτων"
"l10n_gr_54_03_04","Other income tax","54.03.04","liability_current","False","","Λοιποί παρακρατούμενοι φόροι εισοδήματος"
"l10n_gr_54_04","Fiscal stamp due","54.04.00","liability_current","False","","Τέλη χαρτοσήμου"
"l10n_gr_54_05","Other taxes, levies and contributions","54.05.00","liability_current","False","","Λοιποί φόροι, τέλη και εισφορές"
"l10n_gr_55_01","Primary social security","55.01.00","liability_current","False","","Υποχρεώσεις σε ασφαλιστικούς οργανισμούς κύριας ασφάλισης"
"l10n_gr_55_02","Supplementary social security","55.02.00","liability_current","False","","Υποχρεώσεις σε ασφαλιστικούς οργανισμούς επικουρικής ασφάλισης"
"l10n_gr_56_01_01","Accrued expenses - Non-related entities","56.01.01","liability_current","False","","Έξοδα χρήσεως δουλευμένα - μη συνδεδεμένες οντότητες"
"l10n_gr_56_01_02","Accrued expenses - Related entities","56.01.02","liability_current","False","","Έξοδα χρήσεως δουλευμένα - συνδεδεμένες οντότητες"
"l10n_gr_56_02_01","Unearned revenue - Non-related entities","56.02.01","liability_current","False","","Έσοδα επόμενων χρήσεων - μη συνδεδεμένες οντότητες"
"l10n_gr_56_02_02","Unearned revenue - Related entities","56.02.02","liability_current","False","","Έσοδα επόμενων χρήσεων - συνδεδεμένες οντότητες"
"l10n_gr_57_01","Provisions for employee benefits","57.01.00","liability_current","False","","Προβλέψεις για παροχές σε εργαζομένους"
"l10n_gr_57_02_01","Provisions for judicial expenses","57.02.01","liability_current","False","","Προβλέψεις για εκκρεμοδικίες"
"l10n_gr_57_02_02","Provisions for guarantees granted","57.02.02","liability_current","False","","Προβλέψεις για δοσμένες εγγυήσεις"
"l10n_gr_57_02_03","Provisions for environmental restoration","57.02.03","liability_current","False","","Προβλέψεις για αποκατάσταση περιβάλλοντος"
"l10n_gr_57_02_04","Provisions for tax audit surcharges","57.02.04","liability_current","False","","Προβλέψεις για διαφορές φορολογικού ελέγχου"
"l10n_gr_57_02_05","Other provisions","57.02.05","liability_current","False","","Άλλες προβλέψεις"
"l10n_gr_57_03_01","Provisions for related entities - Employee benefits","57.03.01","liability_current","False","","Προβλέψεις για συνδεδεμένες οντότητες - παροχές σε εργαζομένους"
"l10n_gr_57_03_02","Provisions for related entities - Others","57.03.02","liability_current","False","","Προβλέψεις για συνδεδεμένες οντότητες - άλλα"
"l10n_gr_58_00","Government grants","58.00.00","liability_current","False","","Κρατικές επιχορηγήσεις"
"l10n_gr_59_00","Deferred tax liability","59.00.00","liability_current","False","","Αναβαλλόμενοι φόροι παθητικού"
"l10n_gr_60_01_01","Gross employee remuneration - Cost of sales","60.01.01","expense","False","","Μικτές αποδοχές - Κόστος πωλήσεων"
"l10n_gr_60_01_02","Gross employee remuneration - Administrative expenses","60.01.02","expense","False","","Μικτές αποδοχές - Έξοδα διοίκησης"
"l10n_gr_60_01_03","Gross employee remuneration - Selling and distribution expenses","60.01.03","expense","False","","Μικτές αποδοχές - Έξοδα διάθεσης"
"l10n_gr_60_02_01","Employer's social security contributions - Cost of sales","60.02.01","expense","False","","Εργοδοτικές εισφορές - Κόστος πωλήσεων"
"l10n_gr_60_02_02","Employer's social security contributions - Administrative expenses","60.02.02","expense","False","","Εργοδοτικές εισφορές - Έξοδα διοίκησης"
"l10n_gr_60_02_03","Employer's social security contributions - Selling and distribution expenses","60.02.03","expense","False","","Εργοδοτικές εισφορές - Έξοδα διάθεσης"
"l10n_gr_60_03_01","Other compensation - Cost of sales","60.03.01","expense","False","","Λοιπές παροχές - Κόστος πωλήσεων"
"l10n_gr_60_03_02","Other compensation - Administrative expenses","60.03.02","expense","False","","Λοιπές παροχές - Έξοδα διοίκησης"
"l10n_gr_60_03_03","Other compensation - Selling and distribution expenses","60.03.03","expense","False","","Λοιπές παροχές - Έξοδα διάθεσης"
"l10n_gr_60_04_01","Provision for post-employment benefits - Cost of sales","60.04.01","expense","False","","Προβλέψεις για παροχές μετά την έξοδο από την υπηρεσία (καθαρό ποσό) - Κόστος πωλήσεων"
"l10n_gr_60_04_02","Provision for post-employment benefits - Administrative expenses","60.04.02","expense","False","","Προβλέψεις για παροχές μετά την έξοδο από την υπηρεσία (καθαρό ποσό) - Έξοδα διοίκησης"
"l10n_gr_60_04_03","Provision for post-employment benefits - Selling and distribution expenses","60.04.03","expense","False","","Προβλέψεις για παροχές μετά την έξοδο από την υπηρεσία (καθαρό ποσό) - Έξοδα διάθεσης"
"l10n_gr_60_05_01","Employee compensation - Related entities - Cost of sales","60.05.01","expense","False","","Παροχές σε εργαζόμενους σε συνδεδεμένες οντότητες - Κόστος πωλήσεων"
"l10n_gr_60_05_02","Employee compensation - Related entities - Administrative expenses","60.05.02","expense","False","","Παροχές σε εργαζόμενους σε συνδεδεμένες οντότητες - Έξοδα διοίκησης"
"l10n_gr_60_05_03","Employee compensation - Related entities - Selling and distribution expenses","60.05.03","expense","False","","Παροχές σε εργαζόμενους σε συνδεδεμένες οντότητες - Έξοδα διάθεσης"
"l10n_gr_61_01","Impairment - Tangible fixed assets (other than biological)","61.01.00","expense","False","","Απομείωση ενσώματων παγίων (πλην βιολογικών)"
"l10n_gr_61_02","Impairment - Biological assets","61.02.00","expense","False","","Απομείωση βιολογικών περιουσιακών στοιχείων"
"l10n_gr_61_03","Impairment - Intangible assets","61.03.00","expense","False","","Απομείωση άυλων παγίων"
"l10n_gr_61_04","Impairment - Inventory","61.04.00","expense","False","","Απομείωση αποθεμάτων"
"l10n_gr_61_05_01","Impairment of accounts receivable","61.05.01","expense","False","","Απομείωση πελατών"
"l10n_gr_61_05_02","Impairment of notes receivable","61.05.02","expense","False","","Απομείωση αξιογράφων εμπορικών απαιτήσεων"
"l10n_gr_61_05_03","Impairment of held to maturity financial instruments","61.05.03","expense","False","","Απομείωση διακρατούμενων μέχρι τη λήξη επενδύσεων"
"l10n_gr_61_05_04","Impairment of investments in subsidiaries","61.05.04","expense","False","","Απομείωση συμμετοχών σε θυγατρικές"
"l10n_gr_61_05_05","Impairment of investments in associates","61.05.05","expense","False","","Απομείωση συμμετοχών σε συγγενείς"
"l10n_gr_61_05_06","Impairment of investments in joint ventures","61.05.06","expense","False","","Απομείωση συμμετοχών σε κοινοπραξίες"
"l10n_gr_61_06","Impairment of other assets","61.06.00","expense","False","","Απομείωση λοιπών περιουσιακών στοιχείων"
"l10n_gr_61_07_01","Fair value losses - Tangible assets","61.07.01","expense","False","","Ζημιές εύλογης αξίας ενσώματων πάγιων στοιχείων"
"l10n_gr_61_07_02","Fair value losses - Biological assets","61.07.02","expense","False","","Ζημιές εύλογης αξίας βιολογικών περιουσιακών στοιχείων"
"l10n_gr_61_07_03","Fair value losses - Financial assets","61.07.03","expense","False","","Ζημιές εύλογης αξίας χρηματοοικονομικών στοιχείων"
"l10n_gr_62_01_01","Foreign exchange differences on settlement of trade receivables and payable","62.01.01","expense","False","","Χρεωστικές συν/τικές διαφορές διακανονισμού εμπορικών απαιτήσεων και υποχρεώσεων"
"l10n_gr_62_01_02","Foreign exchange differences on settlement of loans","62.01.02","expense","False","","Χρεωστικές συν/τικές διαφορές διακανονισμού δανείων"
"l10n_gr_62_01_03","Foreign exchange differences on settlement of other balance sheet items","62.01.03","expense","False","","Χρεωστικές συν/τικές διαφορές διακανονισμού λοιπών στοιχείων ισολογισμού"
"l10n_gr_62_02_01","Foreign exchange differences from measurement of trade receivables and payable","62.02.01","expense","False","","Χρεωστικές συν/τικές διαφορές επιμέτρησης εμπορικών απαιτήσεων και υποχρεώσεων"
"l10n_gr_62_02_02","Foreign exchange differences measurement of loans","62.02.02","expense","False","","Χρεωστικές συν/τικές διαφορές επιμέτρησης δανείων"
"l10n_gr_62_02_03","Foreign exchange differences measurement of other balance sheet items","62.02.03","expense","False","","Χρεωστικές συν/τικές διαφορές επιμέτρησης λοιπών στοιχείων ισολογισμού"
"l10n_gr_63_01","Losses from the disposal or retirement of tangible assets","63.01.00","expense","False","","Ζημιές από διάθεση-απόσυρση ενσώματων παγίων"
"l10n_gr_63_02","Losses from the disposal or retirement of intangible assets","63.02.00","expense","False","","Ζημιές από διάθεση-απόσυρση άυλων πάγιων στοιχείων"
"l10n_gr_63_03","Losses from the disposal or retirement of financial assets","63.03.00","expense","False","","Ζημιές από διάθεση χρηματοοικονομικών στοιχείων"
"l10n_gr_63_04","Losses from the disposal or retirement of assets to related entities","63.04.00","expense","False","","Ζημιές από διάθεση - απόσυρση περιουσιακών στοιχείων σε συνδεδεμένες οντότητες"
"l10n_gr_64_01_01_01","Fees for services to non-related entities - Cost of sales","64.01.01.01","expense","False","","Αμοιβές για υπηρεσίες - μη συνδεδεμένες οντότητες - Κόστος πωλήσεων"
"l10n_gr_64_01_01_02","Fees for services to non-related entities - Administrative expenses","64.01.01.02","expense","False","","Αμοιβές για υπηρεσίες - μη συνδεδεμένες οντότητες - Έξοδα διοίκησης"
"l10n_gr_64_01_01_03","Fees for services to non-related entities - Selling and distribution expenses","64.01.01.03","expense","False","","Αμοιβές για υπηρεσίες - μη συνδεδεμένες οντότητες - Έξοδα διάθεσης"
"l10n_gr_64_01_02_01","Fees for services to related entities - Cost of sales","64.01.02.01","expense","False","","Αμοιβές για υπηρεσίες - συνδεδεμένες οντότητες - Κόστος πωλήσεων"
"l10n_gr_64_01_02_02","Fees for services to related entities - Administrative expenses","64.01.02.02","expense","False","","Αμοιβές για υπηρεσίες - συνδεδεμένες οντότητες - Έξοδα διοίκησης"
"l10n_gr_64_01_02_03","Fees for services to related entities - Selling and distribution expenses","64.01.02.03","expense","False","","Αμοιβές για υπηρεσίες - συνδεδεμένες οντότητες - Έξοδα διάθεσης"
"l10n_gr_64_02_01","Energy - Cost of sales","64.02.01","expense","False","","Ενέργεια - Κόστος πωλήσεων"
"l10n_gr_64_02_02","Energy - Administrative expenses","64.02.02","expense","False","","Ενέργεια - Έξοδα διοίκησης"
"l10n_gr_64_02_03","Energy - Selling and distribution expenses","64.02.03","expense","False","","Ενέργεια - Έξοδα διάθεσης"
"l10n_gr_64_03_01","Water - Cost of sales","64.03.01","expense","False","","Ύδρευση - Κόστος πωλήσεων"
"l10n_gr_64_03_02","Water - Administrative expenses","64.03.02","expense","False","","Ύδρευση - Έξοδα διοίκησης"
"l10n_gr_64_03_03","Water - Selling and distribution expenses","64.03.03","expense","False","","Ύδρευση - Έξοδα διάθεσης"
"l10n_gr_64_04_01","Telecommunications - Cost of sales","64.04.01","expense","False","","Τηλεπικοινωνίες - Κόστος πωλήσεων"
"l10n_gr_64_04_02","Telecommunications - Administrative expenses","64.04.02","expense","False","","Τηλεπικοινωνίες - Έξοδα διοίκησης"
"l10n_gr_64_04_03","Telecommunications - Selling and distribution expenses","64.04.03","expense","False","","Τηλεπικοινωνίες - Έξοδα διάθεσης"
"l10n_gr_64_05_01_01","Rents to non-related entities - Cost of sales","64.05.01.01","expense","False","","Ενοίκια - μη συνδεδεμένες οντότητες - Κόστος πωλήσεων"
"l10n_gr_64_05_01_02","Rents to non-related entities - Administrative expenses","64.05.01.02","expense","False","","Ενοίκια - μη συνδεδεμένες οντότητες - Έξοδα διοίκησης"
"l10n_gr_64_05_01_03","Rents to non-related entities - Selling and distribution expenses","64.05.01.03","expense","False","","Ενοίκια - μη συνδεδεμένες οντότητες - Έξοδα διάθεσης"
"l10n_gr_64_05_02_01","Rents to related entities - Cost of sales","64.05.02.01","expense","False","","Ενοίκια - συνδεδεμένες οντότητες - Κόστος πωλήσεων"
"l10n_gr_64_05_02_02","Rents to related entities - Administrative expenses","64.05.02.02","expense","False","","Ενοίκια - συνδεδεμένες οντότητες - Έξοδα διοίκησης"
"l10n_gr_64_05_02_03","Rents to related entities - Selling and distribution expenses","64.05.02.03","expense","False","","Ενοίκια - συνδεδεμένες οντότητες - Έξοδα διάθεσης"
"l10n_gr_64_06_01","Insurance - Cost of sales","64.06.01","expense","False","","Ασφάλιστρα - Κόστος πωλήσεων"
"l10n_gr_64_06_02","Insurance - Administrative expenses","64.06.02","expense","False","","Ασφάλιστρα - Έξοδα διοίκησης"
"l10n_gr_64_06_03","Insurance - Selling and distribution expenses","64.06.03","expense","False","","Ασφάλιστρα - Έξοδα διάθεσης"
"l10n_gr_64_07_01","Transportation - Cost of sales","64.07.01","expense","False","","Μεταφορικά - Κόστος πωλήσεων"
"l10n_gr_64_07_02","Transportation - Administrative expenses","64.07.02","expense","False","","Μεταφορικά - Έξοδα διοίκησης"
"l10n_gr_64_07_03","Transportation - Selling and distribution expenses","64.07.03","expense","False","","Μεταφορικά - Έξοδα διάθεσης"
"l10n_gr_64_08_01","Consumables - Cost of sales","64.08.01","expense","False","","Αναλώσιμα - Κόστος πωλήσεων"
"l10n_gr_64_08_02","Consumables - Administrative expenses","64.08.02","expense","False","","Αναλώσιμα - Έξοδα διοίκησης"
"l10n_gr_64_08_03","Consumables - Selling and distribution expenses","64.08.03","expense","False","","Αναλώσιμα - Έξοδα διάθεσης"
"l10n_gr_64_09_03","Repairs and maintenance - Cost of sales","64.09.01","expense","False","","Επισκευές και συντηρήσεις - Κόστος πωλήσεων"
"l10n_gr_64_09_02","Repairs and maintenance - Administrative expenses","64.09.02","expense","False","","Επισκευές και συντηρήσεις - Έξοδα διοίκησης"
"l10n_gr_64_09_01","Repairs and maintenance - Selling and distribution expenses","64.09.03","expense","False","","Επισκευές και συντηρήσεις - Έξοδα διάθεσης"
"l10n_gr_64_10_01","Advertising - Cost of sales","64.10.01","expense","False","","Διαφήμιση και προβολή - Κόστος πωλήσεων"
"l10n_gr_64_10_02","Advertising - Administrative expenses","64.10.02","expense","False","","Διαφήμιση και προβολή - Έξοδα διοίκησης"
"l10n_gr_64_10_03","Advertising - Selling and distribution expenses","64.10.03","expense","False","","Διαφήμιση και προβολή - Έξοδα διάθεσης"
"l10n_gr_64_11_01","Taxes and levies (other than income tax) - Cost of sales","64.11.01","expense","False","","Φόροι και τέλη (πλην φόρου εισοδήματος) - Κόστος πωλήσεων"
"l10n_gr_64_11_02","Taxes and levies (other than income tax) - Administrative expenses","64.11.02","expense","False","","Φόροι και τέλη (πλην φόρου εισοδήματος) - Έξοδα διοίκησης"
"l10n_gr_64_11_03","Taxes and levies (other than income tax) - Selling and distribution expenses","64.11.03","expense","False","","Φόροι και τέλη (πλην φόρου εισοδήματος) - Έξοδα διάθεσης"
"l10n_gr_64_11_04","Taxes and levies (other than income tax) - Other expenses and losses","64.11.04","expense","False","","Φόροι και τέλη (πλην φόρου εισοδήματος) - Λοιπά έξοδα και ζημιές"
"l10n_gr_64_12_01","Other expenses - Cost of sales","64.12.01","expense","False","","Λοιπά έξοδα - Κόστος πωλήσεων"
"l10n_gr_64_12_02","Other expenses - Administrative expenses","64.12.02","expense","False","","Λοιπά έξοδα - Έξοδα διοίκησης"
"l10n_gr_64_12_03","Other expenses - Selling and distribution expenses","64.12.03","expense","False","","Λοιπά έξοδα - Έξοδα διάθεσης"
"l10n_gr_64_13_01","Various expenses - Related entities - Cost of sales","64.13.01","expense","False","","Διάφορα λειτουργικά έξοδα από συνδεδεμένες οντότητες - Κόστος πωλήσεων"
"l10n_gr_64_13_02","Various expenses - Related entities - Administrative expenses","64.13.02","expense","False","","Διάφορα λειτουργικά έξοδα από συνδεδεμένες οντότητες - Έξοδα διοίκησης"
"l10n_gr_64_13_03","Various expenses - Related entities - Selling and distribution expenses","64.13.03","expense","False","","Διάφορα λειτουργικά έξοδα από συνδεδεμένες οντότητες - Έξοδα διάθεσης"
"l10n_gr_64_14","cash difference expense","64.14","expense","False","","εισόδημα από διαφορά μετρητών"
"l10n_gr_65_01","Interest expense - Bank loans","65.01.00","expense","False","","Τόκοι τραπεζικών δανείων"
"l10n_gr_65_02","Interest expense - Loans from related parties","65.02.00","expense","False","","Τόκοι δανείων από συνδεδεμένες οντότητες"
"l10n_gr_65_03","Interest expense - Other loans","65.03.00","expense","False","","Τόκοι λοιπών δανείων"
"l10n_gr_65_04","Interest expense - Liabilities and provisions","65.04.00","expense","False","","Τόκοι λοιπών υποχρεώσεων και προβλέψεων"
"l10n_gr_65_05","Other financial expenses","65.05.00","expense","False","","Λοιπά χρηματοοικονομικά έξοδα"
"l10n_gr_66_01_01","Depreciation of depreciable land improvements - Cost of sales","66.01.01","expense_depreciation","False","","Αποσβέσεις διαμορφώσεων γης - Κόστος πωλήσεων"
"l10n_gr_66_01_02","Depreciation of depreciable land improvements - Administrative expenses","66.01.02","expense_depreciation","False","","Αποσβέσεις διαμορφώσεων γης - Έξοδα διοίκησης"
"l10n_gr_66_01_03","Depreciation of depreciable land improvements - Selling and distribution expenses","66.01.03","expense_depreciation","False","","Αποσβέσεις διαμορφώσεων γης - Έξοδα διάθεσης"
"l10n_gr_66_02_01","Depreciation of buildings and physical infrastructure - Cost of sales","66.02.01","expense_depreciation","False","","Αποσβέσεις κτηρίων - τεχνικών έργων - Κόστος πωλήσεων"
"l10n_gr_66_02_02","Depreciation of buildings and physical infrastructure - Administrative expenses","66.02.02","expense_depreciation","False","","Αποσβέσεις κτηρίων - τεχνικών έργων - Έξοδα διοίκησης"
"l10n_gr_66_02_03","Depreciation of buildings and physical infrastructure - Selling and distribution expenses","66.02.03","expense_depreciation","False","","Αποσβέσεις κτηρίων - τεχνικών έργων - Έξοδα διάθεσης"
"l10n_gr_66_03_01","Depreciation of machinery - Cost of sales","66.03.01","expense_depreciation","False","","Αποσβέσεις μηχανολογικού εξοπλισμού - Κόστος πωλήσεων"
"l10n_gr_66_03_02","Depreciation of machinery - Administrative expenses","66.03.02","expense_depreciation","False","","Αποσβέσεις μηχανολογικού εξοπλισμού - Έξοδα διοίκησης"
"l10n_gr_66_03_03","Depreciation of machinery - Selling and distribution expenses","66.03.03","expense_depreciation","False","","Αποσβέσεις μηχανολογικού εξοπλισμού - Έξοδα διάθεσης"
"l10n_gr_66_04_01","Depreciation of transportation equipment - Cost of sales","66.04.01","expense_depreciation","False","","Αποσβέσεις μεταφορικών μέσων - Κόστος πωλήσεων"
"l10n_gr_66_04_02","Depreciation of transportation equipment - Administrative expenses","66.04.02","expense_depreciation","False","","Αποσβέσεις μεταφορικών μέσων - Έξοδα διοίκησης"
"l10n_gr_66_04_03","Depreciation of transportation equipment - Selling and distribution expenses","66.04.03","expense_depreciation","False","","Αποσβέσεις μεταφορικών μέσων - Έξοδα διάθεσης"
"l10n_gr_66_05_01","Depreciation of other equipment - Cost of sales","66.05.01","expense_depreciation","False","","Αποσβέσεις λοιπού εξοπλισμού - Κόστος πωλήσεων"
"l10n_gr_66_05_02","Depreciation of other equipment - Administrative expenses","66.05.02","expense_depreciation","False","","Αποσβέσεις λοιπού εξοπλισμού - Έξοδα διοίκησης"
"l10n_gr_66_05_03","Depreciation of other equipment - Selling and distribution expenses","66.05.03","expense_depreciation","False","","Αποσβέσεις λοιπού εξοπλισμού - Έξοδα διάθεσης"
"l10n_gr_66_06_01","Depreciation of investment property - Cost of sales","66.06.01","expense_depreciation","False","","Αποσβέσεις επενδύσεων σε ακίνητα - Κόστος πωλήσεων"
"l10n_gr_66_06_02","Depreciation of investment property - Administrative expenses","66.06.02","expense_depreciation","False","","Αποσβέσεις επενδύσεων σε ακίνητα - Έξοδα διοίκησης"
"l10n_gr_66_06_03","Depreciation of investment property - Selling and distribution expenses","66.06.03","expense_depreciation","False","","Αποσβέσεις επενδύσεων σε ακίνητα - Έξοδα διάθεσης"
"l10n_gr_66_07_01","Depreciation of biological fixed assets - Cost of sales","66.07.01","expense_depreciation","False","","Αποσβέσεις πάγιων βιολογικών περιουσιακών στοιχείων - Κόστος πωλήσεων"
"l10n_gr_66_07_02","Depreciation of biological fixed assets - Administrative expenses","66.07.02","expense_depreciation","False","","Αποσβέσεις πάγιων βιολογικών περιουσιακών στοιχείων - Έξοδα διοίκησης"
"l10n_gr_66_07_03","Depreciation of biological fixed assets - Selling and distribution expenses","66.07.03","expense_depreciation","False","","Αποσβέσεις πάγιων βιολογικών περιουσιακών στοιχείων - Έξοδα διάθεσης"
"l10n_gr_66_08_01","Amortization - Cost of sales","66.08.01","expense","False","","Αποσβέσεις άυλων παγίων - Κόστος πωλήσεων"
"l10n_gr_66_08_02","Amortization - Administrative expenses","66.08.02","expense","False","","Αποσβέσεις άυλων παγίων - Έξοδα διοίκησης"
"l10n_gr_66_08_03","Amortization - Selling and distribution expenses","66.08.03","expense","False","","Αποσβέσεις άυλων παγίων - Έξοδα διάθεσης"
"l10n_gr_67_01","Losses from physical disasters","67.01.00","expense","False","","Ζημιές φυσικών καταστροφών"
"l10n_gr_67_02","Losses from other disasters","67.02.00","expense","False","","Ζημιές άλλων καταστροφών"
"l10n_gr_67_03","Other extraordinary expenses and losses","67.03.00","expense","False","","Άλλα ασυνήθη έξοδα και ζημίες"
"l10n_gr_67_04","Fines and penalties","67.04.00","expense","False","","Πρόστιμα, προσαυξήσεις και ποινές"
"l10n_gr_67_05","Extraordinary expenses and losses - Related entities","67.05.00","expense","False","","Ασυνήθη έξοδα και ζημιές από συνδεδεμένες οντότητες"
"l10n_gr_68_01","Provision-expense for judicial expenses","68.01.00","expense","False","","Προβλέψεις για εκκρεμοδικίες"
"l10n_gr_68_02","Provision-expense for guarantees granted","68.02.00","expense","False","","Προβλέψεις για δοσμένες εγγυήσεις"
"l10n_gr_68_03_01","Provision-expense for environmental restoration - Cost of sales","68.03.01","expense","False","","Προβλέψεις για αποκατάσταση περιβάλλοντος - Κόστος πωλήσεων"
"l10n_gr_68_03_02","Provision-expense for environmental restoration - Other expenses","68.03.02","expense","False","","Προβλέψεις για αποκατάσταση περιβάλλοντος - Λοιπά έξοδα και ζημιές"
"l10n_gr_68_04_01","Provision-expense for tax audit surcharges, other than income tax - Administrative expenses","68.04.01","expense","False","","Προβλέψεις για διαφορές φορολογικού ελέγχου πλην φόρου εισοδήματος - Έξοδα διοίκησης"
"l10n_gr_68_04_02","Provision-expense for tax audit surcharges, other than income tax - Other expenses","68.04.02","expense","False","","Προβλέψεις για διαφορές φορολογικού ελέγχου πλην φόρου εισοδήματος - Λοιπά έξοδα και ζημιές"
"l10n_gr_68_05_01","Other provisions - Cost of sales","68.05.01","expense","False","","Άλλες προβλέψεις - Κόστος πωλήσεων"
"l10n_gr_68_05_02","Other provisions - Other expenses","68.05.02","expense","False","","Άλλες προβλέψεις - Λοιπά έξοδα και ζημιές"
"l10n_gr_68_06_01","Provisions for related entities - Cost of sales","68.06.01","expense","False","","Προβλέψεις για συνδεδεμένες οντότητες - Κόστος πωλήσεων"
"l10n_gr_68_06_02","Provisions for related entities - Administrative expenses","68.06.02","expense","False","","Προβλέψεις για συνδεδεμένες οντότητες - Έξοδα διοίκησης"
"l10n_gr_68_06_03","Provisions for related entities - Other expenses","68.06.03","expense","False","","Προβλέψεις για συνδεδεμένες οντότητες - Λοιπά έξοδα και ζημιές"
"l10n_gr_69_01","Current period tax expense","69.01.00","expense","False","","Τρέχων φόρος (έξοδο) περιόδου"
"l10n_gr_69_02","Deferred tax expense","69.02.00","expense","False","","Αναβαλλόμενος φόρος (έξοδο) περιόδου"
"l10n_gr_69_03","Provisions for income tax audit surcharges","69.03.00","expense","False","","Προβλέψεις για διαφορές φορολογικού ελέγχου φόρου εισοδήματος"
"l10n_gr_70_01_01","Gross sales - Non-related entities","70.01.01","income","False","","Πωλήσεις εμπορευμάτων σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_01_02","Returns - Non-related entities","70.01.02","income","False","","Επιστροφές πωλήσεων εμπορευμάτων σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_01_03","Discounts and allowances - Non-related entities","70.01.03","income","False","","Εκπτώσεις πωλήσεων εμπορευμάτων σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_02_01","Gross sales - Related entities","70.02.01","income","False","","Πωλήσεις εμπορευμάτων σε συνδεδεμένες οντότητες"
"l10n_gr_70_02_02","Returns - Related entities","70.02.02","income","False","","Επιστροφές πωλήσεων εμπορευμάτων σε συνδεδεμένες οντότητες"
"l10n_gr_70_02_03","Discounts and allowances - Related entities","70.02.03","income","False","","Εκπτώσεις πωλήσεων εμπορευμάτων σε συνδεδεμένες οντότητες"
"l10n_gr_70_03_01","Sales of finished goods and work in progress - Non-related entities","70.03.01","income","False","","Πωλήσεις προϊόντων έτοιμων και ημιτελών σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_03_02","Returns - Non-related entities","70.03.02","income","False","","Επιστροφές πωλήσεων προϊόντων έτοιμων και ημιτελών σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_03_03","Discounts and allowances - Non-related entities","70.03.03","income","False","","Εκπτώσεις πωλήσεων προϊόντων έτοιμων και ημιτελών σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_04_01","Sales of finished goods and work in progress - Related entities","70.04.01","income","False","","Πωλήσεις προϊόντων έτοιμων και ημιτελών σε συνδεδεμένες οντότητες"
"l10n_gr_70_04_02","Returns - Related entities","70.04.02","income","False","","Επιστροφές πωλήσεων προϊόντων έτοιμων και ημιτελών σε συνδεδεμένες οντότητες"
"l10n_gr_70_04_03","Discounts and allowances - Related entities","70.04.03","income","False","","Εκπτώσεις πωλήσεων προϊόντων έτοιμων και ημιτελών σε συνδεδεμένες οντότητες"
"l10n_gr_70_05_01","Gross sales - Non-related entities","70.05.01","income","False","","Πωλήσεις λοιπών αποθεμάτων σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_05_02","Returns - Non-related entities","70.05.02","income","False","","Επιστροφές πωλήσεων λοιπών αποθεμάτων σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_05_03","Discounts and allowances - Non-related entities","70.05.03","income","False","","Εκπτώσεις πωλήσεων λοιπών αποθεμάτων σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_06_01","Gross sales - Related entities","70.06.01","income","False","","Πωλήσεις λοιπών αποθεμάτων σε συνδεδεμένες οντότητες"
"l10n_gr_70_06_02","Returns - Related entities","70.06.02","income","False","","Επιστροφές πωλήσεων λοιπών αποθεμάτων σε συνδεδεμένες οντότητες"
"l10n_gr_70_06_03","Discounts and allowances - Related entities","70.06.03","income","False","","Εκπτώσεις πωλήσεων λοιπών αποθεμάτων σε συνδεδεμένες οντότητες"
"l10n_gr_70_07_01","Gross sales - Non-related entities","70.07.01","income","False","","Πωλήσεις υπηρεσιών σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_07_02","Returns - Non-related entities","70.07.02","income","False","","Επιστροφές πωλήσεων υπηρεσιών σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_07_03","Discounts and allowances - Non-related entities","70.07.03","income","False","","Εκπτώσεις πωλήσεων υπηρεσιών σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_08_01","Gross sales - Related entities","70.08.01","income","False","","Πωλήσεις υπηρεσιών σε συνδεδεμένες οντότητες"
"l10n_gr_70_08_02","Returns - Related entities","70.08.02","income","False","","Επιστροφές πωλήσεων υπηρεσιών σε συνδεδεμένες οντότητες"
"l10n_gr_70_08_03","Discounts and allowances - Related entities","70.08.03","income","False","","Εκπτώσεις πωλήσεων υπηρεσιών σε συνδεδεμένες οντότητες"
"l10n_gr_71_01","Amortization of government grants for fixed assets","71.01.00","income","False","","Αποσβέσεις επιχορηγήσεων παγίων στοιχείων"
"l10n_gr_71_02","Government grants for interest expenses","71.02.00","income","False","","Επιχορηγήσεις τόκων"
"l10n_gr_71_03","Government grants for other expenses","71.03.00","income","False","","Επιχορηγήσεις λοιπών εξόδων"
"l10n_gr_71_04","Other revenue","71.04.00","income","False","","Άλλα λειτουργικά έσοδα"
"l10n_gr_71_05","Other revenue from related entities","71.05.00","income","False","","Άλλα λειτουργικά έσοδα από συνδεδεμένες οντότητες"
"l10n_gr_71_06","cash difference income","71.06","income_other","False","","εισόδημα από διαφορά μετρητών"
"l10n_gr_72_01","Interest revenue on sales","72.01.00","income","False","","Πιστωτικοί τόκοι πωλήσεων"
"l10n_gr_72_02","Interest revenue from loans and other receivables","72.02.00","income","False","","Πιστωτικοί τόκοι δανείων και απαιτήσεων"
"l10n_gr_72_03","Interest revenue from related entities","72.03.00","income","False","","Πιστωτικοί τόκοι από συνδεδεμένες οντότητες"
"l10n_gr_72_04","Interest revenue from other investments","72.04.00","income","False","","Πιστωτικοί τόκοι άλλων επενδύσεων"
"l10n_gr_73_01_01","Foreign exchange differences on settlement of trade receivables and payable","73.01.01","income_other","False","","Πιστωτικές συν/τικές διαφορές διακανονισμού εμπορικών απαιτήσεων και υποχρ/σεων"
"l10n_gr_73_01_02","Foreign exchange differences on settlement of loans","73.01.02","income_other","False","","Πιστωτικές συν/τικές διαφορές διακανονισμού δανείων"
"l10n_gr_73_01_03","Foreign exchange differences on settlement of other balance sheet items","73.01.03","income_other","False","","Πιστωτικές συν/τικές διαφορές διακανονισμού λοιπών στοιχείων ισολογισμού"
"l10n_gr_73_02_01","Foreign exchange differences from measurement of trade receivables and payable","73.02.01","income_other","False","","Πιστωτικές συν/τικές διαφορές επιμέτρησης εμπορικών απαιτήσεων και υποχρεώσεων"
"l10n_gr_73_02_02","Foreign exchange differences measurement of loans","73.02.02","income_other","False","","Πιστωτικές συν/τικές διαφορές επιμέτρησης δανείων"
"l10n_gr_73_02_03","Foreign exchange differences measurement of other balance sheet items","73.02.03","income_other","False","","Πιστωτικές συν/τικές διαφορές επιμέτρησης λοιπών στοιχείων ισολογισμού"
"l10n_gr_74_01","Dividends from associates","74.01.00","income","False","","Μερίσματα από συμμετοχές σε συγγενείς"
"l10n_gr_74_02","Dividends from subsidiaries","74.02.00","income","False","","Μερίσματα από συμμετοχές σε θυγατρικές"
"l10n_gr_74_03","Dividends from joint ventures","74.03.00","income","False","","Μερίσματα από συμμετοχές σε κοινοπραξίες"
"l10n_gr_74_04","Dividends from other participating interests","74.04.00","income","False","","Μερίσματα από λοιπούς συμμετοχικούς τίτλους"
"l10n_gr_75_01","Gains from the disposal of tangible fixed assets","75.01.00","income","False","","Κέρδη από διάθεση ενσώματων παγίων"
"l10n_gr_75_02","Gains from the disposal of intangible fixed assets","75.02.00","income","False","","Κέρδη από διάθεση άυλων πάγιων στοιχείων"
"l10n_gr_75_03","Gains from the disposal of financial fixed assets","75.03.00","income","False","","Κέρδη από διάθεση χρηματοοικονομικών στοιχείων"
"l10n_gr_75_04","Gains from the disposal of financial fixed assets to related entities","75.04.00","income","False","","Κέρδη από διάθεση μη κυκλοφορούντων περιουσιακών στοιχείων σε συνδεδεμένες οντότητες"
"l10n_gr_76_01","Gains from reversal of provisions for judicial expenses","76.01.00","income","False","","Κέρδη από αναστροφή προβλέψεων για εκκρεμοδικίες"
"l10n_gr_76_02","Gains from reversal of provisions for guarantees given","76.02.00","income","False","","Κέρδη από αναστροφή προβλέψεων για δοσμένες εγγυήσεις"
"l10n_gr_76_03_01","Gains from reversal of provisions for environmental restoration - Cost of sales","76.03.01","income","False","","Κέρδη από αναστροφή προβλέψεων για αποκατάσταση περιβάλλοντος - Κόστος πωλήσεων"
"l10n_gr_76_03_02","Gains from reversal of provisions for environmental restoration - Other gains","76.03.02","income","False","","Κέρδη από αναστροφή προβλέψεων για αποκατάσταση περιβάλλοντος - άλλα κέρδη"
"l10n_gr_76_04","Gains from reversal of provisions for tax audit surcharges, other than income tax","76.04.00","income","False","","Κέρδη από αναστροφή προβλέψεων για διαφορές φορολογικού ελέγχου πλην φόρου εισοδήματος"
"l10n_gr_76_05_01","Gains from reversal of provisions for other expenses and risks - Cost of sales","76.05.01","income","False","","Κέρδη από αναστροφή άλλων προβλέψεων - Κόστος πωλήσεων"
"l10n_gr_76_05_02","Gains from reversal of provisions for other expenses and risks - Administrative expenses","76.05.02","income","False","","Κέρδη από αναστροφή άλλων προβλέψεων - Έξοδα διοίκησης"
"l10n_gr_76_05_03","Gains from reversal of provisions for other expenses and risks - Other gain","76.05.03","income","False","","Κέρδη από αναστροφή άλλων προβλέψεων - άλλα κέρδη"
"l10n_gr_76_06","Gains from reversal of impairment - Tangible assets other than biological","76.06.00","income","False","","Κέρδη από αναστροφή απομείωσης ενσώματων παγίων (πλην βιολογικών)"
"l10n_gr_76_07","Gains from reversal of impairment - Biological assets","76.07.00","income","False","","Κέρδη από αναστροφή απομείωσης βιολογικών περιουσιακών στοιχείων"
"l10n_gr_76_08","Gains from reversal of impairment - Intangible assets","76.08.00","income","False","","Κέρδη από αναστροφή απομείωσης άυλων παγίων"
"l10n_gr_76_09","Gains from reversal of impairment - Inventory","76.09.00","income","False","","Κέρδη από αναστροφή απομείωσης αποθεμάτων"
"l10n_gr_76_10_01","Gains from reversal of impairment - Accounts receivable","76.10.01","income","False","","Κέρδη από αναστροφή απομείωσης πελατών"
"l10n_gr_76_10_02","Gains from reversal of impairment - Notes receivable","76.10.02","income","False","","Κέρδη από αναστροφή απομείωσης αξιογράφων εμπορικών απαιτήσεων"
"l10n_gr_76_10_03","Gains from reversal of impairment - Held to maturity investments","76.10.03","income","False","","Κέρδη από αναστροφή απομείωσης διακρατούμενων μέχρι τη λήξη επενδύσεων"
"l10n_gr_76_10_04","Gains from reversal of impairment - Investments in subsidiaries","76.10.04","income","False","","Κέρδη από αναστροφή απομείωσης συμμετοχών σε θυγατρικές"
"l10n_gr_76_10_05","Gains from reversal of impairment - Investments in associates","76.10.05","income","False","","Κέρδη από αναστροφή απομείωσης συμμετοχών σε συγγενείς"
"l10n_gr_76_10_06","Gains from reversal of impairment - Investments in joint ventures","76.10.06","income","False","","Κέρδη από αναστροφή απομείωσης συμμετοχών σε κοινοπραξίες"
"l10n_gr_76_11","Gains from reversal of impairment - Other assets","76.11.00","income","False","","Κέρδη από αναστροφή απομείωσης λοιπών περιουσιακών στοιχείων"
"l10n_gr_77_01","Gains from fair value measurement - Tangible assets","77.01.00","income","False","","Κέρδη εύλογης αξίας ενσώματων πάγιων στοιχείων"
"l10n_gr_77_02","Gains from fair value measurement - Biological assets","77.02.00","income","False","","Κέρδη εύλογης αξίας βιολογικών περιουσιακών στοιχείων"
"l10n_gr_77_03","Gains from fair value measurement - Financial assets","77.03.00","income","False","","Κέρδη εύλογης αξίας χρηματοοικονομικών στοιχείων"
"l10n_gr_78_01","Current period tax revenue","78.01.00","income","False","","Τρέχων φόρος περιόδου έσοδο"
"l10n_gr_78_02","Deferred tax revenue","78.02.00","income","False","","Αναβαλλόμενος φόρος περιόδου έσοδο"
"l10n_gr_78_03","Gains from the reversal of provisions for income tax audit surcharges","78.03.00","income","False","","Κέρδη από αναστροφή προβλέψεων για διαφορές φορολογικού ελέγχου φόρου εισοδήματος"
"l10n_gr_79_01","Extraordinary revenue and gains from non-related entities","79.01.00","income","False","","Ασυνήθη έσοδα και κέρδη από μη συνδεδεμένες οντότητες"
"l10n_gr_79_02","Extraordinary revenue and gains from related entities","79.02.00","income","False","","Ασυνήθη έσοδα και κέρδη από συνδεδεμένες οντότητες"
"l10n_gr_79_03","Gain from bargain purchase of a business","79.03.00","income","False","","Κέρδος από αγορά οντότητας σε τιμή ευκαιρίας"
"l10n_gr_80_01","Employee compensation","80.01.00","expense","False","","Παροχές σε εργαζόμενους σε ιδιοπαραγωγή"
"l10n_gr_80_02","Depreciation","80.02.00","expense_depreciation","False","","Αποσβέσεις σε ιδιοπαραγωγή"
"l10n_gr_80_03","Other operating expenses","80.03.00","expense","False","","Άλλα λειτουργικά έξοδα σε ιδιοπαραγωγή"
"l10n_gr_80_04","Financial expenses","80.04.00","expense","False","","Χρηματοοικονομικά έξοδα σε ιδιοπαραγωγή"
"l10n_gr_80_05","Provisions","80.05.00","expense","False","","Προβλέψεις σε ιδιοπαραγωγή"
"l10n_gr_81_01","Debit liaison accounts with branches","81.01.00","off_balance","False","","Χρεωστικοί δοσοληπτικοί λογαριασμοί υποκαταστημάτων / κεντρικού"
"l10n_gr_81_02","Credit liaison accounts with branches","81.02.00","off_balance","False","","Πιστωτικοί δοσοληπτικοί λογαριασμοί υποκαταστημάτων / κεντρικού"
"l10n_gr_82_01","Accumulation of all revenue, expenses, gains and losses","82.01.00","equity","False","","Συγκέντρωση αποτελεσματικών λογαριασμών"
"l10n_gr_82_02","Net income after tax for the period","82.02.00","equity","False","","Καθαρό κέρδος περιόδου (μετά από φόρους)"
"l10n_gr_82_03","Net loss after tax for the period","82.03.00","equity","False","","Καθαρή ζημία περιόδου (μετά από φόρους)"

```

## File: data\template\account.fiscal.position-gr.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@gr","zip_from","zip_to"
"fiscal_position_template_domestic","1","Domestic apart from the Aegean Islands","1","1","base.gr","","","","Εγχώρια εκτός από τα νησιά του Αιγαίου","",""
"fiscal_position_template_domestic_aegan","1","Domestic Aegean Islands","1","1","base.gr","","l10n_gr_tax_s24_G","l10n_gr_tax_s17_G","Νησιά του Αιγαίου Εσωτερικού","70","85"
"","","","","","","","l10n_gr_tax_s6_G","l10n_gr_tax_s4_G","","",""
"","","","","","","","l10n_gr_tax_s13_G","l10n_gr_tax_s9_G","","",""
"","","","","","","","l10n_gr_tax_s24_S","l10n_gr_tax_s17_S","","",""
"","","","","","","","l10n_gr_tax_s13_S","l10n_gr_tax_s9_S","","",""
"","","","","","","","l10n_gr_tax_s6_S","l10n_gr_tax_s4_S","","",""
"","","","","","","","l10n_gr_tax_p24_G","l10n_gr_tax_p17_G","","",""
"","","","","","","","l10n_gr_tax_p13_G","l10n_gr_tax_p9_G","","",""
"","","","","","","","l10n_gr_tax_p6_G","l10n_gr_tax_p4_G","","",""
"","","","","","","","l10n_gr_tax_p24_S","l10n_gr_tax_p17_S","","",""
"","","","","","","","l10n_gr_tax_p13_S","l10n_gr_tax_p9_S","","",""
"","","","","","","","l10n_gr_tax_p6_S","l10n_gr_tax_p4_S","","",""
"","","","","","","","l10n_gr_tax_p24_IG","l10n_gr_tax_p17_IG","","",""
"","","","","","","","l10n_gr_tax_p13_IG","l10n_gr_tax_p9_IG","","",""
"","","","","","","","l10n_gr_tax_p6_IG","l10n_gr_tax_p4_IG","","",""
"","","","","","","","l10n_gr_tax_p24_other_imports","l10n_gr_tax_p17_other_imports","","",""
"","","","","","","","l10n_gr_tax_p13_other_imports","l10n_gr_tax_p9_other_imports","","",""
"","","","","","","","l10n_gr_tax_p6_other_imports","l10n_gr_tax_p4_other_imports","","",""
"","","","","","","","l10n_gr_tax_p24_G_eu","l10n_gr_tax_p17_G_eu","","",""
"","","","","","","","l10n_gr_tax_p13_G_eu","l10n_gr_tax_p9_G_eu","","",""
"","","","","","","","l10n_gr_tax_p6_G_eu","l10n_gr_tax_p4_G_eu","","",""
"","","","","","","","l10n_gr_tax_p24_S_eu","l10n_gr_tax_p17_S_eu","","",""
"","","","","","","","l10n_gr_tax_p13_S_eu","l10n_gr_tax_p9_S_eu","","",""
"","","","","","","","l10n_gr_tax_p6_S_eu","l10n_gr_tax_p4_S_eu","","",""
"","","","","","","","l10n_gr_tax_p24_O_eu","l10n_gr_tax_p17_O_eu","","",""
"","","","","","","","l10n_gr_tax_p13_O_eu","l10n_gr_tax_p9_O_eu","","",""
"","","","","","","","l10n_gr_tax_p6_O_eu","l10n_gr_tax_p4_O_eu","","",""
fiscal_position_template_4,"2","Private Intra-Community Regime","1","","","base.europe","","","Ιδιωτικό ενδοκοινοτικό καθεστώς","",""
fiscal_position_template_EU,"3","Intra-Community Regime","1","1","","base.europe","l10n_gr_tax_s24_G","l10n_gr_tax_s0_G_eu","Ενδοκοινοτικό καθεστώς","",""
"","","","","","","","l10n_gr_tax_s17_G","l10n_gr_tax_s0_G_eu","","",""
"","","","","","","","l10n_gr_tax_s13_G","l10n_gr_tax_s0_G_eu","","",""
"","","","","","","","l10n_gr_tax_s6_G","l10n_gr_tax_s0_G_eu","","",""
"","","","","","","","l10n_gr_tax_s9_G","l10n_gr_tax_s0_G_eu","","",""
"","","","","","","","l10n_gr_tax_s4_G","l10n_gr_tax_s0_G_eu","","",""
"","","","","","","","l10n_gr_tax_s24_S","l10n_gr_tax_s0_S_eu","","",""
"","","","","","","","l10n_gr_tax_s17_S","l10n_gr_tax_s0_S_eu","","",""
"","","","","","","","l10n_gr_tax_s13_S","l10n_gr_tax_s0_S_eu","","",""
"","","","","","","","l10n_gr_tax_s6_S","l10n_gr_tax_s0_S_eu","","",""
"","","","","","","","l10n_gr_tax_s9_S","l10n_gr_tax_s0_S_eu","","",""
"","","","","","","","l10n_gr_tax_s4_S","l10n_gr_tax_s0_S_eu","","",""
"","","","","","","","l10n_gr_tax_p24_G","l10n_gr_tax_p24_G_eu","","",""
"","","","","","","","l10n_gr_tax_p13_G","l10n_gr_tax_p13_G_eu","","",""
"","","","","","","","l10n_gr_tax_p6_G","l10n_gr_tax_p6_G_eu","","",""
"","","","","","","","l10n_gr_tax_p17_G","l10n_gr_tax_p17_G_eu","","",""
"","","","","","","","l10n_gr_tax_p9_G","l10n_gr_tax_p9_G_eu","","",""
"","","","","","","","l10n_gr_tax_p4_G","l10n_gr_tax_p4_G_eu","","",""
"","","","","","","","l10n_gr_tax_p24_S","l10n_gr_tax_p24_S_eu","","",""
"","","","","","","","l10n_gr_tax_p13_S","l10n_gr_tax_p13_S_eu","","",""
"","","","","","","","l10n_gr_tax_p6_S","l10n_gr_tax_p6_S_eu","","",""
"","","","","","","","l10n_gr_tax_p17_S","l10n_gr_tax_p17_S_eu","","",""
"","","","","","","","l10n_gr_tax_p9_S","l10n_gr_tax_p9_S_eu","","",""
"","","","","","","","l10n_gr_tax_p4_S","l10n_gr_tax_p4_S_eu","","",""
"fiscal_position_template_import_export","4","Extra-Community Regime","1","","","","l10n_gr_tax_s24_G","l10n_gr_tax_s0_export","Εξωκοινοτικό καθεστώς","",""
"","","","","","","","l10n_gr_tax_s17_G","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_s13_G","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_s6_G","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_s9_G","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_s4_G","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_s24_S","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_s17_S","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_s13_S","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_s6_S","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_s9_S","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_s4_S","l10n_gr_tax_s0_export","","",""
"","","","","","","","l10n_gr_tax_p24_G","l10n_gr_tax_p24_other_imports","","",""
"","","","","","","","l10n_gr_tax_p13_G","l10n_gr_tax_p13_other_imports","","",""
"","","","","","","","l10n_gr_tax_p6_G","l10n_gr_tax_p6_other_imports","","",""
"","","","","","","","l10n_gr_tax_p17_G","l10n_gr_tax_p17_other_imports","","",""
"","","","","","","","l10n_gr_tax_p9_G","l10n_gr_tax_p9_other_imports","","",""
"","","","","","","","l10n_gr_tax_p4_G","l10n_gr_tax_p4_other_imports","","",""
"","","","","","","","l10n_gr_tax_p24_S","l10n_gr_tax_p24_other_imports","","",""
"","","","","","","","l10n_gr_tax_p13_S","l10n_gr_tax_p13_other_imports","","",""
"","","","","","","","l10n_gr_tax_p6_S","l10n_gr_tax_p6_other_imports","","",""
"","","","","","","","l10n_gr_tax_p17_S","l10n_gr_tax_p17_other_imports","","",""
"","","","","","","","l10n_gr_tax_p9_S","l10n_gr_tax_p9_other_imports","","",""
"","","","","","","","l10n_gr_tax_p4_S","l10n_gr_tax_p4_other_imports","","",""

```

## File: data\template\account.group-gr.csv

```csv
"id","code_prefix_start","name","name@gr"
"l10n_gr_1","1","FIXED ASSETS","ΕΝΣΩΜΑΤΑ ΚΑΙ ΑΥΛΑ ΜΗ ΚΥΚΛΟΦΟΡΟΥΝΤΑ (ΠΑΓΙΑ) ΠΕΡΙΟΥΣΙΑΚΑ ΣΤΟΙΧΕΙΑ"
"l10n_gr_10","10","Land","Γη"
"l10n_gr_11","11","Depreciable land improvements","Διαμορφώσεις γης υποκείμενες σε απόσβεση"
"l10n_gr_12","12","Buildings and physical infrastructure","Κτήρια - τεχνικά έργα"
"l10n_gr_13","13","Machinery","Μηχανολογικός εξοπλισμός"
"l10n_gr_14","14","Transportation equipment","Μεταφορικά μέσα"
"l10n_gr_15","15","Other equipment","Λοιπός εξοπλισμός"
"l10n_gr_16","16","Investment property","Επενδύσεις σε ακίνητα"
"l10n_gr_17","17","Biological fixed assets","Πάγια βιολογικά περιουσιακά στοιχεία"
"l10n_gr_17_01","17.01","Living animals","Ζώντα ζώα"
"l10n_gr_17_02","17.02","Trees and plants","Δένδρα και φυτά"
"l10n_gr_18","18","Intangibles","Άυλα"
"l10n_gr_18_01","18.01","Development expenditure","Δαπάνες ανάπτυξης"
"l10n_gr_18_01_01","18.01.01","Acquisition cost of development expenditure","Μικτή αξία κτήσης δαπανών ανάπτυξης"
"l10n_gr_18_01_02","18.01.02","Accumulated depreciation of development expenditure","Σωρευμένες αποσβέσεις δαπανών ανάπτυξης"
"l10n_gr_18_01_03","18.01.03","Accumulated impairment of development expenditure","Σωρευμένες απομειώσεις δαπανών ανάπτυξης"
"l10n_gr_18_02","18.02","Goodwill","Υπεραξία"
"l10n_gr_18_03","18.03","Other intangibles","Λοιπά άυλα"
"l10n_gr_2","2","INVENTORY","ΑΠΟΘΕΜΑΤΑ"
"l10n_gr_20","20","Merchandise","Εμπορεύματα"
"l10n_gr_21","21","Finished products","Προϊόντα"
"l10n_gr_22","22","Biological current assets","Βιολογικά περιουσιακά στοιχεία (κυκλοφορούντα)"
"l10n_gr_22_01","22.01","Living Animals","Ζώντα ζώα"
"l10n_gr_22_02","22.02","Trees and plants","Δένδρα και φυτά"
"l10n_gr_23","23","Work in progress","Παραγωγή σε εξέλιξη"
"l10n_gr_24","24","Materials","Πρώτες ύλες και υλικά"
"l10n_gr_25","25","Packing materials","Υλικά συσκευασίας"
"l10n_gr_26","26","Spare parts","Ανταλλακτικά παγίων"
"l10n_gr_27","27","Other inventory","Λοιπά αποθέματα"
"l10n_gr_3","3","FINANCIAL AND OTHER ASSETS","ΧΡΗΜΑΤΟΟΙΚΟΝΟΜΙΚΑ ΚΑΙ ΛΟΙΠΑ ΠΕΡΙΟΥΣΙΑΚΑ ΣΤΟΙΧΕΙΑ"
"l10n_gr_30","30","Accounts receivable","Πελάτες"
"l10n_gr_30_01","30.01","Accounts receivables from non-related entities","Πελάτες - μη συνδεδεμένες οντότητες"
"l10n_gr_30_01_01","30.01.01","Accounts receivables from non-related entities – nominal amount","Πελάτες μη συνδεδεμένες οντότητες - ονομαστικό ποσό"
"l10n_gr_30_01_02","30.01.02","Interest not accrued on receivables from non-related entities","Μη δουλευμένοι τόκοι μη συνδεδεμένων πελατών"
"l10n_gr_30_01_04","30.01.04","Impairment of receivables from non-related entities","Απομείωση μη συνδεδεμένων πελατών"
"l10n_gr_30_02_01","30.02.01","Accounts receivables from related entities – nominal amount","Συνδεδεμένοι πελάτες - ονομαστικό ποσό"
"l10n_gr_30_02_02","30.02.02","Interest not-accrued on receivables from related entities","Μη δουλευμένοι τόκοι συνδεδεμένων πελατών"
"l10n_gr_30_02_04","30.02.04","Impairment of receivables from related entities","Απομείωση συνδεδεμένων πελατών"
"l10n_gr_31_01_01","31.01.01","Notes receivables from non-related entities – nominal amount","Αξιόγραφα εμπορικών απαιτήσεων μη συνδεδεμένων οντοτήτων - ονομαστικό ποσό"
"l10n_gr_31_01_02","31.01.02","Interest not-accrued on receivables from non-related entities","Μη δουλευμένοι τόκοι αξιογράφων εμπορικών απαιτήσεων μη συνδεδεμένων οντοτήτων"
"l10n_gr_31_01_03","31.01.03","Impairment of receivables from non-related entities","Απομείωση αξιογράφων εμπορικών απαιτήσεων μη συνδεδεμένων οντοτήτων"
"l10n_gr_30_02","30.02","Accounts receivables from related entities","Πελάτες - συνδεδεμένες οντότητες"
"l10n_gr_31_02_01","31.02.01","Notes receivables from related entities – nominal amount","Αξιόγραφα εμπορικών απαιτήσεων συνδεδεμένων οντοτήτων - ονομαστικό ποσό"
"l10n_gr_31_02_02","31.02.02","Interest not-accrued on receivables from related entities","Μη δουλευμένοι τόκοι αξιογράφων εμπορικών απαιτήσεων συνδεδεμένων οντοτήτων"
"l10n_gr_31_02_03","31.02.03","Impairment of receivables from related entities","Απομείωση αξιογράφων εμπορικών απαιτήσεων συνδεδεμένων οντοτήτων"
"l10n_gr_32_01","32.01","Loans given to related parties","Δάνεια χορηγηθέντα σε συνδεδεμένες οντότητες"
"l10n_gr_32_02","32.02","Loans given to personnel and management","Δάνεια χορηγηθέντα στο προσωπικό και στη διοίκηση"
"l10n_gr_32_03","32.03","Other loans given","Λοιπά χορηγηθέντα δάνεια"
"l10n_gr_32_04","32.04","Impairment of loans given","Απομείωση χορηγηθέντων δανείων"
"l10n_gr_33_01_01","33.01.01","Revenue from equity investments receivable – nominal amount","Έσοδα από πάσης φύσεως συμμετοχές εισπρακτέα - ονομαστικό ποσό"
"l10n_gr_33_01_02","33.01.02","Impairment of revenue from equity investments receivable","Απομείωση - έσοδα από πάσης φύσεως συμμετοχές εισπρακτέα"
"l10n_gr_33_02_01","33.02.01","Other receivables from related entities– nominal amount","Άλλες απαιτήσεις από συνδεδεμένες οντότητες - ονομαστικό ποσό"
"l10n_gr_33_02_02","33.02.02","Impairment of other receivables from related entities","Απομείωση - άλλες απαιτήσεις από συνδεδεμένες οντότητες"
"l10n_gr_33_03_01","33.03.01","Other receivables from non-related entities, nominal amount","Άλλες απαιτήσεις από μη συνδεδεμένες οντότητες - ονομαστικό ποσό"
"l10n_gr_33_03_02","33.03.02","Impairment of other receivables from non-related entities","Απομείωση - άλλες απαιτήσεις από μη συνδεδεμένες οντότητες"
"l10n_gr_33_04","33.04.01","Guarantees","Εγγυήσεις"
"l10n_gr_31","31","Notes receivables","Αξιόγραφα εμπορικών απαιτήσεων"
"l10n_gr_31_01","31.01","Notes receivables from non-related entities","Αξιόγραφα εμπορικών απαιτήσεων - μη συνδεδεμένες οντότητες"
"l10n_gr_31_02","31.02","Notes receivables from related entities","Αξιόγραφα εμπορικών απαιτήσεων - συνδεδεμένες οντότητες"
"l10n_gr_32","32","Loans given","Χορηγηθέντα δάνεια"
"l10n_gr_33","33","Other  receivables","Λοιπές απαιτήσεις"
"l10n_gr_33_01","33.01","Revenue from equity investments receivable","Έσοδα από πάσης φύσεως συμμετοχές εισπρακτέα"
"l10n_gr_33_02","33.02","Other receivables from related entities","Άλλες απαιτήσεις από συνδεδεμένες οντότητες"
"l10n_gr_33_03","33.03","Other receivables from non-related entities","Άλλες απαιτήσεις από μη συνδεδεμένες οντότητες"
"l10n_gr_34","34","Financial instruments","Χρηματοοικονομικά περιουσιακά στοιχεία"
"l10n_gr_34_03","34.03","Other financial assets","Λοιπά χρηματοοικονομικά στοιχεία"
"l10n_gr_35","35","Financial instruments held for hedging","Χρηματοοικονομικά στοιχεία για αντιστάθμιση"
"l10n_gr_36","36","Investments (equity)","Συμμετοχές"
"l10n_gr_36_01","36.01","Investment in subsidiaries","Συμμετοχές σε θυγατρικές"
"l10n_gr_36_02","36.02","Investment in associates","Συμμετοχές σε συγγενείς"
"l10n_gr_36_03","36.03","Investment in joint ventures","Συμμετοχές σε κοινοπραξίες"
"l10n_gr_37","37","Prepaid expenses and accrued revenue","Προπληρωμένα έξοδα και δουλευμένα έσοδα περιόδου"
"l10n_gr_37_01","37.01","Prepaid expenses","Προπληρωμένα έξοδα"
"l10n_gr_37_02","37.02","Accrued revenue","Δουλευμένα έσοδα περιόδου"
"l10n_gr_38","38","Cash and cash equivalents","Ταμειακά διαθέσιμα και ισοδύναμα"
"l10n_gr_4","4","EQUITY","ΚΑΘΑΡΗ ΘΕΣΗ"
"l10n_gr_43","43","Treasury titles","Ίδιοι τίτλοι"
"l10n_gr_44","44","Fair value reserves","Διαφορές εύλογης αξίας"
"l10n_gr_48","48","Articles of association and other reserves","Αποθεματικά καταστατικού και λοιπά αποθεματικά"
"l10n_gr_5","5","LIABILITIES","ΥΠΟΧΡΕΩΣΕΙΣ"
"l10n_gr_50","50","Accounts payable","Προμηθευτές"
"l10n_gr_50_01","50.01","Accounts payable – non-related entities","Προμηθευτές - μη συνδεδεμένες οντότητες"
"l10n_gr_50_02","50.02","Accounts payable – related entities","Προμηθευτές — συνδεδεμένες οντότητες"
"l10n_gr_50_03","50.03","Advances to suppliers – non-related entities","Προκαταβολές σε προμηθευτές - μη συνδεδεμένες οντότητες"
"l10n_gr_50_04","50.04","Advances to suppliers – related entities","Προκαταβολές σε προμηθευτές - συνδεδεμένες οντότητες"
"l10n_gr_51","51","Notes payable","Αξιόγραφα εμπορικών υποχρεώσεων"
"l10n_gr_51_01","51.01","Notes payable to non-related entities","Αξιόγραφα εμπορικών υποχρεώσεων — μη συνδεδεμένες οντότητες"
"l10n_gr_51_02","51.02","Notes payable to related entities","Αξιόγραφα εμπορικών υποχρεώσεων - συνδεδεμένες οντότητες"
"l10n_gr_52","52","Bank loans","Τραπεζικά δάνεια"
"l10n_gr_52_01","52.01","Bank loans from non-related entities","Τραπεζικά δάνεια - μη συνδεδεμένες οντότητες"
"l10n_gr_52_02","52.02","Bank loans from related entities","Τραπεζικά δάνεια - συνδεδεμένες οντότητες"
"l10n_gr_53","53","Other liabilities","Λοιπές υποχρεώσεις"
"l10n_gr_53_01","53.01","Loans received from non-related parties","Δάνεια από μη συνδεδεμένες οντότητες"
"l10n_gr_53_02","53.02","Loans received from related parties","Δάνεια από συνδεδεμένες οντότητες"
"l10n_gr_53_03","53.03","Employee compensation payable","Αποδοχές προσωπικού πληρωτέες"
"l10n_gr_53_04","53.04","Liabilities to owners and management","Υποχρεώσεις προς ιδιοκτήτες και διευθυντικό προσωπικό"
"l10n_gr_53_05","53.05","Dividends and amounts of a similar nature payable","Μερίσματα, προμερίσματα και άλλα ποσά συναφούς φύσης πληρωτέα"
"l10n_gr_53_06","53.06","Other liabilities to non-related entities","Άλλες υποχρεώσεις προς μη συνδεδεμένες οντότητες"
"l10n_gr_53_07","53.07","Other liabilities to related entities","Άλλες υποχρεώσεις προς συνδεδεμένες οντότητες"
"l10n_gr_54","54","Taxes and levies payable","Υποχρεώσεις από φόρους και τέλη"
"l10n_gr_54_01","54.01","Income tax payable","Φόρος εισοδήματος πληρωτέος"
"l10n_gr_54_02","54.02","Value Added Tax","Φόρος προστιθέμενης αξίας (ΦΠΑ)"
"l10n_gr_54_03","54.03","Income tax of third parties withheld","Παρακρατούμενοι φόροι εισοδήματος τρίτων"
"l10n_gr_55","55","Social security contributions","Υποχρεώσεις σε ασφαλιστικούς οργανισμούς"
"l10n_gr_56","56","Accruals and advances received","Δουλευμένα έξοδα και έσοδα επομένων χρήσεων"
"l10n_gr_56_01","56.01","Accrued expenses","Έξοδα χρήσεως δουλευμένα"
"l10n_gr_56_02","56.02","Unearned revenue","Έσοδα επόμενων χρήσεων"
"l10n_gr_57","57","Provisions","Προβλέψεις"
"l10n_gr_57_02","57.02","Other provisions","Λοιπές προβλέψεις"
"l10n_gr_57_03","57.03","Provisions for related entities","Προβλέψεις για συνδεδεμένες οντότητες"
"l10n_gr_6","6","EXPENSES AND LOSSES","ΕΞΟΔΑ ΚΑΙ ΖΗΜΙΕΣ"
"l10n_gr_60","60","Employee compensation","Παροχές σε εργαζόμενους"
"l10n_gr_60_01","60.01","Gross employee remuneration","Μικτές αποδοχές"
"l10n_gr_60_02","60.02","Employer's social security contributions","Εργοδοτικές εισφορές"
"l10n_gr_60_03","60.03","Other compensation","Λοιπές παροχές"
"l10n_gr_60_04","60.04","Provision for post-employment benefits","Προβλέψεις για παροχές μετά την έξοδο από την υπηρεσία (καθαρό ποσό)"
"l10n_gr_60_05","60.05","Employee compensation – related entities","Παροχές σε εργαζόμενους σε συνδεδεμένες οντότητες"
"l10n_gr_61","61","Losses from measurement of assets","Ζημιές επιμέτρησης περιουσιακών στοιχείων"
"l10n_gr_61_05","61.05","Impairment – financial assets","Απομείωση χρηματοοικονομικών στοιχείων"
"l10n_gr_61_07","61.07","Losses from fair value measurement","Ζημίες από επιμέτρηση στην εύλογη αξία"
"l10n_gr_62","62","Foreign exchange differences – expense","Χρεωστικές συναλλαγματικές διαφορές"
"l10n_gr_62_01","62.01","Foreign exchange differences on settlement of accounts","Χρεωστικές συναλλαγματικές διαφορές από διακανονισμό"
"l10n_gr_62_02","62.02","Foreign exchange differences from measurement","Χρεωστικές συναλλαγματικές διαφορές επιμέτρησης"
"l10n_gr_63","63","Losses on disposal or retirement of non-current assets","Ζημιές από διάθεση-απόσυρση μη κυκλοφορούντων περιουσιακών στοιχείων"
"l10n_gr_64","64","Other operating expenses","Διάφορα λειτουργικά έξοδα"
"l10n_gr_64_01","64.01","Fees for services","Αμοιβές για υπηρεσίες"
"l10n_gr_64_01_01","64.01.01","Fees for services to non-related entities","Αμοιβές για υπηρεσίες - μη συνδεδεμένες οντότητες"
"l10n_gr_64_01_02","64.01.02","Fees for services to related entities","Αμοιβές για υπηρεσίες - συνδεδεμένες οντότητες"
"l10n_gr_64_02","64.02","Energy","Ενέργεια"
"l10n_gr_64_03","64.03.00","Water","Ύδρευση"
"l10n_gr_64_04","64.04.00","Telecommunications","Τηλεπικοινωνίες"
"l10n_gr_64_05","64.05","Rents","Ενοίκια"
"l10n_gr_64_05_01","64.05.01","Rents to non-related entities","Ενοίκια - μη συνδεδεμένες οντότητες"
"l10n_gr_64_05_02","64.05.02","Rents to related entities","Ενοίκια - συνδεδεμένες οντότητες"
"l10n_gr_64_06","64.06.00","Insurance","Ασφάλιστρα"
"l10n_gr_64_07","64.07.00","Transportation","Μεταφορικά"
"l10n_gr_64_08","64.08.00","Consumables","Αναλώσιμα"
"l10n_gr_64_09","64.09.00","Repairs and maintenance","Επισκευές και συντηρήσεις"
"l10n_gr_64_10","64.10.00","Advertising","Διαφήμιση και προβολή"
"l10n_gr_64_11","64.11.00","Taxes and levies (other than income tax)","Φόροι και τέλη (πλην φόρου εισοδήματος)"
"l10n_gr_64_12","64.12.00","Other expenses","Λοιπά έξοδα"
"l10n_gr_64_13","64.13.00","Various expenses – related entities","Διάφορα λειτουργικά έξοδα από συνδεδεμένες οντότητες"
"l10n_gr_65","65","Financial expenses","Χρεωστικοί τόκοι και συναφή έξοδα"
"l10n_gr_66","66","Depreciation","Αποσβέσεις"
"l10n_gr_66_01","66.01.00","Depreciation of depreciable land improvements","Αποσβέσεις διαμορφώσεων γης"
"l10n_gr_66_02","66.02.00","Depreciation of buildings and physical infrastructure","Αποσβέσεις κτηρίων - τεχνικών έργων"
"l10n_gr_66_03","66.03.00","Depreciation of machinery","Αποσβέσεις μηχανολογικού εξοπλισμού"
"l10n_gr_66_04","66.04.00","Depreciation of transportation equipment","Αποσβέσεις μεταφορικών μέσων"
"l10n_gr_66_05","66.05.00","Depreciation of other equipment","Αποσβέσεις λοιπού εξοπλισμού"
"l10n_gr_66_06","66.06.00","Depreciation of investment property","Αποσβέσεις επενδύσεων σε ακίνητα"
"l10n_gr_66_07","66.07.00","Depreciation of biological fixed assets","Αποσβέσεις πάγιων βιολογικών περιουσιακών στοιχείων"
"l10n_gr_66_08","66.08.00","Amortization","Αποσβέσεις άυλων παγίων"
"l10n_gr_67","67","Extraordinary expenses, losses and fines","Ασυνήθη έξοδα, ζημιές και πρόστιμα"
"l10n_gr_68","68","Provisions","Προβλέψεις (εκτός από προβλέψεις για το προσωπικό)"
"l10n_gr_68_03","68.03","Provision-expense for environmental restoration","Προβλέψεις για αποκατάσταση περιβάλλοντος"
"l10n_gr_68_04","68.04","Provision-expense for tax audit surcharges, other than income tax","Προβλέψεις για διαφορές φορολογικού ελέγχου πλην φόρου εισοδήματος"
"l10n_gr_68_05","68.05","Other provisions","Άλλες προβλέψεις"
"l10n_gr_68_06","68.06","Provisions for related entities","Προβλέψεις για συνδεδεμένες οντότητες"
"l10n_gr_69","69","Income tax expense","Φόρος εισοδήματος"
"l10n_gr_7","7","REVENUES AND GAINS","ΕΣΟΔΑ ΚΑΙ ΚΕΡΔΗ"
"l10n_gr_70","70","Sale of goods and services","Πωλήσεις αγαθών και υπηρεσιών"
"l10n_gr_70_01","70.01","Sales of merchandise (net) – non-related entities","Πωλήσεις εμπορευμάτων (καθαρές) σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_02","70.02","Sales of merchandise (net) – related entities","Πωλήσεις εμπορευμάτων (καθαρές) σε συνδεδεμένες οντότητες"
"l10n_gr_70_03","70.03","Sales of finished goods and work in progress (net) – non-related entities","Πωλήσεις προϊόντων έτοιμων και ημιτελών (καθαρές) σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_04","70.04","Sales of finished goods and work in progress (net) –related entities","Πωλήσεις προϊόντων έτοιμων και ημιτελών (καθαρές) σε συνδεδεμένες οντότητες"
"l10n_gr_70_05","70.05","Sales of other inventory (net) – non-related entities","Πωλήσεις λοιπών αποθεμάτων (καθαρές) σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_06","70.06","Sales of other inventory (net) – related entities","Πωλήσεις λοιπών αποθεμάτων (καθαρές) σε συνδεδεμένες οντότητες"
"l10n_gr_70_07","70.07","Sales of services (net) – non-related entities","Πωλήσεις υπηρεσιών (καθαρές) σε μη συνδεδεμένες οντότητες"
"l10n_gr_70_08","70.08","Sales of services (net) – related entities","Πωλήσεις υπηρεσιών (καθαρές) σε συνδεδεμένες οντότητες"
"l10n_gr_71","71","Other ordinary revenue","Λοιπά συνήθη έσοδα"
"l10n_gr_72","72","Interest revenue","Πιστωτικοί τόκοι και συναφή έσοδα"
"l10n_gr_73","73","Foreign exchange differences – gains","Πιστωτικές συναλλαγματικές διαφορές"
"l10n_gr_73_01","73.01","Foreign exchange differences on settlement of accounts","Πιστωτικές συναλλαγματικές διαφορές από διακανονισμό"
"l10n_gr_73_02","73.02","Foreign exchange differences from measurement","Πιστωτικές συναλλαγματικές διαφορές επιμέτρησης"
"l10n_gr_74","74","Dividends and similar revenue","Έσοδα συμμετοχών"
"l10n_gr_75","75","Gains from the disposal of non-current assets","Κέρδη από διάθεση μη κυκλοφορούντων περιουσιακών στοιχείων"
"l10n_gr_76","76","Gains from reversal of provisions and impairment","Κέρδη από αναστροφή προβλέψεων και απομειώσεων"
"l10n_gr_76_03","76.03","Gains from reversal of provisions for environmental restoration","Κέρδη από αναστροφή προβλέψεων για αποκατάσταση περιβάλλοντος"
"l10n_gr_76_05","76.05","Gains from reversal of provisions for other expenses and risks","Κέρδη από αναστροφή άλλων προβλέψεων"
"l10n_gr_76_10","76.1","Gains from reversal of impairment – financial assets","Κέρδη από αναστροφή απομείωσης χρηματοοικονομικών στοιχείων"
"l10n_gr_77","77","Gains from fair value measurement","Κέρδη από επιμέτρηση στην εύλογη αξία"
"l10n_gr_78","78","Income tax revenue","Φόρος εισοδήματος έσοδο"
"l10n_gr_79","79","Extraordinary revenue and gains","Ασυνήθη έσοδα και κέρδη"
"l10n_gr_8","8","SELF-CONSTRUCTED ASSETS, BRANCHΕS, AND NET INCOME (LOSS)","ΙΔΙΟΠΑΡΑΓΩΓΗ, ΥΠΟΚΑΤΑΣΤΗΜΑΤΑ ΚΑΙ ΑΠΟΤΕΛΕΣΜΑΤΑ ΠΕΡΙΟΔΟΥ"
"l10n_gr_80","80","Expenses charged to self-constructed assets","Έξοδα σε ιδιοπαραγωγή"
"l10n_gr_81","81","Liaison accounts with branches","Δοσοληπτικοί λογαριασμοί υποκαταστημάτων αυτοτελούς παρακολούθησης"
"l10n_gr_82","82","Net income (loss) for the period","Αποτέλεσμα (κέρδη ή ζημίες) περιόδου"

```

## File: data\template\account.tax-gr.csv

```csv
"id","name","description","tax_scope","active","invoice_label","amount","sequence","amount_type","type_tax_use","tax_group_id","active","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/factor_percent","repartition_line_ids/tag_ids","repartition_line_ids/account_id","name@gr","description@gr"
"l10n_gr_tax_s24_G","24% G","Sales VAT 24% goods for Greece (apart from the Aegean Islands)","","","24%","24.0","1","percent","sale","l10n_gr_tax_group_24","","base","invoice","","+303","","","ΦΠΑ πωλήσεων 24% για την Ελλάδα (εκτός των νησιών του Αιγαίου)"
"","","","","","","","","","","","","tax","invoice","","+333","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-303","","",""
"","","","","","","","","","","","","tax","refund","","-333","l10n_gr_54_02_01","",""
"l10n_gr_tax_s13_G","13% G","Sales VAT 13% goods for Greece (apart from the Aegean Islands)","","","13%","13.0","1","percent","sale","l10n_gr_tax_group_13","False","base","invoice","","+301","","","ΦΠΑ πωλήσεων 13% για την Ελλάδα (εκτός από τα νησιά του Αιγαίου)"
"","","","","","","","","","","","","tax","invoice","","+331","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-301","","",""
"","","","","","","","","","","","","tax","refund","","-331","l10n_gr_54_02_01","",""
"l10n_gr_tax_s6_G","6% G","Sales VAT 6% goods for Greece (apart from the Aegean Islands)","","","6%","6.0","1","percent","sale","l10n_gr_tax_group_6","False","base","invoice","","+302","","","ΦΠΑ πωλήσεων 6% για την Ελλάδα (εκτός των νησιών του Αιγαίου)"
"","","","","","","","","","","","","tax","invoice","","+332","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-302","","",""
"","","","","","","","","","","","","tax","refund","","-332","l10n_gr_54_02_01","",""
"l10n_gr_tax_s9_G","9% G","Sales VAT 9% goods for the Aegean Islands","","","9%","9.0","1","percent","sale","l10n_gr_tax_group_9","False","base","invoice","","+304","","","ΦΠΑ πωλήσεων 9% για τα νησιά του Αιγαίου"
"","","","","","","","","","","","","tax","invoice","","+334","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-304","","",""
"","","","","","","","","","","","","tax","refund","","-334","l10n_gr_54_02_01","",""
"l10n_gr_tax_s4_G","4% G","Sales VAT 4% goods for the Aegean Islands","","","4%","4.0","1","percent","sale","l10n_gr_tax_group_4","False","base","invoice","","+305","","","ΦΠΑ πωλήσεων 4% για τα νησιά του Αιγαίου"
"","","","","","","","","","","","","tax","invoice","","+335","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-305","","",""
"","","","","","","","","","","","","tax","refund","","-335","l10n_gr_54_02_01","",""
"l10n_gr_tax_s17_G","17% G","Sales VAT 17% goods for the Aegean Islands","","","17%","17.0","1","percent","sale","l10n_gr_tax_group_17","False","base","invoice","","+306","","","ΦΠΑ πωλήσεων 17% για τα νησιά του Αιγαίου"
"","","","","","","","","","","","","tax","invoice","","+336","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-306","","",""
"","","","","","","","","","","","","tax","refund","","-336","l10n_gr_54_02_01","",""
"l10n_gr_tax_s24_S","24% S","Sales VAT 24% services for Greece (apart from the Aegean Islands)","","","24%","24.0","1","percent","sale","l10n_gr_tax_group_24","","base","invoice","","+303","","","ΦΠΑ πωλήσεων 24% για την Ελλάδα (εκτός των νησιών του Αιγαίου)"
"","","","","","","","","","","","","tax","invoice","","+333","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-303","","",""
"","","","","","","","","","","","","tax","refund","","-333","l10n_gr_54_02_01","",""
"l10n_gr_tax_s13_S","13% S","Sales VAT 13% services for Greece (apart from the Aegean Islands)","","","13%","13.0","1","percent","sale","l10n_gr_tax_group_13","False","base","invoice","","+301","","","ΦΠΑ πωλήσεων 13% για την Ελλάδα (εκτός από τα νησιά του Αιγαίου)"
"","","","","","","","","","","","","tax","invoice","","+331","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-301","","",""
"","","","","","","","","","","","","tax","refund","","-331","l10n_gr_54_02_01","",""
"l10n_gr_tax_s6_S","6% S","Sales VAT 6% services for Greece (apart from the Aegean Islands)","","","6%","6.0","1","percent","sale","l10n_gr_tax_group_6","False","base","invoice","","+302","","","ΦΠΑ πωλήσεων 6% για την Ελλάδα (εκτός των νησιών του Αιγαίου)"
"","","","","","","","","","","","","tax","invoice","","+332","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-302","","",""
"","","","","","","","","","","","","tax","refund","","-332","l10n_gr_54_02_01","",""
"l10n_gr_tax_s9_S","9% S","Sales VAT 9% services for the Aegean Islands","","","9%","9.0","1","percent","sale","l10n_gr_tax_group_9","False","base","invoice","","+304","","","ΦΠΑ πωλήσεων 9% για τα νησιά του Αιγαίου"
"","","","","","","","","","","","","tax","invoice","","+334","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-304","","",""
"","","","","","","","","","","","","tax","refund","","-334","l10n_gr_54_02_01","",""
"l10n_gr_tax_s4_S","4% S","Sales VAT 4% services for the Aegean Islands","","","4%","4.0","1","percent","sale","l10n_gr_tax_group_4","False","base","invoice","","+305","","","ΦΠΑ πωλήσεων 4% για τα νησιά του Αιγαίου"
"","","","","","","","","","","","","tax","invoice","","+335","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-305","","",""
"","","","","","","","","","","","","tax","refund","","-335","l10n_gr_54_02_01","",""
"l10n_gr_tax_s17_S","17% S","Sales VAT 17% services for the Aegean Islands","","","17%","17.0","1","percent","sale","l10n_gr_tax_group_17","False","base","invoice","","+306","","","ΦΠΑ πωλήσεων 17% για τα νησιά του Αιγαίου"
"","","","","","","","","","","","","tax","invoice","","+336","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-306","","",""
"","","","","","","","","","","","","tax","refund","","-336","l10n_gr_54_02_01","",""
"l10n_gr_tax_s0_G_eu","0% EU G","Sales VAT 0% for the EU intra-community supply of goods","consu","","0%","0.0","1","percent","sale","l10n_gr_tax_group_0","","base","invoice","","+342","","","ΦΠΑ πωλήσεων 0% για την ενδοκοινοτική παράδοση αγαθών στην ΕΕ"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-342","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_s0_S_eu","0% EU S","Sales VAT 0% EU for the EU intra-community supply of services","service","","0%","0.0","1","percent","sale","l10n_gr_tax_group_0","","base","invoice","","+345","","","ΦΠΑ πωλήσεων 0% ΕΕ για την ενδοκοινοτική παροχή υπηρεσιών στην ΕΕ"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-345","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_s0_export","0% Export","Sales VAT 0% for the Export & exemption of ships and aircrafts","","","0%","0.0","1","percent","sale","l10n_gr_tax_group_0","","base","invoice","","+348","","","ΦΠΑ πωλήσεων 0% για τις εξαγωγές"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-348","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_s0_deduct","0% Deduct","Sales VAT 0% Other output without TVA with right to deduct","","","0%","0.0","1","percent","sale","l10n_gr_tax_group_0","False","base","invoice","","+349","","","ΦΠΑ πωλήσεων 0% Άλλες εκροές χωρίς TVA με δικαίωμα έκπτωσης"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-349","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_s0_exempt","0% Exempt","Sales VAT 0% Output exempted or excepted, without right to deduct","","","0%","0.0","1","percent","sale","l10n_gr_tax_group_0","False","base","invoice","","+310","","","Φ.Π.Α. πωλήσεων 0% Παραγωγή που απαλλάσσεται ή εξαιρείται, χωρίς δικαίωμα έκπτωσης"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-310","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_p24_G","24% G","Purchases and Expenditures of goods within the Country (VAT 24%)","","","24%","24.0","1","percent","purchase","l10n_gr_tax_group_24","","base","invoice","","+361","","24% Εγχώρια αγορά","Αγορές και δαπάνες στο εσωτερικό της χώρας (ΦΠΑ 24%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p13_G","13% G","Purchases and Expenditures of goods within the Country (VAT 13%)","","","13%","13.0","1","percent","purchase","l10n_gr_tax_group_13","False","base","invoice","","+361","","13% Εγχώρια αγορά","Αγορές και δαπάνες εντός της χώρας (ΦΠΑ 13%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p6_G","6% G","Purchases and Expenditures of goods within the Country (VAT 6%)","","","6%","6.0","1","percent","purchase","l10n_gr_tax_group_6","False","base","invoice","","+361","","6% Εγχώρια αγορά","Αγορές και Δαπάνες στο εσωτερικό της χώρας (ΦΠΑ 6%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p17_G","17% G","Purchases and Expenditures of goods within the Country (VAT 17%)","","","17%","17.0","1","percent","purchase","l10n_gr_tax_group_17","False","base","invoice","","+361","","17% Εγχώρια αγορά","Αγορές και δαπάνες εντός της χώρας (ΦΠΑ 17%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p9_G","9% G","Purchases and Expenditures of goods within the Country (VAT 9%)","","","9%","9.0","1","percent","purchase","l10n_gr_tax_group_9","False","base","invoice","","+361","","19% Εγχώρια αγορά","Αγορές και δαπάνες εντός της χώρας (ΦΠΑ 9%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p4_G","4% G","Purchases and Expenditures of goods within the Country (VAT 4%)","","","4%","4.0","1","percent","purchase","l10n_gr_tax_group_4","False","base","invoice","","+361","","4% Εγχώρια αγορά","Αγορές και δαπάνες εντός της χώρας (ΦΠΑ 4%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p0_G","0% G","Purchases and Expenditures of goods within the Country (VAT 0%)","","","0%","0.0","1","percent","purchase","l10n_gr_tax_group_0","False","base","invoice","","+361","","0% Εγχώρια αγορά","Αγορές και δαπάνες εντός της χώρας (ΦΠΑ 0%)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_p24_S","24% S","Purchases and Expenditures of services within the Country (VAT 24%)","","","24%","24.0","1","percent","purchase","l10n_gr_tax_group_24","","base","invoice","","+361","","24% Εγχώρια αγορά","Αγορές και δαπάνες στο εσωτερικό της χώρας (ΦΠΑ 24%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p13_S","13% S","Purchases and Expenditures of services within the Country (VAT 13%)","","","13%","13.0","1","percent","purchase","l10n_gr_tax_group_13","False","base","invoice","","+361","","13% Εγχώρια αγορά","Αγορές και δαπάνες εντός της χώρας (ΦΠΑ 13%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p6_S","6% S","Purchases and Expenditures of services within the Country (VAT 6%)","","","6%","6.0","1","percent","purchase","l10n_gr_tax_group_6","False","base","invoice","","+361","","6% Εγχώρια αγορά","Αγορές και Δαπάνες στο εσωτερικό της χώρας (ΦΠΑ 6%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p17_S","17% S","Purchases and Expenditures of services within the Country (VAT 17%)","","","17%","17.0","1","percent","purchase","l10n_gr_tax_group_17","False","base","invoice","","+361","","17% Εγχώρια αγορά","Αγορές και δαπάνες εντός της χώρας (ΦΠΑ 17%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p9_S","9% S","Purchases and Expenditures of services within the Country (VAT 9%)","","","9%","9.0","1","percent","purchase","l10n_gr_tax_group_9","False","base","invoice","","+361","","19% Εγχώρια αγορά","Αγορές και δαπάνες εντός της χώρας (ΦΠΑ 9%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p4_S","4% S","Purchases and Expenditures of services within the Country (VAT 4%)","","","4%","4.0","1","percent","purchase","l10n_gr_tax_group_4","False","base","invoice","","+361","","4% Εγχώρια αγορά","Αγορές και δαπάνες εντός της χώρας (ΦΠΑ 4%)"
"","","","","","","","","","","","","tax","invoice","","+381","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","-381","l10n_gr_54_02_02","",""
"l10n_gr_tax_p0_S","0% S","Purchases and Expenditures of services within the Country (VAT 0%)","","","0%","0.0","1","percent","purchase","l10n_gr_tax_group_0","False","base","invoice","","+361","","0% Εγχώρια αγορά","Αγορές και δαπάνες εντός της χώρας (ΦΠΑ 0%)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-361","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_p24_IG","24% IG","Purchases & imports of investment goods (capital goods) (VAT 24%)","","","24%","24.0","1","percent","purchase","l10n_gr_tax_group_24","","base","invoice","","+362","","","Αγορές & εισαγωγές επενδυτικών αγαθών (κεφαλαιουχικών αγαθών) (ΦΠΑ 24%)"
"","","","","","","","","","","","","tax","invoice","","+382","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-362","","",""
"","","","","","","","","","","","","tax","refund","","-382","l10n_gr_54_02_02","",""
"l10n_gr_tax_p13_IG","13% IG","Purchases & imports of investment goods (capital goods) (VAT 13%)","","","13%","13.0","1","percent","purchase","l10n_gr_tax_group_13","False","base","invoice","","+362","","","Αγορές & εισαγωγές επενδυτικών αγαθών (κεφαλαιουχικών αγαθών) (ΦΠΑ 13%)"
"","","","","","","","","","","","","tax","invoice","","+382","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-362","","",""
"","","","","","","","","","","","","tax","refund","","-382","l10n_gr_54_02_02","",""
"l10n_gr_tax_p6_IG","6% IG","Purchases & imports of investment goods (capital goods) (VAT 6%)","","","6%","6.0","1","percent","purchase","l10n_gr_tax_group_6","False","base","invoice","","+362","","","Αγορές & εισαγωγές επενδυτικών αγαθών (κεφαλαιουχικών αγαθών) (ΦΠΑ 6%)"
"","","","","","","","","","","","","tax","invoice","","+382","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-362","","",""
"","","","","","","","","","","","","tax","refund","","-382","l10n_gr_54_02_02","",""
"l10n_gr_tax_p17_IG","17% IG","Purchases & imports of investment goods (capital goods) (VAT 17%)","","","17%","17.0","1","percent","purchase","l10n_gr_tax_group_17","False","base","invoice","","+362","","","Αγορές & εισαγωγές επενδυτικών αγαθών (κεφαλαιουχικά αγαθά) (ΦΠΑ 17%)"
"","","","","","","","","","","","","tax","invoice","","+382","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-362","","",""
"","","","","","","","","","","","","tax","refund","","-382","l10n_gr_54_02_02","",""
"l10n_gr_tax_p9_IG","9% IG","Purchases & imports of investment goods (capital goods) (VAT 9%)","","","9%","9.0","1","percent","purchase","l10n_gr_tax_group_9","False","base","invoice","","+362","","","Αγορές & εισαγωγές επενδυτικών αγαθών (κεφαλαιουχικά αγαθά) (ΦΠΑ 9%)"
"","","","","","","","","","","","","tax","invoice","","+382","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-362","","",""
"","","","","","","","","","","","","tax","refund","","-382","l10n_gr_54_02_02","",""
"l10n_gr_tax_p4_IG","4% IG","Purchases & imports of investment goods (capital goods) (VAT 4%)","","","4%","4.0","1","percent","purchase","l10n_gr_tax_group_4","False","base","invoice","","+362","","","Αγορές & εισαγωγές επενδυτικών αγαθών (κεφαλαιουχικά αγαθά) (ΦΠΑ 4%)"
"","","","","","","","","","","","","tax","invoice","","+382","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-362","","",""
"","","","","","","","","","","","","tax","refund","","-382","l10n_gr_54_02_02","",""
"l10n_gr_tax_p0_IG","0% IG","Purchases & imports of investment goods (capital goods) (VAT 0%)","","","0%","0.0","1","percent","purchase","l10n_gr_tax_group_4","False","base","invoice","","+362","","","Αγορές & εισαγωγές επενδυτικών αγαθών (κεφαλαιουχικά αγαθά) (ΦΠΑ 0%)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-362","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_p24_other_imports","24% Import","Other imports apart from investment goods (capital goods) (VAT 24%)","","","24%","24.0","1","percent","purchase","l10n_gr_tax_group_24","","base","invoice","","+363","","24% Εισαγωγή","Άλλες εισαγωγές εκτός από επενδυτικά αγαθά (κεφαλαιουχικά αγαθά) (ΦΠΑ 24%)"
"","","","","","","","","","","","","tax","invoice","","+383","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-363","","",""
"","","","","","","","","","","","","tax","refund","","-383","l10n_gr_54_02_02","",""
"l10n_gr_tax_p13_other_imports","13% Import","Other imports apart from investment goods (capital goods) (VAT 13%)","","","13%","13.0","1","percent","purchase","l10n_gr_tax_group_13","False","base","invoice","","+363","","13% Εισαγωγή","Άλλες εισαγωγές εκτός από επενδυτικά αγαθά (κεφαλαιουχικά αγαθά) (ΦΠΑ 13%)"
"","","","","","","","","","","","","tax","invoice","","+383","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-363","","",""
"","","","","","","","","","","","","tax","refund","","-383","l10n_gr_54_02_02","",""
"l10n_gr_tax_p6_other_imports","6% Import","Other imports apart from investment goods (capital goods) (VAT 6%)","","","6%","6.0","1","percent","purchase","l10n_gr_tax_group_6","False","base","invoice","","+363","","6% Εισαγωγή","Άλλες εισαγωγές εκτός από επενδυτικά αγαθά (κεφαλαιουχικά αγαθά) (ΦΠΑ 6%)"
"","","","","","","","","","","","","tax","invoice","","+383","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-363","","",""
"","","","","","","","","","","","","tax","refund","","-383","l10n_gr_54_02_02","",""
"l10n_gr_tax_p17_other_imports","17% Import","Other imports apart from investment goods (capital goods) (VAT 17%)","","","17%","17.0","1","percent","purchase","l10n_gr_tax_group_17","False","base","invoice","","+363","","17% Εισαγωγή","Άλλες εισαγωγές εκτός από επενδυτικά αγαθά (κεφαλαιουχικά αγαθά) (ΦΠΑ 17%)"
"","","","","","","","","","","","","tax","invoice","","+383","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-363","","",""
"","","","","","","","","","","","","tax","refund","","-383","l10n_gr_54_02_02","",""
"l10n_gr_tax_p9_other_imports","9% Import","Other imports apart from investment goods (capital goods) (VAT 9%)","","","9%","9.0","1","percent","purchase","l10n_gr_tax_group_9","False","base","invoice","","+363","","9% Εισαγωγή","Άλλες εισαγωγές εκτός από επενδυτικά αγαθά (κεφαλαιουχικά αγαθά) (ΦΠΑ 9%)"
"","","","","","","","","","","","","tax","invoice","","+383","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-363","","",""
"","","","","","","","","","","","","tax","refund","","-383","l10n_gr_54_02_02","",""
"l10n_gr_tax_p4_other_imports","4% Import","Other imports apart from investment goods (capital goods) (VAT 4%)","","","4%","4.0","1","percent","purchase","l10n_gr_tax_group_4","False","base","invoice","","+363","","4% Εισαγωγή","Άλλες εισαγωγές εκτός από επενδυτικά αγαθά (κεφαλαιουχικά αγαθά) (ΦΠΑ 4%)"
"","","","","","","","","","","","","tax","invoice","","+383","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","base","refund","","-363","","",""
"","","","","","","","","","","","","tax","refund","","-383","l10n_gr_54_02_02","",""
"l10n_gr_tax_p0_other_imports","0% Import","Other imports apart from investment goods (capital goods) (VAT 0%)","","","0%","0.0","1","percent","purchase","l10n_gr_tax_group_0","False","base","invoice","","+363","","0% Εισαγωγή","Άλλες εισαγωγές εκτός από επενδυτικά αγαθά (κεφαλαιουχικά αγαθά) (ΦΠΑ 0%)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-363","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_p24_G_eu","24% EU G","Intra-community acquisitions of goods (VAT 24%)","consu","","24%","24.0","1","percent","purchase","l10n_gr_tax_group_24","","base","invoice","","+364||+303","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 24%)"
"","","","","","","","","","","","","tax","invoice","","+384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-333","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-364||-303","","",""
"","","","","","","","","","","","","tax","refund","","-384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+333","l10n_gr_54_02_01","",""
"l10n_gr_tax_p13_G_eu","13% EU G","Intra-community acquisitions of goods (VAT 13%)","consu","","13%","13.0","1","percent","purchase","l10n_gr_tax_group_13","False","base","invoice","","+364||+301","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 13%)"
"","","","","","","","","","","","","tax","invoice","","+384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-331","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-364||-301","","",""
"","","","","","","","","","","","","tax","refund","","-384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+331","l10n_gr_54_02_01","",""
"l10n_gr_tax_p6_G_eu","6% EU G","Intra-community acquisitions of goods (VAT 6%)","consu","","6%","6.0","1","percent","purchase","l10n_gr_tax_group_6","False","base","invoice","","+364||+302","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 6%)"
"","","","","","","","","","","","","tax","invoice","","+384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-332","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-364||-302","","",""
"","","","","","","","","","","","","tax","refund","","-384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+332","l10n_gr_54_02_01","",""
"l10n_gr_tax_p17_G_eu","17% EU G","Intra-community acquisitions of goods (VAT 17%)","consu","","17%","17.0","1","percent","purchase","l10n_gr_tax_group_17","False","base","invoice","","+364||+306","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 17%)"
"","","","","","","","","","","","","tax","invoice","","+384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-336","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-364||-306","","",""
"","","","","","","","","","","","","tax","refund","","-384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+336","l10n_gr_54_02_01","",""
"l10n_gr_tax_p9_G_eu","9% EU G","Intra-community acquisitions of goods (VAT 9%)","consu","","9%","9.0","1","percent","purchase","l10n_gr_tax_group_9","False","base","invoice","","+364||+304","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 9%)"
"","","","","","","","","","","","","tax","invoice","","+384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-334","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-364||-304","","",""
"","","","","","","","","","","","","tax","refund","","-384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+334","l10n_gr_54_02_01","",""
"l10n_gr_tax_p4_G_eu","4% EU G","Intra-community acquisitions of goods (VAT 4%)","consu","","4%","4.0","1","percent","purchase","l10n_gr_tax_group_4","False","base","invoice","","+364||+305","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 4%)"
"","","","","","","","","","","","","tax","invoice","","+384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-335","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-364||-305","","",""
"","","","","","","","","","","","","tax","refund","","-384","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+335","l10n_gr_54_02_01","",""
"l10n_gr_tax_p0_G_eu","0% EU G","Intra-community acquisitions of goods (VAT 0%)","consu","","0%","0.0","1","percent","purchase","l10n_gr_tax_group_0","False","base","invoice","","+364","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 0%)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-364","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_p24_S_eu","24% EU S","Intra-community acquisitions of services (VAT 24%)","service","","24%","24.0","1","percent","purchase","l10n_gr_tax_group_24","","base","invoice","","+365||+303","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 24%)"
"","","","","","","","","","","","","tax","invoice","","+385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-333","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-365||-303","","",""
"","","","","","","","","","","","","tax","refund","","-385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+333","l10n_gr_54_02_01","",""
"l10n_gr_tax_p13_S_eu","13% EU S","Intra-community acquisitions of services (VAT 13%)","service","","13%","13.0","1","percent","purchase","l10n_gr_tax_group_13","False","base","invoice","","+365||+301","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 13%)"
"","","","","","","","","","","","","tax","invoice","","+385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-331","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-365||-301","","",""
"","","","","","","","","","","","","tax","refund","","-385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+331","l10n_gr_54_02_01","",""
"l10n_gr_tax_p6_S_eu","6% EU S","Intra-community acquisitions of services (VAT 6%)","service","","6%","6.0","1","percent","purchase","l10n_gr_tax_group_6","False","base","invoice","","+365||+302","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 6%)"
"","","","","","","","","","","","","tax","invoice","","+385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-332","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-365||-302","","",""
"","","","","","","","","","","","","tax","refund","","-385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+332","l10n_gr_54_02_01","",""
"l10n_gr_tax_p17_S_eu","17% EU S","Intra-community acquisitions of services (VAT 17%)","service","","17%","17.0","1","percent","purchase","l10n_gr_tax_group_17","False","base","invoice","","+365||+306","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 17%)"
"","","","","","","","","","","","","tax","invoice","","+385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-336","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-365||-306","","",""
"","","","","","","","","","","","","tax","refund","","-385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+336","l10n_gr_54_02_01","",""
"l10n_gr_tax_p9_S_eu","9% EU S","Intra-community acquisitions of services (VAT 9%)","service","","9%","9.0","1","percent","purchase","l10n_gr_tax_group_9","False","base","invoice","","+365||+304","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 9%)"
"","","","","","","","","","","","","tax","invoice","","+385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-334","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-365||-304","","",""
"","","","","","","","","","","","","tax","refund","","-385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+334","l10n_gr_54_02_01","",""
"l10n_gr_tax_p4_S_eu","4% EU S","Intra-community acquisitions of services (VAT 4%)","service","","4%","4.0","1","percent","purchase","l10n_gr_tax_group_4","False","base","invoice","","+365||+305","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 4%)"
"","","","","","","","","","","","","tax","invoice","","+385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-335","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-365||-305","","",""
"","","","","","","","","","","","","tax","refund","","-385","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+335","l10n_gr_54_02_01","",""
"l10n_gr_tax_p0_S_eu","0% EU S","Intra-community acquisitions of services (VAT 0%)","service","","0%","0.0","1","percent","purchase","l10n_gr_tax_group_0","False","base","invoice","","+365","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 0%)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-365","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"l10n_gr_tax_p24_O_eu","24% EU Other","Intra-community acquisitions of other than goods and services (VAT 24%)","service","","24%","24.0","1","percent","purchase","l10n_gr_tax_group_24","","base","invoice","","+366||+303","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 24%)"
"","","","","","","","","","","","","tax","invoice","","+386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-333","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-366||-303","","",""
"","","","","","","","","","","","","tax","refund","","-386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+333","l10n_gr_54_02_01","",""
"l10n_gr_tax_p13_O_eu","13% EU Other","Intra-community acquisitions of other than goods and services (VAT 13%)","service","","13%","13.0","1","percent","purchase","l10n_gr_tax_group_13","False","base","invoice","","+366||+301","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 13%)"
"","","","","","","","","","","","","tax","invoice","","+386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-331","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-366||-301","","",""
"","","","","","","","","","","","","tax","refund","","-386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+331","l10n_gr_54_02_01","",""
"l10n_gr_tax_p6_O_eu","6% EU Other","Intra-community acquisitions of other than goods and services (VAT 6%)","service","","6%","6.0","1","percent","purchase","l10n_gr_tax_group_6","False","base","invoice","","+366||+302","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 6%)"
"","","","","","","","","","","","","tax","invoice","","+386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-332","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-366||-302","","",""
"","","","","","","","","","","","","tax","refund","","-386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+332","l10n_gr_54_02_01","",""
"l10n_gr_tax_p17_O_eu","17% EU Other","Intra-community acquisitions of other than goods and services (VAT 17%)","service","","17%","17.0","1","percent","purchase","l10n_gr_tax_group_17","False","base","invoice","","+366||+306","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 17%)"
"","","","","","","","","","","","","tax","invoice","","+386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-336","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-366||-306","","",""
"","","","","","","","","","","","","tax","refund","","-386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+336","l10n_gr_54_02_01","",""
"l10n_gr_tax_p9_O_eu","9% EU Other","Intra-community acquisitions of other than goods and services (VAT 9%)","service","","9%","9.0","1","percent","purchase","l10n_gr_tax_group_9","False","base","invoice","","+366||+304","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 9%)"
"","","","","","","","","","","","","tax","invoice","","+386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-334","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-366||-304","","",""
"","","","","","","","","","","","","tax","refund","","-386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+334","l10n_gr_54_02_01","",""
"l10n_gr_tax_p4_O_eu","4% EU Other","Intra-community acquisitions of other than goods and services (VAT 4%)","service","","4%","4.0","1","percent","purchase","l10n_gr_tax_group_4","False","base","invoice","","+366||+305","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 4%)"
"","","","","","","","","","","","","tax","invoice","","+386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","invoice","-100","-335","l10n_gr_54_02_01","",""
"","","","","","","","","","","","","base","refund","","-366||-305","","",""
"","","","","","","","","","","","","tax","refund","","-386","l10n_gr_54_02_02","",""
"","","","","","","","","","","","","tax","refund","-100","+335","l10n_gr_54_02_01","",""
"l10n_gr_tax_p0_O_eu","0% EU Other","Intra-community acquisitions of other than goods and services (VAT 0%)","service","","0%","0.0","1","percent","purchase","l10n_gr_tax_group_0","False","base","invoice","","+366","","","Ενδοκοινοτικές αποκτήσεις αγαθών (ΦΠΑ 0%)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","-366","","",""
"","","","","","","","","","","","","tax","refund","","","","",""

```

## File: data\template\account.tax.group-gr.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","name@gr"
"l10n_gr_tax_group_4","VAT 4%","base.gr","l10n_gr_54_02_03","l10n_gr_54_02_03","ΦΠΑ 4%"
"l10n_gr_tax_group_6","VAT 6%","base.gr","l10n_gr_54_02_03","l10n_gr_54_02_03","ΦΠΑ 6%"
"l10n_gr_tax_group_9","VAT 9%","base.gr","l10n_gr_54_02_03","l10n_gr_54_02_03","ΦΠΑ 9%"
"l10n_gr_tax_group_13","VAT 13%","base.gr","l10n_gr_54_02_03","l10n_gr_54_02_03","ΦΠΑ 13%"
"l10n_gr_tax_group_17","VAT 17%","base.gr","l10n_gr_54_02_03","l10n_gr_54_02_03","ΦΠΑ 17%"
"l10n_gr_tax_group_24","VAT 24%","base.gr","l10n_gr_54_02_03","l10n_gr_54_02_03","ΦΠΑ 24%"
"l10n_gr_tax_group_0","VAT 0%","base.gr","l10n_gr_54_02_03","l10n_gr_54_02_03","ΦΠΑ 0%"

```

## File: models\template_gr.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('gr')
    def _get_gr_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_gr_30_01_01_01',
            'property_account_payable_id': 'l10n_gr_50_01_01',
            'property_account_expense_categ_id': 'l10n_gr_64_01_01_01',
            'property_account_income_categ_id': 'l10n_gr_70_01_01',
            'code_digits': '6',
        }

    @template('gr', 'res.company')
    def _get_gr_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.gr',
                'bank_account_code_prefix': '38.00.0',
                'cash_account_code_prefix': '38.00.0',
                'transfer_account_code_prefix': '38.00.0',
                'account_default_pos_receivable_account_id': 'l10n_gr_38_01',
                'income_currency_exchange_account_id': 'l10n_gr_73_01_01',
                'expense_currency_exchange_account_id': 'l10n_gr_62_01_01',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_gr_64_12_01',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_gr_70_01_03',
                'default_cash_difference_income_account_id': 'l10n_gr_71_06',
                'default_cash_difference_expense_account_id': 'l10n_gr_64_14',
                'account_sale_tax_id': 'l10n_gr_tax_s24_G',
                'account_purchase_tax_id': 'l10n_gr_tax_p24_G',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_gr

```

