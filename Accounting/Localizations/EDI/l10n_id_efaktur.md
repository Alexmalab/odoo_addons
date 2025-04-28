# Odoo Module: l10n_id_efaktur

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers
from . import wizard

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Indonesia E-faktur',
    'icon': '/account/static/description/l10n.png',
    'version': '1.0',
    'description': """
E-Faktur Menu(Indonesia)
Format: 010.000-16.00000001
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
            'security/ir_rule.xml',
            'views/account_move_views.xml',
            'views/efaktur_document_views.xml',
            'views/efaktur_views.xml',
            'views/res_config_settings_views.xml',
            'views/res_partner_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\download_efaktur.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http, _
from odoo.http import request
from odoo.addons.account.controllers.download_docs import _get_headers


class EfakturDownloadController(http.Controller):

    @http.route('/l10n_id_efaktur/download_attachments/<models("ir.attachment"):attachments>', type='http', auth='user')
    def download_invoice_attachments(self, attachments):
        attachments.check_access('read')
        assert all(attachment.res_id and attachment.res_model == 'l10n_id_efaktur.document' for attachment in attachments)
        if len(attachments) == 1:
            headers = _get_headers(attachments.name, attachments.mimetype, attachments.raw)
            return request.make_response(attachments.raw, headers)
        else:
            filename = _('efaktur') + '.zip'
            content = attachments._build_zip_from_attachments()
            headers = _get_headers(filename, 'zip', content)
            return request.make_response(content, headers)

```

## File: controllers\__init__.py

```python
from . import download_efaktur

```

## File: models\account_move.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError, RedirectWarning
from odoo.tools import float_round, float_repr


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_id_tax_number = fields.Char(string="Tax Number", copy=False)
    l10n_id_replace_invoice_id = fields.Many2one('account.move', string="Replace Invoice", domain="['|', '&', '&', ('state', '=', 'posted'), ('partner_id', '=', partner_id), ('reversal_move_ids', '!=', False), ('state', '=', 'cancel')]", copy=False, index='btree_not_null')
    l10n_id_efaktur_document = fields.Many2one('l10n_id_efaktur.document', readonly=True, copy=False, string="e-Faktur Document")
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
        readonly=False, copy=False,
        compute="_compute_kode_transaksi", store=True)
    l10n_id_efaktur_range = fields.Many2one("l10n_id_efaktur.efaktur.range", string="E-faktur Range", copy=False, domain="[('company_id', '=', company_id), ('available', '>', 0)]")
    l10n_id_need_kode_transaksi = fields.Boolean(compute='_compute_need_kode_transaksi')
    l10n_id_available_range_count = fields.Integer(compute="_compute_available_range_count", compute_sudo=True)
    l10n_id_show_kode_transaksi = fields.Boolean(compute='_compute_show_kode_transaksi')

    @api.depends('company_id')
    def _compute_available_range_count(self):
        # Only invoices under Indonesian company needs computation for l10n_id_available_range_count
        id_moves = self.filtered(lambda x: x.country_code == 'ID')
        other_moves = self - id_moves

        if other_moves:
            other_moves.l10n_id_available_range_count = 0

        if id_moves:
            range_count_per_company = dict(
                self.env['l10n_id_efaktur.efaktur.range']._read_group(
                    [('available', '>', 0), ('company_id', 'in', id_moves.company_id.ids)],
                    ['company_id'],
                    ['__count']
                )
            )
            for company_id, moves in id_moves.grouped('company_id').items():
                moves.l10n_id_available_range_count = range_count_per_company.get(company_id, 0)

    @api.onchange('l10n_id_tax_number')
    def _onchange_l10n_id_tax_number(self):
        for record in self:
            if record.l10n_id_tax_number and record.move_type not in self.get_purchase_types():
                raise UserError(_("You can only change the number manually for a Vendor Bills and Credit Notes"))

    @api.depends('partner_id')
    def _compute_kode_transaksi(self):
        for move in self:
            move.l10n_id_kode_transaksi = move.partner_id.commercial_partner_id.l10n_id_kode_transaksi

    @api.depends('commercial_partner_id', 'invoice_line_ids.tax_ids')
    def _compute_need_kode_transaksi(self):
        for move in self:
            # If there are no taxes at all on every line (0% taxes counts as having a tax) then we don't need a kode transaksi
            move.l10n_id_need_kode_transaksi = (
                move.commercial_partner_id.l10n_id_pkp
                and not move.l10n_id_tax_number
                and move.move_type == 'out_invoice'
                and move.country_code == 'ID'
                and move.invoice_line_ids.tax_ids
            )

    @api.depends('commercial_partner_id')
    def _compute_show_kode_transaksi(self):
        for move in self:
            move.l10n_id_show_kode_transaksi = (
                move.commercial_partner_id.l10n_id_pkp
                and move.move_type == 'out_invoice'
                and move.country_code == 'ID'
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
                # If the code was set on the partner after the invoice was created, we set it on the move at this step so that it triggers the constrains if needed.
                if not move.l10n_id_kode_transaksi and move.commercial_partner_id.l10n_id_kode_transaksi:
                    move.l10n_id_kode_transaksi = move.commercial_partner_id.l10n_id_kode_transaksi
                if not move.l10n_id_kode_transaksi:
                    raise ValidationError(_('You need to put a Kode Transaksi for this partner.'))
                if move.l10n_id_replace_invoice_id.l10n_id_tax_number:
                    if not move.l10n_id_replace_invoice_id.l10n_id_efaktur_document:
                        raise ValidationError(_('Replacement invoice only for invoices on which the e-Faktur is generated. '))
                    rep_efaktur_str = move.l10n_id_replace_invoice_id.l10n_id_tax_number
                    move.l10n_id_tax_number = '%s1%s' % (move.l10n_id_kode_transaksi, rep_efaktur_str[3:])
                else:
                    # Auto-select the smallest range available
                    if not move.l10n_id_efaktur_range:
                        move.l10n_id_efaktur_range = self.env['l10n_id_efaktur.efaktur.range'].search([('company_id', '=', move.company_id.id), ('available', '>', 0)], order="min ASC", limit=1)
                        if not move.l10n_id_efaktur_range:
                            raise ValidationError(_('There is no Efaktur range available. Please configure the range you get from the government in the e-Faktur Ranges menu. '))
                    efaktur_num = move.l10n_id_efaktur_range.pop_number()
                    move.l10n_id_tax_number = '%s0%013d' % (str(move.l10n_id_kode_transaksi), efaktur_num)
        return super()._post(soft)

    def reset_efaktur(self):
        """Reset E-Faktur, so it can be use for other invoice."""
        for move in self:
            if move.l10n_id_efaktur_document:
                raise UserError(_('You have already generated the tax report for this document: %s', move.name))
            self.env['l10n_id_efaktur.efaktur.range'].push_number(move.company_id.id, move.l10n_id_tax_number[3:])
            move.message_post(
                body='e-Faktur Reset: %s ' % (move.l10n_id_tax_number),
                subject="Reset Efaktur")
            move.l10n_id_tax_number = False
        return True

    def download_csv(self):
        return self.l10n_id_efaktur_document.action_download()

    def download_efaktur(self):
        """Collect the data and execute function _generate_efaktur."""
        for record in self:
            if record.state == 'draft':
                raise ValidationError(_('Could not download E-faktur in draft state'))
            if not record.country_code == 'ID':
                raise ValidationError(_("E-faktur is only available on invoices under Indonesian companies"))
            if not record.partner_id.commercial_partner_id.l10n_id_pkp:
                raise ValidationError(_("E-faktur is only available for taxable customers"))
            if not record.move_type == 'out_invoice':
                raise ValidationError(_("E-faktur is only available for invoices"))
            if not record.line_ids.tax_ids:
                raise ValidationError(_('E-faktur is not available for invoices without any taxes.'))
            if not record.l10n_id_tax_number:
                raise ValidationError(_("Please reset %(move_number)s to draft and post it again to generate the eTax number", move_number=record.name))

        # Should prevent users from generating e-Faktur document on invoices across multi-company.
        # Allowing it will cause issues on the invoice/eFaktur document record rule
        if len(self.company_id) > 1:
            raise UserError(_("You are not allowed to generate e-Faktur document from invoices coming from different companies"))

        # All invoices in self have no documents; we can create a new one for them.
        # Or all invoices in self have a document, but it's the same one. Special use case but we allow downloading it.
        if not self.l10n_id_efaktur_document:
            self.l10n_id_efaktur_document = self.env['l10n_id_efaktur.document'].create({
                'invoice_ids': self.ids,
                'company_id': self.company_id.id,
            })
            self.l10n_id_efaktur_document.action_regenerate()
        # If there is more than one document, or all invoices for a document were not selected, the resulting file could cause mistakes;
        # They could get a file with additional invoices for example. In this case, we redirect them to the document view to make it clearer.
        elif len(self.l10n_id_efaktur_document) > 1 or set(self.l10n_id_efaktur_document.invoice_ids.ids) != set(self.ids):
            action_error = {
                'name': _('Document Mismatch'),
                'view_mode': 'list',
                'res_model': 'l10n_id_efaktur.document',
                'type': 'ir.actions.act_window',
                'views': [[False, 'list'], [False, 'form']],
                'domain': [('id', 'in', self.l10n_id_efaktur_document.ids)],
            }
            msg = _("The selected invoices are partially part of one or more e-faktur documents.\n"
                    "Please download them from the e-faktur documents directly.")
            raise RedirectWarning(msg, action_error, _("Display Related Documents"))

        return self.download_csv()

    def _prepare_etax(self):
        # These values are never set
        return {'JUMLAH_PPNBM': 0, 'UANG_MUKA_PPNBM': 0, 'JUMLAH_BARANG': 0, 'TARIF_PPNBM': 0, 'PPNBM': 0}

    def button_draft(self):
        # EXTENDS 'account'
        # When resetting to draft, we want the invoice to be removed from the document they are linked to.
        # That document will be regenerated when downloading later on with only the remaining invoices.
        invoices_with_document = self.filtered(lambda i: i.l10n_id_efaktur_document)
        if invoices_with_document:
            invoices_document = invoices_with_document.l10n_id_efaktur_document

            invoices_document.attachment_id.unlink()
            invoices_with_document.l10n_id_efaktur_document = False

            empty_documents = invoices_document.filtered(lambda d: not d.invoice_ids)
            # We would like to keep them in case the documents in the chatter are important.
            # Users can always delete them manually as needed.
            if empty_documents:
                empty_documents.active = False

            body = _("This invoice has been unlinked from the e-faktur document %(document_link)s following the reset to draft.",
                     document_link=invoices_document._get_html_link(title=f"{invoices_document.id}"))
            invoices_with_document._message_log_batch(bodies={inv.id: body for inv in invoices_with_document})
        return super().button_draft()

```

## File: models\efaktur.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError

import re


class Efaktur(models.Model):
    _name = "l10n_id_efaktur.efaktur.range"
    _description = "Available E-faktur range"
    _rec_names_search = ["min", "max"]

    company_id = fields.Many2one('res.company', required=True, default=lambda self: self.env.company)
    max = fields.Char(required=True)
    min = fields.Char(required=True)
    available = fields.Integer(readonly=True)
    next_num = fields.Integer(compute="_compute_next_num")

    @api.depends('max', 'available')
    def _compute_next_num(self):
        for record in self:
            record.next_num = int(record.max) - record.available + 1

    def pop_number(self):
        """ Consume the availability of a specific range to generate the eTax number for an invoice"""
        self.ensure_one()

        popped = self.next_num
        self.available -= 1

        return popped

    @api.model
    def push_number(self, company_id, number):
        """ Restoring the eTax number that got released after doing reset eFaktur on an invoice so
        that it can be reused

        :param company_id (int): company ID
        :param number (str): number to be restored
        """
        number_int = int(number)
        efaktur_range = self.search([('company_id', '=', company_id), ('min', '<=', number), ('max', '>=', number)], limit=1)

        # if the released number is the last popped number from the range, we simply extend the availability
        if efaktur_range.next_num == int(number) + 1:
            efaktur_range.available += 1
            return

        # if the released number is not the last used number from any range, find the range it belongs and split it
        # i.e range: 1 - 10, available=5, next_number=6, release 3
        # split 1-10 to 1-3 available=1 AND 4-10 available=5
        if efaktur_range:
            if efaktur_range.min == efaktur_range.max:
                efaktur_range.available += 1
            else:
                maximum = efaktur_range.max
                available = efaktur_range.available
                efaktur_range.write({
                    'available': 1,
                    'max': '%013d' % number_int
                })

                self.create({
                    'company_id': company_id,
                    'min': '%013d' % (number_int + 1),
                    'max': maximum,
                    'available': available  # keep original available to not change next_num of that range
                })
        else:
            # in older versions, min value of a range increases after being used
            # to handle the case post migration, we will create the range
            # containing only that number instead
            self.create({
                'company_id': company_id,
                'min': '%013d' % number_int,
                'max': '%013d' % number_int,
                'available': 1
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
            if self.search_count([
                '&', ('id', '!=', record.id), '|', '|',
                '&', ('min', '<=', record.max), ('max', '>=', record.max),
                '&', ('min', '<=', record.min), ('max', '>=', record.min),
                '&', ('min', '>=', record.min), ('max', '<=', record.max),
            ], limit=1):
                raise ValidationError(_('Efaktur interleaving range detected'))

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            # available could be part of vals in case a new range is created
            # due to range being split in two when a number is reset
            if 'available' not in vals:
                vals['available'] = 1 + int(vals['max']) - int(vals['min'])
        return super().create(vals_list)

    def write(self, vals):
        """ Override to determine behaviour of changing min and max of an e-Faktur range

        For unused ranges, availability lowers when minimum is increased or maximum is decreased. Vice versa applies.
        For used ranges, minimum is fixed while maximum can only be udpated to a value above the used number. Availability
        decreases when the maximum is decreased and vice versa.
        """
        diff = 0
        if 'max' in vals and 'available' not in vals:
            # A new max should never be lower than an already used number
            if int(vals['max']) < self.next_num and 'available' not in vals:
                raise ValidationError(_("You are not allowed to change the max to a number lower than already used"))
            diff += int(vals['max']) - int(self.max)
        if 'min' in vals and 'available' not in vals:
            # A new min cannot be set on a used range
            if self.next_num > int(self.min):
                raise ValidationError(_("You are not allowed to change the min of a range that is already in use"))
            # if range was unused, adapt it according to how min is changed
            if int(self.min) == self.next_num:
                diff += int(self.min) - int(vals['min'])
        if diff:
            vals['available'] = self.available + diff
        return super().write(vals)

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

    @api.depends('min', 'max')
    def _compute_display_name(self):
        for efaktur in self:
            efaktur.display_name = "%s - %s" % (efaktur.min, efaktur.max)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_used_ranges(self):
        """ Only allow deletion on ranges that is unused"""
        if any(efaktur_range.next_num > int(efaktur_range.min) for efaktur_range in self):
            raise UserError(_("You can not delete eFaktur range that has been used to generate an eTax number"))

```

## File: models\efaktur_document.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, RedirectWarning
from odoo.tools import float_repr, float_round

FK_HEAD_LIST = ['FK', 'KD_JENIS_TRANSAKSI', 'FG_PENGGANTI', 'NOMOR_FAKTUR', 'MASA_PAJAK', 'TAHUN_PAJAK', 'TANGGAL_FAKTUR', 'NPWP', 'NAMA', 'ALAMAT_LENGKAP', 'JUMLAH_DPP', 'JUMLAH_PPN', 'JUMLAH_PPNBM', 'ID_KETERANGAN_TAMBAHAN', 'FG_UANG_MUKA', 'UANG_MUKA_DPP', 'UANG_MUKA_PPN', 'UANG_MUKA_PPNBM', 'REFERENSI', 'KODE_DOKUMEN_PENDUKUNG']

LT_HEAD_LIST = ['LT', 'NPWP', 'NAMA', 'JALAN', 'BLOK', 'NOMOR', 'RT', 'RW', 'KECAMATAN', 'KELURAHAN', 'KABUPATEN', 'PROPINSI', 'KODE_POS', 'NOMOR_TELEPON']

OF_HEAD_LIST = ['OF', 'KODE_OBJEK', 'NAMA', 'HARGA_SATUAN', 'JUMLAH_BARANG', 'HARGA_TOTAL', 'DISKON', 'DPP', 'PPN', 'TARIF_PPNBM', 'PPNBM']


def _csv_row(data, delimiter=',', quote='"'):
    return quote + (quote + delimiter + quote).join([str(x).replace(quote, '\\' + quote) for x in data]) + quote + '\n'


class EfakturDocument(models.Model):
    _name = "l10n_id_efaktur.document"
    _description = "E-faktur Document"
    _inherit = ["mail.thread", "mail.activity.mixin"]

    name = fields.Char(
        compute='_compute_name',
        store=True,
        readonly=False,
        required=True,
        precompute=True,
    )
    company_id = fields.Many2one('res.company', required=True, readonly=True, default=lambda self: self.env.company)
    active = fields.Boolean(
        string="Active",
        default=True,
    )
    invoice_ids = fields.One2many(
        comodel_name="account.move",
        inverse_name="l10n_id_efaktur_document",
        domain="[('move_type', 'in', ['out_invoice', 'out_refund']), ('company_id', '=', company_id), ('l10n_id_efaktur_document', '=', False), ('l10n_id_tax_number', '!=', False), ('state', '=', 'posted')]",
        tracking=True,
    )
    attachment_id = fields.Many2one(comodel_name="ir.attachment", readonly=True)

    def action_download(self):
        """ Download the e-faktur related attachment """
        for document in self:
            if not document.attachment_id:
                document._generate_csv()
        return {
            'type': 'ir.actions.act_url',
            'url': f'/l10n_id_efaktur/download_attachments/{",".join(map(str, self.attachment_id.ids))}',
        }

    def action_regenerate(self):
        """ Regenerate the e-faktur csv file, based on the invoice in the document.
        All new file generation will log a copy of the attachment to keep track of past generations.
        """
        self._generate_csv()

    def _generate_csv(self, delimiter=','):
        self.ensure_one()
        if self.invoice_ids.filtered(lambda x: not x.l10n_id_kode_transaksi):
            raise UserError(_("Some documents don't have a transaction code"))
        if self.invoice_ids.filtered(lambda x: x.move_type != 'out_invoice'):
            raise UserError(_("Some documents are not Customer Invoices"))

        output_head = self._generate_efaktur_invoice(delimiter)
        raw_data = output_head.encode("utf-8")

        # create/update attachment and link it to efaktur document if new
        if not self.attachment_id:
            attachment = self.env['ir.attachment'].create({
                'raw': raw_data,
                'name': 'efaktur_%s.csv' % (fields.Datetime.to_string(fields.Datetime.now()).replace(" ", "_")),
                'type': 'binary',
                'res_model': 'l10n_id_efaktur.document',
                'res_id': self.id,
            })
            self.attachment_id = attachment.id
        else:
            attachment = self.attachment_id
            self.attachment_id.write({
                'raw': raw_data,
                'name': 'efaktur_%s.csv' % (fields.Datetime.to_string(fields.Datetime.now()).replace(" ", "_")),
            })

        self.message_post(
            body=_("The e-Faktur report has been generated"),
            attachments=[(attachment.name, attachment.raw)]
        )

    def _generate_efaktur_invoice(self, delimiter=','):
        """Generate E-Faktur for customer invoice."""
        # Invoice of Customer

        output_head = '%s%s%s' % (
            _csv_row(FK_HEAD_LIST, delimiter),
            _csv_row(LT_HEAD_LIST, delimiter),
            _csv_row(OF_HEAD_LIST, delimiter),
        )

        idr = self.env.ref('base.IDR')

        for move in self.invoice_ids.filtered(lambda m: m.state == 'posted'):
            eTax = move._prepare_etax()

            commercial_partner = move.partner_id.commercial_partner_id
            nik = str(commercial_partner.l10n_id_nik) if not commercial_partner.vat else ''

            if move.l10n_id_replace_invoice_id:
                number_ref = str(move.l10n_id_replace_invoice_id.name) + " replaced by " + str(move.name) + " " + nik
            elif nik:
                number_ref = str(move.name) + " " + nik
            else:
                number_ref = str(move.name)

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

            etax_name = commercial_partner.name
            if invoice_npwp[:15] == '000000000000000' and commercial_partner.l10n_id_nik:
                etax_name = "%s#NIK#NAMA#%s" % (commercial_partner.l10n_id_nik, etax_name)

            # Here all fields or columns based on eTax Invoice Third Party
            eTax['KD_JENIS_TRANSAKSI'] = move.l10n_id_tax_number[0:2] or 0
            eTax['FG_PENGGANTI'] = move.l10n_id_tax_number[2:3] or 0
            eTax['NOMOR_FAKTUR'] = move.l10n_id_tax_number[3:] or 0
            eTax['MASA_PAJAK'] = move.invoice_date.month
            eTax['TAHUN_PAJAK'] = move.invoice_date.year
            eTax['TANGGAL_FAKTUR'] = move.invoice_date.strftime("%-d/%-m/%Y")
            eTax['NPWP'] = invoice_npwp
            eTax['NAMA'] = etax_name
            eTax['ALAMAT_LENGKAP'] = move.partner_id._display_address(without_company=True).replace('\n', ' ').replace('  ', ' ').strip()
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
                diff_dpp = idr.round(eTax['JUMLAH_DPP'] - sum(sale['DPP'] for sale in sales))
                total_sales_ppn = idr.round(eTax['JUMLAH_PPN'] - sum(sale['PPN'] for sale in sales))
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

    @api.depends('invoice_ids')
    def _compute_name(self):
        """ First compute will be done at creation, from a selection of invoice(s).
        We still want to allow to rename the document to another name if it makes sense.
        """
        for doc in self:
            sorted_invoices = doc.invoice_ids.sorted('name')
            name = []
            if sorted_invoices:
                name.append(sorted_invoices[0].name)
                if len(sorted_invoices) > 1:
                    name.append(sorted_invoices[-1].name)
            doc.name = "%s - Efaktur (%s)" % (fields.Date.context_today(doc).strftime("%Y%m%d"), "....".join(name))

```

## File: models\res_partner.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResPartner(models.Model):
    """Inherit res.partner object to add NPWP field and Kode Transaksi"""
    _inherit = "res.partner"

    l10n_id_pkp = fields.Boolean(string="Is PKP", compute='_compute_l10n_id_pkp', store=True, readonly=False, help="Denoting whether the following partner is taxable")
    l10n_id_nik = fields.Char(string='NIK')
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
        string='Invoice Transaction Code',
        help='Dua digit pertama nomor pajak',
        default='01',
        tracking=True,
    )

    @api.depends('vat', 'country_code')
    def _compute_l10n_id_pkp(self):
        for record in self:
            record.l10n_id_pkp = record.vat and record.country_code == 'ID'

```

## File: models\__init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import efaktur
from . import account_move
from . import res_partner
from . import efaktur_document

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_efaktur_user,access.efaktur.user,model_l10n_id_efaktur_efaktur_range,account.group_account_invoice,1,1,1,1
access_efaktur_document_user,access.efkatur.document.user,model_l10n_id_efaktur_document,account.group_account_invoice,1,1,1,1

```

## File: security\ir_rule.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="efaktur_document_multi_company" model="ir.rule">
        <field name="name">E-Faktur document multi-company</field>
        <field name="model_id" ref="model_l10n_id_efaktur_efaktur_range"/>
        <field name="domain_force">[('company_id', 'in', company_ids)]</field>
    </record>
</odoo>

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
                <field name="partner_shipping_id" position="before">
                    <field name="l10n_id_need_kode_transaksi" invisible="1"/>
                </field>
                <button name="button_draft" position="after">
                    <button name="reset_efaktur" string="Reset E-Faktur" type="object" invisible="country_code != 'ID' or not l10n_id_tax_number or state != 'cancel' or l10n_id_efaktur_document"/>
                </button>
                <xpath expr=".//group[@id='other_tab_group']" position="inside">
                    <group string="Electronic Tax" invisible="country_code != 'ID'">
                        <field name="l10n_id_kode_transaksi" invisible="not l10n_id_show_kode_transaksi" readonly="l10n_id_tax_number" required="l10n_id_need_kode_transaksi"/>
                        <field name="l10n_id_efaktur_range" invisible="state != 'draft' or l10n_id_available_range_count &lt; 2 or not l10n_id_show_kode_transaksi" readonly="l10n_id_tax_number" options="{'no_create': True}"/>
                        <field name="l10n_id_tax_number" invisible="move_type == 'entry'" readonly="move_type in ('out_invoice', 'out_refund', 'out_receipt')"/>
                        <field name="l10n_id_efaktur_document"/>
                        <field name="l10n_id_replace_invoice_id" readonly="state != 'draft'" options="{'m2o_dialog': False, 'no_create': True}"/>
                    </group>
                </xpath>
            </field>
        </record>

        <record id="account_move_efaktur_tree_view" model="ir.ui.view">
            <field name="name">account.move.efaktur.list.view</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_invoice_tree"/>
            <field name="arch" type="xml">
                <field name="invoice_user_id" position="after">
                    <field name="l10n_id_tax_number" optional="show"/>
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
                    <field name="l10n_id_efaktur_document"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\efaktur_document_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="l10n_id_efaktur_document_form_view" model="ir.ui.view">
            <field name="name">l10n_id.efaktur.document.form.view</field>
            <field name="model">l10n_id_efaktur.document</field>
            <field name="arch" type="xml">
                <form string="E-faktur Document" create="false">
                    <header>
                        <button name="action_download" string="Download" class="btn-primary"
                                type="object" groups="account.group_account_invoice" data-hotkey="q"
                                invisible="not invoice_ids"/>
                        <button name="action_regenerate" string="Regenerate File" class="btn-secondary"
                                type="object" groups="account.group_account_invoice" data-hotkey="q"
                                invisible="not invoice_ids"/>
                    </header>
                    <sheet>
                        <div class="oe_title">
                            <h1>
                                <field name="name" placeholder="Draft"/>
                            </h1>
                        </div>
                        <group>
                            <group>
                                <field name="company_id"/>
                            </group>
                        </group>
                        <notebook>
                            <page id="invoice_tab" name="invoice_tab" string="Invoices">
                                <field name="invoice_ids" force_save="1" widget="many2many">
                                    <list create="False" edit="False">
                                        <field name="name"/>
                                        <field name="invoice_date" optional="show" string="Accounting Date"/>
                                        <field name="amount_untaxed_in_currency_signed" string="Tax Excluded" sum="Total" optional="show"/>
                                        <field name="amount_tax_signed" string="Tax" sum="Total" optional="hide"/>
                                        <field name="amount_total_in_currency_signed" string="Total" sum="Total" optional="show"/>
                                        <field name="currency_id" optional="hide"/>
                                        <field name="status_in_payment"
                                               string="Status"
                                               widget="badge"
                                               optional="show"
                                        />
                                    </list>
                                </field>
                            </page>
                        </notebook>
                    </sheet>
                    <chatter/>
                </form>
            </field>
        </record>

        <record id="l10n_id_efaktur_document_list_view" model="ir.ui.view">
            <field name="name">l10n_id.efaktur.document.list.view</field>
            <field name="model">l10n_id_efaktur.document</field>
            <field name="arch" type="xml">
                <list string="E-faktur Document" create="false" sample="1">
                    <header>
                        <button name="action_download" type="object" string="Download" groups="account.group_account_user"/>
                    </header>
                    <field name="name"/>
                    <field name="invoice_ids" options="{'no_quick_create': True}" widget="many2many_tags"/>
                </list>
            </field>
        </record>

        <record id="l10n_id_efaktur_document_filter_view" model="ir.ui.view">
            <field name="name">l10n_id.efaktur.document.filter.view</field>
            <field name="model">l10n_id_efaktur.document</field>
            <field name="arch" type="xml">
                <search string="Search Document">
                    <filter string="Active" name="active" domain="[('active', '=', True)]"/>
                    <filter string="Inactive" name="inactive" domain="[('active', '=', False)]"/>
                </search>
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
            <field name="name">l10n_id_efaktur.efaktur.range.list.view</field>
            <field name="model">l10n_id_efaktur.efaktur.range</field>
            <field name="arch" type="xml">
                <list string="Efaktur Number" editable="bottom">
                    <field name="min"/>
                    <field name="max"/>
                    <field name="available" sum="Total Available"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                    <field name="company_id" column_invisible="True" groups="!base.group_multi_company"/>
                </list>
            </field>
        </record>

        <record id='efaktur_invoice_action' model='ir.actions.act_window'>
            <field name="name">e-Faktur Ranges</field>
            <field name="res_model">l10n_id_efaktur.efaktur.range</field>
            <field name="view_mode">list</field>
            <field name="context">{'search_default_upload': True, 'search_default_used': True}</field>
            <field name="view_id" ref="efaktur_tree_view"/>
            <field name="help" type="html">
                <p class="oe_view_nocontent_create">
                    Put here the ranges of number you were assigned by the government. Those
                    will be assigned to your Customer Invoices upon confirmation.
                </p>
            </field>
        </record>


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
            <xpath expr="//block[@id='invoicing_settings']" position="after">
                <block title="Indonesian Localization" id="l10n_cl_section" invisible="country_code != 'ID'">
                    <setting id="l10n_id_efaktur">
                        <div class="content-group">
                            <button type="action" name="%(l10n_id_efaktur.efaktur_invoice_action)d" string="Configure Your e-Faktur Ranges" class="btn-link" icon="oi-arrow-right"/>
                        </div>
                    </setting>
                </block>
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
                    <field name="l10n_id_pkp" invisible="country_id and country_code != 'ID'"/>
                    <field name="l10n_id_kode_transaksi" invisible="not l10n_id_pkp"/>
                </xpath>
                <page name="accounting" position="inside">
                    <group string="Indonesian Taxes"  invisible="not l10n_id_pkp">
                        <group>
                            <field name="l10n_id_nik"/>
                        </group>
                    </group>
                </page>
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

    def _modify_default_reverse_values(self, origin_move):
        # EXTEND 'account'
        values = super()._modify_default_reverse_values(origin_move)

        # The replacement eFaktur is to "correct" a detail from the original. If an e-Faktur
        # invoice has been sent to the government and the user needs to adjust it, they must send
        # an adjustment invoice, which refers to the original invoice in its tax number.
        if origin_move.l10n_id_efaktur_document:
            values.update({
                'l10n_id_replace_invoice_id': origin_move.id
            })
        return values

```

## File: wizard\__init__.py

```python
from . import account_move_reversal

```

