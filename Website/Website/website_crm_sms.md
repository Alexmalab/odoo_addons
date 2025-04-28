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

    def _check_for_sms_composer(self):
        check = super(WebsiteVisitor, self)._check_for_sms_composer()
        if not check and self.lead_ids:
            sorted_leads = self.lead_ids.filtered(lambda l: l.mobile == self.mobile or l.phone == self.mobile)._sort_by_confidence_level(reverse=True)
            if sorted_leads:
                return True
        return check

    def _prepare_sms_composer_context(self):
        if not self.partner_id and self.lead_ids:
            leads_with_number = self.lead_ids.filtered(lambda l: l.mobile == self.mobile or l.phone == self.mobile)._sort_by_confidence_level(reverse=True)
            if leads_with_number:
                lead = leads_with_number[0]
                return {
                    'default_res_model': 'crm.lead',
                    'default_res_id': lead.id,
                    'number_field_name': 'mobile' if lead.mobile == self.mobile else 'phone',
                }
        return super(WebsiteVisitor, self)._prepare_sms_composer_context()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import website_visitor

```

