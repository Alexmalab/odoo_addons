# Odoo Module: crm_iap_enrich

Category: Sales/CRM

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models


def _synchronize_cron(env):
    cron = env.ref('crm_iap_enrich.ir_cron_lead_enrichment')
    if cron:
        config = env['ir.config_parameter'].get_param('crm.iap.lead.enrich.setting', 'auto')
        cron.active = config == 'auto'

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Lead Enrichment',
    'summary': 'Enrich Leads/Opportunities using email address domain',
    'category': 'Sales/CRM',
    'version': '1.1',
    'depends': [
        'iap_crm',
        'iap_mail',
    ],
    'data': [
        'data/ir_cron.xml',
        'data/ir_action.xml',
        'data/mail_templates.xml',
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

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- VIEWS USED FOR MESSAGING -->
    <template id="mail_message_lead_enrich_notfound">
        <p>Lead Enrichment (based on email address)</p>
        <div style="padding:15px;padding-left:0px;">
            <span> No company data found based on the email address or email address is one of an email provider. No credit was consumed. </span>
        </div>
    </template>

    <template id="mail_message_lead_enrich_no_email">
        <p>Lead Enrichment (based on email address)</p>
        <div style="padding:15px;padding-left:0px;">
            <span>Enrichment could not be done because the email address does not look valid.</span>
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


class Lead(models.Model):
    _inherit = 'crm.lead'

    iap_enrich_done = fields.Boolean(string='Enrichment done', help='Whether IAP service for lead enrichment based on email has been performed on this lead.')
    show_enrich_button = fields.Boolean(string='Allow manual enrich', compute="_compute_show_enrich_button")

    @api.depends('email_from', 'probability', 'iap_enrich_done', 'reveal_id')
    def _compute_show_enrich_button(self):
        for lead in self:
            if not lead.active or not lead.email_from or lead.email_state == 'incorrect' or lead.iap_enrich_done or lead.reveal_id or lead.probability == 100:
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
            '|', ('probability', '<', 100), ('probability', '=', False),
            ('create_date', '>', timeDelta)
        ])
        leads.iap_enrich(from_cron=True)

    @api.model_create_multi
    def create(self, vals_list):
        leads = super(Lead, self).create(vals_list)
        enrich_mode = self.env['ir.config_parameter'].sudo().get_param('crm.iap.lead.enrich.setting', 'auto')
        if enrich_mode == 'auto':
            cron = self.env.ref('crm_iap_enrich.ir_cron_lead_enrichment', raise_if_not_found=False)
            if cron:
                cron._trigger()
        return leads

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
                        # Skip if no email (different from wrong email leading to no email_normalized)
                        if not lead.email_from:
                            continue

                        normalized_email = tools.email_normalize(lead.email_from)
                        if not normalized_email:
                            lead.message_post_with_source(
                                'crm_iap_enrich.mail_message_lead_enrich_no_email',
                                subtype_xmlid='mail.mt_note',
                            )
                            continue

                        email_domain = normalized_email.split('@')[1]
                        # Discard domains of generic email providers as it won't return relevant information
                        if email_domain in iap_tools._MAIL_PROVIDERS:
                            lead.write({'iap_enrich_done': True})
                            lead.message_post_with_source(
                                'crm_iap_enrich.mail_message_lead_enrich_notfound',
                                subtype_xmlid='mail.mt_note',
                            )
                        else:
                            lead_emails[lead.id] = email_domain

                    if lead_emails:
                        try:
                            iap_response = self.env['iap.enrich.api']._request_enrich(lead_emails)
                        except iap_tools.InsufficientCreditError:
                            _logger.info('Lead enrichment failed because of insufficient credit')
                            if not from_cron:
                                self.env['iap.account']._send_no_credit_notification(
                                    service_name='reveal',
                                    title=_("Not enough credits for Lead Enrichment"))
                            # Since there are no credits left, there is no point to process the other batches
                            break
                        except Exception as e:
                            if not from_cron:
                                self.env['iap.account']._send_error_notification(
                                    message=_('An error occurred during lead enrichment'))
                            _logger.info('An error occurred during lead enrichment: %s', e)
                        else:
                            if not from_cron:
                                self.env['iap.account']._send_success_notification(
                                    message=_("The leads/opportunities have successfully been enriched"))
                            _logger.info('Batch of %s leads successfully enriched', len(lead_emails))
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
                lead.message_post_with_source(
                    'crm_iap_enrich.mail_message_lead_enrich_notfound',
                    subtype_xmlid='mail.mt_note',
                )
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
            lead.message_post_with_source(
                'iap_mail.enrich_company',
                render_values=template_values,
                subtype_xmlid='mail.mt_note',
            )

    def _merge_get_fields_specific(self):
        return {
            ** super(Lead, self)._merge_get_fields_specific(),
            'iap_enrich_done': lambda fname, leads: any(lead.iap_enrich_done for lead in leads),
        }

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class ResConfigSettings(models.TransientModel):
    _inherit = "res.config.settings"

    @api.model
    def get_values(self):
        values = super(ResConfigSettings, self).get_values()
        cron = self.sudo().with_context(active_test=False).env.ref('crm_iap_enrich.ir_cron_lead_enrichment', raise_if_not_found=False)
        values['lead_enrich_auto'] = 'auto' if cron and cron.active else 'manual'
        return values

    def set_values(self):
        super().set_values()
        cron = self.sudo().with_context(active_test=False).env.ref('crm_iap_enrich.ir_cron_lead_enrichment', raise_if_not_found=False)
        if cron and cron.active != (self.lead_enrich_auto == 'auto'):
            cron.active = self.lead_enrich_auto == 'auto'

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import crm_lead
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
                <button string="Enrich" name="iap_enrich" type="object" class="btn btn-secondary" data-hotkey="g"
                title="Enrich lead with company data"
                invisible="not show_enrich_button or type == 'opportunity'"/>
                <button string="Enrich" name="iap_enrich" type="object" class="btn btn-secondary" data-hotkey="g"
                title="Enrich opportunity with company data"
                invisible="not show_enrich_button or type == 'lead'"/>
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
        <field name="name">res.config.settings.view.form.inherit.crm.iap.enrich</field>
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

