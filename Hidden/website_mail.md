# Odoo Module: website_mail

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Website Mail',
    'category': 'Hidden',
    'summary': 'Website Module for Mail',
    'version': '0.1',
    'description': """
Module holding mail improvements for website. It holds the follow widget.
""",
    'depends': ['website', 'mail'],
    'data': [
        'views/website_mail_templates.xml',
    ],
    'installable': True,
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            'website_mail/static/src/js/follow.js',
            'website_mail/static/src/css/website_mail.scss',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import http, _
from odoo.http import request
from odoo.exceptions import UserError


class WebsiteMail(http.Controller):

    @http.route(['/website_mail/follow'], type='json', auth="public", website=True)
    def website_message_subscribe(self, id=0, object=None, message_is_follower="on", email=False, **post):
        # TDE FIXME: check this method with new followers
        if not request.env['ir.http']._verify_request_recaptcha_token('website_mail_follow'):
            raise UserError(_("Suspicious activity detected by Google reCaptcha."))
        res_id = int(id)
        is_follower = message_is_follower == 'on'
        record = request.env[object].browse(res_id).exists()
        if not record:
            return False

        record.check_access_rights('read')
        record.check_access_rule('read')

        # search partner_id
        if request.env.user != request.website.user_id:
            partner_ids = request.env.user.partner_id.ids
        else:
            # mail_thread method
            partner_ids = [p.id for p in request.env['mail.thread'].sudo()._mail_find_partner_from_emails([email], records=record.sudo()) if p]
            if not partner_ids or not partner_ids[0]:
                name = email.split('@')[0]
                partner_ids = request.env['res.partner'].sudo().create({'name': name, 'email': email}).ids
        # add or remove follower
        if is_follower:
            record.sudo().message_unsubscribe(partner_ids)
            return False
        else:
            # add partner to session
            request.session['partner_id'] = partner_ids[0]
            record.sudo().message_subscribe(partner_ids)
            return True

    @http.route(['/website_mail/is_follower'], type='json', auth="public", website=True)
    def is_follower(self, records, **post):
        """ Given a list of `models` containing a list of res_ids, return
            the res_ids for which the user is follower and some practical info.

            :param records: dict of models containing record IDS, eg: {
                    'res.model': [1, 2, 3..],
                    'res.model2': [1, 2, 3..],
                    ..
                }

            :returns: [
                    {'is_user': True/False, 'email': 'admin@yourcompany.example.com'},
                    {'res.model': [1, 2], 'res.model2': [1]}
                ]
        """
        user = request.env.user
        partner = None
        public_user = request.website.user_id
        if user != public_user:
            partner = request.env.user.partner_id
        elif request.session.get('partner_id'):
            partner = request.env['res.partner'].sudo().browse(request.session.get('partner_id'))

        res = {}
        if partner:
            for model in records:
                mail_followers_ids = request.env['mail.followers'].sudo().read_group([
                    ('res_model', '=', model),
                    ('res_id', 'in', records[model]),
                    ('partner_id', '=', partner.id)
                ], ['res_id', 'follow_count:count(id)'], ['res_id'])
                # `read_group` will filter out the ones without count result
                for m in mail_followers_ids:
                    res.setdefault(model, []).append(m['res_id'])

        return [{
            'is_user': user != public_user,
            'email': partner.email if partner else "",
        }, res]

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\update.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class PublisherWarrantyContract(models.AbstractModel):
    _inherit = "publisher_warranty.contract"

    @api.model
    def _get_message(self):
        msg = super(PublisherWarrantyContract, self)._get_message()
        msg['website'] = True
        return msg

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import update

```

## File: static\src\js\follow.js

```javascript
odoo.define('website_mail.follow', function (require) {
'use strict';

var publicWidget = require('web.public.widget');
const { ReCaptcha } = require("google_recaptcha.ReCaptchaV3");
const { _t } = require("web.core");

publicWidget.registry.follow = publicWidget.Widget.extend({
    selector: '#wrapwrap:has(.js_follow)',
    disabledInEditableMode: false,

    /**
     * @constructor
     */
    init() {
        this._super(...arguments);
        this._recaptcha = new ReCaptcha();
    },

    async willStart() {
        return this._recaptcha.loadLibs();
    },

    /**
     * @override
     */
    start: function () {
        var self = this;
        this.isUser = false;
        var $jsFollowEls = this.$el.find('.js_follow');

        var always = function (data) {
            self.isUser = data[0].is_user;
            const $jsFollowToEnable = $jsFollowEls.filter(function () {
                const model = this.dataset.object;
                return model in data[1] && data[1][model].includes(parseInt(this.dataset.id));
            });
            self._toggleSubscription(true, data[0].email, $jsFollowToEnable);
            self._toggleSubscription(false, data[0].email, $jsFollowEls.not($jsFollowToEnable));
            $jsFollowEls.removeClass('d-none');
        };

        const records = {};
        for (const el of $jsFollowEls) {
            const model = el.dataset.object;
            if (!(model in records)) {
                records[model] = [];
            }
            records[model].push(parseInt(el.dataset.id));
        }

        this._rpc({
            route: '/website_mail/is_follower',
            params: {
                records: records,
            },
        }).then(always).guardedCatch(always);

        // not if editable mode to allow designer to edit
        if (!this.editableMode) {
            $('.js_follow > .input-group-append.d-none').removeClass('d-none');
            this.$target.find('.js_follow_btn, .js_unfollow_btn').on('click', function (event) {
                event.preventDefault();
                self._onClick(event);
            });
        }
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Toggles subscription state for every given records.
     *
     * @private
     * @param {boolean} follow
     * @param {string} email
     * @param {jQuery} $jsFollowEls
     */
    _toggleSubscription: function (follow, email, $jsFollowEls) {
        if (follow) {
            this._updateSubscriptionDOM(follow, email, $jsFollowEls);
        } else {
            for (const el of $jsFollowEls) {
                const follow = !email && el.getAttribute('data-unsubscribe');
                this._updateSubscriptionDOM(follow, email, $(el));
            }
        }
    },
    /**
     * Updates subscription DOM for every given records.
     * This should not be called directly, use `_toggleSubscription`.
     *
     * @private
     * @param {boolean} follow
     * @param {string} email
     * @param {jQuery} $jsFollowEls
     */
    _updateSubscriptionDOM: function (follow, email, $jsFollowEls) {
        $jsFollowEls.find('input.js_follow_email')
            .val(email || "")
            .attr("disabled", email && (follow || this.isUser) ? "disabled" : false);
        $jsFollowEls.attr("data-follow", follow ? 'on' : 'off');
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    async _onClick(ev) {
        var self = this;
        var $jsFollow = $(ev.currentTarget).closest('.js_follow');
        var $email = $jsFollow.find(".js_follow_email");

        if ($email.length && !$email.val().match(/.+@.+/)) {
            $jsFollow.addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
            return false;
        }
        $jsFollow.removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');

        var email = $email.length ? $email.val() : false;
        if (email || this.isUser) {
            const tokenCaptcha = await this._recaptcha.getToken("website_mail_follow");
            const token = tokenCaptcha.token;

            if (tokenCaptcha.error) {
                self.displayNotification({
                    type: "danger",
                    title: _t("Error"),
                    message: tokenCaptcha.error,
                    sticky: true
                });
                return false;
            }
            this._rpc({
                route: '/website_mail/follow',
                params: {
                    'id': +$jsFollow.data('id'),
                    'object': $jsFollow.data('object'),
                    'message_is_follower': $jsFollow.attr("data-follow") || "off",
                    'email': email,
                    'recaptcha_token_response': token,
                },
            }).then(function (follow) {
                self._toggleSubscription(follow, email, $jsFollow);
            });
        }
    },
});
});

```

## File: views\website_mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="follow">
        <div t-attf-class="input-group js_follow #{div_class}" t-att-data-id="object.id"
                  t-att-data-object="object._name"
                  t-att-data-follow="object.id and object.message_is_follower and 'on' or 'off'"
                  t-att-data-unsubscribe="'unsubscribe' if 'unsubscribe' in request.params else None">
            <input
                  type="email" name="email"
                  class="js_follow_email form-control"
                  placeholder="your email..."
                  groups="base.group_public"/>
            <div t-if="icons_design and not request.env.user.has_group('base.group_public')" class="js_follow_icons_container">
                <button class="btn text-reset js_unfollow_btn">
                    <div class="d-flex align-items-center">
                        <small>Unfollow</small><i class="fa fa-fw ml-1"/>
                    </div>
                </button>
                <button class="btn text-reset js_follow_btn">
                    <div class="d-flex align-items-center">
                        <small>Follow</small><i class="fa fa-fw ml-1"/>
                    </div>
                </button>
            </div>
            <div t-else="" t-attf-class="#{request.env.user.has_group('base.group_public') and 'input-group-append'}">
                <button href="#" t-attf-class="btn btn-secondary js_unfollow_btn">Unsubscribe</button>
                <button href="#" t-attf-class="btn btn-primary js_follow_btn">Subscribe</button>
            </div>
        </div>
    </template>
</odoo>

```

