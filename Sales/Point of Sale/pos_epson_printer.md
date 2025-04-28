# Odoo Module: pos_epson_printer

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
    'name': 'pos_epson_printer',
    'version': '1.0',
    'category': 'Sales/Point of Sale',
    'sequence': 6,
    'summary': 'Epson ePOS Printers in PoS',
    'description': """

Use Epson ePOS Printers without the IoT Box in the Point of Sale
""",
    'depends': ['point_of_sale'],
    'data': [
        'views/pos_config_views.xml',
        'views/res_config_settings_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'point_of_sale.assets': [
            'pos_epson_printer/static/src/js/**/*',
            'pos_epson_printer/static/src/xml/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: models\pos_config.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class PosConfig(models.Model):
    _inherit = 'pos.config'

    epson_printer_ip = fields.Char(string='Epson Printer IP', help="Local IP address of an Epson receipt printer.")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    pos_epson_printer_ip = fields.Char(compute='_compute_pos_epson_printer_ip', store=True, readonly=False)

    @api.depends('pos_epson_printer_ip', 'pos_other_devices')
    def _compute_pos_iface_cashdrawer(self):
        """We are just adding depends on this compute."""
        super()._compute_pos_iface_cashdrawer()

    def _is_cashdrawer_displayed(self, res_config):
        return super()._is_cashdrawer_displayed(res_config) or (res_config.pos_other_devices and bool(res_config.pos_epson_printer_ip))

    @api.depends('pos_other_devices', 'pos_config_id')
    def _compute_pos_epson_printer_ip(self):
        for res_config in self:
            if not res_config.pos_other_devices:
                res_config.pos_epson_printer_ip = ''
            else:
                res_config.pos_epson_printer_ip = res_config.pos_config_id.epson_printer_ip

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import pos_config
from . import res_config_settings

```

## File: static\src\js\pos_epson_printer.js

```javascript
odoo.define('pos_epson_printer.pos_epson_printer', function (require) {
"use strict";

var { PosGlobalState } = require('point_of_sale.models');
var EpsonPrinter = require('pos_epson_printer.Printer');
const Registries = require('point_of_sale.Registries');


const PosEpsonPosGlobalState = (PosGlobalState) => class PosEpsonPosGlobalState extends PosGlobalState {
    after_load_server_data() {
        var self = this;
        return super.after_load_server_data(...arguments).then(function () {
            if (self.config.other_devices && self.config.epson_printer_ip) {
                self.env.proxy.printer = new EpsonPrinter(self.config.epson_printer_ip , self);
            }
        });
    }
}
Registries.Model.extend(PosGlobalState, PosEpsonPosGlobalState);

});

```

## File: static\src\js\printers.js

```javascript

odoo.define('pos_epson_printer.Printer', function (require) {
"use strict";

var core = require('web.core');
var { PrinterMixin, PrintResult, PrintResultGenerator } = require('point_of_sale.Printer');

var QWeb = core.qweb;
var _t = core._t;

class EpsonPrintResultGenerator extends PrintResultGenerator {
    constructor(address) {
        super();
        this.address = address;
    }

    IoTActionError() {
        var printRes = new PrintResult({
            successful: false,
            message: {
                title: _t('Connection to the printer failed'),
                body: _t('Please check if the printer is still connected. \n' +
                    'Some browsers don\'t allow HTTP calls from websites to devices in the network (for security reasons). ' +
                    'If it is the case, you will need to follow Odoo\'s documentation for ' +
                    '\'Self-signed certificate for ePOS printers\' and \'Secure connection (HTTPS)\' to solve the issue'
                ),
            }
        });

        if (window.location.protocol === 'https:') {
            printRes.message.body += _.str.sprintf(
                _t('If you are on a secure server (HTTPS) please make sure you manually accepted the certificate by accessing %s'),
                this.address
            );
        }

        return printRes;
    }

    IoTResultError(printerErrorCode) {
        let message = _t("The printer was successfully reached, but it wasn't able to print.") + '\n';
        if (printerErrorCode) {
            message += '\n' + _t("The following error code was given by the printer:") + '\n' + printerErrorCode;

            const extra_messages = {
                'DeviceNotFound':
                    _t("Check on the printer configuration for the 'Device ID' setting. " +
                        "It should be set to: ") + "\nlocal_printer",
                'EPTR_REC_EMPTY':
                    _t("No paper was detected by the printer"),
            };
            if (printerErrorCode in extra_messages) {
                message += '\n' + extra_messages[printerErrorCode];
            }
            message += "\n" + _t("To find more details on the error reason, please search online for:") + '\n' +
                " Epson Server Direct Print " + printerErrorCode;
        } else {
            message += _t('Please check if the printer has enough paper and is ready to print.');
        }
        return new PrintResult({
            successful: false,
            message: {
                title: _t('Printing failed'),
                body: message,
            },
        });
    }
}

var EpsonPrinter = core.Class.extend(PrinterMixin, {
    init(ip, pos) {
        PrinterMixin.init.call(this, pos);
        var url = window.location.protocol + '//' + ip;
        this.address = url + '/cgi-bin/epos/service.cgi?devid=local_printer';
        this.printResultGenerator = new EpsonPrintResultGenerator(url);
    },


    /**
     * Transform a (potentially colored) canvas into a monochrome raster image.
     * We will use Floyd-Steinberg dithering.
     */
    _canvasToRaster(canvas) {
        var imageData = canvas.getContext('2d').getImageData(0, 0, canvas.width, canvas.height);
        var pixels = imageData.data;
        var width = imageData.width;
        var height = imageData.height;
        var errors = Array.from(Array(width), _ => Array(height).fill(0));
        var rasterData = new Array(width * height).fill(0);

        for (var y = 0; y < height; y++) {
            for (var x = 0; x < width; x++) {
                var idx, oldColor, newColor;

                // Compute grayscale level. Those coefficients were found online
                // as R, G and B have different impacts on the darkness
                // perception (e.g. pure blue is darker than red or green).
                idx = (y * width + x) * 4;
                oldColor = pixels[idx] * 0.299 + pixels[idx+1] * 0.587 + pixels[idx+2] * 0.114;

                // Propagate the error from neighbor pixels 
                oldColor += errors[x][y];
                oldColor = Math.min(255, Math.max(0, oldColor));

                if (oldColor < 128) {
                    // This pixel should be black
                    newColor = 0;
                    rasterData[y * width + x] = 1;
                } else {
                    // This pixel should be white
                    newColor = 255;
                    rasterData[y * width + x] = 0;
                }

                // Propagate the error to the following pixels, based on
                // Floyd-Steinberg dithering.
                var error = oldColor - newColor;
                if (error) {
                    if (x < width - 1) {
                        // Pixel on the right
                        errors[x + 1][y] += 7/16 * error;
                    }
                    if (x > 0 && y < height - 1) {
                        // Pixel on the bottom left
                        errors[x - 1][y + 1] += 3/16 * error;
                    }
                    if (y < height - 1) {
                        // Pixel below
                        errors[x][y + 1] += 5/16 * error;
                    }
                    if (x < width - 1 && y < height - 1) {
                        // Pixel on the bottom right
                        errors[x + 1][y + 1] += 1/16 * error;
                    }
                }
            }
        }

        return rasterData.join('');
    },

    /**
     * Base 64 encode a raster image
     */
    _encodeRaster(rasterData) {
        var encodedData = '';
        for(var i = 0; i < rasterData.length; i+=8){
            var sub = rasterData.substr(i, 8);
            encodedData += String.fromCharCode(parseInt(sub, 2));
        }
        return btoa(encodedData);
    },

    /**
     * Create the raster data from a canvas
     * 
     * @override
     */
    process_canvas(canvas) {
        var rasterData = this._canvasToRaster(canvas);
        var encodedData = this._encodeRaster(rasterData);
        return QWeb.render('ePOSPrintImage', {
            image: encodedData,
            width: canvas.width,
            height: canvas.height,
        });
    },

    /**
     * @override
     */
    open_cashbox() {
        var pulse = QWeb.render('ePOSDrawer');
        this.send_printing_job(pulse);
    },

    /**
     * @override
     */
    async send_printing_job(img) {
        const res = await $.ajax({
            url: this.address,
            method: 'POST',
            data: img,
        });
        const response = $(res).find('response');
        return {"result": response.attr('success') === 'true', "printerErrorCode": response.attr('code')};
    },
});

return EpsonPrinter;

});

```

## File: static\src\xml\epos_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="ePOSTemplate">
        <s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
            <s:Body>
                <epos-print xmlns="http://www.epson-pos.com/schemas/2011/03/epos-print">
                    <t t-out="0"/>
                </epos-print>
            </s:Body>
        </s:Envelope>
    </t>

    <t t-name="ePOSPrintImage">
        <t t-call="ePOSTemplate">
            <image t-att-width="width" t-att-height="height" align="center" t-esc="image"/>
            <cut type="feed"/>
        </t>
    </t>

    <t t-name="ePOSDrawer">
        <t t-call="ePOSTemplate">
            <pulse/>
        </t>
    </t>
</templates>

```

## File: views\pos_config_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pos_iot_config_view_form" model="ir.ui.view">
        <field name="name">pos.iot.config.form.view</field>
        <field name="model">pos.config</field>
        <field name="inherit_id" ref="point_of_sale.pos_config_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='other_devices']//div[hasclass('o_setting_right_pane')]" position="inside">
                <div class="content-group" attrs="{'invisible' : [('other_devices', '=', False)]}">
                    <field name="epson_printer_ip" placeholder="Epson Receipt Printer IP Address" />
                    <div class="row" attrs="{'invisible': [('epson_printer_ip', 'in', [False, ''])]}">
                        <label string="Cashdrawer" for="iface_cashdrawer" class="col-lg-3 o_light_label"/>
                        <field name="iface_cashdrawer"/>
                    </div>
                </div>
                <div role="alert" class="alert alert-warning" attrs="{'invisible': ['|', '|', ('iface_print_via_proxy', '!=', True), ('other_devices', '!=', True), ('epson_printer_ip', 'in', [False, ''])]}">
                    The Epson receipt printer will be used instead of the receipt printer connected to the IoT Box.
                </div>
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
        <field name="name">res.config.settings.view.form.inherit.pos.epson.printer</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="point_of_sale.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='pos_other_devices']//div[hasclass('o_setting_right_pane')]" position="inside">
                <div class="content-group" attrs="{'invisible' : [('pos_other_devices', '=', False)]}">
                    <field name="pos_epson_printer_ip" placeholder="Epson Receipt Printer IP Address" />
                    <div class="row" attrs="{'invisible': [('pos_epson_printer_ip', 'in', [False, ''])]}">
                        <label string="Cashdrawer" for="pos_iface_cashdrawer" class="col-lg-3 o_light_label"/>
                        <field name="pos_iface_cashdrawer"/>
                    </div>
                </div>
                <div role="alert" class="alert alert-warning" attrs="{'invisible': ['|', '|', ('pos_iface_print_via_proxy', '!=', True), ('pos_other_devices', '!=', True), ('pos_epson_printer_ip', 'in', [False, ''])]}">
                    The Epson receipt printer will be used instead of the receipt printer connected to the IoT Box.
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

