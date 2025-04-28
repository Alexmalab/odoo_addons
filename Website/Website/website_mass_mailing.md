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
    'depends': ['website', 'mass_mailing', 'google_recaptcha'],
    'data': [
        'security/ir.model.access.csv',
        'data/ir_model_data.xml',
        'views/snippets/s_popup.xml',
        'views/snippets_templates.xml',
    ],
    'auto_install': ['website', 'mass_mailing'],
    'assets': {
        'web.assets_frontend': [
            'website_mass_mailing/static/src/scss/website_mass_mailing_popup.scss',
            'website_mass_mailing/static/src/js/website_mass_mailing.js',
            'website_mass_mailing/static/src/xml/*.xml',
        ],
        'website.assets_wysiwyg': [
            'website_mass_mailing/static/src/js/website_mass_mailing.editor.js',
            'website_mass_mailing/static/src/js/mass_mailing_form_editor.js',
            'website_mass_mailing/static/src/scss/website_mass_mailing_edit_mode.scss',
            'website_mass_mailing/static/src/snippets/s_popup/options.js',
        ],
        'web.assets_tests': [
            'website_mass_mailing/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import tools, _
from odoo.http import route, request
from odoo.addons.mass_mailing.controllers import main


class MassMailController(main.MassMailController):

    @route('/website_mass_mailing/is_subscriber', type='json', website=True, auth='public')
    def is_subscriber(self, list_id, subscription_type, **post):
        value = self._get_value(subscription_type)
        fname = self._get_fname(subscription_type)
        is_subscriber = False
        if value and fname:
            contacts_count = request.env['mailing.subscription'].sudo().search_count(
                [('list_id', 'in', [int(list_id)]), (f'contact_id.{fname}', '=', value), ('opt_out', '=', False)])
            is_subscriber = contacts_count > 0

        return {'is_subscriber': is_subscriber, 'value': value}

    def _get_value(self, subscription_type):
        value = None
        if subscription_type == 'email':
            if not request.env.user._is_public():
                value = request.env.user.email
            elif request.session.get('mass_mailing_email'):
                value = request.session['mass_mailing_email']
        return value

    def _get_fname(self, subscription_type):
        return 'email' if subscription_type == 'email' else ''

    @route('/website_mass_mailing/subscribe', type='json', website=True, auth='public')
    def subscribe(self, list_id, value, subscription_type, **post):
        if not request.env['ir.http']._verify_request_recaptcha_token('website_mass_mailing_subscribe'):
            return {
                'toast_type': 'danger',
                'toast_content': _("Suspicious activity detected by Google reCaptcha."),
            }

        fname = self._get_fname(subscription_type)
        self.subscribe_to_newsletter(subscription_type, value, list_id, fname)
        return {
            'toast_type': 'success',
            'toast_content': _("Thanks for subscribing!"),
        }

    @staticmethod
    def subscribe_to_newsletter(subscription_type, value, list_id, fname, address_name=None):
        ContactSubscription = request.env['mailing.subscription'].sudo()
        Contacts = request.env['mailing.contact'].sudo()
        if subscription_type == 'email':
            name, value = tools.parse_contact_from_email(value)
            if not name:
                name = address_name
        elif subscription_type == 'mobile':
            name = value

        subscription = ContactSubscription.search(
            [('list_id', '=', int(list_id)), (f'contact_id.{fname}', '=', value)], limit=1)
        if not subscription:
            # inline add_to_list as we've already called half of it
            contact_id = Contacts.search([(fname, '=', value)], limit=1)
            if not contact_id:
                contact_id = Contacts.create({'name': name, fname: value})
            ContactSubscription.create({'contact_id': contact_id.id, 'list_id': int(list_id)})
        elif subscription.opt_out:
            subscription.opt_out = False
        # add email to session
        request.session[f'mass_mailing_{fname}'] = value

```

## File: controllers\website_form.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import _
from odoo.http import request
from odoo.addons.website.controllers.form import WebsiteForm


class WebsiteNewsletterForm(WebsiteForm):

    def _handle_website_form(self, model_name, **kwargs):
        if model_name == 'mailing.contact':
            list_ids = kwargs.get('list_ids')
            if not list_ids:
                return json.dumps({'error': _('Mailing List(s) not found!')})
            list_ids = [int(x) for x in list_ids.split(',')]
            private_list_ids = request.env['mailing.list'].sudo().search([
                ('id', 'in', list_ids), ('is_public', '=', False)])
            if private_list_ids:
                return json.dumps({
                    'error': _('You cannot subscribe to the following list anymore : %s',
                               ', '.join(private_list_ids.mapped('name')))
                })
        return super()._handle_website_form(model_name, **kwargs)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main
from . import website_form

```

## File: data\ir_model_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mass_mailing.model_mailing_contact" model="ir.model">
            <field name="website_form_key">create_mailing_contact</field>
            <field name="website_form_access">True</field>
            <field name="website_form_label">Subscribe to Newsletter</field>
        </record>

        <function model="ir.model.fields" name="formbuilder_whitelist">
            <value>mailing.contact</value>
            <value eval="[
                'name',
                'first_name',
                'last_name',
                'company_name',
                'title_id',
                'email',
                'list_ids',
                'country_id',
                'tag_ids',
            ]"/>
        </function>
    </data>
</odoo>

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResCompany(models.Model):
    _inherit = "res.company"

    def _get_social_media_links(self):
        social_media_links = super()._get_social_media_links()
        website_id = self.env['website'].get_current_website()
        social_media_links.update({
            'social_facebook': website_id.social_facebook or social_media_links.get('social_facebook'),
            'social_linkedin': website_id.social_linkedin or social_media_links.get('social_linkedin'),
            'social_twitter': website_id.social_twitter or social_media_links.get('social_twitter'),
            'social_instagram': website_id.social_instagram or social_media_links.get('social_instagram'),
            'social_tiktok': website_id.social_tiktok or social_media_links.get('social_tiktok'),
        })
        return social_media_links

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_company

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_mailing_list_website_designer,access_mailing_list_website_designer,mass_mailing.model_mailing_list,website.group_website_designer,1,0,0,0

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M0 42V8l25 17L0 42Z" fill="#088BF5"/><path d="M25 25 0 8h50L25 25Z" fill="#2EBCFA"/><path d="M50 8 0 42h46a4 4 0 0 0 4-4V8Z" fill="#144496"/></svg>

```

## File: static\src\img\snippets_thumbs\s_newsletter_block.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="0%" x2="100%" y1="44.579%" y2="55.421%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <rect id="path-2" width="22" height="2" x="33" y="7"/>
    <filter id="filter-3" width="104.5%" height="200%" x="-2.3%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-4" d="M13 8v10a1 1 0 0 0 1 1h14a1 1 0 0 0 1-1V8a1 1 0 0 0-1-1H14a1 1 0 0 0-1 1zm7.445 3.63L15 8h12l-5.445 3.63a1 1 0 0 1-1.11 0zM14 18V8.5l6.428 4.485a1 1 0 0 0 1.144 0L28 8.5V18H14z"/>
    <filter id="filter-5" width="106.2%" height="116.7%" x="-3.1%" y="-4.2%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <rect id="path-6" width="13" height="1" x="33" y="12"/>
    <filter id="filter-7" width="107.7%" height="300%" x="-3.8%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
    <rect id="path-8" width="10" height="1" x="33" y="15"/>
    <filter id="filter-9" width="110%" height="300%" x="-5%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
    <path id="path-10" d="M72 23.571V22.43a.55.55 0 0 0-.17-.402.55.55 0 0 0-.401-.17h-2.286v-2.286a.55.55 0 0 0-.17-.401.55.55 0 0 0-.402-.17H67.43a.55.55 0 0 0-.402.17.55.55 0 0 0-.17.401v2.286h-2.286a.55.55 0 0 0-.401.17.55.55 0 0 0-.17.402v1.142a.55.55 0 0 0 .17.402.55.55 0 0 0 .401.17h2.286v2.286a.55.55 0 0 0 .17.401.55.55 0 0 0 .402.17h1.142a.55.55 0 0 0 .402-.17.55.55 0 0 0 .17-.401v-2.286h2.286a.55.55 0 0 0 .401-.17.55.55 0 0 0 .17-.402zM75 23c0 1.27-.313 2.441-.939 3.514a6.969 6.969 0 0 1-2.547 2.547A6.848 6.848 0 0 1 68 30a6.848 6.848 0 0 1-3.514-.939 6.969 6.969 0 0 1-2.547-2.547A6.848 6.848 0 0 1 61 23c0-1.27.313-2.441.939-3.514a6.969 6.969 0 0 1 2.547-2.547A6.848 6.848 0 0 1 68 16c1.27 0 2.441.313 3.514.939a6.969 6.969 0 0 1 2.547 2.547A6.848 6.848 0 0 1 75 23z"/>
    <filter id="filter-12" width="107.1%" height="114.3%" x="-3.6%" y="-3.6%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_newsletter_block">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(0 16)">
        <g fill="url(#linearGradient-1)" class="image_1" opacity=".4">
          <rect width="82" height="27" class="rectangle"/>
        </g>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-3)" xlink:href="#path-2"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-2"/>
        </g>
        <g class="shape">
          <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-4"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-7)" xlink:href="#path-6"/>
          <use fill="#FFF" fill-opacity=".8" xlink:href="#path-6"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-9)" xlink:href="#path-8"/>
          <use fill="#FFF" fill-opacity=".8" xlink:href="#path-8"/>
        </g>
        <mask id="mask-11" fill="#fff">
          <use xlink:href="#path-10"/>
        </mask>
        <g class="plus_circle">
          <use fill="#000" filter="url(#filter-12)" xlink:href="#path-10"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-10"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\js\mass_mailing_form_editor.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import FormEditorRegistry from "@website/js/form_editor_registry";

FormEditorRegistry.add('create_mailing_contact', {
    formFields: [{
        name: 'name',
        required: true,
        fillWith: "name",
        string: _t('Your Name'),
        type: 'char',
    }, {
        name: 'email',
        modelRequired: true,
        fillWith: "email",
        string: _t('Your Email'),
        type: 'email',
    }, {
        name: 'list_ids',
        relation: 'mailing.list',
        modelRequired: true,
        string: _t('Subscribe to'),
        type: 'many2many',
        fieldName: "name",
    }],
});

```

## File: static\src\js\website_mass_mailing.editor.js

```javascript
/** @odoo-module **/

import { ConfirmationDialog } from "@web/core/confirmation_dialog/confirmation_dialog";
import { _t } from "@web/core/l10n/translation";
import { renderToElement } from "@web/core/utils/render";
import options from "@web_editor/js/editor/snippets.options";

options.registry.mailing_list_subscribe = options.Class.extend({
    init() {
        this._super(...arguments);
        this.orm = this.bindService("orm");
    },

    /**
     * @override
     */
    onBuilt() {
        this._super(...arguments);
        if (this.mailingLists.length) {
            this.$target.attr("data-list-id", this.mailingLists[0][0]);
        } else {
            this.call("dialog", "add", ConfirmationDialog, {
                body: _t("No mailing list found, do you want to create a new one? This will save all your changes, are you sure you want to proceed?"),
                confirm: () => {
                    this.trigger_up("request_save", {
                        reload: false,
                        onSuccess: () => {
                            window.location.href =
                                "/odoo/action-mass_mailing.action_view_mass_mailing_lists";
                        },
                    });
                },
                cancel: () => {
                    this.trigger_up("remove_snippet", {
                        $snippet: this.$target,
                    });
                },
            });
        }
    },
    /**
     * @override
     */
    cleanForSave() {
        const previewClasses = ['o_disable_preview', 'o_enable_preview'];
        const toCleanElsSelector = ".js_subscribe_wrap, .js_subscribed_wrap";
        const toCleanEls = this.$target[0].querySelectorAll(toCleanElsSelector);
        toCleanEls.forEach(element => {
            element.classList.remove(...previewClasses);
        });
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * @see this.selectClass for parameters
     */
    toggleThanksMessage(previewMode, widgetValue, params) {
        const toSubscribeEl = this.$target[0].querySelector(".js_subscribe_wrap");
        const thanksMessageEl =
            this.$target[0].querySelector(".js_subscribed_wrap");

        thanksMessageEl.classList.toggle("o_disable_preview", !widgetValue);
        thanksMessageEl.classList.toggle("o_enable_preview", widgetValue);
        toSubscribeEl.classList.toggle("o_enable_preview", !widgetValue);
        toSubscribeEl.classList.toggle("o_disable_preview", widgetValue);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState(methodName, params) {
        if (methodName !== 'toggleThanksMessage') {
            return this._super(...arguments);
        }
        const toSubscribeElSelector = ".js_subscribe_wrap.o_disable_preview";
        return this.$target[0].querySelector(toSubscribeElSelector) ? "true" : "";
    },
    /**
     * @override
     */
    async _renderCustomXML(uiFragment) {
        this.mailingLists = await this.orm.call(
            "mailing.list",
            "name_search",
            ["", [["is_public", "=", true]]],
            { context: this.options.recordInfo.context }
        );
        if (this.mailingLists.length) {
            const selectEl = uiFragment.querySelector('we-select[data-attribute-name="listId"]');
            for (const mailingList of this.mailingLists) {
                const button = document.createElement('we-button');
                button.dataset.selectDataAttribute = mailingList[0];
                button.textContent = mailingList[1];
                selectEl.appendChild(button);
            }
        }
        const checkboxEl = document.createElement('we-checkbox');
        checkboxEl.setAttribute('string', _t("Display Thanks Message"));
        checkboxEl.dataset.toggleThanksMessage = 'true';
        checkboxEl.dataset.noPreview = 'true';
        checkboxEl.dataset.dependencies = "!form_opt";
        uiFragment.appendChild(checkboxEl);
    },
});

options.registry.recaptchaSubscribe = options.Class.extend({
    /**
     * Toggle the recaptcha legal terms
     */
    toggleRecaptchaLegal: function (previewMode, value, params) {
        const recaptchaLegalEl = this.$target[0].querySelector('.o_recaptcha_legal_terms');
        if (recaptchaLegalEl) {
            recaptchaLegalEl.remove();
        } else {
            const template = document.createElement('template');
            template.content.append(renderToElement("google_recaptcha.recaptcha_legal_terms"));
            this.$target[0].appendChild(template.content.firstElementChild);
        }
    },

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState: function (methodName, params) {
        switch (methodName) {
            case 'toggleRecaptchaLegal':
                return !this.$target[0].querySelector('.o_recaptcha_legal_terms') || '';
        }
        return this._super(...arguments);
    },
});

```

## File: static\src\js\website_mass_mailing.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import publicWidget from "@web/legacy/js/public/public_widget";
import { session } from "@web/session";
import {ReCaptcha} from "@google_recaptcha/js/recaptcha";
import { rpc } from "@web/core/network/rpc";

publicWidget.registry.subscribe = publicWidget.Widget.extend({
    selector: ".js_subscribe",
    disabledInEditableMode: false,
    read_events: {
        'click .js_subscribe_btn': '_onSubscribeClick',
    },

    /**
     * @constructor
     */
    init: function () {
        this._super(...arguments);
        this._recaptcha = new ReCaptcha();
        this.notification = this.bindService("notification");
        if (session.turnstile_site_key) {
            const { turnStile } = odoo.loader.modules.get('@website_cf_turnstile/js/turnstile');
            this._turnstile = turnStile;
        }
    },
    /**
     * @override
     */
    willStart: function () {
        this._recaptcha.loadLibs();
        return this._super(...arguments);
    },
    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);

        if (this.editableMode) {
            // Since there is an editor option to choose whether "Thanks" button
            // should be visible or not, we should not vary its visibility here.
            return def;
        }
        const always = this._updateView.bind(this);
        const inputName = this.el.querySelector('input').name;
        return Promise.all([def, rpc('/website_mass_mailing/is_subscriber', {
            'list_id': this._getListId(),
            'subscription_type': inputName,
        }).then(always, always)]);
    },
    /**
     * @override
     */
    destroy() {
        this._updateView({is_subscriber: false});
        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Modifies the elements to have the view of a subscriber/non-subscriber.
     *
     * @todo should probably be merged with _updateSubscribeControlsStatus
     * @param {Object} data
     */
    _updateView(data) {
        this._updateSubscribeControlsStatus(!!data.is_subscriber);

        // js_subscribe_email is kept by compatibility (it was the old name of js_subscribe_value)
        const valueInputEl = this.el.querySelector('input.js_subscribe_value, input.js_subscribe_email');
        valueInputEl.value = data.value || '';

        // Compat: remove d-none for DBs that have the button saved with it.
        this.el.classList.remove('d-none');
    },
    /**
     * Updates the visibility of the subscribe and subscribed buttons.
     *
     * @param {boolean} isSubscriber
     */
    _updateSubscribeControlsStatus(isSubscriber) {
        const thanksWrapEl = this.el.querySelector('.js_subscribed_wrap');
        const subscribeWrapEl = this.el.querySelector('.js_subscribe_wrap');
        const subscribeBtnEl = this.el.querySelector('.js_subscribe_btn');

        subscribeBtnEl.disabled = isSubscriber;
        subscribeWrapEl.classList.toggle('d-none', isSubscriber);
        thanksWrapEl.classList.toggle('d-none', !isSubscriber);

        // js_subscribe_email is kept by compatibility (it was the old name of js_subscribe_value)
        const valueInputEl = this.el.querySelector('input.js_subscribe_value, input.js_subscribe_email');
        valueInputEl.disabled = isSubscriber;

        // When the website is in edit mode, window.top != window. We don't want turnstile to render during edit mode
        // and mess up the DOM and saving it.
        if (!isSubscriber && this._turnstile && window.top === window) {
            const el = this._turnstile.addTurnstile('website_mass_mailing_subscribe');
            if (el) {
                this._turnstile.addSpinner(subscribeBtnEl);
                el[0].classList.add('mt-3');
                el.insertAfter(this.el);
                this._turnstile.renderTurnstile(el);
            }
        }
    },

    _getListId: function () {
        return this.$el.closest('[data-snippet=s_newsletter_block').data('list-id') || this.$el.data('list-id');
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onSubscribeClick: async function () {
        var self = this;
        const inputName = this.$('input').attr('name');
        const $input = this.$(".js_subscribe_value:visible, .js_subscribe_email:visible"); // js_subscribe_email is kept by compatibility (it was the old name of js_subscribe_value)
        if (inputName === 'email' && $input.length && !$input.val().match(/.+@.+/)) {
            this.$el.addClass('o_has_error').find('.form-control').addClass('is-invalid');
            return false;
        }
        this.$el.removeClass('o_has_error').find('.form-control').removeClass('is-invalid');
        const tokenObj = await this._recaptcha.getToken('website_mass_mailing_subscribe');
        if (tokenObj.error) {
            self.notification.add(tokenObj.error, {
                type: 'danger',
                title: _t("Error"),
                sticky: true,
            });
            return false;
        }
        rpc('/website_mass_mailing/subscribe', {
            'list_id': this._getListId(),
            'value': $input.length ? $input.val() : false,
            'subscription_type': inputName,
            recaptcha_token_response: tokenObj.token,
            turnstile_captcha: this.el.parentElement.querySelector('input[name="turnstile_captcha"]')?.value,
        }).then(function (result) {
            let toastType = result.toast_type;
            if (toastType === 'success') {
                self._updateSubscribeControlsStatus(true);

                const $popup = self.$el.closest('.o_newsletter_modal');
                if ($popup.length) {
                    $popup.modal('hide');
                }
            }
            self.notification.add(result.toast_content, {
                type: toastType,
                title: toastType === 'success' ? _t('Success') : _t('Error'),
                sticky: true,
            });
        });
    },
});

/**
 * This widget tries to fix snippets that were malformed because of a missing
 * upgrade script. Without this, some newsletter snippets coming from users
 * upgraded from a version lower than 16.0 may not be able to update their
 * newsletter block.
 *
 * TODO an upgrade script should be made to fix databases and get rid of this.
 */
publicWidget.registry.fixNewsletterListClass = publicWidget.Widget.extend({
    selector: '.s_newsletter_subscribe_form:not(.s_subscription_list), .s_newsletter_block',

    /**
     * @override
     */
    start() {
        this.$target[0].classList.add('s_newsletter_list');
        return this._super(...arguments);
    },
});

```

## File: static\src\snippets\s_popup\000.js

```javascript
/** @odoo-module **/

import PopupWidget from '@website/snippets/s_popup/000';

PopupWidget.include({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Prevents the (newsletter) popup to be shown if the user is subscribed.
     *
     * @override
     */
    _canShowPopup() {
        if (
            this.$el.is('.o_newsletter_popup') &&
            this.$el.find('input.js_subscribe_value, input.js_subscribe_email').prop('disabled') // js_subscribe_email is kept by compatibility (it was the old name of js_subscribe_value)
        ) {
            return false;
        }
        return this._super(...arguments);
    },
    /**
     * @override
     */
    _canBtnPrimaryClosePopup(primaryBtnEl) {
        if (primaryBtnEl.classList.contains('js_subscribe_btn')) {
            return false;
        }
        return this._super(...arguments);
    },
});

export default PopupWidget;

```

## File: static\src\snippets\s_popup\options.js

```javascript
/** @odoo-module **/

import options from '@web_editor/js/editor/snippets.options';

options.registry.NewsletterLayout = options.registry.SelectTemplate.extend({
    /**
     * @constructor
     */
    init() {
        this._super(...arguments);
        this.containerSelector = '> .container, > .container-fluid, > .o_container_small';
        this.selectTemplateWidgetName = 'newsletter_template_opt';
    },
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

## File: views\snippets_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="remove_external_snippets" inherit_id="website.external_snippets">
    <xpath expr="//t[@id='newsletter_snippet']" position="replace"/>
</template>

<template id="snippets" inherit_id="website.snippets">
    <xpath expr="//t[@id='mass_mailing_newsletter_block_hook']" position="replace">
        <t t-snippet="website_mass_mailing.s_newsletter_block" string="Newsletter Block" t-forbid-sanitize="form" group="contact_and_forms">
            <keywords>form, updates, digest, bulletin, announcements, notifications, communication, email, modal, alert, dialog, prompt, pop-up, subscription</keywords>
        </t>
    </xpath>
    <xpath expr="//t[@id='mass_mailing_newsletter_popup_hook']" position="replace">
        <t t-snippet="website_mass_mailing.s_newsletter_subscribe_popup" string="Newsletter Popup" t-forbid-sanitize="form" group="contact_and_forms"/>
    </xpath>
    <xpath expr="//t[@id='mass_mailing_newsletter_hook']" position="replace">
        <t t-snippet="website_mass_mailing.s_newsletter_subscribe_form" string="Newsletter" t-thumbnail="/website/static/src/img/snippets_thumbs/s_newsletter_subscribe_form.svg" t-forbid-sanitize="form"/>
    </xpath>
</template>

<!--
Users upgraded from a version lower than 16.0 may have those blocks in their
database, without the s_newsletter_list class. See fixNewsletterListClass.
-->
<template id="s_newsletter_subscribe_form" name="Newsletter">
    <div class="s_newsletter_subscribe_form s_newsletter_list js_subscribe" data-vxml="001" data-list-id="0" data-name="Newsletter Form">
        <div class="js_subscribed_wrap d-none">
            <p class="h4-fs text-center text-success"><i class="fa fa-check-circle-o" role="img"/> Thanks for registering!</p>
        </div>
        <div class="js_subscribe_wrap">
            <div class="input-group">
                <input type="email" name="email" class="js_subscribe_value form-control" placeholder="johnsmith@example.com"/>
                <a role="button" href="#" class="btn btn-primary js_subscribe_btn o_submit">Subscribe</a>
            </div>
        </div>
    </div>
</template>

<!--
Users upgraded from a version lower than 16.0 may have those blocks in their
database, without the s_newsletter_list class. See fixNewsletterListClass.
-->
<template id="s_newsletter_block" name="Newsletter Block">
    <section class="s_newsletter_block s_newsletter_list o_cc o_cc2 pt64 pb64" data-list-id="0">
        <div class="container">
            <t t-call="website_mass_mailing.s_newsletter_block_default_template"/>
        </div>
    </section>
</template>

<template id="s_newsletter_block_default_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-6">
            <h2 class="h4-fs">Subscribe to our newsletter</h2>
            <p class="text-muted">Get all the latest news, blog posts and product updates from our company, delivered directly to your inbox.</p>
        </div>
        <div class="col-lg-5 offset-lg-1">
            <t t-snippet-call="website_mass_mailing.s_newsletter_subscribe_form"/>
        </div>
    </div>
</template>

<template id="s_newsletter_block_form_template" groups="base.group_user">
    <section class="s_website_form" data-vcss="001" data-snippet="s_website_form" data-name="Form">
        <div class="o_container_small">
            <form id="newsletter_form" action="/website/form/" method="post" enctype="multipart/form-data" class="o_mark_required"
                  data-mark="*" data-model_name="mailing.contact" data-success-mode="message" hide-change-model="true">
                <div class="s_website_form_rows row s_col_no_bgcolor">
                    <div class="mb-0 py-2 col-12 s_website_form_field s_website_form_model_required" data-type="char" data-name="Field">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <label class="col-form-label col-sm-auto s_website_form_label" style="width: 250px" for="mailing_sub1">
                                <span class="s_website_form_label_content">Name</span>
                                <span class="s_website_form_mark"> *</span>
                            </label>
                            <div t-if="request.env['mailing.contact']._is_name_split_activated()" class="col-sm row">
                                <div class="col-sm-6">
                                    <input id="mailing_sub1" type="text" class="form-control s_website_form_input"
                                           name="first_name" placeholder="First Name" required="1"/>
                                </div>
                                <div class="col-sm-6">
                                    <input type="text" class="form-control s_website_form_input"
                                           name="last_name" placeholder="Last Name" required="1"/>
                                </div>
                            </div>
                            <div t-else="" class="col-sm">
                                <input id="mailing_sub1" type="text" class="form-control s_website_form_input" name="name" required="1" placeholder="John Smith"/>
                            </div>
                        </div>
                    </div>
                    <div class="mb-0 py-2 col-12 s_website_form_field s_website_form_model_required" data-type="email" data-name="Field">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <label class="col-form-label col-sm-auto s_website_form_label" style="width: 250px" for="mailing_sub2">
                                <span class="s_website_form_label_content">Email</span>
                                <span class="s_website_form_mark"> *</span>
                            </label>
                            <div class="col-sm">
                                <input id="mailing_sub2" type='email' class='form-control s_website_form_input' name="email" placeholder="johnsmith@example.com"/>
                            </div>
                        </div>
                    </div>
                    <div class="mb-0 py-2 col-12 s_website_form_field s_website_form_model_required" data-type="many2many" data-name="Field">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <label class="col-sm-auto s_website_form_label" style="width: 250px" for="mailing_list_">
                                <span class="s_website_form_label_content">Subscribe to</span>
                                <span class="s_website_form_mark"> *</span>
                            </label>
                            <div class="col-sm">
                                <div class="row s_col_no_resize s_col_no_bgcolor s_website_form_multiple" data-name="list_ids" data-display="horizontal">
                                    <t t-foreach="request.env['mailing.list'].search([('is_public', '=', True)])" t-as="record">
                                        <div class="checkbox col-12 col-lg-4 col-md-6">
                                            <div class="form-check">
                                                <input type="checkbox" class="s_website_form_input form-check-input" name="list_ids"
                                                    t-attf-id="mailing_list_#{record.id}" t-att-value="record.id" required="1"/>
                                                <label class="form-check-label s_website_form_check_label" t-attf-for="mailing_list_#{record.id}">
                                                    <t t-esc="record.name"/>
                                                </label>
                                            </div>
                                        </div>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="mb-0 py-2 col-12 s_website_form_field s_website_form_custom s_website_form_required" data-type="boolean" data-name="Field">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <label class="col-sm-auto s_website_form_label" style="width: 250px;" for="mailing_sub3">
                                <span class="s_website_form_label_content">I agree to receive updates</span>
                                <span class="s_website_form_mark"> *</span>
                            </label>
                            <div class="col-sm">
                                <div class="form-check">
                                    <input type="checkbox" value="Yes" class="s_website_form_input form-check-input" name="I want to be kept updated"
                                           id="mailing_sub3" required="1"/>
                                </div>
                                <div class="s_website_form_field_description small form-text text-muted">
                                    We send one weekly newsletter per list and always try to keep it interesting. You can unsubscribe at any time.
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="s_website_form_submit s_website_form_no_submit_label col-12 mb-0 py-2 text-end" data-name="Submit Button">
                        <div style="width: 250px;" class="s_website_form_label"/>
                        <span id="s_website_form_result"></span>
                        <a href="#" role="button" class="btn btn-primary ms-auto ms-lg-0 s_website_form_send">Subscribe</a>
                    </div>
                </div>
            </form>
            <div class="s_website_form_end_message d-none">
                <div class="oe_structure">
                    <section class="s_text_block pt64 pb64 o_colored_level o_cc o_cc2 text-center" data-snippet="s_text_block">
                        <div class="container">
                            <i class="fa fa-paper-plane fa-2x mb-3 rounded-circle text-bg-success" role="presentation"/>
                            <h1 class="fw-bolder">Thank you for subscribing!</h1>
                            <p class="lead">You will now be informed about the latest news.</p>
                        </div>
                    </section>
                </div>
            </div>
        </div>
    </section>
</template>

<template id="s_newsletter_subscribe_popup" name="Newsletter Popup">
    <div class="s_popup o_newsletter_popup o_snippet_invisible" data-name="Newsletter Popup" data-vcss="001" data-invisible="1">
        <div class="modal fade s_popup_middle o_newsletter_modal"
             style="background-color: var(--black-50) !important;"
             data-show-after="5000" data-display="afterDelay" data-consents-duration="7"
             data-bs-focus="false" data-bs-backdrop="false" tabindex="-1" role="dialog">
            <div class="modal-dialog d-flex">
                <div class="modal-content oe_structure">
                    <div class="s_popup_close js_close_popup o_we_no_overlay o_not_editable" aria-label="Close">&#215;</div>
                    <section class="s_text_block oe_img_bg o_bg_img_center pt88 pb64" data-snippet="s_text_block" data-name="Text"
                             style="background-image: url('/web/image/website.s_cover_default_image'); background-position: 0 100%;">
                        <div class="container s_allow_columns">
                            <div class="row">
                                <div class="col-lg-12 bg-black-50">
                                    <h2 style="text-align: center;">Always <b>First</b>.</h2>
                                    <p style="text-align: center;">Be the first to find out all the latest news,<br/> products, and trends.</p>
                                </div>
                            </div>
                        </div>
                    </section>
                    <section class="s_text_block" data-snippet="s_text_block" data-name="Text">
                        <div class="container">
                            <div class="row s_nb_column_fixed g-0">
                                <div class="col-lg-8 offset-lg-2 pt32 pb32">
                                    <t t-snippet-call="website_mass_mailing.s_newsletter_subscribe_form"/>
                                </div>
                            </div>
                        </div>
                    </section>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="newsletter_subscribe_options" name="Newsletter Subscribe Options" inherit_id="website.snippet_options">
    <xpath expr="//*[@t-set='so_snippet_addition_selector']" position="inside" t-translation="off">, .o_newsletter_popup</xpath>
    <xpath expr="//div[1]" position="before">
        <div data-js="NewsletterLayout" data-selector=".s_newsletter_block">
            <we-select string="Template"
                       data-name="newsletter_template_opt"
                       data-attribute-name="newsletterTemplate"
                       data-attribute-default-value="email">
                <we-button title="Email Subscription" string="Email Subscription"
                           data-select-template="website_mass_mailing.s_newsletter_block_default_template"
                           data-select-data-attribute="email" data-name="email_opt"/>
                <we-button title="Form Subscription" string="Form Subscription"
                           data-select-template="website_mass_mailing.s_newsletter_block_form_template"
                           data-select-data-attribute="form" data-name="form_opt"/>
            </we-select>
        </div>
        <t t-call="website_mass_mailing.newsletter_subscribe_options_common">
            <t t-set="_selector">.o_newsletter_popup</t>
            <t t-set="_target">.s_newsletter_list</t>
        </t>
        <t t-call="website_mass_mailing.newsletter_subscribe_options_common">
            <t t-set="_selector">.s_newsletter_list</t>
            <t t-set="_exclude">.s_newsletter_block .s_newsletter_list, .o_newsletter_popup .s_newsletter_list</t>
        </t>
        <div data-selector=".js_subscribe" data-drop-near="p, h1, h2, h3, blockquote, .card"/>
    </xpath>
</template>

<template id="newsletter_subscribe_options_common">
    <div data-js="mailing_list_subscribe"
         t-att-data-selector="_selector"
         t-att-data-exclude="_exclude"
         t-att-data-target="_target">
        <we-select string="Newsletter" data-attribute-name="listId" data-dependencies="!form_opt"></we-select>
    </div>
    <div data-js="recaptchaSubscribe"
         t-att-data-selector="_selector"
         t-att-data-exclude="_exclude"
         t-att-data-target="_target">
        <t t-set="recaptcha_public_key" t-value="request.env['ir.config_parameter'].sudo().get_param('recaptcha_public_key')"/>
        <we-checkbox t-if="recaptcha_public_key" string="Show reCaptcha Policy" data-toggle-recaptcha-legal="" data-no-preview="true"/>
    </div>
</template>

<!-- Extend default mass_mailing snippets with website feature -->

<template id="s_mail_block_footer_social" inherit_id="mass_mailing.s_mail_block_footer_social">
    <xpath expr="//div[hasclass('o_mail_footer_links')]" position="inside">
        <t> | <a role="button" href="/contactus" class="btn btn-link">Contact</a></t>
    </xpath>
</template>

<template id="s_mail_block_footer_social_left" inherit_id="mass_mailing.s_mail_block_footer_social_left">
    <xpath expr="//div[hasclass('o_mail_footer_links')]" position="inside">
        <t> | <a role="button" href="/contactus" class="btn btn-link">Contact</a></t>
    </xpath>
</template>

</odoo>

```

## File: views\snippets\s_popup.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<record id="website_mass_mailing.s_popup_000_js" model="ir.asset">
    <field name="name">Popup 000 JS Website Mass Mailing Override</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">website_mass_mailing/static/src/snippets/s_popup/000.js</field>
</record>

</odoo>

```

