# Odoo Module: l10n_pe_pos

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    "name": "Peruvian - Point of Sale with Pe Doc",
    "version": "1.0",
    "category": "Accounting/Localizations/Point of Sale",
    "author": "Vauxoo, Odoo S.A.",
    "description": """
This module brings the technical requirement for the Peruvian regulation.
Install this if you are using the Point of Sale app in Peru.
    """,
    "depends": [
        "l10n_pe",
        "point_of_sale",
    ],
    "data": [
        "data/res_partner_data.xml",
        "views/templates.xml",
    ],
    "assets": {
        "point_of_sale._assets_pos": ["l10n_pe_pos/static/src/**/*"],
    },
    "installable": True,
    "auto_install": True,
    "license": "LGPL-3",
}

```

## File: data\res_partner_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo noupdate="1">

    <record id="partner_pe_cf" model="res.partner">
        <field name="name">Consumidor Final</field>
        <field name="l10n_latam_identification_type_id" ref="l10n_pe.it_DNI" />
        <field name="vat">00000000</field>
    </record>

</odoo>

```

## File: models\l10n_latam_identification_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class L10nLatamIdentificationType(models.Model):
    _name = 'l10n_latam.identification.type'
    _inherit = ['l10n_latam.identification.type', 'pos.load.mixin']

    @api.model
    def _load_pos_data_domain(self, data):
        if self.env.company.country_id.code == "PE":
            return [("l10n_pe_vat_code", "!=", False)]
        else:
            return super()._load_pos_data_domain(data)

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['name']

```

## File: models\l10n_pe_res_city_district.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api


class L10nPeResCityDistrict(models.Model):
    _name = "l10n_pe.res.city.district"
    _inherit = ["l10n_pe.res.city.district", "pos.load.mixin"]

    country_id = fields.Many2one(related="city_id.country_id")
    state_id = fields.Many2one(related="city_id.state_id")

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ["name", "city_id", "country_id", "state_id"]

```

## File: models\pos_config.py

```python
from odoo import models


class PosConfig(models.Model):
    _inherit = 'pos.config'

    def get_limited_partners_loading(self):
        partner_ids = super().get_limited_partners_loading()
        if (self.env.ref('l10n_pe_pos.partner_pe_cf').id,) not in partner_ids:
            partner_ids.append((self.env.ref('l10n_pe_pos.partner_pe_cf').id,))
        return partner_ids

```

## File: models\pos_session.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, api


class PosSession(models.Model):
    _inherit = "pos.session"

    @api.model
    def _load_pos_data_models(self, config_id):
        data = super()._load_pos_data_models(config_id)
        if self.env.company.country_id.code == "PE":
            data += ['l10n_pe.res.city.district', 'l10n_latam.identification.type', 'res.city']
        return data

    def _load_pos_data(self, data):
        data = super()._load_pos_data(data)
        if self.env.company.country_id.code == "PE":
            data['data'][0]['_default_l10n_latam_identification_type_id'] = self.env.ref('l10n_pe.it_DNI').id
            data['data'][0]['_consumidor_final_anonimo_id'] = self.env.ref('l10n_pe_pos.partner_pe_cf').id
        return data

```

## File: models\res_city.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class ResCity(models.Model):
    _name = "res.city"
    _inherit = ["res.city", "pos.load.mixin"]

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ["name", "country_id", "state_id"]

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import _, api, models
from odoo.exceptions import UserError


class ResPartner(models.Model):
    _inherit = "res.partner"

    @api.ondelete(at_uninstall=False)
    def _pe_unlink_except_master_data(self):
        consumidor_final_anonimo = self.env.ref("l10n_pe_pos.partner_pe_cf")
        if consumidor_final_anonimo & self:
            raise UserError(
                _(
                    "Deleting the partner %s is not allowed because it is required by the Peruvian point of sale.",
                    consumidor_final_anonimo.display_name,
                )
            )

    @api.model
    def _load_pos_data_fields(self, config_id):
        fields = super()._load_pos_data_fields(config_id)
        if self.env.company.country_id.code == "PE":
            fields += ["city_id", "l10n_latam_identification_type_id", "l10n_pe_district"]
        return fields

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import l10n_pe_res_city_district
from . import pos_session
from . import res_partner
from . import l10n_latam_identification_type
from . import res_city
from . import pos_config

```

## File: static\src\overrides\components\paymentt_screen\payment_screen.js

```javascript
import { _t } from "@web/core/l10n/translation";
import { AlertDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { PaymentScreen } from "@point_of_sale/app/screens/payment_screen/payment_screen";
import { patch } from "@web/core/utils/patch";

patch(PaymentScreen.prototype, {
    async _isOrderValid(isForceValidate) {
        const res = await super._isOrderValid(...arguments);
        if (!this.pos.isPeruvianCompany() && res) {
            return res;
        }
        if (!res) {
            return false;
        }
        const currentPartner = this.currentOrder.get_partner();
        if (currentPartner && !currentPartner.vat) {
            this.pos.editPartner(currentPartner);
            this.dialog.add(AlertDialog, {
                title: _t("Missing Field"),
                body: _t("An Identification Number Is Required"),
            });
            return false;
        }
        return res;
    },
});

```

## File: static\src\overrides\components\ticket_screen\ticket_screen.js

```javascript
import { TicketScreen } from "@point_of_sale/app/screens/ticket_screen/ticket_screen";
import { patch } from "@web/core/utils/patch";

patch(TicketScreen.prototype, {
    setPartnerToRefundOrder(partner, destinationOrder) {
        if (!this.pos.isPeruvianCompany()) {
            return super.setPartnerToRefundOrder(...arguments);
        }
        if (
            partner &&
            (!destinationOrder.get_partner() ||
                destinationOrder.get_partner().id === this.pos.session._consumidor_final_anonimo_id)
        ) {
            return destinationOrder.set_partner(partner);
        }
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
import { PosStore } from "@point_of_sale/app/store/pos_store";
import { patch } from "@web/core/utils/patch";

patch(PosStore.prototype, {
    // @Override
    async processServerData() {
        await super.processServerData(...arguments);
        if (this.isPeruvianCompany()) {
            this["res.city"] = this.data["res.city"];
            this["l10n_latam.identification.type"] = this.data["l10n_latam.identification.type"];
            this["l10n_pe.res.city.district"] = this.data["l10n_pe.res.city.district"];
        }
    },
    isPeruvianCompany() {
        return this.company.country_id?.code == "PE";
    },
    createNewOrder() {
        const order = super.createNewOrder(...arguments);

        if (this.isPeruvianCompany() && !order.partner_id) {
            order.update({ partner_id: this.session._consumidor_final_anonimo_id });
        }

        return order;
    },
});

```

## File: views\templates.xml

```xml
<?xml version='1.0' encoding='utf-8' ?>
<odoo>

    <template id="ticket_validation_screen" inherit_id="point_of_sale.ticket_validation_screen">
        <!-- We do not let the user create the invoice by themselves. That is why we hide the
        "Get my Invoice" button -->
        <xpath expr="//div[@id='get_my_invoice']" position="attributes">
            <attribute name="t-if" add="(pos_order.company_id.country_code != 'PE')" separator="and" />
        </xpath>
        <xpath expr="//div[@id='get_my_invoice']/.." position="after">
            <h4 class="text-danger" t-if="pos_order.company_id.country_code == 'PE' and not pos_order.is_invoiced">
                Invoice not available. You can contact us for more info.
            </h4>
        </xpath>

        <!-- If the user is NOT logged in we should hide the partner form, this one lost functionality
        because the get my invoice button is not shown and then the partner info is never saved. In they
        want they can sign up and fill data from there -->
        <xpath expr="//div[hasclass('o_portal_details')]" position="attributes">
            <attribute name="t-if" add="(pos_order.company_id.country_code != 'PE')" separator="and" />
        </xpath>
        <xpath expr="//div[@id='get_info_div']" position="attributes">
            <attribute name="t-if" add="(pos_order.company_id.country_code != 'PE')" separator="and" />
        </xpath>
        <xpath expr="//div[@id='get_info_div']" position="after">
            <h4 t-elif="pos_order.company_id.country_code == 'PE'">
                Please
                <a
                    role="button"
                    t-att-href="'/web/login?redirect=/pos/ticket/validate?access_token=%s' % access_token"
                    style="margin-top: -11px"
                >
                    Sign in
                </a>
                in order to save your contact info
            </h4>
        </xpath>

        <!-- Show PE fields preview info, and show warning is a required field need to be completed -->
        <xpath expr="//t[@id='partner_vat']" position="before">
            <t t-if="pos_order.company_id.country_code == 'PE'">
                <t t-if="partner.l10n_latam_identification_type_id">
                    <t t-out="partner.l10n_latam_identification_type_id.name" /> -
                </t>
                <t t-else="">
                    <span class="text-danger">* Please configure your Identification Type</span><br />
                </t>
            </t>
        </xpath>
        <xpath expr="//t[@id='partner_vat']" position="after">
            <t t-else="">
                <span class="text-danger">* Please configure your Identification Number</span><br />
            </t>
        </xpath>

    </template>

</odoo>

```

