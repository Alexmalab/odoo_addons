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
        ],
    'assets': {
        'point_of_sale.assets': [
            'pos_cache/static/**/*',
        ],
    },
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
            products = Product.search(cache.get_product_domain(), order='sequence,default_code,name')
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

    def delete_cache(self):
        # throw away the old caches
        self.cache_ids.unlink()

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def get_products_from_cache(self):
        loading_info = self._loader_params_product_product()
        fields_str = str(loading_info['search_params']['fields'])
        domain_str = str([list(item) if isinstance(item, (list, tuple)) else item for item in loading_info['search_params']['domain']])
        pos_cache = self.env['pos.cache']
        cache_for_user = pos_cache.search([
            ('id', 'in', self.config_id.cache_ids.ids),
            ('compute_user_id', '=', self.env.uid),
            ('product_domain', '=', domain_str),
            ('product_fields', '=', fields_str),
        ])

        if not cache_for_user:
            cache_for_user = pos_cache.create({
                'config_id': self.config_id.id,
                'product_domain': domain_str,
                'product_fields': fields_str,
                'compute_user_id': self.env.uid
            })
            cache_for_user.refresh_cache()

        return cache_for_user.cache2json()

    def _get_pos_ui_product_product(self, params):
        """
        If limited_products_loading is active, prefer the native way of loading products.
        Otherwise, replace the way products are loaded.
            First, we only load the first 100000 products.
            Then, the UI will make further requests of the remaining products.
        """
        if self.config_id.limited_products_loading:
            return super()._get_pos_ui_product_product(params)
        records = self.get_products_from_cache()
        self._process_pos_ui_product_product(records)
        return records[:100000]

    def get_cached_products(self, start, end):
        records = self.get_products_from_cache()
        self._process_pos_ui_product_product(records)
        return records[start:end]

    def get_total_products_count(self):
        records = self.get_products_from_cache()
        return len(records)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import pos_cache
from . import pos_session

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_pos_cache,pos.cache,model_pos_cache,point_of_sale.group_pos_user,1,1,1,1

```

## File: static\src\js\Chrome.js

```javascript
odoo.define('pos_cache.chrome', function (require) {
    'use strict';

const Chrome = require('point_of_sale.Chrome');
const Registries = require('point_of_sale.Registries');

function roundUpDiv(y, x) {
    const remainder = y % x;
    return (y - remainder) / x + (remainder > 0 ? 1 : 0);
}

const PosCacheChrome = (Chrome) => class PosCacheChrome extends Chrome {
    _runBackgroundTasks() {
        super._runBackgroundTasks();
        if (!this.env.pos.config.limited_products_loading) {
            this._loadRemainingProducts();
        }
    }
    async _loadRemainingProducts() {
        const nInitiallyLoaded = Object.keys(this.env.pos.db.product_by_id).length;
        const totalProductsCount = await this.env.pos._getTotalProductsCount();
        const nRemaining = totalProductsCount - nInitiallyLoaded;
        if (!(nRemaining > 0)) return;
        const multiple = 100000;
        const nLoops = roundUpDiv(nRemaining, multiple);
        for (let i = 0; i < nLoops; i++) {
            await this.env.pos._loadCachedProducts(i * multiple + nInitiallyLoaded, (i + 1) * multiple + nInitiallyLoaded);
        }
        this.showNotification(this.env._t('All products are loaded.'), 5000);
    }
};
Registries.Component.extend(Chrome, PosCacheChrome);
});

```

## File: static\src\js\pos_cache.js

```javascript
odoo.define('pos_cache.pos_cache', function (require) {
"use strict";

var { PosGlobalState } = require('point_of_sale.models');
const Registries = require('point_of_sale.Registries');


const PosCachePosGlobalState = (PosGlobalState) => class PosCachePosGlobalState extends PosGlobalState {
    async _getTotalProductsCount() {
        return this.env.services.rpc({
            model: 'pos.session',
            method: 'get_total_products_count',
            args: [[odoo.pos_session_id]],
            context: this.env.session.user_context,
        });
    }
    async _loadCachedProducts(start, end) {
        const products = await this.env.services.rpc({
            model: 'pos.session',
            method: 'get_cached_products',
            args: [[odoo.pos_session_id], start, end],
            context: this.env.session.user_context,
        });
        this._loadProductProduct(products);
    }
}
Registries.Model.extend(PosGlobalState, PosCachePosGlobalState);

});

```

## File: views\pos_cache_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <data>
        <record id="view_pos_config_kanban" model="ir.ui.view">
            <field name="name">pos.config.kanban.view.inherit.pos_cache</field>
            <field name="model">pos.config</field>
            <field name="inherit_id" ref="point_of_sale.view_pos_config_kanban" />
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('o_kanban_card_manage_settings')]" position="inside">
                    <field name='oldest_cache_time' invisible="1" />
                    <div role="menuitem" class="col-12" style="border-left: none;" attrs="{'invisible': [('oldest_cache_time', '=', False)]}">
                        <a name='delete_cache' type="object" >Invalidate cache</a>
                    </div>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

