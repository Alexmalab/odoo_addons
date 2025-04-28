# Odoo Module: l10n_ke

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Kenya - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/kenya.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ke'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This provides a base chart of accounts and taxes template for use in Odoo.
    """,
    'depends': [
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'views/account_move_views.xml',
        'views/account_tax_views.xml',
        'views/l10n_ke_item_code_views.xml',
        'data/l10n_ke.item.code.csv',
        'data/account_tax_report_data.xml',
        'data/account_wh_tax_report_data.xml',
        'security/ir.model.access.csv',
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
    <record id="tax_report_ke" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ke"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_base_column" model="account.report.column">
                <field name="name">Base</field>
                <field name="expression_label">base</field>
            </record>
            <record id="tax_report_tax_column" model="account.report.column">
                <field name="name">VAT</field>
                <field name="expression_label">tax</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_general_rate_sales" model="account.report.line">
                <field name="name">1. Taxable Sales (General Rate 16%)</field>
                <field name="code">box_1</field>
                <field name="expression_ids">
                    <record id="tax_report_general_rate_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">16% Sales Base</field>
                    </record>
                    <record id="tax_report_general_rate_sales_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">16% Sales Tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_other_rate_sales" model="account.report.line">
                <field name="name">2. Taxable Sales (Other Rate 8%)</field>
                <field name="code">box_2</field>
                <field name="expression_ids">
                    <record id="tax_report_other_rate_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8% Sales Base</field>
                    </record>
                    <record id="tax_report_other_rate_sales_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8% Sales Tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_zero_rated_sales" model="account.report.line">
                <field name="name">3. Sales (Zero Rated 0%)</field>
                <field name="code">box_3</field>
                <field name="expression_ids">
                    <record id="tax_report_zero_rated_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Zero Rated Sales Base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_exempt_sales" model="account.report.line">
                <field name="name">4. Sales (Exempt)</field>
                <field name="code">box_4</field>
                <field name="expression_ids">
                    <record id="tax_report_exempt_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Exempt Sales Base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_total_sales" model="account.report.line">
                <field name="name">5. Total Sales</field>
                <field name="code">box_5</field>
                <field name="expression_ids">
                    <record id="tax_report_total_sales_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_1.base + box_2.base + box_3.base + box_4.base</field>
                    </record>
                    <record id="tax_report_total_sales_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_1.tax + box_2.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_output_vat" model="account.report.line">
                <field name="name">6. Total Output VAT</field>
                <field name="code">box_6</field>
                <field name="expression_ids">
                    <record id="tax_report_output_vat_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_1.base + box_2.base + box_3.base</field>
                    </record>
                    <record id="tax_report_output_vat_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_1.tax + box_2.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_general_rate_purchases" model="account.report.line">
                <field name="name">7. Taxable Purchases (General Rate 16%)</field>
                <field name="code">box_7</field>
                <field name="expression_ids">
                    <record id="tax_report_general_rate_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">16% Purchases Base</field>
                    </record>
                    <record id="tax_report_general_rate_purchases_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">16% Purchases Tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_other_rate_purchases" model="account.report.line">
                <field name="name">8. Taxable Purchases (Other Rate 8%)</field>
                <field name="code">box_8</field>
                <field name="expression_ids">
                    <record id="tax_report_other_rate_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8% purchases Base</field>
                    </record>
                    <record id="tax_report_other_rate_purchases_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8% Purchases Tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_zero_rated_purchases" model="account.report.line">
                <field name="name">9. Purchases (Zero Rated 0%)</field>
                <field name="code">box_9</field>
                <field name="expression_ids">
                    <record id="tax_report_zero_rated_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Zero Rated Purchases Base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_exempt_purchases" model="account.report.line">
                <field name="name">10. Purchases (Exempt)</field>
                <field name="code">box_10</field>
                <field name="expression_ids">
                    <record id="tax_report_exempt_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Exempt Purchases Base</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_total_purchases" model="account.report.line">
                <field name="name">11. Total Purchases</field>
                <field name="code">box_11</field>
                <field name="expression_ids">
                    <record id="tax_report_total_purchases_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_7.base + box_8.base + box_9.base + box_10.base</field>
                    </record>
                    <record id="tax_report_total_purchases_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_7.tax + box_8.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_input_vat" model="account.report.line">
                <field name="name">12. Total Input VAT</field>
                <field name="code">box_12</field>
                <field name="expression_ids">
                    <record id="tax_report_input_vat_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_7.base + box_8.base + box_9.base</field>
                    </record>
                    <record id="tax_report_input_vat_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_7.tax + box_8.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_vat_claimable_on_imported_services" model="account.report.line">
                <field name="name">13. VAT Claimable on Services Imported into Kenya</field>
                <field name="code">box_13</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_claimable_on_imported_services_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Import Base</field>
                    </record>
                    <record id="tax_report_vat_claimable_on_imported_services_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Import Tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_input_vat_attributable_to_exempt_supplies" model="account.report.line">
                <field name="name">14. Input VAT attributable to Only Exempt Supplies</field>
                <field name="code">box_14</field>
                <field name="expression_ids">
                    <record id="tax_report_input_vat_attributable_to_exempt_supplies_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable;rounding=2</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_input_vat_attributable_to_taxable_and_exempt_supplies" model="account.report.line">
                <field name="name">15. Input VAT attributable to Taxable and Exempt Supplies</field>
                <field name="code">box_15</field>
                <field name="expression_ids">
                    <record id="tax_report_input_vat_attributable_to_taxable_and_exempt_supplies_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable;rounding=2</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_non_deductible_input_vat" model="account.report.line">
                <field name="name">16. Non-Deductible Input VAT</field>
                <field name="code">box_16</field>
                <field name="expression_ids">
                    <record id="tax_report_non_deductible_input_vat_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_15.tax - (((box_1.tax + box_2.tax + box_3.base) / box_5.base) * box_15.tax)</field>
                        <field name="subformula">ignore_zero_division</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_deductible_input_vat" model="account.report.line">
                <field name="name">17. Deductible Input VAT</field>
                <field name="code">box_17</field>
                <field name="expression_ids">
                    <record id="tax_report_deductible_input_vat_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_12.tax + box_13.tax - box_14.tax - box_16.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_vat_payable_or_credit_due_for_period" model="account.report.line">
                <field name="name">18. VAT Payable/Credit Due for the period</field>
                <field name="code">box_18</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_payable_or_credit_due_for_period_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_6.tax - box_17.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_credit_brought_forward_from_previous_month" model="account.report.line">
                <field name="name">19. Credit Brought Forward from previous month</field>
                <field name="code">box_19</field>
                <field name="expression_ids">
                    <record id="tax_report_credit_brought_forward_from_previous_month_applied_carryover" model="account.report.expression">
                        <field name="label">_applied_carryover_tax</field>
                        <field name="engine">external</field>
                        <field name="formula">most_recent</field>
                        <field name="date_scope">previous_tax_period</field>
                    </record>
                    <record id="tax_report_credit_brought_forward_from_previous_month_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">-box_19._applied_carryover_tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_total_withholding_vat_credit" model="account.report.line">
                <field name="name">20. Total Withholding VAT Credit</field>
                <field name="code">box_20</field>
                <field name="expression_ids">
                    <record id="tax_report_total_withholding_vat_credit_base_tag" model="account.report.expression">
                        <field name="label">base</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">WH base</field>
                    </record>
                    <record id="tax_report_total_withholding_vat_credit_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">WH Sales</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_refund_claim_lodged" model="account.report.line">
                <field name="name">21. Refund Claim Lodged</field>
                <field name="code">box_21</field>
                <field name="expression_ids">
                    <record id="tax_report_refund_claim_lodged_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable;rounding=2</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_total_vat_payable" model="account.report.line">
                <field name="name">22. Total VAT Payable</field>
                <field name="code">box_22</field>
                <field name="expression_ids">
                    <record id="tax_report_total_vat_payable_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_18.tax - box_19.tax - box_20.tax + box_21.tax</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_total_vat_paid" model="account.report.line">
                <field name="name">23. Total VAT Paid</field>
                <field name="code">box_23</field>
                <field name="expression_ids">
                    <record id="tax_report_vat_paid_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable;rounding=2</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_total_credit_adjustment_inventory_approval_order" model="account.report.line">
                <field name="name">24. Total Credit Adjustment/Inventory Approval Order</field>
                <field name="code">box_24</field>
                <field name="expression_ids">
                    <record id="tax_report_total_credit_adjustment_inventory_approval_order_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable;rounding=2</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_total_debit_adjustment_voucher" model="account.report.line">
                <field name="name">25. Total Debit Adjustment Voucher</field>
                <field name="code">box_25</field>
                <field name="expression_ids">
                    <record id="tax_report_total_debit_adjustment_voucher_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable;rounding=2</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_net_vat_payable_or_credit_carried_forward" model="account.report.line">
                <field name="name">26. Net VAT Payable/Credit Carried Forward</field>
                <field name="code">box_26</field>
                <field name="expression_ids">
                    <record id="tax_report_net_vat_payable_or_credit_carried_forward_tax_tag" model="account.report.expression">
                        <field name="label">tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_22.tax - box_23.tax - box_24.tax + box_25.tax</field>
                    </record>
                    <record id="tax_report_net_vat_payable_or_credit_carried_forward_carryover" model="account.report.expression">
                        <field name="label">_carryover_tax</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">box_26.tax</field>
                        <field name="carryover_target">box_19._applied_carryover_tax</field>
                        <field name="subformula">if_below(RON(0))</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_wh_tax_report_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>
    <record id="wh_tax_report_ke" model="account.report">
        <field name="name">WH Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ke"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="wh_tax_report_base_column" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="wh_tax_report_ke_to_be_paid_selected_month" model="account.report.line">
                <field name="name">To be paid before the 20th of the selected month</field>
                <field name="aggregation_formula">-ke_total_wh_vat_selected_month.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="wh_tax_report_ke_total_purchases_selected_month" model="account.report.line">
                        <field name="name">Total purchases subject to WH VAT</field>
                        <field name="code">ke_total_purchases_selected_month</field>
                        <field name="expression_ids">
                            <record id="wh_tax_report_ke_to_be_paid_selected_month_expr" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH base</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                        </field>
                    </record>
                    <record id="wh_tax_report_ke_total_wh_vat_selected_month" model="account.report.line">
                        <field name="name">Total WH VAT</field>
                        <field name="code">ke_total_wh_vat_selected_month</field>
                        <field name="expression_ids">
                            <record id="wh_tax_report_total_wh_vat_selected_month_expr" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH 2%</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="wh_tax_report_ke_to_be_paid_next_month" model="account.report.line">
                <field name="name">To be paid before the 20th of the next month</field>
                <field name="aggregation_formula">-ke_total_wh_vat_next_month.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="wh_tax_report_ke_total_purchases_next_month" model="account.report.line">
                        <field name="name">Total purchases subject to WH VAT</field>
                        <field name="code">ke_total_purchases_next_month</field>
                        <field name="expression_ids">
                            <record id="wh_tax_report_ke_to_be_paid_next_month_expr" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH base</field>
                            </record>
                        </field>
                    </record>
                    <record id="wh_tax_report_ke_total_wh_vat_next_month" model="account.report.line">
                        <field name="name">Total WH VAT</field>
                        <field name="code">ke_total_wh_vat_next_month</field>
                        <field name="expression_ids">
                            <record id="wh_tax_report_total_wh_vat_next_month_expr" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">WH 2%</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_ke.item.code.csv

```csv
id,code,description,tax_rate
item_code_2023_00011100,0001.11.00,Bovine Semen of tariff number 05111000,E
item_code_2023_00021100,0002.11.00,Fish eggs and roes of tariff number 05119110,E
item_code_2023_00031100,0003.11.00,Animal semen other than of bovine of tariff number 05119110,E
item_code_2023_00041100,0004.11.00,Soya beans whether or not broken of tariff numbers 12011000 and 12019000,E
item_code_2023_00051100,0005.11.00,Groundnuts not roasted or otherwise cooked In shell of tariff number 12024100,E
item_code_2023_00061100,0006.11.00,Groundnuts not roasted or otherwise cooked Shelled whether or not broken of tariff number 12024200,E
item_code_2023_00071100,0007.11.00,Copra of tariff number 12030000,E
item_code_2023_00081100,0008.11.00,Linseed whether or not broken of tariff number 12040000,E
item_code_2023_00091100,0009.11.00,Low erucic acid rape or colza seed of tariff number 12051000,E
item_code_2023_00101100,0010.11.00,Other rape or colza seed of tariff number  12059000,E
item_code_2023_00111100,0011.11.00,Sunflower seeds whether or not broken of tariff number 12060000,E
item_code_2023_00121100,0012.11.00,Cotton seeds  whether or not broken  Seed of tariff numbers 12072100 and 12072900,E
item_code_2023_00131100,0013.11.00,Sesamum seeds whether or not broken of tariff number 12074000,E
item_code_2023_00141100,0014.11.00,Mustard seeds whether or  not broken of tariff number 12070500,E
item_code_2023_00151100,0015.11.00,Safflower seeds  whether or  not broken 12076000,E
item_code_2023_00161100,0016.11.00,Other oil seeds and oleaginous fruits whether or not broken of tariff number 12079900,E
item_code_2023_00171101,0017.11.01,Pyrethrum flower of tariff number 12119020,E
item_code_2023_00171102,0017.11.02,Sugarcane of tariff number 12129300,E
item_code_2023_00171103,0017.11.03,Unprocessed produce of plant species camellia sinensis,E
item_code_2023_00181100,0018.11.00,Live animals of chapter 1,E
item_code_2023_00191100,0019.11.00,Meat and edible offals of chapter 2 excluding those of heading 0209 and 0210,E
item_code_2023_00201100,0020.11.00,Fish and crustaceans molluscs and other qauticinvertebrates of chapter 3 excluding those of tariff heading 0305 0306 and 0307,E
item_code_2023_00211100,0021.11.00,Unprocessed milk,E
item_code_2023_00221100,0022.11.00,Fresh birds eggs in shell,E
item_code_2023_00231100,0023.11.00,Edible Vegetables and certain roots and tubers of Chapter 7 excluding those of tariff heading 0711,E
item_code_2023_00241100,0024.11.00,Edible fruits and nuts peal of citrus fruits or melon of chapter 8 excluding those of tariff heading 0811 0812 0813 and 0814,E
item_code_2023_00251100,0025.11.00,Cereals of Chapter 10 excluding seeds of tariff heading 1002,E
item_code_2023_00301101,0030.11.01,Taxable supplies imported or purchased for direct and exclusive use in the geothermal oil or mining prospecting or exploration by a company granted prospecting or exploration license in accordance with Geothermal Resources Act No 12 of 1982 production sharing contracts in accordance with the provisions of Petroleum Act Cap 306 upon recommendation by the Cabinet Secretary responsible for energy or the Cabinet Secretary responsible for mining as the case may be,E
item_code_2023_00321100,0032.11.00,Syringes with or without needles of tariff no 90183100,E
item_code_2023_00351100,0035.11.00,Tubular metal needles and needles for sutures of tariff number 90183200,E
item_code_2023_00361100,0036.11.00,Catheters cannula and the like of tariff number 90183900,E
item_code_2023_00371100,0037.11.00,Blood bags,E
item_code_2023_00381100,0038.11.00,Blood and fluid infusion sets,E
item_code_2023_00391101,0039.11.01,Materials articles and equipment including motor vehicles Which are specially designed for the sole use by disabled blind or physically handicapped persons,E
item_code_2023_00391102,0039.11.02,Materials articles and equipment Including motor vehicles Which are intended for the educational scientific or cultural advancement of the blind for the use of an organization approved by the national government for purposes of the exemption,E
item_code_2023_00391103,0039.11.03,Streptomycins and their derivatives salts thereof 29412000,E
item_code_2023_00391104,0039.11.04,Tetracycline and their derivatives salts thereof 29413000,E
item_code_2023_00391105,0039.11.05,Chloramphenicol and its derivatives salts thereof 29414000,E
item_code_2023_00391106,0039.11.06,Erythromycin and its derivatives salts thereof 29415000,E
item_code_2023_00391107,0039.11.07,Other Antibiotics 29419000,E
item_code_2023_00391108,0039.11.08,Extracts of glands or other organs or of their secretions 30012000,E
item_code_2023_00391111,0039.11.11,Vaccines for Human Medicine 30022000,E
item_code_2023_00391112,0039.11.12,Vaccines for veterinary medicine 30023000,E
item_code_2023_00391113,0039.11.13,Medicaments Containing penicillin or derivatives thereof with penicillanic acid structure or streptomycins or their derivatives 30031000,E
item_code_2023_00391114,0039.11.14,Medicaments containing other antibiotics not put up in measured doses or in forms for retail sale 30032000,E
item_code_2023_00391115,0039.11.15,Medicacaments containing insulin not put up in measured doses or in forms for retail sale 30033100,E
item_code_2023_00391116,0039.11.16,Other medicaments containing hormones or other products of heading No 2937 but not containing antibiotics not put up in measured doses or in forms or pickings for retail sale 30033900,E
item_code_2023_00391117,0039.11.17,Medicaments containing alkaloids or derivatives thereof but not containing hormones or other products of heading No 2937 or antibiotics not put up in measured doses or in forms or pickings for retail sale 30034000,E
item_code_2023_00391118,0039.11.18,Other 30039000,E
item_code_2023_00391119,0039.11.19,Infusion solutions for ingestion other than by mouth not put up in measured doses or in forms or pickings for retail sale 30039010,E
item_code_2023_00391120,0039.11.20,Other medicaments excluding goods of heading No 3002 3005 or 3006 consisting of two or more constituents which have been mixed together for therapeutic or prophylactic uses not put up in measured doses or in forms or packings for retail sale 30039090,E
item_code_2023_00391121,0039.11.21,Medicaments containing penicillins or derivatives thereof with a penicillanic acid structure or streptomycins or their derivatives put up in measured doses or in forms or packings for retail sale 30041000,E
item_code_2023_00391122,0039.11.22,Medicaments containing other antibiotics put up in measured doses or in forms or packings for retail sale 30042000,E
item_code_2023_00391123,0039.11.23,Medicaments containing insulin put up in measured doses or in forms or packings for retail sale 30043100,E
item_code_2023_00391124,0039.11.24,Medicaments containing adrenal cortical hormones put up in measured doses or in forms or packings for retail sale 30043200,E
item_code_2023_00391125,0039.11.25,Other medicaments containing hormones or other products of heading No 2937 but not containing antibiotics put up in measured doses or in forms or packings for retail sale 30043900,E
item_code_2023_00391126,0039.11.26,Containing ephedrine or its salts 30044100,E
item_code_2023_00391127,0039.11.27,Containing pseudoephedrine 30044200,E
item_code_2023_00391128,0039.11.28,Other 30045000,E
item_code_2023_00391129,0039.11.29,Other medicaments containing vitamins or other products of heading No 2936 put up in measured doses or in forms or packings for retail sale 30045000,E
item_code_2023_00391130,0039.11.30,Other medicaments excluding goods of heading No 3002 3005 or 3006 consisting of mixed or unmixed products for therapeutic or prophylactic uses put up in measured doses or in forms or packings for retail sale 30049000,E
item_code_2023_00391131,0039.11.31,Other medicaments excluding goods of heading No 3002 3005 or 3006 consisting of mixed or unmixed products for therapeutic or prophylactic uses put up in measured doses or in forms or packings for retail sale 30049090,E
item_code_2023_00391132,0039.11.32,Adhesive dressings and other articles having an adhesive layer impregnated or coated with pharmaceutical substances or put up in forms of packagings for retail sale for medical surgical dental or veterinary purposes 30051000,E
item_code_2023_00391133,0039.11.33,White absorbent cotton Wadding impregnated or coated with pharmaceutical substances or put up in forms of packagings for retail sale for medical surgical dental or veterinary purposes 30059010,E
item_code_2023_00391134,0039.11.34,Other wadding gauze bandages and similar articles impregnated or coated with pharmaceutical substances or put up in forms of packagings for retail sale for medical surgical dental or veterinary purposes 30059090,E
item_code_2023_00391135,0039.11.35,Sterile surgical catgut similar sterile suture materials and sterile tissue adhesives for surgical wound closure sterile laminnaria and sterile laminaria tents sterile absorbable surgical or dental hemeostatics 30061000,E
item_code_2023_00391136,0039.11.36,Blood grouping reagents 30062000,E
item_code_2023_00391137,0039.11.37,Opacifying preparations for Xray examinations diagnostic reagents designed to be administered to the patient 30063000,E
item_code_2023_00391138,0039.11.38,Dental cements and other dental fillings bone reconstruction cements 30064000,E
item_code_2023_00391139,0039.11.39,Firstaid boxes and kits 30065000,E
item_code_2023_00391140,0039.11.40,Chemical contraceptive preparations based on hormone pr spermicides 30066000,E
item_code_2023_00391141,0039.11.41,Gel preparations to be used in human or veterinary medicine as a lubricant for parts of the body for surgical operations or physical examinations oe as a coupling agent between the body and medical instruments 30067000,E
item_code_2023_00391142,0039.11.42,Appliances identifiable for ostomy use 30069100,E
item_code_2023_00391143,0039.11.43,Waste pharmaceuticals 30069200,E
item_code_2023_00391150,0039.11.50,Aeroplanes and other Aircraft or unladen weight exceeding 2000kg but not exceeding 15000kg 88023000,E
item_code_2023_00391151,0039.11.51,Aeroplanes and other Aircraft of unladen weight exceeding 15000kg 88024000,E
item_code_2023_00391153,0039.11.53,Spacecraft including satellites and suborbital and spacecraft launch vehicles 88026000,E
item_code_2023_00391157,0039.11.57,Sanitary towels pads and tampons 96190010,E
item_code_2023_00391162,0039.11.62,Milk specially prepared for infants and tariff no as listed in the first schedule of VAT Act 2013 04022910,E
item_code_2023_00391165,0039.11.65,Semi milled or wholly milled rice whether polished or glazed 10063000,E
item_code_2023_00391169,0039.11.69,Protein concentrates and textured protein substances 21061000,E
item_code_2023_00391170,0039.11.70,Food preparations specially prepared from infants 21069010,E
item_code_2023_00391171,0039.11.71,Other food preparations not elsewhere specified or included 21069099,E
item_code_2023_00391172,0039.11.72,Vitamin C and its derivatives 29362700,E
item_code_2023_00391173,0039.11.73,Other Heparin and its salts 30019000,E
item_code_2023_00391174,0039.11.74,Other human or animal substances prepared for therapeutic or prophylactic uses not elsewhere specified or included 30019000,E
item_code_2023_00391175,0039.11.75,Malaria diagnostic test kits 30021100,E
item_code_2023_00391176,0039.11.76,Antisera and other blood fractions 30021200,E
item_code_2023_00391177,0039.11.77,Immunological products unmixed not put up in measured doses or in forms or packagings for retail sale 30021200,E
item_code_2023_00391178,0039.11.78,Immunological products mixed not put up in measured doses or in forms or packagings for retail sale 30021400,E
item_code_2023_00391179,0039.11.79,Immunological products put up in measured doses or in forms or packagings for retail sale 30021500,E
item_code_2023_00391180,0039.11.80,Other Antisera other blood fractions and immunological products whether or not modified or obtained by means of biotechnological process 30021900,E
item_code_2023_00391181,0039.11.81,Insulin 30033100,E
item_code_2023_00391182,0039.11.82,Other medicaments containing alkaloids or derivatives containing norephedrine or its salts 30044300,E
item_code_2023_00391183,0039.11.83,Other containing antimalarial active principles described in Subheading Note 2 30046000,E
item_code_2023_00391184,0039.11.84,Food supplements 21069091,E
item_code_2023_00391185,0039.11.85,Milk in powder granules or other solid forms of a fat content by weight exceeding  one point five percent not containing added sugar or other sweetening matter 04022100,E
item_code_2023_00391186,0039.11.86,Other milk in powder granules or other solid forms of a fat content by weight exceeding one point five percent 04022900,E
item_code_2023_00391187,0039.11.87,Other not containing added sugar or other sweetening mater 04029100,E
item_code_2023_00391188,0039.11.88,Other milk 04029900,E
item_code_2023_00391189,0039.11.89,Orthopedic or fracture appliances 90211000,E
item_code_2023_00391190,0039.11.90,Other artificial parts of the body Pacemakers for stimulating heart muscles excluding parts and accessories 90215000,E
item_code_2023_00391191,0039.11.91,Hydrometers and similar floating instruments thermometers pyrometers barometers hygrometers and psychrometers recording or not and any combination of these instruments thermometers and pyrometers not combined with other instruments Other 90251900,E
item_code_2023_00391192,0039.11.92,Airway Guedel and Ambu bags 90192000,E
item_code_2023_00391193,0039.11.93,Blood giving set and infusion sets 90189000,E
item_code_2023_00401100,0040.11.00,Made up fishing nets of manmade textile material of tariff number 56081100,E
item_code_2023_00411100,0041.11.00,Mosquito nets of tariff No 63049110,E
item_code_2023_00431100,0043.11.00,Materials waste residues and byproducts whether or not in the form of pellets and preparations of a kind used in animal feeding of tariff numbers as listed in Part 1 of the first schedule of VAT Act 2013,E
item_code_2023_00441100,0044.11.00,Unprocessed green tea,E
item_code_2023_00481100,0048.11.00,Inputs or raw materials supplied to solar equipment manufacturers for manufacture of solar equipment or deep cycle sealed batteries which exclusively use or store solar power as approved from time to time by the Cabinet Secretary for the National Treasury upon recommendation by the Cabinet Secretary responsible for energy and petroleum,E
item_code_2023_00491100,0049.11.00,Aircraft parts of heading 8803 excluding parts of goods of heading 8801,E
item_code_2023_00511100,0051.11.00,Taxable goods imported or purchased for direct and exclusive use in the implementation of official aid funded projects upon approval by the Cabinet Secretary responsible for the National Treasury,E
item_code_2023_00541100,0054.11.00,Goods imported or purchased locally for use by the local film producers and local filming agents upon recommendation by the Kenya Film Commission subject to approval by the Cabinet Secretary to the National Treasury,E
item_code_2023_00561100,0056.11.00,Inputs or raw materials locally purchased or imported by manufacturers of agricultural machinery and implements upon approval by the Cabinet Secretary responsible for industrialization,E
item_code_2023_00571100,0057.11.00,All goods including material supplies equipment machinery and motor vehicles for official use by the Kenya Defense Forces and the National Police Service,E
item_code_2023_00581100,0058.11.00,Direction finding compasses instruments and appliances for aircraft,E
item_code_2023_00591100,0059.11.00,Wheat seeds of tariff numbers 10011100 and 10019100,E
item_code_2023_00621100,0062.11.00,Taxable goods for direct and exclusive use for the construction of tourism facilities recreational parks of fifty acres or more convention and conference facilities upon recommendation by the Cabinet Secretary responsible for matters relating to recreational parks,E
item_code_2023_00631100,0063.11.00,Taxable goods equipment and apparatus for the direct and exclusive use for construction of specialized hospitals with a minimum bed capacity of fifty with accommodation facilities upon the recommendation by the Cabinet Secretary responsible for health who shall issue guidelines for the criteria to be used to determine eligibility for the exemption provided that notwithstanding this subparagraph any approval granted by the cabinet Secretary before the commencement thereof in respect of the supply of taxable goods and which is in supply of taxable goods and which is in force at such commencement shall continue to apply of the E taxable goods is made in full,E
item_code_2023_00661100,0066.11.00,Inputs or raw materials locally purchased or imported by manufacturers of clean cook stoves approved by the Cabinet Secretary upon recommendation by the Cabinet Secretary for the time being responsible for energy,E
item_code_2023_00661101,0066.11.01,Bioethanol vapor BEV Stoves classified under HS Code 73211100 cooking appliances and plate warmers for liquid fuel,E
item_code_2023_00681100,0068.11.00,Super absorbent polymer SAP of tariff number 39069000,E
item_code_2023_00691100,0069.11.00,Carrier tissue white 1 ply 14 point 5 GSM,E
item_code_2023_00701100,0070.11.00,IP super soft fluff pulp for fluff 310 treated pulp 488 times 125 mm cellulose of tariff number 47032100,E
item_code_2023_00711100,0071.11.00,Perforated PE film 15 to 22 gsm of tariff number 39219000,E
item_code_2023_00721100,0072.11.00,Spun bound nonwoven 15 to 25 gsm of tariff number 56031100,E
item_code_2023_00731100,0073.11.00,Airlid paper with super absorbent polymer 180gsm67 of tariff number 48030000,E
item_code_2023_00741100,0074.11.00,Airlid paper with super absorbent polymer 80gsm67 of tariff number 480300,E
item_code_2023_00771100,0077.11.00,Pressure sensitive adhesive of tariff number 35069100,E
item_code_2023_00781100,0078.11.00,Plain polythene film LPDE of tariff number 39211910,E
item_code_2023_00791100,0079.11.00,Plain polythene film PE of tariff number 39211910,E
item_code_2023_00801100,0080.11.00,PE white 25 40 gsm release paper of tariff number 48114900,E
item_code_2023_00811100,0081.11.00,ADL 25 to 40 gsm of tariff number 56031100,E
item_code_2023_00821100,0082.11.00,Elasticized side tape of tariff number 54024400,E
item_code_2023_00831100,0083.11.00,12 to 16 gsm spun bound piyropo nonwoven cover stock 12gsm spun bound PP nonwoven SMS hydrophobic leg cuffs of tariff number 56031100,E
item_code_2023_00841100,0084.11.00,Polymetric elastic 2 over 3 strands of tariff number 39199010,E
item_code_2023_00891100,0089.11.00,Any other aircraft spare parts imported by aircraft operators or persons engaged in the business of aircraft maintenance upon recommendation by the competent authority responsible for civil aviation,E
item_code_2023_00901100,0090.11.00,Inputs for the manufacture of pesticides upon recommendation by the Cabinet Secretary for the time being responsible for matters relating to agriculture,E
item_code_2023_00911100,0091.11.00,Specially designed locally assembled motor vehicles for transportation of tourists purchased before clearance through Customs by tour operators upon recommendation by the competent authority responsible for tourism promotion provided the vehicles meet the following conditions as specified in the VAT Act 2013 First Schedule section A no 91,E
item_code_2023_00951100,0095.11.00,The supply of natural water excluding bottled water by a National Government County Government any political subdivision thereof or a person approved by the Cabinet Secretary for the time being responsible for water development for domestic or for industrial use,E
item_code_2023_00961101,0096.11.01,Articles of apparel clothing accessories and equipment specially designed for safety or protective purposes for use in registered hospitals and clinics or by county government or local authorities in firefighting,E
item_code_2023_00961102,0096.11.02,Personal protective equipment including facemasks or use by medical personnel in registered hospitals and clinics or by members of the public in the case of a pandemic or a notifiable infectious disease,E
item_code_2023_00991100,0099.11.00,Goods imported by passengers arriving from places outside Kenya as specified in the VAT Act 2013 First Schedule section A no 99,E
item_code_2023_01001100,0100.11.00,Taxable goods for emergency relief purposes for use in specific areas and within a specified period supplied to or imported by the Government or its approved agent a nongovernmental organization or a relief agency authorized by the Cabinet Secretary responsible for disaster management where goods are specified in the VAT Act 2013 First Schedule section A no 100,E
item_code_2023_01011100,0101.11.00,Alcoholic or nonalcoholic beverages supplied to the Kenya Defense Forces Canteen Organization,E
item_code_2023_01031100,0103.11.00,Hearing aids excluding parts and accessories of tariff Number 90214000,E
item_code_2023_01051100,0105.11.00,Locally manufactured motherboards,E
item_code_2023_01061100,0106.11.00,Inputs for the manufacture of motherboards approved by the Cabinet Secretary responsible for information communication technology,E
item_code_2023_01071100,0107.11.00,Plant machinery and equipment used in the construction of a plastics recycling plant,E
item_code_2023_01081100,0108.11.00,The supply of maize corn flour cassava flour wheat or meslin flour and maize flour containing cassava flour by more than ten percent in weight provided that this paragraph,E
item_code_2023_01091100,0109.11.00,Goods imported or purchased locally for the direct and exclusive use in the construction of houses under an affordable housing scheme approved by the Cabinet Secretary on the recommendation of the Cabinet Secretary responsible for matters relating to housing,E
item_code_2023_01101100,0110.11.00,Musical instruments and other musical equipment imported or purchased locally for exclusive use by educational institutions upon recommendation by the Cabinet Secretary responsible for Education,E
item_code_2023_01111100,0111.11.00,Maize corn seeds of tariff no 10051000,E
item_code_2023_01121100,0112.11.00,Taxable goods excluding motor vehicles imported or purchased for direct and exclusive use in geothermal oil or mining prospecting or exploration by a company granted a prospecting or exploration license in accordance with the Energy Act 2019 production sharing contracts in accordance with the Petroleum Act 2019 or a mining license in accordance with the Mining Act 2016 upon recommendation by the cabinet secretary responsible for matters relating to energy or the cabinet secretary responsible for matters relating to mining as the case may be,E
item_code_2023_01131100,0113.11.00,Specialized equipment for the development and generation of solar and wind energy including photovoltaic modules direct current charge controllers direct current inverters and deep cycle batteries that use or store solar power upon recommendation to the commissioner by the cabinet secretary responsible for matters relating to energy,E
item_code_2023_01141100,0114.11.00,Taxable goods supplied to persons that had an agreement or contract with the government prior to 25th April 2020 and the agreement or contract provided for exemption from value added tax provide that this exemption shall apply to the unexpired period of the contract or agreement upon recommendation by the cabinet secretary responsible for matters relating to energy,E
item_code_2023_01151100,0115.11.00,Medical ventilators and the inputs for the manufacture of medical ventilators upon recommendation by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01161100,0116.11.00,Physiotherapy accessories treadmills for cardiology therapy and treatment of tariff umber 95069100 for use by licensed hospitals upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01171100,0117.11.00,Dexpanthenol of tariff number 33049900 used for medical nappy rash treatment by licensed hospitals upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01181100,0118.11.00,Medicaments of tariff number 30034100 30034200 30034300 heading 3002 3005 or 3006 consisting of two or more constituents which have been mixed together for therapeutic or prophylactic uses,E
item_code_2023_01191100,0119.11.00,Diagnostic or laboratory reagents of tariff number 38220000 on a backing prepared diagnostic or laboratory reagents whether or not on a backing other than those of heading 3002 0r 3006 certified reference materials upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01201100,0120.11.00,Electro diagnostic apparatus of tariff numbers 90181100 90181300 90181400 90181900 90182000 90189000 upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01211100,0121.11.00,Other instruments and appliances of tariff number 90184100 used in dental sciences dental drill engines whether or not combined on a single base with other dental equipment upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01221100,0122.11.00,Other instruments and appliances including surgical blades of tariff number 90184900 90185000 90189000 used in dental sciences dental drill engines whether or not combined on a single base with other dental equipment upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01231100,0123.11.00,Ozone therapy Oxygen therapy aerosol therapy artificial respiration or other therapeutic respiration apparatus upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01241100,0124.11.00,Other breathing appliances and gas masks excluding protective masks having neither mechanical parts nor replaceable filters upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01251100,0125.11.00,Artificial teeth and dental fittings of tariff number 90212100 90212900 and artificial parts of the bidy of tariff number 90213100 90213900 90215000 and 90219000 upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01261100,0126.11.00,Apparatus based on the use of x rays whether or not medical surgical or dental of tariff numbers 90221200 90221300 90221400 1nd 90221900 upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01271100,0127.11.00,Apparatus based on the use of alpha beta or gamma radiations whether or not medical surgical or dental of tariff numbers 90222100 90222900 90223000 and 90229000 upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01281100,0128.11.00,Discs tapes solid state nonvolatile storage devices smart cards and other media for the recording of sound or of other phenomena whether or not recorded of tariff number 85238010 including matrices and masters for the production of dics but excluding products of chapter 37 software upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01291100,0129.11.00,Weighing machines excluding balances of a sensitivity of 5 cg or better of tariff number 84233100 including weight operated counting or checking machines weighing machine weights of all kinds upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01301100,0130.11.00,Foetal Doppler Pocket Wgd002 Pc and pulse oximeter finger held Gima band Pc of tariff number 90181900 upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01311100,0131.11.00,Sterilizer Dry Heat Wgd001Grx05A Pc autoclave steam tables tops of tariff number 84192000 upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01321100,0132.11.00,Needle holders and urine bags of tariff heading 3926,E
item_code_2023_01331100,0133.11.00,Tourniquets of tariff number 39269099 for use by licensed hospitals upon approval by the cabinet secretary responsible for matters relating to health,E
item_code_2023_01341100,0134.11.00,Taxable suppliers including fish feeding and handling water operations cold storage fish cages pons construction and maintenance and fish processing and handling imported or purchased for direct and exclusive use on the recommendation of the relevant state department,E
item_code_2023_01351100,0135.11.00,Pre-fabricated biogas digesters,E
item_code_2023_01361100,0136.11.00,Biogas,E
item_code_2023_01371100,0137.11.00,Sustainable fuel briquettes and pellets for household and commercial use,E
item_code_2023_01381100,0138.11.00,The supply of denatured ethanol of tariff number 22072000,E
item_code_2023_01391100,0139.11.00,Tractors other than road tractors for semitrailers,E
item_code_2023_01401100,0140.11.00,Plant and machinery of chapter 84v and 85 imported by manufactures of pharmaceutical products or investors in the manufacture of pharmaceutical products upon the recommendation of the Cabinet Secretary responsible for matters relating health,E
item_code_2023_01411100,0141.11.00,Medical oxygen supplied to registered hospitals,E
item_code_2023_01421100,0142.11.00,"Urine bags, adult diapers, artificial breasts, colostomy or ileostomy bags for medical use.",E
item_code_2023_01431100,0143.11.00,Inputs and raw materials  used in the manufacturer of passenger motor vehicle,E
item_code_2023_01441100,0144.11.00,Locally manufactured passenger motor vehicles Provided that in this paragraph locally manufactured passenger motor vehicle shall mean a motor vehicle for the transportation of passengers which is manufactured in Kenya and whose total value comprises at least thirty per cent of parts designed and manufactured in Kenya by an original equipment manufacturer operating in Kenya,E
item_code_2023_01451100,0145.11.00,Taxable goods inputs and raw materials imported  or locally purchased by company which  goods are specified in the VAT Act 2013 First Schedule section A no 145,E
item_code_2023_01461100,0146.11.00,Such capital goods the exemption of which the Cabinet Secretary may determine to promote investment in the manufacturing sector provided that the value of such investment is less than two billion shillings,E
item_code_2023_00011301,0001.13.01,Petroleum oils and oils obtained from bituminous minerals crude,B
item_code_2023_00011302,0001.13.02,Motor spirit gasoline regular,B
item_code_2023_00011303,0001.13.03,Motor spirit gasoline premium,B
item_code_2023_00011304,0001.13.04,Aviation spirit,B
item_code_2023_00011305,0001.13.05,Spirit type jet fuel,B
item_code_2023_00011306,0001.13.06,Special boiling point spirit and white spirit,B
item_code_2023_00011307,0001.13.07,Other light oils and preparations,B
item_code_2023_00011308,0001.13.08,Partly refined including topped crudes,B
item_code_2023_00011309,0001.13.09,Kerosene type jet fuel,B
item_code_2023_00011310,0001.13.10,Illuminating kerosene IK,B
item_code_2023_00011311,0001.13.11,Other medium petroleum oils and preparations,B
item_code_2023_00011312,0001.13.12,gas oil automotive light amber for high speed engines,B
item_code_2023_00011313,0001.13.13,Other gas oils,B
item_code_2023_00011314,0001.13.14,Natural gas in gaseous state,B
item_code_2023_00011315,0001.13.15,Other natural gas in gaseous state,B
item_code_2023_00011352,0001.13.52,The liquefied petroleum gas including propane is classified under section 5 of the VAT Act subsection ab,B
item_code_2023_00012101,0001.21.01,the operation of current deposit or savings accounts including the provision of account statements,E
item_code_2023_00012102,0001.21.02,the issue transfer receipt or any other dealing with money including money transfer services and accepting over the counter payments of household bills but excluding the services of carriage of cash restocking of cash machines sorting or counting of money,E
item_code_2023_00012103,0001.21.03,issuing of credit and debit cards,E
item_code_2023_00012104,0001.21.04,automated teller machine transactions excluding the supply of automated teller machines and the software to run it,E
item_code_2023_00012105,0001.21.05,telegraphic money transfer services,E
item_code_2023_00012106,0001.21.06,foreign exchange transactions including the supply of foreign drafts and international money orders,E
item_code_2023_00012107,0001.21.07,cheque handling processing clearing and settlement including special clearance or cancellation of cheques,E
item_code_2023_00012108,0001.21.08,the making of any advances or the granting of any credit,E
item_code_2023_00012109,0001.21.09,issuance of securities for money including bills of exchange promissory notes money and postal orders,E
item_code_2023_00012110,0001.21.10,the provision of guarantees letters of credit and acceptance and other forms of documentary credit,E
item_code_2023_00012111,0001.21.11,the issue transfer receipt or any other dealing with bonds Sukuk debentures treasury bills shares and stocks and other forms of security or secondary security Act No 15 of 2017 s9,E
item_code_2023_00012112,0001.21.12,the assignment of a debt for consideration,E
item_code_2023_00012113,0001.21.13,the provision of the above financial services on behalf of another on a commission basis,E
item_code_2023_00012114,0001.21.14,any services set out in items a to that are structured in conformity with Islamic finance Act No 15 of 2017 s9,E
item_code_2023_00022101,0002.21.01,management and related insurance consultancy services,E
item_code_2023_00022102,0002.21.02,actuarial services,E
item_code_2023_00022103,0002.21.03,services of insurance assessors and loss adjusters,E
item_code_2023_00032101,0003.21.01,education services provided by a preprimary primary or secondary school,E
item_code_2023_00032102,0003.21.02,education services provided by a technical college or university,E
item_code_2023_00032103,0003.21.03,education services provided by an institution established for the promotion of adult education vocational training or technical education but shall not apply in respect of business or user training and other consultancy services designed to improve work practices and efficiency of an organization,E
item_code_2023_00042100,0004.21.00,Medical veterinary dental  Ambulance and nursing services,E
item_code_2023_00052100,0005.21.00,Agricultural animal husbandry and horticultural services,E
item_code_2023_00062100,0006.21.00,Burial and cremation services,E
item_code_2023_00072100,0007.21.00,Transportation of passengers by any means of conveyance excluding international air transport or where the means of conveyance is hired or chartered,E
item_code_2023_00082100,0008.21.00,Supply by way of sale renting leasing hiring letting of land or residential premises residential premises means land or a building occupied or capable of being occupied as a residence,E
item_code_2023_00092100,0009.21.00,Community social and welfare services provided by National Government County Government or any political subdivision thereof,E
item_code_2023_00102100,0010.21.00,Tea and coffee brokerage services,E
item_code_2023_00112101,0011.21.01,services rendered by educational political religious welfare and other philanthropic associations to their members or,E
item_code_2023_00112102,0011.21.02,social welfare services provided by charitable organizations registered or which are E from registration by the Registrar of Societies under section 10 of the Societies Act Cap108 or by the NGO Coordination Board under section 10 of the NGO Coordination Act Cap134 and whose income is exempt from tax under paragraph 10 of the 1st Schedule to the Income Tax Act Cap 470,E
item_code_2023_00122101,0012.21.01,stage plays and performances which are conducted by educational institutions approved by the Cabinet Secretary for the time being responsible for education as part of learning,E
item_code_2023_00122102,0012.21.02,sports games or cultural performances conducted under the auspices of the Ministry for the time being responsible for culture and social services,E
item_code_2023_00132101,0013.21.01,Accommodation and restaurant services provided within establishments operated by an educational training institutions approved by the Cabinet Secretary for the time being responsible for education for the use of the staff and students by that institution or,E
item_code_2023_00132102,0013.21.02,Accommodation and restaurant services provided within establishments operated by a medical institution approved by the Cabinet Secretary for the time being responsible for health for the use by the staff and patients of such institutions or,E
item_code_2023_00132103,0013.21.03,Accommodation and restaurant services provided within canteens and cafeterias operated by an employer for the benefit of his employees,E
item_code_2023_00142100,0014.21.00,Conference services conducted for educational institutions as part of learning where such institutions are approved by the Ministry for the time being responsible for Education,E
item_code_2023_00152100,0015.21.00,Car park services provided by National Government County Government any political subdivision therefore by an employer to his employees on the premises of the employer,E
item_code_2023_00162100,0016.21.00,The supply of airtime by any person other than by a provider of cellular mobile telephone services or wireless telephone services,E
item_code_2023_00172100,0017.21.00,Betting gaming and lotteries services,E
item_code_2023_00182100,0018.21.00,Hiring leasing and chartering of aircrafts,E
item_code_2023_00202100,0020.21.00,Taxable services for direct and exclusive use in the implementation of official aid funded projects upon approval by the Cabinet Secretary to the National Treasury,E
item_code_2023_00212100,0021.21.00,Services imported or procured locally for use by the local film producers or local film agents upon recommendation by the Kenya Film Commission subject to approval by the Cabinet Secretary for the National Treasury,E
item_code_2023_00232100,0023.21.00,Supply of sewerage services by the national government a county government any political subdivision thereof or a person approved by the Cabinet Secretary for the time being responsible for water development,E
item_code_2023_00242100,0024.21.00,Entry fees into the national parks and national reserves,E
item_code_2023_00252100,0025.21.00,The services of tour operators excluding in house supplies,E
item_code_2023_00262100,0026.21.00,Taxable services for direct and exclusive use for the construction of tourism facilities recreational parks of fifty acres or more convention and conference facilities upon the recommendation by the Cabinet Secretary responsible formatters relating to recreational parks,E
item_code_2023_00272100,0027.21.00,Taxable services for direct and exclusive use for the construction of specialized hospitals with accommodation facilities upon recommendation by the Cabinet Secretary responsible for health who shall issue guidelines for the criteria to determine the eligibility for the exemption,E
item_code_2023_00292100,0029.21.00,Postal services provide through the supply of postage stamps including rental post boxes or mail bags and subsidiary services thereto,E
item_code_2023_00332100,0033.21.00,The transfer of assets and other transactions related to the transfer of assets into real estate investment trusts and asset backed securities,E
item_code_2023_00011200,0001.12.00,The exportation of goods,C
item_code_2023_00023200,0002.32.00,The supply of goods or taxable services to an export processing zone business as specified in the Export Processing Zones Act Cap 517 as being eligible for duty and tax free importation,C
item_code_2023_00031200,0003.12.00,Ship stores supplied to international sea or air carriers on international voyage or flight,C
item_code_2023_00041200,0004.12.00,The supply of coffee and tea for export to coffee or tea auction centers,C
item_code_2023_00051200,0005.12.00,Transportation of passengers by air carriers on international flight,C
item_code_2023_00062200,0006.22.00,The supply of taxable services to international sea or air carriers on international voyage or flight,C
item_code_2023_00091200,0009.12.00,Goods purchased from duty free shops by passengers departing to places outside Kenya,C
item_code_2023_00102200,0010.22.00,Supply of taxable services in respect of goods in transit,C
item_code_2023_00111200,0011.12.00,Inputs or raw materials either produced locally or imported supplied to pharmaceutical manufacturers in Kenya for manufacturing medicaments as approved from time to time by the Cabinet Secretary in consultation with the Cabinet Secretary responsible for matters relating to health,C
item_code_2023_00123200,0012.32.00,The supply of goods or taxable services to a special economic zone enterprise,C
item_code_2023_00131202,0013.12.02,The supply of ordinary bread,C
item_code_2023_00151200,0015.12.00,Milk and cream not concentrated nor containing added sugar or other sweetening matter of tariff numbers as listed on the second schedule of the VAT Act 2013,C
item_code_2023_00161200,0016.12.00,All inputs and raw materials whether produced locally or imported supplied to manufacturers of agricultural pest control products upon recommendation by the Cabinet Secretary for the time being responsible for agriculture,C
item_code_2023_00191200,0019.12.00,Agricultural pest control products,C
item_code_2023_00202200,0020.22.00,The transportation of goods originating from Kenya to a place outside Kenya,C
item_code_2023_00212200,0021.22.00,Transportation of sugarcane from farms to milling factories,C
item_code_2023_00221200,0022.12.00,The supply of maize corn flour cassava flour wheat or meslin flour and maize flour containing cassava flour by more than ten percent in weight,C
item_code_2023_00232200,0023.22.00,The exportation of taxable services in respect of business process outsourcing,C
item_code_2023_00241200,0024.12.00,The Fertilizers of chapter 21,C
item_code_2023_00251200,0025.12.00,Inputs of raw materials locally purchased or imported by manufacturers of fertilizer as approved from time to time by the Cabinet Secretary responsible for Agriculture,C
item_code_2023_00011201,0001.12.01,Supply to Commonwealth and other Governments Goods consigned to officers or men on board a naval vessel belonging to another Commonwealth Government for their personal use or for consumption on board such vessel,C
item_code_2023_00011202,0001.12.02,Supply to Commonwealth and other Governments Goods for the use of any of the Armed Forces of any allied power,C
item_code_2023_00021201,0002.12.01,Supply to diplomat or First arrivals persons Household and personal effects of any kind imported by entitled personnel or their dependents including one motor vehicle imported or supplied to them prior to clearance through customs within ninety days of their first arrival in Kenya or such longer period not exceeding three hundred and sixty days from the date of his arrival as may be approved by the Commissioner of Customs in specific cases where the entitled personnel have not been granted zero rating status in any other section of this Schedule,C
item_code_2023_00021202,0002.12.02,Supply to diplomat or First arrivals persons One motor vehicle which the ministry responsible for foreign affairs is satisfied as having been supplied or imported as a replacement for a motor vehicle originally imported or supplied under paragraph 1 which has been written off due to accident fire or theft Provided that tax shall be payable at the appropriate rate if the written off motor vehicle is disposed of locally,C
item_code_2023_00023203,0002.32.03,Supply to diplomat or First arrivals persons taxable supplies for the official use of the United Nations or its specialized agencies or any Commonwealth High Commission or of any foreign embassy consulate or diplomatic mission in Kenya,C
item_code_2023_00023204,0002.32.04,Supply to diplomat or First arrivals persons taxable supplies for the use of a high official of the United Nations or its specialized agencies or a member of the diplomatic staff of any Commonwealth or foreign country where specific provision for such zero rating status is made by the Cabinet Secretary responsible for foreign affairs,C
item_code_2023_00021205,0002.12.05,Supply to diplomat or First arrivals persons taxable supplies Goods for the United Nations or any of its specialized agencies for the support of a project in Kenya,C
item_code_2023_00031201,0003.12.01,Supply to donor agencies with bilateral or multilateral agreements Household and personal effects of any kind imported by entitled personnel or their dependents including one motor vehicle imported or supplied to them prior to clearance through customs within ninety days of their first arrival in Kenya or such longer period not exceeding three hundred and sixty days from the date of his arrival as may be approved by the Commissioner of Customs in specific cases where the entitled personnel have not been granted zero rating status in any other section of this Schedule,C
item_code_2023_00031202,0003.12.02,Supply to donor agencies with bilateral or multilateral agreements One motor vehicle which the Commissioner is satisfied is supplied or is imported as a replacement of another motor vehicle originally supplied or imported under paragraph 1 and which has been written off due to accident fire or theft,C
item_code_2023_00043200,0004.32.00,Supply to international and regional organizations Goods services and equipment imported by or supplied to donor agencies international and regional organizations with Diplomatic accreditation or bilateral or multilateral agreements with Kenya for their official use,C
item_code_2023_00053200,0005.32.00,Supply to the War Graves Commission taxable supplies including official vehicles for the establishment and maintenance of war cemeteries by the Commonwealth War Graves Commission but excluding office supplies and equipment and the property of the Commission’s staff,C
item_code_2023_00083200,0008.32.00,Supply to National Red Cross Society and St John Ambulance taxable goods and services supplied or imported for official use in the provision of relief service,C

```

## File: data\template\account.account-ke.csv

```csv
"id","name","code","account_type","tag_ids","reconcile"
"ke0010","Software","0010","asset_non_current","","False"
"ke0020","Patents & Trademarks","0020","asset_non_current","","False"
"ke0030","Fixtures and fittings","0030","asset_fixed","","False"
"ke0040","Land and buildings","0040","asset_fixed","","False"
"ke0050","Motor vehicles","0050","asset_fixed","","False"
"ke0060","Office equipment (inc computer equipment)","0060","asset_fixed","","False"
"ke0070","Plant and machinery","0070","asset_fixed","","False"
"ke0080","Financial assets","0080","asset_non_current","","False"
"ke0090","Biological assets","0090","asset_non_current","","False"
"ke1001","Stock","1001","asset_current","","True"
"ke100110","Stock Interim (Received)","100110","asset_current","","True"
"ke100120","Stock Interim (Delivered)","100120","asset_current","","True"
"ke1002","Work in Progress","1002","asset_current","","False"
"ke1003","Finished Goods","1003","asset_current","","False"
"ke1100","Debtors Control Account","1100","asset_receivable","","True"
"ke110010","Debtors Control Account (POS)","110010","asset_receivable","","True"
"ke1101","Sundry Debtors","1101","asset_receivable","","True"
"ke1102","Other Debtors","1102","asset_current","","False"
"ke1103","Prepayments","1103","asset_prepayments","","False"
"ke1110","Purchase Tax Control Account","1110","asset_current","","False"
"ke1111","KRA - VAT receivable","1111","asset_receivable","","True"
"ke1120","Withholding Tax Advance on Sales","1120","asset_current","","True"
"ke112001","Withholding Tax Advance on Sales - Transition account","112001","asset_current","","True"
"ke120011","Deferred Expenses (Automatics)","120011","asset_current","","False"
"ke2100","Creditors Control Account","2100","liability_payable","","True"
"ke2101","Sundry Creditors","2101","liability_current","","False"
"ke2102","Other Creditors","2102","liability_current","","False"
"ke2103","Accruals","2103","liability_current","","False"
"ke2104","Company Credit Card","2104","liability_credit_card","","False"
"ke2105","Bad debt provision","2105","liability_current","","False"
"ke211000","Deferred Revenue","211000","liability_current","","False"
"ke2200","Sales Tax Control Account","2200","liability_current","","False"
"ke2201","KRA - VAT payable","2201","liability_payable","","True"
"ke2202","Manual Adjustments & VAT","2202","liability_current","","False"
"ke2203","Withholding Tax Payable","2203","liability_current","","True"
"ke220301","Withholding Tax Payable - Transition account","220301","liability_current","","True"
"ke2210","P.A.Y.E. & NI","2210","liability_payable","","True"
"ke2220","Net Wages","2220","liability_payable","","True"
"ke2230","Pension Fund","2230","liability_payable","","True"
"ke2240","Corporation Tax","2240","liability_payable","","True"
"ke2300","Loans","2300","liability_non_current","","False"
"ke2310","Hire Purchase","2310","liability_non_current","","False"
"ke2320","Mortgages","2320","liability_non_current","","False"
"ke3000","Called up share capital","3000","equity","","False"
"ke3010","Share premium account","3010","equity","","False"
"ke3020","Revaluation reserve","3020","equity","","False"
"ke3030","Other reserves","3030","equity","","False"
"ke3040","Capital","3040","equity","","False"
"ke4001","Sales category 1","4001","income","","False"
"ke4002","Sales category 2","4002","income","","False"
"ke4003","Sales category 3","4003","income","","False"
"ke4004","Sales category 4","4004","income","","False"
"ke4005","Bank Interest received","4005","income","","False"
"ke4006","Investment Interest received","4006","income","","False"
"ke4007","Revenue Income","4007","income","","False"
"ke400710","Cash Discount Gain","400710","income_other","","False"
"ke4008","Profits/Losses on disposals of assets","4008","income","","False"
"ke4010","Other Income","4010","income_other","","False"
"ke5001","Cost of sales 1","5001","expense_direct_cost","","False"
"ke5002","Cost of sales 2","5002","expense_direct_cost","","False"
"ke5003","Cost of sales 3","5003","expense_direct_cost","","False"
"ke5004","Cost of sales 4","5004","expense_direct_cost","","False"
"ke5101","Marketing","5101","expense","","False"
"ke5102","Exhibitions and events","5102","expense","","False"
"ke5103","PR","5103","expense","","False"
"ke5104","Distribution vehicles","5104","expense","","False"
"ke5105","Distribution salaries and wages","5105","expense","","False"
"ke5106","Shipping","5106","expense","","False"
"ke5107","Directors pension","5107","expense","","False"
"ke5108","Directors remuneration","5108","expense","","False"
"ke5109","Gross Salaries","5109","expense","","False"
"ke5110","Employers SDL & UIF","5110","expense","","False"
"ke5111","Subcontractors payments","5111","expense","","False"
"ke5112","Rent and rates","5112","expense","","False"
"ke5113","Light / heat and power","5113","expense","","False"
"ke5114","Repairs and maintenance","5114","expense","","False"
"ke5115","Car hire","5115","expense","","False"
"ke5116","Car fuel","5116","expense","","False"
"ke5117","Car maintenance","5117","expense","","False"
"ke5118","Telephone","5118","expense","","False"
"ke5119","Internet & hosting","5119","expense","","False"
"ke5120","Mobiles","5120","expense","","False"
"ke5121","Stationery","5121","expense","","False"
"ke5122","Office consumables","5122","expense","","False"
"ke5123","Postage and Carriage","5123","expense","","False"
"ke5124","Books","5124","expense","","False"
"ke5125","Network costs","5125","expense","","False"
"ke5126","Software expenses","5126","expense","","False"
"ke5127","Other computer costs","5127","expense","","False"
"ke5128","Recruitment fees","5128","expense","","False"
"ke5129","Other admin expenses","5129","expense","","False"
"ke5130","Accounting","5130","expense","","False"
"ke5131","Auditing","5131","expense","","False"
"ke5132","Consultancy","5132","expense","","False"
"ke5133","Legal and professional charges","5133","expense","","False"
"ke5134","Exchange gains/losses","5134","expense","","False"
"ke5135","Other sundry expenses","5135","expense","","False"
"ke5136","Bad debts","5136","expense","","False"
"ke5137","Interest paid","5137","expense","","False"
"ke5138","Bank Charges","5138","expense","","False"
"ke5139","Donations","5139","expense","","False"
"ke5140","Entertaining","5140","expense","","False"
"ke5141","Insurance","5141","expense","","False"
"ke5142","Travel and subsistence","5142","expense","","False"
"ke5143","Corporation tax expense","5143","expense","","False"
"ke5144","Foreign Exchange Gains/Losses","5144","expense","","False"
"ke5145","Price Differences Control Account","5145","expense","","False"
"ke5146","Cash Register Gains/Losses","5146","expense","","False"
"ke5147","Cash Discount Loss","5147","expense","","False"
"ke5201","Software Depreciation","5201","expense_depreciation","","False"
"ke5202","Patents & Trademarks Depreciation","5202","expense_depreciation","","False"
"ke5203","Fixtures and fittings Depreciation","5203","expense_depreciation","","False"
"ke5204","Land and buildings Depreciation","5204","expense_depreciation","","False"
"ke5205","Motor vehicles Depreciation","5205","expense_depreciation","","False"
"ke5206","Office equipment (inc computer equipment) Depreciation","5206","expense_depreciation","","False"
"ke5207","Plant and machinery Depreciation","5207","expense_depreciation","","False"

```

## File: data\template\account.fiscal.position-ke.csv

```csv
"id","sequence","name","auto_apply","country_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_template_national","1","National","1","base.ke","",""
"fiscal_position_template_non_kenyan","2","International","1","","ST16","ST0EXPORT"
"","","","","","ST8","ST0EXPORT"
"","","","","","PT16","PT0"
"","","","","","PT8","PT0"

```

## File: data\template\account.tax-ke.csv

```csv
"id","description","invoice_label","type_tax_use","name","amount_type","amount","tax_group_id","active","l10n_ke_item_code_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/factor_percent","repartition_line_ids/account_id","price_include_override","tax_exigibility","cash_basis_transition_account_id","tax_scope"
"ST16","Sales VAT (16%)","16%","sale","16%","percent","16.0","tax_group_16","1","","base","invoice","+16% Sales Base","","","","","",""
"","","","","","","","","","","tax","invoice","+16% Sales Tax","100","ke2200","","","",""
"","","","","","","","","","","base","refund","-16% Sales Base","","","","","",""
"","","","","","","","","","","tax","refund","-16% Sales Tax","100","ke2200","","","",""
"ST8","Sales VAT (8%)","8%","sale","8%","percent","8.0","tax_group_8","1","","base","invoice","+8% Sales Base","","","","","",""
"","","","","","","","","","","tax","invoice","+8% Sales Tax","100","ke2200","","","",""
"","","","","","","","","","","base","refund","-8% Sales Base","","","","","",""
"","","","","","","","","","","tax","refund","-8% Sales Tax","100","ke2200","","","",""
"ST0","Sales VAT Zero Rated","0%","sale","0%","percent","0.0","tax_group_0","1","","base","invoice","+Zero Rated Sales Base","","","","","",""
"","","","","","","","","","","tax","invoice","","100","","","","",""
"","","","","","","","","","","base","refund","-Zero Rated Sales Base","","","","","",""
"","","","","","","","","","","tax","refund","","100","","","","",""
"STEX","Sales VAT Exempt","Exempt","sale","0% EXEMPT","percent","0.0","tax_group_0","1","","base","invoice","+Exempt Sales Base","","","","","",""
"","","","","","","","","","","tax","invoice","","100","","","","",""
"","","","","","","","","","","base","refund","-Exempt Sales Base","","","","","",""
"","","","","","","","","","","tax","refund","","100","","","","",""
"SWT2","2% WH based on payment","2%","sale","2% WH","percent","-2.0","tax_group_withholding","1","","base","invoice","","","","","on_payment","ke112001",""
"","","","","","","","","","","tax","invoice","-WH Sales","100","ke1120","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","+WH Sales","100","ke1120","","","",""
"SWT3","3% WH","3%","sale","3% WH","percent","-3.0","tax_group_withholding","0","","base","invoice","","","","tax_excluded","","",""
"","","","","","","","","","","tax","invoice","","","ke1120","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke1120","","","",""
"SWT5","5% WH","5%","sale","5% WH","percent","-5.0","tax_group_withholding","0","","base","invoice","","","","tax_excluded","","",""
"","","","","","","","","","","tax","invoice","","","ke1120","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke1120","","","",""
"SWT10","10% WH","10%","sale","10% WH","percent","-10.0","tax_group_withholding","0","","base","invoice","","","","tax_excluded","","",""
"","","","","","","","","","","tax","invoice","","","ke1120","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke1120","","","",""
"SWT12","12% WH","12%","sale","12% WH","percent","-12.0","tax_group_withholding","0","","base","invoice","","","","tax_excluded","","",""
"","","","","","","","","","","tax","invoice","","","ke1120","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke1120","","","",""
"SWT15","15% WH","15%","sale","15% WH","percent","-15.0","tax_group_withholding","0","","base","invoice","","","","tax_excluded","","",""
"","","","","","","","","","","tax","invoice","","","ke1120","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke1120","","","",""
"SWT20","20% WH","20%","sale","20% WH","percent","-20.0","tax_group_withholding","0","","base","invoice","","","","tax_excluded","","",""
"","","","","","","","","","","tax","invoice","","","ke1120","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke1120","","","",""
"SWT25","25% WH","25%","sale","25% WH","percent","-25.0","tax_group_withholding","0","","base","invoice","","","","tax_excluded","","",""
"","","","","","","","","","","tax","invoice","","","ke1120","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke1120","","","",""
"SWT30","30% WH","30%","sale","30% WH","percent","-30.0","tax_group_withholding","0","","base","invoice","","","","tax_excluded","","",""
"","","","","","","","","","","tax","invoice","","","ke1120","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke1120","","","",""
"PT16","Purchases VAT (16%)","16%","purchase","16%","percent","16.0","tax_group_16","1","","base","invoice","+16% Purchases Base","","","","","",""
"","","","","","","","","","","tax","invoice","+16% Purchases Tax","100","ke1110","","","",""
"","","","","","","","","","","base","refund","-16% Purchases Base","","","","","",""
"","","","","","","","","","","tax","refund","-16% Purchases Tax","100","ke1110","","","",""
"PT8","Purchases VAT (8%)","8%","purchase","8%","percent","8.0","tax_group_8","1","","base","invoice","+8% purchases Base","","","","","",""
"","","","","","","","","","","tax","invoice","+8% Purchases Tax","100","ke1110","","","",""
"","","","","","","","","","","base","refund","-8% purchases Base","","","","","",""
"","","","","","","","","","","tax","refund","-8% Purchases Tax","100","ke1110","","","",""
"PT0","Purchases VAT Zero rated (0%)","0%","purchase","0%","percent","0.0","tax_group_0","1","","base","invoice","+Zero Rated Purchases Base","","","","","",""
"","","","","","","","","","","tax","invoice","","100","","","","",""
"","","","","","","","","","","base","refund","-Zero Rated Purchases Base","","","","","",""
"","","","","","","","","","","tax","refund","","100","","","","",""
"PTEX","Purchase VAT Exempt","Exempt","purchase","0% EXEMPT","percent","0.0","tax_group_0","1","","base","invoice","+Exempt Purchases Base","","","","","",""
"","","","","","","","","","","tax","invoice","","100","","","","",""
"","","","","","","","","","","base","refund","-Exempt Purchases Base","","","","","",""
"","","","","","","","","","","tax","refund","","100","","","","",""
"PTIM16S","Import VAT (16%)","16%","purchase","16% IMPORT","percent","16.0","tax_group_16","1","","base","invoice","+Import Base","","","","","","service"
"","","","","","","","","","","tax","invoice","+Import Tax","100","","","","",""
"","","","","","","","","","","base","refund","-Import Base","","","","","",""
"","","","","","","","","","","tax","refund","-Import Tax","100","","","","",""
"PWT2","2% WH based on payment","2%","purchase","2% WH","percent","-2.0","tax_group_withholding","1","","base","invoice","+WH base","","","","on_payment","ke220301",""
"","","","","","","","","","","tax","invoice","+WH 2%","100","ke2203","","","",""
"","","","","","","","","","","base","refund","-WH base","","","","","",""
"","","","","","","","","","","tax","refund","-WH 2%","100","ke2203","","","",""
"PWT3","3% WH","3%","purchase","3% WH","percent","-3.0","tax_group_withholding","0","","base","invoice","","","","","","",""
"","","","","","","","","","","tax","invoice","","","ke2203","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke2203","","","",""
"PWT5","5% WH","5%","purchase","5% WH","percent","-5.0","tax_group_withholding","0","","base","invoice","","","","","","",""
"","","","","","","","","","","tax","invoice","","","ke2203","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke2203","","","",""
"PWT10","10% WH","10%","purchase","10% WH","percent","-10.0","tax_group_withholding","0","","base","invoice","","","","","","",""
"","","","","","","","","","","tax","invoice","","","ke2203","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke2203","","","",""
"PWT12","12% WH","12%","purchase","12% WH","percent","-12.0","tax_group_withholding","0","","base","invoice","","","","","","",""
"","","","","","","","","","","tax","invoice","","","ke2203","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke2203","","","",""
"PWT15","15% WH","15%","purchase","15% WH","percent","-15.0","tax_group_withholding","0","","base","invoice","","","","","","",""
"","","","","","","","","","","tax","invoice","","","ke2203","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke2203","","","",""
"PWT20","20% WH","20%","purchase","20% WH","percent","-20.0","tax_group_withholding","0","","base","invoice","","","","","","",""
"","","","","","","","","","","tax","invoice","","","ke2203","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke2203","","","",""
"PWT25","25% WH","25%","purchase","25% WH","percent","-25.0","tax_group_withholding","0","","base","invoice","","","","","","",""
"","","","","","","","","","","tax","invoice","","","ke2203","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke2203","","","",""
"PWT30","30% WH","30%","purchase","30% WH","percent","-30.0","tax_group_withholding","0","","base","invoice","","","","","","",""
"","","","","","","","","","","tax","invoice","","","ke2203","","","",""
"","","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","","tax","refund","","","ke2203","","","",""
"ST0EXPORT","Export (0%)","0%","sale","0% EXPORT","percent","0","tax_group_0","1","l10n_ke.item_code_2023_00011200","base","invoice","+Exempt Sales Base","","","","","",""
"","","","","","","","","","","tax","invoice","","100","","","","",""
"","","","","","","","","","","base","refund","-Exempt Sales Base","","","","","",""
"","","","","","","","","","","tax","refund","","100","","","","",""

```

## File: data\template\account.tax.group-ke.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id"
"tax_group_16","VAT 16%","base.ke","ke1111","ke2201"
"tax_group_8","VAT 8%","base.ke","ke1111","ke2201"
"tax_group_0","VAT 0%","base.ke","ke1111","ke2201"
"tax_group_withholding","Withholding Tax","base.ke","ke1111","ke2201"

```

## File: models\account_move.py

```python
from odoo import models, fields


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_ke_wh_certificate_number = fields.Char(
        string="Withholding Certificate Number",
        help="Customer withholding certificate number",
    )

    l10n_ke_wh_certificate_date = fields.Date(
        string="Date of Certificate",
    )

```

## File: models\account_tax.py

```python
# -*- coding: utf-8 -*-
from odoo import models, fields, api


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_ke_item_code_id = fields.Many2one(
        'l10n_ke.item.code',
        string='KRA Item Code',
        help='KRA code that describes a tax rate or exemption on specific products or services.',
    )

    @api.onchange('amount')
    def _onchange_l10n_ke_item_code_id(self):
        """ When the amount of the tax changes this field is reset """
        for tax in self:
            if tax._origin.amount != tax.amount:
                tax.l10n_ke_item_code_id = None

```

## File: models\l10n_ke_item_code.py

```python

# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, api


class L10nKeItemCode(models.Model):
    _name = 'l10n_ke.item.code'
    _description = "KRA defined codes that justify a given tax rate / exemption"
    _rec_names_search = ['code', 'description']

    code = fields.Char(string='KRA Item Code')
    description = fields.Char(string='Description')
    tax_rate = fields.Selection([('C', 'Zero Rated'), ('E', 'Exempted'), ('B', 'Taxable at 8%')])

    @api.depends('code', 'description')
    def _compute_display_name(self):
        for item_code in self:
            item_code.display_name = f'{item_code.code} {item_code.description}'

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.


from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_ke_oscu_is_active = fields.Boolean(
        string="Is OSCU active?",
        help="Whether this company is set up for OSCU flows.",
        compute='_compute_l10n_ke_oscu_is_active',
    )

    def _compute_l10n_ke_oscu_is_active(self):
        """ Overridden in enterprise when the OSCU module is used in the company"""
        self.l10n_ke_oscu_is_active = False

```

## File: models\template_ke.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ke')
    def _get_ke_template_data(self):
        return {
            'property_account_receivable_id': 'ke1100',
            'property_account_payable_id': 'ke2100',
            'property_account_expense_categ_id': 'ke5001',
            'property_account_income_categ_id': 'ke4001',
            'property_stock_valuation_account_id': 'ke1001',
            'property_stock_account_output_categ_id': 'ke100120',
            'property_stock_account_input_categ_id': 'ke100110',
            'code_digits': '6',
        }

    @template('ke', 'res.company')
    def _get_ke_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.ke',
                'bank_account_code_prefix': '12000',
                'cash_account_code_prefix': '12500',
                'transfer_account_code_prefix': '12100',
                'account_default_pos_receivable_account_id': 'ke110010',
                'income_currency_exchange_account_id': 'ke5144',
                'expense_currency_exchange_account_id': 'ke5144',
                'account_journal_early_pay_discount_loss_account_id': 'ke5147',
                'account_journal_early_pay_discount_gain_account_id': 'ke400710',
                'default_cash_difference_income_account_id': 'ke5146',
                'default_cash_difference_expense_account_id': 'ke5146',
                'account_sale_tax_id': 'ST16',
                'account_purchase_tax_id': 'PT16',
                'tax_exigibility': 'True',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ke
from . import l10n_ke_item_code
from . import account_tax
from . import account_move
from . import res_company

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
l10n_ke_item_code_readonly,l10n_ke.item.code,model_l10n_ke_item_code,account.group_account_readonly,1,0,0,0
l10n_ke_item_code_invoice,l10n_ke.item.code,model_l10n_ke_item_code,account.group_account_invoice,1,0,0,0

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>
    <record id="l10n_ke.view_move_form" model="ir.ui.view">
        <field name="name">account.move.form</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//group[@name='sale_info_group']/field[@name='delivery_date']" position="after">
                <div invisible="country_code != 'KE' or move_type != 'out_invoice' or payment_state != 'paid'">
                    <field name="l10n_ke_wh_certificate_number"/>
                    <field name="l10n_ke_wh_certificate_date"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_ke_inherit_view_tax_tree" model="ir.ui.view">
        <field name="name">l10n.ke.account.tax.list</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_tree" />
        <field name="arch" type="xml">
            <field name="description" position="after">
                <field name="l10n_ke_item_code_id" optional="hide"/>
            </field>
        </field>
    </record>

    <record id="l10n_ke_inherit_view_tax_form" model="ir.ui.view">
        <field name="name">l10n.ke.inherit.account.tax.form</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='country_id']" position="after">
                <field name="l10n_ke_item_code_id"
                       string="KRA Item Code"
                       domain="[] if amount_type != 'percent' else [('tax_rate', 'in', ['C','E'] if amount == 0 else ['B'] if amount == 8 else [False])]"
                       invisible="country_code != 'KE'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\l10n_ke_item_code_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_l10n_ke_item_code_tree" model="ir.ui.view">
            <field name="name">l10n_ke.item.code.list</field>
            <field name="model">l10n_ke.item.code</field>
            <field name="arch" type="xml">
                <list string="KRA Item Code">
                    <field name="code"/>
                    <field name="description"/>
                    <field name="tax_rate"/>
                </list>
            </field>
        </record>
        <record id="view_l10n_ke_item_code_search" model="ir.ui.view">
            <field name="name">l10n_ke.item.code.search</field>
            <field name="model">l10n_ke.item.code</field>
            <field name="arch" type="xml">
                <search>
                    <field name="code"/>
                    <field name="description"/>
                    <filter string="Exempted" domain="[('tax_rate','=','E')]" name="type_exempted"/>
                    <filter string="Zero Rated" domain="[('tax_rate','=','C')]" name="type_zero_rated"/>
                    <filter string="Taxable at 8%" domain="[('tax_rate','=','B')]" name="type_eight_rated"/>
                    <filter name="group_by_tax_rate" string="By Tax Rate" context="{'group_by': 'tax_rate'}"/>
                </search>
            </field>
        </record>
    </data>
</odoo>

```

