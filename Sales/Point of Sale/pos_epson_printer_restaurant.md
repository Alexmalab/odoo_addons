# Odoo Module: pos_epson_printer_restaurant

Category: Sales/Point of Sale

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
    'name': 'pos_epson_printer_restaurant',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Epson Printers as Order Printers',
    'description': """

Use Epson Printers as Order Printers in the Point of Sale without the IoT Box
""",
    'depends': ['pos_epson_printer', 'pos_restaurant'],
    'data': [
        'views/pos_restaurant_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'point_of_sale.assets': [
            'pos_epson_printer_restaurant/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\pos_restaurant.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class RestaurantPrinter(models.Model):

    _inherit = 'restaurant.printer'

    printer_type = fields.Selection(selection_add=[('epson_epos', 'Use an Epson printer')])
    epson_printer_ip = fields.Char(string='Epson Printer IP Address', help="Local IP address of an Epson receipt printer.")

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class PosSession(models.Model):
    _inherit = 'pos.session'

    def _loader_params_restaurant_printer(self):
        result = super()._loader_params_restaurant_printer()
        result['search_params']['fields'].append('epson_printer_ip')
        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_restaurant
from . import pos_session

```

## File: static\src\js\models.js

```javascript
odoo.define('pos_epson_printer_restaurant.models', function (require) {
"use strict";

var { PosGlobalState } = require('point_of_sale.models');
var EpsonPrinter = require('pos_epson_printer.Printer');
const Registries = require('point_of_sale.Registries');

// The override of create_printer needs to happen after its declaration in
// pos_restaurant. We need to make sure that this code is executed after the
// models file in pos_restaurant.
require('pos_restaurant.models');


const PosEpsonResPosGlobalState = (PosGlobalState) => class PosEpsonResPosGlobalState extends PosGlobalState {
    create_printer(config) {
        if (config.printer_type === "epson_epos") {
            return new EpsonPrinter(config.epson_printer_ip, this);
        } else {
            return super.create_printer(...arguments);
        }
    }
}
Registries.Model.extend(PosGlobalState, PosEpsonResPosGlobalState);
});

```

## File: views\pos_restaurant_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_restaurant_printer_iot_form" model="ir.ui.view">
        <field name="name">pos.restaurant.iot.config.form.view</field>
        <field name="model">restaurant.printer</field>
        <field name="inherit_id" ref="pos_restaurant.view_restaurant_printer_form"/>
        <field name="arch" type="xml">
            <field name="printer_type" position="after">
                <field name="epson_printer_ip" attrs="{'invisible': [('printer_type', '!=', 'epson_epos')]}"/>
            </field>
        </field>
    </record>
</odoo>

```

