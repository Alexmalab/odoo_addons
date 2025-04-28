# Odoo Module: mail_plugin

Category: Sales/CRM

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Mail Plugin',
    'version': '1.0',
    'category': 'Sales/CRM',
    'sequence': 5,
    'summary': 'Allows integration with mail plugins.',
    'description': "Integrate Odoo with your mailbox, get information about contacts directly inside your mailbox, log content of emails as internal notes",
    'depends': [
        'web',
        'contacts',
        'iap'
    ],
    'data': [
        'views/mail_plugin_login.xml',
        'views/res_partner_iap_views.xml',
        'security/ir.model.access.csv',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\authenticate.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import datetime
import hmac
import json
import logging
import odoo
import werkzeug

from odoo import _, http
from odoo.http import request
from werkzeug.exceptions import NotFound

_logger = logging.getLogger(__name__)


class Authenticate(http.Controller):

    @http.route(['/mail_client_extension/auth', '/mail_plugin/auth'], type='http', auth="user", methods=['GET'], website=True)
    def auth(self, **values):
        """
         Once authenticated this route renders the view that shows an app wants to access Odoo.
         The user is invited to allow or deny the app. The form posts to `/mail_client_extension/auth/confirm`.

         old route name "/mail_client_extension/auth is deprecated as of saas-14.3,it is not needed for newer
         versions of the mail plugin but necessary for supporting older versions
         """
        if not request.env.user._is_internal():
            return request.render('mail_plugin.app_error', {'error': _('Access Error: Only Internal Users can link their inboxes to this database.')})
        return request.render('mail_plugin.app_auth', values)

    @http.route(['/mail_client_extension/auth/confirm', '/mail_plugin/auth/confirm'], type='http', auth="user", methods=['POST'])
    def auth_confirm(self, scope, friendlyname, redirect, info=None, do=None, **kw):
        """
        Called by the `app_auth` template. If the user decided to allow the app to access Odoo, a temporary auth code
        is generated and they are redirected to `redirect` with this code in the URL. It should redirect to the app, and
        the app should then exchange this auth code for an access token by calling
        `/mail_client/auth/access_token`.

        old route name "/mail_client_extension/auth/confirm is deprecated as of saas-14.3,it is not needed for newer
        versions of the mail plugin but necessary for supporting older versions
        """
        parsed_redirect = werkzeug.urls.url_parse(redirect)
        params = parsed_redirect.decode_query()
        if do:
            name = friendlyname if not info else f'{friendlyname}: {info}'
            auth_code = self._generate_auth_code(scope, name)
            # params is a MultiDict which does not support .update() with kwargs
            # the state attribute is needed for the gmail connector
            params.update({'success': 1, 'auth_code': auth_code, 'state': kw.get('state', '')})
        else:
            params.update({'success': 0, 'state': kw.get('state', '')})
        updated_redirect = parsed_redirect.replace(query=werkzeug.urls.url_encode(params))
        return request.redirect(updated_redirect.to_url(), local=False)

    # In this case, an exception will be thrown in case of preflight request if only POST is allowed.
    @http.route(['/mail_client_extension/auth/access_token', '/mail_plugin/auth/access_token'], type='json', auth="none", cors="*",
                methods=['POST', 'OPTIONS'])
    def auth_access_token(self, auth_code='', **kw):
        """
        Called by the external app to exchange an auth code, which is temporary and was passed in a URL, for an
        access token, which is permanent, and can be used in the `Authorization` header to authorize subsequent requests

        old route name "/mail_client_extension/auth/access_token is deprecated as of saas-14.3,it is not needed for newer
        versions of the mail plugin but necessary for supporting older versions
        """
        if not auth_code:
            return {"error": "Invalid code"}
        auth_message = self._get_auth_code_data(auth_code)
        if not auth_message:
            return {"error": "Invalid code"}
        request.update_env(user=auth_message['uid'])
        scope = 'odoo.plugin.' + auth_message.get('scope', '')
        api_key = request.env['res.users.apikeys']._generate(
            scope,
            auth_message['name'],
            datetime.datetime.now() + datetime.timedelta(days=1)
        )
        return {'access_token': api_key}

    def _get_auth_code_data(self, auth_code):
        data, auth_code_signature = auth_code.split('.')
        data = base64.b64decode(data)
        auth_code_signature = base64.b64decode(auth_code_signature)
        signature = odoo.tools.misc.hmac(request.env(su=True), 'mail_plugin', data).encode()
        if not hmac.compare_digest(auth_code_signature, signature):
            return None

        auth_message = json.loads(data)
        # Check the expiration
        if datetime.datetime.utcnow() - datetime.datetime.fromtimestamp(auth_message['timestamp']) > datetime.timedelta(
                minutes=3):
            return None

        return auth_message

    # Using UTC explicitly in case of a distributed system where the generation and the signature verification do not
    # necessarily happen on the same server
    def _generate_auth_code(self, scope, name):
        if not request.env.user._is_internal():
            raise NotFound()
        auth_dict = {
            'scope': scope,
            'name': name,
            'timestamp': int(datetime.datetime.utcnow().timestamp()),
            # <- elapsed time should be < 3 mins when verifying
            'uid': request.uid,
        }
        auth_message = json.dumps(auth_dict, sort_keys=True).encode()
        signature = odoo.tools.misc.hmac(request.env(su=True), 'mail_plugin', auth_message).encode()
        auth_code = "%s.%s" % (base64.b64encode(auth_message).decode(), base64.b64encode(signature).decode())
        _logger.info('Auth code created - user %s, scope %s', request.env.user, scope)
        return auth_code

```

## File: controllers\mail_plugin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import json
import logging
import requests
from markupsafe import Markup
from werkzeug.exceptions import Forbidden

from odoo import http, tools, _
from odoo.addons.iap.tools import iap_tools
from odoo.exceptions import AccessError
from odoo.http import request

_logger = logging.getLogger(__name__)


class MailPluginController(http.Controller):

    @http.route('/mail_client_extension/modules/get', type="json", auth="outlook", csrf=False, cors="*")
    def modules_get(self, **kwargs):
        """
            deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary
            for supporting older versions
        """
        return {'modules': ['contacts', 'crm']}

    @http.route('/mail_plugin/partner/enrich_and_create_company',
                type="json", auth="outlook", cors="*")
    def res_partner_enrich_and_create_company(self, partner_id):
        """
        Route used when the user clicks on the create and enrich partner button
        it will try to find a company using IAP, if a company is found
        the enriched company will then be created in the database
        """

        partner = request.env['res.partner'].browse(partner_id).exists()

        if not partner:
            return {'error': _("This partner does not exist")}

        if partner.parent_id:
            return {'error': _("The partner already has a company related to him")}

        normalized_email = partner.email_normalized
        if not normalized_email:
            return {'error': _('The email of this contact is not valid and we can not enrich it')}

        company, enrichment_info = self._create_company_from_iap(normalized_email)

        if company:
            partner.write({'parent_id': company})

        return {
            'enrichment_info': enrichment_info,
            'company': self._get_company_data(company),
        }

    @http.route('/mail_plugin/partner/enrich_and_update_company', type='json', auth='outlook', cors='*')
    def res_partner_enrich_and_update_company(self, partner_id):
        """
        Enriches an existing company using IAP
        """
        partner = request.env['res.partner'].browse(partner_id).exists()

        if not partner:
            return {'error': _("This partner does not exist")}

        if not partner.is_company:
            return {'error': 'Contact must be a company'}

        normalized_email = partner.email_normalized
        if not normalized_email:
            return {'error': 'The email of this contact is not valid and we can not enrich it'}

        domain = tools.email_domain_extract(normalized_email)
        iap_data = self._iap_enrich(domain)

        if 'enrichment_info' in iap_data:  # means that an issue happened with the enrichment request
            return {
                'enrichment_info': iap_data['enrichment_info'],
                'company': self._get_company_data(partner),
            }

        phone_numbers = iap_data.get('phone_numbers')

        partner_values = {}

        if not partner.phone and phone_numbers:
            partner_values.update({'phone': phone_numbers[0]})

        if not partner.iap_enrich_info:
            partner_values.update({'iap_enrich_info': json.dumps(iap_data)})

        if not partner.image_128:
            logo_url = iap_data.get('logo')
            if logo_url:
                try:
                    response = requests.get(logo_url, timeout=2)
                    if response.ok:
                        partner_values.update({'image_1920': base64.b64encode(response.content)})
                except Exception:
                    pass

        model_fields_to_iap_mapping = {
            'street': 'street_name',
            'city': 'city',
            'zip': 'postal_code',
            'website': 'domain',
        }

        # only update keys for which we dont have values yet
        partner_values.update({
            model_field: iap_data.get(iap_key)
            for model_field, iap_key in model_fields_to_iap_mapping.items() if not partner[model_field]
        })

        partner.write(partner_values)

        partner.message_post_with_source(
            'iap_mail.enrich_company',
            render_values=iap_data,
            subtype_xmlid='mail.mt_note',
        )

        return {
            'enrichment_info': {'type': 'company_updated'},
            'company': self._get_company_data(partner),
        }

    @http.route(['/mail_client_extension/partner/get', '/mail_plugin/partner/get']
        , type="json", auth="outlook", cors="*")
    def res_partner_get(self, email=None, name=None, partner_id=None, **kwargs):
        """
        returns a partner given it's id or an email and a name.
        In case the partner does not exist, we return partner having an id -1, we also look if an existing company
        matching the contact exists in the database, if none is found a new company is enriched and created automatically

        old route name "/mail_client_extension/partner/get is deprecated as of saas-14.3, it is not needed for newer
        versions of the mail plugin but necessary for supporting older versions, only the route name is deprecated not
        the entire method.
        """

        if not (partner_id or (name and email)):
            return {'error': _('You need to specify at least the partner_id or the name and the email')}

        if partner_id:
            partner = request.env['res.partner'].browse(partner_id).exists()
            return self._get_contact_data(partner)

        normalized_email = tools.email_normalize(email)
        if not normalized_email:
            return {'error': _('Bad Email.')}

        notification_emails = request.env['mail.alias.domain'].sudo().search([]).mapped('default_from_email')
        if normalized_email in notification_emails:
            return {
                'partner': {
                    'name': _('Notification'),
                    'email': normalized_email,
                    'enrichment_info': {
                        'type': 'odoo_custom_error', 'info': _('This is your notification address. Search the Contact manually to link this email to a record.'),
                    },
                },
            }

        # Search for the partner based on the email.
        # If multiple are found, take the first one.
        partner = request.env['res.partner'].search(['|', ('email', 'in', [normalized_email, email]),
                                                     ('email_normalized', '=', normalized_email)], limit=1)

        response = self._get_contact_data(partner)

        # if no partner is found in the database, we should also return an empty one having id = -1, otherwise older versions of
        # plugin won't work
        if not response['partner']:
            response['partner'] = {
                'id': -1,
                'email': email,
                'name': name,
                'enrichment_info': None,
            }
            company = self._find_existing_company(normalized_email)

            can_create_partner = request.env['res.partner'].has_access('create')

            if not company and can_create_partner:  # create and enrich company
                company, enrichment_info = self._create_company_from_iap(normalized_email)
                response['partner']['enrichment_info'] = enrichment_info
            response['partner']['company'] = self._get_company_data(company)

        return response

    @http.route('/mail_plugin/partner/search', type="json", auth="outlook", cors="*")
    def res_partners_search(self, search_term, limit=30, **kwargs):
        """
        Used for the plugin search contact functionality where the user types a string query in order to search for
        matching contacts, the string query can either be the name of the contact, it's reference or it's email.
        We choose these fields because these are probably the most interesting fields that the user can perform a
        search on.
        The method returns an array containing the dicts of the matched contacts.
        """
        normalized_email = tools.email_normalize(search_term)

        if normalized_email:
            filter_domain = [('email_normalized', 'ilike', search_term)]
        else:
            filter_domain = ['|', '|', ('complete_name', 'ilike', search_term), ('ref', '=', search_term),
                             ('email', 'ilike', search_term)]

        # Search for the partner based on the email.
        # If multiple are found, take the first one.
        partners = request.env['res.partner'].search(filter_domain, limit=limit)

        partners = [
            self._get_partner_data(partner)
            for partner in partners
        ]
        return {"partners": partners}

    @http.route(['/mail_client_extension/partner/create', '/mail_plugin/partner/create'],
                type="json", auth="outlook", cors="*")
    def res_partner_create(self, email, name, company):
        """
        params email: email of the new partner
        params name: name of the new partner
        params company: parent company id of the new partner
        """
        notification_emails = request.env['mail.alias.domain'].sudo().search([]).mapped('default_from_email')
        if tools.email_normalize(email) in notification_emails:
            raise Forbidden()
        # old route name "/mail_client_extension/partner/create is deprecated as of saas-14.3,it is not needed for newer
        # versions of the mail plugin but necessary for supporting older versions
        # TODO search the company again instead of relying on the one provided here?
        # Create the partner if needed.
        partner_info = {
            'name': name,
            'email': email,
        }

        #see if the partner has a parent company
        if company and company > -1:
            partner_info['parent_id'] = company
        partner = request.env['res.partner'].create(partner_info)

        response = {'id': partner.id}
        return response

    @http.route('/mail_plugin/log_mail_content', type="json", auth="outlook", cors="*")
    def log_mail_content(self, model, res_id, message, attachments=None):
        """Log the email on the given record.

        :param model: Model of the record on which we want to log the email
        :param res_id: ID of the record
        :param message: Body of the email
        :param attachments: List of attachments of the email.
            List of tuple: (filename, base 64 encoded content)
        """
        if model not in self._mail_content_logging_models_whitelist():
            raise Forbidden()

        if attachments:
            attachments = [
                (name, base64.b64decode(content))
                for name, content in attachments
            ]

        request.env[model].browse(res_id).message_post(body=Markup(message), attachments=attachments)
        return True

    @http.route('/mail_plugin/get_translations', type="json", auth="outlook", cors="*")
    def get_translations(self):
        return self._prepare_translations()

    def _iap_enrich(self, domain):
        """
        Returns enrichment data for a given domain, in case an error happens the response will
        contain an enrichment_info key explaining what went wrong
        """
        if domain in iap_tools._MAIL_PROVIDERS:
            # Can not enrich the provider domain names (gmail.com; outlook.com, etc)
            return {'enrichment_info': {'type': 'missing_data'}}

        enriched_data = {}
        try:
            response = request.env['iap.enrich.api']._request_enrich({domain: domain})  # The key doesn't matter
        except iap_tools.InsufficientCreditError:
            enriched_data['enrichment_info'] = {'type': 'insufficient_credit', 'info': request.env['iap.account'].get_credits_url('reveal')}
        except Exception:
            enriched_data["enrichment_info"] = {'type': 'other', 'info': 'Unknown reason'}
        else:
            enriched_data = response.get(domain)
            if not enriched_data:
                enriched_data = {'enrichment_info': {'type': 'no_data', 'info': 'The enrichment API found no data for the email provided.'}}
        return enriched_data

    def _find_existing_company(self, email):
        """Find the company corresponding to the given domain and its IAP cache.

        :param email: Email of the company we search
        :return: The partner corresponding to the company
        """
        search = self._get_iap_search_term(email)

        partner_iap = request.env["res.partner.iap"].sudo().search([("iap_search_domain", "=", search)], limit=1)
        if partner_iap:
            return partner_iap.partner_id.sudo(False)

        return request.env["res.partner"].search([("is_company", "=", True), ("email_normalized", "=ilike", "%" + search)], limit=1)

    def _get_company_data(self, company):
        if not company:
            return {'id': -1}

        try:
            company.check_access('read')
        except AccessError:
            return {'id': company.id, 'name': _('No Access')}

        fields_list = ['id', 'name', 'phone', 'mobile', 'email', 'website']

        company_values = dict((fname, company[fname]) for fname in fields_list)
        company_values['address'] = {'street': company.street,
                                     'city': company.city,
                                     'zip': company.zip,
                                     'country': company.country_id.name if company.country_id else ''}
        company_values['additionalInfo'] = json.loads(company.iap_enrich_info) if company.iap_enrich_info else {}
        company_values['image'] = company.image_1920

        return company_values

    def _create_company_from_iap(self, email):
        domain = tools.email_domain_extract(email)
        iap_data = self._iap_enrich(domain)
        if 'enrichment_info' in iap_data:
            return None, iap_data['enrichment_info']

        phone_numbers = iap_data.get('phone_numbers')
        emails = iap_data.get('email')
        new_company_info = {
            'is_company': True,
            'name': iap_data.get("name") or domain,
            'street': iap_data.get("street_name"),
            'city': iap_data.get("city"),
            'zip': iap_data.get("postal_code"),
            'phone': phone_numbers[0] if phone_numbers else None,
            'website': iap_data.get("domain"),
            'email': emails[0] if emails else None
        }

        logo_url = iap_data.get('logo')
        if logo_url:
            try:
                response = requests.get(logo_url, timeout=2)
                if response.ok:
                    new_company_info['image_1920'] = base64.b64encode(response.content)
            except Exception as e:
                _logger.warning('Download of image for new company %s failed, error %s', new_company_info.name, e)

        if iap_data.get('country_code'):
            country = request.env['res.country'].search([('code', '=', iap_data['country_code'].upper())])
            if country:
                new_company_info['country_id'] = country.id
                if iap_data.get('state_code'):
                    state = request.env['res.country.state'].search([
                        ('code', '=', iap_data['state_code']),
                        ('country_id', '=', country.id)
                    ])
                    if state:
                        new_company_info['state_id'] = state.id

        new_company_info.update({
            'iap_search_domain': self._get_iap_search_term(email),
            'iap_enrich_info': json.dumps(iap_data),
        })

        new_company = request.env['res.partner'].create(new_company_info)

        new_company.message_post_with_source(
            'iap_mail.enrich_company',
            render_values=iap_data,
            subtype_xmlid='mail.mt_note',
        )

        return new_company, {'type': 'company_created'}

    def _get_partner_data(self, partner):

        fields_list = ['id', 'name', 'email', 'phone', 'mobile', 'is_company']

        partner_values = dict((fname, partner[fname]) for fname in fields_list)
        partner_values['image'] = partner.image_128
        partner_values['title'] = partner.function
        partner_values['enrichment_info'] = None

        try:
            partner.check_access('write')
            partner_values['can_write_on_partner'] = True
        except AccessError:
            partner_values['can_write_on_partner'] = False

        if not partner_values['name']:
            # Always ensure that the partner has a name
            name, email_normalized = tools.parse_contact_from_email(partner_values['email'])
            partner_values['name'] = name or email_normalized

        return partner_values

    def _get_contact_data(self, partner):
        """
        method used to return partner related values, it can be overridden by other modules if extra information have to
        be returned with the partner (e.g., leads, ...)
        """
        if partner:
            partner_response = self._get_partner_data(partner)
            if partner.company_type == 'company':
                partner_response['company'] = self._get_company_data(partner)
            elif partner.parent_id:
                partner_response['company'] = self._get_company_data(partner.parent_id)
            else:
                partner_response['company'] = self._get_company_data(None)
        else:  # no partner found
            partner_response = {}

        return {
            'partner': partner_response,
            'user_companies': request.env.user.company_ids.ids,
            'can_create_partner': request.env['res.partner'].has_access('create'),
        }

    def _mail_content_logging_models_whitelist(self):
        """
        Returns all models that emails can be logged to and that can be used by the "log_mail_content" method,
        it can be overridden by sub modules in order to whitelist more models
        """
        return ['res.partner']

    def _get_iap_search_term(self, email):
        """Return the domain or the email depending if the domain is blacklisted or not.

        So if the domain is blacklisted, we search based on the entire email address
        (e.g. asbl@gmail.com). But if the domain is not blacklisted, we search based on
        the domain (e.g. bob@sncb.be -> sncb.be)
        """
        domain = tools.email_domain_extract(email)
        return ("@" + domain) if domain not in iap_tools._MAIL_DOMAIN_BLACKLIST else email

    def _translation_modules_whitelist(self):
        """
        Returns the list of modules to be translated
        Other mail plugin modules have to override this method to include their module names
        """
        return ['mail_plugin']

    def _prepare_translations(self):
        lang = request.env['res.users'].browse(request.uid).lang
        translations_per_module = request.env["ir.http"].get_translations_for_webclient(
            self._translation_modules_whitelist(), lang)[0]
        translations_dict = {}
        for module in self._translation_modules_whitelist():
            translations = translations_per_module.get(module, {})
            messages = translations.get('messages', {})
            for message in messages:
                translations_dict.update({message['id']: message['string']})
        return translations_dict

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import authenticate
from . import mail_plugin

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import BadRequest

from odoo import models
from odoo.http import request

class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _auth_method_outlook(cls):
        access_token = request.httprequest.headers.get('Authorization')
        if not access_token:
            raise BadRequest('Access token missing')

        if access_token.startswith('Bearer '):
            access_token = access_token[7:]

        user_id = request.env["res.users.apikeys"]._check_credentials(scope='odoo.plugin.outlook', key=access_token)
        if not user_id:
            raise BadRequest('Access token invalid')

        # take the identity of the API key user
        request.update_env(user=user_id)

        # switch to the user context
        request.update_context(**request.env.user.context_get())

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    iap_enrich_info = fields.Text('IAP Enrich Info', help='IAP response stored as a JSON string',
                                  compute='_compute_partner_iap_info')

    iap_search_domain = fields.Char('Search Domain / Email',
                                compute='_compute_partner_iap_info')

    def _compute_partner_iap_info(self):
        partner_iaps = self.env['res.partner.iap'].sudo().search([('partner_id', 'in', self.ids)])
        partner_iaps_per_partner = {
            partner_iap.partner_id: partner_iap
            for partner_iap in partner_iaps
        }

        for partner in self:
            partner_iap = partner_iaps_per_partner.get(partner)
            if partner_iap:
                partner.iap_enrich_info = partner_iap.iap_enrich_info
                partner.iap_search_domain = partner_iap.iap_search_domain
            else:
                partner.iap_enrich_info = False
                partner.iap_search_domain = False

    @api.model_create_multi
    def create(self, vals_list):
        partners = super().create(vals_list)
        # Not done with inverse method so we do not need to search
        # for existing <res.partner.iap>
        partner_iap_vals_list = [{
            'partner_id': partner.id,
            'iap_enrich_info': vals.get('iap_enrich_info'),
            'iap_search_domain': vals.get('iap_search_domain'),
        } for partner, vals in zip(partners, vals_list) if vals.get('iap_enrich_info') or vals.get('iap_search_domain')]
        self.env['res.partner.iap'].sudo().create(partner_iap_vals_list)
        return partners

    def write(self, vals):
        res = super(ResPartner, self).write(vals)

        if 'iap_enrich_info' in vals or 'iap_search_domain' in vals:
            # Not done with inverse method so we do need to search
            # for existing <res.partner.iap> only once
            partner_iaps = self.env['res.partner.iap'].sudo().search([('partner_id', 'in', self.ids)])
            missing_partners = self
            for partner_iap in partner_iaps:
                if 'iap_enrich_info' in vals:
                    partner_iap.iap_enrich_info = vals['iap_enrich_info']
                if 'iap_search_domain' in vals:
                    partner_iap.iap_search_domain = vals['iap_search_domain']

                missing_partners -= partner_iap.partner_id

            if missing_partners:
                # Create new <res.partner.iap> for missing records
                self.env['res.partner.iap'].sudo().create([
                    {
                        'partner_id': partner.id,
                        'iap_enrich_info': vals.get('iap_enrich_info'),
                        'iap_search_domain': vals.get('iap_search_domain'),
                    } for partner in missing_partners
                ])
        return res

```

## File: models\res_partner_iap.py

```python
# -*- coding: utf-8 -*-


from odoo import fields, models


class ResPartnerIap(models.Model):
    """Technical model which stores the response returned by IAP.

    The goal of this model is to not enrich 2 times the same company. We do it in a
    separate model to not add heavy field (iap_enrich_info) on the <res.partner>
    model.

    We also save the requested domain, so whatever the values are on the <res.partner>,
    we will always retrieve the already enriched <res.partner> and the corresponding
    IAP information.
    """

    _name = 'res.partner.iap'
    _description = 'Partner IAP'

    partner_id = fields.Many2one('res.partner', string='Partner',
                                 ondelete='cascade', required=True)
    iap_search_domain = fields.Char('Search Domain / Email', help='Domain used to find the company')
    iap_enrich_info = fields.Text('IAP Enrich Info', help='IAP response stored as a JSON string', readonly=True)

    _sql_constraints = [('unique_partner_id', 'UNIQUE(partner_id)', 'Only one partner IAP is allowed for one partner')]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import res_partner
from . import res_partner_iap

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
mail_plugin.access_res_partner_iap,access_res_partner_iap,mail_plugin.model_res_partner_iap,base.group_system,1,1,1,1

```

## File: static\src\to_translate\translations_gmail.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
    Terms to translate for the Gmail plugin.
    Named string format is supported (e.g: %(param)s)
    as well as standard string format (%s).
-->
<resources>
    <string>Login</string>
    <string>Logout</string>
    <string>Debug</string>
    <string>Close</string>
    <string>Refresh</string>
    <string>Create</string>
    <string>Search</string>
    <string>Search contact</string>
    <string>No contact found.</string>
    <string>Company</string>
    <string>Company Insights</string>
    <string>Description</string>
    <string>Address</string>
    <string>Phones</string>
    <string>Website</string>
    <string>Industry</string>
    <string>Employees</string>
    <string>%s employees</string>
    <string>employees</string>
    <string>Founded Year</string>
    <string>Keywords</string>
    <string>Company Type</string>
    <string>Annual Revenue</string>
    <string>Create a company</string>
    <string>Contact</string>
    <string>Email already logged on the contact</string>
    <string>This contact does not exist in the Odoo database.</string>
    <string>Can not save the contact</string>
    <string>Save in Odoo</string>
    <string>Log email</string>
    <string>Buy new credits</string>
    <string>Could not connect to database. Try to log out and in.</string>
    <string>Not enough credits to enrich.</string>
    <string>Company Created.</string>
    <string>Error during enrichment</string>
    <string>Our IAP server is down, please come back later.</string>
    <string>Oops, looks like you have exhausted your free enrichment requests. Please log in to try again.</string>
    <string>No insights found for this address.</string>
    <string>Something bad happened. Please, try again later.</string>
    <string>Invalid URL</string>
    <string>No company attached to this contact.</string>
    <string>Attachments could not be logged in Odoo because their total size exceeded the allowed maximum.</string>
    <string>Logged from</string>
    <string>Gmail Inbox</string>
    <string>From:</string>
    <string>Debug Zone</string>
    <string>Debug zone for development purpose.</string>
    <string>Odoo Server URL</string>
    <string>Odoo Access Token</string>
    <string>Clear Translations Cache</string>
    <string>Enrich Company</string>
    <string>No insights for this company.</string>
    <string>Read more</string>
</resources>

```

## File: static\src\to_translate\translations_outlook.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--Terms to translate for the outlook plugin.
    Only named string params e.g: %(param)s are supported by the plugins,
    unnamed params like %s are not supported.-->
<resources>
    <string>Login</string>
    <string>Logout</string>
    <string>Company created</string>
    <string>Company Insights</string>
    <string>Revenues</string>
    <string>Company Type</string>
    <string>Keywords</string>
    <string>Year founded</string>
    <string>Employees</string>
    <string>Industry</string>
    <string>Website</string>
    <string>Phone</string>
    <string>Address</string>
    <string>Contact created</string>
    <string>Buy More</string>
    <string>Search In Odoo</string>
    <string>Search In Database</string>
    <string>Contacts Found (%(count)s)</string>
    <string>Search contact in Odoo...</string>
    <string>Refresh Contact</string>
    <string>Add Contact To Database</string>
    <string>Contact Details</string>
    <string>Log Email Into Contact</string>
    <string>This contact has no email address, no company could be enriched.</string>
    <string>No company attached to this contact</string>
    <string>Create a Company</string>
    <string>No company linked to this contact could be enriched</string>
    <string>No company linked to this contact could be enriched or found in Odoo</string>
    <string>No data found for this email address.</string>
    <string>You don't have enough credit to enrich.</string>
    <string>Something bad happened. Please, try again later.</string>
    <string>Could not autocomplete the company. Internal error. Try again later...</string>
    <string>Could not connect to your database. Please try again.</string>
    <string>An error has occurred when trying to fetch translations.</string>
    <string>This company has no email address and could not be enriched.</string>
    <string>No extra information found</string>
    <string>No additional insights were found for this company</string>
    <string>Company updated</string>
    <string>Enrich Company</string>
    <string>Warning: Attachments could not be logged in Odoo because their total size exceeded the allowed maximum.</string>
    <string>Could not display image %(attachmentName)s, size is over limit.</string>
    <string>Warning: Could not fetch the attachments %(attachments)s as their sizes are bigger then the maximum size of %(size)sMB per each attachment.</string>
    <string>From : %(email)s</string>
    <string>Logged from</string>
    <string>Outlook Inbox</string>
    <string>Save the contact to create the company</string>
</resources>

```

## File: views\mail_plugin_login.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="mail_plugin.app_auth" name="Accept app">
        <t t-call="web.login_layout">
            <t t-set="disable_database_manager" t-value="1"/>
            <form role="form" method="post" action="/mail_plugin/auth/confirm">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <input type="hidden" name="redirect" t-att-value="redirect"/>
                <input type="hidden" name="scope" t-att-value="scope"/>
                <input type="hidden" name="state" t-att-value="state"/>
                <input type="hidden" name="friendlyname" t-att-value="friendlyname"/>
                <p class="text-center">
                    Let <t t-esc="friendlyname" /> access your Odoo database?
                </p>
                <p class="text-center">
                    <button type="submit" name="do" value="1" class="btn btn-link btn-sm">Allow</button>
                    <button type="submit" name="do" class="btn btn-link btn-sm">Deny</button>
                </p>
            </form>
        </t>
    </template>
    <template id="mail_plugin.app_error" name="Accept app">
        <t t-call="web.login_layout">
            <div class="alert alert-danger" role="alert" t-out="error"/>
        </t>
    </template>
</odoo>

```

## File: views\res_partner_iap_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_partner_iap_action" model="ir.actions.act_window">
        <field name="name">IAP Partner</field>
        <field name="res_model">res.partner.iap</field>
        <field name='view_mode'>list,form</field>
    </record>

    <record id="res_partner_iap_view_form" model="ir.ui.view">
        <field name="name">res.partner.iap.view.form</field>
        <field name="model">res.partner.iap</field>
        <field name="arch" type="xml">
            <form string="IAP Partner">
                <sheet>
                    <h1><field name="partner_id"/></h1>
                    <group>
                        <field name="iap_search_domain"/>
                        <field name="iap_enrich_info"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="res_partner_iap_view_tree" model="ir.ui.view">
        <field name="name">res.partner.iap.view.list</field>
        <field name="model">res.partner.iap</field>
        <field name="arch" type="xml">
            <list string="IAP Partner">
                <field name="partner_id"/>
                <field name="iap_search_domain"/>
            </list>
        </field>
    </record>

    <menuitem
        id="res_partner_iap_menu"
        name="IAP Partners"
        parent="iap.iap_root_menu"
        action="res_partner_iap_action"
        sequence="50"/>
</odoo>

```

