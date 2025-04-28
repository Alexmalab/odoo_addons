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
    'name': 'POS Epson Printer',
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
        'views/pos_printer_views.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'point_of_sale._assets_pos': [
            'pos_epson_printer/static/src/**/*',
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

## File: models\pos_printer.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError

class PosPrinter(models.Model):

    _inherit = 'pos.printer'

    printer_type = fields.Selection(selection_add=[('epson_epos', 'Use an Epson printer')])
    epson_printer_ip = fields.Char(string='Epson Printer IP Address', help="Local IP address of an Epson receipt printer.", default="0.0.0.0")

    @api.constrains('epson_printer_ip')
    def _constrains_epson_printer_ip(self):
        for record in self:
            if record.printer_type == 'epson_epos' and not record.epson_printer_ip:
                raise ValidationError(_("Epson Printer IP Address cannot be empty."))

    @api.model
    def _load_pos_data_fields(self, config_id):
        params = super()._load_pos_data_fields(config_id)
        params += ['epson_printer_ip']
        return params

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
from . import pos_printer

```

## File: static\src\app\epson_printer.js

```javascript
import { BasePrinter } from "@point_of_sale/app/printer/base_printer";
import { _t } from "@web/core/l10n/translation";
import { getTemplate } from "@web/core/templates";
import { createElement, append, createTextNode } from "@web/core/utils/xml";

function ePOSPrint(children) {
    let ePOSLayout = getTemplate("pos_epson_printer.ePOSLayout");
    if (!ePOSLayout) {
        throw new Error("'ePOSLayout' not loaded");
    }
    ePOSLayout = ePOSLayout.cloneNode(true);
    const [eposPrintEl] = ePOSLayout.getElementsByTagName("epos-print");
    append(eposPrintEl, children);
    // IMPORTANT: Need to remove `xmlns=""` in the image and cut elements.
    // > Otherwise, the print request will succeed but it the printer device won't actually do the printing.
    return ePOSLayout.innerHTML.replaceAll(`xmlns=""`, "");
}

/**
 * Sends print request to ePos printer that is directly connected to the local network.
 */
export class EpsonPrinter extends BasePrinter {
    setup({ ip }) {
        super.setup(...arguments);
        this.url = window.location.protocol + "//" + ip;
        this.address = this.url + "/cgi-bin/epos/service.cgi?devid=local_printer";
    }

    /**
     * @override
     * Create the raster data from a canvas
     */
    processCanvas(canvas) {
        const rasterData = this.canvasToRaster(canvas);
        const encodedData = this.encodeRaster(rasterData);
        return ePOSPrint([
            createElement(
                "image",
                {
                    width: canvas.width,
                    height: canvas.height,
                    align: "center",
                },
                [createTextNode(encodedData)]
            ),
            createElement("cut", { type: "feed" }),
        ]);
    }

    /**
     * @override
     */
    openCashbox() {
        const pulse = ePOSPrint([createElement("pulse")]);
        this.sendPrintingJob(pulse);
    }

    /**
     * @override
     */
    async sendPrintingJob(img) {
        const res = await fetch(this.address, {
            method: "POST",
            body: img,
        });
        const body = await res.text();
        const parser = new DOMParser();
        const parsedBody = parser.parseFromString(body, "application/xml");
        const response = parsedBody.querySelector("response");
        return {
            result: response.getAttribute("success") === "true",
            printerErrorCode: response.getAttribute("code"),
        };
    }

    /**
     * Transform a (potentially colored) canvas into a monochrome raster image.
     * We will use Floyd-Steinberg dithering.
     */
    canvasToRaster(canvas) {
        const imageData = canvas.getContext("2d").getImageData(0, 0, canvas.width, canvas.height);
        const pixels = imageData.data;
        const width = imageData.width;
        const height = imageData.height;
        const errors = Array.from(Array(width), (_) => Array(height).fill(0));
        const rasterData = new Array(width * height).fill(0);

        for (let y = 0; y < height; y++) {
            for (let x = 0; x < width; x++) {
                let oldColor, newColor;

                // Compute grayscale level. Those coefficients were found online
                // as R, G and B have different impacts on the darkness
                // perception (e.g. pure blue is darker than red or green).
                const idx = (y * width + x) * 4;
                oldColor = pixels[idx] * 0.299 + pixels[idx + 1] * 0.587 + pixels[idx + 2] * 0.114;

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
                const error = oldColor - newColor;
                if (error) {
                    if (x < width - 1) {
                        // Pixel on the right
                        errors[x + 1][y] += (7 / 16) * error;
                    }
                    if (x > 0 && y < height - 1) {
                        // Pixel on the bottom left
                        errors[x - 1][y + 1] += (3 / 16) * error;
                    }
                    if (y < height - 1) {
                        // Pixel below
                        errors[x][y + 1] += (5 / 16) * error;
                    }
                    if (x < width - 1 && y < height - 1) {
                        // Pixel on the bottom right
                        errors[x + 1][y + 1] += (1 / 16) * error;
                    }
                }
            }
        }

        return rasterData.join("");
    }

    /**
     * Base 64 encode a raster image
     */
    encodeRaster(rasterData) {
        let encodedData = "";
        for (let i = 0; i < rasterData.length; i += 8) {
            const sub = rasterData.substr(i, 8);
            encodedData += String.fromCharCode(parseInt(sub, 2));
        }
        return btoa(encodedData);
    }

    /**
     * @override
     */
    getActionError() {
        const printRes = super.getResultsError();
        if (window.location.protocol === "https:") {
            printRes.message.body += _t(
                "If you are on a secure server (HTTPS) please make sure you manually accepted the certificate by accessing %s. ",
                this.url
            );
        }
        return printRes;
    }

    /**
     * @override
     */
    getResultsError(printResult) {
        const errorCode = printResult.printerErrorCode;
        let message =
            _t("The printer was successfully reached, but it wasn't able to print.") + "\n";
        if (errorCode) {
            message +=
                "\n" + _t("The following error code was given by the printer:") + "\n" + errorCode;

            const extra_messages = {
                DeviceNotFound:
                    _t(
                        "Check on the printer configuration for the 'Device ID' setting. " +
                            "It should be set to: "
                    ) + "\nlocal_printer",
                EPTR_REC_EMPTY: _t("No paper was detected by the printer"),
            };
            if (errorCode in extra_messages) {
                message += "\n" + extra_messages[errorCode];
            }
            message +=
                "\n" +
                _t("To find more details on the error reason, please search online for:") +
                "\n" +
                " Epson Server Direct Print " +
                errorCode;
        } else {
            message += _t("Please check if the printer has enough paper and is ready to print.");
        }
        return {
            successful: false,
            errorCode: errorCode,
            message: {
                title: _t("Printing failed"),
                body: message,
            },
        };
    }
}

```

## File: static\src\app\components\epos_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="pos_epson_printer.ePOSLayout">
        <s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/">
            <s:Body>
                <epos-print xmlns="http://www.epson-pos.com/schemas/2011/03/epos-print" />
            </s:Body>
        </s:Envelope>
    </t>
</templates>

```

## File: static\src\overrides\models\pos_store.js

```javascript
import { PosStore } from "@point_of_sale/app/store/pos_store";
import { EpsonPrinter } from "@pos_epson_printer/app/epson_printer";
import { patch } from "@web/core/utils/patch";

patch(PosStore.prototype, {
    afterProcessServerData() {
        var self = this;
        return super.afterProcessServerData(...arguments).then(function () {
            if (self.config.other_devices && self.config.epson_printer_ip) {
                self.hardwareProxy.printer = new EpsonPrinter({ ip: self.config.epson_printer_ip });
            }
        });
    },
    create_printer(config) {
        if (config.printer_type === "epson_epos") {
            return new EpsonPrinter({ ip: config.epson_printer_ip });
        } else {
            return super.create_printer(...arguments);
        }
    },
});

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
            <xpath expr="//setting[@id='other_devices']" position="inside">
                <div class="content-group" invisible="not other_devices">
                    <field name="epson_printer_ip" placeholder="Epson Receipt Printer IP Address" />
                    <div class="row" invisible="epson_printer_ip in [False, '']">
                        <div class="col-lg-5 o_light_label">
                            <label string="Cashdrawer" for="iface_cashdrawer"/>
                        </div>
                        <div class="col-lg-7">
                            <field name="iface_cashdrawer"/>
                        </div>
                    </div>
                </div>
                <div role="alert" class="alert alert-warning" invisible="not iface_print_via_proxy or not other_devices or epson_printer_ip in [False, '']">
                    The Epson receipt printer will be used instead of the receipt printer connected to the IoT Box.
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\pos_printer_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_pos_printer_form" model="ir.ui.view">
        <field name="name">pos.iot.config.form.view</field>
        <field name="model">pos.printer</field>
        <field name="inherit_id" ref="point_of_sale.view_pos_printer_form"/>
        <field name="arch" type="xml">
            <field name="printer_type" position="after">
                <field name="epson_printer_ip" invisible="printer_type != 'epson_epos'"/>
            </field>
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
            <xpath expr="//setting[@id='pos_other_devices']" position="inside">
                <div class="content-group" invisible="not pos_other_devices">
                    <field name="pos_epson_printer_ip" placeholder="Epson Receipt Printer IP Address" />
                    <div class="row" invisible="pos_epson_printer_ip in [False, '']">
                        <label string="Cashdrawer" for="pos_iface_cashdrawer" class="col-lg-3 o_light_label"/>
                        <field name="pos_iface_cashdrawer"/>
                    </div>
                </div>
                <div role="alert" class="alert alert-warning" invisible="not pos_iface_print_via_proxy or not pos_other_devices or pos_epson_printer_ip in [False, '']">
                    The Epson receipt printer will be used instead of the receipt printer connected to the IoT Box.
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

