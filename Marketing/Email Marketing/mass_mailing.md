# Odoo Module: mass_mailing

Category: Marketing/Email Marketing

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Email Marketing',
    'summary': 'Design, send and track emails',
    'version': '2.5',
    'sequence': 60,
    'website': 'https://www.odoo.com/app/email-marketing',
    'category': 'Marketing/Email Marketing',
    'depends': [
        'contacts',
        'mail',
        'utm',
        'link_tracker',
        'web_editor',
        'web_kanban_gauge',
        'social_media',
        'web_tour',
        'digest',
    ],
    'data': [
        'security/res_groups_data.xml',
        'security/ir.model.access.csv',
        'data/digest_data.xml',
        'data/ir_attachment_data.xml',
        'data/ir_config_parameter_data.xml',
        'data/ir_cron_data.xml',
        'data/ir_module_data.xml',
        'data/mailing_data_templates.xml',
        'data/mailing_list_data.xml',
        'data/res_users_data.xml',
        'wizard/mail_compose_message_views.xml',
        'wizard/mailing_contact_import_views.xml',
        'wizard/mailing_contact_to_list_views.xml',
        'wizard/mailing_list_merge_views.xml',
        'wizard/mailing_mailing_test_views.xml',
        'wizard/mailing_mailing_schedule_date_views.xml',
        'report/mailing_trace_report_views.xml',
        'views/mailing_filter_views.xml',
        'views/mailing_trace_views.xml',
        'views/link_tracker_views.xml',
        'views/mailing_contact_views.xml',
        'views/mailing_contact_subscription_views.xml',
        'views/mailing_list_views.xml',
        'views/mailing_mailing_views.xml',
        'views/res_config_settings_views.xml',
        'views/utm_campaign_views.xml',
        'views/mailing_menus.xml',
        'views/assets.xml',
        'views/mailing_templates_portal_layouts.xml',
        'views/mailing_templates_portal_management.xml',
        'views/mailing_templates_portal_unsubscribe.xml',
        'views/themes_templates.xml',
        'views/snippets_themes.xml',
        'views/snippets/s_alert.xml',
        'views/snippets/s_blockquote.xml',
        'views/snippets/s_call_to_action.xml',
        'views/snippets/s_coupon_code.xml',
        'views/snippets/s_cover.xml',
        'views/snippets/s_color_blocks_2.xml',
        'views/snippets/s_company_team.xml',
        'views/snippets/s_comparisons.xml',
        'views/snippets/s_event.xml',
        'views/snippets/s_features.xml',
        'views/snippets/s_features_grid.xml',
        'views/snippets/s_hr.xml',
        'views/snippets/s_image_text.xml',
        'views/snippets/s_masonry_block.xml',
        'views/snippets/s_media_list.xml',
        'views/snippets/s_numbers.xml',
        'views/snippets/s_picture.xml',
        'views/snippets/s_product_list.xml',
        'views/snippets/s_rating.xml',
        'views/snippets/s_references.xml',
        'views/snippets/s_showcase.xml',
        'views/snippets/s_text_block.xml',
        'views/snippets/s_text_highlight.xml',
        'views/snippets/s_text_image.xml',
        'views/snippets/s_three_columns.xml',
        'views/snippets/s_title.xml',
    ],
    'demo': [
        'data/mass_mailing_demo.xml',
    ],
    'application': True,
    'assets': {
        'mass_mailing.mailing_assets': [
            'mass_mailing/static/src/scss/mailing_portal.scss',
            'mass_mailing/static/src/js/mailing_portal.js',
        ],
        'web.assets_backend': [
            'mass_mailing/static/src/scss/mailing_filter_widget.scss',
            'mass_mailing/static/src/scss/mass_mailing.scss',
            'mass_mailing/static/src/scss/mass_mailing_mobile.scss',
            'mass_mailing/static/src/scss/mass_mailing_mobile_preview.scss',
            'mass_mailing/static/src/css/email_template.css',
            'mass_mailing/static/src/views/*.js',
            'mass_mailing/static/src/js/mailing_m2o_filter.js',
            'mass_mailing/static/src/js/mass_mailing.js',
            'mass_mailing/static/src/js/mass_mailing_design_constants.js',
            'mass_mailing/static/src/js/mass_mailing_mobile_preview.js',
            'mass_mailing/static/src/js/mass_mailing_html_field.js',
            'mass_mailing/static/src/js/mailing_mailing_view_form_full_width.js',
            'mass_mailing/static/src/xml/mailing_filter_widget.xml',
            'mass_mailing/static/src/xml/mass_mailing.xml',
            'mass_mailing/static/src/views/*.xml',
        ],
        'mass_mailing.assets_mail_themes': [
            'mass_mailing/static/src/scss/themes/**/*',
        ],
        'mass_mailing.assets_mail_themes_edition': [
            ('include', 'web._assets_helpers'),
            'web/static/src/scss/pre_variables.scss',
            'web/static/lib/bootstrap/scss/_variables.scss',
            'mass_mailing/static/src/scss/mass_mailing.ui.scss',
        ],
        'web_editor.assets_wysiwyg': [
            'mass_mailing/static/src/js/snippets.editor.js',
            'mass_mailing/static/src/js/wysiwyg.js',
            'mass_mailing/static/src/xml/mass_mailing.editor.xml',
            'mass_mailing/static/src/scss/mass_mailing.wysiwyg.scss',
        ],
        'web.assets_common': [
            'mass_mailing/static/src/js/tours/**/*',
        ],
        'web.assets_frontend': [
            'mass_mailing/static/src/js/tours/**/*',
        ],
        'web.qunit_suite_tests': [
            'mass_mailing/static/tests/mass_mailing_favourite_filter_tests.js',
            'mass_mailing/static/src/js/mass_mailing_snippets.js',
            'mass_mailing/static/src/snippets/s_media_list/options.js',
            'mass_mailing/static/src/snippets/s_showcase/options.js',
            'mass_mailing/static/src/snippets/s_rating/options.js',
            'mass_mailing/static/tests/mass_mailing_html_tests.js',
            'mass_mailing/static/tests/mailing_mailing_view_form_tests.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import urllib.parse

from odoo import _, exceptions, http, tools
from odoo.http import request, Response
from odoo.tools import consteq
from lxml import etree
from werkzeug.exceptions import BadRequest, NotFound


class MassMailController(http.Controller):

    def _valid_unsubscribe_token(self, mailing_id, res_id, email, token):
        if not (mailing_id and res_id and email and token):
            return False
        mailing = request.env['mailing.mailing'].sudo().browse(mailing_id)
        return consteq(mailing._unsubscribe_token(res_id, email), token)

    def _log_blacklist_action(self, blacklist_entry, mailing_id, description):
        mailing = request.env['mailing.mailing'].sudo().browse(mailing_id)
        model_display = mailing.mailing_model_id.display_name
        blacklist_entry._message_log(body=description + " ({})".format(model_display))

    # ------------------------------------------------------------
    # SUBSCRIPTION MANAGEMENT
    # ------------------------------------------------------------

    # csrf is disabled here because it will be called by the MUA with unpredictable session at that time
    @http.route(['/mail/mailing/<int:mailing_id>/unsubscribe_oneclick'], type='http', website=True, auth='public',
                methods=["POST"], csrf=False)
    def mailing_unsubscribe_oneclick(self, mailing_id, email=None, res_id=None, token="", **post):
        self.mailing(mailing_id, email=email, res_id=res_id, token=token, **post)
        return Response(status=200)

    @http.route('/mailing/<int:mailing_id>/confirm_unsubscribe', type='http', website=True, auth='public')
    def mailing_confirm_unsubscribe(self, mailing_id, email=None, res_id=None, token="", **post):
        mailing = request.env['mailing.mailing'].sudo().browse(mailing_id)
        # Check access (note that this will also raise AccessDenied if the mailing does not exist)
        if not self._valid_unsubscribe_token(mailing_id, res_id, email, str(token)):
            raise exceptions.AccessDenied()

        unsubscribed_str = _("Are you sure you want to unsubscribe from our mailing list?")
        # Display list name if list is public
        if mailing.mailing_model_real == 'mailing.contact':
            unsubscribed_lists = ', '.join(mailing_list.name for mailing_list in mailing.contact_list_ids if mailing_list.is_public)
            if unsubscribed_lists:
                unsubscribed_str = _(
                    'Are you sure you want to unsubscribe from the mailing list "%(unsubscribed_lists)s"?',
                    unsubscribed_lists=unsubscribed_lists
                )
        unsubscribe_btn = _("Unsubscribe")

        template = etree.fromstring("""
            <t t-call="mass_mailing.layout">
                <div class="container o_unsubscribe_form">
                    <form action="/mailing/confirm_unsubscribe" method="POST" class="col-lg-6 offset-lg-3 mt-4">
                        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                        <input type="hidden" name="email" t-att-value="email"/>
                        <input type="hidden" name="mailing_id" t-att-value="mailing_id"/>
                        <input type="hidden" name="res_id" t-att-value="res_id"/>
                        <input type="hidden" name="token" t-att-value="token"/>
                        <div id="info_state"  class="alert alert-success">
                            <div class="text-center">
                                <p t-out="unsubscribed_str"/>
                                <button type="submit" class="btn btn-primary" t-out="unsubscribe_btn"/>
                            </div>
                        </div>
                    </form>
                </div>
            </t>
        """)
        return request.env['ir.qweb']._render(template, {
            'main_object': mailing,
            'token': token,
            'email': email,
            'mailing_id': mailing_id,
            'unsubscribed_str': unsubscribed_str,
            'res_id': res_id,
            'unsubscribe_btn': unsubscribe_btn,
        })

    # kept for backwards compatibility, must eventually be merged with mailing/<mailing_id>/unsubscribe
    @http.route('/mailing/confirm_unsubscribe', type='http', website=True, auth='public', methods=['POST'])
    def mailing_confirm_unsubscribe_post(self, mailing_id, email=None, res_id=None, token="", **post):
        url_params = urllib.parse.urlencode({'email': email, 'res_id': res_id, 'token': token})
        url = f'/mail/mailing/{int(mailing_id)}/unsubscribe?{url_params}'
        return request.redirect(url)

    # todo: merge this route with /mail/mailing/confirm_unsubscribe on next minor version
    @http.route(['/mail/mailing/<int:mailing_id>/unsubscribe'], type='http', website=True, auth='public')
    def mailing(self, mailing_id, email=None, res_id=None, token="", **post):
        mailing = request.env['mailing.mailing'].sudo().browse(mailing_id)
        if mailing.exists():
            res_id = res_id and int(res_id)
            if not self._valid_unsubscribe_token(mailing_id, res_id, email, str(token)):
                raise exceptions.AccessDenied()

            if mailing.mailing_model_real == 'mailing.contact':
                # Unsubscribe directly + Let the user choose their subscriptions
                mailing.update_opt_out(email, mailing.contact_list_ids.ids, True)

                contacts = request.env['mailing.contact'].sudo().search([('email_normalized', '=', tools.email_normalize(email))])
                subscription_list_ids = contacts.mapped('subscription_list_ids')
                # In many user are found : if user is opt_out on the list with contact_id 1 but not with contact_id 2,
                # assume that the user is not opt_out on both
                # TODO DBE Fixme : Optimise the following to get real opt_out and opt_in
                opt_out_list_ids = subscription_list_ids.filtered(lambda rel: rel.opt_out).mapped('list_id')
                opt_in_list_ids = subscription_list_ids.filtered(lambda rel: not rel.opt_out).mapped('list_id')
                opt_out_list_ids = set([list.id for list in opt_out_list_ids if list not in opt_in_list_ids])

                unique_list_ids = set([list.list_id.id for list in subscription_list_ids])
                list_ids = request.env['mailing.list'].sudo().browse(unique_list_ids).filtered('active')
                unsubscribed_list = ', '.join(str(list.name) for list in mailing.contact_list_ids if list.is_public)
                return request.render('mass_mailing.page_unsubscribe', {
                    'contacts': contacts,
                    'list_ids': list_ids,
                    'opt_out_list_ids': opt_out_list_ids,
                    'unsubscribed_list': unsubscribed_list,
                    'email': email,
                    'mailing_id': mailing_id,
                    'res_id': res_id,
                    'show_blacklist_button': request.env['ir.config_parameter'].sudo().get_param('mass_mailing.show_blacklist_buttons'),
                })
            else:
                opt_in_lists = request.env['mailing.contact.subscription'].sudo().search([
                    ('contact_id.email_normalized', '=', email),
                    ('opt_out', '=', False)
                ]).mapped('list_id').filtered('active')
                blacklist_rec = request.env['mail.blacklist'].sudo()._add(email)
                self._log_blacklist_action(
                    blacklist_rec, mailing_id,
                    _("""Requested blacklisting via unsubscribe link."""))
                return request.render('mass_mailing.page_unsubscribed', {
                    'email': email,
                    'mailing_id': mailing_id,
                    'res_id': res_id,
                    'list_ids': opt_in_lists,
                    'show_blacklist_button': request.env['ir.config_parameter'].sudo().get_param(
                        'mass_mailing.show_blacklist_buttons'),
                })
        return request.redirect('/web')

    @http.route('/mail/mailing/unsubscribe', type='json', auth='public')
    def unsubscribe(self, mailing_id, opt_in_ids, opt_out_ids, email, res_id, token):
        mailing = request.env['mailing.mailing'].sudo().browse(mailing_id)
        if mailing.exists():
            if not self._valid_unsubscribe_token(mailing_id, res_id, email, token):
                return 'unauthorized'
            mailing.update_opt_out(email, opt_in_ids, False)
            mailing.update_opt_out(email, opt_out_ids, True)
            return True
        return 'error'

    @http.route('/mailing/feedback', type='json', auth='public')
    def send_feedback(self, mailing_id, res_id, email, feedback, token):
        mailing = request.env['mailing.mailing'].sudo().browse(mailing_id)
        if mailing.exists() and email:
            if not self._valid_unsubscribe_token(mailing_id, res_id, email, token):
                return 'unauthorized'
            model = request.env[mailing.mailing_model_real]
            records = model.sudo().search([('email_normalized', '=', tools.email_normalize(email))])
            for record in records:
                record.sudo().message_post(body=_("Feedback from %(email)s: %(feedback)s", email=email, feedback=feedback))
            return bool(records)
        return 'error'

    @http.route(['/unsubscribe_from_list'], type='http', website=True, multilang=False, auth='public', sitemap=False)
    def unsubscribe_placeholder_link(self, **post):
        """Dummy route so placeholder is not prefixed by language, MUST have multilang=False"""
        raise NotFound()

    # ------------------------------------------------------------
    # TRACKING
    # ------------------------------------------------------------

    @http.route('/mail/track/<int:mail_id>/<string:token>/blank.gif', type='http', auth='public')
    def track_mail_open(self, mail_id, token, **post):
        """ Email tracking. """
        if not consteq(token, tools.hmac(request.env(su=True), 'mass_mailing-mail_mail-open', mail_id)):
            raise BadRequest()

        request.env['mailing.trace'].sudo().set_opened(domain=[('mail_mail_id_int', 'in', [mail_id])])
        response = Response()
        response.mimetype = 'image/gif'
        response.data = base64.b64decode(b'R0lGODlhAQABAIAAANvf7wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==')

        return response

    @http.route('/r/<string:code>/m/<int:mailing_trace_id>', type='http', auth="public")
    def full_url_redirect(self, code, mailing_trace_id, **post):
        # don't assume geoip is set, it is part of the website module
        # which mass_mailing doesn't depend on
        country_code = request.geoip.get('country_code')

        request.env['link.tracker.click'].sudo().add_click(
            code,
            ip=request.httprequest.remote_addr,
            country_code=country_code,
            mailing_trace_id=mailing_trace_id
        )
        redirect_url = request.env['link.tracker'].get_url_from_code(code)
        if not redirect_url:
            raise NotFound()
        return request.redirect(redirect_url, code=301, local=False)

    # ------------------------------------------------------------
    # MAILING MANAGEMENT
    # ------------------------------------------------------------

    @http.route('/mailing/report/unsubscribe', type='http', website=True, auth='public')
    def turn_off_mailing_reports(self, token, user_id):
        if not token or not user_id:
            raise NotFound()
        user_id = int(user_id)
        correct_token = consteq(token, request.env['mailing.mailing']._get_unsubscribe_token(user_id))
        user = request.env['res.users'].sudo().browse(user_id)
        if correct_token and user.has_group('mass_mailing.group_mass_mailing_user'):
            request.env['ir.config_parameter'].sudo().set_param('mass_mailing.mass_mailing_reports', False)
            if user.has_group('base.group_system'):
                menu_id = request.env.ref('mass_mailing.menu_mass_mailing_global_settings').id
                return request.render('mass_mailing.mailing_report_deactivated', {'menu_id': menu_id})
            return request.render('mass_mailing.mailing_report_deactivated')
        raise NotFound()

    @http.route(['/mailing/<int:mailing_id>/view'], type='http', website=True, auth='public')
    def view(self, mailing_id, email=None, res_id=None, token=""):
        mailing = request.env['mailing.mailing'].sudo().browse(mailing_id)
        if mailing.exists():
            res_id = int(res_id) if res_id else False
            if not self._valid_unsubscribe_token(mailing_id, res_id, email, str(token)) and not request.env.user.has_group('mass_mailing.group_mass_mailing_user'):
                raise exceptions.AccessDenied()

            html_markupsafe = mailing._render_field('body_html', [res_id])[res_id]
            # Update generic URLs (without parameters) to final ones
            html_markupsafe = html_markupsafe.replace('/unsubscribe_from_list',
                                                      mailing._get_unsubscribe_url(email, res_id))

            return request.render('mass_mailing.view', {
                    'body': html_markupsafe,
                })

        return request.redirect('/web')

    # ------------------------------------------------------------
    # BLACKLIST
    # ------------------------------------------------------------

    @http.route('/mailing/blacklist/check', type='json', auth='public')
    def blacklist_check(self, mailing_id, res_id, email, token):
        if not self._valid_unsubscribe_token(mailing_id, res_id, email, token):
            return 'unauthorized'
        if email:
            record = request.env['mail.blacklist'].sudo().with_context(active_test=False).search([('email', '=', tools.email_normalize(email))])
            if record['active']:
                return True
            return False
        return 'error'

    @http.route('/mailing/blacklist/add', type='json', auth='public')
    def blacklist_add(self, mailing_id, res_id, email, token):
        if not self._valid_unsubscribe_token(mailing_id, res_id, email, token):
            return 'unauthorized'
        if email:
            blacklist_rec = request.env['mail.blacklist'].sudo()._add(email)
            self._log_blacklist_action(
                blacklist_rec, mailing_id,
                _("""Requested blacklisting via unsubscription page."""))
            return True
        return 'error'

    @http.route('/mailing/blacklist/remove', type='json', auth='public')
    def blacklist_remove(self, mailing_id, res_id, email, token):
        if not self._valid_unsubscribe_token(mailing_id, res_id, email, token):
            return 'unauthorized'
        if email:
            blacklist_rec = request.env['mail.blacklist'].sudo()._remove(email)
            self._log_blacklist_action(
                blacklist_rec, mailing_id,
                _("""Requested de-blacklisting via unsubscription page."""))
            return True
        return 'error'

    # ------------------------------------------------------------
    # MISCELLANEOUS
    # ------------------------------------------------------------

    @http.route('/mailing/get_preview_assets', type='json', auth='user')
    def get_mobile_preview_styling(self):
        """ This route allows a rpc call to get the styling needed for email template conversion.
        We do this to avoid duplicating the template."""
        if not request.env.user.has_group('mass_mailing.group_mass_mailing_user'):
            raise NotFound
        return request.env['ir.qweb']._render('mass_mailing.iframe_css_assets_edit')

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\digest_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="digest_mail_main" inherit_id="digest.digest_mail_main">
            <xpath expr="//div[hasclass('by_odoo')]/t[last()]" position="after">
                <t t-if="mailing_report_token">
                    –
                    <a t-attf-href="/mailing/report/unsubscribe?token=#{mailing_report_token}&amp;user_id=#{user_id}"
                       target="_blank" style="text-decoration: none;">
                        <span style="color: #8f8f8f;">Turn off Mailing Reports</span>
                    </a>
                </t>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: data\ir_attachment_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <!-- Snippets' Default Images -->
        <record id="mass_mailing.s_cover_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_cover_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_cover.jpg</field>
        </record>
        <record id="mass_mailing.s_media_list_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_media_list_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_media_list_1.jpg</field>
        </record>
        <record id="mass_mailing.s_media_list_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_media_list_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_media_list_2.jpg</field>
        </record>
        <record id="mass_mailing.s_media_list_default_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_media_list_default_image_3.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_media_list_3.jpg</field>
        </record>
        <record id="mass_mailing.s_company_team_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_default_image_1.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_team_member_1.png</field>
        </record>
        <record id="mass_mailing.s_company_team_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_default_image_2.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_team_member_2.png</field>
        </record>
        <record id="mass_mailing.s_company_team_default_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_default_image_3.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_team_member_3.png</field>
        </record>
        <record id="mass_mailing.s_company_team_default_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_default_image_4.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_team_member_4.png</field>
        </record>
        <record id="mass_mailing.s_reference_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_default_image_1.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_references_1.png</field>
        </record>
        <record id="mass_mailing.s_reference_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_default_image_2.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_references_2.png</field>
        </record>
        <record id="mass_mailing.s_reference_default_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_default_image_3.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_references_3.png</field>
        </record>
        <record id="mass_mailing.s_reference_default_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_default_image_4.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_references_4.png</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_product_1.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_product_2.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_3.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_product_3.jpg</field>
        </record>
        <record id="mass_mailing.s_blockquote_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_blockquote_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_team_member_2.png</field>
        </record>
        <record id="mass_mailing.s_image_text_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_image_text_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_image_text.jpg</field>
        </record>
        <record id="mass_mailing.s_event_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_event_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_event_1.jpg</field>
        </record>
        <record id="mass_mailing.s_event_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_event_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_event_2.jpg</field>
        </record>
        <record id="mass_mailing.s_masonry_block_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_masonry_block_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_masonry_block_1.jpg</field>
        </record>
        <record id="mass_mailing.s_masonry_block_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_masonry_block_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_masonry_block_2.jpg</field>
        </record>
        <record id="mass_mailing.s_picture_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_picture_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_picture.jpg</field>
        </record>
        <record id="mass_mailing.s_text_image_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_text_image_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_text_image.jpg</field>
        </record>
        <record id="mass_mailing.s_three_columns_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_three_columns_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_three_cols_1.jpg</field>
        </record>
        <record id="mass_mailing.s_three_columns_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_three_columns_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_three_cols_2.jpg</field>
        </record>
        <record id="mass_mailing.s_three_columns_default_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_three_columns_default_image_3.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/theme_default/s_default_image_three_cols_3.jpg</field>
        </record>
    </data>
</odoo>

```

## File: data\ir_config_parameter_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Enable Mass Mailing Reports -->
        <function model="ir.config_parameter"
                  name="set_param"
                  eval="('mass_mailing.mass_mailing_reports', 'True')"/>
    </data>
</odoo>

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Cron that processes the mass mailing queue -->
        <record id="ir_cron_mass_mailing_queue" model="ir.cron">
            <field name="name">Mail Marketing: Process queue</field>
            <field name="model_id" ref="model_mailing_mailing"/>
            <field name="state">code</field>
            <field name="code">model._process_mass_mailing_queue()</field>
            <field name="user_id" ref="base.user_root" />
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
            <field eval="False" name="doall" />
        </record>
        <!-- Cron that processes the a/b testing -->
        <record id="ir_cron_mass_mailing_ab_testing" model="ir.cron">
            <field name="name">Mail Marketing: A/B Testing</field>
            <field name="model_id" ref="model_utm_campaign"/>
            <field name="state">code</field>
            <field name="code">model._cron_process_mass_mailing_ab_testing()</field>
            <field name="user_id" ref="base.user_root"/>
            <field name="active" eval="False"/>
            <field name="interval_number">1</field>
            <field name="interval_type">days</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\ir_module_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record model="ir.module.category" id="base.module_category_marketing_email_marketing">
            <field name="sequence">19</field>
            <field name="description">Helps you manage your mass mailing to design
    professional emails and reuse templates.</field>
        </record>
    </data>
</odoo>

```

## File: data\mailing_data_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">
    <!--
    Reference: https://litmus.com/community/learning/24-how-to-code-a-responsive-email-from-scratch
    https://www.campaignmonitor.com/css/link-element/link-in-head/
    -->
    <template id="mass_mailing_mail_layout">
        &lt;!DOCTYPE html&gt;
        <html xmlns="http://www.w3.org/1999/xhtml" xmlns:v="urn:schemas-microsoft-com:vml" xmlns:o="urn:schemas-microsoft-com:office:office">
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
                <meta name="format-detection" content="telephone=no"/>
                <meta name="viewport" content="width=device-width; initial-scale=1.0; maximum-scale=1.0; user-scalable=no;"/>
                <meta http-equiv="X-UA-Compatible" content="IE=9; IE=8; IE=7; IE=EDGE" />

                <t t-call="mass_mailing.mass_mailing_mail_style"/>

                <!--
                    Prevent Outlook from distorting images with DPI scaling (see
                    https://www.htmlemailcheck.com/knowledge-base/dpi-scaling-in-outlook-2007-2013/):
                -->
                <!--[if mso]>
                    <xml>
                        <o:OfficeDocumentSettings>
                            <o:AllowPNG/>
                            <o:PixelsPerInch>96</o:PixelsPerInch>
                        </o:OfficeDocumentSettings>
                    </xml>
                <![endif]-->
            </head>
            <body>
                <t t-out="body"/>
            </body>
        </html>
    </template>

    <template id="mass_mailing_mail_style">
        <style>
            .o_layout * {
                box-sizing: border-box !important;
            }
            .o_layout :not(.fa) {
                font-family: Arial,Helvetica Neue,Helvetica,sans-serif;
            }
            /* Remove space around the email design. */
                html,
                body {
                    margin: 0 auto !important;
                    padding: 0 !important;
                    height: 100% !important;
                    width: 100% !important;
                }

                /* Stop Outlook resizing small text. */
                * {
                    -ms-text-size-adjust: 100%;
                }

                /* Stop Outlook from adding extra spacing to tables. */
                table,
                td {
                    mso-table-lspace: 0pt !important;
                    mso-table-rspace: 0pt !important;
                }

                /* Use a better rendering method when resizing images in Outlook IE. */
                img {
                    -ms-interpolation-mode:bicubic;
                }

                /* Prevent Windows 10 Mail from underlining links. Styles for underlined links should be inline. */
                a {
                    text-decoration: none;
                }
            @media screen and (max-width: 768px) {
                .o_mail_snippet_general .container {
                    width: 100% !important;
                }
                .s_header_social table td, .s_header_text_social table td, .s_header_logo table td,
                .o_mail_block_footer_social td,
                .s_showcase td,
                .s_mail_product_list td,
                .s_references td {
                    text-align: center !important;
                }
            }
            @media screen and (max-width: 1135px) {
                .o_stacking_wrapper {
                    width: 100% !important;
                    height: unset !important;
                }
                td {
                    max-width: inherit !important;
                }
                img:only-child:not(.img-fluid) {
                    object-fit: cover;
                    min-width: 100% !important;
                }
                .o_desktop_h100 {
                    height: unset !important;
                }
            }
        </style>
    </template>
</data>

<data noupdate="0">
    <template id="ab_testing_description" name="Mass Mailing: A/B Test Description">
        <div name="ab_testing_description" class="mb-2">
            <t t-if="mailing.ab_testing_completed">
                <p t-if="mailing.ab_testing_pc == 100">
                    This <t t-out="mailing.mailing_type_description"/> is the winner of the A/B testing campaign and has been sent to all remaining recipients.
                </p>
                <p t-else="">The winner has already been sent. Use <b>Compare Version</b> to get an overview of this A/B testing campaign.</p>
            </t>
            <t t-elif="mailing.ab_testing_mailings_count >= 2">
                <p>
                    A sample of <b><t t-out="mailing.ab_testing_pc"/>% of recipients</b> will receive this <b><t t-out="mailing.mailing_type_description"/></b>, and <t t-out="other_ab_testing_pc"/>% receive one of the
                    <b><t t-out="mailing.ab_testing_mailings_count - 1"/> other versions</b> from the same campaign.
                </p>
                <p>
		    <t t-if="mailing.ab_testing_winner_selection == 'manual'">Don't forget to send your preferred version</t>
		    <t t-elif="not mailing.ab_testing_schedule_datetime">Since the date and time for this test has not been scheduled, don't forget to manually send your preferred version.</t>
                    <t t-else="">
                        Then on <b><t t-out="mailing.ab_testing_schedule_datetime.strftime('%b %d, %Y')"/></b> the <t t-out="mailing.mailing_type_description"/> having the <b><t t-out="ab_testing_winner_selection_description"/></b> will be sent
                    </t> to the remaining <t t-out="remaining_ab_testing_pc"/>% of recipients.
                </p>
            </t>
            <t t-else="">
                <p>
                    A sample of <b><t t-out="mailing.ab_testing_pc"/>% of recipients</b> will receive this <b><t t-out="mailing.mailing_type_description"/></b>.
                </p>
                <p>Try different variations in the campaign to compare their <t t-out="ab_testing_winner_selection_description"/>.</p>
                <p>
                    <t t-if="mailing.ab_testing_winner_selection != 'manual'">Once the best version is identified, we will send the best one to the remaining recipients.</t>
                    <t t-else="">
                        The actual <t t-out="mailing.mailing_type_description"/> will be sent to the remaining recipients.
                    </t>
                </p>
            </t>
        </div>
    </template>

    <template id="mass_mailing.mass_mailing_kpi_link_trackers" name="Marketing: mailing link trackers statistic">
        <div class="global_layout" t-if="link_trackers">
            <table bgcolor="#ffffff" cellspacing="0" cellpadding="0" width="650" align="center" border="0" style="width: 100%; max-width: 650px;">
                <tr>
                    <td style="width: 100%;">
                        <table cellspacing="0" cellpadding="0" border="0" width="580" align="center" style="width:100%; max-width:580px;">
                            <tr>
                                <td align="left">
                                    <span style="color:#282f33; font-size: 15px; font-weight: bold; line-height: 30px">
                                        <t t-esc="'Click Rate Report on %i %s Sent' % (object.expected, mailing_type)"/>
                                    </span>
                                </td>
                            </tr>
                        </table>
                    </td>
                </tr>
                <tr>
                    <td style="margin: 0; padding:0;">
                        <table cellspacing="0" cellpadding="0" border="0" width="580" align="center" style="width:100%; max-width:580px;border-collapse: collapse;">
                            <tr style="color: #875a7b; font-size: 16px; font-weight: 500;">
                                <td style="width: 70%;padding: 10px 0; text-align: center; border: 1px solid #e7e7e7;">Button Label</td>
                                <td style="width: 30%;padding: 10px 0; text-align: center; border: 1px solid #e7e7e7;">%Click (Total)</td>
                            </tr>
                            <tr t-foreach="link_trackers" t-as="link_tracker" style="color: #888888; font-size: 15px; font-weight: 300;">
                                <td style="width: 70%;padding: 10px 0; border: 1px solid #e7e7e7;">
                                    <a t-att-href="link_tracker.absolute_url" target="_blank" style="color: #56b3b5; text-decoration: none;" t-esc="link_tracker.label or link_tracker.url"/>
                                </td>
                                <td style="width: 30%;padding: 10px 0; text-align: center;  border: 1px solid #e7e7e7;">
                                    <t t-esc="int(link_tracker.count * 100 / object.sent) if object.sent else 0"/>% (<t t-esc="link_tracker.count"/>)
                                </td>
                            </tr>
                        </table>
                    </td>
                </tr>
            </table>
        </div>
    </template>
</data>
</odoo>

```

## File: data\mailing_list_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="mailing_list_data" model="mailing.list">
            <field name="name">Newsletter</field>
        </record>
        <record id="mass_mailing_contact_0" model="mailing.contact">
            <field name="name" model="res.users" eval="obj().env.ref('base.user_admin').name"/>
            <field name="email" model="res.users" eval="obj().env.ref('base.user_admin').email"/>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_data')])]"/>
        </record>
    </data>
</odoo>

```

## File: data\mass_mailing_data.xml

```xml

```

## File: data\mass_mailing_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="mass_mail_attach_1" model="ir.attachment">
            <field name="datas">bWlncmF0aW9uIHRlc3Q=</field>
            <field name="name">SampleDoc.doc</field>
        </record>

        <!-- Create Extra Mailing List for Demo -->
        <record id="mailing_list_1" model="mailing.list">
            <field name="name">Imported Contacts</field>
        </record>

        <!-- Create Contacts -->
        <record id="mass_mail_contact_1" model="mailing.contact">
            <field name="name">Aristide Antario</field>
            <field name="email">alexandre.antario@example.com</field>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_data')])]"/>
        </record>
        <record id="mass_mail_contact_2" model="mailing.contact">
            <field name="name">Beverly Bridge</field>
            <field name="email">beverly.bridge@example.com</field>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_data')])]"/>
        </record>
        <record id="mass_mail_contact_3" model="mailing.contact">
            <field name="name">Carol Cartridge</field>
            <field name="email">carol.cartridge@example.com</field>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_data'),ref('mass_mailing.mailing_list_1')])]"/>
        </record>
        <record id="mass_mail_contact_4" model="mailing.contact">
            <field name="name">David Dawson</field>
            <field name="email">david.dawson@example.com</field>
        </record>
        <record id="mass_mail_contact_5" model="mailing.contact">
            <field name="name">Elsa Ericson</field>
            <field name="email">elsa.ericson@example.com</field>
            <field name="message_bounce">5</field>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_data')])]"/>
        </record>
        <record id="mass_mail_contact_6" model="mailing.contact">
            <field name="name">Franz Faubourg</field>
            <field name="email">franz.faubourg@example.com</field>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_1')])]"/>
        </record>

        <!-- Create Opt-out Records -->
        <record id="mass_mail_contact_list_rel_1" model="mailing.contact.subscription">
            <field name="list_id" ref="mass_mailing.mailing_list_data"/>
            <field name="contact_id" ref="mass_mailing.mass_mail_contact_4"/>
            <field name="opt_out">True</field>
        </record>
        <record id="mass_mail_contact_list_rel_2" model="mailing.contact.subscription">
            <field name="list_id" ref="mass_mailing.mailing_list_data"/>
            <field name="contact_id" ref="mass_mailing.mass_mail_contact_6"/>
            <field name="opt_out">True</field>
        </record>

        <!-- Create Blacklist Records -->
        <record id="blacklist_1" model="mail.blacklist">
            <field name="email">elsa.ericson@example.com</field>
        </record>

        <!-- Create campaign and mailings -->
        <record id="utm_source_0" model="utm.source">
            <field name="name">Newsletter 1</field>
        </record>
        <record id="mass_mail_campaign_1" model="utm.campaign">
            <field name="name">Newsletter</field>
            <field name="stage_id" ref="utm.campaign_stage_1"/>
            <field name="user_id" ref="base.user_admin"/>
            <field name="tag_ids" eval="[(6,0,[ref('utm.utm_tag_1')])]"/>
        </record>

        <record id="mass_mail_1" model="mailing.mailing">
            <field name="name">Newsletter 1</field>
            <field name="subject">Monthly Newsletter</field>
            <field name="state">done</field>
            <field name="user_id" ref="base.user_admin"/>
            <field name="email_from">info@yourcompany.example.com</field>
            <field name="sent_date" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="campaign_id" ref="mass_mail_campaign_1"/>
            <field name="source_id" ref="mass_mailing.utm_source_0"/>
            <field name="mailing_model_id" ref="base.model_res_partner"/>
            <field name="mailing_domain" eval="[('parent_id', '=', ref('base.res_partner_4'))]"/>
            <field name="reply_to_mode">new</field>
            <field name="reply_to">Info &lt;info@yourcompany.example.com&gt;</field>
            <field name="body_arch" type="html">
<div class="o_layout o_default_theme oe_unremovable oe_unmovable" data-name="Mailing">
    <div class="container o_mail_wrapper oe_unremovable oe_unmovable" style="border-collapse:collapse;">
        <div class="row">
            <div class="col o_mail_no_options o_mail_wrapper_td bg-white oe_structure o_editable" style="text-align:left;width:100%;">
                <div class="o_mail_block_header_logo" data-snippet="s_mail_block_header_logo">
                    <div class="o_mail_snippet_general" style="margin:0px auto 0px auto;background-color:rgb(255, 255, 255);max-width:600px;width:100%;">
                        <div class="container o_mail_h_padding" style="padding:0 20px 0 20px;width:100%;border-collapse:separate;">
                            <div class="row">
                                <div valign="center" width="30%" class="col text-center o_mail_v_padding pb0" style="padding:20px 0 0px 0;vertical-align:middle;text-align:center;">
                                    <a href="http://www.example.com" style="text-decoration:none;font-weight:bold;background-color:transparent;color:rgb(100, 89, 116);">
                                        <img border="0" src="/mass_mailing/static/src/img/theme_default/s_default_image_header_logo.png" style="border-style:none;height:auto;vertical-align:middle;max-width:400px;width:auto"/> ​
                                    </a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="o_mail_block_footer_separator" data-snippet="s_hr" style="margin:0 20px 0 20px;">
                    <div class="o_mail_snippet_general" style="margin:0px auto 0px auto;background-color:rgb(255, 255, 255);max-width:600px;width:100%;">
                        <div class="container" style="width:100%;border-collapse:separate;">
                            <div class="row">
                                <div valign="top" style="padding:20px 0 20px 0;text-align:left;vertical-align:top;width:100%;" class="col o_mail_v_padding o_mail_no_colorpicker">
                                    <div style="background-color:rgb(245, 245, 245);height:2px;width:100%;" class="separator"></div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="o_mail_block_text" data-snippet="s_text_block">
                    <div class="o_mail_snippet_general" style="margin:0px auto 0px auto;background-color:rgb(255, 255, 255);max-width:600px;width:100%;">
                        <div class="container" style="width:100%;border-collapse:separate;">
                            <div class="row">
                                <div class="col-12 o_mail_h_padding o_mail_v_padding o_mail_no_colorpicker" style="padding:20px;text-align:left;vertical-align:top;">
                                    <p style="margin:0px 0 1rem 0;font-size:14px;">
                                        Great stories have personality. Consider telling a great story that provides personality. Writing a story with personality for potential clients will assist with making a relationship connection. This shows up in small quirks like word choices or phrases. Write from your point of view, not from someone else's experience.
                                        <br/>Great stories are for everyone even when only written for just one person. If you try to write with a wide general audience in mind, your story will ring false and be bland. No one will be interested. Write for one person. If it’s genuine for the one, it’s genuine for the rest.
                                    </p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="o_mail_block_footer_social o_mail_footer_social_center" data-snippet="s_mail_block_footer_social">
                    <div class="o_mail_snippet_general" style="margin:0px auto 0px auto;background-color:rgb(255, 255, 255);max-width:600px;width:100%;">
                        <div align="center" class="container" style="border-style:solid none none none;padding:20px 0 20px 0;border-top-color:rgb(245, 245, 245);border-top-width:2px;width:100%;border-collapse:separate;">
                            <div class="row">
                                <div class="col o_mail_footer_links o_default_snippet_text" style="padding:10px 0 10px 0;text-align:center;vertical-align:middle;">
                                    <a href="/unsubscribe_from_list" class="btn btn-link o_default_snippet_text" style="text-decoration:none;border-radius:0.25rem;border-style:solid;padding:0px;cursor:pointer;line-height:1.5;font-size:12px;border-start-color:transparent;border-bottom-color:transparent;border-end-color:transparent;border-top-color:transparent;border-start-width:1px;border-bottom-width:1px;border-end-width:1px;border-top-width:1px;user-select:none;vertical-align:middle;white-space:nowrap;text-align:center;font-weight:bold;display:inline-block;background-color:transparent;color:rgb(100, 89, 116);">Unsubscribe</a> |

                                    <a href="/contactus" class="btn btn-link o_default_snippet_text" style="text-decoration:none;border-radius:0.25rem;border-style:solid;padding:0px;cursor:pointer;line-height:1.5;font-size:12px;border-start-color:transparent;border-bottom-color:transparent;border-end-color:transparent;border-top-color:transparent;border-start-width:1px;border-bottom-width:1px;border-end-width:1px;border-top-width:1px;user-select:none;vertical-align:middle;white-space:nowrap;text-align:center;font-weight:bold;display:inline-block;background-color:transparent;color:rgb(100, 89, 116);">Contact</a>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col" style="text-align:left;vertical-align:middle;">
                                    <p class="o_mail_footer_copy" style="margin:0px 0 1rem 0;text-align:center;font-weight:bold;color:rgb(147, 146, 146);font-size:9px;">
                                        <img src="/web_editor/font_to_img/61945/rgb(147,146,146)/9" data-class="fa fa-copyright" style="border-style:none;max-width:100%;width:100%;vertical-align:middle;height: auto; width: auto;"/>2018 All Rights Reserved
                                    </p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div align="center" class="container" style="width:100%;border-collapse:separate;">
        <div class="row">
            <div align="center" style="padding:16px 0 16px 0;" class="col pt16 pb16">
                Powered by <a target="_blank" href="https://www.odoo.com" style="text-decoration:none;background-color:transparent;color:#875A7B;">Odoo</a>
            </div>
        </div>
    </div>
</div>
</field>
            <field name="attachment_ids" eval="[(4, ref('mass_mail_attach_1'))]"/>
        </record>
        <!-- Generate link tracker information from it -->
        <function model="mailing.mailing" name="convert_links" eval="[ref('mass_mailing.mass_mail_1')]"/>

        <record id="mass_mail_1_stat_0" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111000@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_7"/>
            <field name="email">billy.fox45@example.com</field>
            <field name="trace_status">reply</field>
            <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="open_datetime" eval="(DateTime.today() - relativedelta(days=4)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="reply_datetime" eval="(DateTime.today() - relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="write_date" eval="(DateTime.today() - relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_1" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111001@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_13"/>
            <field name="email">kim.snyder96@example.com</field>
            <field name="trace_status">reply</field>
            <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="open_datetime" eval="(DateTime.today() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="reply_datetime" eval="(DateTime.today() - relativedelta(days=0)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="write_date" eval="(DateTime.today() - relativedelta(days=0)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_2" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111002@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_14"/>
            <field name="email">edith.sanchez68@example.com</field>
            <field name="trace_status">open</field>
            <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="open_datetime" eval="(DateTime.today() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="write_date" eval="(DateTime.today() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_3" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111003@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_24"/>
            <field name="email">theodore.gardner36@example.com</field>
            <field name="trace_status">open</field>
            <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="open_datetime" eval="(DateTime.today() - relativedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="write_date" eval="(DateTime.today() - relativedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_4" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111004@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_32"/>
            <field name="email">sandra.neal80@example.com</field>
            <field name="trace_status">sent</field>
            <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="write_date" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_5" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111005@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_33"/>
            <field name="email">julie.richards84@example.com</field>
            <field name="trace_status">error</field>
            <field name="sent_datetime" eval="False"/>
        </record>
        <record id="mass_mail_1_stat_6" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111006@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_34"/>
            <field name="email">travis.mendoza24@example.com</field>
            <field name="trace_status">bounce</field>
            <field name="sent_datetime" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="write_date" eval="(DateTime.today() - relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_7" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111007@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_34"/>
            <field name="email">travis.mendoza24@example.com</field>
            <field name="trace_status">bounce</field>
            <field name="sent_datetime" eval="False"/>
        </record>

        <!-- Generate some clicks -->
        <function model="link.tracker.click" name="add_click">
            <value model="link.tracker.code"
                search="[('link_id.url', '=', 'http://www.example.com')]"
                use="code"/>
            <value name="ip">100.01.02.03</value>
            <value name="country_code">BE</value>
            <value name="mailing_trace_id" eval="ref('mass_mail_1_stat_0')"/>
        </function>
        <function model="link.tracker.click" name="add_click">
            <value model="link.tracker.code"
                search="[('link_id.url', '=', 'http://www.example.net/page/contactus')]"
                use="code"/>
            <value name="ip">100.01.02.03</value>
            <value name="country_code">BE</value>
            <value name="mailing_trace_id" eval="ref('mass_mail_1_stat_0')"/>
        </function>
        <function model="link.tracker.click" name="add_click">
            <value model="link.tracker.code"
                search="[('link_id.url', '=', 'http://www.example.com')]"
                use="code"/>
            <value name="ip">100.01.02.04</value>
            <value name="country_code">BE</value>
            <value name="mailing_trace_id" eval="ref('mass_mail_1_stat_1')"/>
        </function>
        <function model="link.tracker.click" name="add_click">
            <value model="link.tracker.code"
                search="[('link_id.url', '=', 'http://www.example.net/page/contactus')]"
                use="code"/>
            <value name="ip">100.01.02.04</value>
            <value name="country_code">BE</value>
            <value name="mailing_trace_id" eval="ref('mass_mail_1_stat_0')"/>
        </function>
        <function model="link.tracker.click" name="add_click">
            <value model="link.tracker.code"
                search="[('link_id.url', '=', 'http://www.example.com')]"
                use="code"/>
            <value name="ip">100.01.02.05</value>
            <value name="country_code">BE</value>
            <value name="mailing_trace_id" eval="ref('mass_mail_1_stat_2')"/>
        </function>

    </data>
</odoo>

```

## File: data\res_users_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('mass_mailing.group_mass_mailing_user'))]"/>
        </record>
    </data>
</odoo>

```

## File: doc\changelog.rst

```rst
.. _changelog:

Changelog
=========

`trunk (saas-2)`
----------------

 - added module
```

## File: doc\index.rst

```rst
Mass Mailing module documentation
=================================

Mass Mailing documentation topics
'''''''''''''''''''''''''''''''''

Changelog
'''''''''

.. toctree::
   :maxdepth: 1

   changelog.rst

```

## File: models\ir_http.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class IrHttp(models.AbstractModel):
    _inherit = "ir.http"

    @classmethod
    def _get_translation_frontend_modules_name(cls):
        mods = super()._get_translation_frontend_modules_name()
        return mods + ["mass_mailing"]

```

## File: models\ir_mail_server.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models
from odoo.tools.misc import format_date


class IrMailServer(models.Model):
    _name = 'ir.mail_server'
    _inherit = ['ir.mail_server']

    active_mailing_ids = fields.One2many(
        comodel_name='mailing.mailing',
        inverse_name='mail_server_id',
        string='Active mailing using this mail server',
        readonly=True,
        domain=[('state', '!=', 'done'), ('active', '=', True)])

    def _active_usages_compute(self):
        def format_usage(mailing_id):
            base = _('Mass Mailing "%s"', mailing_id.display_name)
            if not mailing_id.schedule_date:
                return base
            details = _('(scheduled for %s)', format_date(self.env, mailing_id.schedule_date))
            return f'{base} {details}'

        usages_super = super(IrMailServer, self)._active_usages_compute()
        default_mail_server_id = self.env['mailing.mailing']._get_default_mail_server_id()
        for record in self:
            usages = []
            if default_mail_server_id == record.id:
                usages.append(_('Email Marketing uses it as its default mail server to send mass mailings'))
            usages.extend(map(format_usage, record.active_mailing_ids))
            if usages:
                usages_super.setdefault(record.id, []).extend(usages)
        return usages_super

```

## File: models\ir_model.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _


class IrModel(models.Model):
    _inherit = 'ir.model'

    is_mailing_enabled = fields.Boolean(
        string="Mailing Enabled",
        compute='_compute_is_mailing_enabled', search='_search_is_mailing_enabled',
        help="Whether this model supports marketing mailing capabilities (notably email and SMS).",
    )

    def _compute_is_mailing_enabled(self):
        for model in self:
            model.is_mailing_enabled = getattr(self.env[model.model], '_mailing_enabled', False)

    def _search_is_mailing_enabled(self, operator, value):
        if operator not in ('=', '!='):
            raise ValueError(_("Searching Mailing Enabled models supports only direct search using '='' or '!='."))

        valid_models = self.env['ir.model']
        for model in self.search([]):
            if model.model not in self.env or model.is_transient():
                continue
            if getattr(self.env[model.model], '_mailing_enabled', False):
                valid_models |= model

        search_is_mailing_enabled = (operator == '=' and value) or (operator == '!=' and not value)
        if search_is_mailing_enabled:
            return [('id', 'in', valid_models.ids)]
        return [('id', 'not in', valid_models.ids)]

```

## File: models\link_tracker.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class LinkTracker(models.Model):
    _inherit = "link.tracker"

    mass_mailing_id = fields.Many2one('mailing.mailing', string='Mass Mailing')


class LinkTrackerClick(models.Model):
    _inherit = "link.tracker.click"

    mailing_trace_id = fields.Many2one('mailing.trace', string='Mail Statistics')
    mass_mailing_id = fields.Many2one('mailing.mailing', string='Mass Mailing')

    def _prepare_click_values_from_route(self, **route_values):
        click_values = super(LinkTrackerClick, self)._prepare_click_values_from_route(**route_values)

        if click_values.get('mailing_trace_id'):
            trace_sudo = self.env['mailing.trace'].sudo().browse(route_values['mailing_trace_id']).exists()
            if not trace_sudo:
                click_values['mailing_trace_id'] = False
            else:
                if not click_values.get('campaign_id'):
                    click_values['campaign_id'] = trace_sudo.campaign_id.id
                if not click_values.get('mass_mailing_id'):
                    click_values['mass_mailing_id'] = trace_sudo.mass_mailing_id.id

        return click_values

    @api.model
    def add_click(self, code, **route_values):
        click = super(LinkTrackerClick, self).add_click(code, **route_values)

        if click and click.mailing_trace_id:
            click.mailing_trace_id.set_opened()
            click.mailing_trace_id.set_clicked()

        return click

```

## File: models\mailing.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import hashlib
import hmac
import io
import logging
import lxml
import random
import re
import requests
import threading
import werkzeug.urls
from ast import literal_eval
from dateutil.relativedelta import relativedelta
from markupsafe import Markup
from werkzeug.urls import url_join
from PIL import Image, UnidentifiedImageError

from odoo import api, fields, models, tools, _
from odoo.addons.base_import.models.base_import import ImportValidationError
from odoo.exceptions import UserError, ValidationError
from odoo.osv import expression

_logger = logging.getLogger(__name__)

# Syntax of the data URL Scheme: https://tools.ietf.org/html/rfc2397#section-3
# Used to find inline images
image_re = re.compile(r"data:(image/[A-Za-z]+);base64,(.*)")
DEFAULT_IMAGE_TIMEOUT = 3
DEFAULT_IMAGE_MAXBYTES = 10 * 1024 * 1024  # 10MB
DEFAULT_IMAGE_CHUNK_SIZE = 32768

mso_re = re.compile(r"\[if mso\]>[\s\S]*<!\[endif\]")

class MassMailing(models.Model):
    """ Mass Mailing models the sending of emails to a list of recipients for a mass mailing campaign."""
    _name = 'mailing.mailing'
    _description = 'Mass Mailing'
    _inherit = ['mail.thread',
                'mail.activity.mixin',
                'mail.render.mixin',
                'utm.source.mixin'
    ]
    _order = 'calendar_date DESC'
    _rec_name = "subject"

    @api.model
    def default_get(self, fields_list):
        vals = super(MassMailing, self).default_get(fields_list)

        # field sent by the calendar view when clicking on a date block
        # we use it to setup the scheduled date of the created mailing.mailing
        default_calendar_date = self.env.context.get('default_calendar_date')
        if default_calendar_date and ('schedule_type' in fields_list and 'schedule_date' in fields_list) \
           and fields.Datetime.from_string(default_calendar_date) > fields.Datetime.now():
            vals.update({
                'schedule_type': 'scheduled',
                'schedule_date': default_calendar_date
            })

        if 'contact_list_ids' in fields_list and not vals.get('contact_list_ids') and vals.get('mailing_model_id'):
            if vals.get('mailing_model_id') == self.env['ir.model']._get('mailing.list').id:
                mailing_list = self.env['mailing.list'].search([], limit=2)
                if len(mailing_list) == 1:
                    vals['contact_list_ids'] = [(6, 0, [mailing_list.id])]
        return vals

    @api.model
    def _get_default_mail_server_id(self):
        server_id = self.env['ir.config_parameter'].sudo().get_param('mass_mailing.mail_server_id')
        try:
            server_id = literal_eval(server_id) if server_id else False
            return self.env['ir.mail_server'].search([('id', '=', server_id)]).id
        except ValueError:
            return False

    active = fields.Boolean(default=True, tracking=True)
    subject = fields.Char(
        'Subject', required=True, translate=False)
    preview = fields.Char(
        'Preview', translate=False,
        help='Catchy preview sentence that encourages recipients to open this email.\n'
             'In most inboxes, this is displayed next to the subject.\n'
             'Keep it empty if you prefer the first characters of your email content to appear instead.')
    email_from = fields.Char(
        string='Send From',
        compute='_compute_email_from', readonly=False, required=True, store=True,
        precompute=True)
    favorite = fields.Boolean('Favorite', copy=False, tracking=True)
    favorite_date = fields.Datetime(
        'Favorite Date',
        compute='_compute_favorite_date', store=True,
        copy=False,
        help='When this mailing was added in the favorites')
    sent_date = fields.Datetime(string='Sent Date', copy=False)
    schedule_type = fields.Selection(
        [('now', 'Send now'), ('scheduled', 'Send on')],
        string='Schedule', default='now',
        readonly=True, required=True,
        states={'draft': [('readonly', False)], 'in_queue': [('readonly', False)]})
    schedule_date = fields.Datetime(
        string='Scheduled for',
        compute='_compute_schedule_date', readonly=True, store=True,
        copy=True, tracking=True,
        states={'draft': [('readonly', False)], 'in_queue': [('readonly', False)]})
    calendar_date = fields.Datetime(
        'Calendar Date',
        compute='_compute_calendar_date', store=True,
        copy=False,
        help="Date at which the mailing was or will be sent.")
    # don't translate 'body_arch', the translations are only on 'body_html'
    body_arch = fields.Html(string='Body', translate=False, sanitize=False)
    body_html = fields.Html(string='Body converted to be sent by mail', render_engine='qweb', sanitize=False)

   # used to determine if the mail body is empty
    is_body_empty = fields.Boolean(compute="_compute_is_body_empty")
    attachment_ids = fields.Many2many(
        'ir.attachment', 'mass_mailing_ir_attachments_rel',
        'mass_mailing_id', 'attachment_id', string='Attachments')
    keep_archives = fields.Boolean(string='Keep Archives')
    campaign_id = fields.Many2one('utm.campaign', string='UTM Campaign', index=True, ondelete='set null')
    medium_id = fields.Many2one(
        'utm.medium', string='Medium',
        compute='_compute_medium_id', readonly=False, store=True,
        ondelete='restrict',
        help="UTM Medium: delivery method (email, sms, ...)")
    state = fields.Selection(
        [('draft', 'Draft'), ('in_queue', 'In Queue'),
         ('sending', 'Sending'), ('done', 'Sent')],
        string='Status',
        default='draft', required=True,
        copy=False, tracking=True,
        group_expand='_group_expand_states')
    color = fields.Integer(string='Color Index')
    user_id = fields.Many2one(
        'res.users', string='Responsible',
        tracking=True,
        default=lambda self: self.env.user)
    # mailing options
    mailing_type = fields.Selection([('mail', 'Email')], string="Mailing Type", default="mail", required=True)
    mailing_type_description = fields.Char('Mailing Type Description', compute="_compute_mailing_type_description")
    reply_to_mode = fields.Selection(
        [('update', 'Recipient Followers'), ('new', 'Specified Email Address')],
        string='Reply-To Mode',
        compute='_compute_reply_to_mode', readonly=False, store=True,
        help='Thread: replies go to target document. Email: replies are routed to a given email.')
    reply_to = fields.Char(
        string='Reply To',
        compute='_compute_reply_to', readonly=False, store=True,
        help='Preferred Reply-To Address')
    # recipients
    mailing_model_real = fields.Char(
        string='Recipients Real Model', compute='_compute_mailing_model_real')
    mailing_model_id = fields.Many2one(
        'ir.model', string='Recipients Model',
        ondelete='cascade', required=True,
        domain=[('is_mailing_enabled', '=', True)],
        default=lambda self: self.env.ref('mass_mailing.model_mailing_list').id)
    mailing_model_name = fields.Char(
        string='Recipients Model Name',
        related='mailing_model_id.model', readonly=True, related_sudo=True)
    mailing_domain = fields.Char(
        string='Domain',
        compute='_compute_mailing_domain', readonly=False, store=True)
    mail_server_available = fields.Boolean(
        compute='_compute_mail_server_available',
        help="Technical field used to know if the user has activated the outgoing mail server option in the settings")
    mail_server_id = fields.Many2one('ir.mail_server', string='Mail Server',
        default=_get_default_mail_server_id,
        help="Use a specific mail server in priority. Otherwise Odoo relies on the first outgoing mail server available (based on their sequencing) as it does for normal mails.")
    contact_list_ids = fields.Many2many('mailing.list', 'mail_mass_mailing_list_rel', string='Mailing Lists')
    # Mailing Filter
    mailing_filter_id = fields.Many2one(
        'mailing.filter', string='Favorite Filter',
        compute='_compute_mailing_filter_id', readonly=False, store=True,
        domain="[('mailing_model_name', '=', mailing_model_name)]")
    mailing_filter_domain = fields.Char('Favorite filter domain', related='mailing_filter_id.mailing_domain')
    mailing_filter_count = fields.Integer('# Favorite Filters', compute='_compute_mailing_filter_count')
    # A/B Testing
    ab_testing_completed = fields.Boolean(related='campaign_id.ab_testing_completed', store=True)
    ab_testing_description = fields.Html('A/B Testing Description', compute="_compute_ab_testing_description")
    ab_testing_enabled = fields.Boolean(
        string='Allow A/B Testing', default=False,
        help='If checked, recipients will be mailed only once for the whole campaign. '
             'This lets you send different mailings to randomly selected recipients and test '
             'the effectiveness of the mailings, without causing duplicate messages.')
    ab_testing_mailings_count = fields.Integer(related="campaign_id.ab_testing_mailings_count")
    ab_testing_pc = fields.Integer(
        string='A/B Testing percentage',
        default=10,
        help='Percentage of the contacts that will be mailed. Recipients will be chosen randomly.')
    ab_testing_schedule_datetime = fields.Datetime(
        related="campaign_id.ab_testing_schedule_datetime", readonly=False,
        default=lambda self: fields.Datetime.now() + relativedelta(days=1))
    ab_testing_winner_selection = fields.Selection(
        related="campaign_id.ab_testing_winner_selection", readonly=False,
        default="opened_ratio",
        copy=True)
    kpi_mail_required = fields.Boolean('KPI mail required', copy=False)
    # statistics data
    mailing_trace_ids = fields.One2many('mailing.trace', 'mass_mailing_id', string='Emails Statistics')
    total = fields.Integer(compute="_compute_total")
    scheduled = fields.Integer(compute="_compute_statistics")
    expected = fields.Integer(compute="_compute_statistics")
    canceled = fields.Integer(compute="_compute_statistics")
    sent = fields.Integer(compute="_compute_statistics")
    delivered = fields.Integer(compute="_compute_statistics")
    opened = fields.Integer(compute="_compute_statistics")
    clicked = fields.Integer(compute="_compute_statistics")
    replied = fields.Integer(compute="_compute_statistics")
    bounced = fields.Integer(compute="_compute_statistics")
    failed = fields.Integer(compute="_compute_statistics")
    received_ratio = fields.Integer(compute="_compute_statistics", string='Received Ratio')
    opened_ratio = fields.Integer(compute="_compute_statistics", string='Opened Ratio')
    replied_ratio = fields.Integer(compute="_compute_statistics", string='Replied Ratio')
    bounced_ratio = fields.Integer(compute="_compute_statistics", string='Bounced Ratio')
    clicks_ratio = fields.Integer(compute="_compute_clicks_ratio", string="Number of Clicks")
    next_departure = fields.Datetime(compute="_compute_next_departure", string='Scheduled date')
    # UX
    warning_message = fields.Char(
        'Warning Message', compute='_compute_warning_message',
        help='Warning message displayed in the mailing form view')

    _sql_constraints = [(
        'percentage_valid',
        'CHECK(ab_testing_pc >= 0 AND ab_testing_pc <= 100)',
        'The A/B Testing Percentage needs to be between 0 and 100%'
    )]

    @api.constrains('mailing_model_id', 'mailing_filter_id')
    def _check_mailing_filter_model(self):
        """Check that if the favorite filter is set, it must contain the same recipient model as mailing"""
        for mailing in self:
            if mailing.mailing_filter_id and mailing.mailing_model_id != mailing.mailing_filter_id.mailing_model_id:
                raise ValidationError(
                    _("The saved filter targets different recipients and is incompatible with this mailing.")
                )

    @api.depends('mail_server_id', 'create_uid')
    def _compute_email_from(self):
        notification_email = self.env['ir.mail_server']._get_default_from_address()

        for mailing in self:
            user_email = mailing.create_uid.email_formatted or self.env.user.email_formatted
            server = mailing.mail_server_id
            if not server:
                mailing.email_from = mailing.email_from or user_email
            elif mailing.email_from and server._match_from_filter(mailing.email_from, server.from_filter):
                mailing.email_from = mailing.email_from
            elif server._match_from_filter(user_email, server.from_filter):
                mailing.email_from = user_email
            elif server._match_from_filter(notification_email, server.from_filter):
                mailing.email_from = notification_email
            else:
                mailing.email_from = mailing.email_from or user_email

    @api.depends('favorite')
    def _compute_favorite_date(self):
        favorited = self.filtered('favorite')
        (self - favorited).favorite_date = False
        favorited.filtered(lambda mailing: not mailing.favorite_date).favorite_date = fields.Datetime.now()

    def _compute_total(self):
        for mass_mailing in self:
            total = self.env[mass_mailing.mailing_model_real].search_count(mass_mailing._parse_mailing_domain())
            if total and mass_mailing.ab_testing_enabled and mass_mailing.ab_testing_pc < 100:
                total = max(int(total / 100.0 * mass_mailing.ab_testing_pc), 1)
            mass_mailing.total = total

    def _compute_clicks_ratio(self):
        self.env.cr.execute("""
            SELECT COUNT(DISTINCT(stats.id)) AS nb_mails, COUNT(DISTINCT(clicks.mailing_trace_id)) AS nb_clicks, stats.mass_mailing_id AS id
            FROM mailing_trace AS stats
            LEFT OUTER JOIN link_tracker_click AS clicks ON clicks.mailing_trace_id = stats.id
            WHERE stats.mass_mailing_id IN %s
            AND stats.trace_status != 'cancel'
            GROUP BY stats.mass_mailing_id
        """, [tuple(self.ids) or (None,)])
        mass_mailing_data = self.env.cr.dictfetchall()
        mapped_data = dict([(m['id'], 100 * m['nb_clicks'] / m['nb_mails']) for m in mass_mailing_data])
        for mass_mailing in self:
            mass_mailing.clicks_ratio = mapped_data.get(mass_mailing.id, 0)

    def _compute_statistics(self):
        """ Compute statistics of the mass mailing """
        for key in (
            'scheduled', 'expected', 'canceled', 'sent', 'delivered', 'opened',
            'clicked', 'replied', 'bounced', 'failed', 'received_ratio',
            'opened_ratio', 'replied_ratio', 'bounced_ratio',
        ):
            self[key] = False
        if not self.ids:
            return
        # ensure traces are sent to db
        self.env['mailing.trace'].flush_model()
        self.env['mailing.mailing'].flush_model()
        self.env.cr.execute("""
            SELECT
                m.id as mailing_id,
                COUNT(s.id) AS expected,
                COUNT(s.sent_datetime) AS sent,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status = 'outgoing') AS scheduled,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status = 'cancel') AS canceled,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status in ('sent', 'open', 'reply')) AS delivered,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status in ('open', 'reply')) AS opened,
                COUNT(s.links_click_datetime) AS clicked,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status = 'reply') AS replied,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status = 'bounce') AS bounced,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status = 'error') AS failed
            FROM
                mailing_trace s
            RIGHT JOIN
                mailing_mailing m
                ON (m.id = s.mass_mailing_id)
            WHERE
                m.id IN %s
            GROUP BY
                m.id
        """, (tuple(self.ids), ))
        for row in self.env.cr.dictfetchall():
            total = (row['expected'] - row['canceled']) or 1
            row['received_ratio'] = 100.0 * row['delivered'] / total
            row['opened_ratio'] = 100.0 * row['opened'] / total
            row['replied_ratio'] = 100.0 * row['replied'] / total
            row['bounced_ratio'] = 100.0 * row['bounced'] / total
            self.browse(row.pop('mailing_id')).update(row)

    def _compute_next_departure(self):
        # Schedule_date should only be False if schedule_type = "now" or
        # mass_mailing is canceled.
        # A cron.trigger is created when mailing is put "in queue"
        # so we can reasonably expect that the cron worker will
        # execute this based on the cron.trigger's call_at which should
        # be now() when clicking "Send" or schedule_date if scheduled

        for mass_mailing in self:
            if mass_mailing.schedule_date:
                # max in case the user schedules a date in the past
                mass_mailing.next_departure = max(mass_mailing.schedule_date, fields.datetime.now())
            else:
                mass_mailing.next_departure = fields.datetime.now()

    @api.depends('email_from', 'mail_server_id')
    def _compute_warning_message(self):
        self.warning_message = False
        for mailing in self.filtered(lambda mailing: mailing.mailing_type == "mail"):
            mail_server = mailing.mail_server_id
            if mail_server and not mail_server._match_from_filter(mailing.email_from, mail_server.from_filter):
                mailing.warning_message = _(
                    'This email from can not be used with this mail server.\n'
                    'Your emails might be marked as spam on the mail clients.'
                )
            else:
                mailing.warning_message = False

    @api.depends('mailing_type')
    def _compute_medium_id(self):
        for mailing in self:
            if mailing.mailing_type == 'mail' and not mailing.medium_id:
                mailing.medium_id = self.env.ref('utm.utm_medium_email').id

    @api.depends('mailing_model_id')
    def _compute_reply_to_mode(self):
        """ For main models not really using chatter to gather answers (contacts
        and mailing contacts), set reply-to as email-based. Otherwise answers
        by default go on the original discussion thread (business document). Note
        that mailing_model being mailing.list means contacting mailing.contact
        (see mailing_model_name versus mailing_model_real). """
        for mailing in self:
            if mailing.mailing_model_id.model in ['res.partner', 'mailing.list', 'mailing.contact']:
                mailing.reply_to_mode = 'new'
            else:
                mailing.reply_to_mode = 'update'

    @api.depends('reply_to_mode')
    def _compute_reply_to(self):
        for mailing in self:
            if mailing.reply_to_mode == 'new' and not mailing.reply_to:
                mailing.reply_to = self.env.user.email_formatted
            elif mailing.reply_to_mode == 'update':
                mailing.reply_to = False

    @api.depends('mailing_model_id', 'mailing_domain')
    def _compute_mailing_filter_count(self):
        filter_data = self.env['mailing.filter']._read_group([
            ('mailing_model_id', 'in', self.mailing_model_id.ids)
        ], ['mailing_model_id'], ['mailing_model_id'])
        mapped_data = {data['mailing_model_id'][0]: data['mailing_model_id_count'] for data in filter_data}
        for mailing in self:
            mailing.mailing_filter_count = mapped_data.get(mailing.mailing_model_id.id, 0)

    @api.depends('mailing_model_id')
    def _compute_mailing_model_real(self):
        for mailing in self:
            mailing.mailing_model_real = 'mailing.contact' if mailing.mailing_model_id.model == 'mailing.list' else mailing.mailing_model_id.model

    @api.depends('mailing_model_id', 'contact_list_ids', 'mailing_type', 'mailing_filter_id')
    def _compute_mailing_domain(self):
        for mailing in self:
            if not mailing.mailing_model_id:
                mailing.mailing_domain = ''
            elif mailing.mailing_filter_id:
                mailing.mailing_domain = mailing.mailing_filter_id.mailing_domain
            else:
                mailing.mailing_domain = repr(mailing._get_default_mailing_domain())

    @api.depends('mailing_model_name')
    def _compute_mailing_filter_id(self):
        for mailing in self:
            mailing.mailing_filter_id = False

    @api.depends('schedule_type')
    def _compute_schedule_date(self):
        for mailing in self:
            if mailing.schedule_type == 'now' or not mailing.schedule_date:
                mailing.schedule_date = False

    @api.depends('state', 'schedule_date', 'sent_date', 'next_departure')
    def _compute_calendar_date(self):
        for mailing in self:
            if mailing.state == 'done':
                mailing.calendar_date = mailing.sent_date
            elif mailing.state == 'in_queue':
                mailing.calendar_date = mailing.next_departure
            elif mailing.state == 'sending':
                mailing.calendar_date = fields.Datetime.now()
            else:
                mailing.calendar_date = False

    @api.depends('body_arch')
    def _compute_is_body_empty(self):
        for mailing in self:
            mailing.is_body_empty = tools.is_html_empty(mailing.body_arch)

    def _compute_mail_server_available(self):
        self.mail_server_available = self.env['ir.config_parameter'].sudo().get_param('mass_mailing.outgoing_mail_server')

    # Overrides of mail.render.mixin
    @api.depends('mailing_model_real')
    def _compute_render_model(self):
        for mailing in self:
            mailing.render_model = mailing.mailing_model_real

    @api.depends('mailing_type')
    def _compute_mailing_type_description(self):
        for mailing in self:
            mailing.mailing_type_description = dict(self._fields.get('mailing_type').selection).get(mailing.mailing_type)

    @api.depends(lambda self: self._get_ab_testing_description_modifying_fields())
    def _compute_ab_testing_description(self):
        mailing_ab_test = self.filtered('ab_testing_enabled')
        (self - mailing_ab_test).ab_testing_description = False
        for mailing in mailing_ab_test:
            mailing.ab_testing_description = self.env['ir.qweb']._render(
                'mass_mailing.ab_testing_description',
                mailing._get_ab_testing_description_values()
            )

    def _get_ab_testing_description_modifying_fields(self):
        return ['ab_testing_enabled', 'ab_testing_pc', 'ab_testing_schedule_datetime', 'ab_testing_winner_selection', 'campaign_id']

    # ------------------------------------------------------
    # ORM
    # ------------------------------------------------------

    @api.model_create_multi
    def create(self, vals_list):
        ab_testing_cron = self.env.ref('mass_mailing.ir_cron_mass_mailing_ab_testing').sudo()
        for values in vals_list:
            if values.get('ab_testing_schedule_datetime'):
                at = fields.Datetime.from_string(values['ab_testing_schedule_datetime'])
                ab_testing_cron._trigger(at=at)
        mailings = super().create(vals_list)
        mailings._create_ab_testing_utm_campaigns()
        mailings._fix_attachment_ownership()

        for values, mailing in zip(vals_list, mailings):
            if values.get('body_arch'):
                mailing.body_arch = mailing._convert_inline_images_to_urls(mailing.body_arch)
            if values.get('body_html'):
                mailing.body_html = mailing._convert_inline_images_to_urls(mailing.body_html)
        return mailings

    def write(self, values):
        if values.get('body_arch'):
            values['body_arch'] = self._convert_inline_images_to_urls(values['body_arch'])
        if values.get('body_html'):
            values['body_html'] = self._convert_inline_images_to_urls(values['body_html'])
        # If ab_testing is already enabled on a mailing and the campaign is removed, we raise a ValidationError
        if values.get('campaign_id') is False and any(mailing.ab_testing_enabled for mailing in self) and 'ab_testing_enabled' not in values:
            raise ValidationError(_("A campaign should be set when A/B test is enabled"))

        result = super(MassMailing, self).write(values)
        if values.get('ab_testing_enabled'):
            self._create_ab_testing_utm_campaigns()
        self._fix_attachment_ownership()

        if any(self.mapped('ab_testing_schedule_datetime')):
            schedule_date = min(m.ab_testing_schedule_datetime for m in self if m.ab_testing_schedule_datetime)
            ab_testing_cron = self.env.ref('mass_mailing.ir_cron_mass_mailing_ab_testing').sudo()
            ab_testing_cron._trigger(at=schedule_date)

        return result

    def _create_ab_testing_utm_campaigns(self):
        """ Creates the A/B test campaigns for the mailings that do not have campaign set already """
        campaign_vals = [
            mailing._get_default_ab_testing_campaign_values()
            for mailing in self.filtered(lambda mailing: mailing.ab_testing_enabled and not mailing.campaign_id)
        ]
        return self.env['utm.campaign'].create(campaign_vals)

    def _fix_attachment_ownership(self):
        for record in self:
            record.attachment_ids.write({'res_model': record._name, 'res_id': record.id})
        return self

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        self.ensure_one()
        default = dict(default or {}, contact_list_ids=self.contact_list_ids.ids)
        if self.mail_server_id and not self.mail_server_id.active:
            default['mail_server_id'] = self._get_default_mail_server_id()
        if self.ab_testing_enabled:
            default['ab_testing_schedule_datetime'] = self.ab_testing_schedule_datetime
        return super(MassMailing, self).copy(default=default)

    def _group_expand_states(self, states, domain, order):
        return [key for key, val in self._fields['state'].selection]

    # ------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------

    def action_set_favorite(self):
        """Add the current mailing in the favorites list."""
        self.favorite = True

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'message': _(
                    'Design added to the %s Templates!',
                    ', '.join(self.mapped('mailing_model_id.name')),
                ),
                'next': {'type': 'ir.actions.act_window_close'},
                'sticky': False,
                'type': 'info',
            }
        }

    def action_remove_favorite(self):
        """Remove the current mailing from the favorites list."""
        self.favorite = False

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'message': _(
                    'Design removed from the %s Templates!',
                    ', '.join(self.mapped('mailing_model_id.name')),
                ),
                'next': {'type': 'ir.actions.act_window_close'},
                'sticky': False,
                'type': 'info',
            }
        }

    def action_duplicate(self):
        self.ensure_one()
        mass_mailing_copy = self.copy()
        if mass_mailing_copy:
            context = dict(self.env.context)
            context['form_view_initial_mode'] = 'edit'
            action = {
                'type': 'ir.actions.act_window',
                'view_mode': 'form',
                'res_model': 'mailing.mailing',
                'res_id': mass_mailing_copy.id,
                'context': context,
            }
            if self.mailing_type == 'mail':
                action['views'] = [
                    (self.env.ref('mass_mailing.mailing_mailing_view_form_full_width').id, 'form'),
                ]
            return action
        return False

    def action_test(self):
        self.ensure_one()
        ctx = dict(self.env.context, default_mass_mailing_id=self.id, dialog_size='medium')
        return {
            'name': _('Test Mailing'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mailing.mailing.test',
            'target': 'new',
            'context': ctx,
        }

    def action_launch(self):
        self.write({'schedule_type': 'now'})
        return self.action_put_in_queue()

    def action_schedule(self):
        self.ensure_one()
        if self.schedule_date and self.schedule_date > fields.Datetime.now():
            return self.action_put_in_queue()
        action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing.mailing_mailing_schedule_date_action")
        action['context'] = dict(self.env.context, default_mass_mailing_id=self.id, dialog_size='medium')
        return action

    def action_put_in_queue(self):
        self.write({'state': 'in_queue'})
        cron = self.env.ref('mass_mailing.ir_cron_mass_mailing_queue')
        cron._trigger(
            schedule_date or fields.Datetime.now()
            for schedule_date in self.mapped('schedule_date')
        )

    def action_cancel(self):
        self.write({'state': 'draft', 'schedule_date': False, 'schedule_type': 'now', 'next_departure': False})

    def action_retry_failed(self):
        failed_mails = self.env['mail.mail'].sudo().search([
            ('mailing_id', 'in', self.ids),
            ('state', '=', 'exception')
        ])
        failed_mails.mapped('mailing_trace_ids').unlink()
        failed_mails.unlink()
        self.action_put_in_queue()

    def action_view_traces_scheduled(self):
        return self._action_view_traces_filtered('scheduled')

    def action_view_traces_canceled(self):
        return self._action_view_traces_filtered('canceled')

    def action_view_traces_failed(self):
        return self._action_view_traces_filtered('failed')

    def action_view_traces_sent(self):
        return self._action_view_traces_filtered('sent')

    def _action_view_traces_filtered(self, view_filter):
        action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing.mailing_trace_action")
        action['name'] = _('Sent Mailings')
        action['context'] = {'search_default_mass_mailing_id': self.id,}
        filter_key = 'search_default_filter_%s' % (view_filter)
        action['context'][filter_key] = True
        action['views'] = [
            (self.env.ref('mass_mailing.mailing_trace_view_tree_mail').id, 'tree'),
            (self.env.ref('mass_mailing.mailing_trace_view_form').id, 'form')
        ]
        return action

    def action_view_clicked(self):
        model_name = self.env['ir.model']._get('link.tracker').display_name
        recipient = self.env['ir.model']._get(self.mailing_model_real).display_name
        helper_header = _("No %s clicked your mailing yet!", recipient)
        helper_message = _("Link Trackers will measure how many times each link is clicked as well as "
                           "the proportion of %s who clicked at least once in your mailing.", recipient)
        return {
            'name': model_name,
            'type': 'ir.actions.act_window',
            'view_mode': 'tree',
            'res_model': 'link.tracker',
            'domain': [('mass_mailing_id', '=', self.id)],
            'help': Markup('<p class="o_view_nocontent_smiling_face">%s</p><p>%s</p>') % (
                helper_header, helper_message,
            ),
            'context': dict(self._context, create=False)
        }

    def action_view_opened(self):
        return self._action_view_documents_filtered('open')

    def action_view_replied(self):
        return self._action_view_documents_filtered('reply')

    def action_view_bounced(self):
        return self._action_view_documents_filtered('bounce')

    def action_view_delivered(self):
        return self._action_view_documents_filtered('delivered')

    def _action_view_documents_filtered(self, view_filter):
        model_name = self.env['ir.model']._get(self.mailing_model_real).display_name
        helper_header = None
        helper_message = None
        if view_filter == 'reply':
            found_traces = self.mailing_trace_ids.filtered(lambda trace: trace.trace_status == view_filter)
            helper_header = _("No %s replied to your mailing yet!", model_name)
            helper_message = _("To track how many replies this mailing gets, make sure "
                               "its reply-to address belongs to this database.")
        elif view_filter == 'bounce':
            found_traces = self.mailing_trace_ids.filtered(lambda trace: trace.trace_status == view_filter)
            helper_header = _("No %s address bounced yet!", model_name)
            helper_message = _("Bounce happens when a mailing cannot be delivered (fake address, "
                               "server issues, ...). Check each record to see what went wrong.")
        elif view_filter == 'open':
            found_traces = self.mailing_trace_ids.filtered(lambda trace: trace.trace_status in ('open', 'reply'))
            helper_header = _("No %s opened your mailing yet!", model_name)
            helper_message = _("Come back once your mailing has been sent to track who opened your mailing.")
        elif view_filter == 'delivered':
            found_traces = self.mailing_trace_ids.filtered(lambda trace: trace.trace_status in ('sent', 'open', 'reply'))
            helper_header = _("No %s received your mailing yet!", model_name)
            helper_message = _("Wait until your mailing has been sent to check how many recipients you managed to reach.")
        elif view_filter == 'sent':
            found_traces = self.mailing_trace_ids.filtered(lambda trace: trace.sent_datetime)
        else:
            found_traces = self.env['mailing.trace']
        res_ids = found_traces.mapped('res_id')
        action = {
            'name': model_name,
            'type': 'ir.actions.act_window',
            'view_mode': 'tree',
            'res_model': self.mailing_model_real,
            'domain': [('id', 'in', res_ids)],
            'context': dict(self._context, create=False),
        }
        if helper_header and helper_message:
            action['help'] = Markup('<p class="o_view_nocontent_smiling_face">%s</p><p>%s</p>') % (
                helper_header, helper_message,
            ),
        return action

    def action_view_mailing_contacts(self):
        """Show the mailing contacts who are in a mailing list selected for this mailing."""
        self.ensure_one()
        action = self.env['ir.actions.actions']._for_xml_id('mass_mailing.action_view_mass_mailing_contacts')
        if self.contact_list_ids:
            action['context'] = {
                'default_mailing_list_ids': self.contact_list_ids[0].ids,
                'default_subscription_list_ids': [(0, 0, {'list_id': self.contact_list_ids[0].id})],
            }
        action['domain'] = [('list_ids', 'in', self.contact_list_ids.ids)]
        return action

    @api.model
    def action_fetch_favorites(self, extra_domain=None):
        """Return all mailings set as favorite and skip mailings with empty body.

        Return archived mailing templates as well, so the user can archive the templates
        while keeping using it, without cluttering the Kanban view if they're a lot of
        templates.
        """
        domain = [('favorite', '=', True)]
        if extra_domain:
            domain = expression.AND([domain, extra_domain])

        values_list = self.with_context(active_test=False).search_read(
            domain=domain,
            fields=['id', 'subject', 'body_arch', 'user_id', 'mailing_model_id'],
            order='favorite_date DESC',
        )

        values_list = [
            values for values in values_list
            if not tools.is_html_empty(values['body_arch'])
        ]

        # You see first the mailings without responsible, then your mailings and then the others
        values_list.sort(
            key=lambda values:
            values['user_id'][0] != self.env.user.id if values['user_id'] else -1
        )

        return values_list

    def update_opt_out(self, email, list_ids, value):
        if len(list_ids) > 0:
            model = self.env['mailing.contact'].with_context(active_test=False)
            records = model.search([('email_normalized', '=', tools.email_normalize(email))])
            opt_out_records = self.env['mailing.contact.subscription'].search([
                ('contact_id', 'in', records.ids),
                ('list_id', 'in', list_ids),
                ('opt_out', '!=', value)
            ])

            opt_out_records.write({'opt_out': value})
            message = _('The recipient <strong>unsubscribed from %s</strong> mailing list(s)') \
                if value else _('The recipient <strong>subscribed to %s</strong> mailing list(s)')
            for record in records:
                # filter the list_id by record
                record_lists = opt_out_records.filtered(lambda rec: rec.contact_id.id == record.id)
                if len(record_lists) > 0:
                    record.sudo().message_post(body=message % ', '.join(str(list.name) for list in record_lists.mapped('list_id')))

    # ------------------------------------------------------
    # A/B Test
    # ------------------------------------------------------

    def action_compare_versions(self):
        self.ensure_one()
        if not self.campaign_id:
            raise ValueError(_("No mailing campaign has been found"))
        action = {
            'name': _('A/B Tests'),
            'type': 'ir.actions.act_window',
            'view_mode': 'tree,kanban,form,calendar,graph',
            'res_model': 'mailing.mailing',
            'domain': [('campaign_id', '=', self.campaign_id.id), ('ab_testing_enabled', '=', True), ('mailing_type', '=', self.mailing_type)],
        }
        if self.mailing_type == 'mail':
            action['views'] = [
                (False, 'tree'),
                (False, 'kanban'),
                (self.env.ref('mass_mailing.mailing_mailing_view_form_full_width').id, 'form'),
                (False, 'calendar'),
                (False, 'graph'),
            ]
        return action

    def action_send_winner_mailing(self):
        """Send the winner mailing based on the winner selection field.
        This action is used in 2 cases:
            - When the user clicks on a button to send the winner mailing. There is only one mailing in self
            - When the cron is executed to send winner mailing based on the A/B testing schedule datetime. In this
            case 'self' contains all the mailing for the campaigns so we just need to take the first to determine the
            winner.
        If the winner mailing is computed automatically, we sudo the mailings of the campaign in order to sort correctly
        the mailings based on the selection that can be used with sub-modules like CRM and Sales
        """
        if len(self.campaign_id) != 1:
            raise ValueError(_("To send the winner mailing the same campaign should be used by the mailings"))
        if any(mailing.ab_testing_completed for mailing in self):
            raise ValueError(_("To send the winner mailing the campaign should not have been completed."))
        final_mailing = self[0]
        sorted_by = final_mailing._get_ab_testing_winner_selection()['value']
        if sorted_by != 'manual':
            ab_testing_mailings = final_mailing._get_ab_testing_siblings_mailings().sudo()
            selected_mailings = ab_testing_mailings.filtered(lambda m: m.state == 'done').sorted(sorted_by, reverse=True)
            if selected_mailings:
                final_mailing = selected_mailings[0]
            else:
                raise ValidationError(_("No mailing for this A/B testing campaign has been sent yet! Send one first and try again later."))
        return final_mailing.action_select_as_winner()

    def action_select_as_winner(self):
        self.ensure_one()
        if not self.ab_testing_enabled:
            raise ValueError(_("A/B test option has not been enabled"))
        self.campaign_id.write({
            'ab_testing_completed': True,
        })
        final_mailing = self.copy({
            'ab_testing_pc': 100,
        })
        final_mailing.action_launch()
        action = self.env['ir.actions.act_window']._for_xml_id('mass_mailing.action_ab_testing_open_winner_mailing')
        action['res_id'] = final_mailing.id
        if self.mailing_type == 'mail':
            action['views'] = [
                (self.env.ref('mass_mailing.mailing_mailing_view_form_full_width').id, 'form'),
            ]
        return action

    def _get_ab_testing_description_values(self):
        self.ensure_one()

        other_ab_testing_mailings = self._get_ab_testing_siblings_mailings().filtered(lambda m: m.id != self.id)
        other_ab_testing_pc = sum([mailing.ab_testing_pc for mailing in other_ab_testing_mailings])
        return {
            'mailing': self,
            'ab_testing_winner_selection_description': self._get_ab_testing_winner_selection()['description'],
            'other_ab_testing_pc': other_ab_testing_pc,
            'remaining_ab_testing_pc': 100 - (other_ab_testing_pc + self.ab_testing_pc),
        }

    def _get_ab_testing_siblings_mailings(self):
        return self.campaign_id.mailing_mail_ids.filtered(lambda m: m.ab_testing_enabled)

    def _get_ab_testing_winner_selection(self):
        ab_testing_winner_selection_description = dict(
            self._fields.get('ab_testing_winner_selection').related_field.selection
        ).get(self.ab_testing_winner_selection)
        return {
            'value': self.ab_testing_winner_selection,
            'description': ab_testing_winner_selection_description,
        }

    def _get_default_ab_testing_campaign_values(self, values=None):
        values = values or dict()
        return {
            'ab_testing_schedule_datetime': values.get('ab_testing_schedule_datetime') or self.ab_testing_schedule_datetime,
            'ab_testing_winner_selection': values.get('ab_testing_winner_selection') or self.ab_testing_winner_selection,
            'mailing_mail_ids': self.ids,
            'name': _('A/B Test: %s', values.get('subject') or self.subject or fields.Datetime.now()),
            'user_id': values.get('user_id') or self.user_id.id or self.env.user.id,
        }

    # ------------------------------------------------------
    # Email Sending
    # ------------------------------------------------------

    def _get_opt_out_list(self):
        """ Give list of opt-outed emails, depending on specific model-based
        computation if available.

        :return list: opt-outed emails, preferably normalized (aka not records)
        """
        self.ensure_one()
        opt_out = {}
        target = self.env[self.mailing_model_real]
        if hasattr(self.env[self.mailing_model_name], '_mailing_get_opt_out_list'):
            opt_out = self.env[self.mailing_model_name]._mailing_get_opt_out_list(self)
            _logger.info(
                "Mass-mailing %s targets %s, blacklist: %s emails",
                self, target._name, len(opt_out))
        else:
            _logger.info("Mass-mailing %s targets %s, no opt out list available", self, target._name)
        return opt_out

    def _get_link_tracker_values(self):
        self.ensure_one()
        vals = {'mass_mailing_id': self.id}

        if self.campaign_id:
            vals['campaign_id'] = self.campaign_id.id
        if self.source_id:
            vals['source_id'] = self.source_id.id
        if self.medium_id:
            vals['medium_id'] = self.medium_id.id
        return vals

    def _get_seen_list(self):
        """Returns a set of emails already targeted by current mailing/campaign (no duplicates)"""
        self.ensure_one()
        target = self.env[self.mailing_model_real]

        query = """
            SELECT s.email
              FROM mailing_trace s
              JOIN %(target)s t ON (s.res_id = t.id)
              %(join_domain)s
             WHERE s.email IS NOT NULL
              %(where_domain)s
        """

        if self.ab_testing_enabled:
            query += """
               AND s.campaign_id = %%(mailing_campaign_id)s;
            """
        else:
            query += """
               AND s.mass_mailing_id = %%(mailing_id)s
               AND s.model = %%(target_model)s;
            """
        join_domain, where_domain = self._get_seen_list_extra()
        query = query % {'target': target._table, 'join_domain': join_domain, 'where_domain': where_domain}
        params = {'mailing_id': self.id, 'mailing_campaign_id': self.campaign_id.id, 'target_model': self.mailing_model_real}
        self._cr.execute(query, params)
        seen_list = set(m[0] for m in self._cr.fetchall())
        _logger.info(
            "Mass-mailing %s has already reached %s %s emails", self, len(seen_list), target._name)
        return seen_list

    def _get_seen_list_extra(self):
        return ('', '')

    def _get_mass_mailing_context(self):
        """Returns extra context items with pre-filled blacklist and seen list for massmailing"""
        return {
            'post_convert_links': self._get_link_tracker_values(),
        }

    def _get_recipients(self):
        mailing_domain = self._parse_mailing_domain()
        res_ids = self.env[self.mailing_model_real].search(mailing_domain).ids

        # randomly choose a fragment
        if self.ab_testing_enabled and self.ab_testing_pc < 100:
            contact_nbr = self.env[self.mailing_model_real].search_count(mailing_domain)
            topick = 0
            if contact_nbr:
                topick = max(int(contact_nbr / 100.0 * self.ab_testing_pc), 1)
            if self.campaign_id and self.ab_testing_enabled:
                already_mailed = self.campaign_id._get_mailing_recipients()[self.campaign_id.id]
            else:
                already_mailed = set([])
            remaining = set(res_ids).difference(already_mailed)
            if topick > len(remaining) or (len(remaining) > 0 and topick == 0):
                topick = len(remaining)
            res_ids = random.sample(sorted(remaining), topick)
        return res_ids

    def _get_remaining_recipients(self):
        res_ids = self._get_recipients()
        trace_domain = [('model', '=', self.mailing_model_real)]
        if self.ab_testing_enabled and self.ab_testing_pc == 100:
            trace_domain = expression.AND([trace_domain, [('mass_mailing_id', 'in', self._get_ab_testing_siblings_mailings().ids)]])
        else:
            trace_domain = expression.AND([trace_domain, [
                ('res_id', 'in', res_ids),
                ('mass_mailing_id', '=', self.id),
            ]])
        already_mailed = self.env['mailing.trace'].search_read(trace_domain, ['res_id'])
        done_res_ids = {record['res_id'] for record in already_mailed}
        return [rid for rid in res_ids if rid not in done_res_ids]

    def _get_unsubscribe_oneclick_url(self, email_to, res_id):
        url = werkzeug.urls.url_join(
            self.get_base_url(), 'mail/mailing/%(mailing_id)s/unsubscribe_oneclick?%(params)s' % {
                'mailing_id': self.id,
                'params': werkzeug.urls.url_encode({
                    'res_id': res_id,
                    'email': email_to,
                    'token': self._unsubscribe_token(res_id, email_to),
                }),
            }
        )
        return url

    def _get_unsubscribe_url(self, email_to, res_id):
        url = werkzeug.urls.url_join(
            self.get_base_url(), 'mailing/%(mailing_id)s/confirm_unsubscribe?%(params)s' % {
                'mailing_id': self.id,
                'params': werkzeug.urls.url_encode({
                    'res_id': res_id,
                    'email': email_to,
                    'token': self._unsubscribe_token(res_id, email_to),
                }),
            }
        )
        return url

    def _get_view_url(self, email_to, res_id):
        url = werkzeug.urls.url_join(
            self.get_base_url(), 'mailing/%(mailing_id)s/view?%(params)s' % {
                'mailing_id': self.id,
                'params': werkzeug.urls.url_encode({
                    'res_id': res_id,
                    'email': email_to,
                    'token': self._unsubscribe_token(res_id, email_to),
                }),
            }
        )
        return url

    def action_send_mail(self, res_ids=None):
        author_id = self.env.user.partner_id.id

        # If no recipient is passed, we don't want to use the recipients of the first
        # mailing for all the others
        initial_res_ids = res_ids
        for mailing in self:
            if not initial_res_ids:
                res_ids = mailing._get_remaining_recipients()
            if not res_ids:
                raise UserError(_('There are no recipients selected.'))

            composer_values = {
                'author_id': author_id,
                'attachment_ids': [(4, attachment.id) for attachment in mailing.attachment_ids],
                'body': mailing._prepend_preview(mailing.body_html or '', mailing.preview),
                'subject': mailing.subject,
                'model': mailing.mailing_model_real,
                'email_from': mailing.email_from,
                'record_name': False,
                'composition_mode': 'mass_mail',
                'mass_mailing_id': mailing.id,
                'mailing_list_ids': [(4, l.id) for l in mailing.contact_list_ids],
                'reply_to_force_new': mailing.reply_to_mode == 'new',
                'template_id': None,
                'mail_server_id': mailing.mail_server_id.id,
            }
            if mailing.reply_to_mode == 'new':
                composer_values['reply_to'] = mailing.reply_to

            composer = self.env['mail.compose.message'].with_context(active_ids=res_ids).create(composer_values)
            extra_context = mailing._get_mass_mailing_context()
            composer = composer.with_context(active_ids=res_ids, **extra_context)
            # auto-commit except in testing mode
            auto_commit = not getattr(threading.current_thread(), 'testing', False)
            composer._action_send_mail(auto_commit=auto_commit)
            mailing.write({
                'state': 'done',
                'sent_date': fields.Datetime.now(),
                # send the KPI mail only if it's the first sending
                'kpi_mail_required': not mailing.sent_date,
            })
        return True

    def convert_links(self):
        res = {}
        for mass_mailing in self:
            html = mass_mailing.body_html if mass_mailing.body_html else ''

            vals = {'mass_mailing_id': mass_mailing.id}

            if mass_mailing.campaign_id:
                vals['campaign_id'] = mass_mailing.campaign_id.id
            if mass_mailing.source_id:
                vals['source_id'] = mass_mailing.source_id.id
            if mass_mailing.medium_id:
                vals['medium_id'] = mass_mailing.medium_id.id

            res[mass_mailing.id] = mass_mailing._shorten_links(html, vals, blacklist=['/unsubscribe_from_list', '/view'])

        return res

    @api.model
    def _process_mass_mailing_queue(self):
        mass_mailings = self.search([('state', 'in', ('in_queue', 'sending')), '|', ('schedule_date', '<', fields.Datetime.now()), ('schedule_date', '=', False)])
        for mass_mailing in mass_mailings:
            user = mass_mailing.write_uid or self.env.user
            mass_mailing = mass_mailing.with_context(**user.with_user(user).context_get())
            if len(mass_mailing._get_remaining_recipients()) > 0:
                mass_mailing.state = 'sending'
                mass_mailing.action_send_mail()
            else:
                mass_mailing.write({
                    'state': 'done',
                    'sent_date': fields.Datetime.now(),
                    # send the KPI mail only if it's the first sending
                    'kpi_mail_required': not mass_mailing.sent_date,
                })

        if self.env['ir.config_parameter'].sudo().get_param('mass_mailing.mass_mailing_reports'):
            mailings = self.env['mailing.mailing'].search([
                ('kpi_mail_required', '=', True),
                ('state', '=', 'done'),
                ('sent_date', '<=', fields.Datetime.now() - relativedelta(days=1)),
                ('sent_date', '>=', fields.Datetime.now() - relativedelta(days=5)),
            ])
            if mailings:
                mailings._action_send_statistics()

    # ------------------------------------------------------
    # STATISTICS
    # ------------------------------------------------------

    def _action_send_statistics(self):
        """Send an email to the responsible of each finished mailing with the statistics."""
        self.kpi_mail_required = False

        mails_sudo = self.env['mail.mail'].sudo()
        for mailing in self:
            if mailing.user_id:
                mailing = mailing.with_user(mailing.user_id).with_context(
                    lang=mailing.user_id.lang or self._context.get('lang')
                )
            mailing_type = mailing._get_pretty_mailing_type()
            mail_user = mailing.user_id or self.env.user
            mail_company = mail_user.company_id

            link_trackers = self.env['link.tracker'].search(
                [('mass_mailing_id', '=', mailing.id)]
            ).sorted('count', reverse=True)
            link_trackers_body = self.env['ir.qweb']._render(
                'mass_mailing.mass_mailing_kpi_link_trackers',
                {
                    'object': mailing,
                    'link_trackers': link_trackers,
                    'mailing_type': mailing_type,
                },
            )
            rendering_data = {
                'body': tools.html_sanitize(link_trackers_body),
                'company': mail_company,
                'user': mail_user,
                'display_mobile_banner': True,
                ** mailing._prepare_statistics_email_values(),
            }
            if mail_user.has_group('mass_mailing.group_mass_mailing_user'):
                rendering_data['mailing_report_token'] = self._get_unsubscribe_token(mail_user.id)
                rendering_data['user_id'] = mail_user.id

            rendered_body = self.env['ir.qweb']._render(
                'digest.digest_mail_main',
                rendering_data
            )

            full_mail = self.env['mail.render.mixin']._render_encapsulate(
                'digest.digest_mail_layout',
                rendered_body,
            )

            mail_values = {
                'auto_delete': True,
                'author_id': mail_user.partner_id.id,
                'email_from': mail_user.email_formatted,
                'email_to': mail_user.email_formatted,
                'body_html': full_mail,
                'reply_to': mail_company.email_formatted or mail_user.email_formatted,
                'state': 'outgoing',
                'subject': _('24H Stats of %(mailing_type)s "%(mailing_name)s"',
                             mailing_type=mailing._get_pretty_mailing_type(),
                             mailing_name=mailing.subject
                            ),
            }
            mails_sudo += self.env['mail.mail'].sudo().create(mail_values)
        return mails_sudo

    def _prepare_statistics_email_values(self):
        """Return some statistics that will be displayed in the mailing statistics email.

        Each item in the returned list will be displayed as a table, with a title and
        1, 2 or 3 columns.
        """
        self.ensure_one()
        mailing_type = self._get_pretty_mailing_type()
        kpi = {}
        if self.mailing_type == 'mail':
            kpi = {
                'kpi_fullname': _('Engagement on %(expected)i %(mailing_type)s Sent',
                                  expected=self.expected,
                                  mailing_type=mailing_type
                                 ),
                'kpi_col1': {
                    'value': f'{self.received_ratio}%',
                    'col_subtitle': _('RECEIVED (%i)', self.delivered),
                },
                'kpi_col2': {
                    'value': f'{self.opened_ratio}%',
                    'col_subtitle': _('OPENED (%i)', self.opened),
                },
                'kpi_col3': {
                    'value': f'{self.replied_ratio}%',
                    'col_subtitle': _('REPLIED (%i)', self.replied),
                },
                'kpi_action': None,
                'kpi_name': self.mailing_type,
            }

        random_tip = self.env['digest.tip'].search(
            [('group_id.category_id', '=', self.env.ref('base.module_category_marketing_email_marketing').id)]
        )
        if random_tip:
            random_tip = random.choice(random_tip).tip_description

        formatted_date = tools.format_datetime(
            self.env, self.sent_date, self.user_id.tz, 'MMM dd, YYYY', self.user_id.lang
        ) if self.sent_date else False

        web_base_url = self.get_base_url()

        return {
            'title': _('24H Stats of %(mailing_type)s "%(mailing_name)s"',
                       mailing_type=mailing_type,
                       mailing_name=self.subject
                       ),
            'top_button_label': _('More Info'),
            'top_button_url': url_join(web_base_url, f'/web#id={self.id}&model=mailing.mailing&view_type=form'),
            'kpi_data': [
                kpi,
                {
                    'kpi_fullname': _('Business Benefits on %(expected)i %(mailing_type)s Sent',
                                      expected=self.expected,
                                      mailing_type=mailing_type
                                     ),
                    'kpi_action': None,
                    'kpi_col1': {},
                    'kpi_col2': {},
                    'kpi_col3': {},
                    'kpi_name': 'trace',
                },
            ],
            'tips': [random_tip] if random_tip else False,
            'formatted_date': formatted_date,
        }

    def _get_pretty_mailing_type(self):
        return _('Emails')

    def _get_unsubscribe_token(self, user_id):
        """Generate a secure hash for this user. It allows to opt out from
        mailing reports while keeping some security in that process. """
        return tools.hmac(self.env(su=True), 'mailing-report-deactivated', user_id)

    # ------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------

    def _convert_inline_images_to_urls(self, html_content):
        """
        Find inline base64 encoded images, make an attachement out of
        them and replace the inline image with an url to the attachement.
        Find VML v:image elements, crop their source images, make an attachement
        out of them and replace their source with an url to the attachement.
        """
        root = lxml.html.fromstring(html_content)
        did_modify_body = False

        conversion_info = []  # list of tuples (image: base64 image, node: lxml node, old_url: string or None, original_id))
        with requests.Session() as session:
            for node in root.iter(lxml.etree.Element, lxml.etree.Comment):
                if node.tag == 'img':
                    # Convert base64 images in img tags to attachments.
                    match = image_re.match(node.attrib.get('src', ''))
                    if match:
                        image = match.group(2).encode()  # base64 image as bytes
                        conversion_info.append((image, node, None, int(node.attrib.get('data-original-id') or "0")))
                elif 'base64' in (node.attrib.get('style') or ''):
                    # Convert base64 images in inline styles to attachments.
                    for match in re.findall(r'data:image/[A-Za-z]+;base64,.+?(?=&\#34;|\"|\'|&quot;|\))', node.attrib.get('style')):
                        image = re.sub(r'data:image/[A-Za-z]+;base64,', '', match).encode()  # base64 image as bytes
                        conversion_info.append((image, node, match, int(node.attrib.get('data-original-id') or "0")))
                elif mso_re.match(node.text or ''):
                    # Convert base64 images (in img tags or inline styles) in mso comments to attachments.
                    base64_in_element_regex = re.compile(r"""
                        (?:(?!^)|<)[^<>]*?(data:image/[A-Za-z]+;base64,[^<]+?)(?=&\#34;|\"|'|&quot;|\))(?=[^<]+>)
                    """, re.VERBOSE)
                    for match in re.findall(base64_in_element_regex, node.text):
                        image = re.sub(r'data:image/[A-Za-z]+;base64,', '', match).encode()  # base64 image as bytes
                        conversion_info.append((image, node, match, int(node.attrib.get('data-original-id') or "0")))
                    # Crop VML images.
                    for match in re.findall(r'<v:image[^>]*>', node.text):
                        url = re.search(r'src=\s*\"([^\"]+)\"', match)[1]
                        # Make sure we have an absolute URL by adding a scheme and host if needed.
                        absolute_url = url if '//' in url else f"{self.get_base_url()}{url if url.startswith('/') else f'/{url}'}"
                        target_width_match = re.search(r'width:\s*([0-9\.]+)\s*px', match)
                        target_height_match = re.search(r'height:\s*([0-9\.]+)\s*px', match)
                        if target_width_match and target_height_match:
                            target_width = float(target_width_match[1])
                            target_height = float(target_height_match[1])
                            try:
                                image = self._get_image_by_url(absolute_url, session)
                            except (ImportValidationError, UnidentifiedImageError):
                                # Url invalid or doesn't resolve to a valid image.
                                # Note: We choose to ignore errors so as not to
                                # break the entire process just for one image's
                                # responsive cropping behavior).
                                pass
                            else:
                                image_processor = tools.ImageProcess(image)
                                image = image_processor.crop_resize(target_width, target_height, 0, 0)
                                conversion_info.append((base64.b64encode(image.source), node, url, int(node.attrib.get('data-original-id') or "0")))

        # Apply the changes.
        urls = self._create_attachments_from_inline_images([(image, original_id) for (image, _, _, original_id) in conversion_info])
        for ((image, node, old_url, original_id), new_url) in zip(conversion_info, urls):
            did_modify_body = True
            if node.tag == 'img':
                node.attrib['src'] = new_url
            elif 'base64' in (node.attrib.get('style') or ''):
                node.attrib['style'] = node.attrib['style'].replace(old_url, new_url)
            else:
                node.text = node.text.replace(old_url, new_url)

        if did_modify_body:
            return lxml.html.tostring(root, encoding='unicode')
        return html_content

    def _create_attachments_from_inline_images(self, b64images):
        if not b64images:
            return []

        IrAttachment = self.env['ir.attachment']
        existing_attachments = dict(IrAttachment.search([
            ('res_model', '=', 'mailing.mailing'),
            ('res_id', '=', self.id),
        ]).mapped(lambda record: (record.checksum, record)))

        attachments, vals_for_attachs, checksums = [], [], []
        checksums_set, checksum_original_id, new_attachment_by_checksum = set(), {}, {}
        next_img_id = len(existing_attachments)
        for (b64image, original_id) in b64images:
            checksum = IrAttachment._compute_checksum(base64.b64decode(b64image))
            checksums.append(checksum)
            existing_attach = existing_attachments.get(checksum)
            # Existing_attach can be None, in which case it acts as placeholder
            # for attachment to be created.
            attachments.append(existing_attach)
            if original_id:
                checksum_original_id[checksum] = original_id
            if not existing_attach and not checksum in checksums_set:
                # We create only one attachment per checksum
                vals_for_attachs.append({
                    'datas': b64image,
                    'name': f"image_mailing_{self.id}_{next_img_id}",
                    'type': 'binary',
                    'res_id': self.id,
                    'res_model': 'mailing.mailing',
                    'checksum': checksum,
                })
                checksums_set.add(checksum)
                next_img_id += 1
        for vals in vals_for_attachs:
            if vals['checksum'] in checksum_original_id:
                vals['original_id'] = checksum_original_id[vals['checksum']]
            del vals['checksum']

        new_attachments = iter(IrAttachment.create(vals_for_attachs))
        checksum_iter = iter(checksums)
        # Replace None entries by newly created attachments.
        for i in range(len(attachments)):
            checksum = next(checksum_iter)
            if attachments[i]:
                continue
            if checksum in new_attachment_by_checksum:
                attachments[i] = new_attachment_by_checksum[checksum]
            else:
                attachments[i] = next(new_attachments)
                new_attachment_by_checksum[checksum] = attachments[i]

        urls = []
        for attachment in attachments:
            attachment.generate_access_token()
            urls.append('/web/image/%s?access_token=%s' % (attachment.id, attachment.access_token))

        return urls

    def _get_default_mailing_domain(self):
        mailing_domain = []
        if hasattr(self.env[self.mailing_model_name], '_mailing_get_default_domain'):
            mailing_domain = self.env[self.mailing_model_name]._mailing_get_default_domain(self)

        if self.mailing_type == 'mail' and 'is_blacklisted' in self.env[self.mailing_model_name]._fields:
            mailing_domain = expression.AND([[('is_blacklisted', '=', False)], mailing_domain])

        return mailing_domain

    def _get_image_by_url(self, url, session):
        maxsize = int(tools.config.get("import_image_maxbytes", DEFAULT_IMAGE_MAXBYTES))
        _logger.debug("Trying to import image from URL: %s", url)
        try:
            response = session.get(url, timeout=int(tools.config.get("import_image_timeout", DEFAULT_IMAGE_TIMEOUT)))
            response.raise_for_status()

            if response.headers.get('Content-Length') and int(response.headers['Content-Length']) > maxsize:
                raise ImportValidationError(
                    _("File size exceeds configured maximum (%s bytes)", maxsize)
                )

            content = bytearray()
            for chunk in response.iter_content(DEFAULT_IMAGE_CHUNK_SIZE):
                content += chunk
                if len(content) > maxsize:
                    raise ImportValidationError(
                        _("File size exceeds configured maximum (%s bytes)", maxsize)
                    )

            image = Image.open(io.BytesIO(content))
            w, h = image.size
            if w * h > 42e6:
                raise ImportValidationError(
                    _("Image size excessive, imported images must be smaller than 42 million pixel")
                )

            return content
        except UnidentifiedImageError:
            _logger.warning('This file could not be decoded as an image file.', exc_info=True)
            raise
        except Exception as e:
            _logger.exception(e)
            raise ImportValidationError(_("Could not retrieve URL: %s", url)) from e

    def _parse_mailing_domain(self):
        self.ensure_one()
        try:
            mailing_domain = literal_eval(self.mailing_domain)
        except Exception:
            mailing_domain = [('id', 'in', [])]
        return mailing_domain

    def _unsubscribe_token(self, res_id, email):
        """Generate a secure hash for this mailing list and parameters.

        This is appended to the unsubscription URL and then checked at
        unsubscription time to ensure no malicious unsubscriptions are
        performed.

        :param int res_id:
            ID of the resource that will be unsubscribed.

        :param str email:
            Email of the resource that will be unsubscribed.
        """
        secret = self.env["ir.config_parameter"].sudo().get_param("database.secret")
        token = (self.env.cr.dbname, self.id, int(res_id), tools.ustr(email))
        return hmac.new(secret.encode('utf-8'), repr(token).encode('utf-8'), hashlib.sha512).hexdigest()

```

## File: models\mailing_contact.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models, tools
from odoo.exceptions import UserError
from odoo.osv import expression


class MassMailingContactListRel(models.Model):
    """ Intermediate model between mass mailing list and mass mailing contact
        Indicates if a contact is opted out for a particular list
    """
    _name = 'mailing.contact.subscription'
    _description = 'Mass Mailing Subscription Information'
    _table = 'mailing_contact_list_rel'
    _rec_name = 'contact_id'

    contact_id = fields.Many2one('mailing.contact', string='Contact', ondelete='cascade', required=True)
    list_id = fields.Many2one('mailing.list', string='Mailing List', ondelete='cascade', required=True)
    opt_out = fields.Boolean(string='Opt Out',
                             help='The contact has chosen not to receive mails anymore from this list', default=False)
    unsubscription_date = fields.Datetime(string='Unsubscription Date')
    message_bounce = fields.Integer(related='contact_id.message_bounce', store=False, readonly=False)
    is_blacklisted = fields.Boolean(related='contact_id.is_blacklisted', store=False, readonly=False)

    _sql_constraints = [
        ('unique_contact_list', 'unique (contact_id, list_id)',
         'A mailing contact cannot subscribe to the same mailing list multiple times.')
    ]

    @api.model_create_multi
    def create(self, vals_list):
        now = fields.Datetime.now()
        for vals in vals_list:
            if 'opt_out' in vals and not vals.get('unsubscription_date'):
                vals['unsubscription_date'] = now if vals['opt_out'] else False
            if vals.get('unsubscription_date'):
                vals['opt_out'] = True
        return super().create(vals_list)

    def write(self, vals):
        if 'opt_out' in vals and 'unsubscription_date' not in vals:
            vals['unsubscription_date'] = fields.Datetime.now() if vals['opt_out'] else False
        if vals.get('unsubscription_date'):
            vals['opt_out'] = True
        return super(MassMailingContactListRel, self).write(vals)


class MassMailingContact(models.Model):
    """Model of a contact. This model is different from the partner model
    because it holds only some basic information: name, email. The purpose is to
    be able to deal with large contact list to email without bloating the partner
    base."""
    _name = 'mailing.contact'
    _inherit = ['mail.thread.blacklist']
    _description = 'Mailing Contact'
    _order = 'email'
    _mailing_enabled = True

    def default_get(self, fields_list):
        """ When coming from a mailing list we may have a default_list_ids context
        key. We should use it to create subscription_list_ids default value that
        are displayed to the user as list_ids is not displayed on form view. """
        res = super(MassMailingContact, self).default_get(fields_list)
        if 'subscription_list_ids' in fields_list and not res.get('subscription_list_ids'):
            list_ids = self.env.context.get('default_list_ids')
            if 'default_list_ids' not in res and list_ids and isinstance(list_ids, (list, tuple)):
                res['subscription_list_ids'] = [
                    (0, 0, {'list_id': list_id}) for list_id in list_ids]
        return res

    name = fields.Char()
    company_name = fields.Char(string='Company Name')
    title_id = fields.Many2one('res.partner.title', string='Title')
    email = fields.Char('Email')
    list_ids = fields.Many2many(
        'mailing.list', 'mailing_contact_list_rel',
        'contact_id', 'list_id', string='Mailing Lists')
    subscription_list_ids = fields.One2many(
        'mailing.contact.subscription', 'contact_id', string='Subscription Information')
    country_id = fields.Many2one('res.country', string='Country')
    tag_ids = fields.Many2many('res.partner.category', string='Tags')
    opt_out = fields.Boolean(
        'Opt Out',
        compute='_compute_opt_out', search='_search_opt_out',
        help='Opt out flag for a specific mailing list. '
             'This field should not be used in a view without a unique and active mailing list context.')

    @api.model
    def _search_opt_out(self, operator, value):
        # Assumes operator is '=' or '!=' and value is True or False
        if operator != '=':
            if operator == '!=' and isinstance(value, bool):
                value = not value
            else:
                raise NotImplementedError()

        if 'default_list_ids' in self._context and isinstance(self._context['default_list_ids'], (list, tuple)) and len(self._context['default_list_ids']) == 1:
            [active_list_id] = self._context['default_list_ids']
            contacts = self.env['mailing.contact.subscription'].search([('list_id', '=', active_list_id)])
            return [('id', 'in', [record.contact_id.id for record in contacts if record.opt_out == value])]
        return expression.FALSE_DOMAIN if value else expression.TRUE_DOMAIN

    @api.depends('subscription_list_ids')
    @api.depends_context('default_list_ids')
    def _compute_opt_out(self):
        if 'default_list_ids' in self._context and isinstance(self._context['default_list_ids'], (list, tuple)) and len(self._context['default_list_ids']) == 1:
            [active_list_id] = self._context['default_list_ids']
            for record in self:
                active_subscription_list = record.subscription_list_ids.filtered(lambda l: l.list_id.id == active_list_id)
                record.opt_out = active_subscription_list.opt_out
        else:
            for record in self:
                record.opt_out = False

    def get_name_email(self, name):
        name, email = self.env['res.partner']._parse_partner_name(name)
        if name and not email:
            email = name
        if email and not name:
            name = email
        return name, email

    @api.model_create_multi
    def create(self, vals_list):
        """ Synchronize default_list_ids (currently used notably for computed
        fields) default key with subscription_list_ids given by user when creating
        contacts.

        Those two values have the same purpose, adding a list to to the contact
        either through a direct write on m2m, either through a write on middle
        model subscription.

        This is a bit hackish but is due to default_list_ids key being
        used to compute oupt_out field. This should be cleaned in master but here
        we simply try to limit issues while keeping current behavior. """
        default_list_ids = self._context.get('default_list_ids')
        default_list_ids = default_list_ids if isinstance(default_list_ids, (list, tuple)) else []

        for vals in vals_list:
            if vals.get('list_ids') and vals.get('subscription_list_ids'):
                raise UserError(_('You should give either list_ids, either subscription_list_ids to create new contacts.'))

        if default_list_ids:
            for vals in vals_list:
                if vals.get('list_ids'):
                    continue
                current_list_ids = []
                subscription_ids = vals.get('subscription_list_ids') or []
                for subscription in subscription_ids:
                    if len(subscription) == 3:
                        current_list_ids.append(subscription[2]['list_id'])
                for list_id in set(default_list_ids) - set(current_list_ids):
                    subscription_ids.append((0, 0, {'list_id': list_id}))
                vals['subscription_list_ids'] = subscription_ids

        return super(MassMailingContact, self.with_context(default_list_ids=False)).create(vals_list)

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        """ Cleans the default_list_ids while duplicating mailing contact in context of
        a mailing list because we already have subscription lists copied over for newly
        created contact, no need to add the ones from default_list_ids again """
        if self.env.context.get('default_list_ids'):
            self = self.with_context(default_list_ids=False)
        return super().copy(default)

    @api.model
    def name_create(self, name):
        name, email = self.get_name_email(name)
        contact = self.create({'name': name, 'email': email})
        return contact.name_get()[0]

    @api.model
    def add_to_list(self, name, list_id):
        name, email = self.get_name_email(name)
        contact = self.create({'name': name, 'email': email, 'list_ids': [(4, list_id)]})
        return contact.name_get()[0]

    def _message_get_default_recipients(self):
        return {
            r.id: {
                'partner_ids': [],
                'email_to': ','.join(tools.email_normalize_all(r.email)) or r.email,
                'email_cc': False,
            } for r in self
        }

    def action_add_to_mailing_list(self):
        ctx = dict(self.env.context, default_contact_ids=self.ids)
        action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing.mailing_contact_to_list_action")
        action['view_mode'] = 'form'
        action['target'] = 'new'
        action['context'] = ctx

        return action

    @api.model
    def get_import_templates(self):
        return [{
            'label': _('Import Template for Mailing List Contacts'),
            'template': '/mass_mailing/static/xls/mailing_contact.xls'
        }]

```

## File: models\mailing_contact_subscription.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MassMailingContactListRel(models.Model):
    """ Intermediate model between mass mailing list and mass mailing contact
        Indicates if a contact is opted out for a particular list
    """
    _name = 'mailing.contact.subscription'
    _description = 'Mass Mailing Subscription Information'
    _table = 'mailing_contact_list_rel'
    _rec_name = 'contact_id'
    _order = 'list_id DESC, contact_id DESC'

    contact_id = fields.Many2one('mailing.contact', string='Contact', ondelete='cascade', required=True)
    list_id = fields.Many2one('mailing.list', string='Mailing List', ondelete='cascade', required=True)
    opt_out = fields.Boolean(
        string='Opt Out',
        default=False,
        help='The contact has chosen not to receive mails anymore from this list')
    unsubscription_date = fields.Datetime(string='Unsubscription Date')
    message_bounce = fields.Integer(related='contact_id.message_bounce', store=False, readonly=False)
    is_blacklisted = fields.Boolean(related='contact_id.is_blacklisted', store=False, readonly=False)

    _sql_constraints = [
        ('unique_contact_list', 'unique (contact_id, list_id)',
         'A mailing contact cannot subscribe to the same mailing list multiple times.')
    ]

    @api.model_create_multi
    def create(self, vals_list):
        now = fields.Datetime.now()
        for vals in vals_list:
            if 'opt_out' in vals and 'unsubscription_date' not in vals:
                vals['unsubscription_date'] = now if vals['opt_out'] else False
            if vals.get('unsubscription_date'):
                vals['opt_out'] = True
        return super().create(vals_list)

    def write(self, vals):
        if 'opt_out' in vals and 'unsubscription_date' not in vals:
            vals['unsubscription_date'] = fields.Datetime.now() if vals['opt_out'] else False
        if vals.get('unsubscription_date'):
            vals['opt_out'] = True
        return super(MassMailingContactListRel, self).write(vals)

```

## File: models\mailing_filter.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from ast import literal_eval

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class MailingFilter(models.Model):
    """ This model stores mass mailing or marketing campaign domain as filters
    (quite similar to 'ir.filters' but dedicated to mailing apps). Frequently
    used domains can be reused easily. """
    _name = 'mailing.filter'
    _description = 'Mailing Favorite Filters'
    _order = 'create_date DESC'

    # override create_uid field to display default value while creating filter from 'Configuration' menus
    create_uid = fields.Many2one('res.users', 'Saved by', index=True, readonly=True, default=lambda self: self.env.user)
    name = fields.Char(string='Filter Name', required=True)
    mailing_domain = fields.Char(string='Filter Domain', required=True)
    mailing_model_id = fields.Many2one('ir.model', string='Recipients Model', required=True, ondelete='cascade')
    mailing_model_name = fields.Char(string='Recipients Model Name', related='mailing_model_id.model')

    @api.constrains('mailing_domain', 'mailing_model_id')
    def _check_mailing_domain(self):
        """ Check that if the mailing domain is set, it is a valid one """
        for mailing_filter in self:
            if mailing_filter.mailing_domain != "[]":
                try:
                    self.env[mailing_filter.mailing_model_id.model].search_count(literal_eval(mailing_filter.mailing_domain))
                except:
                    raise ValidationError(
                        _("The filter domain is not valid for this recipients.")
                    )

```

## File: models\mailing_list.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, Command, fields, models
from odoo.exceptions import UserError


class MassMailingList(models.Model):
    """Model of a contact list. """
    _name = 'mailing.list'
    _order = 'name'
    _description = 'Mailing List'
    _mailing_enabled = True
    _order = 'create_date DESC'
    # As this model has their own data merge, avoid to enable the generic data_merge on that model.
    _disable_data_merge = True

    name = fields.Char(string='Mailing List', required=True)
    active = fields.Boolean(default=True)
    contact_count = fields.Integer(compute="_compute_mailing_list_statistics", string='Number of Contacts')
    contact_count_email = fields.Integer(compute="_compute_mailing_list_statistics", string="Number of Emails")
    contact_count_opt_out = fields.Integer(compute="_compute_mailing_list_statistics", string="Number of Opted-out")
    contact_pct_opt_out = fields.Float(compute="_compute_mailing_list_statistics", string="Percentage of Opted-out")
    contact_count_blacklisted = fields.Integer(compute="_compute_mailing_list_statistics", string="Number of Blacklisted")
    contact_pct_blacklisted = fields.Float(compute="_compute_mailing_list_statistics", string="Percentage of Blacklisted")
    contact_pct_bounce = fields.Float(compute="_compute_mailing_list_statistics", string="Percentage of Bouncing")
    contact_ids = fields.Many2many(
        'mailing.contact', 'mailing_contact_list_rel', 'list_id', 'contact_id',
        string='Mailing Lists', copy=False)
    mailing_count = fields.Integer(compute="_compute_mailing_list_count", string="Number of Mailing")
    mailing_ids = fields.Many2many(
        'mailing.mailing', 'mail_mass_mailing_list_rel',
        string='Mass Mailings', copy=False)
    subscription_ids = fields.One2many(
        'mailing.contact.subscription', 'list_id',
        string='Subscription Information',
        copy=True, depends=['contact_ids'])
    is_public = fields.Boolean(
        string='Show In Preferences', default=True,
        help='The mailing list can be accessible by recipients in the subscription '
             'management page to allows them to update their preferences.')

    # ------------------------------------------------------
    # COMPUTE / ONCHANGE
    # ------------------------------------------------------

    def _compute_mailing_list_count(self):
        data = {}
        if self.ids:
            self.env.cr.execute('''
                SELECT mailing_list_id, count(*)
                FROM mail_mass_mailing_list_rel
                WHERE mailing_list_id IN %s
                GROUP BY mailing_list_id''', (tuple(self.ids),))
            data = dict(self.env.cr.fetchall())
        for mailing_list in self:
            mailing_list.mailing_count = data.get(mailing_list._origin.id, 0)

    def _compute_mailing_list_statistics(self):
        """ Computes various statistics for this mailing.list that allow users
        to have a global idea of its quality (based on blacklist, opt-outs, ...).

        As some fields depend on the value of each other (mainly percentages),
        we compute everything in a single method. """

        # 1. Fetch contact data and associated counts (total / blacklist / opt-out)
        contact_statistics_per_mailing = self._fetch_contact_statistics()

        # 2. Fetch bounce data
        # Optimized SQL way of fetching the count of contacts that have
        # at least 1 message bouncing for passed mailing.lists """
        bounce_per_mailing = {}
        if self.ids:
            sql = '''
                SELECT mclr.list_id, COUNT(DISTINCT mc.id)
                FROM mailing_contact mc
                LEFT OUTER JOIN mailing_contact_list_rel mclr
                ON mc.id = mclr.contact_id
                WHERE mc.message_bounce > 0
                AND mclr.list_id in %s
                GROUP BY mclr.list_id
            '''
            self.env.cr.execute(sql, (tuple(self.ids),))
            bounce_per_mailing = dict(self.env.cr.fetchall())

        # 3. Compute and assign all counts / pct fields
        for mailing_list in self:
            contact_counts = contact_statistics_per_mailing.get(mailing_list.id, {})
            for field, value in contact_counts.items():
                if field in self._fields:
                    mailing_list[field] = value

            if mailing_list.contact_count != 0:
                mailing_list.contact_pct_opt_out = 100 * (mailing_list.contact_count_opt_out / mailing_list.contact_count)
                mailing_list.contact_pct_blacklisted = 100 * (mailing_list.contact_count_blacklisted / mailing_list.contact_count)
                mailing_list.contact_pct_bounce = 100 * (bounce_per_mailing.get(mailing_list.id, 0) / mailing_list.contact_count)
            else:
                mailing_list.contact_pct_opt_out = 0
                mailing_list.contact_pct_blacklisted = 0
                mailing_list.contact_pct_bounce = 0

    # ------------------------------------------------------
    # ORM overrides
    # ------------------------------------------------------

    def write(self, vals):
        # Prevent archiving used mailing list
        if 'active' in vals and not vals.get('active'):
            mass_mailings = self.env['mailing.mailing'].search_count([
                ('state', '!=', 'done'),
                ('contact_list_ids', 'in', self.ids),
            ])

            if mass_mailings > 0:
                raise UserError(_("At least one of the mailing list you are trying to archive is used in an ongoing mailing campaign."))

        return super(MassMailingList, self).write(vals)

    def name_get(self):
        return [(list.id, "%s (%s)" % (list.name, list.contact_count)) for list in self]

    def copy(self, default=None):
        self.ensure_one()

        default = dict(default or {},
                       name=_('%s (copy)', self.name),)
        return super(MassMailingList, self).copy(default)

    # ------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------

    def action_open_import(self):
        """Open the mailing list contact import wizard."""
        action = self.env['ir.actions.actions']._for_xml_id('mass_mailing.mailing_contact_import_action')
        action['context'] = {
            **self.env.context,
            'default_mailing_list_ids': self.ids,
            'default_subscription_list_ids': [
                Command.create({'list_id': mailing_list.id})
                for mailing_list in self
            ],
        }
        return action

    def action_send_mailing(self):
        """Open the mailing form view, with the current lists set as recipients."""
        view = self.env.ref('mass_mailing.mailing_mailing_view_form_full_width')
        action = self.env["ir.actions.actions"]._for_xml_id('mass_mailing.mailing_mailing_action_mail')

        action.update({
            'context': {
                **self.env.context,
                'default_contact_list_ids': self.ids,
            },
            'target': 'current',
            'view_type': 'form',
            'views': [(view.id, 'form')],
        })

        return action

    def action_view_contacts(self):
        action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing.action_view_mass_mailing_contacts")
        action['domain'] = [('list_ids', 'in', self.ids)]
        action['context'] = {'default_list_ids': self.ids}
        return action

    def action_view_contacts_email(self):
        action = self.action_view_contacts()
        action['context'] = dict(action.get('context', {}), search_default_filter_valid_email_recipient=1)
        return action

    def action_view_mailings(self):
        action = self.env["ir.actions.actions"]._for_xml_id('mass_mailing.mailing_mailing_action_mail')
        action['domain'] = [('contact_list_ids', 'in', self.ids)]
        action['context'] = {'default_mailing_type': 'mail', 'default_contact_list_ids': self.ids}
        return action

    def action_view_contacts_opt_out(self):
        action = self.env["ir.actions.actions"]._for_xml_id('mass_mailing.action_view_mass_mailing_contacts')
        action['domain'] = [('list_ids', 'in', self.id)]
        action['context'] = {'default_list_ids': self.ids, 'create': False, 'search_default_filter_opt_out': 1}
        return action

    def action_view_contacts_blacklisted(self):
        action = self.env["ir.actions.actions"]._for_xml_id('mass_mailing.action_view_mass_mailing_contacts')
        action['domain'] = [('list_ids', 'in', self.id)]
        action['context'] = {'default_list_ids': self.ids, 'create': False, 'search_default_filter_blacklisted': 1}
        return action

    def action_view_contacts_bouncing(self):
        action = self.env["ir.actions.actions"]._for_xml_id('mass_mailing.action_view_mass_mailing_contacts')
        action['domain'] = [('list_ids', 'in', self.id)]
        action['context'] = {'default_list_ids': self.ids, 'create': False, 'search_default_filter_bounce': 1}
        return action

    def action_merge(self, src_lists, archive):
        """
            Insert all the contact from the mailing lists 'src_lists' to the
            mailing list in 'self'. Possibility to archive the mailing lists
            'src_lists' after the merge except the destination mailing list 'self'.
        """
        # Explation of the SQL query with an example. There are the following lists
        # A (id=4): yti@odoo.com; yti@example.com
        # B (id=5): yti@odoo.com; yti@openerp.com
        # C (id=6): nothing
        # To merge the mailing lists A and B into C, we build the view st that looks
        # like this with our example:
        #
        #  contact_id |           email           | row_number |  list_id |
        # ------------+---------------------------+------------------------
        #           4 | yti@odoo.com              |          1 |        4 |
        #           6 | yti@odoo.com              |          2 |        5 |
        #           5 | yti@example.com           |          1 |        4 |
        #           7 | yti@openerp.com           |          1 |        5 |
        #
        # The row_column is kind of an occurence counter for the email address.
        # Then we create the Many2many relation between the destination list and the contacts
        # while avoiding to insert an existing email address (if the destination is in the source
        # for example)
        self.ensure_one()
        # Put destination is sources lists if not already the case
        src_lists |= self
        self.env.flush_all()
        self.env.cr.execute("""
            INSERT INTO mailing_contact_list_rel (contact_id, list_id)
            SELECT st.contact_id AS contact_id, %s AS list_id
            FROM
                (
                SELECT
                    contact.id AS contact_id,
                    contact.email AS email,
                    list.id AS list_id,
                    row_number() OVER (PARTITION BY email ORDER BY email) AS rn
                FROM
                    mailing_contact contact,
                    mailing_contact_list_rel contact_list_rel,
                    mailing_list list
                WHERE contact.id=contact_list_rel.contact_id
                AND COALESCE(contact_list_rel.opt_out,FALSE) = FALSE
                AND contact.email_normalized NOT IN (select email from mail_blacklist where active = TRUE)
                AND list.id=contact_list_rel.list_id
                AND list.id IN %s
                AND NOT EXISTS
                    (
                    SELECT 1
                    FROM
                        mailing_contact contact2,
                        mailing_contact_list_rel contact_list_rel2
                    WHERE contact2.email = contact.email
                    AND contact_list_rel2.contact_id = contact2.id
                    AND contact_list_rel2.list_id = %s
                    )
                ) st
            WHERE st.rn = 1;""", (self.id, tuple(src_lists.ids), self.id))
        self.env.invalidate_all()
        if archive:
            (src_lists - self).action_archive()

    def close_dialog(self):
        return {'type': 'ir.actions.act_window_close'}

    # ------------------------------------------------------
    # MAILING
    # ------------------------------------------------------

    def _mailing_get_default_domain(self, mailing):
        return [('list_ids', 'in', mailing.contact_list_ids.ids)]

    def _mailing_get_opt_out_list(self, mailing):
        """ Check subscription on all involved mailing lists. If user is opt_out
        on one list but not on another if two users with same email address, one
        opted in and the other one opted out, send the mail anyway. """
        # TODO DBE Fixme : Optimize the following to get real opt_out and opt_in
        subscriptions = self.subscription_ids if self else mailing.contact_list_ids.subscription_ids
        opt_out_contacts = subscriptions.filtered(lambda rel: rel.opt_out).mapped('contact_id.email_normalized')
        opt_in_contacts = subscriptions.filtered(lambda rel: not rel.opt_out).mapped('contact_id.email_normalized')
        opt_out = set(c for c in opt_out_contacts if c not in opt_in_contacts)
        return opt_out

    # ------------------------------------------------------
    # UTILITY
    # ------------------------------------------------------

    def _fetch_contact_statistics(self):
        """ Compute number of contacts matching various conditions.
        (see '_get_contact_count_select_fields' for details)

        Will return a dict under the form:
        {
            42: { # 42 being the mailing list ID
                'contact_count': 52,
                'contact_count_email': 35,
                'contact_count_opt_out': 5,
                'contact_count_blacklisted': 2
            },
            ...
        } """

        res = []
        if self.ids:
            self.env.cr.execute(f'''
                SELECT
                    {','.join(self._get_contact_statistics_fields().values())}
                FROM
                    mailing_contact_list_rel r
                    {self._get_contact_statistics_joins()}
                WHERE list_id IN %s
                GROUP BY
                    list_id;
            ''', (tuple(self.ids), ))
            res = self.env.cr.dictfetchall()

        contact_counts = {}
        for res_item in res:
            mailing_list_id = res_item.pop('mailing_list_id')
            contact_counts[mailing_list_id] = res_item

        for mass_mailing in self:
            # adds default 0 values for ids that don't have statistics
            if mass_mailing.id not in contact_counts:
                contact_counts[mass_mailing.id] = {
                    field: 0
                    for field in mass_mailing._get_contact_statistics_fields()
                }

        return contact_counts

    def _get_contact_statistics_fields(self):
        """ Returns fields and SQL query select path in a dictionnary.
        This is done to be easily overridable in subsequent modules.

        - mailing_list_id             id of the associated mailing.list
        - contact_count:              all contacts
        - contact_count_email:        all valid emails
        - contact_count_opt_out:      all opted-out contacts
        - contact_count_blacklisted:  all blacklisted contacts """

        return {
            'mailing_list_id': 'list_id AS mailing_list_id',
            'contact_count': 'COUNT(*) AS contact_count',
            'contact_count_email': '''
                SUM(CASE WHEN
                        (c.email_normalized IS NOT NULL
                        AND COALESCE(r.opt_out,FALSE) = FALSE
                        AND bl.id IS NULL)
                        THEN 1 ELSE 0 END) AS contact_count_email''',
            'contact_count_opt_out': '''
                SUM(CASE WHEN COALESCE(r.opt_out,FALSE) = TRUE
                    THEN 1 ELSE 0 END) AS contact_count_opt_out''',
            'contact_count_blacklisted': '''
                SUM(CASE WHEN bl.id IS NOT NULL
                THEN 1 ELSE 0 END) AS contact_count_blacklisted'''
        }

    def _get_contact_statistics_joins(self):
        """ Extracted to be easily overridable by sub-modules (such as mass_mailing_sms). """
        return """
            LEFT JOIN mailing_contact c ON (r.contact_id=c.id)
            LEFT JOIN mail_blacklist bl on c.email_normalized = bl.email and bl.active"""

```

## File: models\mailing_trace.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class MailingTrace(models.Model):
    """ MailingTrace models the statistics collected about emails. Those statistics
    are stored in a separated model and table to avoid bloating the mail_mail table
    with statistics values. This also allows to delete emails send with mass mailing
    without loosing the statistics about them.

    Note:: State management / Error codes / Failure types summary

      * trace_status
        'outgoing', 'sent', 'opened', 'replied',
        'error', 'bouce', 'cancel'
      * failure_type
        # generic
        'unknown',
        # mass_mailing
        "mail_email_invalid", "mail_smtp", "mail_email_missing"
        # mass mailing mass mode specific codes
        "mail_bl", "mail_optout", "mail_dup"
        # mass_mailing_sms
        'sms_number_missing', 'sms_number_format', 'sms_credit',
        'sms_server', 'sms_acc'
        # mass_mailing_sms mass mode specific codes
        'sms_blacklist', 'sms_duplicate', 'sms_optout',
      * cancel:
        * mail: set in get_mail_values in composer, if email is blacklisted
          (mail) or in opt_out / seen list (mass_mailing) or email_to is void
          or incorrectly formatted (mass_mailing) - based on mail cancel state
        * sms: set in _prepare_mass_sms_trace_values in composer if sms is
          in cancel state; either blacklisted (sms) or in opt_out / seen list
          (sms);
        * void mail / void sms number -> error (mail_missing, sms_number_missing)
        * invalid mail / invalid sms number -> error (RECIPIENT, sms_number_format)
      * exception: set in  _postprocess_sent_message (_postprocess_iap_sent_sms)
        if mail (sms) not sent with failure type, reset if sent;
      * sent: set in _postprocess_sent_message (_postprocess_iap_sent_sms) if
        mail (sms) sent
      * clicked: triggered by add_click
      * opened: triggered by add_click + blank gif (mail) + gateway reply (mail)
      * replied: triggered by gateway reply (mail)
      * bounced: triggered by gateway bounce (mail) or in _prepare_mass_sms_trace_values
        if sms_number_format error when sending sms (sms)
    """
    _name = 'mailing.trace'
    _description = 'Mailing Statistics'
    _rec_name = 'id'
    _order = 'create_date DESC'

    trace_type = fields.Selection([('mail', 'Email')], string='Type', default='mail', required=True)
    display_name = fields.Char(compute='_compute_display_name')
    # mail data
    mail_mail_id = fields.Many2one('mail.mail', string='Mail', index='btree_not_null')
    mail_mail_id_int = fields.Integer(
        string='Mail ID (tech)',
        help='ID of the related mail_mail. This field is an integer field because '
             'the related mail_mail can be deleted separately from its statistics. '
             'However the ID is needed for several action and controllers.',
        index='btree_not_null',
    )
    email = fields.Char(string="Email", help="Normalized email address")
    message_id = fields.Char(string='Message-ID') # email Message-ID (RFC 2392)
    medium_id = fields.Many2one(related='mass_mailing_id.medium_id')
    source_id = fields.Many2one(related='mass_mailing_id.source_id')
    # document
    model = fields.Char(string='Document model', required=True)
    res_id = fields.Many2oneReference(string='Document ID', model_field='model')
    # campaign data
    mass_mailing_id = fields.Many2one('mailing.mailing', string='Mailing', index=True, ondelete='cascade')
    campaign_id = fields.Many2one(
        related='mass_mailing_id.campaign_id',
        string='Campaign',
        store=True, readonly=True, index='btree_not_null')
    # Status
    sent_datetime = fields.Datetime('Sent On')
    open_datetime = fields.Datetime('Opened On')
    reply_datetime = fields.Datetime('Replied On')
    trace_status = fields.Selection(selection=[
        ('outgoing', 'Outgoing'),
        ('sent', 'Sent'),
        ('open', 'Opened'),
        ('reply', 'Replied'),
        ('bounce', 'Bounced'),
        ('error', 'Exception'),
        ('cancel', 'Canceled')], string='Status', default='outgoing')
    failure_type = fields.Selection(selection=[
        # generic
        ("unknown", "Unknown error"),
        # mail
        ("mail_email_invalid", "Invalid email address"),
        ("mail_email_missing", "Missing email address"),
        ("mail_smtp", "Connection failed (outgoing mail server problem)"),
        # mass mode
        ("mail_bl", "Blacklisted Address"),
        ("mail_optout", "Opted Out"),
        ("mail_dup", "Duplicated Email"),
    ], string='Failure type')
    # Link tracking
    links_click_ids = fields.One2many('link.tracker.click', 'mailing_trace_id', string='Links click')
    links_click_datetime = fields.Datetime('Clicked On', help='Stores last click datetime in case of multi clicks.')

    _sql_constraints = [
        # Required on a Many2one reference field is not sufficient as actually
        # writing 0 is considered as a valid value, because this is an integer field.
        # We therefore need a specific constraint check.
        ('check_res_id_is_set',
         'CHECK(res_id IS NOT NULL AND res_id !=0 )',
         'Traces have to be linked to records with a not null res_id.')
    ]

    @api.depends('trace_type', 'mass_mailing_id')
    def _compute_display_name(self):
        for trace in self:
            trace.display_name = '%s: %s (%s)' % (trace.trace_type, trace.mass_mailing_id.name, trace.id)

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            if 'mail_mail_id' in values:
                values['mail_mail_id_int'] = values['mail_mail_id']
        return super(MailingTrace, self).create(values_list)

    def action_view_contact(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': self.model,
            'target': 'current',
            'res_id': self.res_id
        }

    def set_sent(self, domain=None):
        traces = self + (self.search(domain) if domain else self.env['mailing.trace'])
        traces.write({'trace_status': 'sent', 'sent_datetime': fields.Datetime.now(), 'failure_type': False})
        return traces

    def set_opened(self, domain=None):
        """ Reply / Open are a bit shared in various processes: reply implies
        open, click implies open. Let us avoid status override by skipping traces
        that are not already opened or replied. """
        traces = self + (self.search(domain) if domain else self.env['mailing.trace'])
        traces.filtered(lambda t: t.trace_status not in ('open', 'reply')).write({'trace_status': 'open', 'open_datetime': fields.Datetime.now()})
        return traces

    def set_clicked(self, domain=None):
        traces = self + (self.search(domain) if domain else self.env['mailing.trace'])
        traces.write({'links_click_datetime': fields.Datetime.now()})
        return traces

    def set_replied(self, domain=None):
        traces = self + (self.search(domain) if domain else self.env['mailing.trace'])
        traces.write({'trace_status': 'reply', 'reply_datetime': fields.Datetime.now()})
        return traces

    def set_bounced(self, domain=None):
        traces = self + (self.search(domain) if domain else self.env['mailing.trace'])
        traces.write({'trace_status': 'bounce'})
        return traces

    def set_failed(self, domain=None, failure_type=False):
        traces = self + (self.search(domain) if domain else self.env['mailing.trace'])
        traces.write({'trace_status': 'error', 'failure_type': failure_type})
        return traces

    def set_canceled(self, domain=None):
        traces = self + (self.search(domain) if domain else self.env['mailing.trace'])
        traces.write({'trace_status': 'cancel'})
        return traces

```

## File: models\mail_mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
import werkzeug.urls

from odoo import api, fields, models, tools


class MailMail(models.Model):
    """Add the mass mailing campaign data to mail"""
    _inherit = ['mail.mail']

    mailing_id = fields.Many2one('mailing.mailing', string='Mass Mailing')
    mailing_trace_ids = fields.One2many('mailing.trace', 'mail_mail_id', string='Statistics')

    def _get_tracking_url(self):
        token = tools.hmac(self.env(su=True), 'mass_mailing-mail_mail-open', self.id)
        return werkzeug.urls.url_join(self.get_base_url(), 'mail/track/%s/%s/blank.gif' % (self.id, token))

    def _send_prepare_body(self):
        """ Override to add the tracking URL to the body and to add
        trace ID in shortened urls """
        # TDE: temporary addition (mail was parameter) due to semi-new-API
        self.ensure_one()
        body = super(MailMail, self)._send_prepare_body()

        if self.mailing_id and body and self.mailing_trace_ids:
            for match in set(re.findall(tools.URL_REGEX, self.body_html)):
                href = match[0]
                url = match[1]

                parsed = werkzeug.urls.url_parse(url, scheme='http')

                if parsed.scheme.startswith('http') and parsed.path.startswith('/r/'):
                    new_href = href.replace(url, url + '/m/' + str(self.mailing_trace_ids[0].id))
                    body = body.replace(href, new_href)

            # generate tracking URL
            tracking_url = self._get_tracking_url()
            body = tools.append_content_to_html(
                body,
                '<img src="%s"/>' % tracking_url,
                plaintext=False,
            )

        body = self.env['mail.render.mixin']._replace_local_links(body)

        return body

    def _send_prepare_values(self, partner=None):
        # TDE: temporary addition (mail was parameter) due to semi-new-API
        res = super(MailMail, self)._send_prepare_values(partner)
        if self.mailing_id and res.get('email_to'):
            base_url = self.mailing_id.get_base_url()
            emails = tools.email_split(res.get('email_to')[0])
            email_to = emails and emails[0] or False

            unsubscribe_url = self.mailing_id._get_unsubscribe_url(email_to, self.res_id)
            unsubscribe_oneclick_url = self.mailing_id._get_unsubscribe_oneclick_url(email_to, self.res_id)
            view_url = self.mailing_id._get_view_url(email_to, self.res_id)

            # replace links in body
            if not tools.is_html_empty(res.get('body')):
                if f'{base_url}/unsubscribe_from_list' in res['body']:
                    res['body'] = res['body'].replace(
                        f'{base_url}/unsubscribe_from_list',
                        unsubscribe_url,
                    )
                if f'{base_url}/view' in res.get('body'):
                    res['body'] = res['body'].replace(
                        f'{base_url}/view',
                        view_url,
                    )

            # add headers
            res.setdefault("headers", {}).update({
                'List-Unsubscribe': f'<{unsubscribe_oneclick_url}>',
                'List-Unsubscribe-Post': 'List-Unsubscribe=One-Click',
                'Precedence': 'list',
                'X-Auto-Response-Suppress': 'OOF',  # avoid out-of-office replies from MS Exchange
            })
        return res

    def _postprocess_sent_message(self, success_pids, failure_reason=False, failure_type=None):
        mail_sent = not failure_type  # we consider that a recipient error is a failure with mass mailling and show them as failed
        for mail in self:
            if mail.mailing_id:
                if mail_sent is True and mail.mailing_trace_ids:
                    mail.mailing_trace_ids.set_sent()
                elif mail_sent is False and mail.mailing_trace_ids:
                    mail.mailing_trace_ids.set_failed(failure_type=failure_type)
        return super(MailMail, self)._postprocess_sent_message(success_pids, failure_reason=failure_reason, failure_type=failure_type)

```

## File: models\mail_render_mixin.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models


class MailRenderMixin(models.AbstractModel):
    _inherit = "mail.render.mixin"

    @api.model
    def _render_template_postprocess(self, rendered):
        # super will transform relative url to absolute
        rendered = super(MailRenderMixin, self)._render_template_postprocess(rendered)

        # apply shortener after
        if self.env.context.get('post_convert_links'):
            for res_id, html in rendered.items():
                rendered[res_id] = self._shorten_links(
                    html,
                    self.env.context['post_convert_links'],
                    blacklist=['/unsubscribe_from_list', '/view']
                )
        return rendered

```

## File: models\mail_thread.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import datetime

from odoo import api, models, fields, tools

BLACKLIST_MAX_BOUNCED_LIMIT = 5


class MailThread(models.AbstractModel):
    """ Update MailThread to add the support of bounce management in mass mailing traces. """
    _inherit = 'mail.thread'

    @api.model
    def _message_route_process(self, message, message_dict, routes):
        """ Override to update the parent mailing traces. The parent is found
        by using the References header of the incoming message and looking for
        matching message_id in mailing.trace. """
        if routes:
            # even if 'reply_to' in ref (cfr mail/mail_thread) that indicates a new thread redirection
            # (aka bypass alias configuration in gateway) consider it as a reply for statistics purpose
            thread_references = message_dict['references'] or message_dict['in_reply_to']
            msg_references = tools.mail_header_msgid_re.findall(thread_references)
            if msg_references:
                self.env['mailing.trace'].set_opened(domain=[('message_id', 'in', msg_references)])
                self.env['mailing.trace'].set_replied(domain=[('message_id', 'in', msg_references)])
        return super(MailThread, self)._message_route_process(message, message_dict, routes)

    def message_post_with_template(self, template_id, **kwargs):
        # avoid having message send through `message_post*` methods being implicitly considered as
        # mass-mailing
        no_massmail = self.with_context(
            default_mass_mailing_name=False,
            default_mass_mailing_id=False,
        )
        return super(MailThread, no_massmail).message_post_with_template(template_id, **kwargs)

    @api.model
    def _routing_handle_bounce(self, email_message, message_dict):
        """ In addition, an auto blacklist rule check if the email can be blacklisted
        to avoid sending mails indefinitely to this email address.
        This rule checks if the email bounced too much. If this is the case,
        the email address is added to the blacklist in order to avoid continuing
        to send mass_mail to that email address. If it bounced too much times
        in the last month and the bounced are at least separated by one week,
        to avoid blacklist someone because of a temporary mail server error,
        then the email is considered as invalid and is blacklisted."""
        super(MailThread, self)._routing_handle_bounce(email_message, message_dict)

        bounced_email = message_dict['bounced_email']
        bounced_msg_id = message_dict['bounced_msg_id']
        bounced_partner = message_dict['bounced_partner']

        if bounced_msg_id:
            self.env['mailing.trace'].set_bounced(domain=[('message_id', 'in', bounced_msg_id)])
        if bounced_email:
            three_months_ago = fields.Datetime.to_string(datetime.datetime.now() - datetime.timedelta(weeks=13))
            stats = self.env['mailing.trace'].search(['&', '&', ('trace_status', '=', 'bounce'), ('write_date', '>', three_months_ago), ('email', '=ilike', bounced_email)]).mapped('write_date')
            if len(stats) >= BLACKLIST_MAX_BOUNCED_LIMIT and (not bounced_partner or any(p.message_bounce >= BLACKLIST_MAX_BOUNCED_LIMIT for p in bounced_partner)):
                if max(stats) > min(stats) + datetime.timedelta(weeks=1):
                    blacklist_rec = self.env['mail.blacklist'].sudo()._add(bounced_email)
                    blacklist_rec._message_log(
                        body='This email has been automatically blacklisted because of too much bounced.')

    @api.model
    def message_new(self, msg_dict, custom_values=None):
        """ Overrides mail_thread message_new that is called by the mailgateway
            through message_process.
            This override updates the document according to the email.
        """
        defaults = {}

        if isinstance(self, self.pool['utm.mixin']):
            thread_references = msg_dict.get('references', '') or msg_dict.get('in_reply_to', '')
            msg_references = tools.mail_header_msgid_re.findall(thread_references)
            if msg_references:
                traces = self.env['mailing.trace'].search([('message_id', 'in', msg_references)], limit=1)
                if traces:
                    defaults['campaign_id'] = traces.campaign_id.id
                    defaults['source_id'] = traces.mass_mailing_id.source_id.id
                    defaults['medium_id'] = traces.mass_mailing_id.medium_id.id

        if custom_values:
            defaults.update(custom_values)

        return super(MailThread, self).message_new(msg_dict, custom_values=defaults)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class ResCompany(models.Model):
    _inherit = "res.company"

    def _get_social_media_links(self):
        self.ensure_one()
        return {
            'social_facebook': self.social_facebook,
            'social_linkedin': self.social_linkedin,
            'social_twitter': self.social_twitter,
            'social_instagram': self.social_instagram
        }

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    group_mass_mailing_campaign = fields.Boolean(
        string="Mailing Campaigns",
        implied_group='mass_mailing.group_mass_mailing_campaign',
        help="""This is useful if your marketing campaigns are composed of several emails""")
    mass_mailing_outgoing_mail_server = fields.Boolean(
        string="Dedicated Server",
        config_parameter='mass_mailing.outgoing_mail_server',
        help='Use a specific mail server in priority. Otherwise Odoo relies on the first outgoing mail server available (based on their sequencing) as it does for normal mails.')
    mass_mailing_mail_server_id = fields.Many2one(
        'ir.mail_server', string='Mail Server',
        config_parameter='mass_mailing.mail_server_id')
    show_blacklist_buttons = fields.Boolean(
        string="Blacklist Option when Unsubscribing",
        config_parameter='mass_mailing.show_blacklist_buttons',
        help="""Allow the recipient to manage themselves their state in the blacklist via the unsubscription page.""")
    mass_mailing_reports = fields.Boolean(
        string='24H Stat Mailing Reports',
        config_parameter='mass_mailing.mass_mailing_reports',
        help='Check how well your mailing is doing a day after it has been sent.')

    @api.onchange('mass_mailing_outgoing_mail_server')
    def _onchange_mass_mailing_outgoing_mail_server(self):
        if not self.mass_mailing_outgoing_mail_server:
            self.mass_mailing_mail_server_id = False

    def set_values(self):
        super().set_values()
        ab_test_cron = self.env.ref('mass_mailing.ir_cron_mass_mailing_ab_testing').sudo()
        if ab_test_cron and ab_test_cron.active != self.group_mass_mailing_campaign:
            ab_test_cron.active = self.group_mass_mailing_campaign

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class Partner(models.Model):
    _inherit = 'res.partner'
    _mailing_enabled = True

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _


class Users(models.Model):
    _name = 'res.users'
    _inherit = ['res.users']

    @api.model
    def systray_get_activities(self):
        """ Update systray name of mailing.mailing from "Mass Mailing"
            to "Email Marketing".
        """
        activities = super(Users, self).systray_get_activities()
        for activity in activities:
            if activity.get('model') == 'mailing.mailing':
                activity['name'] = _('Email Marketing')
                break
        return activities

```

## File: models\utm_campaign.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'

    mailing_mail_ids = fields.One2many(
        'mailing.mailing', 'campaign_id',
        domain=[('mailing_type', '=', 'mail')],
        string='Mass Mailings',
        groups="mass_mailing.group_mass_mailing_user")
    mailing_mail_count = fields.Integer('Number of Mass Mailing',
        compute="_compute_mailing_mail_count",
        groups="mass_mailing.group_mass_mailing_user")
    is_mailing_campaign_activated = fields.Boolean(compute="_compute_is_mailing_campaign_activated")

    # A/B Testing
    ab_testing_mailings_count = fields.Integer("A/B Test Mailings #", compute="_compute_mailing_mail_count")
    ab_testing_completed = fields.Boolean("A/B Testing Campaign Finished", copy=False)
    ab_testing_schedule_datetime = fields.Datetime('Send Final On',
        default=lambda self: fields.Datetime.now() + relativedelta(days=1),
        help="Date that will be used to know when to determine and send the winner mailing")
    ab_testing_total_pc = fields.Integer("Total A/B test percentage", compute="_compute_ab_testing_total_pc", store=True)
    ab_testing_winner_selection = fields.Selection([
        ('manual', 'Manual'),
        ('opened_ratio', 'Highest Open Rate'),
        ('clicks_ratio', 'Highest Click Rate'),
        ('replied_ratio', 'Highest Reply Rate')], string="Winner Selection", default="opened_ratio",
        help="Selection to determine the winner mailing that will be sent.")

    # stat fields
    received_ratio = fields.Integer(compute="_compute_statistics", string='Received Ratio')
    opened_ratio = fields.Integer(compute="_compute_statistics", string='Opened Ratio')
    replied_ratio = fields.Integer(compute="_compute_statistics", string='Replied Ratio')
    bounced_ratio = fields.Integer(compute="_compute_statistics", string='Bounced Ratio')

    @api.depends('mailing_mail_ids')
    def _compute_ab_testing_total_pc(self):
        for campaign in self:
            campaign.ab_testing_total_pc = sum([
                mailing.ab_testing_pc for mailing in campaign.mailing_mail_ids.filtered('ab_testing_enabled')
            ])

    @api.depends('mailing_mail_ids')
    def _compute_mailing_mail_count(self):
        if self.ids:
            mailing_data = self.env['mailing.mailing']._read_group(
                [('campaign_id', 'in', self.ids), ('mailing_type', '=', 'mail')],
                ['campaign_id', 'ab_testing_enabled'],
                ['campaign_id', 'ab_testing_enabled'],
                lazy=False,
            )
            ab_testing_mapped_data = {}
            mapped_data = {}
            for data in mailing_data:
                if data['ab_testing_enabled']:
                    ab_testing_mapped_data.setdefault(data['campaign_id'][0], []).append(data['__count'])
                mapped_data.setdefault(data['campaign_id'][0], []).append(data['__count'])
        else:
            mapped_data = dict()
            ab_testing_mapped_data = dict()
        for campaign in self:
            campaign.mailing_mail_count = sum(mapped_data.get(campaign._origin.id or campaign.id, []))
            campaign.ab_testing_mailings_count = sum(ab_testing_mapped_data.get(campaign._origin.id or campaign.id, []))

    @api.constrains('ab_testing_total_pc', 'ab_testing_completed')
    def _check_ab_testing_total_pc(self):
        for campaign in self:
            if not campaign.ab_testing_completed and campaign.ab_testing_total_pc >= 100:
                raise ValidationError(_("The total percentage for an A/B testing campaign should be less than 100%"))

    def _compute_statistics(self):
        """ Compute statistics of the mass mailing campaign """
        default_vals = {
            'received_ratio': 0,
            'opened_ratio': 0,
            'replied_ratio': 0,
            'bounced_ratio': 0
        }
        if not self.ids:
            self.update(default_vals)
            return
        self.env.cr.execute("""
            SELECT
                c.id as campaign_id,
                COUNT(s.id) AS expected,
                COUNT(s.sent_datetime) AS sent,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status in ('sent', 'open', 'reply')) AS delivered,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status in ('open', 'reply')) AS open,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status = 'reply') AS reply,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status = 'bounce') AS bounce,
                COUNT(s.trace_status) FILTER (WHERE s.trace_status = 'cancel') AS cancel
            FROM
                mailing_trace s
            RIGHT JOIN
                utm_campaign c
                ON (c.id = s.campaign_id)
            WHERE
                c.id IN %s
            GROUP BY
                c.id
        """, (tuple(self.ids), ))

        all_stats = self.env.cr.dictfetchall()
        stats_per_campaign = {
            stats['campaign_id']: stats
            for stats in all_stats
        }

        for campaign in self:
            stats = stats_per_campaign.get(campaign.id)
            if not stats:
                vals = default_vals
            else:
                total = (stats['expected'] - stats['cancel']) or 1
                delivered = stats['sent'] - stats['bounce']
                vals = {
                    'received_ratio': 100.0 * delivered / total,
                    'opened_ratio': 100.0 * stats['open'] / total,
                    'replied_ratio': 100.0 * stats['reply'] / total,
                    'bounced_ratio': 100.0 * stats['bounce'] / total
                }

            campaign.update(vals)

    def _compute_is_mailing_campaign_activated(self):
        self.is_mailing_campaign_activated = self.env.user.has_group('mass_mailing.group_mass_mailing_campaign')

    def _get_mailing_recipients(self, model=None):
        """Return the recipients of a mailing campaign. This is based on the statistics
        build for each mailing. """
        res = dict.fromkeys(self.ids, {})
        for campaign in self:
            domain = [('campaign_id', '=', campaign.id)]
            if model:
                domain += [('model', '=', model)]
            res[campaign.id] = set(self.env['mailing.trace'].search(domain).mapped('res_id'))
        return res

    @api.model
    def _cron_process_mass_mailing_ab_testing(self):
        """ Cron that manages A/B testing and sends a winner mailing computed based on
        the value set on the A/B testing campaign.
        In case there is no mailing sent for an A/B testing campaign we ignore this campaign
        """
        ab_testing_campaign = self.search([
            ('ab_testing_schedule_datetime', '<=', fields.Datetime.now()),
            ('ab_testing_winner_selection', '!=', 'manual'),
            ('ab_testing_completed', '=', False),
        ])
        for campaign in ab_testing_campaign:
            ab_testing_mailings = campaign.mailing_mail_ids.filtered(lambda m: m.ab_testing_enabled)
            if not ab_testing_mailings.filtered(lambda m: m.state == 'done'):
                continue
            ab_testing_mailings.action_send_winner_mailing()
        return ab_testing_campaign

```

## File: models\utm_medium.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models

from odoo.exceptions import UserError


class UtmMedium(models.Model):
    _inherit = 'utm.medium'

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_mailings(self):
        """ Already handled by ondelete='restrict', but let's show a nice error message """
        linked_mailings = self.env['mailing.mailing'].sudo().search([
            ('medium_id', 'in', self.ids)
        ])

        if linked_mailings:
            raise UserError(_(
                "You cannot delete these UTM Mediums as they are linked to the following mailings in "
                "Mass Mailing:\n%(mailing_names)s",
                mailing_names=', '.join(['"%s"' % subject for subject in linked_mailings.mapped('subject')])))

```

## File: models\utm_source.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models

from odoo.exceptions import UserError


class UtmSource(models.Model):
    _inherit = 'utm.source'

    @api.ondelete(at_uninstall=False)
    def _unlink_except_linked_mailings(self):
        """ Already handled by ondelete='restrict', but let's show a nice error message """
        linked_mailings = self.env['mailing.mailing'].sudo().search([
            ('source_id', 'in', self.ids)
        ])

        if linked_mailings:
            raise UserError(_(
                "You cannot delete these UTM Sources as they are linked to the following mailings in "
                "Mass Mailing:\n%(mailing_names)s",
                mailing_names=', '.join(['"%s"' % subject for subject in linked_mailings.mapped('subject')])))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import ir_mail_server
from . import ir_model
from . import link_tracker
from . import mailing_contact_subscription
from . import mailing_contact
from . import mailing_list
from . import mailing_trace
from . import mailing
from . import mailing_filter
from . import mail_mail
from . import mail_render_mixin
from . import mail_thread
from . import res_company
from . import res_config_settings
from . import res_partner
from . import res_users
from . import utm_campaign
from . import utm_medium
from . import utm_source

```

## File: report\mailing_trace_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, tools


class MailingTraceReport(models.Model):
    _name = 'mailing.trace.report'
    _auto = False
    _description = 'Mass Mailing Statistics'

    # mailing
    name = fields.Char(string='Mass Mail', readonly=True)
    mailing_type = fields.Selection([('mail', 'Mail')], string='Type', default='mail', required=True)
    campaign = fields.Char(string='Mailing Campaign', readonly=True)
    scheduled_date = fields.Datetime(string='Scheduled Date', readonly=True)
    state = fields.Selection(
        [('draft', 'Draft'), ('test', 'Tested'), ('done', 'Sent')],
        string='Status', readonly=True)
    email_from = fields.Char('From', readonly=True)
    # traces
    scheduled = fields.Integer(readonly=True)
    sent = fields.Integer(readonly=True)
    delivered = fields.Integer(readonly=True)
    error = fields.Integer(readonly=True)
    opened = fields.Integer(readonly=True)
    replied = fields.Integer(readonly=True)
    bounced = fields.Integer(readonly=True)
    canceled = fields.Integer(readonly=True)
    clicked = fields.Integer(readonly=True)

    def init(self):
        """Mass Mail Statistical Report: based on mailing.trace that models the various
        statistics collected for each mailing, and mailing.mailing model that models the
        various mailing performed. """
        tools.drop_view_if_exists(self.env.cr, 'mailing_trace_report')
        self.env.cr.execute(self._report_get_request())

    def _report_get_request(self):
        sql_select = 'SELECT %s' % ', '.join(self._report_get_request_select_items())
        sql_from = 'FROM %s' % ' '.join(self._report_get_request_from_items())
        sql_where_items = self._report_get_request_where_items()
        if sql_where_items and len(sql_where_items) == 1:
            sql_where = 'WHERE %s' % sql_where_items[0]
        elif sql_where_items:
            sql_where = 'WHERE %s' % ' AND '.join(sql_where_items)
        else:
            sql_where = ''
        sql_group_by = 'GROUP BY %s' % ', '.join(self._report_get_request_group_by_items())
        return f"CREATE OR REPLACE VIEW mailing_trace_report AS ({sql_select} {sql_from} {sql_where} {sql_group_by} )"

    def _report_get_request_select_items(self):
        return [
            'min(trace.id) as id',
            'utm_source.name as name',
            'mailing.mailing_type',
            'utm_campaign.name as campaign',
            'trace.create_date as scheduled_date',
            'mailing.state',
            'mailing.email_from',
            "COUNT(trace.id) as scheduled",
            'COUNT(trace.sent_datetime) as sent',
            "(COUNT(trace.id) - COUNT(trace.trace_status) FILTER (WHERE trace.trace_status IN ('error', 'bounce', 'cancel'))) as delivered",
            "COUNT(trace.trace_status) FILTER (WHERE trace.trace_status = 'error') as error",
            "COUNT(trace.trace_status) FILTER (WHERE trace.trace_status = 'bounce') as bounced",
            "COUNT(trace.trace_status) FILTER (WHERE trace.trace_status = 'cancel') as canceled",
            "COUNT(trace.trace_status) FILTER (WHERE trace.trace_status = 'open') as opened",
            "COUNT(trace.trace_status) FILTER (WHERE trace.trace_status = 'reply') as replied",
            "COUNT(trace.links_click_datetime) as clicked",
        ]

    def _report_get_request_from_items(self):
        return [
            'mailing_trace as trace',
            'LEFT JOIN mailing_mailing as mailing ON (trace.mass_mailing_id=mailing.id)',
            'LEFT JOIN utm_campaign as utm_campaign ON (mailing.campaign_id = utm_campaign.id)',
            'LEFT JOIN utm_source as utm_source ON (mailing.source_id = utm_source.id)'
        ]

    def _report_get_request_where_items(self):
        return []

    def _report_get_request_group_by_items(self):
        return [
            'trace.create_date',
            'utm_source.name',
            'utm_campaign.name',
            'mailing.mailing_type',
            'mailing.state',
            'mailing.email_from'
        ]

```

## File: report\mailing_trace_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="mailing_trace_report_view_tree" model="ir.ui.view">
            <field name="name">mailing.trace.report.view.tree</field>
            <field name="model">mailing.trace.report</field>
            <field name="arch" type="xml">
                <tree string="Mass Mailing Statistics" sample="1">
                    <field name="name"/>
                    <field name="campaign" groups="mass_mailing.group_mass_mailing_campaign"/>
                    <field name="mailing_type" invisible="1"/>
                    <field name="scheduled_date" string="Scheduled On"/>
                    <field name="state"/>
                    <field name="scheduled"/>
                    <field name="sent"/>
                    <field name="delivered"/>
                    <field name="opened"/>
                    <field name="replied"/>
                    <field name="clicked"/>
                    <field name="canceled" optional="hide"/>
                    <field name="error" optional="hide"/>
                    <field name="bounced" optional="hide"/>
                </tree>
            </field>
        </record>

        <record id="mailing_trace_report_view_pivot" model="ir.ui.view">
            <field name="name">mailing.trace.report.view.pivot</field>
            <field name="model">mailing.trace.report</field>
            <field name="arch" type="xml">
                <pivot string="Mass Mailing Statistics" disable_linking="1" sample="1">
                    <field name="name" type="row"/>
                    <field name="sent" type="measure"/>
                    <field name="scheduled" type="measure"/>
                    <field name="delivered" type="measure"/>
                    <field name="opened" type="measure"/>
                    <field name="replied" type="measure"/>
                    <field name="clicked" type="measure"/>
                    <field name="canceled"/>
                    <field name="error"/>
                    <field name="bounced"/>
                </pivot>
            </field>
        </record>

        <record id="mailing_trace_report_view_graph" model="ir.ui.view">
            <field name="name">mailing.trace.report.view.graph</field>
            <field name="model">mailing.trace.report</field>
            <field name="arch" type="xml">
                <graph string="Mass Mailing Statistics" disable_linking="1" sample="1">
                    <field name="name"/>
                    <field name="sent" type="measure"/>
                    <field name="replied"/>
                    <field name="clicked"/>
                </graph>
            </field>
        </record>

        <record id="mailing_trace_report_view_search" model="ir.ui.view">
            <field name="name">mailing.trace.report.view.search</field>
            <field name="model">mailing.trace.report</field>
            <field name="arch" type="xml">
                <search string="Mass Mailing Statistics">
                    <field name="name" string="Mailing"/>
                    <field name="campaign" string="Campaign" groups="mass_mailing.group_mass_mailing_campaign"/>
                    <filter name="filter_scheduled_date" date="scheduled_date"/>
                    <group expand="0" string="Extended Filters...">
                        <field name="scheduled_date"/>
                    </group>
                    <group expand="1" string="Group By...">
                        <filter string="Mass Mailing Campaign" domain="[]" name="mass_mailing_campaign"
                            context="{'group_by':'campaign'}" groups="mass_mailing.group_mass_mailing_campaign"/>
                        <filter string="State" domain="[]" name="state"
                            context="{'group_by':'state'}"/>
                        <filter string="Sent By" domain="[]" name="sent_by"
                            context="{'group_by':'email_from'}"/>
                        <separator/>
                        <filter string="Scheduled Period" name="scheduled_date"
                            domain="[]" context="{'group_by':'scheduled_date'}"/>
                    </group>
                </search>
            </field>
        </record>

        <!-- Actions and Menuitems -->
       <record id="mailing_trace_report_action_mail" model="ir.actions.act_window">
           <field name="name">Mass Mailing Analysis</field>
           <field name="res_model">mailing.trace.report</field>
           <field name="domain">[('mailing_type', '=', 'mail')]</field>
           <field name="view_mode">graph,pivot,tree</field>
           <field name="help" type="html">
<p class="o_view_nocontent_smiling_face">
    Mass Mailing Statistics allows you to check different mailing related information
    like number of bounced mails, opened mails, replied mails. You can sort out
    your analysis by different groups to get accurate grained analysis.
</p>
</field>
       </record>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mailing_trace_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_utm_tag_mass_mailing_campaign,utm.tag,utm.model_utm_tag,mass_mailing.group_mass_mailing_campaign,1,1,1,1
access_mailing_contact_mm_user,access.mailing.contact.mm.user,model_mailing_contact,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_contact_import_mailing_user,access.mailing.contact.import.mailing.user,model_mailing_contact_import,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_contact_subscription_mm_user,access.mailing.contact.subscription.mm.user,model_mailing_contact_subscription,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_list_mm_user,access.mailing.list.mm.user,model_mailing_list,mass_mailing.group_mass_mailing_user,1,1,1,1
access_utm_stage,utm.stage,utm.model_utm_stage,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_mailing_mm_user,access.mailing.mailing.mm.user,model_mailing_mailing,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_mailing_system,access.mailing.mailing.system,model_mailing_mailing,base.group_system,1,1,1,1
access_mailing_trace_user,mailing.trace.user,model_mailing_trace,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_trace_mm_user,access.mailing.trace.mm.user,model_mailing_trace,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_trace_report_mm_user,access.mailing.trace.report.mm.user,model_mailing_trace_report,mass_mailing.group_mass_mailing_user,1,0,0,0
access_utm_campaign_mass_mailing_user,utm.campaign,utm.model_utm_campaign,mass_mailing.group_mass_mailing_user,1,1,1,1
access_utm_medium,access_utm_source,utm.model_utm_medium,mass_mailing.group_mass_mailing_user,1,1,1,1
access_utm_source,access_utm_source,utm.model_utm_source,mass_mailing.group_mass_mailing_user,1,1,1,1
access_ir_mail_server,access_ir_mail_server,base.model_ir_mail_server,mass_mailing.group_mass_mailing_user,1,0,0,0
access_ir_model,access_ir_model,base.model_ir_model,mass_mailing.group_mass_mailing_user,1,0,0,0
access_mail_blacklist_mass_mailing_user,access.mail.blacklist.mass_mailing_user,mail.model_mail_blacklist,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mail_blacklist_remove_mass_mailing_user,acesss.mail.blacklist.remove.mass_mailing_user,mail.model_mail_blacklist_remove,mass_mailing.group_mass_mailing_user,1,1,1,1
access_link_tracker_mailing,access.link.tracker.mailing,link_tracker.model_link_tracker,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_list_merge,access.mailing.list.merge,model_mailing_list_merge,mass_mailing.group_mass_mailing_user,1,1,1,0
access_mailing_mailing_test,access.mailing.mailing.test,model_mailing_mailing_test,mass_mailing.group_mass_mailing_user,1,1,1,0
access_mailing_contact_to_list,access.mailing.contact.to.list,model_mailing_contact_to_list,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_schedule_date,access.mailing.schedule.date,model_mailing_mailing_schedule_date,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_filter_mailing_user,access.mailing.filter.mailing.user,model_mailing_filter,mass_mailing.group_mass_mailing_user,1,1,1,1

```

## File: security\res_groups_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="group_mass_mailing_user" model="res.groups">
        <field name="name">User</field>
        <field name="category_id" ref="base.module_category_marketing_email_marketing"/>
        <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
    </record>

    <!-- Group to manage campaigns -->
    <record id="group_mass_mailing_campaign" model="res.groups">
        <field name="name">Manage Mass Mailing Campaigns</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>

    <data noupdate="1">
        <record id="group_mass_mailing_user" model="res.groups">
            <field name="implied_ids" eval="[(4, ref('mail.group_mail_template_editor'))]"/>
        </record>
    </data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient><path id="d" d="M16.995 35.708c-2.743 1.538-2.48 5.47.455 6.638L26.564 46c.611 7.72 1.423 11.72 2.436 12 .948.262 3.602-2.35 7.962-7.835l8.923 3.57c.47.185.965.278 1.46.278.653 0 1.299-.163 1.881-.48a3.736 3.736 0 0 0 1.906-2.665C54.969 28.136 56.548 16.513 55.868 16c-.77-.582-13.728 5.987-38.873 19.708zm13.397 16.813v-4.99l2.918 1.166-2.918 3.824zm16.952-2.217L35.08 45.398l11.18-15.631c.853-1.198-.758-2.589-1.89-1.638l-16.865 14.24-8.596-3.446 33.172-18.544-4.737 29.925z"/><path id="e" d="M16.995 33.708c-2.743 1.538-2.48 5.47.455 6.638L26.564 44c.611 7.72 1.423 11.72 2.436 12 .948.262 3.602-2.35 7.962-7.835l8.923 3.57c.47.185.965.278 1.46.278.653 0 1.299-.163 1.881-.48a3.736 3.736 0 0 0 1.906-2.665C54.969 26.136 56.548 14.513 55.868 14c-.77-.582-13.728 5.987-38.873 19.708zm13.397 16.813v-4.99l2.918 1.166-2.918 3.824zm16.952-2.217L35.08 43.398l11.18-15.631c.853-1.198-.758-2.589-1.89-1.638l-16.865 14.24-8.596-3.446 33.172-18.544-4.737 29.925z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M40.1 69H4c-2 0-4-.146-4-4.078V47.679L15.8 35 54 17l-3.062 32.732L40.1 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
```

## File: static\image_shapes\basic\circle.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="800" height="600">
	<defs>
		<clipPath id="clip-path" clipPathUnits="objectBoundingBox">
			<use xlink:href="#filterPath" fill="none"></use>
		</clipPath>
		<path id="filterPath" d="M0.5,1C-0.1667,0.9795-0.1667,0.0203,0.5,0C1.1667,0.0205,1.1667,0.9797,0.5,1z"></path>
	</defs>
	<svg viewBox="0 0 1 1" id="preview" preserveAspectRatio="none">
		<use xlink:href="#filterPath" fill="darkgrey"></use>
	</svg>
	<image xlink:href="" clip-path="url(#clip-path)"></image>
</svg>

```

## File: static\image_shapes\basic\slanted.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="800" height="600">

<defs><clipPath id="clip-path" clipPathUnits="objectBoundingBox"><use xlink:href="#filterPath" fill="none"></use></clipPath><path id="filterPath" d="M0.4128,0h0.5872L0.5872,1H0L0.4128,0z"></path></defs><svg viewBox="0 0 1 1" id="preview" preserveAspectRatio="none"><use xlink:href="#filterPath" fill="darkgrey"></use></svg><image xlink:href="" clip-path="url(#clip-path)"></image></svg>

```

## File: static\image_shapes\basic\triangle.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="800" height="600">

<defs><clipPath id="clip-path" clipPathUnits="objectBoundingBox"><use xlink:href="#filterPath" fill="none"></use></clipPath><path id="filterPath" d="M1,1L0,0h1V1z"></path></defs><svg viewBox="0 0 1 1" id="preview" preserveAspectRatio="none"><use xlink:href="#filterPath" fill="darkgrey"></use></svg><image xlink:href="" clip-path="url(#clip-path)"></image></svg>

```

## File: static\src\img\snippets_options\align_bottom.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="20" height="12" viewBox="0 0 20 12">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy" transform="translate(-202 -5)">
      <g class="align_bottom" transform="translate(202 5)">
        <rect width="8" height="12" x="12" fill="#B8B8B8" class="o_subdle"/>
        <polygon fill="#B8B8B8" points="0 0 9 0 9 1 0 1" class="o_subdle"/>
        <rect width="9" height="6" y="6" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\align_bottom_right.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="20" height="12" viewBox="0 0 20 12">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_4" transform="translate(-203 -5)">
      <g class="align_bottom_right" transform="translate(203 5)">
        <rect width="8" height="12" fill="#B8B8B8" class="o_subdle"/>
        <rect width="9" height="6" x="11" y="6" fill="#FFF" class="o_graphic"/>
        <polygon fill="#B8B8B8" points="11 0 20 0 20 1 11 1" class="o_subdle"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\align_middle.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="20" height="12" viewBox="0 0 20 12">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy" transform="translate(-172 -5)">
      <g class="align_middle" transform="translate(172 5)">
        <rect width="8" height="12" x="12" fill="#B8B8B8" class="o_subdle"/>
        <polygon fill="#B8B8B8" points="0 0 9 0 9 1 0 1" class="o_subdle"/>
        <rect width="9" height="6" y="3" fill="#FFF" class="o_graphic"/>
        <polygon fill="#B8B8B8" points="0 11 9 11 9 12 0 12" class="o_subdle"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\align_middle_right.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="20" height="12" viewBox="0 0 20 12">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_4" transform="translate(-173 -5)">
      <g class="align_middle_right" transform="translate(173 5)">
        <rect width="8" height="12" fill="#B8B8B8" class="o_subdle"/>
        <path fill="#B8B8B8" d="M20 11v1h-9v-1h9zM11 0h9v1h-9V0z" class="o_subdle"/>
        <rect width="9" height="6" x="11" y="3" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\align_stretch.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="20" height="12" viewBox="0 0 20 12">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy" transform="translate(-234 -5)">
      <g class="align_stretch" transform="translate(234 5)">
        <rect width="8" height="12" x="12" fill="#B8B8B8" class="o_subdle"/>
        <rect width="8" height="12" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\align_top.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="20" height="12" viewBox="0 0 20 12">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy" transform="translate(-139 -5)">
      <g class="align_top" transform="translate(139 5)">
        <rect width="8" height="12" x="12" fill="#B8B8B8" class="o_subdle"/>
        <polygon fill="#B8B8B8" points="0 11 9 11 9 12 0 12" class="o_subdle"/>
        <rect width="9" height="6" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\align_top_right.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="20" height="12" viewBox="0 0 20 12">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_4" transform="translate(-139 -5)">
      <g class="align_top_right" transform="translate(139 5)">
        <rect width="8" height="12" fill="#B8B8B8" class="o_subdle"/>
        <rect width="9" height="1" x="11" y="11" fill="#B8B8B8" class="o_subdle"/>
        <rect width="9" height="6" x="11" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\content_width_full.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="8" viewBox="0 0 23 8">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g fill="#FFF" class="3_buttons_copy_2" transform="translate(-228 -7)">
      <g class="content_width_full" transform="translate(228 7)">
        <path d="M23 0v8H0V0h23zm-5 1v2.5h-3v1h3V7l3-3-3-3zM5 1L2 4l3 3V4.5h3v-1H5V1z" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\content_width_normal.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="8" viewBox="0 0 23 8">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_2" transform="translate(-185 -7)">
      <g class="content_width_normal" transform="translate(185 7)">
        <rect width="23" height="8" class="spacer"/>
        <rect width="13" height="8" x="5" fill="#FFF" class="o_graphic"/>
        <polygon fill="#D8D8D8" points="2 0 2 8 3 8 3 0" class="o_subdle"/>
        <polygon fill="#D8D8D8" points="20 0 20 8 21 8 21 0" class="o_subdle"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\content_width_small.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="8" viewBox="0 0 23 8">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g fill="#FFF" class="3_buttons_copy_2" transform="translate(-141 -7)">
      <g class="content_width_small" transform="translate(141 7)">
        <rect width="7" height="8" x="8" class="o_graphic"/>
        <polygon fill-rule="nonzero" points="3 1 6 4 3 7 3 4.5 0 4.5 0 3.5 3 3.5" class="o_graphic"/>
        <polygon fill-rule="nonzero" points="20 1 17 4 20 7 20 4.5 23 4.5 23 3.5 20 3.5" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\image_left.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="11" viewBox="0 0 24 11">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_5" transform="translate(-137 -5)">
      <g class="image_left" transform="translate(137 5)">
        <rect width="8" height="11" x="16" fill="#B8B8B8" class="o_subdle"/>
        <path fill="#FFF" d="M0 0h13v11H0V0zm1 1h11v9H1V1zm3.438 2.286c0 .357-.119.66-.356.91s-.525.375-.863.375c-.339 0-.627-.125-.864-.375A1.275 1.275 0 0 1 2 3.286c0-.357.118-.661.355-.911S2.88 2 3.22 2c.338 0 .626.125.863.375s.356.554.356.91zm6.5 2.571v3H2V7.571L4.031 5.43 5.047 6.5l3.25-3.429 2.64 2.786z" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\image_right.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="11" viewBox="0 0 24 11">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_5" transform="translate(-171 -5)">
      <g class="image_right" transform="translate(171 5)">
        <rect width="8" height="11" fill="#B8B8B8" class="o_subdle"/>
        <path fill="#FFF" d="M11 0h13v11H11V0zm1 1h11v9H12V1zm3.438 2.286c0 .357-.119.66-.356.91s-.525.375-.863.375c-.339 0-.627-.125-.864-.375a1.275 1.275 0 0 1-.355-.91c0-.357.118-.661.355-.911S13.88 2 14.22 2c.338 0 .626.125.863.375s.355.554.355.91zm6.5 2.571v3H13V7.571l2.031-2.142L16.047 6.5l3.25-3.429 2.64 2.786z" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\masonry_template_alternate_image_text.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <linearGradient id="s_masonry_gradient_1" gradientUnits="userSpaceOnUse" x1="82" y1="52" x2="75" y2="45">
            <stop offset="0%" stop-color="#008374"/>
            <stop offset="100%" stop-color="#006a59"/>
        </linearGradient>
        <linearGradient id="s_masonry_gradient_2" gradientUnits="userSpaceOnUse" x1="42" y1="42" x2="22" y2="55">
            <stop offset="0%" stop-color="#00aa89"/>
            <stop offset="100%" stop-color="#009989"/>
        </linearGradient>
    </defs>
    <!-- Background -->
    <rect x="0" y="5" width="240" height="60" fill="#9ccde4"/>
    <!-- Image -->
    <circle cx="45" cy="20" r="6" fill="#f3ed63"/>
    <g transform="translate(-50, 0)">
        <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
        <path fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    </g>

    <!-- Text -->
    <rect x="60" y="5" width="60" height="60" fill="#ffffff"/>
    <rect x="75" y="30" width="30" height="2" fill="#333333"/>
    <rect x="75" y="35" width="30" height="1" fill="#777777"/>
    <rect x="75" y="38" width="30" height="1" fill="#777777"/>
    <rect x="75" y="41" width="20" height="1" fill="#777777"/>

    <!-- Image -->
    <circle cx="165" cy="20" r="6" fill="#f3ed63"/>
    <mask id="o_masonry_mask_3">
        <rect x="50" y="5" width="60" height="60" fill="#ffffff"/>
    </mask>
    <g transform="translate(70, 0)">
        <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
        <path mask="url(#o_masonry_mask_3)" fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    </g>

    <!-- Text -->
    <rect x="180" y="5" width="60" height="60" fill="#ffffff"/>
    <rect x="195" y="30" width="30" height="2" fill="#333333"/>
    <rect x="195" y="35" width="30" height="1" fill="#777777"/>
    <rect x="195" y="38" width="30" height="1" fill="#777777"/>
    <rect x="195" y="41" width="20" height="1" fill="#777777"/>

</svg>

```

## File: static\src\img\snippets_options\masonry_template_alternate_texts.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <!-- Text -->
    <rect x="0" y="5" width="60" height="60" fill="#ffffff"/>
    <rect x="15" y="30" width="30" height="2" fill="#333333"/>
    <rect x="15" y="35" width="30" height="1" fill="#777777"/>
    <rect x="15" y="38" width="30" height="1" fill="#777777"/>
    <rect x="15" y="41" width="20" height="1" fill="#777777"/>
    <!-- Text -->
    <rect x="60" y="5" width="60" height="60" fill="#c9ecfc"/>
    <rect x="75" y="30" width="30" height="2" fill="#333333"/>
    <rect x="75" y="35" width="30" height="1" fill="#777777"/>
    <rect x="75" y="38" width="30" height="1" fill="#777777"/>
    <rect x="75" y="41" width="20" height="1" fill="#777777"/>
    <!-- Text -->
    <rect x="120" y="5" width="60" height="60" fill="#ffffff"/>
    <rect x="135" y="30" width="30" height="2" fill="#333333"/>
    <rect x="135" y="35" width="30" height="1" fill="#777777"/>
    <rect x="135" y="38" width="30" height="1" fill="#777777"/>
    <rect x="135" y="41" width="20" height="1" fill="#777777"/>
    <!-- Text -->
    <rect x="180" y="5" width="60" height="60" fill="#c9ecfc"/>
    <rect x="195" y="30" width="30" height="2" fill="#333333"/>
    <rect x="195" y="35" width="30" height="1" fill="#777777"/>
    <rect x="195" y="38" width="30" height="1" fill="#777777"/>
    <rect x="195" y="41" width="20" height="1" fill="#777777"/>
</svg>

```

## File: static\src\img\snippets_options\masonry_template_alternate_text_image.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <linearGradient id="s_masonry_gradient_1" gradientUnits="userSpaceOnUse" x1="82" y1="52" x2="75" y2="45">
            <stop offset="0%" stop-color="#008374"/>
            <stop offset="100%" stop-color="#006a59"/>
        </linearGradient>
        <linearGradient id="s_masonry_gradient_2" gradientUnits="userSpaceOnUse" x1="42" y1="42" x2="22" y2="55">
            <stop offset="0%" stop-color="#00aa89"/>
            <stop offset="100%" stop-color="#009989"/>
        </linearGradient>
    </defs>
    <!-- Background -->
    <rect x="0" y="5" width="240" height="60" fill="#9ccde4"/>
    <!-- Text -->
    <rect x="0" y="5" width="60" height="60" fill="#ffffff"/>
    <rect x="15" y="30" width="30" height="2" fill="#333333"/>
    <rect x="15" y="35" width="30" height="1" fill="#777777"/>
    <rect x="15" y="38" width="30" height="1" fill="#777777"/>
    <rect x="15" y="41" width="20" height="1" fill="#777777"/>
    <!-- Image -->
    <circle cx="105" cy="20" r="6" fill="#f3ed63"/>
    <mask id="o_masonry_mask_1">
        <rect x="50" y="5" width="60" height="60" fill="#ffffff"/>
    </mask>
    <g transform="translate(10, 0)">
        <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
        <path mask="url(#o_masonry_mask_1)" fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    </g>
    <!-- Text -->
    <rect x="120" y="5" width="60" height="60" fill="#ffffff"/>
    <rect x="135" y="30" width="30" height="2" fill="#333333"/>
    <rect x="135" y="35" width="30" height="1" fill="#777777"/>
    <rect x="135" y="38" width="30" height="1" fill="#777777"/>
    <rect x="135" y="41" width="20" height="1" fill="#777777"/>
    <!-- Image -->
    <circle cx="225" cy="20" r="6" fill="#f3ed63"/>
    <mask id="o_masonry_mask_2">
        <rect x="50" y="5" width="60" height="60" fill="#ffffff"/>
    </mask>
    <g transform="translate(130, 0)">
        <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
        <path mask="url(#o_masonry_mask_2)" fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    </g>
</svg>

```

## File: static\src\img\snippets_options\masonry_template_alternate_text_image_text.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <linearGradient id="s_masonry_gradient_1" gradientUnits="userSpaceOnUse" x1="82" y1="52" x2="75" y2="45">
            <stop offset="0%" stop-color="#008374"/>
            <stop offset="100%" stop-color="#006a59"/>
        </linearGradient>
        <linearGradient id="s_masonry_gradient_2" gradientUnits="userSpaceOnUse" x1="42" y1="42" x2="22" y2="55">
            <stop offset="0%" stop-color="#00aa89"/>
            <stop offset="100%" stop-color="#009989"/>
        </linearGradient>
    </defs>
    <!-- Text -->
    <rect x="0" y="5" width="60" height="60" fill="#ffffff"/>
    <rect x="15" y="30" width="30" height="2" fill="#333333"/>
    <rect x="15" y="35" width="30" height="1" fill="#777777"/>
    <rect x="15" y="38" width="30" height="1" fill="#777777"/>
    <rect x="15" y="41" width="20" height="1" fill="#777777"/>
    <!-- Image -->
    <rect x="60" y="5" width="120" height="60" fill="#9ccde4"/>
    <circle cx="160" cy="25" r="6" fill="#f3ed63"/>
    <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z" transform="translate(60, 0)"/>
    <path fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z" transform="translate(60, 0)"/>
    <!-- Text -->
    <rect x="180" y="5" width="60" height="60" fill="#ffffff"/>
    <rect x="195" y="30" width="30" height="2" fill="#333333"/>
    <rect x="195" y="35" width="30" height="1" fill="#777777"/>
    <rect x="195" y="38" width="30" height="1" fill="#777777"/>
    <rect x="195" y="41" width="20" height="1" fill="#777777"/>
</svg>

```

## File: static\src\img\snippets_options\masonry_template_default.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <linearGradient id="s_masonry_gradient_1" gradientUnits="userSpaceOnUse" x1="82" y1="52" x2="75" y2="45">
            <stop offset="0%" stop-color="#008374"/>
            <stop offset="100%" stop-color="#006a59"/>
        </linearGradient>
        <linearGradient id="s_masonry_gradient_2" gradientUnits="userSpaceOnUse" x1="42" y1="42" x2="22" y2="55">
            <stop offset="0%" stop-color="#00aa89"/>
            <stop offset="100%" stop-color="#009989"/>
        </linearGradient>
    </defs>
    <!-- Image -->
    <rect x="0" y="5" width="120" height="60" fill="#9ccde4"/>
    <circle cx="105" cy="20" r="6" fill="#f3ed63"/>
    <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
    <path fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    <!-- Background -->
    <rect x="120" y="5" width="120" height="60" fill="#c9ecfc"/>
    <!-- Text -->
    <rect x="120" y="5" width="60" height="30" fill="#ffffff"/>
    <rect x="135" y="16" width="30" height="2" fill="#222222"/>
    <rect x="135" y="21" width="30" height="1" fill="#666666"/>
    <rect x="135" y="24" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="135" y="46" width="30" height="2" fill="#222222"/>
    <rect x="135" y="51" width="30" height="1" fill="#666666"/>
    <rect x="135" y="54" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="195" y="16" width="30" height="2" fill="#333333"/>
    <rect x="195" y="21" width="30" height="1" fill="#666666"/>
    <rect x="195" y="24" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="180" y="35" width="60" height="30" fill="#ffffff"/>
    <rect x="195" y="46" width="30" height="2" fill="#333333"/>
    <rect x="195" y="51" width="30" height="1" fill="#666666"/>
    <rect x="195" y="54" width="20" height="1" fill="#666666"/>
</svg>

```

## File: static\src\img\snippets_options\masonry_template_images.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <linearGradient id="s_masonry_gradient_1" gradientUnits="userSpaceOnUse" x1="82" y1="52" x2="75" y2="45">
            <stop offset="0%" stop-color="#008374"/>
            <stop offset="100%" stop-color="#006a59"/>
        </linearGradient>
        <linearGradient id="s_masonry_gradient_2" gradientUnits="userSpaceOnUse" x1="42" y1="42" x2="22" y2="55">
            <stop offset="0%" stop-color="#00aa89"/>
            <stop offset="100%" stop-color="#009989"/>
        </linearGradient>
    </defs>
    <!-- Background -->
    <rect x="0" y="5" width="240" height="60" fill="#9ccde4"/>
    <!-- Image -->
    <circle cx="105" cy="20" r="6" fill="#f3ed63"/>
    <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
    <path fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    <!-- Image -->
    <circle cx="225" cy="20" r="6" fill="#f3ed63"/>
    <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z" transform="translate(120, 0)"/>
    <path fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z" transform="translate(120, 0)"/>
    <!-- Separator -->
    <line x1="119" y1="5" x2="119" y2="65" stroke-width="2" stroke="#ffffff"/>
</svg>

```

## File: static\src\img\snippets_options\masonry_template_image_texts_image.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <linearGradient id="s_masonry_gradient_1" gradientUnits="userSpaceOnUse" x1="82" y1="52" x2="75" y2="45">
            <stop offset="0%" stop-color="#008374"/>
            <stop offset="100%" stop-color="#006a59"/>
        </linearGradient>
        <linearGradient id="s_masonry_gradient_2" gradientUnits="userSpaceOnUse" x1="42" y1="42" x2="22" y2="55">
            <stop offset="0%" stop-color="#00aa89"/>
            <stop offset="100%" stop-color="#009989"/>
        </linearGradient>
    </defs>
    <!-- Background -->
    <rect x="0" y="5" width="240" height="60" fill="#9ccde4"/>
    <!-- Image -->
    <circle cx="45" cy="20" r="6" fill="#f3ed63"/>
    <g transform="translate(-50, 0)">
        <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
        <path fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    </g>
    <!-- Text -->
    <rect x="60" y="5" width="60" height="30" fill="#ffffff"/>
    <rect x="75" y="16" width="30" height="2" fill="#333333"/>
    <rect x="75" y="21" width="30" height="1" fill="#666666"/>
    <rect x="75" y="24" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="60" y="35" width="60" height="30" fill="#c9ecfc"/>
    <rect x="75" y="46" width="30" height="2" fill="#333333"/>
    <rect x="75" y="51" width="30" height="1" fill="#666666"/>
    <rect x="75" y="54" width="20" height="1" fill="#666666"/>
    <!-- Image -->
    <circle cx="225" cy="20" r="6" fill="#f3ed63"/>
    <g transform="translate(120, 0)">
        <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
        <path fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    </g>
</svg>

```

## File: static\src\img\snippets_options\masonry_template_mosaic.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <linearGradient id="s_masonry_gradient_1" gradientUnits="userSpaceOnUse" x1="82" y1="52" x2="75" y2="45">
            <stop offset="0%" stop-color="#008374"/>
            <stop offset="100%" stop-color="#006a59"/>
        </linearGradient>
        <linearGradient id="s_masonry_gradient_2" gradientUnits="userSpaceOnUse" x1="42" y1="42" x2="22" y2="55">
            <stop offset="0%" stop-color="#00aa89"/>
            <stop offset="100%" stop-color="#009989"/>
        </linearGradient>
    </defs>
    <!-- Background -->
    <rect x="0" y="5" width="240" height="60" fill="#9ccde4"/>
    <!-- Text -->
    <rect x="0" y="5" width="60" height="30" fill="#c9ecfc"/>
    <rect x="15" y="16" width="30" height="2" fill="#333333"/>
    <rect x="15" y="21" width="30" height="1" fill="#666666"/>
    <rect x="15" y="24" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="60" y="5" width="60" height="30" fill="#ffffff"/>
    <rect x="75" y="16" width="30" height="2" fill="#333333"/>
    <rect x="75" y="21" width="30" height="1" fill="#666666"/>
    <rect x="75" y="24" width="20" height="1" fill="#666666"/>
    <!-- Image -->
    <circle cx="105" cy="48" r="6" fill="#f3ed63"/>
    <mask id="s_masonry_mask_4">
        <rect x="0" y="28" width="120" height="30" fill="#ffffff"/>
    </mask>
    <g transform="translate(0, 7)" mask="url(#s_masonry_mask_4)" >
        <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
        <path fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    </g>


    <!-- Image -->
    <circle cx="225" cy="18" r="6" fill="#f3ed63"/>
    <!-- <mask id="s_masonry_mask_5">
        <rect x="0" y="28" width="120" height="30" fill="#ffffff"/>
    </mask> -->
    <g transform="translate(120, -23)" mask="url(#s_masonry_mask_4)" >
        <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
        <path fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    </g>

    <!-- Text -->
    <rect x="120" y="35" width="60" height="30" fill="#c9ecfc"/>
    <rect x="135" y="46" width="30" height="2" fill="#222222"/>
    <rect x="135" y="51" width="30" height="1" fill="#666666"/>
    <rect x="135" y="54" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="180" y="35" width="60" height="30" fill="#ffffff"/>
    <rect x="195" y="46" width="30" height="2" fill="#333333"/>
    <rect x="195" y="51" width="30" height="1" fill="#666666"/>
    <rect x="195" y="54" width="20" height="1" fill="#666666"/>
</svg>

```

## File: static\src\img\snippets_options\masonry_template_reversed.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <linearGradient id="s_masonry_gradient_1" gradientUnits="userSpaceOnUse" x1="82" y1="52" x2="75" y2="45">
            <stop offset="0%" stop-color="#008374"/>
            <stop offset="100%" stop-color="#006a59"/>
        </linearGradient>
        <linearGradient id="s_masonry_gradient_2" gradientUnits="userSpaceOnUse" x1="42" y1="42" x2="22" y2="55">
            <stop offset="0%" stop-color="#00aa89"/>
            <stop offset="100%" stop-color="#009989"/>
        </linearGradient>
    </defs>
    <!-- Background -->
    <rect x="0" y="5" width="120" height="60" fill="#c9ecfc"/>
    <!-- Text -->
    <rect x="0" y="5" width="60" height="30" fill="#ffffff"/>
    <rect x="15" y="16" width="30" height="2" fill="#222222"/>
    <rect x="15" y="21" width="30" height="1" fill="#666666"/>
    <rect x="15" y="24" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="15" y="46" width="30" height="2" fill="#222222"/>
    <rect x="15" y="51" width="30" height="1" fill="#666666"/>
    <rect x="15" y="54" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="75" y="16" width="30" height="2" fill="#333333"/>
    <rect x="75" y="21" width="30" height="1" fill="#666666"/>
    <rect x="75" y="24" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="60" y="35" width="60" height="30" fill="#ffffff"/>
    <rect x="75" y="46" width="30" height="2" fill="#333333"/>
    <rect x="75" y="51" width="30" height="1" fill="#666666"/>
    <rect x="75" y="54" width="20" height="1" fill="#666666"/>
    <!-- Image -->
    <rect x="120" y="5" width="120" height="60" fill="#9ccde4"/>
    <circle cx="225" cy="20" r="6" fill="#f3ed63"/>
    <path style="s_masonry_gradient_1" fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z" transform="translate(120, 0)"/>
    <path style="s_masonry_gradient_2" fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z" transform="translate(120, 0)"/>
</svg>

```

## File: static\src\img\snippets_options\masonry_template_texts_image_texts.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="240" height="70" viewBox="0 0 240 70">
    <defs>
        <linearGradient id="s_masonry_gradient_1" gradientUnits="userSpaceOnUse" x1="82" y1="52" x2="75" y2="45">
            <stop offset="0%" stop-color="#008374"/>
            <stop offset="100%" stop-color="#006a59"/>
        </linearGradient>
        <linearGradient id="s_masonry_gradient_2" gradientUnits="userSpaceOnUse" x1="42" y1="42" x2="22" y2="55">
            <stop offset="0%" stop-color="#00aa89"/>
            <stop offset="100%" stop-color="#009989"/>
        </linearGradient>
    </defs>
    <!-- Text -->
    <rect x="0" y="5" width="60" height="30" fill="#ffffff"/>
    <rect x="15" y="16" width="30" height="2" fill="#333333"/>
    <rect x="15" y="21" width="30" height="1" fill="#666666"/>
    <rect x="15" y="24" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="0" y="35" width="60" height="30" fill="#c9ecfc"/>
    <rect x="15" y="46" width="30" height="2" fill="#333333"/>
    <rect x="15" y="51" width="30" height="1" fill="#666666"/>
    <rect x="15" y="54" width="20" height="1" fill="#666666"/>
    <!-- Image -->
    <rect x="60" y="5" width="120" height="60" fill="#9ccde4"/>
    <circle cx="160" cy="25" r="6" fill="#f3ed63"/>
    <g transform="translate(60, 0)">
        <path fill="url(#s_masonry_gradient_1)" d="M75.2,50.2c-4.9,0-9.5,1-13.5,2.8c5.1,3.4,9.1,7.5,11.5,12.1h27.6C97.9,56.5,87.5,50.2,75.2,50.2z"/>
        <path fill="url(#s_masonry_gradient_2)" d="M0,65.1h73.2c-6.7-12.9-25.8-22.2-48.5-22.2c-9,0-17.4,1.5-24.7,4C0,46.9,0,65.1,0,65.1z"/>
    </g>
    <!-- Text -->
    <rect x="180" y="5" width="60" height="30" fill="#c9ecfc"/>
    <rect x="195" y="16" width="30" height="2" fill="#333333"/>
    <rect x="195" y="21" width="30" height="1" fill="#666666"/>
    <rect x="195" y="24" width="20" height="1" fill="#666666"/>
    <!-- Text -->
    <rect x="180" y="35" width="60" height="30" fill="#ffffff"/>
    <rect x="195" y="46" width="30" height="2" fill="#333333"/>
    <rect x="195" y="51" width="30" height="1" fill="#666666"/>
    <rect x="195" y="54" width="20" height="1" fill="#666666"/>
</svg>

```

## File: static\src\img\snippets_options\media_layout_1_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="8" viewBox="0 0 23 8">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_3" transform="translate(-170 -7)">
      <g class="media_layout_1_2" transform="translate(170 7)">
        <rect width="10" height="8" x="13" fill="#B8B8B8" class="o_subdle"/>
        <rect width="11" height="8" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\media_layout_1_2_right.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="8" viewBox="0 0 23 8">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_6" transform="translate(-171 -7)">
      <g class="media_layout_1_2_right" transform="translate(171 7)">
        <rect width="10" height="8" fill="#B8B8B8" class="o_subdle"/>
        <rect width="11" height="8" x="12" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\media_layout_1_3.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="8" viewBox="0 0 23 8">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_3" transform="translate(-204 -7)">
      <g class="media_layout_1_3" transform="translate(204 7)">
        <rect width="14" height="8" x="9" fill="#B8B8B8" class="o_subdle"/>
        <rect width="7" height="8" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\media_layout_1_3_right.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="8" viewBox="0 0 23 8">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_6" transform="translate(-204 -7)">
      <g class="media_layout_1_3_right" transform="translate(204 7)">
        <rect width="14" height="8" fill="#B8B8B8" class="o_subdle"/>
        <rect width="7" height="8" x="16" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\media_layout_1_4.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="8" viewBox="0 0 23 8">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_3" transform="translate(-137 -7)">
      <g class="media_layout_1_4" transform="translate(137 7)">
        <rect width="18" height="8" x="5" fill="#B8B8B8" class="o_subdle"/>
        <rect width="3" height="8" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_options\media_layout_1_4_right.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="8" viewBox="0 0 23 8">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_6" transform="translate(-137 -7)">
      <g class="media_layout_1_4_right" transform="translate(137 7)">
        <rect width="18" height="8" fill="#B8B8B8" class="o_subdle"/>
        <rect width="3" height="8" x="20" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_alert.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <rect id="path-2" width="22" height="2" x="3" y="3"/>
    <filter id="filter-3" width="104.5%" height="200%" x="-2.3%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-4" d="M19 11v1H3v-1h16zm4-3v1H3V8h20z"/>
    <filter id="filter-5" width="105%" height="150%" x="-2.5%" y="-12.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_alert">
      <rect width="82" height="60" class="bg"/>
      <g class="group_2" transform="translate(19 22)">
        <rect width="44" height="17" fill="url(#linearGradient-1)" fill-opacity=".4" class="rectangle_2" rx="1"/>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-3)" xlink:href="#path-2"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-2"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
          <use fill="#FFF" fill-opacity=".8" xlink:href="#path-4"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_blockquote.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="12" height="2" x="11" y="0"/>
    <filter id="filter-2" width="108.3%" height="200%" x="-4.2%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <linearGradient id="linearGradient-3" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <path id="path-4" d="M32 8v1H11V8h21zm-5-3v1H11V5h16z"/>
    <filter id="filter-5" width="104.8%" height="150%" x="-2.4%" y="-12.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_blockquote">
      <rect width="82" height="60" class="bg"/>
      <g class="group_2" transform="translate(20 24)">
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-1"/>
        </g>
        <path fill="url(#linearGradient-3)" d="M1.706 5c.349 0 .652-.114.909-.342C2.872 4.43 3 4.164 3 3.86c0-.147-.024-.29-.071-.427a.98.98 0 0 0-.243-.377 1.215 1.215 0 0 0-.472-.278 2.383 2.383 0 0 0-.755-.1h-.42c.05-.494.235-.914.555-1.26.319-.347.76-.651 1.324-.912L2.588 0A4.93 4.93 0 0 0 .76 1.318C.253 1.897 0 2.472 0 3.04c0 .613.147 1.092.441 1.44.295.346.716.519 1.265.519zm4 0c.349 0 .652-.114.909-.342C6.872 4.43 7 4.164 7 3.86c0-.147-.024-.29-.071-.427a.98.98 0 0 0-.243-.377 1.215 1.215 0 0 0-.472-.278 2.383 2.383 0 0 0-.755-.1h-.42c.05-.494.235-.914.555-1.26.319-.347.76-.651 1.324-.912L6.588 0A4.93 4.93 0 0 0 4.76 1.318C4.253 1.897 4 2.472 4 3.04c0 .613.147 1.092.441 1.44.295.346.716.519 1.265.519z"/>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-4"/>
        </g>
        <path fill="url(#linearGradient-3)" d="M36.411 12a4.93 4.93 0 0 0 1.83-1.318c.506-.579.759-1.154.759-1.723 0-.613-.147-1.092-.441-1.44-.295-.346-.716-.519-1.265-.519-.349 0-.652.114-.909.342-.257.228-.385.494-.385.798 0 .147.024.29.071.427a.98.98 0 0 0 .243.377c.12.12.277.212.472.278.194.067.446.1.755.1h.42c-.05.494-.235.914-.555 1.26-.319.347-.76.651-1.324.912l.33.506zm4 0a4.93 4.93 0 0 0 1.83-1.318c.506-.579.759-1.154.759-1.723 0-.613-.147-1.092-.441-1.44-.295-.346-.716-.519-1.265-.519-.349 0-.652.114-.909.342-.257.228-.385.494-.385.798 0 .147.024.29.071.427a.98.98 0 0 0 .243.377c.12.12.277.212.472.278.194.067.446.1.755.1h.42c-.05.494-.235.914-.555 1.26-.319.347-.76.651-1.324.912l.33.506z"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_call_to_action.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="0%" x2="100%" y1="44.17%" y2="55.83%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <rect id="path-2" width="22" height="2" x="0" y="0"/>
    <filter id="filter-3" width="104.5%" height="200%" x="-2.3%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-4" d="M16 8v1H0V8h16zm4-3v1H0V5h20z"/>
    <filter id="filter-5" width="105%" height="150%" x="-2.5%" y="-12.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
    <path id="path-6" d="M53.22 11.566c.15.145.183.312.1.501-.081.193-.224.29-.427.29h-2.771l1.458 3.453a.47.47 0 0 1-.247.61l-1.284.544a.47.47 0 0 1-.61-.247l-1.385-3.279-2.263 2.263a.446.446 0 0 1-.5.102c-.194-.082-.291-.225-.291-.428V4.465c0-.204.097-.347.29-.429a.45.45 0 0 1 .5.102l7.43 7.428z"/>
    <filter id="filter-8" width="112%" height="115.4%" x="-6%" y="-3.8%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 0   0 0 0 0 0   0 0 0 0 0  0 0 0 1 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_call_to_action">
      <rect width="82" height="60" class="bg"/>
      <g fill="url(#linearGradient-1)" class="group" opacity=".4" transform="translate(0 16)">
        <g class="image_1">
          <rect width="82" height="28" class="rectangle"/>
        </g>
      </g>
      <g class="center_group" transform="translate(15 25)">
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-3)" xlink:href="#path-2"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-2"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
          <use fill="#FFF" fill-opacity=".8" xlink:href="#path-4"/>
        </g>
        <rect width="16" height="7" x="36.5" y=".5" fill="#1C1C1C" stroke="#FFF" class="rectangle" opacity=".703"/>
        <mask id="mask-7" fill="#fff">
          <use xlink:href="#path-6"/>
        </mask>
        <g fill-rule="nonzero" class="mouse_pointer">
          <use fill="#000" filter="url(#filter-8)" xlink:href="#path-6"/>
          <use fill="#FFF" xlink:href="#path-6"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_color_blocks_2.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="27.778%" x2="72.222%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00E2FF"/>
      <stop offset="100%" stop-color="#00A09D"/>
    </linearGradient>
    <linearGradient id="linearGradient-2" x1="27.778%" x2="72.222%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <linearGradient id="linearGradient-3" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <path id="path-4" d="M27 50.429V52H13v-1.571h14zm-1.556-4.715v1.572h-9.333v-1.572h9.333zM27 41v1.571H13V41h14z"/>
    <filter id="filter-5" width="107.1%" height="118.2%" x="-3.6%" y="-4.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
    <rect id="path-6" width="27" height="3" x="7" y="33"/>
    <filter id="filter-7" width="103.7%" height="166.7%" x="-1.9%" y="-16.7%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-8" d="M69 50.429V52H55v-1.571h14zm-1.556-4.715v1.572h-9.333v-1.572h9.333zM69 41v1.571H55V41h14z"/>
    <filter id="filter-9" width="107.1%" height="118.2%" x="-3.6%" y="-4.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
    <rect id="path-10" width="27" height="3" x="49" y="33"/>
    <filter id="filter-11" width="103.7%" height="166.7%" x="-1.9%" y="-16.7%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_color_blocks_2">
      <rect width="82" height="60" class="bg"/>
      <g class="group">
        <path fill="url(#linearGradient-1)" d="M82 0v60H42V0h40zM63 6c-5.523 0-10 4.494-10 10.038 0 5.544 4.477 10.038 10 10.038s10-4.494 10-10.038C73 10.494 68.523 6 63 6z" class="combined_shape" opacity=".4"/>
        <path fill="url(#linearGradient-2)" d="M40 0v60H0V0h40zM20 6c-5.523 0-10 4.494-10 10.038 0 5.544 4.477 10.038 10 10.038s10-4.494 10-10.038C30 10.494 25.523 6 20 6z" class="combined_shape" opacity=".4"/>
        <path fill="url(#linearGradient-3)" d="M20 7a9 9 0 1 1 0 18 9 9 0 0 1 0-18zm43 0a9 9 0 1 1 0 18 9 9 0 0 1 0-18z" class="combined_shape"/>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
          <use fill="#FFF" fill-opacity=".8" xlink:href="#path-4"/>
        </g>
        <g class="rectangle_copy">
          <use fill="#000" filter="url(#filter-7)" xlink:href="#path-6"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-6"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-9)" xlink:href="#path-8"/>
          <use fill="#FFF" fill-opacity=".8" xlink:href="#path-8"/>
        </g>
        <g class="rectangle_copy">
          <use fill="#000" filter="url(#filter-11)" xlink:href="#path-10"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-10"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_company_team.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <path id="path-1" d="M43 16v2H16v-2h27zm5-16v2H16V0h32z"/>
    <filter id="filter-2" width="103.1%" height="111.1%" x="-1.6%" y="-2.8%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <path id="path-3" d="M28 24v1H16v-1h12zm13-3v1H16v-1h25zM39 8v1H16V8h23zm7-3v1H16V5h30z"/>
    <filter id="filter-4" width="103.3%" height="110%" x="-1.7%" y="-2.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <linearGradient id="linearGradient-5" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_company_team">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(17 17)">
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-1"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-3"/>
        </g>
        <path fill="url(#linearGradient-5)" d="M5 16a5 5 0 1 1 0 10 5 5 0 0 1 0-10zM5 0a5 5 0 1 1 0 10A5 5 0 0 1 5 0z" class="combined_shape"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_comparisons.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="0%" x2="100%" y1="45.918%" y2="54.082%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <linearGradient id="linearGradient-2" x1="0%" x2="100%" y1="42.969%" y2="57.031%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <path id="path-3" d="M12 9v2H3V9h9zm37 0v2h-9V9h9z"/>
    <filter id="filter-4" width="102.2%" height="200%" x="-1.1%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <path id="path-5" d="M8 22v1H3v-1h5zm4-2v1H3v-1h9zm-3-3v1H3v-1h6zm2-3v1H3v-1h8z"/>
    <filter id="filter-6" width="111.1%" height="122.2%" x="-5.6%" y="-5.6%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <path id="path-7" d="M27 20v1h-7v-1h7zm5-3v1H20v-1h12zm-4-3v1h-8v-1h8zm3-3v1H20v-1h11z"/>
    <filter id="filter-8" width="108.3%" height="120%" x="-4.2%" y="-5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <path id="path-9" d="M49 22v1h-9v-1h9zm-5-2v1h-4v-1h4zm4-3v1h-8v-1h8zm-2-3v1h-6v-1h6z"/>
    <filter id="filter-10" width="111.1%" height="122.2%" x="-5.6%" y="-5.6%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <rect id="path-11" width="12" height="2" x="20" y="6"/>
    <filter id="filter-12" width="108.3%" height="200%" x="-4.2%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_comparisons">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(15 12)">
        <path fill="#D8D8D8" d="M14 3a1 1 0 0 1 1 1v29a1 1 0 0 1-1 1H1a1 1 0 0 1-1-1V4a1 1 0 0 1 1-1h13zm0 4H1v26h13V7zm37-4a1 1 0 0 1 1 1v29a1 1 0 0 1-1 1H38a1 1 0 0 1-1-1V4a1 1 0 0 1 1-1h13zm0 4H38v26h13V7zM34 0a1 1 0 0 1 1 1v34a1 1 0 0 1-1 1H18a1 1 0 0 1-1-1V1a1 1 0 0 1 1-1h16zm0 4H18v31h16V4z" class="combined_shape"/>
        <rect width="7" height="2" x="4" y="28" fill="url(#linearGradient-1)" class="rectangle"/>
        <rect width="7" height="2" x="41" y="28" fill="url(#linearGradient-1)" class="rectangle"/>
        <rect width="8" height="3" x="22" y="29" fill="url(#linearGradient-2)" class="rectangle"/>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-3"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-5"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-8)" xlink:href="#path-7"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-7"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-10)" xlink:href="#path-9"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-9"/>
        </g>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-12)" xlink:href="#path-11"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-11"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_cover.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="0%" x2="100%" y1="23.23%" y2="76.77%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <rect id="path-2" width="21" height="2" x="0" y="0"/>
    <filter id="filter-3" width="104.8%" height="200%" x="-2.4%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-4" d="M18 8v1H3V8h15zm1-3v1H2V5h17z"/>
    <filter id="filter-5" width="105.9%" height="150%" x="-2.9%" y="-12.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_cover">
      <rect width="82" height="60" class="bg"/>
      <g fill="url(#linearGradient-1)" class="group" opacity=".4">
        <g class="image_1">
          <rect width="82" height="60" class="rectangle"/>
        </g>
      </g>
      <g class="center_group" transform="translate(31 26)">
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-3)" xlink:href="#path-2"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-2"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-4"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_event.svg

```svg
<?xml version="1.0" encoding="utf-8"?>
<!-- Generator: Adobe Illustrator 26.0.3, SVG Export Plug-In . SVG Version: 6.00 Build 0)  -->
<svg version="1.1" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
	 viewBox="0 0 82 60" style="enable-background:new 0 0 82 60;" xml:space="preserve">
<style type="text/css">
	.st0{fill-rule:evenodd;clip-rule:evenodd;fill:url(#path-8_00000095300066650753250720000002666093597020083331_);}
	.st1{fill-rule:evenodd;clip-rule:evenodd;fill:url(#path-8_00000093149943742714831120000011055561218156940933_);}
	.st2{fill-rule:evenodd;clip-rule:evenodd;fill:url(#SVGID_1_);}
	.st3{fill-rule:evenodd;clip-rule:evenodd;}
	.st4{fill-rule:evenodd;clip-rule:evenodd;fill:#FFFFFF;fill-opacity:0.348;}
	.st5{fill-rule:evenodd;clip-rule:evenodd;fill:#FFFFFF;fill-opacity:0.78;}
	.st6{fill-rule:evenodd;clip-rule:evenodd;fill:url(#SVGID_00000153707667966277653250000009137429941846899595_);}
</style>
<g>
	<g>
		
			<linearGradient id="path-8_00000176010670027675938670000004938458339886343844_" gradientUnits="userSpaceOnUse" x1="-1176.5044" y1="56.8991" x2="-1176.5044" y2="57.9573" gradientTransform="matrix(12.2244 0 0 12.2244 14408.0605 -682.6234)">
			<stop  offset="0" style="stop-color:#00A09D"/>
			<stop  offset="1" style="stop-color:#00E2FF"/>
		</linearGradient>
		
			<path id="path-8_00000101807681392685050550000002881477577771196326_" style="fill-rule:evenodd;clip-rule:evenodd;fill:url(#path-8_00000176010670027675938670000004938458339886343844_);" d="
			M26.9,16.1l2.4,2.4L25,22.8l-2.4-2.4L26.9,16.1L26.9,16.1z M25.3,23.5l4.7-4.7c0.2-0.2,0.2-0.2,0.2-0.3c0-0.2,0-0.3-0.2-0.3
			l-2.7-2.7c-0.2-0.2-0.2-0.2-0.3-0.2c-0.2,0-0.3,0-0.3,0.2l-4.7,4.7c-0.2,0.2-0.2,0.2-0.2,0.3c0,0.2,0,0.3,0.2,0.3l2.7,2.7
			c0.2,0.2,0.2,0.2,0.3,0.2C25,23.6,25.1,23.6,25.3,23.5L25.3,23.5z M32,18.7l-6.9,6.9c-0.2,0.2-0.5,0.3-0.6,0.3
			c-0.3,0-0.5-0.2-0.6-0.3l-1-1c0.3-0.3,0.5-0.6,0.5-1c0-0.5-0.2-0.8-0.5-1c-0.3-0.3-0.6-0.5-1-0.5c-0.5,0-0.8,0.2-1,0.5l-1-1
			c-0.2-0.2-0.3-0.5-0.3-0.6c0-0.3,0.2-0.5,0.3-0.6l6.9-6.9c0.2-0.2,0.5-0.3,0.6-0.3c0.3,0,0.5,0.2,0.6,0.3l1,1
			c-0.3,0.3-0.5,0.6-0.5,1c0,0.5,0.2,0.8,0.5,1c0.3,0.3,0.6,0.5,1,0.5c0.5,0,0.8-0.2,1-0.5l1,1c0.2,0.2,0.3,0.5,0.3,0.6
			C32.4,18.2,32.2,18.5,32,18.7L32,18.7z"/>
	</g>
	<g>
		
			<linearGradient id="path-8_00000132811078807879644690000010232793767224656295_" gradientUnits="userSpaceOnUse" x1="-1174.0503" y1="56.8991" x2="-1174.0503" y2="57.9573" gradientTransform="matrix(12.2244 0 0 12.2244 14408.0605 -682.6234)">
			<stop  offset="0" style="stop-color:#00A09D"/>
			<stop  offset="1" style="stop-color:#00E2FF"/>
		</linearGradient>
		
			<path id="path-8_00000013173299018101056130000011854440310922134964_" style="fill-rule:evenodd;clip-rule:evenodd;fill:url(#path-8_00000132811078807879644690000010232793767224656295_);" d="
			M56.9,16.1l2.4,2.4L55,22.8l-2.4-2.4L56.9,16.1L56.9,16.1z M55.3,23.5l4.7-4.7c0.2-0.2,0.2-0.2,0.2-0.3c0-0.2,0-0.3-0.2-0.3
			l-2.7-2.7c-0.2-0.2-0.2-0.2-0.3-0.2c-0.2,0-0.3,0-0.3,0.2l-4.7,4.7c-0.2,0.2-0.2,0.2-0.2,0.3c0,0.2,0,0.3,0.2,0.3l2.7,2.7
			c0.2,0.2,0.2,0.2,0.3,0.2C55.1,23.6,55.3,23.6,55.3,23.5L55.3,23.5z M62,18.7l-6.9,6.9c-0.2,0.2-0.5,0.3-0.6,0.3
			c-0.3,0-0.5-0.2-0.6-0.3l-1-1c0.3-0.3,0.5-0.6,0.5-1c0-0.5-0.2-0.8-0.5-1c-0.3-0.3-0.6-0.5-1-0.5c-0.5,0-0.8,0.2-1,0.5l-1-1
			c-0.2-0.2-0.3-0.5-0.3-0.6c0-0.3,0.2-0.5,0.3-0.6l6.9-6.9c0.2-0.2,0.5-0.3,0.6-0.3c0.3,0,0.5,0.2,0.6,0.3l1,1
			c-0.3,0.3-0.5,0.6-0.5,1c0,0.5,0.2,0.8,0.5,1c0.3,0.3,0.6,0.5,1,0.5c0.5,0,0.8-0.2,1-0.5l1,1c0.2,0.2,0.3,0.5,0.3,0.6
			C62.4,18.2,62.4,18.5,62,18.7L62,18.7z"/>
	</g>
	<g>
		
			<linearGradient id="SVGID_1_" gradientUnits="userSpaceOnUse" x1="-1123.9047" y1="237.1782" x2="-1122.5414" y2="237.3699" gradientTransform="matrix(8 0 0 3 9011 -665)">
			<stop  offset="0" style="stop-color:#00A09D"/>
			<stop  offset="1" style="stop-color:#00E2FF"/>
		</linearGradient>
		<rect x="19.7" y="44.8" class="st2" width="11" height="4"/>
		<g>
			<g>
				<path id="path-7" class="st3" d="M29.3,40.8v1H16v-1H29.3z M34.3,36.8v1H16v-1H34.3z"/>
			</g>
			<g>
				<path id="path-7_00000049188432945054382670000012999828447748560522_" class="st4" d="M29.3,40.8v1H16v-1H29.3z M34.3,36.8v1
					H16v-1H34.3z"/>
			</g>
		</g>
		<g>
			<g>
				<rect id="path-11_00000042727771948489652620000007994651315568149948_" x="16" y="30.8" class="st3" width="20" height="3"/>
			</g>
			<g>
				<rect id="path-11_00000069378958481318337710000011278380222473405829_" x="16" y="30.8" class="st5" width="20" height="3"/>
			</g>
		</g>
	</g>
	<g>
		
			<linearGradient id="SVGID_00000046328548324713994340000006507111683487087778_" gradientUnits="userSpaceOnUse" x1="-1120.0671" y1="237.1759" x2="-1118.7037" y2="237.3676" gradientTransform="matrix(8 0 0 3 9011 -665)">
			<stop  offset="0" style="stop-color:#00A09D"/>
			<stop  offset="1" style="stop-color:#00E2FF"/>
		</linearGradient>
		
			<rect x="50.5" y="44.8" style="fill-rule:evenodd;clip-rule:evenodd;fill:url(#SVGID_00000046328548324713994340000006507111683487087778_);" width="11" height="4"/>
		<g>
			<g>
				<path id="path-7_00000073694205868785683900000016232603875333593771_" class="st3" d="M59.3,40.8v1H46v-1H59.3z M64.3,36.8v1
					H46v-1H64.3z"/>
			</g>
			<g>
				<path id="path-7_00000132075829523470887660000004823225826125989510_" class="st4" d="M59.3,40.8v1H46v-1H59.3z M64.3,36.8v1
					H46v-1H64.3z"/>
			</g>
		</g>
		<g>
			<g>
				<rect id="path-11_00000073705521831743585720000001575955891273760656_" x="46" y="30.8" class="st3" width="20" height="3"/>
			</g>
			<g>
				<rect id="path-11_00000027596513270444160320000018432938704865459642_" x="46" y="30.8" class="st5" width="20" height="3"/>
			</g>
		</g>
	</g>
</g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_features.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <path id="path-2" d="M15 16v2H0v-2h15zm19 0v2H19v-2h15zm19 0v2H38v-2h15z"/>
    <filter id="filter-3" width="101.9%" height="200%" x="-.9%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <path id="path-4" d="M14 28v1H0v-1h14zm19 0v1H19v-1h14zm19 0v1H38v-1h14zm-41-3v1H0v-1h11zm19 0v1H19v-1h11zm19 0v1H38v-1h11zm-35-3v1H0v-1h14zm19 0v1H19v-1h14zm19 0v1H38v-1h14z"/>
    <filter id="filter-5" width="101.9%" height="128.6%" x="-1%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_features">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(15 13)">
        <path fill="url(#linearGradient-1)" d="M7 0a6 6 0 1 1 0 12A6 6 0 0 1 7 0zm19 0a6 6 0 1 1 0 12 6 6 0 0 1 0-12zm19 0a6 6 0 1 1 0 12 6 6 0 0 1 0-12z" class="combined_shape"/>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-3)" xlink:href="#path-2"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-2"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-4"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_features_grid.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="21.12" height="2" x="0" y="0"/>
    <filter id="filter-2" width="104.7%" height="200%" x="-2.4%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <path id="path-3" d="M19 20v1H8v-1h11zm2-2v1H8v-1h13zm-5-9v1H8V9h8zm5-2v1H8V7h13z"/>
    <filter id="filter-4" width="107.7%" height="114.3%" x="-3.8%" y="-3.6%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <linearGradient id="linearGradient-5" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <rect id="path-6" width="24.038" height="2" x="0" y="0"/>
    <filter id="filter-7" width="104.2%" height="200%" x="-2.1%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <path id="path-8" d="M19 20v1H8v-1h11zm3-2v1H8v-1h14zm-3-9v1H8V9h11zm-1-2v1H8V7h10z"/>
    <filter id="filter-9" width="107.1%" height="114.3%" x="-3.6%" y="-3.6%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_features_grid">
      <rect width="82" height="60" class="bg"/>
      <g class="group_3" transform="translate(15 18)">
        <g class="group_2">
          <g class="group">
            <g class="rectangle">
              <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
              <use fill="#FFF" fill-opacity=".78" xlink:href="#path-1"/>
            </g>
            <g class="combined_shape">
              <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
              <use fill="#FFF" fill-opacity=".348" xlink:href="#path-3"/>
            </g>
          </g>
          <path fill="url(#linearGradient-5)" d="M3 18a3 3 0 1 1 0 6 3 3 0 0 1 0-6zM3 7a3 3 0 1 1 0 6 3 3 0 0 1 0-6z" class="combined_shape"/>
        </g>
        <g class="group_2" transform="translate(28)">
          <g class="group">
            <g class="rectangle">
              <use fill="#000" filter="url(#filter-7)" xlink:href="#path-6"/>
              <use fill="#FFF" fill-opacity=".78" xlink:href="#path-6"/>
            </g>
            <g class="combined_shape">
              <use fill="#000" filter="url(#filter-9)" xlink:href="#path-8"/>
              <use fill="#FFF" fill-opacity=".348" xlink:href="#path-8"/>
            </g>
          </g>
          <path fill="url(#linearGradient-5)" d="M3 18a3 3 0 1 1 0 6 3 3 0 0 1 0-6zM3 7a3 3 0 1 1 0 6 3 3 0 0 1 0-6z" class="combined_shape"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_hr.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="44" height="1" x="0" y="5.5"/>
    <filter id="filter-2" width="102.3%" height="300%" x="-1.1%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_hr">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(19 24)">
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-1"/>
        </g>
        <path fill="#FFF" stroke="#FFF" d="M25.925 10.5L22 13.64l-3.925-3.14h7.85zm0-8h-7.85L22-.64l3.925 3.14z" class="combined_shape"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_image_text.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="23.077" height="18.116" x="0" y="0"/>
    <linearGradient id="linearGradient-3" x1="72.875%" x2="40.332%" y1="46.509%" y2="34.249%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-4" x1="88.517%" x2="50%" y1="39.469%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
    <rect id="path-5" width="25" height="2" x="28" y="0"/>
    <filter id="filter-6" width="104%" height="200%" x="-2%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <path id="path-7" d="M51 12v1H28v-1h23zm-4-3v1H28V9h19zm4-3v1H28V6h23z"/>
    <filter id="filter-8" width="104.3%" height="128.6%" x="-2.2%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_image_text">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(15 21)">
        <g class="image_1_border">
          <rect width="24" height="19" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.462 .442)">
            <mask id="mask-2" fill="#fff">
              <use xlink:href="#path-1"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-1"/>
            <ellipse cx="17.769" cy="4.64" fill="#F3EC60" class="oval" mask="url(#mask-2)" rx="3.462" ry="3.314"/>
            <ellipse cx="23.308" cy="19.884" fill="url(#linearGradient-3)" class="oval" mask="url(#mask-2)" rx="10.846" ry="6.628"/>
            <ellipse cx=".231" cy="20.105" fill="url(#linearGradient-4)" class="oval" mask="url(#mask-2)" rx="17.308" ry="10.384"/>
          </g>
          <path fill="#FFF" d="M24 0v19H0V0h24zm-1 1H1v17h22V1z" class="rectangle_2"/>
        </g>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-5"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-8)" xlink:href="#path-7"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-7"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_masonry_block.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="0%" x2="100%" y1="33.944%" y2="66.056%">
      <stop offset="0%" stop-color="#00E2FF"/>
      <stop offset="100%" stop-color="#00A09D"/>
    </linearGradient>
    <linearGradient id="linearGradient-2" x1="0%" x2="100%" y1="31.569%" y2="68.431%">
      <stop offset="0%" stop-color="#00E2FF"/>
      <stop offset="100%" stop-color="#00A09D"/>
    </linearGradient>
    <path id="path-3" d="M20 12v1H4v-1h16zm-6-3v1H4V9h10zm6-3v1H4V6h16z"/>
    <filter id="filter-4" width="106.2%" height="128.6%" x="-3.1%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
    <rect id="path-5" width="21" height="1" x="4" y="3"/>
    <filter id="filter-6" width="104.8%" height="300%" x="-2.4%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-7" d="M52 29v1H34v-1h18zm-6-3v1H34v-1h12zm6-3v1H34v-1h18z"/>
    <filter id="filter-8" width="105.6%" height="128.6%" x="-2.8%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
    <rect id="path-9" width="19" height="1" x="34" y="19"/>
    <filter id="filter-10" width="105.3%" height="300%" x="-2.6%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-11" d="M52 12v1H34v-1h18zm-6-3v1H34V9h12zm6-3v1H34V6h18z"/>
    <filter id="filter-12" width="105.6%" height="128.6%" x="-2.8%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <rect id="path-13" width="21" height="1" x="34" y="3"/>
    <filter id="filter-14" width="104.8%" height="300%" x="-2.4%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <path id="path-15" d="M20 29v1H4v-1h16zm-6-3v1H4v-1h10zm6-3v1H4v-1h16z"/>
    <filter id="filter-16" width="106.2%" height="128.6%" x="-3.1%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <rect id="path-17" width="15" height="1" x="4" y="19"/>
    <filter id="filter-18" width="106.7%" height="300%" x="-3.3%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <rect id="path-19" width="23.442" height="33" x="0" y="0"/>
    <linearGradient id="linearGradient-21" x1="72.875%" x2="40.332%" y1="46.301%" y2="33.313%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-22" x1="88.517%" x2="50%" y1="38.842%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_masonry_block">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(0 15)">
        <g class="group_2" transform="translate(24)">
          <rect width="58" height="33" fill="#D8D8D8" class="rectangle" opacity=".058"/>
          <rect width="30" height="17" fill="url(#linearGradient-1)" class="rectangle" opacity=".4"/>
          <rect width="28" height="17" x="30" y="16" fill="url(#linearGradient-2)" class="rectangle" opacity=".4"/>
          <g class="combined_shape">
            <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
            <use fill="#FFF" fill-opacity=".8" xlink:href="#path-3"/>
          </g>
          <g class="rectangle_copy">
            <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-5"/>
          </g>
          <g class="combined_shape">
            <use fill="#000" filter="url(#filter-8)" xlink:href="#path-7"/>
            <use fill="#FFF" fill-opacity=".8" xlink:href="#path-7"/>
          </g>
          <g class="rectangle_copy">
            <use fill="#000" filter="url(#filter-10)" xlink:href="#path-9"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-9"/>
          </g>
          <g class="combined_shape">
            <use fill="#000" filter="url(#filter-12)" xlink:href="#path-11"/>
            <use fill="#FFF" fill-opacity=".348" xlink:href="#path-11"/>
          </g>
          <g class="rectangle_copy">
            <use fill="#000" filter="url(#filter-14)" xlink:href="#path-13"/>
            <use fill="#FFF" fill-opacity=".78" xlink:href="#path-13"/>
          </g>
          <g class="combined_shape">
            <use fill="#000" filter="url(#filter-16)" xlink:href="#path-15"/>
            <use fill="#FFF" fill-opacity=".348" xlink:href="#path-15"/>
          </g>
          <g class="rectangle_copy">
            <use fill="#000" filter="url(#filter-18)" xlink:href="#path-17"/>
            <use fill="#FFF" fill-opacity=".78" xlink:href="#path-17"/>
          </g>
        </g>
        <g class="image_1_border">
          <rect width="24" height="33" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask">
            <mask id="mask-20" fill="#fff">
              <use xlink:href="#path-19"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-19"/>
            <ellipse cx="16.465" cy="6.325" fill="#F3EC60" class="oval" mask="url(#mask-20)" rx="4.186" ry="4.125"/>
            <ellipse cx="19.256" cy="34.1" fill="url(#linearGradient-21)" class="oval" mask="url(#mask-20)" rx="13.116" ry="8.25"/>
            <ellipse cx="-8.651" cy="34.375" fill="url(#linearGradient-22)" class="oval" mask="url(#mask-20)" rx="20.93" ry="12.925"/>
          </g>
          <path fill="#FFF" d="M24 0v33H0V0h24zm-1 1H1v31h22V1z" class="rectangle_2"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_media_list.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <path id="path-1" d="M32 15v1H21v-1h11zm-4-3v1h-7v-1h7zm4-3v1H21V9h11z"/>
    <filter id="filter-2" width="109.1%" height="128.6%" x="-4.5%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 0   0 0 0 0 0   0 0 0 0 0  0 0 0 0.0995137675 0"/>
    </filter>
    <rect id="path-3" width="23" height="2" x="21" y="5"/>
    <filter id="filter-4" width="104.3%" height="200%" x="-2.2%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 0.259587944   0 0 0 0 0.259629577   0 0 0 0 0.259574831  0 0 0 0.525895979 0"/>
    </filter>
    <path id="path-5" d="M42 33v1H21v-1h21zm-7-3v1H21v-1h14zm7-3v1H21v-1h21z"/>
    <filter id="filter-6" width="104.8%" height="128.6%" x="-2.4%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 0   0 0 0 0 0   0 0 0 0 0  0 0 0 0.0995137675 0"/>
    </filter>
    <rect id="path-7" width="13" height="2" x="21" y="23"/>
    <filter id="filter-8" width="107.7%" height="200%" x="-3.8%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 0.259587944   0 0 0 0 0.259629577   0 0 0 0 0.259574831  0 0 0 0.525895979 0"/>
    </filter>
    <rect id="path-9" width="17.308" height="14.302" x="0" y="0"/>
    <linearGradient id="linearGradient-11" x1="72.875%" x2="40.332%" y1="46.131%" y2="32.548%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-12" x1="88.517%" x2="50%" y1="38.331%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
    <rect id="path-13" width="17.308" height="14.302" x="0" y="0"/>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_media_list">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(15 11)">
        <rect width="53" height="39" class="rectangle"/>
        <rect width="36" height="15" x="17" y="21" fill="#FFF" class="rectangle"/>
        <rect width="36" height="15" x="17" y="3" fill="#FFF" class="rectangle"/>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
          <use fill="#000" fill-opacity=".348" xlink:href="#path-1"/>
        </g>
        <g class="rectangle_copy">
          <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
          <use fill="#000" fill-opacity=".697" xlink:href="#path-3"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
          <use fill="#000" fill-opacity=".348" xlink:href="#path-5"/>
        </g>
        <g class="rectangle_copy">
          <use fill="#000" filter="url(#filter-8)" xlink:href="#path-7"/>
          <use fill="#000" fill-opacity=".697" xlink:href="#path-7"/>
        </g>
        <g class="image_1_border" transform="translate(0 3)">
          <rect width="18" height="15" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.346 .349)">
            <mask id="mask-10" fill="#fff">
              <use xlink:href="#path-9"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-9"/>
            <ellipse cx="13.327" cy="3.663" fill="#F3EC60" class="oval" mask="url(#mask-10)" rx="2.596" ry="2.616"/>
            <ellipse cx="17.481" cy="15.698" fill="url(#linearGradient-11)" class="oval" mask="url(#mask-10)" rx="8.135" ry="5.233"/>
            <ellipse cx=".173" cy="15.872" fill="url(#linearGradient-12)" class="oval" mask="url(#mask-10)" rx="12.981" ry="8.198"/>
          </g>
          <path fill="#FFF" d="M18 0v15H0V0h18zm-1 1H1v13h16V1z" class="rectangle_2"/>
        </g>
        <g class="image_1_border" transform="translate(0 21)">
          <rect width="18" height="15" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.346 .349)">
            <mask id="mask-14" fill="#fff">
              <use xlink:href="#path-13"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-13"/>
            <ellipse cx="13.327" cy="3.663" fill="#F3EC60" class="oval" mask="url(#mask-14)" rx="2.596" ry="2.616"/>
            <ellipse cx="17.481" cy="15.698" fill="url(#linearGradient-11)" class="oval" mask="url(#mask-14)" rx="8.135" ry="5.233"/>
            <ellipse cx=".173" cy="15.872" fill="url(#linearGradient-12)" class="oval" mask="url(#mask-14)" rx="12.981" ry="8.198"/>
          </g>
          <path fill="#FFF" d="M18 0v15H0V0h18zm-1 1H1v13h16V1z" class="rectangle_2"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_numbers.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <path id="path-1" d="M14 26v1H5v-1h9zm-1-3v1H7v-1h6zm1-3v1H5v-1h9z"/>
    <filter id="filter-2" width="111.1%" height="128.6%" x="-5.6%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <rect id="path-3" width="17" height="2" x="1" y="14"/>
    <filter id="filter-4" width="105.9%" height="200%" x="-2.9%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <path id="path-5" d="M40 26v1h-9v-1h9zm-1-3v1h-6v-1h6zm1-3v1h-9v-1h9z"/>
    <filter id="filter-6" width="111.1%" height="128.6%" x="-5.6%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <rect id="path-7" width="17" height="2" x="27" y="14"/>
    <filter id="filter-8" width="105.9%" height="200%" x="-2.9%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_numbers">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(19 16)">
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-1"/>
        </g>
        <g class="rectangle_copy">
          <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-3"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-5"/>
        </g>
        <g class="rectangle_copy">
          <use fill="#000" filter="url(#filter-8)" xlink:href="#path-7"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-7"/>
        </g>
        <path fill="#E4E4E4" fill-rule="nonzero" d="M7.41 10.522v-2.59h1.47V6.489H7.41V.46H5.504L.595 6.674v1.258h4.526v2.59h2.29zM5.12 6.49H1.642l3.48-4.395V6.49zm7.89 4.033c1.207 0 2.2-.317 2.976-.953.777-.636 1.166-1.455 1.166-2.458 0-.57-.116-1.047-.349-1.432a2.858 2.858 0 0 0-.861-.92 3.482 3.482 0 0 0-1.152-.502 5.439 5.439 0 0 0-1.2-.133c-.524 0-.98.07-1.367.212-.387.141-.704.29-.95.444l.28-2.256h5.284V.638h-5.776l-.663 5.03.533.179c.228-.278.486-.503.773-.674.287-.17.642-.256 1.066-.256.547 0 .992.206 1.336.619.345.412.517.94.517 1.582 0 .442-.053.838-.158 1.186-.104.349-.26.65-.465.906a1.803 1.803 0 0 1-1.408.684c-.127 0-.264-.01-.41-.031a1.814 1.814 0 0 1-.383-.092c.05-.142.114-.347.192-.616.077-.269.116-.517.116-.745a.964.964 0 0 0-.345-.776c-.23-.193-.526-.29-.886-.29-.346 0-.623.111-.83.335a1.16 1.16 0 0 0-.311.82c0 .547.312 1.02.936 1.422.625.4 1.404.601 2.338.601z" class="45"/>
        <path fill="#E4E4E4" fill-rule="nonzero" d="M29.128 10.563c1.941-.255 3.473-.91 4.594-1.965 1.12-1.055 1.681-2.412 1.681-4.07 0-1.254-.353-2.252-1.06-2.995C33.638.79 32.686.42 31.487.42c-1.107 0-2.03.316-2.768.947-.739.63-1.108 1.427-1.108 2.389 0 .451.072.87.216 1.254.143.385.343.715.598.988.25.269.547.48.889.636.341.155.708.232 1.1.232.47 0 .892-.056 1.268-.167.376-.112.765-.343 1.166-.694-.064.406-.16.8-.287 1.183A4.04 4.04 0 0 1 32 8.274c-.237.333-.54.625-.91.875-.368.251-.833.456-1.394.616l-.724.123.157.676zM31.54 5.99c-.42 0-.758-.216-1.015-.65-.258-.432-.386-1.013-.386-1.742 0-.834.126-1.472.379-1.914.253-.442.582-.663.988-.663.42 0 .76.29 1.022.871s.393 1.419.393 2.512c0 .087-.002.176-.007.267a4.767 4.767 0 0 0-.007.232 2.9 2.9 0 0 1-.007.222 1.756 1.756 0 0 0-.006.12c-.192.287-.4.483-.623.588a1.7 1.7 0 0 1-.731.157zM42.574 8v-.52a7.37 7.37 0 0 1-.615-.082c-.278-.045-.476-.095-.595-.15a.69.69 0 0 1-.304-.277.831.831 0 0 1-.106-.427V2.538c0-.314.007-.67.02-1.066.014-.397.028-.743.042-1.04h-1.203a3.585 3.585 0 0 1-.298.363 2.502 2.502 0 0 1-.482.396 2.842 2.842 0 0 1-.721.315 3.61 3.61 0 0 1-1.029.13h-.376v.65h1.73v4.333c0 .192-.039.342-.116.451a.623.623 0 0 1-.322.233 3.68 3.68 0 0 1-.612.11c-.298.036-.516.058-.652.067V8h5.64z" class="91"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_picture.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="35.604" height="22.588" x="0" y="0"/>
    <linearGradient id="linearGradient-3" x1="72.875%" x2="40.332%" y1="47.295%" y2="37.799%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-4" x1="88.517%" x2="50%" y1="45.065%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
    <path id="path-5" d="M27 8v1H10V8h17zm-3-3v1H13V5h11z"/>
    <filter id="filter-6" width="105.9%" height="150%" x="-2.9%" y="-12.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <rect id="path-7" width="29" height="2" x="4" y="0"/>
    <filter id="filter-8" width="103.4%" height="200%" x="-1.7%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_picture">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(23 13)">
        <g class="image_1_border" transform="translate(0 11)">
          <rect width="37" height="24" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.698 .706)">
            <mask id="mask-2" fill="#fff">
              <use xlink:href="#path-1"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-1"/>
            <ellipse cx="30.368" cy="5.775" fill="#F3EC60" class="oval" mask="url(#mask-2)" rx="3.84" ry="3.775"/>
            <ellipse cx="35.255" cy="25.059" fill="url(#linearGradient-3)" class="oval" mask="url(#mask-2)" rx="16.406" ry="8.824"/>
            <ellipse cx="-6.061" cy="24" fill="url(#linearGradient-4)" class="oval" mask="url(#mask-2)" rx="30.939" ry="12.706"/>
          </g>
          <path fill="#FFF" d="M37 0v24H0V0h37zm-.698.706H.698v22.588h35.604V.706z" class="rectangle_2"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-5"/>
        </g>
        <g class="rectangle_copy">
          <use fill="#000" filter="url(#filter-8)" xlink:href="#path-7"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-7"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_product_list.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <polygon id="path-2" points="0 10.447 6.5 13.16 6.5 5.5 0 3"/>
    <filter id="filter-3" width="115.4%" height="119.7%" x="-7.7%" y="-4.9%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <polygon id="path-4" points="7.5 13.16 14 10.447 14 3 7.5 5.5"/>
    <filter id="filter-5" width="115.4%" height="119.7%" x="-7.7%" y="-4.9%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <polygon id="path-6" points="0 10.447 6.5 13.16 6.5 5.5 0 3"/>
    <filter id="filter-7" width="115.4%" height="119.7%" x="-7.7%" y="-4.9%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <polygon id="path-8" points="7.5 13.16 14 10.447 14 3 7.5 5.5"/>
    <filter id="filter-9" width="115.4%" height="119.7%" x="-7.7%" y="-4.9%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <polygon id="path-10" points="0 10.447 6.5 13.16 6.5 5.5 0 3"/>
    <filter id="filter-11" width="115.4%" height="119.7%" x="-7.7%" y="-4.9%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <polygon id="path-12" points="7.5 13.16 14 10.447 14 3 7.5 5.5"/>
    <filter id="filter-13" width="115.4%" height="119.7%" x="-7.7%" y="-4.9%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_product_list">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(15 20)">
        <path fill="url(#linearGradient-1)" d="M13 18v2H1v-2h12zm20 0v2H20v-2h13zm20 0v2H38v-2h15z" class="combined_shape"/>
        <g class="box_solid">
          <rect width="14" height="13" class="rectangle"/>
          <polygon fill="#FFF" fill-opacity=".78" points="7 .5 0 2.405 7 5 14 2.405" class="path"/>
          <g class="path">
            <use fill="#000" filter="url(#filter-3)" xlink:href="#path-2"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-2"/>
          </g>
          <g class="path">
            <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
            <use fill="#FFF" fill-opacity=".78" xlink:href="#path-4"/>
          </g>
        </g>
        <g class="box_solid" transform="translate(38)">
          <rect width="14" height="13" class="rectangle"/>
          <polygon fill="#FFF" fill-opacity=".78" points="7 .5 0 2.405 7 5 14 2.405" class="path"/>
          <g class="path">
            <use fill="#000" filter="url(#filter-7)" xlink:href="#path-6"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-6"/>
          </g>
          <g class="path">
            <use fill="#000" filter="url(#filter-9)" xlink:href="#path-8"/>
            <use fill="#FFF" fill-opacity=".78" xlink:href="#path-8"/>
          </g>
        </g>
        <g class="box_solid" transform="translate(19)">
          <rect width="14" height="13" class="rectangle"/>
          <polygon fill="#FFF" fill-opacity=".78" points="7 .5 0 2.405 7 5 14 2.405" class="path"/>
          <g class="path">
            <use fill="#000" filter="url(#filter-11)" xlink:href="#path-10"/>
            <use fill="#FFF" fill-opacity=".95" xlink:href="#path-10"/>
          </g>
          <g class="path">
            <use fill="#000" filter="url(#filter-13)" xlink:href="#path-12"/>
            <use fill="#FFF" fill-opacity=".78" xlink:href="#path-12"/>
          </g>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_rating.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="22" height="2" x="0" y="0"/>
    <filter id="filter-2" width="104.5%" height="200%" x="-2.3%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-3" d="M43 8.1c0 .074-.047.155-.14.242l-1.964 1.785.465 2.52a.71.71 0 0 1 .006.101c0 .07-.02.13-.057.179-.038.049-.093.073-.165.073a.461.461 0 0 1-.217-.06L38.5 11.75l-2.428 1.19a.485.485 0 0 1-.217.06c-.076 0-.132-.024-.17-.073a.283.283 0 0 1-.057-.179c0-.02.004-.054.01-.1l.466-2.521-1.969-1.785c-.09-.09-.135-.171-.135-.242 0-.124.101-.201.303-.232l2.715-.368 1.217-2.293c.068-.138.157-.207.265-.207.108 0 .197.069.265.207L39.982 7.5l2.715.368c.202.03.303.108.303.232z"/>
    <filter id="filter-4" width="111.1%" height="125%" x="-5.6%" y="-6.2%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <path id="path-5" d="M31 8.1c0 .074-.047.155-.14.242l-1.964 1.785.465 2.52a.71.71 0 0 1 .006.101c0 .07-.02.13-.057.179-.038.049-.093.073-.165.073a.461.461 0 0 1-.217-.06L26.5 11.75l-2.428 1.19a.485.485 0 0 1-.217.06c-.076 0-.132-.024-.17-.073a.283.283 0 0 1-.057-.179c0-.02.004-.054.01-.1l.466-2.521-1.969-1.785c-.09-.09-.135-.171-.135-.242 0-.124.101-.201.303-.232l2.715-.368 1.217-2.293c.068-.138.157-.207.265-.207.108 0 .197.069.265.207L27.982 7.5l2.715.368c.202.03.303.108.303.232z"/>
    <filter id="filter-6" width="111.1%" height="125%" x="-5.6%" y="-6.2%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <linearGradient id="linearGradient-7" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_rating">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(20 24)">
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-1"/>
        </g>
        <g class="mask">
          <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-3"/>
        </g>
        <g class="mask">
          <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-5"/>
        </g>
        <path fill="url(#linearGradient-7)" d="M19 8.1c0 .074-.042.155-.125.242l-1.745 1.785.413 2.52a.799.799 0 0 1 .005.101c0 .07-.017.13-.05.179a.167.167 0 0 1-.147.073.376.376 0 0 1-.192-.06L15 11.75l-2.159 1.19a.395.395 0 0 1-.192.06c-.067 0-.118-.024-.151-.073a.307.307 0 0 1-.05-.179c0-.02.002-.054.009-.1l.413-2.521-1.75-1.785c-.08-.09-.12-.171-.12-.242 0-.124.09-.201.27-.232l2.413-.368 1.081-2.293c.061-.138.14-.207.236-.207.096 0 .175.069.236.207L16.317 7.5l2.414.368c.18.03.269.108.269.232z" class="star_copy"/>
        <path fill="url(#linearGradient-7)" d="M8 8.1c0 .074-.042.155-.125.242L6.13 10.127l.413 2.52a.799.799 0 0 1 .005.101c0 .07-.017.13-.05.179A.167.167 0 0 1 6.35 13a.376.376 0 0 1-.192-.06L4 11.75l-2.159 1.19a.395.395 0 0 1-.192.06c-.067 0-.118-.024-.151-.073a.307.307 0 0 1-.05-.179c0-.02.002-.054.009-.1l.413-2.521L.12 8.342C.04 8.252 0 8.171 0 8.1c0-.124.09-.201.27-.232L2.682 7.5l1.081-2.293C3.825 5.069 3.904 5 4 5s.175.069.236.207L5.317 7.5l2.414.368c.18.03.269.108.269.232z" class="star"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_references.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="10.577" height="8.581" x="0" y="0"/>
    <linearGradient id="linearGradient-3" x1="72.875%" x2="40.332%" y1="46.271%" y2="33.176%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-4" x1="88.517%" x2="50%" y1="38.751%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
    <rect id="path-5" width="9.615" height="8.581" x="0" y="0"/>
    <linearGradient id="linearGradient-7" x1="72.875%" x2="40.332%" y1="45.488%" y2="29.644%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-8" x1="88.517%" x2="50%" y1="36.389%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
    <rect id="path-9" width="11.538" height="8.581" x="0" y="0"/>
    <linearGradient id="linearGradient-11" x1="72.875%" x2="40.332%" y1="46.866%" y2="35.864%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-12" x1="88.517%" x2="50%" y1="40.548%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
    <rect id="path-13" width="10.577" height="8.581" x="0" y="0"/>
    <rect id="path-15" width="22" height="2" x="17" y="0"/>
    <filter id="filter-16" width="104.5%" height="200%" x="-2.3%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <rect id="path-17" width="14" height="1" x="21" y="5"/>
    <filter id="filter-18" width="107.1%" height="300%" x="-3.6%" y="-50%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_references">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(14 20)">
        <g class="image_1_border" transform="translate(30 11)">
          <rect width="11" height="9" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.212 .21)">
            <mask id="mask-2" fill="#fff">
              <use xlink:href="#path-1"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-1"/>
            <ellipse cx="8.144" cy="2.198" fill="#F3EC60" class="oval" mask="url(#mask-2)" rx="1.587" ry="1.57"/>
            <ellipse cx="10.683" cy="9.419" fill="url(#linearGradient-3)" class="oval" mask="url(#mask-2)" rx="4.971" ry="3.14"/>
            <ellipse cx=".106" cy="9.523" fill="url(#linearGradient-4)" class="oval" mask="url(#mask-2)" rx="7.933" ry="4.919"/>
          </g>
          <path fill="#FFF" d="M11 0v9H0V0h11zm-1 1H1v7h9V1z" class="rectangle_2"/>
        </g>
        <g class="image_1_border" transform="translate(45 11)">
          <rect width="10" height="9" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.192 .21)">
            <mask id="mask-6" fill="#fff">
              <use xlink:href="#path-5"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-5"/>
            <ellipse cx="7.404" cy="2.198" fill="#F3EC60" class="oval" mask="url(#mask-6)" rx="1.442" ry="1.57"/>
            <ellipse cx="9.712" cy="9.419" fill="url(#linearGradient-7)" class="oval" mask="url(#mask-6)" rx="4.519" ry="3.14"/>
            <ellipse cx=".096" cy="9.523" fill="url(#linearGradient-8)" class="oval" mask="url(#mask-6)" rx="7.212" ry="4.919"/>
          </g>
          <path fill="#FFF" d="M10 0v9H0V0h10zM9 1H1v7h8V1z" class="rectangle_2"/>
        </g>
        <g class="image_1_border" transform="translate(0 11)">
          <rect width="12" height="9" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.23 .21)">
            <mask id="mask-10" fill="#fff">
              <use xlink:href="#path-9"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-9"/>
            <ellipse cx="8.885" cy="2.198" fill="#F3EC60" class="oval" mask="url(#mask-10)" rx="1.731" ry="1.57"/>
            <ellipse cx="11.654" cy="9.419" fill="url(#linearGradient-11)" class="oval" mask="url(#mask-10)" rx="5.423" ry="3.14"/>
            <ellipse cx=".115" cy="9.523" fill="url(#linearGradient-12)" class="oval" mask="url(#mask-10)" rx="8.654" ry="4.919"/>
          </g>
          <path fill="#FFF" d="M12 0v9H0V0h12zm-1 1H1v7h10V1z" class="rectangle_2"/>
        </g>
        <g class="image_1_border" transform="translate(15 11)">
          <rect width="11" height="9" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.212 .21)">
            <mask id="mask-14" fill="#fff">
              <use xlink:href="#path-13"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-13"/>
            <ellipse cx="8.144" cy="2.198" fill="#F3EC60" class="oval" mask="url(#mask-14)" rx="1.587" ry="1.57"/>
            <ellipse cx="10.683" cy="9.419" fill="url(#linearGradient-3)" class="oval" mask="url(#mask-14)" rx="4.971" ry="3.14"/>
            <ellipse cx=".106" cy="9.523" fill="url(#linearGradient-4)" class="oval" mask="url(#mask-14)" rx="7.933" ry="4.919"/>
          </g>
          <path fill="#FFF" d="M11 0v9H0V0h11zm-1 1H1v7h9V1z" class="rectangle_2"/>
        </g>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-16)" xlink:href="#path-15"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-15"/>
        </g>
        <g class="rectangle_copy">
          <use fill="#000" filter="url(#filter-18)" xlink:href="#path-17"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-17"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_showcase.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="1" height="19" x="26" y="0"/>
    <filter id="filter-2" width="200%" height="110.5%" x="-50%" y="-2.6%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <path id="path-3" d="M13 16v1H2v-1h11zm0-11v1H5V5h8z"/>
    <filter id="filter-4" width="109.1%" height="116.7%" x="-4.5%" y="-4.2%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <path id="path-5" d="M13 12v2H0v-2h13zm0-11v2H0V1h13z"/>
    <filter id="filter-6" width="107.7%" height="115.4%" x="-3.8%" y="-3.8%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <rect id="path-7" width="13" height="5" x="20" y="22"/>
    <filter id="filter-8" width="107.7%" height="140%" x="-3.8%" y="-10%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <rect id="path-9" width="11" height="3" x="21" y="23"/>
    <filter id="filter-10" width="109.1%" height="166.7%" x="-4.5%" y="-16.7%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 0   0 0 0 0 0   0 0 0 0 0  0 0 0 0.0995137675 0"/>
    </filter>
    <path id="path-11" d="M51 16v1H40v-1h11zm0-11v1H40V5h11z"/>
    <filter id="filter-12" width="109.1%" height="116.7%" x="-4.5%" y="-4.2%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <path id="path-13" d="M54 12v2H40v-2h14zM50 1v2H40V1h10z"/>
    <filter id="filter-14" width="107.1%" height="115.4%" x="-3.6%" y="-3.8%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <linearGradient id="linearGradient-15" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_showcase">
      <rect width="82" height="60" class="bg"/>
      <g class="group_2" transform="translate(14 17)">
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-1"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-4)" xlink:href="#path-3"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-3"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-5"/>
        </g>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-8)" xlink:href="#path-7"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-7"/>
        </g>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-10)" xlink:href="#path-9"/>
          <use fill="#000" fill-opacity=".348" xlink:href="#path-9"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-12)" xlink:href="#path-11"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-11"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-14)" xlink:href="#path-13"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-13"/>
        </g>
        <path fill="url(#linearGradient-15)" d="M18 12a3 3 0 1 1 0 6 3 3 0 0 1 0-6zm0-11a3 3 0 1 1 0 6 3 3 0 0 1 0-6zm17 11a3 3 0 1 1 0 6 3 3 0 0 1 0-6zm0-11a3 3 0 1 1 0 6 3 3 0 0 1 0-6z" class="combined_shape"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_text_block.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <path id="path-1" d="M53 37v1H15v-1h38zm14-3v1H15v-1h52zm0-3v1H15v-1h52zm-24-5v1H15v-1h28zm24-3v1H15v-1h52z"/>
    <filter id="filter-2" width="101.9%" height="113.3%" x="-1%" y="-3.3%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_text_block">
      <rect width="82" height="60" class="bg"/>
      <g class="combined_shape">
        <use fill="#000" filter="url(#filter-2)" xlink:href="#path-1"/>
        <use fill="#FFF" fill-opacity=".348" xlink:href="#path-1"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_text_highlight.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <linearGradient id="linearGradient-1" x1="50%" x2="50%" y1="0%" y2="100%">
      <stop offset="0%" stop-color="#00A09D"/>
      <stop offset="100%" stop-color="#00E2FF"/>
    </linearGradient>
    <rect id="path-2" width="22" height="2" x="11" y="3"/>
    <filter id="filter-3" width="104.5%" height="200%" x="-2.3%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.4 0"/>
    </filter>
    <path id="path-4" d="M28 11v1H16v-1h12zm2-3v1H14V8h16z"/>
    <filter id="filter-5" width="106.2%" height="150%" x="-3.1%" y="-12.5%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.2 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_text_highlight">
      <rect width="82" height="60" class="bg"/>
      <g class="group_2" transform="translate(19 22)">
        <rect width="44" height="17" fill="url(#linearGradient-1)" fill-opacity=".4" class="rectangle_2" rx="1"/>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-3)" xlink:href="#path-2"/>
          <use fill="#FFF" fill-opacity=".95" xlink:href="#path-2"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-5)" xlink:href="#path-4"/>
          <use fill="#FFF" fill-opacity=".8" xlink:href="#path-4"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_text_image.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="23.077" height="18.116" x="0" y="0"/>
    <linearGradient id="linearGradient-3" x1="72.875%" x2="40.332%" y1="46.509%" y2="34.249%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-4" x1="88.517%" x2="50%" y1="39.469%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
    <rect id="path-5" width="25" height="2" x="0" y="0"/>
    <filter id="filter-6" width="104%" height="200%" x="-2%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <path id="path-7" d="M23 12v1H0v-1h23zm-4-3v1H0V9h19zm4-3v1H0V6h23z"/>
    <filter id="filter-8" width="104.3%" height="128.6%" x="-2.2%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_text_image">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(15 21)">
        <g class="image_1_border" transform="translate(29)">
          <rect width="24" height="19" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.462 .442)">
            <mask id="mask-2" fill="#fff">
              <use xlink:href="#path-1"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-1"/>
            <ellipse cx="17.769" cy="4.64" fill="#F3EC60" class="oval" mask="url(#mask-2)" rx="3.462" ry="3.314"/>
            <ellipse cx="23.308" cy="19.884" fill="url(#linearGradient-3)" class="oval" mask="url(#mask-2)" rx="10.846" ry="6.628"/>
            <ellipse cx=".231" cy="20.105" fill="url(#linearGradient-4)" class="oval" mask="url(#mask-2)" rx="17.308" ry="10.384"/>
          </g>
          <path fill="#FFF" d="M24 0v19H0V0h24zm-1 1H1v17h22V1z" class="rectangle_2"/>
        </g>
        <g class="rectangle">
          <use fill="#000" filter="url(#filter-6)" xlink:href="#path-5"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-5"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-8)" xlink:href="#path-7"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-7"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_three_columns.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="82" height="60" viewBox="0 0 82 60">
  <defs>
    <rect id="path-1" width="14.423" height="11.442" x="0" y="0"/>
    <linearGradient id="linearGradient-3" x1="72.875%" x2="40.332%" y1="46.435%" y2="33.916%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-4" x1="88.517%" x2="50%" y1="39.246%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
    <rect id="path-5" width="15.385" height="11.442" x="0" y="0"/>
    <linearGradient id="linearGradient-7" x1="72.875%" x2="40.332%" y1="46.866%" y2="35.864%">
      <stop offset="0%" stop-color="#008374"/>
      <stop offset="100%" stop-color="#006A59"/>
    </linearGradient>
    <linearGradient id="linearGradient-8" x1="88.517%" x2="50%" y1="40.548%" y2="50%">
      <stop offset="0%" stop-color="#00AA89"/>
      <stop offset="100%" stop-color="#009989"/>
    </linearGradient>
    <rect id="path-9" width="14.423" height="11.442" x="0" y="0"/>
    <path id="path-11" d="M15 16v2H0v-2h15zm19 0v2H19v-2h15zm19 0v2H38v-2h15z"/>
    <filter id="filter-12" width="101.9%" height="200%" x="-.9%" y="-25%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.292012675 0"/>
    </filter>
    <path id="path-13" d="M33 28v1H19v-1h14zm-3-3v1H19v-1h11zm3-3v1H19v-1h14z"/>
    <filter id="filter-14" width="107.1%" height="128.6%" x="-3.6%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <path id="path-15" d="M52 28v1H38v-1h14zm-3-3v1H38v-1h11zm3-3v1H38v-1h14z"/>
    <filter id="filter-16" width="107.1%" height="128.6%" x="-3.6%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
    <path id="path-17" d="M14 28v1H0v-1h14zm-3-3v1H0v-1h11zm3-3v1H0v-1h14z"/>
    <filter id="filter-18" width="107.1%" height="128.6%" x="-3.6%" y="-7.1%" filterUnits="objectBoundingBox">
      <feOffset dy="1" in="SourceAlpha" result="shadowOffsetOuter1"/>
      <feComposite in="shadowOffsetOuter1" in2="SourceAlpha" operator="out" result="shadowOffsetOuter1"/>
      <feColorMatrix in="shadowOffsetOuter1" values="0 0 0 0 1   0 0 0 0 1   0 0 0 0 1  0 0 0 0.0995137675 0"/>
    </filter>
  </defs>
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_three_columns">
      <rect width="82" height="60" class="bg"/>
      <g class="group" transform="translate(15 16)">
        <g class="image_1_border">
          <rect width="15" height="12" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.288 .28)">
            <mask id="mask-2" fill="#fff">
              <use xlink:href="#path-1"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-1"/>
            <ellipse cx="11.106" cy="2.93" fill="#F3EC60" class="oval" mask="url(#mask-2)" rx="2.163" ry="2.093"/>
            <ellipse cx="14.567" cy="12.558" fill="url(#linearGradient-3)" class="oval" mask="url(#mask-2)" rx="6.779" ry="4.186"/>
            <ellipse cx=".144" cy="12.698" fill="url(#linearGradient-4)" class="oval" mask="url(#mask-2)" rx="10.817" ry="6.558"/>
          </g>
          <path fill="#FFF" d="M15 0v12H0V0h15zm-1 1H1v10h13V1z" class="rectangle_2"/>
        </g>
        <g class="image_1_border" transform="translate(18)">
          <rect width="16" height="12" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.308 .28)">
            <mask id="mask-6" fill="#fff">
              <use xlink:href="#path-5"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-5"/>
            <ellipse cx="11.846" cy="2.93" fill="#F3EC60" class="oval" mask="url(#mask-6)" rx="2.308" ry="2.093"/>
            <ellipse cx="15.538" cy="12.558" fill="url(#linearGradient-7)" class="oval" mask="url(#mask-6)" rx="7.231" ry="4.186"/>
            <ellipse cx=".154" cy="12.698" fill="url(#linearGradient-8)" class="oval" mask="url(#mask-6)" rx="11.538" ry="6.558"/>
          </g>
          <path fill="#FFF" d="M16 0v12H0V0h16zm-1 1H1v10h14V1z" class="rectangle_2"/>
        </g>
        <g class="image_1_border" transform="translate(38)">
          <rect width="15" height="12" fill="#FFF" class="rectangle"/>
          <g class="oval___oval_mask" transform="translate(.288 .28)">
            <mask id="mask-10" fill="#fff">
              <use xlink:href="#path-9"/>
            </mask>
            <use fill="#79D1F2" class="mask" xlink:href="#path-9"/>
            <ellipse cx="11.106" cy="2.93" fill="#F3EC60" class="oval" mask="url(#mask-10)" rx="2.163" ry="2.093"/>
            <ellipse cx="14.567" cy="12.558" fill="url(#linearGradient-3)" class="oval" mask="url(#mask-10)" rx="6.779" ry="4.186"/>
            <ellipse cx=".144" cy="12.698" fill="url(#linearGradient-4)" class="oval" mask="url(#mask-10)" rx="10.817" ry="6.558"/>
          </g>
          <path fill="#FFF" d="M15 0v12H0V0h15zm-1 1H1v10h13V1z" class="rectangle_2"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-12)" xlink:href="#path-11"/>
          <use fill="#FFF" fill-opacity=".78" xlink:href="#path-11"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-14)" xlink:href="#path-13"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-13"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-16)" xlink:href="#path-15"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-15"/>
        </g>
        <g class="combined_shape">
          <use fill="#000" filter="url(#filter-18)" xlink:href="#path-17"/>
          <use fill="#FFF" fill-opacity=".348" xlink:href="#path-17"/>
        </g>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_thumbs\s_title.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="82" height="60" viewBox="0 0 82 60">
  <g fill="none" fill-rule="evenodd" class="snippets_thumbs">
    <g class="s_title">
      <rect width="82" height="60" class="bg"/>
      <path fill="#E4E4E4" fill-rule="nonzero" d="M19.076 33.364v-.381c-.415-.015-.71-.066-.887-.154-.302-.156-.454-.456-.454-.9V28.47h3.575v3.457c0 .43-.14.723-.418.88-.18.102-.486.16-.915.175v.38h5.105v-.38c-.4-.015-.686-.068-.857-.161-.298-.156-.447-.454-.447-.894V24.67c0-.43.134-.72.403-.872.16-.092.461-.156.9-.19v-.38h-5.104v.38c.454.03.764.09.93.183.268.151.403.444.403.879v3.105h-3.575V24.67c0-.435.142-.73.425-.886.171-.093.476-.152.916-.176v-.38H14v.38c.444.044.742.11.894.198.249.151.373.44.373.864v7.258c0 .435-.14.73-.417.886-.166.093-.45.15-.85.169v.38h5.076zm9.697.212c.513 0 .967-.095 1.362-.285.61-.298 1.148-.828 1.612-1.59l-.345-.212c-.283.312-.52.534-.71.666-.313.22-.645.33-.996.33-.738 0-1.253-.413-1.546-1.238-.16-.44-.258-.96-.293-1.56h3.824c0-.137-.017-.334-.052-.593-.068-.552-.19-1.001-.366-1.348a2.605 2.605 0 0 0-1.003-1.077 2.675 2.675 0 0 0-1.392-.388c-.864 0-1.603.319-2.215.956-.613.637-.92 1.532-.92 2.684 0 1.275.323 2.203.967 2.784.645.58 1.336.871 2.073.871zm1.062-4.446h-2.014c.02-.776.101-1.367.245-1.772s.412-.608.802-.608c.381 0 .635.176.762.527.127.352.195.97.205 1.853zm7.859 4.446c.249 0 .486-.044.71-.132.357-.136.681-.376.974-.717l-.227-.315a.981.981 0 0 1-.216.18.363.363 0 0 1-.157.032c-.069 0-.13-.038-.187-.114a.448.448 0 0 1-.084-.274v-3.633c0-.889-.283-1.499-.85-1.831-.571-.327-1.282-.49-2.131-.49-.791 0-1.458.168-2 .505-.542.337-.813.793-.813 1.37 0 .322.102.568.304.739.203.17.448.256.736.256.25 0 .468-.077.656-.23.188-.154.282-.373.282-.656a.824.824 0 0 0-.062-.318 1.08 1.08 0 0 0-.165-.275l-.088-.103a.588.588 0 0 1-.088-.124.326.326 0 0 1-.03-.147c0-.151.087-.277.26-.377.174-.1.395-.15.664-.15.478 0 .811.106 1 .319.187.212.281.54.281.985v1.091c-1.386.4-2.404.798-3.054 1.194-.65.395-.974.93-.974 1.604 0 .552.178.958.535 1.22.356.26.74.391 1.15.391.307 0 .62-.054.937-.161.523-.17.991-.464 1.406-.879.054.288.14.506.257.652.205.259.53.388.974.388zm-2.351-1.07c-.186 0-.353-.083-.502-.252-.149-.168-.223-.416-.223-.743 0-.552.256-1.006.769-1.363.302-.21.654-.363 1.054-.461v2.19c-.16.186-.3.322-.417.41-.21.147-.437.22-.681.22zm9.294 1.07c.615-.17 1.045-.277 1.29-.318.243-.042.785-.104 1.625-.187v-.344c-.342-.03-.571-.104-.688-.224-.117-.12-.176-.336-.176-.648v-8.628h-3.23v.366c.459.02.762.075.908.165.147.09.22.324.22.7v2.746c-.288-.303-.527-.513-.718-.63a1.925 1.925 0 0 0-1.055-.293c-.79 0-1.468.343-2.032 1.03-.564.685-.846 1.622-.846 2.808 0 1.065.271 1.907.813 2.527.542.62 1.18.93 1.912.93.43 0 .83-.112 1.2-.337.235-.141.494-.359.777-.652v.99zm-1.304-.85c-.512 0-.859-.353-1.04-1.061-.097-.391-.146-.972-.146-1.744 0-.722.051-1.281.154-1.677.19-.737.552-1.106 1.084-1.106.361 0 .652.12.871.36.22.238.33.419.33.541v3.648c0 .117-.128.32-.385.607-.256.289-.545.433-.868.433zm6.629-7.397c.317 0 .59-.113.817-.34.227-.228.34-.502.34-.824a1.12 1.12 0 0 0-.34-.824 1.116 1.116 0 0 0-.817-.341c-.322 0-.597.114-.824.34-.227.228-.34.502-.34.825 0 .322.113.596.34.824.227.227.502.34.824.34zm1.75 8.035v-.36c-.268-.053-.451-.126-.549-.219-.098-.093-.146-.303-.146-.63v-5.698h-2.879v.366c.303.054.506.136.608.246.103.11.154.316.154.619v4.409c0 .341-.073.578-.22.71-.097.088-.278.154-.542.198v.359h3.574zm4.205 0v-.36c-.27-.053-.452-.126-.55-.219-.097-.093-.146-.303-.146-.63v-4.013c.117-.19.284-.38.501-.568.218-.188.456-.282.715-.282.346 0 .578.154.695.461.069.171.103.428.103.77v3.632c0 .327-.049.537-.147.63-.097.093-.28.166-.549.22v.359h3.523v-.36c-.269-.033-.46-.1-.575-.197-.115-.098-.172-.315-.172-.652V28.53c0-.835-.177-1.419-.531-1.75-.354-.333-.853-.499-1.498-.499-.45 0-.858.119-1.227.355a3.073 3.073 0 0 0-.912.898v-1.077h-2.841v.366c.322.04.536.117.64.235.106.117.158.327.158.63v4.409c0 .341-.065.57-.194.684-.13.115-.33.19-.604.224v.359h3.61zm7.91 3.113c.859 0 1.572-.096 2.138-.286 1.075-.366 1.612-1.038 1.612-2.014 0-.757-.36-1.282-1.077-1.575-.376-.151-.84-.232-1.392-.242l-.974-.014c-.132 0-.317-.004-.556-.011a5.941 5.941 0 0 1-.47-.026.579.579 0 0 1-.343-.154.475.475 0 0 1-.125-.351c0-.19.083-.354.25-.491a1.01 1.01 0 0 1 .46-.234c.093 0 .163.002.21.007.046.005.145.007.296.007.698 0 1.282-.093 1.75-.278.894-.347 1.34-.989 1.34-1.926 0-.298-.052-.574-.157-.828a1.975 1.975 0 0 0-.45-.666h1.216v-.799h-1.978a3.807 3.807 0 0 0-.681-.212 4.256 4.256 0 0 0-.967-.103c-.932 0-1.672.222-2.219.667-.547.444-.82 1.008-.82 1.692 0 .542.156 1.003.468 1.384.313.38.723.674 1.23.879v.102c-.35.118-.71.325-1.08.623-.368.298-.552.657-.552 1.077 0 .351.117.632.351.842.132.117.344.232.637.344v.103c-.44.068-.748.216-.926.443-.178.227-.268.448-.268.663 0 .551.42.942 1.26 1.171.508.137 1.114.206 1.817.206zm.102-5.838c-.415 0-.696-.22-.842-.659-.098-.278-.147-.706-.147-1.282 0-.63.066-1.11.198-1.439.132-.33.396-.494.791-.494.362 0 .617.153.766.461.149.308.223.798.223 1.472 0 .635-.07 1.117-.212 1.447-.142.33-.4.494-.777.494zm.066 5.347c-.625 0-1.102-.088-1.432-.264-.33-.176-.494-.417-.494-.725 0-.18.06-.351.183-.513.068-.088.18-.195.337-.322h2.46c.509 0 .859.065 1.052.194.193.13.29.324.29.582 0 .445-.34.75-1.019.916-.361.088-.82.132-1.377.132z" class="heading"/>
    </g>
  </g>
</svg>

```

## File: static\src\js\mailing_m2o_filter.js

```javascript
/** @odoo-module **/

import { registry } from '@web/core/registry';
import { Dropdown } from "@web/core/dropdown/dropdown";
import { useService } from "@web/core/utils/hooks";
import { Many2OneField } from '@web/views/fields/many2one/many2one_field';
import Domain from 'web.Domain';

const { useState, useEffect } = owl;

export class MailingFilterDropdown extends Dropdown {
    setup() {
        super.setup();
        useEffect((inputFilterEl) => {
            if (inputFilterEl) {
                inputFilterEl.focus();
            }
        }, () => [document.querySelector('.o_mass_mailing_filter_name')]);
    }
}

/**
 * Widget to create / remove favorite filters on mass mailing and/or marketing automation, extended
 * from Many2OneField. This widget is designed specifically for 'mailing_filter_id'
 * field on 'mailing.mailing' and 'marketing.campaign' form view.
 *
 * In edit mode, it will allow to save the latest configured domain
 * in form of the favorite filter, or to remove the store filters.
 *
 */
export class FieldMany2OneMailingFilter extends Many2OneField {
    setup() {
        super.setup();
        this.notification = useService("notification");
        this.filter = useState({
            canSaveFilter: false,
        });
        useEffect(() => this._updateFilterIcons());
    }

    /**
     * Updates the 'Add to favorite' / 'Remove' icons' visibility based on the
     * current state, and shows the custom message when no filter is available
     * for the selected model.
     *
     * The filter can be saved if one of those conditions is matched:
     * - No favorite filter is currently set
     * - User emptied the input
     * - User changed the domain when favorite filter is set
     * - The input is currently being edited, known by the "this.state.isFloating" variable
     *
     * @private
     */
    _updateFilterIcons() {
        const el = document.querySelector('.o_mass_mailing_filter_container');
        if (!el || this.props.readonly) {
            return;
        }
        const filterCount = this.props.record.data.mailing_filter_count;
        const dropdown = document.querySelector('.o_field_mailing_filter > .o_field_many2one_selection > .o_input_dropdown')
        if (dropdown) {
            dropdown.classList.toggle('d-none', !filterCount);
        }
        // By default, domains in recordData are in string format, but adding / removing a leaf from domain widget converts
        // value into object, so we use 'Domain' class to convert them in same (string) format, allowing proper comparison.
        let recordDomain;
        let filterDomain;
        try {
            recordDomain = new Domain(this.props.record.data[this.props.domain_field] || []).toString();
            filterDomain = new Domain(this.props.record.data.mailing_filter_domain || []).toString();
        } catch {
            // Don't raise a traceback if a domain set manually doesn't match the format expected.
            // This can happen when we unfocus the domain editor
            this.filter.canSaveFilter = false;
            this.filter.canRemoveFilter = false;
            return;
        }

        const modelFieldElement = this.props.model_field && document.querySelector(
            `input#${this.props.model_field},div [name="${this.props.model_field}"]`);

        let value = "";
        if (modelFieldElement && modelFieldElement.tagName === "span") {
            value = modelFieldElement.textContent;
        } else if (modelFieldElement && modelFieldElement.tagName === "input") {
            value = modelFieldElement.value;
        }

        el.classList.toggle('d-none', recordDomain === '[]');
        this.filter.canSaveFilter = !this.props.record.data.mailing_filter_id
            || value.length
            || this.state.isFloating
            || filterDomain !== recordDomain;
        this.filter.canRemoveFilter = !this.filter.canSaveFilter
    }

    // HANDLERS

    /**
     * Focus the 'Save' button on 'Tab' key, or directly save the filter on 'Enter'
     *
     * @param {event} ev
     */
    onFilterNameInputKeydown(ev) {
        const btnSave = document.querySelector('.o_mass_mailing_btn_save_filter');
        if (ev.key === 'Tab') {
            ev.preventDefault();
            btnSave.focus();
        } else if (ev.key === 'Enter') {
            btnSave.click();
        }
    }

    /**
     * Deletes the saved filter, but we do not reset the applied domain
     * in this case.
     *
     * @param {event} ev
     */
    async onRemoveFilter(ev) {
        const filterId = this.props.record.data.mailing_filter_id[0];
        const mailingDomain = this.props.record.data[this.props.domain_field];
        // Prevent multiple clicks to avoid trying to deleting same record multiple times.
        ev.target.disabled = true;

        await this.orm.unlink('mailing.filter', [filterId]);
        this.update([{ id: false, name: false }]);
        this.props.record.update({[this.props.domain_field]: mailingDomain});
    }

    /**
     * Creates a new favorite filter, with the name provided from drop-down and
     * with the 'up to date' domain. If the input is blank, displays the warning
     * and keeps the popup open by preventing event propagation.
     *
     * Note: We do not disable the save button here to avoid multiple clicks as for the delete,
     * because as soon as the 'Save' button is clicked, the popup will be closed immediately.
     *
     * @param {event} ev
     */
    async onSaveFilter(ev) {
        const filterInput = document.querySelector('input.o_mass_mailing_filter_name');
        const filterName = filterInput.value.trim();
        if (filterName.length === 0) {
            this.notification.add(
                this.env._t("Please provide a name for the filter"),
                {type: 'danger'}
            );
            // Keep the drop-down open, and re-focus the input
            ev.stopPropagation();
            filterInput.focus();
        } else {
            const newFilterId = await this.env.model.orm.create("mailing.filter", [{
                name: filterName,
                mailing_domain: this.props.record.data[this.props.domain_field],
                mailing_model_id: this.props.record.data[this.props.model_field][0],
            }]);
            this.update([{ id: newFilterId, name: filterName }]);
        }
    }
}
FieldMany2OneMailingFilter.template = 'mass_mailing.MailingFilter';
FieldMany2OneMailingFilter.components = { 
    ...Many2OneField.components,
    MailingFilterDropdown,
};
FieldMany2OneMailingFilter.props = {
    ...Many2OneField.props,
    domain_field: { type: String, optional: true },
    model_field: { type: String, optional: true },
};
FieldMany2OneMailingFilter.defaultProps = {
    ...Many2OneField.defaultProps,
    domain_field: "mailing_domain",
    model_field: "mailing_model_id",
};
FieldMany2OneMailingFilter.extractProps = ({ field, attrs }) => {
    return {
        ...Many2OneField.extractProps({ field, attrs }),
        domain_field: attrs.options.domain_field,
        model_field: attrs.options.model_field,
    }
};

registry.category('fields').add('mailing_filter', FieldMany2OneMailingFilter);

```

## File: static\src\js\mailing_mailing_view_form_full_width.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { formView } from "@web/views/form/form_view";
import { throttleForAnimation } from "@web/core/utils/timing";

const {
    useSubEnv,
    onMounted,
    onWillUnmount,
} = owl;

export class MassMailingFullWidthViewController extends formView.Controller {
    setup() {
        super.setup();
        useSubEnv({
            onIframeUpdated: () => this._updateIframe(),
            mailingFilterTemplates: true,
        });
        this._resizeObserver =  new ResizeObserver(throttleForAnimation(() => {
            this._resizeMailingEditorIframe();
            this._repositionMailingEditorSidebar();
        }));
        onMounted(() => {
            $('.o_content').on('scroll.repositionMailingEditorSidebar', throttleForAnimation(this._repositionMailingEditorSidebar.bind(this)));
        });
        onWillUnmount(() => {
            $('.o_content').off('.repositionMailingEditorSidebar');
            this._resizeObserver.disconnect();
        });
    }
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------
    /**
     * Resize the given iframe so its height fits its contents and initialize a
     * resize observer to resize on each size change in its contents.
     * This also ensures the contents of the sidebar remain visible no matter
     * how much we resize the iframe and scroll down.
     *
     * @private
     * @param {JQuery} ev.data.$iframe
     */
    _updateIframe() {
        const $iframe = $('iframe.wysiwyg_iframe:visible, iframe.o_readonly');
        if (!$iframe.length || !$iframe.contents().length) {
            return;
        }
        const hasIframeChanged = !this.$iframe || !this.$iframe.length || $iframe[0] !== this.$iframe[0];
        this.$iframe = $iframe;
        this._resizeMailingEditorIframe();

        const $iframeDoc = $iframe.contents();
        $iframeDoc.get(0).querySelector('html').classList.add('o_mass_mailing_iframe_full_width');
        const iframeTarget = $iframeDoc.find('#iframe_target');
        if (hasIframeChanged) {
            $iframeDoc.find('body').on('click', '.o_fullscreen_btn', this._onToggleFullscreen.bind(this));
            if (iframeTarget[0]) {
                this._resizeObserver.disconnect();
                this._resizeObserver.observe(iframeTarget[0]);
            }
        }
        if (iframeTarget[0]) {
            const isFullscreen = this._isFullScreen();
            iframeTarget.css({
                display: isFullscreen ? '' : 'flex',
                'flex-direction': isFullscreen ? '' : 'column',
            });
        }
    }
    /**
     * Reposition the sidebar so it always occupies the full available visible
     * height, no matter the scroll position. This way, the sidebar is always
     * visible and as big as possible.
     *
     * @private
     */
    _repositionMailingEditorSidebar() {
        const windowHeight = $(window).height();
        const $iframeDocument = this.$iframe.contents();
        const $sidebar = $iframeDocument.find('#oe_snippets');
        const isFullscreen =  this._isFullScreen();
        if (isFullscreen) {
            $sidebar.height(windowHeight);
            this.$iframe.height(windowHeight);
            $sidebar.css({
                top: '',
                bottom: '',
            });
        } else {
            const iframeTop = this.$iframe.offset().top;
            $sidebar.css({
                height: '',
                top: Math.max(0, $('.o_content').offset().top - iframeTop),
                bottom: this.$iframe.height() - windowHeight + iframeTop,
            });
        }
    }
    /**
     * Switch "scrolling modes" on toggle fullscreen mode: in fullscreen mode,
     * the scroll happens within the iframe whereas in regular mode we pretend
     * there is no iframe and scroll in the top document. Also reposition the
     * sidebar since toggling the fullscreen mode visibly changes the
     * positioning of elements in the document.
     *
     * @private
     */
    _onToggleFullscreen() {
        const isFullscreen = this._isFullScreen();
        const $iframeDoc = this.$iframe.contents();
        const html = $iframeDoc.find('html').get(0);
        html.scrollTop = 0;
        html.classList.toggle('o_fullscreen', isFullscreen);
        const wysiwyg = $iframeDoc.find('.note-editable').data('wysiwyg');
        if (wysiwyg && wysiwyg.snippetsMenu) {
            // Restore the appropriate scrollable depending on the mode.
            this._$scrollable = this._$scrollable || wysiwyg.snippetsMenu.$scrollable;
            wysiwyg.snippetsMenu.$scrollable = isFullscreen ? $iframeDoc.find('.note-editable') : this._$scrollable;
        }
        this._repositionMailingEditorSidebar();
        this._resizeMailingEditorIframe();
    }
    /**
     * Return true if the mailing editor is in full screen mode, false
     * otherwise.
     *
     * @private
     * @returns {boolean}
     */
    _isFullScreen() {
        return window.top.document.body.classList.contains('o_field_widgetTextHtml_fullscreen');
    }
    /**
     * Resize the mailing editor's iframe container so its height fits its
     * contents. This needs to be called whenever the iframe's contents might
     * have changed, eg. when adding/removing content to/from it or when a
     * template is picked.
     *
     * @private
     */
    _resizeMailingEditorIframe() {
        const minHeight = $(window).height() - Math.abs(this.$iframe.offset().top);
        const $iframeDoc = this.$iframe.contents();
        const $themeSelectorNew = $iframeDoc.find('.o_mail_theme_selector_new');
        if ($themeSelectorNew.length) {
            this.$iframe.height(Math.max($themeSelectorNew[0].scrollHeight, minHeight));
        } else {
            const ref = $iframeDoc.find('#iframe_target')[0];
            if (ref) {
                this.$iframe.css({
                    height: this._isFullScreen()
                        ? $(window).height()
                        : Math.max(ref.scrollHeight, minHeight),
                });
            }
        }
    }
}

export const massMailingFormView = {
    ...formView,
    Controller: MassMailingFullWidthViewController,
};

registry.category("views").add("mailing_mailing_view_form_full_width", massMailingFormView);

```

## File: static\src\js\mailing_portal.js

```javascript
odoo.define('mass_mailing.unsubscribe', function (require) {
    'use strict';

    var session = require('web.session');
    var ajax = require('web.ajax');
    var core = require('web.core');
    require('web.dom_ready');

    var _t = core._t;

    var email = $("input[name='email']").val();
    var mailing_id = parseInt($("input[name='mailing_id']").val());
    var res_id = parseInt($("input[name='res_id']").val());
    var token = (location.search.split('token' + '=')[1] || '').split('&')[0];

    if (!$('.o_unsubscribe_form').length) {
        return;
    }
    session.load_translations().then(function () {
        if (email != '' && email != undefined){
            ajax.jsonRpc('/mailing/blacklist/check', 'call', {'email': email, 'mailing_id': mailing_id, 'res_id': res_id, 'token': token})
                .then(function (result) {
                    if (result == 'unauthorized'){
                        $('#button_add_blacklist').hide();
                        $('#button_remove_blacklist').hide();
                    }
                    else if (result == true) {
                        $('#button_remove_blacklist').show();
                        toggle_opt_out_section(false);
                    }
                    else if (result == false) {
                        $('#button_add_blacklist').show();
                        toggle_opt_out_section(true);
                    }
                    else {
                        $('#subscription_info').text(_t('An error occurred. Please try again later or contact us.'));
                        $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-warning').addClass('alert-error');
                    }
                })
                .guardedCatch(function () {
                    $('#subscription_info').text(_t('An error occurred. Please try again later or contact us.'));
                    $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-warning').addClass('alert-error');
                });
        }
        else {
            $('#div_blacklist').hide();
        }

        var unsubscribed_list = $("input[name='unsubscribed_list']").val();
        if (unsubscribed_list){
            $('#subscription_info').html(_.str.sprintf(
                _t("You have been <strong>successfully unsubscribed from %s</strong>."),
                _.escape(unsubscribed_list)
            ));
        }
        else{
            $('#subscription_info').html(_t('You have been <strong>successfully unsubscribed</strong>.'));
        }
    });

    $('#unsubscribe_form').on('submit', function (e) {
        e.preventDefault();

        var checked_ids = [];
        $("input[type='checkbox']:checked").each(function (i){
          checked_ids[i] = parseInt($(this).val());
        });

        var unchecked_ids = [];
        $("input[type='checkbox']:not(:checked)").each(function (i){
          unchecked_ids[i] = parseInt($(this).val());
        });

        ajax.jsonRpc('/mail/mailing/unsubscribe', 'call', {'opt_in_ids': checked_ids, 'opt_out_ids': unchecked_ids, 'email': email, 'mailing_id': mailing_id, 'res_id': res_id, 'token': token})
            .then(function (result) {
                if (result == 'unauthorized'){
                    $('#subscription_info').text(_t('You are not authorized to do this!'));
                    $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-error').addClass('alert-warning');
                }
                else if (result == true) {
                    $('#subscription_info').text(_t('Your changes have been saved.'));
                    $('#info_state').removeClass('alert-info').addClass('alert-success');
                }
                else {
                    $('#subscription_info').text(_t('An error occurred. Your changes have not been saved, try again later.'));
                    $('#info_state').removeClass('alert-info').addClass('alert-warning');
                }
            })
            .guardedCatch(function () {
                $('#subscription_info').text(_t('An error occurred. Your changes have not been saved, try again later.'));
                $('#info_state').removeClass('alert-info').addClass('alert-warning');
            });
    });

    //  ==================
    //      Blacklist
    //  ==================
    $('#button_add_blacklist').click(function (e) {
        e.preventDefault();

        ajax.jsonRpc('/mailing/blacklist/add', 'call', {'email': email, 'mailing_id': mailing_id, 'res_id': res_id, 'token': token})
            .then(function (result) {
                if (result == 'unauthorized'){
                    $('#subscription_info').text(_t('You are not authorized to do this!'));
                    $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-error').addClass('alert-warning');
                }
                else
                {
                    if (result) {
                        $('#subscription_info').html(_t('You have been successfully <strong>added to our blacklist</strong>. '
                               + 'You will not be contacted anymore by our services.'));
                        $('#info_state').removeClass('alert-warning').removeClass('alert-info').removeClass('alert-error').addClass('alert-success');
                        toggle_opt_out_section(false);
                    }
                    else {
                        $('#subscription_info').text(_t('An error occurred. Please try again later or contact us.'));
                        $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-warning').addClass('alert-error');
                    }
                    $('#button_add_blacklist').hide();
                    $('#button_remove_blacklist').show();
                    $('#unsubscribed_info').hide();
                }
            })
            .guardedCatch(function () {
                $('#subscription_info').text(_t('An error occurred. Please try again later or contact us.'));
                $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-warning').addClass('alert-error');
            });
    });

    $('#button_remove_blacklist').click(function (e) {
        e.preventDefault();

        ajax.jsonRpc('/mailing/blacklist/remove', 'call', {'email': email, 'mailing_id': mailing_id, 'res_id': res_id, 'token': token})
            .then(function (result) {
                if (result == 'unauthorized'){
                    $('#subscription_info').text(_t('You are not authorized to do this!'));
                    $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-error').addClass('alert-warning');
                }
                else
                {
                    if (result) {
                        $('#subscription_info').html(_t("You have been successfully <strong>removed from our blacklist</strong>. "
                                + "You are now able to be contacted by our services."));
                        $('#info_state').removeClass('alert-warning').removeClass('alert-info').removeClass('alert-error').addClass('alert-success');
                        toggle_opt_out_section(true);
                    }
                    else {
                        $('#subscription_info').text(_t('An error occurred. Please try again later or contact us.'));
                        $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-warning').addClass('alert-error');
                    }
                    $('#button_add_blacklist').show();
                    $('#button_remove_blacklist').hide();
                    $('#unsubscribed_info').hide();
                }
            })
            .guardedCatch(function () {
                $('#subscription_info').text(_t('An error occurred. Please try again later or contact us.'));
                $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-warning').addClass('alert-error');
            });
    });

    // ==================
    //      Feedback
    // ==================
    $('#button_feedback').click(function (e) {
        var feedback = $("textarea[name='opt_out_feedback']").val();
        e.preventDefault();
        ajax.jsonRpc('/mailing/feedback', 'call', {'mailing_id': mailing_id, 'res_id': res_id, 'email': email, 'feedback': feedback, 'token': token})
            .then(function (result) {
                if (result == 'unauthorized'){
                    $('#subscription_info').text(_t('You are not authorized to do this!'));
                    $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-error').addClass('alert-warning');
                }
                else if (result == true){
                    $('#subscription_info').text(_t('Thank you! Your feedback has been sent successfully!'));
                    $('#info_state').removeClass('alert-warning').removeClass('alert-info').removeClass('alert-error').addClass('alert-success');
                    $("#div_feedback").hide();
                }
                else {
                    $('#subscription_info').text(_t('An error occurred. Please try again later or contact us.'));
                    $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-error').addClass('alert-warning');
                }
            })
            .guardedCatch(function () {
                $('#subscription_info').text(_t('An error occurred. Please try again later or contact us.'));
                $('#info_state').removeClass('alert-info').removeClass('alert-success').removeClass('alert-error').addClass('alert-warning');
            });
    });
});

function toggle_opt_out_section(value) {
    var result = !value;
    $("#div_opt_out").find('*').attr('disabled',result);
    $("#button_add_blacklist").attr('disabled', false);
    $("#button_remove_blacklist").attr('disabled', false);
    if (value) { $('[name="button_subscription"]').addClass('clickable');  }
    else { $('[name="button_subscription"]').removeClass('clickable'); }
}

```

## File: static\src\js\mass_mailing.js

```javascript
odoo.define('mass_mailing.mass_mailing', function (require) {
"use strict";

var KanbanColumn = require('web.KanbanColumn');

KanbanColumn.include({
    init: function () {
        this._super.apply(this, arguments);
        if (this.modelName === 'mailing.mailing') {
            this.draggable = false;
        }
    },
});

});

```

## File: static\src\js\mass_mailing_design_constants.js

```javascript
/** @odoo-module alias=mass_mailing.design_constants**/

export const CSS_PREFIX = '.o_mail_wrapper';

export const BTN_SIZE_STYLES = {
    'btn-sm': {
        'padding': '3px 7.5px',
        'font-size': '0.875rem',
        'line-height': '1.5rem',
    },
    'btn-lg': {
        'padding': '7px 14px',
        'font-size': '1.25rem',
        'line-height': '1.5rem',
    },
    'btn-md': {
        'padding': false, // Property must be removed.
        'font-size': '14px',
        'line-height': false, // Property must be removed.
    },
};
export const DEFAULT_BUTTON_SIZE = 'btn-md';
export const PRIORITY_STYLES = {
    'h1': ['font-family'],
    'h2': ['font-family'],
    'h3': ['font-family'],
    'p': ['font-family'],
    'a:not(.btn)': [],
    'a.btn.btn-primary': [],
    'a.btn.btn-secondary': [],
    'hr': ['border-top-width','border-top-style','border-top-color'],
};
export const RE_CSS_TEXT_MATCH = /([^{]+)([^}]+)/;
export const RE_SELECTOR_ENDS_WITH_GT_STAR = />\s*\*\s*$/;

export const transformFontFamilySelector = selector => {
    if (selector.trim().endsWith(':not(.fa)')) {
        return [selector];
    }
    if (!selector.endsWith('*')) {
        return [`${selector.trim()}:not(.fa)`, `${selector.trim()} :not(.fa)`];
    } else if (RE_SELECTOR_ENDS_WITH_GT_STAR.test(selector)) {
        return [`${selector.replace(RE_SELECTOR_ENDS_WITH_GT_STAR, '').trim()} :not(.fa)`];
    }
}
/**
 * Take a css text and splits each comma-separated selector into separate
 * styles, applying the css prefix to each. Return the modified css text.
 *
 * @param {string} [css]
 * @returns {string}
 */
export const splitCss = css => {
    const styleElement = document.createElement('style');
    styleElement.textContent = css;
    // Temporarily insert the style element in the dom to have a stylesheet.
    document.head.appendChild(styleElement);
    const rules = [...styleElement.sheet.cssRules];
    styleElement.remove();
    const stylesToWrite = {};
    for (const rule of rules) {
        const styles = rule.style;
        for (let selector of rule.selectorText.split(',')) {
            if (!selector.trim().startsWith(CSS_PREFIX)) {
                selector = `${CSS_PREFIX} ${selector.trim()}`;
            }
            for (const style of rule.style) {
                let selectors = [selector];
                if (style === 'font-family') {
                    // Ensure font-family gets passed to all descendants and never
                    // overwrite font awesome.
                    selectors = transformFontFamilySelector(selector);
                }
                for (const selectorToWriteTo of selectors) {
                    if (!stylesToWrite[selectorToWriteTo]) {
                        stylesToWrite[selectorToWriteTo] = [];
                    }
                    stylesToWrite[selectorToWriteTo].push([style, styles[style] + (styles.getPropertyPriority(style) === 'important' ? ' !important' : '')]);
                }
            }
        }
    }
    return Object.entries(stylesToWrite).map(([selector, styles]) => (
        `${selector.trim()} {\n${styles.map(([styleName, style]) => `    ${styleName}: ${style};`).join('\n')}\n}`
    )).join('\n');
};
export const getFontName = fontFamily => fontFamily.split(',')[0].replace(/"/g, '').replace(/([a-z])([A-Z])/g, (v, a, b) => `${a} ${b}`).trim();
export const normalizeFontFamily = fontFamily => fontFamily.replace(/"/g, '').replace(/, /g, ',');
export const initializeDesignTabCss = $editable => {
        let styleElement = $editable.get(0).ownerDocument.querySelector('#design-element');
        if (styleElement) {
            styleElement.textContent = splitCss(styleElement.textContent);
        } else {
            // If a style element can't be found, create one and initialize it.
            styleElement = document.createElement('style');
            styleElement.setAttribute('id', 'design-element');
        }
        // The style element needs to be within the layout of the email in
        // order to be saved along with it.
        $editable.find('.o_layout').prepend(styleElement);
};

export const FONT_FAMILIES = [
    'Arial, "Helvetica Neue", Helvetica, sans-serif', // name: "Arial"
    '"Courier New", Courier, "Lucida Sans Typewriter", "Lucida Typewriter", monospace', // name: "Courier New"
    'Georgia, Times, "Times New Roman", serif', // name: "Georgia"
    '"Helvetica Neue", Helvetica, Arial, sans-serif', // name: "Helvetica Neue"
    '"Lucida Grande", "Lucida Sans Unicode", "Lucida Sans", Geneva, Verdana, sans-serif', // name: "Lucida Grande"
    'Tahoma, Verdana, Segoe, sans-serif', // name: "Tahoma"
    'TimesNewRoman, "Times New Roman", Times, Baskerville, Georgia, serif', // name: "Times New Roman"
    '"Trebuchet MS", "Lucida Grande", "Lucida Sans Unicode", "Lucida Sans", Tahoma, sans-serif', // name: "Trebuchet MS"
    'Verdana, Geneva, sans-serif', // name: "Verdana"
].map(fontFamily => normalizeFontFamily(fontFamily));

export default {
    CSS_PREFIX, BTN_SIZE_STYLES, DEFAULT_BUTTON_SIZE, PRIORITY_STYLES,
    RE_CSS_TEXT_MATCH, FONT_FAMILIES, RE_SELECTOR_ENDS_WITH_GT_STAR,
    splitCss, getFontName, normalizeFontFamily, initializeDesignTabCss,
    transformFontFamilySelector,
}

```

## File: static\src\js\mass_mailing_html_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { _lt } from "@web/core/l10n/translation";
import { standardFieldProps } from "@web/views/fields/standard_field_props";
import { initializeDesignTabCss } from "mass_mailing.design_constants";
import { toInline } from "web_editor.convertInline";
import { loadBundle } from "@web/core/assets";
import { qweb } from 'web.core';
import { useService } from "@web/core/utils/hooks";
import { buildQuery } from "web.rpc";
import { HtmlField } from "@web_editor/js/backend/html_field";
import { getWysiwygClass } from 'web_editor.loader';
import { device } from 'web.config';
import { MassMailingMobilePreviewDialog } from "./mass_mailing_mobile_preview";
import { getRangePosition } from '@web_editor/js/editor/odoo-editor/src/utils/utils';

const {
    useSubEnv,
    onWillUpdateProps,
    status,
} = owl;

export class MassMailingHtmlField extends HtmlField {
    setup() {
        super.setup();

        useSubEnv({
            onWysiwygReset: this._resetIframe.bind(this),
        });
        this.action = useService('action');
        this.rpc = useService('rpc');
        this.dialog = useService('dialog');

        onWillUpdateProps(() => {
            if (this.props.record.data.mailing_model_id && this.wysiwyg) {
                this._hideIrrelevantTemplates();
            }
        });
    }

    get wysiwygOptions() {
        return {
            ...super.wysiwygOptions,
            onIframeUpdated: () => this.onIframeUpdated(),
            snippets: 'mass_mailing.email_designer_snippets',
            resizable: false,
            defaultDataForLinkTools: { isNewWindow: true },
            toolbarTemplate: 'mass_mailing.web_editor_toolbar',
            onWysiwygBlur: () => {
                this.commitChanges();
                this.wysiwyg.odooEditor.toolbarHide();
            },
            ...this.props.wysiwygOptions,
        };
    }

    /**
     * @param {HTMLElement} popover
     * @param {Object} position
     * @override
     */
    positionDynamicPlaceholder(popover, position) {
        const editable = this.wysiwyg.$iframe ? this.wysiwyg.$iframe[0] : this.wysiwyg.$editable[0];
        const relativeParentPosition = editable.getBoundingClientRect();

        let topPosition = relativeParentPosition.top;
        let leftPosition = relativeParentPosition.left;

        const rangePosition = getRangePosition(popover, this.wysiwyg.options.document);
        topPosition += rangePosition.top;
        // Offset the popover to ensure the arrow is pointing at
        // the precise range location.
        leftPosition += rangePosition.left - 14;

        // Apply the position back to the element.
        popover.style.top = topPosition + 'px';
        popover.style.left = leftPosition + 'px';
    }

    async commitChanges(...args) {
        if (this.props.readonly || !this.isRendered) {
            return super.commitChanges(...args);
        }
        if (!this._isDirty() || this._pendingCommitChanges) {
            // In case there is still a pending change while committing the
            // changes from the save button, we need to wait for the previous
            // operation to finish, otherwise the "inline field" of the mass
            // mailing might not be saved.
            return this._pendingCommitChanges;
        }
        this._pendingCommitChanges = (async () => {
            const codeViewEl = this._getCodeViewEl();
            if (codeViewEl) {
                this.wysiwyg.setValue(this._getCodeViewValue(codeViewEl));
            }

            if (this.wysiwyg.$iframeBody.find('.o_basic_theme').length) {
                this.wysiwyg.$iframeBody.find('*').css('font-family', '');
            }

            const $editable = this.wysiwyg.getEditable();
            this.wysiwyg.odooEditor.historyPauseSteps();
            await this.wysiwyg.cleanForSave();
            if (args.length) {
                await super.commitChanges({ ...args[0], urgent: true });
            } else {
                await super.commitChanges({ urgent: true });
            }

            const $editorEnable = $editable.closest('.editor_enable');
            $editorEnable.removeClass('editor_enable');
            // Prevent history reverts.
            this.wysiwyg.odooEditor.observerUnactive('toInline');
            const iframe = document.createElement('iframe');
            iframe.style.height = '0px';
            iframe.style.visibility = 'hidden';
            iframe.setAttribute('sandbox', 'allow-same-origin'); // Make sure no scripts get executed.
            const clonedHtmlNode = $editable[0].closest('html').cloneNode(true);
            // Replace the body to only contain the target as we do not care for
            // other elements (e.g. sidebar, toolbar, ...)
            const clonedBody = clonedHtmlNode.querySelector('body');
            const clonedIframeTarget = clonedHtmlNode.querySelector('#iframe_target');
            clonedBody.replaceChildren(clonedIframeTarget);
            clonedHtmlNode.querySelectorAll('script').forEach(script => script.remove()); // Remove scripts.
            iframe.srcdoc = clonedHtmlNode.outerHTML;
            const iframePromise = new Promise((resolve) => {
                iframe.addEventListener("load", resolve);
            });
            document.body.append(iframe);
            // Wait for the css and images to be loaded.
            await iframePromise;
            const editableClone = iframe.contentDocument.querySelector('.note-editable');
            // The jQuery data are lost because of the cloning operations above.
            // The hacky fix for stable is to simply add it back manually.
            // TODO in master: Update toInline to use an options parameter.
            $(editableClone).data("wysiwyg", this.wysiwyg);
            await toInline($(editableClone), undefined, $(iframe));
            iframe.remove();
            this.wysiwyg.odooEditor.observerActive('toInline');
            const inlineHtml = editableClone.innerHTML;
            $editorEnable.addClass('editor_enable');
            this.wysiwyg.odooEditor.historyUnpauseSteps();
            this.wysiwyg.odooEditor.historyRevertCurrentStep();

            const fieldName = this.props.inlineField;
            await this.props.record.update({[fieldName]: this._unWrap(inlineHtml)});
            this._pendingCommitChanges = null;
        })();
        return this._pendingCommitChanges;
    }
    async startWysiwyg(...args) {
        await super.startWysiwyg(...args);

        await loadBundle({
            jsLibs: [
                '/mass_mailing/static/src/js/mass_mailing_link_dialog_fix.js',
                '/mass_mailing/static/src/js/mass_mailing_snippets.js',
                '/mass_mailing/static/src/snippets/s_masonry_block/options.js',
                '/mass_mailing/static/src/snippets/s_media_list/options.js',
                '/mass_mailing/static/src/snippets/s_showcase/options.js',
                '/mass_mailing/static/src/snippets/s_rating/options.js',
            ],
        });

        if (status(this) === "destroyed") {
            return;
        }

        await this._resetIframe();
    }

    async _resetIframe() {
        if (this._switchingTheme) {
            return;
        }
        this.wysiwyg.$iframeBody.find('.o_mail_theme_selector_new').remove();
        await this._onSnippetsLoaded();

        // Data is removed on save but we need the mailing and its body to be
        // named so they are handled properly by the snippets menu.
        this.wysiwyg.$iframeBody.find('.o_layout').addBack().data('name', 'Mailing');
        // We don't want to drop snippets directly within the wysiwyg.
        this.wysiwyg.$iframeBody.find('.odoo-editor-editable').removeClass('o_editable');

        initializeDesignTabCss(this.wysiwyg.getEditable());
        this.wysiwyg.getEditable().find('img').attr('loading', '');

        this.wysiwyg.odooEditor.observerFlush();
        this.wysiwyg.odooEditor.historyReset();
        this.wysiwyg.$iframeBody.addClass('o_mass_mailing_iframe');

        this.onIframeUpdated();
    }

    async _onSnippetsLoaded() {
        if (this.wysiwyg.snippetsMenu && $(window.top.document).find('.o_mass_mailing_form_full_width')[0]) {
            // In full width form mode, ensure the snippets menu's scrollable is
            // in the form view, not in the iframe.
            this.wysiwyg.snippetsMenu.$scrollable = this.wysiwyg.$el.closestScrollable();
            // Ensure said scrollable keeps its scrollbar at all times to
            // prevent the scrollbar from appearing at awkward moments (ie: when
            // previewing an option)
            this.wysiwyg.snippetsMenu.$scrollable.css('overflow-y', 'scroll');
        }

        // Remove the web editor menu to avoid flicker (we add it back at the
        // end of the method)
        this.wysiwyg.$iframeBody.find('.iframe-utils-zone').addClass('d-none');

        // Filter the fetched templates based on the current model
        const args = this.props.filterTemplates
            ? [[['mailing_model_id', '=', this.props.record.data.mailing_model_id[0]]]]
            : [];

        const rpcQuery = buildQuery({
            model: 'mailing.mailing',
            method: 'action_fetch_favorites',
            args: args,
        })
        // Templates taken from old mailings
        const result = await this.rpc(rpcQuery.route, rpcQuery.params);
        const templatesParams = result.map(values => {
            return {
                id: values.id,
                modelId: values.mailing_model_id[0],
                modelName: values.mailing_model_id[1],
                name: `template_${values.id}`,
                nowrap: true,
                subject: values.subject,
                template: values.body_arch,
                userId: values.user_id[0],
                userName: values.user_id[1],
            };
        });

        const $snippetsSideBar = this.wysiwyg.snippetsMenu.$el;
        const $themes = $snippetsSideBar.find("#email_designer_themes").children();
        const $snippets = $snippetsSideBar.find(".oe_snippet");
        const selectorToKeep = '.o_we_external_history_buttons, .email_designer_top_actions';
        // Overide `d-flex` class which style is `!important`
        $snippetsSideBar.find(`.o_we_website_top_actions > *:not(${selectorToKeep})`).attr('style', 'display: none!important');

        if (!odoo.debug) {
            $snippetsSideBar.find('.o_codeview_btn').hide();
        }
        const $codeview = this.wysiwyg.$iframe.contents().find('textarea.o_codeview');
        // Unbind first the event handler as this method can be called multiple time during the component life.
        $snippetsSideBar.off('click', '.o_codeview_btn');
        $snippetsSideBar.on('click', '.o_codeview_btn', () => {
            this.wysiwyg.odooEditor.observerUnactive();
            $codeview.toggleClass('d-none');
            this.wysiwyg.getEditable().toggleClass('d-none');
            this.wysiwyg.odooEditor.observerActive();

            if ($codeview.hasClass('d-none')) {
                this.wysiwyg.setValue(this._getCodeViewValue($codeview[0]));
            } else {
                $codeview.val(this.wysiwyg.getValue());
            }
            this.wysiwyg.snippetsMenu.activateSnippet(false);
            this.onIframeUpdated();
        });
        const $previewBtn = $snippetsSideBar.find('.o_mobile_preview_btn');
        $previewBtn.off('click');
        $previewBtn.on('click', () => {
            $previewBtn.prop('disabled', true); // Prevent double execution when double-clicking on the button
            let mailingHtml = new DOMParser().parseFromString(this.wysiwyg.getValue(), 'text/html');
            [...mailingHtml.querySelectorAll('a')].forEach(el => {
                el.style.setProperty('pointer-events', 'none');
            });
            this.mobilePreview = this.dialog.add(MassMailingMobilePreviewDialog, {
                title: this.env._t("Mobile Preview"),
                preview: mailingHtml.body.innerHTML,
            }, {
                onClose: () => $previewBtn.prop('disabled', false),
            });
        });

        if (!this._themeParams) {
            // Initialize theme parameters.
            this._themeClassNames = "";
            const displayableThemes =
                device.isMobile ?
                _.filter($themes, theme => !$(theme).data("hideFromMobile")) :
                $themes;
            this._themeParams = _.map(displayableThemes, (theme) => {
                const $theme = $(theme);
                const name = $theme.data("name");
                const classname = "o_" + name + "_theme";
                this._themeClassNames += " " + classname;
                const imagesInfo = _.defaults($theme.data("imagesInfo") || {}, {
                    all: {}
                });
                for (const info of Object.values(imagesInfo)) {
                    _.defaults(info, imagesInfo.all, {
                        module: "mass_mailing",
                        format: "jpg"
                    });
                }
                return {
                    name: name,
                    title: $theme.attr("title") || "",
                    className: classname || "",
                    img: $theme.data("img") || "",
                    template: $theme.html().trim(),
                    nowrap: !!$theme.data('nowrap'),
                    get_image_info: function (filename) {
                        if (imagesInfo[filename]) {
                            return imagesInfo[filename];
                        }
                        return imagesInfo.all;
                    },
                    layoutStyles: $theme.data('layout-styles'),
                };
            });
        }
        $themes.parent().remove();

        if (!this._themeParams.length) {
            return;
        }

        const themesParams = [...this._themeParams];

        // Create theme selection screen and check if it must be forced opened.
        // Reforce it opened if the last snippet is removed.
        const $themeSelectorNew = $(qweb.render("mass_mailing.theme_selector_new", {
            themes: themesParams,
            templates: templatesParams,
            modelName: this.props.record.data.mailing_model_id[1] || '',
        }));

        // Check if editable area is empty.
        const $layout = this.wysiwyg.$iframeBody.find(".o_layout");
        let $mailWrapper = $layout.children(".o_mail_wrapper");
        let $mailWrapperContent = $mailWrapper.find('.o_mail_wrapper_td');
        if (!$mailWrapperContent.length) {
            $mailWrapperContent = $mailWrapper;
        }
        let value;
        if ($mailWrapperContent.length > 0) {
            value = $mailWrapperContent.html();
        } else if ($layout.length) {
            value = $layout.html();
        } else {
            value = this.wysiwyg.getValue();
        }
        let blankEditable = "<p><br></p>";
        const editableAreaIsEmpty = value === "" || value === blankEditable;

        if (editableAreaIsEmpty) {
            // unfold to prevent toolbar from going over the menu
            this.wysiwyg.setSnippetsMenuFolded(false);
            $themeSelectorNew.appendTo(this.wysiwyg.$iframeBody);
        }

        $themeSelectorNew.on('click', '.dropdown-item', async (e) => {
            e.preventDefault();
            e.stopImmediatePropagation();

            const themeName = $(e.currentTarget).attr('id');

            const themeParams = [...themesParams, ...templatesParams].find(theme => theme.name === themeName);

            await this._switchThemes(themeParams);
            this.wysiwyg.$iframeBody.closest('body').removeClass("o_force_mail_theme_choice");

            $themeSelectorNew.remove();

            this.wysiwyg.setSnippetsMenuFolded(device.isMobile || themeName === 'basic');

            this._switchImages(themeParams, $snippets);

            const $editable = this.wysiwyg.$editable.find('.o_editable');
            this.$editorMessageElements = $editable
                .not('[data-editor-message]')
                .attr('data-editor-message', this.env._t('DRAG BUILDING BLOCKS HERE'));
            $editable.filter(':empty').attr('contenteditable', false);

            // Wait the next tick because some mutation have to be processed by
            // the Odoo editor before resetting the history.
            setTimeout(() => {
                this.wysiwyg.historyReset();
                // Update undo/redo buttons
                this.wysiwyg.odooEditor.dispatchEvent(new Event('historyStep'));

                // The selection has been lost when switching theme.
                const document = this.wysiwyg.odooEditor.document;
                const selection = document.getSelection();
                const p = this.wysiwyg.odooEditor.editable.querySelector('p');
                if (p) {
                    const range = document.createRange();
                    range.setStart(p, 0);
                    range.setEnd(p, 0);
                    selection.removeAllRanges();
                    selection.addRange(range);
                }
                // mark selection done for tour testing
                $editable.addClass('theme_selection_done');
                this.onIframeUpdated();
            }, 0);
        });

        // Remove the mailing from the favorites list
        $themeSelectorNew.on('click', '.o_mail_template_preview i.o_mail_template_remove_favorite', async (ev) => {
            ev.stopPropagation();
            ev.preventDefault();

            const $target = $(ev.currentTarget);
            const mailingId = $target.data('id');

            const rpcQuery = buildQuery({
                model: 'mailing.mailing',
                method: 'action_remove_favorite',
                args: [mailingId],
            })
            const action = await this.rpc(rpcQuery.route, rpcQuery.params);

            this.action.doAction(action);

            $target.parents('.o_mail_template_preview').remove();
        });

        // Clear any previous theme class before adding new one.
        this.wysiwyg.$iframeBody.closest('body').removeClass(this._themeClassNames);
        let selectedTheme = this._getSelectedTheme(themesParams);
        if (selectedTheme) {
            this.wysiwyg.$iframeBody.closest('body').addClass(selectedTheme.className);
            this._switchImages(selectedTheme, $snippets);
        } else if (this.wysiwyg.$iframeBody.find('.o_layout').length) {
            themesParams.push({
                name: 'o_mass_mailing_no_theme',
                className: 'o_mass_mailing_no_theme',
                img: "",
                template: this.wysiwyg.$iframeBody.find('.o_layout').addClass('o_mass_mailing_no_theme').clone().find('oe_structure').empty().end().html().trim(),
                nowrap: true,
                get_image_info: function () {}
            });
            selectedTheme = this._getSelectedTheme(themesParams);
        }

        this.wysiwyg.setSnippetsMenuFolded(device.isMobile || (selectedTheme && selectedTheme.name === 'basic'));

        this.wysiwyg.$iframeBody.find('.iframe-utils-zone').removeClass('d-none');
        if (this.env.mailingFilterTemplates && this.wysiwyg) {
            this._hideIrrelevantTemplates();
        }
        this.wysiwyg.odooEditor.activateContenteditable();
    }
    _getCodeViewEl() {
        const codeView = this.wysiwyg &&
            this.wysiwyg.$iframe &&
            this.wysiwyg.$iframe.contents().find('textarea.o_codeview')[0];
        return codeView && !codeView.classList.contains('d-none') && codeView;
    }
    /**
     * The .o_mail_wrapper_td element is where snippets can be dropped into.
     * This getter wraps the codeview value in such element in case it got
     * removed during edition in the codeview, in order to preserve the snippets
     * dropping functionality.
     */
    _getCodeViewValue(codeViewEl) {
        const editable = this.wysiwyg.$editable[0];
        const initialDropZone = editable.querySelector('.o_mail_wrapper_td');
        if (initialDropZone) {
            const parsedHtml = new DOMParser().parseFromString(codeViewEl.value, "text/html");
            if (!parsedHtml.querySelector('.o_mail_wrapper_td')) {
                initialDropZone.replaceChildren(...parsedHtml.body.childNodes);
                return editable.innerHTML;
            }
        }
        return codeViewEl.value;
    }
    /**
     * This method will take the model in argument and will hide all mailing template
     * in the mass mailing widget that do not belong to this model.
     *
     * This will also update the help message in the same widget to include the
     * new model name.
     *
     * @param {Number} modelId
     * @param {String} modelName
     *
     * @private
     */
    _hideIrrelevantTemplates() {
        const iframeContent = this.wysiwyg.$iframe.contents();

        const mailing_model_id = this.props.record.data.mailing_model_id[0];
        iframeContent
            .find(`.o_mail_template_preview[model-id!="${mailing_model_id}"]`)
            .addClass('d-none')
            .removeClass('d-inline-block');

        const sameModelTemplates = iframeContent
            .find(`.o_mail_template_preview[model-id="${mailing_model_id}"]`);

        sameModelTemplates
            .removeClass('d-none')
            .addClass('d-inline-block');

        // Hide or show the help message and preview wrapper based on whether there are any relevant templates
        if (sameModelTemplates.length) {
            iframeContent.find('.o_mailing_template_message').addClass('d-none');
            iframeContent.find('.o_mailing_template_preview_wrapper').removeClass('d-none');
        } else {
            iframeContent.find('.o_mailing_template_message').removeClass('d-none');
            iframeContent.find('.o_mailing_template_message span').text(this.props.record.data.mailing_model_id[1]);
            iframeContent.find('.o_mailing_template_preview_wrapper').addClass('d-none');
        }
    }
    /**
     * Returns the selected theme, if any.
     *
     * @private
     * @param {Object} themesParams
     * @returns {false|Object}
     */
    _getSelectedTheme(themesParams) {
        const $layout = this.wysiwyg.$iframeBody.find(".o_layout");
        let selectedTheme = false;
        if ($layout.length !== 0) {
            _.each(themesParams, function (themeParams) {
                if ($layout.hasClass(themeParams.className)) {
                    selectedTheme = themeParams;
                }
            });
        }
        return selectedTheme;
    }
    /**
     * Swap the previous theme's default images with the new ones.
     * (Redefine the `src` attribute of all images in a $container, depending on the theme parameters.)
     *
     * @private
     * @param {Object} themeParams
     * @param {JQuery} $container
     */
    _switchImages(themeParams, $container) {
        if (!themeParams) {
            return;
        }
        for (const img of $container.find("img")) {
            const $img = $(img);
            const src = $img.attr("src");

            let m = src.match(/^\/web\/image\/\w+\.s_default_image_(?:theme_[a-z]+_)?(.+)$/);
            if (!m) {
                m = src.match(/^\/\w+\/static\/src\/img\/(?:theme_[a-z]+\/)?s_default_image_(.+)\.[a-z]+$/);
            }
            if (!m) {
                return;
            }

            if (themeParams.get_image_info) {
                const file = m[1];
                const imgInfo = themeParams.get_image_info(file);

                const src = imgInfo.format
                    ? `/${imgInfo.module}/static/src/img/theme_${themeParams.name}/s_default_image_${file}.${imgInfo.format}`
                    : `/web/image/${imgInfo.module}.s_default_image_theme_${themeParams.name}_${file}`;

                $img.attr('src', src);
            }
        }
    }
    /**
     * Switch themes or import first theme.
     *
     * @private
     * @param {Object} themeParams
     */
    async _switchThemes(themeParams) {
        if (!themeParams || this.switchThemeLast === themeParams) {
            return;
        }
        this.switchThemeLast = themeParams;

        this.wysiwyg.$iframeBody.closest('body').removeClass(this._themeClassNames).addClass(themeParams.className);

        const old_layout = this.wysiwyg.$editable.find('.o_layout')[0];

        let $newWrapper;
        let $newWrapperContent;
        if (themeParams.nowrap) {
            $newWrapper = $('<div/>', {
                class: 'oe_structure'
            });
            $newWrapperContent = $newWrapper;
        } else {
            // This wrapper structure is the only way to have a responsive
            // and centered fixed-width content column on all mail clients
            $newWrapper = $('<div/>', {
                class: 'container o_mail_wrapper o_mail_regular oe_unremovable',
            });
            $newWrapperContent = $('<div/>', {
                class: 'col o_mail_no_options o_mail_wrapper_td bg-white oe_structure o_editable'
            });
            $newWrapper.append($('<div class="row"/>').append($newWrapperContent));
        }
        const $newLayout = $('<div/>', {
            class: 'o_layout oe_unremovable oe_unmovable bg-200 ' + themeParams.className,
            style: themeParams.layoutStyles,
            'data-name': 'Mailing',
        }).append($newWrapper);

        const $contents = themeParams.template;
        $newWrapperContent.append($contents);
        this._switchImages(themeParams, $newWrapperContent);
        old_layout && old_layout.remove();
        this.wysiwyg.odooEditor.resetContent($newLayout[0].outerHTML);

        $newWrapperContent.find('*').addBack()
            .contents()
            .filter(function () {
                return this.nodeType === 3 && this.textContent.match(/\S/);
            }).parent().addClass('o_default_snippet_text');

        if (themeParams.name === 'basic') {
            this.wysiwyg.$editable[0].focus();
        }
        initializeDesignTabCss(this.wysiwyg.$editable);
        this.wysiwyg.trigger('reload_snippet_dropzones');
        this.onIframeUpdated();
        this.wysiwyg.odooEditor.historyStep(true);
        // The value of the field gets updated upon editor blur. If for any
        // reason, the selection was not in the editable before modifying
        // another field, ensure that the value is properly set.
        this._switchingTheme = true;
        await this.commitChanges();
        this._switchingTheme = false;
    }
    async _getWysiwygClass() {
        return getWysiwygClass({moduleName: 'mass_mailing.wysiwyg'});
    }
    /**
     * @override
     */
    async _setupReadonlyIframe() {
        if (!this.props.value.length) {
            this.props.value = this.props.record.data.body_html;
        }
        await super._setupReadonlyIframe();
    }
}

MassMailingHtmlField.props = {
    ...standardFieldProps,
    ...HtmlField.props,
    filterTemplates: { type: Boolean, optional: true },
    inlineField: { type: String, optional: true },
    iframeHtmlClass: { type: String, optional: true },
};

MassMailingHtmlField.displayName = _lt("Email");
MassMailingHtmlField.extractProps = (...args) => {
    const [{ attrs }] = args;
    const htmlProps = HtmlField.extractProps(...args);
    return {
        ...htmlProps,
        filterTemplates: attrs.options.filterTemplates,
        inlineField: attrs.options['inline-field'],
        iframeHtmlClass: attrs['iframeHtmlClass'],
    };
};
MassMailingHtmlField.fieldDependencies = {
    body_html: { type: 'html' },
};

registry.category("fields").add("mass_mailing_html", MassMailingHtmlField);

```

## File: static\src\js\mass_mailing_link_dialog_fix.js

```javascript

odoo.define('mass_mailing.fix.LinkDialog', function (require) {
'use strict';

const LinkDialog = require('wysiwyg.widgets.LinkDialog');

/**
 * Primary and link buttons are "hacked" by mailing themes scss. We thus
 * have to fix their preview if possible.
 */
LinkDialog.include({
    /**
     * @override
     */
    start() {
        const ret = this._super(...arguments);
        if (!$(this.editable).find('.o_mail_wrapper').length) {
            return ret;
        }

        this.opened().then(() => {
            // Ugly hack to show the real color for link and primary which
            // depend on the mailing themes. Note: the hack is not enough as
            // the mailing theme changes those colors in some environment,
            // sometimes (for example 'btn-primary in this snippet looks like
            // that')... we'll consider this a limitation until a master
            // refactoring of those mailing themes.
            this.__realMMColors = {};
            const $previewArea = $('<div/>').addClass('o_mail_snippet_general');
            $(this.editable).find('.o_layout').append($previewArea);
            _.each(['link', 'primary', 'secondary'], type => {
                const $el = $('<a href="#" class="btn btn-' + type + '"/>');
                $el.appendTo($previewArea);
                this.__realMMColors[type] = {
                    'border-color': $el.css('border-top-color'),
                    'background-color': $el.css('background-color'),
                    'color': $el.css('color'),
                };
                $el.remove();

                this.$('label > .o_btn_preview.btn-' + type)
                    .css(_.pick(this.__realMMColors[type], 'background-color', 'color'));
            });
            $previewArea.remove();

            this._adaptPreview();
        });

        return ret;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _adaptPreview() {
        this._super(...arguments);
        if (this.__realMMColors) {
            var $preview = this.$("#link-preview");
            $preview.css('border-color', '');
            $preview.css('background-color', '');
            $preview.css('color', '');
            _.each(['link', 'primary', 'secondary'], type => {
                if ($preview.hasClass('btn-' + type) || type === 'link' && !$preview.hasClass('btn')) {
                    $preview.css(this.__realMMColors[type]);
                }
            });
        }
    },
});

});

```

## File: static\src\js\mass_mailing_mobile_preview.js

```javascript
/** @odoo-module **/

import { useService } from "@web/core/utils/hooks";
import { Dialog } from "@web/core/dialog/dialog";
const { useEffect, onWillStart } = owl;

export class MassMailingMobilePreviewDialog extends Dialog {
    setup() {
        super.setup();
        this.rpc = useService("rpc");
        onWillStart(async () => {
            this.styleSheets = await this.rpc("/mailing/get_preview_assets");
        });
        useEffect((modalEl) => {
            if (modalEl) {
                const modalBody = modalEl.querySelector('.modal-body');
                const invertIcon = document.createElement("span");
                invertIcon.className = "fa fa-refresh";
                const iframe = document.createElement("iframe");
                iframe.srcdoc = this._getSourceDocument();

                modalEl.classList.add('o_mailing_mobile_preview');
                modalEl.querySelector('.modal-title').append(invertIcon);
                modalEl.querySelector('.modal-header').addEventListener('click', () => modalBody.classList.toggle('o_invert_orientation'));
                modalBody.append(iframe);
            }
        }, () => [document.querySelector(':not(.o_inactive_modal).o_dialog')]);
    }

    _getSourceDocument() {
        return '<!DOCTYPE html><html>' +
                    '<head>' + this.styleSheets + '</head>' +
                    '<body>' + this.props.preview + '</body>' +
                '</html>';
    }
}

MassMailingMobilePreviewDialog.props = {
    ...Dialog.props,
    preview: { type: String },
    close: Function,
};
delete MassMailingMobilePreviewDialog.props.slots;

```

## File: static\src\js\mass_mailing_snippets.js

```javascript
odoo.define('mass_mailing.snippets.options', function (require) {
"use strict";

const options = require('web_editor.snippets.options');
const {loadImage} = require('web_editor.image_processing');
const {ColorpickerWidget} = require('web.Colorpicker');
const SelectUserValueWidget = options.userValueWidgetsRegistry['we-select'];
const weUtils = require('web_editor.utils');
const {
    CSS_PREFIX, BTN_SIZE_STYLES,
    DEFAULT_BUTTON_SIZE, PRIORITY_STYLES, FONT_FAMILIES,
    getFontName, normalizeFontFamily, initializeDesignTabCss,
    transformFontFamilySelector,
} = require('mass_mailing.design_constants');


//--------------------------------------------------------------------------
// Options
//--------------------------------------------------------------------------

// Snippet option for resizing  image and column width inline like excel
options.registry.mass_mailing_sizing_x = options.Class.extend({
    /**
     * @override
     */
    start: function () {
        var def = this._super.apply(this, arguments);

        this.containerWidth = this.$target.parent().closest("td, table, div").width();

        var self = this;
        var offset, sib_offset, target_width, sib_width;

        this.$overlay.find(".o_handle.e, .o_handle.w").removeClass("readonly");
        this.isIMG = this.$target.is("img");
        if (this.isIMG) {
            this.$overlay.find(".o_handle.w").addClass("readonly");
        }

        var $body = $(this.ownerDocument.body);
        this.$overlay.find(".o_handle").on('mousedown', function (event) {
            event.preventDefault();
            var $handle = $(this);
            var compass = false;

            _.each(['n', 's', 'e', 'w'], function (handler) {
                if ($handle.hasClass(handler)) { compass = handler; }
            });
            if (self.isIMG) { compass = "image"; }

            $body.on("mousemove.mass_mailing_width_x", function (event) {
                event.preventDefault();
                offset = self.$target.offset().left;
                target_width = self.get_max_width(self.$target);
                if (compass === 'e' && self.$target.next().offset()) {
                    sib_width = self.get_max_width(self.$target.next());
                    sib_offset = self.$target.next().offset().left;
                    self.change_width(event, self.$target, target_width, offset, true);
                    self.change_width(event, self.$target.next(), sib_width, sib_offset, false);
                }
                if (compass === 'w' && self.$target.prev().offset()) {
                    sib_width = self.get_max_width(self.$target.prev());
                    sib_offset = self.$target.prev().offset().left;
                    self.change_width(event, self.$target, target_width, offset, false);
                    self.change_width(event, self.$target.prev(), sib_width, sib_offset, true);
                }
                if (compass === 'image') {
                    const maxWidth = self.$target.closest("div").width();
                    // Equivalent to `self.change_width` but ensuring `maxWidth` is the maximum:
                    self.$target.css("width", Math.min(maxWidth, Math.round(event.pageX - offset)));
                    self.trigger_up('cover_update');
                }
            });
            $body.one("mouseup", function () {
                $body.off('.mass_mailing_width_x');
            });
        });

        return def;
    },
    change_width: function (event, target, target_width, offset, grow) {
        target.css("width", Math.round(grow ? (event.pageX - offset) : (offset + target_width - event.pageX)));
        this.trigger_up('cover_update');
    },
    get_int_width: function (el) {
        return parseInt($(el).css("width"), 10);
    },
    get_max_width: function ($el) {
        return this.containerWidth - _.reduce(_.map($el.siblings(), this.get_int_width), function (memo, w) { return memo + w; });
    },
    onFocus: function () {
        this._super.apply(this, arguments);

        if (this.$target.is("td, th")) {
            this.$overlay.find(".o_handle.e, .o_handle.w").toggleClass("readonly", this.$target.siblings().length === 0);
        }
    },
});

// Adding compatibility for the outlook compliance of mailings.
// Commit of such compatibility : a14f89c8663c9cafecb1cc26918055e023ecbe42
options.registry.MassMailingBackgroundImage = options.registry.BackgroundImage.extend({
    start: function () {
        this._super();
        const $table_target = this.$target.find('table:first');
        if ($table_target.length) {
            this.$target = $table_target;
        }
    }
});

options.registry.MassMailingImageTools = options.registry.ImageTools.extend({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _getCSSColorValue(color) {
        if (!color || ColorpickerWidget.isCSSColor(color)) {
            return color;
        }
        const doc = this.options.document;
        const tempEl = doc.body.appendChild(doc.createElement('div'));
        tempEl.className = `bg-${color}`;
        const colorValue = window.getComputedStyle(tempEl).getPropertyValue("background-color").trim();
        tempEl.parentNode.removeChild(tempEl);
        return ColorpickerWidget.normalizeCSSColor(colorValue).replace(/"/g, "'");
    },

    /**
     * @override
     */
    async computeShape(svgText, img) {
        const dataURL = await this._super(...arguments);
        const image = await loadImage(dataURL);
        const canvas = document.createElement("canvas");
        const imgFilename = (img.dataset.originalSrc.split("/").pop()).split(".")[0];
        img.dataset.fileName = `${imgFilename}.png`;
        img.dataset.mimetype = "image/png";
        canvas.width = image.width;
        canvas.height = image.height;
        canvas.getContext("2d").drawImage(image, 0, 0, image.width, image.height);
        return canvas.toDataURL(`image/png`, 1.0);
    }
});

options.userValueWidgetsRegistry['we-fontfamilypicker'] = SelectUserValueWidget.extend({
    /**
     * @override
     * @see FONT_FAMILIES
     */
    start: async function () {
        const res = await this._super(...arguments);
        // Populate the `we-select` with the font family buttons
        for (const fontFamily of FONT_FAMILIES) {
            const button = document.createElement('we-button');
            button.style.setProperty('font-family', fontFamily);
            button.dataset.customizeCssProperty = fontFamily;
            button.dataset.cssProperty = 'font-family';
            button.dataset.selectorText = this.el.dataset.selectorText;
            button.textContent = getFontName(fontFamily);
            this.menuEl.appendChild(button);
        };
        return res;
    },
});

options.registry.DesignTab = options.Class.extend({
    /**
     * @override
     */
    init() {
        this._super(...arguments);
        // Set the target on the whole editable so apply-to looks within it.
        this.setTarget(this.options.wysiwyg.getEditable());
    },
    /**
     * @override
     */
    async start() {
        const res = await this._super(...arguments);
        const $editable = this.options.wysiwyg.getEditable();
        this.document = $editable[0].ownerDocument;
        this.$layout = $editable.find('.o_layout');
        initializeDesignTabCss($editable);
        this.styleElement = this.document.querySelector('#design-element');
        // When editing a stylesheet, its content is not updated so it won't be
        // saved along with the mailing. Therefore we need to write its cssText
        // into it. However, when doing that we lose its reference. So we need
        // two separate style elements: one that will be saved and one to hold
        // the stylesheet. Both need to be synchronized, which will be done via
        // `_commitCss`.
        let sheetOwner = this.document.querySelector('#sheet-owner');
        if (!sheetOwner) {
            sheetOwner = document.createElement('style');
            sheetOwner.setAttribute('id', 'sheet-owner');
            this.document.head.appendChild(sheetOwner);
        }
        sheetOwner.disabled = true;
        sheetOwner.textContent = this.styleElement.textContent;
        this.styleSheet = sheetOwner.sheet;
        return res;
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Option method to set a css property in the mailing's custom stylesheet.
     * Note: marks all the styles as important to make sure they take precedence
     * on other stylesheets.
     *
     * @param {boolean|string} previewMode
     * @param {string} widgetValue
     * @param {Object} params
     * @param {string} params.selectorText the css selector for which to apply
     *                                     the css
     * @param {string} params.cssProperty the name of the property to edit
     *                                    (camel cased)
     * @param {string} [params.toggle] if 'true', will remove the property if
     *                                 its value is already the one it's being
     *                                 set to
     * @param {string} [params.activeValue] the value to set, if `widgetValue`
     *                                      is not defined.
     * @returns {Promise|undefined}
     */
    customizeCssProperty(previewMode, widgetValue, params) {
        if (!params.selectorText || !params.cssProperty) {
            return;
        }
        let value = widgetValue || params.activeValue;
        if (params.cssProperty.includes('color')) {
            value = weUtils.normalizeColor(value);
        }
        let selectors = this._getSelectors(params.selectorText);
        const firstSelector = selectors[0].replace(CSS_PREFIX, '').trim();
        if (params.cssProperty === 'font-family') {
            // Ensure font-family gets passed to all descendants and never
            // overwrite font awesome.
            const newSelectors = [];
            for (const selector of selectors) {
                newSelectors.push(...transformFontFamilySelector(selector));
            }
            selectors = [...new Set(newSelectors)];
        }
        for (const selector of selectors) {
            const priority = PRIORITY_STYLES[firstSelector].includes(params.cssProperty) ? ' !important' : '';
            const rule = this._getRule(selector);
            if (rule) {
                // The rule exists: update it.
                if (params.toggle === 'true' && rule.style.getPropertyValue(params.cssProperty) === value) {
                    rule.style.removeProperty(params.cssProperty);
                } else {
                    // Convert the style to css text and add the new style (the
                    // `style` property is readonly, we can only edit
                    // `cssText`).
                    const cssTexts = [];
                    for (const style of rule.style) {
                        const ownPriority = rule.style.getPropertyPriority(style) ? ' !important' : '';
                        if (style !== params.cssProperty) {
                            cssTexts.push(`${style}: ${rule.style[style]}${ownPriority};`);
                        }
                    }
                    cssTexts.push(`${params.cssProperty}: ${value}${priority};`);
                    rule.style.cssText = cssTexts.join('\n'); // Apply the new css text.
                }
            } else {
                // The rule doesn't exist: create it.
                this.styleSheet.insertRule(`${selector} {
                    ${params.cssProperty}: ${value}${priority};
                }`);
            }
        }
        this._commitCss();
    },
    /**
     * Option method to change the size of buttons.
     *
     * @see BTN_SIZE_STYLES
     * @param {boolean|string} previewMode
     * @param {string} widgetValue ('btn-sm'|'btn-md'|'btn-lg'|''|undefined)
     * @param {Object} params
     * @returns {Promise|undefined}
     */
     applyButtonSize(previewMode, widgetValue, params) {
        for (const [styleName, styleValue] of Object.entries(BTN_SIZE_STYLES[widgetValue || params.activeValue || DEFAULT_BUTTON_SIZE])) {
            if (styleValue) {
                this.customizeCssProperty(previewMode, styleValue, Object.assign({}, params, { cssProperty: styleName }));
            } else {
                // If the value is falsy, remove the property.
                for (const selector of this._getSelectors(params.selectorText)) {
                    const rule = this._getRule(selector);
                    if (rule) {
                        rule.style.removeProperty(styleName);
                    }
                }
            }
        }
        this._commitCss();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Apply the stylesheet's css text to the style element that will be saved.
     */
    _commitCss() {
        const cssTexts = [];
        for (const rule of this.styleSheet.cssRules || this.styleSheet.rules) {
            cssTexts.push(rule.cssText);
        }
        this.styleElement.textContent = cssTexts.join('\n');
        // Flush the rules cache for convert_inline, to make sure they are
        // recomputed to account for the change.
        this.options.wysiwyg._rulesCache = undefined;
    },
    /**
     * @override
     */
    async _computeWidgetState(methodName, params) {
        const res = await this._super(...arguments);
        if (res === undefined) {
            switch (methodName) {
                case 'applyButtonSize':
                case 'customizeCssProperty': {
                    if (!params.selectorText) {
                        return;
                    }
                    // Here we parse the selector in order to create a matching
                    // element that we inject into the DOM so we can retrieve
                    // its computed style. We then remove the element from the
                    // DOM, no harm, no foul.
                    const firstSelector = params.selectorText.split(',')[0].replace(CSS_PREFIX, '').trim();
                    const classes = firstSelector.replace(/:not\([^\)]*\)/g, '').match(/\.([\w\d-_]+)/g) || [];
                    const fakeElement = document.createElement(firstSelector.split(/[\.:, ]/)[0]);
                    for (const className of classes) {
                        fakeElement.classList.toggle(className.replace('.', ''), true);
                    }
                    this.$layout.find(CSS_PREFIX).prepend(fakeElement);
                    let res;
                    if (methodName === 'applyButtonSize') {
                        // Match a button size by its padding value.
                        const padding = getComputedStyle(fakeElement).padding;
                        const classIndex = Object.values(BTN_SIZE_STYLES).findIndex(style => style.padding === padding);
                        res = classIndex >= 0 ? Object.keys(BTN_SIZE_STYLES)[classIndex] : DEFAULT_BUTTON_SIZE;
                    } else {
                        fakeElement.style.display = 'none'; // Needed to get width in %.
                        res = getComputedStyle(fakeElement)[params.cssProperty || 'font-family'];
                        if (params.possibleValues && params.possibleValues[1] === FONT_FAMILIES[0]) {
                            // For font-family, we need to normalize it so it
                            // matches an option value.
                            res = normalizeFontFamily(res);
                        }
                        if (params.cssProperty === 'font-weight') {
                            res = parseInt(res) >= 600 ? 'bolder' : '';
                        } else if (res === 'auto') {
                            res = '100%';
                        }
                    }
                    fakeElement.remove();
                    return res;
                }
                case 'applyButtonSize':
                    // Match a button size by its padding value.
                    const rule = this._getRule(this._getSelectors(params.selectorText)[0]);
                    if (rule) {
                        const classIndex = Object.values(BTN_SIZE_STYLES).findIndex(style => style.padding === rule.style.padding);
                        return classIndex >= 0 ? Object.keys(BTN_SIZE_STYLES)[classIndex] : DEFAULT_BUTTON_SIZE;
                    } else {
                        return DEFAULT_BUTTON_SIZE;
                    }
            }
        } else {
            return res;
        }
    },
    /**
     * Take a CSS selector and split it into separate selectors, all prefixed
     * with the `CSS_PREFIX`. Return them as an array.
     *
     * @see CSS_PREFIX
     * @param {string} selectorText
     * @returns {string[]}
     */
    _getSelectors(selectorText) {
        return selectorText.split(',').map(t => `${t.startsWith(CSS_PREFIX) ? '' : CSS_PREFIX + ' '}${t.trim()}`.trim());;
    },
    /**
     * Take a CSS selector and find its matching rule in the mailing's custom
     * stylesheet, if it exists.
     *
     * @param {string} selectorText
     * @returns {CSSStyleRule|undefined}
     */
    _getRule(selectorText) {
        return [...(this.styleSheet.cssRules || this.styleSheet.rules)].find(rule => rule.selectorText === selectorText);
    },
});

});

```

## File: static\src\js\snippets.editor.js

```javascript
odoo.define('mass_mailing.snippets.editor', function (require) {
'use strict';

const {_lt} = require('web.core');
const snippetsEditor = require('web_editor.snippet.editor');

const MassMailingSnippetsMenu = snippetsEditor.SnippetsMenu.extend({
    events: _.extend({}, snippetsEditor.SnippetsMenu.prototype.events, {
        'click .o_we_customize_design_btn': '_onDesignTabClick',
    }),
    custom_events: _.extend({}, snippetsEditor.SnippetsMenu.prototype.custom_events, {
        drop_zone_over: '_onDropZoneOver',
        drop_zone_out: '_onDropZoneOut',
        drop_zone_start: '_onDropZoneStart',
        drop_zone_stop: '_onDropZoneStop',
    }),
    tabs: _.extend({}, snippetsEditor.SnippetsMenu.prototype.tabs, {
        DESIGN: 'design',
    }),
    optionsTabStructure: [
        ['design-options', _lt("Design Options")],
    ],

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    start: function () {
        return this._super(...arguments).then(() => {
            this.$editable = this.options.wysiwyg.getEditable();
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _onClick: function (ev) {
        this._super(...arguments);
        var srcElement = ev.target || (ev.originalEvent && (ev.originalEvent.target || ev.originalEvent.originalTarget)) || ev.srcElement;
        // When we select something and move our cursor too far from the editable area, we get the
        // entire editable area as the target, which causes the tab to shift from OPTIONS to BLOCK.
        // To prevent unnecessary tab shifting, we provide a selection for this specific case.
        if (srcElement.classList.contains('o_mail_wrapper') || srcElement.querySelector('.o_mail_wrapper')) {
            const selection = this.options.wysiwyg.odooEditor.document.getSelection();
            if (selection.anchorNode) {
                const parent = selection.anchorNode.parentElement;
                if (parent) {
                    srcElement = parent;
                }
                this._activateSnippet($(srcElement));
            }
        }
    },
    /**
     * @override
     */
    _insertDropzone: function ($hook) {
        const $hookParent = $hook.parent();
        const $dropzone = this._super(...arguments);
        $dropzone.attr('data-editor-message', $hookParent.attr('data-editor-message'));
        $dropzone.attr('data-editor-sub-message', $hookParent.attr('data-editor-sub-message'));
        return $dropzone;
    },
    /**
     * @override
     */
    _updateRightPanelContent: function ({content, tab}) {
        this._super(...arguments);
        this.$('.o_we_customize_design_btn').toggleClass('active', tab === this.tabs.DESIGN);
    },
    /**
     * @override
     */
    _computeSnippetTemplates: function (html) {
        const $html = $(html);
        const btnSelector = '.note-editable .oe_structure > div.o_mail_snippet_general .btn:not(.btn-link)';
        const $colorpickers = $html.find('[data-selector] > we-colorpicker[data-css-property="background-color"]');
        for (const colorpicker of $colorpickers) {
            const $option = $(colorpicker).parent();
            const selectors = $option.data('selector').split(',');
            const filteredSelectors = selectors.filter(selector => !selector.includes(btnSelector)).join(',');
            $option.attr('data-selector', filteredSelectors);
        }
        html = $html.toArray().map(node => node.outerHTML).join('');
        return this._super(html);
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _onDropZoneOver: function () {
        this.$editable.find('.o_editable').css('background-color', '');
    },
    /**
     * @override
     */
    _onDropZoneOut: function () {
        const $oEditable = this.$editable.find('.o_editable');
        if ($oEditable.find('.oe_drop_zone.oe_insert:not(.oe_vertical):only-child').length) {
            $oEditable[0].style.setProperty('background-color', 'transparent', 'important');
        }
    },
    /**
     * @override
     */
    _onDropZoneStart: function () {
        const $oEditable = this.$editable.find('.o_editable');
        if ($oEditable.find('.oe_drop_zone.oe_insert:not(.oe_vertical):only-child').length) {
            $oEditable[0].style.setProperty('background-color', 'transparent', 'important');
        }
    },
    /**
     * @override
     */
    _onDropZoneStop: function () {
        const $oEditable = this.$editable.find('.o_editable');
        $oEditable.css('background-color', '');
        if (!$oEditable.find('.oe_drop_zone.oe_insert:not(.oe_vertical):only-child').length) {
            $oEditable.attr('contenteditable', true);
        }
        // Refocus again to save updates when calling `_onWysiwygBlur`
        this.$editable.focus();
    },
    /**
     * @override
     */
    _onSnippetRemoved: function () {
        this._super(...arguments);
        const $oEditable = this.$editable.find('.o_editable');
        if (!$oEditable.children().length) {
            $oEditable.empty(); // remove any superfluous whitespace
            $oEditable.attr('contenteditable', false);
        }
    },
    /**
     * @private
     */
    async _onDesignTabClick() {
        // Note: nothing async here but start the loading effect asap
        let releaseLoader;
        try {
            const promise = new Promise(resolve => releaseLoader = resolve);
            this._execWithLoadingEffect(() => promise, false, 0);
            // loader is added to the DOM synchronously
            await new Promise(resolve => requestAnimationFrame(() => requestAnimationFrame(resolve)));
            // ensure loader is rendered: first call asks for the (already done) DOM update,
            // second call happens only after rendering the first "updates"

            if (!this.topFakeOptionEl) {
                let el;
                for (const [elementName, title] of this.optionsTabStructure) {
                    const newEl = document.createElement(elementName);
                    newEl.dataset.name = title;
                    if (el) {
                        el.appendChild(newEl);
                    } else {
                        this.topFakeOptionEl = newEl;
                    }
                    el = newEl;
                }
                this.bottomFakeOptionEl = el;
                this.el.appendChild(this.topFakeOptionEl);
            }

            // Need all of this in that order so that:
            // - the element is visible and can be enabled and the onFocus method is
            //   called each time.
            // - the element is hidden afterwards so it does not take space in the
            //   DOM, same as the overlay which may make a scrollbar appear.
            this.topFakeOptionEl.classList.remove('d-none');
            const editorPromise = this._activateSnippet($(this.bottomFakeOptionEl));
            releaseLoader(); // because _activateSnippet uses the same mutex as the loader
            releaseLoader = undefined;
            const editor = await editorPromise;
            this.topFakeOptionEl.classList.add('d-none');
            editor.toggleOverlay(false);

            this._updateRightPanelContent({
                tab: this.tabs.DESIGN,
            });
        } catch (e) {
            // Normally the loading effect is removed in case of error during the action but here
            // the actual activity is happening outside of the action, the effect must therefore
            // be cleared in case of error as well
            if (releaseLoader) {
                releaseLoader();
            }
            throw e;
        }
    },
});

return MassMailingSnippetsMenu;

});

```

## File: static\src\js\wysiwyg.js

```javascript
odoo.define('mass_mailing.wysiwyg', function (require) {
'use strict';

var Wysiwyg = require('web_editor.wysiwyg');
var MassMailingSnippetsMenu = require('mass_mailing.snippets.editor');
const {closestElement} = require('@web_editor/js/editor/odoo-editor/src/OdooEditor');
const Toolbar = require('web_editor.toolbar');

const MassMailingWysiwyg = Wysiwyg.extend({
    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    startEdition: async function () {
        const res = await this._super(...arguments);
        // Prevent selection change outside of snippets.
        this.$editable.on('mousedown', e => {
            if ($(e.target).is('.o_editable:empty') || e.target.querySelector('.o_editable')) {
                e.preventDefault();
            }
        });
        this.snippetsMenuToolbar = this.toolbar;
        return res;
    },

    toggleLinkTools(options = {}) {
        this._super({
            ...options,
            // Always open the dialog when the sidebar is folded.
            forceDialog: options.forceDialog || this.snippetsMenu.folded
        });
        if (this.snippetsMenu.folded) {
            // Hide toolbar and avoid it being re-displayed after getDeepRange.
            this.odooEditor.document.getSelection().collapseToEnd();
        }
    },

    /**
     * Sets SnippetsMenu fold state and switches toolbar.
     * Instantiates a new floating Toolbar if needed.
     *
     * @param {Boolean} fold
     */
    setSnippetsMenuFolded: async function (fold = true) {
        if (fold) {
            this.snippetsMenu.setFolded(true);
            if (!this.floatingToolbar) {
                // Instantiate and configure new toolbar.
                this.floatingToolbar = new Toolbar(this, 'web_editor.toolbar');
                this.toolbar = this.floatingToolbar;
                await this.toolbar.appendTo(document.createElement('void'));
                this._configureToolbar({ snippets: false });
                this._updateEditorUI();
                this.setCSSVariables(this.toolbar.el);
                this.odooEditor.setupToolbar(this.toolbar.el);
                if (this.odooEditor.isMobile) {
                    document.body.querySelector('.o_mail_body').prepend(this.toolbar.el);
                } else {
                    document.body.append(this.toolbar.el);
                }
            } else {
                this.toolbar = this.floatingToolbar;
            }
            this.toolbar.el.classList.remove('d-none');
            this.odooEditor.autohideToolbar = true;
            this.odooEditor.toolbarHide();
        } else {
            this.snippetsMenu.setFolded(false);
            this.toolbar = this.snippetsMenuToolbar;
            this.odooEditor.autohideToolbar = false;
            if (this.floatingToolbar) {
                this.floatingToolbar.el.classList.add('d-none');
            }
        }
        this.odooEditor.toolbar = this.toolbar.el;
    },

    /**
     * @override
     */
    openMediaDialog: function() {
        this._super(...arguments);
        // Opening the dialog in the outer document does not trigger the selectionChange
        // (that would normally hide the toolbar) in the iframe.
        if (this.snippetsMenu.folded) {
            this.odooEditor.toolbarHide();
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _createSnippetsMenuInstance: function (options={}) {
        return new MassMailingSnippetsMenu(this, Object.assign({
            wysiwyg: this,
            selectorEditableArea: '.o_editable',
        }, options));
    },
    /**
     * @override
     */
    _getPowerboxOptions: function () {
        const options = this._super();
        const {commands} = options;
        const linkCommands = commands.filter(command => command.name === 'Link' || command.name === 'Button');
        for (const linkCommand of linkCommands) {
            // Remove the command if the selection is within a background-image.
            const superIsDisabled = linkCommand.isDisabled;
            linkCommand.isDisabled = () => {
                if (superIsDisabled && superIsDisabled()) {
                    return true;
                } else {
                    const selection = this.odooEditor.document.getSelection();
                    const range = selection.rangeCount && selection.getRangeAt(0);
                    return !!range && !!closestElement(range.startContainer, '[style*=background-image]');
                }
            }
        }
        return {...options, commands};
    },
    /**
     * @override
     */
     _updateEditorUI: function (e) {
        this._super(...arguments);
        // Hide the create-link button if the selection is within a
        // background-image.
        const selection = this.odooEditor.document.getSelection();
        const range = selection.rangeCount && selection.getRangeAt(0);
        const isWithinBackgroundImage = !!range && !!closestElement(range.startContainer, '[style*=background-image]');
        if (isWithinBackgroundImage) {
            this.toolbar.$el.find('#create-link').toggleClass('d-none', true);
        }
    },
    _getEditorOptions: function () {
        const options = this._super(...arguments);
        const finalOptions = { autoActivateContentEditable: false, ...options };
        return finalOptions;
    },
});

return MassMailingWysiwyg;

});

```

## File: static\src\js\tours\mass_mailing_code_view.js

```javascript
odoo.define('mass_mailing.mass_mailing_code_view_tour', function (require) {
    "use strict";

    var tour = require('web_tour.tour');

    tour.register('mass_mailing_code_view_tour', {
        url: '/web',
        test: true,
    }, [tour.stepUtils.showAppsMenuItem(), {
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
    }, {
        trigger: 'button.o_list_button_add',
    }, {
        trigger: 'input#subject',
        content: ('Pick the <b>email subject</b>.'),
        position: 'bottom',
        run: 'text Test'
    }, {
        trigger: 'div[name="contact_list_ids"] .o_input_dropdown input[type="text"]',
        content: 'Click on the dropdown to open it and then start typing to search.',
    }, {
        trigger: 'div[name="contact_list_ids"] .ui-state-active',
        content: 'Select item from dropdown',
        run: 'click',
    }, {
        trigger: 'div[name="body_arch"] iframe #default',
        content: 'Choose this <b>theme</b>.',
        run: 'click',
    }, {
        trigger: 'iframe .o_codeview_btn',
        content: ('Click here to switch to <b>code view</b>'),
        run: 'click'
    }, {
        trigger: 'iframe .o_codeview',
        content: ('Remove all content from codeview'),
        run: function () {
            const iframe = document.querySelector('.wysiwyg_iframe');
            const iframeDocument = iframe.contentWindow.document;
            let element = iframeDocument.querySelector(".o_codeview");
            element.value = '';
        }
    }, {
        trigger: 'iframe .o_codeview_btn',
        content: ('Click here to switch back from <b>code view</b>'),
        run: 'click'
    }, {
        trigger: '[name="body_arch"] iframe .o_mail_wrapper_td',
        content: 'Verify that the dropable zone was not removed',
        run: () => {},
    }, {
        trigger: '[name="body_arch"] iframe #email_designer_default_body [name="Title"] .ui-draggable-handle',
        content: 'Drag the "Title" snippet from the design panel and drop it in the editor',
        run: function (actions) {
            actions.drag_and_drop('[name="body_arch"] iframe .o_editable', this.$anchor);
        }
    }, {
        trigger: '[name="body_arch"] iframe .o_editable h1',
        content: 'Verify that the title was inserted properly in the editor',
        run: () => {},
    }, {
        trigger: 'button.o_form_button_save',
        content: 'Click on the "Save" button to save the changes.',
        run: 'click',
    },
    ...tour.stepUtils.saveForm(),]);
});

```

## File: static\src\js\tours\mass_mailing_editor_tour.js

```javascript
odoo.define('mass_mailing.mass_mailing_editor_tour', function (require) {
    "use strict";

    var tour = require('web_tour.tour');
    const { boundariesIn, setSelection } = require('@web_editor/js/editor/odoo-editor/src/utils/utils');

    tour.register('mass_mailing_editor_tour', {
        url: '/web',
        test: true,
    }, [tour.stepUtils.showAppsMenuItem(), {
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
    }, {
        trigger: 'button.o_list_button_add',
    }, {
        trigger: 'div[name="contact_list_ids"] .o_input_dropdown input[type="text"]',
    }, {
        trigger: 'div[name="contact_list_ids"] .ui-state-active'
    }, {
        content: 'choose the theme "empty" to edit the mailing with snippets',
        trigger: '[name="body_arch"] iframe #empty',
    }, {
        content: 'wait for the editor to be rendered',
        trigger: '[name="body_arch"] iframe .o_editable[data-editor-message="DRAG BUILDING BLOCKS HERE"]',
        run: () => {},
    }, {
        content: 'drag the "Title" snippet from the design panel and drop it in the editor',
        trigger: '[name="body_arch"] iframe #email_designer_default_body [name="Title"] .ui-draggable-handle',
        run: function (actions) {
            actions.drag_and_drop('[name="body_arch"] iframe .o_editable', this.$anchor);
        }
    }, {
        content: 'wait for the snippet menu to finish the drop process',
        trigger: '[name="body_arch"] iframe #email_designer_header_elements:not(:has(.o_we_already_dragging))',
        run: () => {}
    }, {
        content: 'verify that the title was inserted properly in the editor',
        trigger: '[name="body_arch"] iframe .o_editable h1',
        run: () => {},
    }, {
        trigger: 'button.o_form_button_save',
    }, {
        content: 'verify that the save failed (since the field "subject" was not set and it is required)',
        trigger: 'label.o_field_invalid',
        run: () => {},
    }, {
        content: 'verify that the edited mailing body was not lost during the failed save',
        trigger: '[name="body_arch"] iframe .o_editable h1',
        run: () => {},
    }, {
        trigger: 'input#subject',
        run: 'text Test',
    }, {
        trigger: '.o_form_view', // blur previous input
    },
    ...tour.stepUtils.saveForm(),
    {
        trigger: 'iframe .o_editable',
        run: () => {},
    }]);

    tour.register('mass_mailing_basic_theme_toolbar', {
        test: true,
        url: '/web',
    }, [
        tour.stepUtils.showAppsMenuItem(),
        {
            content: "Select the 'Email Marketing' app.",
            trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
        },
        {
            content: "Click on the create button to create a new mailing.",
            trigger: 'button.o_list_button_add',
        },
        {
            content: "Fill in Subject",
            trigger: '#subject',
            run: 'text Test Basic Theme',
        },
        {
            content: "Fill in Mailing list",
            trigger: '#contact_list_ids',
            run: 'text Newsletter',
        },
        {
            content: "Pick 'Newsletter' option",
            trigger: '.o_input_dropdown a:contains(Newsletter)',
        },
        {
            content: "Pick the basic theme",
            trigger: 'iframe #basic',
            extra_trigger: 'iframe .o_mail_theme_selector_new',
        },
        {
            content: "Make sure the snippets menu is hidden",
            trigger: 'iframe html:has(#oe_snippets.d-none)',
            run: () => null, // no click, just check
        },
        {
            content: "Click on the New button to create another mailing",
            trigger: 'button.o_form_button_create',
        },
        {
            content: "Fill in Subject",
            trigger: '#subject',
            extra_trigger: 'iframe .o_mail_theme_selector_new',
            run: 'text Test Newsletter Theme',
        },
        {
            content: "Fill in Mailing list",
            trigger: '#contact_list_ids',
            run: 'text Newsletter',
        },
        {
            content: "Pick 'Newsletter' option",
            trigger: '.o_input_dropdown a:contains(Newsletter)',
        },
        {
            content: "Pick the newsletter theme",
            trigger: 'iframe #newsletter',
            // extra_trigger: 'iframe .o_mail_theme_selector_new',
        },
        {
            content: "Make sure the snippets menu is displayed",
            trigger: 'iframe #oe_snippets',
            run: () => null, // no click, just check
        },
        {
            content: 'Save form',
            trigger: '.o_form_button_save',
        },
        {
            content: 'Go back to previous mailing',
            trigger: 'button.o_pager_previous',
        },
        {
            content: "Make sure the snippets menu is hidden",
            trigger: 'iframe html:has(#oe_snippets.d-none)',
            run: () => null,
        },
        {
            content: "Add some content to be selected afterwards",
            trigger: 'iframe p',
            run: 'text content',
        },
        {
            content: "Select text",
            trigger: 'iframe p:contains(content)',
            run() {
                setSelection(...boundariesIn(this.$anchor[0]), false);
            }
        },
        {
            content: "Make sure the floating toolbar is visible",
            trigger: '#toolbar.oe-floating[style*="visible"]',
            run: () => null,
        },
        {
            content: "Open the color picker",
            trigger: '#toolbar #oe-text-color',
        },
        {
            content: "Pick a color",
            trigger: '#toolbar button[data-color="o-color-1"]',
        },
        {
            content: "Check that color was applied",
            trigger: 'iframe p font.text-o-color-1',
            run: () => null,
        },
        {
            content: 'Save changes',
            trigger: '.o_form_button_save',
        },
        {
            content: "Go to 'Mailings' list view",
            trigger: '.breadcrumb a:contains(Mailings)'
        },
        {
            content: "Open newly created mailing",
            trigger: 'td:contains("Test Basic Theme")',
        },
        {
            content: "Make sure the snippets menu is hidden",
            trigger: 'iframe html:has(#oe_snippets.d-none)',
            run: () => null,
        },
        {
            content: "Select content",
            trigger: 'iframe p:contains(content)',
            run() {
                setSelection(...boundariesIn(this.$anchor[0]), false);
            }
        },
        {
            content: "Make sure the floating toolbar is visible",
            trigger: '#toolbar.oe-floating[style*="visible"]',
            run: () => null,
        },
        ...tour.stepUtils.discardForm(),
    ]);

    tour.register('mass_mailing_campaing_new_mailing', {
        url: '/web',
        test: true,
    }, [
        tour.stepUtils.showAppsMenuItem(),
        {
            content: 'Select the "Email Marketing" app',
            trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
        },
        {
            content: 'Select "Campaings" Navbar item',
            trigger: '.o_nav_entry[data-menu-xmlid="mass_mailing.menu_email_campaigns"]',
        },
        {
            content: 'Select "Newsletter" campaign',
            trigger: '.oe_kanban_card:contains("Test Newsletter")',
        },
        {
            content: 'Add a line (create new mailing)',
            trigger: '.o_field_x2many_list_row_add a',
        },
        {
            content: 'Pick the basic theme',
            trigger: 'iframe',
            run(actions) {
                // For some reason the selectors inside the iframe cannot be triggered.
                const link = this.$anchor[0].contentDocument.querySelector('#basic');
                actions.click(link);
            }
        },
        {
            content: 'Fill in Subject',
            trigger: '#subject',
            run: 'text Test',
        },
        {
            content: 'Fill in Mailing list',
            trigger: '#contact_list_ids',
            run: 'text Test Newsletter',
        },
        {
            content: 'Pick "Newsletter" option',
            trigger: '.o_input_dropdown a:contains(Test Newsletter)',
        },
        {
            content: 'Save form',
            trigger: '.o_form_button_save',
        },
        {
            content: 'Check that newly created record is on the list',
            trigger: '[name="mailing_mail_ids"] td[name="subject"]:contains("Test")',
            run: () => null,
        },
        ...tour.stepUtils.saveForm(),
    ]);
});

```

## File: static\src\js\tours\mass_mailing_snippets_menu_tabs.js

```javascript
/** @odoo-module **/

import tour from 'web_tour.tour';

tour.register('mass_mailing_snippets_menu_tabs', {
    test: true,
    url: '/web',
}, [
    tour.stepUtils.showAppsMenuItem(), {
        content: "Select the 'Email Marketing' app.",
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
    },
    {
        content: "Click on the create button to create a new mailing.",
        trigger: 'button.o_list_button_add',
    },
    {
        content: "Click on the 'Start From Scratch' template.",
        trigger: 'iframe #empty',
    },
    {
        content: "Click on the 'Design' tab.",
        trigger: 'iframe .o_we_customize_design_btn',
    },
    {
        content: "Click on the empty 'DRAG BUILDING BLOCKS HERE' area.",
        trigger: 'iframe .oe_structure.o_mail_no_options',
    },
    {
        content: "Click on the 'Design' tab.",
        trigger: 'iframe .o_we_customize_design_btn',
    },
    {
        content: "Verify that the customize panel is not empty.",
        trigger: 'iframe .o_we_customize_panel .snippet-option-DesignTab',
        run: () => null, // it's a check
    },
    {
        content: "Click on the style tab.",
        trigger: 'iframe .o_we_customize_snippet_btn',
    },
    {
        content: "Click on the 'Design' tab.",
        trigger: 'iframe .o_we_customize_design_btn',
    },
    {
        content: "Verify that the customize panel is not empty.",
        trigger: 'iframe .o_we_customize_panel .snippet-option-DesignTab',
        run: () => null, // it's a check
    },
    ...tour.stepUtils.discardForm(),
]);


tour.register('mass_mailing_snippets_menu_toolbar_new_mailing_mobile', {
    test: true,
    url: '/web',
}, [
    tour.stepUtils.showAppsMenuItem(), {
        content: "Select the 'Email Marketing' app.",
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
    },
    {
        content: "Click on the create button to create a new mailing.",
        trigger: 'button.o_list_button_add',
        mobile: true,
    },
    {
        content: "Check templates available in theme selector",
        trigger: 'iframe .o_mail_theme_selector_new',
        run: function () {
            if (this.$anchor[0].querySelector('#empty')) {
                console.error('The empty template should not be visible on mobile.');
            }
        },
        mobile: true,
    },
    {
        content: "Make sure the toolbar isn't floating",
        trigger: 'iframe',
        run: function () {
            const iframeDocument = this.$anchor[0].contentDocument;
            if (iframeDocument.querySelector('#toolbar.oe-floating')) {
                console.error('There should not be a floating toolbar in the iframe');
            }
        },
        mobile: true,
    },
    {
        content: "Click on the 'Start From Scratch' template.",
        trigger: 'iframe #default',
        mobile: true,
    },
    {
        content: "Select an editable element",
        trigger: 'iframe .s_text_block',
        mobile: true,
    },
    {
        content: "Make sure the snippets menu is hidden",
        trigger: 'iframe',
        run: function () {
            const iframeDocument = this.$anchor[0].contentDocument;
            if (!iframeDocument.querySelector('#oe_snippets.d-none')) {
                console.error('The snippet menu should be hidden');
            }
        },
        mobile: true,
    },
    {
        content: "Make sure the toolbar is there, with the tables formating tool",
        trigger: 'iframe #toolbar.oe-floating #table:not(.d-none)',
        run: () => null, // it's a check
        mobile: true,
    },
]);

tour.register('mass_mailing_snippets_menu_toolbar', {
    test: true,
    url: '/web',
}, [
    tour.stepUtils.showAppsMenuItem(), {
        content: "Select the 'Email Marketing' app.",
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
    },
    {
        content: "Click on the create button to create a new mailing.",
        trigger: 'button.o_list_button_add',
    },
    {
        content: "Wait for the theme selector to load.",
        trigger: 'iframe .o_mail_theme_selector_new',
    },
    {
        content: "Make sure there does not exist a floating toolbar",
        trigger: 'iframe',
        run: function () {
            const iframeDocument = this.$anchor[0].contentDocument;
            if (iframeDocument.querySelector('#toolbar.oe-floating')) {
                console.error('There should not be a floating toolbar in the iframe');
            }
        },
    },
    {
        content: "Make sure the empty template is an option on non-mobile devices.",
        trigger: 'iframe #empty',
        run: () => null,
    },
    {
        content: "Click on the default 'welcome' template.",
        trigger: 'iframe #default',
    },
    { // necessary to wait for the cursor to be placed in the first p
      // and to avoid leaving the page before the selection is added
        content: "Wait for template selection event to be over.",
        trigger: 'iframe .o_editable.theme_selection_done',
    },
    {
        content: "Make sure the snippets menu is not hidden",
        trigger: 'iframe #oe_snippets:not(.d-none)',
        run: () => null,
    },
    {
        content: "Wait for .s_text_block to be populated",
        trigger: 'iframe .s_text_block p',
        run: () => null,
    },
    {
        content: "Click and select p block inside the editor",
        trigger: 'iframe',
        run: function () {
            const iframeWindow = this.$anchor[0].contentWindow;
            const iframeDocument = iframeWindow.document;
            const p = iframeDocument.querySelector('.s_text_block p');
            p.click();
            const selection = iframeWindow.getSelection();
            const range = iframeDocument.createRange();
            range.selectNodeContents(p);
            selection.removeAllRanges();
            selection.addRange(range);
        },
    },
    {
        content: "Make sure the toolbar is there",
        trigger: 'iframe #oe_snippets .o_we_customize_panel #toolbar',
        run: () => null,
    },
    ...tour.stepUtils.discardForm(),
]);

```

## File: static\src\js\tours\mass_mailing_tour.js

```javascript
odoo.define('mass_mailing.mass_mailing_tour', function (require) {
    "use strict";

    const {_t} = require('web.core');
    const {Markup} = require('web.utils');
    var tour = require('web_tour.tour');

    tour.register('mass_mailing_tour', {
        url: '/web',
        rainbowManMessage: _t('Congratulations, I love your first mailing. :)'),
        sequence: 200,
    }, [tour.stepUtils.showAppsMenuItem(), {
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
        content: _t("Let's try the Email Marketing app."),
        width: 225,
        position: 'bottom',
        edition: 'enterprise',
    }, {
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
        content: _t("Let's try the Email Marketing app."),
        edition: 'community',
    }, {
        trigger: '.o_list_button_add',
        extra_trigger: '.o_mass_mailing_mailing_tree',
        content: Markup(_t("Start by creating your first <b>Mailing</b>.")),
        position: 'bottom',
    }, {
        trigger: 'div[name="subject"]',
        content: Markup(_t('Pick the <b>email subject</b>.')),
        position: 'bottom',
        run: 'click',
    }, {
        trigger: 'div[name="contact_list_ids"] > .o_input_dropdown > input[type="text"]',
        run: 'click',
        auto: true,
    }, {
        trigger: 'li.ui-menu-item',
        run: 'click',
        auto: true,
    }, {
        trigger: 'div[name="body_arch"] iframe #newsletter',
        content: Markup(_t('Choose this <b>theme</b>.')),
        position: 'left',
        edition: 'enterprise',
        run: 'click',
    }, {
        trigger: 'div[name="body_arch"] iframe #default',
        content: Markup(_t('Choose this <b>theme</b>.')),
        position: 'right',
        edition: 'community',
        run: 'click',
    }, {
        trigger: 'div[name="body_arch"] iframe div.theme_selection_done div.s_text_block',
        content: _t('Click on this paragraph to edit it.'),
        position: 'top',
        edition: 'enterprise',
        run: 'click',
    }, {
        trigger: 'div[name="body_arch"] iframe div.o_mail_block_title_text',
        content: _t('Click on this paragraph to edit it.'),
        position: 'top',
        edition: 'community',
        run: 'click',
    }, {
        trigger: 'button[name="action_set_favorite"]',
        content: _t('Click on this button to add this mailing to your templates.'),
        position: 'bottom',
        run: 'click',
    }, {
        trigger: 'button[name="action_test"]',
        content: _t("Test this mailing by sending a copy to yourself."),
        position: 'bottom',
    }, {
        trigger: 'button[name="send_mail_test"]',
        content: _t("Check the email address and click send."),
        position: 'bottom',
    }, {
        trigger: 'button[name="action_launch"]',
        content: _t("Ready for take-off!"),
        position: 'bottom',
    }, {
        trigger: '.btn-primary:contains("Ok")',
        content: _t("Don't worry, the mailing contact we created is an internal user."),
        position: 'bottom',
        run: "click",
    }, {
        trigger: '.o_back_button',
        content: Markup(_t("By using the <b>Breadcrumb</b>, you can navigate back to the overview.")),
        position: 'bottom',
        run: 'click',
    }]
    );
});

```

## File: static\src\snippets\s_masonry_block\options.js

```javascript
odoo.define('mass_mailing.masonryOptions', function (require) {
'use strict';

const options = require('web_editor.snippets.options');

options.registry.MasonryLayout = options.registry.SelectTemplate.extend({
    /**
     * @constructor
     */
    init() {
        this._super(...arguments);
        this.containerSelector = '> .container, > .container-fluid, > .o_container_small';
        this.selectTemplateWidgetName = 'masonry_template_opt';
    },
});
});

```

## File: static\src\snippets\s_media_list\options.js

```javascript
odoo.define('mass_mailing.s_media_list_options', function (require) {
'use strict';

const options = require('web_editor.snippets.options');

options.registry.MediaItemLayout = options.Class.extend({

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Change the media item layout.
     *
     * @see this.selectClass for parameters
     */
    layout: function (previewMode, widgetValue, params) {
        const $image = this.$target.find('.s_media_list_img_wrapper');
        const $content = this.$target.find('.s_media_list_body');

        for (const possibleValue of params.possibleValues) {
            $image.removeClass(`col-lg-${possibleValue}`);
            $content.removeClass(`col-lg-${12 - possibleValue}`);
        }
        $image.addClass(`col-lg-${widgetValue}`);
        $content.addClass(`col-lg-${12 - widgetValue}`);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState(methodName, params) {
        switch (methodName) {
            case 'layout': {
                const $image = this.$target.find('.s_media_list_img_wrapper');
                for (const possibleValue of params.possibleValues) {
                    if ($image.hasClass(`col-lg-${possibleValue}`)) {
                        return possibleValue;
                    }
                }
            }
        }
        return this._super(...arguments);
    },
});
});

```

## File: static\src\snippets\s_rating\options.js

```javascript
odoo.define('mass_mailing.s_rating_options', function (require) {
'use strict';

const { ComponentWrapper } = require('web.OwlCompatibility');
const { MediaDialogWrapper } = require('@web_editor/components/media_dialog/media_dialog');
const options = require('web_editor.snippets.options');

options.registry.Rating = options.Class.extend({
    /**
     * @override
     */
    start: function () {
        this.iconType = this.$target[0].dataset.icon;
        this.faClassActiveCustomIcons = this.$target[0].dataset.activeCustomIcon || '';
        this.faClassInactiveCustomIcons = this.$target[0].dataset.inactiveCustomIcon || '';
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Displays the selected icon type.
     *
     * @see this.selectClass for parameters
     */
    setIcons: function (previewMode, widgetValue, params) {
        this.iconType = widgetValue;
        this._renderIcons();
        this.$target[0].dataset.icon = widgetValue;
        delete this.$target[0].dataset.activeCustomIcon;
        delete this.$target[0].dataset.inactiveCustomIcon;
    },
    /**
     * Allows to select a font awesome icon with media dialog.
     *
     * @see this.selectClass for parameters
     */
    customIcon: async function (previewMode, widgetValue, params) {
        const media = document.createElement('i');
        media.className = params.customActiveIcon === 'true' ? this.faClassActiveCustomIcons : this.faClassInactiveCustomIcons;
        const dialog = new ComponentWrapper(this, MediaDialogWrapper, {
            noImages: true,
            noDocuments: true,
            noVideos: true,
            media,
            save: icon => {
                const customClass = icon.className;
                const $activeIcons = this.$target.find('.s_rating_active_icons > i');
                const $inactiveIcons = this.$target.find('.s_rating_inactive_icons > i');
                const $icons = params.customActiveIcon === 'true' ? $activeIcons : $inactiveIcons;
                $icons.removeClass().addClass(customClass);
                this.faClassActiveCustomIcons = $activeIcons.length > 0 ? $activeIcons.attr('class') : customClass;
                this.faClassInactiveCustomIcons = $inactiveIcons.length > 0 ? $inactiveIcons.attr('class') : customClass;
                this.$target[0].dataset.activeCustomIcon = this.faClassActiveCustomIcons;
                this.$target[0].dataset.inactiveCustomIcon = this.faClassInactiveCustomIcons;
                this.$target[0].dataset.icon = 'custom';
                this.iconType = 'custom';
            }
        });
        dialog.mount(document.body);
    },
    /**
     * Sets the number of active icons.
     *
     * @see this.selectClass for parameters
     */
    activeIconsNumber: function (previewMode, widgetValue, params) {
        this.nbActiveIcons = parseInt(widgetValue);
        this._createIcons();
    },
    /**
     * Sets the total number of icons.
     *
     * @see this.selectClass for parameters
     */
    totalIconsNumber: function (previewMode, widgetValue, params) {
        this.nbTotalIcons = Math.max(parseInt(widgetValue), 1);
        this._createIcons();
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState: function (methodName, params) {
        switch (methodName) {
            case 'setIcons': {
                return this.$target[0].dataset.icon;
            }
            case 'activeIconsNumber': {
                this.nbActiveIcons = this.$target.find('.s_rating_active_icons > i').length;
                return this.nbActiveIcons;
            }
            case 'totalIconsNumber': {
                this.nbTotalIcons = this.$target.find('.s_rating_icons i').length;
                return this.nbTotalIcons;
            }
        }
        return this._super(...arguments);
    },
    /**
     * Creates the icons.
     *
     * @private
     */
    _createIcons: function () {
        const $activeIcons = this.$target.find('.s_rating_active_icons');
        const $inactiveIcons = this.$target.find('.s_rating_inactive_icons');
        this.$target.find('.s_rating_icons i').remove();
        for (let i = 0; i < this.nbTotalIcons; i++) {
            if (i < this.nbActiveIcons) {
                $activeIcons.append('<i/> ');
            } else {
                $inactiveIcons.append('<i/> ');
            }
        }
        this._renderIcons();
    },
    /**
     * Renders icons with selected fonts.
     *
     * @private
     */
    _renderIcons: function () {
        const icons = {
            'fa-star': 'fa-star-o',
            'fa-thumbs-up': 'fa-thumbs-o-up',
            'fa-circle': 'fa-circle-o',
            'fa-square': 'fa-square-o',
            'fa-heart': 'fa-heart-o'
        };
        const faClassActiveIcons = (this.iconType === "custom") ? this.faClassActiveCustomIcons : 'fa ' + this.iconType;
        const faClassInactiveIcons = (this.iconType === "custom") ? this.faClassInactiveCustomIcons : 'fa ' + icons[this.iconType];
        const $activeIcons = this.$target.find('.s_rating_active_icons > i');
        const $inactiveIcons = this.$target.find('.s_rating_inactive_icons > i');
        $activeIcons.removeClass().addClass(faClassActiveIcons);
        $inactiveIcons.removeClass().addClass(faClassInactiveIcons);
    },
});
});

```

## File: static\src\snippets\s_showcase\options.js

```javascript
odoo.define('mass_mailing.s_showcase_options', function (require) {
'use strict';

const options = require('web_editor.snippets.options');

options.registry.Showcase = options.Class.extend({
    /**
     * @override
     */
    onMove: function () {
        const $showcaseCol = this.$target.parent().closest('.row > div');
        const isLeftCol = $showcaseCol.index() <= 0;
        const $title = this.$target.children('.s_showcase_title');
        $title.toggleClass('flex-lg-row-reverse', isLeftCol);
        $showcaseCol.find('.s_showcase_icon.ms-3').removeClass('ms-3').addClass('ms-lg-3'); // For compatibility with old version
        $title.find('.s_showcase_icon').toggleClass('me-lg-0 ms-lg-3', isLeftCol);
    },
});
});

```

## File: static\src\views\mailing_contact_view_kanban.js

```javascript
/** @odoo-module **/

import { KanbanController } from "@web/views/kanban/kanban_controller";
import { kanbanView } from '@web/views/kanban/kanban_view';
import { registry } from '@web/core/registry';
import { useService } from "@web/core/utils/hooks";

export class MailingContactController extends KanbanController {
    async setup() {
        super.setup();
        this.actionService = useService("action");
    }

    onImport() {
        const context = this.props.context;
        const actionParams = { additionalContext: context };
        if (!context.default_mailing_list_ids && context.active_model === 'mailing.list' && context.active_ids) {
            actionParams.additionalContext.default_mailing_list_ids = context.active_ids;
        }
        this.actionService.doAction('mass_mailing.mailing_contact_import_action', actionParams);
    }
};

registry.category('views').add('mailing_contact_kanban', {
    ...kanbanView,
    Controller: MailingContactController,
    buttonTemplate: 'MailingContactKanbanView.buttons',
}); 

```

## File: static\src\views\mailing_contact_view_list.js

```javascript
/** @odoo-module **/

import { ListController } from "@web/views/list/list_controller";
import { listView } from '@web/views/list/list_view';
import { registry } from '@web/core/registry';
import { useService } from "@web/core/utils/hooks";

/**
 * List view for the <mailing.contact> model.
 *
 * Add an import button to open the wizard <mailing.contact.import>. This wizard
 * allows the user to import contacts line by line.
 */
export class MailingContactListController extends ListController {
    async setup() {
        super.setup();
        this.actionService = useService("action");
    }

    onImport() {
        const context = this.props.context;
        const actionParams = { additionalContext: context };
        if (!context.default_mailing_list_ids && context.active_model === 'mailing.list' && context.active_ids) {
            actionParams.additionalContext.default_mailing_list_ids = context.active_ids;
        }
        this.actionService.doAction('mass_mailing.mailing_contact_import_action', actionParams);
    }
};

registry.category('views').add('mailing_contact_list', {
    ...listView,
    Controller: MailingContactListController,
    buttonTemplate: 'MailingContactListView.buttons',
}); 

```

## File: static\src\views\mass_mailing_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="mailing_contact_views" xml:space="preserve">
    <t t-name="MailingContactListView.buttons" t-inherit="web.ListView.Buttons" t-inherit-mode="primary" owl="1">
        <xpath expr="//t[@t-if='props.showButtons']" position="inside">
            <button type="button" class="btn btn-secondary o_mass_mailing_import_contact" t-on-click="onImport">
                Import
            </button>
        </xpath>
    </t>

    <t t-name="MailingContactKanbanView.buttons" t-inherit="web.KanbanView.Buttons" t-inherit-mode="primary" owl="1">
        <xpath expr="//div[@t-if='props.showButtons']" position="inside">
            <button type="button" class="btn btn-secondary o_mass_mailing_import_contact" t-on-click="onImport">
                Import
            </button>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\mailing_filter_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <!-- Template for 'mailing_filter' widget, specific for mailing form view -->
    <t t-name="mass_mailing.MailingFilter" t-inherit="web.Many2OneField" primary="True" owl="1">
        <xpath expr="//div[hasclass('o_field_many2one_selection')]" position="inside">
            <div class="o_mass_mailing_filter_container">
                <div t-attf-class="o_mass_mailing_save_filter_container pt-0 pt-sm-1 {{ !this.filter.canSaveFilter ? 'd-none': '' }}">
                    <MailingFilterDropdown class="'o_mass_mailing_filter_dropdown'" togglerClass="'btn py-0 ps-0 ps-sm-3'">
                        <t t-set-slot="toggler">
                            <span class="o_mass_mailing_add_filter">
                                <span t-attf-class="o_mass_mailing_no_filter {{ this.props.record.data.mailing_filter_count ? 'd-none' : '' }}">Save as Favorite Filter</span>
                                <i class="fa fa-floppy-o px-1 py-1"/>
                            </span>
                        </t>
                        <div class="ms-2 fw-light">
                            Add to favorite filters
                        </div>
                        <div class="d-inline-flex">
                            <input t-ref="autofocus" type="text" class="o_mass_mailing_filter_name w-auto ms-2"
                                placeholder='e.g. "VIP Customers"' t-on-keydown="onFilterNameInputKeydown"/>
                            <button class="btn btn-sm btn-primary o_mass_mailing_btn_save_filter mx-2" t-on-click="onSaveFilter">
                                Add
                            </button>
                        </div>
                    </MailingFilterDropdown>
                </div>
                <a t-attf-class="o_mass_mailing_remove_filter btn px-1 pt-1 pb-0 {{ !this.filter.canRemoveFilter ? 'd-none': '' }}" t-on-click="onRemoveFilter">
                    <i class="fa fa-trash px-3 py-1 align-bottom" title="Remove from Favorites"/>
                </a>
            </div>
        </xpath>
    </t>

</templates>

```

## File: static\src\xml\mass_mailing.editor.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="mass_mailing.web_editor_toolbar" t-extend="web_editor.toolbar">
        <t t-jquery="div.btn-group.dropdown" t-operation="attributes">
            <attribute name="class" value="btn-group dropup"/>
        </t>
        <t t-jquery="div.dropdown:not(.btn-group)" t-operation="attributes">
            <attribute name="class" value="dropup"/>
        </t>
    </t>
</templates>

```

## File: static\src\xml\mass_mailing.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <div t-name="mass_mailing.theme_selector" class="o_mail_theme_selector">
        <t t-foreach="themes" t-as="theme">
            <a t-att-id="theme.name" href="#" class="dropdown-item">
                <div class="o_thumb logo" t-attf-style="background-image: url(#{theme.img}_logo.png)"/>
            </a>
        </t>
    </div>
    <div t-name="mass_mailing.theme_selector_new" class="o_mail_theme_selector_new">
        <div t-attf-class="o_mailing_template_message text-white text-start mx-4 pt-2 px-1 #{templates.length ? 'd-none': ''}">
            Click on the ⭐ next to the subject to save this mailing as a <span t-esc="modelName"/> template
        </div>
        <div class="o_mailing_template_preview_wrapper d-inline-block w-100">
            <div t-foreach="templates" t-as="template"
                class="o_mail_template_preview d-inline-block dropdown-item"
                t-att-id="template.name" t-att-model-id="template.modelId">
                <div class="d-inline-flex flex-row align-items-center text-white border px-2 py-2 w-100">
                    <i class="fa fa-star text-warning me-2"/>
                    <span class="text-truncate" t-esc="template.subject"/>
                    <div class="ms-auto">
                        <i class="o_mail_template_remove_favorite fa fa-trash ps-2 me-1" t-att-data-id="template.id" title="Remove from Templates"/>
                        <img t-if="template.userId" t-attf-src="/web/image/res.users/#{template.userId}/avatar_128" t-att-title="template.userName"/>
                    </div>
                </div>
            </div>
        </div>
        <div class="d-inline-block w-100">
            <a t-foreach="themes" t-as="theme"
                t-att-id="theme.name" role="menuitem" href="#" class="dropdown-item px-4">
                <div class="o_thumb small mb-2"  t-attf-style="background-image: url(#{theme.img}_small.png)"/>
                <div class="o_thumb large mb-2" t-attf-style="background-image: url(#{theme.img}_large.png)"/>
                <h5 class="text-white text-capitalize text-truncate" t-esc="theme.title"/>
            </a>
        </div>
    </div>
</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="iframe_css_assets_edit" groups="base.group_user">
        <t t-call-assets="mass_mailing.assets_mail_themes" t-js="false"/>
        <t t-call="mass_mailing.mass_mailing_mail_style"/>
        <t t-call-assets="mass_mailing.assets_mail_themes_edition" t-js="false"/>
        <!-- To view the body_arch field in readonly and have it display exactly
        like in edit, load all the same css. TODO: move all this and the above
        to a separate asset -->
        <t t-call-assets="web.assets_frontend" t-js="false"/>
        <t t-call-assets="web_editor.assets_wysiwyg" t-js="false"/>
    </template>

    <template id="iframe_css_assets_readonly" groups="base.group_user">
        <link rel="stylesheet" type="text/scss" href="/mass_mailing/static/src/css/basic_theme_readonly.css"/>
        <t t-call="mass_mailing.mass_mailing_mail_style"/>
    </template>
</odoo>

```

## File: views\link_tracker_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
	<!-- LINK.TRACKER VIEWS -->
    <record id="link_tracker_view_search" model="ir.ui.view">
        <field name="name">link.tracker.view.search.inherit.mass.mail</field>
        <field name="model">link.tracker</field>
        <field name="inherit_id" ref="link_tracker.link_tracker_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='title']" position="after">
                <field name="mass_mailing_id"/>
            </xpath>
            <xpath expr="//group" position="inside">
                <filter string="Mass Mailing" name="groupby_mass_mailing_id" context="{'group_by': 'mass_mailing_id'}"/>
            </xpath>
        </field>
    </record>

    <record id="link_tracker_view_form" model="ir.ui.view">
        <field name="name">link.tracker.view.form.inherit.mass.mail</field>
        <field name="model">link.tracker</field>
        <field name="inherit_id" ref="link_tracker.link_tracker_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='campaign_id']" position="before">
                <field name="mass_mailing_id"/>
            </xpath>
        </field>
    </record>

    <!-- LINK.TRACKER.CLICK VIEWS -->
    <record id="link_tracker_click_view_search" model="ir.ui.view">
        <field name="name">link.tracker.click.view.search.inherit.mass_mailing</field>
            <field name="model">link.tracker.click</field>
        <field name="inherit_id" ref="link_tracker.link_tracker_click_view_search"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='country_id']" position="after">
                <field name="campaign_id"/>
                <field name="mass_mailing_id"/>
            </xpath>
            <xpath expr="//filter[@name='groupby_country_id']" position="after">
                <filter string="Mass Mailing" name="groupby_mass_mailing_id" context="{'group_by': 'mass_mailing_id'}"/>
            </xpath>
        </field>
    </record>

    <record id="link_tracker_click_view_form" model="ir.ui.view">
        <field name="name">link.tracker.click.view.form.inherit.mass_mailing</field>
        <field name="model">link.tracker.click</field>
        <field name="inherit_id" ref="link_tracker.link_tracker_click_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='country_id']" position="after">
                <field name="campaign_id"/>
                <field name="mass_mailing_id"/>
                <field name="mailing_trace_id"/>
            </xpath>
        </field>
    </record>

    <record id="link_tracker_click_view_tree" model="ir.ui.view">
        <field name="name">link.tracker.click.view.tree.inherit.mass_mailing</field>
        <field name="model">link.tracker.click</field>
        <field name="inherit_id" ref="link_tracker.link_tracker_click_view_tree"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='link_id']" position="after">
                <field name="campaign_id"/>
                <field name="mass_mailing_id"/>
            </xpath>
        </field>
    </record>

    <record id="link_tracker_click_view_graph" model="ir.ui.view">
        <field name="name">link.tracker.click.view.graph.inherit.mass_mailing</field>
        <field name="model">link.tracker.click</field>
        <field name="inherit_id" ref="link_tracker.link_tracker_click_view_graph"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='country_id']" position="after">
                <field name="campaign_id"/>
                <field name="mass_mailing_id"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\mailing_contact_subscription_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="mailing_contact_subscription_view_form">
        <field name="name">mailing.contact.subscription.view.form</field>
        <field name="model">mailing.contact.subscription</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <form string="Mailing List Subscription">
                <sheet>
                    <group>
                        <field name="list_id"/>
                        <field name="is_blacklisted" invisible="1"/>
                        <label for="contact_id" class="oe_inline"/>
                        <div class="o_row o_row_readonly">
                            <i class="fa fa-ban text-danger" role="img" title="This email is blacklisted for mass mailings"
                                aria-label="Blacklisted" attrs="{'invisible': [('is_blacklisted', '=', False)]}" groups="base.group_user"></i>
                            <field name="contact_id"/>
                        </div>
                        <field name="unsubscription_date" readonly="1"/>
                        <field name="opt_out"/>
                        <field name="message_bounce" readonly="1"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record model="ir.ui.view" id="mailing_contact_subscription_view_tree">
        <field name="name">mailing.contact.subscription.view.tree</field>
        <field name="model">mailing.contact.subscription</field>
        <field name="arch" type="xml">
            <tree string="Mailing List Subscriptions">
                <field name="contact_id"/>
                <field name="unsubscription_date"/>
                <field name="opt_out"/>
                <field name="message_bounce"/>
                <field name="is_blacklisted"/>
            </tree>
        </field>
    </record>

    <record model="ir.ui.view" id="mailing_contact_subscription_view_search">
        <field name="name">mailing.contact.subscription.view.search</field>
        <field name="model">mailing.contact.subscription</field>
        <field name="arch" type="xml">
           <search string="Mailing List Subscriptions">
                <field name="contact_id"/>
                <field name="opt_out"/>
                <field name="list_id"/>
            </search>
        </field>
    </record>
</odoo>

```

## File: views\mailing_contact_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
     <record id="mailing_contact_view_search" model="ir.ui.view">
        <field name="name">mailing.contact.view.search</field>
        <field name="model">mailing.contact</field>
        <field name="arch" type="xml">
           <search string="Mailing List Contacts">
                <field name="name"
                    filter_domain="['|', '|', ('name','ilike',self), ('company_name','ilike',self), ('email_normalized','ilike',self)]"
                    string="Name / Email"/>
                <field name="tag_ids"/>
                <field name="list_ids"/>
                <separator/>
                <filter string="Valid Email Recipients"
                    name="filter_valid_email_recipient"
                    domain="[('opt_out', '=', False), ('is_blacklisted', '=', False), ('email_normalized', '!=', False)]"
                    invisible="not context.get('default_list_ids')"/>
                <separator/>
                <filter name="filter_bounce" string="Bounced" domain="[('message_bounce', '>', 0)]"/>
                <filter name="filter_blacklisted" string="Blacklisted" domain="[('is_blacklisted','=',True)]" invisible="1"/>
                <filter name="filter_opt_out" string="Opted-out" domain="[('opt_out', '=', True)]" invisible="1"/>
                <separator/>
                <filter string="Exclude Blacklisted Emails"
                    name="filter_not_email_bl"
                    domain="[('is_blacklisted', '=', False)]"/>
                <separator/>
                <filter string="Exclude Opt Out"
                    name="filter_not_optout"
                    domain="[('opt_out', '=', False)]"
                    invisible="not context.get('default_list_ids')"/>
                <group expand="0" string="Group By">
                    <filter string="Creation Date" name="group_create_date"
                        context="{'group_by': 'create_date'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="mailing_contact_view_tree" model="ir.ui.view">
        <field name="name">mailing.contact.view.tree</field>
        <field name="model">mailing.contact</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <tree string="Mailing List Contacts" sample="1" multi_edit="1" js_class="mailing_contact_list">
                <header>
                    <button name="action_add_to_mailing_list" string="Add to List" type="object"/>
                </header>
                <field name="create_date" optional="show"/>
                <field name="title_id" optional="hide"/>
                <field name="name" readonly="1"/>
                <field name="company_name"/>
                <field name="email" readonly="1"/>
                <field name="is_blacklisted" string="Email Blacklisted"/>
                <field name="country_id" optional="hide"/>
                <field name="message_bounce" sum="Total Bounces" readonly="1"/>
                <field name="opt_out" invisible="'default_list_ids' not in context" readonly="1"/>
                <field name="list_ids" widget="many2many_tags" optional="hide"/>
            </tree>
        </field>
    </record>

    <record id="mailing_contact_view_kanban" model="ir.ui.view">
        <field name="name">mailing.contact.view.kanban</field>
        <field name="model">mailing.contact</field>
        <field name="arch" type="xml">
            <kanban sample="1" js_class="mailing_contact_kanban">
                <field name="name"/>
                <field name="company_name"/>
                <field name="email"/>
                <field name="message_bounce"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div class="o_kanban_record_top">
                                <div class="o_kanban_record_headings">
                                    <strong class="o_kanban_record_title">
                                        <t t-esc="record.name.value"/>
                                    </strong>
                                </div>
                                <span class="badge rounded-pill" title="Number of bounced email.">
                                    <i class="fa fa-exclamation-triangle" role="img" aria-label="Warning" title="Warning"/> <t t-esc="record.message_bounce.value" title=""/>
                                </span>
                            </div>
                            <div class="o_kanban_record_body">
                                <field name="tag_ids"/>
                            </div>
                            <div class="o_kanban_record_bottom">
                                <div class="oe_kanban_bottom_left">
                                    <strong>
                                        <t t-esc="record.email.value"/>
                                    </strong>
                                </div>
                                <div class="oe_kanban_bottom_right">
                                    <t t-esc="record.company_name.value"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="mailing_contact_view_form" model="ir.ui.view">
        <field name="name">mailing.contact.view.form</field>
        <field name="model">mailing.contact</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <form string="Mailing List Contacts">
                <field name="id" invisible="1"/>
                <sheet>
                    <div class="oe_title">
                        <label for="name" string="Contact Name"/>
                        <h1>
                            <field class="text-break" name="name" placeholder="e.g. John Smith"/>
                        </h1>
                        <div>
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" placeholder="Tags" style="width: 100%"/>
                        </div>
                    </div>
                    <group>
                        <group>
                            <label for="email" class="oe_inline"/>
                            <div class="o_row o_row_readonly" name="email_details">
                                 <button name="mail_action_blacklist_remove" class="fa fa-ban text-danger"
                                    title="This email is blacklisted for mass mailings. Click to unblacklist."
                                    type="object" context="{'default_email': email}" groups="base.group_user"
                                    attrs="{'invisible': [('is_blacklisted', '=', False)]}"/>
                                <field name="email" widget="email"/>
                                <field name="is_blacklisted" invisible="1"/>
                            </div>
                            <field name="title_id"/>
                            <field name="company_name"/>
                            <field name="country_id" options="{'no_open': True, 'no_create': True}"/>
                        </group>
                        <group>
                            <field name="create_date" attrs="{'invisible': [('id', '=', False)]}" readonly="1"/>
                            <field name="message_bounce" attrs="{'invisible': [('id', '=', False)]}" readonly="1"/>
                        </group>
                    </group>
                    <field name="subscription_list_ids">
                        <tree editable="bottom">
                           <field name="list_id"/>
                           <field name="unsubscription_date"/>
                           <field name="opt_out"/>
                        </tree>
                    </field>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="mailing_contact_view_pivot" model="ir.ui.view">
        <field name="name">mailing.contact.pivot</field>
        <field name="model">mailing.contact</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <pivot string="Mailing List Contacts" stacked="1" sample="1">
                <field name="create_date" type="row"/>
            </pivot>
        </field>
    </record>

    <record id="mailing_contact_view_graph" model="ir.ui.view">
        <field name="name">mailing.contact.view.graph</field>
        <field name="model">mailing.contact</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <graph string="Mailing List Contacts" sample="1">
                <field name="create_date"/>
            </graph>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_view_mass_mailing_contacts">
        <field name="name">Mailing List Contacts</field>
        <field name="res_model">mailing.contact</field>
        <field name="view_mode">tree,kanban,form,graph,pivot</field>
        <field name="context">{'search_default_filter_not_email_bl': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create a mailing contact
            </p><p>
                Mailing contacts allow you to separate your marketing audience from your business contact directory.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\mailing_filter_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mailing_filter_view_search" model="ir.ui.view">
        <field name="name">mailing.filter.view.search</field>
        <field name="model">mailing.filter</field>
        <field name="arch" type="xml">
           <search string="Mailing Filters">
                <field name="name"/>
                <field name="mailing_model_id"/>
                <filter string="My Filters"
                        name="filter_saved_by_me"
                        domain="[('create_uid', '=', uid)]"
                        help="Filters saved by me"/>
                <group string="Group By">
                    <filter name="groupby_recepient_model"
                            context="{'group_by' : 'mailing_model_id'}"
                            string="Recipients"/>
                </group>
            </search>
        </field>
    </record>

    <record id="mailing_filter_view_tree" model="ir.ui.view">
        <field name="name">mailing.filter.view.tree</field>
        <field name="model">mailing.filter</field>
        <field name="arch" type="xml">
            <tree string="Mailing filters" sample="1">
                <field name="name"/>
                <field name="create_uid" string="Saved by" widget="many2one_avatar_user"/>
                <field name="mailing_model_id" string="Recipients"/>
                <field name="mailing_domain" string="Domain" optional="hide"/>
            </tree>
        </field>
    </record>

    <record id="mailing_filter_view_form" model="ir.ui.view">
        <field name="name">mailing.filter.view.form</field>
        <field name="model">mailing.filter</field>
        <field name="arch" type="xml">
            <form string="Mailing filters">
                <sheet>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="mailing_model_id" options="{'no_create': True, 'no_open': True}"/>
                        </group>
                        <group>
                            <field name="create_uid" widget="many2one_avatar_user"/>
                        </group>
                    </group>
                    <group>
                        <field name="mailing_model_name" invisible="1"/>
                        <field name="mailing_domain" string="Domain" widget="domain"
                               options="{'model': 'mailing_model_name'}"
                               attrs="{'invisible': [('mailing_model_id', '=', False)]}"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="mailing_filter_action" model="ir.actions.act_window">
        <field name="name">Favorite Filters</field>
        <field name="res_model">mailing.filter</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'search_default_filter_saved_by_me': 1}
        </field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No saved filter yet!
            </p><p>
                While designing the mailing, you can define the rules to filter recipients.
                To save the same criteria for future use, you can add it to the favorite list
                by clicking on <i class="fa fa-star-o text-warning"></i> icon next to "Recipients".
            </p>
        </field>
    </record>
</odoo>

```

## File: views\mailing_list_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--  MAILING LIST -->
    <record model="ir.ui.view" id="mailing_list_view_search">
        <field name="name">mailing.list.view.search</field>
        <field name="model">mailing.list</field>
        <field name="arch" type="xml">
            <search string="Mailing Lists">
                <field name="name"/>
                <field name="create_date"/>
                <filter name="inactive" string="Archived" domain="[('active','=',False)]"/>
                <group expand="0" string="Group By">
                    <filter string="Creation Period" name="group_create_date"
                        context="{'group_by': 'create_date'}"/>
                </group>
            </search>
        </field>
    </record>

    <record model="ir.ui.view" id="mailing_list_view_tree">
        <field name="name">mailing.list.view.tree</field>
        <field name="model">mailing.list</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <tree string="Mailing Lists" sample="1">
                <field name="name"/>
                <field name="is_public"/>
                <field name="mailing_count" string="Mailings"/>
                <field name="contact_pct_bounce" string="Bounce (%)"/>
                <field name="contact_pct_opt_out" string="Opt-out (%)"/>
                <field name="contact_pct_blacklisted" string="Blacklist (%s)"/>
                <field name="contact_count" string="Recipients"/>
            </tree>
        </field>
    </record>

    <record model="ir.ui.view" id="mailing_list_view_form">
        <field name="name">mailing.list.form</field>
        <field name="model">mailing.list</field>
        <field name="arch" type="xml">
            <form string="Contact List">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_view_contacts"
                                type="object" icon="fa-user" class="oe_stat_button">
                            <field name="contact_count" string="Recipients" widget="statinfo"/>
                        </button>
                        <button name="action_view_mailings"
                                type="object" icon="fa-envelope-o" class="oe_stat_button">
                            <field name="mailing_count" string="Mailings" widget="statinfo"/>
                        </button>
                        <button name="action_view_contacts_bouncing"
                                type="object" icon="fa-exchange" class="oe_stat_button">
                            <div class="o_field_widget o_stat_info">
                                <div class="oe_inline">
                                    <span class="o_stat_value">
                                        <field name="contact_pct_bounce" widget="statinfo" nolabel="1"/>
                                    </span>
                                    <span class="o_stat_value">%</span>
                                </div>
                                <span class="o_stat_text">Bounce</span>
                            </div>
                        </button>
                        <button name="action_view_contacts_opt_out"
                                type="object" icon="fa-bell-slash-o" class="oe_stat_button">
                            <div class="o_field_widget o_stat_info">
                                <div class="oe_inline">
                                    <span class="o_stat_value">
                                        <field name="contact_pct_opt_out" widget="statinfo" nolabel="1"/>
                                    </span>
                                    <span class="o_stat_value">%</span>
                                </div>
                                <span class="o_stat_text">Opt-out</span>
                            </div>
                        </button>
                        <button name="action_view_contacts_blacklisted"
                                type="object" icon="fa-ban" class="oe_stat_button">
                            <div class="o_field_widget o_stat_info">
                                <div class="oe_inline">
                                    <span class="o_stat_value">
                                        <field name="contact_pct_blacklisted" widget="statinfo" nolabel="1"/>
                                    </span>
                                    <span class="o_stat_value">%</span>
                                </div>
                                <span class="o_stat_text">Blacklist</span>
                            </div>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <label for="name"/>
                        <h1>
                            <field name="name" class="text-break" placeholder="e.g. Consumer Newsletter"/>
                        </h1>
                    </div>
                    <group>
                        <field name="active" invisible="1"/>
                        <field name="is_public"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="mailing_list_view_form_simplified" model="ir.ui.view">
        <field name="name">mailing.list.form.simplified</field>
        <field name="model">mailing.list</field>
        <field name="priority" eval="25"/>
        <field name="arch" type="xml">
            <form string="Contact List">
                <div class="oe_title">
                    <label for="name"/>
                    <h1>
                        <field name="name" placeholder="e.g. Consumer Newsletter"/>
                    </h1>
                </div>
                <group>
                    <field name="is_public"/>
                </group>
            </form>
        </field>
    </record>

    <record id="mailing_list_view_kanban" model="ir.ui.view">
        <field name="name">mailing.list.view.kanban</field>
        <field name="model">mailing.list</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile o_kanban_mailing_list" sample="1">
                <field name="name"/>
                <field name="contact_count"/>
                <field name="contact_count_email"/>
                <field name="contact_count_opt_out"/>
                <field name="contact_count_blacklisted"/>
                <field name="contact_pct_bounce"/>
                <field name="contact_pct_opt_out"/>
                <field name="contact_pct_blacklisted"/>
                <field name="active"/>
                <templates>
                    <t t-name="kanban-box">
                        <!-- Card when the Kanban view is grouped  -->
                        <div class="oe_kanban_global_click o_mailing_list_kanban_grouped">
                            <div class="oe_kanban_content d-flex flex-column h-100">
                                <h2 class="mb-3 o_text_overflow">
                                    <field name="name"/>
                                </h2>
                                <div class="d-flex align-items-center">
                                    <div class="me-3">
                                        <button class="btn btn-primary" name="action_view_contacts" type="object">
                                            <t t-esc="record.contact_count.value"/> <span>Contacts</span>
                                        </button>
                                    </div>
                                    <div class="flex-grow-1 d-flex flex-column align-items-end o_mass_mailing_kanban_contact_links">
                                        <a name="action_view_contacts_email" type="object">
                                            <span>Valid Email Recipients</span>
                                            <span t-esc="record.contact_count_email.value" class="ms-3"/>
                                        </a>
                                    </div>
                                </div>
                                <div class="flex-grow-1 d-flex align-items-end mt-4">
                                    <div class="col-12">
                                        <div class="row mt3">
                                            <div class="col-3 border-end">
                                                <a name="action_view_mailings" type="object" class="d-flex flex-column align-items-center">
                                                    <span><field name="mailing_count"/></span>
                                                    <span class="text-muted">Mailings</span>
                                                </a>
                                            </div>
                                            <div class="col-3 border-end">
                                                <a name="action_view_contacts_bouncing" type="object" class="d-flex flex-column align-items-center">
                                                    <span><field name="contact_pct_bounce"/>%</span>
                                                    <span class="text-muted">Bounce</span>
                                                </a>
                                            </div>
                                            <div class="col-3 border-end">
                                                <a name="action_view_contacts_opt_out" type="object" class="d-flex flex-column align-items-center">
                                                    <span><field name="contact_pct_opt_out"/>%</span>
                                                    <span class="text-muted">Opt-out</span>
                                                </a>
                                            </div>
                                            <div class="col-3">
                                                <a name="action_view_contacts_blacklisted" type="object" class="d-flex flex-column align-items-center">
                                                    <span><field name="contact_pct_blacklisted"/>%</span>
                                                    <span class="text-muted">Blacklist</span>
                                                </a>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <!-- Card when the Kanban view is not grouped -->
                        <div class="oe_kanban_global_click o_mailing_list_kanban_ungrouped d-flex flex-row justify-content-between align-items-center flex-wrap">
                            <div class="col-lg-3 col-sm-6 col-12 py-0 my-auto">
                                <div class="d-flex text-large">
                                    <span class="d-inline-block text-truncate fw-normal text-large"
                                          t-att-title="record.name.value">
                                        <field name="name"/>
                                    </span>
                                </div>
                            </div>
                            <div class="o_mailing_list_kanban_counts col-lg-1 col-12 my-auto d-flex">
                                <div class="d-flex flex-row justify-content-start justify-content-md-end">
                                    <div class="my-auto p-0">
                                        <button name="action_view_contacts" type="object"
                                            class="o_mailing_list_kanban_button o_mailing_list_kanban_big_nb text-primary fw-bold">
                                            <field name="contact_count"/>
                                        </button>
                                   </div>
                                    <div class="my-auto px-0">
                                        <button name="action_view_contacts" type="object"
                                            class="o_mailing_list_kanban_button text-large text-start ms-2">
                                            Total <br/>Contacts
                                        </button>
                                    </div>
                                </div>
                            </div>
                            <div class="o_mailing_list_kanban_stats col-lg-5 col-sm-12 col-12 py-0 my-3 my-sm-auto d-flex justify-content-sm-between justify-content-start flex-wrap">
                                <a class="me-sm-0 me-3 text-large" tabindex="-1"
                                    name="action_view_contacts_email" type="object">
                                    <span class="fw-normal">
                                        <field name="contact_count_email"/>
                                    </span>
                                    <br/>
                                    <span class="text-secondary">
                                        <i class="fa fa-envelope-o"/> Contacts
                                    </span>
                                </a>
                                <a class="me-sm-0 me-3 text-large" tabindex="-1"
                                    name="action_view_mailings" type="object">
                                    <span class="fw-normal">
                                        <field name="mailing_count"/>
                                    </span>
                                    <br/>
                                    <span class="text-secondary">Mailings</span>
                                </a>
                                <hr class="w-100 d-block d-sm-none opacity-0 m-0 p-0"/>
                                <a class="me-sm-0 me-3 text-large" tabindex="-1"
                                    name="action_view_contacts_bouncing" type="object">
                                    <span class="fw-normal">
                                        <field name="contact_pct_bounce"/> %
                                    </span>
                                    <br/>
                                    <span class="text-secondary">Bounce</span>
                                </a>
                                <a class="me-sm-0 me-3 text-large" tabindex="-1"
                                    name="action_view_contacts_opt_out" type="object">
                                    <span class="fw-normal">
                                        <field name="contact_pct_opt_out"/> %
                                    </span>
                                    <br/>
                                    <span class="text-secondary">Opt-Out</span>
                                </a>
                                <a class="me-sm-0 me-3 text-large" tabindex="-1"
                                    name="action_view_contacts_blacklisted" type="object">
                                    <span class="fw-normal">
                                        <field name="contact_pct_blacklisted"/> %
                                    </span>
                                    <br/>
                                    <span class="text-secondary">Blacklist</span>
                                </a>
                            </div>
                            <div class="o_kanban_ungrouped_action_buttons col-12 col-lg-2 py-0 pe-2 ps-md-2 my-md-auto my-2 d-none d-md-flex flex-wrap justify-content-lg-end">
                                <button name="action_open_import" string="Import Contacts"
                                    type="object" class="btn btn-secondary border me-2 text-nowrap">
                                    Import Contacts
                                </button>
                                <button name="action_send_mailing" string="Send Mailing"
                                    type="object" class="btn btn-secondary border me-2 text-nowrap">
                                    Send Mailing
                                </button>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record model="ir.actions.act_window" id="action_view_mass_mailing_lists">
        <field name="name">Mailing Lists</field>
        <field name="res_model">mailing.list</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a Mailing List
          </p><p>
            No need to import mailing lists, you can send mailings to contacts saved in other Odoo apps.
          </p>
        </field>
    </record>
</odoo>

```

## File: views\mailing_mailing_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!--  MAILING !-->
        <record model="ir.ui.view" id="view_mail_mass_mailing_search">
            <field name="name">mailing.mailing.search</field>
            <field name="model">mailing.mailing</field>
            <field name="arch" type="xml">
               <search string="Mailings">
                    <field name="name" string="Mailing" filter_domain="['|', ('name', 'ilike', self), ('subject', 'ilike', self)]"/>
                    <field name="campaign_id" string="Campaign" groups="mass_mailing.group_mass_mailing_campaign"/>
                    <filter string="My Mailings" name="assigned_to_me"
                            domain="[('user_id', '=', uid)]"
                            help="Mailings that are assigned to me"/>
                    <separator/>
                    <filter name="filter_sent_date" date="sent_date"/>
                    <separator/>
                    <filter string="A/B Tests" name="filter_ab_test" domain="[('ab_testing_enabled', '=', True)]"/>
                    <filter string="A/B Tests to review" name="filter_ab_test_to_review"
                        domain="[('ab_testing_enabled', '=', True), ('ab_testing_winner_selection', '=', 'manual'), ('ab_testing_completed', '=', False)]"/>
                    <separator/>
                    <filter name="inactive" string="Archived" domain="[('active', '=', False)]"/>
                    <group expand="0" string="Group By">
                        <filter string="Status" name="group_state" context="{'group_by': 'state'}"/>
                        <filter string="Sent By" name="sent_by" domain="[]" context="{'group_by': 'email_from'}"/>
                        <separator/>
                        <filter string="Sent Period" name="sent_date" domain="[]" context="{'group_by': 'sent_date'}"/>
                    </group>
                </search>
            </field>
        </record>

        <record model="ir.ui.view" id="view_mail_mass_mailing_tree">
            <field name="name">mailing.mailing.tree</field>
            <field name="model">mailing.mailing</field>
            <field name="priority">10</field>
            <field name="arch" type="xml">
                <tree string="Mailings" sample="1" class="o_mass_mailing_mailing_tree">
                    <field name="calendar_date" string="Date" widget="datetime"/>
                    <field name="subject" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                    <field name="mailing_model_id" string="Recipients" optional="hide"/>
                    <field name="user_id" widget="many2one_avatar_user"/>
                    <field name="ab_testing_enabled" string="A/B Test" groups="mass_mailing.group_mass_mailing_campaign"/>
                    <field name="campaign_id" string="Campaign" groups="mass_mailing.group_mass_mailing_campaign" optional="hide"/>
                    <field name="sent" sum="Total" />
                    <field name="received_ratio" class="d-flex align-items-center ps-0 ps-lg-5" widget="progressbar" string="Delivered (%)" avg="Average"/>
                    <field name="opened_ratio" class="d-flex align-items-center ps-0 ps-lg-5" widget="progressbar" string="Opened (%)" avg="Average"/>
                    <field name="bounced_ratio" string="Bounced (%)" optional="hide" avg="Average"/>
                    <field name="clicks_ratio" string="Clicked (%)" avg="Average"/>
                    <field name="replied_ratio" string="Replied (%)" avg="Average"/>
                    <field name="state" decoration-info="state in ['draft', 'in_queue']" decoration-success="state == 'sending' or state == 'done'" widget="badge"/>
                </tree>
            </field>
        </record>

        <!-- Main form view for inheriting from -->
        <record model="ir.ui.view" id="view_mail_mass_mailing_form">
            <field name="name">mailing.mailing.form</field>
            <field name="model">mailing.mailing</field>
            <field name="arch" type="xml">
                <form string="Mailing" class="o_mass_mailing_mailing_form">
                    <header style="min-height:31px;">
                        <button name="action_launch" type="object" class="oe_highlight" string="Send"
                            attrs="{'invisible': [('state', 'in', ('in_queue',  'sending', 'done'))]}" data-hotkey="v"
                            confirm="This will send the email to all recipients. Do you still want to proceed ?"/>
                        <button name="action_schedule" type="object" class="btn-secondary" string="Schedule"
                            attrs="{'invisible': [('state', 'in', ('in_queue',  'sending', 'done'))]}" data-hotkey="x"/>
                        <button name="action_duplicate" type="object" class="btn-secondary" string="Duplicate"
                            data-hotkey="d" attrs="{'invisible': [('state', '!=', 'done')]}"/>
                        <button name="action_test" type="object" class="btn-secondary" string="Test" data-hotkey="k"/>
                        <button name="action_cancel" type="object" attrs="{'invisible': [('state', '!=', 'in_queue')]}" class="btn-secondary" string="Cancel" data-hotkey="z"/>
                        <button name="action_retry_failed" type="object" attrs="{'invisible': ['|', ('state', '!=', 'done'), ('failed', '=', 0)]}" class="oe_highlight" string="Retry" data-hotkey="y"/>

                        <field name="state" readonly="1" widget="statusbar"/>
                    </header>
                    <div class="alert alert-info text-center" role="alert"
                        attrs="{'invisible': ['&amp;','&amp;','&amp;','&amp;','&amp;',('state', '!=', 'in_queue'),('sent', '=', 0),('canceled', '=', 0),('scheduled', '=', 0),('failed', '=', 0),('warning_message', '=', False)]}">
                        <div class="o_mails_canceled" attrs="{'invisible': [('canceled', '=', 0)]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_canceled"
                                    type="object">
                                <strong>
                                    <field name="canceled" class="oe_inline me-2"/>
                                    <span name="canceled_text">emails have been canceled and will not be sent.</span>
                                </strong>
                            </button>
                        </div>
                        <div class="o_mails_scheduled" attrs="{'invisible': [('scheduled', '=', 0)]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_scheduled"
                                    type="object">
                                <strong>
                                    <field name="scheduled" class="oe_inline me-2"/>
                                    <span name="scheduled_text">emails are in queue and will be sent soon.</span>
                                </strong>
                            </button>
                        </div>
                        <div class="o_mails_sent" attrs="{'invisible': ['&amp;', ('sent', '=', 0), ('state', 'in', ('draft', 'test', 'in_queue'))]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_sent"
                                    type="object">
                                <strong>
                                    <field name="sent" class="oe_inline me-2"/>
                                    <span name="sent">emails have been sent.</span>
                                </strong>
                            </button>
                        </div>
                        <div class="o_mails_failed" attrs="{'invisible': ['|', ('state', '!=', 'done'), ('failed', '=', 0)]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_failed"
                                    type="object">
                                <strong>
                                    <field name="failed" class="oe_inline me-2"/>
                                    <span name="failed_text">emails could not be sent.</span>
                                </strong>
                            </button>
                        </div>
                        <div class="o_mails_in_queue" attrs="{'invisible': [('state', '!=', 'in_queue')]}">
                            <strong>
                                <span name="next_departure_text">This mailing is scheduled for </span>
                                <field name="next_departure" class="oe_inline"/>.
                            </strong>
                        </div>
                        <div attrs="{'invisible': [('warning_message', '=', False)]}">
                            <strong><field name="warning_message"/></strong>
                        </div>
                    </div>

                    <sheet>
                        <div class="oe_button_box" name="button_box">
                            <button name="action_view_delivered"
                                id="button_view_delivered"
                                type="object"
                                context="{'search_default_filter_delivered': True}"
                                attrs="{'invisible': [('state', 'in', ('draft','test'))]}"
                                class="oe_stat_button">
                                <field name="received_ratio" string="Received" widget="percentpie"/>
                            </button>
                            <button name="action_view_opened"
                                type="object"
                                context="{'search_default_filter_opened': True}"
                                attrs="{'invisible': [('state', 'in', ('draft','test'))]}"
                                class="oe_stat_button">
                                <field name="opened_ratio" string="Opened" widget="percentpie"/>
                            </button>
                            <button name="action_view_clicked"
                                type="object"
                                context="{'search_default_filter_clicked': True}"
                                attrs="{'invisible': [('state', 'in', ('draft','test'))]}"
                                class="oe_stat_button">
                                <field name="clicks_ratio" string="Clicked" widget="percentpie"/>
                            </button>
                            <button name="action_view_replied"
                                type="object"
                                context="{'search_default_filter_replied': True}"
                                attrs="{'invisible': [('state', 'in', ('draft','test'))]}"
                                class="oe_stat_button">
                                <field name="replied_ratio" string="Replied" widget="percentpie"/>
                            </button>
                            <button name="action_view_bounced"
                                type="object"
                                context="{'search_default_filter_bounced': True}"
                                attrs="{'invisible': [('state', 'in', ('draft','test'))]}"
                                class="oe_stat_button">
                                <field name="bounced_ratio" string="Bounced" widget="percentpie"/>
                            </button>
                        </div>
                        <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                        <group class="o_mass_mailing_mailing_group">
                            <field name="active" invisible="1"/>
                            <field name="create_uid" invisible="1"/>
                            <field name="mailing_type" widget="radio" options="{'horizontal': true}" invisible="1"
                                attrs="{'readonly': [('state', '!=', 'draft')]}" force_save="1"/>
                            <label for="subject">Subject</label>
                            <div class="o_mass_mailing_subject d-flex flex-row align-items-baseline">
                                <field class="text-break" name="subject" string="Subject"
                                    options="{'dynamic_placeholder': true}"
                                    attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"
                                    widget="char_emojis" placeholder="e.g. New Sale on all T-shirts"/>
                                <field name="favorite" invisible="1"/>
                                <button type="object" name="action_set_favorite"
                                    class="o_mass_mailing_favorite p-0"
                                    icon="fa-star-o"
                                    attrs="{'invisible': [('favorite', '=', True)]}"
                                    title="Add to Templates"/>
                                <button type="object" name="action_remove_favorite"
                                    class="o_mass_mailing_favorite p-0"
                                    icon="fa-star"
                                    attrs="{'invisible': [('favorite', '=', False)]}"
                                    title="Remove from Templates"/>
                            </div>
                            <label for="mailing_model_id" string="Recipients"/>
                            <div name="mailing_model_id_container">
                                <div class="d-flex align-items-baseline flex-wrap">
                                    <div class="me-5">
                                        <field name="mailing_model_id" options="{'no_open': True, 'no_create': True}"
                                            attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                    </div>
                                    <div attrs="{'invisible': [('mailing_model_name', '!=', 'mailing.list')]}" class="o_mass_mailing_contact_list_ids pt-1 pt-sm-0">
                                        <label for="contact_list_ids" string="Select mailing lists:" class="oe_edit_only pe-2"/>
                                        <div class="d-inline-flex flex-row align-items-center">
                                            <field name="contact_list_ids" widget="many2many_tags"
                                                placeholder="Select mailing lists..." class="oe_inline mb-0"
                                                context="{'form_view_ref': 'mass_mailing.mailing_list_view_form_simplified'}"
                                                attrs="{
                                                    'required':[('mailing_model_name','=','mailing.list')],
                                                    'readonly': [('state', 'in', ('sending', 'done'))]
                                            }"/>
                                            <button icon="fa-user-plus" type="object" class="btn btn-secondary py-0 px-1 ms-1"
                                                attrs="{'invisible': ['|', '|', ('contact_list_ids', '=', False), ('contact_list_ids', '=', []), ('state', 'in', ('sending', 'done'))]}"
                                                name="action_view_mailing_contacts" title="Add Mailing Contacts"/>
                                        </div>
                                    </div>
                                    <div attrs="{'invisible': [('mailing_model_name', '=', 'mailing.list')]}" class="o_td_label">
                                        <!-- We don't want to display label in edit mode, unless mailing is in sending or done state (where filter will be readonly) -->
                                        <label for="mailing_filter_id" string="Filter" class="oe_read_only me-4"
                                               attrs="{'invisible': ['|', ('state', 'in', ('sending', 'done')), ('mailing_filter_id', '=', False)]}"/>
                                        <label for="mailing_filter_id" string="Filter" class="me-4"
                                               attrs="{'invisible': ['|', ('state', 'not in', ('sending', 'done')), ('mailing_filter_id', '=', False)]}"/>
                                        <field name="mailing_filter_id" placeholder="Reload a favorite filter"
                                               class="w-auto" widget="mailing_filter"
                                               options="{'no_create': 1, 'no_open': 1, 'domain_field': 'mailing_domain', 'model_field': 'mailing_model_id'}"
                                               attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                    </div>
                                    <field name="mailing_filter_count" invisible="1"/>
                                    <field name="mailing_filter_domain" invisible="1"/>
                                </div>

                                <field name="mailing_model_name" invisible="1"/>
                                <field name="mailing_model_real" invisible="1"/>
                                <div class="w-lg-50" attrs="{'invisible': [('mailing_model_name', '=', 'mailing.list')]}">
                                    <field name="mailing_domain" widget="domain" options="{'model': 'mailing_model_real'}"
                                    attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                </div>
                            </div>
                        </group>
                        <notebook>
                            <page string="Mail Body" name="mail_body">
                                <div class="position-relative">
                                    <div class="mt-n2">
                                        <field name="body_arch" class="o_mail_body" widget="mass_mailing_html"
                                            iframeHtmlClass="o_mass_mailing_iframe"
                                            options="{
                                                'snippets': 'mass_mailing.email_designer_snippets',
                                                'cssEdit': 'mass_mailing.iframe_css_assets_edit',
                                                'inline-field': 'body_html',
                                                'dynamic_placeholder': true,
                                                'cssReadonly': 'mass_mailing.iframe_css_assets_edit'
                                        }" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                    </div>
                                    <field name="is_body_empty" invisible="1"/>
                                    <div class="o_view_nocontent oe_read_only" attrs="{'invisible': ['|', ('is_body_empty', '=', False), ('state', 'in', ('sending', 'done'))]}">
                                        <div class="o_nocontent_help">
                                            <p class="o_view_nocontent_smiling_face">
                                                This mailing has no selected design (yet!).
                                            </p>
                                        </div>
                                    </div>
                                </div>
                            </page>
                            <page string="A/B Tests" name="ab_testing">
                                <group>
                                    <group>
                                        <label for="ab_testing_enabled"/>
                                        <span class="d-flex">
                                            <field name="ab_testing_enabled" attrs="{'readonly': [('state', '!=', 'draft')]}" force_save="1"/>
                                            <span class="d-flex" attrs="{'invisible': [('ab_testing_enabled', '=', False)]}">
                                                on <field name="ab_testing_pc" class="mx-1 text-center"
                                                    attrs="{'readonly': [('state', '!=', 'draft')]}"/> %
                                            </span>
                                        </span>
                                        <field name="ab_testing_winner_selection"
                                            attrs="{'required': [('ab_testing_enabled', '=', True), ('mailing_type', '=', 'mail')], 'invisible': ['|', ('ab_testing_enabled', '=', False), ('mailing_type', '!=', 'mail')], 'readonly': [('state', '!=', 'draft')]}"/>
                                        <field name="ab_testing_schedule_datetime"
                                            attrs="{'required': [('ab_testing_enabled', '=', True), ('ab_testing_winner_selection', '!=', 'manual')], 'readonly': ['|', ('ab_testing_enabled', '=', False), ('state', '!=', 'draft')], 'invisible': ['|', ('ab_testing_enabled', '=', False), ('ab_testing_winner_selection', '=', 'manual')]}"/>
                                    </group>
                                    <div>
                                        <field name="ab_testing_mailings_count" invisible="1"/>
                                        <field name="ab_testing_completed" invisible="1"/>
                                        <field name="ab_testing_description" nolabel="1"/>
                                        <div attrs="{'invisible': ['|', ('ab_testing_mailings_count', '&lt;', 2), ('ab_testing_enabled', '=', False)]}">
                                            <button name="action_compare_versions" type="object" class="btn btn-link d-block">
                                                <i class="fa fa-bar-chart"/> Compare Version
                                            </button>
                                            <button name="action_duplicate" type="object" class="btn btn-link d-block" attrs="{'invisible': [('ab_testing_completed', '=', True)]}">
                                                <i class="fa fa-copy"/> Create an Alternative
                                            </button>
                                            <button name="action_send_winner_mailing" type="object" class="btn btn-link d-block" attrs="{'invisible': [('ab_testing_completed', '=', True)]}">
                                                <i class="fa fa-envelope"/> <span name="ab_test_manual" attrs="{'invisible': [('ab_testing_winner_selection', '!=', 'manual')]}">
                                                    Send this version to remaining recipients
                                                </span> <span name="ab_test_auto" attrs="{'invisible': [('ab_testing_winner_selection', '=', 'manual')]}">
                                                    Send Winner Now
                                                </span>
                                            </button>
                                            <button name="action_select_as_winner" type="object" class="btn btn-link d-block"
                                                attrs="{'invisible': ['|', ('ab_testing_completed', '!=', False), ('ab_testing_winner_selection', '!=', 'manual')]}">
                                                <i class="fa fa-envelope"/> Send this as winner
                                            </button>
                                        </div>
                                        <button name="action_duplicate" type="object" class="btn btn-primary"
                                            attrs="{'invisible': ['|', ('ab_testing_mailings_count', '&gt;=', 2), ('ab_testing_enabled', '=', False)]}">
                                            Create an Alternative Version
                                        </button>
                                    </div>
                                </group>
                            </page>
                            <page string="Settings" name="settings">
                                <group>
                                    <group string="Email Content" name="email_content" attrs="{'invisible': [('mailing_type', '!=', 'mail')]}">
                                        <field class="o_text_overflow" name="preview" string="Preview Text"
                                            options="{'dynamic_placeholder': true}"
                                            attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"
                                            widget="char_emojis" placeholder="e.g. Check it out before it's too late!"/>
                                        <field name="email_from" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                        <label for="reply_to"/>
                                        <div name="reply_to_details">
                                            <field name="reply_to_mode" widget="radio"
                                                attrs="{
                                                    'invisible': [('mailing_model_name', 'in', ['mailing.contact', 'res.partner', 'mailing.list'])],
                                                    'readonly': [('state', 'in', ('sending', 'done'))]
                                            }"/>
                                            <field name="reply_to"
                                                attrs="{
                                                    'required': [('reply_to_mode', '=', 'new')],
                                                    'invisible': [('reply_to_mode', '=', 'update')],
                                                    'readonly': [('state', 'in', ('sending', 'done'))]
                                            }"/>
                                            <div style="margin-top:-5px">
                                                <small class="oe_edit_only text-muted mb-2"
                                                    style="font-size:74%"
                                                    attrs="{'invisible': ['|', ('reply_to_mode', '=', 'update'), ('mailing_model_name', 'in', ['mailing.contact', 'res.partner', 'mailing.list'])],}">
                                                    To track replies, this address must belong to this database.
                                                </small>
                                            </div>
                                        </div>
                                        <label for="attachment_ids"/>
                                        <div name="attachment_ids_details">
                                            <field name="attachment_ids"  widget="many2many_binary" string="Attach a file" class="oe_inline"
                                                attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                        </div>
                                    </group>
                                    <group string="Tracking">
                                        <field name="campaign_id"
                                            string="Campaign"
                                            groups="mass_mailing.group_mass_mailing_campaign"
                                            options="{'create_name_field': 'title', 'always_reload': True}"
                                            attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                        <field name="medium_id"
                                             string="Medium"
                                             required="True"
                                             groups="base.group_no_one"
                                             attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                        <field name="source_id"
                                            string="Source"
                                            readonly="1"
                                            required="False"
                                            class="o_text_overflow"
                                            groups="base.group_no_one"
                                            attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                        <field name="user_id" widget="many2one_avatar_user"
                                            domain="[('share', '=', False)]"/>
                                    </group>
                                    <group string="Advanced" groups="base.group_no_one">
                                        <field name="mail_server_available" invisible="1"/>
                                        <field name="name" required="False" string="Name" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                        <field name="mail_server_id" attrs="{'readonly': [('state', 'in', ('sending', 'done'))],
                                         'invisible': [('mail_server_available', '=', False)]}"/>
                                        <field name="keep_archives" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                    </group>
                                </group>
                            </page>
                        </notebook>
                    </sheet>
                    <div class="oe_chatter">
                        <field name="message_follower_ids" groups="base.group_user"/>
                        <field name="message_ids"/>
                        <field name="activity_ids"/>
                    </div>
                </form>
            </field>
        </record>

        <!-- Inherited form view for mass mailing's form specifically -->
        <record model="ir.ui.view" id="mailing_mailing_view_form_full_width">
            <field name="name">mailing.mailing.view.form.full.width</field>
            <field name="inherit_id" ref="mass_mailing.view_mail_mass_mailing_form"/>
            <field name="mode">primary</field>
            <field name="model">mailing.mailing</field>
            <field name="arch" type="xml">
                <xpath expr="//form" position="attributes">
                    <attribute name="js_class">mailing_mailing_view_form_full_width</attribute>
                    <attribute name="class">o_form_view o_mass_mailing_mailing_form o_mass_mailing_form_full_width</attribute>
                </xpath>
                <field name="state" position="before">
                    <xpath expr="//div[hasclass('alert-info')]" position="move"/>
                </field>
                <xpath expr="//div[hasclass('alert-info')]" position="attributes">
                    <attribute name="attrs">{'invisible': ['|',('state', '=', 'draft'),'&amp;',('state', '!=', 'in_queue'),('failed', '=', 0)]}</attribute>
                </xpath>
                <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_canceled')]" position="replace"/>
                <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_sent')]" position="replace"/>
                <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_failed')]//span[@name='failed_text']" position="replace">
                    <span name="failed_text">email(s) not sent.</span>
                </xpath>
                <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_scheduled')]" position="attributes">
                    <attribute name="attrs">{'invisible': [('state', '!=', 'in_queue')]}</attribute>
                </xpath>
                <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_scheduled')]/button" position="attributes">
                    <attribute name="attrs">{'invisible': [('scheduled', '=', 0)]}</attribute>
                </xpath>
                <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_scheduled')]//span[@name='scheduled_text']" position="replace">
                    <span name="scheduled_text">email(s) scheduled for </span>
                </xpath>
                <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_scheduled')]/button" position="after">
                </xpath>
                <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_in_queue')]/strong/span[@name='next_departure_text']" position="attributes">
                    <attribute name="attrs">{'invisible': [('scheduled', '!=', 0)]}</attribute>
                </xpath>
                <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_scheduled')]" position="inside">
                    <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_in_queue')]/strong" position="move"/>
                </xpath>
                <xpath expr="//div[hasclass('alert-info')]/div[hasclass('o_mails_in_queue')]" position="replace"/>
                <xpath expr="//sheet" position="before">
                    <xpath expr="//div[hasclass('oe_button_box')]" position="move"/>
                    <xpath expr="//widget[@name='web_ribbon']" position="move"/>
                    <xpath expr="//group[hasclass('o_mass_mailing_mailing_group')]" position="move"/>
                    <xpath expr="//notebook" position="move"/>
                </xpath>
                <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                    <button name="action_view_traces_sent"
                        attrs="{'invisible': [('state', 'in', ('draft','test'))]}"
                        type="object" class="oe_stat_button" icon="fa-paper-plane">
                        <field name="sent" widget="statinfo" string="Sent"/>
                    </button>
                    <button name="action_view_traces_canceled"
                        attrs="{'invisible': [('state', 'in', ('draft','test'))]}"
                        type="object" class="oe_stat_button" icon="fa-paper-plane-o">
                        <field name="canceled" widget="statinfo" string="Ignored"/>
                    </button>
                </xpath>
                <xpath expr="//notebook" position="inside">
                    <page string="Chat" name="chat"/>
                </xpath>
                <xpath expr="//notebook/page[@name='chat']" position="inside">
                    <xpath expr="//div[hasclass('oe_chatter')]" position="move"/>
                </xpath>
                <xpath expr="//div[hasclass('oe_chatter')]" position="attributes">
                    <attribute name="class" remove="o-aside" add="o-full-width" separator=" "/>
                </xpath>
                <xpath expr="//notebook/page[@name='mail_body']//field[@name='body_arch']" position="attributes">
                    <attribute name="iframeHtmlClass">o_mass_mailing_iframe</attribute>
                </xpath>
                <xpath expr="//notebook/page[@name='mail_body']" position="after">
                    <!-- test_01_mass_mailing_editor_tour -->
                    <field name="body_html" invisible="1"/>
                    <page string="Mail Debug" name="mail_debug" groups="base.group_no_one">
                        <div class="position-relative">
                            <div class="mt-n2">
                                <field name="body_html" class="o_mail_body" widget="html"
                                    readonly="True" force_save="1" options="{'cssReadonly': 'mass_mailing.iframe_css_assets_readonly'}"/>
                            </div>
                            <field name="is_body_empty" invisible="1"/>
                            <div class="o_view_nocontent oe_read_only" attrs="{'invisible': ['|', ('is_body_empty', '=', False), ('state', 'in', ('sending', 'done'))]}">
                                <div class="o_nocontent_help">
                                    <p class="o_view_nocontent_smiling_face">
                                        This mailing has no selected design (yet!).
                                    </p>
                                </div>
                            </div>
                        </div>
                    </page>
                </xpath>
                <xpath expr="//sheet" position="replace"/>
            </field>
        </record>

        <record model="ir.ui.view" id="view_mail_mass_mailing_kanban">
            <field name="name">mailing.mailing.kanban</field>
            <field name="model">mailing.mailing</field>
            <field name="arch" type="xml">
                <kanban default_group_by="state" quick_create="false" sample="1">
                    <field name='state' readonly="1"/>
                    <field name='email_from' readonly="1"/>
                    <field name='color'/>
                    <field name='user_id'/>
                    <field name='expected'/>
                    <field name='failed'/>
                    <field name='total'/>
                    <field name='mailing_model_id'/>
                    <field name='mailing_model_name'/>
                    <field name='sent_date'/>
                    <field name='schedule_date'/>
                    <field name='next_departure'/>
                    <field name='active'/>
                    <templates>
                        <t t-name="kanban-box">
                            <div t-attf-class="{{!selection_mode ? 'oe_kanban_color_' + kanban_getcolor(record.color.raw_value) : ''}} oe_kanban_card oe_kanban_global_click oe_kanban_mass_mailing">
                                <div class="o_dropdown_kanban dropdown" t-if="!selection_mode">
                                    <a role="button" class="dropdown-toggle o-no-caret btn" data-bs-toggle="dropdown" href="#" data-bs-display="static" aria-label="Dropdown menu" title="Dropdown menu">
                                        <span class="fa fa-ellipsis-v"/>
                                    </a>
                                    <div class="dropdown-menu" role="menu">
                                        <ul class="oe_kanban_colorpicker" data-field="color"/>
                                        <t t-if="widget.deletable">
                                            <a role="menuitem" type="delete" class="dropdown-item">Delete</a>
                                        </t>
                                        <a role="menuitem" class="dropdown-item o_kanban_mailing_active" name="toggle_active" type="object">
                                            <t t-if="record.active.raw_value">Archive</t>
                                            <t t-if="!record.active.raw_value">Restore</t>
                                        </a>
                                    </div>
                                </div>
                                 <div class="oe_kanban_content">
                                    <div class="o_kanban_record_top">
                                        <div class="o_kanban_record_headings">
                                            <div class="row"  attrs="{'invisible': [('sent_date', '=', False)]}">
                                                <h3 class="my-1 col-8 o_text_overflow">
                                                    <field name="subject"/>
                                                </h3>
                                                <div class="progress border col-3 px-0 mt-2" style="background-color: inherit; height:12px;">
                                                    <div class="progress-bar" role="progressbar" 
                                                        aria-valuemin="0"
                                                        t-att-aria-valuenow="record.delivered.raw_value"
                                                        t-att-aria-valuemax="record.expected.raw_value"
                                                        t-attf-style="width: #{record.delivered.raw_value * 100 / record.expected.raw_value}%"/>
                                                </div>
                                            </div>
                                            <h3 class="my-1"  attrs="{'invisible': [('sent_date', '!=', False)]}">
                                                <field name="subject"/>
                                            </h3>
                                            <field name="mailing_type" invisible="1"/>
                                            <div class="o_kanban_record_subtitle" attrs="{'invisible': [('sent_date', '=', False)]}">
                                                <h5 style="display: inline;">
                                                    <field name="campaign_id" groups="mass_mailing.group_mass_mailing_campaign"/>
                                                </h5>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="o_kanban_record_body" t-if="!selection_mode" attrs="{'invisible': [('sent_date', '=', False), ('schedule_date', '=', False), ('state', '!=', 'in_queue')]}">
                                        <div>
                                            <span attrs="{'invisible': [('sent_date', '=', False)]}" class="me-1"><b><field name="delivered"/> / <field name="expected"/></b> Delivered to</span>
                                            <span attrs="{'invisible': [('sent_date', '!=', False)]}" class="me-1"><b><field name='total'/></b></span>
                                            <field name='mailing_model_id' attrs="{'invisible': [('mailing_model_name','=','mailing.list')]}"/>
                                            <span attrs="{'invisible': [('mailing_model_name','!=','mailing.list')]}">Mailing Contact</span>
                                        </div>
                                        <div attrs="{'invisible': [('sent_date', '=', False)]}" class="d-flex justify-content-between">
                                            <div name="stat_opened">
                                                <b><field name="opened_ratio" />%</b> Opened 
                                            </div>
                                            <div name="stat_replied">
                                                <b><field name="replied_ratio" />%</b> Replied 
                                            </div>
                                            <div name="stat_clicks">
                                                <b><field name="clicks_ratio" />%</b> Clicks 
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div name="div_responsible_avatar" class="o_kanban_record_bottom">
                                    <div class="oe_kanban_bottom_left">
                                        <span attrs="{'invisible': [('sent_date', '=', False)]}"
                                            t-attf-title="Sent on #{record.sent_date.value}" class="d-inline-flex">
                                            <span class="fa fa-calendar-check-o me-2 small my-auto" aria-label="Sent date"/>
                                            <span class="align-self-baseline"><field name="sent_date" widget="date"/></span>
                                        </span>
                                        <span attrs="{'invisible': [('schedule_date', '=', False)]}"
                                            t-attf-title="Scheduled on #{record.schedule_date.value}" class="d-inline-flex">
                                            <span class="fa fa-hourglass-half me-2 small my-auto" aria-label="Scheduled date"/>
                                            <span class="align-self-baseline"><field name="schedule_date" widget="date"/></span>
                                        </span>
                                        <span attrs="{'invisible': ['|', '|', ('sent_date', '!=', False), ('schedule_date', '!=', False), ('state', '=', 'in_queue')]}"
                                            class="clearfix">
                                            <b><field name='total' class="me-1"/></b>
                                            <field name='mailing_model_id' attrs="{'invisible': [('mailing_model_name','=','mailing.list')]}"/>
                                            <span attrs="{'invisible': [('mailing_model_name','!=','mailing.list')]}">Mailing Contact</span>
                                        </span>
                                        <span attrs="{'invisible': ['|', '|', ('schedule_date', '!=', False), ('state', '!=', 'in_queue'), ('next_departure', '=', False)]}"
                                            t-attf-title="Scheduled on #{record.next_departure.value}" class="d-inline-flex">
                                            <span class="fa fa-hourglass-o me-2 small my-auto" aria-label="Scheduled date"/>
                                            <span class="align-self-baseline">Next Batch</span>
                                        </span>
                                    </div>
                                    <div class="oe_kanban_bottom_right">
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                    </div>
                                </div>
                            </div>
                        </t>
                    </templates>
                </kanban>
            </field>
        </record>

        <record id="mailing_mailing_view_calendar" model="ir.ui.view">
            <field name="name">mailing.mailing.view.calendar</field>
            <field name="model">mailing.mailing</field>
            <field name="arch" type="xml">
                <calendar date_start="calendar_date" string="Mailings" hide_time="true" mode="month" color="state" quick_add="False">
                    <field name="mailing_model_id" string="Recipient" options="{'no_open': True}"/>
                    <field name="user_id" filters="1" invisible="1"/>
                    <field name="state" filters="1" invisible="1"/>
                </calendar>
            </field>
        </record>

        <record id="view_mail_mass_mailing_graph" model="ir.ui.view">
            <field name="name">mailing.mailing.graph</field>
            <field name="model">mailing.mailing</field>
            <field name="arch" type="xml">
                <graph string="Mailing" sample="1">
                    <field name="state"/>
                    <field name="color" invisible="1"/>
                </graph>
            </field>
        </record>

        <record id="mailing_mailing_action_mail" model="ir.actions.act_window">
            <field name="name">Mailings</field>
            <field name="res_model">mailing.mailing</field>
            <field name="view_mode">tree,kanban,form,calendar,graph</field>
            <field name="domain">[('mailing_type', '=', 'mail')]</field>
            <field name="context">{
                    'search_default_assigned_to_me': 1,
                    'default_user_id': uid,
                    'default_mailing_type': 'mail',
            }</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a Mailing
              </p><p>
                Design a striking email, define recipients and track its results.
              </p>
            </field>
        </record>

        <record id="mailing_mailing_action_mail_fullwidth_tree" model="ir.actions.act_window.view">
            <field name="sequence" eval="0"/>
            <field name="view_mode">tree</field>
            <field name="act_window_id" ref="mass_mailing.mailing_mailing_action_mail"/>
        </record>
        <record id="mailing_mailing_action_mail_fullwidth_kanban" model="ir.actions.act_window.view">
            <field name="sequence" eval="1"/>
            <field name="view_mode">kanban</field>
            <field name="act_window_id" ref="mass_mailing.mailing_mailing_action_mail"/>
        </record>
        <record id="mailing_mailing_action_mail_fullwidth_form" model="ir.actions.act_window.view">
            <field name="sequence" eval="2"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="mass_mailing.mailing_mailing_view_form_full_width"/>
            <field name="act_window_id" ref="mass_mailing.mailing_mailing_action_mail"/>
        </record>
        <record id="mailing_mailing_action_mail_fullwidth_calendar" model="ir.actions.act_window.view">
            <field name="sequence" eval="3"/>
            <field name="view_mode">calendar</field>
            <field name="act_window_id" ref="mass_mailing.mailing_mailing_action_mail"/>
        </record>
        <record id="mailing_mailing_action_mail_fullwidth_graph" model="ir.actions.act_window.view">
            <field name="sequence" eval="4"/>
            <field name="view_mode">graph</field>
            <field name="act_window_id" ref="mass_mailing.mailing_mailing_action_mail"/>
        </record>

        <record id="action_view_mass_mailings_from_campaign" model="ir.actions.act_window">
            <field name="name">Mailings</field>
            <field name="res_model">mailing.mailing</field>
            <field name="view_mode">kanban,tree,form,calendar</field>
            <field name="context">{
                'search_default_assigned_to_me': 1,
                'search_default_campaign_id': [active_id],
                'default_campaign_id': active_id,
                'default_user_id': uid,
            }
            </field>
            <field name="domain">[('mailing_type', '=', 'mail')]</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new mailing
              </p><p>
                You don't need to import your mailing lists, you can easily
                send emails<br/> to any contact saved in other Odoo apps.
              </p>
            </field>
        </record>

        <record id="action_create_mass_mailings_from_campaign" model="ir.actions.act_window">
            <field name="name">Mailings</field>
            <field name="res_model">mailing.mailing</field>
            <field name="view_mode">form,kanban,tree</field>
            <field name="context">{
                'search_default_assigned_to_me': 1,
                'search_default_campaign_id': [active_id],
                'default_campaign_id': active_id,
                'default_user_id': uid,
            }
            </field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new mailing
              </p><p>
                You don't need to import your mailing lists, you can easily
                send emails<br/> to any contact saved in other Odoo apps.
              </p>
            </field>
        </record>

        <record id="action_ab_testing_open_winner_mailing" model="ir.actions.act_window">
            <field name="name">A/B Test Winner</field>
            <field name="res_model">mailing.mailing</field>
            <field name="view_mode">form</field>
        </record>
</odoo>

```

## File: views\mailing_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Marketing / Mailing -->
    <menuitem name="Email Marketing"
              id="mass_mailing_menu_root"
              sequence="115"
              web_icon="mass_mailing,static/description/icon.svg"
              groups="mass_mailing.group_mass_mailing_user"/>

    <!-- Mailings -->
    <menuitem name="Mailings"
              id="mass_mailing_menu"
              parent="mass_mailing_menu_root"
              sequence="1"
              action="mailing_mailing_action_mail"/>

    <!-- Mailing lists -->
    <menuitem name="Mailing Lists"
              id="mass_mailing_mailing_list_menu"
              parent="mass_mailing_menu_root"
              sequence="2"/>
    <menuitem name="Mailing Lists"
              id="menu_email_mass_mailing_lists"
              parent="mass_mailing_mailing_list_menu"
              sequence="1"
              action="action_view_mass_mailing_lists"/>
    <menuitem name="Mailing List Contacts"
              id="menu_email_mass_mailing_contacts"
              parent="mass_mailing_mailing_list_menu"
              sequence="2"
              action="action_view_mass_mailing_contacts"/>

    <!-- Campaigns -->
    <menuitem name="Campaigns"
              id="menu_email_campaigns"
              parent="mass_mailing_menu_root"
              sequence="3"
              action="action_view_utm_campaigns"
              groups="mass_mailing.group_mass_mailing_campaign"/>

    <!-- Reporting -->
    <menuitem name="Reporting"
              id="menu_mass_mailing_report"
              sequence="90"
              parent="mass_mailing_menu_root"
              action="mailing_trace_report_action_mail"/>

    <!-- Configuration -->
    <menuitem name="Configuration"
              id="mass_mailing_configuration"
              parent="mass_mailing_menu_root"
              sequence="100"/>
    <!-- Configuration / Settings -->
    <menuitem name="Settings"
              id="menu_mass_mailing_global_settings"
              parent="mass_mailing_configuration"
              sequence="0"
              action="action_mass_mailing_configuration"
              groups="base.group_system"/>
    <!-- Configuration / Campaign stages -->
    <menuitem name="Campaign Stages"
              id="menu_view_mass_mailing_stages"
              parent="mass_mailing_configuration"
              sequence="1"
              groups="mass_mailing.group_mass_mailing_campaign"
              action="utm.action_view_utm_stage"/>
    <!-- Configuration / Utm Tags -->
    <menuitem id="mass_mailing_tag_menu"
              parent="mass_mailing_configuration"
              action="utm.action_view_utm_tag"
              sequence="2"
              groups="mass_mailing.group_mass_mailing_campaign"/>
    <!-- Configuration / Link trackers -->
    <menuitem id="link_tracker_menu_mass_mailing"
              name="Link Tracker"
              parent="mass_mailing_configuration"
              sequence="10"
              action="link_tracker.link_tracker_action"/>
    <!-- Configuration / Blacklist -->
    <menuitem id="mail_blacklist_mm_menu"
              name="Blacklisted Email Addresses"
              action="mail.mail_blacklist_action"
              parent="mass_mailing_configuration"
              sequence="20"/>
    <!-- Configuration / Favorite Filters -->
    <menuitem id="mailing_filter_menu_action"
              action="mailing_filter_action"
              parent="mass_mailing_configuration"
              sequence="30"/>

    <!-- Technical / Mass Mailing -->
    <menuitem id="mailing_mailing_menu_technical"
              name="Mass Mailing"
              sequence="4"
              parent="base.menu_custom"/>
    <menuitem id="menu_email_statistics"
              name="Mailing Traces"
              parent="mass_mailing.mailing_mailing_menu_technical"
              sequence="2"
              action="mailing_trace_action"/>

        
</odoo>

```

## File: views\mailing_templates_portal_layouts.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- new layout for mass_mailing -->
    <template id="mass_mailing.layout" name="Mass Mailing Layout">
        <t t-set="html_data" t-value="{'lang': lang and lang.replace('_', '-')}"/>
        <t t-call="web.frontend_layout">
            <body class="bg-white o_mailing_portal_body">
                <header>
                    <div><title>Odoo</title></div>
                    <div class="text-center">
                        <img t-attf-src="/web/binary/company_logo?company={{ res_company.id }}"/>
                    </div>
                </header>
                <div id="wrap" class="oe_structure oe_empty"/>
                <main>
                    <t t-out="0"/>
                </main>
            </body>
            <xpath expr="//head/t[@t-call-assets][last()]" position="after">
                <t t-call-assets="mass_mailing.mailing_assets" lazy_load="True"/>
            </xpath>
            <xpath expr="//header" position="before">
                <t t-set="no_header" t-value="True"/>
            </xpath>
         </t>
     </template>
</odoo>

```

## File: views\mailing_templates_portal_management.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="mailing_report_deactivated" name="Report Unsubscribed">
        <t t-call="mass_mailing.layout">
            <div class="container mt8">
                <div class="row">
                    <div class="col-lg-6 offset-lg-3">
                        <h3>Mailing Reports Turned Off</h3>
                        <div class="alert alert-success text-center" role="status">
                            <p>
                                Mailing Reports have been turned off for all users. <br/>
                                If needed, they can be turned back on from the
                                <a t-if="menu_id" t-attf-href="/web#menu_id=#{menu_id}">
                                    Settings Menu.
                                </a>
                                <t t-else="">
                                    Settings Menu.
                                </t>
                            </p>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <!-- Dummy layout to "view" a template content (html) -->
    <template id="view" name="Browser View">
&lt;!DOCTYPE html&gt;
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>

        <style type="text/css">
            <!-- Hide the link view online as it is displayed online -->
            .o_snippet_view_in_browser {
                display: none;
            }
        </style>
    </head>
    <body>
        <!-- Raw body inserted here because it is a rendered mailing, therefore internal content -->
        <t t-out="body"/>
    </body>
</html>
    </template>
</odoo>

```

## File: views\mailing_templates_portal_unsubscribe.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="unsubscribe">
        <div class="container o_unsubscribe_form">
            <div class="row">
                <form action="/mail/mailing/unsubscribe" method="POST" id="unsubscribe_form" class="col-lg-6 offset-lg-3 mt-4">
                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
                    <input type="hidden" name="email" t-att-value="email"/>
                    <input type="hidden" name="mailing_id" t-att-value="mailing_id"/>
                    <input type="hidden" name="res_id" t-att-value="res_id"/>
                    <input type="hidden" name="unsubscribed_list" t-att-value="unsubscribed_list"/>

                    <div>
                        <t t-if="contacts">
                            <div id="info_state" class="alert alert-success" role="status">
                                <div id="subscription_info"></div>
                                <div id="div_feedback">
                                    <p>We would appreciate if you provide feedback about why you updated<br/>your subscriptions</p>
                                    <textarea class="form-control"  name="opt_out_feedback" cols="60" rows="3"></textarea>
                                    <br/>
                                    <div class="btn btn-primary text-start" id="button_feedback">Send</div>
                                </div>
                            </div>

                            <h1 class="o_page_header">Mailing Subscriptions</h1>
                            <p>Choose your mailing subscriptions</p>
                            <div id="div_opt_out">
                                <ul class="list-group">
                                    <t t-foreach="list_ids" t-as="list_id">
                                        <t t-if="list_id.is_public == True">
                                            <li class="list-group-item">
                                                <input type="checkbox" name="contact_ids"
                                                    t-att-value="list_id['id']" t-att-checked="None if list_id['id'] in opt_out_list_ids else 'checked'"/>
                                                <t t-esc="list_id.name"/>
                                                <span t-if="list_id['id'] in opt_out_list_ids"
                                                      class="o_mailing_portal_list_unsubscribed">
                                                    Unsubscribed
                                                </span>
                                            </li>
                                        </t>
                                    </t>
                                </ul>

                                <div class="mb64 pt-3">
                                    <div class="btn btn-link float-end pe-0 text-uppercase"
                                         t-if="show_blacklist_button"
                                         id="button_add_blacklist"
                                         style="display:none">Blacklist Me</div>
                                    <div class="btn btn-link float-end pe-0 text-uppercase"
                                         id="button_remove_blacklist"
                                         style="display:none">Come Back</div>
                                    <button type="submit" id="send_form"
                                            class="btn btn-primary">Update my subscriptions</button>
                                </div>
                            </div>

                        </t>
                        <t t-else="">
                            <div class="alert alert-info text-center" role="status">
                                <p>You are not subscribed to any of our mailing list.</p>
                            </div>
                        </t>
                    </div>
                </form>
            </div>
        </div>
    </template>

    <template id="unsubscribed">
        <div class="container o_unsubscribe_form">
            <div class="row">
                <input type="hidden" name="email" t-att-value="email"/>
                <input type="hidden" name="mailing_id" t-att-value="mailing_id"/>
                <input type="hidden" name="res_id" t-att-value="res_id"/>
                <div id="div_blacklist" class="col-lg-6 offset-lg-3">
                    <h1 class="o_page_header">Mailing Subscriptions</h1>

                    <div id="subscription_info" class="alert alert-success text-center" role="status">
                        <p>You have been successfully <strong>unsubscribed</strong>!</p>
                    </div>

                    <div t-if="list_ids" class="alert alert-warning">
                        <p class="text-center">You were still subscribed to those newsletters. You will not receive any news from them anymore:</p>
                        <ul class="list-group mb-4">
                            <t t-foreach="list_ids" t-as="list_id">
                                <t t-if="list_id.is_public == True">
                                    <li class="list-group-item bg-transparent">
                                        <strong><t t-esc="list_id.name"/></strong>
                                    </li>
                                </t>
                            </t>
                        </ul>
                    </div>

                    <div t-if="show_blacklist_button" class="mb64">
                        <div class="btn btn-link float-end"
                             id="button_add_blacklist"
                             style="display:none">Blacklist Me</div>
                        <div class="btn btn-link float-end"
                             id="button_remove_blacklist"
                             style="display:none">Come Back</div>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="page_unsubscribe" name="Unsubscribe">
        <t t-call="mass_mailing.layout">
            <t t-call="mass_mailing.unsubscribe"/>
        </t>
    </template>

    <template id="page_unsubscribed" name="Unsubscribed">
        <t t-call="mass_mailing.layout">
            <t t-call="mass_mailing.unsubscribed"/>
        </t>
    </template>
</odoo>

```

## File: views\mailing_trace_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--  MAILING TRACE !-->
    <record model="ir.ui.view" id="mailing_trace_view_search">
        <field name="name">mailing.trace.view.search</field>
        <field name="model">mailing.trace</field>
        <field name="arch" type="xml">
           <search string="Mail Statistics">
                <field name="mail_mail_id_int"/>
                <field name="message_id"/>
                <field name="email"/>
                <field name="mass_mailing_id"/>
                <filter string="Scheduled" name="filter_scheduled" domain="[('trace_status', '=', 'outgoing')]"/>
                <filter string="Canceled" name="filter_canceled" domain="[('trace_status', '=', 'cancel')]"/>
                <filter string="Sent" name="filter_sent" domain="[('sent_datetime', '!=', False)]"/>
                <filter string="Clicked" name="filter_clicked" domain="[('links_click_datetime', '!=', False)]"/>
                <filter string="Delivered" name="filter_delivered" domain="[('sent_datetime', '!=', False), ('trace_status', 'not in', ['error', 'cancel'])]"/>
                <filter string="Opened" name="filter_opened" domain="[('trace_status', 'in', ['open', 'reply'])]"/>
                <filter string="Replied" name="filter_replied" domain="[('trace_status', '=', 'reply')]"/>
                <filter string="Bounced" name="filter_bounced" domain="[('trace_status', '=', 'bounce')]"/>
                <filter string="Failed" name="filter_failed" domain="[('trace_status', '=', 'error')]"/>
                <group expand="0" string="Group By">
                    <filter string="State" name="state" domain="[]" context="{'group_by': 'trace_status'}"/>
                    <filter string="Open Date" name="group_open_date" domain="[('trace_status', 'in', ['open', 'reply'])]" context="{'group_by': 'open_datetime:day'}"/>
                    <filter string="Reply Date" name="group_reply_date" domain="[('trace_status', '=', 'reply')]" context="{'group_by': 'reply_datetime:day'}"/>
                    <filter string="Last State Update" name="state_update" domain="[]" context="{'group_by': 'write_date'}"/>
                    <filter string="Mass Mailing" name="mass_mailing" domain="[]" context="{'group_by': 'mass_mailing_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="mailing_trace_view_tree" model="ir.ui.view">
        <field name="name">mailing.trace.view.tree</field>
        <field name="model">mailing.trace</field>
        <field name="arch" type="xml">
            <tree string="Mailing Traces" create="0">
                <field name="mass_mailing_id"/>
                <field name="email"/>
                <field name="message_id"/>
                <field name="sent_datetime"/>
                <field name="links_click_datetime"/>
                <field name="trace_status" widget="badge"/>
                <field name="failure_type" optional="show"/>
                <field name="open_datetime" optional="hide"/>
                <field name="reply_datetime" optional="hide"/>
                <button name="action_view_contact" type="object"
                        string="Open Recipient" icon="fa-user"/>
            </tree>
        </field>
    </record>

    <record id="mailing_trace_view_tree_mail" model="ir.ui.view">
        <field name="name">mailing.trace.view.tree.mail</field>
        <field name="model">mailing.trace</field>
        <field name="priority">20</field>
        <field name="arch" type="xml">
            <tree string="Mail Traces" create="0">
                <field name="mass_mailing_id"/>
                <field name="email"/>
                <field name="message_id" optional="hide"/>
                <field name="sent_datetime"/>
                <field name="links_click_datetime"/>
                <field name="trace_status" widget="badge"/>
                <field name="failure_type" optional="show"/>
                <field name="open_datetime" optional="hide"/>
                <field name="reply_datetime" optional="hide"/>
                <button name="action_view_contact" type="object"
                        string="Open Recipient" icon="fa-user"/>
            </tree>
        </field>
    </record>

    <record id="mailing_trace_view_form" model="ir.ui.view">
        <field name="name">mailing.trace.view.form</field>
        <field name="model">mailing.trace</field>
        <field name="arch" type="xml">
            <form string="Mail Statistics" create="0" edit="0">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button name="action_view_contact"
                                type="object" icon="fa-user" class="oe_stat_button">
                            <span widget="statinfo">Open Recipient</span>
                        </button>
                    </div>
                    <group>
                        <group string="Status">
                            <field name="trace_status"/>
                            <field name="failure_type" attrs="{'invisible' : [('failure_type', '=', False)]}"/>
                            <field name="sent_datetime" attrs="{'invisible' : [('sent_datetime', '=', False)]}"/>
                            <field name="links_click_datetime" attrs="{'invisible' : [('links_click_datetime', '=', False)]}"/>
                            <field name="open_datetime" attrs="{'invisible' : [('open_datetime', '=', False)]}"/>
                            <field name="reply_datetime" attrs="{'invisible' : [('reply_datetime', '=', False)]}"/>
                        </group>
                        <group string="Mailing">
                            <field name="trace_type" invisible="1"/>
                            <field name="email" string="Recipient Address"/>
                            <field name="mass_mailing_id"/>
                            <field name="mail_mail_id_int" string="Mail ID" groups="base.group_no_one"/>
                            <field name="message_id" groups="base.group_no_one"/>
                        </group>
                        <group string="Marketing">
                            <field name="campaign_id" groups="mass_mailing.group_mass_mailing_campaign"/>
                            <field name="medium_id"/>
                            <field name="source_id"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_mail_mail_statistics_graph" model="ir.ui.view">
        <field name="name">Mail Statistics Graph</field>
        <field name="model">mailing.trace</field>
        <field name="arch" type="xml">
            <graph string="Mail Statistics" sample="1">
                <field name="write_date" interval="day"/>
                <field name="trace_status"/>
            </graph>
        </field>
    </record>

    <record id="mailing_trace_action" model="ir.actions.act_window">
        <field name="name">Mailing Traces</field>
        <field name="res_model">mailing.trace</field>
        <field name="view_mode">tree,form,graph,pivot</field>
        <field name="domain">[]</field>
    </record>

    <record id="action_view_mail_mail_statistics_mailing" model="ir.actions.act_window">
        <field name="name">Mail Statistics</field>
        <field name="res_model">mailing.trace</field>
        <field name="view_mode">graph,tree,form,pivot</field>
        <field name="domain">[]</field>
        <field name="context">{'search_default_mass_mailing_id': active_id}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No data yet!
            </p>
        </field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.mass.mailing</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="60"/>
            <field name="inherit_id" ref="base.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('settings')]" position="inside">
                    <div class="app_settings_block" data-string="Email Marketing" string="Email Marketing" data-key="mass_mailing" groups="mass_mailing.group_mass_mailing_user">
                      <h2>Email Marketing</h2>
                        <div class="row mt16 o_settings_container" name="managa_mail_campaigns_setting_container">
                            <div class="col-lg-6 o_setting_box col-12" title="This tool is advised if your marketing campaign is composed of several emails.">
                                <div class="o_setting_left_pane" title="This is useful if your marketing campaigns are composed of several emails.">
                                    <field name="group_mass_mailing_campaign"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="group_mass_mailing_campaign"/>
                                    <div class="text-muted">
                                        Manage mass mailing campaigns
                                    </div>
                                </div>
                            </div>
                            <div class="col-lg-6 o_setting_box col-12" name="dedicated_server_setting_container">
                                <div class="o_setting_left_pane" title="Use a specific mail server in priority. Otherwise Odoo relies on the first outgoing mail server available (based on their sequencing) as it does for normal mails.">
                                    <field name="mass_mailing_outgoing_mail_server"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="mass_mailing_outgoing_mail_server"/>
                                    <div class="text-muted">
                                        Pick a dedicated outgoing mail server for your mass mailings
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('mass_mailing_outgoing_mail_server', '=', False)]}">
                                        <div class="mt16">
                                            <field name="mass_mailing_mail_server_id" options="{'no_create': True, 'no_open': True}"
                                                   placeholder="Default Server"/>
                                        </div>
                                        <div class="mt8">
                                            <button type="action" name="base.action_ir_mail_server_list" string="Configure Outgoing Mail Servers" icon="fa-arrow-right" class="oe_link"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-6 o_setting_box col-xs-12" name="allow_blacklist_setting_container">
                                <div class="o_setting_left_pane" title="Allow the recipient to manage themselves their state in the blacklist via the unsubscription page.
                                If the option is active, the 'Blacklist Me' button is hidden on the unsubscription page.
                                The 'come Back' button will always be visible in any case to allow leads and partners to re-subscribe.">
                                    <field name="show_blacklist_buttons"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="show_blacklist_buttons"/>
                                    <div class="text-muted">
                                        Allow recipients to blacklist themselves
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-6 o_setting_box col-xs-12" name="mass_mailing_reports_setting_container">
                                <div class="o_setting_left_pane" title="Send a report to the mailing responsible one day after the mailing has been sent.">
                                    <field name="mass_mailing_reports"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="mass_mailing_reports"/>
                                    <div class="text-muted">
                                        Check how well your mailing is doing a day after it has been sent
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="action_mass_mailing_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="type">ir.actions.act_window</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'mass_mailing', 'bin_size': False}</field>
        </record>
</odoo>

```

## File: views\snippets_themes.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<!-- Snippets & Themes Menu -->
<template id="email_designer_snippets" inherit_id="web_editor.snippets" primary="True" groups="base.group_user">
    <xpath expr="//div[hasclass('o_we_website_top_actions')]" position="inside">
        <div class="email_designer_top_actions">
            <button class="o_codeview_btn btn btn-primary">
                <i class="fa fa-code"></i>
            </button>
            <button class="o_mobile_preview_btn btn btn-primary">
                <i class="fa fa-lg fa-mobile"/>
            </button>
            <button class="o_fullscreen_btn btn btn-primary">
                <img class="img-fluid" src="/web_editor/font_to_img/61541/rgb(255,255,255)/16" alt="Fullscreen"/>
            </button>
        </div>
    </xpath>
    <xpath expr="//div[@id='snippets_menu']" position="inside">
        <button type="button" tabindex="3" class="o_we_customize_design_btn text-uppercase" accesskey="2">
            <span>Design</span>
        </button>
    </xpath>
    <xpath expr="//t[@id='default_snippets']" position="replace">
        <t id="default_snippets">
            <t t-set="company_id" t-value="res_company"/>
            <div id="email_designer_themes" class="d-none">
                <div data-name="basic"
                     title="Plain Text"
                     data-nowrap="1"
                     data-img="/mass_mailing/static/src/img/theme_imgs/basic_thumb"
                     data-images-info='{"logo": {"format": "png"}}'>
                    <t t-call="mass_mailing.theme_basic_template"/>
                </div>
                <div data-name="empty"
                     title="Start From Scratch"
                     data-img="/mass_mailing/static/src/img/theme_imgs/empty_thumb"
                     data-images-info='{"logo": {"format": "png"}}'
                     data-hide-from-mobile="true">
                    <t t-call="mass_mailing.theme_empty_template"/>
                </div>
                <div data-name="default"
                     title="Welcome Message"
                     data-img="/mass_mailing/static/src/img/theme_imgs/default_thumb"
                     data-images-info='{"logo": {"format": "png"}, "header_logo": {"format": "png"}}'>
                    <t t-call="mass_mailing.theme_default_template"/>
                </div>
            </div>
            <div id="email_designer_default_headers" class="o_panel">
                <div class="o_panel_header">Headers</div>
                <div class="o_panel_body" id="email_designer_header_elements">
                    <t t-snippet="mass_mailing.s_mail_block_header_social" string="Left Logo" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/block_header_social.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_header_text_social" string="Left Text" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/block_header_text_social.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_header_logo" string="Centered Logo" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/block_header_logo.png"/>
                    <t t-snippet="mass_mailing.s_cover" string="Cover" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_cover.svg"/>
                    <t t-snippet="mass_mailing.s_mail_block_header_view" string="View Online" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/block_header_browser.png"/>
                </div>
            </div>
            <div id="email_designer_default_body" class="o_panel">
                <div class="o_panel_header">Body</div>
                <div class="o_panel_body" id="email_designer_body_elements">
                    <t t-snippet="mass_mailing.s_title" string="Title" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_title.svg"/>
                    <t t-snippet="mass_mailing.s_text_block" string="Text" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_text_block.svg"/>
                    <t t-snippet="mass_mailing.s_comparisons" string="Comparisons" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_comparisons.svg"/>
                    <t t-snippet="mass_mailing.s_color_blocks_2" string="Big Boxes" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_color_blocks_2.svg"/>
                    <t t-snippet="mass_mailing.s_three_columns" string="Columns" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_three_columns.svg"/>
                    <t t-snippet="mass_mailing.s_image_text" string="Image - Text" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_image_text.svg"/>
                    <t t-snippet="mass_mailing.s_text_image" string="Text - Image" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_text_image.svg"/>
                    <t t-snippet="mass_mailing.s_picture" string="Picture" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_picture.svg"/>
                    <t t-snippet="mass_mailing.s_features" string="Features" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_features.svg"/>
                    <t t-snippet="mass_mailing.s_numbers" string="Numbers" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_numbers.svg"/>
                    <t t-snippet="mass_mailing.s_masonry_block" string="Masonry" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_masonry_block.svg"/>
                    <t t-snippet="mass_mailing.s_media_list" string="Media List" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_media_list.svg"/>
                    <t t-snippet="mass_mailing.s_showcase" string="Showcase" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_showcase.svg"/>
                </div>
            </div>
            <div id="email_designer_default_extra" class="o_panel">
                <div class="o_panel_header">Marketing Content</div>
                <div class="o_panel_body" id="email_designer_marketing_elements">
                    <t t-snippet="mass_mailing.s_company_team" string="Team" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_company_team.svg"/>
                    <t t-snippet="mass_mailing.s_call_to_action" string="Call to Action" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_call_to_action.svg"/>
                    <t t-snippet="mass_mailing.s_references" string="References" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_references.svg"/>
                    <t t-snippet="mass_mailing.s_coupon_code" string="Promo Code" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/block_discount2.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_discount1" string="Discount Offer" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/block_discount1.png"/>
                    <t t-snippet="mass_mailing.s_event" string="Event" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_event.svg"/>
                    <t t-snippet="mass_mailing.s_product_list" string="Items" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_product_list.svg"/>
                    <t t-snippet="mass_mailing.s_features_grid" string="Features Grid" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_features_grid.svg"/>
                </div>
            </div>
            <div id="email_designer_default_inner" class="o_panel">
                <div class="o_panel_header">Inner Content</div>
                <div class="o_panel_body" id="email_designer_inner_elements">
                    <t t-snippet="mass_mailing.s_alert" string="Alert" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_alert.svg"/>
                    <t t-snippet="mass_mailing.s_rating" string="Rating" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_rating.svg"/>
                    <t t-snippet="mass_mailing.s_blockquote" string="Blockquote" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_blockquote.svg"/>
                    <t t-snippet="mass_mailing.s_hr" string="Separator" t-thumbnail="/web_editor/static/src/img/snippets_thumbs/s_hr.svg"/>
                    <t t-snippet="mass_mailing.s_text_highlight" string="Text Highlight" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_text_highlight.svg"/>
                </div>
            </div>
            <div id="email_designer_default_footer" class="o_panel">
                <div class="o_panel_header">Footers</div>
                <div class="o_panel_body" id="email_designer_footer_elements">
                    <t t-snippet="mass_mailing.s_mail_block_footer_social" string="Footer Center" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/block_footer_social.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_footer_social_left" string="Footer Left" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/block_footer_social_left.png"/>
                </div>
            </div>
        </t>
    </xpath>
    <xpath expr="//div[@id='snippet_options']/t" position="attributes">
        <attribute name="t-call">mass_mailing.snippet_options</attribute>
    </xpath>
</template>

<!-- Snippet Templates -->
<template id="s_mail_block_header_social" name="Left Logo">
    <div class="s_header_social o_mail_block_header_social o_mail_snippet_general pt16 pb16">
        <div class="container">
            <div class="row">
                <div class="col-lg-4 pt16 pb16">
                    <a t-att-href="(company_id.website) or '#'" style="text-decoration:none;float:none;" target="_blank">
                         <img t-if="company_id.logo" border="0" t-att-src="image_data_uri(company_id.logo)" style="height:auto;max-width:100%;" />
                    </a>
                </div>
                <div class="col-lg-8 o_mail_header_social" style="text-align: right;">
                    <t t-call="mass_mailing.social_links"/>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_mail_block_header_text_social" name="Left Text">
    <div class="s_header_text_social o_mail_block_header_text_social o_mail_snippet_general">
        <div class="container">
            <div class="row">
                <div class="col-lg-4 pt16 pb16">
                    <h1>
                        <a t-att-href="(company_id.website) or '#'" target="_blank">
                            My Company
                        </a>
                    </h1>
                </div>
                <div class="col-lg-8 o_mail_header_social" style="text-align: right;">
                    <t t-call="mass_mailing.social_links"/>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_mail_block_header_logo" name="Centered Logo">
    <div class="s_header_logo o_mail_block_header_logo o_mail_snippet_general">
        <div class="container">
            <div class="row">
                <div class="col-lg-3"/>
                <div class="col-lg-6" style="text-align: center;">
                    <a t-att-href="(company_id.website) or '#'" style="text-decoration:none;" target="_blank">
                        <img t-if="company_id.logo" border="0" t-att-src="image_data_uri(company_id.logo)" style="height:auto;max-width:100%;"/>
                    </a>
                </div>
                <div class="col-lg-3" style="text-align: right;"/>
            </div>
        </div>
    </div>
</template>

<template id="s_mail_block_header_view" name="View Online">
    <div class="o_snippet_view_in_browser o_mail_snippet_general pt16 pb16" style="text-align: center; padding-left: 15px; padding-right: 15px;">
        <a href="/view">
            View Online
        </a>
    </div>
</template>

<template id="s_mail_block_discount1" name="Discount Offer">
    <div class="s_discount1 o_mail_block_discount1 o_mail_snippet_general pt0 pb16">
        <div class="container">
            <div class="row">
                <div class="col-lg-6 pt16">
                    <h3 style="text-align: center;"><font class="text-o-color-2"><span style="font-weight:bolder;">-20%</span></font></h3>
                    <p style="text-align: center;">ON YOUR NEXT ORDER!</p>
                    <div style="text-align: center;">
                        <a role="button" href="#" class="btn btn-primary">Redeem Discount!</a>
                    </div>
                </div>
                <div class="col-lg-6 pt16">
                    <p>We are continuing to grow and we miss seeing you be a part of it! We've increased store hours and have lot's of new brands available. To welcome you back please accept this 20% discount on you next purchase by clicking the button.</p>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_mail_block_footer_social" name="Footer Center">
    <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general bg-200 pt16">
        <div class="container">
            <div class="row">
                <div class="col-lg o_mail_footer_social" style="text-align: center;">
                    <t t-call="mass_mailing.social_links"/>
                </div>
            </div>
            <div class="row">
                <div class="col-lg o_mail_footer_links" style="text-align: center;">
                    <a role="button" href="/unsubscribe_from_list" class="btn btn-link">Unsubscribe</a>
                </div>
            </div>
            <div class="row">
                <div class="col-lg">
                    <p style="text-align: center;">
                        © <t t-esc="datetime.datetime.now().year"/> All Rights Reserved
                    </p>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_mail_block_footer_social_left" name="Footer Left">
    <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_left o_mail_snippet_general pt16">
        <div class="container">
            <div class="row">
                <div class="col-lg o_mail_footer_description">
                    <p t-if="res_company">
                        <strong><t t-esc="res_company.partner_id.name"/></strong>
                    </p>
                    <div class="o_mail_footer_links">
                        <a role="button" href="/unsubscribe_from_list" class="btn btn-link">Unsubscribe</a>
                    </div>
                </div>
                <div class="col-lg" align="right">
                    <div class="o_mail_footer_social pb16"><t t-call="mass_mailing.social_links"/></div>
                    <p class="o_mail_footer_copy">© <t t-esc="datetime.datetime.now().year"/> All Rights Reserved</p>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="social_links">

    <t t-set="social_links" t-value="company_id._get_social_media_links()"/>

    <a t-att-href="social_links.get('social_facebook')" aria-label="Facebook" title="Facebook">
        <span class="fa fa-facebook"></span>
    </a>&amp;nbsp;&amp;nbsp;
    <a t-att-href="social_links.get('social_linkedin')" style="margin-left:10px" aria-label="LinkedIn" title="LinkedIn">
        <span class="fa fa-linkedin"></span>
    </a>&amp;nbsp;&amp;nbsp;
    <a t-att-href="social_links.get('social_twitter')" style="margin-left:10px" aria-label="Twitter" title="Twitter">
        <span class="fa fa-twitter"></span>
    </a>&amp;nbsp;&amp;nbsp;
    <a t-att-href="social_links.get('social_instagram')" style="margin-left:10px" aria-label="Instagram" title="Instagram">
        <span class="fa fa-instagram"></span>
    </a>
</template>

<!-- Border -->
<template id="snippet_options_border_line_widgets">
    <we-row t-att-string="label">
        <we-input data-name="border_width_opt"
                t-att-data-apply-to="apply_to"
                data-select-style="0"
                t-attf-data-css-property="border-#{direction and ('%s-' % direction) or ''}width"
                data-unit="px"
                t-att-data-extra-class="with_bs_class and 'border'"
                t-att-data-variable="width_variable"/>
        <we-select t-attf-data-css-property="border-#{direction and ('%s-' % direction) or ''}style"
                data-dependencies="border_width_opt"
                t-att-data-apply-to="apply_to"
                t-att-data-variable="style_variable">
            <we-button title="Solid" data-select-style="solid"><div class="o_we_fake_img_item o_we_border_preview" style="border-style: solid;"/></we-button>
            <we-button title="Dashed" data-select-style="dashed"><div class="o_we_fake_img_item o_we_border_preview" style="border-style: dashed;"/></we-button>
            <we-button title="Dotted" data-select-style="dotted"><div class="o_we_fake_img_item o_we_border_preview" style="border-style: dotted;"/></we-button>
            <we-button title="Double" data-select-style="double"><div class="o_we_fake_img_item o_we_border_preview" style="border-style: double; border-left: none; border-right: none;"/></we-button>
        </we-select>
        <we-colorpicker data-dependencies="border_width_opt"
                        t-att-data-apply-to="apply_to"
                        data-select-style="true"
                        t-attf-data-css-property="border-#{direction and ('%s-' % direction) or ''}color"
                        data-color-prefix="border-"
                        t-att-data-color="color_variable"/>
    </we-row>
</template>

<template id="snippet_options_border_widgets">
    <t t-call="mass_mailing.snippet_options_border_line_widgets">
        <t t-set="label">Border</t>
        <t t-set="with_bs_class" t-value="True"/>
    </t>
    <we-input string="Round Corners"
            t-att-data-apply-to="apply_to"
            t-att-data-dependencies="not so_rounded_no_dependencies and 'border_width_opt,bg_color_opt'"
            data-select-style="0" data-css-property="border-radius"
            data-unit="px" data-extra-class="rounded"
            t-att-data-variable="radius_variable"/>
</template>

<template id="snippet_options_background_options" inherit_id="web_editor.snippet_options_background_options" primary="True">
    <xpath expr="//div[@data-js='BackgroundImage']" position="attributes">
        <attribute name="data-js">MassMailingBackgroundImage</attribute>
    </xpath>
</template>

<!--
TODO these next two templates should have been removed in 16.0 as they were
emptied pre-release while waiting for a proper upgrade script to remove them
after backport. They will now be removed in master.
-->
<template id="snippet_options_extra_shapes" inherit_id="web_editor.snippet_options">
    <xpath expr="//div"><!-- TODO remove me in master (do not xpath this) --></xpath>
</template>
<template id="snippet_options_image_styles" inherit_id="web_editor.snippet_options">
    <xpath expr="//div"><!-- TODO remove me in master (do not xpath this) --></xpath>
</template>

<!-- Mass Mailing Snippet Options -->
<template id="snippet_options" inherit_id="web_editor.snippet_options" primary="True">
    <!-- =================================================================== -->
    <!-- Modify generic snippet options                                      -->
    <!-- =================================================================== -->

    <xpath expr="//t[@t-set='no_animations']" position="attributes">
        <attribute name="t-value">True</attribute>
    </xpath>

    <!-- Extra shapes -->
    <xpath expr="//div[hasclass('o_we_image_shape')]//we-select-page[hasclass('o_we_basic_shapes')]" position="inside">
        <we-button data-set-img-shape="mass_mailing/basic/circle" data-select-label="Circle"/>
        <we-button data-set-img-shape="mass_mailing/basic/triangle" data-select-label="Triangle"/>
        <we-button data-set-img-shape="mass_mailing/basic/slanted" data-select-label="Slanted"/>
    </xpath>
    <xpath expr="//div[hasclass('o_we_image_shape')]//we-select-page[hasclass('o_we_basic_shapes')]" position="attributes">
        <attribute name="t-if">True</attribute>
    </xpath>

    <!-- Image styles -->
    <!-- TODO review to only modify what's really necessary and potentially -->
    <!-- make the changes in web_editor directly so that website can benefit -->
    <!-- from it. -->
    <xpath expr="//div[hasclass('o_we_image_options')]" position="replace">
        <div data-selector="span.fa, i.fa, img">
            <we-select string="Alignment" data-state-to-first-class="true">
                <we-button data-select-class="float-start" title="Align Left">Left</we-button>
                <we-button data-select-class="mx-auto" title="Align Center">Center</we-button>
                <we-button data-select-class="float-end" title="Align Right">Right</we-button>
            </we-select>

            <t t-call="mass_mailing.snippet_options_border_line_widgets">
                <t t-set="label">Border</t>
                <t t-set="with_bs_class" t-value="True"></t>
            </t>

            <we-input string="Round Corners"
            data-select-style="0" data-css-property="border-radius"
            data-unit="px" data-extra-class="rounded"
            t-att-data-variable="radius_variable"/>

            <we-row string="Padding ⭤">
                <we-input data-select-style="" data-unit="px" data-css-property="padding-left"/>
                <we-input data-select-style="" data-unit="px" data-css-property="padding-right"/>
            </we-row>
            <we-row string="Padding ↕">
                <we-input data-select-style="" data-unit="px" data-css-property="padding-top"/>
                <we-input data-select-style="" data-unit="px" data-css-property="padding-bottom"/>
            </we-row>
        </div>
    </xpath>

    <!-- Transform is _very_ badly supported in mail clients -->
    <xpath expr="//we-button[@data-name='image_transform_opt']" position="replace" />

    <xpath expr="//div[@data-js='ImageTools']" position="attributes">
        <attribute name="data-js">MassMailingImageTools</attribute>
    </xpath>

    <!-- =================================================================== -->
    <!-- Adding mass_mailing specific snippet options                        -->
    <!-- =================================================================== -->

    <xpath expr="." position="inside">

    <!-- Border | Columns -->
    <div data-js="Box"
         data-selector=".row > div"
         data-exclude=".o_mail_wrapper_td, .s_col_no_bgcolor, .s_col_no_bgcolor.row > div, .s_image_gallery .row > div">
        <t t-call="mass_mailing.snippet_options_border_widgets"/>
    </div>
    <div data-js="layout_column"
        data-selector=".o_mail_snippet_general"
        data-target="> *:has(> .row:not(.s_nb_column_fixed)), > .s_allow_columns">
        <we-select string="Columns" data-no-preview="true">
            <we-button data-select-count="0" data-name="zero_cols_opt">None</we-button>
            <we-button data-select-count="1">1</we-button>
            <we-button data-select-count="2">2</we-button>
            <we-button data-select-count="3">3</we-button>
            <we-button data-select-count="4">4</we-button>
            <we-button data-select-count="5">5</we-button>
            <we-button data-select-count="6">6</we-button>
        </we-select>
    </div>

    <!-- Move snippets around -->
    <div data-js="SnippetMove" data-selector=".o_mail_snippet_general">
        <we-button class="fa fa-fw fa-angle-up" data-move-snippet="prev" data-no-preview="true" data-name="move_up_opt"/>
        <we-button class="fa fa-fw fa-angle-down" data-move-snippet="next" data-no-preview="true" data-name="move_down_opt"/>
    </div>
    <div data-js="SnippetMove"
         data-selector=".row:not(.s_col_no_resize) > div"
         data-exclude=".s_showcase .row > div"
         data-name="move_horizontally_opt">
        <we-button class="fa fa-fw fa-angle-left" data-move-snippet="prev" data-no-preview="true" data-name="move_left_opt"/>
        <we-button class="fa fa-fw fa-angle-right" data-move-snippet="next" data-no-preview="true" data-name="move_right_opt"/>
    </div>

    <div id="so_width" data-selector=".s_mail_alert .s_alert, .s_mail_blockquote, .s_mail_text_highlight">
        <we-select string="Width">
            <we-button data-select-class="w-25">25%</we-button>
            <we-button data-select-class="w-50">50%</we-button>
            <we-button data-select-class="w-75">75%</we-button>
            <we-button data-select-class="w-100" data-name="so_width_100">100%</we-button>
        </we-select>
    </div>

    <div id="so_block_align" data-selector=".s_mail_alert .s_alert, .s_mail_blockquote, .s_mail_text_highlight">
        <we-button-group string="Alignment" data-dependencies="!so_width_100">
            <we-button class="fa fa-fw fa-align-left" title="Left" data-select-class="me-auto"/>
            <we-button class="fa fa-fw fa-align-center" title="Center" data-select-class="mx-auto"/>
            <we-button class="fa fa-fw fa-align-right" title="Right" data-select-class="ms-auto"/>
        </we-button-group>
    </div>
    <!-- TODO there is no data-js associated to this but a data-option-name, -->
    <!-- somehow it acts as data-js... it will be reviewed in master. -->
    <div data-option-name="minHeight" data-selector=".o_mail_snippet_general" data-exclude=".o_mail_snippet_general .row > div *">
        <we-button-group string="Height">
            <we-button data-name="minheight_auto_opt" data-select-class="" title="Fit content">Auto</we-button>
            <we-button data-select-class="o_height_400" title="400px">50%</we-button>
            <we-button data-select-class="o_height_800" title="800px">100%</we-button>
        </we-button-group>
    </div>
    <div data-js="mass_mailing_sizing_x"
        data-selector="img, .mv, .col_mv, td, th"
        data-exclude=".o_mail_no_resize, .o_mail_no_options"/>

    <div data-js="table_row"
        data-selector="tr:has(> .row), tr:has(> .col_mv)"
        data-exclude=".o_mail_no_options"
        data-drop-near="tr:has(> .row), tr:has(> .col_mv)"/>

    <div data-js="table_column"
        data-selector=".col>td, .col>th"
        data-exclude=".o_mail_no_options"
        data-drop-near=".col>td, .col>th"/>

    <div data-js="table_column_mv"
        data-selector=".col_mv, td, th"
        data-exclude=".o_mail_no_options"
        data-drop-near=".col_mv, td, th"/>

    <t t-set="mailing_content_selector" t-translation="off">.note-editable > div:not(.o_layout), .note-editable .oe_structure > div, .oe_snippet_body</t>
    <div data-js="content"
        t-att-data-selector="mailing_content_selector"
        data-exclude=".o_mail_no_options"
        data-drop-near="[data-oe-field='body_html']:not(:has(.o_layout)) > *, .oe_structure > *"
        data-drop-in="[data-oe-field='body_html']:not(:has(.o_layout)), .oe_structure"/>

    <t t-set="so_snippet_addition_selector" t-translation="off">.o_mail_snippet_general</t>
    <div id="so_snippet_addition"
        t-att-data-selector="so_snippet_addition_selector"
        data-drop-in=":not(p).oe_structure:not(.oe_structure_solo), :not(.o_mega_menu):not(p)[data-oe-type=html], :not(p).oe_structure.oe_structure_solo:not(:has(> section, > div))"/>

    <t t-set="so_content_addition_selector" t-translation="off">.s_mail_blockquote, .s_mail_alert, .s_rating, .s_hr, .s_mail_text_highlight</t>
    <div id="so_content_addition"
        t-att-data-selector="so_content_addition_selector"
        t-attf-data-drop-near="p, h1, h2, h3, ul, ol, .row > div > img, #{so_content_addition_selector}"
        data-drop-in=".content, nav"/>

    <div data-js="sizing_y"
        data-selector=".o_mail_snippet_general, .o_mail_snippet_general .row > div"
        data-exclude=".o_mail_no_resize, .o_mail_no_options"/>

    <div data-js="sizing_x"
        data-selector=".row > div"
        data-drop-near=".row:not(.s_col_no_resize) > div"
        data-exclude=".o_mail_no_resize, .o_mail_no_options"/>

    <div data-selector=".note-editable .oe_structure > div:not(:has(> .o_mail_snippet_general)),
        .note-editable .oe_structure > div.o_mail_snippet_general,
        .note-editable .oe_structure > div.o_mail_snippet_general .o_cc,
        .note-editable .oe_structure > div.o_mail_snippet_general .btn:not(.btn-link)"
        data-exclude=".o_mail_no_colorpicker, .o_mail_no_options, .s_mail_color_blocks_2, .s_mail_color_blocks_2 .row > div">
        <we-colorpicker string="Background Color"
            data-select-style="true"
            data-no-transparency="true"
            data-css-property="background-color"
            data-color-prefix="bg-"/>
    </div>

    <div data-selector=".s_mail_color_blocks_2 .row > div">
        <we-colorpicker string="Background Color"
            data-select-style="true"
            data-no-transparency="true"
            data-css-property="background-color"
            data-color-prefix="bg-"/>
    </div>
    <!-- Allow customised padding-x on snippets -->
    <div data-selector="[class*='col-lg-'], .s_discount2, .s_text_block, .s_media_list, .s_picture, .s_rating">
        <we-row string="Padding ⭤">
            <we-input data-select-style="" data-unit="px" data-css-property="padding-left"/>
            <we-input data-select-style="" data-unit="px" data-css-property="padding-right"/>
        </we-row>
    </div>

    <!-- Allow changing background images in Masonry and Cover -->
    <t t-call="mass_mailing.snippet_options_background_options">
        <t t-set="selector" t-value="'.s_masonry_block .row > div, .s_cover .oe_img_bg'"/>
        <t t-set="with_images" t-value="True"/>
        <t t-set="with_videos" t-value="false"/>
        <t t-set="with_shapes" t-value="false"/>
    </t>

    <!-- COLOR | .s_three_columns | .s_comparisons | .s_event  -->
    <div data-js="Box"
         data-selector=".s_three_columns .row > div, .s_comparisons .row > div, .s_mail_block_event .row > div"
         data-target=".card-body">
        <we-colorpicker string="Background Color"
            data-select-style="true"
            data-no-transparency="true"
            data-css-property="background-color"
            data-color-prefix="bg-"/>
    </div>
    <!-- BORDER | .s_three_columns | .s_comparisons | .s_event  -->
    <div data-js="Box"
         data-selector=".s_three_columns .row > div, .s_comparisons .row > div, .s_mail_block_event .row > div"
         data-target=".card">
        <t t-call="mass_mailing.snippet_options_border_widgets">
            <t t-set="so_rounded_no_dependencies" t-value="True"/>
        </t>
    </div>
    <!-- COLOR, BORDER | .o_mail_block_discount2 -->
    <div data-js="Box"
         data-selector=".o_mail_block_discount2"
         data-target="table">
        <t t-call="mass_mailing.snippet_options_border_widgets">
        </t>
    </div>

    <!--  Vertical Alignment -->
    <div data-option-name="vAlignment" id="row_valign_snippet_option" data-selector=".s_text_image, .s_image_text, .s_three_columns, s_mail_block_event" data-target=".row">
        <we-button-group string="Vert. Alignment" title="Vertical Alignment">
            <we-button title="Align Top"
                       data-select-class="align-items-start"
                       data-img="/mass_mailing/static/src/img/snippets_options/align_top.svg"/>
            <we-button title="Align Middle"
                       data-select-class="align-items-center"
                       data-img="/mass_mailing/static/src/img/snippets_options/align_middle.svg"/>
            <we-button title="Align Bottom"
                       data-select-class="align-items-end"
                       data-img="/mass_mailing/static/src/img/snippets_options/align_bottom.svg"/>
            <we-button title="Stretch to Equal Height"
                       data-select-class="align-items-stretch"
                       data-img="/mass_mailing/static/src/img/snippets_options/align_stretch.svg"/>
        </we-button-group>
    </div>

    <!-- DESIGN OPTIONS -->
    <div data-js="DesignTab" data-selector="design-options" data-no-check="true">
        <!-- BODY WIDTH -->
        <we-button-group string="Body Width" data-apply-to=".o_mail_wrapper" data-no-preview="true">
            <we-button data-select-class="o_mail_small"
                        data-img="/mass_mailing/static/src/img/snippets_options/content_width_small.svg"
                        title="Small"/>
            <we-button data-select-class="o_mail_regular"
                        data-img="/mass_mailing/static/src/img/snippets_options/content_width_normal.svg"
                        title="Regular"/>
            <we-button data-select-class=""
                        data-img="/mass_mailing/static/src/img/snippets_options/content_width_full.svg"
                        title="Full"/>
        </we-button-group>
        <we-colorpicker string="Background Color"
                        data-apply-to=".o_layout, > div:not(.o_layout)"
                        data-select-style="true"
                        data-no-transparency="true"
                        data-css-property="background-color"
                        data-color-prefix="bg-"/>
        <!-- HEADING 1 -->
        <we-row string="Heading 1" class="o_design_tab_title">
            <we-input data-customize-css-property=""
                        data-css-property="font-size"
                        data-selector-text="h1"
                        data-unit="px"/>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="color"
                            data-selector-text="h1"
                            data-no-transparency="true"
                            data-color-prefix="text-"/>
        </we-row>
        <we-row string="" class="o_we_sublevel_1 o_short_title">
            <we-fontfamilypicker data-selector-text="h1"/>
            <span>​</span> <!-- Separate the select from the buttons (styling) -->
            <we-button title="Bold" class="fa fa-fw fa-bold" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="bolder"
                        data-selector-text="h1"
                        data-css-property="font-weight"/>
            <we-button title="Italic" class="fa fa-fw fa-italic" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="italic"
                        data-selector-text="h1"
                        data-css-property="font-style"/>
            <we-button title="Underline" class="fa fa-fw fa-underline" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="underline"
                        data-selector-text="h1"
                        data-css-property="text-decoration-line"/>
        </we-row>
        <!-- HEADING 2 -->
        <we-row string="Heading 2" class="o_design_tab_title">
            <we-input data-customize-css-property=""
                        data-css-property="font-size"
                        data-selector-text="h2"
                        data-unit="px"/>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="color"
                            data-selector-text="h2"
                            data-no-transparency="true"
                            data-color-prefix="text-"/>
        </we-row>
        <we-row string="" class="o_we_sublevel_1 o_short_title">
            <we-fontfamilypicker data-selector-text="h2"/>
            <span>​</span> <!-- Separate the select from the buttons (styling) -->
            <we-button title="Bold" class="fa fa-fw fa-bold" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="bolder"
                        data-selector-text="h2"
                        data-css-property="font-weight"/>
            <we-button title="Italic" class="fa fa-fw fa-italic" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="italic"
                        data-selector-text="h2"
                        data-css-property="font-style"/>
            <we-button title="Underline" class="fa fa-fw fa-underline" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="underline"
                        data-selector-text="h2"
                        data-css-property="text-decoration-line"/>
        </we-row>
        <!-- HEADING 3 -->
        <we-row string="Heading 3" class="o_design_tab_title">
            <we-input data-customize-css-property=""
                        data-css-property="font-size"
                        data-selector-text="h3"
                        data-unit="px"/>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="color"
                            data-selector-text="h3"
                            data-no-transparency="true"
                            data-color-prefix="text-"/>
        </we-row>
        <we-row string="" class="o_we_sublevel_1 o_short_title">
            <we-fontfamilypicker data-selector-text="h3"/>
            <span>​</span> <!-- Separate the select from the buttons (styling) -->
            <we-button title="Bold" class="fa fa-fw fa-bold" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="bolder"
                        data-selector-text="h3"
                        data-css-property="font-weight"/>
            <we-button title="Italic" class="fa fa-fw fa-italic" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="italic"
                        data-selector-text="h3"
                        data-css-property="font-style"/>
            <we-button title="Underline" class="fa fa-fw fa-underline" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="underline"
                        data-selector-text="h3"
                        data-css-property="text-decoration-line"/>
        </we-row>
        <!-- TEXT -->
        <we-row string="Text" class="o_design_tab_title">
            <we-input data-customize-css-property=""
                        data-css-property="font-size"
                        data-selector-text="p, p > *, li, li > *"
                        data-unit="px"/>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="color"
                            data-selector-text="p, p > *, li, li > *"
                            data-no-transparency="true"
                            data-color-prefix="text-"/>
        </we-row>
        <we-row string="" class="o_we_sublevel_1 o_short_title">
            <we-fontfamilypicker data-selector-text="p, p > *, li, li > *"/>
            <span>​</span> <!-- Separate the select from the buttons (styling) -->
            <we-button title="Bold" class="fa fa-fw fa-bold" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="bolder"
                        data-selector-text="p, p > *, li, li > *"
                        data-css-property="font-weight"/>
            <we-button title="Italic" class="fa fa-fw fa-italic" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="italic"
                        data-selector-text="p, p > *, li, li > *"
                        data-css-property="font-style"/>
            <we-button title="Underline" class="fa fa-fw fa-underline" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="underline"
                        data-selector-text="p, p > *, li, li > *"
                        data-css-property="text-decoration-line"/>
        </we-row>
        <!-- LINKS -->
        <we-row string="Links" class="o_design_tab_title">
            <we-colorpicker data-customize-css-property=""
                            data-css-property="color"
                            data-selector-text="a:not(.btn), a.btn.btn-link"
                            data-no-transparency="true"
                            data-color-prefix="text-"/>
            <we-button title="Underline" class="fa fa-fw fa-underline" data-no-preview="true" data-toggle="true"
                        data-customize-css-property="underline"
                        data-selector-text="a:not(.btn), a.btn.btn-link"
                        data-css-property="text-decoration-line"/>
        </we-row>
        <!-- PRIMARY BUTTONS -->
        <we-row string="Primary Buttons" class="o_design_tab_title">
            <we-input data-customize-css-property=""
                        data-css-property="font-size"
                        data-selector-text="a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary"
                        data-unit="px"/>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="color"
                            data-selector-text="a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary"
                            data-no-transparency="true"
                            data-color-prefix="text-"/>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="background-color"
                            data-selector-text="a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary"
                            data-no-transparency="true"
                            data-color-prefix="bg-"/>
        </we-row>
        <we-select string="Size" class="o_we_sublevel_1" data-selector-text="a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary">
            <we-button data-apply-button-size="btn-sm">Small</we-button>
            <we-button data-apply-button-size="btn-md">Medium</we-button>
            <we-button data-apply-button-size="btn-lg">Large</we-button>
        </we-select>
        <we-row string="Border" class="o_we_sublevel_1">
            <we-input data-customize-css-property=""
                        data-css-property="border-width"
                        data-selector-text="a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary"
                        data-unit="px"/>
            <we-select data-selector-text="a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary" data-css-property="border-style">
                <we-button title="Solid" data-customize-css-property="solid">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: solid;"/>
                </we-button>
                <we-button title="Dashed" data-customize-css-property="dashed">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: dashed;"/>
                </we-button>
                <we-button title="Dotted" data-customize-css-property="dotted">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: dotted;"/>
                </we-button>
                <we-button title="Double" data-customize-css-property="double">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: double; border-left: none; border-right: none;"/>
                </we-button>
            </we-select>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="border-color"
                            data-selector-text="a.btn.btn-primary, a.btn.btn-outline-primary, a.btn.btn-fill-primary"
                            data-no-transparency="true"
                            data-color-prefix="border-"/>
        </we-row>
        <!-- SECONDARY BUTTONS -->
        <we-row string="Secondary Buttons" class="o_design_tab_title">
            <we-input data-customize-css-property=""
                        data-css-property="font-size"
                        data-selector-text="a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary"
                        data-unit="px"/>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="color"
                            data-selector-text="a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary"
                            data-no-transparency="true"
                            data-color-prefix="text-"/>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="background-color"
                            data-selector-text="a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary"
                            data-no-transparency="true"
                            data-color-prefix="bg-"/>
        </we-row>
        <we-select string="Size" class="o_we_sublevel_1" data-selector-text="a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary">
            <we-button data-apply-button-size="btn-sm">Small</we-button>
            <we-button data-apply-button-size="btn-md">Medium</we-button>
            <we-button data-apply-button-size="btn-lg">Large</we-button>
        </we-select>
        <we-row string="Border" class="o_we_sublevel_1">
            <we-input data-customize-css-property=""
                        data-css-property="border-width"
                        data-selector-text="a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary"
                        data-unit="px"/>
            <we-select data-selector-text="a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary" data-css-property="border-style">
                <we-button title="Solid" data-customize-css-property="solid">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: solid;"/>
                </we-button>
                <we-button title="Dashed" data-customize-css-property="dashed">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: dashed;"/>
                </we-button>
                <we-button title="Dotted" data-customize-css-property="dotted">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: dotted;"/>
                </we-button>
                <we-button title="Double" data-customize-css-property="double">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: double; border-left: none; border-right: none;"/>
                </we-button>
            </we-select>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="border-color"
                            data-selector-text="a.btn.btn-secondary, a.btn.btn-outline-secondary, a.btn.btn-fill-secondary"
                            data-no-transparency="true"
                            data-color-prefix="border-"/>
        </we-row>
        <!-- SEPARATORS -->
        <we-row string="Separators" class="o_design_tab_title">
            <we-input data-customize-css-property=""
                    data-css-property="border-top-width"
                    data-selector-text="hr"
                    data-unit="px"/>
            <we-select data-selector-text="hr" data-css-property="border-top-style">
                <we-button title="Solid" data-customize-css-property="solid">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: solid;"/>
                </we-button>
                <we-button title="Dashed" data-customize-css-property="dashed">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: dashed;"/>
                </we-button>
                <we-button title="Dotted" data-customize-css-property="dotted">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: dotted;"/>
                </we-button>
                <we-button title="Double" data-customize-css-property="double">
                    <div class="o_we_fake_img_item o_we_border_preview" style="border-style: double; border-left: none; border-right: none;"/>
                </we-button>
            </we-select>
            <we-colorpicker data-customize-css-property=""
                            data-css-property="border-top-color"
                            data-selector-text="hr"
                            data-no-transparency="true"
                            data-color-prefix="border-"/>
        </we-row>
        <we-select string="Width" class="o_we_sublevel_1" data-selector-text="hr" data-css-property="width">
            <we-button data-customize-css-property="25%">25%</we-button>
            <we-button data-customize-css-property="50%">50%</we-button>
            <we-button data-customize-css-property="75%">75%</we-button>
            <we-button data-customize-css-property="100%">100%</we-button>
        </we-select>
    </div>

    </xpath>
</template>

</odoo>

```

## File: views\themes_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Theme "Basic" template -->
    <template id="theme_basic_template">
        <div class="o_mail_no_options">
            <p><br/></p>
            <p><br/></p>
            <p>
                <br/>
                <a href="/unsubscribe_from_list">Unsubscribe</a>
            </p>
        </div>
    </template>

    <template id="theme_empty_template">
    </template>

    <!-- Default Theme -->
    <template id="theme_default_template">
        <style id="design-element">
            h2 {
                font-weight: bolder;
            }
            p, p > *, li, li > * {
                color: #6c757d;
            }
            hr {
                border-top-color: #ced4da !important;
            }
        </style>
        <div class="s_header_logo o_mail_block_header_logo o_mail_snippet_general pt16 pb16">
            <div class="container">
                <div class="row">
                    <div class="col-lg-4"/>
                    <div class="col-lg-4" style="text-align: center;">
                        <a t-att-href="(company_id.website) or '#'" style="text-decoration:none;" target="_blank">
                            <img border="0" src="/mass_mailing/static/src/img/theme_default/s_default_image_header_logo.png" style="height:auto; max-width:100%;" width="180" class="img-fluid"/>
                        </a>
                    </div>
                    <div class="col-lg-4" style="text-align: right;"/>
                </div>
            </div>
        </div>
        <div class="s_text_block o_mail_snippet_general pt40 pb16" style="padding-left: 15px; padding-right: 15px;">
            <div class="container s_allow_columns">
                <h2>Thank you for joining us!</h2>
                <p><br/>We want to take this opportunity to welcome you to our ever-growing community!
                <br/>Your platform is ready for work, it will help you reduce the costs of digital signatures, attract new customers and increase sales.</p>
                <p><img src="/mass_mailing/static/src/img/theme_default/signature.png" style="width:125px; margin-top:8px;margin-bottom:-25px;" alt="Signature" class="img-fluid"/></p>
                <p>Michael Fletcher<br/>
                   <span style="font-size: 12px; font-weight: bolder;">Customer Service</span>
                </p>
                <p style="text-align: center;">
                    <a role="button" href="#" class="btn btn-primary">LOGIN</a>
                </p>
            </div>
        </div>
        <div class="s_hr pt16 pb16" data-snippet="s_hr" data-name="Separator">
            <hr class="s_hr_1px s_hr_solid"/>
        </div>
        <t t-call="mass_mailing.s_mail_block_footer_social_left"/>
    </template>
</odoo>

```

## File: views\utm_campaign_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="utm_campaign_view_form">
        <field name="name">utm.campaign.view.form</field>
        <field name="model">utm.campaign</field>
        <field name="inherit_id" ref="utm.utm_campaign_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <field name="is_mailing_campaign_activated" invisible="1"/>
                <button name="%(action_create_mass_mailings_from_campaign)d"
                    type="action" class="oe_highlight" attrs="{'invisible': [('is_mailing_campaign_activated', '=', False)]}"
                    groups="mass_mailing.group_mass_mailing_user" string="Send Mailing"/>
            </xpath>
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button name="%(action_view_mass_mailings_from_campaign)d"
                    type="action" class="oe_stat_button order-9" icon="fa-envelope-o"
                    attrs="{'invisible': ['|', ('mailing_mail_count', '=', 0), ('is_mailing_campaign_activated', '=', False)]}"
                    groups="mass_mailing.group_mass_mailing_user">
                    <field name="mailing_mail_count" widget="statinfo" string="Mailings"/>
                </button>
            </xpath>
            <xpath expr="//notebook" position="inside">
                <page string="Mailings" name="mailings"
                    attrs="{'invisible': ['|', ('mailing_mail_count', '=', 0), ('is_mailing_campaign_activated', '=', False)]}"
                    groups="mass_mailing.group_mass_mailing_user">
                    <field name="mailing_mail_ids" nolabel="1">
                        <tree>
                            <field name="calendar_date" string="Date"/>
                            <field name="subject" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                            <field name="mailing_model_id" string="Recipients" optional="hide"/>
                            <field name="user_id" widget="many2one_avatar_user"/>
                            <field name="ab_testing_enabled" string="A/B Test"
                                groups="mass_mailing.group_mass_mailing_campaign"
                                attrs="{'column_invisible': [('parent.ab_testing_mailings_count', '=', 0)]}"/>
                            <field name="campaign_id" string="Campaign" groups="mass_mailing.group_mass_mailing_campaign" optional="hide"/>
                            <field name="received_ratio" string="Delivered (%)" avg="Average of Delivered"/>
                            <field name="opened_ratio" string="Opened (%)" avg="Average of Opened"/>
                            <field name="bounced_ratio" string="Bounced (%)" optional="hide" avg="Average of Bounced"/>
                            <field name="clicks_ratio" string="Clicked (%)" avg="Average of Clicked"/>
                            <field name="replied_ratio" string="Replied (%)" avg="Average of Replied"/>
                            <field name="state" decoration-info="state in ('draft', 'in_queue')" decoration-success="state in ('sending', 'done')" widget="badge"/>
                            <button name="action_duplicate" type="object" string="Duplicate"/>
                        </tree>
                    </field>
                </page>
            </xpath>
            <xpath expr="//notebook" position="after">
                <field name="ab_testing_mailings_count" invisible="1"/>
                <group name="ab_test_group" groups="mass_mailing.group_mass_mailing_campaign" attrs="{'invisible': [('ab_testing_mailings_count', '=', 0)]}">
                    <group string="A/B Test">
                        <field name="ab_testing_completed" invisible="1"/>
                        <field name="ab_testing_winner_selection" attrs="{'readonly': [('ab_testing_completed', '=', True)]}"/>
                        <field name="ab_testing_schedule_datetime"
                            attrs="{'invisible': [('ab_testing_winner_selection', '=', 'manual')], 'readonly': [('ab_testing_completed', '=', True)]}"/>
                    </group>
                </group>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="utm_campaign_view_kanban">
        <field name="name">utm.campaign.view.kanban</field>
        <field name="model">utm.campaign</field>
        <field name="inherit_id" ref="utm.utm_campaign_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="mailing_mail_ids" groups="mass_mailing.group_mass_mailing_user"/>
                <field name="is_mailing_campaign_activated"/>
            </xpath>
            <xpath expr="//ul[@id='o_utm_actions']">
                <a name="%(action_view_mass_mailings_from_campaign)d" type="action"
                    t-attf-class="oe_mailings #{record.mailing_mail_ids.raw_value.length === 0 ? 'text-muted' : ''}"
                    t-if="record.is_mailing_campaign_activated.raw_value"
                    groups="mass_mailing.group_mass_mailing_user">
                    <t t-out="record.mailing_mail_ids.raw_value.length"/> Mailings
                </a>
            </xpath>
        </field>
    </record>

    <record id="action_view_utm_campaigns" model="ir.actions.act_window">
        <field name="name">Campaigns</field>
        <field name="res_model">utm.campaign</field>
        <field name="view_mode">kanban,tree,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            Create a mailing campaign
            </p><p>
            Campaigns are the perfect tool to track results across multiple mailings.
            </p>
        </field>
        <field name="domain">[('is_auto_campaign', '=', False)]</field>
    </record>
</odoo>

```

## File: views\snippets\s_alert.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="s_alert" name="Alert">
    <div class="s_mail_alert o_mail_snippet_general pt16 pb16 mx-auto">
        <div class="container">
            <div class="row">
                <div class="col-lg-12">
                    <div class="s_alert s_alert_md s_alert_info w-100" style="background-color: rgb(209 236 241); border-width: 1px !important; border-color: rgb(190 229 235) !important;">
                        <div class="s_alert_icon" valign="top">
                            <i class="fa fa-2x fa-info-circle" style="color: rgb(12 84 96);"/>
                        </div>
                        <div class="s_alert_content">
                            <h3><span style="color: rgb(12 84 96); font-size: 16px; font-weight: bolder;">Explain the benefits you offer</span></h3>
                            <p><font style="color: rgb(12 84 96);">Don't write about products or services here, write about solutions.</font></p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_alert_options" inherit_id="mass_mailing.snippet_options">
    <!-- Keep those options in separate xpath for options order -->
    <xpath expr="//div[@id='so_width']" position="after">
        <div data-selector=".s_mail_alert .s_alert">
            <we-select string="Size">
                <we-button data-select-class="s_alert_sm">Small</we-button>
                <we-button data-select-class="s_alert_md">Medium</we-button>
                <we-button data-select-class="s_alert_lg">Large</we-button>
            </we-select>
            <we-colorpicker string="Background Color" data-name="alert_colorpicker_opt"
                data-select-style="true"
                data-css-property="background-color"
                data-color-prefix="alert-"/>
        </div>
        <div data-selector=".s_mail_alert .s_alert">
            <t t-call="mass_mailing.snippet_options_border_widgets">
                <t t-set="so_rounded_no_dependencies" t-value="True"/>
            </t>
        </div>
    </xpath>
</template>

<!-- Assets -->
<record id="mass_mailing.s_alert_001_scss" model="ir.asset">
    <field name="name">Alert 001 SCSS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">mass_mailing/static/src/snippets/s_alert/000.scss</field>
</record>

</odoo>

```

## File: views\snippets\s_blockquote.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_blockquote" name="Blockquote">
    <div class="s_mail_blockquote blockquote o_mail_snippet_general o_cc o_cc2 pt16 pb16 mb-0 w-100">
        <div class="container">
            <div class="row">
                <div class="col-lg-1 pb80" style="text-align: right;">
                    <i class="fa fa-quote-left"/>
                </div>
                <div class="col-lg-10">
                    <div>
                        <p><i>Write a quote here from one of your customers. Quotes are a great way to build confidence in your products or services.</i></p>
                        <div>
                            <img src="/web_editor/image_shape/mass_mailing.s_company_team_default_image_2/mass_mailing/basic/circle.svg" style="width: 40px" class="img me-2" data-shape="mass_mailing/basic/circle" data-file-name="team_member_2-circle.svg" data-shape-colors=";;;;" data-original-mimetype="image/png"/>
                            <p style="font-weight: bolder; font-size: 14px;"><b>John DOE</b> &#8226; CEO of MyCompany</p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-1 pt80" style="text-align: left;">
                    <i class="fa fa-quote-right"/>
                </div>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_call_to_action.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_call_to_action" name="Call to Action">
    <div class="s_call_to_action o_mail_snippet_general o_cc o_cc3 pt48 pb24">
        <div class="container">
            <div class="row">
                <div class="col-lg-9">
                    <h3><span style="font-weight: bolder;">50,000+ companies</span> run Odoo.</h3>
                    <p>Join us and make your company a better place.</p>
                </div>
                <div class="col-lg-3" style="text-align: center;">
                    <a href="#" class="btn btn-primary btn-lg">Contact us</a>
                </div>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_color_blocks_2.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="s_color_blocks_2" name="Big Boxes">
        <div class="s_mail_color_blocks_2 o_mail_snippet_general">
            <div class="container">
                <div class="row">
                    <div class="col-lg-6 o_cc o_cc3 pt32 pb32">
                        <i class="fa fa-shield fa-5x m-3 mx-auto d-block"/>
                        <h2 style="text-align: center;">A color block</h2>
                        <p style="text-align: center;">Color blocks are a simple and effective way to <b>present and highlight your content</b>. Choose an image or a color for the background. You can even resize and duplicate the blocks to create your own layout. Add images or icons to customize the blocks.</p>
                        <p style="text-align: center;"><a href="#" class="btn btn-primary">More Details</a></p>
                    </div>
                    <div class="col-lg-6 o_cc o_cc5 pt32 pb32">
                        <i class="fa fa-cube fa-5x m-3 mx-auto d-block"/>
                        <h2 style="text-align: center;">Another color block</h2>
                        <p style="text-align: center;">Color blocks are a simple and effective way to <b>present and highlight your content</b>. Choose an image or a color for the background. You can even resize and duplicate the blocks to create your own layout. Add images or icons to customize the blocks.</p>
                        <p style="text-align: center;"><a href="#" class="btn btn-primary">More Details</a></p>
                    </div>
                </div>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\snippets\s_company_team.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_company_team" name="Team">
    <div class="s_company_team o_mail_snippet_general">
        <div class="container">
            <div class="row">
                <div class="col-lg-6 pt24 pb24">
                    <div class="container">
                        <div class="row">
                            <div class="col-lg-3 pb16 px-0">
                                <img src="/web_editor/image_shape/mass_mailing.s_company_team_default_image_1/mass_mailing/basic/circle.svg" class="img-fluid mx-auto" data-shape="mass_mailing/basic/circle" data-file-name="team_member_1-circle.svg" data-shape-colors=";;;;" data-original-mimetype="image/png"/>
                            </div>
                            <div class="col-lg-9">
                                <h3>Tony Fred, CEO</h3>
                                <p>Founder and chief visionary, Tony is the driving force behind the company. He loves to keep his hands full by participating in the development of the software, marketing, and customer experience strategies.</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 pt24 pb24">
                    <div class="container">
                        <div class="row">
                            <div class="col-lg-3 px-0 pb16">
                                <img src="/web_editor/image_shape/mass_mailing.s_company_team_default_image_2/mass_mailing/basic/circle.svg" class="img-fluid mx-auto" data-shape="mass_mailing/basic/circle" data-file-name="team_member_2-circle.svg" data-shape-colors=";;;;" data-original-mimetype="image/png"/>
                            </div>
                            <div class="col-lg-9">
                                <h3>Mich Stark, COO</h3>
                                <p>Mich loves taking on challenges. With his multi-year experience as Commercial Director in the software industry, Mich has helped the company to get where it is today. Mich is among the best minds.</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 pt24 pb24">
                    <div class="container">
                        <div class="row">
                            <div class="col-lg-3 px-0 pb16">
                                <img src="/web_editor/image_shape/mass_mailing.s_company_team_default_image_3/mass_mailing/basic/circle.svg" class="img-fluid mx-auto" data-shape="mass_mailing/basic/circle" data-file-name="team_member_3-circle.svg" data-shape-colors=";;;;" data-original-mimetype="image/png"/>
                            </div>
                            <div class="col-lg-9">
                                <h3>Aline Turner, CTO</h3>
                                <p>Aline is one of the iconic people in life who can say they love what they do. She mentors 100+ in-house developers and looks after the community of thousands of developers.</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 pt24 pb24">
                    <div class="container">
                        <div class="row">
                            <div class="col-lg-3 px-0 pb16">
                                <img src="/web_editor/image_shape/mass_mailing.s_company_team_default_image_4/mass_mailing/basic/circle.svg" class="img-fluid mx-auto" data-shape="mass_mailing/basic/circle" data-file-name="team_member_4-circle.svg" data-shape-colors=";;;;" data-original-mimetype="image/png"/>
                            </div>
                            <div class="col-lg-9">
                                <h3>Iris Joe, CFO</h3>
                                <p>Iris, with her international experience, helps us easily understand the numbers and improves them. She is determined to drive success and delivers her professional acumen to bring the company to the next level.</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_comparisons.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_comparisons" name="Comparisons">
    <div class="s_comparisons o_mail_snippet_general pt32 pb32">
        <div class="container">
            <div class="row">
                <div class="col-lg-6 s_col_no_bgcolor pt32 pb16">
                    <div class="card o_cc o_cc2">
                        <div class="card-header"><h2 style="text-align: center;"><span style="font-weight: bolder;">DEFAULT</span></h2></div>
                        <div class="bg-white pt24 pb24">
                            <ul class="list-group list-group-flush">
                                <li class="list-group-item">
                                    <h3 class="card-title" style="text-align: center;">$8</h3>
                                    <p class="card-title" style="text-align: center;"><span style="font-size: 11px">user / month (billed annually)</span></p>
                                </li>
                                <li class="list-group-item"><p style="text-align: center;">Basic features</p></li>
                                <li class="list-group-item"><p style="text-align: center;">Basic management</p></li>
                                <li class="list-group-item"><p style="text-align: center;">No customization</p></li>
                                <li class="list-group-item"><p style="text-align: center;">No support</p></li>
                            </ul>
                        </div>
                        <div class="card-footer" style="text-align: center;">
                            <a href="#" class="btn btn-primary">More</a>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 s_col_no_bgcolor pt32 pb16">
                    <div class="card o_cc o_cc3">
                        <div class="card-header"><h2 style="text-align: center;"><span style="font-weight: bolder;">PRO</span></h2></div>
                        <div class="bg-white pt24 pb24">
                            <ul class="list-group list-group-flush">
                                <li class="list-group-item">
                                    <h3 class="card-title" style="text-align: center;">$18</h3>
                                    <p class="card-title" style="text-align: center;"><span style="font-size: 11px">user / month (billed annually)</span></p>
                                </li>
                                <li class="list-group-item">
                                    <p style="text-align: center;"><span style="font-weight:bolder">Advanced</span>
                                    features</p>
                                </li>
                                <li class="list-group-item">
                                    <p style="text-align: center;"><span style="font-weight:bolder">Total</span>
                                    management</p>
                                </li>
                                <li class="list-group-item">
                                    <p style="text-align: center;"><span style="font-weight:bolder">Fully customizable</span></p>
                                </li>
                                <li class="list-group-item">
                                    <p style="text-align: center;"><span style="font-weight:bolder">24/7 Support</span></p>
                                </li>
                            </ul>
                        </div>
                        <div class="card-footer" style="text-align: center;">
                            <a href="#" class="btn btn-primary">More</a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_coupon_code.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="s_coupon_code" name="Promo Code">
        <div class="s_discount2 o_mail_block_discount2 o_mail_snippet_general pt32 pb32" style="padding-left: 15px; padding-right: 15px;">
            <h2 style="text-align: center;"><span style="font-weight: bolder;">GET $20 OFF</span></h2>
            <p style="text-align: center;">
                Here's your coupon code - but hurry! Ends 9/28
            </p>
            <table border="0" cellpadding="0" cellspacing="0" align="center" class="border" style="border-collapse:collapse; mso-table-lspace:0pt; mso-table-rspace:0pt;">
                <tr>
                    <td width="50" height="50" align="center" class="o_mail_no_resize mx-auto o_cc o_cc3" style="width:50px!important; min-width: 50px; max-width:5.6rem; text-align: center;"><i class="fa fa-2x fa-ticket"></i></td>
                    <td width="200" height="50" align="center" class="o_cc" style="font-size: 15px; line-height: 22px; font-weight: 700; min-width: 150px; width: 200px; text-align: center;"><p class="mb0">ENDOFSUMMER20</p></td>
                </tr>
            </table>
            <br/>
            <p style="text-align:center;">
                <a role="button" href="#" class="btn btn-primary">Use now</a>
            </p>
        </div>
    </template>
</odoo>

```

## File: views\snippets\s_cover.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="s_cover" name="Cover">
        <div class="s_cover o_mail_snippet_general">
            <div class="container">
                <div class="row">
                    <div class="col-lg-12 oe_img_bg  pt56 pb48" style="background-image: url('/web/image/mass_mailing.s_cover_default_image');">
                        <h1 style="text-align: center;"><font style="font-size: 62px; font-weight: bold;">Catchy Headline</font></h1>
                        <p class="lead" style="text-align: center;">Write one or two paragraphs describing your product, services or a specific feature.<br/> To be successful your content needs to be useful to your readers.</p>
                    </div>
                </div>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\snippets\s_event.xml

```xml
<odoo>

    <template id="s_event" name="Event">
        <div class="s_mail_block_event o_mail_snippet_general bg-100">
            <div class="container">
                <div class="row align-items-center">
                    <div class="s_col_no_bgcolor col-lg-6 pt32 pb32">
                        <div class="card bg-white h-100 rounded" style="border-radius: 5px !important; border-color: rgb(233, 236, 239) !important;">
                            <img class="card-img-top" src="/web/image/mass_mailing.s_event_default_image_1" alt=""/>
                            <div class="card-body">
                                <h3 class="card-title">
                                    <font style="font-size: 18px;">Event One</font>
                                </h3>
                                <p><font class="text-o-color-1">25 September 2022 - 4:30 PM</font></p>
                                <p>London, United Kingdom</p>
                                <p>
                                    <a href="#" target="_blank" class="btn btn-primary">Register Now</a>
                                </p>
                            </div>
                        </div>
                    </div>
                    <div class="s_col_no_bgcolor col-lg-6 pt32 pb32">
                        <div class="card bg-white h-100 rounded" style="border-radius: 5px !important; border-color: rgb(233, 236, 239) !important;">
                            <img class="card-img-top" src="/web/image/mass_mailing.s_event_default_image_2" alt=""/>
                            <div class="card-body">
                                <h3 class="card-title">
                                    <font style="font-size: 18px">Event Two</font>
                                </h3>
                                <p><font class="text-o-color-1">26 September 2022 - 1:30 PM</font></p>
                                <p>London, United Kingdom</p>
                                <p>
                                    <a href="#" target="_blank" class="btn btn-primary">Register Now</a>
                                </p>
                            </div>
                        </div>
                    </div>

                </div>
            </div>
        </div>
    </template>

</odoo>

```

## File: views\snippets\s_features.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_features" name="Features">
    <div class="s_features o_mail_snippet_general">
        <div class="container">
            <div class="row">
                <div class="col-lg-4 o_cc pt16 pb16" style="text-align: center;">
                    <i class="fa fa-3x fa-gear bg-primary" style="padding: 10px; border-radius: 3px !important;"></i>
                    <br/>
                    <br/>
                    <h3>First Feature</h3>
                    <p>Tell what's the value for the customer for this feature.</p>
                </div>
                <div class="col-lg-4 o_cc pt16 pb16" style="text-align: center;">
                    <i class="fa fa-3x fa-photo bg-o-color-5" style="padding: 10px; border-radius: 3px !important;"></i>
                    <br/>
                    <br/>
                    <h3>Second Feature</h3>
                    <p>Write what the customer would like to know, not what you want to show.</p>
                </div>
                <div class="col-lg-4 o_cc pt16 pb16" style="text-align: center;">
                    <i class="fa fa-3x fa-leaf bg-secondary" style="padding: 10px; border-radius: 3px !important;"></i>
                    <br/>
                    <br/>
                    <h3>Third Feature</h3>
                    <p>A small explanation of this great feature, in clear words.</p>
                </div>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_features_grid.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_features_grid" name="Features Grid">
    <div class="s_mail_features_grid o_mail_snippet_general pt48 pb24">
        <div class="container">
            <div class="row">
                <div class="col-lg-6 o_cc">
                    <div class="container">
                        <div class="row g-0">
                            <div class="col-lg-12 pb24">
                                <h2>First list of Features</h2>
                                <p>Add a great slogan.</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0" style="text-align: center;">
                                <i class="fa fa-2x fa-font-awesome bg-primary" style="padding: 10px; border-radius: 50px !important;"></i>
                            </div>
                            <div class="col-lg-10 pb16">
                                <h3>Change Icons</h3>
                                <p>Double click an icon to replace it with one of your choice.</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0" style="text-align: center;">
                                <i class="fa fa-2x fa-files-o bg-primary" style="padding: 10px; border-radius: 50px !important;"></i>
                            </div>
                            <div class="col-lg-10 pb16">
                                <h3>Duplicate</h3>
                                <p>Duplicate blocks and columns to add more features.</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0" style="text-align: center;">
                                <i class="fa fa-2x fa-trash bg-primary" style="padding: 10px; border-radius: 50px !important;"></i>
                            </div>
                            <div class="col-lg-10 pb16">
                                <h3>Delete Blocks</h3>
                                <p>Select and delete blocks to remove features.</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 o_cc">
                    <div class="container">
                        <div class="row g-0">
                            <div class="col-lg-12 pb24">
                                <h2>Second list of Features</h2>
                                <p>Add a great slogan.</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0" style="text-align: center;">
                                <i class="fa fa-2x fa-magic bg-secondary" style="padding: 10px; border-radius: 3px !important;"></i>
                            </div>
                            <div class="col-lg-10 pb16">
                                <h3>Great Value</h3>
                                <p>Turn every feature into a benefit for your reader.</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0" style="text-align: center;">
                                <i class="fa fa-2x fa-eyedropper bg-secondary" style="padding: 10px; border-radius: 3px !important;"></i>
                            </div>
                            <div class="col-lg-10 pb16">
                                <h3>Edit Styles</h3>
                                <p>You can edit colors and backgrounds to highlight features.</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0" style="text-align: center;">
                                <i class="fa fa-2x fa-picture-o bg-secondary" style="padding: 10px; border-radius: 3px !important;"></i>
                            </div>
                            <div class="col-lg-10 pb16">
                                <h3>Sample Icons</h3>
                                <p>All these icons are completely free for commercial use.</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<record id="mass_mailing.s_features_grid_000_scss" model="ir.asset">
    <field name="name">Features grid 000 SCSS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">mass_mailing/static/src/snippets/s_features_grid/000.scss</field>
</record>

</odoo>

```

## File: views\snippets\s_hr.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_hr" name="Separator" inherit_id="web_editor.s_hr" primary="True">
    <xpath expr="//div[hasclass('s_hr')]" position="attributes">
        <attribute name="class" add="o_mail_snippet_general pt16 pb16" remove="pt32 pb32" separator=" "/>
    </xpath>
    <xpath expr="//hr" position="attributes">
        <attribute name="class" remove="w-100 mx-auto" separator=" "/>
    </xpath>
</template>

<template id="s_hr_options" inherit_id="mass_mailing.snippet_options">
    <xpath expr="." position="inside">
        <div data-selector=".s_hr" data-target="hr">
            <t t-call="mass_mailing.snippet_options_border_line_widgets">
                <t t-set="label">Border</t>
                <t t-set="direction" t-value="'top'"/>
            </t>
            <we-select string="Width">
                <we-button data-select-class="w-25">25%</we-button>
                <we-button data-select-class="w-50">50%</we-button>
                <we-button data-select-class="w-75">75%</we-button>
                <we-button data-select-class="w-100" data-name="so_width_100">100%</we-button>
            </we-select>
            <we-button-group string="Alignment" data-dependencies="!so_width_100">
                <we-button class="fa fa-fw fa-align-left" title="Left" data-select-class="me-auto"/>
                <we-button class="fa fa-fw fa-align-center" title="Center" data-select-class="mx-auto"/>
                <we-button class="fa fa-fw fa-align-right" title="Right" data-select-class="ms-auto"/>
            </we-button-group>
        </div>
    </xpath>
</template>

<record id="mass_mailing.s_hr_000_scss" model="ir.asset">
    <field name="name">Hr 000 SCSS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">mass_mailing/static/src/snippets/s_hr/000.scss</field>
</record>

</odoo>

```

## File: views\snippets\s_image_text.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_image_text" name="Image - Text">
    <div class="s_text_image o_mail_snippet_general pt32 pb32">
        <div class="container">
            <div class="row align-items-center">
                <div class="col-lg-6 o_cc px-0">
                    <img src="/web/image/mass_mailing.s_image_text_default_image" class="img w-100" />
                </div>
                <div class="col-lg-6 o_cc pt16 pb16">
                    <h3>Omnichannel sales</h3>
                    <p style="text-align: justify;">Get your inside sales (CRM) fully integrated with online sales (eCommerce), in-store sales (Point of Sale) and marketplaces like eBay and Amazon.</p>
                    <div style="text-align: left;">
                        <a href="#" class="btn btn-link">Read More</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_masonry_block.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<!-- Template -->
<template id="s_masonry_block" name="Masonry">
    <div class="s_masonry_block o_mail_snippet_general" data-vcss="001">
        <div class="container text-white">
            <t t-call="mass_mailing.s_masonry_block_default_template"/>
        </div>
    </div>
</template>

<!-- Templates -->
<template id="s_masonry_block_default_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-6 oe_img_bg text-center pb224 pt224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_1);">
            <p><br/></p>
        </div>
        <div class="col-lg-6 s_col_no_resize o_masonry_grid_container">
            <div class="row h-100">
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc3" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc4" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_masonry_block_reversed_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-6 s_col_no_resize o_masonry_grid_container">
            <div class="row h-100">
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc3" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc4" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
            </div>
        </div>
        <div class="col-lg-6 oe_img_bg text-center pb224 pt224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_1);">
            <p><br/></p>
        </div>
    </div>
</template>

<template id="s_masonry_block_images_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-6 oe_img_bg text-center pb224 pt224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_1);">
            <p><br/></p>
        </div>
        <div class="col-lg-6 oe_img_bg text-center pb224 pt224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_2);">
            <p><br/></p>
        </div>
    </div>
</template>

<template id="s_masonry_block_image_texts_image_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-3 oe_img_bg text-center pt224 pb224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_2);">
            <p><br/></p>
        </div>
        <div class="col-lg-3 s_col_no_resize o_masonry_grid_container">
            <div class="row h-100">
                <div class="col-lg-12 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-12 pt24 pb8 text-center o_cc o_cc4" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
            </div>
        </div>
        <div class="col-lg-6 oe_img_bg text-center pt224 pb224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_1);">
            <p><br/></p>
        </div>
    </div>
</template>

<template id="s_masonry_block_mosaic_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-6 s_col_no_resize o_masonry_grid_container">
            <div class="row">
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc3" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
            </div>
            <div class="row">
                <div class="col-lg-12 oe_img_bg text-center pt224 pb224" data-name="Block"
                    style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_1);">
                    <p><br/></p>
                </div>
            </div>
        </div>
        <div class="col-lg-6 s_col_no_resize o_masonry_grid_container">
            <div class="row">
                <div class="col-lg-12 oe_img_bg text-center pt224 pb224" data-name="Block"
                    style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_2);">
                    <p><br/></p>
                </div>
            </div>
            <div class="row">
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-6 pt24 pb8 text-center o_cc o_cc4" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_masonry_block_texts_image_texts_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-3 s_col_no_resize o_masonry_grid_container">
            <div class="row h-100">
                <div class="col-lg-12 pt24 pb8 text-center o_cc o_cc3" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-12 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
            </div>
        </div>
        <div class="col-lg-6 oe_img_bg text-center pt224 pb224" data-name="Block"
            style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_1);">
            <p><br/></p>
        </div>
        <div class="col-lg-3 s_col_no_resize o_masonry_grid_container">
            <div class="row h-100">
                <div class="col-lg-12 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
                <div class="col-lg-12 pt24 pb8 text-center o_cc o_cc4" data-name="Block">
                    <h3>A great title</h3>
                    <p>And a great subtitle</p>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_masonry_block_alternation_text_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-3 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
            <h3>A great title</h3>
            <p>And a great subtitle</p>
        </div>
        <div class="col-lg-3 pt24 pb8 text-center o_cc o_cc4" data-name="Block">
            <h3>A great title</h3>
            <p>And a great subtitle</p>
        </div>
        <div class="col-lg-3 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
            <h3>A great title</h3>
            <p>And a great subtitle</p>
        </div>
        <div class="col-lg-3 pt24 pb8 text-center o_cc o_cc3" data-name="Block">
            <h3>A great title</h3>
            <p>And a great subtitle</p>
        </div>
    </div>
</template>

<template id="s_masonry_block_alternation_text_image_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-3 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
            <h3>A great title</h3>
            <p>And a great subtitle</p>
        </div>
        <div class="col-lg-3 oe_img_bg text-center pt224 pb224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_1);">
            <p><br/></p>
        </div>
        <div class="col-lg-3 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
            <h3>A great title</h3>
            <p>And a great subtitle</p>
        </div>
        <div class="col-lg-3 oe_img_bg text-center pt224 pb224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_2);">
            <p><br/></p>
        </div>
    </div>
</template>

<template id="s_masonry_block_alternation_image_text_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-3 oe_img_bg text-center pt224 pb224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_1);">
            <p><br/></p>
        </div>
        <div class="col-lg-3 pt24 pb8 text-center o_cc o_cc4" data-name="Block">
            <h3>A great title</h3>
            <p>And a great subtitle</p>
        </div>
        <div class="col-lg-3 oe_img_bg text-center pt224 pb224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_2);">
            <p><br/></p>
        </div>
        <div class="col-lg-3 pt24 pb8 text-center o_cc o_cc3" data-name="Block">
            <h3>A great title</h3>
            <p>And a great subtitle</p>
        </div>
    </div>
</template>

<template id="s_masonry_block_alternation_text_image_text_template" groups="base.group_user">
    <div class="row">
        <div class="col-lg-3 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
            <h3>A great title</h3>
            <p>And a great subtitle</p>
        </div>
        <div class="col-lg-6 oe_img_bg text-center pt224 pb224" data-name="Block" style="background-image: url(/web/image/mass_mailing.s_masonry_block_default_image_1);">
            <p><br/></p>
        </div>
        <div class="col-lg-3 pt24 pb8 text-center o_cc o_cc2" data-name="Block">
            <h3>A great title</h3>
            <p>And a great subtitle</p>
        </div>
    </div>
</template>

<!-- Options -->
<template id="s_masonry_block_options" inherit_id="mass_mailing.snippet_options">
    <xpath expr="//div[@data-js='layout_column']" position="after">
        <div data-js="MasonryLayout" data-selector=".s_masonry_block">
            <we-select string="Template"
                data-name="masonry_template_opt"
                data-attribute-name="masonryTemplate"
                data-attribute-default-value="default">
                <we-button title="Default"
                    data-select-template="mass_mailing.s_masonry_block_default_template"
                    data-select-data-attribute="default"
                    data-img="/mass_mailing/static/src/img/snippets_options/masonry_template_default.svg"/>
                <we-button title="Default Reversed"
                    data-select-template="mass_mailing.s_masonry_block_reversed_template"
                    data-select-data-attribute="default_reversed"
                    data-img="/mass_mailing/static/src/img/snippets_options/masonry_template_reversed.svg"/>
                <we-button title="Images"
                    data-select-template="mass_mailing.s_masonry_block_images_template"
                    data-select-data-attribute="images"
                    data-img="/mass_mailing/static/src/img/snippets_options/masonry_template_images.svg"/>
                <we-button title="Image Text Image"
                    data-select-template="mass_mailing.s_masonry_block_image_texts_image_template"
                    data-select-data-attribute="image_text_image"
                    data-img="/mass_mailing/static/src/img/snippets_options/masonry_template_image_texts_image.svg"/>
                <we-button title="Mosaic"
                    data-select-template="mass_mailing.s_masonry_block_mosaic_template"
                    data-select-data-attribute="mosaic"
                    data-img="/mass_mailing/static/src/img/snippets_options/masonry_template_mosaic.svg"/>
                <we-button title="Text Image Text"
                    data-select-template="mass_mailing.s_masonry_block_texts_image_texts_template"
                    data-select-data-attribute="text_image_text"
                    data-img="/mass_mailing/static/src/img/snippets_options/masonry_template_texts_image_texts.svg"/>
                <we-button title="Alternate Text"
                    data-select-template="mass_mailing.s_masonry_block_alternation_text_template"
                    data-select-data-attribute="alternate_text"
                    data-img="/mass_mailing/static/src/img/snippets_options/masonry_template_alternate_texts.svg"/>
                <we-button title="Alternate Text Image"
                    data-select-template="mass_mailing.s_masonry_block_alternation_text_image_template"
                    data-select-data-attribute="alternate_text_image"
                    data-img="/mass_mailing/static/src/img/snippets_options/masonry_template_alternate_text_image.svg"/>
                <we-button title="Alternate Image Text"
                    data-select-template="mass_mailing.s_masonry_block_alternation_image_text_template"
                    data-select-data-attribute="alternate_image_text"
                    data-img="/mass_mailing/static/src/img/snippets_options/masonry_template_alternate_image_text.svg"/>
                <we-button title="Alternate Text Image Text"
                    data-select-template="mass_mailing.s_masonry_block_alternation_text_image_text_template"
                    data-select-data-attribute="alternate_text_image_text"
                    data-img="/mass_mailing/static/src/img/snippets_options/masonry_template_alternate_text_image_text.svg"/>
            </we-select>
        </div>
    </xpath>
</template>

<!-- Assets -->
<record id="mass_mailing.s_masonry_block_001_scss" model="ir.asset">
    <field name="name">Masonry block 001 SCSS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">mass_mailing/static/src/snippets/s_masonry_block/001.scss</field>
</record>

</odoo>

```

## File: views\snippets\s_media_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_media_list" name="Media List">
    <div class="s_media_list o_mail_snippet_general pt8 pb8 o_cc o_cc2" data-vcss="001">
        <div class="container">
            <div class="row s_nb_column_fixed">
                <div class="col-lg-12 s_media_list_item pt8 pb8" data-name="Media item">
                    <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                        <div class="col-lg-4 s_media_list_img_wrapper align-self-stretch px-0">
                            <img src="/web/image/mass_mailing.s_media_list_default_image_1" class="s_media_list_img h-100 w-100" alt=""/>
                        </div>
                        <div class="col-lg-8 s_media_list_body">
                            <h3>Media heading</h3>
                            <p>Use this snippet to build various types of components that feature a left- or right-aligned image alongside textual content. Duplicate the element to create a list that fits your needs.</p>
                            <a href="#" class="btn btn-primary">Discover</a>
                        </div>
                    </div>
                </div>
                <div class="col-lg-12 s_media_list_item pt8 pb8" data-name="Media item">
                    <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                        <div class="col-lg-4 s_media_list_img_wrapper align-self-stretch px-0">
                            <img src="/web/image/mass_mailing.s_media_list_default_image_2" class="s_media_list_img h-100 w-100" alt=""/>
                        </div>
                        <div class="col-lg-8 s_media_list_body">
                            <h3>Event heading</h3>
                            <p>Speakers from all over the world will join our experts to give inspiring talks on various topics. Stay on top of the latest business management trends &amp; technologies</p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-12 s_media_list_item pt8 pb8" data-name="Media item">
                    <div class="row s_col_no_resize s_col_no_bgcolor align-items-center o_cc o_cc1">
                        <div class="col-lg-4 s_media_list_img_wrapper align-self-stretch px-0">
                            <img src="/web/image/mass_mailing.s_media_list_default_image_3" class="s_media_list_img h-100 w-100" alt=""/>
                        </div>
                        <div class="col-lg-8 s_media_list_body">
                            <h3>Post heading</h3>
                            <p>Use this component for creating a list of featured elements to which you want to bring attention.</p>
                            <a href="#">Continue reading <i class="fa fa-long-arrow-right align-middle ms-1"/></a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<record id="mass_mailing.s_media_list_001_scss" model="ir.asset">
    <field name="name">Media list 001 SCSS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">mass_mailing/static/src/snippets/s_media_list/001.scss</field>
</record>

</odoo>

```

## File: views\snippets\s_numbers.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="s_numbers" name="Numbers">
        <div class="s_numbers o_mail_snippet_general o_cc o_cc1">
            <div class="container">
                <div class="row">
                    <div class="col-lg-4 pt24 pb24 o_cc o_cc2" style="text-align: center;">
                        <span style="font-size: 48px;">12</span>
                        <p>Useful options</p>
                    </div>
                    <div class="col-lg-4 pt24 pb24 o_cc o_cc4" style="text-align: center;">
                        <span style="font-size: 48px;">45</span>
                        <p>Beautiful snippets</p>
                    </div>
                    <div class="col-lg-4 pt24 pb24 o_cc o_cc2" style="text-align: center;">
                        <span style="font-size: 48px;">8</span>
                        <p>Amazing pages</p>
                    </div>
                </div>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\snippets\s_picture.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_picture" name="Picture">
    <div class="s_picture o_mail_snippet_general pt48 pb24 o_cc o_cc2" style="padding-left: 15px; padding-right: 15px;">
        <div class="container s_allow_columns">
            <h2 style="text-align: center;">
                <font style="font-size: 48px;">A Punchy Headline</font>
            </h2>
            <p style="text-align: center;">With strong technical foundations, Odoo's framework is unique. It provides <span style="font-weight: bolder;">top notch usability that scales across all apps</span>.</p>
            <img src="/web/image/mass_mailing.s_picture_default_image" style="padding: 10px;" class="mx-auto d-block mw-100" width="500" alt=""/>
            <p style="text-align: center;"><font style="font-size: 12px;">Add a caption to enhance the meaning of this image.</font></p>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_product_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_product_list" name="Items">
    <div class="s_mail_product_list o_mail_snippet_general">
        <div class="container">
            <div class="row">
                <div class="col-lg-4 o_cc pt16 pb16">
                    <a href="">
                        <img src="/web/image/mass_mailing.s_product_list_default_image_1" alt="" class="img img-fluid" style="padding: 15px 0px"/>
                    </a>
                    <p style="text-align: center;">Check out all our furniture</p>
                    <p style="text-align: center;"><a class="btn btn-primary" href="#">Furniture</a></p>
                </div>
                <div class="col-lg-4 o_cc pt16 pb16">
                    <a href="">
                        <img src="/web/image/mass_mailing.s_product_list_default_image_2" alt="" class="img img-fluid" style="padding: 15px 0px"/>
                    </a>
                    <p style="text-align: center;">Check out all our clothes</p>
                    <p style="text-align: center;"><a class="btn btn-primary" href="#">Clothes</a></p>
                </div>
                <div class="col-lg-4 o_cc pt16 pb16">
                    <a href="">
                        <img src="/web/image/mass_mailing.s_product_list_default_image_3" alt="" class="img img-fluid" style="padding: 15px 0px"/>
                    </a>
                    <p style="text-align: center;">Check out all our books</p>
                    <p style="text-align: center;"><a class="btn btn-primary" href="#">Books</a></p>
                </div>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_rating.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_rating" name="Rating">
    <div class="s_rating o_mail_snippet_general pt16 pb16" style="padding-left: 15px; padding-right: 15px;" data-vcss="001" data-icon="fa-star">
        <h3>Quality</h3>
        <div class="s_rating_icons o_not_editable">
            <span class="s_rating_active_icons">
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
                <i class="fa fa-star"></i>
            </span>
            <span class="s_rating_inactive_icons">
                <i class="fa fa-star-o"></i>
                <i class="fa fa-star-o"></i>
            </span>
        </div>
    </div>
</template>

<template id="s_rating_options" inherit_id="mass_mailing.snippet_options">
    <xpath expr="." position="inside">
        <div data-js="Rating" data-selector=".s_rating">
            <we-select string="Icon">
                <we-button data-set-icons="fa-star"><i class="fa fa-fw fa-star"/> Stars</we-button>
                <we-button data-set-icons="fa-thumbs-up"><i class="fa fa-fw fa-thumbs-up"/> Thumbs</we-button>
                <we-button data-set-icons="fa-circle"><i class="fa fa-fw fa-circle"/> Circles</we-button>
                <we-button data-set-icons="fa-square"><i class="fa fa-fw fa-square"/> Squares</we-button>
                <we-button data-set-icons="fa-heart"><i class="fa fa-fw fa-heart"/> Hearts</we-button>
                <we-button data-set-icons="custom" class="d-none">Custom</we-button>
            </we-select>
            <we-row string="&#8985; Active">
                <we-colorpicker data-select-style="" data-apply-to=".s_rating_active_icons" data-css-property="color" data-color-prefix="text-"/>
                <we-button data-custom-icon="true" data-custom-active-icon="true" data-no-preview="true">
                    <i class="fa fa-fw fa-refresh me-1"/> Replace Icon
                </we-button>
            </we-row>
            <we-row string="&#8985; Inactive">
                <we-colorpicker data-select-style="" data-apply-to=".s_rating_inactive_icons" data-css-property="color" data-color-prefix="text-"/>
                <we-button data-custom-icon="true" data-custom-active-icon="false" data-no-preview="true">
                    <i class="fa fa-fw fa-refresh me-1"/> Replace Icon
                </we-button>
            </we-row>
            <we-row string="Score">
                <we-input data-active-icons-number="true" data-step="1"/>
                <span class="mx-2">/</span>
                <we-input data-total-icons-number="true" data-step="1"/>
            </we-row>
            <we-button-group string="Size" data-apply-to=".s_rating_icons">
                <we-button data-select-class="" title="Small" data-img="/website/static/src/img/snippets_options/size_small.svg"/>
                <we-button data-select-class="fa-2x" title="Medium" data-img="/website/static/src/img/snippets_options/size_medium.svg"/>
                <we-button data-select-class="fa-3x" title="Large" data-img="/website/static/src/img/snippets_options/size_large.svg"/>
            </we-button-group>
            <we-checkbox string="Display Inline" data-select-class="s_rating_inline" data-no-preview="true"/>
        </div>
    </xpath>
</template>

<record id="mass_mailing.s_rating_000_scss" model="ir.asset">
    <field name="name">Rating 000 SCSS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">mass_mailing/static/src/snippets/s_rating/000.scss</field>
    <field name="active" eval="False"/>
</record>

<record id="mass_mailing.s_rating_001_scss" model="ir.asset">
    <field name="name">Rating 001 SCSS</field>
    <field name="bundle">web.assets_frontend</field>
    <field name="path">mass_mailing/static/src/snippets/s_rating/001.scss</field>
</record>

</odoo>

```

## File: views\snippets\s_references.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_references" name="References">
    <div class="s_references o_mail_snippet_general pt32 pb32">
        <div class="container">
            <h2 style="text-align: center;">Our References</h2>
            <p style="text-align: center;">We are in good company.</p>
            <div class="row">
                <div class="col-lg-3 pt16 pb16">
                    <img src="/web/image/mass_mailing.s_reference_default_image_1" class="img img-fluid mx-auto" alt=""/>
                </div>
                <div class="col-lg-3 pt16 pb16">
                    <img src="/web/image/mass_mailing.s_reference_default_image_2" class="img img-fluid mx-auto" alt=""/>
                </div>
                <div class="col-lg-3 pt16 pb16">
                    <img src="/web/image/mass_mailing.s_reference_default_image_3" class="img img-fluid mx-auto" alt=""/>
                </div>
                <div class="col-lg-3 pt16 pb16">
                    <img src="/web/image/mass_mailing.s_reference_default_image_4" class="img img-fluid mx-auto" alt=""/>
                </div>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_showcase.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_showcase" name="Showcase">
    <div class="s_showcase o_mail_snippet_general pt48 pb48">
        <!-- TODO: (below) issue with height: `fit-content` is not supported, can we calculate it in px in translation ?
        empty div height is 0 unless table has defined height (div.col-1 > div.w-50.h100.border-end)-->
        <div class="container" style="height: fit-content;">
            <div class="row g-0 s_col_no_resize s_col_no_bgcolor s_nb_column_fixed">
                <div class="col-lg-6 pb24" style="text-align: right; padding-right: 70px;" align="right">
                    <div class="mb-2">
                        <h3 class="d-inline-block">First feature</h3>
                        <i class="fa fa-2x fa-desktop text-secondary ms-3"/>
                    </div>
                    <p>A short description of this great feature.</p>
                </div>
                <div class="col-lg-6 pb24 border-start" style="padding-left: 70px;" align="left">
                    <div class="mb-2">
                        <i class="fa fa-2x fa-heart text-secondary me-3"/>
                        <h3 class="d-inline-block">Another feature</h3>
                    </div>
                    <p>A short description of this great feature.</p>
                </div>
            </div>
            <div class="row g-0 s_col_no_resize s_col_no_bgcolor s_nb_column_fixed">
                <div class="col-lg-6" style="text-align: right; padding-right: 70px;" align="right">
                    <div class="mb-2">
                        <h3 class="d-inline-block">Second feature</h3>
                        <i class="fa fa-2x fa-paint-brush text-secondary ms-3"/>
                    </div>
                    <p>A short description of this great feature.</p>
                </div>
                <div class="col-lg-6 border-start" style="padding-left: 70px;" align="left">
                    <div class="mb-2">
                        <i class="fa fa-2x fa-gift text-secondary me-3"/>
                        <h3 class="d-inline-block">Last Feature</h3>
                    </div>
                    <p>A short description of this great feature.</p>
                </div>
            </div>
        </div>
        <div class="container pt32" style="text-align: center;" align="center">
            <a href="#" class="btn btn-primary">Discover all the features</a>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_text_block.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="s_text_block" name="Text">
        <div class="s_text_block o_mail_snippet_general pt40 pb40" style="padding-left: 15px; padding-right: 15px;">
            <div class="container s_allow_columns">
                <p> The open source model of Odoo has allowed us to leverage thousands of developers and
                    business experts to build hundreds of apps in just a few years.</p>
                <p> With strong technical foundations, Odoo's framework is unique.
                    It provides top notch usability that scales across all apps.</p>
                <p> Usability improvements made on Odoo will automatically apply to all
                    of our fully integrated apps.</p>
                <p> That way, Odoo evolves much faster than any other solution.</p>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\snippets\s_text_highlight.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_text_highlight" name="Text Highlight">
    <div class="s_mail_text_highlight o_mail_snippet_general o_cc o_cc3 pt32 pb32 w-100">
        <h3 style="text-align: center;">Text Highlight</h3>
        <p style="text-align: center;">Put the focus on what you have to say!</p>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_text_image.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_text_image" name="Text - Image">
    <div class="s_text_image o_mail_snippet_general">
        <div class="container">
            <div class="row align-items-center">
                <div class="col-lg-6 o_cc pt16 pb16">
                    <h3>A unique value</h3>
                    <p style="text-align: justify;">The open source model of Odoo has allowed us to leverage thousands of developers and business experts to build hundreds of apps in just a few years.</p>
                    <div style="text-align: left;">
                        <a href="#" class="btn btn-link">Read More</a>
                    </div>
                </div>
                <div class="col-lg-6 px-0">
                    <img src="/web/image/mass_mailing.s_text_image_default_image" class="img w-100"/>
                </div>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_three_columns.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_three_columns" name="Columns">
    <div class="s_three_columns o_mail_snippet_general o_cc o_cc2 pt32 pb32">
        <div class="container">
            <div class="row d-flex align-items-stretch">
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16">
                    <div class="card text-bg-white h-100">
                        <img class="card-img-top" src="/web/image/mass_mailing.s_three_columns_default_image_1" alt=""/>
                        <div class="card-body">
                            <h3 class="card-title">Feature One</h3>
                            <p class="card-text">Adapt these three columns to fit your design need. To duplicate, delete or move columns, select the column and use the top icons to perform your action.</p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16">
                    <div class="card text-bg-white h-100">
                        <img class="card-img-top" src="/web/image/mass_mailing.s_three_columns_default_image_2" alt=""/>
                        <div class="card-body">
                            <h3 class="card-title">Feature Two</h3>
                            <p class="card-text">To add a fourth column, reduce the size of these three columns using the right icon of each block. Then, duplicate one of the columns to create a new one as a copy.</p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16">
                    <div class="card text-bg-white h-100">
                        <img class="card-img-top" src="/web/image/mass_mailing.s_three_columns_default_image_3" alt=""/>
                        <div class="card-body">
                            <h3 class="card-title">Feature Three</h3>
                            <p class="card-text">Delete the above image or replace it with a picture that illustrates your message. Click on the picture to change its <em>rounded corner</em> style.</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_title.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_title" name="Title">
    <div class="s_title o_mail_snippet_general pt32 pb32">
        <div class="container s_allow_columns">
            <h1 style="text-align:center">Your Title</h1>
        </div>
    </div>
</template>

</odoo>

```

## File: wizard\mailing_contact_import.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, tools, _
from odoo.tools.misc import clean_context


class MailingContactImport(models.TransientModel):
    _name = 'mailing.contact.import'
    _description = 'Mailing Contact Import'

    mailing_list_ids = fields.Many2many('mailing.list', string='Lists')
    contact_list = fields.Text('Contact List', help='Contact list that will be imported, one contact per line')

    def action_import(self):
        """Import each lines of "contact_list" as a new contact."""
        self.ensure_one()
        contacts = tools.email_split_tuples(', '.join((self.contact_list or '').splitlines()))
        if not contacts:
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'message': _('No valid email address found.'),
                    'next': {'type': 'ir.actions.act_window_close'},
                    'sticky': False,
                    'type': 'warning',
                }
            }

        if len(contacts) > 5000:
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'message': _('You have to much emails, please upload a file.'),
                    'type': 'warning',
                    'sticky': False,
                    'next': self.action_open_base_import(),
                }
            }

        all_emails = list({values[1].lower() for values in contacts})

        existing_contacts = self.env['mailing.contact'].search([
            ('email_normalized', 'in', all_emails),
            ('list_ids', 'in', self.mailing_list_ids.ids),
        ])
        existing_contacts = {
            contact.email_normalized: contact
            for contact in existing_contacts
        }

        # Remove duplicated record, keep only the first non-empty name for each email address
        unique_contacts = {}
        for name, email in contacts:
            email = email.lower()
            if unique_contacts.get(email, {}).get('name'):
                continue

            if email in existing_contacts and not self.mailing_list_ids < existing_contacts[email].list_ids:
                existing_contacts[email].list_ids |= self.mailing_list_ids
            if email not in existing_contacts:
                unique_contacts[email] = {
                    'name': name,
                    'list_ids': self.mailing_list_ids.ids,
                }

        if not unique_contacts:
            return {
                'type': 'ir.actions.client',
                'tag': 'display_notification',
                'params': {
                    'message': _('No contacts were imported. All email addresses are already in the mailing list.'),
                    'next': {'type': 'ir.actions.act_window_close'},
                    'sticky': False,
                    'type': 'warning',
                }
            }

        new_contacts = self.env['mailing.contact'].with_context(clean_context(self.env.context)).create([
            {
                'email': email,
                **values,
            }
            for email, values in unique_contacts.items()
        ])

        ignored = len(contacts) - len(unique_contacts)

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'message': (
                    _('%i Contacts have been imported.', len(unique_contacts))
                    + (_(' %i duplicates have been ignored.', ignored) if ignored else '')
                ),
                'type': 'success',
                'sticky': False,
                'next': {
                    'context': self.env.context,
                    'domain': [('id', 'in', new_contacts.ids)],
                    'name': _('New contacts imported'),
                    'res_model': 'mailing.contact',
                    'type': 'ir.actions.act_window',
                    'view_mode': 'list',
                    'views': [[False, 'list'], [False, 'form']],
                },
            }
        }

    def action_open_base_import(self):
        """Open the base import wizard to import mailing list contacts with a xlsx file."""
        self.ensure_one()

        return {
            'type': 'ir.actions.client',
            'tag': 'import',
            'name': _('Import Mailing Contacts'),
            'params': {
                'context': self.env.context,
                'model': 'mailing.contact',
            }
        }

```

## File: wizard\mailing_contact_import_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="mailing_contact_import_view_form" model="ir.ui.view">
        <field name="name">mailing.contact.import.view.form</field>
        <field name="model">mailing.contact.import</field>
        <field name="arch" type="xml">
            <form string="Import Mailing Contacts">
                <group>
                    <field name="mailing_list_ids" string="Import contacts in"
                        widget="many2many_tags" placeholder="Select mailing lists"
                        options="{'no_create': True}"/>
                </group>
                <p>
                    Write or paste email addresses in the field below.
                    Each line will be imported as a mailing list contact.
                </p>
                <label for="contact_list" class="mb-2">Contact List</label>
                <field name="contact_list"
                    nolabel="1" default_focus="1"
                    placeholder='"Damien Roberts" &lt;d.roberts@example.com&gt;&#10;"Rick Sanchez" &lt;rick_sanchez@example.com&gt;&#10;victor_hugo@example.com'/>
                <p class="text-muted mb-0">
                    Want to import country, company name and more?
                    <button type="object" name="action_open_base_import"
                        class="fw-normal px-0 btn btn-link">
                        Upload a file
                    </button>
                </p>
                <footer>
                    <button string="Import" type="object" name="action_import"
                        class="btn-primary" data-hotkey="i"/>
                    <button string="Discard" class="btn-secondary"
                        special="cancel" data-hotkey="z"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="mailing_contact_import_action" model="ir.actions.act_window">
        <field name="name">Import Mailing Contacts</field>
        <field name="res_model">mailing.contact.import</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\mailing_contact_to_list.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, _

class MailingContactToList(models.TransientModel):
    _name = "mailing.contact.to.list"
    _description = "Add Contacts to Mailing List"

    contact_ids = fields.Many2many('mailing.contact', string='Contacts')
    mailing_list_id = fields.Many2one('mailing.list', string='Mailing List', required=True)

    def action_add_contacts(self):
        """ Simply add contacts to the mailing list and close wizard. """
        return self._add_contacts_to_mailing_list({'type': 'ir.actions.act_window_close'})

    def action_add_contacts_and_send_mailing(self):
        """ Add contacts to the mailing list and redirect to a new mailing on
        this list. """
        self.ensure_one()

        action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing.mailing_mailing_action_mail")
        action['views'] = [[False, "form"]]
        action['target'] = 'current'
        action['context'] = {
            'default_contact_list_ids': [self.mailing_list_id.id]
        }
        return self._add_contacts_to_mailing_list(action)

    def _add_contacts_to_mailing_list(self, action):
        self.ensure_one()

        previous_count = len(self.mailing_list_id.contact_ids)
        self.mailing_list_id.write({
            'contact_ids': [
                (4, contact.id)
                for contact in self.contact_ids
                if contact not in self.mailing_list_id.contact_ids]
            })

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'info',
                'message': _("%s Mailing Contacts have been added. ",
                             len(self.mailing_list_id.contact_ids) - previous_count
                            ),
                'sticky': False,
                'next': action,
            }
        }

```

## File: wizard\mailing_contact_to_list_views.xml

```xml
<?xml version="1.0"?>
<odoo>

    <record id="mailing_contact_to_list_view_form" model="ir.ui.view" >
        <field name="name">mailing.contact.to.list.view.form</field>
            <field name="model">mailing.contact.to.list</field>
            <field name="arch" type="xml">
                <form string="Send a Sample Mail">
                    <group>
                        <field name="mailing_list_id" options="{'no_create_edit': True, 'no_open': True}"/>
                        <field name="contact_ids" invisible="1"/>
                    </group>
                    <footer>
                        <button string="Add" name="action_add_contacts" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Add and Send Mailing" name="action_add_contacts_and_send_mailing" type="object" class="btn-primary" data-hotkey="w"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z" />
                    </footer>
                </form>
            </field>
    </record>

    <record id="mailing_contact_to_list_action" model="ir.actions.act_window">
        <field name="name">Add Selected Contacts to a Mailing List</field>
        <field name="res_model">mailing.contact.to.list</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>

</odoo>

```

## File: wizard\mailing_list_merge.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class MassMailingListMerge(models.TransientModel):
    _name = 'mailing.list.merge'
    _description = 'Merge Mass Mailing List'

    @api.model
    def default_get(self, fields):
        res = super(MassMailingListMerge, self).default_get(fields)

        if not res.get('src_list_ids') and 'src_list_ids' in fields:
            if self.env.context.get('active_model') != 'mailing.list':
                raise UserError(_('You can only apply this action from Mailing Lists.'))
            src_list_ids = self.env.context.get('active_ids')
            res.update({
                'src_list_ids': [(6, 0, src_list_ids)],
            })
        if not res.get('dest_list_id') and 'dest_list_id' in fields:
            src_list_ids = res.get('src_list_ids') or self.env.context.get('active_ids')
            res.update({
                'dest_list_id': src_list_ids and src_list_ids[0] or False,
            })
        return res

    src_list_ids = fields.Many2many('mailing.list', string='Mailing Lists')
    dest_list_id = fields.Many2one('mailing.list', string='Destination Mailing List')
    merge_options = fields.Selection([
        ('new', 'Merge into a new mailing list'),
        ('existing', 'Merge into an existing mailing list'),
    ], 'Merge Option', required=True, default='new')
    new_list_name = fields.Char('New Mailing List Name')
    archive_src_lists = fields.Boolean('Archive source mailing lists', default=True)

    def action_mailing_lists_merge(self):
        if self.merge_options == 'new':
            self.dest_list_id = self.env['mailing.list'].create({
                'name': self.new_list_name,
            }).id
        self.dest_list_id.action_merge(self.src_list_ids, self.archive_src_lists)
        return self.dest_list_id

```

## File: wizard\mailing_list_merge_views.xml

```xml
<?xml version="1.0"?>
<odoo>
	<!-- Merge Mailing List  -->
    <record id="mailing_list_merge_view_form" model="ir.ui.view">
        <field name="name">mailing.list.merge.form</field>
        <field name="model">mailing.list.merge</field>
        <field name="arch" type="xml">
            <form string="Merge Mass Mailing List">
                <group>
                    <field name="merge_options" widget="selection"/>
                    <field name="new_list_name" attrs="{'invisible': [('merge_options', '=', 'existing')], 'required': [('merge_options', '=', 'new')]}"/>
                    <field name="dest_list_id" attrs="{'invisible': [('merge_options', '=', 'new')], 'required': [('merge_options', '=', 'existing')]}"/>
                    <field name="archive_src_lists"/>
                </group>
                <field name="src_list_ids">
                    <tree>
                        <field name="name"/>
                        <field name="contact_count" string="Number of Recipients"/>
                    </tree>
                </field>
                <footer>
                    <button name="action_mailing_lists_merge" type="object" string="Merge" class="btn-primary" data-hotkey="q"/>
                    <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="mailing_list_merge_action" model="ir.actions.act_window">
        <field name="name">Merge</field>
        <field name="res_model">mailing.list.merge</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
        <field name="binding_model_id" ref="model_mailing_list"/>
        <field name="binding_view_types">list</field>
    </record>
</odoo>

```

## File: wizard\mailing_mailing_schedule_date.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MailingMailingScheduleDate(models.TransientModel):
    _name = "mailing.mailing.schedule.date"
    _description = "schedule a mailing"

    schedule_date = fields.Datetime(string='Scheduled for')
    mass_mailing_id = fields.Many2one('mailing.mailing', required=True)

    def action_schedule_date(self):
        self.mass_mailing_id.write({'schedule_type': 'scheduled', 'schedule_date': self.schedule_date})
        self.mass_mailing_id.action_put_in_queue()

```

## File: wizard\mailing_mailing_schedule_date_views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <record id="mailing_mailing_schedule_date_view_form"  model="ir.ui.view">
        <field name="name">mailing.mailing.schedule.date.view.form</field>
        <field name="model">mailing.mailing.schedule.date</field>
        <field name="arch" type="xml">
            <form string="Take Future Schedule Date">
                <group>
                    <group>
                        <field name="schedule_date" string="Send on" required="1"/>
                    </group>
                </group>
                <footer>
                    <button string="Schedule" name="action_schedule_date" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Discard " class="btn-secondary" special="cancel" data-hotkey="z" />
                </footer>
            </form>
        </field>
    </record>

    <record id="mailing_mailing_schedule_date_action" model="ir.actions.act_window">
        <field name="name">When do you want to send your mailing?</field>
        <field name="res_model">mailing.mailing.schedule.date</field>
        <field name="type">ir.actions.act_window</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\mailing_mailing_test.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup

from odoo import _, fields, models, tools


class TestMassMailing(models.TransientModel):
    _name = 'mailing.mailing.test'
    _description = 'Sample Mail Wizard'

    email_to = fields.Text(string='Recipients', required=True,
                           help='Carriage-return-separated list of email addresses.', default=lambda self: self.env.user.email_formatted)
    mass_mailing_id = fields.Many2one('mailing.mailing', string='Mailing', required=True, ondelete='cascade')

    def send_mail_test(self):
        self.ensure_one()
        ctx = dict(self.env.context)
        ctx.pop('default_state', None)
        self = self.with_context(ctx)

        mails_sudo = self.env['mail.mail'].sudo()
        valid_emails = []
        invalid_candidates = []
        for candidate in self.email_to.splitlines():
            test_email = tools.email_split(candidate)
            if test_email:
                valid_emails.append(test_email[0])
            else:
                invalid_candidates.append(candidate)

        mailing = self.mass_mailing_id
        record = self.env[mailing.mailing_model_real].search([], limit=1)

        # If there is atleast 1 record for the model used in this mailing, then we use this one to render the template
        # Downside: Qweb syntax is only tested when there is atleast one record of the mailing's model
        if record:
            # Returns a proper error if there is a syntax error with Qweb
            body = mailing.with_context(preserve_comments=True)._render_field('body_html', record.ids, post_process=True)[record.id]
            preview = mailing._render_field('preview', record.ids, post_process=True)[record.id]
            full_body = mailing._prepend_preview(Markup(body), preview)
            subject = mailing._render_field('subject', record.ids)[record.id]
        else:
            full_body = mailing._prepend_preview(mailing.body_html, mailing.preview)
            subject = mailing.subject

        # Convert links in absolute URLs before the application of the shortener
        full_body = self.env['mail.render.mixin']._replace_local_links(full_body)

        for valid_email in valid_emails:
            mail_values = {
                'email_from': mailing.email_from,
                'reply_to': mailing.reply_to,
                'email_to': valid_email,
                'subject': subject,
                'body_html': self.env['ir.qweb']._render('mass_mailing.mass_mailing_mail_layout', {'body': full_body}, minimal_qcontext=True),
                'is_notification': True,
                'mailing_id': mailing.id,
                'attachment_ids': [(4, attachment.id) for attachment in mailing.attachment_ids],
                'auto_delete': False,  # they are manually deleted after notifying the document
                'mail_server_id': mailing.mail_server_id.id,
            }
            mail = self.env['mail.mail'].sudo().create(mail_values)
            mails_sudo |= mail
        mails_sudo.send()

        notification_messages = []
        if invalid_candidates:
            notification_messages.append(
                _('Mailing addresses incorrect: %s', ', '.join(invalid_candidates)))

        for mail_sudo in mails_sudo:
            if mail_sudo.state == 'sent':
                notification_messages.append(
                    _('Test mailing successfully sent to %s', mail_sudo.email_to))
            elif mail_sudo.state == 'exception':
                notification_messages.append(
                    _('Test mailing could not be sent to %s:<br>%s',
                        mail_sudo.email_to,
                        mail_sudo.failure_reason)
                )

        # manually delete the emails since we passed 'auto_delete: False'
        mails_sudo.unlink()

        if notification_messages:
            self.mass_mailing_id._message_log(body='<ul>%s</ul>' % ''.join(
                ['<li>%s</li>' % notification_message for notification_message in notification_messages]
            ))

        return True

```

## File: wizard\mailing_mailing_test_views.xml

```xml
<?xml version="1.0"?>
<odoo>

        <record model="ir.ui.view" id="view_mail_mass_mailing_test_form">
            <field name="name">mailing.mailing.test.form</field>
            <field name="model">mailing.mailing.test</field>
            <field name="arch" type="xml">
                <form string="Send a Sample Mail">
                    <p class="text-muted">
                        Send a sample mailing for testing purpose to the address below.
                    </p>
                    <group>
                        <field name="email_to" placeholder="email1@example.com&#10;email2@example.com"/>
                    </group>
                    <footer>
                        <button string="Send" name="send_mail_test" type="object" class="btn-primary" data-hotkey="q"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" data-hotkey="z"/>
                    </footer>
                </form>
            </field>
        </record>

        <record id="action_mail_mass_mailing_test" model="ir.actions.act_window">
            <field name="name">Mailing Test</field>
            <field name="res_model">mailing.mailing.test</field>
            <field name="view_mode">form</field>
            <field name="target">new</field>
        </record>

</odoo>

```

## File: wizard\mail_compose_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MailComposeMessage(models.TransientModel):
    _inherit = 'mail.compose.message'

    mass_mailing_id = fields.Many2one('mailing.mailing', string='Mass Mailing', ondelete='cascade')
    campaign_id = fields.Many2one('utm.campaign', string='Mass Mailing Campaign', ondelete='set null')
    mass_mailing_name = fields.Char(string='Mass Mailing Name', help='If set, a mass mailing will be created so that you can track its results in the Email Marketing app.')
    mailing_list_ids = fields.Many2many('mailing.list', string='Mailing List')

    def get_mail_values(self, res_ids):
        """ Override method that generated the mail content by creating the
        mailing.trace values in the o2m of mail_mail, when doing pure
        email mass mailing. """
        now = fields.Datetime.now()
        self.ensure_one()
        res = super(MailComposeMessage, self).get_mail_values(res_ids)
        # use only for allowed models in mass mailing
        if self.composition_mode == 'mass_mail' and \
                (self.mass_mailing_name or self.mass_mailing_id) and \
                self.env['ir.model'].sudo().search_count([('model', '=', self.model), ('is_mail_thread', '=', True)]):
            mass_mailing = self.mass_mailing_id
            if not mass_mailing:
                mass_mailing = self.env['mailing.mailing'].create({
                    'campaign_id': self.campaign_id.id,
                    'name': self.mass_mailing_name,
                    'subject': self.subject,
                    'state': 'done',
                    'reply_to_mode': self.reply_to_mode,
                    'reply_to': self.reply_to if self.reply_to_mode == 'new' else False,
                    'sent_date': now,
                    'body_html': self.body,
                    'mailing_model_id': self.env['ir.model']._get(self.model).id,
                    'mailing_domain': self.active_domain,
                    'attachment_ids': [(6, 0, self.attachment_ids.ids)],
                })
                self.mass_mailing_id = mass_mailing.id

            recipients_info = self._process_recipient_values(res)
            for res_id in res_ids:
                mail_values = res[res_id]
                if mail_values.get('body_html'):
                    body = self.env['ir.qweb']._render('mass_mailing.mass_mailing_mail_layout',
                                {'body': mail_values['body_html']},
                                minimal_qcontext=True, raise_if_not_found=False)
                    if body:
                        mail_values['body_html'] = body

                trace_vals = {
                    'message_id': mail_values['message_id'],
                    'model': self.model,
                    'res_id': res_id,
                    'mass_mailing_id': mass_mailing.id,
                    # if mail_to is void, keep falsy values to allow searching / debugging traces
                    'email': recipients_info[res_id]['mail_to'][0] if recipients_info[res_id]['mail_to'] else '',
                }
                # propagate failed states to trace when still-born
                if mail_values.get('state') == 'cancel':
                    trace_vals['trace_status'] = 'cancel'
                elif mail_values.get('state') == 'exception':
                    trace_vals['trace_status'] = 'error'
                if mail_values.get('failure_type'):
                    trace_vals['failure_type'] = mail_values['failure_type']

                mail_values.update({
                    'mailing_id': mass_mailing.id,
                    'mailing_trace_ids': [(0, 0, trace_vals)],
                    # email-mode: keep original message for routing
                    'is_notification': mass_mailing.reply_to_mode == 'update',
                    'auto_delete': not mass_mailing.keep_archives,
                })
        return res

    def _get_done_emails(self, mail_values_dict):
        seen_list = super(MailComposeMessage, self)._get_done_emails(mail_values_dict)
        if self.mass_mailing_id:
            seen_list += self.mass_mailing_id._get_seen_list()
        return seen_list

    def _get_optout_emails(self, mail_values_dict):
        opt_out_list = super(MailComposeMessage, self)._get_optout_emails(mail_values_dict)
        if self.mass_mailing_id:
            opt_out_list += self.mass_mailing_id._get_opt_out_list()
        return opt_out_list

```

## File: wizard\mail_compose_message_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- Add mass mail campaign to the mail.compose.message form view -->
        <record model="ir.ui.view" id="email_compose_form_mass_mailing">
            <field name="name">mail.compose.message.form.mass_mailing</field>
            <field name="model">mail.compose.message</field>
            <field name="inherit_id" ref="mail.email_compose_message_wizard_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='notify']" position="after">
                    <field name="campaign_id" groups="mass_mailing.group_mass_mailing_campaign"
                        attrs="{'invisible': [('composition_mode', '!=', 'mass_mail')]}"/>
                    <field name="mass_mailing_name"
                        attrs="{'invisible': [('composition_mode', '!=', 'mass_mail')]}"/>
                </xpath>
                <xpath expr="//button[@name='action_send_mail'][not(hasclass('o_mail_send'))]" position="attributes">
                    <!-- 'Log' button -->
                    <attribute name="attrs">
                        {'invisible': [
                            '|',
                                ('is_log', '=', False),
                                '&amp;',
                                    ('mass_mailing_name', '!=', ''),
                                    ('mass_mailing_name', '!=', False)
                        ]}
                    </attribute>
                </xpath>
                <xpath expr="//button[hasclass('o_mail_send')]" position="attributes">
                    <!-- 'Send' button -->
                    <attribute name="attrs">
                        {'invisible': [
                            '|',
                                ('is_log', '=', True),
                                '&amp;',
                                    ('mass_mailing_name', '!=', ''),
                                    ('mass_mailing_name', '!=', False)
                        ]}
                    </attribute>
                </xpath>
                <xpath expr="//button[@name='action_send_mail']" position="after">
                    <button string="Send Mass Mailing" name="action_send_mail" type="object" class="btn-primary o_mail_send"
                        attrs="{'invisible': ['|', ('mass_mailing_name', '==', ''), ('mass_mailing_name', '==', False)]}" data-hotkey="q"/>
                </xpath>
            </field>
        </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_compose_message
from . import mailing_contact_import
from . import mailing_contact_to_list
from . import mailing_list_merge
from . import mailing_mailing_test
from . import mailing_mailing_schedule_date

```

