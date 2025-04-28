# Odoo Module: crm_mail_plugin

Category: Sales/CRM

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'CRM Mail Plugin',
    'version': '1.0',
    'category': 'Sales/CRM',
    'sequence': 5,
    'summary': 'Turn emails received in your mailbox into leads and log their content as internal notes.',
    'description': "Turn emails received in your mailbox into leads and log their content as internal notes.",
    'website': 'https://www.odoo.com/app/crm',
    'depends': [
        'crm',
        'mail_plugin',
    ],
    'data': [
        'views/crm_mail_plugin_lead.xml',
        'views/crm_lead_views.xml'
    ],
    'web.assets_backend': [
        'crm_mail_plugin/static/src/to_translate/**/*',
    ],
    'installable': True,
    'application': False,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\crm_client.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.tools import html2plaintext

from .mail_plugin import MailPluginController


class CrmClient(MailPluginController):

    @http.route(route='/mail_client_extension/log_single_mail_content',
                type="json", auth="outlook", cors="*")
    def log_single_mail_content(self, lead, message, **kw):
        """
            deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary
            for supporting older versions
        """
        crm_lead = request.env['crm.lead'].browse(lead)
        crm_lead.message_post(body=message)

    @http.route('/mail_client_extension/lead/get_by_partner_id', type="json", auth="outlook", cors="*")
    def crm_lead_get_by_partner_id(self, partner, limit=5, offset=0, **kwargs):
        """
            deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary
            for supporting older versions
        """
        partner_instance = request.env['res.partner'].browse(partner)
        return {'leads': self._fetch_partner_leads(partner_instance, limit, offset)}

    @http.route('/mail_client_extension/lead/create_from_partner', type='http', auth='user', methods=['GET'])
    def crm_lead_redirect_create_form_view(self, partner_id):
        """
            deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary
            for supporting older versions
        """
        server_action = http.request.env.ref("crm_mail_plugin.lead_creation_prefilled_action")
        return request.redirect(
            '/web#action=%s&model=crm.lead&partner_id=%s' % (server_action.id, int(partner_id)))

    @http.route('/mail_plugin/lead/create', type='json', auth='outlook', cors="*")
    def crm_lead_create(self, partner_id, email_body, email_subject):
        partner = request.env['res.partner'].browse(partner_id).exists()
        if not partner:
            return {'error': 'partner_not_found'}

        record = request.env['crm.lead'].with_company(partner.company_id).create({
            'name': html2plaintext(email_subject),
            'partner_id': partner_id,
            'description': email_body,
        })

        return {'lead_id': record.id}

    @http.route('/mail_client_extension/lead/open', type='http', auth='user')
    def crm_lead_open(self, lead_id):
        """
            deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary
            for supporting older versions
        """
        action = http.request.env.ref("crm.crm_lead_view_form")
        url = '/web#id=%s&action=%s&model=crm.lead&edit=1&model=crm.lead' % (lead_id, action.id)
        return request.redirect(url)

```

## File: controllers\mail_plugin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo.http import request
from odoo.tools.misc import formatLang

from odoo.addons.mail_plugin.controllers import mail_plugin

_logger = logging.getLogger(__name__)


class MailPluginController(mail_plugin.MailPluginController):

    def _fetch_partner_leads(self, partner, limit=5, offset=0):
        """
        Returns an array containing partner leads, each lead will have the following structure :
        {
            id: the lead's id,
            name: the lead's name,
            expected_revenue: the expected revenue field value
            probability: the value of the probability field,
            recurring_revenue: the value of the recurring_revenue field if the lead has a recurring revenue
            recurring_plan: the value of the recurring plan field if the lead has a recurring revenue
        }
        """

        partner_leads = request.env['crm.lead'].search(
            [('partner_id', '=', partner.id)], offset=offset, limit=limit)
        recurring_revenues = request.env.user.has_group('crm.group_use_recurring_revenues')

        leads = []
        for lead in partner_leads:
            lead_values = {
                'lead_id': lead.id,
                'name': lead.name,
                'expected_revenue': formatLang(request.env, lead.expected_revenue, monetary=True,
                                               currency_obj=lead.company_currency),
                'probability': lead.probability,
            }

            if recurring_revenues:
                lead_values.update({
                    'recurring_revenue': formatLang(request.env, lead.recurring_revenue, monetary=True,
                                                    currency_obj=lead.company_currency),
                    'recurring_plan': lead.recurring_plan.name,
                })

            leads.append(lead_values)

        return leads

    def _get_contact_data(self, partner):
        """
        Return the leads key only if the current user can create leads. So, if he can not
        create leads, the section won't be visible on the addin side (like if the CRM
        module was not installed on the database).
        """
        contact_values = super(MailPluginController, self)._get_contact_data(partner)

        if not request.env['crm.lead'].check_access_rights('create', raise_exception=False):
            return contact_values

        if not partner:
            contact_values['leads'] = []
        else:
            contact_values['leads'] = self._fetch_partner_leads(partner)
        return contact_values

    def _mail_content_logging_models_whitelist(self):
        models_whitelist = super(MailPluginController, self)._mail_content_logging_models_whitelist()
        if not request.env['crm.lead'].check_access_rights('create', raise_exception=False):
            return models_whitelist
        return models_whitelist + ['crm.lead']

    def _translation_modules_whitelist(self):
        modules_whitelist = super(MailPluginController, self)._translation_modules_whitelist()
        if not request.env['crm.lead'].check_access_rights('create', raise_exception=False):
            return modules_whitelist
        return modules_whitelist + ['crm_mail_plugin']

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_client
from . import mail_plugin

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class Lead(models.Model):
    _inherit = 'crm.lead'

    @api.model
    def _form_view_auto_fill(self):
        """
            deprecated as of saas-14.3, not needed for newer versions of the mail plugin but necessary
            for supporting older versions
        """
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'crm.lead',
            'context': {
                'default_partner_id': self.env.context.get('params', {}).get('partner_id'),
            }
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead

```

## File: static\src\to_translate\translations_gmail.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
    Terms to translate for the Gmail plugin
-->
<ressources>
    <string>Log the email on the lead</string>
    <string>Email already logged on the lead</string>
    <string>Could not create the lead</string>
    <string>Opportunities (%s)</string>
    <string>%(expected_revenue)s at %(probability)s%</string>
    <string>%(expected_revenue)s + %(recurring_revenue)s %(recurring_plan)s at %(probability)s%</string>
    <string>Save Contact to create new Opportunities.</string>
    <string>You can only create opportunities for existing customers.</string>
</ressources>

```

## File: static\src\to_translate\translations_outlook.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--Terms to translate for the outlook plugin -->
<ressources>
    <string>Log Email Into Lead</string>
    <string>Opportunities</string>
    <string>Opportunities (%(count)s)</string>
    <string>%(expected_revenue)s at %(probability)s%</string>
    <string>%(expected_revenue)s + %(recurring_revenue)s %(recurring_plan)s at %(probability)s%</string>
    <string>Save Contact to create new Opportunities.</string>
    <string>No opportunities found for this contact.</string>
    <string>The Contact needs to exist to create Opportunity.</string>
</ressources>

```

## File: views\crm_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
    <odoo>
    <!-- Called by the plugin to open a lead in edit mode -->
        <record id="crm_lead_action_form_edit" model="ir.actions.act_window">
              <field name="name">Lead: redirect form in edit mode</field>
              <field name="type">ir.actions.act_window</field>
              <field name="res_model">crm.lead</field>
              <field name="view_mode">form</field>
              <field name="view_id" ref="crm.crm_lead_view_form"/>
              <field name="context">
                  {
                        'form_view_initial_mode': 'edit',
                  }
              </field>
        </record>
    </odoo>

```

## File: views\crm_mail_plugin_lead.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--deprecated as of saas-14.3-->
    <record id="lead_creation_prefilled_action" model="ir.actions.server">
      <field name="name">Redirection to the lead creation form with prefilled info</field>
      <field name="type">ir.actions.server</field>
      <field name="model_id" ref="model_crm_lead"/>
      <field name="state">code</field>
      <field name="code">action = model._form_view_auto_fill()</field>
    </record>
</odoo>

```

