# Odoo Module: l10n_sa_edi_pos

Category: Accounting/Localizations/EDI

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Saudi Arabia - E-invoicing (Simplified)',
    'countries': ['sa'],
    'version': '0.1',
    'depends': [
        'l10n_sa_pos',
        'l10n_sa_edi',
    ],
    'summary': """
        ZATCA E-Invoicing, support for PoS
    """,
    'description': """
E-invoice implementation for Saudi Arabia; Integration with ZATCA (POS)
    """,
    'category': 'Accounting/Localizations/EDI',
    'license': 'LGPL-3',
    'assets': {
        'point_of_sale._assets_pos': [
            'l10n_sa_edi_pos/static/src/overrides/**/*.js',
        ],
    }
}

```

## File: models\account_edi_xml_ubl_21_zatca.py

```python
from odoo import models


class AccountEdiXmlUBL21Zatca(models.AbstractModel):
    _inherit = "account.edi.xml.ubl_21.zatca"

    def _l10n_sa_get_payment_means_code(self, invoice):
        """
            Return payment means code to be used to set the value on the XML file
        """
        res = super()._l10n_sa_get_payment_means_code(invoice)
        if invoice._l10n_sa_is_simplified() and invoice.sudo().pos_order_ids:
            res = invoice.sudo().pos_order_ids.payment_ids[0].payment_method_id.type
        return res

```

## File: models\account_move.py

```python
from odoo import models


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _l10n_sa_check_refund_reason(self):
        return super()._l10n_sa_check_refund_reason() or (self.pos_order_ids and self.pos_order_ids[0].refunded_orders_count > 0 and self.ref)

```

## File: models\pos_config.py

```python
from odoo import models, _
from odoo.exceptions import RedirectWarning


class PosConfig(models.Model):
    _inherit = 'pos.config'

    def open_ui(self):
        for config in self:
            if (
                    config.company_id.country_id.code == 'SA'
                    and config.invoice_journal_id
                    and (config.invoice_journal_id.edi_format_ids.filtered(lambda f: f.code == "sa_zatca")
                         and not config.invoice_journal_id._l10n_sa_ready_to_submit_einvoices())
            ):
                msg = _("The invoice journal of the point of sale %s must be properly onboarded "
                        "according to ZATCA specifications.\n", config.name)
                action = {
                    "view_mode": "form",
                    "res_model": "account.journal",
                    "type": "ir.actions.act_window",
                    "res_id": config.invoice_journal_id.id,
                    "views": [[False, "form"]],
                }
                raise RedirectWarning(msg, action, _('Go to Journal configuration'))
        return super().open_ui()

```

## File: models\__init__.py

```python
from . import pos_config
from . import account_edi_xml_ubl_21_zatca
from . import account_move

```

## File: static\src\overrides\models\models.js

```javascript
/** @odoo-module */

import { Order } from "@point_of_sale/app/store/models";
import { patch } from "@web/core/utils/patch";

patch(Order.prototype, {
    setup(_defaultObj, options) {
        super.setup(...arguments);
        if (this.pos.isSACompany) {
            this.to_invoice = true;
        }
    },
    is_to_invoice() {
        if (this.pos.isSACompany) {
            return true;
        }
        return super.is_to_invoice(...arguments);
    },
    set_to_invoice(to_invoice) {
        if (this.pos.isSACompany) {
            this.assert_editable();
            this.to_invoice = true;
        } else {
            super.set_to_invoice(...arguments);
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
    get isSACompany() {
        return this.company.country?.code == "SA";
    },
});

```

