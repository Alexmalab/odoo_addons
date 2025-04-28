# Odoo Module: l10n_iq

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": "Iraq - Accounting",
    "countries": ["iq"],
    "description": """
This is the base module to manage the accounting chart for Iraq in Odoo.
==============================================================================
Iraq accounting basic charts and localization.
Activates:
- Chart of accounts
- Taxes
    """,
    "category": "Accounting/Localizations/Account Charts",
    "version": "1.0",
    "depends": [
        "account",
    ],
    "data": ["data/res.country.state.csv"],
    "demo": ["demo/demo_company.xml"],
    "license": "LGPL-3",
}

```

## File: data\res.country.state.csv

```csv
"id","country_id/id","name","code"
state_iq_01,base.iq,"Al Anbar","01"
state_iq_01_ar,base.iq,"الأنبار","02"
state_iq_02,base.iq,"Al Basrah","03"
state_iq_02_ar,base.iq,"البصرة","04"
state_iq_03,base.iq,"Al Muthanna","05"
state_iq_03_ar,base.iq,"المثنى","06"
state_iq_04,base.iq,"Al Qādisiyyah","07"
state_iq_04_ar,base.iq,"القادسية","08"
state_iq_05,base.iq,"Sulaymaniyah","09"
state_iq_05_ar,base.iq,"السليمانية","10"
state_iq_06,base.iq,"Babil","11"
state_iq_06_ar,base.iq,"بابل","12"
state_iq_07,base.iq,"Baghdad","13"
state_iq_07_ar,base.iq,"بغداد","14"
state_iq_08,base.iq,"Duhok","15"
state_iq_08_ar,base.iq,"دهوك","16"
state_iq_09,base.iq,"Dhi Qar","17"
state_iq_09_ar,base.iq,"ذي قار","18"
state_iq_10,base.iq,"Diyala","19"
state_iq_10_ar,base.iq,"ديالى","20"
state_iq_11,base.iq,"Erbil","21"
state_iq_11_ar,base.iq,"أربيل","22"
state_iq_12,base.iq,"Karbala'","23"
state_iq_12_ar,base.iq,"كربلاء","24"
state_iq_13,base.iq,"Kirkuk","25"
state_iq_13_ar,base.iq,"كركوك","26"
state_iq_14,base.iq,"Maysan","27"
state_iq_14_ar,base.iq,"ميسان","28"
state_iq_15,base.iq,"Ninawa","29"
state_iq_15_ar,base.iq,"نينوى","30"
state_iq_16,base.iq,"Wasit","31"
state_iq_16_ar,base.iq,"واسط","32"
state_iq_17,base.iq,"Najaf","33"
state_iq_17_ar,base.iq,"النجف","34"
state_iq_18,base.iq,"Salah Al Din","35"
state_iq_18_ar,base.iq,"صلاح الدين","36"

```

## File: data\template\account.account-iq.csv

```csv
"id","name","code","account_type","reconcile","name@ar_001"
"iq_account_100102","Bank Suspense Account","100102","asset_current","False","حساب التعليق البنكي"
"iq_account_100103","Outstanding Receipts","100103","asset_current","False","الإيصالات المستحقة"
"iq_account_100104","Outstanding Payments","100104","asset_current","False","المدفوعات المستحقة"
"iq_account_100106","Credit cards","100106","asset_current","False","البطاقات الائتمانية"
"iq_account_100107","Post Dated Cheques Received","100107","asset_current","False","الشيكات مؤجلة الصرف المستلمة"
"iq_account_100201","Accounts Receivable","100201","asset_receivable","True","الحسابات المدينة"
"iq_account_100202","Accounts Receivable (PoS)","100202","asset_receivable","True","الحسابات المدينة (نقطة البيع)"
"iq_account_100203","Other Receivable","100203","asset_current","False","المستحقات الأخرى"
"iq_account_100301","Deposit - Office Rent","100301","asset_current","False","إيداع - إيجار المكتب"
"iq_account_100302","Deposits - Customs","100302","asset_current","False","الإيداعات - الجمارك"
"iq_account_100303","Deposit to Immigration (Visa)","100303","asset_current","False","إيداع للهجرة (فيزا)"
"iq_account_100304","Deposit Others","100304","asset_current","False","إيداع آخر"
"iq_account_100401","Prepaid Medical Insurance","100401","asset_current","False","التأمين الصحي مسبق الدفع"
"iq_account_100402","Prepaid Life Insurance","100402","asset_current","False","التأمين على الحياة مسبق الدفع"
"iq_account_100403","Prepaid Office Rent","100403","asset_current","False","إيجار المكتب مسبق الدفع"
"iq_account_100404","Prepaid Other Insurance","100404","asset_current","False","التأمينات الأخرى مسبقة الدفع"
"iq_account_100405","Prepaid License Fees","100405","asset_current","False","رسوم الرخصة مسبقة الدفع"
"iq_account_100406","Prepaid Maintenance","100406","asset_current","False","الصيانة مسبقة الدفع"
"iq_account_100407","Prepaid Employees Housing","100407","asset_current","False","سكن الموظفين مسبق الدفع"
"iq_account_100408","Prepaid Schooling Fees","100408","asset_current","False","الرسوم الدراسية مسبقة الدفع"
"iq_account_100409","Prepaid Consultancy Fees","100409","asset_current","False","الرسوم الاستشارية مسبقة الدفع"
"iq_account_100410","Prepaid Legal Fees","100410","asset_current","False","الرسوم القانونية مسبقة الدفع"
"iq_account_100411","Prepaid Sponsorship Fees","100411","asset_current","False","رسوم الكفالة مسبقة الدفع"
"iq_account_100412","Prepaid Advertisement Expenses","100412","asset_current","False","نفقات الإعلان مسبقة الدفع"
"iq_account_100413","Prepaid Bank Guarantee","100413","asset_current","False","ضمان البنك مسبق الدفع"
"iq_account_100414","Prepaid Finance charge for Loans","100414","asset_current","False","رسوم التمويل مسبقة الدفع للقروض"
"iq_account_100415","Other Prepayments","100415","asset_current","False","المدفوعات المسبقة الأخرى"
"iq_account_100416","Prepaid Expenses","100416","asset_current","False","المصروفات المدفوعة مقدما"
"iq_account_100501","Handling Difference in Inventory","100501","asset_current","False","التعامل مع الفرق في المخزون"
"iq_account_100502","Inventory Valuation","100502","asset_current","False","تقييم المخزون"
"iq_account_100503","Stock Incoming","100503","asset_current","False","الأسهم الواردة"
"iq_account_100504","Stock Outgoing","100504","asset_current","False","الأسهم الصادرة"
"iq_account_100505","Work in Progress (Inventory)","100505","asset_current","False","العمل قيد التنفيذ (المخزون)"
"iq_account_100601","Accumulated Depreciation of Motor Vehicles","100601","asset_fixed","False","حساب الإهلاك للمركبات"
"iq_account_100602","Amortisation on Leasehold Improvement","100602","asset_fixed","False","الاستهلاك عند تحسين العقارات المستأجرة"
"iq_account_100603","Leasehold Improvement","100603","asset_fixed","False","تحسين العقارات المستأجرة"
"iq_account_100604","Furniture and Equipment","100604","asset_fixed","False","الأثاث والمعدات"
"iq_account_100605","Computer Hardware & Software","100605","asset_fixed","False","أجهزة وبرامج الحاسوب"
"iq_account_100606","Accumulated Depreciation of Furniture & Office Equipment","100606","asset_fixed","False","حساب الإهلاك للأثاث والأدوات المكتبية"
"iq_account_100607","Accumulated Depreciation of Computer Hardware & Software","100607","asset_fixed","False","حساب الإهلاك لبرامج وأجهزة الحاسوب"
"iq_account_100701","Registration of Trademarks","100701","asset_current","False","تسجيل العلامات التجارية"
"iq_account_100801","Right of use Asset (IFRS 16)","100801","asset_fixed","False","حق استخدام الأصل (IFRS 16)"
"iq_account_100802","Accumulated Depreciation Right of use Asset (IFRS 16)","100802","asset_fixed","False","حق استخدام الأصل للإهلاك المتراكم (IFRS 16)"
"iq_account_201001","Withholding Tax Payable","201001","liability_current","False","ضريبة الاستقطاع المستحقة الدفع"
"iq_account_200101","Payables","200101","liability_payable","True","المبالغ مستحقة الدفع"
"iq_account_200102","Trade Payables","200102","liability_payable","True","الذمم التجارية الدائنة"
"iq_account_200103","Employees Payables","200103","liability_payable","True","المبالغ المستحقة للموظفين"
"iq_account_200104","Credit Notes to Customers","200104","liability_current","False","الإشعارات الدائنة للعملاء"
"iq_account_200201","Accrued - Salaries","200201","liability_current","False","مستحق - المرتبات"
"iq_account_200202","Accrued - Commissions","200202","liability_current","False","مستحق - العمولات"
"iq_account_200203","Accrued - Staff Bonus","200203","liability_current","False","مكافآت الموظفين المستحقة"
"iq_account_200204","Accrued Other Personnel Cost","200204","liability_current","False","تكاليف الموظفين الآخرين المستحقة"
"iq_account_200205","Accrued - Sponsorship","200205","liability_current","False","مستحق - الكفالة"
"iq_account_200301","Accrued - Utilities","200301","liability_current","False","مستحق - المرافق"
"iq_account_200302","Accrued - Telephone","200302","liability_current","False","مستحق - الهاتف"
"iq_account_200303","Accrued - Audit Fees","200303","liability_current","False","مستحق - رسوم التدقيق"
"iq_account_200304","Accrued - Office Rent","200304","liability_current","False","مستحق - إيجار المكتب"
"iq_account_200305","Accrued Others","200305","liability_current","False","المستحقات الأخرى"
"iq_account_200306","Accrued Iraq Customs","200306","liability_current","False","جمارك العراق المستحقة" 
"iq_account_200401","Deferred income","200401","liability_current","False","الدخل المؤجل"
"iq_account_200501","Leave Tickets Provision","200501","liability_non_current","False","حكم تذاكر الطيران"
"iq_account_200502","Leave Days Provision","200502","liability_non_current","False","حكم أيام الإجازة"
"iq_account_200503","End of Service Provision","200503","liability_non_current","False","حكم نهاية الخدمة"
"iq_account_200504","Income Tax Provision","200504","liability_non_current","False","حكم ضريبة الدخل"
"iq_account_200901","VAT Input","200901","asset_current","False","مدخلات ضريبة القيمة المضافة"
"iq_account_200902","VAT Output","200902","liability_current","False","مخرجات ضريبة القيمة المضافة"
"iq_account_200903","VAT Receivable","200903","asset_non_current","False","ضريبة القيمة المضافة مستحقة الدفع"
"iq_account_200904","VAT Payable","200904","liability_non_current","False","ضريبة القيمة المضافة المستحقة"
"iq_account_200905","Tax Payable","200905","liability_current","False","الضريبة المستحقة"
"iq_account_200906","Tax Receivable","200906","asset_current","False","ضريبة مستحقة القبض"
"iq_account_300100","Retained Earnings","300100","equity","False","الأرباح المستبقاة"
"iq_account_300101","Undistributed Profits/Losses","300101","equity_unaffected","False","الأرباح/الخسائر غير الموزعة"
"iq_account_400100","Income Clearing Account","400100","income","False","حساب مقاصة الدخل"
"iq_account_400101","Sales Account","400101","income","False","حساب المبيعات"
"iq_account_400102","Sales of I/C","400102","income","False","المبيعات بين الشركات التابعة"
"iq_account_400103","Sales from Other Region","400103","income","False","المبيعات من منطقة أخرى"
"iq_account_400104","Management Consultancy Fees","400104","income","False","رسوم الاستشارة الإدارية"
"iq_account_400105","Advertising Income","400105","income","False","دخل الإعلان"
"iq_account_400201","Other Income","400201","income_other","False","دخل آخر"
"iq_account_400301","Gain on Difference on Exchange","400301","income_other","False","أرباح فرق صرف العملة"
"iq_account_400302","Cash Difference Gain","400302","income_other","False","أرباح فرق النقد"
"iq_account_400303","Excess In Till","400303","income_other","False","الفائض في صندوق النقود"
"iq_account_400304","Cash Discount Gain","400304","income_other","False","مكاسب الخصم النقدي"
"iq_account_500101","Cost of Goods Sold in Trading","500101","expense_direct_cost","False","تكاليف البضائع المباعة في التجارة"
"iq_account_500102","Cost Of Goods Sold I/C Sales","500102","expense_direct_cost","False","تكاليف البضائع المباعة - المبيعات بين الشركات التابعة"
"iq_account_500200","Expense Clearing Account","500200","expense","False","حساب مقاصة النفقات"
"iq_account_500201","Medical Insurance","500201","expense","False","التأمين الصحي"
"iq_account_500202","End of Service Indemnity","500202","expense","False","تعويض نهاية الخدمة"
"iq_account_500203","Sponsorship Fees","500203","expense","False","رسوم الكفالة"
"iq_account_500301","Basic Salary","500301","expense","False","الراتب الأساسي"
"iq_account_500302","Housing Allowance","500302","expense","False","بدل السكن"
"iq_account_500303","Transportation Allowance","500303","expense","False","بدل المواصلات"
"iq_account_500304","Leave Ticket","500304","expense","False","تذكرة الطيران"
"iq_account_500305","Leave Salary","500305","expense","False","راتب الإجازة"
"iq_account_500306","Sales Commission","500306","expense","False","عمولة المبيعات"
"iq_account_500307","Visa Expenses","500307","expense","False","نفقات الفيزا"
"iq_account_500308","Staff Other Allowances","500308","expense","False","نفقات الموظفين الأخرى"
"iq_account_500309","Air tickets","500309","expense","False","تذاكر الطيران"
"iq_account_500401","Office Rent","500401","expense","False","إيجار المكتب"
"iq_account_500402","Warehouse Rent","500402","expense","False","إيجار المستودع"
"iq_account_500403","Water & Electricity","500403","expense","False","الماء والكهرباء"
"iq_account_500404","Other Utility Charges","500404","expense","False","رسوم المرافق الأخرى"
"iq_account_500501","Audit Fees","500501","expense","False","رسوم التدقيق"
"iq_account_500502","Legal fees","500502","expense","False","الرسوم القانونية"
"iq_account_500503","Trade License Fees","500503","expense","False","رسوم الرخصة التجارية"
"iq_account_500504","Others - Professional Fees","500504","expense","False","غير ذلك - الرسوم المهنية"
"iq_account_500505","Insurance","500505","expense","False","التأمين"
"iq_account_500506","Previous Year Adjustments Account","500506","expense","False","حساب تعديلات العام الماضي"
"iq_account_500601","Credit Card Charges","500601","expense","False","رسوم البطاقة الائتمانية"
"iq_account_500602","Other Bank Charges","500602","expense","False","الرسوم البنكية الأخرى"
"iq_account_500603","Bank Finance & Loan Charges","500603","expense","False","رسوم القروض والتمويل البنكي"
"iq_account_500651","Income Tax Expense","500651","expense","False","نفقات ضريبة الدخل"
"iq_account_500701","Other - Advertising Expenses","500701","expense","False","غير ذلك - نفقات الإعلان"
"iq_account_500702","Training","500702","expense","False","التدريب"
"iq_account_500703","Consultancy Fees","500703","expense","False","الرسوم الاستشارية"
"iq_account_500801","Amortisation on Leasehold Improvement","500801","expense_depreciation","False","الاستهلاك عند تحسين العقارات المستأجرة"
"iq_account_500802","Vehicle Expenses","500802","expense_depreciation","False","نفقات المركبات"
"iq_account_500803","Depreciation of Motor Vehicles","500803","expense_depreciation","False","إهلاك المركبات"
"iq_account_500804","Depreciation of Furniture & Office Equipment","500804","expense_depreciation","False","إهلاك الأثاث والمعدات المكتبية"
"iq_account_500805","Depreciation of Computer Hard & Soft","500805","expense_depreciation","False","إهلاك أجهزة وبرامج الحاسوب"
"iq_account_500851","Depreciation on Right of use Asset (IFRS 16)","500851","expense_depreciation","False","إهلاك حق استخدام الأصل (IFRS 16)"
"iq_account_500901","Loss on Fixed Assets Disposal","500901","expense","False","خسائر التصرف في الأصول الثابتة"
"iq_account_500902","Cash Shortage","500902","expense","False","القصور النقدي"
"iq_account_500903","Loss on Difference on Exchange","500903","expense","False","حسائر فرق صرف العملة"
"iq_account_500904","Write Off Receivables & Payables","500904","expense","False","شطب الحسابات المدينة والدائنة"
"iq_account_500905","Write Off Inventory","500905","expense","False","شطب المخزون"
"iq_account_500906","Others - Provision & Write Off","500906","expense","False","غير ذلك - المَحافظ والتعديلات"
"iq_account_500907","Others","500907","expense","False","غير ذلك"
"iq_account_500908","Other Non-Operating Expenses","500908","expense","False","النفقات الأخرى غير التشغيلية"
"iq_account_500909","Cash Difference Loss","500909","expense","False","خسائر فريق النقد"
"iq_account_501101","Telephone","501101","expense","False","الهاتف"
"iq_account_501102","Others - Communication","501102","expense","False","غير ذلك - التواصل"
"iq_account_501103","Maintenance","501103","expense","False","الصيانة"
"iq_account_501104","Security & Guard","501104","expense","False","الأمن والحراسة"
"iq_account_501105","Cleaning","501105","expense","False","التنظيف"
"iq_account_501106","Others - Office Various Expenses","501106","expense","False","غير ذلك - نفقات المكتب المختلفة"
"iq_account_501107","Cash Discount Loss","501107","expense","False","خسارة الخصم النقدي"

```

## File: data\template\account.group-iq.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@ar_001"
"iq_group_01","1000","1000","Liquidity","السيولة"
"iq_group_02","1001","1001","Liquidity","السيولة"
"iq_group_03","1002","1002","Receivables","الحسابات المدينة"
"iq_group_04","1003","1003","Deposits","الإيداعات"
"iq_group_05","1004","1004","Prepaid Expenses","النفقات مسبقة الدفع"
"iq_group_06","1005","1005","Inventory","المخزون"
"iq_group_07","1006","1006","Company Assets & Depreciation","أصول الشركة والإهلاك"
"iq_group_08","1007","1007","Licensing and Copyrights","الترخيص وحقوق النشر"
"iq_group_09","1008","1008","IFRS16 Assets & Depreciation","أصول IFRS16 والإهلاك"
"iq_group_10","1009","1009","Liquidity","السيولة"
"iq_group_11","2001","2001","Payables","الحسابات الدائنة"
"iq_group_12","2002","2002","Accrued Employee Expenses","نفقات الموظفين المستحقة"
"iq_group_13","2003","2003","Accrued Expenses","النفقات المستحقة"
"iq_group_14","2004","2004","Deferrals","التأجيلات"
"iq_group_15","2005","2005","Provisions","الأحكام"
"iq_group_16","2009","2009","VAT","ضريبة القيمة المضافة"
"iq_group_17","2010","2010","Withholding","الاستقطاع"
"iq_group_18","4001","4001","Operating Income","الإيرادات التشغيلية"
"iq_group_19","4002","4002","Non-Operating Income","الإيرادات غير التشغيلية"
"iq_group_20","4003","4003","Other gains & losses - Other Income","المكاسب والخسائر الأخرى - إيرادات أخرى"
"iq_group_21","5001","5001","Cost of Sales","تكلفة المبيعات"
"iq_group_22","5002","5002","Employees Expenses","مصروفات الموظفين"
"iq_group_23","5003","5003","Payroll Expenses","مصروفات الرواتب"
"iq_group_24","5004","5004","Office and Location Expenses","نفقات المكتب والموقع"
"iq_group_25","5005","5005","Company Expenses","مصروفات الشركة"
"iq_group_26","500600","500649","Finance Expenses","المصروفات المالية"
"iq_group_27","500650","500699","Income Tax","ضريبة الدخل"
"iq_group_28","5007","5007","Misc. Company Expenses","متفرقات. مصروفات الشركة"
"iq_group_29","500800","500849","Assets Depreciation Expenses","مصروفات استهلاك الأصول"
"iq_group_30","500851","500899","IFRS16 Depreciation","IFRS16 الإهلاك"
"iq_group_31","5009","5009","Other gains & losses - Expenses","الأرباح والخسائر الأخرى - المصروفات"
"iq_group_32","5011","5011","Misc. Office Expenses","متفرقات. نفقات مكتبية"

```

## File: data\template\account.tax-iq.csv

```csv
"id","name","type_tax_use","amount","amount_type","description","invoice_label","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","name@ar_001"
"iq_standard_sale_300_alcohol","300%","sale","300.0","percent","Imposed on alcohol and tobacco (cigarettes)","300%","iq_tax_group_sales","base","invoice","","300%. ضريبة على الكحول والتبغ"
"","","","","","","","","tax","invoice","iq_account_200902",""
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","iq_account_200902",""
"iq_standard_sale_15_cars_travel","15%","sale","15.0","percent","Imposed cars and travel tickets","15%","iq_tax_group_sales","base","invoice","","15%. ضريبة على السيارات وتذاكر السفر"
"","","","","","","","","tax","invoice","iq_account_200902",""
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","iq_account_200902",""
"iq_standard_sale_20_mobile_internet","20%","sale","20.0","percent","Imposed on mobile recharge cards and internet","20%","iq_tax_group_sales","base","invoice","","20%. ضريبة على بطاقات إعادة شحن الهاتف والإنترنت"
"","","","","","","","","tax","invoice","iq_account_200902",""
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","iq_account_200902",""
"iq_standard_purchase_3_3_WH","3.3% WH","purchase","-3.3","percent","For all Payments for contracted oil and gas companies","3.3% WH","iq_tax_group_withholding","base","invoice","","3.3%. ضريبة استقطاع للشركات النفطية والغاز"
"","","","","","","","","tax","invoice","iq_account_201001",""
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","iq_account_201001",""
"iq_standard_purchase_7_WH","7% WH","purchase","-7.0","percent","For all Payments for contracted oil and gas companies","7% WH","iq_tax_group_withholding","base","invoice","","7%. ضريبة استقطاع للشركات النفطية والغاز"
"","","","","","","","","tax","invoice","iq_account_201001",""
"","","","","","","","","base","refund","",""
"","","","","","","","","tax","refund","iq_account_201001",""

```

## File: data\template\account.tax.group-iq.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","name@ar_001"
"iq_tax_group_16","16% Tax","base.iq","iq_account_200906","iq_account_200905","10% ضريبة"
"iq_tax_group_10","10% Tax","base.iq","iq_account_200906","iq_account_200905","10% ضريبة"
"iq_tax_group_4","4% Tax","base.iq","iq_account_200906","iq_account_200905","4% ضريبة"
"iq_tax_group_0","0% Tax","base.iq","iq_account_200906","iq_account_200905","0% ضريبة"
"iq_tax_group_sales","Sales Tax","base.iq","iq_account_200903","iq_account_200904","ضريبة المبيعات"
"iq_tax_group_withholding","Withholding Tax","base.iq","iq_account_200903","iq_account_200904","ضريبة الاستقطاع"

```

## File: i18n_extra\l10n_iq.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_iq
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 16.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2023-03-28 07:33+0000\n"
"PO-Revision-Date: 2023-03-28 07:33+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"


#. module: l10n_iq
#: model:ir.model,name:l10n_iq.model_account_chart_template
msgid "Account Chart Template"
msgstr ""

```

## File: models\template_iq.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = "account.chart.template"

    @template("iq")
    def _get_iq_template_data(self):
        return {
            "property_account_receivable_id": "iq_account_100201",
            "property_account_payable_id": "iq_account_200101",
            "property_account_expense_categ_id": "iq_account_500101",
            "property_account_income_categ_id": "iq_account_400101",
            "property_account_expense_id": "iq_account_500101",
            "property_account_income_id": "iq_account_400101",
            "property_stock_valuation_account_id": "iq_account_100502",
            "property_stock_account_input_categ_id": "iq_account_100503",
            "property_stock_account_output_categ_id": "iq_account_100504",
            "property_stock_account_production_cost_id": "iq_account_100505",
            "code_digits": "6",
        }

    @template("iq", "res.company")
    def _get_iq_res_company(self):
        return {
            self.env.company.id: {
                "account_fiscal_country_id": "base.iq",
                "bank_account_code_prefix": "1000",
                "cash_account_code_prefix": "1009",
                "transfer_account_code_prefix": "1001",
                "account_default_pos_receivable_account_id": "iq_account_100202",
                "income_currency_exchange_account_id": "iq_account_400301",
                "expense_currency_exchange_account_id": "iq_account_500903",
                "account_journal_suspense_account_id": "iq_account_100102",
                "account_journal_early_pay_discount_loss_account_id": "iq_account_501107",
                "account_journal_early_pay_discount_gain_account_id": "iq_account_400304",
                "account_journal_payment_debit_account_id": "iq_account_100103",
                "account_journal_payment_credit_account_id": "iq_account_100104",
                "default_cash_difference_income_account_id": "iq_account_400302",
                "default_cash_difference_expense_account_id": "iq_account_500909",
                "deferred_expense_account_id": "iq_account_100416",
                "deferred_revenue_account_id": "iq_account_200401",
            },
        }

```

## File: models\__init__.py

```python
from . import template_iq

```

