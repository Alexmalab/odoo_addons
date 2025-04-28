# Odoo Module: website_event_crm_questions

Category: Marketing/Events

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import tests

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Website Event CRM Questions',
    'version': '1.0',
    'category': 'Marketing/Events',
    'website': 'https://www.odoo.com/app/events',
    'description': """
        Add information when we build the description of a lead to include
        the questions and answers linked to the registrations.
    """,
    'depends': ['website_event_crm', 'website_event_questions'],
    'data': [],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: models\event_registration.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from markupsafe import Markup


class EventRegistration(models.Model):
    _inherit = 'event.registration'

    def _get_lead_description_registration(self, line_suffix=''):
        """Add the questions and answers linked to the registrations into the description of the lead."""
        reg_description = super(EventRegistration, self)._get_lead_description_registration(line_suffix=line_suffix)
        if not self.registration_answer_ids:
            return reg_description

        answer_descriptions = []
        for answer in self.registration_answer_ids:
            answer_value = answer.value_answer_id.name if answer.question_type == "simple_choice" else answer.value_text_box
            answer_value = Markup("<br/>").join(["    %s" % line for line in answer_value.split('\n')])
            answer_descriptions.append(Markup("  - %s<br/>%s") % (answer.question_id.title, answer_value))
        return Markup("%s%s<br/>%s") % (reg_description, _("Questions"), Markup('<br/>').join(answer_descriptions))

    def _get_lead_description_fields(self):
        res = super(EventRegistration, self)._get_lead_description_fields()
        res.append('registration_answer_ids')
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_registration

```

