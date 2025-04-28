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
            <div class="col-sm-2 p-0 text-center text-md-end order-first order-md-last">
                <img t-attf-src="{{logo}}" alt="Company Logo" style="max-width: 80px;"/>
            </div>
            </div>
            <hr/>

            <div class="col-sm-12 row m-0 p-0">
                <div t-if="company_type" class="my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-building text-primary"/>
                    <b>Company type</b>
                </div>
                <div t-if="company_type" class="my-1 col-sm-9" t-esc="company_type" />
                <div t-if="founded_year" class="my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-calendar text-primary"/>
                    <b>Founded</b>
                </div>
                <div t-if="founded_year" class="my-1 col-sm-9" t-esc="founded_year" />
                <t t-set="sectors" t-value="[]" />
                <t t-if="sector_primary" t-set="sectors" t-value="sectors + [sector_primary]" />
                <t t-if="industry" t-set="sectors" t-value="sectors + [industry]" />
                <t t-if="industry_group" t-set="sectors" t-value="sectors + [industry_group]" />
                <t t-if="sub_industry" t-set="sectors" t-value="sectors + [sub_industry]" />
                <div t-if="sectors" class="my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-industry text-primary"/>
                    <b>Sectors</b>
                </div>
                <div t-if="sectors" class="my-1 col-sm-9">
                    <t t-foreach="sectors" t-as="inner_sector">
                        <label t-esc="inner_sector" class="o_tag o_tag_color_7" style="font-weight:normal; padding: 2px 10px; margin: 1px 0px; border-radius: 13px; display: inline-block;"/>
                    </t>
                </div>
                <div t-if="employees" class="my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-users text-primary"/>
                    <b>Employees</b>
                </div>
                <div t-if="employees" class="my-1 col-sm-9" t-esc="'%.0f' % employees" />
                <div t-if="estimated_annual_revenue" class="my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-money text-primary"/>
                    <b>Estimated revenue</b>
                </div>
                <div t-if="estimated_annual_revenue" class="my-1 col-sm-9">
                    <span t-esc="estimated_annual_revenue" /><span> per year</span>
                </div>
                <div t-if="phone_numbers" class="my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-phone text-primary"/>
                    <b>Phone</b>
                </div>
                <div t-if="phone_numbers" class="col-sm-9">
                    <t t-foreach="phone_numbers" t-as="phone_number">
                        <a t-attf-href="tel:{{phone_number}}" t-esc="phone_number" class="o_tag o_tag_color_7"  style="font-weight:normal; padding: 2px 10px; margin: 1px 0px; border-radius: 13px; display: inline-block;"/>
                    </t>
                </div>
                <div t-if="email" class="my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-envelope text-primary"/>
                    <b>Email</b>
                </div>
                <div t-if="email" class="col-sm-9">
                    <t t-foreach="email" t-as="email_item">
                        <a target="_top" t-attf-href="mailto:{{email_item}}" t-esc="email_item" class="o_tag o_tag_color_7"  style="font-weight:normal; padding: 2px 10px; margin: 1px 0px; border-radius: 13px; display: inline-block;"/>
                    </t>
                </div>
                <div t-if="timezone" class="my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-globe text-primary"/>
                    <b>Timezone</b>
                </div>
                <div t-if="timezone" class="my-1 col-sm-9" t-esc="timezone.replace('_', ' ')" />
                <div t-if="tech" class="my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-cube text-primary"/>
                    <b>Technologies Used</b>
                </div>
                <div t-if="tech" class="my-1 col-sm-9">
                    <t t-foreach="tech" t-as="tech_item">
                        <label t-esc="tech_item.replace('_', ' ').title()" class="o_tag o_tag_color_7"  style="font-weight:normal; padding: 2px 10px; margin: 1px 0px; border-radius: 13px; display: inline-block;"/>
                    </t>
                </div>
                <div t-if="twitter_bio" class="my-1 p-0 col-sm-3">
                    <i class="fa fa-fw me-2 fa-twitter text-primary"/>
                    <b>Twitter</b>
                </div>
                <div t-if="twitter_bio" class="my-1 col-sm-9">
                    <div t-if="twitter_followers"><t t-esc="twitter_followers"/> followers</div>
                    <div t-esc="twitter_bio" />
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
    def _send_iap_bus_notification(self, service_name, title, error_type=False):
        param = {
            'title': title,
            'error_type': 'danger' if error_type else 'success'
        }
        if error_type == 'credit':
            param['url'] = self.env['iap.account'].get_credits_url(service_name)
        self.env['bus.bus']._sendone(self.env.user.partner_id, 'iap_notification', param)

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

import { Markup } from 'web.utils';
import { registry } from "@web/core/registry";

export const iapNotificationService = {
    dependencies: ["bus_service", "notification"],

    start(env, { bus_service, notification }) {
        bus_service.addEventListener('notification', ({ detail: notifications }) => {
            for (const { payload, type } of notifications) {
                if (type === 'iap_notification') {
                    if (payload.error_type == 'success') {
                        displaySuccessIapNotification(payload);
                    } else if (payload.error_type == 'danger') {
                        displayFailureIapNotification(payload);
                    }
                }
            }
        });
        bus_service.start();

        /**
         * Displays the IAP success notification on user's screen
         */
        function displaySuccessIapNotification(notif) {
            notification.add(notif.title, {
                type: notif.error_type,
            });
        }

        /**
         * Displays the IAP failure notification on user's screen
         */
        function displayFailureIapNotification(notif) {
            const message = Markup`<a class='btn btn-link' href='${notif.url}' target='_blank' ><i class='fa fa-arrow-right'></i> ${env._t("Buy more credits")}</a>`;
            notification.add(message, {
                type: notif.error_type,
                title: notif.title
            });
        }
    }
};

registry.category("services").add("iapNotification", iapNotificationService);

```

