# Odoo Module: base_geolocalize

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
    'name': 'Partners Geolocation',
    'version': '2.1',
    'category': 'Hidden/Tools',
    'description': """
Partners Geolocation
========================
    """,
    'depends': ['base_setup'],
    'data': [
        'security/ir.model.access.csv',
        'views/geo_provider_view.xml',
        'views/res_partner_views.xml',
        'views/res_config_settings_views.xml',
        'data/data.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="geoprovider_open_street" model="base.geo_provider">
        <field name="tech_name">openstreetmap</field>
        <field name="name">Open Street Map</field>
    </record>

    <record id="geoprovider_google_map" model="base.geo_provider">
        <field name="tech_name">googlemap</field>
        <field name="name">Google Place Map</field>
    </record>
</odoo>

```

## File: models\base_geocoder.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import requests
import logging

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError


_logger = logging.getLogger(__name__)


class GeoProvider(models.Model):
    _name = "base.geo_provider"
    _description = "Geo Provider"

    tech_name = fields.Char(string="Technical Name")
    name = fields.Char()


class GeoCoder(models.AbstractModel):
    """
    Abstract class used to call Geolocalization API and convert addresses
    into GPS coordinates.
    """
    _name = "base.geocoder"
    _description = "Geo Coder"

    @api.model
    def _get_provider(self):
        prov_id = self.env['ir.config_parameter'].sudo().get_param('base_geolocalize.geo_provider')
        if prov_id:
            provider = self.env['base.geo_provider'].browse(int(prov_id))
        if not prov_id or not provider.exists():
            provider = self.env['base.geo_provider'].search([], limit=1)
        return provider

    @api.model
    def geo_query_address(self, street=None, zip=None, city=None, state=None, country=None):
        """ Converts address fields into a valid string for querying
        geolocation APIs.
        :param street: street address
        :param zip: zip code
        :param city: city
        :param state: state
        :param country: country
        :return: formatted string
        """
        provider = self._get_provider().tech_name
        if hasattr(self, '_geo_query_address_' + provider):
            # Makes the transformation defined for provider
            return getattr(self, '_geo_query_address_' + provider)(street, zip, city, state, country)
        else:
            # By default, join the non-empty parameters
            return self._geo_query_address_default(street=street, zip=zip, city=city, state=state, country=country)

    @api.model
    def geo_find(self, addr, **kw):
        """Use a location provider API to convert an address string into a latitude, longitude tuple.
        Here we use Openstreetmap Nominatim by default.
        :param addr: Address string passed to API
        :return: (latitude, longitude) or None if not found
        """
        provider = self._get_provider().tech_name
        try:
            service = getattr(self, '_call_' + provider)
            result = service(addr, **kw)
        except AttributeError:
            raise UserError(_(
                'Provider %s is not implemented for geolocation service.',
                provider))
        except UserError:
            raise
        except Exception:
            _logger.debug('Geolocalize call failed', exc_info=True)
            result = None
        return result

    @api.model
    def _call_openstreetmap(self, addr, **kw):
        """
        Use Openstreemap Nominatim service to retrieve location
        :return: (latitude, longitude) or None if not found
        """
        if not addr:
            _logger.info('invalid address given')
            return None
        url = 'https://nominatim.openstreetmap.org/search'
        try:
            headers = {'User-Agent': 'Odoo (http://www.odoo.com/contactus)'}
            response = requests.get(url, headers=headers, params={'format': 'json', 'q': addr})
            _logger.info('openstreetmap nominatim service called')
            if response.status_code != 200:
                _logger.warning('Request to openstreetmap failed.\nCode: %s\nContent: %s', response.status_code, response.content)
            result = response.json()
        except Exception as e:
            self._raise_query_error(e)
        geo = result[0]
        return float(geo['lat']), float(geo['lon'])

    @api.model
    def _call_googlemap(self, addr, **kw):
        """ Use google maps API. It won't work without a valid API key.
        :return: (latitude, longitude) or None if not found
        """
        apikey = self.env['ir.config_parameter'].sudo().get_param('base_geolocalize.google_map_api_key')
        if not apikey:
            raise UserError(_(
                "API key for GeoCoding (Places) required.\n"
                "Visit https://developers.google.com/maps/documentation/geocoding/get-api-key for more information."
            ))
        url = "https://maps.googleapis.com/maps/api/geocode/json"
        params = {'sensor': 'false', 'address': addr, 'key': apikey}
        if kw.get('force_country'):
            params['components'] = 'country:%s' % kw['force_country']
        try:
            result = requests.get(url, params).json()
        except Exception as e:
            self._raise_query_error(e)

        try:
            if result['status'] == 'ZERO_RESULTS':
                return None
            if result['status'] != 'OK':
                _logger.debug('Invalid Gmaps call: %s - %s',
                              result['status'], result.get('error_message', ''))
                error_msg = _('Unable to geolocate, received the error:\n%s'
                              '\n\nGoogle made this a paid feature.\n'
                              'You should first enable billing on your Google account.\n'
                              'Then, go to Developer Console, and enable the APIs:\n'
                              'Geocoding, Maps Static, Maps Javascript.\n', result.get('error_message'))
                raise UserError(error_msg)
            geo = result['results'][0]['geometry']['location']
            return float(geo['lat']), float(geo['lng'])
        except (KeyError, ValueError):
            _logger.debug('Unexpected Gmaps API answer %s', result.get('error_message', ''))
            return None

    @api.model
    def _geo_query_address_default(self, street=None, zip=None, city=None, state=None, country=None):
        address_list = [
            street,
            ("%s %s" % (zip or '', city or '')).strip(),
            state,
            country
        ]
        address_list = [item for item in address_list if item]
        return tools.ustr(', '.join(address_list))

    @api.model
    def _geo_query_address_googlemap(self, street=None, zip=None, city=None, state=None, country=None):
        # put country qualifier in front, otherwise GMap gives wrong# results
        #  e.g. 'Congo, Democratic Republic of the' =>  'Democratic Republic of the Congo'
        if country and ',' in country and (
                country.endswith(' of') or country.endswith(' of the')):
            country = '{1} {0}'.format(*country.split(',', 1))
        return self._geo_query_address_default(street=street, zip=zip, city=city, state=state, country=country)

    def _raise_query_error(self, error):
        raise UserError(_('Error with geolocation server:') + ' %s' % error)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    geoloc_provider_id = fields.Many2one(
        'base.geo_provider',
        string='API',
        config_parameter='base_geolocalize.geo_provider',
        default=lambda x: x.env['base.geocoder']._get_provider()
    )
    geoloc_provider_techname = fields.Char(related='geoloc_provider_id.tech_name', readonly=True)
    geoloc_provider_googlemap_key = fields.Char(
        string='Google Map API Key',
        config_parameter='base_geolocalize.google_map_api_key',
        help="Visit https://developers.google.com/maps/documentation/geocoding/get-api-key for more information."
    )

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import config


class ResPartner(models.Model):
    _inherit = "res.partner"

    date_localization = fields.Date(string='Geolocation Date')

    def write(self, vals):
        # Reset latitude/longitude in case we modify the address without
        # updating the related geolocation fields
        if any(field in vals for field in ['street', 'zip', 'city', 'state_id', 'country_id']) \
                and not all('partner_%s' % field in vals for field in ['latitude', 'longitude']):
            vals.update({
                'partner_latitude': 0.0,
                'partner_longitude': 0.0,
            })
        return super().write(vals)

    @api.model
    def _geo_localize(self, street='', zip='', city='', state='', country=''):
        geo_obj = self.env['base.geocoder']
        search = geo_obj.geo_query_address(street=street, zip=zip, city=city, state=state, country=country)
        result = geo_obj.geo_find(search, force_country=country)
        if result is None:
            search = geo_obj.geo_query_address(city=city, state=state, country=country)
            result = geo_obj.geo_find(search, force_country=country)
        return result

    def geo_localize(self):
        # We need country names in English below
        if not self._context.get('force_geo_localize') \
                and (self._context.get('import_file') \
                     or any(config[key] for key in ['test_enable', 'test_file', 'init', 'update'])):
            return False
        partners_not_geo_localized = self.env['res.partner']
        for partner in self.with_context(lang='en_US'):
            result = self._geo_localize(partner.street,
                                        partner.zip,
                                        partner.city,
                                        partner.state_id.name,
                                        partner.country_id.name)

            if result:
                partner.write({
                    'partner_latitude': result[0],
                    'partner_longitude': result[1],
                    'date_localization': fields.Date.context_today(partner)
                })
            else:
                partners_not_geo_localized |= partner
        if partners_not_geo_localized:
            self.env['bus.bus']._sendone(self.env.user.partner_id, 'simple_notification', {
                'type': 'danger',
                'title': _("Warning"),
                'message': _('No match found for %(partner_names)s address(es).', partner_names=', '.join(partners_not_geo_localized.mapped('name')))
            })
        return True

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import base_geocoder
from . import res_config_settings
from . import res_partner

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_base_geo_provider,access_base_geo_provider,model_base_geo_provider,base.group_user,1,0,0,0

```

## File: views\geo_provider_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="view_geo_provider_form" model="ir.ui.view">
            <field name="name">base.geo_provider.form</field>
            <field name="model">base.geo_provider</field>
            <field name="arch" type="xml">
                <form create="0" delete="0">
                    <sheet>
                        <group>
                            <group>
                                <field name="name"/>
                            </group>
                            <group>
                                <field name="tech_name"/>
                            </group>
                        </group>
                    </sheet>
                </form>
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
        <field name="name">res.config.settings.view.form.inherit.web.geoloclaize</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='base_geolocalize_warning']" position="replace">
                <div id="geolocalize_params"
                    class="content-group"
                    invisible="not module_base_geolocalize">
                    API: <field name="geoloc_provider_id"/>
                    <field name="geoloc_provider_techname" invisible="1"/>
                    <div invisible="geoloc_provider_techname != 'googlemap'">
                        Key: <field name='geoloc_provider_googlemap_key'/>
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
        <record id="view_crm_partner_geo_form" model="ir.ui.view">
            <field name="name">res.partner.geolocation.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <xpath expr="//notebook[last()]" position="inside">
                    <page string="Partner Assignment" name="geo_location">
                        <group>
                            <group string="Geolocation">
                                <label for="date_localization" string="Geo Location"/>
                                <div>
                                    <span>Lat : <field name="partner_latitude" nolabel="1" class="oe_inline"/></span>
                                    <br/>
                                    <span>Long: <field name="partner_longitude" nolabel="1" class="oe_inline"/></span>
                                    <br/>
                                    <span invisible="not date_localization">Updated on:
                                        <field name="date_localization" nolabel="1" readonly="1" class="oe_inline"/>
                                        <br/>
                                    </span>
                                    <div name="geo_localize_button">
                                        <button invisible="partner_latitude != 0 or partner_longitude != 0"
                                                icon="fa-gear" string="Compute based on address" title="Compute Localization"
                                                name="geo_localize" type="object" class="btn btn-link p-0"/>
                                        <button invisible="partner_latitude == 0 and partner_longitude == 0"
                                                icon="fa-refresh" string="Refresh" title="Refresh Localization"
                                                name="geo_localize" type="object" class="btn btn-link p-0"/>
                                    </div>
                                </div>
                            </group>
                        </group>
                    </page>
                </xpath>
            </field>
        </record>
</odoo>

```

