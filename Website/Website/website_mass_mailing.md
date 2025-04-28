# Odoo Module: website_mass_mailing

Category: Website/Website

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
    'name': 'Newsletter Subscribe Button',
    'summary': 'Attract visitors to subscribe to mailing lists',
    'description': """
This module brings a new building block with a mailing list widget to drop on any page of your website.
On a simple click, your visitors can subscribe to mailing lists managed in the Email Marketing app.
    """,
    'version': '1.0',
    'category': 'Website/Website',
    'depends': ['website', 'mass_mailing'],
    'data': [
        'security/ir.model.access.csv',
        'views/website_mass_mailing_templates.xml',
        'views/snippets_templates.xml',
        'views/mailing_list_views.xml',
        'views/website_mass_mailing_views.xml',
    ],
    'qweb': [
        'static/src/xml/*.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import route, request
from odoo.osv import expression
from odoo.addons.mass_mailing.controllers.main import MassMailController


class MassMailController(MassMailController):

    @route('/website_mass_mailing/is_subscriber', type='json', website=True, auth="public")
    def is_subscriber(self, list_id, **post):
        email = None
        if not request.env.user._is_public():
            email = request.env.user.email
        elif request.session.get('mass_mailing_email'):
            email = request.session['mass_mailing_email']

        is_subscriber = False
        if email:
            contacts_count = request.env['mailing.contact.subscription'].sudo().search_count([('list_id', 'in', [int(list_id)]), ('contact_id.email', '=', email), ('opt_out', '=', False)])
            is_subscriber = contacts_count > 0

        return {'is_subscriber': is_subscriber, 'email': email}

    @route('/website_mass_mailing/subscribe', type='json', website=True, auth="public")
    def subscribe(self, list_id, email, **post):
        ContactSubscription = request.env['mailing.contact.subscription'].sudo()
        Contacts = request.env['mailing.contact'].sudo()
        name, email = Contacts.get_name_email(email)

        subscription = ContactSubscription.search([('list_id', '=', int(list_id)), ('contact_id.email', '=', email)], limit=1)
        if not subscription:
            # inline add_to_list as we've already called half of it
            contact_id = Contacts.search([('email', '=', email)], limit=1)
            if not contact_id:
                contact_id = Contacts.create({'name': name, 'email': email})
            ContactSubscription.create({'contact_id': contact_id.id, 'list_id': int(list_id)})
        elif subscription.opt_out:
            subscription.opt_out = False
        # add email to session
        request.session['mass_mailing_email'] = email
        mass_mailing_list = request.env['mailing.list'].sudo().browse(list_id)
        return {'toast_content': mass_mailing_list.toast_content}

    @route(['/website_mass_mailing/get_content'], type='json', website=True, auth="public")
    def get_mass_mailing_content(self, newsletter_id, **post):
        PopupModel = request.env['website.mass_mailing.popup'].sudo()
        data = self.is_subscriber(newsletter_id, **post)
        domain = expression.AND([request.website.website_domain(), [('mailing_list_id', '=', newsletter_id)]])
        mass_mailing_popup = PopupModel.search(domain, limit=1)
        if mass_mailing_popup:
            data['popup_content'] = mass_mailing_popup.popup_content
        else:
            data.update(PopupModel.default_get(['popup_content']))
        return data

    @route(['/website_mass_mailing/set_content'], type='json', website=True, auth="user")
    def set_mass_mailing_content(self, newsletter_id, content, **post):
        PopupModel = request.env['website.mass_mailing.popup']
        domain = expression.AND([request.website.website_domain(), [('mailing_list_id', '=', newsletter_id)]])
        mass_mailing_popup = PopupModel.search(domain, limit=1)
        if mass_mailing_popup:
            mass_mailing_popup.write({'popup_content': content})
        else:
            PopupModel.create({
                'mailing_list_id': newsletter_id,
                'popup_content': content,
                'website_id': request.website.id,
            })
        return True

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\mailing_list.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MailingList(models.Model):
    _inherit = 'mailing.list'

    def _default_toast_content(self):
        return '<p>Thanks for subscribing!</p>'

    website_popup_ids = fields.One2many('website.mass_mailing.popup', 'mailing_list_id', string="Website Popups")
    toast_content = fields.Html(default=_default_toast_content, translate=True)

```

## File: models\website_mass_mailing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MassMailingPopup(models.Model):
    _name = 'website.mass_mailing.popup'
    _description = "Mailing list popup"

    def _default_popup_content(self):
        return self.env['ir.ui.view'].render_template('website_mass_mailing.s_newsletter_block')

    mailing_list_id = fields.Many2one('mailing.list')
    website_id = fields.Many2one('website')
    popup_content = fields.Html(string="Website Popup Content", default=_default_popup_content, translate=True, sanitize=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mailing_list
from . import website_mass_mailing

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mass_mailing_popup,website.mail.popup,model_website_mass_mailing_popup,mass_mailing.group_mass_mailing_user,1,1,1,1

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#CD7690"/><stop offset="100%" stop-color="#CA5377"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M29.423 69H4c-2 0-4-.146-4-4.078V40.993l22.445-18.95 7.33-3.392H53.22l1.443 1.761.245 15.702L29.423 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#000" d="M48.592 44.35c-.31.781-.682 1.543-1.116 2.285a16.757 16.757 0 0 1-6.14 6.112C38.753 54.25 35.93 55 32.87 55c-3.06 0-5.883-.75-8.467-2.253a16.757 16.757 0 0 1-6.14-6.112C16.754 44.06 16 41.25 16 38.203c0-3.047.754-5.857 2.262-8.43a16.757 16.757 0 0 1 6.14-6.113c1.72-1 3.546-1.667 5.478-2.001v4.28l-.042-.005c-.059 0-.176.015-.352.044a1.64 1.64 0 0 1-.45.022.409.409 0 0 1-.296-.175c-.059-.117-.059-.263 0-.438.014-.058.044-.073.088-.043a3.287 3.287 0 0 1-.242-.208 2.141 2.141 0 0 0-.22-.186c-.673.219-1.362.518-2.064.897a.543.543 0 0 0 .263-.022c.073-.03.169-.077.286-.142.117-.066.19-.106.22-.12.497-.205.805-.256.922-.154l.11-.11c.205.234.351.416.439.548-.103-.059-.322-.066-.659-.022-.293.087-.454.175-.483.262.102.175.139.306.11.394a3.426 3.426 0 0 1-.253-.219 1.741 1.741 0 0 0-.318-.24.88.88 0 0 0-.33-.11c-.234 0-.395.007-.483.022a13.836 13.836 0 0 0-5.162 4.855c.103.102.19.16.264.175.058.015.095.08.11.197a.835.835 0 0 0 .054.24c.022.044.107.023.253-.065.132.117.154.255.066.416.015-.015.337.182.966.59.279.248.432.401.462.46.044.16-.03.291-.22.393a1.12 1.12 0 0 0-.198-.197c-.117-.102-.183-.131-.197-.087-.044.073-.04.208.01.404.052.197.129.288.231.274-.102 0-.172.116-.208.35a5.03 5.03 0 0 0-.055.776c0 .284-.008.456-.022.514l.044.022c-.044.175-.004.426.12.754.125.329.282.47.473.427-.19.044-.044.357.439.94.088.117.146.183.176.197.044.03.131.084.263.164s.242.153.33.219a.934.934 0 0 1 .22.23c.058.072.131.236.219.492.088.255.19.426.308.514-.03.087.04.233.208.437.169.204.245.372.23.503a.107.107 0 0 0-.054.022.107.107 0 0 1-.055.022c.044.102.157.204.34.306.184.102.297.197.34.284.016.044.03.117.045.219a.802.802 0 0 0 .066.24c.029.059.088.073.175.044.03-.291-.146-.743-.527-1.356a17.95 17.95 0 0 1-.373-.634 1.254 1.254 0 0 1-.12-.339 1.662 1.662 0 0 0-.1-.317c.03 0 .073.01.132.033.059.022.12.047.187.076.066.03.12.059.164.088.044.029.06.05.044.065-.044.102-.029.23.044.383.074.153.161.288.264.405a30.038 30.038 0 0 0 .637.7c.088.087.19.23.307.426.118.197.118.295 0 .295.132 0 .279.073.44.22.16.145.285.29.373.436.073.117.132.307.176.57.044.262.08.437.11.524.029.102.091.2.186.295.096.095.187.164.275.208l.351.175.286.153c.073.03.209.106.406.23.092.058.176.107.25.147v.302h1.75c.025.027.05.056.076.087.205.247.359.4.461.459.527.277.93.357 1.208.24-.029.015-.025.07.011.164.037.095.095.208.176.34a21.426 21.426 0 0 0 .318.502c.074.088.205.197.396.328.19.132.322.241.395.329.088-.059.14-.124.154-.197-.044.116.007.262.154.437.146.175.278.248.395.219.205-.044.308-.277.308-.7-.454.219-.813.087-1.077-.394a.425.425 0 0 0-.055-.12 1.39 1.39 0 0 1-.142-.372.309.309 0 0 1 0-.164c.014-.044.05-.065.11-.065.131 0 .204-.026.22-.077.014-.051 0-.142-.045-.273a4.395 4.395 0 0 1-.088-.285c-.014-.116-.095-.262-.241-.437a26.286 26.286 0 0 1-.017-.02h14.246zm-13.285 6.632c1.95-.34 3.714-1.036 5.291-2.088l-3.676-3.675c-.105.08-.168.135-.187.164a1.084 1.084 0 0 0-.132.262 1.32 1.32 0 0 1-.11.241c-.029-.058-.113-.106-.252-.142-.14-.037-.209-.077-.209-.12.03.145.059.4.088.765.03.365.066.642.11.831.102.452.014.802-.264 1.05-.395.364-.608.656-.637.875-.058.32.03.51.264.568 0 .102-.059.252-.176.449-.117.197-.168.353-.154.47 0 .087.015.204.044.35zM55 27.7v10.785c0 .598-.201 1.11-.603 1.535-.402.426-.886.639-1.45.639H34.053c-.565 0-1.049-.213-1.45-.639A2.156 2.156 0 0 1 32 38.486V27.701c.376.443.809.837 1.296 1.181 3.098 2.228 5.224 3.79 6.38 4.687.487.38.883.677 1.186.89.304.212.708.43 1.213.652.505.221.976.332 1.412.332h.026c.436 0 .907-.11 1.412-.332.505-.222.909-.44 1.213-.652.303-.213.7-.51 1.187-.89 1.454-1.114 3.585-2.676 6.392-4.687A7.216 7.216 0 0 0 55 27.701zm0-4.177c0 .715-.21 1.399-.629 2.051a6.297 6.297 0 0 1-1.566 1.67c-3.217 2.364-5.22 3.836-6.006 4.416-.086.063-.268.201-.546.414-.278.213-.509.385-.693.516-.184.131-.406.279-.667.442a3.793 3.793 0 0 1-.738.366 1.94 1.94 0 0 1-.642.123h-.026c-.197 0-.41-.041-.642-.123a3.793 3.793 0 0 1-.738-.366c-.26-.163-.483-.31-.667-.442a26.904 26.904 0 0 1-.693-.516c-.278-.213-.46-.351-.546-.414-.778-.58-1.9-1.406-3.362-2.48a592.34 592.34 0 0 1-2.631-1.935c-.53-.38-1.031-.904-1.502-1.57-.47-.665-.706-1.283-.706-1.853 0-.707.178-1.295.533-1.766.355-.471.862-.707 1.52-.707h18.893c.557 0 1.038.213 1.444.639.407.425.61.937.61 1.535z" opacity=".3"/><path fill="#FFF" d="M48.592 41.35c-.31.781-.682 1.543-1.116 2.285a16.757 16.757 0 0 1-6.14 6.112C38.753 51.25 35.93 52 32.87 52c-3.06 0-5.883-.75-8.467-2.253a16.757 16.757 0 0 1-6.14-6.112C16.754 41.06 16 38.25 16 35.203c0-3.047.754-5.857 2.262-8.43a16.757 16.757 0 0 1 6.14-6.113c1.72-1 3.546-1.667 5.478-2.001v4.28l-.042-.005c-.059 0-.176.015-.352.044a1.64 1.64 0 0 1-.45.022.409.409 0 0 1-.296-.175c-.059-.117-.059-.263 0-.438.014-.058.044-.073.088-.043a3.287 3.287 0 0 1-.242-.208 2.141 2.141 0 0 0-.22-.186c-.673.219-1.362.518-2.064.897a.543.543 0 0 0 .263-.022c.073-.03.169-.077.286-.142.117-.066.19-.106.22-.12.497-.205.805-.256.922-.154l.11-.11c.205.234.351.416.439.548-.103-.059-.322-.066-.659-.022-.293.087-.454.175-.483.262.102.175.139.306.11.394a3.426 3.426 0 0 1-.253-.219 1.741 1.741 0 0 0-.318-.24.88.88 0 0 0-.33-.11c-.234 0-.395.007-.483.022a13.836 13.836 0 0 0-5.162 4.855c.103.102.19.16.264.175.058.015.095.08.11.197a.835.835 0 0 0 .054.24c.022.044.107.023.253-.065.132.117.154.255.066.416.015-.015.337.182.966.59.279.248.432.401.462.46.044.16-.03.291-.22.393a1.12 1.12 0 0 0-.198-.197c-.117-.102-.183-.131-.197-.087-.044.073-.04.208.01.404.052.197.129.288.231.274-.102 0-.172.116-.208.35a5.03 5.03 0 0 0-.055.776c0 .284-.008.456-.022.514l.044.022c-.044.175-.004.426.12.754.125.329.282.47.473.427-.19.044-.044.357.439.94.088.117.146.183.176.197.044.03.131.084.263.164s.242.153.33.219a.934.934 0 0 1 .22.23c.058.072.131.236.219.492.088.255.19.426.308.514-.03.087.04.233.208.437.169.204.245.372.23.503a.107.107 0 0 0-.054.022.107.107 0 0 1-.055.022c.044.102.157.204.34.306.184.102.297.197.34.284.016.044.03.117.045.219a.802.802 0 0 0 .066.24c.029.059.088.073.175.044.03-.291-.146-.743-.527-1.356a17.95 17.95 0 0 1-.373-.634 1.254 1.254 0 0 1-.12-.339 1.662 1.662 0 0 0-.1-.317c.03 0 .073.01.132.033.059.022.12.047.187.076.066.03.12.059.164.088.044.029.06.05.044.065-.044.102-.029.23.044.383.074.153.161.288.264.405a30.038 30.038 0 0 0 .637.7c.088.087.19.23.307.426.118.197.118.295 0 .295.132 0 .279.073.44.22.16.145.285.29.373.436.073.117.132.307.176.57.044.262.08.437.11.524.029.102.091.2.186.295.096.095.187.164.275.208l.351.175.286.153c.073.03.209.106.406.23.092.058.176.107.25.147v.302h1.75c.025.027.05.056.076.087.205.247.359.4.461.459.527.277.93.357 1.208.24-.029.015-.025.07.011.164.037.095.095.208.176.34a21.426 21.426 0 0 0 .318.502c.074.088.205.197.396.328.19.132.322.241.395.329.088-.059.14-.124.154-.197-.044.116.007.262.154.437.146.175.278.248.395.219.205-.044.308-.277.308-.7-.454.219-.813.087-1.077-.394a.425.425 0 0 0-.055-.12 1.39 1.39 0 0 1-.142-.372.309.309 0 0 1 0-.164c.014-.044.05-.065.11-.065.131 0 .204-.026.22-.077.014-.051 0-.142-.045-.273a4.395 4.395 0 0 1-.088-.285c-.014-.116-.095-.262-.241-.437a26.286 26.286 0 0 1-.017-.02h14.246zm-13.285 6.632c1.95-.34 3.714-1.036 5.291-2.088l-3.676-3.675c-.105.08-.168.135-.187.164a1.084 1.084 0 0 0-.132.262 1.32 1.32 0 0 1-.11.241c-.029-.058-.113-.106-.252-.142-.14-.037-.209-.077-.209-.12.03.145.059.4.088.765.03.365.066.642.11.831.102.452.014.802-.264 1.05-.395.364-.608.656-.637.875-.058.32.03.51.264.568 0 .102-.059.252-.176.449-.117.197-.168.353-.154.47 0 .087.015.204.044.35zM55 24.7v10.785c0 .598-.201 1.11-.603 1.535-.402.426-.886.639-1.45.639H34.053c-.565 0-1.049-.213-1.45-.639A2.156 2.156 0 0 1 32 35.486V24.701c.376.443.809.837 1.296 1.181 3.098 2.228 5.224 3.79 6.38 4.687.487.38.883.677 1.186.89.304.212.708.43 1.213.652.505.221.976.332 1.412.332h.026c.436 0 .907-.11 1.412-.332.505-.222.909-.44 1.213-.652.303-.213.7-.51 1.187-.89 1.454-1.114 3.585-2.676 6.392-4.687A7.216 7.216 0 0 0 55 24.701zm0-4.177c0 .715-.21 1.399-.629 2.051a6.297 6.297 0 0 1-1.566 1.67c-3.217 2.364-5.22 3.836-6.006 4.416-.086.063-.268.201-.546.414-.278.213-.509.385-.693.516-.184.131-.406.279-.667.442a3.793 3.793 0 0 1-.738.366 1.94 1.94 0 0 1-.642.123h-.026c-.197 0-.41-.041-.642-.123a3.793 3.793 0 0 1-.738-.366c-.26-.163-.483-.31-.667-.442a26.904 26.904 0 0 1-.693-.516c-.278-.213-.46-.351-.546-.414-.778-.58-1.9-1.406-3.362-2.48a592.34 592.34 0 0 1-2.631-1.935c-.53-.38-1.031-.904-1.502-1.57-.47-.665-.706-1.283-.706-1.853 0-.707.178-1.295.533-1.766.355-.471.862-.707 1.52-.707h18.893c.557 0 1.038.213 1.444.639.407.425.61.937.61 1.535z"/></g></g></svg>
```

## File: static\src\js\website_mass_mailing.editor.js

```javascript
odoo.define('website_mass_mailing.editor', function (require) {
'use strict';

var core = require('web.core');
var rpc = require('web.rpc');
var WysiwygMultizone = require('web_editor.wysiwyg.multizone');
var WysiwygTranslate = require('web_editor.wysiwyg.multizone.translate');
var options = require('web_editor.snippets.options');
var wUtils = require('website.utils');
var _t = core._t;


options.registry.mailing_list_subscribe = options.Class.extend({
    popup_template_id: "editor_new_mailing_list_subscribe_button",
    popup_title: _t("Add a Newsletter Subscribe Button"),

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Allows to select mailing list.
     *
     * @see this.selectClass for parameters
     */
    select_mailing_list: function (previewMode, value) {
        var self = this;
        var def = wUtils.prompt({
            'id': this.popup_template_id,
            'window_title': this.popup_title,
            'select': _t("Newsletter"),
            'init': function (field, dialog) {
                return rpc.query({
                    model: 'mailing.list',
                    method: 'name_search',
                    args: ['', [['is_public', '=', true]]],
                    context: self.options.recordInfo.context,
                }).then(function (data) {
                    $(dialog).find('.btn-primary').prop('disabled', !data.length);
                    return data;
                });
            },
        });
        def.then(function (result) {
            self.$target.attr("data-list-id", result.val);
        });
        return def;
    },
    /**
     * @override
     */
    onBuilt: function () {
        var self = this;
        this._super();
        this.select_mailing_list('click').guardedCatch(function () {
            self.getParent()._onRemoveClick($.Event( "click" ));
        });
    },
});

options.registry.newsletter_popup = options.registry.mailing_list_subscribe.extend({
    popup_template_id: "editor_new_mailing_list_subscribe_popup",
    popup_title: _t("Add a Newsletter Subscribe Popup"),

    /**
     * @override
     */
    start: function () {
        var self = this;
        this.$target.on('click.newsletter_popup_option', '.o_edit_popup', function (ev) {
            // So that the snippet is not enabled again by the editor
            ev.stopPropagation();
            self.$target.data('quick-open', true);
            self._refreshPublicWidgets();
        });
        this.$target.on('shown.bs.modal.newsletter_popup_option hide.bs.modal.newsletter_popup_option', function () {
            self.$target.closest('.o_editable').trigger('content_changed');
            self.trigger_up('deactivate_snippet');
        });
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    cleanForSave: function () {
        var self = this;
        var content = this.$target.data('content');
        if (content) {
            this.trigger_up('get_clean_html', {
                $layout: $('<div/>').html(content),
                callback: function (html) {
                    self.$target.data('content', html);
                },
            });
        }
        this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    destroy: function () {
        this.$target.off('.newsletter_popup_option');
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    select_mailing_list: function () {
        var self = this;
        return this._super.apply(this, arguments).then(function () {
            self.$target.data('quick-open', true);
            self.$target.removeData('content');
            self._refreshPublicWidgets();
        });
    },
});

WysiwygMultizone.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _saveElement: function (outerHTML, recordInfo, editable) {
        var self = this;
        var defs = [this._super.apply(this, arguments)];
        var $popups = $(editable).find('.o_newsletter_popup');
        _.each($popups, function (popup) {
            var $popup = $(popup);
            var content = $popup.data('content');
            if (content) {
                defs.push(self._rpc({
                    route: '/website_mass_mailing/set_content',
                    params: {
                        'newsletter_id': parseInt($popup.attr('data-list-id')),
                        'content': content,
                    },
                }));
            }
        });
        return Promise.all(defs);
    },
});

WysiwygTranslate.include({
    /**
     * @override
     */
    start: function () {
        this.$target.on('click.newsletter_popup_option', '.o_edit_popup', function (ev) {
            alert(_t('Website popups can only be translated through mailing list configuration in the Email Marketing app.'));
        });
        this._super.apply(this, arguments);
    },
});

});

```

## File: static\src\js\website_mass_mailing.js

```javascript
odoo.define('mass_mailing.website_integration', function (require) {
"use strict";

var config = require('web.config');
var core = require('web.core');
var Dialog = require('web.Dialog');
var utils = require('web.utils');
var publicWidget = require('web.public.widget');

var _t = core._t;

publicWidget.registry.subscribe = publicWidget.Widget.extend({
    selector: ".js_subscribe",
    disabledInEditableMode: false,
    read_events: {
        'click .js_subscribe_btn': '_onSubscribeClick',
    },

    /**
     * @override
     */
    start: function () {
        var self = this;
        var def = this._super.apply(this, arguments);
        this.$popup = this.$target.closest('.o_newsletter_modal');
        if (this.$popup.length) {
            // No need to check whether the user subscribed or not if the input
            // is in a popup as the popup won't open if he did subscribe.
            return def;
        }

        var always = function (data) {
            var isSubscriber = data.is_subscriber;
            self.$('.js_subscribe_btn').prop('disabled', isSubscriber);
            var $input = self.$('input.js_subscribe_email');
            // Handle removed input (eg. inside sanitized Html field)
            $input = $input.length ? $input : $(
                '<input type="email" name="email" class="js_subscribe_email form-control"/>'
            ).prependTo(self.$el);
            $input.val(data.email || "").prop('disabled', isSubscriber);
            // Compat: remove d-none for DBs that have the button saved with it.
            self.$target.removeClass('d-none');
            self.$('.js_subscribe_btn').toggleClass('d-none', !!isSubscriber);
            self.$('.js_subscribed_btn').toggleClass('d-none', !isSubscriber);
        };
        return Promise.all([def, this._rpc({
            route: '/website_mass_mailing/is_subscriber',
            params: {
                'list_id': this.$target.data('list-id'),
            },
        }).then(always).guardedCatch(always)]);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onSubscribeClick: function () {
        var self = this;
        var $email = this.$(".js_subscribe_email:visible");

        if ($email.length && !$email.val().match(/.+@.+/)) {
            this.$target.addClass('o_has_error').find('.form-control').addClass('is-invalid');
            return false;
        }
        this.$target.removeClass('o_has_error').find('.form-control').removeClass('is-invalid');
        this._rpc({
            route: '/website_mass_mailing/subscribe',
            params: {
                'list_id': this.$target.data('list-id'),
                'email': $email.length ? $email.val() : false,
            },
        }).then(function (result) {
            self.$(".js_subscribe_btn").addClass('d-none');
            self.$(".js_subscribed_btn").removeClass('d-none');
            self.$('input.js_subscribe_email').prop('disabled', !!result);
            if (self.$popup.length) {
                self.$popup.modal('hide');
            }
            self.displayNotification({
                type: 'success',
                title: _t("Success"),
                message: result.toast_content,
                sticky: true,
            });
        });
    },
});

publicWidget.registry.newsletter_popup = publicWidget.Widget.extend({
    selector: ".o_newsletter_popup",
    disabledInEditableMode: false,

    /**
     * @override
     */
    start: function () {
        var self = this;
        var defs = [this._super.apply(this, arguments)];
        this.websiteID = this._getContext().website_id;
        this.listID = parseInt(this.$target.attr('data-list-id'));
        if (!this.listID || (utils.get_cookie(_.str.sprintf("newsletter-popup-%s-%s", this.listID, this.websiteID)) && !self.editableMode)) {
            return Promise.all(defs);
        }
        if (this.$target.data('content') && this.editableMode) {
            // To avoid losing user changes.
            this._dialogInit(this.$target.data('content'));
            this.$target.removeData('quick-open');
            this.massMailingPopup.open();
        } else {
            defs.push(this._rpc({
                route: '/website_mass_mailing/get_content',
                params: {
                    newsletter_id: self.listID,
                },
            }).then(function (data) {
                self._dialogInit(data.popup_content, data.email || '');
                if (!self.editableMode && !data.is_subscriber) {
                    if (config.device.isMobile) {
                        setTimeout(function () {
                            self._showBanner();
                        }, 5000);
                    } else {
                        $(document).on('mouseleave.open_popup_event', self._showBanner.bind(self));
                    }
                } else {
                    $(document).off('mouseleave.open_popup_event');
                }
                // show popup after choosing a newsletter
                if (self.$target.data('quick-open')) {
                    self.massMailingPopup.open();
                    self.$target.removeData('quick-open');
                }
            }));
        }

        return Promise.all(defs);
    },
    /**
     * @override
     */
    destroy: function () {
        if (this.massMailingPopup) {
            this.massMailingPopup.close();
        }
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @param {string} content
     * @private
     */
    _dialogInit: function (content, email) {
        var self = this;
        this.massMailingPopup = new Dialog(this, {
            technical: false,
            $content: $('<div/>').html(content),
            $parentNode: this.$target,
            backdrop: !this.editableMode,
            dialogClass: 'p-0' + (this.editableMode ? ' oe_structure oe_empty' : ''),
            renderFooter: false,
            size: 'medium',
        });
        this.massMailingPopup.opened().then(function () {
            var $modal = self.massMailingPopup.$modal;
            $modal.find('header button.close').on('mouseup', function (ev) {
                ev.stopPropagation();
            });
            $modal.addClass('o_newsletter_modal');
            $modal.find('.oe_structure').attr('data-editor-message', _t('DRAG BUILDING BLOCKS HERE'));
            $modal.find('.modal-dialog').addClass('modal-dialog-centered');
            $modal.find('.js_subscribe').data('list-id', self.listID)
                  .find('input.js_subscribe_email').val(email);
            self.trigger_up('widgets_start_request', {
                editableMode: self.editableMode,
                $target: $modal,
            });
        });
        this.massMailingPopup.on('closed', this, function () {
            var $modal = self.massMailingPopup.$modal;
            if ($modal) { // The dialog might have never been opened
                self.$el.data('content', $modal.find('.modal-body').html());
            }
        });
    },
    /**
     * @private
     */
    _showBanner: function () {
        this.massMailingPopup.open();
        utils.set_cookie(_.str.sprintf("newsletter-popup-%s-%s", this.listID, this.websiteID), true);
        $(document).off('mouseleave.open_popup_event');
    },
});
});

```

## File: static\src\xml\website_mass_mailing.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
<t t-name="website_mass_mailing.edition.wrapper">
    <div class="modal fade show d-block o_newsletter_modal">
        <div role="dialog" class="modal-dialog modal-dialog-centered">
            <div class="modal-content">
                <header class="modal-header">
                    <button type="button" class="close" aria-label="Close" tabindex="-1">×</button>
                </header>
                <div id="wrapper" class="modal-body p-0 oe_structure oe_empty"></div>
            </div>
        </div>
    </div>
</t>
</templates>

```

## File: views\mailing_list_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_list_view_form" model="ir.ui.view">
        <field name="name">mailing.list.view.form.inherit.website</field>
        <field name="model">mailing.list</field>
        <field name="inherit_id" ref="mass_mailing.mailing_list_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//sheet" position="inside">
                <notebook>
                    <page string="Website popups">
                        <field name="website_popup_ids">
                            <tree>
                                <field name="website_id"/>
                            </tree>
                        </field>
                    </page>
                </notebook>
            </xpath>
            <xpath expr="//div[hasclass('oe_title')]" position="after">
                <group>
                    <field name="toast_content"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\snippets_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="iframe_css_assets_edit" name="CSS assets for wysiwyg iframe content for popup">
    <t t-call-assets="web.assets_common" t-js="false"/>
    <t t-call-assets="web.assets_frontend" t-js="false"/>
    <t t-call-assets="web_editor.assets_wysiwyg" t-js="false"/>
    <t t-call-assets="website.assets_editor" t-js="false"/>
</template>

<template id="remove_external_snippets" inherit_id="website.external_snippets">
    <xpath expr="//t[@id='newsletter_snippet']" position="replace"/>
    <xpath expr="//t[@id='newsletter_popup_snippet']" position="replace"/>
    <xpath expr="//t[@id='newsletter_block_snippet']" position="replace"/>
</template>

<template id="snippets" inherit_id="website.snippets">
    <xpath expr="//div[@id='snippet_feature']//t[@t-snippet][last()]" position="after">
        <t t-snippet="website_mass_mailing.s_newsletter_subscribe_popup" t-thumbnail="/website/static/src/img/snippets_thumbs/newsletter_subscribe_popup.png"/>
        <t t-snippet="website_mass_mailing.s_newsletter_block" t-thumbnail="/website/static/src/img/snippets_thumbs/s_newsletter_block.png"/>
    </xpath>

    <xpath expr="//div[@id='snippet_content']//t[@t-snippet][last()]" position="after">
        <t t-snippet="website_mass_mailing.s_newsletter_subscribe_form" t-thumbnail="/website/static/src/img/snippets_thumbs/s_newsletter_subscribe_form.png"/>
    </xpath>
</template>

<template id="s_newsletter_subscribe_form" name="Newsletter">
    <div class="input-group js_subscribe" data-list-id="0">
        <input type="email" name="email" class="js_subscribe_email form-control" placeholder="your email..."/>
        <span class="input-group-append">
            <a role="button" href="#" class="btn btn-primary js_subscribe_btn">Subscribe</a>
            <a role="button" href="#" class="btn btn-success js_subscribed_btn d-none" disabled="disabled">Thanks</a>
        </span>
    </div>
</template>

<template id="s_newsletter_block" name="Newsletter block">
    <section class="s_newsletter_block pt64 pb64 oe_img_bg oe_custom_bg" style="background-image: url('/web/image/website.library_image_17');">
        <div class="container">
            <div class="row">
                <div class="col-lg-7 offset-lg-5 pt32 pb32 bg-white-75">
                    <div class="text-center">
                        <h1 class="text-uppercase font-weight-bold">Always First.</h1>
                        <p>Be the first to find out all the latest news, products and trends.</p>
                        <t t-call="website_mass_mailing.s_newsletter_subscribe_form"/>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_newsletter_subscribe_popup" name="Newsletter Popup">
    <div class="o_newsletter_popup" data-list-id="0">
        <div class="alert alert-warning css_non_editable_mode_hidden o_not_editable" role="alert">
            <strong>Newsletter Popup!</strong> The newsletter popup is active on this page. <a class="o_edit_popup" data-toggle="modal" href="#">Edit Popup</a>
        </div>
    </div>
</template>

<template id="newsletter_subscribe_options" name="Newsletter Subscribe Options" inherit_id="website.snippet_options">
    <xpath expr="//*[@id='so_snippet_addition']" position="attributes">
        <attribute name="data-selector" add=".o_newsletter_popup" separator=","/>
    </xpath>
    <xpath expr="//*[@id='row_valign_snippet_option']" position="attributes">
        <attribute name="data-selector" separator="," add=".s_newsletter_block"/>
    </xpath>
    <xpath expr="//div" position="after">
        <t t-set="selector" t-value="'.js_subscribe'"/>
        <div data-js="mailing_list_subscribe"
            t-att-data-selector="selector"
            t-attf-data-exclude=".o_newsletter_modal #{selector}">
            <we-button data-select_mailing_list="" data-no-preview="true">Change Newsletter</we-button>
        </div>
        <div t-att-data-selector="selector" data-drop-near="p, h1, h2, h3, blockquote, .card"/>
        <div data-js="newsletter_popup"
            data-selector=".o_newsletter_popup">
            <we-button data-select_mailing_list="" data-no-preview="true">Change Newsletter</we-button>
        </div>
    </xpath>
</template>

<!-- Extend default mass_mailing snippets with website feature -->
<template id="social_links">
    <t t-if="not website">
        <t t-set="website" t-value="request.env['website'].search([], limit=1)"/>
    </t>
    <t t-if="website.social_facebook">
        <a t-att-href="website.social_facebook" aria-label="Facebook" title="Facebook">
          <span class="fa fa-facebook"></span>
        </a>
    </t>
    <t t-if="website.social_linkedin">
        <a t-att-href="website.social_linkedin" style="margin-left:10px" aria-label="LinkedIn" title="LinkedIn">
            <span class="fa fa-linkedin"></span>
        </a>
    </t>
    <t t-if="website.social_twitter">
        <a t-att-href="website.social_twitter" style="margin-left:10px" aria-label="Twitter" title="Twitter">
            <span class="fa fa-twitter"></span>
        </a>
    </t>
    <t t-if="website.social_instagram">
        <a t-att-href="website.social_instagram" style="margin-left:10px" aria-label="Instagram" title="Instagram">
            <span class="fa fa-instagram"></span>
        </a>
    </t>
</template>

<template id="s_mail_block_header_social" inherit_id="mass_mailing.s_mail_block_header_social">
    <xpath expr="//td[hasclass('o_mail_logo_container')]" position="after">
        <td width="30%" class="text-right o_mail_no_resize">
            <div class="o_mail_header_social">
                <t t-call="website_mass_mailing.social_links"/>
            </div>
        </td>
    </xpath>
</template>

<template id="s_mail_block_header_text_social" inherit_id="mass_mailing.s_mail_block_header_text_social">
    <xpath expr="//table//td" position="after">
        <td width="30%" class="text-right o_mail_no_resize">
            <div class="o_mail_header_social">
                <t t-call="website_mass_mailing.social_links"/>
            </div>
        </td>
    </xpath>
</template>

<template id="s_mail_block_footer_social" inherit_id="mass_mailing.s_mail_block_footer_social">
    <xpath expr="//td[hasclass('o_mail_footer_links')]" position="inside">
        <t> | <a role="button" href="/contactus" class="btn btn-link">Contact</a></t>
    </xpath>
    <xpath expr="//tr" position="before">
        <tr>
            <td class="o_mail_footer_social">
                <t t-call="website_mass_mailing.social_links"/>
            </td>
        </tr>
    </xpath>
</template>

<template id="s_mail_block_footer_social_left" inherit_id="mass_mailing.s_mail_block_footer_social_left">
    <xpath expr="//div[hasclass('o_mail_footer_links')]" position="inside">
        <t> | <a role="button" href="/contactus" class="btn btn-link">Contact</a></t>
    </xpath>
    <xpath expr="//td" position="after">
        <td class="o_mail_footer_social">
            <t t-call="website_mass_mailing.social_links"/>
        </td>
    </xpath>
</template>

</odoo>

```

## File: views\website_mass_mailing_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="assets_frontend" inherit_id="website.assets_frontend">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website_mass_mailing/static/src/scss/website_mass_mailing_popup.scss"/>
    </xpath>
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_mass_mailing/static/src/js/website_mass_mailing.js"/>
    </xpath>
</template>

<template id="assets_editor" inherit_id="website.assets_wysiwyg">
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_mass_mailing/static/src/js/website_mass_mailing.editor.js"/>
    </xpath>
</template>

</odoo>

```

## File: views\website_mass_mailing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_mail_mass_mailing_popup_form" model="ir.ui.view">
        <field name="name">website.mass_mailing.popup.form</field>
        <field name="model">website.mass_mailing.popup</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="mailing_list_id"/>
                        <field name="website_id" groups="website.group_multi_website"/>
                    </group>
                    <label for="popup_content"/>
                    <field name="popup_content" widget="html" options="{
                            'snippets': 'website.snippets',
                            'cssEdit': 'website_mass_mailing.iframe_css_assets_edit',
                            'wrapper': 'website_mass_mailing.edition.wrapper',
                        }"/>
                </sheet>
            </form>
        </field>
    </record>
</odoo>

```

