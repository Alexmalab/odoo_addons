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
            'partner_autocomplete/static/src/js/*',
            'partner_autocomplete/static/src/xml/*',
        ],
        'web.tests_assets': [
            'partner_autocomplete/static/lib/**/*',
        ],
        'web.qunit_suite_tests': [
            'partner_autocomplete/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\cron.xml

```xml
<odoo>
    <record id="ir_cron_partner_autocomplete" model="ir.cron">
        <field name="name">Partner Autocomplete: Sync with remote DB</field>
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
            results = self._contact_iap('/api/dnb/1', action, params, timeout=timeout)
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


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        """ Add information about iap enrich to perform """
        session_info = super(Http, self).session_info()
        if session_info.get('is_admin'):
            session_info['iap_company_enrich'] = not self.env.user.company_id.iap_enrich_auto_done
        return session_info

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import threading

from odoo.addons.iap.tools import iap_tools
from odoo import api, fields, models, tools

_logger = logging.getLogger(__name__)

COMPANY_AC_TIMEOUT = 5


class ResCompany(models.Model):
    _name = 'res.company'
    _inherit = 'res.company'

    partner_gid = fields.Integer('Company database ID', related="partner_id.partner_gid", store=True)
    iap_enrich_auto_done = fields.Boolean('Enrich Done')

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        if not getattr(threading.current_thread(), 'testing', False):
            res.iap_enrich_auto()
        return res

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)

        if view_type == 'form':
            for node in arch.xpath(
                "//field[@name='name']"
                "|//field[@name='vat']"
            ):
                node.attrib['widget'] = 'field_partner_autocomplete'

        return arch, view

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

        company_data = self.env['res.partner'].enrich_by_domain(company_domain, timeout=COMPANY_AC_TIMEOUT)
        if not company_data or company_data.get("error"):
            return

        company_data = {field: value for field, value in company_data.items()
                        if field in self.partner_id._fields and value and (field == 'image_1920' or not self.partner_id[field])}

        # for company: from state_id / country_id display_name like to IDs
        company_data.update(self._enrich_extract_m2o_id(company_data, ['state_id', 'country_id']))

        self.partner_id.write(company_data)
        return True

    def _enrich_extract_m2o_id(self, iap_data, m2o_fields):
        """ Extract m2O ids from data (because of res.partner._format_data_company) """
        extracted_data = {}
        for m2o_field in m2o_fields:
            relation_data = iap_data.get(m2o_field)
            if relation_data and isinstance(relation_data, dict):
                extracted_data[m2o_field] = relation_data.get('id', False)
        return extracted_data

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
import logging
import re

from stdnum.eu.vat import check_vies

from odoo import api, fields, models, _

_logger = logging.getLogger(__name__)


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

        if country:
            iap_data['country_id'] = {'id': country.id, 'display_name': country.display_name}
        if state:
            iap_data['state_id'] = {'id': state.id, 'display_name': state.display_name}

        return iap_data

    @api.model
    def _iap_replace_language_codes(self, iap_data):
        if lang := iap_data.pop('preferred_language', False):
            if installed_lang := (
                self.env['res.lang'].search([('code', '=', lang), ('iso_code', '=', lang)])  # specific lang (e.g.: fr_BE)
                or
                self.env['res.lang'].search([('code', 'ilike', lang[:2]), ('iso_code', 'ilike', lang[:2])], limit=1)  # fallback to generic lang (e.g. fr)
            ):
                iap_data['lang'] = installed_lang.code
        return iap_data

    @api.model
    def _format_data_company(self, iap_data):
        self._iap_replace_location_codes(iap_data)
        self._iap_replace_language_codes(iap_data)
        return iap_data

    # Deprecated since DnB
    @api.model
    def autocomplete(self, query, timeout=15):
        return []

    # Deprecated since DnB
    @api.model
    def enrich_company(self, company_domain, partner_gid, vat, timeout=15):
        return {}

    # Deprecated since DnB
    @api.model
    def read_by_vat(self, vat, timeout=15):
        return []

    # Deprecated since DnB
    def check_gst_in(self, vat):
        return False

    @api.model
    def autocomplete_by_name(self, query, query_country_id, timeout=15):
        if query_country_id is False:  # If it's 0, we purposely do not want to filter on the country
            query_country_id = self.env.company.country_id.id
        query_country_code = self.env['res.country'].browse(query_country_id).code
        response, _ = self.env['iap.autocomplete.api']._request_partner_autocomplete('search_by_name', {
            'query': query,
            'query_country_code': query_country_code,
        }, timeout=timeout)
        if response and not response.get("error"):
            results = []
            for suggestion in response.get("data"):
                results.append(self._format_data_company(suggestion))
            return results
        else:
            return []

    @api.model
    def autocomplete_by_vat(self, vat, query_country_id, timeout=15):
        query_country_id = query_country_id or self.env.company.country_id.id
        query_country_code = self.env['res.country'].browse(query_country_id).code
        response, _ = self.env['iap.autocomplete.api']._request_partner_autocomplete('search_by_vat', {
            'query': vat,
            'query_country_code': query_country_code,
        }, timeout=timeout)
        if response and not response.get("error"):
            results = []
            for suggestion in response.get("data"):
                results.append(self._format_data_company(suggestion))
            return results
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
                    })]
            return []

    @api.model
    def _process_enriched_response(self, response, error):
        if response and response.get('data'):
            result = self._format_data_company(response.get('data'))
        else:
            result = {}

        if response and response.get('credit_error'):
            result.update({
                'error': True,
                'error_message': 'Insufficient Credit'
            })
        elif response and response.get('error'):
            result.update({
                'error': True,
                'error_message': _('Unable to enrich company (no credit was consumed).'),
            })
        elif error:
            result.update({
                'error': True,
                'error_message': error
            })
        return result

    @api.model
    def enrich_by_duns(self, duns, timeout=15):
        response, error = self.env['iap.autocomplete.api']._request_partner_autocomplete('enrich_by_duns', {
            'duns': duns,
        }, timeout=timeout)
        return self._process_enriched_response(response, error)

    @api.model
    def enrich_by_gst(self, gst, timeout=15):
        response, error = self.env['iap.autocomplete.api']._request_partner_autocomplete('enrich_by_gst', {
            'gst': gst,
        }, timeout=timeout)
        return self._process_enriched_response(response, error)

    @api.model
    def enrich_by_domain(self, domain, timeout=15):
        response, error = self.env['iap.autocomplete.api']._request_partner_autocomplete('enrich_by_domain', {
            'domain': domain,
        }, timeout=timeout)
        return self._process_enriched_response(response, error)

    def iap_partner_autocomplete_add_tags(self, unspsc_codes):
        """Called by JS to create the activity tags from the UNSPSC codes"""
        self.ensure_one()

        # If the UNSPSC module is installed, we might have a translation, so let's use it
        if self.env['ir.module.module']._get('product_unspsc').state == 'installed':
            tag_names = self.env['product.unspsc.code']\
                            .with_context(active_test=False)\
                            .search([('code', 'in', [unspsc_code for unspsc_code, __ in unspsc_codes])])\
                            .mapped('name')
        # If it's not, then we use the default English name provided by DnB
        else:
            tag_names = [unspsc_name for __, unspsc_name in unspsc_codes]

        tag_ids = self.env['res.partner.category']
        for tag_name in tag_names:
            if existing_tag := self.env['res.partner.category'].search([('name', '=', tag_name)]):
                tag_ids |= existing_tag
            else:
                tag_ids |= self.env['res.partner.category'].create({'name': tag_name})
        self.category_id = tag_ids

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)

        if view_type == 'form':
            for node in arch.xpath(
                "//field[@name='name']"
                "|//field[@name='vat']"
            ):
                node.attrib['widget'] = 'field_partner_autocomplete'

        return arch, view

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
        pass  # Deprecated since DnB

    def add_to_queue(self, partner_id):
        pass  # Deprecated since DnB

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
access_partner_autocomplete_sync_portal,res.partner.autocomplete.sync.portal,model_res_partner_autocomplete_sync,base.group_portal,1,0,1,0
access_partner_autocomplete_sync_user,res.partner.autocomplete.sync.user,model_res_partner_autocomplete_sync,base.group_user,1,1,1,0
access_partner_autocomplete_sync_system,res.partner.autocomplete.sync.system,model_res_partner_autocomplete_sync,base.group_system,1,1,1,1
```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M0 8h33a4 4 0 0 1 4 4v13H4a4 4 0 0 1-4-4V8Z" fill="#FC868B"/><path d="M4 17a4 4 0 0 1 4-4h38a4 4 0 0 1 4 4v8H4v-8Z" fill="#985184"/><path d="M37 13v12H4v-8a4 4 0 0 1 4-4h29Z" fill="#712258"/><path d="M4 31h46v6H4v-6Z" fill="#F78613"/><path d="M4 25h46v6H4v-6Z" fill="#962B48"/><path d="M4 37h46v1a4 4 0 0 1-4 4H8a4 4 0 0 1-4-4v-1Z" fill="#FBB945"/></svg>

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

## File: static\src\js\partner_autocomplete_component.js

```javascript
/** @odoo-module **/

import { AutoComplete } from "@web/core/autocomplete/autocomplete";


export class PartnerAutoComplete extends AutoComplete {
    static template = "partner_autocomplete.PartnerAutoComplete";

    setup() {
        super.setup();
        this.shouldSearchWorldwide = false;
    }

    // Override of AutoComplete
    loadOptions(options, request) {
        if (typeof options === "function") {
            return options(request, this.shouldSearchWorldwide);
        } else {
            return options;
        }
    }

    async searchWorldwide(ev){
        this.shouldSearchWorldwide = true;
        ev.preventDefault();
        super.close();
        super.open(true);
    }
}

```

## File: static\src\js\partner_autocomplete_core.js

```javascript
/** @odoo-module **/
/* global checkVATNumber */

import { loadJS } from "@web/core/assets";
import { _t } from "@web/core/l10n/translation";
import { KeepLast } from "@web/core/utils/concurrency";
import { useService } from "@web/core/utils/hooks";
import { renderToMarkup } from "@web/core/utils/render";
import { getDataURLFromFile } from "@web/core/utils/urls";
import { browser } from "@web/core/browser/browser";

/**
 * Get list of companies via Autocomplete API
 *
 * @param {string} value
 * @returns {Promise}
 * @private
 */
export function usePartnerAutocomplete() {
    const keepLastOdoo = new KeepLast();

    const notification = useService("notification");
    const orm = useService("orm");

    function sanitizeVAT(value) {
        return value ? value.replace(/[^A-Za-z0-9]/g, '') : '';
    }

    async function isVATNumber(value) {
        // Lazyload jsvat only if the component is being used.
        await loadJS("/partner_autocomplete/static/lib/jsvat.js");

        // checkVATNumber is defined in library jsvat.
        // It validates that the input has a valid VAT number format
        return checkVATNumber(sanitizeVAT(value));
    }

    function isGSTNumber(value) {
        // Check if the input is a valid GST number.
        let isGST = false;
        if (value && value.length === 15) {
            const allGSTinRe = [
                /\d{2}[a-zA-Z]{5}\d{4}[a-zA-Z][1-9A-Za-z][Zz1-9A-Ja-j][0-9a-zA-Z]/, // Normal, Composite, Casual GSTIN
                /\d{4}[A-Z]{3}\d{5}[UO]N[A-Z0-9]/, // UN/ON Body GSTIN
                /\d{4}[a-zA-Z]{3}\d{5}NR[0-9a-zA-Z]/, // NRI GSTIN
                /\d{2}[a-zA-Z]{4}[a-zA-Z0-9]\d{4}[a-zA-Z][1-9A-Za-z][DK][0-9a-zA-Z]/, // TDS GSTIN
                /\d{2}[a-zA-Z]{5}\d{4}[a-zA-Z][1-9A-Za-z]C[0-9a-zA-Z]/ // TCS GSTIN
            ];

            isGST = allGSTinRe.some((re) => re.test(value));
        }

        return isGST;
    }

    async function autocomplete(value, queryCountryId) {
        value = value.trim();
        const isVAT = await isVATNumber(value);
        if (isVAT){
            value = sanitizeVAT(value);
        }
        const isGST = isGSTNumber(value);
        return await getSuggestions(value, isVAT || isGST, queryCountryId);
    }

    /**
     * Get enrichment data
     *
     * @param {Object} company
     * @returns {Promise}
     * @private
     */
    function enrichCompany(company) {
        if (isGSTNumber(company.query)){
            return orm.call('res.partner', 'enrich_by_gst', [company.query]);
        }
        return orm.call('res.partner', 'enrich_by_duns', [company.duns]);
    }

    function removeUselessFields(company, fieldsToKeep) {
        // Delete attribute to avoid "Field_changed" errors (these fields will be populated in the form)
        for (const field in company){
            if (!fieldsToKeep.includes(field)){
                delete company[field]
            }
        }
        return company;
    };

    /**
     * Get enriched data + logo before populating partner form
     *
     * @param {Object} company
     * @returns {Promise}
     */
    function getCreateData(company, fieldsToKeep) {
        return new Promise((resolve) => {
            // Fetch additional company info via Autocomplete Enrichment API
            const enrichPromise = enrichCompany(company);

            // Get logo
            const logoPromise = company.logoUrl ? getCompanyLogo(company.logoUrl) : false;
            whenAll([enrichPromise, logoPromise]).then(([companyData, logoData]) => {

                if (companyData.error) {
                    if (companyData.error_message === 'Insufficient Credit') {
                        notifyNoCredits();
                    }
                    else if (companyData.error_message === 'No Account Token') {
                        notifyAccountToken();
                    }
                    else {
                        notification.add(companyData.error_message);
                    }
                    companyData = {
                        ...company,
                        ...companyData,
                    };
                }

                resolve({
                    company: companyData,
                    logo: logoData
                });
            })
        });
    }

    async function fetchNoCaching(url) {
        try {
            const response = await browser.fetch(
                url,
                {
                    method: 'GET',
                    cache: 'no-cache',
                }
            );
            return await response.json();
        } catch {
            return {}
        }
    }

    /**
     * Use Clearbit API to get the company logo if there is a match with the company name or domain
     *
     * @param {string} value
     * @returns {Promise}
     * @private
     */
    async function getClearbitLogoUrl(company) {
        let clearbitData = await fetchNoCaching(encodeURI(`https://autocomplete.clearbit.com/v1/companies/suggest?query=${company.name}`));
        if (!clearbitData.length) {
            if (company.domain) {
                clearbitData = await fetchNoCaching(encodeURI(`https://autocomplete.clearbit.com/v1/companies/suggest?query=${company.domain}`));
            }
            if (!clearbitData.length) {
                return '';
            }
        }
        const firstResult = clearbitData[0];
        if (
            firstResult.name.toLowerCase() === company.name.toLowerCase()
            ||
            (
                company.domain !== undefined
                &&
                firstResult.domain === company.domain
            )
        ){
            return firstResult.logo;
        }
        return '';
    }

    /**
     * Get the company logo as Base 64 image from url
     *
     * @param {string} url
     * @returns {Promise}
     * @private
     */
    async function getCompanyLogo(logoUrl) {
        try {
            if (!logoUrl) {
                return false;
            }
            const base64Image = await getBase64Image(logoUrl);
            // base64Image equals "data:" if image not available on given url
            return base64Image ? base64Image.replace(/^data:image[^;]*;base64,?/, '') : false;
        }
        catch {
            return false;
        }
    }

    /**
     * Returns a promise which will be resolved with the base64 data of the
     * image fetched from the given url.
     *
     * @private
     * @param {string} url : the url where to find the image to fetch
     * @returns {Promise}
     */
    function getBase64Image(url) {
        return new Promise((resolve, reject) => {
            const xhr = new XMLHttpRequest();
            xhr.onload = () => {
                getDataURLFromFile(xhr.response).then(resolve);
            };
            xhr.open('GET', url);
            xhr.responseType = 'blob';
            xhr.onerror = reject;
            xhr.send();
        });
    }

    /**
     * Use Odoo Autocomplete API to return suggestions
     *
     * @param {string} value
     * @param {boolean} isVAT
     * @returns {Promise}
     * @private
     */
    async function getSuggestions(value, isVAT, queryCountryId) {
        const method = isVAT ? 'autocomplete_by_vat' : 'autocomplete_by_name';

        const prom = orm.silent.call(
            'res.partner',
            method,
            [value, queryCountryId],
        );

        const suggestions = await keepLastOdoo.add(prom);
        await Promise.all(suggestions.map(async (suggestion) => {
            suggestion.query = value;  // Save queried value (name, VAT) for later
            suggestion.logoUrl = await getClearbitLogoUrl(suggestion);
            suggestion.description = '';
            if (suggestion.city){
                suggestion.description += suggestion.city;
            }
            // Show country name only if searching worldwide
            if (queryCountryId === 0 && suggestion.country_id && suggestion.country_id.display_name) {
                suggestion.description +=  ', ' + suggestion.country_id.display_name;
            }
            return suggestion;
        }));
        return suggestions;
    }

    /**
     * Utility to wait for multiple promises
     * Promise.all will reject all promises whenever a promise is rejected
     * This utility will continue
     *
     * @param {Promise[]} promises
     * @returns {Promise}
     * @private
     */
    function whenAll(promises) {
        return Promise.all(promises.map((p) => {
            return Promise.resolve(p);
        }));
    }

    /**
     * @private
     * @returns {Promise}
     */
    async function notifyNoCredits() {
        const url = await orm.call(
            'iap.account',
            'get_credits_url',
            ['partner_autocomplete'],
        );
        const title = _t('Not enough credits for Partner Autocomplete');
        const content = renderToMarkup('partner_autocomplete.InsufficientCreditNotification', {
            credits_url: url
        });
        notification.add(content, {
            title,
        });
    }

    async function notifyAccountToken() {
        const url = await orm.call(
            'iap.account',
            'get_config_account_url',
            []
        );
        const title = _t('IAP Account Token missing');
        if (url) {
            const content = renderToMarkup('partner_autocomplete.AccountTokenMissingNotification', {
                account_url: url
            });
            notification.add(content, {
                title,
            });
        }
        else {
            notification.add(title);
        }
    }
    return { autocomplete, getCreateData, removeUselessFields };
}

```

## File: static\src\js\partner_autocomplete_fieldchar.js

```javascript
/** @odoo-module **/

import { useChildRef, useService } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";
import { _t } from "@web/core/l10n/translation";
import { CharField, charField } from "@web/views/fields/char/char_field";
import { useInputField } from "@web/views/fields/input_field_hook";

import { usePartnerAutocomplete } from "@partner_autocomplete/js/partner_autocomplete_core";
import { PartnerAutoComplete } from "@partner_autocomplete/js/partner_autocomplete_component";

export class PartnerAutoCompleteCharField extends CharField {
    static template = "partner_autocomplete.PartnerAutoCompleteCharField";
    static components = {
        ...CharField.components,
        PartnerAutoComplete,
    };
    setup() {
        super.setup();

        this.orm = useService("orm");
        this.partnerAutocomplete = usePartnerAutocomplete();

        this.inputRef = useChildRef();
        useInputField({ getValue: () => this.props.record.data[this.props.name] || "", parse: (v) => this.parse(v), ref: this.inputRef});
    }

    async validateSearchTerm(request) {
        return request && request.length > 2;
    }

    get sources() {
        return [
            {
                options: async (request, shouldSearchWorldWide) => {
                    if (await this.validateSearchTerm(request)) {
                        let queryCountryId = this.props.record.data?.country_id ? this.props.record.data.country_id[0] : false;
                        if (shouldSearchWorldWide){
                            queryCountryId = 0;
                        }
                        const suggestions = await this.partnerAutocomplete.autocomplete(request, queryCountryId);
                        suggestions.forEach((suggestion) => {
                            suggestion.classList = "partner_autocomplete_dropdown_char";
                        });
                        return suggestions;
                    }
                    else {
                        return [];
                    }
                },
                optionTemplate: "partner_autocomplete.DropdownOption",
                placeholder: _t('Searching Autocomplete...'),
            },
        ];
    }

    async onSelect(option) {
        let data = await this.partnerAutocomplete.getCreateData(Object.getPrototypeOf(option));
        if (!data?.company) {
            return;
        }

        if (data.logo) {
            const logoField = this.props.record.resModel === 'res.partner' ? 'image_1920' : 'logo';
            data.company[logoField] = data.logo;
        }

        // Format the many2one fields
        const many2oneFields = ['country_id', 'state_id'];
        many2oneFields.forEach((field) => {
            if (data.company[field]) {
                data.company[field] = [data.company[field].id, data.company[field].display_name];
            }
        });

        // Save UNSPSC codes (tags)
        const unspsc_codes = data.company.unspsc_codes

        // Delete useless fields before updating record
        data.company = this.partnerAutocomplete.removeUselessFields(data.company, Object.keys(this.props.record.fields));

        // Update record with retrieved values
        if (data.company.name) {
            await this.props.record.update({name: data.company.name});  // Needed otherwise name it is not saved
        }
        await this.props.record.update(data.company);

        // Add UNSPSC codes (tags)
        if (this.props.record.resModel === 'res.partner' && unspsc_codes) {
            // We must first save the record so that we can then create the tags (many2many)
            const saved = await this.props.record.save();
            if (saved){
                await this.props.record.load();
                await this.orm.call("res.partner", "iap_partner_autocomplete_add_tags", [this.props.record.resId, unspsc_codes]);
                await this.props.record.load();
            }
        }
        if (this.props.setDirty) {
            this.props.setDirty(false);
        }
    }
}

PartnerAutoCompleteCharField.template = "partner_autocomplete.PartnerAutoCompleteCharField";
PartnerAutoCompleteCharField.components = {
    ...CharField.components,
    PartnerAutoComplete,
};

export const partnerAutoCompleteCharField = {
    ...charField,
    component: PartnerAutoCompleteCharField,
};

registry.category("fields").add("field_partner_autocomplete", partnerAutoCompleteCharField);

```

## File: static\src\js\partner_autocomplete_many2one.js

```javascript
/** @odoo-module **/

import { Many2XAutocomplete } from '@web/views/fields/relational_utils';
import { Many2OneField, many2OneField } from '@web/views/fields/many2one/many2one_field';
import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";

import { usePartnerAutocomplete } from "@partner_autocomplete/js/partner_autocomplete_core";
import { PartnerAutoComplete } from "@partner_autocomplete/js/partner_autocomplete_component";

export class PartnerMany2XAutocomplete extends Many2XAutocomplete {
    static template = "partner_autocomplete.PartnerAutoCompleteMany2XField";
    static components = {
        ...Many2XAutocomplete.components,
        PartnerAutoComplete,
    };

    setup() {
        super.setup();
        this.partnerAutocomplete = usePartnerAutocomplete();
    }

    validateSearchTerm(request) {
        return request && request.length > 2;
    }

    get sources() {
        const sources = super.sources;
        if (!this.props.canCreate)
        {
            return sources;
        }
        return sources.concat(
            {
                options: async (request, shouldSearchWorldWide) => {
                    if (this.validateSearchTerm(request)) {
                        let queryCountryId = false;
                        if (shouldSearchWorldWide){
                            queryCountryId = 0;
                        }
                        const suggestions = await this.partnerAutocomplete.autocomplete(request, queryCountryId);
                        suggestions.forEach((suggestion) => {
                            suggestion.classList = "partner_autocomplete_dropdown_many2one";
                            suggestion.isFromPartnerAutocomplete = true;
                        });
                        return suggestions;
                    }
                    else {
                        return [];
                    }
                },
                optionTemplate: "partner_autocomplete.DropdownOption",
                placeholder: _t("Searching Autocomplete..."),
            },
        );
    }

    async onSelect(option, params) {
        if (option.isFromPartnerAutocomplete) {  // Checks that it is a partner autocomplete option
            const data = await this.partnerAutocomplete.getCreateData(Object.getPrototypeOf(option));
            if (!data?.company) {
                return;
            }
            let context = {
                'default_is_company': true
            };

            for (const [key, val] of Object.entries(data.company)) {
                context['default_' + key] = val && val.id ? val.id : val;
            }

            if (data.logo) {
                context.default_image_1920 = data.logo;
            }
            return this.openMany2X({ context });
        }
        else {
            return super.onSelect(option, params);
        }
    }

}

PartnerMany2XAutocomplete.props = {
    ...Many2XAutocomplete.props,
    canCreate: { type: Boolean, optional: true },
}

export class PartnerAutoCompleteMany2one extends Many2OneField {
    get Many2XAutocompleteProps() {
        return {
            ...super.Many2XAutocompleteProps,
            canCreate: this.props.canCreate,
        };
    }
}

PartnerAutoCompleteMany2one.components = {
    ...Many2OneField.components,
    Many2XAutocomplete: PartnerMany2XAutocomplete,
}

export const partnerAutoCompleteMany2one = {
    ...many2OneField,
    component: PartnerAutoCompleteMany2one,
};

registry.category("fields").add("res_partner_many2one", partnerAutoCompleteMany2one);

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
    <t t-name="partner_autocomplete.PartnerAutoComplete" t-inherit="web.AutoComplete" t-inherit-mode="primary">
        <xpath expr="//ul" position="inside">
            <t t-if="this.sources &amp;&amp; this.sources.at(-1)?.options.length !== 0 &amp;&amp; !this.shouldSearchWorldwide">  <!-- 1 because IAP sources (potential partners) are loaded after DB sources (existing partners)-->
                <li
                    class="o-autocomplete--dropdown-item ui-menu-item d-block"
                    t-on-pointerdown="() => this.ignoreBlur = true"
                    t-on-mouseleave="() => this.onOptionMouseLeave()"
                    t-on-click="(ev) => this.searchWorldwide(ev)"
                >
                    <a
                        t-attf-id="{{props.id or 'autocomplete'}}_{{source_index}}_{{option_index}}"
                        role="option"
                        href="#"
                        class="dropdown-item ui-menu-item-wrapper text-truncate text-info"
                        t-att-class="{ 'ui-state-active': isActiveSourceOption([source_index, option_index]) }"
                        t-att-aria-selected="isActiveSourceOption([source_index, option_index]) ? 'true' : 'false'"
                    >
                        Search Worldwide 🌎
                    </a>
                </li>
            </t>
        </xpath>
    </t>

    <t t-name="partner_autocomplete.PartnerAutoCompleteCharField" t-inherit="web.CharField" t-inherit-mode="primary">
        <xpath expr="//t[@t-else='']" position="before">
            <t t-elif="props.record.resModel !== 'res.partner' || props.record.data.company_type === 'company'">
                <PartnerAutoComplete
                    value="props.record.data[props.name] || ''"
                    sources="sources"
                    onSelect.bind="onSelect"
                    input="inputRef"
                    placeholder="props.placeholder || ''"
                />
            </t>
        </xpath>
    </t>

    <t t-name="partner_autocomplete.PartnerAutoCompleteMany2XField" t-inherit="web.Many2XAutocomplete" t-inherit-mode="primary">
        <xpath expr="//AutoComplete" position="replace">
            <PartnerAutoComplete
                value="this.props.value || ''"
                autoSelect="true"
                sources="sources"
                onSelect.bind="onSelect"
                onChange.bind="onChange"
                input="inputRef"
                placeholder="placeholder || ''"
            />
        </xpath>
    </t>

    <t t-name="partner_autocomplete.DropdownOption">
        <div style="display: flex; justify-content: space-between; align-items: center; height: 24px;">
            <div>
                <span t-esc="option.name"/>,
                <span class="text-muted" t-esc="option.description"/>
            </div>
            <img style="width: 24px; max-height: 100%" class="ms-1" t-att-src="option.logoUrl" onerror="this.style.display='none'"/>
        </div>
    </t>

    <t t-name="partner_autocomplete.InsufficientCreditNotification">
        <div class="o-hidden-ios">
            <a class="btn btn-link" t-att-href="credits_url"><i class="oi oi-arrow-right"/> Buy more credits</a>
        </div>
    </t>

    <t t-name="partner_autocomplete.AccountTokenMissingNotification">
        <div class="">
            <a class="btn btn-link" t-att-href="account_url" ><i class="oi oi-arrow-right"/> Set Your Account Token</a>
        </div>
    </t>
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
            <setting id="partner_autocomplete" position="inside">
                <widget name="iap_buy_more_credits" service_name="partner_autocomplete" hide_service="1"/>
            </setting>
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
            <xpath expr="//field[last()]" position="after">
                <field name="partner_gid" invisible="True"/>
                <field name="additional_info" invisible="True"/>
            </xpath>
        </field>
    </record>
</odoo>

```

