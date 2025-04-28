# Odoo Module: iap_mail

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- encoding: utf-8 -*-

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


{
    'name': "IAP / Mail",
    'summary': """Bridge between IAP and mail""",
    'description': """Bridge between IAP and mail""",
    'category': 'Hidden/Tools',
    'version': '1.0',
    'depends': [
        'iap',
        'mail',
    ],
    'installable': True,
    'auto_install': True,
    'data': [
        'data/mail_templates.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'iap_mail/static/src/js/**/*',
            'iap_mail/static/src/scss/iap_mail.scss',
        ],
        "web.dark_mode_assets_backend": [
            'iap_mail/static/src/scss/iap_mail.dark.scss',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="enrich_company">
        <p t-esc="flavor_text" />
        <div class="o_partner_autocomplete_enrich_info p-3 mt-3 mb-3 me-5">
            <div class="row p-0 m-0">
            <div class="col-sm-10 p-0">
                <h4>
                    <span class="me-3 align-middle" t-esc="name"/>
                    <a t-if="twitter" class="ms-2" target="_blank" t-attf-href="http://www.twitter.com/{{twitter}}">
                        <img src="/web_editor/font_to_img/61569/rgb(0,132,180)/22"/>
                    </a>
                    <a t-if="facebook" class="ms-2" target="_blank" t-attf-href="http://www.facebook.com/{{facebook}}">
                        <img src="/web_editor/font_to_img/61570/rgb(59,89,152)/22"/>
                    </a>
                    <a t-if="linkedin" class="ms-2" target="_blank" t-attf-href="https://www.linkedin.com/{{linkedin}}">
                        <img src="/web_editor/font_to_img/61580/rgb(0,119,181)/22"/>
                    </a>
                    <a t-if="crunchbase" class="ms-2" target="_blank" t-attf-href="https://www.crunchbase.com/{{crunchbase}}">
                        <img width="19px" height="19px" src="/partner_autocomplete/static/img/crunchbase.ico"/>
                    </a>
                </h4>
                <p t-esc="description"/>
            </div>
            <div t-if="logo" class="col-sm-2 p-0 text-center text-md-end order-first order-md-last">
                <img t-attf-src="{{logo}}" alt="" style="max-width: 80px;"/>
            </div>
            </div>
            <hr/>

            <div class="col-sm-12 row m-0 p-0">
                <div t-if="company_type" class="d-flex my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-building text-primary"/>
                    <b>Company type</b>
                </div>
                <div t-if="company_type" class="my-1 col-sm-9" t-esc="company_type" />
                <div t-if="founded_year" class="d-flex my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-calendar text-primary"/>
                    <b>Founded</b>
                </div>
                <div t-if="founded_year" class="my-1 col-sm-9" t-esc="founded_year" />
                <t t-set="sectors" t-value="[]" />
                <t t-if="sector_primary" t-set="sectors" t-value="sectors + [sector_primary]" />
                <t t-if="industry" t-set="sectors" t-value="sectors + [industry]" />
                <t t-if="industry_group" t-set="sectors" t-value="sectors + [industry_group]" />
                <t t-if="sub_industry" t-set="sectors" t-value="sectors + [sub_industry]" />
                <div t-if="sectors" class="d-flex my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-industry text-primary"/>
                    <b>Sectors</b>
                </div>
                <div t-if="sectors" class="my-1 col-sm-9">
                    <t t-foreach="sectors" t-as="inner_sector">
                        <label t-esc="inner_sector" class="o_tag o_tag_color_7" style="font-weight:normal; padding: 2px 10px; margin: 1px 0px; border-radius: 13px; display: inline-block;"/>
                    </t>
                </div>
                <div t-if="employees" class="d-flex my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-users text-primary"/>
                    <b>Employees</b>
                </div>
                <div t-if="employees" class="my-1 col-sm-9" t-esc="'%.0f' % employees" />
                <div t-if="estimated_annual_revenue" class="d-flex my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-money text-primary"/>
                    <b>Estimated revenue</b>
                </div>
                <div t-if="estimated_annual_revenue" class="my-1 col-sm-9">
                    <span t-esc="estimated_annual_revenue" /><span> per year</span>
                </div>
                <div t-if="phone_numbers" class="d-flex my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-phone text-primary"/>
                    <b>Phone</b>
                </div>
                <div t-if="phone_numbers" class="col-sm-9">
                    <t t-foreach="phone_numbers" t-as="phone_number">
                        <a t-attf-href="tel:{{phone_number}}" t-esc="phone_number" class="o_tag o_tag_color_7"  style="font-weight:normal; padding: 2px 10px; margin: 1px 0px; border-radius: 13px; display: inline-block;"/>
                    </t>
                </div>
                <div t-if="email" class="d-flex my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-envelope text-primary"/>
                    <b>Email</b>
                </div>
                <div t-if="email" class="col-sm-9">
                    <t t-foreach="email" t-as="email_item">
                        <a target="_top" t-attf-href="mailto:{{email_item}}" t-esc="email_item" class="o_tag o_tag_color_7"  style="font-weight:normal; padding: 2px 10px; margin: 1px 0px; border-radius: 13px; display: inline-block;"/>
                    </t>
                </div>
                <div t-if="timezone" class="d-flex my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-globe text-primary"/>
                    <b>Timezone</b>
                </div>
                <div t-if="timezone" class="my-1 col-sm-9" t-esc="timezone.replace('_', ' ')" />
                <div t-if="tech" class="d-flex my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-cube text-primary"/>
                    <b>Technologies Used</b>
                </div>
                <div t-if="tech" class="my-1 col-sm-9">
                    <t t-foreach="tech" t-as="tech_item">
                        <label t-esc="tech_item.replace('_', ' ').title()" class="o_tag o_tag_color_7"  style="font-weight:normal; padding: 2px 10px; margin: 1px 0px; border-radius: 13px; display: inline-block;"/>
                    </t>
                </div>
                <div t-if="twitter_bio" class="d-flex my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-twitter text-primary"/>
                    <b>Twitter</b>
                </div>
                <div t-if="twitter_bio" class="my-1 col-sm-9">
                    <div class="d-flex gap-2">
                        <a t-if="twitter" target="_blank" t-attf-href="http://www.twitter.com/{{twitter}}" class="text-nowrap">
                            http://www.twitter.com/<t t-esc="twitter"/>
                        </a>
                        <span t-if="twitter"> • </span>
                        <div t-if="twitter_followers" class="text-nowrap"><t t-esc="twitter_followers"/> followers</div>
                    </div>
                    <div t-esc="twitter_bio" class="mt-1"/>
                </div>
            </div>
        </div>
    </template>
</odoo>

```

## File: models\iap_account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class IapAccount(models.Model):
    _inherit = 'iap.account'

    @api.model
    def _send_success_notification(self, message, title=None):
        self._send_status_notification(message, 'success', title=title)

    @api.model
    def _send_error_notification(self, message, title=None):
        self._send_status_notification(message, 'danger', title=title)

    @api.model
    def _send_status_notification(self, message, status, title=None):
        params = {
            'message': message,
            'type': status,
        }

        if title is not None:
            params['title'] = title

        self.env['bus.bus']._sendone(self.env.user.partner_id, 'iap_notification', params)

    @api.model
    def _send_no_credit_notification(self, service_name, title):
        params = {
            'title': title,
            'type': 'no_credit',
            'get_credits_url': self.env['iap.account'].get_credits_url(service_name),
        }
        self.env['bus.bus']._sendone(self.env.user.partner_id, 'iap_notification', params)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import iap_account

```

## File: static\src\js\services\iap_notification_service.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";

import { markup } from "@odoo/owl";

export const iapNotificationService = {
    dependencies: ["bus_service", "notification"],

    start(env, { bus_service, notification }) {
        bus_service.subscribe("iap_notification", (params) => {
            if (params.type == "no_credit") {
                displayCreditErrorNotification(params);
            } else {
                displayNotification(params);
            }
        });
        bus_service.start();

        function displayNotification(params) {
            notification.add(params.message, {
                title: params.title,
                type: params.type,
            });
        }

        function displayCreditErrorNotification(params) {
            // ℹ️ `_t` can only be inlined directly inside JS template literals
            // after Babel has been updated to version 2.12.
            const translatedText = _t("Buy more credits");
            const message = markup(`
            <a class='btn btn-link' href='${params.get_credits_url}' target='_blank'>
                <i class='oi oi-arrow-right'></i>
                ${translatedText}
            </a>`);
            notification.add(message, {
                title: params.title,
                type: 'danger',
            });
        }
    }
};

registry.category("services").add("iapNotification", iapNotificationService);

```

