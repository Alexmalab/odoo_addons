# Odoo Module: pos_self_order_epson_printer

Category: Sales/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': 'POS Self Order Epson Printer',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Epson ePOS Printers in PoS Kiosk',
    'description': "Use Epson ePOS Printers without the IoT Box in the PoS Kiosk",
    'depends': ['pos_epson_printer', 'pos_self_order'],
    'installable': True,
    'auto_install': True,
    'assets': {
        'pos_self_order.assets': [
            'pos_epson_printer/static/src/app/epson_printer.js',
            'pos_epson_printer/static/src/app/components/epos_templates.xml',
            'pos_self_order_epson_printer/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: static\src\overrides\models\self_order_service.js

```javascript
import { EpsonPrinter } from "@pos_epson_printer/app/epson_printer";
import { SelfOrder } from "@pos_self_order/app/self_order_service";
import { patch } from "@web/core/utils/patch";

patch(SelfOrder.prototype, {
    async setup() {
        await super.setup(...arguments);
        if (!this.config.epson_printer_ip || !this.config.other_devices) {
            return;
        }
        this.printer.setPrinter(
            new EpsonPrinter({
                ip: this.config.epson_printer_ip,
            })
        );
    },
    create_printer(printer) {
        if (printer.printer_type === "epson_epos") {
            return new EpsonPrinter({ ip: printer.epson_printer_ip });
        }
        return super.create_printer(...arguments);
    },
});

```

