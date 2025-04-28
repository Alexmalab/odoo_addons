# Odoo Module: partner_autocomplete

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': "Partner Autocomplete",
    'summary': "Auto-complete partner companies' data",
    'version': '1.1',
    'description': """
       Auto-complete partner companies' data
    """,
    'author': "Odoo SA",
    'category': 'Hidden/Tools',
    'depends': [
        'iap_mail',
    ],
    'data': [
        'security/ir.model.access.csv',
        'views/res_partner_views.xml',
        'views/res_company_views.xml',
        'views/res_config_settings_views.xml',
        'data/cron.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'partner_autocomplete/static/src/scss/partner_autocomplete.scss',
            'partner_autocomplete/static/src/js/partner_autocomplete_core.js',
            'partner_autocomplete/static/src/js/partner_autocomplete_fieldchar.js',
            'partner_autocomplete/static/src/js/partner_autocomplete_many2one.js',
            'partner_autocomplete/static/src/js/web_company_autocomplete.js',
        ],
        'web.tests_assets': [
            'partner_autocomplete/static/lib/**/*',
        ],
        'web.qunit_suite_tests': [
            'partner_autocomplete/static/tests/**/*',
        ],
        'web.assets_qweb': [
            'partner_autocomplete/static/src/xml/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\cron.xml

```xml
<odoo>
    <record id="ir_cron_partner_autocomplete" model="ir.cron">
        <field name="name">Partner Autocomplete : Sync with remote DB</field>
        <field name="model_id" ref="model_res_partner_autocomplete_sync"/>
        <field name="state">code</field>
        <field name="code">model.start_sync()</field>
        <field name="interval_number">60</field>
        <field name="interval_type">minutes</field>
        <field name="numbercall">-1</field>
        <field eval="False" name="doall"/>
    </record>
</odoo>

```

## File: models\iap_autocomplete_api.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, models, exceptions, _
from odoo.addons.iap.tools import iap_tools
from requests.exceptions import HTTPError

_logger = logging.getLogger(__name__)


class IapAutocompleteEnrichAPI(models.AbstractModel):
    _name = 'iap.autocomplete.api'
    _description = 'IAP Partner Autocomplete API'
    _DEFAULT_ENDPOINT = 'https://partner-autocomplete.odoo.com'

    @api.model
    def _contact_iap(self, local_endpoint, action, params, timeout=15):
        if self.env.registry.in_test_mode():
            raise exceptions.ValidationError(_('Test mode'))
        account = self.env['iap.account'].get('partner_autocomplete')
        if not account.account_token:
            raise ValueError(_('No account token'))
        params.update({
            'db_uuid': self.env['ir.config_parameter'].sudo().get_param('database.uuid'),
            'account_token': account.account_token,
            'country_code': self.env.company.country_id.code,
            'zip': self.env.company.zip,
        })
        base_url = self.env['ir.config_parameter'].sudo().get_param('iap.partner_autocomplete.endpoint', self._DEFAULT_ENDPOINT)
        return iap_tools.iap_jsonrpc(base_url + local_endpoint + '/' + action, params=params, timeout=timeout)

    @api.model
    def _request_partner_autocomplete(self, action, params, timeout=15):
        """ Contact endpoint to get autocomplete data.

        :return tuple: results, error code
        """
        try:
            results = self._contact_iap('/iap/partner_autocomplete', action, params, timeout=timeout)
        except exceptions.ValidationError:
            return False, 'Insufficient Credit'
        except (ConnectionError, HTTPError, exceptions.AccessError, exceptions.UserError) as exception:
            _logger.warning('Autocomplete API error: %s', str(exception))
            return False, str(exception)
        except iap_tools.InsufficientCreditError as exception:
            _logger.warning('Insufficient Credits for Autocomplete Service: %s', str(exception))
            return False, 'Insufficient Credit'
        except ValueError:
            return False, 'No account token'
        return results, False

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.http import request


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        """ Add information about iap enrich to perform """
        session_info = super(Http, self).session_info()
        if session_info.get('is_admin'):
            session_info['iap_company_enrich'] = not request.env.user.company_id.iap_enrich_auto_done
        return session_info

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging
import threading

from odoo.addons.iap.tools import iap_tools
from odoo import api, fields, models, tools, _

_logger = logging.getLogger(__name__)

COMPANY_AC_TIMEOUT = 5


class ResCompany(models.Model):
    _name = 'res.company'
    _inherit = 'res.company'

    partner_gid = fields.Integer('Company database ID', related="partner_id.partner_gid", inverse="_inverse_partner_gid", store=True)
    iap_enrich_auto_done = fields.Boolean('Enrich Done')

    def _inverse_partner_gid(self):
        for company in self:
            company.partner_id.partner_gid = company.partner_gid

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        if not getattr(threading.current_thread(), 'testing', False):
            res.iap_enrich_auto()
        return res

    def iap_enrich_auto(self):
        """ Enrich company. This method should be called by automatic processes
        and a protection is added to avoid doing enrich in a loop. """
        if self.env.user._is_system():
            for company in self.filtered(lambda company: not company.iap_enrich_auto_done):
                company._enrich()
            self.iap_enrich_auto_done = True
        return True

    def _enrich(self):
        """ This method calls the partner autocomplete service from IAP to enrich
        partner related fields of the company.

        :return bool: either done, either failed """
        self.ensure_one()
        _logger.info("Starting enrich of company %s (%s)", self.name, self.id)

        company_domain = self._get_company_domain()
        if not company_domain:
            return False

        company_data = self.env['res.partner'].enrich_company(company_domain, False, self.vat, timeout=COMPANY_AC_TIMEOUT)
        if company_data.get('error'):
            return False
        additional_data = company_data.pop('additional_info', False)

        # Keep only truthy values that are not already set on the target partner
        # Erase image_1920 even if something is in it. Indeed as partner_autocomplete is probably installed as a
        # core app (mail -> iap -> partner_autocomplete auto install chain) it is unlikely that people already
        # updated their company logo.
        self.env['res.partner']._iap_replace_logo(company_data)
        company_data = {field: value for field, value in company_data.items()
                        if field in self.partner_id._fields and value and (field == 'image_1920' or not self.partner_id[field])}

        # for company and childs: from state_id / country_id name_get like to IDs
        company_data.update(self._enrich_extract_m2o_id(company_data, ['state_id', 'country_id']))
        if company_data.get('child_ids'):
            company_data['child_ids'] = [
                dict(child_data, **self._enrich_extract_m2o_id(child_data, ['state_id', 'country_id']))
                for child_data in company_data['child_ids']
            ]

        # handle o2m values, e.g. {'bank_ids': ['acc_number': 'BE012012012', 'acc_holder_name': 'MyWebsite']}
        self._enrich_replace_o2m_creation(company_data)

        self.partner_id.write(company_data)

        if additional_data:
            template_values = json.loads(additional_data)
            template_values['flavor_text'] = _("Company auto-completed by Odoo Partner Autocomplete Service")
            self.partner_id.message_post_with_view(
                'iap_mail.enrich_company',
                values=template_values,
                subtype_id=self.env.ref('mail.mt_note').id,
            )
        return True

    def _enrich_extract_m2o_id(self, iap_data, m2o_fields):
        """ Extract m2O ids from data (because of res.partner._format_data_company) """
        extracted_data = {}
        for m2o_field in m2o_fields:
            relation_data = iap_data.get(m2o_field)
            if relation_data and isinstance(relation_data, dict):
                extracted_data[m2o_field] = relation_data.get('id', False)
        return extracted_data

    def _enrich_replace_o2m_creation(self, iap_data):
        for o2m_field, values in iap_data.items():
            if isinstance(values, list):
                commands = [(
                    0, 0, create_value
                ) for create_value in values if isinstance(create_value, dict)]
                if commands:
                    iap_data[o2m_field] = commands
                else:
                    iap_data.pop(o2m_field, None)
        return iap_data

    def _get_company_domain(self):
        """ Extract the company domain to be used by IAP services.
        The domain is extracted from the website or the email information.
        e.g:
            - www.info.proximus.be -> proximus.be
            - info@proximus.be -> proximus.be """
        self.ensure_one()

        company_domain = tools.email_domain_extract(self.email) if self.email else False
        if company_domain and company_domain not in iap_tools._MAIL_PROVIDERS:
            return company_domain

        company_domain = tools.url_domain_extract(self.website) if self.website else False
        if not company_domain or company_domain in ['localhost', 'example.com']:
            return False

        return company_domain

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    partner_autocomplete_insufficient_credit = fields.Boolean('Insufficient credit', compute="_compute_partner_autocomplete_insufficient_credit")

    def _compute_partner_autocomplete_insufficient_credit(self):
        self.partner_autocomplete_insufficient_credit = self.env['iap.account'].get_credits('partner_autocomplete') <= 0

    def redirect_to_buy_autocomplete_credit(self):
        Account = self.env['iap.account']
        return {
            'type': 'ir.actions.act_url',
            'url': Account.get_credits_url('partner_autocomplete'),
            'target': '_new',
        }

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import json
import logging
import requests
import re

from stdnum.eu.vat import check_vies

from odoo import api, fields, models, tools, _

_logger = logging.getLogger(__name__)

PARTNER_AC_TIMEOUT = 5


class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    partner_gid = fields.Integer('Company database ID')
    additional_info = fields.Char('Additional info')

    @api.model
    def _iap_replace_location_codes(self, iap_data):
        country_code, country_name = iap_data.pop('country_code', False), iap_data.pop('country_name', False)
        state_code, state_name = iap_data.pop('state_code', False), iap_data.pop('state_name', False)

        country, state = None, None
        if country_code:
            country = self.env['res.country'].search([['code', '=ilike', country_code]])
        if not country and country_name:
            country = self.env['res.country'].search([['name', '=ilike', country_name]])

        if country:
            if state_code:
                state = self.env['res.country.state'].search([
                    ('country_id', '=', country.id), ('code', '=ilike', state_code)
                ], limit=1)
            if not state and state_name:
                state = self.env['res.country.state'].search([
                    ('country_id', '=', country.id), ('name', '=ilike', state_name)
                ], limit=1)
        else:
            _logger.info('Country code not found: %s', country_code)

        if country:
            iap_data['country_id'] = {'id': country.id, 'display_name': country.display_name}
        if state:
            iap_data['state_id'] = {'id': state.id, 'display_name': state.display_name}

        return iap_data

    @api.model
    def _iap_replace_logo(self, iap_data):
        if iap_data.get('logo'):
            try:
                iap_data['image_1920'] = base64.b64encode(
                    requests.get(iap_data['logo'], timeout=PARTNER_AC_TIMEOUT).content
                )
            except Exception:
                iap_data['image_1920'] = False
            finally:
                iap_data.pop('logo')
            # avoid keeping falsy images (may happen that a blank page is returned that leads to an incorrect image)
            if iap_data['image_1920']:
                try:
                    tools.base64_to_image(iap_data['image_1920'])
                except Exception:
                    iap_data.pop('image_1920')
        return iap_data

    @api.model
    def _format_data_company(self, iap_data):
        self._iap_replace_location_codes(iap_data)

        if iap_data.get('child_ids'):
            child_ids = []
            for child in iap_data.get('child_ids'):
                child_ids.append(self._iap_replace_location_codes(child))
            iap_data['child_ids'] = child_ids

        if iap_data.get('additional_info'):
            iap_data['additional_info'] = json.dumps(iap_data['additional_info'])

        return iap_data

    @api.model
    def autocomplete(self, query, timeout=15):
        suggestions, _ = self.env['iap.autocomplete.api']._request_partner_autocomplete('search', {
            'query': query,
        }, timeout=timeout)
        if suggestions:
            results = []
            for suggestion in suggestions:
                results.append(self._format_data_company(suggestion))
            return results
        else:
            return []

    @api.model
    def enrich_company(self, company_domain, partner_gid, vat, timeout=15):
        response, error = self.env['iap.autocomplete.api']._request_partner_autocomplete('enrich', {
            'domain': company_domain,
            'partner_gid': partner_gid,
            'vat': vat,
        }, timeout=timeout)
        if response and response.get('company_data'):
            result = self._format_data_company(response.get('company_data'))
        else:
            result = {}

        if response and response.get('credit_error'):
            result.update({
                'error': True,
                'error_message': 'Insufficient Credit'
            })
        elif error:
            result.update({
                'error': True,
                'error_message': error
            })

        return result

    @api.model
    def read_by_vat(self, vat, timeout=15):
        vies_vat_data, _ = self.env['iap.autocomplete.api']._request_partner_autocomplete('search_vat', {
            'vat': vat,
        }, timeout=timeout)
        if vies_vat_data:
            return [self._format_data_company(vies_vat_data)]
        else:
            vies_result = None
            try:
                _logger.info('Calling VIES service to check VAT for autocomplete: %s', vat)
                vies_result = check_vies(vat, timeout=timeout)
            except Exception:
                _logger.exception("Failed VIES VAT check.")
            if vies_result:
                name = vies_result['name']
                if vies_result['valid'] and name != '---':
                    address = list(filter(bool, vies_result['address'].split('\n')))
                    street = address[0]
                    zip_city_record = next(filter(lambda addr: re.match(r'^\d.*', addr), address[1:]), None)
                    zip_city = zip_city_record.split(' ', 1) if zip_city_record else [None, None]
                    street2 = next((addr for addr in filter(lambda addr: addr != zip_city_record, address[1:])), None)
                    return [self._iap_replace_location_codes({
                        'name': name,
                        'vat': vat,
                        'street': street,
                        'street2': street2,
                        'city': zip_city[1],
                        'zip': zip_city[0],
                        'country_code': vies_result['countryCode'],
                        'skip_enrich': True,
                    })]
            return []

    @api.model
    def _is_company_in_europe(self, country_code):
        country = self.env['res.country'].search([('code', '=ilike', country_code)])
        if country:
            country_id = country.id
            europe = self.env.ref('base.europe')
            if not europe:
                europe = self.env["res.country.group"].search([('name', '=', 'Europe')], limit=1)
            if not europe or country_id not in europe.country_ids.ids:
                return False
        return True

    def _is_vat_syncable(self, vat):
        vat_country_code = vat[:2]
        partner_country_code = self.country_id.code if self.country_id else ''
        return self._is_company_in_europe(vat_country_code) and (partner_country_code == vat_country_code or not partner_country_code)

    def _is_synchable(self):
        already_synched = self.env['res.partner.autocomplete.sync'].search([('partner_id', '=', self.id), ('synched', '=', True)])
        return self.is_company and self.partner_gid and not already_synched

    def _update_autocomplete_data(self, vat):
        self.ensure_one()
        if vat and self._is_synchable() and self._is_vat_syncable(vat):
            self.env['res.partner.autocomplete.sync'].sudo().add_to_queue(self.id)

    @api.model_create_multi
    def create(self, vals_list):
        partners = super(ResPartner, self).create(vals_list)
        if len(vals_list) == 1:
            partners._update_autocomplete_data(vals_list[0].get('vat', False))
            if partners.additional_info:
                template_values = json.loads(partners.additional_info)
                template_values['flavor_text'] = _("Partner created by Odoo Partner Autocomplete Service")
                partners.message_post_with_view(
                    'iap_mail.enrich_company',
                    values=template_values,
                    subtype_id=self.env.ref('mail.mt_note').id,
                )
                partners.write({'additional_info': False})

        return partners

    def write(self, values):
        res = super(ResPartner, self).write(values)
        if len(self) == 1:
            self._update_autocomplete_data(values.get('vat', False))

        return res

```

## File: models\res_partner_autocomplete_sync.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
from odoo import api, fields, models

_logger = logging.getLogger(__name__)

class ResPartnerAutocompleteSync(models.Model):
    _name = 'res.partner.autocomplete.sync'
    _description = 'Partner Autocomplete Sync'

    partner_id = fields.Many2one('res.partner', string="Partner", ondelete='cascade')
    synched = fields.Boolean('Is synched', default=False)

    @api.model
    def start_sync(self):
        to_sync_items = self.search([('synched', '=', False)])
        for to_sync_item in to_sync_items:
            partner = to_sync_item.partner_id

            params = {
                'partner_gid': partner.partner_gid,
            }

            if partner.vat and partner._is_vat_syncable(partner.vat):
                params['vat'] = partner.vat
                _, error = self.env['iap.autocomplete.api']._request_partner_autocomplete('update', params)
                if error:
                    _logger.warning('Send Partner to sync failed: %s', str(error))

            to_sync_item.write({'synched': True})

    def add_to_queue(self, partner_id):
        to_sync = self.search([('partner_id', '=', partner_id)])
        if not to_sync:
            to_sync = self.create({'partner_id': partner_id})
        return to_sync

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import iap_autocomplete_api
from . import ir_http
from . import res_partner
from . import res_company
from . import res_config_settings
from . import res_partner_autocomplete_sync

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_partner_autocomplete_sync_all_all,res.partner.autocomplete.sync.all,model_res_partner_autocomplete_sync,,1,0,0,0
access_partner_autocomplete_sync_portal,res.partner.autocomplete.sync.portal,model_res_partner_autocomplete_sync,base.group_portal,1,0,1,0
access_partner_autocomplete_sync_user,res.partner.autocomplete.sync.user,model_res_partner_autocomplete_sync,base.group_user,1,1,1,0
access_partner_autocomplete_sync_system,res.partner.autocomplete.sync.system,model_res_partner_autocomplete_sync,base.group_system,1,1,1,1
```

## File: static\description\icon.svg

```svg
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 70 70">
  <defs>
    <mask id="mask" x="-0.01" y="0" width="70.01" height="70" maskUnits="userSpaceOnUse">
      <g id="b">
        <path id="a" d="M4,0H65c4,0,5,1,5,5V65c0,4-1,5-5,5H4c-3,0-4-1-4-5V5C0,1,1,0,4,0Z" fill="#fff" fill-rule="evenodd"/>
      </g>
    </mask>
    <linearGradient id="linear-gradient" x1="-1261.07" y1="477.94" x2="-1262.07" y2="476.94" gradientTransform="matrix(70, 0, 0, -70, 88344.99, 33455.73)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#d5cc95"/>
      <stop offset="1" stop-color="#c1b66c"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M4,69a3.66,3.66,0,0,1-4-4v-45L7.07,13l31.7-1a2.56,2.56,0,0,0-.18,2l2.89-2.88,4,3.29,2.35,2.35L50.69,16l6.65-6.6L61.9,14l-6.91,7-1.44,2.87-2.12,2.1,0,3.1,2.91-2.91,6,.14,2.33,2.64-.91,27.6L49.23,69Z" fill="#494949" fill-rule="evenodd" opacity="0.32" style="isolation: isolate"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <g>
        <g opacity="0.4">
          <path d="M45.64,28.11H7.53a1.65,1.65,0,0,1-1.66-1.65V17.67A1.66,1.66,0,0,1,7.53,16h29a2.85,2.85,0,0,1,.21-2H7.53a3.67,3.67,0,0,0-3.66,3.66v8.79a3.66,3.66,0,0,0,3.66,3.65H45.64Z"/>
          <path d="M56.77,28.11H52.34v2h4.43a2.9,2.9,0,0,1,2.9,2.9V54.62a2.9,2.9,0,0,1-2.9,2.89H14.91A2.9,2.9,0,0,1,12,54.62V33a2.9,2.9,0,0,1,2.9-2.9H45.64v-2H14.91A4.91,4.91,0,0,0,10,33V54.62a4.9,4.9,0,0,0,4.9,4.89H56.77a4.9,4.9,0,0,0,4.9-4.89V33A4.91,4.91,0,0,0,56.77,28.11Z"/>
          <g>
            <path d="M12.92,19.63,11,24.2a.66.66,0,0,1-.19.27.5.5,0,0,1-.29.09.46.46,0,0,1-.36-.14.52.52,0,0,1-.11-.33,1.11,1.11,0,0,1,0-.17l2.25-5.38a.56.56,0,0,1,.22-.28.44.44,0,0,1,.32-.08.58.58,0,0,1,.3.09.55.55,0,0,1,.2.27l2.21,5.27a.63.63,0,0,1,.05.22.5.5,0,0,1-.16.39.52.52,0,0,1-.35.14.46.46,0,0,1-.29-.1.68.68,0,0,1-.2-.27l-1.94-4.51ZM11.25,23.1l.5-1h2.42l.18,1Z"/>
            <path d="M20,25.66a.58.58,0,0,1-.18.43.56.56,0,0,1-.43.17.54.54,0,0,1-.42-.17.57.57,0,0,1-.17-.43v-8a.62.62,0,0,1,.17-.44.59.59,0,0,1,.44-.18.54.54,0,0,1,.42.18.62.62,0,0,1,.17.44Z"/>
          </g>
          <g>
            <path d="M18.24,35.8,16.3,40.37a.52.52,0,0,1-.2.26.42.42,0,0,1-.28.1.46.46,0,0,1-.36-.14.48.48,0,0,1-.12-.33.55.55,0,0,1,0-.17l2.24-5.38a.57.57,0,0,1,.22-.28.47.47,0,0,1,.32-.08.53.53,0,0,1,.3.09.57.57,0,0,1,.21.27L20.87,40a.6.6,0,0,1,0,.22.49.49,0,0,1-.15.38.5.5,0,0,1-.35.15.43.43,0,0,1-.29-.1.6.6,0,0,1-.2-.27L18,35.85Zm-1.66,3.47.49-1H19.5l.17,1Z"/>
            <path d="M24.5,34.38a1.79,1.79,0,0,1,1.28.41,1.6,1.6,0,0,1,.43,1.2,1.28,1.28,0,0,1-.82,1.23,2.48,2.48,0,0,1-1,.18l0-.41a3.19,3.19,0,0,1,.64.08,2.31,2.31,0,0,1,.7.27,1.53,1.53,0,0,1,.78,1.43,2.26,2.26,0,0,1-.19,1,1.45,1.45,0,0,1-.51.58,1.94,1.94,0,0,1-.68.29,3.37,3.37,0,0,1-.69.07H22.23a.53.53,0,0,1-.55-.54V34.93a.53.53,0,0,1,.16-.39.53.53,0,0,1,.39-.16Zm-.17,1.07h-1.6l.12-.15v1.63l-.11-.08h1.62a.73.73,0,0,0,.47-.17.6.6,0,0,0,.21-.5.73.73,0,0,0-.2-.56A.72.72,0,0,0,24.33,35.45Zm.08,2.46H22.77l.08-.07v1.9l-.09-.09h1.71a.94.94,0,0,0,.66-.22.86.86,0,0,0,.24-.66.92.92,0,0,0-.16-.59.7.7,0,0,0-.39-.22A2.17,2.17,0,0,0,24.41,37.91Z"/>
            <path d="M18.24,47.8,16.3,52.37a.52.52,0,0,1-.2.26.42.42,0,0,1-.28.1.46.46,0,0,1-.36-.14.48.48,0,0,1-.12-.33.55.55,0,0,1,0-.17l2.24-5.38a.57.57,0,0,1,.22-.28.47.47,0,0,1,.32-.08.53.53,0,0,1,.3.09.57.57,0,0,1,.21.27L20.87,52a.6.6,0,0,1,0,.22.49.49,0,0,1-.15.38.5.5,0,0,1-.35.15.43.43,0,0,1-.29-.1.6.6,0,0,1-.2-.27L18,47.85Zm-1.66,3.47.49-1H19.5l.17,1Z"/>
            <path d="M24.5,46.38a1.79,1.79,0,0,1,1.28.41,1.6,1.6,0,0,1,.43,1.2,1.28,1.28,0,0,1-.82,1.23,2.48,2.48,0,0,1-1,.18l0-.41a3.19,3.19,0,0,1,.64.08,2.31,2.31,0,0,1,.7.27,1.53,1.53,0,0,1,.78,1.43,2.26,2.26,0,0,1-.19,1,1.45,1.45,0,0,1-.51.58,1.94,1.94,0,0,1-.68.29,3.37,3.37,0,0,1-.69.07H22.23a.53.53,0,0,1-.55-.54V46.93a.53.53,0,0,1,.16-.39.53.53,0,0,1,.39-.16Zm-.17,1.07h-1.6l.12-.15v1.63l-.11-.08h1.62a.73.73,0,0,0,.47-.17.6.6,0,0,0,.21-.5.73.73,0,0,0-.2-.56A.72.72,0,0,0,24.33,47.45Zm.08,2.46H22.77l.08-.07v1.9l-.09-.09h1.71a.94.94,0,0,0,.66-.22.86.86,0,0,0,.24-.66.92.92,0,0,0-.16-.59.7.7,0,0,0-.39-.22A2.17,2.17,0,0,0,24.41,49.91Z"/>
            <path d="M32,46.69a.48.48,0,0,1,.28.37.56.56,0,0,1-.13.46.4.4,0,0,1-.3.18.74.74,0,0,1-.38-.07,2.18,2.18,0,0,0-.45-.16,2.73,2.73,0,0,0-.5,0,2.33,2.33,0,0,0-.87.15,1.85,1.85,0,0,0-1.06,1.11,2.33,2.33,0,0,0-.14.85,2.54,2.54,0,0,0,.16,1,1.84,1.84,0,0,0,.43.67,1.72,1.72,0,0,0,.66.4,2.39,2.39,0,0,0,.82.13,2.59,2.59,0,0,0,.48,0,1.54,1.54,0,0,0,.47-.16.74.74,0,0,1,.38-.07.49.49,0,0,1,.31.19.56.56,0,0,1,.13.47.48.48,0,0,1-.28.34,2.55,2.55,0,0,1-.48.2,2.78,2.78,0,0,1-.5.11,2.92,2.92,0,0,1-.51,0,3.87,3.87,0,0,1-1.23-.2,2.93,2.93,0,0,1-1-.62,2.59,2.59,0,0,1-.72-1,3.58,3.58,0,0,1-.26-1.41,3.3,3.3,0,0,1,.24-1.27,3.07,3.07,0,0,1,.67-1,3.25,3.25,0,0,1,1-.66,3.46,3.46,0,0,1,1.3-.24,3.27,3.27,0,0,1,1.48.35Z"/>
          </g>
          <path d="M40.79,21.86h3.15A4.34,4.34,0,0,1,44.63,20l-2.54-2.53a2.81,2.81,0,0,1-.82.13,2.6,2.6,0,1,1,2.18-1.17L45.8,18.8A4.56,4.56,0,0,1,48.47,18a4.61,4.61,0,0,1,3.1,1.2L55,15.69a3.27,3.27,0,1,1,2.57,1.24,3.31,3.31,0,0,1-1.54-.38l-3.47,4a4.49,4.49,0,0,1,.45,2,4.54,4.54,0,0,1-3.58,4.42v4.63a3.49,3.49,0,1,1-1.2,0V27a4.55,4.55,0,0,1-4.33-3.94H40.76a3.46,3.46,0,1,1,0-1.2Zm7.7,4.2a3.6,3.6,0,1,0-3.64-3.6A3.61,3.61,0,0,0,48.49,26.06Z" fill-rule="evenodd"/>
        </g>
        <g>
          <path d="M47.66,26.1H9.54a1.66,1.66,0,0,1-1.66-1.66V15.66A1.66,1.66,0,0,1,9.54,14h29v0a2.77,2.77,0,0,1,.2-2H9.54a3.66,3.66,0,0,0-3.66,3.66v8.78A3.66,3.66,0,0,0,9.54,28.1H47.66Z" fill="#fff"/>
          <path d="M58.78,26.1H54.36v2h4.42a2.9,2.9,0,0,1,2.9,2.9V52.6a2.9,2.9,0,0,1-2.9,2.9H16.92A2.9,2.9,0,0,1,14,52.6V31a2.9,2.9,0,0,1,2.9-2.9H47.66v-2H16.92A4.91,4.91,0,0,0,12,31V52.6a4.91,4.91,0,0,0,4.9,4.9H58.78a4.91,4.91,0,0,0,4.9-4.9V31A4.91,4.91,0,0,0,58.78,26.1Z" fill="#fff"/>
          <g>
            <path d="M14.93,17.61,13,22.19a.59.59,0,0,1-.2.26.47.47,0,0,1-.28.09.45.45,0,0,1-.36-.13.48.48,0,0,1-.12-.33.51.51,0,0,1,0-.17l2.24-5.39a.62.62,0,0,1,.22-.28.53.53,0,0,1,.32-.08.54.54,0,0,1,.3.1.52.52,0,0,1,.2.26l2.22,5.28a.59.59,0,0,1,0,.21.5.5,0,0,1-.15.39.51.51,0,0,1-.35.14.48.48,0,0,1-.29-.09.71.71,0,0,1-.21-.27l-1.93-4.51Zm-1.67,3.48.5-1h2.43l.17,1Z" fill="#fff"/>
            <path d="M22.05,23.64a.64.64,0,0,1-.17.44.62.62,0,0,1-.44.17.59.59,0,0,1-.42-.17.64.64,0,0,1-.17-.44v-8a.6.6,0,0,1,.18-.44.63.63,0,0,1,.86,0,.63.63,0,0,1,.16.44Z" fill="#fff"/>
          </g>
          <g>
            <path d="M20.26,33.78l-2,4.57a.49.49,0,0,1-.47.36.46.46,0,0,1-.37-.13.52.52,0,0,1-.11-.33,1,1,0,0,1,0-.17l2.25-5.39a.56.56,0,0,1,.22-.28.51.51,0,0,1,.32-.08.6.6,0,0,1,.3.1.52.52,0,0,1,.2.26L22.88,38a.63.63,0,0,1,.05.21.48.48,0,0,1-.16.39.47.47,0,0,1-.35.14.52.52,0,0,1-.29-.09.79.79,0,0,1-.2-.27L20,33.84Zm-1.67,3.48.5-1h2.43l.17,1Z" fill="#fff"/>
            <path d="M26.51,32.37a1.8,1.8,0,0,1,1.29.4,1.59,1.59,0,0,1,.42,1.2,1.31,1.31,0,0,1-.21.74,1.28,1.28,0,0,1-.61.5,2.46,2.46,0,0,1-1,.17l0-.4a2.66,2.66,0,0,1,.64.08,2.41,2.41,0,0,1,.7.27,1.5,1.5,0,0,1,.55.54,1.57,1.57,0,0,1,.22.88,2.09,2.09,0,0,1-.19,1,1.48,1.48,0,0,1-.51.59,1.88,1.88,0,0,1-.67.28,3.42,3.42,0,0,1-.7.08H24.24a.53.53,0,0,1-.39-.16.53.53,0,0,1-.16-.39v-5.2a.53.53,0,0,1,.16-.39.53.53,0,0,1,.39-.16Zm-.16,1.06h-1.6l.11-.14v1.63l-.1-.08h1.62a.67.67,0,0,0,.46-.18.58.58,0,0,0,.21-.49.7.7,0,0,0-.19-.56A.72.72,0,0,0,26.35,33.43Zm.07,2.47H24.78l.08-.07v1.89l-.09-.09h1.71a.92.92,0,0,0,.66-.22,1.07,1.07,0,0,0,.08-1.24.73.73,0,0,0-.38-.23A2.19,2.19,0,0,0,26.42,35.9Z" fill="#fff"/>
            <path d="M20.26,45.78l-2,4.57a.49.49,0,0,1-.47.36.46.46,0,0,1-.37-.13.52.52,0,0,1-.11-.33,1,1,0,0,1,0-.17l2.25-5.39a.56.56,0,0,1,.22-.28.51.51,0,0,1,.32-.08.6.6,0,0,1,.3.1.52.52,0,0,1,.2.26L22.88,50a.63.63,0,0,1,.05.21.48.48,0,0,1-.16.39.47.47,0,0,1-.35.14.52.52,0,0,1-.29-.09.79.79,0,0,1-.2-.27L20,45.84Zm-1.67,3.48.5-1h2.43l.17,1Z" fill="#fff"/>
            <path d="M26.51,44.37a1.8,1.8,0,0,1,1.29.4,1.59,1.59,0,0,1,.42,1.2,1.31,1.31,0,0,1-.21.74,1.28,1.28,0,0,1-.61.5,2.46,2.46,0,0,1-1,.17l0-.4a2.66,2.66,0,0,1,.64.08,2.41,2.41,0,0,1,.7.27,1.5,1.5,0,0,1,.55.54,1.57,1.57,0,0,1,.22.88,2.09,2.09,0,0,1-.19,1,1.48,1.48,0,0,1-.51.59,1.88,1.88,0,0,1-.67.28,3.42,3.42,0,0,1-.7.08H24.24a.53.53,0,0,1-.39-.16.53.53,0,0,1-.16-.39v-5.2a.53.53,0,0,1,.16-.39.53.53,0,0,1,.39-.16Zm-.16,1.06h-1.6l.11-.14v1.63l-.1-.08h1.62a.67.67,0,0,0,.46-.18.58.58,0,0,0,.21-.49.7.7,0,0,0-.19-.56A.72.72,0,0,0,26.35,45.43Zm.07,2.47H24.78l.08-.07v1.89l-.09-.09h1.71a.92.92,0,0,0,.66-.22,1.07,1.07,0,0,0,.08-1.24.73.73,0,0,0-.38-.23A2.19,2.19,0,0,0,26.42,47.9Z" fill="#fff"/>
            <path d="M34,44.68a.46.46,0,0,1,.27.36.52.52,0,0,1-.13.46.4.4,0,0,1-.3.18.62.62,0,0,1-.37-.07,2.47,2.47,0,0,0-.46-.15,2,2,0,0,0-.5-.06,2.41,2.41,0,0,0-.87.16A1.94,1.94,0,0,0,31,46a2.08,2.08,0,0,0-.41.67,2.63,2.63,0,0,0-.14.86,2.7,2.7,0,0,0,.16.95,1.9,1.9,0,0,0,.44.68,1.73,1.73,0,0,0,.65.4,2.47,2.47,0,0,0,.82.13,2.72,2.72,0,0,0,.49,0,1.63,1.63,0,0,0,.47-.16.61.61,0,0,1,.37-.06.47.47,0,0,1,.31.19.54.54,0,0,1,.13.46.46.46,0,0,1-.27.35,3.19,3.19,0,0,1-.48.19l-.5.12a3,3,0,0,1-.52,0,3.61,3.61,0,0,1-1.23-.21,3.16,3.16,0,0,1-1-.61,3,3,0,0,1-.72-1,3.65,3.65,0,0,1-.26-1.41,3.35,3.35,0,0,1,.24-1.28,3,3,0,0,1,.68-1,3,3,0,0,1,1-.67,3.4,3.4,0,0,1,1.29-.24,3.33,3.33,0,0,1,.78.09A2.89,2.89,0,0,1,34,44.68Z" fill="#fff"/>
          </g>
          <path d="M42.81,19.84H46A4.48,4.48,0,0,1,46.64,18l-2.53-2.54A2.58,2.58,0,1,1,45.9,13a2.52,2.52,0,0,1-.43,1.41l2.35,2.35a4.62,4.62,0,0,1,5.76.36l3.47-3.46a3.18,3.18,0,0,1-.7-2,3.26,3.26,0,1,1,1.73,2.85l-3.48,4a4.45,4.45,0,0,1,.46,2,4.54,4.54,0,0,1-3.58,4.42v4.64a3.49,3.49,0,1,1-1.2,0V25A4.56,4.56,0,0,1,46,21H42.78a3.46,3.46,0,1,1,0-1.2ZM50.5,24a3.6,3.6,0,1,0-3.63-3.6A3.62,3.62,0,0,0,50.5,24Z" fill="#fff" fill-rule="evenodd"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\lib\jsvat.js

```javascript
/*==================================================================

Application:   Utility Function
Author:        John Gardner
Website:       http://www.braemoor.co.uk/software/vat.shtml

Version:       V1.0
Date:          30th July 2005
Description:   Used to check the validity of an EU country VAT number

Version:       V1.1
Date:          3rd August 2005
Description:   Lithuanian legal entities & Maltese check digit checks added.

Version:       V1.2
Date:          20th October 2005
Description:   Italian checks refined (thanks Matteo Mike Peluso).

Version:       V1.3
Date:          16th November 2005
Description:   Error in GB numbers ending in 00 fixed (thanks Guy Dawson).

Version:       V1.4
Date:          28th September 2006
Description:   EU-type numbers added.

Version:       V1.5
Date:          1st January 2007
Description:   Romanian and Bulgarian numbers added.

Version:       V1.6
Date:          7th January 2007
Description:   Error with Slovenian numbers (thanks to Ales Hotko).

Version:       V1.7
Date:          10th February 2007
Description:   Romanian check digits added.
               Thanks to Dragu Costel for the test suite.

Version:       V1.8
Date:          3rd August 2007
Description:   IE code modified to allow + and * in old format numbers.
               Thanks to Antonin Moy of Sphere Solutions for pointing out the error.

Version:       V1.9
Date:          6th August 2007
Description:   BE code modified to make a specific check that the leading character of 10 digit
               numbers is 0 (belts and braces).

Version:       V1.10
Date:          10th August 2007
Description:   Cypriot check digit support added.
               Check digit validation support for non-standard UK numbers

Version:       V1.11
Date:          25th September 2007
Description:   Spain check digit support for personal numbers.
               Author: David Perez Carmona

Version:       V1.12
Date:          23rd November 2009
Description:   GB code modified to take into account new style check digits.
               Thanks to Guy Dawson of Crossflight Ltd for pointing out the necessity.

Version:       V1.13
Date:          7th July 2012
Description:   EL, GB, SE and BE formats updated - thanks to Joost Van Biervliet of VAT Applications

Version:       V.14
Date:          8th April 2013
Description:   BE Pattern match refined
               BG Add check digit checks for all four types of VAT number
               CY Pattern match improved
               CZ Personal pattern match checking improved
               CZ Personal check digits incorporated
               EE improved pattern match
               ES Physical person number checking refined
               GB Check digit support provided for 12 digit VAT codes and range checks included
               IT Bug removed to allow 999 and 888 issuing office codes
               LT temporarily registered taxpayers check digit support added
               LV Natural persons checks added
               RO improved pattern match
               SK improved pattern match and added check digit support

               Thanks to Theo Vroom for his help in this latest release.

Version:      V1.15
Date:         15th April 2013
              Swedish algorithm re-implemented.

Version:      V1.16
Date:         25th July 2013
              Support for Croatian numbers added

Version       V1.17
              10th September 2013
              Support for Norwegian MVA numbers added (yes, I know that Norway is not in the EU!)

Version       V1.18
              29th October 2013
              Partial support for new style Irish numbers.
              See http://www.revenue.ie/en/practitioner/ebrief/2013/no-032013.html
              Thanks to Simon Leigh for drawing the author's attention to this.

Version       V1.19
              31st October 2013
              Support for Serbian PBI numbers added (yes, I know that Serbia is not in the EU!)

Version       V1.20
              1st November 2013
              Support for Swiss MWST numbers added (yes, I know that Switzerland is not in the EU!)

Version       V1.21
              16th December 2014
              Non-critical code tidies to French and Danish regular expressions.
              Thanks to Bill Seddon of Lyquidity Solutions

Version       V1.22
              14th January 2014
              Non-critical code tidy to regular expression for new format Irish numbers.
              Thanks to Olivier Reubens of UNIT4 C-Logic N.V.

Version       V1.23
              10th April 2014
              Support for Russian INN numbers added (yes, I know that Russia is not in the EU!).
              Thanks to Marco Cesaratto of Arki Tech, Italy

Version       V1.24
              4th June 2014
              Check digit validation supported for Irish Type 3 numbers
              Thanks to Olivier Reubens of UNIT4 C-Logic N.V.

Version       V1.25
              29th July 2014
              Code improvements
              Thanks to Sébastien Boelpaep and Nate Kerkhofs

Version       V1.26
              4th May 2015
              Code improvements to regular expressions
              Thanks to Robert Gust-Bardon of webcraft.ch

Version       V1.27
              3rd December 2015
              Extend Swiss optional suffix to allow TVA and ITA
              Thanks to Oskars Petermanis

Version       V1.28
              30th August 2016
              Correct Swiss optional suffix to allow TVA and IVA
              Thanks to Jan Verhaegen

Version       V1.29
              29th July 2017
              Correct Czeck Republic checking of Individual type 2 - Special Cases
              Thanks to Andreas Wuermser of Auer Packaging UK

Parameters:    toCheck - VAT number be checked.

This function checks the value of the parameter for a valid European VAT number.

If the number is found to be invalid format, the function returns a value of false. Otherwise it
returns the VAT number re-formatted.

Example call:

  if (checkVATNumber (myVATNumber))
      alert ("VAT number has a valid format")
  else
      alert ("VAT number has invalid format");

---------------------------------------------------------------------------------------------------*/

var checkVATNumber = (function (){
    // Array holds the regular expressions for the valid VAT number
    var vatexp = new Array();

    // Note - VAT codes without the "**" in the comment do not have check digit checking.

    vatexp.push(/^(AT)U(\d{8})$/);                           //** Austria
    vatexp.push(/^(BE)(\d{9,10})$/);                         //** Belgium
    vatexp.push(/^(BG)(\d{9,10})$/);                         //** Bulgaria
    vatexp.push(/^(CHE)(\d{9})(MWST|TVA|IVA)?$/);            //** Switzerland
    vatexp.push(/^(CY)([0-59]\d{7}[A-Z])$/);                 //** Cyprus
    vatexp.push(/^(CZ)(\d{8,10})(\d{3})?$/);                 //** Czech Republic
    vatexp.push(/^(DE)([1-9]\d{8})$/);                       //** Germany
    vatexp.push(/^(DK)(\d{8})$/);                            //** Denmark
    vatexp.push(/^(EE)(10\d{7})$/);                          //** Estonia
    vatexp.push(/^(EL)(\d{9})$/);                            //** Greece
    vatexp.push(/^(ES)([A-Z]\d{8})$/);                       //** Spain (National juridical entities)
    vatexp.push(/^(ES)([A-HN-SW]\d{7}[A-J])$/);              //** Spain (Other juridical entities)
    vatexp.push(/^(ES)([0-9YZ]\d{7}[A-Z])$/);                //** Spain (Personal entities type 1)
    vatexp.push(/^(ES)([KLMX]\d{7}[A-Z])$/);                 //** Spain (Personal entities type 2)
    vatexp.push(/^(EU)(\d{9})$/);                            //** EU-type
    vatexp.push(/^(FI)(\d{8})$/);                            //** Finland
    vatexp.push(/^(FR)(\d{11})$/);                           //** France (1)
    vatexp.push(/^(FR)([A-HJ-NP-Z]\d{10})$/);                // France (2)
    vatexp.push(/^(FR)(\d[A-HJ-NP-Z]\d{9})$/);               // France (3)
    vatexp.push(/^(FR)([A-HJ-NP-Z]{2}\d{9})$/);              // France (4)
    vatexp.push(/^(GB|XI)(\d{9})$/);                         //** UK & Northern Ireland (Standard)
    vatexp.push(/^(GB|XI)(\d{12})$/);                        //** UK & Northern Ireland (Branches)
    vatexp.push(/^(GB)(GD\d{3})$/);                          //** UK (Government)
    vatexp.push(/^(GB)(HA\d{3})$/);                          //** UK (Health authority)
    vatexp.push(/^(HR)(\d{11})$/);                           //** Croatia
    vatexp.push(/^(HU)(\d{8})$/);                            //** Hungary
    vatexp.push(/^(IE)(\d{7}[A-W])$/);                       //** Ireland (1)
    vatexp.push(/^(IE)([7-9][A-Z\*\+)]\d{5}[A-W])$/);        //** Ireland (2)
    vatexp.push(/^(IE)(\d{7}[A-W][AH])$/);                   //** Ireland (3)
    vatexp.push(/^(IT)(\d{11})$/);                           //** Italy
    vatexp.push(/^(LV)(\d{11})$/);                           //** Latvia
    vatexp.push(/^(LT)(\d{9}|\d{12})$/);                     //** Lithunia
    vatexp.push(/^(LU)(\d{8})$/);                            //** Luxembourg
    vatexp.push(/^(MT)([1-9]\d{7})$/);                       //** Malta
    vatexp.push(/^(NL)(\d{9})B\d{2}$/);                      //** Netherlands
    vatexp.push(/^(NO)(\d{9})$/);                            //** Norway (not EU)
    vatexp.push(/^(PL)(\d{10})$/);                           //** Poland
    vatexp.push(/^(PT)(\d{9})$/);                            //** Portugal
    vatexp.push(/^(RO)([1-9]\d{1,9})$/);                     //** Romania
    vatexp.push(/^(RU)(\d{10}|\d{12})$/);                    //** Russia
    vatexp.push(/^(RS)(\d{9})$/);                            //** Serbia
    vatexp.push(/^(SI)([1-9]\d{7})$/);                       //** Slovenia
    vatexp.push(/^(SK)([1-9]\d[2346-9]\d{7})$/);             //** Slovakia Republic
    vatexp.push(/^(SE)(\d{10}01)$/);                         //** Sweden

    function checkVATNumber(toCheck) {
        // Load up the string to check
        var VATNumber = toCheck.toUpperCase();

        // Remove spaces etc. from the VAT number to help validation
        VATNumber = VATNumber.replace(/(\s|-|\.)+/g, '');

        // Assume we're not going to find a valid VAT number
        var valid = false;

        // Check the string against the regular expressions for all types of VAT numbers
        for (var i = 0; i < vatexp.length; i++) {

            // Have we recognised the VAT number?
            if (vatexp[i].test(VATNumber)) {

                // Yes - we have
                var cCode = RegExp.$1;                             // Isolate country code
                var cNumber = RegExp.$2;                           // Isolate the number

                // Call the appropriate country VAT validation routine depending on the country code
                if (eval(cCode + "VATCheckDigit ('" + cNumber + "')")) valid = VATNumber;

                // Having processed the number, we break from the loop
                break;
            }
        }

        // Return with either an error or the reformatted VAT number
        return valid;
    }

    function ATVATCheckDigit(vatnumber) {

        // Checks the check digits of an Austrian VAT number.

        var total = 0;
        var multipliers = [1, 2, 1, 2, 1, 2, 1];
        var temp = 0;

        // Extract the next digit and multiply by the appropriate multiplier.
        for (var i = 0; i < 7; i++) {
            temp = Number(vatnumber.charAt(i)) * multipliers[i];
            if (temp > 9)
                total += Math.floor(temp / 10) + temp % 10;
            else
                total += temp;
        }

        // Establish check digit.
        total = 10 - (total + 4) % 10;
        if (total == 10) total = 0;

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(7, 8))
            return true;
        else
            return false;
    }

    function BEVATCheckDigit(vatnumber) {

        // Checks the check digits of a Belgium VAT number.

        // Nine digit numbers have a 0 inserted at the front.
        if (vatnumber.length == 9) vatnumber = "0" + vatnumber;

        // Modulus 97 check on last nine digits
        if (97 - vatnumber.slice(0, 8) % 97 == vatnumber.slice(8, 10))
            return true;
        else
            return false;
    }

    function BGVATCheckDigit(vatnumber) {
        var temp, total, multipliers, i;

        // Checks the check digits of a Bulgarian VAT number.

        if (vatnumber.length == 9) {
            // Check the check digit of 9 digit Bulgarian VAT numbers.
            total = 0;

            // First try to calculate the check digit using the first multipliers
            temp = 0;
            for (i = 0; i < 8; i++) temp += Number(vatnumber.charAt(i)) * (i + 1);

            // See if we have a check digit yet
            total = temp % 11;
            if (total != 10) {
                if (total == vatnumber.slice(8))
                    return true;
                else
                    return false;
            }

            // We got a modulus of 10 before so we have to keep going. Calculate the new check digit using
            // the different multipliers
            temp = 0;
            for (i = 0; i < 8; i++) temp += Number(vatnumber.charAt(i)) * (i + 3);

            // See if we have a check digit yet. If we still have a modulus of 10, set it to 0.
            total = temp % 11;
            if (total == 10) total = 0;
            if (total == vatnumber.slice(8))
                return true;
            else
                return false;
        }

        // 10 digit VAT code - see if it relates to a standard physical person
        if ((/^\d\d[0-5]\d[0-3]\d\d{4}$/).test(vatnumber)) {

            // Check month
            var month = Number(vatnumber.slice(2, 4));
            if ((month > 0 && month < 13) || (month > 20 && month < 33) || (month > 40 && month < 53)) {

                // Extract the next digit and multiply by the counter.
                multipliers = [2, 4, 8, 5, 10, 9, 7, 3, 6];
                total = 0;
                for (i = 0; i < 9; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

                // Establish check digit.
                total = total % 11;
                if (total == 10) total = 0;

                // Check to see if the check digit given is correct, If not, try next type of person
                if (total == vatnumber.substr(9, 1)) return true;
            }
        }

        // It doesn't relate to a standard physical person - see if it relates to a foreigner.

        // Extract the next digit and multiply by the counter.
        multipliers = [21, 19, 17, 13, 11, 9, 7, 3, 1];
        total = 0;
        for (i = 0; i < 9; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Check to see if the check digit given is correct, If not, try next type of person
        if (total % 10 == vatnumber.substr(9, 1)) return true;

        // Finally, if not yet identified, see if it conforms to a miscellaneous VAT number

        // Extract the next digit and multiply by the counter.
        multipliers = [4, 3, 2, 7, 6, 5, 4, 3, 2];
        total = 0;
        for (i = 0; i < 9; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digit.
        total = 11 - total % 11;
        if (total == 10) return false;
        if (total == 11) total = 0;

        // Check to see if the check digit given is correct, If not, we have an error with the VAT number
        if (total == vatnumber.substr(9, 1))
            return true;
        else
            return false;
    }

    function CHEVATCheckDigit(vatnumber) {

        // Checks the check digits of a Swiss VAT number.

        // Extract the next digit and multiply by the counter.
        var multipliers = [5, 4, 3, 2, 7, 6, 5, 4];
        var total = 0;
        for (var i = 0; i < 8; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digit.
        total = 11 - total % 11;
        if (total == 10) return false;
        if (total == 11) total = 0;

        // Check to see if the check digit given is correct, If not, we have an error with the VAT number
        if (total == vatnumber.substr(8, 1))
            return true;
        else
            return false;
    }

    function CYVATCheckDigit(vatnumber) {

        // Checks the check digits of a Cypriot VAT number.

        // Not allowed to start with '12'
        if (Number(vatnumber.slice(0, 2) == 12)) return false;

        // Extract the next digit and multiply by the counter.
        var total = 0;
        for (var i = 0; i < 8; i++) {
            var temp = Number(vatnumber.charAt(i));
            if (i % 2 == 0) {
                switch (temp) {
                    case 0:
                        temp = 1;
                        break;
                    case 1:
                        temp = 0;
                        break;
                    case 2:
                        temp = 5;
                        break;
                    case 3:
                        temp = 7;
                        break;
                    case 4:
                        temp = 9;
                        break;
                    default:
                        temp = temp * 2 + 3;
                }
            }
            total += temp;
        }

        // Establish check digit using modulus 26, and translate to char. equivalent.
        total = total % 26;
        total = String.fromCharCode(total + 65);

        // Check to see if the check digit given is correct
        if (total == vatnumber.substr(8, 1))
            return true;
        else
            return false;
    }

    function CZVATCheckDigit(vatnumber) {

        // Checks the check digits of a Czech Republic VAT number.

        var total = 0;
        var multipliers = [8, 7, 6, 5, 4, 3, 2];

        var czexp = new Array();
        czexp[0] = (/^\d{8}$/);                                       //  8 digit legal entities
        // Note - my specification says that that the following should have a range of 0-3 in the fourth
        // digit, but the valid number CZ395601439 did not confrm, so a range of 0-9 has been allowed.
        czexp[1] = (/^[0-5][0-9][0|1|5|6][0-9][0-3][0-9]\d{3}$/);     //  9 digit individuals
        czexp[2] = (/^6\d{8}$/);                                      //  9 digit individuals (Special cases)
        czexp[3] = (/^\d{2}[0-3|5-8][0-9][0-3][0-9]\d{4}$/);          // 10 digit individuals
        var i = 0;
        var a;

        // Legal entities
        if (czexp[0].test(vatnumber)) {

            // Extract the next digit and multiply by the counter.
            for (i = 0; i < 7; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

            // Establish check digit.
            total = 11 - total % 11;
            if (total == 10) total = 0;
            if (total == 11) total = 1;

            // Compare it with the last character of the VAT number. If it's the same, then it's valid.
            if (total == vatnumber.slice(7, 8))
                return true;
            else
                return false;
        }

        // Individuals type 1 (Standard) - 9 digits without check digit
        else if (czexp[1].test(vatnumber)) {
            if (Number(vatnumber.slice(0, 2)) > 62) return false;
            return true;
        }

        // Individuals type 2 (Special Cases) - 9 digits including check digit
        else if (czexp[2].test(vatnumber)) {

            // Extract the next digit and multiply by the counter.
            for (i = 0; i < 7; i++) total += Number(vatnumber.charAt(i + 1)) * multipliers[i];

            // Establish check digit pointer into lookup table
            if (total % 11 == 0)
                a = total + 11;
            else
                a = Math.ceil(total / 11) * 11;
            var pointer = a - total;

            // Convert calculated check digit according to a lookup table;
            var lookup = [8, 7, 6, 5, 4, 3, 2, 1, 0, 9, 8];
            if (lookup[pointer - 1] == vatnumber.slice(8, 9))
                return true;
            else
                return false;
        }

        // Individuals type 3 - 10 digits
        else if (czexp[3].test(vatnumber)) {
            var temp = Number(vatnumber.slice(0, 2)) + Number(vatnumber.slice(2, 4)) + Number(vatnumber.slice(4, 6)) + Number(vatnumber.slice(6, 8)) + Number(vatnumber.slice(8));
            if (temp % 11 == 0 && Number(vatnumber) % 11 == 0)
                return true;
            else
                return false;
        }

        // else error
        return false;
    }

    function DEVATCheckDigit(vatnumber) {

        // Checks the check digits of a German VAT number.

        var product = 10;
        var sum = 0;
        var checkdigit = 0;
        for (var i = 0; i < 8; i++) {

            // Extract the next digit and implement peculiar algorithm!.
            sum = (Number(vatnumber.charAt(i)) + product) % 10;
            if (sum == 0) {
                sum = 10;
            }
            product = (2 * sum) % 11;
        }

        // Establish check digit.
        if (11 - product == 10) {
            checkdigit = 0;
        } else {
            checkdigit = 11 - product;
        }

        // Compare it with the last two characters of the VAT number. If the same, then it is a valid
        // check digit.
        if (checkdigit == vatnumber.slice(8, 9))
            return true;
        else
            return false;
    }

    function DKVATCheckDigit(vatnumber) {

        // Checks the check digits of a Danish VAT number.

        var total = 0;
        var multipliers = [2, 7, 6, 5, 4, 3, 2, 1];

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 8; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digit.
        total = total % 11;

        // The remainder should be 0 for it to be valid..
        if (total == 0)
            return true;
        else
            return false;
    }

    function EEVATCheckDigit(vatnumber) {

        // Checks the check digits of an Estonian VAT number.

        var total = 0;
        var multipliers = [3, 7, 1, 3, 7, 1, 3, 7];

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 8; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digits using modulus 10.
        total = 10 - total % 10;
        if (total == 10) total = 0;

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(8, 9))
            return true;
        else
            return false;
    }

    function ELVATCheckDigit(vatnumber) {

        // Checks the check digits of a Greek VAT number.

        var total = 0;
        var multipliers = [256, 128, 64, 32, 16, 8, 4, 2];

        //eight character numbers should be prefixed with an 0.
        if (vatnumber.length == 8) {
            vatnumber = "0" + vatnumber;
        }

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 8; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digit.
        total = total % 11;
        if (total > 9) {
            total = 0;
        }

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(8, 9))
            return true;
        else
            return false;
    }

    function ESVATCheckDigit(vatnumber) {

        // Checks the check digits of a Spanish VAT number.

        var total = 0;
        var temp = 0;
        var multipliers = [2, 1, 2, 1, 2, 1, 2];
        var esexp = new Array();
        esexp[0] = (/^[A-H|J|U|V]\d{8}$/);
        esexp[1] = (/^[A-H|N-S|W]\d{7}[A-J]$/);
        esexp[2] = (/^[0-9|Y|Z]\d{7}[A-Z]$/);
        esexp[3] = (/^[K|L|M|X]\d{7}[A-Z]$/);
        var i = 0;

        // National juridical entities
        if (esexp[0].test(vatnumber)) {

            // Extract the next digit and multiply by the counter.
            for (i = 0; i < 7; i++) {
                temp = Number(vatnumber.charAt(i + 1)) * multipliers[i];
                if (temp > 9)
                    total += Math.floor(temp / 10) + temp % 10;
                else
                    total += temp;
            }
            // Now calculate the check digit itself.
            total = 10 - total % 10;
            if (total == 10) {
                total = 0;
            }

            // Compare it with the last character of the VAT number. If it's the same, then it's valid.
            if (total == vatnumber.slice(8, 9))
                return true;
            else
                return false;
        }

        // Juridical entities other than national ones
        else if (esexp[1].test(vatnumber)) {

            // Extract the next digit and multiply by the counter.
            for (i = 0; i < 7; i++) {
                temp = Number(vatnumber.charAt(i + 1)) * multipliers[i];
                if (temp > 9)
                    total += Math.floor(temp / 10) + temp % 10;
                else
                    total += temp;
            }

            // Now calculate the check digit itself.
            total = 10 - total % 10;
            total = String.fromCharCode(total + 64);

            // Compare it with the last character of the VAT number. If it's the same, then it's valid.
            if (total == vatnumber.slice(8, 9))
                return true;
            else
                return false;
        }

        // Personal number (NIF) (starting with numeric of Y or Z)
        else if (esexp[2].test(vatnumber)) {
            var tempnumber = vatnumber;
            if (tempnumber.substring(0, 1) == 'Y') tempnumber = tempnumber.replace(/Y/, "1");
            if (tempnumber.substring(0, 1) == 'Z') tempnumber = tempnumber.replace(/Z/, "2");
            return tempnumber.charAt(8) == 'TRWAGMYFPDXBNJZSQVHLCKE'.charAt(Number(tempnumber.substring(0, 8)) % 23);
        }

        // Personal number (NIF) (starting with K, L, M, or X)
        else if (esexp[3].test(vatnumber)) {
            return vatnumber.charAt(8) == 'TRWAGMYFPDXBNJZSQVHLCKE'.charAt(Number(vatnumber.substring(1, 8)) % 23);
        }

        else return false;
    }

    function EUVATCheckDigit(vatnumber) {

        // We know little about EU numbers apart from the fact that the first 3 digits represent the
        // country, and that there are nine digits in total.
        return true;
    }

    function FIVATCheckDigit(vatnumber) {

        // Checks the check digits of a Finnish VAT number.

        var total = 0;
        var multipliers = [7, 9, 10, 5, 8, 4, 2];

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 7; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digit.
        total = 11 - total % 11;
        if (total > 9) {
            total = 0;
        }

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(7, 8))
            return true;
        else
            return false;
    }

    function FRVATCheckDigit(vatnumber) {

        // Checks the check digits of a French VAT number.

        if (!(/^\d{11}$/).test(vatnumber)) return true;

        // Extract the last nine digits as an integer.
        var total = vatnumber.substring(2);

        // Establish check digit.
        total = (total * 100 + 12) % 97;

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(0, 2))
            return true;
        else
            return false;
    }

    function GBVATCheckDigit(vatnumber) {

        // Checks the check digits of a UK VAT number.

        var multipliers = [8, 7, 6, 5, 4, 3, 2];

        // Government departments
        if (vatnumber.substr(0, 2) == 'GD') {
            if (vatnumber.substr(2, 3) < 500)
                return true;
            else
                return false;
        }

        // Health authorities
        if (vatnumber.substr(0, 2) == 'HA') {
            if (vatnumber.substr(2, 3) > 499)
                return true;
            else
                return false;
        }

        // Standard and commercial numbers
        var total = 0;

        // 0 VAT numbers disallowed!
        if (Number(vatnumber.slice(0)) == 0) return false;

        // Check range is OK for modulus 97 calculation
        var no = Number(vatnumber.slice(0, 7));

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 7; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Old numbers use a simple 97 modulus, but new numbers use an adaptation of that (less 55). Our
        // VAT number could use either system, so we check it against both.

        // Establish check digits by subtracting 97 from total until negative.
        var cd = total;
        while (cd > 0) {
            cd = cd - 97;
        }

        // Get the absolute value and compare it with the last two characters of the VAT number. If the
        // same, then it is a valid traditional check digit. However, even then the number must fit within
        // certain specified ranges.
        cd = Math.abs(cd);
        if (cd == vatnumber.slice(7, 9) && no < 9990001 && (no < 100000 || no > 999999) && (no < 9490001 || no > 9700000)) return true;

        // Now try the new method by subtracting 55 from the check digit if we can - else add 42
        if (cd >= 55)
            cd = cd - 55;
        else
            cd = cd + 42;
        if (cd == vatnumber.slice(7, 9) && no > 1000000)
            return true;
        else
            return false;
    }

    function HRVATCheckDigit(vatnumber) {

        // Checks the check digits of a Croatian VAT number using ISO 7064, MOD 11-10 for check digit.

        var product = 10;
        var sum = 0;
        var checkdigit = 0;

        for (var i = 0; i < 10; i++) {

            // Extract the next digit and implement the algorithm
            sum = (Number(vatnumber.charAt(i)) + product) % 10;
            if (sum == 0) {
                sum = 10;
            }
            product = (2 * sum) % 11;
        }

        // Now check that we have the right check digit
        if ((product + vatnumber.slice(10, 11) * 1) % 10 == 1)
            return true;
        else
            return false;
    }

    function HUVATCheckDigit(vatnumber) {

        // Checks the check digits of a Hungarian VAT number.

        var total = 0;
        var multipliers = [9, 7, 3, 1, 9, 7, 3];

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 7; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digit.
        total = 10 - total % 10;
        if (total == 10) total = 0;

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(7, 8))
            return true;
        else
            return false;
    }

    function IEVATCheckDigit(vatnumber) {

        // Checks the check digits of an Irish VAT number.

        var total = 0;
        var multipliers = [8, 7, 6, 5, 4, 3, 2];

        // If the code is type 1 format, we need to convert it to the new before performing the validation.
        if (/^\d[A-Z\*\+]/.test(vatnumber)) vatnumber = "0" + vatnumber.substring(2, 7) + vatnumber.substring(0, 1) + vatnumber.substring(7, 8);

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 7; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // If the number is type 3 then we need to include the trailing A or H in the calculation
        if (/^\d{7}[A-Z][AH]$/.test(vatnumber)) {

            // Add in a multiplier for the character A (1*9=9) or H (8*9=72)
            if (vatnumber.charAt(8) == 'H')
                total += 72;
            else
                total += 9;
        }

        // Establish check digit using modulus 23, and translate to char. equivalent.
        total = total % 23;
        if (total == 0)
            total = "W";
        else
            total = String.fromCharCode(total + 64);

        // Compare it with the eighth character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(7, 8))
            return true;
        else
            return false;
    }

    function ITVATCheckDigit(vatnumber) {

        // Checks the check digits of an Italian VAT number.

        var total = 0;
        var multipliers = [1, 2, 1, 2, 1, 2, 1, 2, 1, 2];
        var temp;

        // The last three digits are the issuing office, and cannot exceed more 201, unless 999 or 888
        if (Number(vatnumber.slice(0, 7)) == 0) return false;
        temp = Number(vatnumber.slice(7, 10));
        if ((temp < 1) || (temp > 201) && temp != 999 && temp != 888) return false;

        // Extract the next digit and multiply by the appropriate
        for (var i = 0; i < 10; i++) {
            temp = Number(vatnumber.charAt(i)) * multipliers[i];
            if (temp > 9)
                total += Math.floor(temp / 10) + temp % 10;
            else
                total += temp;
        }

        // Establish check digit.
        total = 10 - total % 10;
        if (total > 9) {
            total = 0;
        }

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(10, 11))
            return true;
        else
            return false;
    }

    function LTVATCheckDigit(vatnumber) {

        // Checks the check digits of a Lithuanian VAT number.
        var total, multipliers, i;

        // 9 character VAT numbers are for legal persons
        if (vatnumber.length == 9) {

            // 8th character must be one
            if (!(/^\d{7}1/).test(vatnumber)) return false;

            // Extract the next digit and multiply by the counter+1.
            total = 0;
            for (i = 0; i < 8; i++) total += Number(vatnumber.charAt(i)) * (i + 1);

            // Can have a double check digit calculation!
            if (total % 11 == 10) {
                multipliers = [3, 4, 5, 6, 7, 8, 9, 1];
                total = 0;
                for (i = 0; i < 8; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];
            }

            // Establish check digit.
            total = total % 11;
            if (total == 10) {
                total = 0;
            }

            // Compare it with the last character of the VAT number. If it's the same, then it's valid.
            if (total == vatnumber.slice(8, 9))
                return true;
            else
                return false;
        }

        // 12 character VAT numbers are for temporarily registered taxpayers
        else {

            // 11th character must be one
            if (!(/^\d{10}1/).test(vatnumber)) return false;

            // Extract the next digit and multiply by the counter+1.
            total = 0;
            multipliers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 2];
            for (i = 0; i < 11; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

            // Can have a double check digit calculation!
            if (total % 11 == 10) {
                multipliers = [3, 4, 5, 6, 7, 8, 9, 1, 2, 3, 4];
                total = 0;
                for (i = 0; i < 11; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];
            }

            // Establish check digit.
            total = total % 11;
            if (total == 10) {
                total = 0;
            }

            // Compare it with the last character of the VAT number. If it's the same, then it's valid.
            if (total == vatnumber.slice(11, 12))
                return true;
            else
                return false;
        }
    }

    function LUVATCheckDigit(vatnumber) {

        // Checks the check digits of a Luxembourg VAT number.

        if (vatnumber.slice(0, 6) % 89 == vatnumber.slice(6, 8))
            return true;
        else
            return false;
    }

    function LVVATCheckDigit(vatnumber) {

        // Checks the check digits of a Latvian VAT number.

        // Differentiate between legal entities and natural bodies. For the latter we simply check that
        // the first six digits correspond to valid DDMMYY dates.
        if ((/^[0-3]/).test(vatnumber)) {
            if ((/^[0-3][0-9][0-1][0-9]/).test(vatnumber))
                return true;
            else
                return false;
        }

        else {

            var total = 0;
            var multipliers = [9, 1, 4, 8, 3, 10, 2, 5, 7, 6];

            // Extract the next digit and multiply by the counter.
            for (var i = 0; i < 10; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

            // Establish check digits by getting modulus 11.
            if (total % 11 == 4 && vatnumber[0] == 9) total = total - 45;
            if (total % 11 == 4)
                total = 4 - total % 11;
            else if (total % 11 > 4)
                total = 14 - total % 11;
            else if (total % 11 < 4)
                total = 3 - total % 11;

            // Compare it with the last character of the VAT number. If it's the same, then it's valid.
            if (total == vatnumber.slice(10, 11))
                return true;
            else
                return false;
        }
    }

    function MTVATCheckDigit(vatnumber) {

        // Checks the check digits of a Maltese VAT number.

        var total = 0;
        var multipliers = [3, 4, 6, 7, 8, 9];

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 6; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digits by getting modulus 37.
        total = 37 - total % 37;

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(6, 8) * 1)
            return true;
        else
            return false;
    }

    function NLVATCheckDigit(vatnumber) {

        // Checks the check digits of a Dutch VAT number.

        var total = 0;
        var multipliers = [9, 8, 7, 6, 5, 4, 3, 2];

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 8; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digits by getting modulus 11.
        total = total % 11;
        if (total > 9) {
            total = 0;
        }

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(8, 9))
            return true;
        else
            return false;
    }

    function NOVATCheckDigit(vatnumber) {

        // Checks the check digits of a Norwegian VAT number.
        // See http://www.brreg.no/english/coordination/number.html

        var total = 0;
        var multipliers = [3, 2, 7, 6, 5, 4, 3, 2];

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 8; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digits by getting modulus 11. Check digits > 9 are invalid
        total = 11 - total % 11;
        if (total == 11) {
            total = 0;
        }
        if (total < 10) {

            // Compare it with the last character of the VAT number. If it's the same, then it's valid.
            if (total == vatnumber.slice(8, 9))
                return true;
            else
                return false;
        }
    }

    function PLVATCheckDigit(vatnumber) {

        // Checks the check digits of a Polish VAT number.

        var total = 0;
        var multipliers = [6, 5, 7, 2, 3, 4, 5, 6, 7];

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 9; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digits subtracting modulus 11 from 11.
        total = total % 11;
        if (total > 9) {
            total = 0;
        }

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(9, 10))
            return true;
        else
            return false;
    }

    function PTVATCheckDigit(vatnumber) {

        // Checks the check digits of a Portugese VAT number.

        var total = 0;
        var multipliers = [9, 8, 7, 6, 5, 4, 3, 2];

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 8; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digits subtracting modulus 11 from 11.
        total = 11 - total % 11;
        if (total > 9) {
            total = 0;
        }

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(8, 9))
            return true;
        else
            return false;
    }

    function ROVATCheckDigit(vatnumber) {

        // Checks the check digits of a Romanian VAT number.

        var multipliers = [7, 5, 3, 2, 1, 7, 5, 3, 2];

        // Extract the next digit and multiply by the counter.
        var VATlen = vatnumber.length;
        multipliers = multipliers.slice(10 - VATlen);
        var total = 0;
        for (var i = 0; i < vatnumber.length - 1; i++) {
            total += Number(vatnumber.charAt(i)) * multipliers[i];
        }

        // Establish check digits by getting modulus 11.
        total = (10 * total) % 11;
        if (total == 10) total = 0;

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (total == vatnumber.slice(vatnumber.length - 1, vatnumber.length))
            return true;
        else
            return false;
    }

    function RSVATCheckDigit(vatnumber) {

        // Checks the check digits of a Serbian VAT number using ISO 7064, MOD 11-10 for check digit.

        var product = 10;
        var sum = 0;
        var checkdigit = 0;

        for (var i = 0; i < 8; i++) {

            // Extract the next digit and implement the algorithm
            sum = (Number(vatnumber.charAt(i)) + product) % 10;
            if (sum == 0) {
                sum = 10;
            }
            product = (2 * sum) % 11;
        }

        // Now check that we have the right check digit
        if ((product + vatnumber.slice(8, 9) * 1) % 10 == 1)
            return true;
        else
            return false;
    }

    function RUVATCheckDigit(vatnumber) {

        // Checks the check digits of a Russian INN number
        // See http://russianpartner.biz/test_inn.html for algorithm

        var i;

        // 10 digit INN numbers
        if (vatnumber.length == 10) {
            var total = 0;
            var multipliers = [2, 4, 10, 3, 5, 9, 4, 6, 8, 0];
            for (i = 0; i < 10; i++) {
                total += Number(vatnumber.charAt(i)) * multipliers[i];
            }
            total = total % 11;
            if (total > 9) {
                total = total % 10;
            }

            // Compare it with the last character of the VAT number. If it is the same, then it's valid
            if (total == vatnumber.slice(9, 10))
                return true;
            else
                return false;

            // 12 digit INN numbers
        } else if (vatnumber.length == 12) {
            var total1 = 0;
            var multipliers1 = [7, 2, 4, 10, 3, 5, 9, 4, 6, 8, 0];
            var total2 = 0;
            var multipliers2 = [3, 7, 2, 4, 10, 3, 5, 9, 4, 6, 8, 0];

            for (i = 0; i < 11; i++) total1 += Number(vatnumber.charAt(i)) * multipliers1[i];
            total1 = total1 % 11;
            if (total1 > 9) {
                total1 = total1 % 10;
            }

            for (i = 0; i < 11; i++) total2 += Number(vatnumber.charAt(i)) * multipliers2[i];
            total2 = total2 % 11;
            if (total2 > 9) {
                total2 = total2 % 10;
            }

            // Compare the first check with the 11th character and the second check with the 12th and last
            // character of the VAT number. If they're both the same, then it's valid
            if ((total1 == vatnumber.slice(10, 11)) && (total2 == vatnumber.slice(11, 12)))
                return true;
            else
                return false;
        }
    }

    function SEVATCheckDigit(vatnumber) {
        var i;

        // Calculate R where R = R1 + R3 + R5 + R7 + R9, and Ri = INT(Ci/5) + (Ci*2) modulo 10
        var R = 0;
        var digit;
        for (i = 0; i < 9; i = i + 2) {
            digit = Number(vatnumber.charAt(i));
            R += Math.floor(digit / 5) + ((digit * 2) % 10);
        }

        // Calculate S where S = C2 + C4 + C6 + C8
        var S = 0;
        for (i = 1; i < 9; i = i + 2) S += Number(vatnumber.charAt(i));

        // Calculate the Check Digit
        var cd = (10 - (R + S) % 10) % 10;

        // Compare it with the last character of the VAT number. If it's the same, then it's valid.
        if (cd == vatnumber.slice(9, 10))
            return true;
        else
            return false;
    }

    function SIVATCheckDigit(vatnumber) {

        // Checks the check digits of a Slovenian VAT number.

        var total = 0;
        var multipliers = [8, 7, 6, 5, 4, 3, 2];

        // Extract the next digit and multiply by the counter.
        for (var i = 0; i < 7; i++) total += Number(vatnumber.charAt(i)) * multipliers[i];

        // Establish check digits using modulus 11
        total = 11 - total % 11;
        if (total == 10) {
            total = 0;
        }

        // Compare the number with the last character of the VAT number. If it is the
        // same, then it's a valid check digit.
        if (total != 11 && total == vatnumber.slice(7, 8))
            return true;
        else
            return false;
    }

    function SKVATCheckDigit(vatnumber) {

        // Checks the check digits of a Slovakian VAT number.

        // Check that the modulus of the whole VAT number is 0 - else error
        if (Number(vatnumber % 11) == 0)
            return true;
        else
            return false;
    }

    XIVATCheckDigit = GBVATCheckDigit;  // Northern Ireland VAT numbers follow the same rules as GB

    return checkVATNumber;
})();
```

## File: static\src\js\partner_autocomplete_core.js

```javascript
/* global checkVATNumber */
odoo.define('partner.autocomplete.Mixin', function (require) {
'use strict';

var concurrency = require('web.concurrency');

var core = require('web.core');
var Qweb = core.qweb;
var utils = require('web.utils');
var _t = core._t;

/**
 * This mixin only works with classes having EventDispatcherMixin in 'web.mixins'
 */
var PartnerAutocompleteMixin = {
    _dropPreviousOdoo: new concurrency.DropPrevious(),
    _dropPreviousClearbit: new concurrency.DropPrevious(),
    _timeout : 1000, // Timeout for Clearbit autocomplete in ms

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Get list of companies via Autocomplete API
     *
     * @param {string} value
     * @returns {Promise}
     * @private
     */
    _autocomplete: function (value) {
        var self = this;
        value = value.trim();
        var isVAT = this._isVAT(value);
        var odooSuggestions = [];
        var clearbitSuggestions = [];
        return new Promise(function (resolve, reject) {
            var odooPromise = self._getOdooSuggestions(value, isVAT).then(function (suggestions){
                odooSuggestions = suggestions;
            });

            // Only get Clearbit suggestions if not a VAT number
            var clearbitPromise = isVAT ? false : self._getClearbitSuggestions(value).then(function (suggestions){
                clearbitSuggestions = suggestions;
            });

            var concatResults = function () {
                // Add Clearbit result with Odoo result (with unique domain)
                if (clearbitSuggestions && clearbitSuggestions.length) {
                    var websites = odooSuggestions.map(function (suggestion) {
                        return suggestion.website;
                    });
                    clearbitSuggestions.forEach(function (suggestion) {
                        if (websites.indexOf(suggestion.domain) < 0) {
                            websites.push(suggestion.domain);
                            odooSuggestions.push(suggestion);
                        }
                    });
                }

                odooSuggestions = _.filter(odooSuggestions, function (suggestion) {
                    return !suggestion.ignored;
                });
                _.each(odooSuggestions, function(suggestion){
                delete suggestion.ignored;
                });
                return resolve(odooSuggestions);
            };

            self._whenAll([odooPromise, clearbitPromise]).then(concatResults, concatResults);
        });

    },

    /**
     * Get enrichment data
     *
     * @param {Object} company
     * @param {string} company.website
     * @param {string} company.partner_gid
     * @param {string} company.vat
     * @returns {Promise}
     * @private
     */
    _enrichCompany: function (company) {
        return this._rpc({
            model: 'res.partner',
            method: 'enrich_company',
            args: [company.website, company.partner_gid, company.vat],
        });
    },

    /**
     * Get the company logo as Base 64 image from url
     *
     * @param {string} url
     * @returns {Promise}
     * @private
     */
    _getCompanyLogo: function (url) {
        return this._getBase64Image(url).then(function (base64Image) {
            // base64Image equals "data:" if image not available on given url
            return base64Image ? base64Image.replace(/^data:image[^;]*;base64,?/, '') : false;
        }).catch(function () {
            return false;
        });
    },

    /**
     * Get enriched data + logo before populating partner form
     *
     * @param {Object} company
     * @returns {Promise}
     */
    _getCreateData: function (company) {
        var self = this;

        var removeUselessFields = function (company) {
            var fields = 'label,description,domain,logo,legal_name,ignored,email,skip_enrich'.split(',');
            fields.forEach(function (field) {
                delete company[field];
            });

            var notEmptyFields = "country_id,state_id".split(',');
            notEmptyFields.forEach(function (field) {
                if (!company[field]) delete company[field];
            });
        };

        return new Promise(function (resolve) {
            // Fetch additional company info via Autocomplete Enrichment API
            var enrichPromise = !company.skip_enrich ? self._enrichCompany(company) : false;

            // Get logo
            var logoPromise = company.logo ? self._getCompanyLogo(company.logo) : false;
            self._whenAll([enrichPromise, logoPromise]).then(function (result) {
                var company_data = result[0];
                var logo_data = result[1];

                // The vat should be returned for free. This is the reason why
                // we add it into the data of 'company' even if an error such as
                // an insufficient credit error is raised. 
                if (company_data.error && company_data.vat) {
                    company.vat = company_data.vat;
                }

                if (company_data.error) {
                    if (company_data.error_message === 'Insufficient Credit') {
                        self._notifyNoCredits();
                    } else if (company_data.error_message === 'No Account Token') {
                        self._notifyAccountToken();
                    } else {
                        self.displayNotification({ message: company_data.error_message });
                    }
                    company_data = company;
                }

                if (_.isEmpty(company_data)) {
                    company_data = company;
                }

                // Delete attribute to avoid "Field_changed" errors
                removeUselessFields(company_data);

                // Assign VAT coming from parent VIES VAT query
                if (company.vat) {
                    company_data.vat = company.vat;
                }
                resolve({
                    company: company_data,
                    logo: logo_data
                });
            });
        });
    },

    /**
     * Check connectivity
     *
     * @returns {boolean}
     */
    _isOnline: function () {
        return navigator && navigator.onLine;
    },

    /**
     * Validate: Not empty and length > 1
     *
     * @param {string} search_val
     * @param {string} onlyVAT : Only valid VAT Number search
     * @returns {boolean}
     * @private
     */
    _validateSearchTerm: function (search_val, onlyVAT) {
        if (onlyVAT) return this._isVAT(search_val);
        else return search_val && search_val.length > 2;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Returns a promise which will be resolved with the base64 data of the
     * image fetched from the given url.
     *
     * @private
     * @param {string} url : the url where to find the image to fetch
     * @returns {Promise}
     */
    _getBase64Image: function (url) {
        return new Promise(function (resolve, reject) {
            var xhr = new XMLHttpRequest();
            xhr.onload = function () {
                utils.getDataURLFromFile(xhr.response).then(resolve);
            };
            xhr.open('GET', url);
            xhr.responseType = 'blob';
            xhr.onerror = reject;
            xhr.send();
        });
    },

    /**
     * Use Clearbit Autocomplete API to return suggestions
     *
     * @param {string} value
     * @returns {Promise}
     * @private
     */
    _getClearbitSuggestions: function (value) {
        var url = 'https://autocomplete.clearbit.com/v1/companies/suggest?query=' + value;
        var def = $.ajax({
            url: url,
            dataType: 'json',
            timeout: this._timeout,
            success: function (suggestions) {
                suggestions.map(function (suggestion) {
                    suggestion.label = suggestion.name;
                    suggestion.website = suggestion.domain;
                    suggestion.description = suggestion.website;
                    return suggestion;
                });
                return suggestions;
            },
        });

        return this._dropPreviousClearbit.add(def);
    },

    /**
     * Use Odoo Autocomplete API to return suggestions
     *
     * @param {string} value
     * @param {boolean} isVAT
     * @returns {Promise}
     * @private
     */
    _getOdooSuggestions: function (value, isVAT) {
        var method = isVAT ? 'read_by_vat' : 'autocomplete';

        var def = this._rpc({
            model: 'res.partner',
            method: method,
            args: [value],
        }, {
            shadow: true,
        }).then(function (suggestions) {
            suggestions.map(function (suggestion) {
                suggestion.logo = suggestion.logo || '';
                suggestion.label = suggestion.legal_name || suggestion.name;
                if (suggestion.vat) suggestion.description = suggestion.vat;
                else if (suggestion.website) suggestion.description = suggestion.website;

                if (suggestion.country_id && suggestion.country_id.display_name) {
                    if (suggestion.description) suggestion.description += _.str.sprintf(' (%s)', suggestion.country_id.display_name);
                    else suggestion.description += suggestion.country_id.display_name;
                }

                return suggestion;
            });
            return suggestions;
        });

        return this._dropPreviousOdoo.add(def);
    },
    /**
     * Check if searched value is possibly a VAT : 2 first chars = alpha + min 5 numbers
     *
     * @param {string} search_val
     * @returns {boolean}
     * @private
     */
    _isVAT: function (search_val) {
        var str = this._sanitizeVAT(search_val);
        return checkVATNumber(str);
    },

    /**
     * Sanitize search value by removing all not alphanumeric
     *
     * @param {string} search_value
     * @returns {string}
     * @private
     */
    _sanitizeVAT: function (search_value) {
        return search_value ? search_value.replace(/[^A-Za-z0-9]/g, '') : '';
    },

    /**
     * Utility to wait for multiple promises
     * Promise.all will reject all promises whenever a promise is rejected
     * This utility will continue
     *
     * @param {Promise[]} promises
     * @returns {Promise}
     * @private
     */
    _whenAll: function (promises) {
        return Promise.all(promises.map(function (p) {
            return Promise.resolve(p);
        }));
    },

    /**
     * @private
     * @returns {Promise}
     */
    _notifyNoCredits: function () {
        var self = this;
        return this._rpc({
            model: 'iap.account',
            method: 'get_credits_url',
            args: ['partner_autocomplete'],
        }).then(function (url) {
            var title = _t('Not enough credits for Partner Autocomplete');
            var content = Qweb.render('partner_autocomplete.insufficient_credit_notification', {
                credits_url: url
            });
            self.displayNotification({
                title,
                message: utils.Markup(content),
                className: 'o_partner_autocomplete_no_credits_notify',
            });
        });
    },

    _notifyAccountToken: function () {
        var self = this;
        return this._rpc({
            model: 'iap.account',
            method: 'get_config_account_url',
            args: []
        }).then(function (url) {
            var title = _t('IAP Account Token missing');
            if (url){
                var content = Qweb.render('partner_autocomplete.account_token', {
                    account_url: url
                });
                self.displayNotification({
                    title,
                    message: utils.Markup(content),
                    className: 'o_partner_autocomplete_no_credits_notify',
                });
            }
            else {
                self.displayNotification({ title });
            }
        });
    },
};

return PartnerAutocompleteMixin;

});

```

## File: static\src\js\partner_autocomplete_fieldchar.js

```javascript
odoo.define('partner.autocomplete.fieldchar', function (require) {
'use strict';

var basic_fields = require('web.basic_fields');
var core = require('web.core');
var field_registry = require('web.field_registry');
var AutocompleteMixin = require('partner.autocomplete.Mixin');

var QWeb = core.qweb;

var FieldChar = basic_fields.FieldChar;

/**
 * FieldChar extension to suggest existing companies when changing the company
 * name on a res.partner view (indeed, it is designed to change the "name",
 * "website" and "image" fields of records of this model).
 */
var FieldAutocomplete = FieldChar.extend(AutocompleteMixin, {
    className: 'o_field_partner_autocomplete',
    debounceSuggestions: 400,
    resetOnAnyFieldChange: true,

    jsLibs: [
        '/partner_autocomplete/static/lib/jsvat.js'
    ],

    events: _.extend({}, FieldChar.prototype.events, {
        'keyup': '_onKeyup',
        'mousedown .o_partner_autocomplete_suggestion': '_onMousedown',
        'focusout': '_onFocusout',
        'mouseenter .o_partner_autocomplete_suggestion': '_onHoverDropdown',
        'click .o_partner_autocomplete_suggestion': '_onSuggestionClicked',
    }),

    /**
     * @constructor
     * Prepares the basic rendering of edit mode by setting the root to be a
     * div.dropdown.open.
     * @see FieldChar.init
     */
    init: function () {
        this._super.apply(this, arguments);

        // If the autocomplete is applied to vat field, only search valid vat number
        this.onlyVAT = this.name === 'vat';

        if (this.mode === 'edit') {
            this.tagName = 'div';
            this.className += ' dropdown open w-100';
        }

        if (this.debounceSuggestions > 0) {
            this._suggestCompanies = _.debounce(this._suggestCompanies.bind(this), this.debounceSuggestions);
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Check if the autocomplete should be active
     * Active :
     *  - only when creating new record
     *  - on model res.partner and is_company=true
     *  - on model res.company
     *
     * @returns {boolean}
     * @private
     */
    _isActive: function () {
        return this.model === 'res.company' ||
            (
                this.model === 'res.partner'
                && this.record.data.is_company
                && !(this.record.data && this.record.data.id)
            );
    },

    /**
     *
     * @private
     */
    _removeDropdown: function () {
        if (this.$dropdown) {
            this.$dropdown.remove();
            this.$dropdown = undefined;
        }
    },

    /**
     * Adds the <input/> element and prepares it. Note: the dropdown rendering
     * is handled outside of the rendering routine (but instead by reacting to
     * user input).
     *
     * @override
     * @private
     */
    _renderEdit: function () {
        this.$el.empty();
        // Prepare and add the input
        this._prepareInput().appendTo(this.$el);
    },

    /**
     * Selects the given company suggestions by notifying changes to the view
     * for the "name", "website" and "image" fields. This is of course intended
     * to work only with the "res.partner" form view.
     *
     * @private
     * @param {Object} company
     */
    _selectCompany: function (company) {
        var self = this;
        this._getCreateData(company).then(function (data) {
            if (data.logo) {
                var logoField = self.model === 'res.partner' ? 'image_1920' : 'logo';
                data.company[logoField] = data.logo;
            }

            // Some fields are unnecessary in res.company
            if (self.model === 'res.company') {
                var fields = 'comment,child_ids,bank_ids,additional_info'.split(',');
                fields.forEach(function (field) {
                    delete data.company[field];
                });
            }

            self._setOne2ManyField('bank_ids', data.company.bank_ids);
            delete data.company.bank_ids;

            self.trigger_up('field_changed', {
                dataPointID: self.dataPointID,
                changes: data.company,
                onSuccess: function () {
                    // update the input's value directly
                    if (self.onlyVAT)
                        self.$input.val(self._formatValue(company.vat));
                    else
                        self.$input.val(self._formatValue(company.name));
                },
            });
        });
        this._removeDropdown();
    },

    _setOne2ManyField: function (field, list) {
        var self = this;
        var viewType = this.record.viewType;
        if (list && this.record.fieldsInfo[viewType] && this.record.fieldsInfo[viewType][field]) {
            list.forEach(function (item) {
                var changes = {};
                changes[field] = {
                    operation: 'CREATE',
                    data: item,
                };

                self.trigger_up('field_changed', {
                    dataPointID: self.dataPointID,
                    changes: changes,
                });
            });
        }
    },

    /**
     * Shows the dropdown with the suggestions. If one is
     * already opened, it removes the old one before rerendering the dropdown.
     *
     * @private
     */
    _showDropdown: function () {
        this._removeDropdown();
        if (this.suggestions.length > 0) {
            this.$dropdown = $(QWeb.render('partner_autocomplete.dropdown', {
                suggestions: this.suggestions,
            }));
            this.$dropdown.appendTo(this.$el);
            // We need to make sure that the element containing the dropdown menu doesn't have the
            // style attribute overflow: hidden, otherwise the dropdown menu will be hidden by it
            this.$el.removeClass('o_text_overflow');
            this.$input.addClass('o_text_overflow');
        }
    },

    /**
     * Shows suggestions according to the given value.
     * Note: this method is debounced (@see init).
     *
     * @private
     * @param {string} value - searched term
     */
    _suggestCompanies: function (value) {
        var self = this;
        if (this._validateSearchTerm(value, this.onlyVAT) && this._isOnline()) {
            return this._autocomplete(value).then(function (suggestions) {
                if (suggestions && suggestions.length) {
                    self.suggestions = suggestions;
                    self._showDropdown();
                } else {
                    self._removeDropdown();
                }
            });
        } else {
            this._removeDropdown();
        }
    },


    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Called on focusout -> removes the suggestions dropdown.
     *
     * @private
     */
    _onFocusout: function () {
        this._removeDropdown();
    },

    /**
     * Called when hovering a suggestion in the dropdown -> sets it as active.
     *
     * @private
     * @param {Event} e
     */
    _onHoverDropdown: function (e) {
        this.$dropdown.find('.active').removeClass('active');
        $(e.currentTarget).parent().addClass('active');
    },

    /**
     * @override of FieldChar (called when the user is typing text)
     * Checks the <input/> value and shows suggestions according to
     * this value.
     *
     * @private
     */
    _onInput: function () {
        this._super.apply(this, arguments);
        if (this._isActive()) {
            this._suggestCompanies(this.$input.val());
        }
    },

    /**
     * @override of FieldChar
     * Changes the "up" and "down" key behavior when the dropdown is opened (to
     * navigate through dropdown suggestions).
     * Triggered by keydown to execute the navigation multiple times when the
     * user keeps the "down" or "up" pressed.
     *
     * @private
     * @param {Event} e
     */
    _onKeydown: function (e) {
        switch (e.which) {
            case $.ui.keyCode.UP:
            case $.ui.keyCode.DOWN:
                if (!this.$dropdown) {
                    break;
                }
                e.preventDefault();
                var $suggestions = this.$dropdown.children();
                var $active = $suggestions.filter('.active');
                var $to;
                if ($active.length) {
                    $to = e.which === $.ui.keyCode.DOWN ?
                        $active.next() :
                        $active.prev();
                } else {
                    $to = $suggestions.first();
                }
                if ($to.length) {
                    $active.removeClass('active');
                    $to.addClass('active');
                }
                return;
        }
        this._super.apply(this, arguments);
    },

    /**
     * Called on keyup events to:
     * -> remove the suggestions dropdown when hitting the "escape" key
     * -> select the highlighted suggestion when hitting the "enter" key
     *
     * @private
     * @param {Event} e
     */
    _onKeyup: function (e) {
        switch (e.which) {
            case $.ui.keyCode.ESCAPE:
                e.preventDefault();
                this._removeDropdown();
                break;
            case $.ui.keyCode.ENTER:
                if (!this.$dropdown) {
                    break;
                }
                e.preventDefault();
                var $active = this.$dropdown.find('.o_partner_autocomplete_suggestion.active');
                if (!$active.length) {
                    return;
                }
                this._selectCompany(this.suggestions[$active.data('index')]);
                break;
        }
    },

    /**
     * Called on mousedown event on a suggestion -> prevent default
     * action so that the <input/> element does not lose the focus.
     *
     * @private
     * @param {Event} e
     */
    _onMousedown: function (e) {
        e.preventDefault(); // prevent losing focus on suggestion click
    },

    /**
     * Called when a dropdown suggestion is clicked -> trigger_up changes for
     * some fields in the view (not only this <input/> one) with the associated
     * data (@see _selectCompany).
     *
     * @private
     * @param {Event} e
     */
    _onSuggestionClicked: function (e) {
        e.preventDefault();
        this._selectCompany(this.suggestions[$(e.currentTarget).data('index')]);
    },
});

field_registry.add('field_partner_autocomplete', FieldAutocomplete);

return FieldAutocomplete;
});

```

## File: static\src\js\partner_autocomplete_many2one.js

```javascript
odoo.define('partner.autocomplete.many2one', function (require) {
'use strict';

var FieldMany2One = require('web.relational_fields').FieldMany2One;
var core = require('web.core');
var AutocompleteMixin = require('partner.autocomplete.Mixin');
var field_registry = require('web.field_registry');

var _t = core._t;

var PartnerField = FieldMany2One.extend(AutocompleteMixin, {
    jsLibs: [
        '/partner_autocomplete/static/lib/jsvat.js'
    ],

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this._addAutocompleteSource(this._searchSuggestions, {
            placeholder: _t('Searching Autocomplete...'),
            order: 20,
            validation: this._validateSearchTerm,
        });

        this.additionalContext['show_vat'] = true;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Action : create popup form with pre-filled values from Autocomplete
     *
     * @param {Object} company
     * @returns {Promise}
     * @private
     */
    _createPartner: function (company) {
        var self = this;
        self.$('input').val('');

        return self._getCreateData(company).then(function (data){
            var context = {
                'default_is_company': true
            };
            _.each(data.company, function (val, key) {
                context['default_' + key] = val && val.id ? val.id : val;
            });

            // if(data.company.street_name && !data.company.street_number) context.default_street_number = '';
            if (data.logo) context.default_image_1920 = data.logo;

            return self._searchCreatePopup("form", false, context);
        });
    },

    /**
     * Modify autocomplete results rendering
     * Add logo in the autocomplete results if logo is provided
     *
     * @private
     */
    _modifyAutompleteRendering: function (){
        var api = this.$input.data('ui-autocomplete');
        // FIXME: bugfix to prevent traceback in mobile apps due to override
        // of Many2one widget with native implementation.
        if (!api) {
            return;
        }
        api._renderItem = function(ul, item){
            ul.addClass('o_partner_autocomplete_dropdown');
            var $a = $('<a/>')["html"](item.label);
            if (item.logo){
                var $img = $('<img/>').attr('src', item.logo);
                $a.append($img);
            }

            return $("<li></li>")
                .data("item.autocomplete",item)
                .append($a)
                .appendTo(ul)
                .addClass(item.classname);
        };
    },

    /**
     * @override
     * @private
     */
    _renderEdit: function (){
        this._super.apply(this, arguments);
        this._modifyAutompleteRendering();
    },

    /**
     * Query Autocomplete and add results to the popup
     *
     * @override
     * @param search_val {string}
     * @returns {Promise}
     * @private
     */
    _searchSuggestions: function (search_val) {
        var self = this;
        return new Promise(function (resolve, reject) {
            if (self._isOnline()) {

                self._autocomplete(search_val).then(function (suggestions) {
                    var choices = [];
                    if (suggestions && suggestions.length) {
                        _.each(suggestions, function (suggestion) {
                            var label = '<i class="fa fa-magic text-muted"/> ';
                            label += _.str.sprintf('%s, <span class="text-muted">%s</span>', suggestion.label, suggestion.description);

                            choices.push({
                                label: label,
                                action: function () {
                                    self._createPartner(suggestion);
                                },
                                logo: suggestion.logo,
                                classname: 'o_partner_autocomplete_dropdown_item',
                            });
                        });
                    }

                    resolve(choices);
                });
            } else {
               resolve([]);
            }
        });
    },
});

field_registry.add('res_partner_many2one', PartnerField);

return PartnerField;
});

```

## File: static\src\js\web_company_autocomplete.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { session } from "@web/session";

export const companyAutocompleteService = {
    dependencies: ["orm", "company"],

    start(env, { orm, company }) {
        if (session.iap_company_enrich) {
            const currentCompanyId = company.currentCompany.id;
            orm.silent.call("res.company", "iap_enrich_auto", [currentCompanyId], {});
        }
    },
};

registry
    .category("services")
    .add("partner_autocomplete.companyAutocomplete", companyAutocompleteService);

```

## File: static\src\xml\partner_autocomplete.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
    <div t-name="partner_autocomplete.dropdown" class="o_partner_autocomplete_dropdown dropdown-menu show" role="menu">
        <t t-foreach="suggestions" t-as="info">
            <a role="menuitem" href="#"
                t-attf-class="dropdown-item o_partner_autocomplete_suggestion clearfix#{info_index == 0 and ' active' or ''}"
                t-att-data-index="info_index">
                <img t-att-src="info['logo']" onerror="this.src='/base/static/img/company_image.png'" alt="Placeholder"/>
                <div class="o_partner_autocomplete_info">
                    <strong><t t-esc="info['label'] or '&#160;'"/></strong>
                    <div><t t-esc="info['description']"/></div>
                </div>
            </a>
        </t>
    </div>

    <!--
        @param {string} credits_url
    -->
    <div t-name="partner_autocomplete.insufficient_credit_notification" class="o-hidden-ios">
        <a class="btn btn-link" t-att-href="credits_url"><i class="fa fa-arrow-right"/> Buy more credits</a>
    </div>

    <div t-name="partner_autocomplete.account_token" class="">
        <a class="btn btn-link" t-att-href="account_url" ><i class="fa fa-arrow-right"/> Set Your Account Token</a>
    </div>
</templates>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_company_form_inherit_partner_autocomplete" model="ir.ui.view">
        <field name="name">res.company.form.inherit.web.partner.autocomplete</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_title')]/h1/field[@name='name']" position="attributes">
                <attribute name="widget">field_partner_autocomplete</attribute>
            </xpath>
            <xpath expr="//field[@name='vat']" position="attributes">
                <attribute name="widget">field_partner_autocomplete</attribute>
            </xpath>
            <xpath expr="//field[@name='company_registry']" position="after">
                <field name="partner_gid" invisible="1"/>
                <field name="iap_enrich_auto_done" invisible="1"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.partner.autcomplete</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="partner_autocomplete_settings" position="inside">
                <widget name="iap_buy_more_credits" service_name="partner_autocomplete" hide_service="1"/>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_res_partner_form_inherit_partner_autocomplete" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.partner.autocomplete</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_title')]/h1/field[@id='company']" position="attributes">
                <attribute name="widget">field_partner_autocomplete</attribute>
            </xpath>
            <xpath expr="//div[hasclass('oe_title')]/h1/field[@id='individual']" position="attributes">
                <attribute name="widget">field_partner_autocomplete</attribute>
            </xpath>
            <xpath expr="//field[@name='vat']" position="attributes">
                <attribute name="widget">field_partner_autocomplete</attribute>
            </xpath>
            <xpath expr="//field[last()]" position="after">
                <field name="partner_gid" invisible="True"/>
                <field name="additional_info" invisible="True"/>
            </xpath>
        </field>
    </record>

    <record id="view_partner_simple_form_inherit_partner_autocomplete" model="ir.ui.view">
        <field name="name">res.partner.simplified.form.inherit.partner.autocomplete</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_simple_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_title')]/h1/field[@id='company']" position="attributes">
                <attribute name="widget">field_partner_autocomplete</attribute>
            </xpath>
            <xpath expr="//div[hasclass('oe_title')]/h1/field[@id='individual']" position="attributes">
                <attribute name="widget">field_partner_autocomplete</attribute>
            </xpath>
            <xpath expr="//field[last()]" position="after">
                <field name="partner_gid" invisible="True"/>
                <field name="additional_info" invisible="True"/>
            </xpath>
        </field>
    </record>
</odoo>

```

