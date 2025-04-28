# Odoo Module: snailmail

Category: Tools

This file contains the source code of the Odoo module.

## File: country_utils.py

```python
SNAILMAIL_COUNTRIES = {
    "AC": "Ascension",
    "AD": "Andorra",
    "AE": "United Arab Emirates",
    "AF": "Afghanistan",
    "AG": "Antigua and Barbuda",
    "AI": "Anguilla",
    "AL": "Albania",
    "AM": "Armenia",
    "AN": "Netherlands Antilles",
    "AO": "Angola",
    "AQ": "Antarctica",
    "AR": "Argentina",
    "AS": "American Samoa",
    "AT": "Austria",
    "AU": "Australia",
    "AW": "Aruba",
    "AX": "Aland Islands",
    "AZ": "Azerbaijan",
    "BA": "Bosnia and Herzegovina",
    "BB": "Barbados",
    "BD": "Bangladesh",
    "BE": "Belgium",
    "BF": "Burkina Faso",
    "BG": "Bulgaria",
    "BH": "Bahrain",
    "BI": "Burundi",
    "BJ": "Benin",
    "BL": "Saint Barth\u00e9lemy",
    "BM": "Bermuda",
    "BN": "Brunei",
    "BO": "Bolivia",
    "BQ": "Bonaire Sint Eustatius and Saba",
    "BR": "Brazil",
    "BS": "Bahamas",
    "BT": "Bhutan",
    "BV": "Bouvet Island",
    "BW": "Botswana",
    "BY": "Belarus",
    "BZ": "Belize",
    "CA": "Canada",
    "CC": "Cocos (Keeling) Islands",
    "CD": "Congo Democratic Republic",
    "CF": "Central African Republic",
    "CG": "Congo Republic",
    "CH": "Switzerland",
    "CI": "C\u00f4te d'Ivoire",
    "CK": "Cook Islands",
    "CL": "Chile",
    "CM": "Cameroon",
    "CN": "China",
    "CO": "Colombia",
    "CR": "Costa Rica",
    "CU": "Cuba",
    "CV": "Cape Verde",
    "CW": "Curacao",
    "CX": "Christmas Island",
    "CY": "Cyprus",
    "CZ": "Czech Republic",
    "DE": "Germany",
    "DG": "Diego Garcia",
    "DJ": "Djibouti",
    "DK": "Denmark",
    "DM": "Dominica",
    "DO": "Dominican Republic",
    "DZ": "Algeria",
    "EC": "Ecuador",
    "EE": "Estonia",
    "EG": "Egypt",
    "EH": "Western Sahara",
    "ER": "Eritrea",
    "ES": "Spain",
    "ET": "Ethiopia",
    "FI": "Finland",
    "FJ": "Fiji",
    "FK": "Falkland Islands",
    "FM": "Micronesia",
    "FO": "Faroe Islands",
    "FR": "France",
    "GA": "Gabon",
    "GB": "Great Britain",
    "GD": "Grenada",
    "GE": "Georgia",
    "GF": "French Guiana",
    "GG": "Guernsey",
    "GH": "Ghana",
    "GI": "Gibraltar",
    "GL": "Greenland",
    "GM": "Gambia",
    "GN": "Guinea Republic",
    "GP": "Guadeloupe",
    "GQ": "Equatorial Guinea",
    "GR": "Greece",
    "GS": "South Georgia and Sandwich",
    "GT": "Guatemala",
    "GU": "Guam",
    "GW": "Guinea-Bissau",
    "GY": "Guyana",
    "HK": "Hong Kong",
    "HM": "Heard Island And Mcdonald Islands",
    "HN": "Honduras",
    "HR": "Croatia",
    "HT": "Haiti",
    "HU": "Hungary",
    "IC": "Canary Islands",
    "ID": "Indonesia",
    "IE": "Ireland",
    "IL": "Israel",
    "IM": "Isle of Man",
    "IN": "India",
    "IO": "British Indian Ocean Territory",
    "IQ": "Iraq",
    "IR": "Iran",
    "IS": "Iceland",
    "IT": "Italy",
    "JE": "Jersey",
    "JM": "Jamaica",
    "JO": "Jordan",
    "JP": "Japan",
    "KE": "Kenya",
    "KG": "Kyrgyzstan",
    "KH": "Cambodia",
    "KI": "Kiribati",
    "KM": "Comoros",
    "KN": "Saint Kitts and Nevis",
    "KP": "Korea Dem. Peo. Rep.",
    "KR": "Korea (South Korea) Republic",
    "KW": "Kuwait",
    "KY": "Cayman Islands",
    "KZ": "Kazakstan",
    "LA": "Laos People's Democratic Republic",
    "LB": "Lebanon",
    "LC": "Saint Lucia",
    "LI": "Liechtenstein",
    "LK": "Sri Lanka",
    "LR": "Liberia",
    "LS": "Lesotho",
    "LT": "Lithuania",
    "LU": "Luxembourg",
    "LV": "Latvia",
    "LY": "Libyan",
    "MA": "Morocco",
    "MC": "Monaco",
    "MD": "Moldova",
    "ME": "Montenegro",
    "MF": "Saint Martin",
    "MG": "Madagascar",
    "MH": "Marshall Islands",
    "MK": "Macedonia",
    "ML": "Mali",
    "MM": "Myanmar",
    "MN": "Mongolia",
    "MO": "Macao",
    "MP": "Mariana Islands",
    "MQ": "Martinique",
    "MR": "Mauritania",
    "MS": "Montserrat",
    "MT": "Malta",
    "MU": "Mauritius",
    "MV": "Maldives",
    "MW": "Malawi",
    "MX": "Mexico",
    "MY": "Malaysia",
    "MZ": "Mozambique",
    "NA": "Namibia",
    "NC": "New Caledonia",
    "NE": "Niger",
    "NF": "Norfolk Island",
    "NG": "Nigeria",
    "NI": "Nicaragua",
    "NL": "Netherlands",
    "NO": "Norway",
    "NP": "Nepal",
    "NR": "Nauru",
    "NU": "Niue",
    "NZ": "New Zealand",
    "OM": "Oman",
    "PA": "Panama",
    "PE": "Peru",
    "PF": "French Polynesia",
    "PG": "Papua New Guinea",
    "PH": "Philippines",
    "PK": "Pakistan",
    "PL": "Poland",
    "PM": "Saint Pierre and Miquelon",
    "PN": "Pitcairn Island",
    "PR": "Puerto Rico",
    "PS": "Palestine",
    "PT": "Portugal",
    "PW": "Palau",
    "PY": "Paraguay",
    "QA": "Qatar",
    "RE": "R\u00e9union",
    "RO": "Romania",
    "RS": "Serbia",
    "RU": "Russian Federation",
    "RW": "Rwanda",
    "SA": "Saudi Arabia",
    "SB": "Solomon Islands",
    "SC": "Seychelles",
    "SD": "Sudan",
    "SE": "Sweden",
    "SG": "Singapore",
    "SH": "Ascension StHelena & Tristan",
    "SI": "Slovenia",
    "SJ": "Svalbard and Jan Mayen",
    "SK": "Slovakia",
    "SL": "Sierra Leone",
    "SM": "San Marino",
    "SN": "Senegal",
    "SO": "Somalia",
    "SR": "Suriname",
    "SS": "South Sudan",
    "ST": "Sao Tome and Principe",
    "SV": "El Salvardor",
    "SX": "Sint Maarten",
    "SY": "Syria",
    "SZ": "Swaziland",
    "TA": "Tristan da Cunha",
    "TC": "Turks and Caicos",
    "TD": "Chad",
    "TF": "French Southern Territories",
    "TG": "Togo",
    "TH": "Thailand",
    "TJ": "Tajikistan",
    "TK": "Tokelau Islands",
    "TL": "Timor-Leste",
    "TM": "Turkmenistan",
    "TN": "Tunisia",
    "TO": "Tonga",
    "TR": "Turkey",
    "TT": "Trinidad and Tobago",
    "TV": "Tuvalu",
    "TW": "China Taiwan",
    "TZ": "Tanzania",
    "UA": "Ukraine",
    "UG": "Uganda",
    "US": "United States of America",
    "UY": "Uruguay",
    "UZ": "Uzbekistan",
    "VA": "Vatican City State",
    "VC": "St. Vincent and Grenadines",
    "VE": "Venezuela",
    "VG": "Virgin Islands british",
    "VI": "Virgin Islands",
    "VN": "Vietnam",
    "VU": "Vanuatu",
    "WF": "Wallis and Futuna Islands",
    "WS": "Western Samoa",
    "XZ": "Kosovo",
    "YE": "Yemen",
    "YT": "Mayotte",
    "ZA": "South Africa",
    "ZM": "Zambia",
    "ZW": "Zimbabwe"
}
```

## File: __init__.py

```python
# -*- coding: utf-8 -*-

from . import models
from . import country_utils
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
{
    'name': "Snail Mail",
    'description': """
Allows users to send documents by post
=====================================================
        """,
    'category': 'Tools',
    'version': '0.1',
    'depends': ['iap', 'mail'],
    'data': [
        'data/snailmail_data.xml',
        'views/report_assets.xml',
        'views/snailmail_views.xml',
        'views/assets.xml',
        'wizard/snailmail_letter_cancel_views.xml',
        'wizard/snailmail_letter_format_error_views.xml',
        'wizard/snailmail_letter_missing_required_fields_views.xml',
        'security/ir.model.access.csv',
    ],
    'qweb': [
        'static/src/xml/thread.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\snailmail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record model="ir.cron" id="snailmail_print">
            <field name="name">Snailmail: process letters queue</field>
            <field name="model_id" ref="model_snailmail_letter"/>
            <field name="state">code</field>
            <field name="code">model._snailmail_cron()</field>
            <field name="interval_number">1</field>
            <field name="interval_type">hours</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: models\ir_actions_report.py

```python
# -*- coding: utf-8 -*-

from odoo import models, fields, api, _


class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    def retrieve_attachment(self, record):
        # Override this method in order to force to re-render the pdf in case of
        # using snailmail
        if self.env.context.get('snailmail_layout'):
            return False
        return super(IrActionsReport, self).retrieve_attachment(record)

    @api.model
    def get_paperformat(self):
        # force the right format (euro/A4) when sending letters, only if we are not using the l10n_DE layout
        res = super(IrActionsReport, self).get_paperformat()
        if self.env.context.get('snailmail_layout') and res != self.env.ref('l10n_de.paperformat_euro_din', False):
            paperformat_id = self.env.ref('base.paperformat_euro')
            return paperformat_id
        else:
            return res

```

## File: models\ir_qweb_fields.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class Contact(models.AbstractModel):
    _inherit = 'ir.qweb.field.contact'

    @api.model
    def value_to_html(self, value, options):
        if self.env.context.get('snailmail_layout'):
           value = value.with_context(snailmail_layout=self.env.context['snailmail_layout'])
        return super(Contact, self).value_to_html(value, options)

    @api.model
    def record_to_html(self, record, field_name, options):
        if self.env.context.get('snailmail_layout'):
           record = record.with_context(snailmail_layout=self.env.context['snailmail_layout'])
        return super(Contact, self).record_to_html(record, field_name, options)

```

## File: models\mail_message.py

```python

from odoo import api, fields, models

class Message(models.Model):
    _inherit = 'mail.message'

    snailmail_error = fields.Boolean("Snailmail message in error", compute="_compute_snailmail_error", search="_search_snailmail_error")
    snailmail_status = fields.Char("Snailmail Status", compute="_compute_snailmail_error")
    letter_ids = fields.One2many(comodel_name='snailmail.letter', inverse_name='message_id')
    message_type = fields.Selection(selection_add=[('snailmail', 'Snailmail')])

    def _get_message_format_fields(self):
        res = super(Message, self)._get_message_format_fields()
        res.append('snailmail_error')
        res.append('snailmail_status')
        return res

    @api.depends('letter_ids', 'letter_ids.state')
    def _compute_snailmail_error(self):
        for message in self:
            if message.message_type == 'snailmail' and message.letter_ids:
                message.snailmail_error = message.letter_ids[0].state == 'error'
                message.snailmail_status = message.letter_ids[0].error_code if message.letter_ids[0].state == 'error' else message.letter_ids[0].state
            else:
                message.snailmail_error = False
                message.snailmail_status = ''

    def _search_snailmail_error(self, operator, operand):
        if operator == '=' and operand:
            return ['&', ('letter_ids.state', '=', 'error'), ('letter_ids.user_id', '=', self.env.user.id)]
        return ['!', '&', ('letter_ids.state', '=', 'error'), ('letter_ids.user_id', '=', self.env.user.id)] 

    def cancel_letter(self):
        self.mapped('letter_ids').cancel()

    def send_letter(self):
        self.mapped('letter_ids')._snailmail_print()

    def message_fetch_failed(self):
        res = super(Message, self).message_fetch_failed()
        failed_letters = self.letter_ids.fetch_failed_letters()
        return res + failed_letters

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class Company(models.Model):
    _inherit = "res.company"

    snailmail_color = fields.Boolean(string='Color', default=True)
    snailmail_cover = fields.Boolean(string='Add a Cover Page', default=False)
    snailmail_duplex = fields.Boolean(string='Both sides', default=False)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-	
# Part of Odoo. See LICENSE file for full copyright and licensing details.	

from odoo import fields, models	


class ResConfigSettings(models.TransientModel):	
    _inherit = 'res.config.settings'	

    snailmail_color = fields.Boolean(string='Print In Color', related='company_id.snailmail_color', readonly=False)
    snailmail_cover = fields.Boolean(string='Add a Cover Page', related='company_id.snailmail_cover', readonly=False)
    snailmail_duplex = fields.Boolean(string='Print Both sides', related='company_id.snailmail_duplex', readonly=False)

```

## File: models\res_partner.py

```python

# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models
from odoo.addons.snailmail.country_utils import SNAILMAIL_COUNTRIES


class ResPartner(models.Model):
    _inherit = "res.partner"

    def write(self, vals):
        for id in self.ids:
            letter_address_vals = {}
            address_fields = ['street', 'street2', 'city', 'zip', 'state_id', 'country_id']
            for field in address_fields:
                if field in vals:
                    letter_address_vals[field] = vals[field]

            if len(letter_address_vals):
                letter_ids = self.env['snailmail.letter'].search([('state', 'not in', ['sent', 'canceled']), ('partner_id', '=', id)])
                letter_ids.write(letter_address_vals)

        return super(ResPartner, self).write(vals)

    def _get_country_name(self):
        # when sending a letter, thus rendering the report with the snailmail_layout,
        # we need to override the country name to its english version following the
        # dictionary imported in country_utils.py
        country_code = self.country_id.code
        if self.env.context.get('snailmail_layout') and country_code in SNAILMAIL_COUNTRIES:
            return SNAILMAIL_COUNTRIES.get(country_code)

        return super(ResPartner, self)._get_country_name()

    @api.model
    def _get_address_format(self):
        # When sending a letter, the fields 'street' and 'street2' should be on a single line to fit in the address area
        if self.env.context.get('snailmail_layout') and self.street2:
            return "%(street)s, %(street2)s\n%(city)s %(state_code)s %(zip)s\n%(country_name)s"

        return super(ResPartner, self)._get_address_format()

```

## File: models\snailmail_letter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re
import base64
import io

from PyPDF2 import PdfFileReader, PdfFileMerger, PdfFileWriter
from reportlab.platypus import Frame, Paragraph, KeepInFrame
from reportlab.lib.units import mm
from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.pdfgen.canvas import Canvas

from odoo import fields, models, api, _
from odoo.addons.iap import jsonrpc
from odoo.exceptions import UserError, AccessError
from odoo.tools.safe_eval import safe_eval

DEFAULT_ENDPOINT = 'https://iap-snailmail.odoo.com'
PRINT_ENDPOINT = '/iap/snailmail/1/print'
DEFAULT_TIMEOUT = 30

ERROR_CODES = [
    'MISSING_REQUIRED_FIELDS',
    'CREDIT_ERROR',
    'TRIAL_ERROR',
    'NO_PRICE_AVAILABLE',
    'FORMAT_ERROR',
    'UNKNOWN_ERROR',
]


class SnailmailLetter(models.Model):
    _name = 'snailmail.letter'
    _description = 'Snailmail Letter'

    user_id = fields.Many2one('res.users', 'Sent by')
    model = fields.Char('Model', required=True)
    res_id = fields.Integer('Document ID', required=True)
    partner_id = fields.Many2one('res.partner', string='Recipient', required=True)
    company_id = fields.Many2one('res.company', string='Company', required=True, readonly=True,
        default=lambda self: self.env.company.id)
    report_template = fields.Many2one('ir.actions.report', 'Optional report to print and attach')

    attachment_id = fields.Many2one('ir.attachment', string='Attachment', ondelete='cascade')
    attachment_datas = fields.Binary('Document', related='attachment_id.datas')
    attachment_fname = fields.Char('Attachment Filename', related='attachment_id.name')
    color = fields.Boolean(string='Color', default=lambda self: self.env.company.snailmail_color)
    cover = fields.Boolean(string='Cover Page', default=lambda self: self.env.company.snailmail_cover)
    duplex = fields.Boolean(string='Both side', default=lambda self: self.env.company.snailmail_duplex)
    state = fields.Selection([
        ('pending', 'In Queue'),
        ('sent', 'Sent'),
        ('error', 'Error'),
        ('canceled', 'Canceled')
        ], 'Status', readonly=True, copy=False, default='pending', required=True,
        help="When a letter is created, the status is 'Pending'.\n"
             "If the letter is correctly sent, the status goes in 'Sent',\n"
             "If not, it will got in state 'Error' and the error message will be displayed in the field 'Error Message'.")
    error_code = fields.Selection([(err_code, err_code) for err_code in ERROR_CODES], string="Error")
    info_msg = fields.Char('Information')
    display_name = fields.Char('Display Name', compute="_compute_display_name")

    reference = fields.Char(string='Related Record', compute='_compute_reference', readonly=True, store=False)

    message_id = fields.Many2one('mail.message', string="Snailmail Status Message")

    street = fields.Char('Street')
    street2 = fields.Char('Street2')
    zip = fields.Char('Zip')
    city = fields.Char('City')
    state_id = fields.Many2one("res.country.state", string='State')
    country_id = fields.Many2one('res.country', string='Country')

    @api.depends('reference', 'partner_id')
    def _compute_display_name(self):
        for letter in self:
            if letter.attachment_id:
                letter.display_name = "%s - %s" % (letter.attachment_id.name, letter.partner_id.name)
            else:
                letter.display_name = letter.partner_id.name

    @api.depends('model', 'res_id')
    def _compute_reference(self):
        for res in self:
            res.reference = "%s,%s" % (res.model, res.res_id)

    @api.model
    def create(self, vals):
        msg_id = self.env[vals['model']].browse(vals['res_id']).message_post(
            body=_("Letter sent by post with Snailmail"),
            message_type='snailmail'
        )
        partner_id = self.env['res.partner'].browse(vals['partner_id'])
        vals.update({
            'message_id': msg_id.id,
            'street': partner_id.street,
            'street2': partner_id.street2,
            'zip': partner_id.zip,
            'city': partner_id.city,
            'state_id': partner_id.state_id.id,
            'country_id': partner_id.country_id.id,
        })
        return super(SnailmailLetter, self).create(vals)

    def _fetch_attachment(self):
        """
        This method will check if we have any existent attachement matching the model
        and res_ids and create them if not found.
        """
        self.ensure_one()
        obj = self.env[self.model].browse(self.res_id)
        if not self.attachment_id:
            report = self.report_template
            if not report:
                report_name = self.env.context.get('report_name')
                report = self.env['ir.actions.report']._get_report_from_name(report_name)
                if not report:
                    return False
                else:
                    self.write({'report_template': report.id})
                # report = self.env.ref('account.account_invoices')
            if report.print_report_name:
                report_name = safe_eval(report.print_report_name, {'object': obj})
            elif report.attachment:
                report_name = safe_eval(report.attachment, {'object': obj})
            else:
                report_name = 'Document'
            filename = "%s.%s" % (report_name, "pdf")
            paperformat = report.get_paperformat()
            if (paperformat.format == 'custom' and paperformat.page_width != 210 and paperformat.page_height != 297) or paperformat.format != 'A4':
                raise UserError(_("Please use an A4 Paper format."))
            if not self.cover:
                raise UserError(_("Snailmails without covers are no longer supported in Odoo 13.\nPlease enable the 'Add a Cover Page' option in your Invoicing settings or upgrade your Odoo."))
            pdf_bin, unused_filetype = report.with_context(snailmail_layout=not self.cover, lang='en_US').render_qweb_pdf(self.res_id)
            pdf_bin = self._overwrite_margins(pdf_bin)
            if self.cover:
                pdf_bin = self._append_cover_page(pdf_bin)
            attachment = self.env['ir.attachment'].create({
                'name': filename,
                'datas': base64.b64encode(pdf_bin),
                'res_model': 'snailmail.letter',
                'res_id': self.id,
                'type': 'binary',  # override default_type from context, possibly meant for another model!
            })
            self.write({'attachment_id': attachment.id})

        return self.attachment_id

    def _count_pages_pdf(self, bin_pdf):
        """ Count the number of pages of the given pdf file.
            :param bin_pdf : binary content of the pdf file
        """
        pages = 0
        for match in re.compile(b"/Count\s+(\d+)").finditer(bin_pdf):
            pages = int(match.group(1))
        return pages

    def _snailmail_create(self, route):
        """
        Create a dictionnary object to send to snailmail server.

        :return: Dict in the form:
        {
            account_token: string,    //IAP Account token of the user
            documents: [{
                pages: int,
                pdf_bin: pdf file
                res_id: int (client-side res_id),
                res_model: char (client-side res_model),
                address: {
                    name: char,
                    street: char,
                    street2: char (OPTIONAL),
                    zip: int,
                    city: char,
                    state: char (state code (OPTIONAL)),
                    country_code: char (country code)
                }
                return_address: {
                    name: char,
                    street: char,
                    street2: char (OPTIONAL),
                    zip: int,
                    city: char,at
                    state: char (state code (OPTIONAL)),
                    country_code: char (country code)
                }
            }],
            options: {
                color: boolean (true if color, false if black-white),
                duplex: boolean (true if duplex, false otherwise),
                currency_name: char
            }
        }
        """
        account_token = self.env['iap.account'].get('snailmail').account_token
        dbuuid = self.env['ir.config_parameter'].sudo().get_param('database.uuid')
        documents = []

        for letter in self:
            recipient_name = letter.partner_id.name or letter.partner_id.parent_id and letter.partner_id.parent_id.name
            if not recipient_name:
                letter.write({
                    'info_msg': _('Invalid recipient name.'),
                    'state': 'error',
                    'error_code': 'MISSING_REQUIRED_FIELDS'
                    })
                continue
            document = {
                # generic informations to send
                'letter_id': letter.id,
                'res_model': letter.model,
                'res_id': letter.res_id,
                'contact_address': letter.partner_id.with_context(snailmail_layout=True, show_address=True).name_get()[0][1],
                'address': {
                    'name': recipient_name,
                    'street': letter.partner_id.street,
                    'street2': letter.partner_id.street2,
                    'zip': letter.partner_id.zip,
                    'state': letter.partner_id.state_id.code if letter.partner_id.state_id else False,
                    'city': letter.partner_id.city,
                    'country_code': letter.partner_id.country_id.code
                },
                'return_address': {
                    'name': letter.company_id.partner_id.name,
                    'street': letter.company_id.partner_id.street,
                    'street2': letter.company_id.partner_id.street2,
                    'zip': letter.company_id.partner_id.zip,
                    'state': letter.company_id.partner_id.state_id.code if letter.company_id.partner_id.state_id else False,
                    'city': letter.company_id.partner_id.city,
                    'country_code': letter.company_id.partner_id.country_id.code,
                }
            }
            # Specific to each case:
            # If we are estimating the price: 1 object = 1 page
            # If we are printing -> attach the pdf
            if route == 'estimate':
                document.update(pages=1)
            else:
                # adding the web logo from the company for future possible customization
                document.update({
                    'company_logo': letter.company_id.logo_web and letter.company_id.logo_web.decode('utf-8') or False,
                })
                attachment = letter._fetch_attachment()
                if attachment:
                    document.update({
                        'pdf_bin': route == 'print' and attachment.datas.decode('utf-8'),
                        'pages': route == 'estimate' and self._count_pages_pdf(base64.b64decode(attachment.datas)),
                    })
                else:
                    letter.write({
                        'info_msg': 'The attachment could not be generated.',
                        'state': 'error',
                        'error_code': 'ATTACHMENT_ERROR'
                        })
                    continue
                if letter.company_id.external_report_layout_id == self.env.ref('l10n_de.external_layout_din5008', False):
                    document.update({
                        'rightaddress': 0,
                    })
            documents.append(document)

        return {
            'account_token': account_token,
            'dbuuid': dbuuid,
            'documents': documents,
            'options': {
                'color': self and self[0].color,
                'cover': self and self[0].cover,
                'duplex': self and self[0].duplex,
                'currency_name': 'EUR',
            },
            # this will not raise the InsufficientCreditError which is the behaviour we want for now
            'batch': True,
        }

    def _get_error_message(self, error):
        if error == 'CREDIT_ERROR':
            link = self.env['iap.account'].get_credits_url(service_name='snailmail')
            return _('You don\'t have enough credits to perform this operation.<br>Please go to your <a href=%s target="new">iap account</a>.') % link
        if error == 'TRIAL_ERROR':
            link = self.env['iap.account'].get_credits_url(service_name='snailmail', trial=True)
            return _('You don\'t have an IAP account registered for this service.<br>Please go to <a href=%s target="new">iap.odoo.com</a> to claim your free credits.') % link
        if error == 'NO_PRICE_AVAILABLE':
            return _('The country of the partner is not covered by Snailmail.')
        if error == 'MISSING_REQUIRED_FIELDS':
            return _('One or more required fields are empty.')
        if error == 'FORMAT_ERROR':
            return _('The attachment of the letter could not be sent. Please check its content and contact the support if the problem persists.')
        else:
            return _('An unknown error happened. Please contact the support.')
        return error

    def _snailmail_print(self, immediate=True):
        valid_address_letters = self.filtered(lambda l: l._is_valid_address(l))
        invalid_address_letters = self - valid_address_letters
        invalid_address_letters._snailmail_print_invalid_address()
        if valid_address_letters and immediate:
            for letter in valid_address_letters:
                letter._snailmail_print_valid_address()
                self.env.cr.commit()

    def _snailmail_print_invalid_address(self):
        for letter in self:
            letter.write({
                'state': 'error',
                'error_code': 'MISSING_REQUIRED_FIELDS',
                'info_msg': _('The address of the recipient is not complete')
            })
        self.send_snailmail_update()

    def _snailmail_print_valid_address(self):
        """
        get response
        {
            'request_code': RESPONSE_OK, # because we receive 200 if good or fail
            'total_cost': total_cost,
            'credit_error': credit_error,
            'request': {
                'documents': documents,
                'options': options
                }
            }
        }
        """
        endpoint = self.env['ir.config_parameter'].sudo().get_param('snailmail.endpoint', DEFAULT_ENDPOINT)
        timeout = int(self.env['ir.config_parameter'].sudo().get_param('snailmail.timeout', DEFAULT_TIMEOUT))
        params = self._snailmail_create('print')
        try:
            response = jsonrpc(endpoint + PRINT_ENDPOINT, params=params, timeout=timeout)
        except AccessError as ae:
            for doc in params['documents']:
                letter = self.browse(doc['letter_id'])
                letter.state = 'error'
                letter.error_code = 'UNKNOWN_ERROR'
            raise ae
        for doc in response['request']['documents']:
            if doc.get('sent') and response['request_code'] == 200:
                note = _('The document was correctly sent by post.<br>The tracking id is %s' % doc['send_id'])
                letter_data = {'info_msg': note, 'state': 'sent', 'error_code': False}
            else:
                error = doc['error'] if response['request_code'] == 200 else response['reason']

                note = _('An error occured when sending the document by post.<br>Error: %s') % self._get_error_message(error)
                letter_data = {
                    'info_msg': note,
                    'state': 'error',
                    'error_code': error if error in ERROR_CODES else 'UNKNOWN_ERROR'
                }

            letter = self.browse(doc['letter_id'])
            letter.write(letter_data)
        self.send_snailmail_update()

    def send_snailmail_update(self):
        notifications = []
        for letter in self:
            notifications.append([
                (self._cr.dbname, 'res.partner', letter.user_id.partner_id.id),
                {'type': 'snailmail_update', 'elements': letter._format_snailmail_failures()}
            ])
        self.env['bus.bus'].sendmany(notifications)

    def snailmail_print(self):
        self.write({'state': 'pending'})
        if len(self) == 1:
            self._snailmail_print()

    def cancel(self):
        self.write({'state': 'canceled', 'error_code': False})
        self.send_snailmail_update()

    @api.model
    def _snailmail_cron(self, autocommit=True):
        letters_send = self.search([
            '|',
            ('state', '=', 'pending'),
            '&',
            ('state', '=', 'error'),
            ('error_code', 'in', ['TRIAL_ERROR', 'CREDIT_ERROR', 'ATTACHMENT_ERROR', 'MISSING_REQUIRED_FIELDS'])
        ])
        for letter in letters_send:
            letter._snailmail_print()
            if letter.error_code == 'CREDIT_ERROR':
                break  # avoid spam
            # Commit after every letter sent to avoid to send it again in case of a rollback
            if autocommit:
                self.env.cr.commit()

    @api.model
    def fetch_failed_letters(self):
        failed_letters = self.search([('state', '=', 'error'), ('user_id.id', '=', self.env.user.id), ('res_id', '!=', 0), ('model', '!=', False)])
        return failed_letters._format_snailmail_failures()

    @api.model
    def _is_valid_address(self, record):
        record.ensure_one()
        required_keys = ['street', 'city', 'zip', 'country_id']
        return all(record[key] for key in required_keys)

    def _format_snailmail_failures(self):
        """
        A shorter message to notify a failure update
        """
        failures_infos = []
        for letter in self:
            info = {
                'message_id': letter.message_id.id,
                'record_name': letter.message_id.record_name,
                'model_name': self.env['ir.model']._get(letter.model).display_name,
                'uuid': letter.message_id.message_id,
                'res_id': letter.res_id,
                'model': letter.model,
                'last_message_date': letter.message_id.date,
                'module_icon': '/snailmail/static/img/snailmail_failure.png',
                'snailmail_status': letter.error_code if letter.state == 'error' else '',
                'snailmail_error': letter.state == 'error',
                'failure_type': 'snailmail',
            }
            failures_infos.append(info)
        return failures_infos

    def _append_cover_page(self, invoice_bin: bytes):
        address_split = self.partner_id.with_context(show_address=True, lang='en_US')._get_name().split('\n')
        address_split[0] = self.partner_id.name or self.partner_id.parent_id and self.partner_id.parent_id.name or address_split[0]
        address = '<br/>'.join(address_split)
        address_x = 118 * mm
        address_y = 60 * mm
        frame_width = 85.5 * mm
        frame_height = 25.5 * mm

        cover_buf = io.BytesIO()
        canvas = Canvas(cover_buf, pagesize=A4)
        styles = getSampleStyleSheet()

        frame = Frame(address_x, A4[1] - address_y - frame_height, frame_width, frame_height)
        story = [Paragraph(address, styles['Normal'])]
        address_inframe = KeepInFrame(0, 0, story)
        frame.addFromList([address_inframe], canvas)
        canvas.save()
        cover_buf.seek(0)

        invoice = PdfFileReader(io.BytesIO(invoice_bin))
        cover_bin = io.BytesIO(cover_buf.getvalue())
        cover_file = PdfFileReader(cover_bin)
        merger = PdfFileMerger()

        merger.append(cover_file, import_bookmarks=False)
        merger.append(invoice, import_bookmarks=False)

        out_buff = io.BytesIO()
        merger.write(out_buff)
        return out_buff.getvalue()

    def _overwrite_margins(self, invoice_bin: bytes):
        """
        Fill the margins with white for validation purposes.
        """
        pdf_buf = io.BytesIO()
        canvas = Canvas(pdf_buf, pagesize=A4)
        canvas.setFillColorRGB(255, 255, 255)
        page_width = A4[0]
        page_height = A4[1]

        # Horizontal Margin
        hmargin_width = page_width
        hmargin_height = 5 * mm

        # Vertical Margin
        vmargin_width = 5 * mm
        vmargin_height = page_height

        # Bottom left square
        sq_width = 15 * mm

        # Draw the horizontal margins
        canvas.rect(0, 0, hmargin_width, hmargin_height, stroke=0, fill=1)
        canvas.rect(0, page_height, hmargin_width, -hmargin_height, stroke=0, fill=1)

        # Draw the vertical margins
        canvas.rect(0, 0, vmargin_width, vmargin_height, stroke=0, fill=1)
        canvas.rect(page_width, 0, -vmargin_width, vmargin_height, stroke=0, fill=1)

        # Draw the bottom left white square
        canvas.rect(0, 0, sq_width, sq_width, stroke=0, fill=1)
        canvas.save()
        pdf_buf.seek(0)

        new_pdf = PdfFileReader(pdf_buf)
        curr_pdf = PdfFileReader(io.BytesIO(invoice_bin))
        out = PdfFileWriter()
        for page in curr_pdf.pages:
            page.mergePage(new_pdf.getPage(0))
            out.addPage(page)
        out_stream = io.BytesIO()
        out.write(out_stream)
        out_bin = out_stream.getvalue()
        out_stream.close()
        return out_bin

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import res_company
from . import res_partner
from . import res_config_settings
from . import snailmail_letter
from . import ir_actions_report
from . import ir_qweb_fields
from . import mail_message

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_snailmail_letter_user,snailmail.letter.user,model_snailmail_letter,base.group_user,1,1,1,0
access_snailmail_letter_system,snailmail.letter.system,model_snailmail_letter,base.group_system,1,1,1,1
```

## File: static\src\js\mail_failure.js

```javascript
odoo.define('snailmail.model.MailFailure', function (require) {
"use strict";

var MailFailure = require('mail.model.MailFailure');
var core = require('web.core');
var _t = core._t;

MailFailure.include({

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    getPreview: function () {
        var preview = this._super.apply(this, arguments);
        if (this._failureType === 'snailmail') {
            _.extend(preview, {
                body: _t("An error occured when sending a letter with Snailmail"),
                id: 'snailmail_failure',
            });
        }
        return preview;
    },
});

});

```

## File: static\src\js\message.js

```javascript
odoo.define('snailmail.model.Message', function (require) {
"use strict";

var Message = require('mail.model.Message');

Message.include({

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Cancels the 'snailmail.letter' which has a message_id corresponding to 
     * the ID of this message, then update the status of the message
     * 
     * @returns {Deferred}
     */
    cancelLetter: function () {
        var self = this;
        var mailBus = this.call('mail_service', 'getMailBus');
        return this._rpc({
            model: 'mail.message',
            method: 'cancel_letter',
            args: [[this.getID()]],
        }).then(function () {
            self.setSnailmailStatus('canceled');
            self.setSnailmailError(false);
            mailBus.trigger('update_message', self);
        });
    },
    /**
     * @return {string}
     */
    getSnailmailStatus: function () {
        return this._snailmailStatus.toLowerCase();
    },
    /**
     * Is the sent snailmail letter in error
     * 
     * @return {Boolean}
     */
    getSnailmailError: function () {
        return this._snailmailError;
    },
    /**
     * Retries to send the 'snailmail.letter' corresponding to this message
     * 
     * @returns {Deferred}
     */
    resendLetter: function () {
        return this._rpc({
            model: 'mail.message',
            method: 'send_letter',
            args: [[this.getID()]],
        });
    },
    /**
     * @param {string} snailmailStatus 
     */
    setSnailmailStatus: function (snailmailStatus) {
        this._snailmailStatus = snailmailStatus;
    },
    /**
     * @param {Boolean} snailmailError
     */
    setSnailmailError: function (snailmailError) {
        this._snailmailError = snailmailError;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @override
     */
    _setInitialData: function (data){
        this._super.apply(this, arguments);
        this._snailmailStatus = data.snailmail_status;
        this._snailmailError = data.snailmail_error;
    },
});

});

```

## File: static\src\js\snailmail_external_layout.js

```javascript
// Change address font-size if needed
document.addEventListener('DOMContentLoaded', function (evt) {
    var recipientAddress = document.querySelector('.address.row div[name="address"]').getElementsByTagName('address')[0];
    var height = parseFloat(window.getComputedStyle(recipientAddress, null).getPropertyValue('height'));
    var fontSize = parseFloat(window.getComputedStyle(recipientAddress, null).getPropertyValue('font-size'));
    recipientAddress.style.fontSize = (85/height) * fontSize + 'px';
});

```

## File: static\src\js\snailmail_notification_manager.js

```javascript
odoo.define('snailmail.NotificationManager', function (require) {
"use strict";

var MailManager = require('mail.Manager');
var MailFailure = require('mail.model.MailFailure');

MailManager.include({

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     * @param {Object} data structure depending on the type
     * @param {integer} data.id
     */
    _handlePartnerNotification: function (data) {
        if (data.type === 'snailmail_update') {
            this._handleSnailmailUpdateNotification(data);
        } else {
            this._super.apply(this, arguments);
        }
    },
    /**
     * Updates message in thread and systray when there's an update in a snailmail letter
     *
     * @private
     * @param {Object} datas
     * @param {Object[]} datas.elements list of snailmail failure data
     * @param {string} datas.elements[].message_id ID of related message that
     *   has a snailmail failure.
     * @param {boolean} datas.elements[].snailmail_error letter is in error
     * @param {string} datas.elements[].snailmail_status status of the letter
     */
    _handleSnailmailUpdateNotification: function (datas) {
        var self = this;
        _.each(datas.elements, function (data) {
            var isNewFailure = data.snailmail_error;
            var matchedFailure = _.find(self._mailFailures, function (failure) {
                return failure.getMessageID() === data.message_id;
            });

            if (matchedFailure) {
                var index = _.findIndex(self._mailFailures, matchedFailure);
                if (isNewFailure) {
                    self._mailFailures[index] = new MailFailure(self, data);
                } else {
                    self._mailFailures.splice(index, 1);
                }
            } else if (isNewFailure) {
                self._mailFailures.push(new MailFailure(self, data));
            }
            var message = _.find(self._messages, function (msg) {
                return msg.getID() === data.message_id;
            });
            if (message) {
                message.setSnailmailStatus(data.snailmail_status);
                message.setSnailmailError(data.snailmail_error);
                self._mailBus.trigger('update_message', message);
            }
        });
        this._mailBus.trigger('update_needaction', this.needactionCounter);
    },
});

});

```

## File: static\src\js\systray_messaging_menu.js

```javascript
odoo.define('snailmail.systray.MessagingMenu', function (require) {
"use strict";

var MessagingMenu = require('mail.systray.MessagingMenu');

MessagingMenu.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    
    /**
     * Called when clicking on a preview related to a snailmail failure
     *
     * @private
     * @param {$.Element} $target DOM of preview element clicked
     */
    _clickSnailmailFailurePreview: function ($target) {
        var documentID = $target.data('document-id');
        var documentModel = $target.data('document-model');
        if (documentModel && documentID) {
            this._openDocument(documentModel, documentID);
        } else if (documentModel !== 'mail.channel') {
            // preview of mail failures grouped to different document of same model
            this.do_action({
                name: "Snailmail failures",
                type: 'ir.actions.act_window',
                view_mode: 'kanban,list,form',
                views: [[false, 'kanban'], [false, 'list'], [false, 'form']],
                target: 'current',
                res_model: documentModel,
                domain: [['message_ids.snailmail_error', '=', true]],
            });
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @override
     */
   _onClickPreview: function (ev) {
       var $target = $(ev.currentTarget);
       var previewID = $target.data('preview-id');
       
        if (previewID === 'snailmail_failure') {
            this._clickSnailmailFailurePreview($target);
        } else {
            this._super.apply(this, arguments);
        }
   },
    /**
     * @private
     * @override
     */
    _onClickPreviewMarkAsRead: function (ev) {
        ev.stopPropagation();
        var $preview = $(ev.currentTarget).closest('.o_mail_preview');
        var previewID = $preview.data('preview-id');
        if (previewID === 'snailmail_failure') {
            var documentModel = $preview.data('document-model');
            var unreadCounter = $preview.data('unread-counter');
            this.do_action('snailmail.snailmail_letter_cancel_action', {
                additional_context: {
                    default_model: documentModel,
                    unread_counter: unreadCounter
                }
            });
        } else {
            this._super.apply(this, arguments);
        }
    },
});

});

```

## File: static\src\js\thread_widget.js

```javascript
odoo.define('snailmail.widget.Thread', function (require) {
"use strict";

var ThreadWidget = require('mail.widget.Thread');
var Dialog = require('web.Dialog');

var core = require('web.core');
var QWeb = core.qweb;
var _t = core._t;

ThreadWidget.include({
    dependencies: ['bus_service'],
    events: _.extend({}, ThreadWidget.prototype.events, {
        'click .o_thread_message_snailmail_missing_required_fields': '_onClickMissingRequiredFields',
        'click .o_thread_message_snailmail_credit_error': '_onClickCreditError',
        'click .o_thread_message_snailmail_trial_error': '_onClickTrialError',
        'click .o_thread_message_snailmail_no_price_available': '_onClickNoPriceAvailable',
        'click .o_thread_message_snailmail_format_error': '_onClickFormatError',
        'click .o_thread_message_snailmail_unknown_error': '_onClickUnknownError',
    }),
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this._enabledOptions = _.defaults(this._enabledOptions, {
            displaySnailmailIcons: true,
        });
        this._disabledOptions = _.defaults(this._disabledOptions, {
            displaySnailmailIcons: false,
        });
    },
    /**
     * @override
     */
    render: function (thread, options) {
        this._super.apply(this, arguments);
        var messages = _.clone(thread.getMessages({ domain: options.domain || [] }));
        this._messages  = messages;
        this._renderMessageSnailmailPopover();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {integer} messageID 
     * @param {string} content 
     */
    _openCreditErrorDialog: function (messageID, content) {
        var message = _.find(this._messages, function (message) {
            return message.getID() === messageID;
        });
        new Dialog(this, {
            size: 'medium',
            title: _t("Failed letter"),
            $content: $('<div>').html(content),
            buttons: [
                {text: _t("Re-send letter"), classes: 'btn-primary', close: true, click: _.bind(message.resendLetter, message)},
                {text: _t("Cancel letter"), close: true, click: _.bind(message.cancelLetter, message)},
                {text: _t("Close"), close: true},
            ],
        }).open();
    },
    /**
     * @private
     * @param {integer} messageID 
     * @param {string} content 
     */
    _openGenericErrorDialog: function (messageID, content) {
        var message = _.find(this._messages, function (message) {
            return message.getID() === messageID;
        });
        new Dialog(this, {
            size: 'medium',
            title: _t("Failed letter"),
            $content: $('<div>').html(content),
            buttons: [
                {text: _t("Cancel letter"), classes: 'btn-primary', close: true, click: _.bind(message.cancelLetter, message)},
                {text: _t("Close"), close: true},
            ],
        }).open();
    },
    /**
     * Render the popover when mouse-hovering on the mail icon of a message
     * in the thread. There is at most one such popover at any given time.
     *
     * @private
     * @param {mail.model.AbstractMessage[]} messages list of messages in the
     *   rendered thread, for which popover on mouseover interaction is
     *   permitted.
     */
    _renderMessageSnailmailPopover: function () {
        var self = this;
        if (this._messageSnailmailPopover) {
            this._messageSnailmailPopover.popover('hide');
        }
        if (!this.$('.o_thread_snailmail_tooltip').length) {
            return;
        }
        this._messageSnailmailPopover = this.$('.o_thread_snailmail_tooltip').popover({
            html: true,
            boundary: 'viewport',
            placement: 'auto',
            trigger: 'hover',
            offset: '0, 1',
            content: function () {
                var messageID = $(this).data('message-id');
                var message = _.find(self._messages, function (message) {
                    return message.getID() === messageID;
                });
                return QWeb.render('snailmail.widget.Thread.Message.SnailmailTooltip', {
                    status: message.getSnailmailStatus()
                });
            },
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {MouseEvent} event
     */
    _onClickCreditError: function (ev) {
        var self = this;
        var messageID = $(ev.currentTarget).data('message-id');
        this._rpc({
            model: 'iap.account',
            method: 'get_credits_url',
            args: ['snailmail'],
        }).then(function (link) {
            var content = _.str.sprintf(_t(
                '<p>The letter could not be sent due to insufficient credits on your IAP account.</p>' +
                '<div class= "text-right">' +
                '<a class="btn btn-link buy_credits" href=%s target="_blank">' +
                '<i class= "fa fa-arrow-right"/> Buy credits' +
                '</a>' +
                '</div>'
            ), link);
            self._openCreditErrorDialog(messageID, content);
        });
    },
    /**
     * @private
     * @param {MouseEvent} event
     */
    _onClickFormatError: function (ev) {
        var messageID = $(ev.currentTarget).data('message-id');
        this.do_action('snailmail.snailmail_letter_format_error_action', {
            additional_context: {
                message_id: messageID,
            }
        });
    },
    /**
     * @private
     * @param {MouseEvent} event
     */
    _onClickMissingRequiredFields: function (ev) {
        var self = this;

        var messageID = $(ev.currentTarget).data('message-id');
        var domain = [['message_id', '=', messageID]];
        this._rpc({
            model: 'snailmail.letter',
            method: 'search',
            args: [domain],
        }).then(function (letterIds) {
            self.do_action('snailmail.snailmail_letter_missing_required_fields_action', {
                additional_context: {
                    letter_id: letterIds[0]
                },
                on_close: function () {
                    self.trigger_up('reload');
                }
            });
        });
    },
    /**
     * @private
     * @param {MouseEvent} event
     */
    _onClickNoPriceAvailable: function (ev) {
        var messageID = $(ev.currentTarget).data('message-id');
        var content = _t('<p>The country to which you want to send the letter is not supported by our service.</p>');
        this._openGenericErrorDialog(messageID, content);
    },
    /**
     * @private
     * @param {MouseEvent} event
     */
    _onClickTrialError: function (ev) {
        var self = this;
        var messageID = $(ev.currentTarget).data('message-id');
        this._rpc({
            model: 'iap.account',
            method: 'get_credits_url',
            args: ['snailmail', '', 0, true],
        }).then(function (link) {
            var content = _.str.sprintf(_t(
                '<p>You need credits on your IAP account to send a letter.</p>' +
                '<div class= "text-right">' +
                '<a class="btn btn-link buy_credits" href=%s>' +
                '<i class= "fa fa-arrow-right"/> Buy credits' +
                '</a>' +
                '</div>'
            ), link);
            self._openCreditErrorDialog(messageID, content);
        });
    },
    /**
     * @private
     * @param {MouseEvent} event
     */
    _onClickUnknownError: function (ev) {
        var messageID = $(ev.currentTarget).data('message-id');
        var content = _t('<p>An unknown error occured. Please contact our <a href="https://www.odoo.com/help" target="new">support</a> for further assistance.</p>');
        this._openGenericErrorDialog(messageID, content);
    },
});

});

```

## File: static\src\xml\thread.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template id="template" xml:space="preserve">

    <t t-extend="mail.widget.Thread.Message">
        <t t-jquery=".o_thread_tooltip_container" t-operation="after">
            <span t-if="message.getType() === 'snailmail' and options.displaySnailmailIcons" class="o_thread_snailmail_tooltip_container">
                <t t-set="thread_icon_class" t-value="'o_thread_snailmail_tooltip o_thread_message_snailmail o_thread_message_snailmail_' + message.getSnailmailStatus()" />
                <t t-if="message.getSnailmailError()">
                    <t t-set="thread_icon_class" t-value="thread_icon_class + ' o_thread_message_snailmail_error'"/>
                </t>
                <i t-att-class="thread_icon_class + ' fa fa-paper-plane'" t-att-data-message-id="message.getID()"/>
            </span>
        </t>
    </t>

    <t t-name="snailmail.widget.Thread.Message.SnailmailTooltip">
        <span class="d-inline-block text-center o_thread_tooltip_snailmail">
            <t t-if="status === 'sent'">
                <i class='fa fa-check o_thread_tooltip_snailmail_icon' title="Sent" role="img" aria-label="Sent"/>
                Sent
            </t>
            <t t-elif="status === 'canceled'">
                <i class='fa fa-trash-o o_thread_tooltip_snailmail_icon' title="Canceled" role="img" aria-label="Canceled"/>
                Canceled
            </t>
            <t t-elif="status === 'pending'">
                <i class='fa fa-clock-o o_thread_tooltip_snailmail_icon' title="Awaiting Dispatch" role="img" aria-label="Awaiting Dispatch"/>
                Awaiting Dispatch
            </t>
            <t t-else="">
                <i class='fa fa-exclamation text-danger o_thread_tooltip_snailmail_icon' title="Error" role="img" aria-label="Error"/>
                Error
            </t>
        </span>
    </t>

</template>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="iap assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/snailmail/static/src/js/mail_failure.js"></script>
            <script type="text/javascript" src="/snailmail/static/src/js/message.js"></script>
            <script type="text/javascript" src="/snailmail/static/src/js/snailmail_notification_manager.js"></script>
            <script type="text/javascript" src="/snailmail/static/src/js/systray_messaging_menu.js"></script>
            <script type="text/javascript" src="/snailmail/static/src/js/thread_widget.js"></script>
            <link rel="stylesheet" type="text/scss" href="/snailmail/static/src/scss/thread.scss"/>
        </xpath>
    </template>

    <template id="qunit_suite" name="snailmail_tests" inherit_id="web.qunit_suite">
        <xpath expr="//t[@t-set='head']" position="inside">
            <script type="text/javascript" src="/snailmail/static/tests/chatter_snailmail_tests.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\report_assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_assets_snailmail">
        <t t-call="web._assets_helpers"/>
        <link rel="stylesheet" type="text/scss" href="/snailmail/static/src/scss/snailmail_external_layout_asset.scss"/>
        <script type="text/javascript" src="/snailmail/static/src/js/snailmail_external_layout.js"/>
    </template>
    <template id="report_layout" inherit_id="web.report_layout">
        <xpath expr="//head" position="inside">
            <t t-if="env and env.context.get('snailmail_layout')" t-call-assets="snailmail.report_assets_snailmail"/>
        </xpath>
    </template>
    <template id="minimal_layout" inherit_id="web.minimal_layout">
        <xpath expr="//head" position="inside">
            <t t-if="env and env.context.get('snailmail_layout')" t-call-assets="snailmail.report_assets_snailmail"/>
        </xpath>
    </template>
</odoo>

```

## File: views\snailmail_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="snailmail_letter_list">
        <field name="name">snailmail.letter.tree</field>
        <field name="model">snailmail.letter</field>
        <field name="arch" type="xml">
            <tree decoration-danger="state=='error'" decoration-muted="state=='sent'" string="Letters">
                <field name="attachment_id" string="Document"/>
                <field name="partner_id"/>
                <field name="user_id"/>
                <field name="state" invisible="1"/>
                <field name="info_msg" widget="html"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record model="ir.ui.view" id="snailmail_letter_form">
        <field name="name">snailmail.letter.form</field>
        <field name="model">snailmail.letter</field>
        <field name="arch" type="xml">
            <form>
                <header>
                    <button name="snailmail_print" string="Send Now" type="object" states="pending,error" class="oe_highlight"/>
                    <button name="cancel" string="Cancel" type="object" states="pending,error"/>
                    <field name="state" widget="statusbar" statusbar_visible="pending,sent,cancel"/>
                </header>
                <sheet>
                    <div class="oe_title">
                        <h1><field name="display_name"/></h1>
                    </div>
                    <group>
                        <field name="reference" widget="reference"/>
                        <field name="attachment_datas" filename="attachment_fname"/>
                        <field name="attachment_fname" invisible="1"/>
                        <field name="partner_id"/>
                        <field name="user_id"/>
                        <field name="info_msg" widget="html"/>
                    </group>
                    <group groups="base.group_no_one">
                        <field name="model"/>
                        <field name="res_id"/>
                        <field name="color"/>
                        <field name="duplex"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_mail_letters">
        <field name="name">Snailmail Letters</field>
        <field name="res_model">snailmail.letter</field>
        <field name="view_mode">form,tree</field>
        <field name="domain">[('state', '!=', 'draft')]</field>
        <field name="view_id" ref="snailmail_letter_list" />
    </record>

    <menuitem id="menu_snailmail_letters" parent="base.menu_email" action="action_mail_letters"
              sequence="20"/>
</odoo>

```

## File: wizard\snailmail_letter_cancel.py

```python

from odoo import _, api, fields, models

class SnailmailLetterCancel(models.TransientModel):
    _name = 'snailmail.letter.cancel'
    _description = 'Dismiss notification for resend by model'

    model = fields.Char(string='Model')
    help_message = fields.Char(string='Help message', compute='_compute_help_message')

    @api.depends('model')
    def _compute_help_message(self):
        for wizard in self:
            wizard.help_message = _("Are you sure you want to discard %s snailmail delivery failures. You won't be able to re-send these letters later!") % (wizard._context.get('unread_counter'))

    def cancel_resend_action(self):
        author_id = self.env.user.id
        for wizard in self:
            letters = self.env['snailmail.letter'].search([
                ('state', 'not in', ['sent', 'canceled', 'pending']),
                ('user_id', '=', author_id),
                ('model', '=', wizard.model)
            ])
            for letter in letters:
                letter.cancel()
        return {'type': 'ir.actions.act_window_close'}

```

## File: wizard\snailmail_letter_cancel_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="snailmail_letter_cancel" model="ir.ui.view">
        <field name="name">snailmail.letter.cancel.form</field>
        <field name="model">snailmail.letter.cancel</field>
        <field name="groups_id" eval="[(4,ref('base.group_user'))]"/>
        <field name="arch" type="xml">
            <form string="Cancel notification in failure">
                <field name="model" invisible='1'/>
                <field name="help_message"/>
                <p>If you want to re-send them, click Cancel now, then click on the notification and review them one by one by clicking on the red paper-plane next to each message.</p>
                <footer>  
                    <button string="Discard delivery failures" name="cancel_resend_action" type="object" class="btn-primary" />
                    <button string="Cancel" class="btn-secondary" special="cancel" />
                </footer>
            </form>
        </field>
    </record>

    <record id="snailmail_letter_cancel_action" model="ir.actions.act_window">
        <field name="name">Discard snailmail delivery failures</field>
        <field name="res_model">snailmail.letter.cancel</field>
        <field name="type">ir.actions.act_window</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\snailmail_letter_format_error.py

```python

from odoo import api, fields, models

class SnailmailLetterFormatError(models.TransientModel):
    _name = 'snailmail.letter.format.error'
    _description = 'Format Error Sending a Snailmail Letter'

    message_id = fields.Many2one('mail.message')
    snailmail_cover = fields.Boolean(string='Add a Cover Page')

    @api.model
    def default_get(self, fields):
        res = super(SnailmailLetterFormatError, self).default_get(fields)
        snailmail_cover = self.env.company.snailmail_cover
        res.update({
            'message_id': self.env.context.get('message_id'),
            'snailmail_cover': snailmail_cover,
        })
        return res

    def update_resend_action(self):
        self.env.company.write({'snailmail_cover': self.snailmail_cover})
        letters_to_resend = self.env['snailmail.letter'].search([
            ('error_code', '=', 'FORMAT_ERROR'),
        ])
        for letter in letters_to_resend:
            letter.attachment_id.unlink()
            letter.write({'cover': self.snailmail_cover})
            letter.snailmail_print()

    def cancel_letter_action(self):
        self.message_id.cancel_letter()

```

## File: wizard\snailmail_letter_format_error_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="snailmail_letter_format_error" model="ir.ui.view">
        <field name="name">snailmail.letter.format.error.form</field>
        <field name="model">snailmail.letter.format.error</field>
        <field name="groups_id" eval="[(4,ref('base.group_user'))]"/>
        <field name="arch" type="xml">
            <form string="Cancel notification in failure">
                <p>Our service cannot read your letter due to its format.<br/>
                Please modify the format of the template or update your settings
                to automatically add a blank cover page to all letters.</p>
                <field name="snailmail_cover"/>
                <label string="Add a Cover Page" class="o_light_label" for="snailmail_cover"/>
                <footer>  
                    <button string="Update Config and Re-send" name="update_resend_action" type="object" class="btn-primary" />
                    <button string="Cancel Letter" name="cancel_letter_action" type="object" class="btn-primary" />
                    <button string="Close" class="btn-secondary" special="cancel" />
                </footer>
            </form>
        </field>
    </record>

    <record id="snailmail_letter_format_error_action" model="ir.actions.act_window">
        <field name="name">Format Error</field>
        <field name="res_model">snailmail.letter.format.error</field>
        <field name="type">ir.actions.act_window</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\snailmail_letter_missing_required_fields.py

```python

from odoo import _, api, fields, models

class SnailmailLetterMissingRequiredFields(models.TransientModel):
    _name = 'snailmail.letter.missing.required.fields'
    _description = 'Update address of partner'

    partner_id = fields.Many2one('res.partner')
    letter_id = fields.Many2one('snailmail.letter')

    street = fields.Char('Street')
    street2 = fields.Char('Street2')
    zip = fields.Char('Zip')
    city = fields.Char('City')
    state_id = fields.Many2one("res.country.state", string='State')
    country_id = fields.Many2one('res.country', string='Country')

    @api.model
    def default_get(self, fields):
        rec = super(SnailmailLetterMissingRequiredFields, self).default_get(fields)
        letter_id = self.env['snailmail.letter'].browse(self.env.context.get('letter_id'))
        rec.update({
            'partner_id': letter_id.partner_id.id,
            'letter_id': letter_id.id,
            'street': letter_id.street,
            'street2': letter_id.street2,
            'zip': letter_id.zip,
            'city': letter_id.city,
            'state_id': letter_id.state_id.id,
            'country_id': letter_id.country_id.id,
        })
        return rec

    def update_address_cancel(self):
        self.letter_id.cancel()

    def update_address_save(self):
        address_data = {
            'street': self.street,
            'street2': self.street2,
            'zip': self.zip,
            'city': self.city,
            'state_id': self.state_id.id,
            'country_id': self.country_id.id,
        }
        self.partner_id.write(address_data)
        letters_to_resend = self.env['snailmail.letter'].search([
            ('partner_id', '=', self.partner_id.id),
            ('error_code', '=', 'MISSING_REQUIRED_FIELDS'),
        ])
        letters_to_resend.write(address_data)
        letters_to_resend.snailmail_print()

```

## File: wizard\snailmail_letter_missing_required_fields_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="snailmail_letter_missing_required_fields" model="ir.ui.view">
        <field name="name">snailmail.letter.missing.required.fields.form</field>
        <field name="model">snailmail.letter.missing.required.fields</field>
        <field name="arch" type="xml">
            <form>
                <p>The customer address is not complete. Update the address here and re-send the letter.</p>
                <group>
                    <label for="partner_id" string="Address"/>
                    <div class="o_address_format">
                        <field name="partner_id" readonly="1" options="{'no_open': True}"/>
                        <field name="street" placeholder="Street..." class="o_address_street"/>
                        <field name="street2" placeholder="Street 2..." class="o_address_street"/>
                        <field name="city" placeholder="City" class="o_address_city"/>
                        <field name="state_id" class="o_address_state" placeholder="State" options='{"no_open": True}'/>
                        <field name="zip" placeholder="ZIP" class="o_address_zip"/>
                        <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'/>
                    </div>
                </group>
                <footer>
                    <button string="Update address and re-send" type="object" name="update_address_save" class="btn-primary"/>
                    <button string="Cancel letter" type="object" name="update_address_cancel" class="btn-secondary"/>
                    <button string="Close" special='cancel' class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="snailmail_letter_missing_required_fields_action" model="ir.actions.act_window">
        <field name="name">Failed letter</field>
        <field name="res_model">snailmail.letter.missing.required.fields</field>
        <field name="type">ir.actions.act_window</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
from . import snailmail_letter_cancel
from . import snailmail_letter_format_error
from . import snailmail_letter_missing_required_fields

```

