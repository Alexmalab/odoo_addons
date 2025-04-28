# Odoo Module: website_event_booth

Category: Marketing/Events

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
    'name': 'Online Event Booths',
    'category': 'Marketing/Events',
    'version': '1.0',
    'summary': 'Events, display your booths on your website',
    'description': """
Display your booths on your website for the users to register.
    """,
    'depends': ['website_event', 'event_booth'],
    'data': [
        'security/ir.model.access.csv',
        'security/event_booth_security.xml',
        'views/event_type_views.xml',
        'views/event_event_views.xml',
        'views/event_booth_registration_templates.xml',
        'views/event_booth_templates.xml',
    ],
    'demo': [
        'data/event_demo.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_frontend': [
            '/website_event_booth/static/src/js/booth_register.js',
            '/website_event_booth/static/src/scss/website_event_booth.scss',
            'website_event_booth/static/src/xml/event_booth_registration_templates.xml',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\event_booth.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import werkzeug
from werkzeug.exceptions import Forbidden, NotFound

from odoo import exceptions, http, tools
from odoo.http import request
from odoo.addons.website_event.controllers.main import WebsiteEventController


class WebsiteEventBoothController(WebsiteEventController):

    @http.route('/event/<model("event.event"):event>/booth', type='http', auth='public', website=True, sitemap=True)
    def event_booth_main(self, event):
        try:
            event.check_access_rights('read')
            event.check_access_rule('read')
        except exceptions.AccessError:
            raise Forbidden()

        event_sudo = event.sudo()
        values = {
            'event': event_sudo,
            'event_booths': event_sudo.event_booth_ids,
            'available_booth_category_ids': event_sudo.event_booth_category_available_ids,
            'main_object': event,
        }
        return request.render('website_event_booth.event_booth_registration', values)

    @http.route('/event/<model("event.event"):event>/booth/register',
                type='http', auth='public', methods=['POST'], website=True, sitemap=False)
    def event_booth_register(self, event, booth_category_id, event_booth_ids):
        # `event_booth_id` in `requests.params` only contains the first
        # checkbox, we re-parse the form using getlist to get them all
        event_booth_ids = request.httprequest.form.getlist('event_booth_ids')

        return request.redirect(('/event/%s/booth/register_form?' % event.id) + werkzeug.urls.url_encode({
            'booth_ids': ','.join(event_booth_ids),
            'booth_category_id': int(booth_category_id),
        }))

    @http.route('/event/<model("event.event"):event>/booth/register_form',
                type='http', auth='public', methods=['GET'], website=True, sitemap=False)
    def event_booth_contact_form(self, event, booth_ids=None, booth_category_id=None):
        if not booth_ids or not booth_category_id:
            raise NotFound()

        booth_category = request.env['event.booth.category'].sudo().browse(int(booth_category_id))
        event_booths = request.env['event.booth'].sudo().browse([int(booth_id) for booth_id in booth_ids.split(',')])
        default_contact = {}
        if not request.env.user._is_public():
            default_contact = {
                'name': request.env.user.partner_id.name,
                'email': request.env.user.partner_id.email,
                'phone': request.env.user.partner_id.phone,
                'mobile': request.env.user.partner_id.mobile,
            }
        else:
            visitor = request.env['website.visitor']._get_visitor_from_request()
            if visitor.email:
                default_contact = {
                    'name': visitor.name,
                    'email': visitor.email,
                    'mobile': visitor.mobile,
                }
        return request.render(
            'website_event_booth.event_booth_registration_details',
            {'event': event.sudo(),
             'default_contact': default_contact,
             'booth_category': booth_category,
             'event_booths': event_booths,
             'redirect_url': werkzeug.urls.url_quote(request.httprequest.full_path),
            }
        )

    def _get_requested_booths(self, event, event_booth_ids):
        booth_ids = json.loads(event_booth_ids)
        booths = request.env['event.booth'].sudo().search([
            ('event_id', '=', event.id),
            ('state', '=', 'available'),
            ('id', 'in', booth_ids)
        ])
        if booth_ids != booths.ids or len(booths.booth_category_id) != 1:
            return request.env['event.booth']
        return booths

    @http.route('/event/<model("event.event"):event>/booth/confirm',
                type='http', auth='public', methods=['POST'], website=True, sitemap=False)
    def event_booth_registration_confirm(self, event, booth_category_id, event_booth_ids, **kwargs):
        booths = self._get_requested_booths(event, event_booth_ids)

        error_code = self._check_booth_registration_values(booths, kwargs['contact_email'])
        if error_code:
            return json.dumps({'error': error_code})

        booth_values = self._prepare_booth_registration_values(event, kwargs)
        booths.action_confirm(booth_values)

        return self._prepare_booth_registration_success_values(event.name, booth_values)

    def _check_booth_registration_values(self, booths, contact_email, booth_category=False):
        if not booths:
            return 'boothError'

        if booth_category and not booth_category.exists():
            return 'boothCategoryError'

        email_normalized = tools.email_normalize(contact_email)
        if request.env.user._is_public() and email_normalized:
            partner = request.env['res.partner'].sudo().search([
                ('email_normalized', '=', email_normalized)
            ], limit=1)
            if partner:
                return 'existingPartnerError'

        return False

    def _prepare_booth_registration_values(self, event, kwargs):
        return self._prepare_booth_registration_partner_values(event, kwargs)

    def _prepare_booth_registration_partner_values(self, event, kwargs):
        if request.env.user._is_public():
            conctact_email_normalized = tools.email_normalize(kwargs['contact_email'])
            contact_name_email = tools.formataddr((kwargs['contact_name'], conctact_email_normalized))
            partner = request.env['res.partner'].sudo().find_or_create(contact_name_email)
            if not partner.name and kwargs.get('contact_name'):
                partner.name = kwargs['contact_name']
            if not partner.phone and kwargs.get('contact_phone'):
                partner.phone = kwargs['contact_phone']
            if not partner.mobile and kwargs.get('contact_mobile'):
                partner.mobile = kwargs['contact_mobile']
        else:
            partner = request.env.user.partner_id
        return {
            'partner_id': partner.id,
            'contact_name': kwargs.get('contact_name') or partner.name,
            'contact_email': kwargs.get('contact_email') or partner.email,
            'contact_mobile': kwargs.get('contact_mobile') or partner.mobile,
            'contact_phone': kwargs.get('contact_phone') or partner.phone,
        }

    def _prepare_booth_registration_success_values(self, event_name, booth_values):
        return json.dumps({
            'success': True,
            'event_name': event_name,
            'contact': {
                'name': booth_values.get('contact_name'),
                'email': booth_values.get('contact_email'),
                'phone': booth_values.get('contact_phone'),
                'mobile': booth_values.get('contact_mobile'),
            },
        })

    @http.route('/event/booth/check_availability', type='json', auth='public', methods=['POST'])
    def check_booths_availability(self, event_booth_ids=None):
        if not event_booth_ids:
            return {}
        booths = request.env['event.booth'].sudo().browse(event_booth_ids)
        return {
            'unavailable_booths': booths.filtered(lambda booth: not booth.is_available).ids
        }

    @http.route(['/event/booth_category/get_available_booths'], type='json', auth='public')
    def get_booth_category_available_booths(self, event_id, booth_category_id):
        booth_ids = request.env['event.booth'].sudo().search([
            ('event_id', '=', int(event_id)),
            ('booth_category_id', '=', int(booth_category_id)),
            ('state', '=', 'available')
        ])

        return [
            {'id': booth.id, 'name': booth.name}
            for booth in booth_ids
        ]

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_booth

```

## File: data\event_demo.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event.event_2" model="event.event">
        <field name="exhibition_map" type="base64" file="website_event_booth/static/src/img/exhibition-map.gif"/>
        <field name="website_menu" eval="True"/>
        <field name="booth_menu" eval="True"/>
    </record>

    <record id="event.event_7" model="event.event">
        <field name="exhibition_map" type="base64" file="website_event_booth/static/src/img/exhibition-map.gif"/>
        <field name="booth_menu" eval="True"/>
    </record>

</data></odoo>

```

## File: models\event_event.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.addons.http_routing.models.ir_http import slug


class Event(models.Model):
    _inherit = 'event.event'

    exhibition_map = fields.Image(string='Exhibition Map', max_width=1024, max_height=1024)
    # frontend menu management
    booth_menu = fields.Boolean(
        string='Booth Register', compute='_compute_booth_menu',
        readonly=False, store=True)
    booth_menu_ids = fields.One2many(
        'website.event.menu', 'event_id', string='Event Booths Menus',
        domain=[('menu_type', '=', 'booth')])

    @api.depends('event_type_id', 'website_menu')
    def _compute_booth_menu(self):
        for event in self:
            if event.event_type_id and event.event_type_id != event._origin.event_type_id:
                event.booth_menu = event.event_type_id.booth_menu
            elif event.website_menu and (event.website_menu != event._origin.website_menu or not event.booth_menu):
                event.booth_menu = True
            elif not event.website_menu:
                event.booth_menu = False

    # ------------------------------------------------------------
    # WEBSITE MENU MANAGEMENT
    # ------------------------------------------------------------

    def toggle_booth_menu(self, val):
        self.booth_menu = val

    def _get_menu_update_fields(self):
        return super(Event, self)._get_menu_update_fields() + ['booth_menu']

    def _update_website_menus(self, menus_update_by_field=None):
        super(Event, self)._update_website_menus(menus_update_by_field=menus_update_by_field)
        for event in self:
            if event.menu_id and (not menus_update_by_field or event in menus_update_by_field.get('booth_menu')):
                event._update_website_menu_entry('booth_menu', 'booth_menu_ids', 'booth')

    def _get_menu_type_field_matching(self):
        res = super(Event, self)._get_menu_type_field_matching()
        res['booth'] = 'booth_menu'
        return res

    def _get_website_menu_entries(self):
        self.ensure_one()
        return super(Event, self)._get_website_menu_entries() + [
            (_('Get A Booth'), '/event/%s/booth' % slug(self), False, 90, 'booth')
        ]

```

## File: models\event_type.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class EventType(models.Model):
    _inherit = 'event.type'

    booth_menu = fields.Boolean(
        string='Booths on Website', compute='_compute_booth_menu',
        readonly=False, store=True)

    @api.depends('website_menu')
    def _compute_booth_menu(self):
        for event_type in self:
            event_type.booth_menu = event_type.website_menu

```

## File: models\website_event_menu.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class EventMenu(models.Model):
    _inherit = "website.event.menu"

    menu_type = fields.Selection(
        selection_add=[('booth', 'Event Booth Menus')], ondelete={'booth': 'cascade'})

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import event_type
from . import event_event
from . import website_event_menu

```

## File: security\event_booth_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <record id="ir_rule_event_booth_public" model="ir.rule">
        <field name="name">Event Booth: public/portal: published read</field>
        <field name="model_id" ref="event_booth.model_event_booth"/>
        <field name="domain_force">[('event_id.website_published', '=', True)]</field>
        <field name="groups" eval="[(4, ref('base.group_public')), (4, ref('base.group_portal'))]"/>
        <field name="perm_read" eval="True"/>
        <field name="perm_write" eval="False"/>
        <field name="perm_create" eval="False"/>
        <field name="perm_unlink" eval="False"/>
    </record>

</data></odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_event_booth_category,event.booth.category,event_booth.model_event_booth_category,,1,0,0,0
access_event_booth_all,event.booth.public,event_booth.model_event_booth,,1,0,0,0

```

## File: static\src\js\booth_register.js

```javascript
odoo.define('website_event_booth.booth_registration', function (require) {
'use strict';

var core = require('web.core');
var dom = require('web.dom');
var publicWidget = require('web.public.widget');
var QWeb = core.qweb;
var _t = core._t;

publicWidget.registry.boothRegistration = publicWidget.Widget.extend({
    selector: '.o_wbooth_registration',
    events: {
        'change input[name="booth_category_id"]': '_onChangeBoothType',
        'change .form-check > input[type="checkbox"]': '_onChangeBooth',
        'click .o_wbooth_registration_submit': '_onSubmitBoothSelectionClick',
        'click .o_wbooth_registration_confirm': '_onConfirmRegistrationClick',
    },

    start() {
        this.eventId = parseInt(this.$el.data('event-id'));
        this.activeType = false;
        this.boothCache = {};
        return this._super.apply(this, arguments).then(() => {
            this.$('input[name="booth_category_id"]:enabled:first').click();
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _check_booths_availability(eventBoothIds) {
        const self = this;
        return this._rpc({
            route: "/event/booth/check_availability",
            params: {
                event_booth_ids: eventBoothIds,
            },
        }).then(function (result) {
            if (result.unavailable_booths.length) {
                self.$('input[name="event_booth_ids"]').each(function (i, el) {
                    if (result.unavailable_booths.includes(parseInt(el.value))) {
                        $(el).closest('.form-check').addClass('text-danger');
                    }
                });
                self.$('.o_wbooth_unavailable_booth_alert').removeClass('d-none');
                return Promise.resolve(false);
            }
            return Promise.resolve(true);
        })
    },

    _countSelectedBooths() {
        return this.$('.form-check > input[type="checkbox"]:checked').length;
    },

    _fillBooths() {
        var $boothElem = this.$('.o_wbooth_booths');
        $boothElem.empty();
        $.each(this.boothCache[this.activeType], function (key, booth) {
            let $checkbox = dom.renderCheckbox({
                text: booth.name,
                prop: {
                    name: 'event_booth_ids',
                    value: booth.id
                }
            });
            $boothElem.append($checkbox);
        });
    },

    /**
     * Check if the confirmation form is valid by testing each of its inputs
     *
     * @private
     * @param $form
     * @return {boolean} - true if no errors else false
     */
    _isConfirmationFormValid($form) {
        const formErrors = [];

        $form.find('.form-control').each(function () {
            let input = $(this);
            input.removeClass('is-invalid');
            if (input.length && !input[0].checkValidity()) {
                input.addClass('is-invalid');
                formErrors.push('invalidFormInputs');
            }
        });

        this._updateErrorDisplay(formErrors);
        return formErrors.length === 0;
    },

    _showBoothCategoryDescription() {
        this.$('.o_wbooth_booth_description').addClass('d-none');
        this.$('#o_wbooth_booth_description_' + this.activeType).removeClass('d-none');
    },

    /**
     * Display the errors with a custom message when confirming
     * the registration if there is any.
     *
     * @private
     * @param errors
     */
    _updateErrorDisplay(errors) {
        this.$('.o_wbooth_registration_error_section').toggleClass('d-none', !errors.length);
        this.$('.o_wbooth_registration_error_signin').addClass('d-none');

        let errorMessages = [];
        let $errorMessage = this.$('.o_wbooth_registration_error_message');

        if (errors.includes('invalidFormInputs')) {
            errorMessages.push(_t("Please fill out the form correctly."));
        }

        if (errors.includes('boothError')) {
            errorMessages.push(_t("Booth registration failed."));
        }

        if (errors.includes('boothCategoryError')) {
            errorMessages.push(_t("The booth category doesn't exist."));
        }

        if (errors.includes('existingPartnerError')) {
            errorMessages.push(_t("It looks like your email is linked to an existing account."));
            this.$('.o_wbooth_registration_error_signin').removeClass('d-none');
        }

        $errorMessage.text(errorMessages.join(' ')).change();
    },

    _updateUiAfterBoothCategoryChange() {
        this._fillBooths();
        this._showBoothCategoryDescription();
        this._updateUiAfterBoothChange(this._countSelectedBooths());
    },

    _updateUiAfterBoothChange(boothCount) {
        let $button = this.$('button.o_wbooth_registration_submit');
        $button.attr('disabled', !boothCount);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onChangeBooth(ev) {
        $(ev.currentTarget).closest('.form-check').removeClass('text-danger');
        this._updateUiAfterBoothChange(this._countSelectedBooths());
    },

    /**
     * Load all the booths related to the chosen booth category and
     * add them to a local dictionary to avoid making rpc each time the
     * user change the booth category.
     *
     * Then the selection input will be filled with the fetched booth values.
     *
     * @param ev
     * @private
     */
    _onChangeBoothType(ev) {
        ev.preventDefault();
        this.activeType = parseInt(ev.currentTarget.value);
        if (this.boothCache[this.activeType] === undefined) {
            var self = this;
            this._rpc({
                route: '/event/booth_category/get_available_booths',
                params: {
                    event_id: this.eventId,
                    booth_category_id: this.activeType,
                },
            }).then(function (result) {
                self.boothCache[self.activeType] = result;
                self._updateUiAfterBoothCategoryChange();
            });
        } else {
            this._updateUiAfterBoothCategoryChange();
        }
    },

    async _onSubmitBoothSelectionClick(ev) {
        ev.preventDefault();
        let $form = this.$('.o_wbooth_registration_form');
        let event_booth_ids = this.$('input[name=event_booth_ids]:checked').map(function () {
            return parseInt($(this).val());
        }).get();
        if (await this._check_booths_availability(event_booth_ids)) {
            $form.submit();
        }
    },

    /**
     * Submit the confirmation form if no errors are present after validation.
     *
     * If the submission succeed, we replace the form with a success message template.
     *
     * @param ev
     * @return {Promise<void>}
     * @private
     */
    async _onConfirmRegistrationClick(ev) {
        ev.preventDefault();
        ev.stopPropagation();

        $(ev.currentTarget).addClass('disabled').attr('disabled', 'disabled');

        const $form = this.$('#o_wbooth_contact_details_form');
        if (this._isConfirmationFormValid($form)) {
            const formData = new FormData($form[0]);
            const response = await $.ajax({
                url: `/event/${encodeURIComponent(this.$el.data('eventId'))}/booth/confirm`,
                data: formData,
                processData: false,
                contentType: false,
                type: 'POST',
            });

            const jsonResponse = response && JSON.parse(response);
            if (jsonResponse.success) {
                $form.replaceWith($(QWeb.render('event_booth_registration_complete', {
                    'event_name': jsonResponse.event_name,
                    'contact': jsonResponse.contact,
                })));
            } else if (jsonResponse.redirect) {
                window.location.href = jsonResponse.redirect;
            } else if (jsonResponse.error) {
                this._updateErrorDisplay(jsonResponse.error);
            }
        }

        $(ev.currentTarget).removeClass('disabled').removeAttr('disabled');
    },

});

return publicWidget.registry.boothRegistration;
});

```

## File: static\src\xml\event_booth_registration_templates.xml

```xml
<templates>

    <t t-name="event_booth_registration_complete">
        <div class="col-12 p-5">
            <div class="row mb-3">
                <div class="col-12">
                    <h3>Booth Registration completed!</h3>
                    <span class="h4 text-muted" t-out="event_name"/>
                </div>
            </div>
            <div class="d-flex flex-column">
                <span t-if="contact.name" t-out="contact.name" class="fw-bold"/>
                <span t-if="contact.email">
                    <i class="fa fa-fw fa-envelope me-2"/>
                    <t t-out="contact.email"/>
                </span>
                <span t-if="contact.phone">
                    <i class="fa fa-fw fa-phone me-2"/>
                    <t t-out="contact.phone"/>
                </span>
                <span t-if="contact.mobile">
                    <i class="fa fa-fw fa-mobile me-2"/>
                    <t t-out="contact.mobile"/>
                </span>
            </div>
        </div>
    </t>

</templates>

```

## File: views\event_booth_registration_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <template id="event_booth_registration_details" name="Event Booth Registration Details">
        <t t-call="website_event_booth.event_booth_layout">
            <form method="post"
                id="o_wbooth_contact_details_form"
                t-att-data-event-id="event.id"
                class="col-12 px-5 py-2 js_website_submit_form">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <input type="hidden" name="booth_category_id" t-att-value="booth_category.id"/>
                <input type="hidden" name="event_booth_ids" t-att-value="event_booths.ids"/>
                <div id="o_wbooth_contact_details">
                    <h4 class="mt32">
                        <strong>Contact Details</strong>
                    </h4>
                    <div class="row mb-3">
                        <label class="col-form-label col-sm-auto">
                            <span>Name</span>
                            <span> *</span>
                        </label>
                        <div class="col-sm">
                            <input type="text" class="form-control" name="contact_name" required="True"
                                   t-att-value="default_contact.get('name', '')"/>
                        </div>
                    </div>
                    <div class="row mb-3">
                        <label class="col-form-label col-sm-auto">
                            <span>Email</span>
                            <span> *</span>
                        </label>
                        <div class="col-sm">
                            <input type="email" class="form-control" name="contact_email" required="True"
                                   t-att-value="default_contact.get('email', '')"/>
                        </div>
                    </div>
                    <div class="row mb-3">
                        <label class="col-form-label col-sm-auto">Phone</label>
                        <div class="col-sm">
                            <input type="tel" class="form-control" name="contact_phone"
                                   t-att-value="default_contact.get('phone', '')"/>
                        </div>
                    </div>
                    <div class="row mb-3">
                        <label class="col-form-label col-sm-auto">Mobile</label>
                        <div class="col-sm">
                            <input type="tel" class="form-control" name="contact_mobile"
                                   t-att-value="default_contact.get('mobile', '')"/>
                        </div>
                    </div>
                </div>
                <div class="o_wbooth_registration_error_section alert alert-danger d-none mt-4" role="alert">
                    <i class="fa fa-exclamation-triangle me-2" role="img" aria-label="Error" title="Error"/>
                    <span class="o_wbooth_registration_error_message"/>
                    <a class="o_wbooth_registration_error_signin d-none"
                        t-attf-href="/web/login?redirect={{redirect_url}}">
                        Please Sign In.
                    </a>
                </div>
                <div class="mb-3">
                    <div class="d-grid col-sm-6 offset-sm-3 mt-5">
                        <button type="submit" class="btn btn-primary fw-bold o_wbooth_registration_confirm">
                            Book my Booths
                        </button>
                    </div>
                </div>
            </form>
        </t>
    </template>

</data></odoo>

```

## File: views\event_booth_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <template id="event_booth_layout" name="Event Booth Layout">
        <t t-call="website_event.layout">
            <div class="o_wbooth_registration" t-att-data-event-id="event.id"
                 itemscope="itemscope" itemtype="http://schema.org/Event">
                <t t-call="website.record_cover">
                    <t t-set="_record" t-value="event"/>
                    <t t-set="use_filters" t-value="True"/>
                    <t t-set="use_text_align" t-value="True"/>
                    <t t-set="additionnal_classes" t-value="'pb128'"/>

                    <div class="container d-flex flex-column flex-grow-1">
                        <h1 class="text-white my-5">Get A Booth</h1>
                    </div>
                </t>
                <section class="mt-n5">
                    <div class="container overflow-hidden">
                        <div class="o_wevent_theme_bg_base row mb64 g-0 rounded shadow-sm">
                            <t t-out="0"/>
                        </div>
                    </div>
                </section>
                <div t-if="event.is_finished" class="container d-flex flex-column flex-grow-1">
                    <span class="alert alert-info">This event is finished. It's no longer possible to book a booth.</span>
                </div>
                <div t-elif="not event_booths" class="container d-flex flex-column flex-grow-1">
                    <span class="alert alert-info">This event is not open to exhibitors registration,
                        <span t-if="request.env.user.has_group('event.group_event_manager')">
                            you can
                            <a class="text-nowrap" t-attf-href="/web#id=#{event.id}&amp;view_type=form&amp;model=event.event">
                                <i class="fa fa-gear me-1" role="img" aria-label="Configure" title="Configure event booths"/><em>Configure Booths</em>
                            </a>
                            for this event.
                        </span>
                        <span t-else="">
                            check our <a href="/event">list of future events</a>.
                        </span>
                    </span>
                </div>
            </div>
        </t>
    </template>

    <template id="event_booth_registration" name="Event Booth Registration">
        <t t-call="website_event_booth.event_booth_layout">
            <t t-if="event_booths and not event.is_finished">
                <div t-attf-class="#{'col-lg-9' if available_booth_category_ids else 'col-lg-12'} p-4">
                    <form method="post" class="form-horizontal justify-content-center mt32 js_website_submit_form o_wbooth_registration_form"
                          t-attf-action="/event/#{slug(event)}/booth/register" t-att-data-event-id="event.id">
                        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                        <div class="row mb-3 justify-content-center">
                            <t t-foreach="event.event_booth_category_ids" t-as="booth_category">
                                <t t-set="booth_category_unavailable" t-value="booth_category not in available_booth_category_ids"/>
                                <div class="col-md-4">
                                    <label t-attf-class="d-block #{'o_wbooth_category_unavailable' if booth_category_unavailable else ''}">
                                        <input type="radio" name="booth_category_id" t-att-value="booth_category.id"
                                               t-att-disabled="booth_category_unavailable"/>
                                        <div>
                                            <h5 name="booth_category_name" class="m-0 text-truncate" t-out="booth_category.name"/>
                                            <span class="img img-responsive">
                                                <div t-field="booth_category.image_1920" t-options="{'widget': 'image', 'preview_image': 'image_256', 'class': 'mt-2'}"/>
                                            </span>
                                        </div>
                                        <div t-if="booth_category_unavailable" class="o_ribbon_right bg-danger">
                                            <span class="text-nowrap">Sold Out</span>
                                        </div>
                                    </label>
                                </div>
                            </t>
                        </div>
                        <t t-if="available_booth_category_ids">
                            <div t-if="event.exhibition_map" class="row">
                                <div class="d-grid col-sm-6 offset-sm-3 mb-4">
                                    <button type="button" class="btn btn-info" data-bs-toggle="modal" data-bs-target="#mapModal">View Floor Plan</button>
                                    <div role="dialog" id="mapModal" class="modal" tabindex="-1">
                                        <div class="modal-dialog modal-lg">
                                            <div class="modal-content">
                                                <div class="modal-body">
                                                    <div t-field="event.exhibition_map" t-options="{'widget': 'image'}" class="img img-responsive"/>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-sm-2 offset-sm-1">
                                    <label for="booth_id" class="control-label">Booths</label>
                                </div>
                                <div class="col-sm-6 o_wbooth_booths"/>
                            </div>
                            <div class="row">
                                <div class="alert alert-danger col-12 o_wbooth_unavailable_booth_alert d-none" role="alert">
                                    <i class="fa fa-exclamation-triangle"/>
                                    <span>Sorry, several booths are now sold out. Please change your choices before validating again.</span>
                                </div>
                            </div>
                            <div class="row" name="booth_registration_submit">
                                <div class="col-sm-6 offset-sm-3 mt-5">
                                    <button type="submit" class="btn btn-primary btn-block fw-bold o_wbooth_registration_submit" disabled="true">
                                        <span>Book my Booths</span>
                                    </button>
                                </div>
                            </div>
                        </t>
                        <div t-else="" class="alert alert-info">
                            <span>Sorry, all the booths are sold out. <a href="/contactus">Contact Us</a> if you have any question.</span>
                        </div>
                    </form>
                </div>
                <div t-if="available_booth_category_ids" class="o_wevent_booth_category col-lg-3 hidden-sm hidden-xs p-4">
                    <t t-foreach="event.event_booth_category_ids" t-as="booth_category">
                        <div t-attf-id="o_wbooth_booth_description_#{booth_category.id}"
                             class="o_wbooth_booth_description d-none" t-field="booth_category.description"/>
                    </t>
                </div>
            </t>
        </t>
    </template>

</data></odoo>

```

## File: views\event_event_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_event_view_form" model="ir.ui.view">
        <field name="name">event.event.view.form.inherit.website.event.booth</field>
        <field name="model">event.event</field>
        <field name="inherit_id" ref="website_event.event_event_view_form"/>
        <field name="arch" type="xml">
            <field name="address_id" position="after">
                <field name="exhibition_map"/>
            </field>
            <field name="website_menu" position="after">
                <label for="booth_menu"/>
                <field name="booth_menu"/>
            </field>
        </field>
    </record>

</data></odoo>

```

## File: views\event_type_views.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo><data>

    <record id="event_type_view_form" model="ir.ui.view">
        <field name="name">event.type.view.form.inherit.website.event.booth</field>
        <field name="model">event.type</field>
        <field name="inherit_id" ref="website_event.event_type_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//span[@name='website_menu']" position='after'>
                <span>
                    <label for="booth_menu" string="Booth Menu Item"/>
                    <field name="booth_menu"/>
                </span>
            </xpath>
        </field>
    </record>

</data></odoo>

```

