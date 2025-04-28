# Odoo Module: l10n_be_invoice_bba

Category: Accounting/Accounting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Noviat nv/sa (www.noviat.be). All rights reserved.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Noviat nv/sa (www.noviat.be). All rights reserved.

{
    'name': 'Belgium - Structured Communication',
    'version': '1.2',
    'author': 'Noviat',
    'category': 'Accounting/Accounting',
    'description': """

Add Structured Communication to customer invoices.
--------------------------------------------------

Using BBA structured communication simplifies the reconciliation between invoices and payments.
You can select the structured communication as payment communication in Invoicing/Accounting settings.
Two algorithms are suggested:

    1) Invoice Number +++RRR/RRRR/RRRDD+++
        **R..R =** Invoice Number, **DD =** Check Digits
    2) Customer Reference +++RRR/RRRR/SSSDD+++
        **R..R =** Customer Reference without non-numeric characters, **SSS =** Sequence Number, **DD =** Check Digits
    """,
    'depends': ['account', 'l10n_be'],
    'data': [
        'data/mail_template_data.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="account.email_template_edi_invoice" model="mail.template">
            <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear ${object.partner_id.name}
        % if object.partner_id.parent_id:
            (${object.partner_id.parent_id.name})
        % endif
        <br /><br />
        Here is, in attachment, your 
        % if object.name:
            invoice <strong>${object.name}</strong>
        % else:
            invoice
        % endif
        % if object.invoice_origin:
            (with reference: ${object.invoice_origin} )
        % endif
        amounting in <strong>${format_amount(object.amount_total, object.currency_id)}</strong>
        from ${object.company_id.name}.
        % if object.invoice_payment_state=='paid':
            This invoice is already paid.
        % else:
            Please remit payment at your earliest convenience.
            % if object.invoice_reference_type=='structured' and object.ref:
                Please use the following communication for your payment: <strong>${object.ref}</strong>.
            % endif
        % endif
        <br /><br />
        Do not hesitate to contact us if you have any questions.
    </p>
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Noviat nv/sa (www.noviat.be). All rights reserved.

import random
import re

from odoo import api, fields, models, _
from odoo.exceptions import UserError

"""
account.move object: add support for Belgian structured communication
"""


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _get_invoice_reference_be_partner(self):
        """ This computes the reference based on the belgian national standard
            “OGM-VCS”.
            For instance, if an invoice is issued for the partner with internal
            reference 'food buyer 654', the digits will be extracted and used as
            the data. This will lead to a check number equal to 72 and the
            reference will be '+++000/0000/65472+++'.
            If no reference is set for the partner, its id in the database will
            be used.
        """
        self.ensure_one()
        bbacomm = (re.sub('\D', '', self.partner_id.ref or '') or str(self.partner_id.id))[-10:].rjust(10, '0')
        base = int(bbacomm)
        mod = base % 97 or 97
        reference = '+++%s/%s/%s%02d+++' % (bbacomm[:3], bbacomm[3:7], bbacomm[7:], mod)
        return reference

    def _get_invoice_reference_be_invoice(self):
        """ This computes the reference based on the belgian national standard
            “OGM-VCS”.
            The data of the reference is the database id number of the invoice.
            For instance, if an invoice is issued with id 654, the check number
            is 72 so the reference will be '+++000/0000/65472+++'.
        """
        self.ensure_one()
        base = self.id
        bbacomm = str(base).rjust(10, '0')
        base = int(bbacomm)
        mod = base % 97 or 97
        reference = '+++%s/%s/%s%02d+++' % (bbacomm[:3], bbacomm[3:7], bbacomm[7:], mod)
        return reference

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[('be', 'Belgium')])

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2011 Noviat nv/sa (www.noviat.be). All rights reserved.

from . import account_invoice
from . import account_journal
```

