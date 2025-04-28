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

from odoo import http, tools
from odoo.http import request
from odoo.addons.website_event.controllers.main import WebsiteEventController


class WebsiteEventBoothController(WebsiteEventController):

    @http.route('/event/<model("event.event"):event>/booth', type='http', auth='public', website=True, sitemap=False)
    def event_booth_main(self, event, booth_category_id=False, booth_ids=False):
        if not event.has_access('read'):
            raise Forbidden()

        booth_category_id = int(booth_category_id) if booth_category_id else False
        return request.render(
            'website_event_booth.event_booth_registration',
            self._prepare_booth_main_values(event, booth_category_id=booth_category_id, booth_ids=booth_ids)
        )

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

        return request.render(
            'website_event_booth.event_booth_registration_details',
            self._prepare_booth_contact_form_values(event, booth_ids, booth_category_id)
        )

    def _prepare_booth_contact_form_values(self, event, booth_ids, booth_category_id):
        booth_category = request.env['event.booth.category'].sudo().browse(int(booth_category_id))
        event_booths = request.env['event.booth'].sudo().browse([int(booth_id) for booth_id in booth_ids.split(',')])
        default_contact = {}

        if not request.env.user._is_public():
            default_contact = {
                'name': request.env.user.partner_id.name,
                'email': request.env.user.partner_id.email,
                'phone': request.env.user.partner_id.phone or request.env.user.partner_id.mobile,
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

        return {
            'booth_category': booth_category,
            'default_contact': default_contact,
            'event': event.sudo(),
            'event_booths': event_booths,
            'hide_sponsors': True,
            'redirect_url': werkzeug.urls.url_quote(request.httprequest.full_path),
        }

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

    def _prepare_booth_main_values(self, event, booth_category_id=False, booth_ids=False):
        event_sudo = event.sudo()
        available_booth_categories = event_sudo.event_booth_category_available_ids
        chosen_booth_category = available_booth_categories.filtered(lambda cat: cat.id == booth_category_id)
        default_booth_category = available_booth_categories[0] if available_booth_categories else request.env['event.booth.category']
        return {
            'available_booth_category_ids': available_booth_categories,
            'event': event_sudo,
            'event_booths': event_sudo.event_booth_ids,
            'hide_sponsors': True,
            'main_object': event_sudo,
            'selected_booth_category_id': (chosen_booth_category or default_booth_category).id,
            'selected_booth_ids': booth_ids if booth_category_id == chosen_booth_category.id and booth_ids else False,
        }

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
        else:
            partner = request.env.user.partner_id
        return {
            'partner_id': partner.id,
            'contact_name': kwargs.get('contact_name') or partner.name,
            'contact_email': kwargs.get('contact_email') or partner.email,
            'contact_phone': kwargs.get('contact_phone') or partner.phone or partner.mobile,
        }

    def _prepare_booth_registration_success_values(self, event_name, booth_values):
        return json.dumps({
            'success': True,
            'event_name': event_name,
            'contact': {
                'name': booth_values.get('contact_name'),
                'email': booth_values.get('contact_email'),
                'phone': booth_values.get('contact_phone'),
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
            (_('Get A Booth'), '/event/%s/booth' % self.env['ir.http']._slug(self), False, 90, 'booth')
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
access_event_booth_public,event.booth.public,event_booth.model_event_booth,base.group_public,1,0,0,0
access_event_booth_portal,event.booth.public,event_booth.model_event_booth,base.group_portal,1,0,0,0
access_event_booth_employee,event.booth.public,event_booth.model_event_booth,base.group_user,1,0,0,0
access_event_booth_category_public,event.booth.category.public,event_booth.model_event_booth_category,base.group_public,1,0,0,0

```

## File: static\src\js\booth_register.js

```javascript
/** @odoo-module **/

import { renderToElement, renderToFragment } from "@web/core/utils/render";
import publicWidget from "@web/legacy/js/public/public_widget";
import { _t } from "@web/core/l10n/translation";
import { rpc } from "@web/core/network/rpc";
import { post } from "@web/core/network/http_service";
import { redirect } from "@web/core/utils/urls";

publicWidget.registry.boothRegistration = publicWidget.Widget.extend({
    selector: '.o_wbooth_registration',
    events: {
        'change input[name="booth_category_id"]': '_onChangeBoothType',
        'change .form-check > input[type="checkbox"]': '_onChangeBooth',
        'click .o_wbooth_registration_submit': '_onSubmitBoothSelectionClick',
        'click .o_wbooth_registration_confirm': '_onConfirmRegistrationClick',
    },

    start() {
        this.eventId = parseInt(this.el.dataset.eventId);
        this.activeBoothCategoryId = false;
        this.boothCache = {};
        this.boothsFirstRendering = true;
        this.selectedBoothIds = [];
        return this._super.apply(this, arguments).then(() => {
            this.selectedBoothCategory = this.el.querySelector('input[name="booth_category_id"]:checked');
            if (this.selectedBoothCategory) {
                this.selectedBoothIds = this.el.querySelector('.o_wbooth_booths').dataset.selectedBoothIds.split(',').map(Number);
                this.activeBoothCategoryId = this.selectedBoothCategory.value;
                this._fetchBoothsAndUpdateUI();
            }
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _check_booths_availability(eventBoothIds) {
        const self = this;
        return rpc("/event/booth/check_availability", {
            event_booth_ids: eventBoothIds,
        }).then(function (result) {
            if (result.unavailable_booths.length) {
                for (const el of self.el.querySelectorAll("input[name='event_booth_ids']")) {
                    if (result.unavailable_booths.includes(parseInt(el.value))) {
                        el.closest(".form-check").classList.add("text-danger");
                    }
                }
                self.el
                    .querySelector(".o_wbooth_unavailable_booth_alert")
                    .classList.remove("d-none");
                return Promise.resolve(false);
            }
            return Promise.resolve(true);
        })
    },

    _countSelectedBooths() {
        return this.el.querySelectorAll(".form-check > input[type='checkbox']:checked").length;
    },

    _fillBooths() {
        const boothsElem = this.el.querySelector('.o_wbooth_booths');
        boothsElem.replaceChildren(renderToFragment('event_booth_checkbox_list', {
            'event_booth_ids': this.boothCache[this.activeBoothCategoryId],
            'selected_booth_ids': this.boothsFirstRendering ? this.selectedBoothIds : [],
        }));

        this.boothsFirstRendering = false;
    },

    /**
     * Check if the confirmation form is valid by testing each of its inputs
     *
     * @private
     * @param formEl
     * @return {boolean} - true if no errors else false
     */
    _isConfirmationFormValid(formEl) {
        const formErrors = [];
        for (const el of formEl.querySelectorAll(".form-control")) {
            el.classList.remove("is-invalid");
            if (!el.checkValidity()) {
                el.classList.add("is-invalid");
                formErrors.push('invalidFormInputs');
            }
        }

        this._updateErrorDisplay(formErrors);
        return formErrors.length === 0;
    },

    _showBoothCategoryDescription() {
        for (const el of this.el.querySelectorAll(".o_wbooth_booth_category_description")) {
            el.classList.add("d-none");
        }
        this.el
            .querySelector("#o_wbooth_booth_description_" + this.activeBoothCategoryId)
            .classList.remove("d-none");
    },

    /**
     * Display the errors with a custom message when confirming
     * the registration if there is any.
     *
     * @private
     * @param errors
     */
    _updateErrorDisplay(errors) {
        this.el
            .querySelector(".o_wbooth_registration_error_section")
            .classList.toggle("d-none", !errors.length);

        const errorSigninEl = this.el
            .querySelector('.o_wbooth_registration_error_signin');
        if (errorSigninEl) {
            errorSigninEl.classList.add('d-none');
        }

        let errorMessages = [];
        const errorMessageEl = this.el.querySelector(".o_wbooth_registration_error_message");

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
            if (errorSigninEl) {
                errorSigninEl.classList.remove('d-none');
            }
        }

        errorMessageEl.textContent = errorMessages.join(" ");
        errorMessageEl.dispatchEvent(new Event("change"));
    },

    _updateUiAfterBoothCategoryChange() {
        this._fillBooths();
        this._showBoothCategoryDescription();
        this._updateUiAfterBoothChange(this._countSelectedBooths());
    },

    _updateUiAfterBoothChange(boothCount) {
        const buttonEl = this.el.querySelector("button.o_wbooth_registration_submit");
        if (buttonEl) {
            buttonEl.disabled = !boothCount;
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onChangeBooth(ev) {
        ev.currentTarget.closest(".form-check").classList.remove("text-danger");
        this._updateUiAfterBoothChange(this._countSelectedBooths());
    },

    _onChangeBoothType(ev) {
        ev.preventDefault();
        this.activeBoothCategoryId = parseInt(ev.currentTarget.value);
        this._fetchBoothsAndUpdateUI();
    },

    /**
     * Load all the booths related to the activeBoothCategoryId booth category and
     * add them to a local dictionary to avoid making rpc each time the
     * user change the booth category.
     *
     * Then the selection input will be filled with the fetched booth values.
     *
     * @private
     */
    _fetchBoothsAndUpdateUI() {
        if (this.boothCache[this.activeBoothCategoryId] === undefined) {
            var self = this;
            rpc('/event/booth_category/get_available_booths', {
                event_id: this.eventId,
                booth_category_id: this.activeBoothCategoryId,
            }).then(function (result) {
                self.boothCache[self.activeBoothCategoryId] = result;
                self._updateUiAfterBoothCategoryChange();
            });
        } else {
            this._updateUiAfterBoothCategoryChange();
        }
    },

    async _onSubmitBoothSelectionClick(ev) {
        ev.preventDefault();
        const formEl = this.el.querySelector(".o_wbooth_registration_form");
        const eventBoothIds = [
            ...this.el.querySelectorAll("input[name=event_booth_ids]:checked"),
        ].map((el) => parseInt(el.value));
        if (await this._check_booths_availability(eventBoothIds)) {
            formEl.submit();
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

        ev.currentTarget.classList.add("disabled");
        ev.currentTarget.disabled = true;

        const formEl = this.el.querySelector("#o_wbooth_contact_details_form");
        if (this._isConfirmationFormValid(formEl)) {
            const formData = new FormData(formEl);
            const jsonResponse = await post(`/event/${encodeURIComponent(this.el.dataset.eventId)}/booth/confirm`, formData);
            if (jsonResponse.success) {
                this.el.querySelector('.o_wevent_booth_order_progress').remove();
                const boothCategoryId = this.el.querySelector('input[name=booth_category_id]').value;
                const boothRegistrationCompleteFormEl = renderToElement("event_booth_registration_complete", {
                    booth_category_id: boothCategoryId,
                    event_id: this.eventId,
                    event_name: jsonResponse.event_name,
                    contact: jsonResponse.contact,
                });
                formEl.insertAdjacentElement("afterend", boothRegistrationCompleteFormEl);
                formEl.remove();
            } else if (jsonResponse.redirect) {
                redirect(jsonResponse.redirect);
            } else if (jsonResponse.error) {
                this._updateErrorDisplay(jsonResponse.error);
            }
        }

        ev.currentTarget.classList.remove("disabled");
        ev.currentTarget.removeAttribute("disabled");
    },

});

export default publicWidget.registry.boothRegistration;

```

## File: static\src\xml\event_booth_registration_templates.xml

```xml
<templates>

    <t t-name="event_booth_checkbox_list">
        <div t-foreach="event_booth_ids" t-as="booth" t-key="booth_index" class="form-check">
            <input type="checkbox" name="event_booth_ids" t-attf-id="booth_#{booth.id}"
                t-att-value="booth.id" t-att-checked="selected_booth_ids.includes(booth.id) or None" class="form-check-input me-2"/>
            <label t-out="booth.name" t-attf-for="booth_#{booth.id}"/>
        </div>
    </t>

    <t t-name="event_booth_registration_complete"> 
        <div class="col-12">
            <div class="row my-3">
                <div class="col-12">
                    <h4>Booth Registration completed!</h4>
                    <h5 class="text-muted" t-out="event_name"/>
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
            <div class="d-flex flex-wrap align-items-center justify-content-between mt-3">
                <div class="d-flex align-items-center">
                    <a class="d-inline d-md-none btn btn-light me-2" t-attf-href="/event/#{slug(event)}/booth?#{keep_query('booth_category_id', 'booth_ids')}" title="Go back">
                        <i class="oi oi-chevron-left" role="img"/>
                    </a>
                    <h4 class="my-0">Get A Booth</h4>
                </div>
                <t t-call="website_event_booth.event_booth_order_progress">
                    <t t-set="step" t-value="'STEP_DETAILS_FORM'"/>
                </t>
            </div>
            <div class="oe_structure oe_empty" id="oe_structure_website_event_booth_registration_inner_1"/>
            <form method="post"
                id="o_wbooth_contact_details_form"
                t-att-data-event-id="event.id"
                class="col-12 col-lg-9 col-xl-8 js_website_submit_form">
                <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                <input type="hidden" name="booth_category_id" t-att-value="booth_category.id"/>
                <input type="hidden" name="event_booth_ids" t-att-value="event_booths.ids"/>
                <div id="o_wbooth_contact_details" class="pt-4">
                    <h5 class="h5 my-3">Contact Details</h5>
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
                </div>
                <div class="o_wbooth_registration_error_section alert alert-danger d-none mt-4" role="alert">
                    <i class="fa fa-exclamation-triangle me-2" role="img" aria-label="Error" title="Error"/>
                    <span class="o_wbooth_registration_error_message"/>
                    <a class="o_wbooth_registration_error_signin d-none"
                        t-attf-href="/web/login?redirect={{redirect_url}}">
                        Please Sign In.
                    </a>
                </div>
                <div class="row pt24 pb48">
                    <label class="col-form-label col-sm-auto d-none d-sm-inline"/>
                    <div class="col-sm">
                        <button type="submit" class="btn btn-primary o_wbooth_registration_confirm">
                            <span>Book my Booths</span>
                        </button>
                    </div>
                </div>
            </form>
            <div class="oe_structure oe_empty" id="oe_structure_website_event_booth_registration_inner_2"/>
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
            <div class="oe_structure oe_empty" id="oe_structure_website_event_booth_1"/>
            <div class="o_wbooth_registration" t-att-data-event-id="event.id">
                <section>
                    <div class="container overflow-hidden">
                        <div class="row g-0">
                            <t t-out="0"/>
                        </div>
                    </div>
                </section>
                <div t-if="event.is_finished" class="container">
                    <div class="row">
                        <div class="col-12 text-center">
                            <div t-call="website_event.event_empty_events_svg" class="my-4"/>
                            <h2>Event Finished</h2>
                            <p>It's no longer possible to book a booth.</p>
                        </div>
                    </div>
                </div>
                <div t-elif="not event_booths" class="container">
                    <div class="row">
                        <div class="col-12 text-center">
                            <div t-call="website_event.event_empty_events_svg" class="my-4"/>
                            <h2>Registration Not Open.</h2>
                            <p>This event is not open to exhibitors registration at this time.</p>
                            <p>Check our <a href="/event" title="List of Future Events" aria-label="Link to list of future events">list of future events</a>.</p>
                            <div class="o_not_editable my-3" groups="event.group_event_manager">
                                <a class="btn o_wevent_cta mb-4" target="_blank" t-attf-href="/odoo/event.event/{{event.id}}">
                                    <span class="fa fa-gear me-1"/> Configure Booths
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
             <div class="oe_structure oe_empty" id="oe_structure_website_event_booth_2"/>
        </t>
    </template>

    <template id="event_booth_registration" name="Event Booth Registration">
        <t t-call="website_event_booth.event_booth_layout">
            <t t-if="event_booths and not event.is_finished">
                <div class="d-flex flex-wrap align-items-center justify-content-between my-3">
                    <h4 class="my-0">Get A Booth</h4>
                    <t t-call="website_event_booth.event_booth_order_progress">
                        <t t-set="step" t-value="'STEP_BOOTH_SELECTION'"/>
                    </t>
                </div>
                <div class="oe_structure oe_empty" id="oe_structure_website_event_booth_inner_1"/>
                <div t-attf-class="#{'col-lg-12' if available_booth_category_ids else 'col-lg-12'}">
                    <form method="post" class="form-horizontal js_website_submit_form o_wbooth_registration_form mt-1"
                          t-attf-action="/event/#{slug(event)}/booth/register" t-att-data-event-id="event.id">
                        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                        <div class="row mb-3">
                            <h5 class="mt-0 mb-3">Choose your type of booth</h5>
                            <t t-foreach="event.event_booth_category_ids" t-as="booth_category">
                                <t t-set="booth_category_unavailable" t-value="booth_category not in available_booth_category_ids"/>
                                <div t-attf-class="col-md-6 col-lg-4 mb-4 {{ (len(event.event_booth_category_ids) &gt; 3) and 'col-xxl-3' }}">
                                    <label t-attf-class="d-block h-100 #{'o_wbooth_category_unavailable overflow-hidden' if booth_category_unavailable else ''}">
                                        <input type="radio" name="booth_category_id" t-att-value="booth_category.id" t-att-disabled="booth_category_unavailable"
                                            t-att-checked="booth_category.id == selected_booth_category_id"/>
                                        <div class="card h-100">
                                            <div t-field="booth_category.image_1920" class="card-img-top border-bottom"
                                                t-options='{"widget": "image", "qweb_img_responsive": False, "class": "img img-fluid h-100 w-100 mw-100", "style": "max-height: 208px; min-height: 208px; object-fit: cover"}'/>
                                            <div class="card-body d-flex flex-wrap w-100 gap-2 justify-content-between flex-grow-0 pb-0">
                                                <h5 name="booth_category_name" class="card-title my-0" t-out="booth_category.name"/>
                                                <span class="booth_category_price"></span>
                                            </div>
                                             <div class="w-100 small" t-attf-id="o_wbooth_booth_description_#{booth_category.id}" t-field="booth_category.description"/>
                                        </div>
                                        <div t-if="booth_category_unavailable" class="o_ribbon_right text-bg-danger">
                                            <span class="text-nowrap">Sold Out</span>
                                        </div>
                                    </label>
                                </div>
                            </t>
                        </div>
                        <t t-if="available_booth_category_ids">
                            <div class="row">
                                <div class="d-flex flex-wrap align-items-center gap-2 mb-3">
                                    <h5 class="my-0">Location</h5>
                                    <div t-if="event.exhibition_map" class="ms-2 small">
                                        <a class="text-decoration-none text-center" href="#" data-bs-toggle="modal" data-bs-target="#mapModal"><i class="fa fa-map-o me-1"/>View Plan</a>
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
                                <div class="col-12 o_wbooth_booths d-flex flex-wrap align-items-center gap-2" t-att-data-selected-booth-ids="selected_booth_ids or ''"/>
                                <div class="row">
                                    <div class="alert alert-danger col-12 o_wbooth_unavailable_booth_alert d-none" role="alert">
                                        <i class="fa fa-exclamation-triangle"/>
                                        <span>Sorry, several booths are now sold out. Please change your choices before validating again.</span>
                                    </div>
                                </div>
                                <div class="pt24 pb48" name="booth_registration_submit">
                                    <div class="d-flex align-items-center justify-content-end gap-2">
                                        <button type="submit" class="o_wbooth_registration_submit btn btn-primary btn-block" disabled="true">
                                            <span>Book my Booth<small>(s)</small></span>
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </t>
                        <div t-else="" class="alert alert-info">
                            <span>Sorry, all the booths are sold out. <a class="alert-link" href="/contactus">Contact Us</a> if you have any question.</span>
                        </div>
                    </form>
                </div>
            </t>
        </t>
    </template>

    <template id="event_booth_order_progress">
        <ul class="o_wevent_booth_order_progress d-none d-md-block list-unstyled px-3 py-2 text-bg-light rounded m-0">
            <li t-attf-class="position-relative float-start m-0 text-center">
                <a t-if="step!='STEP_BOOTH_SELECTION'" class="d-inline-flex align-items-center text-decoration-none" t-attf-href="/event/#{slug(event)}/booth?#{keep_query('booth_category_id', 'booth_ids')}">
                    <span>Booth Selection</span><span class="fa fa-angle-right d-inline-block align-middle mx-2 mx-lg-3 opacity-75"/>
                </a>
                <span t-else="" class="d-inline-flex align-items-center text-decoration-none text-reset">
                    <span>Booth Selection</span><span class="fa fa-angle-right d-inline-block align-middle mx-2 mx-lg-3 opacity-75"/>
                </span>
            </li>
            <li t-attf-class="d-inline-flex align-items-center position-relative float-start m-0 text-center #{'' if step=='STEP_DETAILS_FORM' else 'opacity-75'}">
                Contact Details<span class="fa fa-angle-right d-inline-block align-middle mx-2 mx-lg-3 opacity-75"/>
            </li>
            <li class="position-relative float-start m-0 text-center opacity-75">
                <span>Confirmed</span>
            </li>
        </ul>
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

