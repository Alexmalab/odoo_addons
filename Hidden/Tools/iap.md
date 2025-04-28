# Odoo Module: iap

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import tools

# compatibility imports
from odoo.addons.iap.tools.iap_tools import iap_jsonrpc as jsonrpc
from odoo.addons.iap.tools.iap_tools import iap_authorize as authorize
from odoo.addons.iap.tools.iap_tools import iap_cancel as cancel
from odoo.addons.iap.tools.iap_tools import iap_capture as capture
from odoo.addons.iap.tools.iap_tools import iap_charge as charge
from odoo.addons.iap.tools.iap_tools import InsufficientCreditError

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'In-App Purchases',
    'category': 'Hidden/Tools',
    'version': '1.1',
    'summary': 'Basic models and helpers to support In-App purchases.',
    'description': """
This module provides standard tools (account model, context manager and helpers)
to support In-App purchases inside Odoo. """,
    'depends': [
        'web',
        'base_setup'
    ],
    'data': [
        'security/ir.model.access.csv',
        'security/ir_rule.xml',
        'views/iap_views.xml',
        'views/res_config_settings.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'iap/static/src/**/*.js',
            'iap/static/src/**/*.xml',
        ],
        'web.tests_assets': [
            'iap/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
UPDATE iap_account SET account_token = REGEXP_REPLACE(account_token, '(\+.*)?$', '+disabled');

```

## File: models\iap_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import uuid
import werkzeug.urls

from odoo import api, fields, models
from odoo.addons.iap.tools import iap_tools

_logger = logging.getLogger(__name__)

DEFAULT_ENDPOINT = 'https://iap.odoo.com'


class IapAccount(models.Model):
    _name = 'iap.account'
    _rec_name = 'service_name'
    _description = 'IAP Account'

    service_name = fields.Char()
    account_token = fields.Char(default=lambda s: uuid.uuid4().hex)
    company_ids = fields.Many2many('res.company')

    @api.model_create_multi
    def create(self, vals_list):
        accounts = super().create(vals_list)
        if self.env['ir.config_parameter'].sudo().get_param('database.is_neutralized'):
            # Disable new accounts on a neutralized database
            for account in accounts:
                account.account_token = f"{account.account_token.split('+')[0]}+disabled"
        return accounts

    @api.model
    def get(self, service_name, force_create=True):
        domain = [
            ('service_name', '=', service_name),
            '|',
                ('company_ids', 'in', self.env.companies.ids),
                ('company_ids', '=', False)
        ]
        accounts = self.search(domain, order='id desc')
        accounts_without_token = accounts.filtered(lambda acc: not acc.account_token)
        if accounts_without_token:
            with self.pool.cursor() as cr:
                # In case of a further error that will rollback the database, we should
                # use a different SQL cursor to avoid undo the accounts deletion.

                # Flush the pending operations to avoid a deadlock.
                self.env.flush_all()
                IapAccount = self.with_env(self.env(cr=cr))
                # Need to use sudo because regular users do not have delete right
                IapAccount.search(domain + [('account_token', '=', False)]).sudo().unlink()
                accounts = accounts - accounts_without_token
        if not accounts:
            with self.pool.cursor() as cr:
                # Since the account did not exist yet, we will encounter a NoCreditError,
                # which is going to rollback the database and undo the account creation,
                # preventing the process to continue any further.

                # Flush the pending operations to avoid a deadlock.
                self.env.flush_all()
                IapAccount = self.with_env(self.env(cr=cr))
                account = IapAccount.search(domain, order='id desc', limit=1)
                if not account:
                    if not force_create:
                        return account
                    account = IapAccount.create({'service_name': service_name})
                # fetch 'account_token' into cache with this cursor,
                # as self's cursor cannot see this account
                account_token = account.account_token
            account = self.browse(account.id)
            self.env.cache.set(account, IapAccount._fields['account_token'], account_token)
            return account
        accounts_with_company = accounts.filtered(lambda acc: acc.company_ids)
        if accounts_with_company:
            return accounts_with_company[0]
        return accounts[0]

    @api.model
    def get_credits_url(self, service_name, base_url='', credit=0, trial=False):
        """ Called notably by ajax crash manager, buy more widget, partner_autocomplete, sanilmail. """
        dbuuid = self.env['ir.config_parameter'].sudo().get_param('database.uuid')
        if not base_url:
            endpoint = iap_tools.iap_get_endpoint(self.env)
            route = '/iap/1/credit'
            base_url = endpoint + route
        account_token = self.get(service_name).account_token
        d = {
            'dbuuid': dbuuid,
            'service_name': service_name,
            'account_token': account_token,
            'credit': credit,
        }
        if trial:
            d.update({'trial': trial})
        return '%s?%s' % (base_url, werkzeug.urls.url_encode(d))

    @api.model
    def get_account_url(self):
        """ Called only by res settings """
        route = '/iap/services'
        endpoint = iap_tools.iap_get_endpoint(self.env)
        all_accounts = self.search([
            '|',
            ('company_ids', '=', self.env.company.id),
            ('company_ids', '=', False),
        ])

        global_account_per_service = {
            account.service_name: account.account_token
            for account in all_accounts.filtered(lambda acc: not acc.company_ids)
        }
        company_account_per_service = {
            account.service_name: account.account_token
            for account in all_accounts.filtered(lambda acc: acc.company_ids)
        }

        # Prioritize company specific accounts over global accounts
        account_per_service = {**global_account_per_service, **company_account_per_service}

        parameters = {'tokens': list(account_per_service.values())}

        return '%s?%s' % (endpoint + route, werkzeug.urls.url_encode(parameters))

    @api.model
    def get_config_account_url(self):
        """ Called notably by ajax partner_autocomplete. """
        account = self.env['iap.account'].get('partner_autocomplete')
        action = self.env.ref('iap.iap_account_action')
        menu = self.env.ref('iap.iap_account_menu')
        no_one = self.user_has_groups('base.group_no_one')
        if account:
            url = "/web#id=%s&action=%s&model=iap.account&view_type=form&menu_id=%s" % (account.id, action.id, menu.id)
        else:
            url = "/web#action=%s&model=iap.account&view_type=form&menu_id=%s" % (action.id, menu.id)
        return no_one and url

    @api.model
    def get_credits(self, service_name):
        account = self.get(service_name, force_create=False)
        credit = 0

        if account:
            route = '/iap/1/balance'
            endpoint = iap_tools.iap_get_endpoint(self.env)
            url = endpoint + route
            params = {
                'dbuuid': self.env['ir.config_parameter'].sudo().get_param('database.uuid'),
                'account_token': account.account_token,
                'service_name': service_name,
            }
            try:
                credit = iap_tools.iap_jsonrpc(url=url, params=params)
            except Exception as e:
                _logger.info('Get credit error : %s', str(e))
                credit = -1

        return credit

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
    _inherit = 'res.config.settings'

    @api.model
    def _redirect_to_iap_account(self):
        return {
            'type': 'ir.actions.act_url',
            'url': self.env['iap.account'].get_account_url(),
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import iap_account
from . import res_config_settings
from . import iap_enrich_api

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_client_iap_account_manager,iap.account.manager,model_iap_account,base.group_system,1,1,1,1
access_client_iap_account_user,iap.account.user,model_iap_account,base.group_user,1,0,1,0

```

## File: security\ir_rule.xml

```xml
<odoo noupdate="1">

    <record id="user_iap_account" model="ir.rule">
      <field name="name">User IAP Account</field>
      <field name="model_id" ref="model_iap_account"/>
      <field name="groups" eval="[(4, ref('base.group_user'))]"/>
      <!-- partners can CUD services linked to themselves -->
      <field name="domain_force">['|', ('company_ids', '=', False), ('company_ids', 'in', company_ids)]</field>
    </record>

</odoo>
```

## File: static\description\icon.svg

```svg
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 70 70">
  <defs>
    <mask id="mask" x="0" y="0" width="70" height="70" maskUnits="userSpaceOnUse">
      <g id="b">
        <path id="a" d="M4,0H65c4,0,5,1,5,5V65c0,4-1,5-5,5H4c-3,0-4-1-4-5V5C0,1,1,0,4,0Z" fill="#fff" fill-rule="evenodd"/>
      </g>
    </mask>
    <linearGradient id="linear-gradient" x1="-1615.93" y1="477.94" x2="-1616.93" y2="476.94" gradientTransform="matrix(70, 0, 0, -70, 113184.99, 33455.73)" gradientUnits="userSpaceOnUse">
      <stop offset="0" stop-color="#b06161"/>
      <stop offset="0.46" stop-color="#984e4e"/>
      <stop offset="1" stop-color="#7c3838"/>
    </linearGradient>
  </defs>
  <g mask="url(#mask)">
    <g>
      <path d="M0,0H70V70H0Z" fill-rule="evenodd" fill="url(#linear-gradient)"/>
      <path d="M4,1H65c2.67,0,4.33.67,5,2V0H0V3C.67,1.67,2,1,4,1Z" fill="#fff" fill-opacity="0.38" fill-rule="evenodd"/>
      <path d="M4,69a3.66,3.66,0,0,1-4-4V35.5L17.47,18l7-3.37,6.65,3.1v8.18l2.16.64,3.49-3.49L48,23l3.58,2L50.35,48,29.21,69.08Z" fill="#393939" fill-rule="evenodd" opacity="0.32" style="isolation: isolate"/>
      <path d="M4,69H65c2.67,0,4.33-1,5-3v4H0V66A3.92,3.92,0,0,0,4,69Z" fill-opacity="0.38" fill-rule="evenodd"/>
      <g>
        <g opacity="0.4">
          <path d="M47.06,24.7H32.7v.88l3.94,1.82h0l.09.05h9.94a.4.4,0,0,1,.34.44V31H39.31v6.5h7.74v9.71a.34.34,0,0,1-.34.34H20.3a.33.33,0,0,1-.33-.34V44.06l-2.65,1.22-.07,0v2.24A2.73,2.73,0,0,0,20,50.3H47.06a2.73,2.73,0,0,0,2.71-2.75V27.45A2.73,2.73,0,0,0,47.06,24.7Z"/>
          <path d="M36.29,30.76a.5.5,0,0,0,0-.13h0a.63.63,0,0,0-.1-.12l0,0L36,30.4h0L29.7,27.5V20a.44.44,0,0,0,0-.16v0a.61.61,0,0,0-.05-.12v0a.52.52,0,0,0-.1-.11l0,0a.41.41,0,0,0-.11-.07h0L22.7,16.33a.65.65,0,0,0-.51,0l-6.67,3.08h0a.41.41,0,0,0-.11.07l0,0a.52.52,0,0,0-.1.11v0a.61.61,0,0,0,0,.12v0a.41.41,0,0,0,0,.16V27.5l-6.3,2.89h0l-.11.08,0,0a.4.4,0,0,0-.1.12v0l-.06.12v0a.77.77,0,0,0,0,.15v8a.63.63,0,0,0,.34.56l6.66,3.08a.67.67,0,0,0,.52,0l6.36-2.94,6.36,2.94a.67.67,0,0,0,.52,0L36,39.48a.62.62,0,0,0,.36-.56V31a1.1,1.1,0,0,0,0-.16Zm-20.5,2.59L10.59,31l5.12-2.36L20.91,31Zm6,5.2L16.41,41V34.42l5.39-2.48Zm.62-16.19L17.22,20l5.2-2.4,5.2,2.4ZM23,23.43l5.44-2.51v6.62L23,30.06Zm6,9.92L23.92,31l5.2-2.4,5.12,2.36L29,33.35Zm6.05,5.18L29.66,41V34.42l5.43-2.52Z"/>
        </g>
        <g>
          <path d="M49.06,22.7H34.7v.88l3.94,1.82h0l.09.05h9.94a.4.4,0,0,1,.34.44V29H41.31v6.5h7.74v9.71a.34.34,0,0,1-.34.34H22.3a.33.33,0,0,1-.33-.34V42.06l-2.65,1.22-.07,0v2.24A2.73,2.73,0,0,0,22,48.3H49.06a2.73,2.73,0,0,0,2.71-2.75V25.45A2.73,2.73,0,0,0,49.06,22.7Z" fill="#fff"/>
          <path d="M38.29,28.76a.5.5,0,0,0,0-.13h0a.63.63,0,0,0-.1-.12l0,0L38,28.4h0L31.7,25.5V18a.44.44,0,0,0,0-.16v0a.61.61,0,0,0-.05-.12v0a.52.52,0,0,0-.1-.11l0,0a.41.41,0,0,0-.11-.07h0L24.7,14.33a.65.65,0,0,0-.51,0l-6.67,3.08h0a.41.41,0,0,0-.11.07l0,0a.52.52,0,0,0-.1.11v0a.61.61,0,0,0-.05.12v0a.41.41,0,0,0,0,.16V25.5l-6.3,2.89h0l-.11.08,0,0a.4.4,0,0,0-.1.12v0l-.06.12v0a.77.77,0,0,0,0,.15v8a.63.63,0,0,0,.34.56l6.66,3.08a.59.59,0,0,0,.52,0l6.36-2.94,6.36,2.94a.67.67,0,0,0,.52,0L38,37.48a.62.62,0,0,0,.36-.56V29a1.1,1.1,0,0,0,0-.16Zm-20.5,2.59L12.59,29l5.12-2.36L22.91,29Zm6,5.2L18.41,39V32.42l5.39-2.48Zm.62-16.19L19.22,18l5.2-2.4,5.2,2.4ZM25,21.43l5.44-2.51v6.62L25,28.06Zm6,9.92L25.92,29l5.2-2.4,5.12,2.36L31,31.35Zm6.05,5.18L31.66,39V32.42l5.43-2.52Z" fill="#fff"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\action_buttons_widget\action_buttons_widget.js

```javascript
/** @odoo-modules */

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";

const { Component } = owl;

class IAPActionButtonsWidget extends Component {
    setup() {
        this.orm = useService("orm");
        this.action = useService("action");
    }

    async onViewServicesClicked() {
        const url = await this.orm.silent.call("iap.account", "get_account_url");
        this.action.doAction({
            type: "ir.actions.act_url",
            url: url,
        });
    }

    async onBuyLinkClicked() {
        const url = await this.orm.silent.call("iap.account", "get_credits_url", [this.props.serviceName]);
        this.action.doAction({
            type: "ir.actions.act_url",
            url: url,
        });
    }
}
IAPActionButtonsWidget.template = "iap.ActionButtonsWidget";
IAPActionButtonsWidget.extractProps = ({ attrs }) => {
    return {
        serviceName: attrs.service_name,
        showServiceButtons: !Boolean(attrs.hide_service),
    };
};

registry.category("view_widgets").add("iap_buy_more_credits", IAPActionButtonsWidget);

```

## File: static\src\action_buttons_widget\action_buttons_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template id="template" xml:space="preserve">

    <t t-name="iap.ActionButtonsWidget" owl="1">
        <div class="mt-2 row">
            <div class="col-sm">
                <button t-on-click="onBuyLinkClicked" class="btn btn-link buy_credits px-0 o-hidden-ios"><i class="fa fa-arrow-right"/> Buy credits</button><br/>
                <button t-if="props.showServiceButtons" t-on-click="onViewServicesClicked" class="btn btn-link px-0 o-hidden-ios"><i class="fa fa-arrow-right me-1"/> View My Services</button>
            </div>
        </div>
    </t>

</template>

```

## File: static\src\img\iap_logo.svg

```svg
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<svg
   xmlns:dc="http://purl.org/dc/elements/1.1/"
   xmlns:cc="http://creativecommons.org/ns#"
   xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
   xmlns:svg="http://www.w3.org/2000/svg"
   xmlns="http://www.w3.org/2000/svg"
   xmlns:sodipodi="http://sodipodi.sourceforge.net/DTD/sodipodi-0.dtd"
   xmlns:inkscape="http://www.inkscape.org/namespaces/inkscape"
   width="680"
   height="143"
   version="1.1"
   id="svg889"
   sodipodi:docname="iap_logo.svg"
   inkscape:version="0.92.3 (2405546, 2018-03-11)">
  <metadata
     id="metadata895">
    <rdf:RDF>
      <cc:Work
         rdf:about="">
        <dc:format>image/svg+xml</dc:format>
        <dc:type
           rdf:resource="http://purl.org/dc/dcmitype/StillImage" />
        <dc:title></dc:title>
      </cc:Work>
    </rdf:RDF>
  </metadata>
  <defs
     id="defs893" />
  <sodipodi:namedview
     pagecolor="#ffffff"
     bordercolor="#666666"
     borderopacity="1"
     objecttolerance="10"
     gridtolerance="10"
     guidetolerance="10"
     inkscape:pageopacity="0"
     inkscape:pageshadow="2"
     inkscape:window-width="1920"
     inkscape:window-height="1015"
     id="namedview891"
     showgrid="false"
     inkscape:zoom="1.9852941"
     inkscape:cx="390.21297"
     inkscape:cy="71.751852"
     inkscape:window-x="0"
     inkscape:window-y="0"
     inkscape:window-maximized="1"
     inkscape:current-layer="svg889" />
  <g
     fill="none"
     fill-rule="evenodd"
     id="g887">
    <g
       id="g881"
       fill-rule="nonzero">
      <path
         d="M397.168 143.475c-30.083 0-54.47-24.387-54.47-54.47 0-30.084 24.387-54.472 54.47-54.472 30.084 0 54.471 24.388 54.471 54.471 0 30.084-24.387 54.471-54.47 54.471zm0-22.365c17.732 0 32.106-14.375 32.106-32.106S414.9 56.9 397.168 56.9c-17.73 0-32.105 14.374-32.105 32.105 0 17.731 14.374 32.106 32.105 32.106zm-115.435 22.365c-30.083 0-54.47-24.387-54.47-54.47 0-30.084 24.387-54.472 54.47-54.472 30.084 0 54.471 24.388 54.471 54.471 0 30.084-24.387 54.471-54.47 54.471zm0-22.365c17.732 0 32.106-14.375 32.106-32.106S299.465 56.9 281.733 56.9c-17.73 0-32.105 14.374-32.105 32.105 0 17.731 14.374 32.106 32.105 32.106z"
         id="path875"
         fill="#888" />
      <path
         d="M222.306 88.575l.001.43c0 30.083-24.387 54.47-54.47 54.47-30.084 0-54.471-24.387-54.471-54.47 0-30.084 24.387-54.472 54.47-54.472a54.23 54.23 0 0 1 32.84 11.009V10.825C200.676 4.847 205.52 0 211.498 0c5.977 0 10.822 4.851 10.822 10.825v77.191c0 .188-.005.374-.014.559zm-54.47 32.535c17.732 0 32.106-14.375 32.106-32.106S185.568 56.9 167.837 56.9c-17.732 0-32.106 14.374-32.106 32.105 0 17.731 14.374 32.106 32.106 32.106z"
         id="path877"
         fill="#888" />
      <path
         d="M54.97 143.354C24.887 143.354.5 118.967.5 88.884.5 58.8 24.887 34.411 54.97 34.411c30.084 0 54.471 24.388 54.471 54.471 0 30.084-24.387 54.471-54.47 54.471zm-.5-22.147c17.732 0 32.106-14.374 32.106-32.106 0-17.73-14.374-32.105-32.105-32.105-17.731 0-32.105 14.374-32.105 32.105 0 17.732 14.374 32.106 32.105 32.106z"
         id="path879"
         fill="#9C5789" />
    </g>
    <g
       aria-label="IAP"
       style="font-size:134px;font-family:Helvetica;letter-spacing:-4.4380002;fill:#8f8f8f"
       id="text885">
      <path
         d="M 503.996,45.314 H 491.4 V 143 h 12.596 z"
         style=""
         id="path903" />
      <path
         d="m 572.41003,113.654 10.05,29.346 h 13.936 l -34.304,-97.686 h -16.08 l -34.84,97.686 h 13.266 l 10.318,-29.346 z m -3.484,-10.452 h -31.088 l 16.08,-44.488 z"
         style=""
         id="path905" />
      <path
         d="m 618.61984,101.594 h 30.686 c 7.638,0 13.668,-2.278 18.894,-6.968 5.896,-5.36 8.442,-11.658 8.442,-20.636 0,-18.358 -10.854,-28.676 -30.15,-28.676 h -40.334 V 143 h 12.462 z m 0,-10.988 V 56.302 h 25.996 c 11.926,0 19.028,6.432 19.028,17.152 0,10.72 -7.102,17.152 -19.028,17.152 z"
         style=""
         id="path907" />
    </g>
  </g>
</svg>

```

## File: static\src\js\iap_buy_more_credits.js

```javascript
odoo.define('iap.buy_more_credits', function (require) {
'use strict';

var widgetRegistry = require('web.widget_registry');
var Widget = require('web.Widget');

var core = require('web.core');
var rpc = require('web.rpc');
const utils = require('web.utils');

var QWeb = core.qweb;

var IAPBuyMoreCreditsWidget = Widget.extend({
    className: 'o_field_iap_buy_more_credits',

    /**
     * @constructor
     * Prepares the basic rendering of edit mode by setting the root to be a
     * div.dropdown.open.
     * @see FieldChar.init
     */
    init: function (parent, data, options) {
        this._super.apply(this, arguments);
        this.service_name = options.attrs.service_name;
        this.hideService = utils.toBoolElse(options.attrs.hide_service || '', false);
    },

    /**
     * @override
     */
    start: function () {
        this.$widget = $(QWeb.render('iap.buy_more_credits', {'hideService': this.hideService}));
        this.$buyLink = this.$widget.find('.buy_credits');
        this.$widget.appendTo(this.$el);
        this.$buyLink.click(this._getLink.bind(this));
        if (!this.hideService) {
            this.el.querySelector('.o_iap_view_my_services').addEventListener('click', this._getMyServices.bind(this));
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    _getLink: function () {
        var self = this;
        return rpc.query({
            model: 'iap.account',
            method: 'get_credits_url',
            args: [this.service_name],
        }, {
            shadow: true,
        }).then(function (url) {
            return self.do_action({
                type: 'ir.actions.act_url',
                url: url,
            });
        });
    },

    /**
     * @private
     */
    _getMyServices() {
        return this._rpc({
            model: 'iap.account',
            method: 'get_account_url',
        }).then(url => {
            this.do_action({type: 'ir.actions.act_url', url: url});
        });
    },
});

widgetRegistry.add('iap_buy_more_credits', IAPBuyMoreCreditsWidget);

return IAPBuyMoreCreditsWidget;
});

```

## File: static\src\js\insufficient_credit_error_handler.js

```javascript
/** @odoo-module */
import { Dialog } from "@web/core/dialog/dialog";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";

const { Component, onWillStart } = owl;

class InsufficientCreditDialog extends Component {
    setup() {
        this.orm = useService("orm");
        onWillStart(this.onWillStart);
    }

    async onWillStart() {
        const { errorData } = this.props;
        this.url = await this.orm.call("iap.account", "get_credits_url", [], {
            base_url: errorData.base_url,
            service_name: errorData.service_name,
            credit: errorData.credit,
            trial: errorData.trial,
        });
        this.style = errorData.body ? "padding:0;" : "";
        const { _t } = this.env;
        const { isEnterprise } = odoo.info;
        if (errorData.trial && isEnterprise) {
            this.buttonMessage = _t("Start a Trial at Odoo");
        } else {
            this.buttonMessage = _t("Buy credits");
        }
    }

    buyCredits() {
        window.open(this.url, "_blank");
        this.props.close();
    }
}
InsufficientCreditDialog.components = { Dialog };
InsufficientCreditDialog.template = "iap.InsufficientCreditDialog";

function insufficientCreditHandler(env, error, originalError) {
    if (!originalError) {
        return false;
    }
    const { data } = originalError;
    if (data && data.name === "odoo.addons.iap.tools.iap_tools.InsufficientCreditError") {
        env.services.dialog.add(InsufficientCreditDialog, {
            errorData: JSON.parse(data.message),
        });
        return true;
    }
    return false;
}

registry
    .category("error_handlers")
    .add("insufficientCreditHandler", insufficientCreditHandler, { sequence: 0 });

```

## File: static\src\xml\iap_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template id="template" xml:space="preserve">

    <!-- LAYOUT TEMPLATES -->
    <t t-name="iap.InsufficientCreditDialog" owl="1">
        <Dialog>
            <div t-att-style="style">
                <t t-if="props.errorData.body">
                    <div t-raw="props.errorData.body"/>
                </t>
                <t t-if="!props.errorData.body">
                    <t t-if="props.errorData.message">
                        <span t-esc="props.errorData.message"/>
                    </t>
                    <t t-if="!props.errorData.message">
                        <span>Insufficient credit to perform this service.</span>
                    </t>
                </t>
                <t t-set-slot="footer">
                    <button class="btn btn-primary" t-on-click="buyCredits" t-esc="buttonMessage"/>
                    <button class="btn" t-on-click="close">Cancel</button>
                </t>
            </div>
        </Dialog>
    </t>

    <t t-extend="DashboardMain">
        <t t-jquery=".o_web_settings_apps" t-operation="after">
            <div class="o_web_settings_iap"></div>
        </t>
    </t>
    
    <div t-name="iap.buy_more_credits" class="mt-2 row">
        <div class="col-sm">
            <button class="btn btn-link buy_credits px-0 o-hidden-ios"><i class="fa fa-arrow-right"/> Buy credits</button><br/>
            <button t-if="!hideService" class="btn btn-link o_iap_view_my_services px-0 o-hidden-ios"><i class="fa fa-arrow-right me-1"/>View My Services</button>
        </div>
    </div>
</template>

```

## File: tools\iap_tools.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import contextlib
import logging
import json
import requests
import uuid
from unittest.mock import patch

from odoo import exceptions, _
from odoo.tests.common import BaseCase
from odoo.tools import email_normalize, exception_to_unicode, pycompat

_logger = logging.getLogger(__name__)

DEFAULT_ENDPOINT = 'https://iap.odoo.com'


# We need to mock iap_jsonrpc during tests as we don't want to perform real calls to RPC endpoints
def iap_jsonrpc_mocked(*args, **kwargs):
    raise exceptions.AccessError("Unavailable during tests.")


iap_patch = patch('odoo.addons.iap.tools.iap_tools.iap_jsonrpc', iap_jsonrpc_mocked)


def setUp(self):
    old_setup_func(self)
    iap_patch.start()
    self.addCleanup(iap_patch.stop)


old_setup_func = BaseCase.setUp
BaseCase.setUp = setUp

#----------------------------------------------------------
# Tools globals
#----------------------------------------------------------

_MAIL_PROVIDERS = {
    'gmail.com', 'hotmail.com', 'yahoo.com', 'qq.com', 'outlook.com', '163.com', 'yahoo.fr', 'live.com', 'hotmail.fr', 'icloud.com', '126.com',
    'me.com', 'free.fr', 'ymail.com', 'msn.com', 'mail.com', 'orange.fr', 'aol.com', 'wanadoo.fr', 'live.fr', 'mail.ru', 'yahoo.co.in',
    'rediffmail.com', 'hku.hk', 'googlemail.com', 'gmx.de', 'sina.com', 'skynet.be', 'laposte.net', 'yahoo.co.uk', 'yahoo.co.id', 'web.de',
    'gmail.com ', 'outlook.fr', 'telenet.be', 'yahoo.es', 'naver.com', 'hotmail.co.uk', 'gmai.com', 'foxmail.com', 'hku.hku', 'bluewin.ch',
    'sfr.fr', 'libero.it', 'mac.com', 'rocketmail.com', 'protonmail.com', 'gmx.com', 'gamil.com', 'hotmail.es', 'gmx.net', 'comcast.net',
    'yahoo.com.mx', 'linkedin.com', 'yahoo.com.br', 'yahoo.in', 'yahoo.ca', 't-online.de', '139.com', 'yandex.ru', 'yahoo.com.hk','yahoo.de',
    'yeah.net', 'yandex.com', 'nwytg.net', 'neuf.fr', 'yahoo.com.ar', 'outlook.es', 'abv.bg', 'aliyun.com', 'yahoo.com.tw', 'ukr.net', 'live.nl',
    'wp.pl', 'hotmail.it', 'live.com.mx', 'zoho.com', 'live.co.uk', 'sohu.com', 'twoomail.com', 'yahoo.com.sg', 'yahoo.com.vn',
    'windowslive.com', 'gmail', 'vols.utk.edu', 'email.com', 'tiscali.it', 'yahoo.it', 'gmx.ch', 'trbvm.com', 'nwytg.com', 'mvrht.com', 'nyit.edu',
    'o2.pl', 'live.cn', 'gmial.com', 'seznam.cz', 'live.be', 'videotron.ca', 'gmil.com', 'live.ca', 'hotmail.de', 'sbcglobal.net', 'connect.hku.hk',
    'yahoo.com.au', 'att.net', 'live.in', 'btinternet.com', 'gmx.fr', 'voila.fr', 'shaw.ca', 'prodigy.net.mx', 'vip.qq.com', 'yahoo.com.ph',
    'bigpond.com', '7thcomputing.com', 'freenet.de', 'alice.it', 'esi.dz',
    'bk.ru', 'mail.odoo.com', 'gmail.con', 'fiu.edu', 'gmal.com', 'useemlikefun.com', 'google.com', 'trbvn.com', 'yopmail.com', 'ya.ru',
    'hotmail.co.th', 'arcor.de', 'hotmail.ca', '21cn.com', 'live.de', 'outlook.de', 'gmailcom', 'unal.edu.co', 'tom.com', 'yahoo.gr',
    'gmx.at', 'inbox.lv', 'ziggo.nl', 'xs4all.nl', 'sapo.pt', 'live.com.au', 'nate.com', 'online.de', 'sina.cn', 'gmail.co', 'rogers.com',
    'mailinator.com', 'cox.net', 'hotmail.be', 'verizon.net', 'yahoo.co.jp', 'usa.com', 'consultant.com', 'hotmai.com', '189.cn',
    'sky.com', 'eezee-it.com', 'opayq.com', 'maildrop.cc', 'home.nl', 'virgilio.it', 'outlook.be', 'hanmail.net', 'uol.com.br', 'hec.ca',
    'terra.com.br', 'inbox.ru', 'tin.it', 'list.ru', 'hotmail.com ', 'safecoms.com', 'smile.fr', 'sprintit.fi', 'uniminuto.edu.co',
    'bol.com.br', 'bellsouth.net', 'nirmauni.ac.in', 'ldc.edu.in', 'ig.com.br', 'engineer.com', 'scarlet.be', 'inbox.com', 'gmaill.com',
    'freemail.hu', 'live.it', 'blackwaretech.com', 'byom.de', 'dispostable.com', 'dayrep.com', 'aim.com', 'prixgen.com', 'gmail.om',
    'asterisk-tech.mn', 'in.com', 'aliceadsl.fr', 'lycos.com', 'topnet.tn', 'teleworm.us', 'kedgebs.com', 'supinfo.com', 'posteo.de',
    'yahoo.com ', 'op.pl', 'gmail.fr', 'grr.la', 'oci.fr', 'aselcis.com', 'optusnet.com.au', 'mailcatch.com', 'rambler.ru', 'protonmail.ch',
    'prisme.ch', 'bbox.fr', 'orbitalu.com', 'netcourrier.com', 'iinet.net.au', 'cegetel.net', 'proton.me', 'dbmail.com', 'club-internet.fr', 'outlook.jp',
    # Dummy entries
    'example.com',
}
_MAIL_DOMAIN_BLACKLIST = _MAIL_PROVIDERS | {'odoo.com'}

# List of country codes for which we should offer state filtering when mining new leads.
# See crm.iap.lead.mining.request#_compute_available_state_ids() or task-2471703 for more details.
_STATES_FILTER_COUNTRIES_WHITELIST = set([
    'AR', 'AU', 'BR', 'CA', 'IN', 'MY', 'MX', 'NZ', 'AE', 'US'
])


#----------------------------------------------------------
# Tools
#----------------------------------------------------------

def mail_prepare_for_domain_search(email, min_email_length=0):
    """ Return an email address to use for a domain-based search. For generic
    email providers like gmail (see ``_MAIL_DOMAIN_BLACKLIST``) we consider
    each email as being independant (and return the whole email). Otherwise
    we return only the right-part of the email (aka "mydomain.com" if email is
    "Raoul Lachignole" <raoul@mydomain.com>).

    :param integer min_email_length: skip if email has not the sufficient minimal
      length, indicating a probably fake / wrong value (skip if 0);
    """
    if not email:
        return False
    email_tocheck = email_normalize(email, strict=False)
    if not email_tocheck:
        email_tocheck = email.casefold()

    if email_tocheck and min_email_length and len(email_tocheck) < min_email_length:
        return False

    parts = email_tocheck.rsplit('@', maxsplit=1)
    if len(parts) == 1:
        return email_tocheck
    email_domain = parts[1]
    if email_domain not in _MAIL_DOMAIN_BLACKLIST:
        return '@' + email_domain
    return email_tocheck


#----------------------------------------------------------
# Helpers for both clients and proxy
#----------------------------------------------------------

def iap_get_endpoint(env):
    url = env['ir.config_parameter'].sudo().get_param('iap.endpoint', DEFAULT_ENDPOINT)
    return url

#----------------------------------------------------------
# Helpers for clients
#----------------------------------------------------------

class InsufficientCreditError(Exception):
    pass


class IAPServerError(Exception):
    pass


def iap_jsonrpc(url, method='call', params=None, timeout=15):
    """
    Calls the provided JSON-RPC endpoint, unwraps the result and
    returns JSON-RPC errors as exceptions.
    """
    payload = {
        'jsonrpc': '2.0',
        'method': method,
        'params': params,
        'id': uuid.uuid4().hex,
    }

    _logger.info('iap jsonrpc %s', url)
    try:
        req = requests.post(url, json=payload, timeout=timeout)
        req.raise_for_status()
        response = req.json()
        if 'error' in response:
            name = response['error']['data'].get('name').rpartition('.')[-1]
            if name == 'InsufficientCreditError':
                credit_error = InsufficientCreditError(response['error']['data'].get('message'))
                credit_error.data = response['error']['data']
                raise credit_error
            else:
                raise IAPServerError("An error occurred on the IAP server")
        return response.get('result')
    except requests.exceptions.Timeout:
        _logger.warning("iap jsonrpc %s timed out", url)
        raise exceptions.AccessError(
            _('The request to the service timed out. Please contact the author of the app. The URL it tried to contact was %s', url)
        )
    except (requests.exceptions.RequestException, IAPServerError) as e:
        _logger.warning("iap jsonrpc %s failed, %s: %s", url, e.__class__.__name__, exception_to_unicode(e))
        raise exceptions.AccessError(
            _('The url that this service requested returned an error. Please contact the author of the app. The url it tried to contact was %s', url)
        )

#----------------------------------------------------------
# Helpers for proxy
#----------------------------------------------------------

class IapTransaction(object):

    def __init__(self):
        self.credit = None


def iap_authorize(env, key, account_token, credit, dbuuid=False, description=None, credit_template=None, ttl=4320):
    endpoint = iap_get_endpoint(env)
    params = {
        'account_token': account_token,
        'credit': credit,
        'key': key,
        'description': description,
        'ttl': ttl,
    }
    if dbuuid:
        params.update({'dbuuid': dbuuid})
    try:
        transaction_token = iap_jsonrpc(endpoint + '/iap/1/authorize', params=params)
    except InsufficientCreditError as e:
        if credit_template:
            arguments = json.loads(e.args[0])
            arguments['body'] = pycompat.to_text(env['ir.qweb']._render(credit_template))
            e.args = (json.dumps(arguments),)
        raise e
    return transaction_token


def iap_cancel(env, transaction_token, key):
    endpoint = iap_get_endpoint(env)
    params = {
        'token': transaction_token,
        'key': key,
    }
    r = iap_jsonrpc(endpoint + '/iap/1/cancel', params=params)
    return r


def iap_capture(env, transaction_token, key, credit):
    endpoint = iap_get_endpoint(env)
    params = {
        'token': transaction_token,
        'key': key,
        'credit_to_capture': credit,
    }
    r = iap_jsonrpc(endpoint + '/iap/1/capture', params=params)
    return r


@contextlib.contextmanager
def iap_charge(env, key, account_token, credit, dbuuid=False, description=None, credit_template=None, ttl=4320):
    """
    Account charge context manager: takes a hold for ``credit``
    amount before executing the body, then captures it if there
    is no error, or cancels it if the body generates an exception.

    :param str key: service identifier
    :param str account_token: user identifier
    :param int credit: cost of the body's operation
    :param description: a description of the purpose of the charge,
                        the user will be able to see it in their
                        dashboard
    :type description: str
    :param credit_template: a QWeb template to render and show to the
                            user if their account does not have enough
                            credits for the requested operation
    :param int ttl: transaction time to live in hours.
                    If the credit are not captured when the transaction
                    expires, the transaction is canceled
    :type credit_template: str
    """
    transaction_token = iap_authorize(env, key, account_token, credit, dbuuid, description, credit_template, ttl)
    try:
        transaction = IapTransaction()
        transaction.credit = credit
        yield transaction
    except Exception as e:
        r = iap_cancel(env,transaction_token, key)
        raise e
    else:
        r = iap_capture(env,transaction_token, key, transaction.credit)

```

## File: tools\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import iap_tools

```

## File: views\iap_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- iap Client Account Views -->
    <record id="iap_account_view_form" model="ir.ui.view">
        <field name="name">iap.account.form</field>
        <field name="model">iap.account</field>
        <field name="arch" type="xml">
            <form string="IAP Account">
                <sheet>
                    <group name="account" string="Account Information">
                        <field name="service_name"/>
                        <field name="company_ids" widget="many2many_tags" domain="[('id', 'in', allowed_company_ids)]"/>
                        <field name="account_token"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    <record id="iap_account_view_tree" model="ir.ui.view">
        <field name="name">iap.account.tree</field>
        <field name="model">iap.account</field>
        <field name="arch" type="xml">
            <tree string="IAP Accounts">
                <field name="service_name"/>
                <field name="company_ids" widget="many2many_tags"/>
                <field name="account_token" readonly="1"/>
            </tree>
        </field>
    </record>
    <!-- Actions -->
    <record id="iap_account_action" model="ir.actions.act_window">
        <field name="name">IAP Account</field>
        <field name="res_model">iap.account</field>
        <field name='view_mode'>tree,form</field>
    </record>

    <!-- Menus -->
    <menuitem
        id="iap_root_menu"
        name="IAP"
        parent="base.menu_custom"
        sequence="5"/>

    <menuitem
        id="iap_account_menu"
        name="IAP Accounts"
        parent="iap_root_menu"
        action="iap_account_action"
        sequence="10"/>

</odoo>

```

## File: views\res_config_settings.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="open_iap_account" model="ir.actions.server">
        <field name="name">Open IAP Account</field>
        <field name="model_id" ref="base_setup.model_res_config_settings"/>
        <field name="binding_model_id" ref="base_setup.model_res_config_settings"/>
        <field name="state">code</field>
        <field name="code">
if records:
    action = records._redirect_to_iap_account()
        </field>
    </record>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.base.setup.iap</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='contacts_settings']" position="inside">
                <div id="iap_portal">
                    <div class='row mt16 o_settings_container iap_portal' name="iap_purchases_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="iap_credits_setting">
                            <div class='o_setting_right_pane'>
                                <div class="o_form_label">
                                Odoo IAP
                                <a href="https://www.odoo.com/documentation/16.0/applications/general/in_app_purchase.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <a href="https://www.odoo.com/documentation/16.0/developer/misc/api/iap.html" title="Documentation" class="ms-1 o_doc_link" target="_blank"></a>
                                </div>
                                <div class="text-muted">
                                    View your IAP Services and recharge your credits
                                </div>
                                <div class='mt8'>
                                    <button name="%(iap.open_iap_account)d" icon="fa-arrow-right" type="action" string="View My Services" class="btn-link"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

