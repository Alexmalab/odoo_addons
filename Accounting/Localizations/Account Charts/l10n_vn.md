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
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/vietnam.html',
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
    ],
    'data': [
        'data/account_tax_report_data.xml',
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
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_01_vn" model="account.report.line">
                <field name="name">Purchase of Goods and Services</field>
                <field name="aggregation_formula">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES.balance + UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_01_01_vn" model="account.report.line">
                        <field name="name">VAT on purchase of goods and services</field>
                        <field name="code">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES</field>
                        <field name="aggregation_formula">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_0.balance + VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_5.balance + VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_10.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services 0%</field>
                                <field name="code">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_01_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on purchase of goods and services 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services 5%</field>
                                <field name="code">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_01_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on purchase of goods and services 5%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_01_01_vn" model="account.report.line">
                                <field name="name">VAT on purchase of goods and services 10%</field>
                                <field name="code">VAT_ON_PURCHASE_OF_GOODS_AND_SERVICES_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_01_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on purchase of goods and services 10%</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_02_01_vn" model="account.report.line">
                        <field name="name">Untaxed Purchase of Goods and Services</field>
                        <field name="code">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES</field>
                        <field name="aggregation_formula">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_0.balance + UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_5.balance + UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_10.balance + UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_VAT_EXEMPTION.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_02_01_vn" model="account.report.line">
                                <field name="name">Untaxed Purchase of Goods and Services taxed 0%</field>
                                <field name="code">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_02_01_vn" model="account.report.line">
                                <field name="name">Untaxed Purchase of Goods and Services taxed 5%</field>
                                <field name="code">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed 5%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_02_01_vn" model="account.report.line">
                                <field name="name">Untaxed Purchase of Goods and Services taxed 10%</field>
                                <field name="code">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed 10%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_04_02_01_vn" model="account.report.line">
                                <field name="name">Untaxed Purchase of Goods and Services taxed VAT Exemption</field>
                                <field name="code">UNTAXED_PURCHASE_OF_GOODS_AND_SERVICES_TAXED_VAT_EXEMPTION</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_04_02_01_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed Purchase of Goods and Services taxed VAT Exemption</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_02_vn" model="account.report.line">
                <field name="name">Sales of Goods and Services</field>
                <field name="aggregation_formula">VAT_ON_SALES_OF_GOODS_AND_SERVICES.balance + UNTAXED_SALES_OF_GOODS_AND_SERVICES.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_01_02_vn" model="account.report.line">
                        <field name="name">VAT on sales of goods and services</field>
                        <field name="code">VAT_ON_SALES_OF_GOODS_AND_SERVICES</field>
                        <field name="aggregation_formula">VAT_ON_SALES_OF_GOODS_AND_SERVICES_0.balance + VAT_ON_SALES_OF_GOODS_AND_SERVICES_5.balance + VAT_ON_SALES_OF_GOODS_AND_SERVICES_10.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services 0%</field>
                                <field name="code">VAT_ON_SALES_OF_GOODS_AND_SERVICES_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_01_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on sales of goods and services 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services 5%</field>
                                <field name="code">VAT_ON_SALES_OF_GOODS_AND_SERVICES_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_01_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on sales of goods and services 5%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_01_02_vn" model="account.report.line">
                                <field name="name">VAT on sales of goods and services 10%</field>
                                <field name="code">VAT_ON_SALES_OF_GOODS_AND_SERVICES_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_01_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">VAT on sales of goods and services 10%</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_02_02_vn" model="account.report.line">
                        <field name="name">Untaxed Sales of Goods and Services</field>
                        <field name="code">UNTAXED_SALES_OF_GOODS_AND_SERVICES</field>
                        <field name="aggregation_formula">UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_0.balance + UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_5.balance + UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_10.balance + UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_VAT_EXEMPTION.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_01_02_02_vn" model="account.report.line">
                                <field name="name">Untaxed sales of goods and services taxed 0%</field>
                                <field name="code">UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_0</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_01_02_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed 0%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_02_02_02_vn" model="account.report.line">
                                <field name="name">Untaxed sales of goods and services taxed 5%</field>
                                <field name="code">UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_5</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_02_02_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed 5%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_03_02_02_vn" model="account.report.line">
                                <field name="name">Untaxed sales of goods and services taxed 10%</field>
                                <field name="code">UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_10</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_03_02_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Untaxed sales of goods and services taxed 10%</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_04_02_02_vn" model="account.report.line">
                                <field name="name">Untaxed sales of goods and services taxed VAT Exemption</field>
                                <field name="code">UNTAXED_SALES_OF_GOODS_AND_SERVICES_TAXED_VAT_EXEMPTION</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_04_02_02_vn_tag" model="account.report.expression">
                                        <field name="label">balance</field>
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
"chart1121","Vietnamese Dong","1121","asset_cash","False","Tiền Việt Nam"
"chart1122","Foreign currencies","1122","asset_cash","False","Ngoại tệ"
"chart1123","Monetary Gold","1123","asset_cash","False","Vàng tiền tệ"
"chart1211","Shares","1211","asset_current","False","Cổ phiếu"
"chart1212","Bonds","1212","asset_current","False","Trái phiếu"
"chart1218","Other securities and financial instruments","1218","asset_current","False","Chứng khoán và công cụ tài chính khác"
"chart1281","Term deposits","1281","asset_current","False","Tiền gửi có kỳ hạn"
"chart1282","Bonds","1282","asset_current","False","Trái phiếu"
"chart1283","Lending loans","1283","asset_current","False","Cho vay"
"chart1288","Other held to maturity investments","1288","asset_current","False","Các khoản đầu tư khác nắm giữ đến ngày đáo hạn"
"chart131","Trade receivables","131","asset_receivable","True","Phải thu của khách hàng"
"chart132","Trade receivables(pos)","132","asset_receivable","True","Phải thu của khách hàng(pos)"
"chart1331","VAT on purchase of goods and services","1331","asset_current","False","Thuế GTGT HHDV mua vào"
"chart1332","VAT on purchase of fixed assets","1332","asset_current","False","Thuế GTGT được khấu trừ của tài sản cố định"
"chart1361","Working capital provided to sub-units","1361","asset_receivable","True","Vốn kinh doanh ở các đơn vị trực thuộc"
"chart1362","Intra-company receivables on foreign exchange","1362","asset_receivable","True","Phải thu nội bộ về chênh lệch tỷ giá"
"chart1363","Intra-company receivables on borrowing costs eligible to be capitalized","1363","asset_receivable","True","Phải thu nội bộ về chi phí đi vay đủ điều kiện được vốn hóa"
"chart1368","Other intra-company receivables","1368","asset_receivable","True","Phải thu nội bộ khác"
"chart1381","Shortage of assets awaiting resolution","1381","asset_receivable","True","Tài sản thiếu chờ xử lý"
"chart1385","Receivables from privatization","1385","asset_receivable","True","Phải thu về cổ phần hóa"
"chart1388","Other receivables","1388","asset_receivable","True","Phải thu khác"
"chart141","Advances","141","asset_receivable","True","Tạm ứng"
"chart151","Goods in transit","151","asset_current","False","Hàng mua đang đi đường"
"chart152","Raw materials","152","asset_current","False","Nguyên liệu, vật liệu"
"chart1531","Tools and supplies","1531","asset_current","False","Công cụ, dụng cụ"
"chart1532","Reusable packaging materials","1532","asset_current","False","Bao bì luân chuyển"
"chart1533","Instruments for renting","1533","asset_current","False","Đồ dùng cho thuê"
"chart1534","Equipment and spare parts for replacement","1534","asset_current","False","Thiết bị, phụ tùng thay thế"
"chart1541","Construction contracts","1541","asset_current","False","Xây lắp"
"chart1542","Other work-in-progress products","1542","asset_current","False","Sản phẩm khác"
"chart1543","Services","1543","asset_current","False","Dịch vụ"
"chart1544","Warranty costs","1544","asset_current","False","Chi phí bảo hành xây lắp"
"chart1551","Finished products - inventory","1551","asset_current","False","Thành phẩm nhập kho"
"chart1557","Finished products - real estates","1557","asset_current","False","Thành phẩm bất động sản"
"chart1561","Purchase costs","1561","asset_current","False","Giá mua hàng hóa"
"chart1562","Incidental purchase costs","1562","asset_current","False","Chi phí thu mua hàng hoá"
"chart1567","Properties held for sale","1567","asset_current","False","Hàng hóa bất động sản"
"chart157","Outward goods on consignment","157","asset_current","False","Hàng gửi đi bán"
"chart158","Goods in bonded warehouse","158","asset_current","False","Hàng hóa kho bảo thuế"
"chart1611","Expenditure brought forward","1611","asset_current","False","Chi sự nghiệp năm trước"
"chart1612","Expenditure of current year","1612","asset_current","False","Chi sự nghiệp năm nay"
"chart171","Government bonds purchased for resale","171","asset_current","False","Giao dịch mua bán lại trái phiếu chính phủ"
"chart2111","Buildings and structures","2111","asset_non_current","False","Nhà cửa, vật kiến trúc"
"chart2112","Machinery and equipment","2112","asset_non_current","False","Máy móc, thiết bị"
"chart2113","Means of transportation and transmission","2113","asset_non_current","False","Phương tiện vận tải, truyền dẫn"
"chart2114","Office equipment and furniture","2114","asset_non_current","False","Thiết bị, dụng cụ quản lý"
"chart2115","Perennial plants, working animals and farm livestocks","2115","asset_non_current","False","Cây lâu năm, súc vật làm việc và cho sản phẩm"
"chart2118","Other fixed assets","2118","asset_non_current","False","Tài sản cố định khác"
"chart2121","Finance lease tangible fixed assets","2121","asset_non_current","False","TSCĐ hữu hình thuê tài chính"
"chart2122","Finance lease intangible fixed assets","2122","asset_non_current","False","TSCĐ vô hình thuê tài chính"
"chart2131","Land use rights","2131","asset_non_current","False","Quyền sử dụng đất"
"chart2132","Copyrights","2132","asset_non_current","False","Quyền phát hành"
"chart2133","Patents and inventions","2133","asset_non_current","False","Bản quyền, bằng sáng chế"
"chart2134","Product labels and trademarks","2134","asset_non_current","False","Nhãn hiệu, tên thương mại"
"chart2135","Computer software","2135","asset_non_current","False","Chương trình phần mềm"
"chart2136","Licenses and franchises","2136","asset_non_current","False","Giấy phép và giấy phép nhượng quyền"
"chart2138","Other intangible fixed assets","2138","asset_non_current","False","TSCĐ vô hình khác"
"chart2141","Depreciation of tangible fixed assets","2141","asset_non_current","False","Hao mòn TSCĐ hữu hình"
"chart2142","Depreciation of finance lease assets","2142","asset_non_current","False","Hao mòn TSCĐ thuê tài chính"
"chart2143","Amortization of intangible assets","2143","asset_non_current","False","Hao mòn TSCĐ vô hình"
"chart2147","Depreciation of investment properties","2147","asset_non_current","False","Hao mòn bất động sản đầu tư"
"chart217","Investment properties","217","asset_non_current","False","Bất động sản đầu tư"
"chart221","Investment in subsidiaries","221","asset_non_current","False","Đầu tư vào công ty con"
"chart222","Investment in joint ventures and associates","222","asset_non_current","False","Đầu tư vào công ty liên doanh, liên kết"
"chart2281","Equity investments in other entities Other investment","2281","asset_non_current","False","Đầu tư góp vốn vào đơn vị khác"
"chart2291","Allowances for decline in value of trading securities","2291","asset_non_current","False","Dự phòng giảm giá chứng khoán kinh doanh"
"chart2292","Allowances for impairment of investments in other entities","2292","asset_non_current","False","Dự phòng tổn thất đầu tư vào đơn vị khác"
"chart2293","Allowances for doubtful debts","2293","asset_non_current","False","Dự phòng phải thu khó đòi"
"chart2294","Allowances for inventories","2294","asset_non_current","False","Dự phòng giảm giá hàng tồn kho"
"chart2411","Fixed assets prior to commissioning","2411","asset_non_current","False","Mua sắm TSCĐ"
"chart2412","Construction works","2412","asset_non_current","False","Xây dựng cơ bản"
"chart2413","Major repairs of fixed assets","2413","asset_non_current","False","Sửa chữa lớn TSCĐ"
"chart242","Prepaid expenses","242","asset_prepayments","False","Chi phí trả trước"
"chart243","Deferred tax assets","243","asset_non_current","False","Tài sản thuế thu nhập hoãn lại"
"chart244","Mortgage, collaterals and deposits","244","asset_non_current","False","Cầm cố, thế chấp, ký quỹ, ký cược"
"chart331","Trade payables","331","liability_payable","True","Phải trả cho người bán"
"chart33311","Output VAT","33311","liability_current","False","Thuế GTGT đầu ra"
"chart33312","VAT on imported goods","33312","liability_current","False","Thuế GTGT hàng nhập khẩu"
"chart3332","Special consumption tax","3332","liability_current","False","Thuế tiêu thụ đặc biệt"
"chart3333","Import and export tax","3333","liability_current","False","Thuế xuất, nhập khẩu"
"chart3334","Corporate income tax","3334","liability_current","False","Thuế thu nhập doanh nghiệp"
"chart3335","Personal income tax","3335","liability_current","False","Thuế thu nhập cá nhân"
"chart3336","Tax on use of natural resources","3336","liability_current","False","Thuế nhà đất, tiền thuê đất"
"chart3337","Land and housing tax, and rental charges","3337","liability_current","False","Thuế nhà đất, tiền thuê đất"
"chart33381","Environment protection tax","33381","liability_current","False","Thuế bảo vệ môi trường"
"chart33382","Other taxes","33382","liability_current","False","Các loại thuế khác"
"chart3339","Fees, charges and other payables","3339","liability_payable","True","Phí, lệ phí và các khoản phải nộp khác"
"chart3341","Payables to staff","3341","liability_payable","True","Phải trả công nhân viên"
"chart3348","Payables to others","3348","liability_payable","True","Phải trả người lao động khác"
"chart335","Accrued expenses","335","liability_payable","True","Chi phí phải trả"
"chart3361","Intra-company payables for operating capital received","3361","liability_payable","True","Phải trả nội bộ về vốn kinh doanh"
"chart3362","Intra-company payables for foreign exchange differences","3362","liability_payable","True","Phải trả nội bộ về chênh lệch tỷ giá"
"chart3363","Intra-company payables for borrowing costs eligible to be capitalized","3363","liability_payable","True","Phải trả nội bộ về chi phí đi vay đủ điều kiện được vốn hoá"
"chart3368","Other inter-company payables","3368","liability_payable","True","Phải trả nội bộ khác"
"chart337","Progress billings for construction contracts","337","liability_payable","True","Thanh toán theo tiến độ kế hoạch hợp đồng xây dựng"
"chart3381","Surplus of assets awaiting resolution","3381","liability_payable","True","Tài sản thừa chờ giải quyết"
"chart3382","Trade union fees","3382","liability_payable","True","Kinh phí công đoàn"
"chart3383","Social insurance","3383","liability_payable","True","Bảo hiểm xã hội"
"chart3384","Health insurance","3384","liability_payable","True","Bảo hiểm y tế"
"chart3385","Payables on equitization","3385","liability_payable","True","Phải trả về cổ phần hóa"
"chart3386","Unemployment insurance","3386","liability_payable","True","Bảo hiểm thất nghiệp"
"chart3387","Unearned revenue","3387","liability_payable","True","Doanh thu chưa thực hiện"
"chart3388","Other payables","3388","liability_payable","True","Phải trả, phải nộp khác"
"chart3411","Borrowing loans liabilities","3411","liability_current","False","Các khoản đi vay"
"chart3412","Finance lease liabilities","3412","liability_current","False","Nợ thuê tài chính"
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
"chart412","Differences upon asset revaluation","412","equity","False","Chênh lệch đánh giá lại tài sản"
"chart4131","Exchange rate differences on revaluation of monetary items denominated in foreign currency","4131","equity","False","Chênh lệch tỷ giá do đánh giá lại các khoản mục tiền tệ có gốc ngoại tệ"
"chart4132","Exchange rate differences in pre-operating period","4132","equity","False","Chênh lệch tỷ giá hối đoái trong giai đoạn trước hoạt động"
"chart414","Investment and development fund","414","equity","False","Quỹ đầu tư phát triển"
"chart417","Enterprise reorganization assistance fund","417","equity","False","Quỹ hỗ trợ sắp xếp doanh nghiệp"
"chart418","Other equity funds","418","equity","False","Các quỹ khác thuộc vốn chủ sở hữu"
"chart419","Treasury shares","419","equity","False","Cổ phiếu quỹ"
"chart4211","Undistributed profit after tax brought forward","4211","equity","False","Lợi nhuận sau thuế chưa phân phối năm trước"
"chart4212","Undistributed profit(loss) after tax for the current year","4212","equity","False","Lợi nhuận sau thuế chưa phân phối năm nay"
"chart441","Capital expenditure funds","441","equity","False","Nguồn vốn đầu tư xây dựng cơ bản"
"chart4611","Non-business funds bought forward","4611","equity","False","Nguồn kinh phí sự nghiệp năm trước"
"chart4612","Non-business funds for current year","4612","equity","False","Nguồn kinh phí sự nghiệp năm nay"
"chart466","Non-business funds used for fixed asset acquisitions","466","equity","False","Nguồn kinh phí sự nghiệp đã hình thành TSCĐ"
"chart5111","Revenue from sales of merchandises","5111","income","False","Doanh thu bán hàng hoá"
"chart5112","Revenue from sales of finished goods","5112","income","False","Doanh thu bán các thành phẩm"
"chart5113","Revenue from services rendered","5113","income","False","Doanh thu cung cấp dịch vụ"
"chart5114","Revenue from government grants","5114","income","False","Doanh thu trợ cấp, trợ giá"
"chart5117","Revenue from investment properties","5117","income","False","Doanh thu kinh doanh bất động sản đầu tư"
"chart5118","Other revenue","5118","income","False","Doanh thu khác"
"chart515","Financial income","515","income","False","Doanh thu hoạt động tài chính"
"chart5211","Trade discounts","5211","income","False","Chiết khấu thương mại"
"chart5212","Sales returns","5212","income","False","Hàng bán bị trả lại"
"chart5213","Sales rebates","5213","income","False","Giảm giá hàng bán"
"chart6111","Purchases of raw materials","6111","expense_direct_cost","False","Mua nguyên liệu, vật liệu"
"chart621","Direct raw material costs","621","expense_direct_cost","False","Chi phí nguyên liệu, vật liệu trực tiếp"
"chart622","Direct labour costs","622","expense_direct_cost","False","Chi phí nhân công trực tiếp"
"chart6231","Labour costs","6231","expense","False","Chi phí nhân công"
"chart6232","Material costs","6232","expense","False","Chi phí nguyên, vật liệu"
"chart6233","Production tools and instruments","6233","expense","False","Chi phí dụng cụ sản xuất"
"chart6234","Depreciation expense","6234","expense","False","Chi phí khấu hao máy thi công"
"chart6237","Outside services","6237","expense","False","Chi phí dịch vụ mua ngoài"
"chart6238","Other expenses","6238","expense","False","Chi phí bằng tiền khác"
"chart6271","Factory staff costs","6271","expense","False","Chi phí nhân viên phân xưởng"
"chart6272","Material costs","6272","expense","False","Chi phí nguyên, vật liệu"
"chart6273","Production tools and instruments","6273","expense","False","Chi phí dụng cụ sản xuất"
"chart6274","Fixed asset depreciation","6274","expense_depreciation","False","Chi phí khấu hao TSCĐ"
"chart6277","Outside services","6277","expense","False","Chi phí dịch vụ mua ngoài"
"chart6278","Other expenses","6278","expense","False","Chi phí bằng tiền khác"
"chart631","Production costs","631","expense_direct_cost","False","Giá thành sản xuất"
"chart632","Costs of goods sold","632","expense_direct_cost","False","Giá vốn hàng bán"
"chart635","Financial expenses","635","expense","False","Chi phí tài chính"
"chart6411","Staff expenses","6411","expense","False","Chi phí nhân viên"
"chart6412","Materials and packing materials","6412","expense","False","Chi phí nguyên vật liệu, bao bì"
"chart6413","Tools and instruments","6413","expense","False","Chi phí dụng cụ, đồ dùng"
"chart6414","Fixed asset deprecation","6414","expense_depreciation","False","Chi phí khấu hao TSCĐ"
"chart6415","Warranty expenses","6415","expense","False","Chi phí bảo hành"
"chart6417","Outside services","6417","expense","False","Chi phí dịch vụ mua ngoài"
"chart6418","Other expenses","6418","expense","False","Chi phí bằng tiền khác"
"chart6421","Staff expenses","6421","expense","False","Chi phí nhân viên"
"chart6422","Office supply expenses","6422","expense","False","Chi phí vật liệu quản lý"
"chart6423","Office equipment expenses","6423","expense","False","Chi phí đồ dùng văn phòng"
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
"chart9993","Cash Discount Loss","9993","expense","False",""
"chart9994","Cash Discount Income","9994","income_other","False",""

```

## File: data\template\account.tax-vn.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@vi_VN"
"tax_purchase_vat10","10%","Deductible VAT 10%","Deductible VAT 10%","10.0","percent","purchase","tax_group_10","base","invoice","+Untaxed Purchase of Goods and Services taxed 10%","","Thuế GTGT được khấu trừ 10%"
"","","","","","","","","tax","invoice","+VAT on purchase of goods and services 10%","chart1331",""
"","","","","","","","","base","refund","-Untaxed Purchase of Goods and Services taxed 10%","",""
"","","","","","","","","tax","refund","-VAT on purchase of goods and services 10%","chart1331",""
"tax_purchase_vat5","5%","Deductible VAT 5%","Deductible VAT 5%","5.0","percent","purchase","tax_group_5","base","invoice","+Untaxed Purchase of Goods and Services taxed 5%","","Thuế GTGT được khấu trừ 5%"
"","","","","","","","","tax","invoice","+VAT on purchase of goods and services 5%","chart1331",""
"","","","","","","","","base","refund","-Untaxed Purchase of Goods and Services taxed 5%","",""
"","","","","","","","","tax","refund","-VAT on purchase of goods and services 5%","chart1331",""
"tax_purchase_vat0","0%","Deductible VAT 0%","Deductible VAT 0%","0.0","percent","purchase","tax_group_0","base","invoice","+Untaxed Purchase of Goods and Services taxed 0%","","Thuế GTGT được khấu trừ 0%"
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-Untaxed Purchase of Goods and Services taxed 0%","",""
"","","","","","","","","tax","refund","","",""
"tax_purchase_vat_exemption","VAT EXEMPTION","VAT Exemption","VAT EXEMPTION","0.0","percent","purchase","tax_group_exemption","base","invoice","+Untaxed Purchase of Goods and Services taxed VAT Exemption","","Không thuộc đối tượng chịu thuế GTGT"
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-Untaxed Purchase of Goods and Services taxed VAT Exemption","",""
"","","","","","","","","tax","refund","","",""
"tax_sale_vat10","10%","Value Added Tax (VAT) 10%","Value Added Tax (VAT) 10%","10.0","percent","sale","tax_group_10","base","invoice","+Untaxed sales of goods and services taxed 10%","","Thuế GTGT phải nộp 10%"
"","","","","","","","","tax","invoice","+VAT on sales of goods and services 10%","chart33311",""
"","","","","","","","","base","refund","-Untaxed sales of goods and services taxed 10%","",""
"","","","","","","","","tax","refund","-VAT on sales of goods and services 10%","chart33311",""
"tax_sale_vat5","5%","Value Added Tax (VAT) 5%","Value Added Tax (VAT) 5%","5.0","percent","sale","tax_group_5","base","invoice","+Untaxed sales of goods and services taxed 5%","","Thuế GTGT phải nộp 5%"
"","","","","","","","","tax","invoice","+VAT on sales of goods and services 5%","chart33311",""
"","","","","","","","","base","refund","-Untaxed sales of goods and services taxed 5%","",""
"","","","","","","","","tax","refund","-VAT on sales of goods and services 5%","chart33311",""
"tax_sale_vat0","0%","Value Added Tax (VAT) 0%","Value Added Tax (VAT) 0%","0.0","percent","sale","tax_group_0","base","invoice","+Untaxed sales of goods and services taxed 0%","","Thuế GTGT phải nộp 0%"
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-Untaxed sales of goods and services taxed 0%","",""
"","","","","","","","","tax","refund","","",""
"tax_sale_vat_exemption","VAT EXEMPTION","VAT Exemption","VAT EXEMPTION","0.0","percent","sale","tax_group_exemption","base","invoice","+Untaxed sales of goods and services taxed VAT Exemption","","Không thuộc đối tượng chịu thuế GTGT"
"","","","","","","","","tax","invoice","","",""
"","","","","","","","","base","refund","-Untaxed sales of goods and services taxed VAT Exemption","",""
"","","","","","","","","tax","refund","","",""

```

## File: data\template\account.tax.group-vn.csv

```csv
"id","name","country_id","name@vi_VN"
"tax_group_0","VAT 0%","base.vn","Thuế GTGT 0%"
"tax_group_5","VAT 5%","base.vn","Thuế GTGT 5%"
"tax_group_10","VAT 10%","base.vn","Thuế GTGT 10%"
"tax_group_exemption","VAT EXEMPTION","base.vn","Không thuộc đối tượng chịu thuế GTGT"

```

## File: i18n_extra\l10n_vn.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_vn
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 14.0\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2021-09-30 03:34+0000\n"
"PO-Revision-Date: 2021-09-30 03:34+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_vn
#: model:ir.model.fields.selection,name:l10n_vn.selection__res_partner_bank__proxy_type__atm_card
msgid "ATM Card Number"
msgstr ""

#. module: l10n_vn
#: model:ir.model,name:l10n_vn.model_account_chart_template
msgid "Account Chart Template"
msgstr ""

#. module: l10n_vn
#: model:account.report.column,name:l10n_vn.tax_report_balance
msgid "Balance"
msgstr ""

#. module: l10n_vn
#: model:ir.model.fields.selection,name:l10n_vn.selection__res_partner_bank__proxy_type__bank_acc
msgid "Bank Account"
msgstr ""

#. module: l10n_vn
#: model:ir.model,name:l10n_vn.model_res_partner_bank
msgid "Bank Accounts"
msgstr ""

#. module: l10n_vn
#. odoo-python
#: code:addons/l10n_vn/models/res_bank.py:0
#, python-format
msgid ""
"Can't generate a Vietnamese QR banking code with a currency other than VND."
msgstr ""

#. module: l10n_vn
#: model_terms:ir.ui.view,arch_db:l10n_vn.view_partner_bank_form_inherit_account
msgid "Documentation"
msgstr ""

#. module: l10n_vn
#: model:ir.model.fields.selection,name:l10n_vn.selection__res_partner_bank__proxy_type__merchant_id
msgid "Merchant ID"
msgstr ""

#. module: l10n_vn
#. odoo-python
#: code:addons/l10n_vn/models/res_bank.py:0
#, python-format
msgid ""
"Missing Bank Identifier Code.\n"
"Please configure the Bank Identifier Code inside the bank settings."
msgstr ""

#. module: l10n_vn
#: model:ir.model.fields.selection,name:l10n_vn.selection__res_partner_bank__proxy_type__payment_service
msgid "Payment Service"
msgstr ""

#. module: l10n_vn
#: model:ir.model.fields,field_description:l10n_vn.field_account_setup_bank_manual_config__proxy_type
#: model:ir.model.fields,field_description:l10n_vn.field_res_partner_bank__proxy_type
msgid "Proxy Type"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_vn
msgid "Purchase of Goods and Services"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_vn
msgid "Sales of Goods and Services"
msgstr ""

#. module: l10n_vn
#: model:account.report,name:l10n_vn.tax_report
msgid "Tax Report"
msgstr ""

#. module: l10n_vn
#. odoo-python
#: code:addons/l10n_vn/models/res_bank.py:0
#, python-format
msgid ""
"The QR Code Type must be either Merchant ID, ATM Card Number or Bank Account"
" to generate a Vietnam Bank QR code for account number %s."
msgstr ""

#. module: l10n_vn
#. odoo-python
#: code:addons/l10n_vn/models/res_bank.py:0
#, python-format
msgid ""
"The proxy type %s is not supported for Vietnamese partners. It must be "
"either Merchant ID, ATM Card Number or Bank Account"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_01_vn
msgid "Untaxed Purchase of Goods and Services"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_02_01_vn
msgid "Untaxed Purchase of Goods and Services taxed 0%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_03_02_01_vn
msgid "Untaxed Purchase of Goods and Services taxed 10%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_02_01_vn
msgid "Untaxed Purchase of Goods and Services taxed 5%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_04_02_01_vn
msgid "Untaxed Purchase of Goods and Services taxed VAT Exemption"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_02_vn
msgid "Untaxed Sales of Goods and Services"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_02_02_vn
msgid "Untaxed sales of goods and services taxed 0%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_03_02_02_vn
msgid "Untaxed sales of goods and services taxed 10%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_02_02_vn
msgid "Untaxed sales of goods and services taxed 5%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_04_02_02_vn
msgid "Untaxed sales of goods and services taxed VAT Exemption"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_01_vn
msgid "VAT on purchase of goods and services"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_01_01_vn
msgid "VAT on purchase of goods and services 0%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_03_01_01_vn
msgid "VAT on purchase of goods and services 10%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_01_01_vn
msgid "VAT on purchase of goods and services 5%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_02_vn
msgid "VAT on sales of goods and services"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_01_01_02_vn
msgid "VAT on sales of goods and services 0%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_03_01_02_vn
msgid "VAT on sales of goods and services 10%"
msgstr ""

#. module: l10n_vn
#: model:account.report.line,name:l10n_vn.account_tax_report_line_02_01_02_vn
msgid "VAT on sales of goods and services 5%"
msgstr ""

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

FIXED_ACCOUNTS_TYPE = {
    'asset_prepayments': ['242'],
    'expense_depreciation': ['6274', '6414', '6424'],
}


def _fix_accounts_type(env):
    for correct_account_type, accounts_code in FIXED_ACCOUNTS_TYPE.items():
        domains_per_company = []
        for company in env['res.company'].with_context(active_test=False).search([('chart_template', '=', 'vn')]):
            doamin = expression.AND([
                [('company_id', '=', company.id), ('account_type', '!=', correct_account_type)],
                expression.OR([
                    [('code', 'like', f'{code}%')] for code in accounts_code
                ])
            ])
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
        bank_vn.display_qr_setting = self.env.company.qr_code
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
            'code_digits': '0',
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
                'anglo_saxon_accounting': False,
                'account_fiscal_country_id': 'base.vn',
                'bank_account_code_prefix': '112',
                'cash_account_code_prefix': '111',
                'transfer_account_code_prefix': '113',
                'account_default_pos_receivable_account_id': 'chart131',
                'income_currency_exchange_account_id': 'chart515',
                'expense_currency_exchange_account_id': 'chart635',
                'account_journal_early_pay_discount_loss_account_id': 'chart9993',
                'account_journal_early_pay_discount_gain_account_id': 'chart9994',
                'account_sale_tax_id': 'tax_sale_vat10',
                'account_purchase_tax_id': 'tax_purchase_vat10',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import res_bank
from . import template_vn

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
                    <a href='https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/vietnam.html' target='_blank'>Documentation</a>
                </p>
            </field>
        </field>
    </record>

</odoo>

```

