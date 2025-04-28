# Odoo Module: l10n_ar_pos

Category: Accounting/Localizations/Point of Sale

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Argentinean - Point of Sale with AR Doc',
    'version': '1.0',
    'category': 'Accounting/Localizations/Point of Sale',
    'description': """
This module brings the technical requirement for the Argentinean regulation.
Install this if you are using the Point of Sale app in Argentina.
    """,
    'depends': [
        'l10n_ar',
        'point_of_sale',
    ],
    'countries': ['ar'],
    'data': [
        'views/templates.xml',
    ],
    'assets': {
        'point_of_sale._assets_pos': [
            'l10n_ar_pos/static/src/**/*'
        ],
    },
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
from odoo import api, models


class PosSession(models.Model):

    _inherit = 'pos.session'

    @api.model
    def _pos_ui_models_to_load(self):
        res = super()._pos_ui_models_to_load()
        if self.company_id.country_code == 'AR':
            res += ['l10n_ar.afip.responsibility.type', 'l10n_latam.identification.type']
        return res

    def _loader_params_res_partner(self):
        vals = super()._loader_params_res_partner()
        if self.company_id.country_code == 'AR':
            vals['search_params']['fields'] += ['l10n_ar_afip_responsibility_type_id', 'l10n_latam_identification_type_id']
        return vals

    def _pos_data_process(self, loaded_data):
        super()._pos_data_process(loaded_data)
        if self.company_id.country_code == 'AR':
            loaded_data['consumidor_final_anonimo_id'] = self.env.ref('l10n_ar.par_cfa').id

    def _get_pos_ui_l10n_ar_afip_responsibility_type(self, params):
        return self.env['l10n_ar.afip.responsibility.type'].search_read(**params['search_params'])

    def _loader_params_l10n_ar_afip_responsibility_type(self):
        return {'search_params': {'domain': [], 'fields': ['name']}}

    def _get_pos_ui_l10n_latam_identification_type(self, params):
        return self.env['l10n_latam.identification.type'].search_read(**params['search_params'])

    def _loader_params_l10n_latam_identification_type(self):
        """ filter only identification types used in Argentina"""
        return {
            'search_params': {
                'domain': [('l10n_ar_afip_code', '!=', False), ('active', '=', True)],
                'fields': ['name']},
        }

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
from odoo import models, api, _
from odoo.exceptions import UserError


class ResPartner(models.Model):

    _inherit = 'res.partner'

    @api.ondelete(at_uninstall=False)
    def _ar_unlink_except_master_data(self):
        consumidor_final_anonimo = self.env.ref('l10n_ar.par_cfa').id
        for partner in self.ids:
            if partner == consumidor_final_anonimo:
                raise UserError(_('Deleting this partner is not allowed.'))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
from . import pos_session
from . import res_partner

```

## File: static\src\overrides\components\partner_editor\partner_editor.js

```javascript
/** @odoo-module */

import { PartnerDetailsEdit } from "@point_of_sale/app/screens/partner_list/partner_editor/partner_editor";
import { patch } from "@web/core/utils/patch";

patch(PartnerDetailsEdit.prototype, {
    setup(){
        super.setup(...arguments);
        if (this.pos.isArgentineanCompany()) {
            this.intFields.push("l10n_ar_afip_responsibility_type_id");
            this.intFields.push("l10n_latam_identification_type_id");
            this.changes.l10n_ar_afip_responsibility_type_id =
                this.props.partner.l10n_ar_afip_responsibility_type_id &&
                this.props.partner.l10n_ar_afip_responsibility_type_id[0];
            this.changes.l10n_latam_identification_type_id =
                this.props.partner.l10n_latam_identification_type_id &&
                this.props.partner.l10n_latam_identification_type_id[0];
            }
        },
});

```

## File: static\src\overrides\components\partner_editor\partner_editor.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="PartnerDetailsEdit" t-inherit="point_of_sale.PartnerDetailsEdit" t-inherit-mode="extension">
        <xpath expr="//div[hasclass('partner-details-box')]" position="inside">
            <t t-if="pos.isArgentineanCompany()">

                <div class="partner-detail col">
                    <label class="form-label" for="art">AFIP Responsibility</label>
                    <select class="detail form-select" name="l10n_ar_afip_responsibility_type_id" id="art"
                        t-model="changes.l10n_ar_afip_responsibility_type_id">
                        <t t-foreach="pos.l10n_ar_afip_responsibility_types" t-as="afip_responsibility" t-key="afip_responsibility.id">
                            <option t-att-value="afip_responsibility.id">
                                <t t-esc="afip_responsibility.name" />
                            </option>
                        </t>
                    </select>
                </div>

                <div class="partner-detail col">
                    <label class="form-label" for="id_type">Identification Type</label>
                    <select class="detail form-select" name="l10n_latam_identification_type_id" id="id_type"
                        t-model="changes.l10n_latam_identification_type_id">
                        <t t-foreach="pos.l10n_latam_identification_types" t-as="l10n_latam_identification_type" t-key="l10n_latam_identification_type.id">
                            <option t-att-value="l10n_latam_identification_type.id">
                                <t t-esc="l10n_latam_identification_type.name" />
                            </option>
                        </t>
                    </select>
                </div>
            </t>
        </xpath>
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
        super.createPartner(...arguments);
        if (this.pos.isArgentineanCompany()) {
            this.state.editModeProps.partner.l10n_latam_identification_type_id = [
                this.pos.l10n_latam_identification_types[0].id,
                this.pos.l10n_latam_identification_types[0].name,
            ];
            this.state.editModeProps.partner.l10n_ar_afip_responsibility_type_id = [
                this.pos.l10n_ar_afip_responsibility_types[0].id,
                this.pos.l10n_ar_afip_responsibility_types[0].name,
            ];
        }
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
        if (this.pos.isArgentineanCompany()) {
            if (
                partner &&
                (!destinationOrder.get_partner() ||
                    destinationOrder.get_partner().id === this.pos.consumidorFinalAnonimoId)
            ) {
                destinationOrder.set_partner(partner);
            }
        } else {
            super.setPartnerToRefundOrder(...arguments);
        }
    },
});

```

## File: static\src\overrides\models\pos_store.js

```javascript
/** @odoo-module */

import { PosStore } from "@point_of_sale/app/store/pos_store";
import { Order } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(PosStore.prototype, {
    // @Override
    async _processData(loadedData) {
        await super._processData(...arguments);
        if (this.isArgentineanCompany()) {
            this.l10n_ar_afip_responsibility_types = loadedData["l10n_ar.afip.responsibility.type"];
            this.l10n_latam_identification_types = loadedData["l10n_latam.identification.type"];
            this.consumidorFinalAnonimoId = loadedData["consumidor_final_anonimo_id"];
        }
    },
    isArgentineanCompany() {
        return this.company.country?.code == "AR";
    },
});

patch(Order.prototype, {
    setup() {
        super.setup(...arguments);
        if (this.pos.isArgentineanCompany()) {
            if (!this.partner) {
                this.partner = this.pos.db.partner_by_id[this.pos.consumidorFinalAnonimoId];
            }
        }
    },
});

```

## File: views\templates.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>

    <template id="ticket_validation_screen" inherit_id="point_of_sale.ticket_validation_screen">

        <!-- We do not let the user create the invoice by then self. That is why we hide the "Get my Invoice" button -->
        <xpath expr="//div[@id='get_my_invoice']" position="attributes">
            <attribute name="t-if">pos_order.company_id.country_code != 'AR'</attribute>
        </xpath>
        <xpath expr="//div[@id='get_my_invoice']/.." position="after">
            <h4 class="text-danger" t-if="pos_order.company_id.country_code == 'AR' and not pos_order.is_invoiced">Invoice not available. You can contact us for more info</h4>
        </xpath>

        <!-- If the user is NOT log in we should hide the partner form, this one lost functionality because the get my invoice button is not shown and then the partner info is never save. In they want they can sign up and fill data from there -->
        <xpath expr="//div[hasclass('o_portal_details')]" position="attributes">
            <attribute name="t-if">pos_order.company_id.country_code != 'AR'</attribute>
        </xpath>
        <xpath expr="//div[@id='get_info_div']" position="attributes">
            <attribute name="t-if">pos_order.company_id.country_code != 'AR'</attribute>
        </xpath>
        <xpath expr="//div[@id='get_info_div']" position="after">
                <t t-elif="pos_order.company_id.country_code == 'AR'">
                    <h4> Please
                    <a role="button" t-att-href="'/web/login?redirect=/pos/ticket/validate?access_token=%s' % access_token" style="margin-top: -11px"> Sign in</a>
                    in order to save your contact info</h4>
                </t>
        </xpath>

    </template>

</odoo>

```

