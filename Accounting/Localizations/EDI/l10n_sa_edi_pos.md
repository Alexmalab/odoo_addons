# Odoo Module: l10n_sa_edi_pos

Category: Accounting/Localizations/EDI

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
    'name': 'Saudi Arabia - E-invoicing (Simplified)',
    'icon': '/l10n_sa/static/description/icon.png',
    'version': '0.1',
    'depends': [
        'l10n_sa_pos',
        'l10n_sa_edi',
    ],
    'author': 'Odoo S.A.',
    'summary': """
        ZATCA E-Invoicing, support for PoS
    """,
    'description': """
E-invoice implementation for Saudi Arabia; Integration with ZATCA (POS)
    """,
    'category': 'Accounting/Localizations/EDI',
    'license': 'LGPL-3',
    'assets': {
        'point_of_sale.assets': [
            'l10n_sa_edi_pos/static/src/js/pos_models.js',
            'l10n_sa_edi_pos/static/src/js/PaymentScreen.js',
        ],
    }
}

```

## File: models\account_edi_xml_ubl_21_zatca.py

```python
# -*- coding: utf-8 -*-
from odoo import models


class AccountEdiXmlUBL21Zatca(models.AbstractModel):
    _inherit = "account.edi.xml.ubl_21.zatca"

    def _l10n_sa_get_payment_means_code(self, invoice):
        """
            Return payment means code to be used to set the value on the XML file
        """
        res = super()._l10n_sa_get_payment_means_code(invoice)
        if invoice._l10n_sa_is_simplified() and invoice.sudo().pos_order_ids.payment_ids:
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
from odoo import models, api, _
from odoo.exceptions import ValidationError


class PosConfig(models.Model):
    _inherit = 'pos.config'

    @api.constrains('company_id', 'invoice_journal_id')
    def _check_company_invoice_journal(self):
        """
            Override to make sure POS invoice journal was probably onboarded before being used
        """
        super()._check_company_invoice_journal()
        for config in self:
            if config.company_id.country_id.code == 'SA' and config.invoice_journal_id and not config.invoice_journal_id._l10n_sa_ready_to_submit_einvoices():
                raise ValidationError(_("The invoice journal of the point of sale %s must be properly onboarded according to ZATCA specifications.", config.name))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
from . import pos_config
from . import account_edi_xml_ubl_21_zatca
from . import account_move

```

## File: static\src\js\PaymentScreen.js

```javascript
odoo.define('l10n_sa_edi_pos.PaymentScreen', function(require) {
    "use strict";

    const PaymentScreen = require('point_of_sale.PaymentScreen');
    const Registries = require('point_of_sale.Registries');


    const PosSAPaymentScreen = PaymentScreen => class extends PaymentScreen {
        //@Override
        toggleIsToInvoice() {
            // If the company is Saudi, POS orders should always be Invoiced
            if (this.currentOrder.pos.company.country && this.currentOrder.pos.company.country.code === 'SA') return false
            return super.toggleIsToInvoice(...arguments);
        }
    };

    Registries.Component.extend(PaymentScreen, PosSAPaymentScreen);

    return PosSAPaymentScreen;
})
```

## File: static\src\js\pos_models.js

```javascript
odoo.define("l10n_sa_edi_pos.models", function (require) {
    "use strict";

    const { Order } = require('point_of_sale.models');
    const Registries = require('point_of_sale.Registries');

    const L10nSAPosOrder = (Order) => class L10nSAPosOrder extends Order {
        constructor() {
            super(...arguments);
            if (this.pos.company.country && this.pos.company.country.code === 'SA') {
                this.set_to_invoice(true);
            }
        }
    }

    Registries.Model.extend(Order, L10nSAPosOrder);
});

```

