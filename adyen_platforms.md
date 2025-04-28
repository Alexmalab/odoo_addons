# Odoo Module: adyen_platforms

Category: 

This file contains the source code of the Odoo module.

## File: util.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import hashlib
import hmac
import json
import requests
import time
import werkzeug.urls

class AdyenProxyAuth(requests.auth.AuthBase):
    def __init__(self, adyen_account_id):
        super(AdyenProxyAuth, self).__init__()
        self.adyen_account_id = adyen_account_id

    def __call__(self, request):
        h = hmac.new(self.adyen_account_id.proxy_token.encode('utf-8'), digestmod=hashlib.sha256)

        # Craft the message (timestamp|url path|query params|body content)
        msg_timestamp = int(time.time())
        parsed_url = werkzeug.urls.url_parse(request.path_url)
        body = request.body
        if isinstance(body, bytes):
            body = body.decode('utf-8')
        body = json.loads(body)

        message = '%s|%s|%s|%s' % (
            msg_timestamp,  # timestamp
            parsed_url.path,  # url path
            json.dumps(werkzeug.urls.url_decode(parsed_url.query), sort_keys=True),  # url query params sorted by key
            json.dumps(body, sort_keys=True))  # request body

        h.update(message.encode('utf-8'))  # digest the message

        request.headers.update({
            'oe-adyen-uuid': self.adyen_account_id.adyen_uuid,
            'oe-signature': base64.b64encode(h.digest()),
            'oe-timestamp': msg_timestamp,
        })

        return request

```

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Adyen for Platforms',
    'version': '1.0',
    'category': '',
    'summary': 'Base Module for Adyen for Platforms',
    'description': 'Base Module for Adyen for Platforms, used in eCommerce and PoS',
    'depends': ['mail', 'web'],
    'data': [
        'data/adyen_platforms_data.xml',
        'security/ir.model.access.csv',
        'views/adyen_account_templates.xml',
        'views/adyen_account_views.xml',
        'views/adyen_transaction_views.xml',
        'views/assets.xml',
    ],
    'qweb': [
        "static/src/xml/adyen_account_templates.xml",
        "static/src/xml/adyen_transactions_templates.xml",
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\adyen_platforms.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request


class AdyenPlatformsController(http.Controller):

    @http.route('/adyen_platforms/create_account', type='http', auth='user', website=True)
    def adyen_platforms_create_account(self, creation_token):
        request.session['adyen_creation_token'] = creation_token
        return request.redirect('/web?#action=adyen_platforms.adyen_account_action_create')

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import adyen_platforms

```

## File: data\adyen_platforms_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record model="ir.config_parameter" id="adyen_platforms_proxy_url">
            <field name="key">adyen_platforms.proxy_url</field>
            <field name="value">https://payment-adyen.odoo.com/payment_proxy_adyen/</field>
        </record>

        <record model="ir.config_parameter" id="adyen_platforms_onboarding_url">
            <field name="key">adyen_platforms.onboarding_url</field>
            <field name="value">https://www.odoo.com/odoo_adyen/</field>
        </record>

        <record id="adyen_sync_cron" model="ir.cron">
            <field name="name">Adyen Sync</field>
            <field name="model_id" ref="model_adyen_account"/>
            <field name="state">code</field>
            <field name="code">model._sync_adyen_cron()</field>
            <field name="user_id" ref="base.user_root"/>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: models\adyen_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import os
import requests
import uuid
from werkzeug.urls import url_join

from odoo import api, fields, models, _
from odoo.http import request
from odoo.exceptions import UserError, ValidationError
from odoo.tools import date_utils

from odoo.addons.adyen_platforms.util import AdyenProxyAuth

ADYEN_AVAILABLE_COUNTRIES = ['US', 'AT', 'AU', 'BE', 'CA', 'CH', 'CZ', 'DE', 'ES', 'FI', 'FR', 'GB', 'GR', 'HR', 'IE', 'IT', 'LT', 'LU', 'NL', 'PL', 'PT']
TIMEOUT = 60


class AdyenAddressMixin(models.AbstractModel):
    _name = 'adyen.address.mixin'
    _description = 'Adyen for Platforms Address Mixin'

    country_id = fields.Many2one('res.country', string='Country', domain=[('code', 'in', ADYEN_AVAILABLE_COUNTRIES)], required=True)
    country_code = fields.Char(related='country_id.code')
    state_id = fields.Many2one('res.country.state', string='State', domain="[('country_id', '=?', country_id)]")
    state_code = fields.Char(related='state_id.code')
    city = fields.Char('City', required=True)
    zip = fields.Char('ZIP', required=True)
    street = fields.Char('Street', required=True)
    house_number_or_name = fields.Char('House Number Or Name', required=True)


class AdyenIDMixin(models.AbstractModel):
    _name = 'adyen.id.mixin'
    _description = 'Adyen for Platforms ID Mixin'

    id_type = fields.Selection(string='Photo ID type', selection=[
        ('PASSPORT', 'Passport'),
        ('ID_CARD', 'ID Card'),
        ('DRIVING_LICENSE', 'Driving License'),
    ])
    id_front = fields.Binary('Photo ID Front', help="Allowed formats: jpg, pdf, png. Maximum allowed size: 4MB.")
    id_front_filename = fields.Char()
    id_back = fields.Binary('Photo ID Back', help="Allowed formats: jpg, pdf, png. Maximum allowed size: 4MB.")
    id_back_filename = fields.Char()

    def write(self, vals):
        res = super(AdyenIDMixin, self).write(vals)

        # Check file formats
        if vals.get('id_front'):
            self._check_file_requirements(vals.get('id_front'), vals.get('id_front_filename'))
        if vals.get('id_back'):
            self._check_file_requirements(vals.get('id_back'), vals.get('id_back_filename'))

        for adyen_account in self:
            if vals.get('id_front'):
                document_type = adyen_account.id_type
                if adyen_account.id_type in ['ID_CARD', 'DRIVING_LICENSE']:
                    document_type += '_FRONT'
                adyen_account._upload_photo_id(document_type, adyen_account.id_front, adyen_account.id_front_filename)
            if vals.get('id_back') and adyen_account.id_type in ['ID_CARD', 'DRIVING_LICENSE']:
                document_type = adyen_account.id_type + '_BACK'
                adyen_account._upload_photo_id(document_type, adyen_account.id_back, adyen_account.id_back_filename)
            return res

    @api.model
    def _check_file_requirements(self, content, filename):
        file_extension = os.path.splitext(filename)[1]
        file_size = int(len(content) * 3/4) # Compute file_size in bytes
        if file_extension not in ['.jpeg', '.jpg', '.pdf', '.png']:
            raise ValidationError(_('Allowed file formats for photo IDs are jpeg, jpg, pdf or png'))
        if file_size >> 20 > 4 or (file_size >> 10 < 1 and file_extension == '.pdf') or (file_size >> 10 < 100 and file_extension != '.pdf') :
            raise ValidationError(_('Photo ID file size must be between 100kB (1kB for PDFs) and 4MB'))

    def _upload_photo_id(self, document_type, content, filename):
        # The request to be sent to Adyen will be different for Individuals,
        # Shareholders, etc. This method should be implemented by the models
        # inheriting this mixin
        raise NotImplementedError()


class AdyenAccount(models.Model):
    _name = 'adyen.account'
    _inherit = ['mail.thread', 'adyen.id.mixin', 'adyen.address.mixin']

    _description = 'Adyen for Platforms Account'
    _rec_name = 'full_name'

    # Credentials
    proxy_token = fields.Char('Proxy Token')
    adyen_uuid = fields.Char('Adyen UUID')
    account_holder_code = fields.Char('Account Holder Code', default=lambda self: uuid.uuid4().hex)

    company_id = fields.Many2one('res.company', default=lambda self: self.env.company)
    payout_ids = fields.One2many('adyen.payout', 'adyen_account_id', string='Payouts')
    shareholder_ids = fields.One2many('adyen.shareholder', 'adyen_account_id', string='Shareholders')
    bank_account_ids = fields.One2many('adyen.bank.account', 'adyen_account_id', string='Bank Accounts')
    transaction_ids = fields.One2many('adyen.transaction', 'adyen_account_id', string='Transactions')
    transactions_count = fields.Integer(compute='_compute_transactions_count')

    is_business = fields.Boolean('Is a business', required=True)

    # Contact Info
    full_name = fields.Char(compute='_compute_full_name')
    email = fields.Char('Email', required=True)
    phone_number = fields.Char('Phone Number', required=True)

    # Individual
    first_name = fields.Char('First Name')
    last_name = fields.Char('Last Name')
    date_of_birth = fields.Date('Date of birth')
    document_number = fields.Char('ID Number',
        help="The type of ID Number required depends on the country:\n"
             "US: Social Security Number (9 digits or last 4 digits)\n"
             "Canada: Social Insurance Number\nItaly: Codice fiscale\n"
             "Australia: Document Number")
    document_type = fields.Selection(string='Document Type', selection=[
        ('ID', 'ID'),
        ('PASSPORT', 'Passport'),
        ('VISA', 'Visa'),
        ('DRIVINGLICENSE', 'Driving license'),
    ], default='ID')

    # Business
    legal_business_name = fields.Char('Legal Business Name')
    doing_business_as = fields.Char('Doing Business As')
    registration_number = fields.Char('Registration Number')

    # KYC
    kyc_status = fields.Selection(string='KYC Status', selection=[
        ('awaiting_data', 'Data to provide'),
        ('pending', 'Waiting for validation'),
        ('passed', 'Confirmed'),
        ('failed', 'Failed'),
    ], required=True, default='pending')
    kyc_status_message = fields.Char('KYC Status Message', readonly=True)

    _sql_constraints = [
        ('adyen_uuid_uniq', 'UNIQUE(adyen_uuid)', 'Adyen UUID should be unique'),
    ]

    @api.depends('transaction_ids')
    def _compute_transactions_count(self):
        for adyen_account_id in self:
            adyen_account_id.transactions_count = len(adyen_account_id.transaction_ids)

    @api.depends('first_name', 'last_name', 'legal_business_name')
    def _compute_full_name(self):
        for adyen_account_id in self:
            if adyen_account_id.is_business:
                adyen_account_id.full_name = adyen_account_id.legal_business_name
            else:
                adyen_account_id.full_name = "%s %s" % (adyen_account_id.first_name, adyen_account_id.last_name)

    @api.model
    def create(self, values):
        adyen_account_id = super(AdyenAccount, self).create(values)
        self.env.company.adyen_account_id = adyen_account_id.id

        # Create account on odoo.com, proxy and Adyen
        response = adyen_account_id._adyen_rpc('create_account_holder', adyen_account_id._format_data())

        # Save adyen_uuid and proxy_token, that have been generated by odoo.com and the proxy
        adyen_account_id.with_context(update_from_adyen=True).write({
            'adyen_uuid': response['adyen_uuid'],
            'proxy_token': response['proxy_token'],
        })

        # A default payout is created for all adyen accounts
        adyen_account_id.env['adyen.payout'].with_context(update_from_adyen=True).create({
            'code': response['adyen_response']['accountCode'],
            'adyen_account_id': adyen_account_id.id,
        })
        return adyen_account_id

    def write(self, vals):
        res = super(AdyenAccount, self).write(vals)
        if not self.env.context.get('update_from_adyen'):
            self._adyen_rpc('update_account_holder', self._format_data())
        return res

    def unlink(self):
        for adyen_account_id in self:
            adyen_account_id._adyen_rpc('close_account_holder', {
                'accountHolderCode': adyen_account_id.account_holder_code,
            })
        return super(AdyenAccount, self).unlink()

    @api.model
    def action_create_redirect(self):
        '''
        Accessing the FormView to create an Adyen account needs to be done through this action.
        The action will redirect the user to accounts.odoo.com to link an Odoo user_id to the Adyen
        account. After logging in on odoo.com the user will be redirected to his DB with a token in
        the URL. This token is then needed to create the Adyen account.
        '''
        if self.env.company.adyen_account_id:
            # An account already exists, show it
            return {
                'name': _('Adyen Account'),
                'view_mode': 'form',
                'res_model': 'adyen.account',
                'res_id': self.env.company.adyen_account_id.id,
                'type': 'ir.actions.act_window',
            }
        return_url = url_join(self.env['ir.config_parameter'].sudo().get_param('web.base.url'), 'adyen_platforms/create_account')
        onboarding_url = self.env['ir.config_parameter'].sudo().get_param('adyen_platforms.onboarding_url')
        return {
            'type': 'ir.actions.act_url',
            'url': url_join(onboarding_url, 'get_creation_token?return_url=%s' % return_url),
        }

    def action_show_transactions(self):
        return {
            'name': _('Transactions'),
            'view_mode': 'tree,form',
            'domain': [('adyen_account_id', '=', self.id)],
            'res_model': 'adyen.transaction',
            'type': 'ir.actions.act_window',
            'context': {'group_by': ['adyen_payout_id']}
        }

    def _upload_photo_id(self, document_type, content, filename):
        self._adyen_rpc('upload_document', {
            'documentDetail': {
                'accountHolderCode': self.account_holder_code,
                'documentType': document_type,
                'filename': filename,
            },
            'documentContent': content.decode(),
        })

    def _format_data(self):
        data = {
            'accountHolderCode': self.account_holder_code,
            'accountHolderDetails': {
                'address': {
                    'country': self.country_id.code,
                    'stateOrProvince': self.state_id.code or None,
                    'city': self.city,
                    'postalCode': self.zip,
                    'street': self.street,
                    'houseNumberOrName': self.house_number_or_name,
                },
                'email': self.email,
                'fullPhoneNumber': self.phone_number,
            },
            'legalEntity': 'Business' if self.is_business else 'Individual',
        }

        if self.is_business:
            data['accountHolderDetails']['businessDetails'] = {
                'legalBusinessName': self.legal_business_name,
                'doingBusinessAs': self.doing_business_as,
                'registrationNumber': self.registration_number,
            }
        else:
            data['accountHolderDetails']['individualDetails'] = {
                'name': {
                    'firstName': self.first_name,
                    'lastName': self.last_name,
                    'gender': 'UNKNOWN',
                },
                'personalData': {
                    'dateOfBirth': str(self.date_of_birth),
                }
            }

            # documentData cannot be present in the data if not set
            if self.document_number:
                data['accountHolderDetails']['individualDetails']['personalData']['documentData'] = [{
                    'number': self.document_number,
                    'type': self.document_type,
                }]

        return data

    def _adyen_rpc(self, operation, adyen_data={}):
        if operation == 'create_account_holder':
            url = self.env['ir.config_parameter'].sudo().get_param('adyen_platforms.onboarding_url')
            params = {
                'creation_token': request.session.get('adyen_creation_token'),
                'adyen_data': adyen_data,
            }
            auth = None
        else:
            url = self.env['ir.config_parameter'].sudo().get_param('adyen_platforms.proxy_url')
            params = {
                'adyen_uuid': self.adyen_uuid,
                'adyen_data': adyen_data,
            }
            auth = AdyenProxyAuth(self)

        payload = {
            'jsonrpc': '2.0',
            'params': params,
        }
        try:
            req = requests.post(url_join(url, operation), json=payload, auth=auth, timeout=TIMEOUT)
            req.raise_for_status()
        except requests.exceptions.Timeout:
            raise UserError(_('A timeout occured while trying to reach the Adyen proxy.'))
        except Exception as e:
            raise UserError(_('The Adyen proxy is not reachable, please try again later.'))
        response = req.json()

        if 'error' in response:
            name = response['error']['data'].get('name').rpartition('.')[-1]
            if name == 'ValidationError':
                raise ValidationError(response['error']['data'].get('arguments')[0])
            else:
                raise UserError(_("We had troubles reaching Adyen, please retry later or contact the support if the problem persists"))

        result = response.get('result')
        if 'verification' in result:
            self._update_kyc_status(result['verification'])

        return result

    @api.model
    def _sync_adyen_cron(self):
        self._sync_adyen_kyc_status()
        self.env['adyen.transaction'].sync_adyen_transactions()
        self.env['adyen.payout']._process_payouts()

    @api.model
    def _sync_adyen_kyc_status(self):
        for adyen_account_id in self.search([]):
            data = adyen_account_id._adyen_rpc('get_account_holder', {
                'accountHolderCode': adyen_account_id.account_holder_code,
            })
            adyen_account_id._update_kyc_status(data['verification'])

    def _update_kyc_status(self, checks):
        all_checks_status = []

        # Account Holder Checks
        account_holder_checks = checks.get('accountHolder', {})
        account_holder_messages = []
        for check in account_holder_checks.get('checks'):
            all_checks_status.append(check['status'])
            kyc_status_message = self._get_kyc_message(check)
            if kyc_status_message:
                account_holder_messages.append(kyc_status_message)

        # Shareholders Checks
        shareholder_checks = checks.get('shareholders', {})
        shareholder_messages = []
        kyc_status_message = False
        for sc in shareholder_checks:
            shareholder_status = []
            shareholder_id = self.shareholder_ids.filtered(lambda shareholder: shareholder.shareholder_uuid == sc['shareholderCode'])
            for check in sc.get('checks'):
                all_checks_status.append(check['status'])
                shareholder_status.append(check['status'])
                kyc_status_message = self._get_kyc_message(check)
                if kyc_status_message:
                    shareholder_messages.append('[%s] %s' % (shareholder_id.display_name, kyc_status_message))
            shareholder_id.with_context(update_from_adyen=True).write({
                'kyc_status': self.get_status(shareholder_status),
                'kyc_status_message': kyc_status_message,
            })

        # Bank Account Checks
        bank_account_checks = checks.get('bankAccounts', {})
        bank_account_messages = []
        kyc_status_message = False
        for bac in bank_account_checks:
            bank_account_status = []
            bank_account_id = self.bank_account_ids.filtered(lambda bank_account: bank_account.bank_account_uuid == bac['bankAccountUUID'])
            for check in bac.get('checks'):
                all_checks_status.append(check['status'])
                bank_account_status.append(check['status'])
                kyc_status_message = self._get_kyc_message(check)
                if kyc_status_message:
                    bank_account_messages.append('[%s] %s' % (bank_account_id.display_name, kyc_status_message))
            bank_account_id.with_context(update_from_adyen=True).write({
                'kyc_status': self.get_status(bank_account_status),
                'kyc_status_message': kyc_status_message,
            })

        kyc_status = self.get_status(all_checks_status)
        kyc_status_message = self.env['ir.qweb']._render('adyen_platforms.kyc_status_message', {
            'kyc_status': dict(self._fields['kyc_status'].selection)[kyc_status],
            'account_holder_messages': account_holder_messages,
            'shareholder_messages': shareholder_messages,
            'bank_account_messages': bank_account_messages,
        })

        if kyc_status_message.decode() != self.kyc_status_message:
            self.sudo().message_post(body = kyc_status_message, subtype_xmlid="mail.mt_comment") # Message from Odoo Bot

        self.with_context(update_from_adyen=True).write({
            'kyc_status': kyc_status,
            'kyc_status_message': kyc_status_message,
        })

    @api.model
    def get_status(self, statuses):
        if any(status in ['FAILED'] for status in statuses):
            return 'failed'
        if any(status in ['INVALID_DATA', 'RETRY_LIMIT_REACHED', 'AWAITING_DATA'] for status in statuses):
            return 'awaiting_data'
        if any(status in ['DATA_PROVIDED', 'PENDING'] for status in statuses):
            return 'pending'
        return 'passed'

    @api.model
    def _get_kyc_message(self, check):
        if check.get('summary', {}).get('kycCheckDescription'):
            return check['summary']['kycCheckDescription']
        if check.get('requiredFields', {}):
            return _('Missing required fields: ') + ', '.join(check.get('requiredFields'))
        return ''


class AdyenShareholder(models.Model):
    _name = 'adyen.shareholder'
    _inherit = ['adyen.id.mixin', 'adyen.address.mixin']
    _description = 'Adyen for Platforms Shareholder'
    _rec_name = 'full_name'

    adyen_account_id = fields.Many2one('adyen.account', ondelete='cascade')
    shareholder_reference = fields.Char('Reference', default=lambda self: uuid.uuid4().hex)
    shareholder_uuid = fields.Char('UUID') # Given by Adyen
    first_name = fields.Char('First Name', required=True)
    last_name = fields.Char('Last Name', required=True)
    full_name = fields.Char(compute='_compute_full_name')
    date_of_birth = fields.Date('Date of birth', required=True)
    document_number = fields.Char('ID Number', 
            help="The type of ID Number required depends on the country:\n"
             "US: Social Security Number (9 digits or last 4 digits)\n"
             "Canada: Social Insurance Number\nItaly: Codice fiscale\n"
             "Australia: Document Number")

    # KYC
    kyc_status = fields.Selection(string='KYC Status', selection=[
        ('awaiting_data', 'Data to provide'),
        ('pending', 'Waiting for validation'),
        ('passed', 'Confirmed'),
        ('failed', 'Failed'),
    ], required=True, default='pending')
    kyc_status_message = fields.Char('KYC Status Message', readonly=True)

    @api.depends('first_name', 'last_name')
    def _compute_full_name(self):
        for adyen_shareholder_id in self:
            adyen_shareholder_id.full_name = '%s %s' % (adyen_shareholder_id.first_name, adyen_shareholder_id.last_name)

    @api.model
    def create(self, values):
        adyen_shareholder_id = super(AdyenShareholder, self).create(values)
        response = adyen_shareholder_id.adyen_account_id._adyen_rpc('update_account_holder', adyen_shareholder_id._format_data())
        shareholders = response['accountHolderDetails']['businessDetails']['shareholders']
        created_shareholder = next(shareholder for shareholder in shareholders if shareholder['shareholderReference'] == adyen_shareholder_id.shareholder_reference)
        adyen_shareholder_id.with_context(update_from_adyen=True).write({
            'shareholder_uuid': created_shareholder['shareholderCode'],
        })
        return adyen_shareholder_id

    def write(self, vals):
        res = super(AdyenShareholder, self).write(vals)
        if not self.env.context.get('update_from_adyen'):
            self.adyen_account_id._adyen_rpc('update_account_holder', self._format_data())
        return res

    def unlink(self):
        for shareholder_id in self:
            shareholder_id.adyen_account_id._adyen_rpc('delete_shareholders', {
                'accountHolderCode': shareholder_id.adyen_account_id.account_holder_code,
                'shareholderCodes': [shareholder_id.shareholder_uuid],
            })
        return super(AdyenShareholder, self).unlink()

    def _upload_photo_id(self, document_type, content, filename):
        self.adyen_account_id._adyen_rpc('upload_document', {
            'documentDetail': {
                'accountHolderCode': self.adyen_account_id.account_holder_code,
                'shareholderCode': self.shareholder_uuid,
                'documentType': document_type,
                'filename': filename,
            },
            'documentContent': content.decode(),
        })

    def _format_data(self):
        data = {
            'accountHolderCode': self.adyen_account_id.account_holder_code,
            'accountHolderDetails': {
                'businessDetails': {
                    'shareholders': [{
                        'shareholderCode': self.shareholder_uuid or None,
                        'shareholderReference': self.shareholder_reference,
                        'address': {
                            'city': self.city,
                            'country': self.country_code,
                            'houseNumberOrName': self.house_number_or_name,
                            'postalCode': self.zip,
                            'stateOrProvince': self.state_id.code or None,
                            'street': self.street,
                        },
                        'name': {
                            'firstName': self.first_name,
                            'lastName': self.last_name,
                            'gender': 'UNKNOWN'
                        },
                        'personalData': {
                            'dateOfBirth': str(self.date_of_birth),
                        }
                    }]
                }
            }
        }

        # documentData cannot be present in the data if not set
        if self.document_number:
            data['accountHolderDetails']['businessDetails']['shareholders'][0]['personalData']['documentData'] = [{
                'number': self.document_number,
                'type': 'ID',
            }]

        return data

class AdyenBankAccount(models.Model):
    _name = 'adyen.bank.account'
    _description = 'Adyen for Platforms Bank Account'

    adyen_account_id = fields.Many2one('adyen.account', ondelete='cascade')
    bank_account_reference = fields.Char('Reference', default=lambda self: uuid.uuid4().hex)
    bank_account_uuid = fields.Char('UUID') # Given by Adyen
    owner_name = fields.Char('Owner Name', required=True)
    country_id = fields.Many2one('res.country', string='Country', domain=[('code', 'in', ADYEN_AVAILABLE_COUNTRIES)], required=True)
    country_code = fields.Char(related='country_id.code')
    currency_id = fields.Many2one('res.currency', string='Currency', required=True)
    iban = fields.Char('IBAN')
    account_number = fields.Char('Account Number')
    branch_code = fields.Char('Branch Code')
    bank_code = fields.Char('Bank Code')
    account_type = fields.Selection(string='Account Type', selection=[
        ('checking', 'Checking'),
        ('savings', 'Savings'),
    ])
    owner_country_id = fields.Many2one('res.country', string='Owner Country')
    owner_state_id = fields.Many2one('res.country.state', 'Owner State', domain="[('country_id', '=?', owner_country_id)]")
    owner_street = fields.Char('Owner Street')
    owner_city = fields.Char('Owner City')
    owner_zip = fields.Char('Owner ZIP')
    owner_house_number_or_name = fields.Char('Owner House Number or Name')

    bank_statement = fields.Binary('Bank Statement', help="You need to provide a bank statement to allow payouts. \
        The file must be a bank statement, a screenshot of your online banking environment, a letter from the bank or a cheque and must contain \
        the logo of the bank or it's name in a unique font, the bank account details, the name of the account holder.\
        Allowed formats: jpg, pdf, png. Maximum allowed size: 10MB.")
    bank_statement_filename = fields.Char()

    # KYC
    kyc_status = fields.Selection(string='KYC Status', selection=[
        ('awaiting_data', 'Data to provide'),
        ('pending', 'Waiting for validation'),
        ('passed', 'Confirmed'),
        ('failed', 'Failed'),
    ], required=True, default='pending')
    kyc_status_message = fields.Char('KYC Status Message', readonly=True)

    @api.model
    def create(self, values):
        adyen_bank_account_id = super(AdyenBankAccount, self).create(values)
        response = adyen_bank_account_id.adyen_account_id._adyen_rpc('update_account_holder', adyen_bank_account_id._format_data())
        bank_accounts = response['accountHolderDetails']['bankAccountDetails']
        created_bank_account = next(bank_account for bank_account in bank_accounts if bank_account['bankAccountReference'] == adyen_bank_account_id.bank_account_reference)
        adyen_bank_account_id.with_context(update_from_adyen=True).write({
            'bank_account_uuid': created_bank_account['bankAccountUUID'],
        })
        return adyen_bank_account_id

    def write(self, vals):
        res = super(AdyenBankAccount, self).write(vals)
        if not self.env.context.get('update_from_adyen'):
            self.adyen_account_id._adyen_rpc('update_account_holder', self._format_data())
        if 'bank_statement' in vals:
            self._upload_bank_statement(vals['bank_statement'], vals['bank_statement_filename'])
        return res

    def unlink(self):
        for bank_account_id in self:
            bank_account_id.adyen_account_id._adyen_rpc('delete_bank_accounts', {
                'accountHolderCode': bank_account_id.adyen_account_id.account_holder_code,
                'bankAccountUUIDs': [bank_account_id.bank_account_uuid],
            })
        return super(AdyenBankAccount, self).unlink()

    def _format_data(self):
        return {
            'accountHolderCode': self.adyen_account_id.account_holder_code,
            'accountHolderDetails': {
                'bankAccountDetails': [{
                    'accountNumber': self.account_number or None,
                    'accountType': self.account_type or None,
                    'bankAccountReference': self.bank_account_reference,
                    'bankAccountUUID': self.bank_account_uuid or None,
                    'bankCode': self.bank_code or None,
                    'branchCode': self.branch_code or None,
                    'countryCode': self.country_code,
                    'currencyCode': self.currency_id.name,
                    'iban': self.iban or None,
                    'ownerCity': self.owner_city or None,
                    'ownerCountryCode': self.owner_country_id.code or None,
                    'ownerHouseNumberOrName': self.owner_house_number_or_name or None,
                    'ownerName': self.owner_name,
                    'ownerPostalCode': self.owner_zip or None,
                    'ownerState': self.owner_state_id.code or None,
                    'ownerStreet': self.owner_street or None,
                }],
            }
        }

    def _upload_bank_statement(self, content, filename):
        file_extension = os.path.splitext(filename)[1]
        file_size = len(content.encode('utf-8'))
        if file_extension not in ['.jpeg', '.jpg', '.pdf', '.png']:
            raise ValidationError(_('Allowed file formats for bank statements are jpeg, jpg, pdf or png'))
        if file_size >> 20 > 10 or (file_size >> 10 < 10 and file_extension != '.pdf') :
            raise ValidationError(_('Bank statements must be greater than 10kB (except for PDFs) and smaller than 10MB'))

        self.adyen_account_id._adyen_rpc('upload_document', {
            'documentDetail': {
                'accountHolderCode': self.adyen_account_id.account_holder_code,
                'bankAccountUUID': self.bank_account_uuid,
                'documentType': 'BANK_STATEMENT',
                'filename': filename,
            },
            'documentContent': content,
        })


class AdyenPayout(models.Model):
    _name = 'adyen.payout'
    _description = 'Adyen for Platforms Payout'

    @api.depends('payout_schedule')
    def _compute_next_scheduled_payout(self):
        today = fields.date.today()
        for adyen_payout_id in self:
            adyen_payout_id.next_scheduled_payout = date_utils.end_of(today, adyen_payout_id.payout_schedule)

    adyen_account_id = fields.Many2one('adyen.account', ondelete='cascade')
    adyen_bank_account_id = fields.Many2one('adyen.bank.account', string='Bank Account',
        help='The bank account to which the payout is to be made. If left blank, a bank account is automatically selected')
    name = fields.Char('Name', default='Default', required=True)
    code = fields.Char('Account Code')
    payout_schedule = fields.Selection(string='Schedule', selection=[
        ('day', 'Daily'),
        ('week', 'Weekly'),
        ('month', 'Monthly'),
    ], default='week', required=True)
    next_scheduled_payout = fields.Date('Next scheduled payout', compute=_compute_next_scheduled_payout, store=True)
    transaction_ids = fields.One2many('adyen.transaction', 'adyen_payout_id', string='Transactions')

    @api.model
    def create(self, values):
        adyen_payout_id = super(AdyenPayout, self).create(values)
        if not adyen_payout_id.env.context.get('update_from_adyen'):
            response = adyen_payout_id.adyen_account_id._adyen_rpc('create_payout', {
                'accountHolderCode': adyen_payout_id.adyen_account_id.account_holder_code,
            })
            adyen_payout_id.with_context(update_from_adyen=True).write({
                'code': response['accountCode'],
            })
        return adyen_payout_id

    def unlink(self):
        for adyen_payout_id in self:
            adyen_payout_id.adyen_account_id._adyen_rpc('close_payout', {
                'accountCode': adyen_payout_id.code,
            })
        return super(AdyenPayout, self).unlink()

    @api.model
    def _process_payouts(self):
        for adyen_payout_id in self.search([('next_scheduled_payout', '<', fields.Date.today())]):
            adyen_payout_id.send_payout_request(notify=False)
            adyen_payout_id._compute_next_scheduled_payout()

    def send_payout_request(self, notify=True):
        response = self.adyen_account_id._adyen_rpc('account_holder_balance', {
            'accountHolderCode': self.adyen_account_id.account_holder_code,
        })
        balances = next(account_balance['detailBalance']['balance'] for account_balance in response['balancePerAccount'] if account_balance['accountCode'] == self.code)
        if notify and not balances:
            self.env['bus.bus'].sendone(
                (self._cr.dbname, 'res.partner', self.env.user.partner_id.id),
                {'type': 'simple_notification', 'title': _('No pending balance'), 'message': _('No balance is currently awaitng payout.')}
            )
        for balance in balances:
            response = self.adyen_account_id._adyen_rpc('payout_request', {
                'accountCode': self.code,
                'accountHolderCode': self.adyen_account_id.account_holder_code,
                'bankAccountUUID': self.adyen_bank_account_id.bank_account_uuid or None,
                'amount': balance,
            })
            if notify and response['resultCode'] == 'Received':
                currency_id = self.env['res.currency'].search([('name', '=', balance['currency'])])
                value = round(balance['value'] / (10 ** currency_id.decimal_places), 2) # Convert from minor units
                amount = str(value) + currency_id.symbol if currency_id.position == 'after' else currency_id.symbol + str(value)
                message = _('Successfully sent payout request for %s', amount)
                self.env['bus.bus'].sendone(
                    (self._cr.dbname, 'res.partner', self.env.user.partner_id.id),
                    {'type': 'simple_notification', 'title': _('Payout Request sent'), 'message': message}
                )

    def _fetch_transactions(self, page=1):
        response = self.adyen_account_id._adyen_rpc('get_transactions', {
            'accountHolderCode': self.adyen_account_id.account_holder_code,
            'transactionListsPerAccount': [{
                'accountCode': self.code,
                'page': page,
            }]
        })
        transaction_list = response['accountTransactionLists'][0]
        return transaction_list['transactions'], transaction_list['hasNextPage']

```

## File: models\adyen_transaction.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from datetime import datetime
from pytz import UTC

from odoo import api, fields, models
from odoo.tools import DEFAULT_SERVER_DATETIME_FORMAT


class AdyenTransaction(models.Model):
    _name = 'adyen.transaction'
    _description = 'Adyen for Platforms Transaction'
    _order = 'date desc'

    adyen_account_id = fields.Many2one('adyen.account')
    reference = fields.Char('Reference')
    amount = fields.Float('Amount')
    currency_id = fields.Many2one('res.currency', string='Currency')
    date = fields.Datetime('Date')
    description = fields.Char('Description')
    status = fields.Selection(string='Type', selection=[
        ('PendingCredit', 'Pending Credit'),
        ('CreditFailed', 'Credit Failed'),
        ('Credited', 'Credited'),
        ('Converted', 'Converted'),
        ('PendingDebit', 'Pending Debit'),
        ('DebitFailed', 'Debit Failed'),
        ('Debited', 'Debited'),
        ('DebitReversedReceived', 'Debit Reversed Received'),
        ('DebitedReversed', 'Debit Reversed'),
        ('ChargebackReceived', 'Chargeback Received'),
        ('Chargeback', 'Chargeback'),
        ('ChargebackReversedReceived', 'Chargeback Reversed Received'),
        ('ChargebackReversed', 'Chargeback Reversed'),
        ('Payout', 'Payout'),
        ('PayoutReversed', 'Payout Reversed'),
        ('FundTransfer', 'Fund Transfer'),
        ('PendingFundTransfer', 'Pending Fund Transfer'),
        ('ManualCorrected', 'Manual Corrected'),
    ])
    adyen_payout_id = fields.Many2one('adyen.payout')

    @api.model
    def sync_adyen_transactions(self):
        ''' Method called by cron to sync transactions from Adyen.
            Updates the status of pending transactions and create missing ones.
        '''
        for payout_id in self.env['adyen.payout'].search([]):
            page = 1
            has_next_page = True
            new_transactions = True
            pending_statuses = ['PendingCredit', 'PendingDebit', 'DebitReversedReceived', 'ChargebackReceived', 'ChargebackReversedReceived', 'PendingFundTransfer']
            pending_transaction_ids = payout_id.transaction_ids.filtered(lambda tr: tr.status in pending_statuses)

            while has_next_page and (new_transactions or pending_transaction_ids):
                # Fetch next transaction page
                transactions, has_next_page = payout_id._fetch_transactions(page)
                for transaction in transactions:
                    transaction_reference = transaction.get('paymentPspReference') or transaction.get('pspReference')
                    transaction_id = payout_id.transaction_ids.filtered(lambda tr: tr.reference == transaction_reference)
                    if transaction_id:
                        new_transactions = False
                        if transaction_id in pending_transaction_ids:
                            # Update transaction status
                            transaction_id.sudo().write({
                                'status': transaction['transactionStatus'],
                            })
                            pending_transaction_ids -= transaction_id
                    else:
                        currency_id = self.env['res.currency'].search([('name', '=', transaction['amount']['currency'])])
                        # New transaction
                        self.env['adyen.transaction'].sudo().create({
                            'adyen_account_id': payout_id.adyen_account_id.id,
                            'reference': transaction_reference,
                            'amount': transaction['amount']['value'] / (10 ** currency_id.decimal_places),
                            'currency_id': currency_id.id,
                            'date': datetime.strptime(transaction['creationDate'], '%Y-%m-%dT%H:%M:%S%z').astimezone(UTC).strftime(DEFAULT_SERVER_DATETIME_FORMAT),
                            'description': transaction.get('description'),
                            'status': transaction['transactionStatus'],
                            'adyen_payout_id': payout_id.id,
                        })
                page += 1

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = "res.company"

    adyen_account_id = fields.Many2one('adyen.account', string='Adyen Account', readonly=True)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import adyen_account
from . import adyen_transaction
from . import res_company

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_adyen_account_group_erp_manager,adyen.account,model_adyen_account,base.group_erp_manager,1,1,1,1
access_adyen_shareholder_group_erp_manager,adyen.shareholder,model_adyen_shareholder,base.group_erp_manager,1,1,1,1
access_adyen_bank_account_group_erp_manager,adyen.bank.account,model_adyen_bank_account,base.group_erp_manager,1,1,1,1
access_adyen_payout_group_erp_manager,adyen.payout,model_adyen_payout,base.group_erp_manager,1,1,1,1
access_adyen_transaction_group_erp_manager,adyen.transaction,model_adyen_transaction,base.group_erp_manager,1,0,0,0

```

## File: static\src\js\adyen_account_fields.js

```javascript
odoo.define('adyen_platforms.fields', function (require) {
"use strict";

var core = require('web.core');
var FieldSelection = require('web.relational_fields').FieldSelection;
var field_registry = require('web.field_registry');

var qweb = core.qweb;

var AdyenKYCStatusTag = FieldSelection.extend({
    _render: function () {
        this.$el.append(qweb.render('AdyenKYCStatusTag', {
            value: this.value,
        }));
    },
});

field_registry.add("adyen_kyc_status_tag", AdyenKYCStatusTag);

});

```

## File: static\src\js\adyen_account_views.js

```javascript
odoo.define('adyen_platforms.account_views', function (require) {
"use strict";

var core = require('web.core');
var Dialog = require('web.Dialog');
var FormController = require('web.FormController');
var FormView = require('web.FormView');
var viewRegistry = require('web.view_registry');

var _t = core._t;
var QWeb = core.qweb;

var AdyenAccountFormController = FormController.extend({
    _saveRecord: function (recordID, options) {
        if(this.model.isNew(this.handle) && this.canBeSaved()) {
            var _super = this._super.bind(this, recordID, options);
            var buttons = [
                {
                    text: _t("Create"),
                    classes: 'btn-primary o_adyen_confirm',
                    close: true,
                    disabled: true,
                    click: function () {
                        this.close();
                        _super();
                    },
                },
                {
                    text: _t("Cancel"),
                    close: true,
                }
            ];

            var dialog = new Dialog(this, {
                size: 'extra-large',
                buttons: buttons,
                title: _t("Confirm your Adyen Account Creation"),
                $content: QWeb.render('AdyenAccountCreationConfirmation', {
                    data: this.model.get(this.handle).data,
                }),
            });

            dialog.open().opened(function () {
                dialog.$el.on('change', '.opt_in_checkbox', function (ev) {
                    ev.preventDefault();
                    dialog.$footer.find('.o_adyen_confirm')[0].disabled = !ev.currentTarget.checked;
                });
            });
        } else if (!this.model.isNew(this.handle)) {
            return this._super.apply(this, arguments);
        }
    },
});

var AdyenAccountFormView = FormView.extend({
    config: _.extend({}, FormView.prototype.config, {
        Controller: AdyenAccountFormController,
    }),
});

viewRegistry.add('adyen_account_form', AdyenAccountFormView);

});

```

## File: static\src\js\adyen_transactions.js

```javascript
odoo.define('adyen_platforms.transactions', function (require) {
"use strict";

var ListController = require('web.ListController');
var ListView = require('web.ListView');
var viewRegistry = require('web.view_registry');

var TransactionsListController = ListController.extend({
    buttons_template: 'AdyenTransactionsListView.buttons',
    events: _.extend({}, ListController.prototype.events, {
        'click .o_button_sync_transactions': '_onTransactionsSync',
    }),

    _onTransactionsSync: function () {
        var self = this;
        this._rpc({
            model: 'adyen.transaction',
            method: 'sync_adyen_transactions',
            args: [],
        }).then(function () {
            self.trigger_up('reload');
        });
    }
});

var TransactionsListView = ListView.extend({
    config: _.extend({}, ListView.prototype.config, {
        Controller: TransactionsListController,
    }),
});

viewRegistry.add('adyen_transactions_tree', TransactionsListView);
});

```

## File: static\src\xml\adyen_account_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="AdyenAccountCreationConfirmation">
        <main>
            <div class="alert alert-info">
                You can start processing payments as soon as your application is sent.<br/>
                Payouts will be blocked until your application has been accepted.<br/>
                We will notify you when the status of the review changes, or if additional data is required.<br/>
            </div>

            <h3>Submitted data</h3>
            <table class="table table-sm">
                <t t-if="data.is_business">
                    <tr>
                        <td>Legal Entity</td>
                        <td>Business</td>
                    </tr>
                    <tr>
                        <td>Legal Business Name</td>
                        <td t-esc="data.legal_business_name"/>
                    </tr>
                    <tr>
                        <td>Doing Business As</td>
                        <td t-esc="data.doing_business_as"/>
                    </tr>
                    <tr>
                        <td>Registration Number</td>
                        <td t-esc="data.registration_number"/>
                    </tr>
                </t>
                <t t-else="">
                    <tr>
                        <td>Legal Entity</td>
                        <td>Individual</td>
                    </tr>
                    <tr>
                        <td>First Name</td>
                        <td t-esc="data.first_name"/>
                    </tr>
                    <tr>
                        <td>Last Name</td>
                        <td t-esc="data.last_name"/>
                    </tr>
                    <tr>
                        <td>Date of Birth</td>
                        <td t-esc="data.date_of_birth.format('YYYY-MM-DD')"/>
                    </tr>
                </t>
                <tr>
                    <td>Email</td>
                    <td t-esc="data.email"/>
                </tr>
                <tr>
                    <td>Phone Number</td>
                    <td t-esc="data.phone_number"/>
                </tr>
                <tr>
                    <td>Address</td>
                    <td>
                        <t t-esc="data.street"/> <t t-esc="data.house_number_or_name"/>,
                        <t t-esc="data.city"/> <t t-if="data.state_code" t-esc="data.state_code"/> <t t-esc="data.zip"/>,
                        <t t-esc="data.country_code"/>
                    </td>
                </tr>
            </table>

            <h3>Disclaimers</h3>
            <p>WARNING: Please note that the right to use the payment services is only for
            sales in your own name. You may not resell, hire or on any other basis allow third
            parties to use the payment services to enable such third parties to be paid for their
            services. You may not use the payment services for different types of product and
            services than as registered with your application. In particular you confirm that you
            will not use the payment services for any type of product or service appearing in the
            <a href="https://www.odoo.com/odoo_adyen/static/src/pdf/marketpay_prohibited_products_and_services.pdf">
            Prohibited Products and Services List of our Processor</a>. If we or our
            Processor at any time discover that the information you provided about your
            business is incorrect or has changed without informing us or if you violate any of
            these conditions, the services may be suspended and/or terminated with
            immediate effect and fines may be applied by the Credit Card Schemes and/or the
            authorities for unregistered or inappropriate use of payment services which will in
            such case be payable by you.</p>

            <h3>Contractual Relationship</h3>
            <p>The payment processing services ordered by you by placing this order will be
            provided to you by Adyen N.V. (hereafter “Processor”), with which you are
            entering into a direct agreement by confirming this order. Odoo S.A.
            (hereafter “We /Us”) will assist and support you in your use of the services to be
            provided by the Processor and we will provide you first line assistance with and
            enable you to connect to the systems of Processor to be able to use its services.
            For this purpose, you hereby instruct Processor to provide us access to your data
            and setting in Processor’s systems which are used by Processor to provide the
            services and authorise us to manage these on your behalf.</p>

            <h3>Pricing</h3>
            <ul>
                <li>Payment and processing fees are listed on <a href="http://www.adyen.com/pricing">adyen.com/pricing</a>.</li>
                <li>Payouts cost 0.20€ (EU) or 0.22$ (US) each.</li>
                <li>Chargebacks cost 7.5€ each.</li>
                <li>Onboarding and KYC cost 5€.</li>
            </ul>

            <input type="checkbox" class="opt_in_checkbox"/>
            I confirm I have taken notice of and accept the following terms and restrictions:
            <ul>
                <li>Adyen MarketPay Terms and Conditions (click <a href="https://www.odoo.com/odoo_adyen/static/src/pdf/marketpay_terms_and_conditions.pdf">here</a> to download and review)</li>
                <li>Adyen Restricted and Prohibited Products and Services list (click <a href="https://www.odoo.com/odoo_adyen/static/src/pdf/marketpay_prohibited_products_and_services.pdf">here</a> to download and review)</li>
            </ul>
        </main>
    </t>

    <t t-name="AdyenKYCStatusTag">
        <t t-if="value === 'awaiting_data'">
            <span class="badge badge-pill badge-warning">Data to provide</span>
        </t>
        <t t-if="value === 'pending'">
            <span class="badge badge-pill badge-info">Waiting for validation</span>
        </t>
        <t t-if="value === 'passed'">
            <span class="badge badge-pill badge-success">Confirmed</span>
        </t>
        <t t-if="value === 'failed'">
            <span class="badge badge-pill badge-danger">Failed</span>
        </t>
    </t>
</templates>

```

## File: static\src\xml\adyen_transactions_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-extend="ListView.buttons" t-name="AdyenTransactionsListView.buttons">
        <t t-jquery="button.o_list_button_discard" t-operation="after">
            <button type="button" class="btn btn-primary o_button_sync_transactions">
                Sync Transactions
            </button>
        </t>
    </t>
</templates>

```

## File: views\adyen_account_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="kyc_status_message">
        <p>New KYC Status: <t- t-esc="kyc_status"/></p>
        <p t-if="account_holder_messages + shareholder_messages + bank_account_messages">
            Reason(s):
            <ul>
                <li t-if="account_holder_messages" t-foreach="account_holder_messages" t-as="message">
                    <t t-esc="message"/>
                </li>
                <li t-if="shareholder_messages">
                    Shareholders:
                    <ul>
                        <li t-foreach="shareholder_messages" t-as="message">
                            <t t-esc="message"/>
                        </li>
                    </ul>
                </li>
                <li t-if="bank_account_messages">
                    Bank Accounts:
                    <ul>
                        <li t-foreach="bank_account_messages" t-as="message">
                            <t t-esc="message"/>
                        </li>
                    </ul>
                </li>
            </ul>
        </p>
    </template>
</odoo>

```

## File: views\adyen_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="adyen_account_view_form" model="ir.ui.view">
        <field name="name">adyen.account.view.form</field>
        <field name="model">adyen.account</field>
        <field name="arch" type="xml">
            <form string="Adyen Account" create="false" js_class="adyen_account_form">
                <header>
                    <field name="kyc_status" widget="statusbar" statusbar_visible="awaiting_data,pending,passed"/>
                </header>
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_show_transactions" type="object"
                            class="oe_stat_button" icon="fa-credit-card">
                            <div class="o_stat_info">
                                <field name="transactions_count" class="o_stat_value"/>
                                <span class="o_stat_text"> Transactions</span>
                            </div>
                        </button>
                    </div>

                    <group>
                        <field name="adyen_uuid" invisible="1"/>

                        <group>
                            <field name="is_business" attrs="{'readonly': [('id', '!=', False)]}"/>
                        </group>

                        <group>
                            <field name="company_id" readonly="1"/>
                        </group>

                        <group string="Individual" attrs="{'invisible': [('is_business', '=', True)]}">
                            <field name="first_name"
                                attrs="{'required': [('is_business', '=', False)]}"/>
                            <field name="last_name"
                                attrs="{'required': [('is_business', '=', False)]}"/>
                            <field name="date_of_birth"
                                attrs="{'required': [('is_business', '=', False)]}"/>
                            <field name="document_number"
                                attrs="{'invisible': [('country_code', 'not in', ['AU', 'CA', 'GR', 'IT', 'US'])], 'required': [('is_business', '=', False), ('country_id', 'in', ['AU', 'CA', 'GR', 'IT', 'US'])]}"/>
                            <field name="document_type"
                                attrs="{'invisible': [('country_code', '!=', 'AU')], 'required': [('is_business', '=', False), ('country_id', '=', 'AU')]}"/>
                            <field name="id_type" attrs="{'invisible': [('adyen_uuid', '=', False)]}"/>
                            <field name="id_front" filename="id_front_filename" attrs="{'invisible': [('adyen_uuid', '=', False)]}"/>
                            <field name="id_front_filename" invisible="1"/>
                            <field name="id_back" filename="id_back_filename"
                                attrs="{'invisible': [('id_type', 'not in', ['ID_CARD', 'DRIVING_LICENSE'])]}"/>
                            <field name="id_back_filename" invisible="1"/> 
                        </group>

                        <group string="Business" attrs="{'invisible': [('is_business', '=', False)]}">
                            <field name="legal_business_name"
                                attrs="{'required': [('is_business', '=', True)]}"/>
                            <field name="doing_business_as"
                                attrs="{'required': [('is_business', '=', True)]}"/>
                            <field name="registration_number"
                                attrs="{'required': [('is_business', '=', True)]}"/>
                        </group>

                        <group string="Contact">
                            <field name="email" widget="email"/>
                            <field name="phone_number"/>
                            <label for="street" string="Address"/>
                            <div class="o_address_format">
                                <field name="street" placeholder="Street" class="o_address_street"/>
                                <field name="house_number_or_name" placeholder="House number or name" class="o_address_street"/>
                                <field name="city" placeholder="City" class="o_address_city"/>
                                <field name="state_id" class="o_address_state" placeholder="State" options="{'no_open': True, 'no_quick_create': True}"
                                    attrs="{'required': [('country_code', 'in', ['AU', 'CA', 'IT', 'US'])]}"/>
                                <field name="state_code" invisible="1"/>
                                <field name="zip" placeholder="ZIP" class="o_address_zip"/>
                                <field name="country_id" placeholder="Country" class="o_address_country" options="{'no_open': True, 'no_create': True}"/>
                                <field name="country_code" invisible="1"/>
                            </div>
                        </group>
                    </group>
                    <notebook attrs="{'invisible': [('adyen_uuid', '=', False)]}">
                        <page string="Shareholders" attrs="{'invisible': [('is_business', '=', False)], 'required': [('is_business', '=', True)]}">
                            <field name="shareholder_ids">
                                <tree>
                                    <field name="first_name"/>
                                    <field name="last_name"/>
                                    <field name="kyc_status" widget="adyen_kyc_status_tag"/>
                                </tree>
                            </field>
                        </page>
                        <page string="Bank Accounts">
                            <field name="bank_account_ids">
                                <tree>
                                    <field name="owner_name"/>
                                    <field name="iban"/>
                                    <field name="account_number"/>
                                    <field name="kyc_status" widget="adyen_kyc_status_tag"/>
                                </tree>
                            </field>
                        </page>
                        <page string="Payouts">
                            <field name="payout_ids">
                                <tree>
                                    <field name="name"/>
                                    <field name="payout_schedule"/>
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="adyen_shareholder_view_form" model="ir.ui.view">
        <field name="name">adyen.shareholder.view.form</field>
        <field name="model">adyen.shareholder</field>
        <field name="arch" type="xml">
            <form string="Adyen Shareholder">
                <header>
                    <field name="kyc_status" widget="statusbar" statusbar_visible="awaiting_data,pending,passed"/>
                </header>
                <sheet>
                    <div class="alert alert-warning" role="alert"
                        attrs="{'invisible': [('kyc_status_message', '=', False)]}">
                        <field name="kyc_status_message"/>
                    </div>
                    <group>
                        <field name="shareholder_uuid" invisible="1"/>
                        <group>
                            <field name="first_name"/>
                            <field name="last_name"/>
                            <field name="date_of_birth"/>
                        </group>
                        <group>
                            <field name="document_number"
                                attrs="{'invisible': [('country_code', 'not in', ['IT', 'US'])], 'required': [('country_code', 'in', ['IT', 'US'])]}"/>
                            <field name="id_type" attrs="{'invisible': [('shareholder_uuid', '=', False)]}"/>
                            <field name="id_front" filename="id_front_filename" attrs="{'invisible': [('shareholder_uuid', '=', False)]}"/>
                            <field name="id_front_filename" invisible="1"/>
                            <field name="id_back" filename="id_back_filename"
                                attrs="{'required': [('id_type', 'in', ['ID_CARD', 'DRIVING_LICENSE'])], 'invisible': [('id_type', 'not in', ['ID_CARD', 'DRIVING_LICENSE'])]}"/>
                            <field name="id_back_filename" invisible="1"/> 
                        </group>
                        <group>
                            <label for="street" string="Address"/>
                            <div class="o_address_format">
                                <field name="street" placeholder="Street" class="o_address_street"/>
                                <field name="house_number_or_name" placeholder="House number or name" class="o_address_street"/>
                                <field name="city" placeholder="City" class="o_address_city"/>
                                <field name="state_id" class="o_address_state" placeholder="State" options="{'no_open': True, 'no_quick_create': True}"
                                    attrs="{'required': [('country_code', 'in', ['AU', 'CA', 'IT', 'US'])]}"/>
                                <field name="zip" placeholder="ZIP" class="o_address_zip"/>
                                <field name="country_id" placeholder="Country" class="o_address_country" options="{'no_open': True, 'no_create': True}"/>
                                <field name="country_code" invisible="1"/>
                            </div>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="adyen_bank_account_view_form" model="ir.ui.view">
        <field name="name">adyen.bank.account.view.form</field>
        <field name="model">adyen.bank.account</field>
        <field name="arch" type="xml">
            <form string="Adyen Bank Account">
                <header>
                    <field name="kyc_status" widget="statusbar" statusbar_visible="awaiting_data,pending,passed"/>
                </header>
                <sheet>
                    <div class="alert alert-warning" role="alert"
                        attrs="{'invisible': [('kyc_status_message', '=', False)]}">
                        <field name="kyc_status_message"/>
                    </div>
                    <group>
                        <field name="bank_account_uuid" invisible="1"/>
                        <group>
                            <field name="country_id" options='{"no_open": True, "no_create": True}'/>
                            <field name="country_code" invisible="1"/>
                            <field name="currency_id"/>
                            <field name="iban"
                                attrs="{'invisible': [('country_code', 'not in', ['AT', 'BE', 'CH', 'CZ', 'DE', 'ES', 'FI', 'FR', 'GB', 'GR', 'HR', 'IE', 'IT', 'LT', 'LU', 'NL', 'PL', 'PT'])],
                                        'required': [('country_code', 'in', ['AT', 'BE', 'CH', 'CZ', 'DE', 'ES', 'FI', 'FR', 'GB', 'GR', 'HR', 'IE', 'IT', 'LT', 'LU', 'NL', 'PL', 'PT'])]}"/>
                            <field name="account_type"
                                attrs="{'invisible': [('country_code', '!=', 'US')], 'required': [('country_code', '=', 'US')]}"/>
                            <field name="account_number"
                                attrs="{'invisible': [('country_code', 'not in', ['AU', 'CA', 'US'])], 'required': [('country_code', 'in', ['AU', 'CA', 'US'])]}"/>
                            <field name="branch_code"
                                attrs="{'invisible': [('country_code', 'not in', ['AU', 'CA', 'US'])], 'required': [('country_code', 'in', ['AU', 'CA', 'US'])]}"/>
                            <field name="bank_code"
                                attrs="{'invisible': [('country_code', '!=', 'CA')], 'required': [('country_code', '=', 'CA')]}"/>
                            <field name="bank_statement" filename="bank_statement_filename" attrs="{'invisible': [('bank_account_uuid', '=', False)]}"/>
                            <field name="bank_statement_filename" invisible="1"/>
                        </group>
                        <group>
                            <field name="owner_name"/>
                            <label for="owner_street" string="Owner Address" attrs="{'invisible': [('country_code', 'not in', ['CA', 'US'])]}"/>
                            <div class="o_address_format" attrs="{'invisible': [('country_code', 'not in', ['CA', 'US'])]}">
                                <field name="owner_street" placeholder="Street" class="o_address_street" attrs="{'required': [('country_code', 'in', ['CA', 'US'])]}"/>
                                <field name="owner_house_number_or_name" placeholder="House number or name" class="o_address_street" attrs="{'required': [('country_code', 'in', ['CA', 'US'])]}"/>
                                <field name="owner_city" placeholder="City" class="o_address_city" attrs="{'required': [('country_code', 'in', ['CA', 'US'])]}"/>
                                <field name="owner_state_id" class="o_address_state" placeholder="State" options="{'no_open': True, 'no_quick_create': True}"
                                    attrs="{'required': [('country_code', 'in', ['CA', 'US'])]}"/>
                                <field name="owner_zip" placeholder="ZIP" class="o_address_zip" attrs="{'required': [('country_code', 'in', ['CA', 'US'])]}"/>
                                <field name="owner_country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'
                                    attrs="{'required': [('country_code', 'in', ['CA', 'US'])]}"/>
                            </div>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="adyen_payout_view_form" model="ir.ui.view">
        <field name="name">adyen.payout.view.form</field>
        <field name="model">adyen.payout</field>
        <field name="arch" type="xml">
            <form string="Adyen Payout">
                <header>
                    <button name="send_payout_request" string="Request a payout now" class="oe_highlight" type="object" attrs="{'invisible': [('code', '=', False)]}"/>
                </header>
                <sheet>
                    <group>
                        <field name="code" invisible="1"/>
                        <field name="name"/>
                        <field name="payout_schedule"/>
                        <field name="adyen_bank_account_id"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="adyen_account_action_create" model="ir.actions.act_window">
        <field name="name">Create an Adyen Account</field>
        <field name="res_model">adyen.account</field>
        <field name="view_mode">form</field>
    </record>
</odoo>

```

## File: views\adyen_transaction_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="adyen_transaction_view_form" model="ir.ui.view">
        <field name="name">adyen.transaction.view.form</field>
        <field name="model">adyen.transaction</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="reference"/>
                        <field name="amount" widget='monetary'/>
                        <field name="currency_id" invisible="1"/>
                        <field name="date"/>
                        <field name="description"/>
                        <field name="status"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="adyen_transaction_view_list" model="ir.ui.view">
        <field name="name">adyen.transaction.view.list</field>
        <field name="model">adyen.transaction</field>
        <field name="arch" type="xml">
            <tree js_class="adyen_transactions_tree">
                <field name="reference"/>
                <field name="amount" widget='monetary'/>
                <field name="currency_id" invisible="1"/>
                <field name="date"/>
                <field name="description"/>
                <field name="status"/>
            </tree>
        </field>
    </record>
</odoo>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" inherit_id="web.assets_backend" name="Adyen for Platforms Backend Assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/adyen_platforms/static/src/js/adyen_account_fields.js"></script>
            <script type="text/javascript" src="/adyen_platforms/static/src/js/adyen_account_views.js"></script>
            <script type="text/javascript" src="/adyen_platforms/static/src/js/adyen_transactions.js"></script>
        </xpath>
    </template>
</odoo>

```

