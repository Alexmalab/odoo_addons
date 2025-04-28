# Odoo Module: test_website

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers

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
contained in a separate module as we are testing module install/uninstall/upgrade
and we don't want to reload the website module every time, including it's possible
dependencies. Neither we want to add in website module some routes, views and
models which only purpose is to run tests.""",
    'depends': [
        'website',
    ],
    'data': [
        'views/templates.xml',
        'data/test_website_data.xml',
    ],
    'installable': True,
    'application': False,
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

    @http.route('/ignore_args/converteronly/<string:a>/', type='http', auth="public", website=True, sitemap=False)
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

    @http.route('/ignore_args/converter/<string:a>/', type='http', auth="public", website=True, sitemap=False)
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

    @http.route(['/get'], type='http', auth="public", methods=['GET'], website=True)
    def get_method(self, **kw):
        return request.make_response('get')

    @http.route(['/post'], type='http', auth="public", methods=['POST'], website=True)
    def post_method(self, **kw):
        return request.make_response('post')

    @http.route(['/get_post'], type='http', auth="public", methods=['GET', 'POST'], website=True)
    def get_post_method(self, **kw):
        return request.make_response('get_post')

    @http.route(['/get_post_nomultilang'], type='http', auth="public", methods=['GET', 'POST'], website=True, multilang=False)
    def get_post_method_no_multilang(self, **kw):
        return request.make_response('get_post_nomultilang')

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

    </data>
</odoo>

```

## File: static\src\js\test_error.js

```javascript
odoo.define('website_forum.test_error', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

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
        return this._rpc({
            route: $link.attr('href'),
        });
    }
});
});

```

## File: views\templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_frontend" inherit_id="website.assets_frontend">
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/test_website/static/src/js/test_error.js"></script>
        </xpath>
    </template>

    <template id="assets_tests" name="Test Website Assets Tests" inherit_id="web.assets_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/test_website/static/tests/tours/reset_views.js"></script>
            <script type="text/javascript" src="/test_website/static/tests/tours/error_views.js"></script>
            <script type="text/javascript" src="/test_website/static/tests/tours/json_auth.js"></script>
        </xpath>
    </template>

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
                    <form id='post' action="/post">post</form>>
                    <a id='get_post' href="/get_post">get_post</a>
                    <a id='get_post_nomultilang' href="/get_post_nomultilang">get_post_nomultilang</a>
                </div>
            </t>
        </field>
    </record>
</odoo>

```

