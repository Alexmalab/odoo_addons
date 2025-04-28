# Odoo Module: snailmail

Category: Hidden/Tools

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
    'category': 'Hidden/Tools',
    'version': '0.3',
    'depends': [
        'iap_mail',
        'mail'
    ],
    'data': [
        'data/snailmail_data.xml',
        'views/report_assets.xml',
        'views/snailmail_views.xml',
        'wizard/snailmail_confirm_views.xml',
        'wizard/snailmail_letter_cancel_views.xml',
        'wizard/snailmail_letter_format_error_views.xml',
        'wizard/snailmail_letter_missing_required_fields_views.xml',
        'security/ir.model.access.csv',
    ],
    'auto_install': True,
    'assets': {
        'mail.assets_discuss_public': [
            'snailmail/static/src/components/*/*',
            'snailmail/static/src/models/*/*.js',
        ],
        'snailmail.report_assets_snailmail': [
            ('include', 'web._assets_helpers'),
            'web/static/lib/bootstrap/scss/_variables.scss',
            'snailmail/static/src/scss/snailmail_external_layout_asset.scss',
            'snailmail/static/src/js/snailmail_external_layout.js',
        ],
        'web.assets_backend': [
            'snailmail/static/src/components/*/*.js',
            'snailmail/static/src/components/*/*.scss',
            'snailmail/static/src/models/*/*.js',
        ],
        'web.tests_assets': [
            'snailmail/static/tests/**/*',
        ],
        'web.qunit_suite_tests': [
            'snailmail/static/src/components/*/tests/*.js',
        ],
        'web.assets_qweb': [
            'snailmail/static/src/components/*/*.xml',
        ],
    },
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
    letter_ids = fields.One2many(comodel_name='snailmail.letter', inverse_name='message_id')
    message_type = fields.Selection(selection_add=[
        ('snailmail', 'Snailmail')
    ], ondelete={'snailmail': lambda recs: recs.write({'message_type': 'email'})})

    @api.depends('letter_ids', 'letter_ids.state')
    def _compute_snailmail_error(self):
        for message in self:
            if message.message_type == 'snailmail' and message.letter_ids:
                message.snailmail_error = message.letter_ids[0].state == 'error'
            else:
                message.snailmail_error = False

    def _search_snailmail_error(self, operator, operand):
        if operator == '=' and operand:
            return ['&', ('letter_ids.state', '=', 'error'), ('letter_ids.user_id', '=', self.env.user.id)]
        return ['!', '&', ('letter_ids.state', '=', 'error'), ('letter_ids.user_id', '=', self.env.user.id)]

    def cancel_letter(self):
        self.mapped('letter_ids').cancel()

    def send_letter(self):
        self.mapped('letter_ids')._snailmail_print()

```

## File: models\mail_notification.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class Notification(models.Model):
    _inherit = 'mail.notification'

    notification_type = fields.Selection(selection_add=[('snail', 'Snailmail')], ondelete={'snail': 'cascade'})
    letter_id = fields.Many2one('snailmail.letter', string="Snailmail Letter", index=True, ondelete='cascade')
    failure_type = fields.Selection(selection_add=[
        ('sn_credit', "Snailmail Credit Error"),
        ('sn_trial', "Snailmail Trial Error"),
        ('sn_price', "Snailmail No Price Available"),
        ('sn_fields', "Snailmail Missing Required Fields"),
        ('sn_format', "Snailmail Format Error"),
        ('sn_error', "Snailmail Unknown Error"),
    ])

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
        letter_address_vals = {}
        address_fields = ['street', 'street2', 'city', 'zip', 'state_id', 'country_id']
        for field in address_fields:
            if field in vals:
                letter_address_vals[field] = vals[field]

        if letter_address_vals:
            letters = self.env['snailmail.letter'].search([
                ('state', 'not in', ['sent', 'canceled']),
                ('partner_id', 'in', self.ids),
            ])
            letters.write(letter_address_vals)

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

from PyPDF2 import PdfFileReader, PdfFileWriter
from reportlab.platypus import Frame, Paragraph, KeepInFrame
from reportlab.lib.units import mm
from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.pdfgen.canvas import Canvas

from odoo import fields, models, api, _
from odoo.addons.iap.tools import iap_tools
from odoo.exceptions import AccessError, UserError
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
    notification_ids = fields.One2many('mail.notification', 'letter_id', "Notifications")

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

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            msg_id = self.env[vals['model']].browse(vals['res_id']).message_post(
                body=_("Letter sent by post with Snailmail"),
                message_type='snailmail',
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
        letters = super().create(vals_list)

        notification_vals = []
        for letter in letters:
            notification_vals.append({
                'mail_message_id': letter.message_id.id,
                'res_partner_id': letter.partner_id.id,
                'notification_type': 'snail',
                'letter_id': letter.id,
                'is_read': True,  # discard Inbox notification
                'notification_status': 'ready',
            })

        self.env['mail.notification'].sudo().create(notification_vals)

        letters.attachment_id.check('read')
        return letters

    def write(self, vals):
        res = super().write(vals)
        if 'attachment_id' in vals:
            self.attachment_id.check('read')
        return res

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
            pdf_bin, unused_filetype = report.with_context(snailmail_layout=not self.cover, lang='en_US')._render_qweb_pdf(self.res_id)
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
        for match in re.compile(rb"/Count\s+(\d+)").finditer(bin_pdf):
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

        batch = len(self) > 1
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
                        'error_code': 'UNKNOWN_ERROR'
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
            return _('You don\'t have enough credits to perform this operation.<br>Please go to your <a href=%s target="new">iap account</a>.', link)
        if error == 'TRIAL_ERROR':
            link = self.env['iap.account'].get_credits_url(service_name='snailmail', trial=True)
            return _('You don\'t have an IAP account registered for this service.<br>Please go to <a href=%s target="new">iap.odoo.com</a> to claim your free credits.', link)
        if error == 'NO_PRICE_AVAILABLE':
            return _('The country of the partner is not covered by Snailmail.')
        if error == 'MISSING_REQUIRED_FIELDS':
            return _('One or more required fields are empty.')
        if error == 'FORMAT_ERROR':
            return _('The attachment of the letter could not be sent. Please check its content and contact the support if the problem persists.')
        else:
            return _('An unknown error happened. Please contact the support.')
        return error

    def _get_failure_type(self, error):
        if error == 'CREDIT_ERROR':
            return 'sn_credit'
        if error == 'TRIAL_ERROR':
            return 'sn_trial'
        if error == 'NO_PRICE_AVAILABLE':
            return 'sn_price'
        if error == 'MISSING_REQUIRED_FIELDS':
            return 'sn_fields'
        if error == 'FORMAT_ERROR':
            return 'sn_format'
        else:
            return 'sn_error'

    def _snailmail_print(self, immediate=True):
        valid_address_letters = self.filtered(lambda l: l._is_valid_address(l))
        invalid_address_letters = self - valid_address_letters
        invalid_address_letters._snailmail_print_invalid_address()
        if valid_address_letters and immediate:
            for letter in valid_address_letters:
                letter._snailmail_print_valid_address()
                self.env.cr.commit()

    def _snailmail_print_invalid_address(self):
        error = 'MISSING_REQUIRED_FIELDS'
        error_message = _("The address of the recipient is not complete")
        self.write({
            'state': 'error',
            'error_code': error,
            'info_msg': error_message,
        })
        self.notification_ids.sudo().write({
            'notification_status': 'exception',
            'failure_type': self._get_failure_type(error),
            'failure_reason': error_message,
        })
        self.message_id._notify_message_notification_update()

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
            response = iap_tools.iap_jsonrpc(endpoint + PRINT_ENDPOINT, params=params, timeout=timeout)
        except AccessError as ae:
            for doc in params['documents']:
                letter = self.browse(doc['letter_id'])
                letter.state = 'error'
                letter.error_code = 'UNKNOWN_ERROR'
            raise ae
        for doc in response['request']['documents']:
            if doc.get('sent') and response['request_code'] == 200:
                note = _('The document was correctly sent by post.<br>The tracking id is %s', doc['send_id'])
                letter_data = {'info_msg': note, 'state': 'sent', 'error_code': False}
                notification_data = {
                    'notification_status': 'sent',
                    'failure_type': False,
                    'failure_reason': False,
                }
            else:
                error = doc['error'] if response['request_code'] == 200 else response['reason']

                note = _('An error occurred when sending the document by post.<br>Error: %s', self._get_error_message(error))
                letter_data = {
                    'info_msg': note,
                    'state': 'error',
                    'error_code': error if error in ERROR_CODES else 'UNKNOWN_ERROR'
                }
                notification_data = {
                    'notification_status': 'exception',
                    'failure_type': self._get_failure_type(error),
                    'failure_reason': note,
                }

            letter = self.browse(doc['letter_id'])
            letter.write(letter_data)
            letter.notification_ids.sudo().write(notification_data)
        self.message_id._notify_message_notification_update()

    def snailmail_print(self):
        self.write({'state': 'pending'})
        self.notification_ids.sudo().write({
            'notification_status': 'ready',
            'failure_type': False,
            'failure_reason': False,
        })
        self.message_id._notify_message_notification_update()
        if len(self) == 1:
            self._snailmail_print()

    def cancel(self):
        self.write({'state': 'canceled', 'error_code': False})
        self.notification_ids.sudo().write({
            'notification_status': 'canceled',
        })
        self.message_id._notify_message_notification_update()

    @api.model
    def _snailmail_cron(self, autocommit=True):
        letters_send = self.search([
            '|',
            ('state', '=', 'pending'),
            '&',
            ('state', '=', 'error'),
            ('error_code', 'in', ['TRIAL_ERROR', 'CREDIT_ERROR', 'MISSING_REQUIRED_FIELDS'])
        ])
        for letter in letters_send:
            letter._snailmail_print()
            if letter.error_code == 'CREDIT_ERROR':
                break  # avoid spam
            # Commit after every letter sent to avoid to send it again in case of a rollback
            if autocommit:
                self.env.cr.commit()

    @api.model
    def _is_valid_address(self, record):
        record.ensure_one()
        required_keys = ['street', 'city', 'zip', 'country_id']
        return all(record[key] for key in required_keys)

    def _append_cover_page(self, invoice_bin: bytes):
        out_writer = PdfFileWriter()
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
        out_writer.appendPagesFromReader(cover_file)

        # Add a blank buffer page to avoid printing behind the cover page
        if self.duplex:
            out_writer.addBlankPage()

        out_writer.appendPagesFromReader(invoice)

        out_buff = io.BytesIO()
        out_writer.write(out_buff)
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
from . import mail_notification
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
access_snailmail_letter_cancel,access.snailmail.letter.cancel,model_snailmail_letter_cancel,base.group_user,1,1,1,0
access_snailmail_letter_format_error,access.snailmail.letter.format.error,model_snailmail_letter_format_error,base.group_user,1,1,1,0
access_snailmail_letter_missing_required_fields,access.snailmail.letter.missing.required.fields,model_snailmail_letter_missing_required_fields,base.group_user,1,1,1,0

```

## File: static\description\icon.svg

```svg
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 70 70">
  <defs>
    <mask id="mask" x="0" y="0" width="70" height="70" maskUnits="userSpaceOnUse">
      <g id="b">
        <path id="a" d="M4,0H65c4,0,5,1,5,5V65c0,4-1,5-5,5H4c-3,0-4-1-4-5V5C0,1,1,0,4,0Z" fill="#fff" fill-rule="evenodd"/>
      </g>
    </mask>
    <linearGradient id="linear-gradient" x1="-1438.5" y1="477.94" x2="-1439.5" y2="476.94" gradientTransform="matrix(70, 0, 0, -70, 100764.99, 33455.73)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#cd7690"/>
      <stop offset="1" stop-color="#ca5377"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M34.48,69H4c-2,0-4-.15-4-4.08V41.77L16.94,24.83l6.56-2.94,15-15,.71,2.26,11.87-.16-3.77,7.15L40,23.42v1.49l3-3h3c3.89-.06,8.64,27,8.64,27Z" fill="#393939" fill-rule="evenodd" opacity="0.32" style="isolation: isolate"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <g>
        <g opacity="0.4">
          <path d="M36.52,36.47A1.5,1.5,0,0,1,35,35V10.37a1.5,1.5,0,1,1,3,0V35A1.5,1.5,0,0,1,36.52,36.47Z"/>
          <path d="M44.45,18.62H36.52V10.37H48.17a1,1,0,0,1,.88,1.56l-3.72,6.19A1,1,0,0,1,44.45,18.62Z" fill-rule="evenodd"/>
          <path d="M33,63.89a1.51,1.51,0,0,1-1.5-1.5V49.92a1.5,1.5,0,0,1,3,0V62.39A1.5,1.5,0,0,1,33,63.89Z"/>
          <path d="M45.57,23.91H41V35a4.5,4.5,0,0,1-9,0V23.91H20.11a7.3,7.3,0,0,0-7.19,7.4v19a1,1,0,0,0,1,1H51.78a1,1,0,0,0,1-1v-19A7.31,7.31,0,0,0,45.57,23.91ZM26.65,49.32H14.92v-18a5.32,5.32,0,0,1,4.92-5.39,6.68,6.68,0,0,1,6.81,6.39Z" fill-rule="evenodd"/>
        </g>
        <g>
          <path d="M38.54,34.45A1.5,1.5,0,0,1,37,33V8.34a1.5,1.5,0,1,1,3,0V33A1.5,1.5,0,0,1,38.54,34.45Z" fill="#fff"/>
          <path d="M46.47,16.59H38.54V8.34H50.19a1,1,0,0,1,.88,1.57l-3.72,6.18A1,1,0,0,1,46.47,16.59Z" fill="#fff" fill-rule="evenodd"/>
          <path d="M35,61.86a1.5,1.5,0,0,1-1.5-1.5V47.89a1.5,1.5,0,0,1,3,0V60.36A1.5,1.5,0,0,1,35,61.86Z" fill="#fff"/>
          <path d="M47.59,21.89H43V33a4.5,4.5,0,0,1-9,0V21.89H22.14a7.31,7.31,0,0,0-7.2,7.4v19a1,1,0,0,0,1,1H53.8a1,1,0,0,0,1-1v-19A7.31,7.31,0,0,0,47.59,21.89ZM28.67,47.3H16.94v-18a5.31,5.31,0,0,1,4.92-5.39,6.66,6.66,0,0,1,6.81,6.39Z" fill="#fff" fill-rule="evenodd"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\components\message\message.js

```javascript
/** @odoo-module **/

import { Message } from '@mail/components/message/message';

import { patch } from 'web.utils';

const { useState } = owl;

patch(Message.prototype, 'snailmail/static/src/components/message/message.js', {
    /**
     * @override
     */
    _constructor() {
        this._super(...arguments);
        this.snailmailState = useState({
            // Determine if the error dialog is displayed.
            hasDialog: false,
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _onClickFailure() {
        if (this.messageView.message.message_type === 'snailmail') {
            /**
             * Messages from snailmail are considered to have at most one
             * notification. The failure type of the whole message is considered
             * to be the same as the one from that first notification, and the
             * click action will depend on it.
             */
            switch (this.messageView.message.notifications[0].failure_type) {
                case 'sn_credit':
                    // URL only used in this component, not received at init
                    this.messaging.fetchSnailmailCreditsUrl();
                    this.snailmailState.hasDialog = true;
                    break;
                case 'sn_error':
                    this.snailmailState.hasDialog = true;
                    break;
                case 'sn_fields':
                    this.messageView.message.openMissingFieldsLetterAction();
                    break;
                case 'sn_format':
                    this.messageView.message.openFormatLetterAction();
                    break;
                case 'sn_price':
                    this.snailmailState.hasDialog = true;
                    break;
                case 'sn_trial':
                    // URL only used in this component, not received at init
                    this.messaging.fetchSnailmailCreditsUrlTrial();
                    this.snailmailState.hasDialog = true;
                    break;
            }
        } else {
            this._super(...arguments);
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onDialogClosedSnailmailError() {
        this.snailmailState.hasDialog = false;
    },
});

```

## File: static\src\components\message\message.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-inherit="mail.Message" t-inherit-mode="extension">
        <xpath expr="//*[@name='failureIcon']" position="replace">
            <t t-if="messageView.message.message_type === 'snailmail'">
                <i class="o_Message_notificationIcon fa fa-paper-plane"/>
            </t>
            <t t-else="">$0</t>
        </xpath>

        <xpath expr="//*[@name='notificationIcon']" position="replace">
            <t t-if="messageView.message.message_type === 'snailmail'">
                <i class="o_Message_notificationIcon fa fa-paper-plane"/>
            </t>
            <t t-else="">$0</t>
        </xpath>

        <xpath expr="//*[@name='rootCondition']" position="inside">
            <t t-if="snailmailState.hasDialog">
                <SnailmailErrorDialog messageLocalId="messageView.message.localId" t-on-dialog-closed="_onDialogClosedSnailmailError"/>
            </t>
        </xpath>

        <!--
            It was decided that the information displayed for snailmail messages
            has to be different than for standard messages, see task-1907998.
        -->
        <xpath expr="//NotificationPopover" position="replace">
            <t t-if="messageView.message.message_type === 'snailmail'">
                <SnailmailNotificationPopover messageLocalId="messageView.message.localId"/>
            </t>
            <t t-else="">$0</t>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\notification_group\notification_group.js

```javascript
/** @odoo-module **/

import { NotificationGroup } from '@mail/components/notification_group/notification_group';

import { patch } from 'web.utils';

patch(NotificationGroup.prototype, 'snailmail/static/src/components/notification_group/notification_group.js', {

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    image() {
        if (this.group.notification_type === 'snail') {
            return '/snailmail/static/img/snailmail_failure.png';
        }
        return this._super(...arguments);
    },
});

```

## File: static\src\components\notification_group\notification_group.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-inherit="mail.NotificationGroup" t-inherit-mode="extension">
        <xpath expr="//*[hasclass('o_NotificationGroup_inlineText')]" position="inside">
            <t t-if="group.notification_type === 'snail'">
                An error occurred when sending a letter with Snailmail.
            </t>
        </xpath>
    </t>

</templates>

```

## File: static\src\components\snailmail_error_dialog\snailmail_error_dialog.js

```javascript
/** @odoo-module **/

import { registerMessagingComponent } from '@mail/utils/messaging_component';

import Dialog from 'web.OwlDialog';

const { Component } = owl;
const { useRef } = owl.hooks;

class SnailmailErrorDialog extends Component {

    /**
     * @override
     */
    constructor(...args) {
        super(...args);
        // to manually trigger the dialog close event
        this._dialogRef = useRef('dialog');
    }

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @returns {boolean}
     */
    get hasCreditsError() {
        return (
            this.notification.failure_type === 'sn_credit' ||
            this.notification.failure_type === 'sn_trial'
        );
    }

    /**
     * @returns {mail.message}
     */
    get message() {
        return this.messaging && this.messaging.models['mail.message'].get(this.props.messageLocalId);
    }

    /**
     * @returns {mail.notification}
     */
    get notification() {
        // Messages from snailmail are considered to have at most one notification.
        return this.message.notifications[0];
    }

    /**
     * @returns {string}
     */
    get title() {
        return this.env._t("Failed letter");
    }

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onClickCancelLetter() {
        this._dialogRef.comp._close();
        this.message.cancelLetter();
    }

    /**
     * @private
     */
    _onClickClose() {
        this._dialogRef.comp._close();
    }

    /**
     * @private
     */
    _onClickResendLetter() {
        this._dialogRef.comp._close();
        this.message.resendLetter();
    }

}

Object.assign(SnailmailErrorDialog, {
    components: { Dialog },
    props: {
        messageLocalId: String,
    },
    template: 'snailmail.SnailmailErrorDialog',
});

registerMessagingComponent(SnailmailErrorDialog);

export default SnailmailErrorDialog;

```

## File: static\src\components\snailmail_error_dialog\snailmail_error_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="snailmail.SnailmailErrorDialog" owl="1">
        <Dialog contentClass="'o_SnailmailErrorDialog'" title="title" size="'medium'" t-ref="dialog">
            <t t-if="message and notification">
                <t t-if="notification.failure_type === 'sn_credit'">
                    <p class="o_SnailmailErrorDialog_contentCredit">
                        The letter could not be sent due to insufficient credits on your IAP account.
                    </p>
                    <t t-if="messaging.snailmail_credits_url">
                        <div class="text-right">
                            <a class="btn btn-link" t-att-href="messaging.snailmail_credits_url" target="_blank">
                                <i class="fa fa-arrow-right"/> Buy credits
                            </a>
                        </div>
                    </t>
                </t>
                <t t-elif="notification.failure_type === 'sn_trial'">
                    <p class="o_SnailmailErrorDialog_contentTrial">
                        You need credits on your IAP account to send a letter.
                    </p>
                    <t t-if="messaging.snailmail_credits_url_trial">
                        <div class="text-right">
                            <a class="btn btn-link" t-att-href="messaging.snailmail_credits_url_trial">
                                <i class="fa fa-arrow-right"/> Buy credits
                            </a>
                        </div>
                    </t>
                </t>
                <t t-elif="notification.failure_type === 'sn_price'">
                    <p class="o_SnailmailErrorDialog_contentPrice">
                        The country to which you want to send the letter is not supported by our service.
                    </p>
                </t>
                <t t-elif="notification.failure_type === 'sn_error'">
                    <p class="o_SnailmailErrorDialog_contentError">
                        An unknown error occurred. Please contact our <a href="https://www.odoo.com/help" target="new">support</a> for further assistance.
                    </p>
                </t>

                <t t-set-slot="buttons">
                    <t t-if="hasCreditsError">
                        <button class="o_SnailmailErrorDialog_resendLetterButton btn btn-primary" t-on-click="_onClickResendLetter">Re-send letter</button>
                    </t>
                    <button class="o_SnailmailErrorDialog_cancelLetterButton btn"
                        t-att-class="{
                            'btn-primary': !hasCreditsError,
                            'btn-secondary': hasCreditsError,
                        }"
                        t-on-click="_onClickCancelLetter"
                    >
                        Cancel letter
                    </button>
                    <button class="o_SnailmailErrorDialog_closeButton btn btn-secondary" t-on-click="_onClickClose">Close</button>
                </t>
            </t>
        </Dialog>
    </t>

</templates>

```

## File: static\src\components\snailmail_notification_popover\snailmail_notification_popover.js

```javascript
/** @odoo-module **/

import { registerMessagingComponent } from '@mail/utils/messaging_component';

const { Component } = owl;

class SnailmailNotificationPopover extends Component {

    /**
     * @returns {string}
     */
    get iconClass() {
        switch (this.notification.notification_status) {
            case 'sent':
                return 'fa fa-check';
            case 'ready':
                return 'fa fa-clock-o';
            case 'canceled':
                return 'fa fa-trash-o';
            default:
                return 'fa fa-exclamation text-danger';
        }
    }

    /**
     * @returns {string}
     */
    get iconTitle() {
        switch (this.notification.notification_status) {
            case 'sent':
                return this.env._t("Sent");
            case 'ready':
                return this.env._t("Awaiting Dispatch");
            case 'canceled':
                return this.env._t("Canceled");
            default:
                return this.env._t("Error");
        }
    }

    /**
     * @returns {mail.message}
     */
    get message() {
        return this.messaging && this.messaging.models['mail.message'].get(this.props.messageLocalId);
    }

    /**
     * @returns {mail.notification}
     */
    get notification() {
        // Messages from snailmail are considered to have at most one notification.
        return this.message.notifications[0];
    }

}

Object.assign(SnailmailNotificationPopover, {
    props: {
        messageLocalId: String,
    },
    template: 'snailmail.SnailmailNotificationPopover',
});

registerMessagingComponent(SnailmailNotificationPopover);

export default SnailmailNotificationPopover;

```

## File: static\src\components\snailmail_notification_popover\snailmail_notification_popover.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="snailmail.SnailmailNotificationPopover" owl="1">
        <div class="o_SnailmailNotificationPopover">
            <t t-if="message and notification">
                <i class="o_SnailmailNotificationPopover_icon" t-att-class="iconClass" role="img"/>
                <span t-esc="iconTitle"/>
            </t>
        </div>
    </t>

</templates>

```

## File: static\src\js\snailmail_external_layout.js

```javascript
// Change address font-size if needed
document.addEventListener('DOMContentLoaded', function (evt) {
    var recipientAddress = document.querySelector(".address.row > div[name='address'] > address");
    let baseSize = 120;
    if (!recipientAddress) {
        recipientAddress = document.querySelector("div .row.fallback_header > div.col-5.offset-7 > div:first-child");
    }
    var style = window.getComputedStyle(recipientAddress, null); 
    var height = parseFloat(style.getPropertyValue('height'));
    var fontSize = parseFloat(style.getPropertyValue('font-size'));
    recipientAddress.style.fontSize = (baseSize / (height / fontSize)) + 'px';
});

```

## File: static\src\models\message\message.js

```javascript
/** @odoo-module **/

import { registerInstancePatchModel } from '@mail/model/model_core';

registerInstancePatchModel('mail.message', 'snailmail/static/src/models/message.message.js', {

    //----------------------------------------------------------------------
    // Public
    //----------------------------------------------------------------------

    /**
     * Cancels the 'snailmail.letter' corresponding to this message.
     *
     * @returns {Deferred}
     */
    async cancelLetter() {
        // the result will come from longpolling: message_notification_update
        await this.async(() => this.env.services.rpc({
            model: 'mail.message',
            method: 'cancel_letter',
            args: [[this.id]],
        }));
    },
    /**
     * Opens the action about 'snailmail.letter' format error.
     */
    openFormatLetterAction() {
        this.env.bus.trigger('do-action', {
            action: 'snailmail.snailmail_letter_format_error_action',
            options: {
                additional_context: {
                    message_id: this.id,
                },
            },
        });
    },
    /**
     * Opens the action about 'snailmail.letter' missing fields.
     */
    async openMissingFieldsLetterAction() {
        const letterIds = await this.async(() => this.env.services.rpc({
            model: 'snailmail.letter',
            method: 'search',
            args: [[['message_id', '=', this.id]]],
        }));
        this.env.bus.trigger('do-action', {
            action: 'snailmail.snailmail_letter_missing_required_fields_action',
            options: {
                additional_context: {
                    default_letter_id: letterIds[0],
                },
            },
        });
    },
    /**
     * Retries to send the 'snailmail.letter' corresponding to this message.
     */
    async resendLetter() {
        // the result will come from longpolling: message_notification_update
        await this.async(() => this.env.services.rpc({
            model: 'mail.message',
            method: 'send_letter',
            args: [[this.id]],
        }));
    },
});

```

## File: static\src\models\messaging\messaging.js

```javascript
/** @odoo-module **/

import {
    registerInstancePatchModel,
    registerFieldPatchModel,
} from '@mail/model/model_core';
import { attr } from '@mail/model/model_field';

registerInstancePatchModel('mail.messaging', 'snailmail/static/src/models/messaging/messaging.js', {
    async fetchSnailmailCreditsUrl() {
        const snailmail_credits_url = await this.async(() => this.env.services.rpc({
            model: 'iap.account',
            method: 'get_credits_url',
            args: ['snailmail'],
        }));
        this.update({
            snailmail_credits_url,
        });
    },
    async fetchSnailmailCreditsUrlTrial() {
        const snailmail_credits_url_trial = await this.async(() => this.env.services.rpc({
            model: 'iap.account',
            method: 'get_credits_url',
            args: ['snailmail', '', 0, true],
        }));
        this.update({
            snailmail_credits_url_trial,
        });
    },
});

registerFieldPatchModel('mail.messaging', 'snailmail/static/src/models/messaging/messaging.js', {
    snailmail_credits_url: attr(),
    snailmail_credits_url_trial: attr(),
});

```

## File: static\src\models\notification_group\notification_group.js

```javascript
/** @odoo-module **/

import {
    registerInstancePatchModel,
} from '@mail/model/model_core';

registerInstancePatchModel('mail.notification_group', 'snailmail/static/src/models/notification_group/notification_group.js', {

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    openCancelAction() {
        if (this.notification_type !== 'snail') {
            return this._super(...arguments);
        }
        this.env.bus.trigger('do-action', {
            action: 'snailmail.snailmail_letter_cancel_action',
            options: {
                additional_context: {
                    default_model: this.res_model,
                    unread_counter: this.notifications.length,
                },
            },
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _openDocuments() {
        if (this.notification_type !== 'snail') {
            return this._super(...arguments);
        }
        this.env.bus.trigger('do-action', {
            action: {
                name: this.env._t("Snailmail Failures"),
                type: 'ir.actions.act_window',
                view_mode: 'kanban,list,form',
                views: [[false, 'kanban'], [false, 'list'], [false, 'form']],
                target: 'current',
                res_model: this.res_model,
                domain: [['message_ids.snailmail_error', '=', true]],
            },
        });
        if (this.messaging.device.isMobile) {
            // messaging menu has a higher z-index than views so it must
            // be closed to ensure the visibility of the view
            this.messaging.messagingMenu.close();
        }
    },
});

```

## File: views\report_assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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
                    <field name="state" widget="statusbar" statusbar_visible="pending,sent,canceled"/>
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
              sequence="50"/>
</odoo>

```

## File: wizard\snailmail_confirm.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class SnailmailConfirm(models.AbstractModel):
    _name = 'snailmail.confirm'
    _description = 'Snailmail Confirm'

    model_name = fields.Char()

    @api.model
    def show_warning(self):
        return not self.env['ir.config_parameter'].sudo().get_param('%s.warning_shown' % self._name, False)

    def action_open(self):
        view = self.env.ref('snailmail.snailmail_confirm_view')
        return {
            'name': _('Snailmail'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': self._name,
            'views': [(view.id, 'form')],
            'view_id': view.id,
            'target': 'new',
            'res_id': self.id,
            'context': self.env.context
        }

    def action_confirm(self):
        self.env['ir.config_parameter'].sudo().set_param('%s.warning_shown' % self._name, True)
        self._confirm()
        return self._continue()

    def action_cancel(self):
        self.env['ir.config_parameter'].sudo().set_param('%s.warning_shown' % self._name, True)
        return self._continue()

    """
    Called whether the user confirms or cancels posting the letter, e.g. to continue the action
    """
    def _continue(self):
        pass

    """
    Called only when the user confirms sending the letter
    """
    def _confirm(self):
        pass

```

## File: wizard\snailmail_confirm_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="snailmail_confirm_view" model="ir.ui.view">
        <field name="name">snailmail.confirm.view</field>
        <field name="model">snailmail.confirm</field>
        <field name="arch" type="xml">
            <form string="Snailmail Confirmation">
                <p>You are about to send this <field name="model_name" readonly="1" class="oe_inline" /> by post. Are you sure you want to continue?</p>
                <footer>  
                    <button string="Confirm" name="action_confirm" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" name="action_cancel" type="object" class="btn-secondary" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>
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
            wizard.help_message = _("Are you sure you want to discard %s snailmail delivery failures? You won't be able to re-send these letters later!") % (wizard._context.get('unread_counter'))

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
                    <button string="Discard delivery failures" name="cancel_resend_action" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
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

    message_id = fields.Many2one(
        'mail.message',
        default=lambda self: self.env.context.get('message_id', None),
    )
    snailmail_cover = fields.Boolean(
        string='Add a Cover Page',
        default=lambda self: self.env.company.snailmail_cover,
    )

    def update_resend_action(self):
        self.env.company.write({'snailmail_cover': self.snailmail_cover})
        letters_to_resend = self.env['snailmail.letter'].search([
            ('error_code', '=', 'FORMAT_ERROR'),
        ])
        for letter in letters_to_resend:
            old_attachment = letter.attachment_id
            letter.attachment_id = False
            old_attachment.unlink()
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
                    <button string="Update Config and Re-send" name="update_resend_action" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel Letter" name="cancel_letter_action" type="object" class="btn-primary" data-hotkey="w"/>
                    <button string="Close" class="btn-secondary" special="cancel" data-hotkey="z"/>
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
        defaults = super(SnailmailLetterMissingRequiredFields, self).default_get(fields)
        if defaults.get('letter_id'):
            letter = self.env['snailmail.letter'].browse(defaults.get('letter_id'))
            defaults.update({
                'partner_id': letter.partner_id.id,
                'street': letter.street,
                'street2': letter.street2,
                'zip': letter.zip,
                'city': letter.city,
                'state_id': letter.state_id.id,
                'country_id': letter.country_id.id,
            })
        return defaults

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
                <!-- Field present for correct default_get behavior -->
                <field name="letter_id" invisible="1"/>
                <p>The customer address is not complete. Update the address here and re-send the letter.</p>
                <group>
                    <label for="partner_id" string="Address"/>
                    <div class="o_address_format">
                        <field name="partner_id" readonly="1" options="{'no_open': True}" force_save="1"/>
                        <field name="street" placeholder="Street..." class="o_address_street"/>
                        <field name="street2" placeholder="Street 2..." class="o_address_street"/>
                        <field name="city" placeholder="City" class="o_address_city"/>
                        <field name="state_id" class="o_address_state" placeholder="State" options='{"no_open": True}'/>
                        <field name="zip" placeholder="ZIP" class="o_address_zip"/>
                        <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'/>
                    </div>
                </group>
                <footer>
                    <button string="Update address and re-send" type="object" name="update_address_save" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel letter" type="object" name="update_address_cancel" class="btn-secondary" data-hotkey="w"/>
                    <button string="Close" special='cancel' class="btn-secondary" data-hotkey="z"/>
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
from . import snailmail_confirm
from . import snailmail_letter_cancel
from . import snailmail_letter_format_error
from . import snailmail_letter_missing_required_fields

```

