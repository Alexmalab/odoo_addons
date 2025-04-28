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
Module holding mail improvements for website.
It is responsible of comments moderation for published documents (forum, slides, blog, ...)
""",
    'depends': ['website', 'mail'],
    'data': [
        'views/assets.xml',
        'views/website_mail_templates.xml',
        'security/website_mail_security.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import http
from odoo.http import request


class WebsiteMail(http.Controller):

    @http.route(['/website_mail/follow'], type='json', auth="public", website=True)
    def website_message_subscribe(self, id=0, object=None, message_is_follower="on", email=False, **post):
        # TDE FIXME: check this method with new followers
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
    def is_follower(self, model, res_id, **post):
        user = request.env.user
        partner = None
        public_user = request.website.user_id
        if user != public_user:
            partner = request.env.user.partner_id
        elif request.session.get('partner_id'):
            partner = request.env['res.partner'].sudo().browse(request.session.get('partner_id'))

        values = {
            'is_user': user != public_user,
            'email': partner.email if partner else "",
            'is_follower': False,
            'alias_name': False,
        }

        record = request.env[model].sudo().browse(int(res_id))
        if record and partner:
            values['is_follower'] = bool(request.env['mail.followers'].search_count([
                ('res_model', '=', model),
                ('res_id', '=', record.id),
                ('partner_id', '=', partner.id)
            ]))
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.osv import expression
from odoo.tools import html2plaintext
from odoo.exceptions import AccessError


class MailMessage(models.Model):
    _inherit = 'mail.message'

    @api.model
    def default_get(self, fields_list):
        defaults = super(MailMessage, self).default_get(fields_list)

        # Note: explicitly implemented in default_get() instead of field default,
        # to avoid setting to True for all existing messages during upgrades.
        # TODO: this default should probably be dynamic according to the model
        # on which the messages are attached, thus moved to create().
        if 'website_published' in fields_list:
            defaults.setdefault('website_published', True)

        return defaults

    description = fields.Char(compute="_compute_description", help='Message description: either the subject, or the beginning of the body')
    website_published = fields.Boolean(string='Published', help="Visible on the website as a comment", copy=False)

    @api.model
    def _non_employee_message_domain(self):
        domain = super(MailMessage, self)._non_employee_message_domain()
        return expression.AND([domain, [('website_published', '=', True)]])

    def _compute_description(self):
        for message in self:
            if message.subject:
                message.description = message.subject
            else:
                plaintext_ct = '' if not message.body else html2plaintext(message.body)
                message.description = plaintext_ct[:30] + '%s' % (' [...]' if len(plaintext_ct) >= 30 else '')

    @api.model
    def _search(self, args, offset=0, limit=None, order=None, count=False, access_rights_uid=None):
        """ Override that adds specific access rights of mail.message, to restrict
        messages to published messages for public users. """
        if self.user_has_groups('base.group_public'):
            args = expression.AND([[('website_published', '=', True)], list(args)])

        return super(MailMessage, self)._search(args, offset=offset, limit=limit, order=order,
                                                count=count, access_rights_uid=access_rights_uid)

    def check_access_rule(self, operation):
        """ Add Access rules of mail.message for non-employee user:
            - read:
                - raise if the type is comment and subtype NULL (internal note)
        """
        if self.user_has_groups('base.group_public'):
            self.env.cr.execute('SELECT id FROM "%s" WHERE website_published IS NOT TRUE AND id = ANY (%%s)' % (self._table), (self.ids,))
            if self.env.cr.fetchall():
                raise AccessError(
                    _('The requested operation cannot be completed due to security restrictions. Please contact your system administrator.\n\n(Document type: %s, Operation: %s)') % (self._description, operation)
                    + ' - ({} {}, {} {})'.format(_('Records:'), self.ids[:6], _('User:'), self._uid)
                )
        return super(MailMessage, self).check_access_rule(operation=operation)

    def _portal_message_format(self, fields_list):
        fields_list += ['website_published']
        return super(MailMessage, self)._portal_message_format(fields_list)

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

from . import mail_message
from . import update

```

## File: security\website_mail_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="mail_message_rule_public" model="ir.rule">
            <field name="name">mail.message: portal/public: read published messages</field>
            <field name="model_id" ref="mail.model_mail_message"/>
            <field name="domain_force">[('website_published', '=', True)]</field>
            <field name="groups" eval="[(4, ref('base.group_portal')), (4, ref('base.group_public'))]"/>
        </record>

    </data>
</odoo>

```

## File: static\src\js\follow.js

```javascript
odoo.define('website_mail.follow', function (require) {
'use strict';

var publicWidget = require('web.public.widget');

publicWidget.registry.follow = publicWidget.Widget.extend({
    selector: '.js_follow',
    disabledInEditableMode: false,

    start: function () {
        var self = this;
        this.is_user = false;

        var always = function (data) {
            self.is_user = data.is_user;
            self.email = data.email;
            self.toggle_subscription(data.is_follower, data.email);
            self.$target.removeClass('d-none');
        };

        this._rpc({
            route: '/website_mail/is_follower',
            params: {
                model: this.$target.data('object'),
                res_id: this.$target.data('id'),
            },
        }).then(always).guardedCatch(always);

        // not if editable mode to allow designer to edit
        if (!this.editableMode) {
            $('.js_follow > .input-group-append.d-none').removeClass('d-none');
            this.$target.find('.js_follow_btn, .js_unfollow_btn').on('click', function (event) {
                event.preventDefault();
                self._onClick();
            });
        }
        return this._super.apply(this, arguments);
    },
    _onClick: function () {
        var self = this;
        var $email = this.$target.find(".js_follow_email");

        if ($email.length && !$email.val().match(/.+@.+/)) {
            this.$target.addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
            return false;
        }
        this.$target.removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');

        var email = $email.length ? $email.val() : false;
        if (email || this.is_user) {
            this._rpc({
                route: '/website_mail/follow',
                params: {
                    'id': +this.$target.data('id'),
                    'object': this.$target.data('object'),
                    'message_is_follower': this.$target.attr("data-follow") || "off",
                    'email': email,
                },
            }).then(function (follow) {
                self.toggle_subscription(follow, email);
            });
        }
    },
    toggle_subscription: function (follow, email) {
        follow = follow || (!email && this.$target.attr('data-unsubscribe'));
        if (follow) {
            this.$target.find(".js_follow_btn").addClass('d-none');
            this.$target.find(".js_unfollow_btn").removeClass('d-none');
        }
        else {
            this.$target.find(".js_follow_btn").removeClass('d-none');
            this.$target.find(".js_unfollow_btn").addClass('d-none');
        }
        this.$target.find('input.js_follow_email')
            .val(email || "")
            .attr("disabled", email && (follow || this.is_user) ? "disabled" : false);
        this.$target.attr("data-follow", follow ? 'on' : 'off');
    },
});
});

```

## File: static\src\js\portal_chatter.js

```javascript
odoo.define('website_mail.thread', function (require) {
'use strict';

var portalChatter = require('portal.chatter');

/**
 * Extends Frontend Chatter to handle rating
 */
portalChatter.PortalChatter.include({
    xmlDependencies: (portalChatter.PortalChatter.prototype.xmlDependencies || [])
        .concat(['/website_mail/static/src/xml/portal_chatter.xml']),
});
});

```

## File: static\src\xml\portal_chatter.xml

```xml
<templates id="template" xml:space="preserve">

    <t t-name="website_mail.publish_short">
        <t t-if="is_publisher" t-ignore="true">
            <div t-attf-class="float-right js_publish_management #{object.website_published and 'css_published' or 'css_unpublished'}" t-att-data-id="res_id" t-att-data-object="res_model" t-att-data-controller="publish_controller">
                <button class="btn btn-danger js_publish_btn">Unpublished</button>
                <button class="btn btn-success js_publish_btn">Published</button>
            </div>
        </t>
    </t>

    <t t-extend="portal.chatter_messages">
        <t t-jquery=".o_portal_chatter_message_title" t-operation="before">
            <t t-call="website_mail.publish_short">
                <t t-set="res_model" t-value="'mail.message'"/>
                <t t-set="res_id" t-value="message.id"/>
                <t t-set="object" t-value="message"/>
                <t t-set="is_publisher" t-value="widget.options['is_user_publisher']"/>
            </t>
        </t>
    </t>

</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="head" inherit_id="website.assets_frontend" name="Mail customization">
        <xpath expr="." position="inside">
            <script type="text/javascript" src="/website_mail/static/src/js/follow.js"></script>
            <script type="text/javascript" src="/website_mail/static/src/js/portal_chatter.js"></script>
            <link rel="stylesheet" type="text/scss" href="/website_mail/static/src/css/website_mail.scss"/>
        </xpath>
    </template>
</odoo>

```

## File: views\website_mail_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="follow">
        <div class="input-group js_follow" t-att-data-id="object.id"
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
            <div t-else="" t-attf-class="#{request.env.user.has_group('base.group_public') and 'input-group-append'} #{div_class}">
                <button href="#" t-attf-class="btn btn-secondary js_unfollow_btn">Unsubscribe</button>
                <button href="#" t-attf-class="btn btn-primary js_follow_btn">Subscribe</button>
            </div>
        </div>
    </template>
</odoo>

```

