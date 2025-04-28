# Odoo Module: pos_cache

Category: Sales/Point of Sale

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
    'name': "pos_cache",

    'summary': "Enable a cache on products for a lower POS loading time.",

    'description': """
This creates a product cache per POS config. It drastically lowers the
time it takes to load a POS session with a lot of products.
    """,

    'category': 'Sales/Point of Sale',
    'version': '1.0',
    'depends': ['point_of_sale'],
    'data': [
        'data/pos_cache_data.xml',
        'security/ir.model.access.csv',
        'views/pos_cache_views.xml',
        'views/pos_cache_templates.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.point_of_sale.controllers.main import PosController
from odoo import http
from odoo.http import request


class PosCache(PosController):

    @http.route()
    def load_onboarding_data(self):
        super().load_onboarding_data()
        request.env["pos.cache"].refresh_all_caches()

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import main

```

## File: data\pos_cache_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record model="ir.cron" id="refresh_pos_cache_cron">
            <field name="name">PoS: refresh cache</field>
            <field name="model_id" ref="model_pos_cache"/>
            <field name="state">code</field>
            <field name="code">model.refresh_all_caches()</field>
            <field name="active" eval="False"/>
            <field name="interval_number">1</field>
            <field name="interval_type">hours</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: doc\cache.rst

```rst
POS Cache
+++++++++

This module enables a cache for the products in the pos configs. Each POS Config has his own Cache.

The Cache is updated every hour by a cron.

============
Compute user
============

As it's a bad practice to use the admin in a multi-company configuration, a field permit to force a user to compute
the cache. A badly chosen user can result in wrong taxes in POS in a multi-company environment.

```

## File: models\pos_cache.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import json
from ast import literal_eval

from odoo import models, fields, api
from odoo.tools import date_utils


class pos_cache(models.Model):
    _name = 'pos.cache'
    _description = 'Point of Sale Cache'

    cache = fields.Binary(attachment=True)
    product_domain = fields.Text(required=True)
    product_fields = fields.Text(required=True)

    config_id = fields.Many2one('pos.config', ondelete='cascade', required=True)
    compute_user_id = fields.Many2one('res.users', 'Cache compute user', required=True)

    @api.model
    def refresh_all_caches(self):
        self.env['pos.cache'].search([]).refresh_cache()

    def refresh_cache(self):
        for cache in self:
            Product = self.env['product.product'].with_user(cache.compute_user_id.id)
            products = Product.search(cache.get_product_domain())
            prod_ctx = products.with_context(pricelist=cache.config_id.pricelist_id.id,
                display_default_code=False, lang=cache.compute_user_id.lang)
            res = prod_ctx.read(cache.get_product_fields())
            cache.write({
                'cache': base64.encodebytes(json.dumps(res, default=date_utils.json_default).encode('utf-8')),
            })

    @api.model
    def get_product_domain(self):
        return literal_eval(self.product_domain)

    @api.model
    def get_product_fields(self):
        return literal_eval(self.product_fields)

    def cache2json(self):
        return json.loads(base64.decodebytes(self.cache).decode('utf-8'))


class pos_config(models.Model):
    _inherit = 'pos.config'

    @api.depends('cache_ids')
    def _get_oldest_cache_time(self):
        for cache in self:
            pos_cache = self.env['pos.cache']
            oldest_cache = pos_cache.search([('config_id', '=', cache.id)], order='write_date', limit=1)
            cache.oldest_cache_time = oldest_cache.write_date

    cache_ids = fields.One2many('pos.cache', 'config_id')
    oldest_cache_time = fields.Datetime(compute='_get_oldest_cache_time', string='Oldest cache time', readonly=True)
    limit_products_per_request = fields.Integer(compute='_compute_limit_products_per_request')

    def _compute_limit_products_per_request(self):
        limit = self.env['ir.config_parameter'].sudo().get_param('pos_cache.limit_products_per_request', 0)
        self.update({'limit_products_per_request': int(limit)})

    def get_products_from_cache(self, fields, domain):
        fields_str = str(fields)
        domain_str = str(domain)
        pos_cache = self.env['pos.cache']
        cache_for_user = pos_cache.search([
            ('id', 'in', self.cache_ids.ids),
            ('compute_user_id', '=', self.env.uid),
            ('product_domain', '=', domain_str),
            ('product_fields', '=', fields_str),
        ])

        if not cache_for_user:
            cache_for_user = pos_cache.create({
                'config_id': self.id,
                'product_domain': domain_str,
                'product_fields': fields_str,
                'compute_user_id': self.env.uid
            })
            cache_for_user.refresh_cache()

        return cache_for_user.cache2json()

    def delete_cache(self):
        # throw away the old caches
        self.cache_ids.unlink()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import pos_cache

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_pos_cache,pos.cache,model_pos_cache,point_of_sale.group_pos_user,1,1,1,1

```

## File: static\src\js\pos_cache.js

```javascript
odoo.define('pos_cache.pos_cache', function (require) {
"use strict";

var models = require('point_of_sale.models');
var core = require('web.core');
var rpc = require('web.rpc');
var _t = core._t;

var posmodel_super = models.PosModel.prototype;
models.PosModel = models.PosModel.extend({
    load_server_data: function () {
        var self = this;

        var product_index = _.findIndex(this.models, function (model) {
            return model.model === "product.product";
        });

        var product_model = self.models[product_index];

        // We don't want to load product.product the normal
        // uncached way, so get rid of it.
        if (product_index !== -1) {
            this.models.splice(product_index, 1);
            this.product_model = product_model;
        }
        return posmodel_super.load_server_data.apply(this, arguments).then(function () {
          // After loading the server data we have to add the product model as it is needed
          self.models.push(self.product_model)

          // Give both the fields and domain to pos_cache in the
          // backend. This way we don't have to hardcode these
          // values in the backend and they automatically stay in
          // sync with whatever is defined (and maybe extended by
          // other modules) in js.
          var product_fields =  typeof self.product_model.fields === 'function'  ? self.product_model.fields(self)  : self.product_model.fields;
          var product_domain =  typeof self.product_model.domain === 'function'  ? self.product_model.domain(self)  : self.product_model.domain;
            var limit_products_per_request = self.config.limit_products_per_request;
            var cur_request = 0;
            function next(resolve, reject){
                var domain = product_domain;
                if (limit_products_per_request){
                    domain = domain.slice();
                    // implement offset-limit via id, because "pos.cache"
                    // doesn't have such fields and we can add them in master
                    // branch only
                    domain.unshift(['id', '>', cur_request * limit_products_per_request],
                                   ['id', '<=', (cur_request + 1) * limit_products_per_request]);
                }
                return rpc.query({
                    model: 'pos.config',
                    method: 'get_products_from_cache',
                    args: [self.pos_session.config_id[0], product_fields, domain],
                }).then(function (products) {
                    self.db.add_products(_.map(products, function (product) {
                        product.categ = _.findWhere(self.product_categories, {'id': product.categ_id[0]});
                        product.pos = self;
                        return new models.Product({}, product);
                    }));
                    if (limit_products_per_request) {
                        cur_request++;
                        // check that we have more products
                        domain = product_domain.slice();
                        domain.unshift(['id', '>', cur_request * limit_products_per_request]);
                        rpc.query({
                            model: 'product.product',
                            method: 'search_read',
                            args: [domain, ['id']],
                            kwargs: {
                                limit: 1,
                            }
                        }).then(function (products) {
                            if (products.length){
                                next(resolve, reject);
                            } else {
                                resolve();
                            }
                        });
                    } else {
                        resolve();
                    }
                });
            }
            self.setLoadingMessage(_t('Loading') + ' product.product', 1);

            return new Promise((resolve, reject) => {
                next(resolve, reject);
            });
        });
    },
});

});

```

## File: views\pos_cache_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="assets" inherit_id="point_of_sale.assets">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/pos_cache/static/src/js/pos_cache.js"></script>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\pos_cache_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="pos_config_view_form_inherit_pos_cache" model="ir.ui.view">
            <field name="name">pos.config.form.inherit.pos.cache</field>
            <field name="model">pos.config</field>
            <field name="inherit_id" ref="point_of_sale.pos_config_view_form" />
            <field name="arch" type="xml">
                <xpath expr="//field[@name='currency_id']" position="after">
                    <field name='oldest_cache_time'/>
                    <button name='delete_cache' type="object"
                        string="Invalidate cache"
                        attrs="{'invisible': [('oldest_cache_time', '=', False)]}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

