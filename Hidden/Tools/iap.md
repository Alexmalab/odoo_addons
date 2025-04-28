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
        'views/assets.xml',
        'views/iap_views.xml',
        'views/res_config_settings.xml',
    ],
    'qweb': [
        'static/src/xml/iap_templates.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

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

    @api.model
    def create(self, vals):
        account = super().create(vals)
        if self.env['ir.config_parameter'].sudo().get_param('database.is_neutralized') and account.account_token:
            # Disable new accounts on a neutralized database
            account.account_token = f"{account.account_token.split('+')[0]}+disabled"
        return account

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
                self.flush()
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
                self.flush()
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

## File: static\src\js\crash_manager.js

```javascript
odoo.define('iap.CrashManager', function (require) {
"use strict";

var ajax = require('web.ajax');
var core = require('web.core');
var CrashManager = require('web.CrashManager').CrashManager;
var Dialog = require('web.Dialog');

var _t = core._t;
var QWeb = core.qweb;

CrashManager.include({
    /**
     * Change button string depending if it's Enterprise and trial available
     * (Only change the text message in the button, doesn't change IAP side validation)
     *
     * @param {boolean} isTrial
     * @returns {string}
     * @private
     */
    _getButtonMessage: function (isTrial){
        var isEnterprise = _.last(odoo.session_info.server_version_info) === 'e';
        return isTrial && isEnterprise ? _t('Start a Trial at Odoo') : _t('Buy credits');
    },
    /**
     * @override
     */
    rpc_error: function (error) {
        var self = this;
        if (error.data.name === "odoo.addons.iap.tools.iap_tools.InsufficientCreditError") {
            var error_data = JSON.parse(error.data.message);
            ajax.jsonRpc('/web/dataset/call_kw', 'call', {
                model:  'iap.account',
                method: 'get_credits_url',
                args: [],
                kwargs: {
                    base_url: error_data.base_url,
                    service_name: error_data.service_name,
                    credit: error_data.credit,
                    trial: error_data.trial
                }
            }).then(function (url) {
                var content = $(QWeb.render('iap.redirect_to_odoo_credit', {
                        data: error_data,
                    }));
                if (error_data.body) {
                    content.css('padding', 0);
                }
                new Dialog(this, {
                    size: 'large',
                    title: error_data.title || _t("Insufficient Balance"),
                    $content: content,
                    buttons: [{
                        text: self._getButtonMessage(error_data.trial),
                        classes : "btn-primary",
                        click: function () {
                            window.open(url, '_blank');
                        },
                        close:true,
                    }, {
                        text: _t("Cancel"),
                        close: true,
                    }],
                }).open();
            });
        } else {
            this._super.apply(this, arguments);
        }
    },
});

});

```

## File: static\src\js\iap_buy_more_credits.js

```javascript
odoo.define('iap.buy_more_credits', function (require) {
'use strict';

var widgetRegistry = require('web.widget_registry');
var Widget = require('web.Widget');

var core = require('web.core');

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
    },

    /**
     * @override
     */
    start: function () {
        this.$widget = $(QWeb.render('iap.buy_more_credits'));
        this.$buyLink = this.$widget.find('.buy_credits');
        this.$widget.appendTo(this.$el);
        this.$buyLink.click(this._getLink.bind(this));
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    _getLink: function () {
        var self = this;
        return this._rpc({
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
});

widgetRegistry.add('iap_buy_more_credits', IAPBuyMoreCreditsWidget);

return IAPBuyMoreCreditsWidget;
});

```

## File: static\src\js\iap_credit.js

```javascript
odoo.define('iap.redirect_odoo_credit_widget', function(require) {
"use strict";

var AbstractAction = require('web.AbstractAction');
var core = require('web.core');


var IapOdooCreditRedirect = AbstractAction.extend({
    template: 'iap.redirect_to_odoo_credit',
    events : {
        "click .redirect_confirm" : "odoo_redirect",
    },
    init: function (parent, action) {
        this._super(parent, action);
        this.url = action.params.url;
    },

    odoo_redirect: function () {
        window.open(this.url, '_blank');
        this.do_action({type: 'ir.actions.act_window_close'});
        // framework.redirect(this.url);
    },

});
core.action_registry.add('iap_odoo_credit_redirect', IapOdooCreditRedirect);
});

```

## File: static\src\xml\iap_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template id="template" xml:space="preserve">

    <!-- LAYOUT TEMPLATES -->
    <div t-name="iap.redirect_to_odoo_credit">
        <t t-if="data.body">
            <div t-raw="data.body"/>
        </t>
        <t t-if="!data.body">
            <t t-if="data.message">
                <span t-esc="data.message"/>
            </t>
            <t t-if="!data.message">
                <span>Insufficient credit to perform this service.</span>
            </t>
        </t>
    </div>

    <t t-extend="DashboardMain">
        <t t-jquery=".o_web_settings_apps" t-operation="after">
            <div class="o_web_settings_iap"></div>
        </t>
    </t>
    
    <div t-name="iap.buy_more_credits" class="mt-2 row">
        <div class="col-sm">
            <button class="btn btn-link buy_credits o-hidden-ios"><i class="fa fa-arrow-right"/> Buy credits</button>
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
from odoo.tools import pycompat

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
            message = response['error']['data'].get('message')
            if name == 'InsufficientCreditError':
                e_class = InsufficientCreditError
            elif name == 'AccessError':
                e_class = exceptions.AccessError
            elif name == 'UserError':
                e_class = exceptions.UserError
            else:
                raise requests.exceptions.ConnectionError()
            e = e_class(message)
            e.data = response['error']['data']
            raise e
        return response.get('result')
    except (ValueError, requests.exceptions.ConnectionError, requests.exceptions.MissingSchema, requests.exceptions.Timeout, requests.exceptions.HTTPError) as e:
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
        'ttl': ttl
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
def iap_charge(env, key, account_token, credit, dbuuid=False, description=None, credit_template=None):
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
    :type credit_template: str
    """
    transaction_token = iap_authorize(env, key, account_token, credit, dbuuid, description, credit_template)
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

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="iap assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/iap/static/src/js/iap_credit.js"></script>
            <script type="text/javascript" src="/iap/static/src/js/crash_manager.js"></script>
            <script type="text/javascript" src="/iap/static/src/js/iap_buy_more_credits.js"></script>
        </xpath>
    </template>

    <template id="tests_assets" name="iap tests assets" inherit_id="web.tests_assets">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/iap/static/tests/helpers/mock_server.js"/>
        </xpath>
    </template>
</odoo>

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
            <xpath expr="//div[@id='multi_company']" position="before">
                <div id="iap_portal">
                    <h2>In-App Purchases</h2>
                    <div class='row mt16 o_settings_container iap_portal' name="iap_purchases_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="iap_credits_setting">
                            <div class='o_setting_right_pane'>
                                <div class="o_form_label">
                                Odoo IAP
                                <a href="https://www.odoo.com/documentation/14.0/applications/general/in_app_purchase/in_app_purchase.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <a href="https://www.odoo.com/documentation/14.0/developer/webservices/iap.html" title="Documentation" class="ml-1 o_doc_link" target="_blank"></a>
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

