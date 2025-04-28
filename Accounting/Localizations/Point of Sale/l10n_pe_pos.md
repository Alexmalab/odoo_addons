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
    "countries": [
        "pe",
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

## File: models\l10n_pe_res_city_district.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class L10nPeResCityDistrict(models.Model):
    _inherit = "l10n_pe.res.city.district"

    country_id = fields.Many2one(related="city_id.country_id")
    state_id = fields.Many2one(related="city_id.state_id")

```

## File: models\pos_session.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class PosSession(models.Model):
    _inherit = "pos.session"

    def _is_pe_company(self):
        return self.company_id.country_code == "PE"

    def _pos_ui_models_to_load(self):
        res = super()._pos_ui_models_to_load()
        if self._is_pe_company():
            models = ["l10n_latam.identification.type", "l10n_pe.res.city.district", "res.city"]
            res += [model for model in models if model not in res]
        return res

    def _loader_params_res_partner(self):
        vals = super()._loader_params_res_partner()
        if self._is_pe_company():
            vals["search_params"]["fields"] += ["city_id", "l10n_latam_identification_type_id", "l10n_pe_district"]
        return vals

    def _pos_data_process(self, loaded_data):
        res = super()._pos_data_process(loaded_data)
        if self._is_pe_company():
            loaded_data["consumidor_final_anonimo_id"] = self.env.ref("l10n_pe_pos.partner_pe_cf").id
        return res

    def _get_pos_ui_res_city(self, params):
        return self.env["res.city"].search_read(**params["search_params"])

    def _loader_params_res_city(self):
        return {"search_params": {"domain": [], "fields": ["name", "country_id", "state_id"]}}

    def _get_pos_ui_l10n_pe_res_city_district(self, params):
        return self.env["l10n_pe.res.city.district"].search_read(**params["search_params"])

    def _loader_params_l10n_pe_res_city_district(self):
        return {"search_params": {"domain": [], "fields": ["name", "city_id", "country_id", "state_id"]}}

    def _get_pos_ui_l10n_latam_identification_type(self, params):
        return self.env["l10n_latam.identification.type"].search_read(**params["search_params"])

    def _loader_params_l10n_latam_identification_type(self):
        """filter only identification types used in Peru"""
        return {
            "search_params": {
                "domain": [
                    ("l10n_pe_vat_code", "!=", False),
                    ("active", "=", True),
                ],
                "fields": ["name"],
            },
        }

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

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import l10n_pe_res_city_district
from . import pos_session
from . import res_partner

```

## File: static\src\overrides\components\partner_editor\partner_editor.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
import { PartnerDetailsEdit } from "@point_of_sale/app/screens/partner_list/partner_editor/partner_editor";
import { patch } from "@web/core/utils/patch";

patch(PartnerDetailsEdit.prototype, {
    setup() {
        const res = super.setup(...arguments);
        if (!this.pos.isPeruvianCompany()) {
            return res;
        }
        this.intFields.push("city_id", "l10n_latam_identification_type_id", "l10n_pe_district");
        this.changes.city_id = this.props.partner.city_id && this.props.partner.city_id[0];
        this.changes.l10n_latam_identification_type_id =
            this.props.partner.l10n_latam_identification_type_id &&
            this.props.partner.l10n_latam_identification_type_id[0];
        this.changes.l10n_pe_district = this.props.partner.l10n_pe_district && this.props.partner.l10n_pe_district[0];
        return res;
    },
    saveChanges() {
        if (this.pos.isPeruvianCompany() && (!this.props.partner.vat && !this.changes.vat)) {
            return this.popup.add(ErrorPopup, {
                title: _t("Missing Field"),
                body: _t("A Identification Number Is Required"),
            });
        }
        return super.saveChanges(...arguments);
    },
});

```

## File: static\src\overrides\components\partner_editor\partner_editor.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<templates id="template" xml:space="preserve">

    <t t-name="l10n_pe_pos.PartnerDetailsEdit" t-inherit="point_of_sale.PartnerDetailsEdit" t-inherit-mode="extension">
        <xpath expr="//div[input[@t-attf-name='{{item}}']]" position="attributes">
            <attribute name="t-if">!pos.isPeruvianCompany() || item != 'City'</attribute>
        </xpath>

        <xpath expr="//div[select[@name='country_id']]" position="after">
            <xpath expr="//div[select[@name='state_id']]" position="move" />

            <t t-if="pos.isPeruvianCompany()">
                <div class="partner-detail col">
                    <label class="form-label label" for="city">City</label>
                    <select
                        id="city"
                        name="city_id"
                        class="detail form-select"
                        t-model="changes.city_id"
                        t-att-class="{'border-danger': missingFields.includes('city_id')}"
                    >
                        <option value="">None</option>
                        <option
                            t-foreach="pos.cities"
                            t-as="city"
                            t-key="city.id"
                            t-if="changes.state_id == city.state_id[0] and changes.country_id == city.country_id[0]"
                            t-att-value="city.id"
                            t-out="city.name"
                        />
                    </select>
                </div>

                <div class="partner-detail col">
                    <label class="form-label label" for="district">District</label>
                    <select
                        id="district"
                        name="l10n_pe_district"
                        class="detail form-select"
                        t-model="changes.l10n_pe_district"
                        t-att-class="{'border-danger': missingFields.includes('l10n_pe_district')}"
                    >
                        <option value="">None</option>
                        <option
                            t-foreach="pos.l10n_pe_districts"
                            t-as="district"
                            t-key="district.id"
                            t-if="changes.city_id == district.city_id[0] and changes.state_id == district.state_id[0] and changes.country_id == district.country_id[0]"
                            t-att-value="district.id"
                            t-out="district.name"
                        />
                    </select>
                </div>

                <div class="partner-detail col">
                <label class="form-label label" for="identification_type">Identification Type</label>
                    <select
                        id="identification_type"
                        class="detail form-select"
                        name="l10n_latam_identification_type_id"
                        t-model="changes.l10n_latam_identification_type_id"
                        t-att-class="{'border-danger': missingFields.includes('l10n_latam_identification_type_id')}"
                    >
                        <option
                            t-foreach="pos.l10n_latam_identification_types"
                            t-as="l10n_latam_identification_type"
                            t-key="l10n_latam_identification_type.id"
                            t-att-value="l10n_latam_identification_type.id"
                            t-out="l10n_latam_identification_type.name"
                        />
                    </select>
                </div>
            </t>
        </xpath>
        <label for="vat" position="attributes">
            <attribute name="t-if">!pos.isPeruvianCompany()</attribute>
        </label>
        <label for="vat" position="after">
            <label t-if="pos.isPeruvianCompany()" class="form-label label" for="vat">Identification Number</label>
        </label>
    </t>

</templates>

```

## File: static\src\overrides\components\partner_list\partner_list.js

```javascript
/** @odoo-module */

import { PartnerListScreen } from "@point_of_sale/app/screens/partner_list/partner_list";
import { patch } from "@web/core/utils/patch";

patch(PartnerListScreen.prototype, {
    createPartner() {
        const res = super.createPartner(...arguments);
        if (!this.pos.isPeruvianCompany()) {
            return res;
        }
        this.state.editModeProps.partner.city_id = [this.pos.cities[0].id, this.pos.cities[0].name];
        this.state.editModeProps.partner.l10n_latam_identification_type_id = [
            this.pos.l10n_latam_identification_types[0].id,
            this.pos.l10n_latam_identification_types[0].name,
        ];
        this.state.editModeProps.partner.l10n_pe_district = [
            this.pos.l10n_pe_districts[0].id,
            this.pos.l10n_pe_districts[0].name,
        ];
        return res;
    },
});

```

## File: static\src\overrides\components\paymentt_screen\payment_screen.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { ErrorPopup } from "@point_of_sale/app/errors/popups/error_popup";
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
            this.popup.add(ErrorPopup, {
                title: _t("Missing Field"),
                body: _t("A Identification Number Is Required"),
            });
            this.selectPartner(true, ["vat"]);
            return false;
        }
        return res;
    },
});

```

## File: static\src\overrides\components\ticket_screen\ticket_screen.js

```javascript
/** @odoo-module */

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
                destinationOrder.get_partner().id === this.pos.consumidorFinalAnonimoId)
        ) {
            return destinationOrder.set_partner(partner);
        }
    },
});

```

## File: static\src\overrides\models\pos_order.js

```javascript
/** @odoo-module */

import { Order } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(Order.prototype, {
    setup() {
        super.setup(...arguments);
        if (this.pos.isPeruvianCompany() && !this.partner) {
            this.partner = this.pos.db.partner_by_id[this.pos.consumidorFinalAnonimoId];
        }
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
/** @odoo-module */

import { PosStore } from "@point_of_sale/app/store/pos_store";
import { patch } from "@web/core/utils/patch";

patch(PosStore.prototype, {
    // @Override
    async _processData(loadedData) {
        await super._processData(...arguments);
        if (this.isPeruvianCompany()) {
            this.cities = loadedData["res.city"];
            this.consumidorFinalAnonimoId = loadedData["consumidor_final_anonimo_id"];
            this.l10n_latam_identification_types = loadedData["l10n_latam.identification.type"];
            this.l10n_pe_districts = loadedData["l10n_pe.res.city.district"];
        }
    },
    isPeruvianCompany() {
        return this.company.country?.code == "PE";
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
            <attribute name="t-if" add="(pos_order.company_id.country_code != 'PE')" separator=" and " />
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
            <attribute name="t-if" add="(pos_order.company_id.country_code != 'PE')" separator=" and " />
        </xpath>
        <xpath expr="//div[@id='get_info_div']" position="attributes">
            <attribute name="t-if" add="(pos_order.company_id.country_code != 'PE')" separator=" and " />
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

