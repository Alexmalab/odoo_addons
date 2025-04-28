# Odoo Module: website_mail_group

Category: Uncategorized

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
    'name': "Website Mail Group",
    'summary': "Add a website snippet for the mail groups.",
    'version': '1.0',
    'depends': ['mail_group', 'website'],
    'auto_install': True,
    'data': [
        'views/snippets/s_group.xml',
        'views/snippets/snippets.xml',
        'views/mail_group_views.xml',
        'views/website_mail_group_menus.xml',
    ],
    'assets': {
        'website.assets_wysiwyg': [
            'website_mail_group/static/src/snippets/s_group/options.js',
        ],
        'web.assets_frontend': [
            'website_mail_group/static/src/snippets/s_group/000.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.exceptions import AccessError
from odoo.http import request


class WebsiteMailGroup(http.Controller):
    @http.route('/group/is_member', type='json', auth='public', website=True)
    def group_is_member(self, group_id=0, email=None, **kw):
        """Return the email of the member if found, otherwise None."""
        group = request.env['mail.group'].browse(int(group_id)).exists()
        if not group:
            return

        token = kw.get('token')

        if token and token != group._generate_group_access_token():
            return

        if token:
            group = group.sudo()

        try:
            group.check_access_rights('read')
            group.check_access_rule('read')
        except AccessError:
            return

        if not request.env.user._is_public():
            email = request.env.user.email_normalized
            partner_id = request.env.user.partner_id.id
        else:
            partner_id = None

        member = group.sudo()._find_member(email, partner_id)

        return {
            'is_member': bool(member),
            'email': member.email if member else email,
        }

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\mail_group.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.addons.http_routing.models.ir_http import slug


class MailGroup(models.Model):
    _name = 'mail.group'
    _inherit = 'mail.group'

    def action_go_to_website(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'target': 'self',
            'url': '/groups/%s' % slug(self),
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_group

```

## File: static\src\snippets\s_group\000.js

```javascript
odoo.define('website_mail_group.mail_group', function (require) {
'use strict';

const core = require('web.core');
const publicWidget = require('web.public.widget');
const _t = core._t;
const MailGroup = require('mail_group.mail_group');

MailGroup.include({
    start: async function () {
        await this._super(...arguments);

        // Can not be done in the template of the snippets
        // Because it's rendered only once when the admin add the snippets
        // for the first time, we make a RPC call to setup the widget properly
        const email = (new URL(document.location.href)).searchParams.get('email');
        const response = await this._rpc({
            route: '/group/is_member',
            params: {
                'group_id': this.mailgroupId,
                'email': email,
                'token': this.token,
            },
        });

        if (!response) {
            // We do not access to the mail group, just remove the widget
            this.$el.empty();
            return;
        }

        this.$el.removeClass('d-none');

        const userEmail = response.email;
        this.isMember = response.is_member;

        if (userEmail && userEmail.length) {
            const emailInput = this.$el.find('.o_mg_subscribe_email');
            emailInput.val(userEmail);
            emailInput.attr('readonly', 1);
        }

        if (this.isMember) {
            this.$target.find('.o_mg_subscribe_btn').text(_t('Unsubscribe')).removeClass('btn-primary').addClass('btn-outline-primary');
        }

        this.$el.data('isMember', this.isMember);
    },
    /**
     * @override
     */
    destroy: function () {
        this.el.classList.add('d-none');
        this._super(...arguments);
    },
});

// TODO should probably have a better way to handle this, maybe the invisible
// block system could be extended to handle this kind of things. Here we only
// do the same as the non-edit mode public widget: showing and hiding the widget
// but without the rest. Arguably could just enable the whole widget in edit
// mode but not stable-friendly.
publicWidget.registry.MailGroupEditMode = publicWidget.Widget.extend({
    selector: MailGroup.prototype.selector,
    disabledInEditableMode: false,

    /**
     * @override
     */
    start: function () {
        if (this.editableMode) {
            this.el.classList.remove('d-none');
        }
        return this._super(...arguments);
    },
    /**
     * @override
     */
    destroy: function () {
        if (this.editableMode) {
            this.el.classList.add('d-none');
        }
        this._super(...arguments);
    },
});

});

```

## File: static\src\snippets\s_group\options.js

```javascript
odoo.define('website_mail_group.s_group_options', function (require) {
'use strict';

const core = require('web.core');
const options = require('web_editor.snippets.options');
const wUtils = require('website.utils');
const _t = core._t;

options.registry.Group = options.Class.extend({
    /**
     * @override
     */
    async start() {
        await this._super(...arguments);
        this.mailGroups = await this._getMailGroups();
    },
    /**
     * If we have already created groups => select the first one
     * else => modal prompt (create a new group)
     *
     * @override
     */
    onBuilt() {
        if (this.mailGroups.length) {
            this.$target[0].dataset.id = this.mailGroups[0][0];
        } else {
            const widget = this._requestUserValueWidgets('create_mail_group_opt')[0];
            widget.$el.click();
        }
    },

    cleanForSave: function () {
        // TODO: this should probably be done by the public widget, not the
        // option code, not important enough to try and fix in stable though.
        const emailInput = this.$target.find('.o_mg_subscribe_email');
        emailInput.val('');
        emailInput.removeAttr('readonly');
        this.$target.find('.o_mg_subscribe_btn').text(_t('Subscribe'));
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Creates a new mail.group through a modal prompt.
     *
     * @see this.selectClass for parameters
     */
    createGroup: async function (previewMode, widgetValue, params) {
        const result = await wUtils.prompt({
            id: "editor_new_mail_group_subscribe",
            window_title: _t("New Mail Group"),
            input: _t("Name"),
        });

        const name = result.val;
        if (!name) {
            return;
        }

        const groupId = await this._rpc({
            model: 'mail.group',
            method: 'create',
            args: [{
                name: name,
            }],
        });

        this.$target.attr("data-id", groupId);
        return this._rerenderXML();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async _renderCustomXML(uiFragment) {
        const groups = await this._getMailGroups();
        const menuEl = uiFragment.querySelector('.select_discussion_list');
        for (const group of groups) {
            const el = document.createElement('we-button');
            el.dataset.selectDataAttribute = group[0];
            el.textContent = group[1];
            menuEl.appendChild(el);
        }
    },
    /**
     * @private
     * @return {Promise}
     */
    _getMailGroups() {
        return this._rpc({
            model: 'mail.group',
            method: 'name_search',
            args: [''],
        });
    },
});
});

```

## File: views\mail_group_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="mail_group_view_form" model="ir.ui.view">
        <field name="name">mail.group.view.form</field>
        <field name="model">mail.group</field>
        <field name="inherit_id" ref="mail_group.mail_group_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button type="object" class="oe_stat_button" icon="fa-globe" name="action_go_to_website">
                    <div class="o_form_field o_stat_info">
                        <span class="o_stat_text">Go to <br/>Website</span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\website_mail_group_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem name="Mailing Lists"
        id="mail_group_menu_website_root"
        sequence="200"
        parent="website.menu_website_global_configuration"/>
    <menuitem id="mail_group_menu_website"
        name="Mailing Lists"
        action="mail_group.mail_group_action"
        parent="website_mail_group.mail_group_menu_website_root"
        sequence="50"/>
    <menuitem id="mail_group_moderation_menu_website"
        name="Moderation Rules"
        action="mail_group.mail_group_moderation_action"
        parent="website_mail_group.mail_group_menu_website_root"
        groups="mail_group.group_mail_group_manager"
        sequence="51"/>
</odoo>

```

## File: views\snippets\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="remove_external_snippets" inherit_id="website.external_snippets">
        <xpath expr="//t[@t-install='website_mail_group']" position="replace"/>
    </template>

    <template id="snippets" inherit_id="website.snippets" name="Snippet Subscribe">
        <xpath expr="//t[@id='mail_group_hook']" position="replace">
            <t t-snippet="website_mail_group.s_group" string="Discussion Group" t-thumbnail="/website/static/src/img/snippets_thumbs/s_group.svg"/>
        </xpath>
    </template>
</odoo>

```

## File: views\snippets\s_group.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="s_group" name="Discussion Group">
        <div class="s_group o_mail_group"
             data-id="0" data-object="mail.group" data-follow="off">
            <div class="input-group o_mg_subscribe_form">
                <input class="o_mg_subscribe_email form-control" type="email" name="email" placeholder="your email..."/>
                <button href="#" class="btn btn-primary o_mg_subscribe_btn">Subscribe</button>
            </div>
        </div>
    </template>
    <template id="s_group_options" inherit_id="website.snippet_options">
        <xpath expr="." position="inside">
            <div data-js='Group'
                 data-selector=".s_group"
                 data-drop-near="p, h1, h2, h3, blockquote, .card">
                <we-row>
                    <we-select class="select_discussion_list" data-attribute-name="id" data-no-preview="true">
                        <!-- 'we-button' added programmatically with DB data -->
                    </we-select>
                    <we-button class="fa fa-fw fa-plus" title="Create a public discussion group in your backend"
                               data-create-group="" data-no-preview="true" data-name="create_mail_group_opt"/>
                </we-row>
            </div>
        </xpath>
    </template>
    <record id="website_mail_group.s_group_000_js" model="ir.asset">
        <field name="name">Group 000 JS</field>
        <field name="bundle">web.assets_frontend</field>
        <field name="path">website_mail_group/static/src/snippets/s_group/000.js</field>
    </record>
</odoo>

```

