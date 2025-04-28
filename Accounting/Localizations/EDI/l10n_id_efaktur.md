# Odoo Module: l10n_id_efaktur

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Indonesia E-faktur',
    'icon': '/l10n_id/static/description/icon.png',
    'version': '1.0',
    'description': """
        E-Faktur Menu(Indonesia)
        Format : 010.000-16.00000001
        * 2 (dua) digit pertama adalah Kode Transaksi
        * 1 (satu) digit berikutnya adalah Kode Status
        * 3 (tiga) digit berikutnya adalah Kode Cabang
        * 2 (dua) digit pertama adalah Tahun Penerbitan
        * 8 (delapan) digit berikutnya adalah Nomor Urut

        To be able to export customer invoices as e-Faktur,
        you need to put the ranges of numbers you were assigned
        by the government in Accounting > Customers > e-Faktur

        When you validate an invoice, where the partner has the ID PKP
        field checked, a tax number will be assigned to that invoice.
        Afterwards, you can filter the invoices still to export in the
        invoices list and click on Action > Download e-Faktur to download
        the csv and upload it to the site of the government.

        You can replace an already sent invoice by another by indicating
        the replaced invoice and the new one and you can reset an invoice
        you have not already sent to the government to reuse its number.
    """,
    'category': 'Accounting/Localizations/EDI',
    'depends': ['l10n_id'],
    'data': [
            'security/ir.model.access.csv',
            'views/account_move_views.xml',
            'views/efaktur_views.xml',
            'views/res_config_settings_views.xml',
            'views/res_partner_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\account_move.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import re
from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError, RedirectWarning
from odoo.tools import float_round, float_repr

FK_HEAD_LIST = ['FK', 'KD_JENIS_TRANSAKSI', 'FG_PENGGANTI', 'NOMOR_FAKTUR', 'MASA_PAJAK', 'TAHUN_PAJAK', 'TANGGAL_FAKTUR', 'NPWP', 'NAMA', 'ALAMAT_LENGKAP', 'JUMLAH_DPP', 'JUMLAH_PPN', 'JUMLAH_PPNBM', 'ID_KETERANGAN_TAMBAHAN', 'FG_UANG_MUKA', 'UANG_MUKA_DPP', 'UANG_MUKA_PPN', 'UANG_MUKA_PPNBM', 'REFERENSI', 'KODE_DOKUMEN_PENDUKUNG']

LT_HEAD_LIST = ['LT', 'NPWP', 'NAMA', 'JALAN', 'BLOK', 'NOMOR', 'RT', 'RW', 'KECAMATAN', 'KELURAHAN', 'KABUPATEN', 'PROPINSI', 'KODE_POS', 'NOMOR_TELEPON']

OF_HEAD_LIST = ['OF', 'KODE_OBJEK', 'NAMA', 'HARGA_SATUAN', 'JUMLAH_BARANG', 'HARGA_TOTAL', 'DISKON', 'DPP', 'PPN', 'TARIF_PPNBM', 'PPNBM']


def _csv_row(data, delimiter=',', quote='"'):
    return quote + (quote + delimiter + quote).join([str(x).replace(quote, '\\' + quote) for x in data]) + quote + '\n'


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_id_tax_number = fields.Char(string="Tax Number", copy=False)
    l10n_id_replace_invoice_id = fields.Many2one('account.move', string="Replace Invoice", domain="['|', '&', '&', ('state', '=', 'posted'), ('partner_id', '=', partner_id), ('reversal_move_id', '!=', False), ('state', '=', 'cancel')]", copy=False, index='btree_not_null')
    l10n_id_attachment_id = fields.Many2one('ir.attachment', readonly=True, copy=False)
    l10n_id_csv_created = fields.Boolean('CSV Created', compute='_compute_csv_created', copy=False)
    l10n_id_kode_transaksi = fields.Selection([
            ('01', '01 To the Parties that is not VAT Collector (Regular Customers)'),
            ('02', '02 To the Treasurer'),
            ('03', '03 To other VAT Collectors other than the Treasurer'),
            ('04', '04 Other Value of VAT Imposition Base'),
            ('05', '05 Specified Amount (Article 9A Paragraph (1) VAT Law)'),
            ('06', '06 to individuals holding foreign passports'),
            ('07', '07 Deliveries that the VAT is not Collected'),
            ('08', '08 Deliveries that the VAT is Exempted'),
            ('09', '09 Deliveries of Assets (Article 16D of VAT Law)'),
        ], string='Kode Transaksi', help='Dua digit pertama nomor pajak',
        readonly=False, states={'posted': [('readonly', True)], 'cancel': [('readonly', True)]}, copy=False,
        compute="_compute_kode_transaksi", store=True)
    l10n_id_need_kode_transaksi = fields.Boolean(compute='_compute_need_kode_transaksi')

    @api.onchange('l10n_id_tax_number')
    def _onchange_l10n_id_tax_number(self):
        for record in self:
            if record.l10n_id_tax_number and record.move_type not in self.get_purchase_types():
                raise UserError(_("You can only change the number manually for a Vendor Bills and Credit Notes"))

    @api.depends('l10n_id_attachment_id')
    def _compute_csv_created(self):
        for record in self:
            record.l10n_id_csv_created = bool(record.l10n_id_attachment_id)

    @api.depends('partner_id')
    def _compute_kode_transaksi(self):
        for move in self:
            move.l10n_id_kode_transaksi = move.partner_id.commercial_partner_id.l10n_id_kode_transaksi

    @api.depends('partner_id', 'line_ids.tax_ids')
    def _compute_need_kode_transaksi(self):
        for move in self:
            # If there are no taxes at all on every line (0% taxes counts as having a tax) then we don't need a kode transaksi
            move.l10n_id_need_kode_transaksi = (
                move.partner_id.commercial_partner_id.l10n_id_pkp
                and not move.l10n_id_tax_number
                and move.move_type == 'out_invoice'
                and move.country_code == 'ID'
                and move.line_ids.tax_ids
            )

    @api.constrains('l10n_id_kode_transaksi', 'line_ids', 'partner_id')
    def _constraint_kode_ppn(self):
        ppn_tag = self.env.ref('l10n_id.ppn_tag')
        for move in self.filtered(lambda m: m.l10n_id_need_kode_transaksi and m.l10n_id_kode_transaksi != '08'):
            if any(ppn_tag.id in line.tax_tag_ids.ids for line in move.line_ids if line.display_type == 'product') \
                    and any(ppn_tag.id not in line.tax_tag_ids.ids for line in move.line_ids if line.display_type == 'product'):
                raise UserError(_('Cannot mix VAT subject and Non-VAT subject items in the same invoice with this kode transaksi.'))
        for move in self.filtered(lambda m: m.l10n_id_need_kode_transaksi and m.l10n_id_kode_transaksi == '08'):
            if any(ppn_tag.id in line.tax_tag_ids.ids for line in move.line_ids if line.display_type == 'product'):
                raise UserError('Kode transaksi 08 is only for non VAT subject items.')

    @api.constrains('l10n_id_tax_number')
    def _constrains_l10n_id_tax_number(self):
        for record in self.filtered('l10n_id_tax_number'):
            if record.l10n_id_tax_number != re.sub(r'\D', '', record.l10n_id_tax_number):
                record.l10n_id_tax_number = re.sub(r'\D', '', record.l10n_id_tax_number)
            if len(record.l10n_id_tax_number) != 16:
                raise UserError(_('A tax number should have 16 digits'))
            elif record.l10n_id_tax_number[:2] not in dict(self._fields['l10n_id_kode_transaksi'].selection).keys():
                raise UserError(_('A tax number must begin by a valid Kode Transaksi'))
            elif record.l10n_id_tax_number[2] not in ('0', '1'):
                raise UserError(_('The third digit of a tax number must be 0 or 1'))

    def _post(self, soft=True):
        """Set E-Faktur number after validation."""
        for move in self:
            if move.l10n_id_need_kode_transaksi:
                if not move.l10n_id_kode_transaksi:
                    raise ValidationError(_('You need to put a Kode Transaksi for this partner.'))
                if move.l10n_id_replace_invoice_id.l10n_id_tax_number:
                    if not move.l10n_id_replace_invoice_id.l10n_id_attachment_id:
                        raise ValidationError(_('Replacement invoice only for invoices on which the e-Faktur is generated. '))
                    rep_efaktur_str = move.l10n_id_replace_invoice_id.l10n_id_tax_number
                    move.l10n_id_tax_number = '%s1%s' % (move.l10n_id_kode_transaksi, rep_efaktur_str[3:])
                else:
                    efaktur = self.env['l10n_id_efaktur.efaktur.range'].pop_number(move.company_id.id)
                    if not efaktur:
                        raise ValidationError(_('There is no Efaktur number available.  Please configure the range you get from the government in the e-Faktur menu. '))
                    move.l10n_id_tax_number = '%s0%013d' % (str(move.l10n_id_kode_transaksi), efaktur)
        return super()._post(soft)

    def reset_efaktur(self):
        """Reset E-Faktur, so it can be use for other invoice."""
        for move in self:
            if move.l10n_id_attachment_id:
                raise UserError(_('You have already generated the tax report for this document: %s', move.name))
            self.env['l10n_id_efaktur.efaktur.range'].push_number(move.company_id.id, move.l10n_id_tax_number[3:])
            move.message_post(
                body='e-Faktur Reset: %s ' % (move.l10n_id_tax_number),
                subject="Reset Efaktur")
            move.l10n_id_tax_number = False
        return True

    def download_csv(self):
        action = {
            'type': 'ir.actions.act_url',
            'url': "web/content/?model=ir.attachment&id=" + str(self.l10n_id_attachment_id.id) + "&filename_field=name&field=datas&download=true&name=" + self.l10n_id_attachment_id.name,
            'target': 'self'
        }
        return action

    def download_efaktur(self):
        """Collect the data and execute function _generate_efaktur."""
        for record in self:
            if record.state == 'draft':
                raise ValidationError(_('Could not download E-faktur in draft state'))

            if not record.l10n_id_tax_number:
                if not record.l10n_id_need_kode_transaksi:
                    raise ValidationError(_('E-faktur is not available for invoices without any taxes.'))
                raise ValidationError(_('Connect %(move_number)s with E-faktur to download this report', move_number=record.name))

        self._generate_efaktur(',')
        return self.download_csv()

    def _generate_efaktur_invoice(self, delimiter):
        """Generate E-Faktur for customer invoice."""
        # Invoice of Customer

        output_head = '%s%s%s' % (
            _csv_row(FK_HEAD_LIST, delimiter),
            _csv_row(LT_HEAD_LIST, delimiter),
            _csv_row(OF_HEAD_LIST, delimiter),
        )

        idr = self.env.ref('base.IDR')

        for move in self.filtered(lambda m: m.state == 'posted'):
            eTax = move._prepare_etax()

            commercial_partner = move.partner_id.commercial_partner_id
            nik = str(commercial_partner.l10n_id_nik) if not commercial_partner.vat else ''

            if move.l10n_id_replace_invoice_id:
                number_ref = str(move.l10n_id_replace_invoice_id.name) + " replaced by " + str(move.name) + " " + nik
            elif nik:
                number_ref = str(move.name) + " " + nik
            else:
                number_ref = str(move.name)

            street = ', '.join([x for x in (move.partner_id.street, move.partner_id.street2) if x])

            invoice_npwp = ''
            if commercial_partner.vat and len(commercial_partner.vat) >= 15:
                invoice_npwp = commercial_partner.vat
            elif commercial_partner.l10n_id_nik:
                invoice_npwp = commercial_partner.l10n_id_nik
            if not invoice_npwp:
                action_error = {
                    'view_mode': 'form',
                    'res_model': 'res.partner',
                    'type': 'ir.actions.act_window',
                    'res_id': commercial_partner.id,
                    'views': [[self.env.ref('base.view_partner_form').id, 'form']],
                }
                msg = _("Please make sure that you've input the appropriate NPWP or NIK for the following customer")
                raise RedirectWarning(msg, action_error, _("Edit Customer Information"))
            invoice_npwp = invoice_npwp.replace('.', '').replace('-', '')

            etax_name = commercial_partner.l10n_id_tax_name or move.partner_id.name
            if invoice_npwp[:15] == '000000000000000' and commercial_partner.l10n_id_nik:
                etax_name = "%s#NIK#NAMA#%s" % (commercial_partner.l10n_id_nik, etax_name)

            # Here all fields or columns based on eTax Invoice Third Party
            eTax['KD_JENIS_TRANSAKSI'] = move.l10n_id_tax_number[0:2] or 0
            eTax['FG_PENGGANTI'] = move.l10n_id_tax_number[2:3] or 0
            eTax['NOMOR_FAKTUR'] = move.l10n_id_tax_number[3:] or 0
            eTax['MASA_PAJAK'] = move.invoice_date.month
            eTax['TAHUN_PAJAK'] = move.invoice_date.year
            eTax['TANGGAL_FAKTUR'] = '{0}/{1}/{2}'.format(move.invoice_date.day, move.invoice_date.month, move.invoice_date.year)
            eTax['NPWP'] = invoice_npwp
            eTax['NAMA'] = etax_name
            eTax['ALAMAT_LENGKAP'] = move.partner_id.contact_address.replace('\n', '').strip() if eTax['NPWP'] == '000000000000000' else commercial_partner.l10n_id_tax_address or street
            eTax['JUMLAH_DPP'] = int(float_round(move.amount_untaxed, 0))  # currency rounded to the unit
            eTax['JUMLAH_PPN'] = int(float_round(move.amount_tax, 0, rounding_method="DOWN"))  # tax amount ALWAYS rounded down
            eTax['ID_KETERANGAN_TAMBAHAN'] = '1' if move.l10n_id_kode_transaksi == '07' else ''
            eTax['REFERENSI'] = number_ref
            eTax['KODE_DOKUMEN_PENDUKUNG'] = '0'

            lines = move.line_ids.filtered(lambda x: x.move_id._is_downpayment() and x.price_unit < 0 and x.display_type == 'product')
            eTax['FG_UANG_MUKA'] = 0
            eTax['UANG_MUKA_DPP'] = float_repr(abs(sum(lines.mapped(lambda l: float_round(l.price_subtotal, 0)))), 0)
            eTax['UANG_MUKA_PPN'] = float_repr(abs(sum(lines.mapped(lambda l: float_round(l.price_total - l.price_subtotal, 0)))), 0)

            fk_values_list = ['FK'] + [eTax[f] for f in FK_HEAD_LIST[1:]]

            # HOW TO ADD 2 line to 1 line for free product
            free, sales = [], []

            for line in move.line_ids.filtered(lambda l: l.display_type == 'product'):
                # *invoice_line_unit_price is price unit use for harga_satuan's column
                # *invoice_line_quantity is quantity use for jumlah_barang's column
                # *invoice_line_total_price is bruto price use for harga_total's column
                # *invoice_line_discount_m2m is discount price use for diskon's column
                # *line.price_subtotal is subtotal price use for dpp's column
                # *tax_line or free_tax_line is tax price use for ppn's column
                free_tax_line = tax_line = 0.0

                for tax in line.tax_ids:
                    if tax.amount > 0:
                        tax_line += line.price_subtotal * (tax.amount / 100.0)

                discount = 1 - (line.discount / 100)
                # guarantees price to be tax-excluded
                invoice_line_total_price = line.price_subtotal / discount if discount else 0
                invoice_line_unit_price = invoice_line_total_price / line.quantity if line.quantity else 0

                line_dict = {
                    'KODE_OBJEK': line.product_id.default_code or '',
                    'NAMA': line.product_id.name or '',
                    'HARGA_SATUAN': float_repr(idr.round(invoice_line_unit_price), idr.decimal_places),
                    'JUMLAH_BARANG': line.quantity,
                    'HARGA_TOTAL': idr.round(invoice_line_total_price),
                    'DPP': line.price_subtotal,
                    'product_id': line.product_id.id,
                }

                if line.price_subtotal < 0:
                    for tax in line.tax_ids:
                        free_tax_line += (line.price_subtotal * (tax.amount / 100.0)) * -1.0

                    line_dict.update({
                        'DISKON': float_round(invoice_line_total_price - line.price_subtotal, 0),
                        'PPN': free_tax_line,
                    })
                    free.append(line_dict)
                elif line.price_subtotal != 0.0:
                    invoice_line_discount_m2m = invoice_line_total_price - line.price_subtotal

                    line_dict.update({
                        'DISKON': float_round(invoice_line_discount_m2m, 0),
                        'PPN': tax_line,
                    })
                    sales.append(line_dict)

            sub_total_before_adjustment = sub_total_ppn_before_adjustment = 0.0

            # We are finding the product that has affected
            # by free product to adjustment the calculation
            # of discount and subtotal.
            # - the price total of free product will be
            # included as a discount to related of product.
            for sale in sales:
                for f in free:
                    if f['product_id'] == sale['product_id']:
                        sale['DISKON'] = sale['DISKON'] - f['DISKON'] + f['PPN']
                        sale['DPP'] = sale['DPP'] + f['DPP']

                        tax_line = 0

                        for tax in line.tax_ids:
                            if tax.amount > 0:
                                tax_line += sale['DPP'] * (tax.amount / 100.0)

                        sale['PPN'] = tax_line

                        free.remove(f)

                sub_total_before_adjustment += sale['DPP']
                sub_total_ppn_before_adjustment += sale['PPN']

                sale.update({
                    # Use the db currency rounding to float_round the DPP/PPN.
                    # As we will correct them we need them to be close to the final result.
                    'DPP': idr.round(sale['DPP']),
                    'PPN': idr.round(sale['PPN']),
                    'DISKON': float_repr(sale['DISKON'], 0),
                })


            # The total of the base (DPP) and taxes (PPN) must be a integer, equal to the JUMLAH_DPP and JUMLAH_PPN
            # To do so, we adjust the first line in order to achieve the correct total
            if sales:
                diff_dpp = idr.round(eTax['JUMLAH_DPP'] - sum([sale['DPP'] for sale in sales]))
                total_sales_ppn = idr.round(eTax['JUMLAH_PPN'] - sum([sale['PPN'] for sale in sales]))
                # We will add the differences to the first line for which adding the difference will not result in a negative value.
                for sale in sales:
                    if sale['DPP'] + diff_dpp >= 0 and sale['PPN'] + total_sales_ppn >= 0:
                        sale['HARGA_TOTAL'] += diff_dpp
                        sale['DPP'] += diff_dpp
                        diff_dpp = 0
                        sale['PPN'] += total_sales_ppn
                        total_sales_ppn = 0
                        break

                # We couldn't adjust everything in a single line as their values is too low.
                # So we will instead slit the adjustment in multiple lines.
                if diff_dpp or total_sales_ppn:
                    for sale in sales:
                        # DPP
                        sale_dpp = sale['DPP']
                        sale["DPP"] = max(0, sale["DPP"] + diff_dpp)
                        diff_dpp -= (sale["DPP"] - sale_dpp)
                        sale['HARGA_TOTAL'] = sale["DPP"]
                        # PPN
                        sale_ppn = sale['PPN']
                        sale["PPN"] = max(0, sale["PPN"] + total_sales_ppn)
                        total_sales_ppn -= (sale["PPN"] - sale_ppn)

            # Values now being corrected, we can format them for the CSV
            for sale in sales:
                sale.update({
                    'HARGA_TOTAL': float_repr(sale['HARGA_TOTAL'], idr.decimal_places),
                    'DPP': float_repr(sale['DPP'], idr.decimal_places),
                    'PPN': float_repr(sale['PPN'], idr.decimal_places),
                })

            output_head += _csv_row(fk_values_list, delimiter)
            for sale in sales:
                of_values_list = ['OF'] + [str(sale[f]) for f in OF_HEAD_LIST[1:-2]] + ['0', '0']
                output_head += _csv_row(of_values_list, delimiter)

        return output_head

    def _prepare_etax(self):
        # These values are never set
        return {'JUMLAH_PPNBM': 0, 'UANG_MUKA_PPNBM': 0, 'JUMLAH_BARANG': 0, 'TARIF_PPNBM': 0, 'PPNBM': 0}

    def _generate_efaktur(self, delimiter):
        if self.filtered(lambda x: not x.l10n_id_kode_transaksi):
            raise UserError(_('Some documents don\'t have a transaction code'))
        if self.filtered(lambda x: x.move_type != 'out_invoice'):
            raise UserError(_('Some documents are not Customer Invoices'))

        output_head = self._generate_efaktur_invoice(delimiter)
        my_utf8 = output_head.encode("utf-8")
        out = base64.b64encode(my_utf8)

        attachment = self.env['ir.attachment'].create({
            'datas': out,
            'name': 'efaktur_%s.csv' % (fields.Datetime.to_string(fields.Datetime.now()).replace(" ", "_")),
            'type': 'binary',
        })

        for record in self:
            record.message_post(attachment_ids=[attachment.id])
        self.l10n_id_attachment_id = attachment.id
        return {
            'type': 'ir.actions.client',
            'tag': 'reload',
        }

```

## File: models\efaktur.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError

import re


class Efaktur(models.Model):
    _name = "l10n_id_efaktur.efaktur.range"
    _description = "Available E-faktur range"

    company_id = fields.Many2one('res.company', required=True, default=lambda self: self.env.company)
    max = fields.Char(compute='_compute_default', store=True, readonly=False)
    min = fields.Char(compute='_compute_default', store=True, readonly=False)
    available = fields.Integer(compute='_compute_available', store=True)

    @api.model
    def pop_number(self, company_id):
        range = self.search([('company_id', '=', company_id)], order="min ASC", limit=1)
        if not range:
            return None

        popped = int(range.min)
        if int(range.min) >= int(range.max):
            range.unlink()
        else:
            range.min = '%013d' % (popped + 1)
        return popped

    @api.model
    def push_number(self, company_id, number):
        return self.push_numbers(company_id, number, number)

    @api.model
    def push_numbers(self, company_id, min, max):
        range_sup = self.search([('min', '=', '%013d' % (int(max) + 1))])
        if range_sup:
            range_sup.min = '%013d' % int(min)
            max = range_sup.max

        range_low = self.search([('max', '=', '%013d' % (int(max) - 1))])
        if range_low:
            range_sup.unlink()
            range_low.max = '%013d' % int(max)

        if not range_sup and not range_low:
            self.create({
                'company_id': company_id,
                'max': '%013d' % int(max),
                'min': '%013d' % int(min),
            })


    @api.constrains('min', 'max')
    def _constrains_min_max(self):
        for record in self:
            if not len(record.min) == 13 or not len(record.max) == 13:
                raise ValidationError(_("There should be 13 digits in each number."))

            if record.min[:-8] != record.max[:-8]:
                raise ValidationError(_("First 5 digits should be same in Start Number and End Number."))

            if int(record.min[-8:]) > int(record.max[-8:]):
                raise ValidationError(_("Last 8 digits of End Number should be greater than the last 8 digit of Start Number"))

            if (int(record.max) - int(record.min)) > 10000:
                raise ValidationError(_("The difference between the two numbers must not be greater than 10.000"))

            # The number of records should always be very small, so it is ok to search in loop
            if self.search([
                '&', ('id', '!=', record.id), '|', '|',
                '&', ('min', '<=', record.max), ('max', '>=', record.max),
                '&', ('min', '<=', record.min), ('max', '>=', record.min),
                '&', ('min', '>=', record.min), ('max', '<=', record.max),
            ]):
                raise ValidationError(_('Efaktur interleaving range detected'))

    @api.depends('min', 'max')
    def _compute_available(self):
        for record in self:
            record.available = 1 + int(record.max) - int(record.min)

    @api.depends('company_id')
    def _compute_default(self):
        for record in self:
            query = """
                SELECT MAX(SUBSTRING(l10n_id_tax_number FROM 4))
                FROM account_move
                WHERE l10n_id_tax_number IS NOT NULL
                  AND company_id = %s
            """
            self.env.cr.execute(query, [record.company_id.id])
            max_used = int(self.env.cr.fetchone()[0] or 0)
            max_available = int(self.env['l10n_id_efaktur.efaktur.range'].search([('company_id', '=', record.company_id.id)], order='max DESC', limit=1).max)
            record.min = record.max = '%013d' % (max(max_available, max_used) + 1)

    @api.onchange('min')
    def _onchange_min(self):
        min_val = re.sub(r'\D', '', str(self.min)) or 0
        self.min = '%013d' % int(min_val)
        if not self.max or int(self.min) > int(self.max):
            self.max = self.min

    @api.onchange('max')
    def _onchange_max(self):
        max_val = re.sub(r'\D', '', str(self.max)) or 0
        self.max = '%013d' % int(max_val)
        if not self.min or int(self.min) > int(self.max):
            self.min = self.max

```

## File: models\res_partner.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResPartner(models.Model):
    """Inherit res.partner object to add NPWP field and Kode Transaksi"""
    _inherit = "res.partner"

    l10n_id_pkp = fields.Boolean(string="ID PKP", compute='_compute_l10n_id_pkp', store=True, readonly=False)
    l10n_id_nik = fields.Char(string='NIK')
    l10n_id_tax_address = fields.Char('Tax Address')
    l10n_id_tax_name = fields.Char('Tax Name')
    l10n_id_kode_transaksi = fields.Selection([
            ('01', '01 To the Parties that is not VAT Collector (Regular Customers)'),
            ('02', '02 To the Treasurer'),
            ('03', '03 To other VAT Collectors other than the Treasurer'),
            ('04', '04 Other Value of VAT Imposition Base'),
            ('05', '05 Specified Amount (Article 9A Paragraph (1) VAT Law)'),
            ('06', '06 to individuals holding foreign passports'),
            ('07', '07 Deliveries that the VAT is not Collected'),
            ('08', '08 Deliveries that the VAT is Exempted'),
            ('09', '09 Deliveries of Assets (Article 16D of VAT Law)'),
        ],
        string='Kode Transaksi',
        help='Dua digit pertama nomor pajak',
        default='01',
    )

    @api.depends('vat', 'country_code')
    def _compute_l10n_id_pkp(self):
        for record in self:
            record.l10n_id_pkp = record.vat and record.country_code == 'ID'


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_id_tax_address = fields.Char('Tax Address', related='company_id.partner_id.l10n_id_tax_address', readonly=False)
    l10n_id_tax_name = fields.Char('Tax Name', related='company_id.partner_id.l10n_id_tax_address', readonly=False)

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import efaktur
from . import account_move
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_efaktur_user,access.efaktur.user,model_l10n_id_efaktur_efaktur_range,account.group_account_invoice,1,1,1,1

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="account_move_efaktur_form_view" model="ir.ui.view">
            <field name="name">account.move.efaktur.form.view</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <field name="partner_id" position="after">
                    <field name="l10n_id_need_kode_transaksi" invisible="1"/>
                    <field name="l10n_id_attachment_id" invisible="1"/>
                    <field name="l10n_id_kode_transaksi" attrs="{'invisible': ['|', ('country_code', '!=', 'ID'), ('l10n_id_need_kode_transaksi', '=', False)], 'required': [('l10n_id_need_kode_transaksi', '=', True)]}"/>
                    <field name="l10n_id_replace_invoice_id" attrs="{'invisible': [('country_code', '!=', 'ID')], 'readonly': [('state', '!=','draft')]}" options="{'m2o_dialog': False, 'no_create': True}"/>
                </field>
                <button name="button_draft" position="after">
                    <button name="reset_efaktur" string="Reset E-Faktur" type="object" attrs="{'invisible':['|', ('country_code', '!=', 'ID'), '|', '|', ('l10n_id_tax_number','=',False), ('state','!=','cancel'), ('l10n_id_attachment_id','!=',False)]}"/>
                </button>
                <xpath expr=".//group[@id='other_tab_group']" position="inside">
                    <group string="Electronic Tax" attrs="{'invisible': [('country_code', '!=', 'ID')]}">
                        <field name="l10n_id_tax_number" attrs="{'invisible': [('move_type', '=', 'entry')], 'readonly': [('move_type', 'in', ('out_invoice', 'out_refund', 'out_receipt'))]}"/>
                        <field name="l10n_id_csv_created" attrs="{'invisible': [('move_type', 'not in', ('out_invoice', 'out_refund'))]}"/>
                    </group>
                </xpath>
            </field>
        </record>

        <record id="account_move_efaktur_tree_view" model="ir.ui.view">
            <field name="name">account.move.efaktur.tree.view</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_invoice_tree"/>
            <field name="arch" type="xml">
                <field name="invoice_user_id" position="after">
                    <field name="l10n_id_tax_number" optional="show"/>
                    <field name="l10n_id_csv_created" optional="hide"/>
                </field>
            </field>
        </record>

        <record id="dowload_efaktur_action" model="ir.actions.server">
            <field name="name">Download e-Faktur</field>
            <field name="model_id" ref="account.model_account_move"/>
            <field name="binding_model_id" ref="account.model_account_move"/>
            <field name="state">code</field>
            <field name="code">action = records.download_efaktur()</field>
        </record>

        <record id="view_account_invoice_filter" model="ir.ui.view">
            <field name="name">account.move.select.l10n_id.inherit</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_account_invoice_filter"/>
            <field name="arch" type="xml">
                <field name="name" position="after">
                    <field name="l10n_id_tax_number"/>
                    <field name="l10n_id_attachment_id"/>
                    <group>
                        <filter string="To Generate e-Faktur" name="tax_csv_upload" domain="[('l10n_id_tax_number', '!=', False), ('l10n_id_attachment_id', '=', False), ('state', '=', 'posted')]" />
                    </group>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\efaktur_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="efaktur_tree_view" model="ir.ui.view">
            <field name="name">l10n_id_efaktur.efaktur.range.tree.view</field>
            <field name="model">l10n_id_efaktur.efaktur.range</field>
            <field name="arch" type="xml">
                <tree string="Efaktur Number" editable="bottom">
                    <field name="min"/>
                    <field name="max"/>
                    <field name="available" sum="Total Available"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="company_id" invisible="1" groups="!base.group_multi_company"/>
                </tree>
            </field>
        </record>

        <record id='efaktur_invoice_action' model='ir.actions.act_window'>
            <field name="name">e-Faktur</field>
            <field name="res_model">l10n_id_efaktur.efaktur.range</field>
            <field name="view_mode">tree</field>
            <field name="context">{'search_default_upload': True, 'search_default_used': True}</field>
            <field name="view_id" ref="efaktur_tree_view"/>
            <field name="help" type="html">
                <p class="oe_view_nocontent_create">
                    In order to be able to export customer invoices as e-Faktur
                    for the Indonesian government, you need to put here the ranges
                    of numbers you were assigned by the government.
                    When you validate an invoice, a number will be assigned based on these ranges.
                    Afterwards, you can filter the invoices still to export in the
                    invoices list and click on Action > Download e-Faktur
                </p>
            </field>
        </record>

        <menuitem id="menu_efaktur_action" name="e-Faktur"
            parent="account.menu_finance_receivables"
            groups="account.group_account_manager"
            action="efaktur_invoice_action" sequence="111"/>

    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.chilean.loc</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='invoicing_settings']" position="after">
                <div attrs="{'invisible': [('country_code', '!=', 'ID')]}">
                    <div id="l10n_cl_title">
                        <h2>Indonesian Localization</h2>
                    </div>
                    <div id="l10n_cl_section" class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane"/>
                            <div class="o_setting_right_pane">
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                                <div class="content-group">
                                    <div class="row mt16">
                                        <label for="l10n_id_tax_address" class="col-lg-3"/>
                                        <field name="l10n_id_tax_address"/>
                                    </div>
                                    <div class="row mt16">
                                        <label for="l10n_id_tax_name" class="col-lg-3"/>
                                        <field name="l10n_id_tax_name"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
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
        <record id="res_partner_tax_form_view" model="ir.ui.view">
            <field name="name">res.partner.tax.form.view</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group/group" position="inside">
                    <field name="l10n_id_pkp" attrs="{'invisible': [('country_id', '!=', False), ('country_code', '!=', 'ID')]}"/>
                    <field name="l10n_id_kode_transaksi" attrs="{'invisible': [('l10n_id_pkp', '!=', True)]}"/>
                </xpath>
                <page name="accounting" position="inside">
                    <group string="Indonesian Taxes"  attrs="{'invisible': [('l10n_id_pkp', '!=', True)]}">
                        <group>
                            <field name="l10n_id_nik"/>
                        </group>
                        <group>
                            <field name="l10n_id_tax_address"/>
                            <field name="l10n_id_tax_name"/>
                        </group>
                    </group>
                </page>
            </field>
        </record>
    </data>
</odoo>

```

