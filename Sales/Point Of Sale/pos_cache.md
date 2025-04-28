# Odoo Module: pos_cache

Category: Sales/Point Of Sale

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
    'name': "pos_cache",

    'summary': "Enable a cache on products for a lower POS loading time.",

    'description': """
This creates a product cache per POS config. It drastically lowers the
time it takes to load a POS session with a lot of products.
    """,

    'category': 'Sales/Point Of Sale',
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
                'cache': base64.encodebytes(json.dumps(res).encode('utf-8')),
            })

    @api.model
    def get_product_domain(self):
        return literal_eval(self.product_domain)

    @api.model
    def get_product_fields(self):
        return literal_eval(self.product_fields)

    @api.model
    def get_cache(self, domain, fields):
        if not self.cache or domain != self.get_product_domain() or fields != self.get_product_fields():
            self.product_domain = str(domain)
            self.product_fields = str(fields)
            self.refresh_cache()

        return json.loads(base64.decodebytes(self.cache).decode('utf-8'))


class pos_config(models.Model):
    _inherit = 'pos.config'

    @api.depends('cache_ids')
    def _get_oldest_cache_time(self):
        for cache in self:
            pos_cache = self.env['pos.cache']
            oldest_cache = pos_cache.search([('config_id', '=', cache.id)], order='write_date', limit=1)
            cache.oldest_cache_time = oldest_cache.write_date

    # Use a related model to avoid the load of the cache when the pos load his config
    cache_ids = fields.One2many('pos.cache', 'config_id')
    oldest_cache_time = fields.Datetime(compute='_get_oldest_cache_time', string='Oldest cache time', readonly=True)

    def _get_cache_for_user(self):
        pos_cache = self.env['pos.cache']
        cache_for_user = pos_cache.search([('id', 'in', self.cache_ids.ids), ('compute_user_id', '=', self.env.uid)])

        if cache_for_user:
            return cache_for_user[0]
        else:
            return None

    def get_products_from_cache(self, fields, domain):
        cache_for_user = self._get_cache_for_user()

        if cache_for_user:
            return cache_for_user.get_cache(domain, fields)
        else:
            pos_cache = self.env['pos.cache']
            pos_cache.create({
                'config_id': self.id,
                'product_domain': str(domain),
                'product_fields': str(fields),
                'compute_user_id': self.env.uid
            })
            new_cache = self._get_cache_for_user()
            return new_cache.get_cache(domain, fields)

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
        }
        return posmodel_super.load_server_data.apply(this, arguments).then(function () {
          // Give both the fields and domain to pos_cache in the
          // backend. This way we don't have to hardcode these
          // values in the backend and they automatically stay in
          // sync with whatever is defined (and maybe extended by
          // other modules) in js.
          var product_fields =  typeof product_model.fields === 'function'  ? product_model.fields(self)  : product_model.fields;
          var product_domain =  typeof product_model.domain === 'function'  ? product_model.domain(self)  : product_model.domain;
            var records = rpc.query({
                    model: 'pos.config',
                    method: 'get_products_from_cache',
                    args: [self.pos_session.config_id[0], product_fields, product_domain],
                });
            self.chrome.loading_message(_t('Loading') + ' product.product', 1);
            return records.then(function (products) {
                self.db.add_products(_.map(products, function (product) {
                    product.categ = _.findWhere(self.product_categories, {'id': product.categ_id[0]});
                    product.pos = self;
                    return new models.Product({}, product);
                }));
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

