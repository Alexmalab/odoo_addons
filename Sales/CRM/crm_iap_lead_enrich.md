# Odoo Module: crm_iap_lead_enrich

Category: Sales/CRM

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

from odoo.api import Environment, SUPERUSER_ID


def _synchronize_cron(cr, registry):
    env = Environment(cr, SUPERUSER_ID, {'active_test': False})
    cron = env.ref('crm_iap_lead_enrich.ir_cron_lead_enrichment')
    if cron:
        config = env['ir.config_parameter'].get_param('crm.iap.lead.enrich.setting', 'manual')
        cron.active = config != 'manual'

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Lead Enrichment',
    'summary': 'Enrich Leads/Opportunities using email address domain',
    'version': '1.1',
    'category': 'Sales/CRM',
    'version': '1.1',
    'depends': [
        'iap_crm',
        'iap_mail',
    ],
    'data': [
        'data/ir_cron.xml',
        'data/ir_action.xml',
        'data/mail_data.xml',
        'views/crm_lead_views.xml',
        'views/res_config_settings_view.xml',
    ],
    'post_init_hook': '_synchronize_cron',
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\ir_action.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="action_enrich_mail" model="ir.actions.server" >
            <field name="name">Enrich</field>
            <field name="model_id" ref="model_crm_lead"/>
            <field name="binding_model_id" ref="crm.model_crm_lead"/>
            <field name="state">code</field>
            <field name="code">   
    if records:
        records.iap_enrich()
            </field>
        </record>

    </data>
</odoo>

```

## File: data\ir_cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_lead_enrichment" model="ir.cron">
        <field name="name">CRM: enrich leads (IAP)</field>
        <field name="model_id" ref="crm.model_crm_lead"/>
        <field name="user_id" ref="base.user_root"/>
        <field name="state">code</field>
        <field name="code">model._iap_enrich_leads_cron()</field>
        <field name="interval_number">1</field>
        <field name="interval_type">hours</field>
        <field name="numbercall">-1</field>
        <field name="doall" eval="False"/>
    </record>

</odoo>

```

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- VIEWS USED FOR MESSAGING -->
    <template id="mail_message_lead_enrich_notfound">
        <p>Lead Enrichment based on email address</p>
        <div style="background-color:#ffffff;padding:15px;">
            <span> No company data found based on the email address or email address is one of an email provider. No credit was consumed. </span>
        </div>
    </template>

    <template id="mail_message_lead_enrich_no_email">
        <p>Lead Enrichment based on email address</p>
        <div style="background-color:#ffffff;padding:15px;">
            <span>Enrichment could not be done as no email address was provided.</span>
        </div>
    </template>

    <template id="mail_message_lead_enrich_no_credit">
        <p>Lead enriched based on email address</p>
        <div style="background-color:#ffffff;padding:15px;">
            <span>Your balance for Lead Enrichment is insufficient. Please go to your <a t-attf-href="{{url}}" target="_blank">IAP account</a> to buy credits.</span>
        </div>
    </template>
</data></odoo>

```

## File: models\crm_lead.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime
import logging
from psycopg2 import OperationalError


from odoo import _, api, fields, models, tools
from odoo.addons.iap.tools import iap_tools

_logger = logging.getLogger(__name__)

EMAIL_PROVIDERS = ['gmail.com', 'hotmail.com', 'yahoo.com', 'qq.com',
                   'outlook.com', '163.com', 'yahoo.fr', 'live.com',
                   'hotmail.fr', 'icloud.com', '126.com', 'me.com',
                   'free.fr', 'ymail.com', 'msn.com', 'mail.com']

class Lead(models.Model):
    _inherit = 'crm.lead'

    iap_enrich_done = fields.Boolean(string='Enrichment done', help='Whether IAP service for lead enrichment based on email has been performed on this lead.')
    show_enrich_button = fields.Boolean(string='Allow manual enrich', compute="_compute_show_enrich_button")

    @api.depends('email_from', 'probability', 'iap_enrich_done', 'reveal_id')
    def _compute_show_enrich_button(self):
        config = self.env['ir.config_parameter'].sudo().get_param('crm.iap.lead.enrich.setting', 'manual')
        if not config or config != 'manual':
            self.show_enrich_button = False
            return
        for lead in self:
            if not lead.active or not lead.email_from or lead.iap_enrich_done or lead.reveal_id or lead.probability == 100:
                lead.show_enrich_button = False
            else:
                lead.show_enrich_button = True

    @api.model
    def _iap_enrich_leads_cron(self):
        timeDelta = fields.datetime.now() - datetime.timedelta(hours=1)
        # Get all leads not lost nor won (lost: active = False)
        leads = self.search([
            ('iap_enrich_done', '=', False),
            ('reveal_id', '=', False),
            ('probability', '<', 100),
            ('create_date', '>', timeDelta)
        ])
        leads.iap_enrich(from_cron=True)

    def iap_enrich(self, from_cron=False):
        # Split self in a list of sub-recordsets or 50 records to prevent timeouts
        batches = [self[index:index + 50] for index in range(0, len(self), 50)]
        for leads in batches:
            lead_emails = {}
            with self._cr.savepoint():
                try:
                    self._cr.execute(
                        "SELECT 1 FROM {} WHERE id in %(lead_ids)s FOR UPDATE NOWAIT".format(self._table),
                        {'lead_ids': tuple(leads.ids)}, log_exceptions=False)
                    for lead in leads:
                        # If lead is lost, active == False, but is anyway removed from the search in the cron.
                        if lead.probability == 100 or lead.iap_enrich_done:
                            continue

                        normalized_email = tools.email_normalize(lead.email_from)
                        if not normalized_email:
                            lead.message_post_with_view(
                                'crm_iap_lead_enrich.mail_message_lead_enrich_no_email',
                                subtype_id=self.env.ref('mail.mt_note').id)
                            continue

                        email_domain = normalized_email.split('@')[1]
                        # Discard domains of generic email providers as it won't return relevant information
                        if email_domain in EMAIL_PROVIDERS:
                            lead.write({'iap_enrich_done': True})
                            lead.message_post_with_view(
                                'crm_iap_lead_enrich.mail_message_lead_enrich_notfound',
                                subtype_id=self.env.ref('mail.mt_note').id)
                        else:
                            lead_emails[lead.id] = email_domain

                    if lead_emails:
                        try:
                            iap_response = self.env['iap.enrich.api']._request_enrich(lead_emails)
                        except iap_tools.InsufficientCreditError:
                            _logger.info('Sent batch %s enrich requests: failed because of credit', len(lead_emails))
                            if not from_cron:
                                data = {
                                    'url': self.env['iap.account'].get_credits_url('reveal'),
                                }
                                leads[0].message_post_with_view(
                                    'crm_iap_lead_enrich.mail_message_lead_enrich_no_credit',
                                    values=data,
                                    subtype_id=self.env.ref('mail.mt_note').id)
                            # Since there are no credits left, there is no point to process the other batches
                            break
                        except Exception as e:
                            _logger.info('Sent batch %s enrich requests: failed with exception %s', len(lead_emails), e)
                        else:
                            _logger.info('Sent batch %s enrich requests: success', len(lead_emails))
                            self._iap_enrich_from_response(iap_response)
                except OperationalError:
                    _logger.error('A batch of leads could not be enriched :%s', repr(leads))
                    continue
            # Commit processed batch to avoid complete rollbacks and therefore losing credits.
            if not self.env.registry.in_test_mode():
                self.env.cr.commit()

    @api.model
    def _iap_enrich_from_response(self, iap_response):
        """ Handle from the service and enrich the lead accordingly

        :param iap_response: dict{lead_id: company data or False}
        """
        for lead in self.search([('id', 'in', list(iap_response.keys()))]):  # handle unlinked data by performing a search
            iap_data = iap_response.get(str(lead.id))
            if not iap_data:
                lead.write({'iap_enrich_done': True})
                lead.message_post_with_view('crm_iap_lead_enrich.mail_message_lead_enrich_notfound', subtype_id=self.env.ref('mail.mt_note').id)
                continue

            values = {'iap_enrich_done': True}
            lead_fields = ['partner_name', 'reveal_id', 'street', 'city', 'zip']
            iap_fields = ['name', 'clearbit_id', 'location', 'city', 'postal_code']
            for lead_field, iap_field in zip(lead_fields, iap_fields):
                if not lead[lead_field] and iap_data.get(iap_field):
                    values[lead_field] = iap_data[iap_field]

            if not lead.phone and iap_data.get('phone_numbers'):
                values['phone'] = iap_data['phone_numbers'][0]
            if not lead.mobile and iap_data.get('phone_numbers') and len(iap_data['phone_numbers']) > 1:
                values['mobile'] = iap_data['phone_numbers'][1]
            if not lead.country_id and iap_data.get('country_code'):
                country = self.env['res.country'].search([('code', '=', iap_data['country_code'].upper())])
                values['country_id'] = country.id
            else:
                country = lead.country_id
            if not lead.state_id and country and iap_data.get('state_code'):
                state = self.env['res.country.state'].search([
                    ('code', '=', iap_data['state_code']),
                    ('country_id', '=', country.id)
                ])
                values['state_id'] = state.id

            lead.write(values)

            template_values = iap_data
            template_values['flavor_text'] = _("Lead enriched based on email address")
            lead.message_post_with_view(
                'iap_mail.enrich_company',
                values=template_values,
                subtype_id=self.env.ref('mail.mt_note').id
            )

```

## File: models\iap_enrich_api.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api
from odoo.addons.iap.tools import iap_tools


class IapEnrichAPI(models.AbstractModel):
    _name = 'iap.enrich.api'
    _description = 'IAP Lead Enrichment API'
    _DEFAULT_ENDPOINT = 'https://iap-services.odoo.com'

    @api.model
    def _contact_iap(self, local_endpoint, params):
        account = self.env['iap.account'].get('reveal')
        dbuuid = self.env['ir.config_parameter'].sudo().get_param('database.uuid')
        params['account_token'] = account.account_token
        params['dbuuid'] = dbuuid
        base_url = self.env['ir.config_parameter'].sudo().get_param('enrich.endpoint', self._DEFAULT_ENDPOINT)
        return iap_tools.iap_jsonrpc(base_url + local_endpoint, params=params, timeout=300)

    @api.model
    def _request_enrich(self, lead_emails):
        """ Contact endpoint to get enrichment data.

        :param lead_emails: dict{lead_id: email}
        :return: dict{lead_id: company data or False}
        :raise: several errors, notably
          * InsufficientCreditError: {
            "credit": 4.0,
            "service_name": "reveal",
            "base_url": "https://iap.odoo.com/iap/1/credit",
            "message": "You don't have enough credits on your account to use this service."
            }
        """
        params = {
            'domains': lead_emails,
        }
        return self._contact_iap('/iap/clearbit/1/lead_enrichment_email', params=params)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ResConfigSettings(models.TransientModel):
    _inherit = "res.config.settings"

    @api.onchange('lead_enrich_auto')
    def _onchange_cron_lead_enrich(self):
        """ change the active status of the cron according to the settings"""
        if self.module_crm_iap_lead_enrich == True:
            cron = self.sudo().with_context(active_test=False).env.ref('crm_iap_lead_enrich.ir_cron_lead_enrichment')
            if cron:
                cron.active = self.lead_enrich_auto != 'manual'

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
from . import iap_enrich_api
from . import res_config_settings

```

## File: views\crm_lead_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="crm_lead_view_form" model="ir.ui.view">
        <field name="name">crm.lead.view.form.inherit.iap.lead.enrich</field>
        <field name="model">crm.lead</field>
        <field name="inherit_id" ref="crm.crm_lead_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='%(crm.action_crm_lead2opportunity_partner)d']" position="after">
                <field name="show_enrich_button" invisible="1"/>
                <button string="Enrich" name="iap_enrich" type="object" class="btn btn-secondary"
                title="Enrich this lead with company data based on the email address"
                attrs="{'invisible':['|',('show_enrich_button', '!=', True),('type','=','opportunity')]}"/>
                <button string="Enrich" name="iap_enrich" type="object" class="btn btn-secondary"
                title="Enrich this opportunity with company data based on the email address"
                attrs="{'invisible':['|',('show_enrich_button', '!=', True),('type','=','lead')]}"/>
            </xpath>
        </field>
    </record>
</odoo>


```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<data>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">crm_iap_lead_enrich rechargement</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="crm.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <field name="lead_enrich_auto" position="after">
                <widget name="iap_buy_more_credits" service_name="reveal"/>
            </field>
        </field>
    </record>

</data>

```

