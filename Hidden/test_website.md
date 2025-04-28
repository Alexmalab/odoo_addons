# Odoo Module: test_website

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Website Test',
    'version': '1.0',
    'category': 'Hidden',
    'sequence': 9876,
    'summary': 'Website Test, mainly for module install/uninstall tests',
    'description': """This module contains tests related to website. Those are
present in a separate module as we are testing module install/uninstall/upgrade
and we don't want to reload the website module every time, including it's possible
dependencies. Neither we want to add in website module some routes, views and
models which only purpose is to run tests.""",
    'depends': [
        'web_unsplash',
        'website',
        'theme_default',
    ],
    'demo': [
        'data/test_website_demo.xml',
    ],
    'data': [
        'security/test_website_security.xml',
        'security/ir.model.access.csv',
        'views/templates.xml',
        'views/test_model_multi_website_views.xml',
        'views/test_model_views.xml',
        'data/test_website_data.xml',
    ],
    'installable': True,
    'assets': {
        'test_website.test_bundle': [
            'http://test.external.link/javascript1.js',
            '/web/static/src/libs/fontawesome/css/font-awesome.css',
            'http://test.external.link/style1.css',
            '/web/static/src/module_loader.js',
            'http://test.external.link/javascript2.js',
            'http://test.external.link/style2.css',
        ],
        'web.assets_frontend': [
            'test_website/static/src/js/test_error.js',
        ],
        'web.assets_tests': [
            'test_website/static/tests/tours/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import werkzeug

from odoo import http
from odoo.http import request
from odoo.addons.portal.controllers.web import Home
from odoo.exceptions import UserError, ValidationError, AccessError, MissingError, AccessDenied


class WebsiteTest(Home):

    @http.route('/test_view', type='http', auth='public', website=True, sitemap=False)
    def test_view(self, **kwargs):
        return request.render('test_website.test_view')

    @http.route('/ignore_args/converteronly/<string:a>', type='http', auth="public", website=True, sitemap=False)
    def test_ignore_args_converter_only(self, a):
        return request.make_response(json.dumps(dict(a=a, kw=None)))

    @http.route('/ignore_args/none', type='http', auth="public", website=True, sitemap=False)
    def test_ignore_args_none(self):
        return request.make_response(json.dumps(dict(a=None, kw=None)))

    @http.route('/ignore_args/a', type='http', auth="public", website=True, sitemap=False)
    def test_ignore_args_a(self, a):
        return request.make_response(json.dumps(dict(a=a, kw=None)))

    @http.route('/ignore_args/kw', type='http', auth="public", website=True, sitemap=False)
    def test_ignore_args_kw(self, a, **kw):
        return request.make_response(json.dumps(dict(a=a, kw=kw)))

    @http.route('/ignore_args/converter/<string:a>', type='http', auth="public", website=True, sitemap=False)
    def test_ignore_args_converter(self, a, b='youhou', **kw):
        return request.make_response(json.dumps(dict(a=a, b=b, kw=kw)))

    @http.route('/ignore_args/converter/<string:a>/nokw', type='http', auth="public", website=True, sitemap=False)
    def test_ignore_args_converter_nokw(self, a, b='youhou'):
        return request.make_response(json.dumps(dict(a=a, b=b)))

    @http.route('/multi_company_website', type='http', auth="public", website=True, sitemap=False)
    def test_company_context(self):
        return request.make_response(json.dumps(request.context.get('allowed_company_ids')))

    @http.route('/test_lang_url/<model("res.country"):country>', type='http', auth='public', website=True, sitemap=False)
    def test_lang_url(self, **kwargs):
        return request.render('test_website.test_view')

    # Test Session

    @http.route('/test_get_dbname', type='json', auth='public', website=True, sitemap=False)
    def test_get_dbname(self, **kwargs):
        return request.env.cr.dbname

    # Test Error

    @http.route('/test_error_view', type='http', auth='public', website=True, sitemap=False)
    def test_error_view(self, **kwargs):
        return request.render('test_website.test_error_view')

    @http.route('/test_user_error_http', type='http', auth='public', website=True, sitemap=False)
    def test_user_error_http(self, **kwargs):
        raise UserError("This is a user http test")

    @http.route('/test_user_error_json', type='json', auth='public', website=True, sitemap=False)
    def test_user_error_json(self, **kwargs):
        raise UserError("This is a user rpc test")

    @http.route('/test_validation_error_http', type='http', auth='public', website=True, sitemap=False)
    def test_validation_error_http(self, **kwargs):
        raise ValidationError("This is a validation http test")

    @http.route('/test_validation_error_json', type='json', auth='public', website=True, sitemap=False)
    def test_validation_error_json(self, **kwargs):
        raise ValidationError("This is a validation rpc test")

    @http.route('/test_access_error_json', type='json', auth='public', website=True, sitemap=False)
    def test_access_error_json(self, **kwargs):
        raise AccessError("This is an access rpc test")

    @http.route('/test_access_error_http', type='http', auth='public', website=True, sitemap=False)
    def test_access_error_http(self, **kwargs):
        raise AccessError("This is an access http test")

    @http.route('/test_missing_error_json', type='json', auth='public', website=True, sitemap=False)
    def test_missing_error_json(self, **kwargs):
        raise MissingError("This is a missing rpc test")

    @http.route('/test_missing_error_http', type='http', auth='public', website=True, sitemap=False)
    def test_missing_error_http(self, **kwargs):
        raise MissingError("This is a missing http test")

    @http.route('/test_internal_error_json', type='json', auth='public', website=True, sitemap=False)
    def test_internal_error_json(self, **kwargs):
        raise werkzeug.exceptions.InternalServerError()

    @http.route('/test_internal_error_http', type='http', auth='public', website=True, sitemap=False)
    def test_internal_error_http(self, **kwargs):
        raise werkzeug.exceptions.InternalServerError()

    @http.route('/test_access_denied_json', type='json', auth='public', website=True, sitemap=False)
    def test_denied_error_json(self, **kwargs):
        raise AccessDenied("This is an access denied rpc test")

    @http.route('/test_access_denied_http', type='http', auth='public', website=True, sitemap=False)
    def test_denied_error_http(self, **kwargs):
        raise AccessDenied("This is an access denied http test")

    @http.route(['/get'], type='http', auth="public", methods=['GET'], website=True, sitemap=False)
    def get_method(self, **kw):
        return request.make_response('get')

    @http.route(['/post'], type='http', auth="public", methods=['POST'], website=True, sitemap=False)
    def post_method(self, **kw):
        return request.make_response('post')

    @http.route(['/get_post'], type='http', auth="public", methods=['GET', 'POST'], website=True, sitemap=False)
    def get_post_method(self, **kw):
        return request.make_response('get_post')

    @http.route(['/get_post_nomultilang'], type='http', auth="public", methods=['GET', 'POST'], website=True, multilang=False, sitemap=False)
    def get_post_method_no_multilang(self, **kw):
        return request.make_response('get_post_nomultilang')

    # Test Perfs

    @http.route(['/empty_controller_test'], type='http', auth='public', website=True, multilang=False, sitemap=False)
    def empty_controller_test(self, **kw):
        return 'Basic Controller Content'

    # Test Redirects
    @http.route(['/test_website/country/<model("res.country"):country>'], type='http', auth="public", website=True, sitemap=True)
    def test_model_converter_country(self, country, **kw):
        return request.render('test_website.test_redirect_view', {'country': country})

    @http.route(['/test_website/200/<model("test.model"):rec>'], type='http', auth="public", website=True, sitemap=False)
    def test_model_converter_seoname(self, rec, **kw):
        return request.make_response('ok')

    @http.route(['/test_website/model_item/<int:record_id>'], type='http', methods=['GET'], auth="public", website=True, sitemap=False)
    def test_model_item(self, record_id):
        record = request.env['test.model'].browse(record_id)
        values = {
            'record': record,
            'main_object': record,
            'tag': record.tag_id,
        }
        return request.render("test_website.model_item", values)

    @http.route(['/test_website/model_item_sudo/<int:record_id>'], type='http', methods=['GET'], auth="public", website=True, sitemap=False)
    def test_model_item_sudo(self, record_id):
        values = {
            'record': request.env['test.model'].sudo().browse(record_id),
        }
        return request.render("test_website.model_item", values)

    @http.route(['/test_website/test_redirect_view_qs'], type='http', auth="public", website=True, sitemap=False)
    def test_redirect_view_qs(self, **kw):
        return request.render('test_website.test_redirect_view_qs')

    @http.route([
        '/test_countries_308',
        '/test_countries_308/<model("test.model"):rec>',
    ], type='http', auth='public', website=True, sitemap=False)
    def test_countries_308(self, **kwargs):
        return request.make_response('ok')

    # Test Sitemap
    def sitemap_test(env, rule, qs):
        if not qs or qs.lower() in '/test_website_sitemap':
            yield {'loc': '/test_website_sitemap'}

    @http.route([
        '/test_website_sitemap',
        '/test_website_sitemap/something/<model("test.model"):rec>',
    ], type='http', auth='public', website=True, sitemap=sitemap_test)
    def test_sitemap(self, rec=None, **kwargs):
        return request.make_response('Sitemap Testing Page')

    @http.route('/test_model/<model("test.model"):test_model>', type='http', auth='public', website=True, sitemap=False)
    def test_model(self, test_model, **kwargs):
        return request.render('test_website.test_model_page_layout', {'main_object': test_model, 'test_model': test_model})

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\test_website_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="test_model_publish" model="ir.rule">
            <field name="name">Public user: read only website published</field>
            <field name="model_id" ref="test_website.model_test_model"/>
            <field name="groups" eval="[(4, ref('base.group_public'))]"/>
            <field name="domain_force">[('website_published','=', True)]</field>
            <field name="perm_read" eval="True"/>
        </record>

        <!-- SOME DEFAULT TEST.MODEL RECORDS WITH DIFFERENT WEBSITE_ID -->
        <record id="test_tag_generic" model="test.tag">
            <field name="name">Test Tag</field>
        </record>
        <record id="test_tag_2" model="test.tag">
            <field name="name">Test Tag #2</field>
        </record>
        <record id="test_tag_3" model="test.tag">
            <field name="name">Test Tag #3</field>
        </record>
        <record id="test_model_generic" model="test.model">
            <field name="name">Test Model</field>
            <field name="tag_id" ref="test_website.test_tag_generic"/>
        </record>
        <record id="test_submodel_generic" model="test.submodel">
            <field name="name">Test Submodel</field>
            <field name="test_model_id" ref="test_website.test_model_generic"/>
            <field name="tag_id" ref="test_website.test_tag_generic"/>
        </record>
        <record id="test_model_multi_generic" model="test.model.multi.website">
            <field name="name">Test Multi Model Generic</field>
        </record>
        <record id="test_model_multi_website_1" model="test.model.multi.website">
            <field name="name">Test Multi Model Website 1</field>
            <field name="website_id" ref="website.default_website"/>
        </record>

        <!-- RECORDS FOR RESET VIEWS TESTS -->
        <record id="test_view" model="ir.ui.view">
            <field name="name">Test View</field>
            <field name="type">qweb</field>
            <field name="key">test_website.test_view</field>
            <field name="arch" type="xml">
                <t name="Test View" priority="29" t-name="test_website.test_view">
                    <t t-call="website.layout">
                        <p>Test View</p>
                        <p>placeholder</p>
                    </t>
                </t>
            </field>
        </record>
        <record id="test_page_view" model="ir.ui.view">
            <field name="name">Test Page View</field>
            <field name="type">qweb</field>
            <field name="key">test_website.test_page_view</field>
            <field name="arch" type="xml">
                <t name="Test Page View" priority="29" t-name="test_website.test_page_view">
                    <t t-call="website.layout">
                        <div id="oe_structure_test_website_page" class="oe_structure oe_empty"/>
                        <p>Test Page View</p>
                        <p>placeholder</p>
                    </t>
                </t>
            </field>
        </record>
        <record id="test_error_view" model="ir.ui.view">
            <field name="name">Test Error View</field>
            <field name="type">qweb</field>
            <field name="key">test_website.test_error_view</field>
            <field name="arch" type="xml">
                <t name="Test Error View" t-name="test_website.test_error_view">
                    <t t-call="website.layout">
                    <div class="container">
                        <h1>Test Error View</h1>
                        <div class="row">
                            <ul class="list-group http_error col-6">
                                <li class="list-group-item list-group-item-primary"><h2>http Errors</h2></li>
                                <li class="list-group-item"><a href="/test_user_error_http">http UserError (400)</a></li>
                                <li class="list-group-item"><a href="/test_validation_error_http">http ValidationError (400)</a></li>
                                <li class="list-group-item"><a href="/test_missing_error_http">http MissingError (400)</a></li>
                                <li class="list-group-item"><a href="/test_access_error_http">http AccessError (403)</a></li>
                                <li class="list-group-item"><a href="/test_access_denied_http">http AccessDenied (403)</a></li>
                                <li class="list-group-item"><a href="/test_internal_error_http">http InternalServerError (500)</a></li>
                                <li class="list-group-item"><a href="/test_not_found_http">http NotFound (404)</a></li>
                            </ul>
                            <ul class="list-group rpc_error col-6">
                                <li class="list-group-item list-group-item-primary"><h2>rpc Warnings</h2></li>
                                <li class="list-group-item"><a href="/test_user_error_json">rpc UserError</a></li>
                                <li class="list-group-item"><a href="/test_validation_error_json">rpc ValidationError</a></li>
                                <li class="list-group-item"><a href="/test_missing_error_json">rpc MissingError</a></li>
                                <li class="list-group-item"><a href="/test_access_error_json">rpc AccessError</a></li>
                                <li class="list-group-item"><a href="/test_access_denied_json">rpc AccessDenied</a></li>
                                <li class="list-group-item list-group-item-primary"><h2>rpc Errors</h2></li>
                                <li class="list-group-item"><a href="/test_internal_error_json">rpc InternalServerError</a></li>
                            </ul>
                        </div>
                    </div>
                    </t>
                </t>
            </field>
        </record>
        <record id="test_page" model="website.page">
            <field name="is_published">True</field>
            <field name="url">/test_page_view</field>
            <field name="view_id" ref="test_page_view"/>
            <field name="website_indexed" eval="False"/>
        </record>
        <record id="test_view_to_be_t_called" model="ir.ui.view">
            <field name="name">Test View To Be t-called</field>
            <field name="type">qweb</field>
            <field name="key">test_website.test_view_to_be_t_called</field>
            <field name="arch" type="xml">
                <t name="Test View To Be t-called" priority="29" t-name="test_website.test_view_to_be_t_called">
                    <p>Test View To Be t-called</p>
                    <p>placeholder</p>
                </t>
            </field>
        </record>
        <template id="test_view_child_broken" inherit_id="test_website.test_view" active="False">
            <xpath expr="//p[last()]" position="replace">
                <p>Test View Child Broken</p>
                <p>placeholder</p>
            </xpath>
        </template>

        <!-- RECORDS FOR MODULE OPERATION TESTS -->
        <template id="update_module_base_view">
            <div>I am a base view</div>
        </template>

        <!-- RECORDS FOR REDIRECT TESTS -->
        <template id="test_redirect_view">
            <t t-esc="country.name"/>
            <t t-if="not request.env.user._is_public()" t-esc="'Logged In'"/>
            <!-- `href` is send through `url_for` for non editor users -->
            <a href="/test_website/country/andorra-1">I am a link</a>
        </template>
        <template id="test_redirect_view_qs">
            <a href="/empty_controller_test?a=a">Home</a>
        </template>

        <record id="test_image_progress" model="website.page">
            <field name="name">Test Image Progress</field>
            <field name="url">/test_image_progress</field>
            <field name="type">qweb</field>
            <field name="key">test_website.test_image_progress</field>
            <field name="arch" type="xml">
                <t t-call="website.layout">
                    <div id="wrap" class="oe_structure oe_empty"/>
                </t>
            </field>
            <field name="website_indexed" eval="False"/>
        </record>

        <!-- Test model record -->
        <record id="test_model_record" model="test.model">
            <field name="name">Test Model Record</field>
        </record>
    </data>
</odoo>

```

## File: data\test_website_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record id="test_model_multi_website_2" model="test.model.multi.website">
    <field name="name">Test Model Multi Website 2</field>
    <field name="website_id" ref="website.website2"/>
</record>

</odoo>

```

## File: models\model.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.translate import html_translate


class Website(models.Model):
    _inherit = "website"

    name_translated = fields.Char(translate=True)


class TestModel(models.Model):
    _name = 'test.model'
    _inherit = [
        'website.seo.metadata',
        'website.published.mixin',
        'website.searchable.mixin',
    ]
    _description = 'Website Model Test'

    name = fields.Char(required=True, translate=True)
    submodel_ids = fields.One2many('test.submodel', 'test_model_id', "Submodels")
    website_description = fields.Html(
        string="Description for the website",
        translate=html_translate,
        sanitize_overridable=True,
        sanitize_attributes=False,
        sanitize_form=False,
    )
    tag_id = fields.Many2one('test.tag')

    @api.model
    def _search_get_detail(self, website, order, options):
        return {
            'model': 'test.model',
            'base_domain': [],
            'search_fields': ['name', 'submodel_ids.name', 'submodel_ids.tag_id.name'],
            'fetch_fields': ['name'],
            'mapping': {
                'name': {'name': 'name', 'type': 'text', 'match': True},
                'website_url': {'name': 'name', 'type': 'text', 'truncate': False},
            },
            'icon': 'fa-check-square-o',
            'order': 'name asc, id desc',
        }

    def open_website_url(self):
        self.ensure_one()
        return self.env['website'].get_client_action(f'/test_model/{self.id}')


class TestSubModel(models.Model):
    _name = 'test.submodel'
    _description = 'Website Submodel Test'

    name = fields.Char(required=True)
    test_model_id = fields.Many2one('test.model')
    tag_id = fields.Many2one('test.tag')


class TestTag(models.Model):
    _name = 'test.tag'
    _description = 'Website Tag Test'

    name = fields.Char(required=True)


class TestModelMultiWebsite(models.Model):
    _name = 'test.model.multi.website'
    _inherit = [
        'website.published.multi.mixin',
    ]
    _description = 'Multi Website Model Test'

    name = fields.Char(required=True)
    # `cascade` is needed as there is demo data for this model which are bound
    # to website 2 (demo website). But some tests are unlinking the website 2,
    # which would fail if the `cascade` is not set. Note that the website 2 is
    # never set on any records in all other modules.
    website_id = fields.Many2one('website', string='Website', ondelete='cascade')


class TestModelExposed(models.Model):
    _name = "test.model.exposed"
    _inherit = [
        'website.seo.metadata',
        'website.published.mixin',
    ]
    _description = 'Website Model Test Exposed'
    _rec_name = "name"

    name = fields.Char()

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    def action_website_test_setting(self):
        return self.env['website'].get_client_action('/')

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models
from odoo.exceptions import AccessError


class Website(models.Model):
    _inherit = "website"

    some_translatable_field = fields.Char(string="A translatable field",
                                          translate=True, default='something')

    def _search_get_details(self, search_type, order, options):
        result = super()._search_get_details(search_type, order, options)
        if search_type in ['test']:
            result.append(self.env['test.model']._search_get_detail(self, order, options))
        return result

```

## File: models\__init__.py

```python
from . import model
from . import res_config_settings
from . import website

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_test_model_public,access_test_model,model_test_model,base.group_public,1,0,0,0
access_test_model_portal,access_test_model,model_test_model,base.group_portal,1,0,0,0
access_test_model_employee,access_test_model,model_test_model,base.group_user,1,0,0,0
access_test_model_admin,access_test_model,model_test_model,base.group_system,1,1,1,1
access_test_submodel_public,access_test_submodel,model_test_submodel,base.group_public,1,0,0,0
access_test_submodel_portal,access_test_submodel,model_test_submodel,base.group_portal,1,0,0,0
access_test_submodel_employee,access_test_submodel,model_test_submodel,base.group_user,1,0,0,0
access_test_tag_public,access_test_tag,model_test_tag,base.group_public,1,0,0,0
access_test_tag_portal,access_test_tag,model_test_tag,base.group_portal,1,0,0,0
access_test_tag_employee,access_test_tag,model_test_tag,base.group_user,1,0,0,0
access_test_model_test_admin,access_test_model,model_test_model,test_website.group_test_website_admin,1,1,1,1
access_test_model_exposed_employee,access_test_model_exposed,model_test_model_exposed,base.group_user,1,0,0,0
access_test_model_exposed_test_admin,access_test_model_exposed,model_test_model_exposed,test_website.group_test_website_admin,1,1,1,1
access_test_model_multi_website_public,access_test_model_multi_website,model_test_model_multi_website,base.group_public,1,0,0,0
access_test_model_multi_website_portal,access_test_model_multi_website,model_test_model_multi_website,base.group_portal,1,0,0,0
access_test_model_multi_website_employee,access_test_model_multi_website,model_test_model_multi_website,base.group_user,1,0,0,0
access_test_model_multi_website_test_admin,access_test_model_multi_website,model_test_model_multi_website,test_website.group_test_website_admin,1,1,1,1
access_test_model_multi_website,access_test_model_multi_website,model_test_model_multi_website,base.group_user,1,0,0,0
access_test_model_tester,access_test_model,model_test_model,test_website.group_test_website_tester,1,1,1,1

```

## File: security\test_website_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="module_category_test_website" model="ir.module.category">
        <field name="name">Test website</field>
        <field name="sequence" eval="200" />
    </record>

    <record id="test_website.group_test_website_admin" model="res.groups">
        <field name="name">Test Administrator</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <record model="ir.module.category" id="test_website.module_category_test_website">
        <field name="name">Tests about Website with additional model</field>
        <field name="sequence">24</field>
    </record>

    <record id="group_test_website_tester" model="res.groups">
        <field name="name">Tester</field>
        <field name="category_id" ref="test_website.module_category_test_website"/>
    </record>

    <record id="base.user_admin" model="res.users">
        <field name="groups_id" eval="[(4, ref('test_website.group_test_website_tester'))]"/>
    </record>
</odoo>

```

## File: static\src\js\test_error.js

```javascript
/** @odoo-module **/

import { rpc } from "@web/core/network/rpc";
import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.testError = publicWidget.Widget.extend({
    selector: '.rpc_error',
    events: {
        'click a': '_onRpcErrorClick',
    },

    //----------------------------------------------------------------------
    // Handlers
    //----------------------------------------------------------------------

    /**
     * make a rpc call with the href of the DOM element clicked
     * @private
     * @param {Event} ev
     * @returns {Promise}
     */
    _onRpcErrorClick: function (ev) {
        ev.preventDefault();
        var $link = $(ev.currentTarget);
        return rpc($link.attr('href'));
    }
});

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="multi_url" model="website.page">
        <field name="name">Multi URL test</field>
        <field name="url">/multi_url</field>
        <field name="website_published">False</field>
        <field name="type">qweb</field>
        <field name="key">test_website.multi_url</field>
        <field name="website_published">True</field>
        <field name="arch" type="xml">
            <t t-name='multi_url'>
                <div>
                    <a id='get' href="/get">get</a>
                    <form id='post' action="/post">post</form>
                    <a id='get_post' href="/get_post">get_post</a>
                    <a id='get_post_nomultilang' href="/get_post_nomultilang">get_post_nomultilang</a>
                </div>
            </t>
        </field>
    </record>

    <!-- /model_item item page -->
    <template id="model_item" name="Model item">
        <t t-call="website.layout">
            <div id="wrap">
                <section t-cache="record">
                    <div class="container">
                        <div class="row">
                            <div class="col" t-field="record.name"/>
                        </div>
                        <div class="row">
                            <div class="col" t-field="record.website_description"/>
                        </div>
                    </div>
                </section>
                <t t-call="test_website.test_form"/>
            </div>
        </t>
    </template>

    <template id="test_form" name="Test Form">
        <span class="hidden" data-for="test_form" t-att-data-values="{'tag_id': tag and tag.id or ''}" />
        <section class="s_website_form pt16 pb16 o_colored_level" data-vcss="001" data-snippet="s_website_form" data-name="Form">
            <div class="container">
                <form id="test_form" action="#fake" method="post" enctype="multipart/form-data" class="o_mark_required" data-mark="*" data-pre-fill="true" data-success-mode="redirect" data-success-page="/success" data-model_name="test.model" hide-change-model="true">
                    <div class="s_website_form_rows row s_col_no_bgcolor">
                        <div class="mb-0 py-2 s_website_form_field col-12 s_website_form_required" data-type="char" data-name="Field">
                            <div class="row s_col_no_resize s_col_no_bgcolor">
                                <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="testform1">
                                    <span class="s_website_form_label_content">Name</span>
                                    <span class="s_website_form_mark"> *</span>
                                </label>
                                <div class="col-sm">
                                    <input type="text" class="form-control s_website_form_input" name="name" required="1" data-fill-with="name" id="testform1"/>
                                </div>
                            </div>
                        </div>
                        <div class="mb-0 py-2 s_website_form_field col-12 s_website_form_dnone" data-name="Field"
                             data-type="record" data-model="test.tag">
                            <div class="row s_col_no_resize s_col_no_bgcolor">
                                <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="testmodel2">
                                    <span class="s_website_form_label_content">Tag</span>
                                </label>
                                <div class="col-sm">
                                    <input type="hidden" class="form-control s_website_form_input" name="tag_id" id="testmodel2" />
                                </div>
                            </div>
                        </div>
                        <div class="mb-0 py-2 col-12 s_website_form_submit" data-name="Submit Button">
                            <div style="width: 200px;" class="s_website_form_label"/>
                            <a href="#" role="button" class="btn btn-primary btn-lg s_website_form_send o_default_snippet_text">Submit</a>
                            <span id="s_website_form_result"/>
                        </div>
                    </div>
                </form>
            </div>
        </section>
    </template>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.test.website</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//setting[@id='plausbile_setting']" position="after">
                <div class="col-12 col-lg-6 o_setting_box" id="website_test_setting">
                    <button type="object" name="action_website_test_setting" string="Website test setting" class="btn-link" icon="fa-arrow-right"/>
                </div>
            </xpath>
        </field>
    </record>

    <!-- Front end page for test model -->
    <template id="test_model_page_layout" name="Test Model">
        <t t-call="website.layout">
            <t t-set="additional_title" t-value="test_model.name" />
            <span t-field="test_model.name"/>
        </t>
    </template>

</odoo>

```

## File: views\test_model_multi_website_views.xml

```xml
<?xml version="1.0"?>
<odoo>

<!-- test.model.multi.website views -->
<record id="test_model_multi_website_view_kanban" model="ir.ui.view">
    <field name="name">test.model.multi.website.kanban</field>
    <field name="model">test.model.multi.website</field>
    <field name="arch" type="xml">
        <kanban js_class="website_pages_kanban" class="o_kanban_mobile" action="open_website_url" type="object" sample="1">
            <templates>
                <t t-name="card">
                    <field name="name" class="text-truncate fw-bolder mb-auto"/>
                    <div class="text-muted fw-bolder" t-if="record.website_id.value" groups="website.group_multi_website">
                        <i class="fa fa-globe me-1" title="Website"/>
                        <field name="website_id"/>
                    </div>
                    <div class="d-flex border-top mt-2 pt-2">
                        <field name="is_published" widget="boolean_toggle"/>
                        <t t-if="record.is_published.raw_value">Published</t>
                        <t t-else="">Not Published</t>
                    </div>
                </t>
            </templates>
        </kanban>
    </field>
</record>
<record id="test_model_multi_website_view_list" model="ir.ui.view">
    <field name="name">Test Multi Model Pages List</field>
    <field name="model">test.model.multi.website</field>
    <field name="priority">99</field>
    <field name="arch" type="xml">
        <list js_class="website_pages_list" type="object" action="open_website_url" multi_edit="1">
            <field name="name"/>
            <field name="website_url"/>
            <field name="website_id" groups="website.group_multi_website"/>
        </list>
    </field>
</record>
<record id="action_test_model_multi_website" model="ir.actions.act_window">
    <field name="name">Test Multi Model Pages</field>
    <field name="res_model">test.model.multi.website</field>
    <field name="view_mode">list,kanban,form</field>
    <field name="view_ids" eval="[(5, 0, 0),
        (0, 0, {'view_mode': 'list', 'view_id': ref('test_model_multi_website_view_list')}),
        (0, 0, {'view_mode': 'kanban', 'view_id': ref('test_model_multi_website_view_kanban')}),
    ]"/>
</record>

<!-- js_class bug records -->
<record id="test_model_multi_website_view_list_js_class_bug" model="ir.ui.view">
    <field name="name">Test Multi Model Pages list js_class bug</field>
    <field name="model">test.model.multi.website</field>
    <field name="priority">99</field>
    <!-- Omitting `website_pages_list` on purpose to test it does not crash -->
    <field name="arch" type="xml">
        <list type="object" action="open_website_url" multi_edit="1">
            <field name="name"/>
            <field name="website_url"/>
            <field name="website_id" groups="website.group_multi_website"/>
        </list>
    </field>
</record>
<record id="action_test_model_multi_website_js_class_bug" model="ir.actions.act_window">
    <field name="name">Test Multi Model Pages js_class bug</field>
    <field name="res_model">test.model.multi.website</field>
    <field name="view_mode">list,kanban,form</field>
    <field name="view_ids" eval="[(5, 0, 0),
        (0, 0, {'view_mode': 'list', 'view_id': ref('test_model_multi_website_view_list_js_class_bug')}),
        (0, 0, {'view_mode': 'kanban', 'view_id': ref('test_model_multi_website_view_kanban')}),
    ]"/>
</record>

</odoo>

```

## File: views\test_model_views.xml

```xml
<?xml version="1.0"?>
<odoo>

<!-- test.model views -->
<record id="test_model_view_kanban" model="ir.ui.view">
    <field name="name">test.model.kanban</field>
    <field name="model">test.model</field>
    <field name="arch" type="xml">
        <kanban js_class="website_pages_kanban" class="o_kanban_mobile" action="open_website_url" type="object" sample="1">
            <templates>
                <t t-name="card">
                    <field class="text-truncate mb-auto fw-bolder" name="name"/>
                    <div class="d-flex border-top mt-2 pt-2">
                        <field name="is_published" widget="boolean_toggle"/>
                        <t t-if="record.is_published.raw_value">Published</t>
                        <t t-else="">Not Published</t>
                    </div>
                </t>
            </templates>
        </kanban>
    </field>
</record>
<record id="test_model_view_list" model="ir.ui.view">
    <field name="name">Test Model Pages List</field>
    <field name="model">test.model</field>
    <field name="priority">99</field>
    <field name="arch" type="xml">
        <list js_class="website_pages_list" type="object" action="open_website_url" multi_edit="1">
            <field name="name"/>
            <field name="website_url"/>
        </list>
    </field>
</record>
<record id="action_test_model" model="ir.actions.act_window">
    <field name="name">Test Model Pages</field>
    <field name="res_model">test.model</field>
    <field name="view_mode">list,kanban,form</field>
    <field name="view_ids" eval="[(5, 0, 0),
        (0, 0, {'view_mode': 'list', 'view_id': ref('test_model_view_list')}),
        (0, 0, {'view_mode': 'kanban', 'view_id': ref('test_model_view_kanban')}),
    ]"/>
</record>

<!-- Backend access to test model -->
<record id="action_test_website_test_model" model="ir.actions.act_window">
    <field name="name">Test Model</field>
    <field name="type">ir.actions.act_window</field>
    <field name="res_model">test.model</field>
    <field name="view_id" eval="False"/>
</record>

<menuitem name="Test Model"
    id="menu_test_website_test_model"
    action="action_test_website_test_model"
    parent="website.menu_website_global_configuration"
    sequence="100"
    groups="base.group_no_one"/>

<record id="view_test_model_form" model="ir.ui.view">
    <field name="name">test.model.form</field>
    <field name="model">test.model</field>
    <field name="arch" type="xml">
        <form string="Test Model">
            <sheet>
                <div name="button_box" position="inside">
                    <field name="is_published" widget="website_redirect_button"/>
                </div>
                <div class="oe_title" name="title">
                    <label for="name" string="Name"/>
                    <h1>
                        <field name="name"/>
                    </h1>
                </div>
            </sheet>
        </form>
    </field>
</record>

</odoo>

```

