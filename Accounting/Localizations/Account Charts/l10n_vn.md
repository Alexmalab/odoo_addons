# Odoo Module: l10n_vn

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
    'name': 'Vietnam - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['vn'],
    'version': '2.0.3',
    'author': 'General Solutions',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/vietnam.html',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the module to manage the accounting chart, bank information for Vietnam in Odoo.
========================================================================================

- This module applies to companies based in Vietnamese Accounting Standard (VAS)
  with Chart of account under Circular No. 200/2014/TT-BTC
- Add Vietnamese bank information (like name, bic ..) as announced and yearly updated by State Bank
  of Viet Nam (https://sbv.gov.vn/webcenter/portal/en/home/sbv/paytreasury/bankidno).
- Add VietQR feature for invoice

**Credits:**
    - General Solutions.
    - Trobz
    - Jean Nguyen - The Bean Family (https://github.com/anhjean/vietqr) for VietQR.

""",
    'depends': [
        'account_qr_code_emv',
        'base_iban',
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_tax_report_data.xml',
        'views/account_move_views.xml',
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
        <field name="country_id" ref="base.vn"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_amount_untaxed" model="account.report.column">
                <field name="name">Base Amount</field>
                <field name="expression_label">amount_untaxed</field>
            </record>
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_01_vn" model="account.report.line">
                <field name="name">Purchase of Goods and Services</field>
                <field name="aggregation_formula">VAT_PURCHASE.balance + VAT_PURCHASE_IMPORTED.balance + VAT_PURCHASE_IMPORT_TAX.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_01_vn_tag_base" model="account.report.expression">
                        <field name="label">amount_untaxed</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">VAT_PURCHASE.amount_untaxed + VAT_PURCHASE_IMPORTED.amount_untaxed + VAT_PURCHASE_IMPORT_TAX.amount_untaxed</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_01_01_vn" model="account.report.line">
                        <field name="name">VAT on purchase of goods and services</field>
                        <field name="code">VAT_PURCHASE</field>
                        <field name="aggregation_formula">VAT_PURCHASE_0.balance + VAT_PURCHASE_5.balance + VAT_PURCHASE_8.balance + VAT_PURCHASE_10.balance</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_01_01_vn_tag_base" model="account.report.expression">
                                <field name="label">amount_untaxed</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VAT_PURCHASE_0.amount_untaxed + VAT_PURCHASE_5.amount_untaxed + VAT_PURCHASE_8.amount_untaxed + VAT_PURCHASE_10.amount_untaxed + VAT_PURCHASE_EXEMPTION.amount_untaxed</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services 0%</field>
                                <field name="code">VAT_PURCHASE_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_01_01_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed 0%</field>
                                    </record>
                                    <record id="account_tax_report_line_01_01_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on purchase of goods and services 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services 5%</field>
                                <field name="code">VAT_PURCHASE_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_01_01_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed 5%</field>
                                    </record>
                                    <record id="account_tax_report_line_02_01_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on purchase of goods and services 5%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_04_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services 8%</field>
                                <field name="code">VAT_PURCHASE_8</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_04_01_01_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed 8%</field>
                                    </record>
                                    <record id="account_tax_report_line_04_01_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on purchase of goods and services 8%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services 10%</field>
                                <field name="code">VAT_PURCHASE_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_01_01_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed 10%</field>
                                    </record>
                                    <record id="account_tax_report_line_03_01_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on purchase of goods and services 10%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_05_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services Exemption</field>
                                <field name="code">VAT_PURCHASE_EXEMPTION</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_05_01_01_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed VAT Exemption</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_02_01_vn" model="account.report.line">
                        <field name="name">VAT on purchase of imported goods</field>
                        <field name="code">VAT_PURCHASE_IMPORTED</field>
                        <field name="aggregation_formula">VAT_PURCHASE_IMPORTED_0.balance + VAT_PURCHASE_IMPORTED_5.balance + VAT_PURCHASE_IMPORTED_8.balance + VAT_PURCHASE_IMPORTED_10.balance</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_02_01_vn_tag_base" model="account.report.expression">
                                <field name="label">amount_untaxed</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VAT_PURCHASE_IMPORTED_0.amount_untaxed + VAT_PURCHASE_IMPORTED_5.amount_untaxed + VAT_PURCHASE_IMPORTED_8.amount_untaxed + VAT_PURCHASE_IMPORTED_10.amount_untaxed</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_02_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of imported goods 0%</field>
                                <field name="code">VAT_PURCHASE_IMPORTED_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_02_01_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_purchase_import_0_base</field>
                                    </record>
                                    <record id="account_tax_report_line_01_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_purchase_import_0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_02_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of imported goods 5%</field>
                                <field name="code">VAT_PURCHASE_IMPORTED_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_02_01_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_purchase_import_5_base</field>
                                    </record>
                                    <record id="account_tax_report_line_02_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_purchase_import_5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_04_02_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of imported goods 8%</field>
                                <field name="code">VAT_PURCHASE_IMPORTED_8</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_04_02_01_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_purchase_import_8_base</field>
                                    </record>
                                    <record id="account_tax_report_line_04_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_purchase_import_8</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_02_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of imported goods 10%</field>
                                <field name="code">VAT_PURCHASE_IMPORTED_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_02_01_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_purchase_import_10_base</field>
                                    </record>
                                    <record id="account_tax_report_line_03_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">tax_purchase_import_10</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_03_01_vn" model="account.report.line">
                        <field name="name">Import Tax</field>
                        <field name="code">VAT_PURCHASE_IMPORT_TAX</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_03_01_vn_tag_base" model="account.report.expression">
                                <field name="label">amount_untaxed</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_import_base</field>
                            </record>
                            <record id="account_tax_report_line_03_01_vn_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">tax_import</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_02_vn" model="account.report.line">
                <field name="name">Sales of Goods and Services</field>
                <field name="aggregation_formula">VAT_SALES.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_02_vn_tag_base" model="account.report.expression">
                        <field name="label">amount_untaxed</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">VAT_SALES.amount_untaxed</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_01_02_vn" model="account.report.line">
                        <field name="name">VAT on sales of goods and services</field>
                        <field name="code">VAT_SALES</field>
                        <field name="aggregation_formula">VAT_SALES_0.balance + VAT_SALES_5.balance + VAT_SALES_8.balance + VAT_SALES_10.balance</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_01_02_vn_tag_base" model="account.report.expression">
                                <field name="label">amount_untaxed</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VAT_SALES_0.amount_untaxed + VAT_SALES_5.amount_untaxed + VAT_SALES_10.amount_untaxed + VAT_SALES_EXEMPTION.amount_untaxed</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services 0%</field>
                                <field name="code">VAT_SALES_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_01_02_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed 0%</field>
                                    </record>
                                    <record id="account_tax_report_line_01_01_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on sales of goods and services 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services 5%</field>
                                <field name="code">VAT_SALES_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_01_02_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed 5%</field>
                                    </record>
                                    <record id="account_tax_report_line_02_01_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on sales of goods and services 5%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_04_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services 8%</field>
                                <field name="code">VAT_SALES_8</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_04_01_02_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed 8%</field>
                                    </record>
                                    <record id="account_tax_report_line_04_01_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on sales of goods and services 8%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services 10%</field>
                                <field name="code">VAT_SALES_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_01_02_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed 10%</field>
                                    </record>
                                    <record id="account_tax_report_line_03_01_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on sales of goods and services 10%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_05_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services Exemption</field>
                                <field name="code">VAT_SALES_EXEMPTION</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_05_01_02_vn_tag_base" model="account.report.expression">
                                        <field name="label">amount_untaxed</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed VAT Exemption</field>
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

## File: data\template\account.account-vn.csv

```csv
"id","name","code","account_type","reconcile","name@vi_VN"
"chart1111","Vietnamese dong on hand","1111","asset_cash","False","Tiền Việt Nam - Tiền mặt"
"chart1112","Foreign currencies on hand","1112","asset_cash","False","Ngoại tệ - Tiền mặt"
"chart1113","Monetary gold on hand","1113","asset_cash","False","Vàng tiền tệ - Tiền mặt"
"chart1121","Vietnamese dong in bank","1121","asset_cash","False","Tiền Việt Nam - Tiền gửi ngân hàng"
"chart1122","Foreign currencies in bank","1122","asset_cash","False","Ngoại tệ - Tiền gửi ngân hàng"
"chart1123","Monetary gold in bank","1123","asset_cash","False","Vàng tiền tệ - Tiền gửi ngân hàng"
"chart1131","Vietnamese dong in transit","1131","asset_cash","False","Tiền Việt Nam - Tiền đang chuyển"
"chart1132","Foreign currencies in transit","1132","asset_cash","False","Ngoại tệ - Tiền đang chuyển"
"chart1211","Shares","1211","asset_current","False","Cổ phiếu"
"chart1212","Bonds","1212","asset_current","False","Trái phiếu"
"chart1218","Securities and other financial instruments","1218","asset_current","False","Chứng khoán và công cụ tài chính khác"
"chart1281","Time deposits","1281","asset_current","False","Tiền gửi có kỳ hạn"
"chart1282","Bonds","1282","asset_current","False","Trái phiếu"
"chart1283","Loans","1283","asset_current","False","Cho vay"
"chart1288","Other investments held to maturity","1288","asset_current","False","Các khoản đầu tư khác nắm giữ đến ngày đáo hạn"
"chart131","Trade receivables","131","asset_receivable","True","Phải thu của khách hàng"
"chart132","Trade receivables(pos)","132","asset_receivable","True","Phải thu của khách hàng(pos)"
"chart1331","Deductible VAT of goods and services","1331","asset_current","False","Thuế GTGT HHDV mua vào"
"chart1332","Deductible VAT of fixed assets","1332","asset_current","False","Thuế GTGT được khấu trừ của tài sản cố định"
"chart1361","Working capital provided to sub-units","1361","asset_receivable","True","Vốn kinh doanh ở các đơn vị trực thuộc"
"chart1362","Internal receivables on foreign exchange difference","1362","asset_receivable","True","Phải thu nội bộ về chênh lệch tỷ giá"
"chart1363","Internal receivables on borrowing costs eligible for capitalization","1363","asset_receivable","True","Phải thu nội bộ về chi phí đi vay đủ điều kiện được vốn hóa"
"chart1368","Other internal receivables","1368","asset_receivable","True","Phải thu nội bộ khác"
"chart1381","Shortage of assets awaiting resolution","1381","asset_receivable","True","Tài sản thiếu chờ xử lý"
"chart1385","Privatization receivables","1385","asset_receivable","True","Phải thu về cổ phần hóa"
"chart1388","Other receivables","1388","asset_receivable","True","Phải thu khác"
"chart141","Advances","141","asset_receivable","True","Tạm ứng"
"chart151","Goods in transit","151","asset_current","False","Hàng mua đang đi đường"
"chart152","Raw materials","152","asset_current","False","Nguyên liệu, vật liệu"
"chart1531","Tools & supplies","1531","asset_current","False","Công cụ, dụng cụ"
"chart1532","Packaging rotation","1532","asset_current","False","Bao bì luân chuyển"
"chart1533","Instruments for rent","1533","asset_current","False","Đồ dùng cho thuê"
"chart1534","Equipment & spare parts","1534","asset_current","False","Thiết bị, phụ tùng thay thế"
"chart1541","Construction contracts","1541","asset_current","False","Xây lắp"
"chart1542","Other work-in-progress products","1542","asset_current","False","Sản phẩm khác"
"chart1543","Services","1543","asset_current","False","Dịch vụ"
"chart1544","Warranty costs","1544","asset_current","False","Chi phí bảo hành xây lắp"
"chart1551","Finished goods - inventory","1551","asset_current","False","Thành phẩm nhập kho"
"chart1557","Finished goods - real estates","1557","asset_current","False","Thành phẩm bất động sản"
"chart1561","Purchase costs","1561","asset_current","False","Giá mua hàng hóa"
"chart1562","Incidental expense","1562","asset_current","False","Chi phí thu mua hàng hoá"
"chart1567","Property Inventories","1567","asset_current","False","Hàng hóa bất động sản"
"chart157","Outward goods on consignment","157","asset_current","False","Hàng gửi đi bán"
"chart158","Goods in bonded warehouse","158","asset_current","False","Hàng hóa kho bảo thuế"
"chart1611","Previous years expenditure","1611","asset_current","False","Chi sự nghiệp năm trước"
"chart1612","Current year expenditure","1612","asset_current","False","Chi sự nghiệp năm nay"
"chart171","Government bonds purchase-resale","171","asset_current","False","Giao dịch mua bán lại trái phiếu chính phủ"
"chart2111","Buildings & structures","2111","asset_non_current","False","Nhà cửa, vật kiến trúc"
"chart2112","Machinery & equipment","2112","asset_non_current","False","Máy móc, thiết bị"
"chart2113","Transportation & transmission vehicles","2113","asset_non_current","False","Phương tiện vận tải, truyền dẫn"
"chart2114","Office equipment and furniture","2114","asset_non_current","False","Thiết bị, dụng cụ quản lý"
"chart2115","Perennial plants, working and producing animals","2115","asset_non_current","False","Cây lâu năm, súc vật làm việc và cho sản phẩm"
"chart2118","Other tangible fixed assets","2118","asset_non_current","False","Tài sản cố định khác"
"chart2121","Financial leased tangible fixed assets","2121","asset_non_current","False","TSCĐ hữu hình thuê tài chính"
"chart2122","Financial leased intangible fixed assets","2122","asset_non_current","False","TSCĐ vô hình thuê tài chính"
"chart2131","Land use rights","2131","asset_non_current","False","Quyền sử dụng đất"
"chart2132","Copyrights","2132","asset_non_current","False","Quyền phát hành"
"chart2133","Patents","2133","asset_non_current","False","Bản quyền, bằng sáng chế"
"chart2134","Trademarks and brand name","2134","asset_non_current","False","Nhãn hiệu, tên thương mại"
"chart2135","Computer software","2135","asset_non_current","False","Chương trình phần mềm"
"chart2136","Licenses & franchises","2136","asset_non_current","False","Giấy phép và giấy phép nhượng quyền"
"chart2138","Other intangible fixed assets","2138","asset_non_current","False","TSCĐ vô hình khác"
"chart2141","Depreciation of tangible fixed assets","2141","asset_non_current","False","Hao mòn TSCĐ hữu hình"
"chart2142","Depreciation of financial leased assets","2142","asset_non_current","False","Hao mòn TSCĐ thuê tài chính"
"chart2143","Depreciation of intangible fixed assets","2143","asset_non_current","False","Hao mòn TSCĐ vô hình"
"chart2147","Depreciation of investment properties","2147","asset_non_current","False","Hao mòn bất động sản đầu tư"
"chart217","Investment properties","217","asset_non_current","False","Bất động sản đầu tư"
"chart221","Investment in subsidiaries","221","asset_non_current","False","Đầu tư vào công ty con"
"chart222","Investment in joint ventures and associates","222","asset_non_current","False","Đầu tư vào công ty liên doanh, liên kết"
"chart2281","Equity investments in other entities","2281","asset_non_current","False","Đầu tư góp vốn vào đơn vị khác"
"chart2288","Other investment","2288","asset_non_current","False","Đầu tư khác"
"chart2291","Provision for decline in value of trading securities","2291","asset_non_current","False","Dự phòng giảm giá chứng khoán kinh doanh"
"chart2292","Provision for investment loss in other entities","2292","asset_non_current","False","Dự phòng tổn thất đầu tư vào đơn vị khác"
"chart2293","Provision for doubtful debts","2293","asset_non_current","False","Dự phòng phải thu khó đòi"
"chart2294","Provision for reserve inventories","2294","asset_non_current","False","Dự phòng giảm giá hàng tồn kho"
"chart2411","Acquisition of fixed assets","2411","asset_non_current","False","Mua sắm TSCĐ"
"chart2412","Construction in progress","2412","asset_non_current","False","Xây dựng cơ bản"
"chart2413","Major repairs of fixed assets","2413","asset_non_current","False","Sửa chữa lớn TSCĐ"
"chart242","Prepaid expenses","242","asset_prepayments","False","Chi phí trả trước"
"chart243","Deferred tax assets","243","asset_non_current","False","Tài sản thuế thu nhập hoãn lại"
"chart244","Mortgage, collaterals and deposits","244","asset_non_current","False","Cầm cố, thế chấp, ký quỹ, ký cược"
"chart331","Trade payables","331","liability_payable","True","Phải trả cho người bán"
"chart33311","Output VAT","33311","liability_current","False","Thuế GTGT đầu ra"
"chart33312","VAT on imported goods","33312","liability_current","False","Thuế GTGT hàng nhập khẩu"
"chart3332","Special consumption tax","3332","liability_current","False","Thuế tiêu thụ đặc biệt"
"chart3333","Import & export tax","3333","liability_current","False","Thuế xuất, nhập khẩu"
"chart3334","Corporate income tax","3334","liability_current","False","Thuế thu nhập doanh nghiệp"
"chart3335","Personal income tax","3335","liability_current","False","Thuế thu nhập cá nhân"
"chart3336","Natural resources using tax","3336","liability_current","False","Thuế nhà đất, tiền thuê đất"
"chart3337","Land & housing tax, land rental charges","3337","liability_current","False","Thuế nhà đất, tiền thuê đất"
"chart33381","Environment protection tax","33381","liability_current","False","Thuế bảo vệ môi trường"
"chart33382","Other taxes","33382","liability_current","False","Các loại thuế khác"
"chart3339","Fees & charges & other payables","3339","liability_payable","True","Phí, lệ phí và các khoản phải nộp khác"
"chart3341","Payables to staff","3341","liability_payable","True","Phải trả công nhân viên"
"chart3348","Payables to others","3348","liability_payable","True","Phải trả người lao động khác"
"chart335","Accrued expenses","335","liability_payable","True","Chi phí phải trả"
"chart3361","Internal payables for working capital received","3361","liability_payable","True","Phải trả nội bộ về vốn kinh doanh"
"chart3362","Internal payables for foreign exchange differences","3362","liability_payable","True","Phải trả nội bộ về chênh lệch tỷ giá"
"chart3363","Internal payables for borrowing costs eligible for capitalization","3363","liability_payable","True","Phải trả nội bộ về chi phí đi vay đủ điều kiện được vốn hoá"
"chart3368","Other internal payables","3368","liability_payable","True","Phải trả nội bộ khác"
"chart337","Progress billings for construction contracts","337","liability_payable","True","Thanh toán theo tiến độ kế hoạch hợp đồng xây dựng"
"chart3381","Surplus of assets awaiting for resolution","3381","liability_payable","True","Tài sản thừa chờ giải quyết"
"chart3382","Trade union fees","3382","liability_payable","True","Kinh phí công đoàn"
"chart3383","Social insurance","3383","liability_payable","True","Bảo hiểm xã hội"
"chart3384","Health insurance","3384","liability_payable","True","Bảo hiểm y tế"
"chart3385","Payables on equitization","3385","liability_payable","True","Phải trả về cổ phần hóa"
"chart3386","Unemployment insurance","3386","liability_payable","True","Bảo hiểm thất nghiệp"
"chart3387","Unearned revenue","3387","liability_payable","True","Doanh thu chưa thực hiện"
"chart3388","Other payables","3388","liability_payable","True","Phải trả, phải nộp khác"
"chart3411","Borrowings","3411","liability_current","False","Các khoản đi vay"
"chart3412","Financial leased liabilities","3412","liability_current","False","Nợ thuê tài chính"
"chart3431","Ordinary bonds","3431","liability_current","False","Trái phiếu thường"
"chart34311","Par value of bonds","34311","liability_current","False","Mệnh giá trái phiếu"
"chart34312","Bond discounts","34312","liability_current","False","Chiết khấu trái phiếu"
"chart34313","Bond premiums","34313","liability_current","False","Phụ trội trái phiếu"
"chart3432","Convertible bonds","3432","liability_current","False","Trái phiếu chuyển đổi"
"chart344","Deposits received","344","liability_current","False","Nhận ký quỹ, ký cược"
"chart347","Deferred tax liabilities","347","liability_current","False","Thuế thu nhập hoãn lại phải trả"
"chart3521","Product warranty provisions","3521","liability_current","False","Dự phòng bảo hành sản phẩm hàng hóa"
"chart3522","Construction warranty provisions","3522","liability_current","False","Dự phòng bảo hành công trình xây dựng"
"chart3523","Enterprise restructuring provisions","3523","liability_current","False","Dự phòng tái cơ cấu doanh nghiệp"
"chart3524","Other provisions","3524","liability_current","False","Dự phòng phải trả khác"
"chart3531","Bonus fund","3531","liability_current","False","Quỹ khen thưởng"
"chart3532","Welfare fund","3532","liability_current","False","Quỹ phúc lợi"
"chart3533","Welfare fund used for fixed asset acquisitions","3533","liability_current","False","Quỹ phúc lợi đã hình thành TSCĐ"
"chart3534","Management bonus fund","3534","liability_current","False","Quỹ thưởng ban quản lý điều hành công ty"
"chart3561","Science and technology development fund","3561","liability_current","False","Quỹ phát triển khoa học và công nghệ"
"chart3562","Science and technology development fund used for fixed asset acquisition","3562","liability_current","False","Quỹ phát triển khoa học và công nghệ đã hình thành TSCĐ"
"chart357","Price stabilization fund","357","liability_current","False","Quỹ bình ổn giá"
"chart41111","Ordinary shares with voting rights","41111","equity","False","Cổ phiếu phổ thông có quyền biểu quyết"
"chart41112","Preference shares","41112","equity","False","Cổ phiếu ưu đãi"
"chart4112","Capital surplus","4112","equity","False","Thặng dư vốn cổ phần"
"chart4113","Conversion options on convertible bonds","4113","equity","False","Quyền chọn chuyển đổi trái phiếu"
"chart4118","Other capital","4118","equity","False","Vốn khác"
"chart412","Revaluation differences on asset","412","equity","False","Chênh lệch đánh giá lại tài sản"
"chart4131","Exchange rate differences on revaluation of monetary items denominated in foreign currency","4131","equity","False","Chênh lệch tỷ giá do đánh giá lại các khoản mục tiền tệ có gốc ngoại tệ"
"chart4132","Exchange rate differences in pre-operating period","4132","equity","False","Chênh lệch tỷ giá hối đoái trong giai đoạn trước hoạt động"
"chart414","Investment & development fund","414","equity","False","Quỹ đầu tư phát triển"
"chart417","Enterprise reorganization assistance fund","417","equity","False","Quỹ hỗ trợ sắp xếp doanh nghiệp"
"chart418","Other equity funds","418","equity","False","Các quỹ khác thuộc vốn chủ sở hữu"
"chart419","Treasury stocks","419","equity","False","Cổ phiếu quỹ"
"chart4211","Undistributed profit after tax of previous year","4211","equity","False","Lợi nhuận sau thuế chưa phân phối năm trước"
"chart4212","Undistributed profit after tax of current year","4212","equity","False","Lợi nhuận sau thuế chưa phân phối năm nay"
"chart441","Capital expenditure funds","441","equity","False","Nguồn vốn đầu tư xây dựng cơ bản"
"chart4611","Non-business funds of previous year","4611","equity","False","Nguồn kinh phí sự nghiệp năm trước"
"chart4612","Non-business funds of current year","4612","equity","False","Nguồn kinh phí sự nghiệp năm nay"
"chart466","Non-business funds used for fixed asset acquisitions","466","equity","False","Nguồn kinh phí sự nghiệp đã hình thành TSCĐ"
"chart5111","Revenue from sales of merchandises","5111","income","False","Doanh thu bán hàng hoá"
"chart5112","Revenue from sales of finished goods","5112","income","False","Doanh thu bán các thành phẩm"
"chart5113","Revenue from services rendered","5113","income","False","Doanh thu cung cấp dịch vụ"
"chart5114","Revenue from government grants","5114","income","False","Doanh thu trợ cấp, trợ giá"
"chart5117","Revenue from investment properties","5117","income","False","Doanh thu kinh doanh bất động sản đầu tư"
"chart5118","Other revenue","5118","income","False","Doanh thu khác"
"chart515","Financial income","515","income","False","Doanh thu hoạt động tài chính"
"chart5211","Sales discounts","5211","income","False","Chiết khấu thương mại"
"chart5212","Sales allowances","5212","income","False","Giảm giá hàng bán"
"chart5213","Sales returns","5213","income","False","Hàng bán bị trả lại"
"chart6111","Purchases of raw materials","6111","expense_direct_cost","False","Mua nguyên liệu, vật liệu"
"chart6112","Purchases of goods","6112","expense_direct_cost","False","Mua hàng hoá"
"chart621","Direct raw material costs","621","expense_direct_cost","False","Chi phí nguyên liệu, vật liệu trực tiếp"
"chart622","Direct labour costs","622","expense_direct_cost","False","Chi phí nhân công trực tiếp"
"chart6231","Labour costs","6231","expense","False","Chi phí nhân công"
"chart6232","Material costs","6232","expense","False","Chi phí nguyên, vật liệu"
"chart6233","Tools and instruments","6233","expense","False","Chi phí dụng cụ sản xuất"
"chart6234","Equipment depreciation expense","6234","expense","False","Chi phí khấu hao máy thi công"
"chart6237","Outside services","6237","expense","False","Chi phí dịch vụ mua ngoài"
"chart6238","Other expenses","6238","expense","False","Chi phí bằng tiền khác"
"chart6271","Factory staff costs","6271","expense","False","Chi phí nhân viên phân xưởng"
"chart6272","Material costs","6272","expense","False","Chi phí nguyên, vật liệu"
"chart6273","Tools and instruments","6273","expense","False","Chi phí dụng cụ sản xuất"
"chart6274","Fixed asset depreciation","6274","expense_depreciation","False","Chi phí khấu hao TSCĐ"
"chart6277","Outside services","6277","expense","False","Chi phí dịch vụ mua ngoài"
"chart6278","Other expenses","6278","expense","False","Chi phí bằng tiền khác"
"chart631","Production costs","631","expense_direct_cost","False","Giá thành sản xuất"
"chart632","Costs of goods sold","632","expense_direct_cost","False","Giá vốn hàng bán"
"chart635","Financial expenses","635","expense","False","Chi phí tài chính"
"chart6411","Employees costs","6411","expense","False","Chi phí nhân viên"
"chart6412","Materials and packing materials","6412","expense","False","Chi phí nguyên vật liệu, bao bì"
"chart6413","Tools and instruments","6413","expense","False","Chi phí dụng cụ, đồ dùng"
"chart6414","Fixed asset depreciation","6414","expense_depreciation","False","Chi phí khấu hao TSCĐ"
"chart6415","Warranty expenses","6415","expense","False","Chi phí bảo hành"
"chart6417","Outside services","6417","expense","False","Chi phí dịch vụ mua ngoài"
"chart6418","Other expenses","6418","expense","False","Chi phí bằng tiền khác"
"chart6421","Employees management costs","6421","expense","False","Chi phí nhân viên"
"chart6422","Office supply expenses","6422","expense","False","Chi phí vật liệu quản lý"
"chart6423","Stationery costs","6423","expense","False","Chi phí đồ dùng văn phòng"
"chart6424","Fixed asset depreciation","6424","expense_depreciation","False","Chi phí khấu hao TSCĐ"
"chart6425","Taxes, fees and charges","6425","expense","False","Thuế, phí và lệ phí"
"chart6426","Provision expenses","6426","expense","False","Chi phí dự phòng"
"chart6427","Outside services","6427","expense","False","Chi phí dịch vụ mua ngoài"
"chart6428","Other expenses","6428","expense","False","Chi phí bằng tiền khác"
"chart711","Other Income","711","income_other","False","Thu nhập khác"
"chart811","Other Expenses","811","expense","False","Chi phí khác"
"chart8211","Current tax expense","8211","expense","False","Chi phí thuế thu nhập doanh nghiệp hiện hành"
"chart8212","Deferred tax expense","8212","expense","False","Chi phí thuế thu nhập doanh nghiệp hoãn lại"
"chart911","Income Summary","911","equity_unaffected","False","Xác định kết quả kinh doanh"

```

## File: data\template\account.tax-vn.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","include_base_amount","description@vi_VN"
"tax_purchase_vat10","10%","Deductible VAT 10%","Deductible VAT 10%","10.0","percent","purchase","tax_group_10","base","invoice","+Untaxed Purchase of Goods and Services taxed 10%","","False","Thuế GTGT được khấu trừ 10%"
"","","","","","","","","tax","invoice","+VAT on purchase of goods and services 10%","chart1331","",""
"","","","","","","","","base","refund","-Untaxed Purchase of Goods and Services taxed 10%","","",""
"","","","","","","","","tax","refund","-VAT on purchase of goods and services 10%","chart1331","",""
"tax_purchase_vat8","8%","Deductible VAT 8%","Deductible VAT 8%","8.0","percent","purchase","tax_group_8","base","invoice","+Untaxed Purchase of Goods and Services taxed 8%","","False","Thuế GTGT được khấu trừ 8%"
"","","","","","","","","tax","invoice","+VAT on purchase of goods and services 8%","chart1331","",""
"","","","","","","","","base","refund","-Untaxed Purchase of Goods and Services taxed 8%","","",""
"","","","","","","","","tax","refund","-VAT on purchase of goods and services 8%","chart1331","",""
"tax_purchase_vat5","5%","Deductible VAT 5%","Deductible VAT 5%","5.0","percent","purchase","tax_group_5","base","invoice","+Untaxed Purchase of Goods and Services taxed 5%","","False","Thuế GTGT được khấu trừ 5%"
"","","","","","","","","tax","invoice","+VAT on purchase of goods and services 5%","chart1331","",""
"","","","","","","","","base","refund","-Untaxed Purchase of Goods and Services taxed 5%","","",""
"","","","","","","","","tax","refund","-VAT on purchase of goods and services 5%","chart1331","",""
"tax_purchase_vat0","0%","Deductible VAT 0%","Deductible VAT 0%","0.0","percent","purchase","tax_group_0","base","invoice","+Untaxed Purchase of Goods and Services taxed 0%","","False","Thuế GTGT được khấu trừ 0%"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-Untaxed Purchase of Goods and Services taxed 0%","","",""
"","","","","","","","","tax","refund","","","",""
"tax_purchase_vat_exemption","VAT EXEMPTION","VAT Exemption","VAT EXEMPTION","0.0","percent","purchase","tax_group_exemption","base","invoice","+Untaxed Purchase of Goods and Services taxed VAT Exemption","","False","Không thuộc đối tượng chịu thuế GTGT"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-Untaxed Purchase of Goods and Services taxed VAT Exemption","","",""
"","","","","","","","","tax","refund","","","",""
"tax_purchase_import_10","10% Import","Deductible VAT 10% For Imported Goods","Deductible VAT 10% For Imported Goods","10.0","percent","purchase","tax_group_10","base","invoice","+tax_purchase_import_10_base","","False","Thuế GTGT được khấu trừ cho hàng nhập khẩu 10%"
"","","","","","","","","tax","invoice","+tax_purchase_import_10","chart33312","",""
"","","","","","","","","base","refund","-tax_purchase_import_10_base","","",""
"","","","","","","","","tax","refund","-tax_purchase_import_10","chart33312","",""
"tax_purchase_import_8","8% Import","Deductible VAT 8% For Imported Goods","Deductible VAT 8% For Imported Goods","8.0","percent","purchase","tax_group_8","base","invoice","+tax_purchase_import_8_base","","False","Thuế GTGT được khấu trừ cho hàng nhập khẩu 8%"
"","","","","","","","","tax","invoice","+tax_purchase_import_8","chart33312","",""
"","","","","","","","","base","refund","-tax_purchase_import_8_base","","",""
"","","","","","","","","tax","refund","-tax_purchase_import_8","chart33312","",""
"tax_purchase_import_5","5% Import","Deductible VAT 5% For Imported Goods","Deductible VAT 5% For Imported Goods","5.0","percent","purchase","tax_group_5","base","invoice","+tax_purchase_import_5_base","","False","Thuế GTGT được khấu trừ cho hàng nhập khẩu 5%"
"","","","","","","","","tax","invoice","+tax_purchase_import_5","chart33312","",""
"","","","","","","","","base","refund","-tax_purchase_import_5_base","","",""
"","","","","","","","","tax","refund","-tax_purchase_import_5","chart33312","",""
"tax_purchase_import_0","0% Import","Deductible VAT 0% For Imported Goods","Deductible VAT 0% For Imported Goods","0.0","percent","purchase","tax_group_0","base","invoice","+tax_purchase_import_0_base","","False","Thuế GTGT được khấu trừ cho hàng nhập khẩu 0%"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-tax_purchase_import_0_base","","",""
"","","","","","","","","tax","refund","","","",""
"tax_import","Import Tax","Import Tax","Import Tax","5.0","percent","purchase","tax_group_import","base","invoice","+tax_import_base","","True","Thuế nhập khẩu"
"","","","","","","","","tax","invoice","+tax_import","chart3333","",""
"","","","","","","","","base","refund","-tax_import_base","","",""
"","","","","","","","","tax","refund","-tax_import","chart3333","",""
"tax_sale_vat10","10%","Value Added Tax (VAT) 10%","Value Added Tax (VAT) 10%","10.0","percent","sale","tax_group_10","base","invoice","+Untaxed sales of goods and services taxed 10%","","False","Thuế GTGT phải nộp 10%"
"","","","","","","","","tax","invoice","+VAT on sales of goods and services 10%","chart33311","",""
"","","","","","","","","base","refund","-Untaxed sales of goods and services taxed 10%","","",""
"","","","","","","","","tax","refund","-VAT on sales of goods and services 10%","chart33311","",""
"tax_sale_vat8","8%","Value Added Tax (VAT) 8%","Value Added Tax (VAT) 8%","8.0","percent","sale","tax_group_8","base","invoice","+Untaxed sales of goods and services taxed 8%","","False","Thuế GTGT phải nộp 8%"
"","","","","","","","","tax","invoice","+VAT on sales of goods and services 8%","chart33311","",""
"","","","","","","","","base","refund","-Untaxed sales of goods and services taxed 8%","","",""
"","","","","","","","","tax","refund","-VAT on sales of goods and services 8%","chart33311","",""
"tax_sale_vat5","5%","Value Added Tax (VAT) 5%","Value Added Tax (VAT) 5%","5.0","percent","sale","tax_group_5","base","invoice","+Untaxed sales of goods and services taxed 5%","","False","Thuế GTGT phải nộp 5%"
"","","","","","","","","tax","invoice","+VAT on sales of goods and services 5%","chart33311","",""
"","","","","","","","","base","refund","-Untaxed sales of goods and services taxed 5%","","",""
"","","","","","","","","tax","refund","-VAT on sales of goods and services 5%","chart33311","",""
"tax_sale_vat0","0%","Value Added Tax (VAT) 0%","Value Added Tax (VAT) 0%","0.0","percent","sale","tax_group_0","base","invoice","+Untaxed sales of goods and services taxed 0%","","False","Thuế GTGT phải nộp 0%"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-Untaxed sales of goods and services taxed 0%","","",""
"","","","","","","","","tax","refund","","","",""
"tax_sale_vat_exemption","VAT EXEMPTION","VAT Exemption","VAT EXEMPTION","0.0","percent","sale","tax_group_exemption","base","invoice","+Untaxed sales of goods and services taxed VAT Exemption","","False","Không thuộc đối tượng chịu thuế GTGT"
"","","","","","","","","tax","invoice","","","",""
"","","","","","","","","base","refund","-Untaxed sales of goods and services taxed VAT Exemption","","",""
"","","","","","","","","tax","refund","","","",""

```

## File: data\template\account.tax.group-vn.csv

```csv
"id","name","country_id","name@vi_VN"
"tax_group_0","VAT 0%","base.vn","Thuế GTGT 0%"
"tax_group_5","VAT 5%","base.vn","Thuế GTGT 5%"
"tax_group_8","VAT 8%","base.vn","Thuế GTGT 8%"
"tax_group_10","VAT 10%","base.vn","Thuế GTGT 10%"
"tax_group_exemption","VAT EXEMPTION","base.vn","Không thuộc đối tượng chịu thuế GTGT"
"tax_group_import","VAT IMPORT","base.vn","Thuế nhập khẩu"

```

## File: migrations\14.0.2.0.1\post-migration.py

```python
# -*- coding: utf-8 -*-
from odoo import api, SUPERUSER_ID

FIXED_ACCOUNTS_MAP = {
    '5221': '5211',
    '5222': '5212',
    '5223': '5213'
    }


def _fix_revenue_deduction_accounts_code(env):
    vn_template = env.ref('l10n_vn.vn_template')
    for company in env['res.company'].with_context(active_test=False).search([('chart_template_id', '=', vn_template.id)]):
        for incorrect_code, correct_code in FIXED_ACCOUNTS_MAP.items():
            account = env['account.account'].search([('code', '=', incorrect_code), ('company_id', '=', company.id)])
            if account:
                account.write({'code': correct_code})


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    _fix_revenue_deduction_accounts_code(env)

```

## File: migrations\17.0.2.0.2\post-migration.py

```python
from odoo import api, SUPERUSER_ID
from odoo.osv import expression
from odoo.release import version
from odoo.tools import parse_version

FIXED_ACCOUNTS_TYPE = {
    'asset_prepayments': ['242'],
    'expense_depreciation': ['6274', '6414', '6424'],
}


def _fix_accounts_type(env):
    for correct_account_type, accounts_code in FIXED_ACCOUNTS_TYPE.items():
        domains_per_company = []
        for company in env['res.company'].with_context(active_test=False).search([('chart_template', '=', 'vn')]):
            if parse_version(version) > parse_version("saas~17.5"):
                company_domain = [('company_ids', 'in', company.ids), ('account_type', '!=', correct_account_type)]
            else:
                company_domain = [('company_id', '=', company.id), ('account_type', '!=', correct_account_type)]
            doamin = expression.AND([company_domain, expression.OR([[('code', 'like', f'{code}%')] for code in accounts_code])])
            domains_per_company.append(doamin)
        accounts = env['account.account'].search(expression.OR(domains_per_company))
        accounts.account_type = correct_account_type


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    _fix_accounts_type(env)

```

## File: migrations\2.0.3\end-migrate_update_taxes.py

```python
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'vn')], order="parent_path"):
        env['account.chart.template'].try_loading('vn', company)

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_vn_e_invoice_number = fields.Char(
        string='eInvoice Number',
        help='Electronic Invoicing number.',
        copy=False,
    )

```

## File: models\res_bank.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError


class ResPartnerBank(models.Model):
    _inherit = 'res.partner.bank'

    proxy_type = fields.Selection(selection_add=[('merchant_id', 'Merchant ID'),
                                                 ('payment_service', 'Payment Service'),
                                                 ('atm_card', 'ATM Card Number'),
                                                 ('bank_acc', 'Bank Account')],
                                  ondelete={'merchant_id': 'set default', 'payment_service': 'set default', 'atm_card': 'set default', 'bank_acc': 'set default'})

    @api.constrains('proxy_type')
    def _check_vn_proxy(self):
        for bank in self.filtered(lambda b: b.country_code == 'VN'):
            if bank.proxy_type not in ['merchant_id', 'payment_service', 'atm_card', 'bank_acc', 'none', False]:
                raise ValidationError(_("The QR Code Type must be either Merchant ID, ATM Card Number or Bank Account to generate a Vietnam Bank QR code for account number %s.", bank.acc_number))

    @api.depends('country_code')
    def _compute_display_qr_setting(self):
        bank_vn = self.filtered(lambda b: b.country_code == 'VN')
        bank_vn.display_qr_setting = True
        super(ResPartnerBank, self - bank_vn)._compute_display_qr_setting()

    def _get_merchant_account_info(self):
        if self.country_code == 'VN':
            proxy_type_mapping = {
                'merchant_id': 'QRPUSH',
                'payment_service': 'QRPUSH',
                'atm_card': 'QRIBFTTC',
                'bank_acc': 'QRIBFTTA',
            }
            payment_network = [
                (0, self.bank_bic),
                (1, self.proxy_value),
            ]
            vals = [
                (0, 'A000000727'),
                (1, ''.join([self._serialize(*val) for val in payment_network])),
                (2, proxy_type_mapping[self.proxy_type]),
            ]
            return (38, ''.join([self._serialize(*val) for val in vals]))
        return super()._get_merchant_account_info()

    def _get_additional_data_field(self, comment):
        if self.country_code == 'VN':
            return self._serialize(8, comment)
        return super()._get_additional_data_field(comment)

    def _get_error_messages_for_qr(self, qr_method, debtor_partner, currency):
        if qr_method == 'emv_qr' and self.country_code == 'VN':
            if currency.name not in ['VND']:
                return _("Can't generate a Vietnamese QR banking code with a currency other than VND.")
            if not self.bank_bic:
                return _("Missing Bank Identifier Code.\n"
                         "Please configure the Bank Identifier Code inside the bank settings.")
            return None

        return super()._get_error_messages_for_qr(qr_method, debtor_partner, currency)

    def _check_for_qr_code_errors(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        if qr_method == 'emv_qr' and self.country_code == 'VN' and self.proxy_type not in ['merchant_id', 'payment_service', 'atm_card', 'bank_acc']:
            return _("The proxy type %s is not supported for Vietnamese partners. It must be either Merchant ID, ATM Card Number or Bank Account", self.proxy_type)

        return super()._check_for_qr_code_errors(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

```

## File: models\template_vn.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('vn')
    def _get_vn_template_data(self):
        return {
            'code_digits': '4',
            'property_account_receivable_id': 'chart131',
            'property_account_payable_id': 'chart331',
            'property_account_expense_categ_id': 'chart1561',
            'property_account_income_categ_id': 'chart5111',
            'display_invoice_amount_total_words': True,
        }

    @template('vn', 'res.company')
    def _get_vn_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.vn',
                'bank_account_code_prefix': '112',
                'cash_account_code_prefix': '111',
                'transfer_account_code_prefix': '113',
                'account_default_pos_receivable_account_id': 'chart131',
                'income_currency_exchange_account_id': 'chart515',
                'expense_currency_exchange_account_id': 'chart635',
                'account_journal_early_pay_discount_loss_account_id': 'chart635',
                'account_journal_early_pay_discount_gain_account_id': 'chart515',
                'account_sale_tax_id': 'tax_sale_vat10',
                'account_purchase_tax_id': 'tax_purchase_vat10',
                'transfer_account_id': 'chart1131',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move
from . import res_bank
from . import template_vn

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_invoice_form_inherit_l10n_vn" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n.vn</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@id='other_tab_entry']" position="after">
                <page id="l10n_vn"
                      string="Vietnamese Electronic Invoicing"
                      invisible="move_type not in ['out_invoice', 'out_refund'] or country_code != 'VN'">
                    <group>
                        <group>
                            <field name="l10n_vn_e_invoice_number"/>
                        </group>
                    </group>
                </page>
            </xpath>
        </field>
    </record>

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
                <p invisible="country_code != 'VN'">
                    <widget name="documentation_link" path="/applications/finance/fiscal_localizations/vietnam.html" label="Documentation"/>
                </p>
            </field>
        </field>
    </record>

</odoo>

```

