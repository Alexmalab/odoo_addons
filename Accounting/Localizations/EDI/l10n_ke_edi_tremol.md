# Odoo Module: l10n_ke_edi_tremol

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Kenya Tremol Device EDI Integration",
    'summary': """
            Kenya Tremol Device EDI Integration
        """,
    'description': """
       This module integrates with the Kenyan G03 Tremol control unit device to the KRA through TIMS.
    """,
    'author': 'Odoo',
    'category': 'Accounting/Localizations/EDI',
    'version': '1.0',
    'license': 'LGPL-3',
    'depends': ['l10n_ke'],
    'data': [
        'views/account_move_view.xml',
        'views/product_view.xml',
        'views/report_invoice.xml',
        'views/res_config_settings_view.xml',
        'views/res_partner_views.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'l10n_ke_edi_tremol/static/src/js/send_invoice.js',
        ],
    },
}

```

## File: data\neutralize.sql

```sql
-- neutralize connection to tremol controle unit
UPDATE res_company
SET l10n_ke_cu_proxy_address = '';
```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import json
import re
from datetime import datetime

from odoo import models, fields, _
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)

class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_ke_cu_datetime = fields.Datetime(string='CU Signing Date and Time', copy=False)
    l10n_ke_cu_serial_number = fields.Char(string='CU Serial Number', copy=False)
    l10n_ke_cu_invoice_number = fields.Char(string='CU Invoice Number', copy=False)
    l10n_ke_cu_qrcode = fields.Char(string='CU QR Code', copy=False)

    # -------------------------------------------------------------------------
    # HELPERS
    # -------------------------------------------------------------------------

    def _l10n_ke_fmt(self, string, length, ljust=True):
        """ Function for common formatting behaviour

        :param string: string to be formatted/encoded
        :param length: integer length to justify (if enabled), and then truncate the string to
        :param ljust:  boolean representing whether the string should be justified
        :returns:      byte-string justified/truncated, with all non-alphanumeric characters removed
        """
        if not string:
            string = ''
        return re.sub('[^A-Za-z0-9 ]+', '', str(string)).encode('cp1251').ljust(length if ljust else 0)[:length]

    # -------------------------------------------------------------------------
    # CHECKS
    # -------------------------------------------------------------------------

    def _l10n_ke_validate_move(self):
        """ Returns list of errors related to misconfigurations per move

        Find misconfigurations on the move, the lines of the move, and the
        taxes on those lines that would result in rejection by the KRA.
        """
        errors = []
        for move in self:
            move_errors = []
            if move.country_code != 'KE':
                move_errors.append(_("This invoice is not a Kenyan invoice and therefore can not be sent to the device."))

            if move.company_id.currency_id != self.env.ref('base.KES'):
                move_errors.append(_("This invoice's company currency is not in Kenyan Shillings, conversion to KES is not possible."))

            if move.state != 'posted':
                move_errors.append(_("This invoice/credit note has not been posted. Please confirm it to continue."))

            if move.move_type not in ('out_refund', 'out_invoice'):
                move_errors.append(_("The document being sent should be an invoice or credit note."))

            if any([move.l10n_ke_cu_invoice_number, move.l10n_ke_cu_serial_number, move.l10n_ke_cu_qrcode, move.l10n_ke_cu_datetime]):
                move_errors.append(_("The document already has details related to the fiscal device. Please make sure that the invoice has not already been sent."))

            # The credit note should refer to the control unit number (receipt number) of the original
            # invoice to which it relates.
            if move.move_type == 'out_refund' and not move.reversed_entry_id.l10n_ke_cu_invoice_number:
                move_errors.append(_("This credit note must reference the previous invoice, and this previous invoice must have already been submitted."))

            for line in self.invoice_line_ids.filtered(lambda l: l.display_type == 'product'):
                vat_taxes = line.tax_ids.filtered(lambda tax: tax.amount in (16, 8, 0))
                if not vat_taxes or len(vat_taxes) > 1:
                    move_errors.append(_("On line %s, you must select one and only one VAT tax.", line.name))
                else:
                    if vat_taxes[0].amount == 0 and not (line.product_id and line.product_id.l10n_ke_hsn_code and line.product_id.l10n_ke_hsn_name):
                        move_errors.append(_("On line %s, a product with a HS Code and HS Name must be selected, since the tax is 0%% or exempt.", line.name))

            if move_errors:
                errors.append((move.name, move_errors))

        return errors

    # -------------------------------------------------------------------------
    # SERIALISERS
    # -------------------------------------------------------------------------

    def _l10n_ke_cu_open_invoice_message(self):
        """ Serialise the required fields for opening an invoice

        :returns: a list containing one byte-string representing the <CMD> and
                  <DATA> of the message sent to the fiscal device.
        """
        headquarter_address = (self.commercial_partner_id.street or '') + (self.commercial_partner_id.street2 or '')
        customer_address = (self.partner_id.street or '') + (self.partner_id.street2 or '')
        postcode_and_city = (self.partner_id.zip or '') + '' +  (self.partner_id.city or '')
        vat = (self.commercial_partner_id.vat or '').strip() if self.commercial_partner_id.country_id.code == 'KE' else ''
        invoice_elements = [
            b'1',                                                   # Reserved - 1 symbol with value '1'
            b'     0',                                              # Reserved - 6 symbols with value ‘     0’
            b'0',                                                   # Reserved - 1 symbol with value '0'
            b'1' if self.move_type == 'out_invoice' else b'A',      # 1 symbol with value '1' (new invoice), 'A' (credit note), or '@' (debit note)
            self._l10n_ke_fmt(self.commercial_partner_id.name, 30), # 30 symbols for Company name
            self._l10n_ke_fmt(vat, 14),                             # 14 Symbols for the client PIN number
            self._l10n_ke_fmt(headquarter_address, 30),             # 30 Symbols for customer headquarters
            self._l10n_ke_fmt(customer_address, 30),                # 30 Symbols for the address
            self._l10n_ke_fmt(postcode_and_city, 30),               # 30 symbols for the customer post code and city
            self._l10n_ke_fmt('', 30),                              # 30 symbols for the exemption number
        ]
        if self.move_type == 'out_refund':
            invoice_elements.append(self._l10n_ke_fmt(self.reversed_entry_id.l10n_ke_cu_invoice_number, 19)), # 19 symbols for related invoice number
        invoice_elements.append(re.sub('[^A-Za-z0-9 ]+', '', self.name)[-15:].ljust(15).encode('cp1251'))     # 15 symbols for trader system invoice number

        # Command: Open fiscal record (0x30)
        return [b'\x30' + b';'.join(invoice_elements)]

    def _l10n_ke_cu_lines_messages(self):
        """ Serialise the data of each line on the invoice

        This function transforms the lines in order to handle the differences
        between the KRA expected data and the lines in odoo.

        If a discount line (as a negative line) has been added to the invoice
        lines, find a suitable line/lines to distribute the discount accross

        :returns: List of byte-strings representing each command <CMD> and the
                  <DATA> of the line, which will be sent to the fiscal device
                  in order to add a line to the opened invoice.
        """
        def is_discount_line(line):
            return line.price_subtotal < 0.0

        def is_candidate(discount_line, other_line):
            """ If the of one line match those of the discount line, the discount can be distributed accross that line """
            discount_taxes = discount_line.tax_ids.flatten_taxes_hierarchy()
            other_line_taxes = other_line.tax_ids.flatten_taxes_hierarchy()
            return set(discount_taxes.ids) == set(other_line_taxes.ids)

        lines = self.invoice_line_ids.filtered(lambda l: l.display_type == 'product' and l.quantity and l.price_total)
        # The device expects all monetary values in Kenyan Shillings
        if self.currency_id == self.company_id.currency_id:
            currency_rate = 1
        # In the case of a refund, use the currency rate of the original invoice
        elif self.move_type == 'out_refund' and self.reversed_entry_id:
            currency_rate = abs(self.reversed_entry_id.amount_total_signed / self.reversed_entry_id.amount_total)
        else:
            currency_rate = abs(self.amount_total_signed / self.amount_total)

        discount_dict = {line.id: line.discount for line in lines if line.price_total > 0}
        for line in lines:
            if not is_discount_line(line):
                continue
            # Search for non-discount lines
            candidate_vals_list = [l for l in lines if not is_discount_line(l) and is_candidate(l, line)]
            candidate_vals_list = sorted(candidate_vals_list, key=lambda x: x.price_unit * x.quantity, reverse=True)
            line_to_discount = abs(line.price_unit * line.quantity)
            for candidate in candidate_vals_list:
                still_to_discount = abs(candidate.price_unit * candidate.quantity * (100.0 - discount_dict[candidate.id]) / 100.0)
                if line_to_discount >= still_to_discount:
                    discount_dict[candidate.id] = 100.0
                    line_to_discount -= still_to_discount
                else:
                    rest_to_discount = abs((line_to_discount / (candidate.price_unit * candidate.quantity)) * 100.0)
                    discount_dict[candidate.id] += rest_to_discount
                    break

        vat_class = {16.0: 'A', 8.0: 'B'}
        msgs = []
        tax_details = self._prepare_invoice_aggregated_taxes()
        for line in self.invoice_line_ids.filtered(lambda l: l.display_type == 'product' and l.quantity and l.price_total > 0 and not discount_dict.get(l.id) >= 100):
            # Here we use the original discount of the line, since it the distributed discount has not been applied in the price_total
            price_total = 0
            percentage = 0
            for tax in tax_details['tax_details_per_record'][line]['tax_details']:
                if tax['tax'].amount in (16, 8, 0): # This should only occur once
                    line_tax_details = tax_details['tax_details_per_record'][line]['tax_details'][tax]
                    price_total = abs(line_tax_details['base_amount_currency']) + abs(line_tax_details['tax_amount_currency'])
                    percentage = tax['tax'].amount

            price = round(price_total / abs(line.quantity) * 100 / (100 - line.discount), line.currency_id.decimal_places) * currency_rate
            price = ('%.5f' % price).rstrip('0').rstrip('.')

            # Letter to classify tax, 0% taxes are handled conditionally, as the tax can be zero-rated or exempt
            letter = ''
            if percentage in vat_class:
                letter = vat_class[percentage]
            else:
                report_line_ids = line.tax_ids.invoice_repartition_line_ids.tag_ids._get_related_tax_report_expressions().report_line_id.ids
                try:
                    exempt_report_line = self.env.ref('l10n_ke.tax_report_line_exempt_sales')
                except ValueError:
                    raise UserError(_("Tax exempt report line cannot be found, please update the l10n_ke module."))
                letter = 'E' if exempt_report_line.id in report_line_ids else 'C'

            uom = line.product_uom_id and line.product_uom_id.name or ''
            hscode = re.sub('[^0-9.]+', '', line.product_id.l10n_ke_hsn_code)[:10].ljust(10).encode('cp1251') if letter not in ('A', 'B') else b''.ljust(10)
            hsname = self._l10n_ke_fmt(line.product_id.l10n_ke_hsn_name, 20) if letter not in ('A', 'B') else b''.ljust(20)
            line_data = b';'.join([
                self._l10n_ke_fmt(line.name, 36),               # 36 symbols for the article's name
                self._l10n_ke_fmt(letter, 1),                   # 1 symbol for article's vat class ('A', 'B', 'C', 'D', or 'E')
                price[:15].encode('cp1251'),                    # 1 to 15 symbols for article's price with up to 5 digits after decimal point
                self._l10n_ke_fmt(uom, 3),                      # 3 symbols for unit of measure
                hscode,                                         # 10 symbols for HS code in the format xxxx.xx.xx (can be empty)
                hsname,                                         # 20 symbols for the HS name (can be empty)
                str(percentage).encode('cp1251')[:5]            # up to 5 symbols for vat rate
            ])
            # 1 to 10 symbols for quantity
            line_data += b'*' + str(abs(line.quantity)).encode('cp1251')[:10]
            if discount_dict.get(line.id):
                # 1 to 7 symbols for percentage of discount/addition
                discount_sign = b'-' if discount_dict[line.id] > 0 else b'+'
                discount = discount_sign + str(abs(discount_dict[line.id])).encode('cp1251')[:6]
                line_data += b',' + discount + b'%'

            # Command: Sale of article (0x31)
            msgs += [b'\x31' + line_data]
        return msgs

    def _l10n_ke_get_cu_messages(self):
        """ Composes a list of all the command and data parts of the messages
            required for the fiscal device to open an invoice, add lines and
            subsequently close it.
        """
        self.ensure_one()
        msgs = self._l10n_ke_cu_open_invoice_message()
        msgs += self._l10n_ke_cu_lines_messages()
        # Command: Close fiscal reciept (0x38)
        msgs += [b'\x38']
        # Command: Read date and time (0x68)
        msgs += [b'\x68']
        return msgs

    # -------------------------------------------------------------------------
    # POST COMMANDS / RECEIVE DATA
    # -------------------------------------------------------------------------

    def l10n_ke_action_cu_post(self):
        """ Returns the client action descriptor dictionary for sending the
            invoice(s) to the fiscal device.
        """
        # Check the configuration of the invoice
        errors = self._l10n_ke_validate_move()
        if errors:
            error_msg = ""
            for move, error_list in errors:
                error_list = '\n'.join(error_list)
                error_msg += _("Invalid invoice configuration on %s:\n%s\n\n", move, error_list)
            raise UserError(error_msg)
        return {
            'type': 'ir.actions.client',
            'tag': 'post_send',
            'params': {
                'invoices': {
                    move.id: {
                        'messages': json.dumps([msg.decode('cp1251') for msg in move._l10n_ke_get_cu_messages()]),
                        'proxy_address': move.company_id.l10n_ke_cu_proxy_address,
                        'company_vat': move.company_id.vat
                    } for move in self
                }
            }
        }

    def l10n_ke_cu_response(self, response):
        """ Set the fields related to the fiscal device on the invoice.

        This is intended to be utilized by an RPC call from the javascript
        client action.
        """
        move = self.browse(int(response['move_id']))
        replies = [msg for msg in response['replies']]
        move.update({
            'l10n_ke_cu_serial_number': response['serial_number'],
            'l10n_ke_cu_invoice_number': replies[-2].split(';')[0],
            'l10n_ke_cu_qrcode': replies[-2].split(';')[1].strip(),
            'l10n_ke_cu_datetime': datetime.strptime(replies[-1], '%d-%m-%Y %H:%M'),
        })

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ProductTemplate(models.Model):
    _inherit = "product.template"

    l10n_ke_hsn_code = fields.Char(
        string='KRA Item Code',
        help='Product code needed when not 16% VAT rated. ',
    )
    l10n_ke_hsn_name = fields.Char(
        string='KRA Item Description',
        help='Product code description needed when not 16% VAT rated. ',
    )

class ProductProduct(models.Model):
    _inherit = "product.product"

    l10n_ke_hsn_code = fields.Char(
        string='HSN code',
        related='product_tmpl_id.l10n_ke_hsn_code',
        help="Product code needed in case of not 16%. ",
        readonly=False,
    )
    l10n_ke_hsn_name = fields.Char(
        string='HSN description',
        related='product_tmpl_id.l10n_ke_hsn_name',
        help="Product code description needed in case of not 16%. ",
        readonly=False,
    )

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_ke_cu_proxy_address = fields.Char(
        default="http://localhost:8069",
        string='Fiscal Device Proxy Address',
        help='The address of the proxy server for the fiscal device.',
    )

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_ke_cu_proxy_address = fields.Char(related='company_id.l10n_ke_cu_proxy_address', readonly=False)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_ke_exemption_number = fields.Char(
        string='Exemption Number',
        help='The exemption number of the partner. Provided by the Kenyan government.',
    )

    def _commercial_fields(self):
        return super()._commercial_fields() + ['l10n_ke_exemption_number']

```

## File: models\__init__.py

```python
from . import account_move
from . import product
from . import res_company
from . import res_config_settings
from . import res_partner

```

## File: static\src\js\send_invoice.js

```javascript
odoo.define('l10n_ke_edi_tremol.action_post_send_invoice', function (require) {
    const core = require('web.core');
    const ajax = require('web.ajax');
    const Dialog = require('web.Dialog');
    var rpc = require('web.rpc');
    var _t = core._t;

    async function post_send(parent, {params}) {
        let refresh = false;
        for (let move_id in params.invoices) {
            try {
                const res = await ajax.post(
                    params.invoices[move_id].proxy_address + '/hw_proxy/l10n_ke_cu_send', {
                        messages: params.invoices[move_id].messages,
                        company_vat: params.invoices[move_id].company_vat
                    }
                );
                const res_obj = JSON.parse(res);
                if (res_obj.status === "ok") {
                    try {
                        await rpc.query({
                            model: 'account.move',
                            method: 'l10n_ke_cu_response',
                            args: [[], {'replies': res_obj.replies, 'serial_number': res_obj.serial_number, 'move_id': move_id}],
                        });
                        refresh = true;
                    } catch (_e) {
                        Dialog.alert(this, _t("Error trying to connect to Odoo. Check your internet connection"));
                        break;
                    }
                } else {
                    Dialog.alert(this, _t("Posting an invoice has failed, with the message: \n") + res_obj.status);
                    break;
                }
            } catch(_e) {
                Dialog.alert(this, _t("Error trying to connect to the middleware. Is the middleware running?"));
                break;
            }
        }
        if (refresh) {
            parent.services.action.doAction({
                'type': 'ir.actions.client',
                'tag': 'reload',
            });
        }
    }
    core.action_registry.add('post_send', post_send);
    return post_send;
});

```

## File: views\account_move_view.xml

```xml
<odoo>
    <data>
        <record id="l10n_ke_inherit_account_move_form" model="ir.ui.view">
            <field name="name">l10n.ke.inherit.account.move.form</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="priority" eval="40"/>
            <field name="arch" type="xml">
                <xpath expr="//header/button[@name='action_post']" position="after">
                    <field name="l10n_ke_cu_qrcode" invisible="1"/>
                    <button name="l10n_ke_action_cu_post" type="object"
                            class="oe_highlight"
                            groups="account.group_account_manager"
                            string="Send To Fiscal Device"
                            attrs="{'invisible': ['|', '|', '|', ('country_code', '!=', 'KE'), ('l10n_ke_cu_qrcode', '!=', False), ('state', '!=', 'posted'), ('move_type', 'not in', ['out_invoice', 'out_refund'])]}"/>
                </xpath>
                <xpath expr="//group[@id='header_right_group']" position="inside">
                    <field name="l10n_ke_cu_invoice_number" attrs="{'invisible': [('country_code', '!=', 'KE')]}" readonly="1"/>
                </xpath>
                <notebook position="inside">
                    <page string="Tremol GO3 Fiscal Device" attrs="{'invisible': [('country_code', '!=', 'KE')]}">
                        <group>
                            <group>
                                <field name="l10n_ke_cu_qrcode" widget="url" readonly="1"/>
                                <field name="l10n_ke_cu_serial_number" readonly="1"/>
                                <field name="l10n_ke_cu_datetime" readonly="1"/>
                            </group>
                        </group>
                    </page>
                </notebook>
            </field>
        </record>

        <record id="l10n_ke_inherit_account_move_tree_view" model="ir.ui.view">
            <field name="name">l10n.ke.inherit.account.move.tree</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_out_invoice_tree" />
            <field name="arch" type="xml">
                <field name="state" position="after">
                    <field name="l10n_ke_cu_invoice_number" optional="hide"/>
                </field>
            </field>
        </record>

        <record id="action_send_invoices_to_device" model="ir.actions.server">
            <field name="name">Send to fiscal device</field>
            <field name="model_id" ref="account.model_account_move"/>
            <field name="binding_model_id" ref="account.model_account_move"/>
            <field name="binding_view_types">list</field>
            <field name="state">code</field>
            <field name="code">
                action = records.l10n_ke_action_cu_post()
            </field>
        </record>

        <record id="l10n_ke_inherit_account_move_search_view" model="ir.ui.view">
            <field name="name">l10n.ke.inherit.account.move.search</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_account_invoice_filter" />
            <field name="arch" type="xml">
                <xpath expr="//field[@name='journal_id']" position="after">
                    <field name="l10n_ke_cu_invoice_number" string="Kenya CU Invoice Number" operator="ilike" />
                </xpath>
                <xpath expr="//filter[@name='cancel']" position="after">
                    <separator/>
                    <filter name="l10n_ke_edi_to_send" string="To Send to TIMS" domain="[('l10n_ke_cu_invoice_number', '=', False), ('state', '=', 'posted'), ('move_type', 'in', ['out_invoice', 'out_refund']), ('country_code', '=', 'KE')]"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>



```

## File: views\product_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_ke_inherit_product_template_form_view" model="ir.ui.view">

        <field name="name">l10n.ke.inherit.product.template.form.inherit</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="account.product_template_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='invoicing']//group[@name='accounting']" position="inside">
                <group name="HS Code" string="HS Code" attrs="{'invisible': [('product_variant_count', '>', 1), ('is_product_variant', '=', False)]}">
                    <field name="l10n_ke_hsn_code"/>
                    <field name="l10n_ke_hsn_name"/>
                </group>
            </xpath>
        </field>
    </record>

    <record id="l10n_ke_inherit_product_product_form_view" model="ir.ui.view">
        <field name="name">l10n.ke.inherit.product.product.form</field>
        <field name="model">product.product</field>
        <field name="inherit_id" ref="product.product_variant_easy_edit_view"/>
        <field name="arch" type="xml">
            <xpath expr="//sheet" position="inside">
                <group>
                    <group name="KRA Item Code" string="KRA Item Code">
                        <field name="l10n_ke_hsn_code"/>
                        <field name="l10n_ke_hsn_name"/>
                    </group>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="l10n_ke_invoice" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@id='qrcode']" position="before">
            <div t-if="o.country_code == 'KE'" id="l10n_ke_control_unit_information" style="page-break-inside:avoid;">
                <b>Kenyan Fiscal Device Info</b>
                <div class="row mt-4 mb-4">
                    <div class="col-auto col-3 mw-100 mb-2">
                        <p>
                            <b>Invoice Number: </b><br></br>
                            <span t-field="o.l10n_ke_cu_invoice_number"/>
                        </p>
                        <p>
                            <b>Serial Number: </b><br></br>
                            <span t-field="o.l10n_ke_cu_serial_number"/>
                        </p>
                        <p>
                            <b>Date and Time of Signing: </b><br></br>
                            <span t-field="o.l10n_ke_cu_datetime"/>
                        </p>
                    </div>
                    <div class="col-auto col-3 mw-100 mb-2">
                        <p t-if="o.l10n_ke_cu_qrcode">
                            <strong class="text-center">TIMS URL</strong><br/><br/>
                            <img style="display:block;"  t-att-src="'/report/barcode/?barcode_type=%s&amp;value=%s&amp;width=%s&amp;height=%s' % ('QR', quote_plus(o.l10n_ke_cu_qrcode), 130, 130)" alt="QR Code"/>
                        </p>
                    </div>
                </div>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">l10n.ke.tremol.inherit.res.config.settings.form</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='account_vendor_bills']" position="after">
                <div attrs="{'invisible':[('country_code', '!=', 'KE')]}">
                    <h2>Kenya TIMS Integration</h2>
                    <div class="row mt16 o_settings_container" id="l10n_ke_cu_details">
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_right_pane">
                                <span class="o_form_label">Tremol Device Settings</span>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                                <div class="text-muted">
                                    The tremol device makes use of a proxy server, which can be running locally on your computer or on an IoT Box.
                                    The proxy server must be on the same network as the fiscal device.
                                </div>
                                <div class="content-group">
                                    <div class="row mt8">
                                        <label for="l10n_ke_cu_proxy_address" class="col-lg-5 o_light_label"/>
                                        <field name="l10n_ke_cu_proxy_address"/>
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
    <record id="res_partner_view_form" model="ir.ui.view">
        <field name="name">l10n.ke.tremol.inherit.res.partner.form</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <group name="accounting_entries" position="after">
                <group string="Kenya Accounting Details" name="l10n_ke_details">
                    <field name="l10n_ke_exemption_number"/>
                </group>
            </group>
        </field>
    </record>
</odoo>

```

