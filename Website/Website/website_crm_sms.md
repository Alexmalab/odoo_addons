# Odoo Module: website_crm_sms

Category: Website/Website

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
    'name': 'Send SMS to Visitor with leads',
    'category': 'Website/Website',
    'sequence': 54,
    'summary': 'Allows to send sms to website visitor that have lead',
    'version': '1.0',
    'description': """Allows to send sms to website visitor if the visitor is linked to a lead.""",
    'depends': ['website_sms', 'crm'],
    'data': [],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\website_visitor.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class WebsiteVisitor(models.Model):
    _inherit = 'website.visitor'

    def _prepare_visitor_send_sms_values(self):
        visitor_sms_values = super(WebsiteVisitor, self)._prepare_visitor_send_sms_values()
        if not visitor_sms_values:
            leads_with_number = self.lead_ids.filtered(lambda l: l.mobile == self.mobile or l.phone == self.mobile)._sort_by_confidence_level(reverse=True)
            if leads_with_number:
                lead = leads_with_number[0]
                return {
                    'res_model': 'crm.lead',
                    'res_id': lead.id,
                    'partner_ids': [lead.id],
                    'number_field_name': 'mobile' if lead.mobile == self.mobile else 'phone',
                }
        return visitor_sms_values

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website_visitor

```

