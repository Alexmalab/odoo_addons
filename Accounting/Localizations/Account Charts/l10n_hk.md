# Odoo Module: l10n_hk

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
    'name': 'Hong Kong - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/hong_kong.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['hk'],
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': ' This is the base module to manage chart of accounting and localization for Hong Kong ',
    'depends': [
        'account_qr_code_emv',
    ],
    'data': [
        'data/account_chart_template_data.xml',
        'views/res_bank_views.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="base.CNY" model="res.currency">
        <field name="active" eval="True"/>
    </record>

    <!-- Account Chart template -->
    </odoo>

```

## File: data\template\account.account-hk.csv

```csv
"id","name","code","account_type","tag_ids","reconcile"
"l10n_hk_11","Fixed Assets","11","asset_fixed","","False"
"l10n_hk_1110","Furniture and Fixtures","1110","asset_non_current","","False"
"l10n_hk_1130","Equipment","1130","asset_non_current","","False"
"l10n_hk_1140","Decoration","1140","asset_non_current","","False"
"l10n_hk_1160","Investments","1160","asset_non_current","","False"
"l10n_hk_12","Current Assets","12","asset_current","","False"
"l10n_hk_1240","Account Receivable","1240","asset_receivable","","True"
"l10n_hk_1241","Utility & Rental Deposit","1241","asset_current","","True"
"l10n_hk_1242","Supplier Prepayments","1242","asset_current","","False"
"l10n_hk_1243","Account Receivable (PoS)","1243","asset_receivable","","True"
"l10n_hk_1245","Sundry Deposits","1245","asset_current","","False"
"l10n_hk_1246","Other Receivable","1246","asset_receivable","","True"
"l10n_hk_1250","Stock Interim Account (Received)","1250","asset_current","","False"
"l10n_hk_1260","Stock Interim Account (Delivered)","1260","asset_current","","False"
"l10n_hk_1270","Stock Valuation Account","1270","asset_current","","False"
"l10n_hk_21","Non-current Liabilities","21","liability_non_current","","False"
"l10n_hk_22","Current Liabilities","22","liability_current","","False"
"l10n_hk_2210","Accruals","2210","liability_current","","False"
"l10n_hk_221001","MPF (Employer)","221001","liability_current","","True"
"l10n_hk_221002","MPF (Employee)","221002","liability_current","","True"
"l10n_hk_2211","Account Payable","2211","liability_payable","","True"
"l10n_hk_2212","Receipt in Advance (Customer Prepayments)","2212","liability_current","","False"
"l10n_hk_2214","Provision for Taxation","2214","liability_current","","False"
"l10n_hk_2215","Proposed Dividend","2215","liability_current","","False"
"l10n_hk_2216","Other Payable","2216","liability_payable","","True"
"l10n_hk_31","Paid Capital","31","liability_current","","False"
"l10n_hk_32","Accumulated Profit & Loss","32","liability_current","","False"
"l10n_hk_33","Profit & Loss Account","33","liability_current","","False"
"l10n_hk_34","Short-term Borrowing","34","liability_current","","False"
"l10n_hk_41","Trade Income","41","income","account.account_tag_operating","False"
"l10n_hk_42","Other Income and Gains","42","income","account.account_tag_operating","False"
"l10n_hk_4210","Sundry Income","4210","income","account.account_tag_operating","False"
"l10n_hk_4220","Exchange Adjustment","4220","income","account.account_tag_operating","False"
"l10n_hk_4230","Bank Interest Income","4230","income","account.account_tag_operating","False"
"l10n_hk_4240","Foreign Exchange Gain","4240","income","account.account_tag_operating","False"
"l10n_hk_4250","Cash Discount Gain","4250","income_other","account.account_tag_operating","False"
"l10n_hk_51","Costs","51","expense","account.account_tag_operating","False"
"l10n_hk_5101","Trade Costs","5101","expense","account.account_tag_operating","False"
"l10n_hk_5105","Misc. Costs","5105","expense","account.account_tag_operating","False"
"l10n_hk_5106","Costs of Transportation","5106","expense","account.account_tag_operating","False"
"l10n_hk_5111","Declaration Fees","5111","expense","account.account_tag_operating","False"
"l10n_hk_5112","Packing Fees","5112","expense","account.account_tag_operating","False"
"l10n_hk_52","Expenses","52","expense","account.account_tag_operating","False"
"l10n_hk_5201","Bank Charges","5201","expense","account.account_tag_operating","False"
"l10n_hk_5202","Entertainment","5202","expense","account.account_tag_operating","False"
"l10n_hk_5203","Electricity & Water Fees","5203","expense","account.account_tag_operating","False"
"l10n_hk_5205","Postage & Stamps","5205","expense","account.account_tag_operating","False"
"l10n_hk_5206","Printing & Stationery","5206","expense","account.account_tag_operating","False"
"l10n_hk_5207","Rent & Rates","5207","expense","account.account_tag_operating","False"
"l10n_hk_5208","Sundry Expenses","5208","expense","account.account_tag_operating","False"
"l10n_hk_5209","Telecommunication Expenses","5209","expense","account.account_tag_operating","False"
"l10n_hk_5210","Traffic Fees","5210","expense","account.account_tag_operating","False"
"l10n_hk_5211","IT Expenses","5211","expense","account.account_tag_operating","False"
"l10n_hk_5214","Insurance","5214","expense","account.account_tag_operating","False"
"l10n_hk_5215","Sales Commission","5215","expense","account.account_tag_operating","False"
"l10n_hk_5216","Overseas Traveling","5216","expense","account.account_tag_operating","False"
"l10n_hk_5217","MPF Contribution","5217","expense","account.account_tag_operating","False"
"l10n_hk_5218","Wages & Salaries","5218","expense","account.account_tag_operating","False"
"l10n_hk_5219","Bonus Payment","5219","expense","account.account_tag_operating","False"
"l10n_hk_5221","Taxation","5221","expense","account.account_tag_operating","False"
"l10n_hk_5222","Local Delivery","5222","expense","account.account_tag_operating","False"
"l10n_hk_5223","Management Fees","5223","expense","account.account_tag_operating","False"
"l10n_hk_5224","Depreciation","5224","expense","account.account_tag_operating","False"
"l10n_hk_5225","Audit Fees","5225","expense","account.account_tag_operating","False"
"l10n_hk_5226","Bad Debts","5226","expense","account.account_tag_operating","False"
"l10n_hk_5228","Legal & Professional Fees","5228","expense","account.account_tag_operating","False"
"l10n_hk_5229","Dividend","5229","expense","account.account_tag_operating","False"
"l10n_hk_5231","Disposal","5231","expense","account.account_tag_operating","False"
"l10n_hk_5234","Repair and Maintenance","5234","expense","account.account_tag_operating","False"
"l10n_hk_5235","Advertising","5235","expense","account.account_tag_operating","False"
"l10n_hk_5240","Foreign Exchange Loss","5240","expense","account.account_tag_operating","False"
"l10n_hk_5250","Cash Discount Loss","5250","expense","account.account_tag_operating","False"

```

## File: models\res_bank.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.tools import single_email_re


class ResPartnerBank(models.Model):
    _inherit = 'res.partner.bank'

    proxy_type = fields.Selection(selection_add=[('id', "FPS ID"), ('mobile', "Mobile Number"), ('email', "Email Address")],
                                  ondelete={'id': 'set default', 'mobile': 'set default', 'email': 'set default'})

    @api.constrains('proxy_type', 'proxy_value', 'partner_id')
    def _check_hk_proxy(self):
        auto_mobn_re = re.compile(r"^[+]\d{1,3}-\d{6,12}$")
        for bank in self.filtered(lambda b: b.country_code == 'HK'):
            if bank.proxy_type not in ['id', 'mobile', 'email', 'none', False]:
                raise ValidationError(_("The FPS Type must be either ID, Mobile or Email to generate a FPS QR code for account number %s.", bank.acc_number))
            if bank.proxy_type == 'id' and (not bank.proxy_value or len(bank.proxy_value) not in [7, 9]):
                raise ValidationError(_("Invalid FPS ID! Please enter a valid FPS ID with length 7 or 9 for account number %s.", bank.acc_number))
            if bank.proxy_type == 'mobile' and (not bank.proxy_value or not auto_mobn_re.match(bank.proxy_value)):
                raise ValidationError(_("Invalid Mobile! Please enter a valid mobile number with format +852-67891234 for account number %s.", bank.acc_number))
            if bank.proxy_type == 'email' and (not bank.proxy_value or not single_email_re.match(bank.proxy_value)):
                raise ValidationError(_("Invalid Email! Please enter a valid email address for account number %s.", bank.acc_number))

    @api.depends('country_code')
    def _compute_display_qr_setting(self):
        bank_hk = self.filtered(lambda b: b.country_code == 'HK')
        bank_hk.display_qr_setting = self.env.company.qr_code
        super(ResPartnerBank, self - bank_hk)._compute_display_qr_setting()

    # Follow the documentation of FPS QR Code Standard [1]
    # [1]: https://www.hkma.gov.hk/media/eng/doc/key-functions/financial-infrastructure/infrastructure/retail-payment-initiatives/Common_QR_Code_Specification.pdf
    def _get_merchant_account_info(self):
        if self.country_code == 'HK':
            fps_type_mapping = {
                'id': 2,
                'mobile': 3,
                'email': 4,
            }
            fps_type = fps_type_mapping[self.proxy_type]
            merchant_account_vals = [
                (0, 'hk.com.hkicl'),                                 # GUID
                (fps_type, self.proxy_value),                        # Proxy Type and Proxy Value
            ]
            merchant_account_info = ''.join([self._serialize(*val) for val in merchant_account_vals])
            return (26, merchant_account_info)
        return super()._get_merchant_account_info()

    def _get_additional_data_field(self, comment):
        if self.country_code == 'HK':
            return self._serialize(5, comment)
        return super()._get_additional_data_field(comment)

    def _get_error_messages_for_qr(self, qr_method, debtor_partner, currency):
        if qr_method == 'emv_qr' and self.country_code == 'HK':
            if currency.name not in ['HKD', 'CNY']:
                return _("Can't generate a FPS QR code with a currency other than HKD or CNY.")
            return None

        return super()._get_error_messages_for_qr(qr_method, debtor_partner, currency)

    def _check_for_qr_code_errors(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        if qr_method == 'emv_qr' and self.country_code == 'HK' and self.proxy_type not in ['id', 'mobile', 'email']:
            return _("The FPS Type must be either ID, Mobile or Email to generate a FPS QR code.")

        return super()._check_for_qr_code_errors(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

```

## File: models\template_hk.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('hk')
    def _get_hk_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_hk_1240',
            'property_account_payable_id': 'l10n_hk_2211',
            'property_account_income_categ_id': 'l10n_hk_41',
            'property_account_expense_categ_id': 'l10n_hk_51',
            'code_digits': '6',
        }

    @template('hk', 'res.company')
    def _get_hk_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.hk',
                'bank_account_code_prefix': '1200',
                'cash_account_code_prefix': '1210',
                'transfer_account_code_prefix': '111220',
                'account_default_pos_receivable_account_id': 'l10n_hk_1243',
                'income_currency_exchange_account_id': 'l10n_hk_4240',
                'expense_currency_exchange_account_id': 'l10n_hk_5240',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_hk_5250',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_hk_4250',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import res_bank
from . import template_hk

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
                <p invisible="country_code != 'HK'">
                    <a href='https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/hong_kong.html' target='_blank'>Documentation</a>
                </p>
            </field>
        </field>
    </record>

</odoo>

```

