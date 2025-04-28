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

## File: models\l10n_ar_afip_responsibility_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class L10nArAfipResponsibilityType(models.Model):
    _name = 'l10n_ar.afip.responsibility.type'
    _inherit = ['l10n_ar.afip.responsibility.type', 'pos.load.mixin']

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['name']

```

## File: models\l10n_latam_identification_type.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class L10nArAfipResponsibilityType(models.Model):
    _name = 'l10n_latam.identification.type'
    _inherit = ['l10n_latam.identification.type', 'pos.load.mixin']

    @api.model
    def _load_pos_data_domain(self, data):
        if self.env.company.country_id.code == "AR":
            return [('l10n_ar_afip_code', '!=', False), ('active', '=', True)]
        else:
            return super()._load_pos_data_domain(data)

    @api.model
    def _load_pos_data_fields(self, config_id):
        return ['name']

```

## File: models\pos_config.py

```python
from odoo import models


class PosConfig(models.Model):
    _inherit = 'pos.config'

    def get_limited_partners_loading(self):
        partner_ids = super().get_limited_partners_loading()
        if (self.env.ref('l10n_ar.par_cfa').id,) not in partner_ids:
            partner_ids.append((self.env.ref('l10n_ar.par_cfa').id,))
        return partner_ids

```

## File: models\pos_session.py

```python
# -*- coding: utf-8 -*-
from odoo import models, api


class PosSession(models.Model):
    _inherit = 'pos.session'

    @api.model
    def _load_pos_data_models(self, config_id):
        data = super()._load_pos_data_models(config_id)
        if self.env.company.country_id.code == 'AR':
            data += ['l10n_ar.afip.responsibility.type', 'l10n_latam.identification.type']
        return data

    def _load_pos_data(self, data):
        data = super()._load_pos_data(data)
        if self.env.company.country_id.code == 'AR':
            data['data'][0]['_consumidor_final_anonimo_id'] = self.env.ref('l10n_ar.par_cfa').id
        return data

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

    @api.model
    def _load_pos_data_fields(self, config_id):
        params = super()._load_pos_data_fields(config_id)
        if self.env.company.country_id.code == 'AR':
            params += ['l10n_ar_afip_responsibility_type_id', 'l10n_latam_identification_type_id']
        return params

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
from . import pos_session
from . import res_partner
from . import l10n_ar_afip_responsibility_type
from . import l10n_latam_identification_type
from . import pos_config

```

## File: static\src\overrides\components\product_screen\product_screen.js

```javascript
import { ProductScreen } from "@point_of_sale/app/screens/product_screen/product_screen";
import { patch } from "@web/core/utils/patch";
import { onMounted } from "@odoo/owl";

patch(ProductScreen.prototype, {
    setup() {
        super.setup(...arguments);

        onMounted(() => {
            if (this.pos.isArgentineanCompany() && !this.pos.get_order().partner_id) {
                this.pos.get_order().update({
                    partner_id: this.pos.session._consumidor_final_anonimo_id,
                });
            }
        });
    },
});

```

## File: static\src\overrides\components\ticket_screen\ticket_screen.js

```javascript
import { TicketScreen } from "@point_of_sale/app/screens/ticket_screen/ticket_screen";
import { patch } from "@web/core/utils/patch";

patch(TicketScreen.prototype, {
    setPartnerToRefundOrder(partner, destinationOrder) {
        if (this.pos.isArgentineanCompany()) {
            if (
                partner &&
                (!destinationOrder.get_partner() ||
                    destinationOrder.get_partner().id ===
                        this.pos.session._consumidor_final_anonimo_id)
            ) {
                destinationOrder.set_partner(partner);
            }
        } else {
            super.setPartnerToRefundOrder(...arguments);
        }
    },
});

```

## File: static\src\overrides\models\pos_order.js

```javascript
import { PosOrder } from "@point_of_sale/app/models/pos_order";
import { patch } from "@web/core/utils/patch";

patch(PosOrder.prototype, {
    setup() {
        super.setup(...arguments);
        if (this.isArgentineanCompany()) {
            if (!this.partner_id) {
                this.update({ partner_id: this.session._consumidor_final_anonimo_id });
            }
        }
    },
    isArgentineanCompany() {
        return this.company.country_id?.code == "AR";
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
        await super.processServerData();

        if (this.isArgentineanCompany()) {
            this["l10n_latam.identification.type"] =
                this.models["l10n_latam.identification.type"].getFirst();
            this["l10n_ar.afip.responsibility.type"] =
                this.models["l10n_ar.afip.responsibility.type"].getFirst();
        }
    },
    isArgentineanCompany() {
        return this.company.country_id?.code == "AR";
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

