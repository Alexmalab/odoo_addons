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
        'data/services.xml',
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
    },
    'license': 'LGPL-3',
}

```

## File: data\neutralize.sql

```sql
-- happy path
UPDATE iap_account
SET account_token = REGEXP_REPLACE(account_token, '(\+.*)?$', '+disabled')
WHERE LENGTH(account_token) <= 33;
-- Legacy (invalid) records (pre v17)
UPDATE iap_account
SET account_token = 'dummy_value+disabled'
WHERE LENGTH(account_token) > 33
    AND account_token NOT LIKE '%+disabled';
```

## File: data\services.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- The reveal service is defined in the iap module, as a lot of different modules depend on it. -->
        <record id="iap_service_reveal" model="iap.service">
            <field name="name">Lead Generation</field>
            <field name="technical_name">reveal</field>
            <field name="description">Get quality leads and opportunities: convert your website visitors into leads, generate leads based on a set of criteria and enrich the company data of your opportunities.</field>
            <field name="unit_name">Credits</field>
            <field name="integer_balance">True</field>
        </record>
    </data>
</odoo>

```

## File: models\iap_account.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import secrets
import threading
import uuid
import werkzeug.urls

from odoo import api, fields, models, _
from odoo.addons.iap.tools import iap_tools
from odoo.exceptions import AccessError, UserError
from odoo.tools import get_lang

_logger = logging.getLogger(__name__)

DEFAULT_ENDPOINT = 'https://iap.odoo.com'


class IapAccount(models.Model):
    _name = 'iap.account'
    _description = 'IAP Account'

    name = fields.Char()
    service_id = fields.Many2one('iap.service', required=True)
    service_name = fields.Char(related='service_id.technical_name')
    service_locked = fields.Boolean(default=False)  # If True, the service can't be edited anymore
    description = fields.Char(related='service_id.description')
    account_token = fields.Char(
        default=lambda s: uuid.uuid4().hex,
        help="Account token is your authentication key for this service. Do not share it.",
        size=43)
    company_ids = fields.Many2many('res.company')

    # Dynamic fields, which are received from iap server and set when loading the view
    balance = fields.Char(readonly=True)
    warning_threshold = fields.Float("Email Alert Threshold")
    warning_user_ids = fields.Many2many('res.users', string="Email Alert Recipients")
    state = fields.Selection([('banned', 'Banned'), ('registered', "Registered"), ('unregistered', "Unregistered")], readonly=True)

    @api.constrains('warning_threshold', 'warning_user_ids')
    def validate_warning_alerts(self):
        for account in self:
            if account.warning_threshold < 0:
                raise UserError(_("Please set a positive email alert threshold."))
            users_with_no_email = [user.name for user in self.warning_user_ids if not user.email]
            if users_with_no_email:
                raise UserError(_(
                    "One of the email alert recipients doesn't have an email address set. Users: %s",
                    ",".join(users_with_no_email),
                ))

    def web_read(self, *args, **kwargs):
        if not self.env.context.get('disable_iap_fetch'):
            self._get_account_information_from_iap()
        return super().web_read(*args, **kwargs)

    def web_save(self, *args, **kwargs):
        return super(IapAccount, self.with_context(disable_iap_fetch=True)).web_save(*args, **kwargs)

    def write(self, values):
        res = super().write(values)
        if (
            not self.env.context.get('disable_iap_update')
            and any(warning_attribute in values for warning_attribute in ('warning_threshold', 'warning_user_ids'))
        ):
            route = '/iap/1/update-warning-email-alerts'
            endpoint = iap_tools.iap_get_endpoint(self.env)
            url = werkzeug.urls.url_join(endpoint, route)
            for account in self:
                data = {
                    'account_token': account.account_token,
                    'warning_threshold': account.warning_threshold,
                    'warning_emails': [{
                        'email': user.email,
                        'lang_code': user.lang or get_lang(self.env).code,
                    } for user in account.warning_user_ids],
                }
                try:
                    iap_tools.iap_jsonrpc(url=url, params=data)
                except AccessError as e:
                    _logger.warning("Update of the warning email configuration has failed: %s", str(e))
        return res

    @staticmethod
    def is_running_test_suite():
        return hasattr(threading.current_thread(), 'testing') and threading.current_thread().testing

    def _get_account_information_from_iap(self):
        # During testing, we don't want to call the iap server
        if self.is_running_test_suite():
            return
        route = '/iap/1/get-accounts-information'
        endpoint = iap_tools.iap_get_endpoint(self.env)
        url = werkzeug.urls.url_join(endpoint, route)
        params = {
            'iap_accounts': [{
                'token': account.account_token,
                'service': account.service_id.technical_name,
            } for account in self if account.service_id],
            'dbuuid': self.env['ir.config_parameter'].sudo().get_param('database.uuid'),
        }
        try:
            accounts_information = iap_tools.iap_jsonrpc(url=url, params=params)
        except AccessError as e:
            _logger.warning("Fetch of the IAP accounts information has failed: %s", str(e))
            return

        for token, information in accounts_information.items():
            information.pop('link_to_service_page', None)
            accounts = self.filtered(lambda acc: secrets.compare_digest(acc.account_token, token))

            for account in accounts:
                # Default rounding of 4 decimal places to avoid large decimals
                balance_amount = round(information['balance'], None if account.service_id.integer_balance else 4)
                balance = f"{balance_amount} {account.service_id.unit_name or ''}"

                account_info = self._get_account_info(account, balance, information)
                account.with_context(disable_iap_update=True, tracking_disable=True).write(account_info)

    def _get_account_info(self, account_id, balance, information):
        return {
            'balance': balance,
            'warning_threshold': information['warning_threshold'],
            'state': information['registered'],
            'service_locked': True,  # The account exist on IAP, prevent the edition of the service
        }

    @api.model_create_multi
    def create(self, vals_list):
        accounts = super().create(vals_list)
        for account in accounts:
            if not account.name:
                account.name = account.service_id.name

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
            service = self.env['iap.service'].search([('technical_name', '=', service_name)], limit=1)
            if not service:
                raise UserError("No service exists with the provided technical name")
            if self.is_running_test_suite():
                # During testing, we don't want to commit the creation of a new IAP account to the database
                return self.sudo().create({'service_id': service.id})

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
                    account = IapAccount.create({'service_id': service.id})
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
    def get_account_id(self, service_name):
        return self.get(service_name).id

    @api.model
    def get_credits_url(self, service_name, base_url='', credit=0, trial=False, account_token=False):
        """ Called notably by ajax crash manager, buy more widget, partner_autocomplete, sanilmail. """
        dbuuid = self.env['ir.config_parameter'].sudo().get_param('database.uuid')
        if not base_url:
            endpoint = iap_tools.iap_get_endpoint(self.env)
            route = '/iap/1/credit'
            base_url = werkzeug.urls.url_join(endpoint, route)
        if not account_token:
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

    def action_buy_credits(self):
        return {
            'type': 'ir.actions.act_url',
            'url': self.env['iap.account'].get_credits_url(
                account_token=self.account_token,
                service_name=self.service_name,
            ),
        }

    @api.model
    def get_config_account_url(self):
        """ Called notably by ajax partner_autocomplete. """
        account = self.env['iap.account'].get('partner_autocomplete')
        menu = self.env.ref('iap.iap_account_menu')
        if not self.env.user.has_group('base.group_no_one'):
            return False
        if account:
            url = f"/odoo/action-iap.iap_account_action/{account.id}?menu_id={menu.id}"
        else:
            url = f"/odoo/action-iap.iap_account_action?menu_id={menu.id}"
        return url

    @api.model
    def get_credits(self, service_name):
        account = self.get(service_name, force_create=False)
        credit = 0

        if account:
            route = '/iap/1/balance'
            endpoint = iap_tools.iap_get_endpoint(self.env)
            url = werkzeug.urls.url_join(endpoint, route)
            params = {
                'dbuuid': self.env['ir.config_parameter'].sudo().get_param('database.uuid'),
                'account_token': account.account_token,
                'service_name': service_name,
            }
            try:
                credit = iap_tools.iap_jsonrpc(url=url, params=params)
            except AccessError as e:
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

## File: models\iap_service.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class IapService(models.Model):
    _name = 'iap.service'
    _description = 'IAP Service'

    name = fields.Char(required=True)
    technical_name = fields.Char(readonly=True, required=True)
    description = fields.Char(required=True, translate=True)
    unit_name = fields.Char(required=True, translate=True)
    integer_balance = fields.Boolean(required=True)

    _sql_constraints = [
        ('unique_technical_name', 'UNIQUE(technical_name)', 'Only one service can exist with a specific technical_name')
    ]

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import iap_account
from . import iap_enrich_api
from . import iap_service

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_client_iap_account_manager,iap.account.manager,model_iap_account,base.group_system,1,1,1,1
access_client_iap_account_user,iap.account.user,model_iap_account,base.group_user,1,0,1,0
access_iap_service_manager,iap.service.manager,model_iap_service,base.group_system,1,1,1,1
access_iap_service_user,iap.service.user,model_iap_service,base.group_user,1,0,0,0

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
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M0 12h50v26a4 4 0 0 1-4 4H0V12Z" fill="#03AF89"/><path d="M4 21a4 4 0 0 1-4-4v-5a4 4 0 0 1 4-4h46v9a4 4 0 0 1-4 4H4Z" fill="#FBB945"/><path d="M0 16h50v4a4 4 0 0 1-4 4H4a4 4 0 0 1-4-4v-4Z" fill="#1A6F66"/></svg>

```

## File: static\src\action_buttons_widget\action_buttons_widget.js

```javascript
/** @odoo-modules */

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";
import { Component } from "@odoo/owl";

class IAPActionButtonsWidget extends Component {
    static template = "iap.ActionButtonsWidget";
    static props = {
        ...standardWidgetProps,
        serviceName: String,
        showServiceButtons: Boolean,
    };

    setup() {
        this.orm = useService("orm");
        this.action = useService("action");
    }

    async onViewServicesClicked() {
        this.action.doAction("iap.iap_account_action");
    }

    async onManageServiceLinkClicked() {
        const account_id = await this.orm.silent.call("iap.account", "get_account_id", [this.props.serviceName]);
        this.action.doAction({
            type: "ir.actions.act_window",
            res_model: "iap.account",
            res_id: account_id,
            views: [[false, "form"]],
        });
    }
}

export const iapActionButtonsWidget = {
    component: IAPActionButtonsWidget,
    extractProps: ({ attrs }) => {
        return {
            serviceName: attrs.service_name,
            showServiceButtons: !Boolean(attrs.hide_service),
        };
    },
};
registry.category("view_widgets").add("iap_buy_more_credits", iapActionButtonsWidget);

```

## File: static\src\action_buttons_widget\action_buttons_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template id="template" xml:space="preserve">

    <t t-name="iap.ActionButtonsWidget">
        <div class="row">
            <div class="col-sm">
                <button t-on-click="onManageServiceLinkClicked" class="btn btn-link px-0 o-hidden-ios text-nowrap"><i class="oi oi-arrow-right"/> Manage Service &amp; Buy Credits</button><br/>
                <button t-if="props.showServiceButtons" t-on-click="onViewServicesClicked" class="btn btn-link px-0 o-hidden-ios text-nowrap"><i class="oi oi-arrow-right"/> View My Services</button>
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

## File: static\src\js\insufficient_credit_error_handler.js

```javascript
/** @odoo-module */
import { Dialog } from "@web/core/dialog/dialog";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { _t } from "@web/core/l10n/translation";
import { Component, onWillStart } from "@odoo/owl";

class InsufficientCreditDialog extends Component {
    static components = { Dialog };
    static template = "iap.InsufficientCreditDialog";
    static props = {
        errorData: Object,
        close: Function,
    };
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
    <t t-name="iap.InsufficientCreditDialog">
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
import threading
import uuid

from odoo import exceptions, _
from odoo.tools import email_normalize, exception_to_unicode

_logger = logging.getLogger(__name__)

DEFAULT_ENDPOINT = 'https://iap.odoo.com'


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
    'eim.ae',
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
    if hasattr(threading.current_thread(), 'testing') and threading.current_thread().testing:
        raise exceptions.AccessError("Unavailable during tests.")

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
        _logger.info("iap jsonrpc %s responded in %.3f seconds", url, req.elapsed.total_seconds())
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
            _("An error occurred while reaching %s. Please contact Odoo support if this error persists.", url)
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
            arguments['body'] = env['ir.qweb']._render(credit_template)
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
                        <group>
                            <field name="name" />
                            <field name="service_id" readonly="service_locked" options="{'no_open': True, 'no_create': True}"/>
                            <field name="description" />
                            <field name="company_ids" widget="many2many_tags"
                                domain="[('id', 'in', allowed_company_ids)]" />
                            <field name="account_token" groups="base.group_no_one"/>
                        </group>
                        <group name="external">
                            <label for="balance" class="oe_inline"/>
                            <div>
                                <field name="balance" class="oe_inline"/>
                                <button type="object"
                                    name="action_buy_credits" class="btn-link py-0">
                                    <i class="oi oi-arrow-right"/>
                                    Buy Credit
                                </button>
                            </div>
                            <field name="warning_threshold"/>
                            <field name="warning_user_ids"
                                   invisible="warning_threshold == 0"
                                   required="warning_threshold > 0"
                                   widget="many2many_tags" />
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    <record id="iap_account_view_tree" model="ir.ui.view">
        <field name="name">iap.account.list</field>
        <field name="model">iap.account</field>
        <field name="arch" type="xml">
            <list string="IAP Accounts">
                <field name="name" />
                <field name="service_id" />
                <field name="company_ids" widget="many2many_tags" />
                <field name="account_token" groups="base.group_no_one"/>
                <field name="balance" />
                <field name="warning_threshold" optional="hidden" />
                <field name="description" optional="hidden" />
            </list>
        </field>
    </record>
    <!-- Actions -->
    <record id="iap_account_action" model="ir.actions.act_window">
        <field name="name">IAP Account</field>
        <field name="res_model">iap.account</field>
        <field name="path">iap-accounts</field>
        <field name='view_mode'>list,form</field>
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
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.base.setup.iap</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='contacts_settings']" position="inside">
                <div id="iap_portal">
                    <block class='iap_portal' name="iap_purchases_setting_container">
                        <setting id="iap_credits_setting" string="Odoo IAP" help ="View your IAP Services and recharge your credits" documentation="/applications/general/in_app_purchase.html">
                            <button name="iap.iap_account_action" icon="oi-arrow-right" type="action" string="View My Services" class="btn-link"/>
                        </setting>
                    </block>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

