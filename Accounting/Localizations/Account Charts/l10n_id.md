# Odoo Module: l10n_id

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models
from . import controllers

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Indonesian - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['id'],
    'version': '1.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the latest Indonesian Odoo localisation necessary to run Odoo accounting for SMEs with:
=================================================================================================
    - generic Indonesian chart of accounts
    - tax structure""",
    'author': 'vitraining.com',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/indonesia.html',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'auto_install': ['account'],
    'data': [
        'security/ir.model.access.csv',
        'data/account_tax_template_data.xml',
        'data/ir_cron.xml',
        'views/account_move_views.xml',
        'views/res_bank.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python

from odoo.addons.account.controllers.portal import PortalAccount
from odoo import http
from odoo.http import request


class Portal(PortalAccount):
    @http.route()
    def portal_my_invoice_detail(self, **kw):
        """ Override
        force QR code generation from QRIS to come only from portal"""
        request.update_context(is_online_qr=True)
        return super().portal_my_invoice_detail(**kw)

```

## File: controllers\__init__.py

```python
from . import portal

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ppn_tag" model="account.account.tag">
        <field name="name">PPN - 08</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.id"/>
    </record>
    </odoo>

```

## File: data\ir_cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="qris_fetch_cron" model="ir.cron">
        <field name="name">QRIS Fetch Status</field>
        <field name="model_id" ref="account.model_account_move"/>
        <field name="interval_number">1</field>
        <field name="interval_type">hours</field>
        <field name="user_id" ref="base.user_root"/>
        <field name="state">code</field>
        <field name="code">model._l10n_id_cron_update_payment_status()</field>
    </record>
</odoo>

```

## File: data\template\account.account-id.csv

```csv
"id","code","name","account_type","reconcile","name@id"
"l10n_id_11110001","11110001","Cash","asset_cash","False",""
"l10n_id_11110010","11110010","Petty Cash","asset_cash","False","Kas Kecil"
"l10n_id_11110020","11110020","Cash in Hand","asset_cash","False","Kas Belum Disetor"
"l10n_id_11120001","11120001","Bank Suspense","liability_current","True",""
"l10n_id_11120002","11120002","Outstanding Receipts","asset_current","True",""
"l10n_id_11120003","11120003","Outstanding Payments","asset_current","True",""
"l10n_id_11120004","11120004","Bank","asset_cash","False",""
"l10n_id_11210010","11210010","Account Receivable","asset_receivable","True","Piutang Usaha"
"l10n_id_11210011","11210011","Account Receivable (PoS)","asset_receivable","True","Piutang Usaha (PoS)"
"l10n_id_11210020","11210020","Employee Liabilities","asset_current","True","Piutang Karyawan"
"l10n_id_11300180","11300180","Other Inventory","asset_current","False","Persediaan Lainnya"
"l10n_id_11410010","11410010","Building Rent","asset_prepayments","False","Sewa Bangunan"
"l10n_id_11410020","11410020","Prepaid Insurance","asset_prepayments","False","Asuransi Dibayar Dimuka"
"l10n_id_11410030","11410030","Prepaid Advertisement-Free","asset_prepayments","False","Beban Iklan Dibayar Dimuka"
"l10n_id_11510010","11510010","Prepaid Tax PPh 21","asset_prepayments","False",""
"l10n_id_11510020","11510020","Prepaid Tax Pph 22","asset_prepayments","False","Pajak Dibayar Dimuka PPH 22"
"l10n_id_11510030","11510030","Prepaid Tax Pph 23","asset_prepayments","False","Pajak Dibayar Dimuka PPH 23"
"l10n_id_11510040","11510040","Prepaid Tax Pph 25","asset_prepayments","False","Pajak Dibayar Dimuka PPH 25"
"l10n_id_11510050","11510050","Prepaid Tax Pph 28A","asset_prepayments","False",""
"l10n_id_11510060","11510060","Prepaid Tax 4 (2)","asset_prepayments","False",""
"l10n_id_11800000","11800000","Down Payment","asset_prepayments","False","Uang Muka Pembelian"
"l10n_id_12210010","12210010","Office Building","asset_fixed","False","Bangunan Kantor"
"l10n_id_12210020","12210020","Vehicle","asset_fixed","False","Kendaraan"
"l10n_id_12210030","12210030","Office Supplies","asset_fixed","False","Peralatan Kantor"
"l10n_id_12281010","12281010","Accumulation Building Depreciation","asset_prepayments","False","Akumulasi Penyusutan Bangunan Kantor"
"l10n_id_12281020","12281020","Accumulation Vehicle Depreciation","asset_prepayments","False","Akumulasi Penyusutan Kendaraan"
"l10n_id_12281030","12281030","Accumulation Office Supplies Depreciation","asset_prepayments","False","Akumulasi Penyusutan Peralatan Kantor"
"l10n_id_21100010","21100010","Trade Receivable","liability_payable","True","Hutang Usaha"
"l10n_id_21100020","21100020","Shareholder Deposit","liability_current","False","Hutang Pemegang Saham"
"l10n_id_21100030","21100030","Third-Party Deposit","liability_current","False","Hutang Pihak Ketiga"
"l10n_id_21100040","21100040","Salary Deposit","liability_current","False","Hutang Gaji"
"l10n_id_21210010","21210010","Tax Payable Pph 21","liability_current","False","Hutang Pajak PPh 21"
"l10n_id_21210020","21210020","Tax Payable Pph 23","liability_current","False","Hutang Pajak PPh 23"
"l10n_id_21210030","21210030","Tax Payable Pph 25","liability_current","False","Hutang Pajak PPh 25"
"l10n_id_21210040","21210040","Tax Payable 4 (2)","liability_current","False","Hutang Pajak Pasal 4 (2)"
"l10n_id_21210050","21210050","Tax Payable Pph 29","liability_current","False","Hutang Pajak PPh 29"
"l10n_id_21221010","21221010","VAT Purchase","liability_current","False","PPN Pembelian"
"l10n_id_21221020","21221020","VAT Sales","liability_current","False","PPN Penjualan"
"l10n_id_22110010","22110010","Bank Loan","liability_current","False","Hutang Bank"
"l10n_id_22110020","22110020","Leasing Deposit","liability_current","False","Hutang Leasing"
"l10n_id_25110010","25110010","Accrued Payable Electricity","liability_current","False","BYMHD Listrik"
"l10n_id_25110020","25110020","Accrued Payable Jamsostek","liability_current","False","BYMHD Jamsostek"
"l10n_id_25110030","25110030","Accrued Payable Water","liability_current","False","BYMHD Air"
"l10n_id_25110040","25110040","Accrued Payable Telp & Internet","liability_current","False","BYMHD Telepon"
"l10n_id_25110050","25110050","Accrued Payable Security Management","liability_current","False","BYMHD Jasa Pengelola Keamanan"
"l10n_id_25110060","25110060","Accrued Payable Bank","liability_current","False","BYMHD Bank"
"l10n_id_25110070","25110070","Accrued Payable PBB","liability_current","False","BYMHD PBB"
"l10n_id_25110080","25110080","Accrued Payable Business License","liability_current","False","BYMHD Izin Usaha"
"l10n_id_25110090","25110090","Accrued Payable Insurance","liability_current","False","BYMHD Asuransi"
"l10n_id_25110100","25110100","Accrued Payable Education","liability_current","False","BYMHD Pendidikan dan Latihan"
"l10n_id_25110110","25110110","Accrued Payable Health Insurance/BPJS","liability_current","False","BYMHD Jaminan Kesehatan/BPJS"
"l10n_id_28110010","28110010","Advance Sales","liability_current","False","Uang Muka Penjualan"
"l10n_id_28110020","28110020","Customer Deposit","liability_current","False","Deposit Customer"
"l10n_id_29000000","29000000","Interim Stock","liability_current","False","Stok Interim"
"l10n_id_31100010","31100010","Authorized Capital","equity","False","Modal Dasar"
"l10n_id_31100020","31100020","Paid Capital","equity","False","Modal Yang Disetor"
"l10n_id_31100030","31100030","Unpaid Capital","equity","False","Modal Yang Belum Disetor"
"l10n_id_31100040","31100040","Prive (Personal Retrieval)","equity","False","Prive (Pengambilan Pribadi)"
"l10n_id_31210010","31210010","Capital Reserves","equity","False","Cadangan Modal"
"l10n_id_31510010","31510010","Past Profit & Loss","equity","False","Laba Rugi Tahun Lalu"
"l10n_id_31510020","31510020","Ongoing Profit & Loss","equity","False","Laba Rugi Tahun Berjalan"
"l10n_id_39000000","39000000","Historical Balance","equity","True","Historical Balance"
"l10n_id_41000010","41000010","Sales","income","False","Penjualan"
"l10n_id_42000060","42000060","Sales Refund","income","False","Retur Penjualan"
"l10n_id_42000070","42000070","Sales Discount","income","False","Discount Penjualan"
"l10n_id_51000010","51000010","Cost of Goods Sold","expense_direct_cost","False","Harga Pokok Penjualan"
"l10n_id_61100010","61100010","Employee Salary","expense","False","Gaji Karyawan"
"l10n_id_61100020","61100020","Employee Bonus / Benefits","expense","False","Tunjangan/ Bonus Karyawan"
"l10n_id_61100030","61100030","Employee Overtime Pay","expense","False","Lembur Karyawan"
"l10n_id_61100100","61100100","Pph 21 Benefit","expense","False","Tunjangan PPH Pasal 21"
"l10n_id_63110060","63110060","Phone","expense","False","Telepon"
"l10n_id_63110080","63110080","Electricity","expense","False","Listrik"
"l10n_id_63110100","63110100","Research & Development","expense","False","Research & Development"
"l10n_id_63110120","63110120","Office Equipment","expense","False","Perlengkapan Kantor"
"l10n_id_64110020","64110020","Post Necessities","expense","False","Keperluan Pos"
"l10n_id_63110140","63110140","Other Necessities","expense","False","Keperluan Lain-lain"
"l10n_id_65110010","65110010","Licensing Fees","expense","False","Biaya Perizinan"
"l10n_id_65110020","65110020","Bank Administration Fees","expense","False","Biaya Administrasi Bank"
"l10n_id_65110030","65110030","Consultant Fees","expense","False","Biaya Konsultan"
"l10n_id_65110040","65110040","Rental Costs","expense","False","Biaya Sewa"
"l10n_id_65110050","65110050","Insurance Costs","expense","False",""
"l10n_id_65110060","65110060","Building Maintenance Costs","expense","False","Biaya Pemeliharaan & Perawatan Gedung"
"l10n_id_65110070","65110070","Taxes","expense","False","Pajak"
"l10n_id_65110080","65110080","Asset Maintenance Costs","expense","False","Biaya Pemeliharaan & Perawatan Aset"
"l10n_id_65110090","65110090","Shipping Costs","expense","False","Biaya Pengiriman Dokumen/Barang"
"l10n_id_66110010","66110010","Vehicle Fuel","expense","False","BBM kendaraan"
"l10n_id_66110020","66110020","Vehicle Service","expense","False","Service kendaraan"
"l10n_id_66110030","66110030","Vehicle Parking & Toll Fee","expense","False","Parkir & tol kendaraan"
"l10n_id_66110040","66110040","Vehicle Taxes","expense","False","Pajak Kendaraan"
"l10n_id_66110050","66110050","Vehicle Insurance","expense","False","Asuransi Kendaraan"
"l10n_id_67100010","67100010","Office Building","expense_depreciation","False","Bangunan Kantor"
"l10n_id_67100020","67100020","Vehicle","expense_depreciation","False","Kendaraan"
"l10n_id_67100030","67100030","Office Supplies","expense_depreciation","False","Peralatan Kantor"
"l10n_id_69000000","69000000","Other Expenses","expense","False","Biaya Lain-lain"
"l10n_id_81100010","81100010","Interest Income","income_other","False","Pendapatan Bunga"
"l10n_id_81100020","81100020","Deposit Income","income_other","False","Pendapatan Deposit"
"l10n_id_81100030","81100030","Foreign Exchange Gain","income_other","False","Keuntungan Selisih Kurs"
"l10n_id_81100040","81100040","Other Income","income_other","False","Pendapatan lainnya"
"l10n_id_81100050","81100050","Gain on Sale of Fixed Assets","income_other","False","Keuntungan Atas Penjualan Aktiva Tetap"
"l10n_id_91100010","91100010","Interest Expense","expense","False","Beban Bunga"
"l10n_id_91100020","91100020","Foreign Exchange Loss","expense","False","Kerugian Selisih Kurs"
"l10n_id_91100030","91100030","Loss on Sale of Fixed Assets","expense","False","Kerugian Atas Penjualan Aktiva Tetap"
"l10n_id_99900001","99900001","Cash Difference Loss","expense","False",""
"l10n_id_99900002","99900002","Cash Difference Gain","income","False",""
"l10n_id_99900003","99900003","Cash Discount Loss","expense","False",""
"l10n_id_99900004","99900004","Cash Discount Gain","income_other","False",""
"l10n_id_999999","999999","Undistributed Profits/Losses","equity_unaffected","False",""

```

## File: data\template\account.tax-id.csv

```csv
"id","description","invoice_label","type_tax_use","tax_group_id","name","amount_type","amount","repartition_line_ids/repartition_type","repartition_line_ids/tag_ids","repartition_line_ids/document_type","repartition_line_ids/account_id"
"tax_ST1","ST1","11%","sale","default_tax_group","11%","percent","11.0","base","l10n_id.ppn_tag","invoice",""
"","","","","","","","","tax","","invoice","l10n_id_21221020"
"","","","","","","","","base","l10n_id.ppn_tag","refund",""
"","","","","","","","","tax","","refund","l10n_id_21221020"
"tax_PT1","PT1","11%","purchase","default_tax_group","11%","percent","11.0","base","l10n_id.ppn_tag","invoice",""
"","","","","","","","","tax","","invoice","l10n_id_21221010"
"","","","","","","","","base","l10n_id.ppn_tag","refund",""
"","","","","","","","","tax","","refund","l10n_id_21221010"
"tax_ST0","ST0","0%","sale","default_tax_group","0%","percent","0.0","base","","invoice",""
"","","","","","","","","tax","","invoice","l10n_id_21221020"
"","","","","","","","","base","","refund",""
"","","","","","","","","tax","","refund","l10n_id_21221020"
"tax_ST2","ST2","0%","sale","default_tax_group","0% EXEMPT","percent","0.0","base","","invoice",""
"","","","","","","","","tax","","invoice","l10n_id_21221020"
"","","","","","","","","base","","refund",""
"","","","","","","","","tax","","refund","l10n_id_21221020"
"tax_PT2","PT2","0%","purchase","default_tax_group","0% EXEMPT","percent","0.0","base","","invoice",""
"","","","","","","","","tax","","invoice","l10n_id_21221010"
"","","","","","","","","base","","refund",""
"","","","","","","","","tax","","refund","l10n_id_21221010"
"tax_PT0","PT0","0%","purchase","default_tax_group","0%","percent","0.0","base","","invoice",""
"","","","","","","","","tax","","invoice","l10n_id_21221010"
"","","","","","","","","base","","refund",""
"","","","","","","","","tax","","refund","l10n_id_21221010"
"tax_ST3","ST3","12%","sale","default_tax_group","12%","percent","12.0","base","l10n_id.ppn_tag","invoice",""
"","","","","","","","","tax","","invoice","l10n_id_21221020"
"","","","","","","","","base","l10n_id.ppn_tag","refund",""
"","","","","","","","","tax","","refund","l10n_id_21221020"
"tax_PT3","PT3","12%","purchase","default_tax_group","12%","percent","12.0","base","l10n_id.ppn_tag","invoice",""
"","","","","","","","","tax","","invoice","l10n_id_21221010"
"","","","","","","","","base","l10n_id.ppn_tag","refund",""
"","","","","","","","","tax","","refund","l10n_id_21221010"
"tax_luxury_sales","Luxury","Luxury Goods (ID)","sale","l10n_id_tax_group_luxury_goods","20%","percent","20.0","base","l10n_id.ppn_tag","invoice",""
"","","","","","","","","tax","","invoice","l10n_id_21221020"
"","","","","","","","","base","l10n_id.ppn_tag","refund",""
"","","","","","","","","tax","","refund","l10n_id_21221020"

```

## File: data\template\account.tax.group-id.csv

```csv
"id","country_id","name","name@id"
"default_tax_group","base.id","Taxes","Pajak"
"l10n_id_tax_group_luxury_goods","base.id","Luxury Good Taxes (ID)","Pajak Barang Mewah (ID)"

```

## File: migrations\1.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'id')], order="parent_path"):
        env['account.chart.template'].try_loading('id', company)

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, _


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_id_qris_transaction_ids = fields.Many2many('l10n_id.qris.transaction')

    def _generate_qr_code(self, silent_errors=False):
        """
        Adds information about which invoice is triggering the creation of the QR-Code, so that we can link both together.
        """
        # EXTENDS account
        return super(
            AccountMove,
            self.with_context(qris_model="account.move", qris_model_id=str(self.id)),
        )._generate_qr_code(silent_errors)

    def _l10n_id_cron_update_payment_status(self):
        """
        This cron will:
            - Get all invoices that are not paid, and have details about QRIS qr codes.
            - For each invoices, get information about the payment state of the QR using the API.
            - If the QR is not paid and it has been more than 30m, we discard that qr id (no longer valid)
            - If it is paid, we will register the payment on the invoices.
        """
        invoices = self.search([
            ('payment_state', '=', 'not_paid'),
            ('l10n_id_qris_transaction_ids', '!=', False)
        ])
        return invoices._l10n_id_update_payment_status()

    def action_l10n_id_update_payment_status(self):
        """
        This action will:
            - Get all invoices that are not paid, and have details about QRIS qr codes.
            - For each invoices, get information about the payment state of the QR using the API.
            - If the QR is not paid and it has been more than 30m, we discard that qr id (no longer valid)
            - If it is paid, we will register the payment on the invoices.
        """
        invoices = self.filtered_domain([
            ('payment_state', '=', 'not_paid'),
            ('l10n_id_qris_transaction_ids', '!=', False)
        ])
        return invoices._l10n_id_update_payment_status()

    def _l10n_id_update_payment_status(self):
        """ Starts by fetching the QR statuses for the invoices in self, then update said invoices based on the statuses """
        qr_statuses = self._l10n_id_get_qris_qr_statuses()
        return self._l10n_id_process_invoices(qr_statuses)

    def _l10n_id_get_qris_qr_statuses(self):
        """
        Query the API in order to get updated information on the status of each QR codes linked to the invoices in self.
        If the QR has been paid, only the paid information is returned.

        :return: a list with the format:
            {
                invoice: {
                    'paid': True,
                    'qr_statuses': [],
                },
                invoice: {
                    'paid': False,
                    'qr_statuses': [],
                }
            }
        """
        result = {}
        for invoice in self:
            result[invoice.id] = invoice.l10n_id_qris_transaction_ids._l10n_id_get_qris_qr_statuses()
        return result

    def _l10n_id_process_invoices(self, invoices_statuses):
        """
        Receives the list of invoices and their statuses, and update them using it.
        For paid invoices we will register the payment and log a note, while for unpaid ones we will discard expired
        QR data and keep the non-expired ones for the next run.
        """
        paid_invoices = self.env['account.move']
        paid_messages = {}
        for invoice in self:
            statuses = invoices_statuses.get(invoice.id)
            # Paid invoice: we simply prepare a message to notify of the payment with details if possible.
            if statuses['paid']:
                paid_status = statuses['qr_statuses'][0]
                if 'qris_payment_customername' in paid_status and 'qris_payment_methodby' in paid_status:
                    message = _(
                        "This invoice was paid by %(customer)s using QRIS with the payment method %(method)s.",
                        customer=paid_status['qris_payment_customername'],
                        method=paid_status['qris_payment_methodby'],
                    )
                else:
                    message = _("This invoice was paid using QRIS.")
                paid_invoices |= invoice
                paid_messages[invoice.id] = message

        # Update paid invoices
        if paid_invoices:
            paid_invoices._message_log_batch(bodies=paid_messages)
            # Finally, register the payment:
            return self.env['account.payment.register'].with_context(
                active_model='account.move', active_ids=paid_invoices.ids
            ).create({'group_payment': False}).action_create_payments()

```

## File: models\qris_transaction.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from datetime import timedelta
from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class QRISTransaction(models.Model):
    """QRIS Transaction

    General table to store a certian unique transaction with QRIS details attached
    """
    _name = "l10n_id.qris.transaction"
    _description = "Record of QRIS transactions"

    model = fields.Char(string="Model")  # payment in respond to which model
    model_id = fields.Char(string="Model ID")  # id/uuid

    # Fields that store the QRIS details coming from API request
    qris_invoice_id = fields.Char(readonly=True)
    qris_amount = fields.Integer(readonly=True)
    qris_content = fields.Char(readonly=True)
    qris_creation_datetime = fields.Datetime(readonly=True)

    bank_id = fields.Many2one("res.partner.bank", help="Bank used to generate the current QRIS transaction")
    paid = fields.Boolean(help="Payment Status of QRIS")

    def _get_supported_models(self):
        return ['account.move']

    @api.constrains('model')
    def _constraint_model(self):
        # only allow supported models
        if self.model not in self._get_supported_models():
            raise ValidationError(_("QRIS capability is not extended to model %s yet!", self.model))

    def _get_record(self):
        """ Get the backend invoice record that the qris transaction is handling
        To be overriden in other modules"""
        self.ensure_one()
        if self.model != 'account.move':
            return
        return self.env['account.move'].browse(int(self.model_id)).exists()

    @api.model
    def _get_latest_transaction(self, model, model_id):
        """ Find latest transaction associated to the model and model_id """
        return self.search([('model', '=', model), ('model_id', '=', model_id)], order='qris_creation_datetime desc', limit=1)

    def _l10n_id_get_qris_qr_statuses(self):
        """ Fetch the result of the transaction

        :param invoice_bank_id (Model <res.partner.bank>): bank (with QRIS configuration)
        :returns tuple(bool, dict): paid/unpaid status and status_response from QRIS
        """
        # storing all failure transactions in case final result is unpaid
        unpaid_status_data = []

        # Looping to make requests is far from ideal, but we have no choices as they don't allow getting multiple QR result at once.
        # Ensure to loop in reverse and check from the most recent QR code.
        for transaction in self.sorted(lambda t: t.qris_creation_datetime):
            status_response = self.sudo().bank_id._l10n_id_qris_fetch_status(transaction)
            if status_response['data'].get('qris_status') == 'paid':
                transaction.paid = True
                return {
                    'paid': True,
                    'qr_statuses': [status_response['data']]
                }
            else:
                unpaid_status_data.append(status_response['data'])

        return {
            'paid': False,
            'qr_statuses': unpaid_status_data
        }

    @api.autovacuum
    def _gc_remove_pointless_qris_transactions(self):
        """ Removes unpaid transactions that have been for more than 35 minutes.
        These can no longer be paid and status will no longer change
        """
        time_limit = fields.Datetime.now() - timedelta(seconds=2100)
        transactions = self.env['l10n_id.qris.transaction'].search([('qris_creation_datetime', '<=', time_limit), ('paid', '=', False)])
        transactions.unlink()

```

## File: models\res_bank.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import datetime
import requests
import pytz
from urllib.parse import urljoin

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

QRIS_TIMEOUT = 35  # They say that the time to get a response vary between 6 to 30s


def _l10n_id_make_qris_request(endpoint, params):
    """ Make an API request to QRIS, using the given path and params. """
    url = urljoin('https://qris.online/restapi/qris/', endpoint)
    try:
        response = requests.get(url, params=params, timeout=QRIS_TIMEOUT)
        response.raise_for_status()
        response = response.json()
    except requests.exceptions.HTTPError as err:
        raise ValidationError(_("Communication with QRIS failed. QRIS returned with the following error: %s", err))
    except (requests.RequestException, ValueError):
        raise ValidationError(_("Could not establish a connection to the QRIS API."))

    return response


class ResBank(models.Model):
    _inherit = "res.partner.bank"

    l10n_id_qris_api_key = fields.Char("QRIS API Key", groups="base.group_system")
    l10n_id_qris_mid = fields.Char("QRIS Merchant ID", groups="base.group_system")

    @api.model
    def _get_available_qr_methods(self):
        # EXTENDS account
        rslt = super()._get_available_qr_methods()
        rslt.append(('id_qr', _("QRIS"), 40))
        return rslt

    def _get_error_messages_for_qr(self, qr_method, debtor_partner, currency):
        # EXTENDS account
        if qr_method == 'id_qr':
            if self.country_code != 'ID':
                return _("You cannot generate a QRIS QR code with a bank account that is not in Indonesia.")
            if currency.name not in ['IDR']:
                return _("You cannot generate a QRIS QR code with a currency other than IDR")
            if not (self.sudo().l10n_id_qris_api_key and self.sudo().l10n_id_qris_mid):
                return _("To use QRIS QR code, Please setup the QRIS API Key and Merchant ID on the bank's configuration")
            return None

        return super()._get_error_messages_for_qr(qr_method, debtor_partner, currency)

    def _check_for_qr_code_errors(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        # EXTENDS account
        if qr_method == 'id_qr':
            if not amount:
                return _("The amount must be set to generate a QR code.")

        return super()._check_for_qr_code_errors(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

    def _get_qr_vals(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        """ Getting content for the QR through calling QRIS API and storing the QRIS transaction as a record"""
        # EXTENDS account
        if qr_method == "id_qr":
            model = self._context.get('qris_model')
            model_id = self._context.get('qris_model_id')

            # qris_trx is to help us fetch the backend record associated to the model and model_id.
            # we are using model and model_id instead of model.browse(id) because while executing this method
            # not all backend records are created already. For example, pos.order record isn't created until
            # payment is completed on the PoS interace.
            qris_trx = self.env['l10n_id.qris.transaction']._get_latest_transaction(model, model_id)

            # QRIS codes are valid for 30 minutes. To leave some margin, we will return the same QR code we already
            # generated if the invoice is re-accessed before 25m. Otherwise, a new QR code is generated
            # Additionally, we want to check that it's requesting for the same amount as it's possible to change
            # amount in apps like PoS.
            if qris_trx and qris_trx.qris_amount == int(amount):
                now = fields.Datetime.now()
                latest_qr_date = qris_trx.qris_creation_datetime

                if (now - latest_qr_date).total_seconds() < 1500:
                    return qris_trx['qris_content']

            params = {
                "do": "create-invoice",
                "apikey": self.l10n_id_qris_api_key,
                "mID": self.l10n_id_qris_mid,
                "cliTrxNumber": free_communication or structured_communication,
                "cliTrxAmount": int(amount)
            }
            response = _l10n_id_make_qris_request('show_qris.php', params)
            if response.get("status") == "failed":
                raise ValidationError(response.get("data"))
            data = response.get('data')

            # create a new transaction line while also converting the qris_request_date to UTC time
            if model and model_id:
                new_trx = self.env['l10n_id.qris.transaction'].create({
                    'model': model,
                    'model_id': model_id,
                    'qris_invoice_id': data.get('qris_invoiceid'),
                    'qris_amount': int(amount),
                    # Since the QRIS response is always returned with "Asia/Jakarta" timezone which is UTC+07:00
                    'qris_creation_datetime': fields.Datetime.to_datetime(data.get('qris_request_date')) - datetime.timedelta(hours=7),
                    'qris_content': data.get('qris_content'),
                    'bank_id': self.id
                })

                # Search the backend record and attach the qris transaction to the record if it exists.
                trx_record = new_trx._get_record()
                if trx_record:
                    trx_record.l10n_id_qris_transaction_ids |= new_trx

            return data.get('qris_content')

        return super()._get_qr_vals(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

    def _get_qr_code_generation_params(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        # EXTENDS account
        if qr_method == 'id_qr':
            if not self._context.get('is_online_qr'):
                return {}
            return {
                'barcode_type': 'QR',
                'width': 120,
                'height': 120,
                'value': self._get_qr_vals(qr_method, amount, currency, debtor_partner, free_communication, structured_communication),
            }
        return super()._get_qr_code_generation_params(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

    def _l10n_id_qris_fetch_status(self, qr_data):
        """
        using self and the given data, fetches the status of a specific QR code generated by QRIS
        Expected values in the qr_data dict are:
            - invoice_id returned when generating a QR code
            - the amount present in the qr code
            - the datetime at which the QR code was generated
        """
        return _l10n_id_make_qris_request('checkpaid_qris.php', {
            'do': 'checkStatus',
            'apikey': self.l10n_id_qris_api_key,
            'mID': self.l10n_id_qris_mid,
            'invid': qr_data['qris_invoice_id'],
            'trxvalue': qr_data['qris_amount'],
            'trxdate': qr_data['qris_creation_datetime'],
        })

```

## File: models\template_id.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('id')
    def _get_id_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_id_11210010',
            'property_account_payable_id': 'l10n_id_21100010',
            'property_account_expense_categ_id': 'l10n_id_51000010',
            'property_account_income_categ_id': 'l10n_id_41000010',
            'property_stock_account_input_categ_id': 'l10n_id_29000000',
            'property_stock_account_output_categ_id': 'l10n_id_29000000',
            'property_stock_valuation_account_id': 'l10n_id_11300180',
            'code_digits': '8',
        }

    @template('id', 'res.company')
    def _get_id_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.id',
                'bank_account_code_prefix': '1112',
                'cash_account_code_prefix': '1111',
                'transfer_account_code_prefix': '1999999',
                'account_default_pos_receivable_account_id': 'l10n_id_11210011',
                'income_currency_exchange_account_id': 'l10n_id_81100010',
                'expense_currency_exchange_account_id': 'l10n_id_91100010',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_id_99900003',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_id_99900004',
                'account_sale_tax_id': 'tax_ST1',
                'account_purchase_tax_id': 'tax_PT1',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import template_id
from . import res_bank
from . import qris_transaction

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_id_qris_transaction,access_l10n_id_qris_transaction,model_l10n_id_qris_transaction,account.group_account_invoice,1,1,1,1

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="action_fetch_qris_status" model="ir.actions.server">
            <field name="name">Check QRIS Payment Status</field>
            <field name="groups_id" eval="[(4, ref('account.group_account_invoice'))]"/>
            <field name="model_id" ref="account.model_account_move"/>
            <field name="binding_model_id" ref="account.model_account_move"/>
            <field name="binding_view_types">list,form</field>
            <field name="state">code</field>
            <field name="code">
                if records:
                    action = records.action_l10n_id_update_payment_status()
            </field>
        </record>
</odoo>

```

## File: views\res_bank.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_partner_bank_form_inherit_account" model="ir.ui.view">
        <field name="name">res.partner.bank.form.inherit.account</field>
        <field name="model">res.partner.bank</field>
        <field name="inherit_id" ref="base.view_partner_bank_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='currency_id']" position="after">
                <field name="l10n_id_qris_api_key" invisible="country_code != 'ID'"/>
                <field name="l10n_id_qris_mid" invisible="country_code != 'ID'" />
            </xpath>
        </field>
    </record>

</odoo>

```

