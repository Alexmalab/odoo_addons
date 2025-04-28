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
        'data/crm_livechat_chatbot_data.xml',
        'views/chatbot_script_views.xml',
        'views/chatbot_script_step_views.xml',
    ],
    'depends': [
        'crm',
        'im_livechat'
    ],
    'description': 'Create new lead with using /lead command in the channel',
    'auto_install': True,
    'license': 'LGPL-3',
    'assets': {
        'web.assets_backend': {
            'crm_livechat/static/src/core/*',
        },
        'web.qunit_suite_tests': [
            'crm_livechat/static/tests/**/*.js',
        ],
    },
}

```

## File: data\crm_livechat_chatbot_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data noupdate="1">

    <!--
        This provides a working lead generation chatbot for people to work with.
        It's placed into 'data' to give them a starting point.
        From that record, they can duplicate / adapt / delete / ...
    -->

    <record id="chatbot_script_lead_generation_bot" model="chatbot.script">
        <field name="title">Lead Generation Bot</field>
        <field name="image_1920" type="base64" file="mail/static/src/img/odoobot.png"/>
    </record>

    <record id="chatbot_script_lead_generation_step_welcome" model="chatbot.script.step">
        <field name="message">Hi there, what brings you to our website today? 👋</field>
        <field name="sequence">1</field>
        <field name="step_type">free_input_multi</field>
        <field name="chatbot_script_id" ref="chatbot_script_lead_generation_bot"/>
    </record>

    <record id="chatbot_script_lead_generation_step_forward_operator" model="chatbot.script.step">
        <field name="sequence">2</field>
        <field name="step_type">forward_operator</field>
        <field name="chatbot_script_id" ref="chatbot_script_lead_generation_bot"/>
    </record>

    <record id="chatbot_script_lead_generation_step_noone_available" model="chatbot.script.step">
        <field name="message">Hu-ho, it looks like none of our operators are available 🙁</field>
        <field name="sequence">3</field>
        <field name="step_type">text</field>
        <field name="chatbot_script_id" ref="chatbot_script_lead_generation_bot"/>
    </record>

    <record id="chatbot_script_welcome_step_pricing_email" model="chatbot.script.step">
        <field name="message">Would you mind leaving your email address so that we can reach you back?</field>
        <field name="sequence">4</field>
        <field name="step_type">question_email</field>
        <field name="chatbot_script_id" ref="chatbot_script_lead_generation_bot"/>
    </record>

    <record id="chatbot_script_welcome_step_just_looking" model="chatbot.script.step">
        <field name="message">Thank you, you should hear back from us very soon!</field>
        <field name="sequence">5</field>
        <field name="step_type">create_lead</field>
        <field name="chatbot_script_id" ref="chatbot_script_lead_generation_bot"/>
    </record>

</data></odoo>

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

## File: models\chatbot_script.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ChatbotScript(models.Model):
    _inherit = 'chatbot.script'

    lead_count = fields.Integer(
        string='Generated Lead Count', compute='_compute_lead_count')

    def _compute_lead_count(self):
        leads_data = self.env['crm.lead'].with_context(active_test=False).sudo()._read_group(
            [('source_id', 'in', self.mapped('source_id').ids)], ['source_id'], ['__count'])
        mapped_leads = {source.id: count for source, count in leads_data}
        for script in self:
            script.lead_count = mapped_leads.get(script.source_id.id, 0)

    def action_view_leads(self):
        self.ensure_one()
        action = self.env['ir.actions.act_window']._for_xml_id('crm.crm_lead_all_leads')
        action['domain'] = [('source_id', '=', self.source_id.id)]
        action['context'] = {'create': False}
        return action

```

## File: models\chatbot_script_step.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models, fields


class ChatbotScriptStep(models.Model):
    _inherit = 'chatbot.script.step'

    step_type = fields.Selection(
        selection_add=[('create_lead', 'Create Lead')], ondelete={'create_lead': 'cascade'})
    crm_team_id = fields.Many2one(
        'crm.team', string='Sales Team', ondelete='set null',
        help="Used in combination with 'create_lead' step type in order to automatically "
             "assign the created lead/opportunity to the defined team")

    def _chatbot_crm_prepare_lead_values(self, discuss_channel, description):
        return {
            'description': description + discuss_channel._get_channel_history(),
            'name': _("%s's New Lead", self.chatbot_script_id.title),
            'source_id': self.chatbot_script_id.source_id.id,
            'team_id': self.crm_team_id.id,
            'type': 'lead' if self.crm_team_id.use_leads else 'opportunity',
            'user_id': False,
        }

    def _process_step(self, discuss_channel):
        self.ensure_one()

        posted_message = super()._process_step(discuss_channel)

        if self.step_type == 'create_lead':
            self._process_step_create_lead(discuss_channel)

        return posted_message

    def _process_step_create_lead(self, discuss_channel):
        """ When reaching a 'create_lead' step, we extract the relevant information: visitor's
        email, phone and conversation history to create a crm.lead.

        We use the email and phone to update the environment partner's information (if not a public
        user) if they differ from the current values.

        The whole conversation history will be saved into the lead's description for reference.
        This also allows having a question of type 'free_input_multi' to let the visitor explain
        their interest / needs before creating the lead. """

        customer_values = self._chatbot_prepare_customer_values(
            discuss_channel, create_partner=False, update_partner=True)
        if self.env.user._is_public():
            create_values = {
                'email_from': customer_values['email'],
                'phone': customer_values['phone'],
            }
        else:
            partner = self.env.user.partner_id
            create_values = {
                'partner_id': partner.id,
                'company_id': partner.company_id.id,
            }

        create_values.update(self._chatbot_crm_prepare_lead_values(
            discuss_channel, customer_values['description']))

        self.env['crm.lead'].create(create_values)

```

## File: models\discuss_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _
from odoo.tools import html2plaintext


class DiscussChannel(models.Model):
    _inherit = 'discuss.channel'

    def execute_command_lead(self, **kwargs):
        partner = self.env.user.partner_id
        key = kwargs['body']
        if key.strip() == '/lead':
            msg = _('Create a new lead (/lead lead title)')
        else:
            lead = self._convert_visitor_to_lead(partner, key)
            msg = _('Created a new lead: %s', lead._get_html_link())
        self._send_transient_message(partner, msg)

    def _convert_visitor_to_lead(self, partner, key):
        """ Create a lead from channel /lead command
        :param partner: internal user partner (operator) that created the lead;
        :param key: operator input in chat ('/lead Lead about Product')
        """
        # if public user is part of the chat: consider lead to be linked to an
        # anonymous user whatever the participants. Otherwise keep only share
        # partners (no user or portal user) to link to the lead.
        customers = self.env['res.partner']
        for customer in self.with_context(active_test=False).channel_partner_ids.filtered(lambda p: p != partner and p.partner_share):
            if customer.is_public:
                customers = self.env['res.partner']
                break
            else:
                customers |= customer

        utm_source = self.env.ref('crm_livechat.utm_source_livechat', raise_if_not_found=False)
        return self.env['crm.lead'].create({
            'name': html2plaintext(key[5:]),
            'partner_id': customers[0].id if customers else False,
            'user_id': False,
            'team_id': False,
            'description': self._get_channel_history(),
            'referred': partner.name,
            'source_id': utm_source and utm_source.id,
        })

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import chatbot_script
from . import chatbot_script_step
from . import discuss_channel

```

## File: static\src\core\channel_commands.js

```javascript
/** @odoo-module */

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";

registry.category("discuss.channel_commands").add("lead", {
    help: _t("Create a new lead (/lead lead title)"),
    methodName: "execute_command_lead",
});

```

## File: views\chatbot_script_step_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="chatbot_script_step_view_form" model="ir.ui.view">
        <field name="name">chatbot.script.step.view.form.inherit.crm.livechat</field>
        <field name="model">chatbot.script.step</field>
        <field name="inherit_id" ref="im_livechat.chatbot_script_step_view_form"/>
        <field name="arch" type="xml">
            <field name="step_type" position="after">
                <field name="crm_team_id" invisible="step_type != 'create_lead'"
                    options="{'no_open': True}"/>
            </field>
        </field>
    </record>

</data></odoo>

```

## File: views\chatbot_script_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="chatbot_script_view_form" model="ir.ui.view">
        <field name="name">chatbot.script.view.form.inherit.crm.livechat</field>
        <field name="model">chatbot.script</field>
        <field name="inherit_id" ref="im_livechat.chatbot_script_view_form"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button name="action_view_leads" type="object" class="oe_stat_button"
                        icon="fa-star" invisible="lead_count == 0">
                    <field string="Leads" name="lead_count" widget="statinfo" />
                </button>
            </div>
        </field>
    </record>

</data></odoo>

```

