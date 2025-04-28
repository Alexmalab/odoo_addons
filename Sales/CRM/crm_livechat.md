# Odoo Module: crm_livechat

Category: Sales/CRM

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
    'name': 'CRM Livechat',
    'category': 'Sales/CRM',
    'summary': 'Create lead from livechat conversation',
    'data': [
        'data/utm_data.xml',
    ],
    'depends': [
        'crm',
        'im_livechat'
    ],
    'description': 'Create new lead with using /lead command in the channel',
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\utm_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="utm.source" id="utm_source_livechat">
        <field name="name">Livechat</field>
    </record>
</odoo>

```

## File: models\mail_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.tools import html2plaintext, html_escape


class MailChannel(models.Model):
    _inherit = 'mail.channel'

    def _define_command_lead(self):
        return {'help': _('Create a new lead (/lead lead title)')}

    def _execute_command_lead(self, **kwargs):
        partner = self.env.user.partner_id
        key = kwargs['body']
        channel_partners = self.env['mail.channel.partner'].search([
            ('partner_id', '!=', partner.id),
            ('channel_id', '=', self.id)], limit=1
        )
        if key.strip() == '/lead':
            msg = self._define_command_lead()['help']
        else:
            lead = self._convert_visitor_to_lead(partner, channel_partners, key)
            msg = _('Created a new lead: <a href="#" data-oe-id="%s" data-oe-model="crm.lead">%s</a>') % (lead.id, html_escape(lead.name))
        self._send_transient_message(partner, msg)

    def _convert_visitor_to_lead(self, partner, channel_partners, key):
        description = ''.join(
            '%s: %s\n' % (message.author_id.name or self.anonymous_name, message.body)
            for message in self.channel_message_ids.sorted('id')
        )
        utm_source = self.env.ref('crm_livechat.utm_source_livechat', raise_if_not_found=False)
        lead = self.env['crm.lead'].create({
            'name': html2plaintext(key[5:]),
            'partner_id': channel_partners.partner_id.id,
            'user_id': None,
            'team_id': None,
            'description': html2plaintext(description),
            'referred': partner.name,
            'source_id': utm_source and utm_source.id,
        })
        lead._onchange_partner_id()
        return lead

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_channel

```

