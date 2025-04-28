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
    'description': "",
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
        'security/mass_mailing_security.xml',
        'security/ir.model.access.csv',
        'data/mail_data.xml',
        'data/mailing_data_templates.xml',
        'data/mass_mailing_data.xml',
        'data/ir_attachment_data.xml',
        'wizard/mail_compose_message_views.xml',
        'wizard/mailing_contact_to_list_views.xml',
        'wizard/mailing_list_merge_views.xml',
        'wizard/mailing_mailing_test_views.xml',
        'wizard/mailing_mailing_schedule_date_views.xml',
        'views/mailing_mailing_views_menus.xml',
        'views/mailing_trace_views.xml',
        'views/link_tracker_views.xml',
        'views/mailing_contact_views.xml',
        'views/mailing_list_views.xml',
        'views/mailing_mailing_views.xml',
        'views/res_config_settings_views.xml',
        'views/utm_campaign_views.xml',
        'report/mailing_trace_report_views.xml',
        'views/assets.xml',
        'views/mass_mailing_templates_portal.xml',
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
        'web.assets_backend': [
            'mass_mailing/static/src/scss/mass_mailing.scss',
            'mass_mailing/static/src/scss/mass_mailing_mobile.scss',
            'mass_mailing/static/src/css/email_template.css',
            'mass_mailing/static/src/js/mass_mailing.js',
            'mass_mailing/static/src/js/mass_mailing_widget.js',
            'mass_mailing/static/src/js/mailing_mailing_view_form_full_width.js',
            'mass_mailing/static/src/js/unsubscribe.js',
        ],
        'mass_mailing.assets_mail_themes': [
            'mass_mailing/static/src/scss/themes/**/*',
        ],
        'mass_mailing.assets_mail_themes_edition': [
            ('include', 'web._assets_helpers'),
            'web/static/lib/bootstrap/scss/_variables.scss',
            'mass_mailing/static/src/scss/mass_mailing.ui.scss',
        ],
        'web_editor.assets_wysiwyg': [
            'mass_mailing/static/src/js/snippets.editor.js',
            'mass_mailing/static/src/js/wysiwyg.js',
        ],
        'web.assets_common': [
            'mass_mailing/static/src/js/tours/**/*',
        ],
        'web.qunit_suite_tests': [
            'mass_mailing/static/tests/field_html_test.js',
            'mass_mailing/static/src/js/mass_mailing_snippets.js',
            'mass_mailing/static/src/snippets/s_blockquote/options.js',
            'mass_mailing/static/src/snippets/s_media_list/options.js',
            'mass_mailing/static/src/snippets/s_showcase/options.js',
            'mass_mailing/static/src/snippets/s_rating/options.js',
            'mass_mailing/static/tests/mass_mailing_html_tests.js',
        ],
        'web.assets_qweb': [
            'mass_mailing/static/src/xml/*.xml',
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

import werkzeug

from odoo import _, exceptions, http, tools
from odoo.http import request, Response
from odoo.tools import consteq
from werkzeug.exceptions import BadRequest


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

    @http.route(['/unsubscribe_from_list'], type='http', website=True, multilang=False, auth='public', sitemap=False)
    def unsubscribe_placeholder_link(self, **post):
        """Dummy route so placeholder is not prefixed by language, MUST have multilang=False"""
        raise werkzeug.exceptions.NotFound()

    # csrf is disabled here because it will be called by the MUA with unpredictable session at that time
    @http.route(['/mail/mailing/<int:mailing_id>/unsubscribe_oneclick'], type='http', website=True, auth='public',
                methods=["POST"], csrf=False)
    def mailing_unsubscribe_oneclick(self, mailing_id, email=None, res_id=None, token="", **post):
        self.mailing(mailing_id, email=email, res_id=res_id, token=token, **post)
        return Response(status=200)

    @http.route(['/mail/mailing/<int:mailing_id>/unsubscribe'], type='http', website=True, auth='public')
    def mailing(self, mailing_id, email=None, res_id=None, token="", **post):
        mailing = request.env['mailing.mailing'].sudo().browse(mailing_id)
        if mailing.exists():
            res_id = res_id and int(res_id)
            if not self._valid_unsubscribe_token(mailing_id, res_id, email, str(token)):
                raise exceptions.AccessDenied()

            if mailing.mailing_model_real == 'mailing.contact':
                # Unsubscribe directly + Let the user choose his subscriptions
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
                list_ids = request.env['mailing.list'].sudo().browse(unique_list_ids)
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
                ]).mapped('list_id')
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

    @http.route('/mail/track/<int:mail_id>/<string:token>/blank.gif', type='http', auth='public')
    def track_mail_open(self, mail_id, token, **post):
        """ Email tracking. """
        if not consteq(token, tools.hmac(request.env(su=True), 'mass_mailing-mail_mail-open', mail_id)):
            raise BadRequest()

        request.env['mailing.trace'].sudo().set_opened(domain=[('mail_mail_id_int', 'in', [mail_id])])
        response = werkzeug.wrappers.Response()
        response.mimetype = 'image/gif'
        response.data = base64.b64decode(b'R0lGODlhAQABAIAAANvf7wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==')

        return response

    @http.route(['/mailing/<int:mailing_id>/view'], type='http', website=True, auth='public')
    def view(self, mailing_id, email=None, res_id=None, token=""):
        mailing = request.env['mailing.mailing'].sudo().browse(mailing_id)
        if mailing.exists():
            res_id = int(res_id) if res_id else False
            if not self._valid_unsubscribe_token(mailing_id, res_id, email, str(token)) and not request.env.user.has_group('mass_mailing.group_mass_mailing_user'):
                raise exceptions.AccessDenied()

            res = mailing.convert_links()
            base_url = mailing.get_base_url().rstrip('/')
            urls_to_replace = [
                (base_url + '/unsubscribe_from_list', mailing._get_unsubscribe_url(email, res_id)),
                (base_url + '/view', mailing._get_view_url(email, res_id))
            ]
            for url_to_replace, new_url in urls_to_replace:
                if url_to_replace in res[mailing_id]:
                    res[mailing_id] = res[mailing_id].replace(url_to_replace, new_url if new_url else '#')

            res[mailing_id] = res[mailing_id].replace(
                'class="o_snippet_view_in_browser"',
                'class="o_snippet_view_in_browser" style="display: none;"'
            )

            if res_id:
                res[mailing_id] = mailing._render_field('body_html', [res_id], post_process=True)[res_id]

            return request.render('mass_mailing.view', {
                    'body': res[mailing_id],
                })

        return request.redirect('/web')

    @http.route('/r/<string:code>/m/<int:mailing_trace_id>', type='http', auth="public")
    def full_url_redirect(self, code, mailing_trace_id, **post):
        # don't assume geoip is set, it is part of the website module
        # which mass_mailing doesn't depend on
        country_code = request.session.get('geoip', False) and request.session.geoip.get('country_code', False)

        request.env['link.tracker.click'].sudo().add_click(
            code,
            ip=request.httprequest.remote_addr,
            country_code=country_code,
            mailing_trace_id=mailing_trace_id
        )
        return request.redirect(request.env['link.tracker'].get_url_from_code(code), code=301, local=False)

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

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\ir_attachment_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Snippets' Default Images (to be replaced by themes) -->
        <record id="mass_mailing.s_media_list_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_media_list_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_media_list_1.jpg</field>
        </record>
        <record id="mass_mailing.s_media_list_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_media_list_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_media_list_2.jpg</field>
        </record>
        <record id="mass_mailing.s_media_list_default_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_media_list_default_image_3.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_media_list_3.jpg</field>
        </record>
        <record id="mass_mailing.s_company_team_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_1.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_team_member_1.png</field>
        </record>
        <record id="mass_mailing.s_company_team_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_2.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_team_member_2.png</field>
        </record>
        <record id="mass_mailing.s_company_team_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_3.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_team_member_3.png</field>
        </record>
        <record id="mass_mailing.s_company_team_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_4.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_team_member_4.png</field>
        </record>
        <record id="mass_mailing.s_reference_demo_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_1.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_1.png</field>
        </record>
        <record id="mass_mailing.s_reference_demo_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_2.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_2.png</field>
        </record>
        <record id="mass_mailing.s_reference_demo_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_3.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_3.png</field>
        </record>
        <record id="mass_mailing.s_reference_demo_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_4.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_4.png</field>
        </record>
        <record id="mass_mailing.s_reference_demo_image_5" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_5.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_5.png</field>
        </record>
        <record id="mass_mailing.s_reference_default_image_6" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_default_image_6.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_6.png</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/furniture.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/clothes.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_3.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/books.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_4.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/essentials_oils.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_5" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_5.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/services.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_6" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_6.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/multimedia.jpg</field>
        </record>
        <record id="mass_mailing.s_blockquote_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_blockquote_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_team_member_2.png</field>
        </record>
        <record id="mass_mailing.s_blockquote_cover_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_blockquote_cover_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_blockquote_cover.jpg</field>
        </record>
        <record id="mass_mailing.s_masonry_block_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_masonry_block_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_masonry_block_1.jpg</field>
        </record>
        <record id="mass_mailing.s_masonry_block_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_masonry_block_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_cover.jpg</field>
        </record>
    </data>
</odoo>

```

## File: data\mailing_data_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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
		    <t t-if="mailing.ab_testing_winner_selection == 'manual'">Don't forget to send your prefered version</t>
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
                                <td style="width: 70%;padding: 10px 0; text-align: center; border: 1px solid #e7e7e7;">
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

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!--
    Reference: https://litmus.com/community/learning/24-how-to-code-a-responsive-email-from-scratch
    https://www.campaignmonitor.com/css/link-element/link-in-head/
    -->
    <template id="mass_mailing_mail_layout">
        &lt;!DOCTYPE html&gt;
        <html xmlns="http://www.w3.org/1999/xhtml">
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
                <meta name="format-detection" content="telephone=no"/>
                <meta name="viewport" content="width=device-width; initial-scale=1.0; maximum-scale=1.0; user-scalable=no;"/>
                <meta http-equiv="X-UA-Compatible" content="IE=9; IE=8; IE=7; IE=EDGE" />

                <t t-call="mass_mailing.mass_mailing_mail_style"/>
            </head>
            <body>
                <t t-out="body"/>
            </body>
        </html>
    </template>
</data></odoo>

```

## File: data\mass_mailing_data.xml

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
        <record id="mailing_list_data" model="mailing.list">
            <field name="name">Newsletter</field>
        </record>
        <record id="mass_mailing_contact_0" model="mailing.contact">
            <field name="name" model="res.users" eval="obj().env.ref('base.user_admin').name"/>
            <field name="email" model="res.users" eval="obj().env.ref('base.user_admin').email"/>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_data')])]"/>
        </record>
        <!-- Snippets' Default Images (to be replaced by themes) -->
        <record id="mass_mailing.s_media_list_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_media_list_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_media_list_1.jpg</field>
        </record>
        <record id="mass_mailing.s_media_list_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_media_list_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_media_list_2.jpg</field>
        </record>
        <record id="mass_mailing.s_media_list_default_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_media_list_default_image_3.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_media_list_3.jpg</field>
        </record>
        <record id="mass_mailing.s_company_team_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_1.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_team_member_1.png</field>
        </record>
        <record id="mass_mailing.s_company_team_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_2.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_team_member_2.png</field>
        </record>
        <record id="mass_mailing.s_company_team_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_3.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_team_member_3.png</field>
        </record>
        <record id="mass_mailing.s_company_team_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_company_team_image_4.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_team_member_4.png</field>
        </record>
        <record id="mass_mailing.s_reference_demo_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_1.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_1.png</field>
        </record>
        <record id="mass_mailing.s_reference_demo_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_2.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_2.png</field>
        </record>
        <record id="mass_mailing.s_reference_demo_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_3.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_3.png</field>
        </record>
        <record id="mass_mailing.s_reference_demo_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_4.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_4.png</field>
        </record>
        <record id="mass_mailing.s_reference_demo_image_5" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_demo_image_5.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_5.png</field>
        </record>
        <record id="mass_mailing.s_reference_default_image_6" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_reference_default_image_6.png</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_references_6.png</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/furniture.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/clothes.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_3" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_3.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/books.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_4" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_4.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/essentials_oils.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_5" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_5.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/services.jpg</field>
        </record>
        <record id="mass_mailing.s_product_list_default_image_6" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_product_list_default_image_6.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/multimedia.jpg</field>
        </record>
        <record id="mass_mailing.s_blockquote_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_blockquote_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_team_member_2.png</field>
        </record>
        <record id="mass_mailing.s_blockquote_cover_default_image" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_blockquote_cover_default_image.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_blockquote_cover.jpg</field>
        </record>
        <record id="mass_mailing.s_masonry_block_default_image_1" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_masonry_block_default_image_1.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_masonry_block_1.jpg</field>
        </record>
        <record id="mass_mailing.s_masonry_block_default_image_2" model="ir.attachment">
            <field name="public" eval="True"/>
            <field name="name">s_masonry_block_default_image_2.jpg</field>
            <field name="type">url</field>
            <field name="url">/mass_mailing/static/src/img/snippets_demo/s_cover.jpg</field>
        </record>
    </data>
</odoo>

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
                        <div class="container o_mail_table_styles o_mail_h_padding" style="padding:0 20px 0 20px;width:100%;border-collapse:separate;">
                            <div class="row">
                                <div valign="center" width="30%" class="col text-center o_mail_v_padding pb0" style="padding:20px 0 0px 0;vertical-align:middle;text-align:center;">
                                    <a href="http://www.example.com" style="text-decoration:none;font-weight:bold;background-color:transparent;color:rgb(100, 89, 116);">
                                        <img border="0" src="/mass_mailing/static/src/img/theme_default/s_default_image_logo.png" style="border-style:none;height:auto;vertical-align:middle;max-width:400px;width:auto"/> ​
                                    </a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="o_mail_block_footer_separator" data-snippet="s_hr" style="margin:0 20px 0 20px;">
                    <div class="o_mail_snippet_general" style="margin:0px auto 0px auto;background-color:rgb(255, 255, 255);max-width:600px;width:100%;">
                        <div class="container o_mail_table_styles o_mail_full_width_padding" style="width:100%;border-collapse:separate;">
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
                        <div class="container o_mail_table_styles" style="width:100%;border-collapse:separate;">
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
                        <div align="center" class="o_mail_table_styles container o_mail_full_width_padding" style="border-style:solid none none none;padding:20px 0 20px 0;border-top-color:rgb(245, 245, 245);border-top-width:2px;width:100%;border-collapse:separate;">
                            <div class="row">
                                <div class="col o_mail_footer_links o_default_snippet_text" style="padding:10px 0 10px 0;text-align:center;vertical-align:middle;">
                                    <a href="/unsubscribe_from_list" class="btn btn-link o_default_snippet_text" style="text-decoration:none;border-radius:0.25rem;border-style:solid;padding:0px;cursor:pointer;line-height:1.5;font-size:12px;border-left-color:transparent;border-bottom-color:transparent;border-right-color:transparent;border-top-color:transparent;border-left-width:1px;border-bottom-width:1px;border-right-width:1px;border-top-width:1px;user-select:none;vertical-align:middle;white-space:nowrap;text-align:center;font-weight:bold;display:inline-block;background-color:transparent;color:rgb(100, 89, 116);">Unsubscribe</a> |

                                    <a href="/contactus" class="btn btn-link o_default_snippet_text" style="text-decoration:none;border-radius:0.25rem;border-style:solid;padding:0px;cursor:pointer;line-height:1.5;font-size:12px;border-left-color:transparent;border-bottom-color:transparent;border-right-color:transparent;border-top-color:transparent;border-left-width:1px;border-bottom-width:1px;border-right-width:1px;border-top-width:1px;user-select:none;vertical-align:middle;white-space:nowrap;text-align:center;font-weight:bold;display:inline-block;background-color:transparent;color:rgb(100, 89, 116);">Contact</a>
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
    <div align="center" class="container o_mail_table_styles o_mail_full_width_padding" style="width:100%;border-collapse:separate;">
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

import hashlib
import hmac
import logging
import lxml
import random
import re
import threading
import werkzeug.urls
from ast import literal_eval
from dateutil.relativedelta import relativedelta
from werkzeug.urls import url_join

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError, ValidationError
from odoo.osv import expression

_logger = logging.getLogger(__name__)

# Syntax of the data URL Scheme: https://tools.ietf.org/html/rfc2397#section-3
# Used to find inline images
image_re = re.compile(r"data:(image/[A-Za-z]+);base64,(.*)")


class MassMailing(models.Model):
    """ Mass Mailing models the sending of emails to a list of recipients for a mass mailing campaign."""
    _name = 'mailing.mailing'
    _description = 'Mass Mailing'
    _inherit = ['mail.thread', 'mail.activity.mixin', 'mail.render.mixin']
    _order = 'sent_date DESC'
    _inherits = {'utm.source': 'source_id'}
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
    subject = fields.Char('Subject', help='Subject of your Mailing', required=True, translate=False)
    preview = fields.Char(
        'Preview', translate=False,
        help='Catchy preview sentence that encourages recipients to open this email.\n'
             'In most inboxes, this is displayed next to the subject.\n'
             'Keep it empty if you prefer the first characters of your email content to appear instead.')
    email_from = fields.Char(string='Send From', required=True, store=True, readonly=False, compute='_compute_email_from',
                             default=lambda self: self.env.user.email_formatted)
    sent_date = fields.Datetime(string='Sent Date', copy=False)

    schedule_type = fields.Selection([('now', 'Send now'), ('scheduled', 'Send on')], string='Schedule',
                                     default='now', required=True, readonly=True,
                                     states={'draft': [('readonly', False)], 'in_queue': [('readonly', False)]})
    schedule_date = fields.Datetime(string='Scheduled for', tracking=True, readonly=True,
                                    states={'draft': [('readonly', False)], 'in_queue': [('readonly', False)]},
                                    compute='_compute_schedule_date', store=True, copy=True)
    calendar_date = fields.Datetime('Calendar Date', compute='_compute_calendar_date', store=True, copy=False,
        help="Date at which the mailing was or will be sent.")
    # don't translate 'body_arch', the translations are only on 'body_html'
    body_arch = fields.Html(string='Body', translate=False, sanitize=False)
    body_html = fields.Html(string='Body converted to be sent by mail', render_engine='qweb', sanitize=False)
    is_body_empty = fields.Boolean(compute="_compute_is_body_empty",
                                   help='Technical field used to determine if the mail body is empty')
    attachment_ids = fields.Many2many('ir.attachment', 'mass_mailing_ir_attachments_rel',
        'mass_mailing_id', 'attachment_id', string='Attachments')
    keep_archives = fields.Boolean(string='Keep Archives')
    campaign_id = fields.Many2one('utm.campaign', string='UTM Campaign', index=True)
    source_id = fields.Many2one('utm.source', string='Source', required=True, ondelete='cascade',
                                help="This is the link source, e.g. Search Engine, another domain, or name of email list")
    medium_id = fields.Many2one(
        'utm.medium', string='Medium',
        compute='_compute_medium_id', readonly=False, store=True,
        help="UTM Medium: delivery method (email, sms, ...)")
    state = fields.Selection([('draft', 'Draft'), ('in_queue', 'In Queue'), ('sending', 'Sending'), ('done', 'Sent')],
        string='Status', required=True, tracking=True, copy=False, default='draft', group_expand='_group_expand_states')
    color = fields.Integer(string='Color Index')
    user_id = fields.Many2one('res.users', string='Responsible', tracking=True,  default=lambda self: self.env.user)
    # mailing options
    mailing_type = fields.Selection([('mail', 'Email')], string="Mailing Type", default="mail", required=True)
    mailing_type_description = fields.Char('Mailing Type Description', compute="_compute_mailing_type_description")
    reply_to_mode = fields.Selection([
        ('update', 'Recipient Followers'), ('new', 'Specified Email Address')],
        string='Reply-To Mode', compute='_compute_reply_to_mode',
        readonly=False, store=True,
        help='Thread: replies go to target document. Email: replies are routed to a given email.')
    reply_to = fields.Char(
        string='Reply To', compute='_compute_reply_to', readonly=False, store=True,
        help='Preferred Reply-To Address')
    # recipients
    mailing_model_real = fields.Char(string='Recipients Real Model', compute='_compute_mailing_model_real')
    mailing_model_id = fields.Many2one(
        'ir.model', string='Recipients Model', ondelete='cascade', required=True,
        domain=[('is_mailing_enabled', '=', True)],
        default=lambda self: self.env.ref('mass_mailing.model_mailing_list').id)
    mailing_model_name = fields.Char(
        string='Recipients Model Name', related='mailing_model_id.model',
        readonly=True, related_sudo=True)
    mailing_domain = fields.Char(
        string='Domain', compute='_compute_mailing_domain',
        readonly=False, store=True)
    mail_server_available = fields.Boolean(
        compute='_compute_mail_server_available',
        help="Technical field used to know if the user has activated the outgoing mail server option in the settings")
    mail_server_id = fields.Many2one('ir.mail_server', string='Mail Server',
        default=_get_default_mail_server_id,
        help="Use a specific mail server in priority. Otherwise Odoo relies on the first outgoing mail server available (based on their sequencing) as it does for normal mails.")
    contact_list_ids = fields.Many2many('mailing.list', 'mail_mass_mailing_list_rel', string='Mailing Lists')
    # A/B Testing
    ab_testing_completed = fields.Boolean(related='campaign_id.ab_testing_completed', store=True)
    ab_testing_description = fields.Html('A/B Testing Description', compute="_compute_ab_testing_description")
    ab_testing_enabled = fields.Boolean(string='Allow A/B Testing', default=False,
        help='If checked, recipients will be mailed only once for the whole campaign. '
             'This lets you send different mailings to randomly selected recipients and test '
             'the effectiveness of the mailings, without causing duplicate messages.')
    ab_testing_mailings_count = fields.Integer(related="campaign_id.ab_testing_mailings_count")
    ab_testing_pc = fields.Integer(string='A/B Testing percentage',
        help='Percentage of the contacts that will be mailed. Recipients will be chosen randomly.', default=10)
    ab_testing_schedule_datetime = fields.Datetime(related="campaign_id.ab_testing_schedule_datetime", readonly=False,
        default=lambda self: fields.Datetime.now() + relativedelta(days=1))
    ab_testing_winner_selection = fields.Selection(related="campaign_id.ab_testing_winner_selection",
        default="opened_ratio", readonly=False, copy=True)

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

    @api.depends('mail_server_id')
    def _compute_email_from(self):
        user_email = self.env.user.email_formatted
        notification_email = self.env['ir.mail_server']._get_default_from_address()

        for mailing in self:
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
        self.flush()
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
        for mailing in self:
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
    def _compute_mailing_model_real(self):
        for mailing in self:
            mailing.mailing_model_real = (mailing.mailing_model_id.model != 'mailing.list') and mailing.mailing_model_id.model or 'mailing.contact'

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

    @api.depends('mailing_model_id', 'contact_list_ids', 'mailing_type')
    def _compute_mailing_domain(self):
        for mailing in self:
            if not mailing.mailing_model_id:
                mailing.mailing_domain = ''
            else:
                mailing.mailing_domain = repr(mailing._get_default_mailing_domain())

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
        now = fields.Datetime.now()
        ab_testing_cron = self.env.ref('mass_mailing.ir_cron_mass_mailing_ab_testing').sudo()
        for values in vals_list:
            if values.get('subject') and not values.get('name'):
                values['name'] = "%s %s" % (values['subject'], now)
            if values.get('body_html'):
                values['body_html'] = self._convert_inline_images_to_urls(values['body_html'])
            if values.get('ab_testing_schedule_datetime'):
                at = fields.Datetime.from_string(values['ab_testing_schedule_datetime'])
                ab_testing_cron._trigger(at=at)
        mailings = super().create(vals_list)
        campaign_vals = [
            mailing._get_default_ab_testing_campaign_values()
            for mailing in mailings
            if mailing.ab_testing_enabled and not mailing.campaign_id
        ]
        self.env['utm.campaign'].create(campaign_vals)

        mailings._fix_attachment_ownership()

        return mailings

    def write(self, values):
        if values.get('body_html'):
            values['body_html'] = self._convert_inline_images_to_urls(values['body_html'])
        # When ab_testing_enabled is checked we create a campaign if there is none set.
        if values.get('ab_testing_enabled') and not values.get('campaign_id'):
            # Compute the values of the A/B test campaign based on the first mailing
            values['campaign_id'] = self.env['utm.campaign'].create(self[0]._get_default_ab_testing_campaign_values(values)).id
        # If ab_testing is already enabled on a mailing and the campaign is removed, we raise a ValidationError
        if values.get('campaign_id') is False and any(mailing.ab_testing_enabled for mailing in self) and 'ab_testing_enabled' not in values:
            raise ValidationError(_("A campaign should be set when A/B test is enabled"))

        result = super(MassMailing, self).write(values)
        self._fix_attachment_ownership()

        if any(self.mapped('ab_testing_schedule_datetime')):
            schedule_date = min(m.ab_testing_schedule_datetime for m in self if m.ab_testing_schedule_datetime)
            ab_testing_cron = self.env.ref('mass_mailing.ir_cron_mass_mailing_ab_testing').sudo()
            ab_testing_cron._trigger(at=schedule_date)

        return result

    def _fix_attachment_ownership(self):
        for record in self:
            record.attachment_ids.write({'res_model': record._name, 'res_id': record.id})
        return self

    @api.returns('self', lambda value: value.id)
    def copy(self, default=None):
        self.ensure_one()
        default = dict(default or {},
                       name=_('%s (copy)', self.name),
                       contact_list_ids=self.contact_list_ids.ids)
        return super(MassMailing, self).copy(default=default)

    def _group_expand_states(self, states, domain, order):
        return [key for key, val in self._fields['state'].selection]

    # ------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------

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
        ctx = dict(self.env.context, default_mass_mailing_id=self.id)
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
        else:
            action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing.mailing_mailing_schedule_date_action")
            action['context'] = dict(self.env.context, default_mass_mailing_id=self.id)
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
        return {
            'name': model_name,
            'type': 'ir.actions.act_window',
            'view_mode': 'tree',
            'res_model': 'link.tracker',
            'domain': [('mass_mailing_id.id', '=', self.id)],
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
        if view_filter in ('reply', 'bounce'):
            found_traces = self.mailing_trace_ids.filtered(lambda trace: trace.trace_status == view_filter)
        elif view_filter == 'open':
            found_traces = self.mailing_trace_ids.filtered(lambda trace: trace.trace_status in ('open', 'reply'))
        elif view_filter == 'click':
            found_traces = self.mailing_trace_ids.filtered(lambda trace: trace.links_click_datetime)
        elif view_filter == 'delivered':
            found_traces = self.mailing_trace_ids.filtered(lambda trace: trace.trace_status in ('sent', 'open', 'reply'))
        elif view_filter == 'sent':
            found_traces = self.mailing_trace_ids.filtered(lambda trace: trace.sent_datetime)
        else:
            found_traces = self.env['mailing.trace']
        res_ids = found_traces.mapped('res_id')
        model_name = self.env['ir.model']._get(self.mailing_model_real).display_name
        return {
            'name': model_name,
            'type': 'ir.actions.act_window',
            'view_mode': 'tree',
            'res_model': self.mailing_model_real,
            'domain': [('id', 'in', res_ids)],
            'context': dict(self._context, create=False)
        }

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
            self.get_base_url(), 'mail/mailing/%(mailing_id)s/unsubscribe?%(params)s' % {
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
                'body': mailing._prepend_preview(mailing.body_html, mailing.preview),
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

            rendered_body = self.env['ir.qweb']._render(
                'digest.digest_mail_main',
                {
                    'body': tools.html_sanitize(link_trackers_body),
                    'company': mail_company,
                    'user': mail_user,
                    'display_mobile_banner': True,
                    ** mailing._prepare_statistics_email_values()
                },
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
        if self.mailing_type == 'mail':
            return _('Emails')

    # ------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------

    def _get_default_mailing_domain(self):
        mailing_domain = []
        if hasattr(self.env[self.mailing_model_name], '_mailing_get_default_domain'):
            mailing_domain = self.env[self.mailing_model_name]._mailing_get_default_domain(self)

        if self.mailing_type == 'mail' and 'is_blacklisted' in self.env[self.mailing_model_name]._fields:
            mailing_domain = expression.AND([[('is_blacklisted', '=', False)], mailing_domain])

        return mailing_domain

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

    def _convert_inline_images_to_urls(self, body_html):
        """
        Find inline base64 encoded images, make an attachement out of
        them and replace the inline image with an url to the attachement.
        """
        base64_in_element_regex = re.compile(r"""
                # Group 1: element until the base64 data
                (<[^>]+\b(?:src="|style=["'][^"']+\burl\((?:&\#34;|"|'|&quot;)?))
                data:image/[A-Za-z]+;base64,
                (.*?) # Group 2: base64 image
                ((?:(?:&\#34;|"|'|&quot;)?\))|") # Group 3: closing the property or attribute
            """, re.VERBOSE)
        do_match = True
        while do_match:
            (body_html, do_match) = re.subn(base64_in_element_regex, lambda x: x[1] + self._image_to_url(x[2].encode()) + x[3], body_html)
        return body_html

    def _image_to_url(self, b64image: bytes):
        """Store an image in an attachement and returns an url"""
        attachment = self.env['ir.attachment'].create({
            'datas': b64image,
            'name': "cropped_image_mailing_{}".format(self.id),
            'type': 'binary',})

        attachment.generate_access_token()

        return '/web/image/%s?access_token=%s' % (
            attachment.id, attachment.access_token)

```

## File: models\mailing_contact.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools
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

    def default_get(self, fields):
        """ When coming from a mailing list we may have a default_list_ids context
        key. We should use it to create subscription_list_ids default value that
        are displayed to the user as list_ids is not displayed on form view. """
        res = super(MassMailingContact, self).default_get(fields)
        if 'subscription_list_ids' in fields and not res.get('subscription_list_ids'):
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
    subscription_list_ids = fields.One2many('mailing.contact.subscription', 'contact_id', string='Subscription Information')
    country_id = fields.Many2one('res.country', string='Country')
    tag_ids = fields.Many2many('res.partner.category', string='Tags')
    opt_out = fields.Boolean('Opt Out', compute='_compute_opt_out', search='_search_opt_out',
                             help='Opt out flag for a specific mailing list.'
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
        else:
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

        if default_list_ids:
            for vals in vals_list:
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

```

## File: models\mailing_list.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models
from odoo.exceptions import UserError


class MassMailingList(models.Model):
    """Model of a contact list. """
    _name = 'mailing.list'
    _order = 'name'
    _description = 'Mailing List'
    _mailing_enabled = True
    # As this model has his own data merge, avoid to enable the generic data_merge on that model.
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
    mailing_ids = fields.Many2many('mailing.mailing', 'mail_mass_mailing_list_rel', string='Mass Mailings', copy=False)
    subscription_ids = fields.One2many(
        'mailing.contact.subscription', 'list_id', string='Subscription Information',
        copy=True, depends=['contact_ids'])
    is_public = fields.Boolean(default=True, help="The mailing list can be accessible by recipient in the unsubscription"
                                                  " page to allows him to update his subscription preferences.")

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
        self.env['mailing.contact'].flush(['email', 'email_normalized'])
        self.env['mailing.contact.subscription'].flush(['contact_id', 'opt_out', 'list_id'])
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
        self.flush()
        self.invalidate_cache()
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
                    for field in self._get_contact_statistics_fields().keys()
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
            'contact_count_blacklisted': f'''
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
    mail_mail_id = fields.Many2one('mail.mail', string='Mail', index=True)
    mail_mail_id_int = fields.Integer(
        string='Mail ID (tech)',
        help='ID of the related mail_mail. This field is an integer field because '
             'the related mail_mail can be deleted separately from its statistics. '
             'However the ID is needed for several action and controllers.',
        index=True,
    )
    email = fields.Char(string="Email", help="Normalized email address")
    message_id = fields.Char(string='Message-ID', help="Technical field for the email Message-ID (RFC 2392)")
    medium_id = fields.Many2one(related='mass_mailing_id.medium_id')
    source_id = fields.Many2one(related='mass_mailing_id.source_id')
    # document
    model = fields.Char(string='Document model', required=True)
    res_id = fields.Many2oneReference(string='Document ID', model_field='model', required=True)
    # campaign data
    mass_mailing_id = fields.Many2one('mailing.mailing', string='Mailing', index=True, ondelete='cascade')
    campaign_id = fields.Many2one(
        related='mass_mailing_id.campaign_id',
        string='Campaign',
        store=True, readonly=True, index=True)
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

    @api.model_create_multi
    def create(self, values_list):
        """ Override mail_mail creation to create an entry in mail.mail.statistics """
        # TDE note: should be after 'all values computed', to have values (FIXME after merging other branch holding create refactoring)
        mails = super(MailMail, self).create(values_list)
        for mail, values in zip(mails, values_list):
            if values.get('mailing_trace_ids'):
                mail.mailing_trace_ids.write({'message_id': mail.message_id})
        return mails

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

    group_mass_mailing_campaign = fields.Boolean(string="Mailing Campaigns", implied_group='mass_mailing.group_mass_mailing_campaign', help="""This is useful if your marketing campaigns are composed of several emails""")
    mass_mailing_outgoing_mail_server = fields.Boolean(string="Dedicated Server", config_parameter='mass_mailing.outgoing_mail_server',
        help='Use a specific mail server in priority. Otherwise Odoo relies on the first outgoing mail server available (based on their sequencing) as it does for normal mails.')
    mass_mailing_mail_server_id = fields.Many2one('ir.mail_server', string='Mail Server', config_parameter='mass_mailing.mail_server_id')
    show_blacklist_buttons = fields.Boolean(string="Blacklist Option when Unsubscribing",
                                                 config_parameter='mass_mailing.show_blacklist_buttons',
                                                 help="""Allow the recipient to manage himself his state in the blacklist via the unsubscription page.""")

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

## File: models\utm.py

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
        string='Mass Mailings')
    mailing_mail_count = fields.Integer('Number of Mass Mailing', compute="_compute_mailing_mail_count")

    # A/B Testing
    ab_testing_mailings_count = fields.Integer("A/B Test Mailings #", compute="_compute_mailing_mail_count")
    ab_testing_completed = fields.Boolean("A/B Testing Campaign Finished")
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
            mailing_data = self.env['mailing.mailing'].read_group(
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
            campaign.mailing_mail_count = sum(mapped_data.get(campaign.id, []))
            campaign.ab_testing_mailings_count = sum(ab_testing_mapped_data.get(campaign.id, []))

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

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_model
from . import link_tracker
from . import mailing_contact
from . import mailing_list
from . import mailing_trace
from . import mailing
from . import mail_mail
from . import mail_render_mixin
from . import mail_thread
from . import res_config_settings
from . import res_partner
from . import res_users
from . import utm
from . import res_company

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

       <menuitem name="Reporting" id="menu_mass_mailing_report" sequence="99"
            parent="mass_mailing_menu_root"
            action="mailing_trace_report_action_mail"
            groups="mass_mailing.group_mass_mailing_user"/>
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
access_mailing_contact_subscription_mm_user,access.mailing.contact.subscription.mm.user,model_mailing_contact_subscription,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_list_mm_user,access.mailing.list.mm.user,model_mailing_list,mass_mailing.group_mass_mailing_user,1,1,1,1
access_utm_stage,utm.stage,utm.model_utm_stage,mass_mailing.group_mass_mailing_user,1,1,1,1
access_utm_campaign_mass_mailing_user,utm.campaign,utm.model_utm_campaign,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_mailing_mm_user,access.mailing.mailing.mm.user,model_mailing_mailing,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_mailing_system,access.mailing.mailing.system,model_mailing_mailing,base.group_system,1,1,1,1
access_mailing_trace_user,mailing.trace.user,model_mailing_trace,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_trace_mm_user,access.mailing.trace.mm.user,model_mailing_trace,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_trace_report_mm_user,access.mailing.trace.report.mm.user,model_mailing_trace_report,mass_mailing.group_mass_mailing_user,1,0,0,0
access_utm_source,access_utm_source,utm.model_utm_source,mass_mailing.group_mass_mailing_user,1,1,1,0
access_ir_mail_server,access_ir_mail_server,base.model_ir_mail_server,mass_mailing.group_mass_mailing_user,1,0,0,0
access_ir_model,access_ir_model,base.model_ir_model,mass_mailing.group_mass_mailing_user,1,0,0,0
access_mail_blacklist_mass_mailing_user,access.mail.blacklist.mass_mailing_user,mail.model_mail_blacklist,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mail_blacklist_remove_mass_mailing_user,acesss.mail.blacklist.remove.mass_mailing_user,mail.model_mail_blacklist_remove,mass_mailing.group_mass_mailing_user,1,1,1,1
access_link_tracker_mailing,access.link.tracker.mailing,link_tracker.model_link_tracker,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_list_merge,access.mailing.list.merge,model_mailing_list_merge,mass_mailing.group_mass_mailing_user,1,1,1,0
access_mailing_mailing_test,access.mailing.mailing.test,model_mailing_mailing_test,mass_mailing.group_mass_mailing_user,1,1,1,0
access_mailing_contact_to_list,access.mailing.contact.to.list,model_mailing_contact_to_list,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_schedule_date,access.mailing.schedule.date,model_mailing_mailing_schedule_date,mass_mailing.group_mass_mailing_user,1,1,1,1

```

## File: security\mass_mailing_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.module.category" id="base.module_category_marketing_email_marketing">
        <field name="sequence">19</field>
        <field name="description">Helps you manage your mass mailing to design
professional emails and reuse templates.</field>
    </record>

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
        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('mass_mailing.group_mass_mailing_user'))]"/>
        </record>

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

## File: static\src\img\snippets_demo\size_large.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="12" viewBox="0 0 23 12">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_7" transform="translate(-203 -5)">
      <g class="size_large" transform="translate(203 5)">
        <path fill="#B8B8B8" d="M23 0v12H0V0h23zm-1 1H1v10h21V1z" class="o_subdle"/>
        <rect width="19" height="8" x="2" y="2" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_demo\size_medium.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="12" viewBox="0 0 23 12">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_7" transform="translate(-170 -5)">
      <g class="size_medium" transform="translate(170 5)">
        <path fill="#B8B8B8" d="M23 0v12H0V0h23zm-4 2H4v8h15V2z" class="o_subdle"/>
        <rect width="13" height="6" x="5" y="3" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

```

## File: static\src\img\snippets_demo\size_small.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="23" height="12" viewBox="0 0 23 12">
  <g fill="none" fill-rule="evenodd" class="symbols">
    <g class="3_buttons_copy_7" transform="translate(-137 -5)">
      <g class="size_small" transform="translate(137 5)">
        <path fill="#B8B8B8" d="M23 0v12H0V0h23zm-8 3H8v6h7V3z" class="o_subdle"/>
        <rect width="5" height="4" x="9" y="4" fill="#FFF" class="o_graphic"/>
      </g>
    </g>
  </g>
</svg>

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

## File: static\src\js\mailing_mailing_view_form_full_width.js

```javascript
/** @odoo-module **/

import FormView from 'web.FormView';
import FormController from 'web.FormController';
import FormRenderer from 'web.FormRenderer';
import { bus, _t } from 'web.core';
import viewRegistry from 'web.view_registry';
import config from 'web.config';

const MassMailingFullWidthFormController = FormController.extend({
    custom_events: _.extend({}, FormController.prototype.custom_events,{
        iframe_updated: '_onIframeUpdated',
    }),

    /**
     * @override
     */
    init() {
        this._super(...arguments);
        bus.on('DOM_updated', this, this._onDomUpdated);
        this._resizeObserver =  new ResizeObserver(entries => {
            // We wrap this in requestAnimationFrame to greatly mitigate
            // the "ResizeObserver loop limit exceeded" error.
            window.requestAnimationFrame(() => {
                if (!Array.isArray(entries) || !entries.length) {
                    return;
                }
                this._onResizeIframeContents(entries);
            });
        });
    },
    /**
     * @override
     */
    destroy() {
        bus.off('DOM_updated', this, this._onDomUpdated);
        this._super(...arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Resize the mailing editor's iframe container so its height fits its
     * contents. This needs to be called whenever the iframe's contents might
     * have changed, eg. when adding/removing content to/from it or when a
     * template is picked.
     *
     * @private
     */
    _resizeMailingEditorIframe() {
        const VERTICAL_OFFSET = 12; // Vertical offset picked for visual design purposes.
        const minHeight = $(window).height() - Math.abs(this.$iframe.offset().top) - (VERTICAL_OFFSET / 2);
        const $iframeDoc = this.$iframe.contents();
        const $themeSelectorNew = $iframeDoc.find('.o_mail_theme_selector_new');
        if ($themeSelectorNew.length) {
            this.$iframe.height(Math.max($themeSelectorNew[0].scrollHeight + VERTICAL_OFFSET, minHeight));
        } else {
            const ref = $iframeDoc.find('#iframe_target')[0];
            if (ref) {
                this.$iframe.css({
                    height: this._isFullScreen()
                        ? $(window).height()
                        : Math.max(ref.scrollHeight + VERTICAL_OFFSET, minHeight),
                });
            }
        }
    },
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
            $sidebar.css({
                top: '',
                bottom: '',
            });
        } else {
            const iframeTop = this.$iframe.offset().top;
            $sidebar.css({
                height: '',
                top: Math.max(0, this.$('.o_content').offset().top - iframeTop),
                bottom: this.$iframe.height() - windowHeight + iframeTop,
            });
        }
    },
    /**
     * Return true if the mailing editor is in full screen mode, false
     * otherwise.
     *
     * @private
     * @returns {boolean}
     */
    _isFullScreen() {
        return window.top.document.body.classList.contains('o_field_widgetTextHtml_fullscreen');
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Assume the iframe was updated on each dom_updated event.
     *
     * @private
     */
    _onDomUpdated() {
        const data = { $iframe: this.$('iframe.wysiwyg_iframe:visible, iframe.o_readonly:visible') };
        this._onIframeUpdated({ data });
    },
    /**
     * Resize the given iframe so its height fits its contents and initialize a
     * resize observer to resize on each size change in its contents.
     * This also ensures the contents of the sidebar remain visible no matter
     * how much we resize the iframe and scroll down.
     *
     * @private
     * @param {JQuery} ev.data.$iframe
     */
    _onIframeUpdated(ev) {
        const $iframe = ev.data.$iframe;
        if (!$iframe.length || !$iframe.contents().length) {
            return;
        }
        const hasIframeChanged = $iframe !== this.$iframe;
        this.$iframe = $iframe;
        this._resizeMailingEditorIframe();

        const $iframeDoc = $iframe.contents();
        const iframeTarget = $iframeDoc.find('#iframe_target');
        if (hasIframeChanged) {
            $iframeDoc.find('body').on('click', '.o_fullscreen_btn', this._onToggleFullscreen.bind(this));
            this.$('.o_content').on('scroll', this._repositionMailingEditorSidebar.bind(this));
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
    },
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
        const $iframeDoc = this.$iframe.contents();
        const iframeTarget = $iframeDoc.find('#iframe_target');
        const isFullscreen = this._isFullScreen();
        iframeTarget.css({
            display: isFullscreen ? '' : 'flex',
            'flex-direction': isFullscreen ? '' : 'column',
        });
        const wysiwyg = $iframeDoc.find('.note-editable').data('wysiwyg');
        if (wysiwyg && wysiwyg.snippetsMenu) {
            // Restore the appropriate scrollable depending on the mode.
            this._$scrollable = this._$scrollable || wysiwyg.snippetsMenu.$scrollable;
            wysiwyg.snippetsMenu.$scrollable = isFullscreen ? $iframeDoc.find('.note-editable') : this._$scrollable;
        }
        this._repositionMailingEditorSidebar();
    },
    /**
     * Resize the iframe and reposition the sidebar whenever the contents of the
     * iframe change height.
     *
     * @private
     */
    _onResizeIframeContents() {
        this._resizeMailingEditorIframe();
        this._repositionMailingEditorSidebar();
    },
});

const MassMailingFullWidthFormRenderer = FormRenderer.extend({
    /**
     * Overload the rendering of the header in order to add a child to it: move
     * the alert after the statusbar.
     *
     * @private
     * @override
     */
    _renderTagHeader: function (node) {
        const $statusbar = this._super(...arguments);
        const alert = node.children.find(child => child.tag === "div" && child.attrs.role === "alert");
        const $alert = this._renderGenericTag(alert);
        $statusbar.find('.o_statusbar_buttons').after($alert);
        return $statusbar;
    },
    /**
     * Increase the default number of button boxes before folding since the form
     * without sheet is a lot bigger and more space is available for them.
     *
     * @private
     * @override
     */
    _renderButtonBoxNbButtons: function () {
        return [2, 2, 2, 4, 6, 7][config.device.size_class] || 10;
    },
});

export const MassMailingFullWidthFormView = FormView.extend({
    config: Object.assign({}, FormView.prototype.config, {
        Controller: MassMailingFullWidthFormController,
        Renderer: MassMailingFullWidthFormRenderer,
    }),
});

viewRegistry.add('mailing_mailing_view_form_full_width', MassMailingFullWidthFormView);

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

                this.$('.form-group .o_btn_preview.btn-' + type)
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

## File: static\src\js\mass_mailing_snippets.js

```javascript
odoo.define('mass_mailing.snippets.options', function (require) {
"use strict";

var options = require('web_editor.snippets.options');
const {ColorpickerWidget} = require('web.Colorpicker');
const {_t} = require('web.core');

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
options.registry.BackgroundImage = options.registry.BackgroundImage.extend({
    start: function () {
        this._super();
        if (this.snippets && this.snippets.split('.')[0] === "mass_mailing") {
            var $table_target = this.$target.find('table:first');
            if ($table_target.length) {
                this.$target = $table_target;
            }
        }
    }
});

options.registry.ImageTools.include({

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async updateUIVisibility() {
        await this._super(...arguments);

        // Transform is _very_ badly supported in mail clients. Hide the option.
        const transformEl = this.el.querySelector('[data-transform="true"]');
        if (transformEl) {
            transformEl.classList.toggle('d-none', true);
        }
    },
});

options.registry.ImageOptimize.include({

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async updateUIVisibility() {
        await this._super(...arguments);

        // The image shape option should work correctly with this update of the
        // ImageOptimize option but unfortunately, SVG support in mail clients
        // prevents the final rendering of the image. For now, we disable the
        // feature.
        const imgShapeContainerEl = this.el.querySelector('.o_we_image_shape');
        if (imgShapeContainerEl) {
            // Hidden from view as the feature is not yet supported in emails
            imgShapeContainerEl.classList.add('d-none');
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _getCSSColorValue(color) {
        const doc = this.options.document;
        if (doc && doc.querySelector('.o_mass_mailing_iframe') && !ColorpickerWidget.isCSSColor(color)) {
            const tempEl = doc.body.appendChild(doc.createElement('div'));
            tempEl.className = `bg-${color}`;
            const colorValue = window.getComputedStyle(tempEl).getPropertyValue("background-color").trim();
            tempEl.parentNode.removeChild(tempEl);
            return ColorpickerWidget.normalizeCSSColor(colorValue).replace(/"/g, "'");
        }
        return this._super(...arguments);
    },
    /**
     * @override
     */
    async _renderCustomWidgets(uiFragment) {
        await this._super(...arguments);

        const imgShapeTitleEl = uiFragment.querySelector('.o_we_image_shape we-title');
        if (imgShapeTitleEl) {
            const warningEl = document.createElement('i');
            warningEl.classList.add('fa', 'fa-exclamation-triangle', 'ml-1');
            warningEl.title = _t("Be aware that this option may not work on many mail clients");
            imgShapeTitleEl.appendChild(warningEl);
        }
    },
});

options.registry.Parallax = options.Class.extend({

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    async _computeWidgetVisibility(widgetName, params) {
        // Parallax is not supported in emails.
        return false;
    },
});

});

```

## File: static\src\js\mass_mailing_widget.js

```javascript
odoo.define('mass_mailing.FieldHtml', function (require) {
'use strict';

var config = require('web.config');
var core = require('web.core');
var FieldHtml = require('web_editor.field.html');
var fieldRegistry = require('web.field_registry');
var convertInline = require('web_editor.convertInline');

var _t = core._t;


var MassMailingFieldHtml = FieldHtml.extend({
    xmlDependencies: (FieldHtml.prototype.xmlDependencies || []).concat(["/mass_mailing/static/src/xml/mass_mailing.xml"]),
    jsLibs: [
        '/mass_mailing/static/src/js/mass_mailing_link_dialog_fix.js',
        '/mass_mailing/static/src/js/mass_mailing_snippets.js',
        '/mass_mailing/static/src/snippets/s_blockquote/options.js',
        '/mass_mailing/static/src/snippets/s_masonry_block/options.js',
        '/mass_mailing/static/src/snippets/s_media_list/options.js',
        '/mass_mailing/static/src/snippets/s_showcase/options.js',
        '/mass_mailing/static/src/snippets/s_rating/options.js',
    ],

    custom_events: _.extend({}, FieldHtml.prototype.custom_events, {
        snippets_loaded: '_onSnippetsLoaded',
    }),
    _wysiwygSnippetsActive: true,

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        if (!this.nodeOptions.snippets) {
            this.nodeOptions.snippets = 'mass_mailing.email_designer_snippets';
        }
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * Commit the change in 'style-inline' on an other field nodeOptions:
     *
     * - inline-field: fieldName to save the html value converted into inline code
     *
     * @override
     */
    commitChanges: async function () {
        var self = this;
        if (this.mode === 'readonly' || !this.isRendered) {
            return this._super();
        }
        var fieldName = this.nodeOptions['inline-field'];

        var $editable = this.wysiwyg.getEditable();
        if (this._$codeview && !this._$codeview.hasClass('d-none')) {
            $editable.html(self.value);
        }

        if (this.$content.find('.o_basic_theme').length) {
            this.$content.find('*').css('font-family', '');
        }

        await this.wysiwyg.cleanForSave();
        return this.wysiwyg.saveModifiedImages(this.$content).then(async function () {
            self._isDirty = self.wysiwyg.isDirty();
            await self._doAction();

            const $editorEnable = $editable.closest('.editor_enable');
            $editorEnable.removeClass('editor_enable');
            convertInline.toInline($editable, self.cssRules, self.wysiwyg.$iframe);
            $editorEnable.addClass('editor_enable');

            self.trigger_up('field_changed', {
                dataPointID: self.dataPointID,
                changes: _.object([fieldName], [self._unWrap($editable.html())])
            });

            $editable.html(self.value);
            if (self._isDirty && self.mode === 'edit') {
                return self._doAction();
            }
        });
    },
    /**
     * The html_frame widget is opened in an iFrame that has its URL encoded
     * with all the key/values returned by this method.
     *
     * Some fields can get very long values and we want to omit them for the URL building.
     *
     * @override
     */
    getDatarecord: function () {
        return _.omit(this._super(), [
            'mailing_domain',
            'contact_list_ids',
            'body_html',
            'attachment_ids'
        ]);
    },
    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Adds automatic editor messages on drag&drop zone elements.
     *
     * @private
     */
     _addEditorMessages: function () {
        const $editable = this.wysiwyg.getEditable().find('.o_editable');
        this.$editorMessageElements = $editable
            .not('[data-editor-message]')
            .attr('data-editor-message', _t('DRAG BUILDING BLOCKS HERE'));
        $editable.filter(':empty').attr('contenteditable', false);
    },
    /**
     * @override
     */
     _createWysiwygIntance: async function () {
        await this._super(...arguments);
        // Data is removed on save but we need the mailing and its body to be
        // named so they are handled properly by the snippets menu.
        this.$content.find('.o_layout').addBack().data('name', 'Mailing');
        // We don't want to drop snippets directly within the wysiwyg.
        this.$content.removeClass('o_editable');
        this.wysiwyg.getEditable().find('img').attr('loading', '');
    },
    /**
     * Returns true if the editable area is empty.
     *
     * @private
     * @param {JQuery} [$layout]
     * @returns {Boolean}
     */
    _editableAreaIsEmpty: function ($layout) {
        $layout = $layout || this.$content.find(".o_layout");
        var $mailWrapper = $layout.children(".o_mail_wrapper");
        var $mailWrapperContent = $mailWrapper.find('.o_mail_wrapper_td');
        if (!$mailWrapperContent.length) { // compatibility
            $mailWrapperContent = $mailWrapper;
        }
        var value;
        if ($mailWrapperContent.length > 0) {
            value = $mailWrapperContent.html();
        } else if ($layout.length) {
            value = $layout.html();
        } else {
            value = this.wysiwyg.getValue();
        }
        var blankEditable = "<p><br></p>";
        return value === "" || value === blankEditable;
    },
    /**
     * @override
     */
    _renderEdit: function () {
        this._wysiwygSnippetsActive = !$(this.value).is('.o_layout.o_basic_theme');
        if (!this.value) {
            this.value = this.recordData[this.nodeOptions['inline-field']];
        }
        return this._super.apply(this, arguments);
    },

    /**
     * @override
     */
    _renderReadonly: function () {
        if (!this.value) {
            this.value = this.recordData[this.nodeOptions['inline-field']];
        }
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     * @returns {JQuery}
     */
    _renderTranslateButton: function () {
        var fieldName = this.nodeOptions['inline-field'];
        if (_t.database.multi_lang && this.record.fields[fieldName].translate && this.res_id) {
            return $('<button>', {
                    type: 'button',
                    'class': 'o_field_translate fa fa-globe btn btn-primary',
                })
                .on('click', this._onTranslate.bind(this));
        }
        return $();
    },
    /**
     * Returns the selected theme, if any.
     *
     * @private
     * @param {Object} themesParams
     * @returns {false|Object}
     */
    _getSelectedTheme: function (themesParams) {
        var $layout = this.$content.find(".o_layout");
        var selectedTheme = false;
        if ($layout.length !== 0) {
            _.each(themesParams, function (themeParams) {
                if ($layout.hasClass(themeParams.className)) {
                    selectedTheme = themeParams;
                }
            });
        }
        return selectedTheme;
    },
    /**
     * Swap the previous theme's default images with the new ones.
     * (Redefine the `src` attribute of all images in a $container, depending on the theme parameters.)
     *
     * @private
     * @param {Object} themeParams
     * @param {JQuery} $container
     */
    _switchImages: function (themeParams, $container) {
        if (!themeParams) {
            return;
        }
        $container.find("img").each(function () {
            var $img = $(this);
            var src = $img.attr("src");

            var m = src.match(/^\/web\/image\/\w+\.s_default_image_(?:theme_[a-z]+_)?(.+)$/);
            if (!m) {
                m = src.match(/^\/\w+\/static\/src\/img\/(?:theme_[a-z]+\/)?s_default_image_(.+)\.[a-z]+$/);
            }
            if (!m) {
                return;
            }

            var file = m[1];
            var img_info = themeParams.get_image_info(file);

            if (img_info.format) {
                src = "/" + img_info.module + "/static/src/img/theme_" + themeParams.name + "/s_default_image_" + file + "." + img_info.format;
            } else {
                src = "/web/image/" + img_info.module + ".s_default_image_theme_" + themeParams.name + "_" + file;
            }

            $img.attr("src", src);
        });
        $container.find('.o_mail_block_cover .oe_img_bg').each(function () {
            $(this).css('background-image', `url('/mass_mailing_themes/static/src/img/theme_${themeParams.name}/s_default_image_block_banner.jpg')`);
        });
    },
    /**
     * Switch themes or import first theme.
     *
     * @private
     * @param {Object} themeParams
     */
    _switchThemes: function (themeParams) {
        if (!themeParams || this.switchThemeLast === themeParams) {
            return;
        }
        this.switchThemeLast = themeParams;

        this.$lastContent = this.$content.find('.o_mail_wrapper_td').contents();

        this.$content.closest('body').removeClass(this._allClasses).addClass(themeParams.className);

        const old_layout = this.$content.find('.o_layout')[0];

        var $new_wrapper;
        var $newWrapperContent;
        if (themeParams.nowrap) {
            $new_wrapper = $('<div/>', {
                class: 'oe_structure'
            });
            $newWrapperContent = $new_wrapper;
        } else {
            // This wrapper structure is the only way to have a responsive
            // and centered fixed-width content column on all mail clients
            $new_wrapper = $('<div/>', {
                class: 'container o_mail_wrapper o_mail_regular oe_unremovable',
            });
            $newWrapperContent = $('<div/>', {
                class: 'col o_mail_no_options o_mail_wrapper_td bg-white oe_structure o_editable'
            });
            $new_wrapper.append($('<div class="row"/>').append($newWrapperContent));
        }
        var $newLayout = $('<div/>', {
            class: 'o_layout oe_unremovable oe_unmovable bg-200 ' + themeParams.className,
            'data-name': 'Mailing',
        }).append($new_wrapper);

        const $contents = themeParams.template;
        $newWrapperContent.append($contents);
        this._switchImages(themeParams, $newWrapperContent);
        old_layout && old_layout.remove();
        this.$content.empty().append($newLayout);

        $newWrapperContent.find('*').addBack()
            .contents()
            .filter(function () {
                return this.nodeType === 3 && this.textContent.match(/\S/);
            }).parent().addClass('o_default_snippet_text');

        if (themeParams.name === 'basic') {
            this.$content[0].focus();
        }
        this.wysiwyg.trigger('reload_snippet_dropzones');
        this.trigger_up('iframe_updated', { $iframe: this.wysiwyg.$iframe });
        this.wysiwyg.odooEditor.historyStep(true);
    },

    /**
     * @private
     * @override
     */
    _toggleCodeView: function ($codeview) {
        this._super(...arguments);
        if ($codeview.hasClass('d-none')) {
            this.trigger_up('iframe_updated', { $iframe: this.wysiwyg.$iframe });
        }
    },

    /**
     * @override
     */
    _getWysiwygOptions: function () {
        const options = this._super.apply(this, arguments);
        options.resizable = false;
        options.defaultDataForLinkTools = { isNewWindow: true };
        if (this._wysiwygSnippetsActive) {
            options.wysiwygAlias = 'mass_mailing.wysiwyg';
        } else {
            delete options.snippets;
        }
        return options;
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _onLoadWysiwyg: function () {
        // Let the global hotkey manager know about our iframe.
        this.call('hotkey', 'registerIframe', this.wysiwyg.$iframe[0]);

        if (this.snippetsLoaded) {
            this._onSnippetsLoaded(this.snippetsLoaded);
        }
        this._super();
        this.wysiwyg.odooEditor.observerFlush();
        this.wysiwyg.odooEditor.historyReset();
        this.wysiwyg.$iframeBody.addClass('o_mass_mailing_iframe');
        this.trigger_up('iframe_updated', { $iframe: this.wysiwyg.$iframe });
    },
    /**
     * @private
     * @param {boolean} activateSnippets
     */
    _restartWysiwygIntance: async function (activateSnippets = true) {
        this.wysiwyg.destroy();
        this.$el.empty();
        this._wysiwygSnippetsActive = activateSnippets;
        await this._createWysiwygIntance();
    },
    /**
     * @private
     * @param {OdooEvent} ev
     */
    _onSnippetsLoaded: function (ev) {
        var self = this;
        if (this.wysiwyg.snippetsMenu && $(window.top.document).find('.o_mass_mailing_form_full_width')[0]) {
            // In full width form mode, ensure the snippets menu's scrollable is
            // in the form view, not in the iframe.
            this.wysiwyg.snippetsMenu.$scrollable = this.$el.closestScrollable();
            // Ensure said scrollable keeps its scrollbar at all times to
            // prevent the scrollbar from appearing at awkward moments (ie: when
            // previewing an option)
            this.wysiwyg.snippetsMenu.$scrollable.css('overflow-y', 'scroll');
        }
        if (!this.$content) {
            this.snippetsLoaded = ev;
            return;
        }
        var $snippetsSideBar = ev.data;
        var $themes = $snippetsSideBar.find("#email_designer_themes").children();
        var $snippets = $snippetsSideBar.find(".oe_snippet");
        var selectorToKeep = '.o_we_external_history_buttons, .email_designer_top_actions';
        // Overide `d-flex` class which style is `!important`
        $snippetsSideBar.find(`.o_we_website_top_actions > *:not(${selectorToKeep})`).attr('style', 'display: none!important');
        var $snippets_menu = $snippetsSideBar.find("#snippets_menu");
        var $selectTemplateBtn = $snippets_menu.find('.o_we_select_template');

        if (config.device.isMobile) {
            $snippetsSideBar.hide();
            this.$content.attr('style', 'padding-left: 0px !important');
        }

        if (!odoo.debug) {
            $snippetsSideBar.find('.o_codeview_btn').hide();
        }
        this._$codeview = this.wysiwyg.$iframe.contents().find('textarea.o_codeview');
        $snippetsSideBar.on('click', '.o_codeview_btn', () => this._toggleCodeView(this._$codeview));

        if ($themes.length === 0) {
            return;
        }

        /**
         * Initialize theme parameters.
         */
        this._allClasses = "";
        var themesParams = _.map($themes, function (theme) {
            var $theme = $(theme);
            var name = $theme.data("name");
            var classname = "o_" + name + "_theme";
            self._allClasses += " " + classname;
            var imagesInfo = _.defaults($theme.data("imagesInfo") || {}, {
                all: {}
            });
            _.each(imagesInfo, function (info) {
                info = _.defaults(info, imagesInfo.all, {
                    module: "mass_mailing",
                    format: "jpg"
                });
            });
            return {
                name: name,
                className: classname || "",
                img: $theme.data("img") || "",
                template: $theme.html().trim(),
                nowrap: !!$theme.data('nowrap'),
                get_image_info: function (filename) {
                    if (imagesInfo[filename]) {
                        return imagesInfo[filename];
                    }
                    return imagesInfo.all;
                }
            };
        });
        $themes.parent().remove();

        /**
         * Create theme selection screen and check if it must be forced opened.
         * Reforce it opened if the last snippet is removed.
         */
        const $themeSelector = $(core.qweb.render("mass_mailing.theme_selector", {
            themes: themesParams
        }));
        const $themeSelectorNew = $(core.qweb.render("mass_mailing.theme_selector_new", {
            themes: themesParams
        }));


        let firstChoice = this._editableAreaIsEmpty();
        if (firstChoice) {
            $themeSelectorNew.appendTo(this.wysiwyg.$iframeBody);
        }

        /**
         * Add proposition to install enterprise themes if not installed.
         */
        var $mail_themes_upgrade = $themeSelector.find(".o_mass_mailing_themes_upgrade");
        $mail_themes_upgrade.on("click", function (e) {
            e.stopImmediatePropagation();
            e.preventDefault();
            self.do_action("mass_mailing.action_mass_mailing_configuration");
        });

        $selectTemplateBtn.on('click', () => {
            $snippetsSideBar.data('snippetMenu').activateCustomTab($themeSelector);
            /**
             * Ensure the parent of the theme selector is not used as parent for a
             * tooltip as it is overflow auto and would result in the tooltip being
             * hidden by the body of the mail.
             */
            $themeSelector.parent().addClass('o_forbidden_tooltip_parent');
            $selectTemplateBtn.addClass('active');
        });

        /**
         * Switch theme when a theme button is hovered. Confirm change if the theme button
         * is pressed.
         */
        var selectedTheme = false;
        $themeSelector.on("mouseenter", ".dropdown-item", function (e) {
            e.preventDefault();
            var themeParams = themesParams[$(e.currentTarget).index()];
            self.wysiwyg.odooEditor.automaticStepSkipStack();
            self._switchThemes(themeParams);
        });
        $themeSelector.on("mouseleave", ".dropdown-item", function (e) {
            if (self.$lastContent) {
                self._switchThemes(Object.assign({}, selectedTheme, {template: self.$lastContent}));
            } else {
                self._switchThemes(selectedTheme);
            }
        });
        $themeSelector.on("click", '[data-toggle="dropdown"]', function (e) {
            var $menu = $themeSelector.find('.dropdown-menu');
            var isVisible = $menu.hasClass('show');
            if (isVisible) {
                e.preventDefault();
                e.stopImmediatePropagation();
                $menu.removeClass('show');
            }
        });

        const selectTheme = (e) => {
            e.preventDefault();
            e.stopImmediatePropagation();
            const themeParams = themesParams[$(e.currentTarget).index()];
            self._switchImages(themeParams, $snippets);

            selectedTheme = themeParams;

            // Notify form view
            $themeSelector.find('.dropdown-item.selected').removeClass('selected');
            $themeSelector.find('.dropdown-item:eq(' + themesParams.indexOf(selectedTheme) + ')').addClass('selected');

            // Invalidate previous content.
            self.$lastContent = undefined;
        };

        $themeSelector.on("click", ".dropdown-item", selectTheme);
        $themeSelectorNew.on("click", ".dropdown-item", async (e) => {
            e.preventDefault();
            e.stopImmediatePropagation();
            const themeParams = themesParams[$(e.currentTarget).index()];

            if (themeParams.name === "basic") {
                await this._restartWysiwygIntance(false);
            }
            this._switchThemes(themeParams);
            this.$content.closest('body').removeClass("o_force_mail_theme_choice");

            $themeSelectorNew.remove();

            if ($mail_themes_upgrade.length) {
                $snippets_menu.empty();
            }

            selectTheme(e);
            this._addEditorMessages();
            // Wait the next tick because some mutation have to be processed by
            // the Odoo editor before resetting the history.
            setTimeout(() => {
                this.wysiwyg.historyReset();
            }, 0);
        });

        /**
         * On page load, check the selected theme and force switching to it (body needs the
         * theme style for its edition toolbar).
         */
        selectedTheme = this._getSelectedTheme(themesParams);
        if (selectedTheme) {
            this.$content.closest('body').addClass(selectedTheme.className);
            $themeSelector.find('.dropdown-item:eq(' + themesParams.indexOf(selectedTheme) + ')').addClass('selected');
            this._switchImages(selectedTheme, $snippets);
        } else if (this.$content.find('.o_layout').length) {
            themesParams.push({
                name: 'o_mass_mailing_no_theme',
                className: 'o_mass_mailing_no_theme',
                img: "",
                template: this.$content.find('.o_layout').addClass('o_mass_mailing_no_theme').clone().find('oe_structure').empty().end().html().trim(),
                nowrap: true,
                get_image_info: function () {}
            });
            selectedTheme = this._getSelectedTheme(themesParams);
        }
    },
    /**
     * @override
     * @param {MouseEvent} ev
     */
    _onTranslate: function (ev) {
        this.trigger_up('translate', {
            fieldName: this.nodeOptions['inline-field'],
            id: this.dataPointID,
            isComingFromTranslationAlert: false,
        });
    },
});

fieldRegistry.add('mass_mailing_html', MassMailingFieldHtml);

return MassMailingFieldHtml;

});

```

## File: static\src\js\snippets.editor.js

```javascript
odoo.define('mass_mailing.snippets.editor', function (require) {
'use strict';

const snippetsEditor = require('web_editor.snippet.editor');

const MassMailingSnippetsMenu = snippetsEditor.SnippetsMenu.extend({
    custom_events: _.extend({}, snippetsEditor.SnippetsMenu.prototype.custom_events, {
        drop_zone_over: '_onDropZoneOver',
        drop_zone_out: '_onDropZoneOut',
        drop_zone_start: '_onDropZoneStart',
        drop_zone_stop: '_onDropZoneStop',
    }),

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
});

return MassMailingSnippetsMenu;

});

```

## File: static\src\js\unsubscribe.js

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

## File: static\src\js\wysiwyg.js

```javascript
odoo.define('mass_mailing.wysiwyg', function (require) {
'use strict';

var Wysiwyg = require('web_editor.wysiwyg');
var MassMailingSnippetsMenu = require('mass_mailing.snippets.editor');

const MassMailingWysiwyg = Wysiwyg.extend({
    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    start: async function () {
        const res = await this._super(...arguments);
        // Prevent selection change outside of snippets.
        this.$editable.on('mousedown', e => {
            if ($(e.target).is('.o_editable:empty') || e.target.querySelector('.o_editable')) {
                e.preventDefault();
            }
        });
        return res;
    },

    /**
     * @override
     */
     setValue: function (currentValue) {
        const initialDropZone = this.$editable[0].querySelector('.o_mail_wrapper_td');
        const parsedHtml = new DOMParser().parseFromString(currentValue, "text/html");
        if (initialDropZone && !parsedHtml.querySelector('.o_mail_wrapper_td')) {
            initialDropZone.replaceChildren(...parsedHtml.body.childNodes);
        } else {
            this._super(...arguments);
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
        trigger: 'input[name="subject"]',
        content: ('Pick the <b>email subject</b>.'),
        position: 'bottom',
        run: 'text Test',
    }, {
        trigger: 'div[name="contact_list_ids"] .o_input_dropdown > input[type="text"]',
        content: 'Click on the dropdown to open it and then start typing to search.',
    }, {
        trigger: 'li.ui-menu-item',
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
    }]);
});

```

## File: static\src\js\tours\mass_mailing_editor_tour.js

```javascript
odoo.define('mass_mailing.mass_mailing_editor_tour', function (require) {
    "use strict";

    var tour = require('web_tour.tour');

    tour.register('mass_mailing_editor_tour', {
        url: '/web',
        test: true,
    }, [tour.stepUtils.showAppsMenuItem(), {
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
    }, {
        trigger: 'button.o_list_button_add',
    }, {
        trigger: 'div[name="contact_list_ids"] .o_input_dropdown > input[type="text"]',
    }, {
        trigger: 'li.ui-menu-item',
    }, {
        content: 'choose the theme "empty" to edit the mailing with snippets',
        trigger: '[name="body_arch"] iframe #empty',
    }, {
        content: 'wait for the editor to be rendered',
        trigger: '[name="body_arch"] iframe .o_editable',
        run: () => {},
    }, {
        content: 'drag the "Title" snippet from the design panel and drop it in the editor',
        trigger: '[name="body_arch"] iframe #email_designer_default_body [name="Title"] .ui-draggable-handle',
        run: function (actions) {
            actions.drag_and_drop('[name="body_arch"] iframe .o_editable', this.$anchor);
        }
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
        trigger: 'input[name="subject"]',
        run: 'text Test',
    }, {
        trigger: 'button.o_form_button_save',
    }, {
        trigger: 'iframe.o_readonly',
        run: () => {},
    }]);
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
        content: "Click on the 'select a template' tab.",
        trigger: 'iframe .o_we_select_template',
    },
    {
        content: "Click on the empty 'DRAG BUILDING BLOCKS HERE' area.",
        extra_trigger: 'iframe .o_we_customize_panel .o_mail_theme_selector',
        trigger: 'iframe .oe_structure.o_mail_no_options',
    },
    {
        content: "Click on the 'select a template' tab.",
        trigger: 'iframe .o_we_select_template',
    },
    {
        content: "Verify that the customize panel is not empty.",
        trigger: 'iframe .o_we_customize_panel > .o_mail_theme_selector',
        run: () => null, // it's a check
    },
    {
        content: "Click on the style tab.",
        trigger: 'iframe .o_we_customize_snippet_btn',
    },
    {
        content: "Click on the 'select a template' tab.",
        trigger: 'iframe .o_we_select_template',
    },
    {
        content: "Verify that the customize panel is not empty.",
        trigger: 'iframe .o_we_customize_panel > .o_mail_theme_selector',
        run: () => null, // it's a check
    },
]);

```

## File: static\src\js\tours\mass_mailing_tour.js

```javascript
odoo.define('mass_mailing.mass_mailing_tour', function (require) {
    "use strict";

    const {_t} = require('web.core');
    const {Markup} = require('web.utils');
    var tour = require('web_tour.tour');
    var now = moment();

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
        trigger: 'input[name="subject"]',
        content: Markup(_t('Pick the <b>email subject</b>.')),
        position: 'bottom',
        run: 'text ' + now.format("MMMM") + " Newsletter",
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
        trigger: 'div[name="body_arch"] iframe div.s_text_block',
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

## File: static\src\js\tours\mass_mailing_undo_icon_image.js

```javascript
odoo.define('mass_mailing.mass_mailing_undo_icon_to_image_change', function (require) {
    "use strict";

    var tour = require('web_tour.tour');

    tour.register('mass_mailing_undo_icon_to_image_change', {
        url: '/web',
        test: true,
    }, [tour.stepUtils.showAppsMenuItem(), {
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
    }, {
        trigger: 'button.o_list_button_add',
    }, {
        trigger: 'div[name="contact_list_ids"] .o_input_dropdown > input[type="text"]',
    }, {
        trigger: 'li.ui-menu-item',
    }, {
        content: 'choose the theme "empty" to edit the mailing with snippets',
        trigger: '[name="body_arch"] iframe #empty',
    }, {
        content: 'wait for the editor to be rendered',
        trigger: '[name="body_arch"] iframe .o_editable',
        run: () => { },
    }, {
        content: 'drag the "Features" snippet from the design panel and drop it in the editor',
        trigger: '[name="body_arch"] iframe #email_designer_default_body [name="Features"] .ui-draggable-handle',
        run: function (actions) {
            actions.drag_and_drop('[name="body_arch"] iframe .o_editable', this.$anchor);
        }
    }, {
        content: 'select and click the gear icon',
        trigger: '[name="body_arch"] iframe .o_editable .fa-gear',
        run: function (actions) {
            const document = this.$anchor[0].ownerDocument;
            const range = document.createRange();
            range.selectNodeContents(this.$anchor[0]);
            const sel = document.defaultView.getSelection();
            sel.removeAllRanges();
            sel.addRange(range);
            actions.click();
        },
    }, {
        content: 'replace media',
        trigger: '[name="body_arch"] iframe we-button:contains("Replace")',
        run: 'click'
    }, {
        content: 'check that the modal is open',
        trigger: '.modal-dialog',
        run: () => { },
    }, {
        content: 'click on image tab',
        trigger: '.modal-dialog a[aria-controls="editor-media-image"]',
    }, {
        content: 'choose an image',
        trigger: '.modal-dialog .o_existing_attachment_cell img',
    }, {
        content: 'verify that icon has been changed',
        trigger: '[name="body_arch"] iframe .o_editable img[data-original-id]',
        run: () => { },
    }, {
        content: 'select and click the image',
        trigger: '[name="body_arch"] iframe .o_editable img[data-original-id]',
        run: function (actions) {
            const document = this.$anchor[0].ownerDocument;
            const range = document.createRange();
            range.selectNodeContents(this.$anchor[0]);
            const sel = document.defaultView.getSelection();
            sel.removeAllRanges();
            sel.addRange(range);
            actions.click();
        },
    }, {
        content: 'verify that image is fully loaded',
        trigger: '[name="body_arch"] iframe we-title:contains("Image")',
        run: () => { },
    }, {
        content: 'click undo',
        trigger: '[name="body_arch"] iframe .o_we_external_history_buttons button[data-action="undo"]',
        run: 'click',
    }, {
        content: 'Check that the change is reverted and now we have the icon back',
        trigger: '[name="body_arch"] iframe .o_editable .fa-gear',
        run: () => { },
    },
    ]);
});

```

## File: static\src\snippets\s_blockquote\options.js

```javascript
odoo.define('mass_mailing.s_blockquote_options', function (require) {
'use strict';

const options = require('web_editor.snippets.options');

options.registry.Blockquote = options.Class.extend({

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Change blockquote design.
     *
     * @see this.selectClass for parameters
     */
    display: function (previewMode, widgetValue, params) {

        // Classic
        this.$target.find('.s_blockquote_avatar').toggleClass('d-none', widgetValue !== 'classic');

        // Cover
        const $blockquote = this.$target.find('.s_blockquote_content');
        if (widgetValue === 'cover') {
            $blockquote.css({"background-image": "url('/web/image/mass_mailing.s_blockquote_cover_default_image')"});
            $blockquote.addClass('oe_img_bg o_bg_img_center');
            if (!$blockquote.find('.o_we_bg_filter').length) {
                const bgFilterEl = document.createElement('div');
                bgFilterEl.classList.add('o_we_bg_filter', 'bg-white-50');
                $blockquote.prepend(bgFilterEl);
            }
        } else {
            $blockquote.css({"background-image": ""});
            $blockquote.css({"background-position": ""});
            $blockquote.removeClass('oe_img_bg o_bg_img_center');
            $blockquote.find('.o_we_bg_filter').remove();
            $blockquote.find('.s_blockquote_filter').contents().unwrap(); // Compatibility
        }

        // Minimalist
        this.$target.find('.s_blockquote_icon').toggleClass('d-none', widgetValue === 'minimalist');
        this.$target.find('footer').toggleClass('d-none', widgetValue === 'minimalist');
    },
});
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

const weWidgets = require('wysiwyg.widgets');
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
        return new Promise(resolve => {
            const dialog = new weWidgets.MediaDialog(
                this,
                {noImages: true, noDocuments: true, noVideos: true, mediaWidth: 1920},
                $('<i/>')
            );
            this._saving = false;
            dialog.on('save', this, function (attachments) {
                this._saving = true;
                const customClass = 'fa ' + attachments.className;
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
                resolve();
            });
            dialog.on('closed', this, function () {
                if (!this._saving) {
                    resolve();
                }
            });
            dialog.open();
        });
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
        $showcaseCol.find('.s_showcase_icon.ml-3').removeClass('ml-3').addClass('ml-lg-3'); // For compatibility with old version
        $title.find('.s_showcase_icon').toggleClass('mr-lg-0 ml-lg-3', isLeftCol);
    },
});
});

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
        <t t-foreach="themes" t-as="theme">
            <a t-att-id="theme.name" role="menuitem" href="#" class="dropdown-item">
                <div class="o_thumb small"  t-attf-style="background-image: url(#{theme.img}_small.png)"/>
                <div class="o_thumb large" t-attf-style="background-image: url(#{theme.img}_large.png)"/>
            </a>
        </t>
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
        <t t-call-assets="web.assets_common" t-js="false"/>
        <t t-call-assets="web.assets_frontend" t-js="false"/>
        <t t-call-assets="web_editor.assets_wysiwyg" t-js="false"/>
    </template>

    <template id="iframe_css_assets_readonly" groups="base.group_user">
        <link rel="stylesheet" type="text/scss" href="/mass_mailing/static/src/css/basic_theme_readonly.css"/>
        <t t-call="mass_mailing.mass_mailing_mail_style"/>
    </template>

    <template id="mass_mailing_mail_style">
        <style>
            .o_layout * {
                box-sizing: border-box !important;
            }
            .o_layout :not(.fa) {
                font-family: Arial, sans-serif !important;
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
                .o_mail_table_styles {
                    width: 100% !important;
                }
            }
        </style>
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

    <!-- MENU TO HANLDE LINK DATA IN MM -->
    <menuitem id="link_tracker_menu_mass_mailing"
        name="Link Tracker"
        parent="mass_mailing_configuration"
        sequence="5"
        action="link_tracker.link_tracker_action"/>

</odoo>

```

## File: views\mailing_contact_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!--  MAILING CONTACT SUBSCRIPTION -->
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
            <tree string="Mailing List Contacts" sample="1">
                <header>
                    <button name="action_add_to_mailing_list" string="Add to List" type="object"/>
                </header>
                <field name="create_date"/>
                <field name="name"/>
                <field name="company_name"/>
                <field name="email"/>
                <field name="is_blacklisted" string="Email Blacklisted"/>
                <field name="message_bounce" sum="Total Bounces"/>
                <field name="opt_out" invisible="'default_list_ids' not in context"/>
            </tree>
        </field>
    </record>

    <record id="mailing_contact_view_kanban" model="ir.ui.view">
        <field name="name">mailing.contact.view.kanban</field>
        <field name="model">mailing.contact</field>
        <field name="arch" type="xml">
            <kanban sample="1">
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
                                <span class="badge badge-pill" title="Number of bounced email.">
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
                            <field name="tag_ids" widget="many2many_tags" options="{'color_field': 'color'}" placeholder="Tags" style="width: 100%%"/>
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

    <menuitem name="Mailing List Contacts" id="menu_email_mass_mailing_contacts"
        parent="mass_mailing_mailing_list_menu" sequence="4"
        action="action_view_mass_mailing_contacts"/>
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
                <group>
                    <group>
                        <div class="oe_title">
                            <label for="name"/>
                            <h1>
                                <field name="name" placeholder="e.g. Consumer Newsletter"/>
                            </h1>
                        </div>
                    </group>
                </group>
                <group>
                    <field name="is_public"/>
                </group>
            </form>
        </field>
    </record>

    <record id="mailing_list_view_form_simplified_footer" model="ir.ui.view">
        <field name="name">mailing.list.form.simplified.footer</field>
        <field name="model">mailing.list</field>
        <field name="inherit_id" ref="mailing_list_view_form_simplified"/>
        <field name="mode">primary</field>
        <field name="priority" eval="30"/>
        <field name="arch" type="xml">
            <xpath expr="//form" position="inside">
                <footer>
                    <button string="Create" name="close_dialog" type="object" class="btn-primary" data-hotkey="q"/>
                    <button string="Discard" class="btn-secondary" special="cancel" data-hotkey="z"/>
                </footer>
            </xpath>
        </field>
    </record>

    <record id="open_create_mass_mailing_list" model="ir.actions.act_window">
        <field name="name">Create a Mailing List</field>
        <field name="res_model">mailing.list</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="mailing_list_view_form_simplified_footer"/>
        <field name="target">new</field>
    </record>

    <record id="mailing_list_view_kanban" model="ir.ui.view">
        <field name="name">mailing.list.view.kanban</field>
        <field name="model">mailing.list</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile o_kanban_mailing_list" on_create="mass_mailing.open_create_mass_mailing_list" sample="1">
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
                        <div class="oe_kanban_global_click">
                            <div class="oe_kanban_content d-flex flex-column h-100">
                                <h2 class="mb-3 o_text_overflow">
                                    <field name="name"/>
                                </h2>
                                <div class="d-flex align-items-center">
                                    <div class="mr-3">
                                        <button class="btn btn-primary" name="action_view_contacts" type="object">
                                            <t t-esc="record.contact_count.value"/>
                                            <span>Contacts</span>
                                        </button>
                                    </div>
                                    <div class="flex-grow-1 d-flex flex-column align-items-end o_mass_mailing_kanban_contact_links">
                                        <a name="action_view_contacts_email" type="object">
                                            <span>Valid Email Recipients</span>
                                            <span t-esc="record.contact_count_email.value" class="ml-3"/>
                                        </a>
                                    </div>
                                </div>
                                <div class="flex-grow-1 d-flex align-items-end mt-4">
                                    <div class="col-12">
                                        <div class="row mt3">
                                            <div class="col-3 border-right">
                                                <a name="action_view_mailings" type="object" class="d-flex flex-column align-items-center">
                                                    <span><field name="mailing_count"/></span>
                                                    <span class="text-muted">Mailings</span>
                                                </a>
                                            </div>
                                            <div class="col-3 border-right">
                                                <a name="action_view_contacts_bouncing" type="object" class="d-flex flex-column align-items-center">
                                                    <span><field name="contact_pct_bounce"/>%</span>
                                                    <span class="text-muted">Bounce</span>
                                                </a>
                                            </div>
                                            <div class="col-3 border-right">
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

    <menuitem name="Mailing Lists" id="menu_email_mass_mailing_lists"
        parent="mass_mailing_mailing_list_menu" sequence="3"
        action="action_view_mass_mailing_lists"/>
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
                    <field name="sent"/>
                    <field name="received_ratio" class="d-flex align-items-center pl-0 pl-lg-5" widget="progressbar" string="Delivered (%)"/>
                    <field name="opened_ratio" class="d-flex align-items-center pl-0 pl-lg-5" widget="progressbar" string="Opened (%)"/>
                    <field name="bounced_ratio" string="Bounced (%)" optional="hide"/>
                    <field name="clicks_ratio" string="Clicked (%)"/>
                    <field name="replied_ratio" string="Replied (%)"/>
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
                                    <field name="canceled" class="oe_inline mr-2"/>
                                    <span name="canceled_text">emails have been canceled and will not be sent.</span>
                                </strong>
                            </button>
                        </div>
                        <div class="o_mails_scheduled" attrs="{'invisible': [('scheduled', '=', 0)]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_scheduled"
                                    type="object">
                                <strong>
                                    <field name="scheduled" class="oe_inline mr-2"/>
                                    <span name="scheduled_text">emails are in queue and will be sent soon.</span>
                                </strong>
                            </button>
                        </div>
                        <div class="o_mails_sent" attrs="{'invisible': ['&amp;', ('sent', '=', 0), ('state', 'in', ('draft', 'test', 'in_queue'))]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_sent"
                                    type="object">
                                <strong>
                                    <field name="sent" class="oe_inline mr-2"/>
                                    <span name="sent">emails have been sent.</span>
                                </strong>
                            </button>
                        </div>
                        <div class="o_mails_failed" attrs="{'invisible': ['|', ('state', '!=', 'done'), ('failed', '=', 0)]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_failed"
                                    type="object">
                                <strong>
                                    <field name="failed" class="oe_inline mr-2"/>
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
                            <field name="mailing_type" widget="radio" options="{'horizontal': true}" invisible="1"
                                attrs="{'readonly': [('state', '!=', 'draft')]}" force_save="1"/>
                            <field class="text-break" name="subject" string="Subject" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}" widget="char_emojis" placeholder="e.g. New Sale on all T-shirts"/>
                            <label for="mailing_model_id" string="Recipients"/>
                            <div name="mailing_model_id_container">
                                <div class="row">
                                    <div class="col-xs-12 col-md-3" >
                                        <field name="mailing_model_id" options="{'no_open': True, 'no_create': True}"
                                            attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                    </div>
                                    <div attrs="{'invisible': [('mailing_model_name', '!=', 'mailing.list')]}" class="col-xs-12 col-md-9 pt-1">
                                        <label for="contact_list_ids" string="Select mailing lists:" class="oe_edit_only"/>
                                        <field name="contact_list_ids" widget="many2many_tags"
                                            placeholder="Select mailing lists..." class="oe_inline"
                                            context="{'form_view_ref': 'mass_mailing.mailing_list_view_form_simplified'}"
                                            attrs="{
                                                'required':[('mailing_model_name','=','mailing.list')],
                                                'readonly': [('state', 'in', ('sending', 'done'))]
                                        }"/>
                                    </div>
                                </div>

                                <field name="mailing_model_name" invisible="1"/>
                                <field name="mailing_model_real" invisible="1"/>
                                <div attrs="{'invisible': [('mailing_model_name', '=', 'mailing.list')]}">
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
                                            options="{
                                                'snippets': 'mass_mailing.email_designer_snippets',
                                                'cssEdit': 'mass_mailing.iframe_css_assets_edit',
                                                'inline-field': 'body_html',
                                                'cssReadonly': 'mass_mailing.iframe_css_assets_edit'
                                        }" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                    </div>
                                    <field name="is_body_empty" invisible="1"/>
                                    <div class="o_view_nocontent oe_read_only" attrs="{'invisible': ['|', ('is_body_empty', '=', False), ('state', 'in', ('sending', 'done'))]}">
                                        <div class="o_nocontent_help">
                                            <p class="o_view_nocontent_smiling_face">
                                                No template picked yet.
                                            </p>
                                            <p>
                                                Start editing your mailing to design something awesome.
                                            </p>
                                        </div>
                                    </div>
                                </div>
                            </page>
                            <page string="Dynamic Placeholder Generator"
                                name="dynamic_placeholder_generator"
                                groups="base.group_no_one">
                                <group>
                                    <field name="model_object_field" attrs="{'invisible': True}"/>
                                    <field name="model_object_field"
                                        domain="[('model_id','=',mailing_model_real),('ttype','!=','one2many'),('ttype','!=','many2many')]"/>
                                    <field name="sub_object" readonly="1"/> 
                                    <field name="sub_model_object_field" 
                                        domain="[('model_id','=',sub_object),('ttype','!=','one2many'),('ttype','!=','many2many')]"
                                        attrs="{'readonly':[('sub_object','=',False)],'required':[('sub_object','!=',False)]}"/>
                                    <field name="null_value"/>
                                    <field name="copyvalue"/>
                                </group>
                            </page>
                            <page string="A/B Tests" name="ab_testing" groups="mass_mailing.group_mass_mailing_campaign">
                                <group>
                                    <group>
                                        <label for="ab_testing_enabled"/>
                                        <span class="d-flex">
                                            <field name="ab_testing_enabled" attrs="{'readonly': [('state', '!=', 'draft')]}" force_save="1"/>
                                            <span class="col" attrs="{'invisible': [('ab_testing_enabled', '=', False)]}">
                                                on <field name="ab_testing_pc" class="col-6 text-center"
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
                                    <group string="Email Content" attrs="{'invisible': [('mailing_type', '!=', 'mail')]}">
                                        <field class="o_text_overflow" name="preview" string="Preview Text" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}" widget="char_emojis" placeholder="e.g. Check it out before it's too late!"/>
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
                                        <field name="user_id" widget="many2one_avatar_user" domain="[('share', '=', False)]"/>
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
                    <attribute name="class">o_mass_mailing_mailing_form o_mass_mailing_form_full_width</attribute>
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
                <xpath expr="//notebook/page[@name='mail_body']" position="after">
                    <page string="Mail Debug" name="mail_debug" groups="base.group_no_one">
                        <div class="position-relative">
                            <div class="mt-n2">
                                <field name="body_html" class="o_mail_body" widget="html"
                                    options="{'cssReadonly': 'mass_mailing.iframe_css_assets_readonly', 'notEditable': True}"/>
                            </div>
                            <field name="is_body_empty" invisible="1"/>
                            <div class="o_view_nocontent oe_read_only" attrs="{'invisible': ['|', ('is_body_empty', '=', False), ('state', 'in', ('sending', 'done'))]}">
                                <div class="o_nocontent_help">
                                    <p class="o_view_nocontent_smiling_face">
                                        No template picked yet.
                                    </p>
                                    <p>
                                        Start editing your mailing to design something awesome.
                                    </p>
                                </div>
                            </div>
                        </div>
                    </page>
                </xpath>
                <xpath expr="//notebook/page[@name='dynamic_placeholder_generator']" position="inside">
                    <group/>
                </xpath>
                <xpath expr="//notebook/page[@name='dynamic_placeholder_generator']/group[2]" position="inside">
                    <xpath expr="//notebook/page[@name='dynamic_placeholder_generator']/group[1]" position="move"/>
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
                                    <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" href="#" data-display="static" aria-label="Dropdown menu" title="Dropdown menu">
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
                                            <span attrs="{'invisible': [('sent_date', '=', False)]}"><b><field name="delivered"/> / <field name="expected"/></b> Delivered to</span>
                                            <span attrs="{'invisible': [('sent_date', '!=', False)]}"><b><field name='total'/></b></span>
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
                                            <span class="fa fa-calendar-check-o mr-2 small my-auto" aria-label="Sent date"/>
                                            <span class="align-self-baseline"><field name="sent_date" widget="date"/></span>
                                        </span>
                                        <span attrs="{'invisible': [('schedule_date', '=', False)]}"
                                            t-attf-title="Scheduled on #{record.schedule_date.value}" class="d-inline-flex">
                                            <span class="fa fa-hourglass-half mr-2 small my-auto" aria-label="Scheduled date"/>
                                            <span class="align-self-baseline"><field name="schedule_date" widget="date"/></span>
                                        </span>
                                        <span attrs="{'invisible': ['|', '|', ('sent_date', '!=', False), ('schedule_date', '!=', False), ('state', '=', 'in_queue')]}"
                                            class="oe_clear">
                                            <b><field name='total'/></b>
                                            <field name='mailing_model_id' attrs="{'invisible': [('mailing_model_name','=','mailing.list')]}"/>
                                            <span attrs="{'invisible': [('mailing_model_name','!=','mailing.list')]}">Mailing Contact</span>
                                        </span>
                                        <span attrs="{'invisible': ['|', '|', ('schedule_date', '!=', False), ('state', '!=', 'in_queue'), ('next_departure', '=', False)]}"
                                            t-attf-title="Scheduled on #{record.next_departure.value}" class="d-inline-flex">
                                            <span class="fa fa-hourglass-o mr-2 small my-auto" aria-label="Scheduled date"/>
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

        <menuitem name="Mailings" id="mass_mailing_menu"
            parent="mass_mailing_menu_root"
            sequence="1"
            action="mailing_mailing_action_mail"
            groups="mass_mailing.group_mass_mailing_user"/>

</odoo>

```

## File: views\mailing_mailing_views_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Marketing / Mailing -->
    <menuitem name="Email Marketing" id="mass_mailing_menu_root" sequence="115" web_icon="mass_mailing,static/description/icon.png"/>
    <menuitem name="Mailing Lists" id="mass_mailing_mailing_list_menu"
        parent="mass_mailing_menu_root" sequence="2" groups="mass_mailing.group_mass_mailing_user"/>

    <!-- Marketing / Configuration -->
    <menuitem name="Configuration" id="mass_mailing_configuration"
        parent="mass_mailing_menu_root"
        sequence="100"
        groups="mass_mailing.group_mass_mailing_user"/>

    <!-- Configuration / Blacklist -->
    <menuitem id="mail_blacklist_mm_menu" name="Blacklisted Email Addresses"
        action="mail.mail_blacklist_action"
        parent="mass_mailing_configuration"/>

    <!-- Technical / Mass Mailing -->
    <menuitem id="mailing_mailing_menu_technical"
        name="Mass Mailing"
        sequence="4"
        parent="base.menu_custom"/>
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

    <!-- Add in Technical/Email -->
    <menuitem id="menu_email_statistics"
        name="Mailing Traces"
        parent="mass_mailing.mailing_mailing_menu_technical" sequence="2"
        action="mailing_trace_action"/>
</odoo>

```

## File: views\mass_mailing_templates_portal.xml

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
                                    <div class="btn btn-primary text-left" id="button_feedback">Send</div>
                                </div>
                            </div>

                            <h1 class="o_page_header">Mailing Subscriptions</h1>
                            <p>Choose your mailing subscriptions</p>
                            <div id="div_opt_out">
                                <ul class="list-group">
                                    <t t-foreach="list_ids" t-as="list_id">
                                        <t t-if="list_id.is_public == True">
                                            <li class="list-group-item">
                                                <input type="checkbox" class="mail_list_checkbox" name="contact_ids"
                                                    t-att-value="list_id['id']" t-att-checked="None if list_id['id'] in opt_out_list_ids else 'checked'"/>
                                                <t t-esc="list_id.name"/>
                                                <span t-if="list_id['id'] in opt_out_list_ids"
                                                      class="o_mass_mailing_unsubscribed">
                                                    Unsubscribed
                                                </span>
                                            </li>
                                        </t>
                                    </t>
                                </ul>

                                <div class="mb64 pt-3">
                                    <div t-if="show_blacklist_button">
                                        <div class="btn btn-secondary pull-right" id="button_add_blacklist" style="display:none">Blacklist Me</div>
                                    </div>
                                    <div class="btn btn-secondary pull-right" id="button_remove_blacklist" style="display:none">Come Back</div>
                                    <button type="submit" id="send_form" class="btn btn-primary">Update my subscriptions</button>
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
                        <div class="btn btn-secondary pull-right" id="button_add_blacklist" style="display:none">Blacklist Me</div>
                        <div class="btn btn-secondary pull-right" id="button_remove_blacklist" style="display:none">Come Back</div>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="view" name="Browser View">
        <!-- Raw body inserted here because it is a rendered mailing, therefore internal content -->
        <t t-out="body"/>
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

    <!-- new layout for mass_mailing -->
    <template id="mass_mailing.layout" name="Mass Mailing Layout">
        <t t-set="html_data" t-value="{'lang': lang and lang.replace('_', '-')}"/>
        <t t-call="web.layout">
            <t t-set="head">
                <t t-call-assets="web.assets_common"/>
                <t t-call-assets="web.assets_backend"/>
            </t>
            <body class="o_white_body">
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
            <xpath expr="//footer" position="replace">
                <div class="container mt16 mb8">
                    <div class="pull-right" t-ignore="true" t-if="not editable">
                        Create a <a target="_blank" href="https://www.odoo.com/app/website">free website</a> with
                        <a target="_blank" class="label label-danger" href="https://www.odoo.com/app/website">Odoo</a>
                    </div>
                    <div class="pull-left text-muted" itemscope="itemscope" itemtype="https://schema.org/Organization">
                        <t t-call="web.debug_icon"/>
                        Copyright &amp;copy; <span t-field="res_company.name" itemprop="name">Company name</span>
                    </div>
                </div>
            </xpath>
         </t>
     </template>
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
                                        Manage campaigns and A/B test your mailings
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
                                        Use a dedicated server for mailings
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('mass_mailing_outgoing_mail_server', '=', False)]}">
                                        <div class="mt16">
                                            <field name="mass_mailing_mail_server_id" options="{'no_create': True, 'no_open': True}"/>
                                        </div>
                                        <div class="mt8">
                                            <button type="action" name="base.action_ir_mail_server_list" string="Configure Email Server" icon="fa-arrow-right" class="oe_link"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-md-6 o_setting_box col-xs-12" name="allow_blacklist_setting_container">
                                <div class="o_setting_left_pane" title="Allow the recipient to manage himself his state in the blacklist via the unsubscription page.
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

        <menuitem id="menu_mass_mailing_global_settings" name="Settings"
            parent="mass_mailing_configuration" sequence="0" action="action_mass_mailing_configuration" groups="base.group_system"/>
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
            <button class="o_fullscreen_btn btn btn-primary">
                <img src="/web_editor/font_to_img/61541/rgb(255,255,255)/16" alt="Fullscreen"/>
            </button>
        </div>
    </xpath>
    <xpath expr="//div[@id='snippets_menu']" position="inside">
        <button type="button" class="o_we_select_template text-uppercase"><span>Select a template</span></button>
    </xpath>
    <xpath expr="//t[@id='default_snippets']" position="replace">
        <t id="default_snippets">
            <t t-set="company_id" t-value="res_company"/>
            <div id="email_designer_themes">
                <div data-name="basic"
                     data-nowrap="1"
                     data-img="/mass_mailing/static/src/img/theme_imgs/basic_thumb"
                     data-images-info='{"logo": {"format": "png"}}'>
                    <t t-call="mass_mailing.theme_basic_template"/>
                </div>
                <div data-name="empty"
                     data-img="/mass_mailing/static/src/img/theme_imgs/empty_thumb"
                     data-images-info='{"logo": {"format": "png"}}'>
                    <t t-call="mass_mailing.theme_empty_template"/>
                </div>
                <div data-name="default"
                     data-img="/mass_mailing/static/src/img/theme_imgs/default_thumb"
                     data-images-info='{"logo": {"format": "png"}}'>
                    <t t-call="mass_mailing.theme_default_template"/>
                </div>
            </div>
            <div id="email_designer_default_headers" class="o_panel">
                <div class="o_panel_header">Headers</div>
                <div class="o_panel_body" id="email_designer_header_elements">
                    <t t-snippet="mass_mailing.s_mail_block_header_social" t-thumbnail="/mass_mailing/static/src/img/blocks/block_header_social.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_header_text_social" t-thumbnail="/mass_mailing/static/src/img/blocks/block_header_text_social.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_header_logo" t-thumbnail="/mass_mailing/static/src/img/blocks/block_header_logo.png"/>
                    <t t-snippet="mass_mailing.s_cover" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_cover.svg"/>
                    <t t-snippet="mass_mailing.s_mail_block_header_view" t-thumbnail="/mass_mailing/static/src/img/blocks/block_header_browser.png"/>
                </div>
            </div>
            <div id="email_designer_default_body" class="o_panel">
                <div class="o_panel_header">Body</div>
                <div class="o_panel_body" id="email_designer_body_elements">
                    <t t-snippet="mass_mailing.s_title" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_title.svg"/>
                    <t t-snippet="mass_mailing.s_text_block" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_text_block.svg"/>
                    <t t-snippet="mass_mailing.s_comparisons" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_comparisons.svg"/>
                    <t t-snippet="mass_mailing.s_color_blocks_2" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_color_blocks_2.svg"/>
                    <t t-snippet="mass_mailing.s_three_columns" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_three_columns.svg"/>
                    <t t-snippet="mass_mailing.s_image_text" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_image_text.svg"/>
                    <t t-snippet="mass_mailing.s_text_image" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_text_image.svg"/>
                    <t t-snippet="mass_mailing.s_picture" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_picture.svg"/>
                    <t t-snippet="mass_mailing.s_features" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_features.svg"/>
                    <t t-snippet="mass_mailing.s_numbers" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_numbers.svg"/>
                    <t t-snippet="mass_mailing.s_masonry_block" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_masonry_block.svg"/>
                    <t t-snippet="mass_mailing.s_media_list" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_media_list.svg"/>
                    <t t-snippet="mass_mailing.s_showcase" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_showcase.svg"/>
                </div>
            </div>
            <div id="email_designer_default_extra" class="o_panel">
                <div class="o_panel_header">Marketing Content</div>
                <div class="o_panel_body" id="email_designer_marketing_elements">
                    <t t-snippet="mass_mailing.s_company_team" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_company_team.svg"/>
                    <t t-snippet="mass_mailing.s_call_to_action" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_call_to_action.svg"/>
                    <t t-snippet="mass_mailing.s_references" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_references.svg"/>
                    <t t-snippet="mass_mailing.s_coupon_code" t-thumbnail="/mass_mailing/static/src/img/blocks/block_discount2.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_discount1" t-thumbnail="/mass_mailing/static/src/img/blocks/block_discount1.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_event" t-thumbnail="/mass_mailing/static/src/img/blocks/block_event.png"/>
                    <t t-snippet="mass_mailing.s_product_list" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_product_list.svg"/>
                    <t t-snippet="mass_mailing.s_features_grid" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_features_grid.svg"/>
                </div>
            </div>
            <div id="email_designer_default_inner" class="o_panel">
                <div class="o_panel_header">Inner Content</div>
                <div class="o_panel_body" id="email_designer_inner_elements">
                    <t t-snippet="mass_mailing.s_alert" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_alert.svg"/>
                    <t t-snippet="mass_mailing.s_rating" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_rating.svg"/>
                    <t t-snippet="mass_mailing.s_blockquote" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_blockquote.svg"/>
                    <t t-snippet="mass_mailing.s_hr" t-thumbnail="/web_editor/static/src/img/snippets_thumbs/s_hr.svg"/>
                    <t t-snippet="mass_mailing.s_text_highlight" t-thumbnail="/mass_mailing/static/src/img/snippets_thumbs/s_text_highlight.svg"/>
                </div>
            </div>
            <div id="email_designer_default_footer" class="o_panel">
                <div class="o_panel_header">Footers</div>
                <div class="o_panel_body" id="email_designer_footer_elements">
                    <t t-snippet="mass_mailing.s_mail_block_footer_social" t-thumbnail="/mass_mailing/static/src/img/blocks/block_footer_social.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_footer_social_left" t-thumbnail="/mass_mailing/static/src/img/blocks/block_footer_social_left.png"/>
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
    <div class="o_mail_snippet_general o_mail_block_header_social s_header_social">
        <div class="o_mail_table_styles container">
            <div class="row">
                <div class="col-lg-8 pt16 pb16">
                    <a t-att-href="(company_id.website) or '#'" style="text-decoration:none;float:none;">
                         <img t-if="company_id.logo" border="0" t-att-src="image_data_uri(company_id.logo)" style="height:auto;max-width:200px;max-height:48px;" />
                    </a>
                </div>
                <div class="col-lg-4 text-right o_mail_no_resize">
                    <div class="o_mail_header_social">
                        <t t-call="mass_mailing.social_links"/>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_mail_block_header_text_social" name="Left Text">
    <div class="o_mail_snippet_general o_mail_block_header_text_social s_header_text_social">
        <div class="o_mail_table_styles container">
            <div class="row">
                <div class="col-lg-8 pt16 pb16">
                    <h3>
                        <a t-att-href="(company_id.website) or '#'">
                            My Company
                        </a>
                    </h3>
                </div>
                <div class="col-lg-4 text-right o_mail_no_resize">
                    <div class="o_mail_header_social">
                        <t t-call="mass_mailing.social_links"/>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_banner" name="Banner">
    <div class="o_mail_block_banner">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" border="0" class="o_mail_table_styles">
                <tbody>
                    <tr>
                        <td width="100%" valign="top" class="o_mail_full_width_padding o_mail_no_colorpicker">
                            <a href="#">
                                <img src="/mass_mailing/static/src/img/theme_basic/s_default_image_block_banner.jpg" class="d-block mx-auto"/>
                            </a>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<template id="s_mail_block_header_logo" name="Centered Logo">
    <div class="o_mail_snippet_general o_mail_block_header_logo s_header_logo">
        <div class="container o_mail_table_styles">
            <div class="row">
                <div class="col-lg-4"/>
                <div class="col-lg-4 text-center pt16 pb16">
                    <a t-att-href="(company_id.website) or '#'" style="text-decoration:none;">
                        <img t-if="company_id.logo" border="0" t-att-src="image_data_uri(company_id.logo)" style="height:auto;max-width:200px;max-height:48px;width:auto"/>
                    </a>
                </div>
                <div class="col-lg-4 text-right"/>
            </div>
        </div>
    </div>
</template>

<template id="s_mail_block_header_view" name="View Online">
    <div class="o_mail_snippet_general o_snippet_view_in_browser pt16 pb16 px-3 text-center">
        <a href="/view">
            View Online
        </a>
    </div>
</template>

<template id="s_mail_block_title_text" name="Title Content">
    <div class="o_mail_snippet_general o_mail_block_title_text s_title_text pt16 pb16 px-3">
        <h2 class="mt0">Thank you for joining us!</h2>
        <p>We want to take this opportunity to welcome you to our ever-growing community!<br/></p>
        <p>Your platform is ready for work. It will help you reduce the costs of digital signage, attract new customers and increase sales.</p>
        <p>Enjoy,</p>
        <img src="/mass_mailing/static/src/img/theme_default/demo/signature.png" style="width:125px; margin-top:8px;margin-bottom:-25px;" alt="Demo Signature"/>
        <p>
            <small>
                <strong>Michael Fletcher</strong><br/>
                <small>Community Manager</small>
            </small>
        </p>
        <div class="pt16 pb16 text-center">
            <a role="button" href="#" class="btn btn-primary">LOGIN</a>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_paragraph" name="Paragraph">
    <div class="o_mail_block_paragraph">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" border="0" class="o_mail_table_styles">
                <tbody>
                    <tr>
                        <td width="100%" class="o_mail_h_padding o_mail_v_padding o_mail_no_colorpicker">
                            <p> The open source model of Odoo has allowed us to leverage thousands of developers and
                                business experts to build hundreds of apps in just a few years.</p>
                            <p> With strong technical foundations, Odoo's framework is unique.
                                It provides top notch usability that scales across all apps.</p>
                            <p> Usability improvements made on Odoo will automatically apply to all
                                of our fully integrated apps.</p>
                            <p> That way, Odoo evolves much faster than any other solution.</p>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_title_sub" name="Title - Subtitle">
    <div class="o_mail_block_title_sub">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles">
                <tbody>
                    <tr>
                        <td class="o_mail_h_padding o_mail_v_padding o_mail_no_colorpicker">
                            <h2 class="o_mail_no_margin">Check this out!</h2>
                            <p class="o_mail_no_margin">Apps That Help You Grow Your Business!</p>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_comparison_table" name="Comparison">
    <div class="o_mail_block_comparison_table">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles">
                <tbody>
                    <tr>
                        <td class="o_mail_h_padding o_mail_v_padding o_mail_col_mv">
                            <table cellspacing="0" cellpadding="0" class="o_mail_table_styles">
                                <thead>
                                    <tr>
                                        <th class="bg-o-color-2 o_mail_v_padding o_mail_col_mv">
                                            <h2 class="text-center o_mail_no_margin">DEFAULT</h2>
                                        </th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr>
                                        <td class="text-center o_mail_col_mv bg-gray-lighter">
                                            <p class="o_mail_display_coupon text-center" style="margin-top:10px;">$8</p>
                                            <small>user / month (billed annually)</small>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td valign="top" class="o_mail_v_padding o_mail_col_mv bg-gray-lighter">
                                            <div class="separator"></div>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="o_mail_col_mv bg-gray-lighter">
                                            <p>Basic features</p>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="o_mail_col_mv bg-gray-lighter">
                                            <p>Basic management</p>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="o_mail_col_mv bg-gray-lighter">
                                            <p>No customization</p>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="o_mail_col_mv bg-gray-lighter">
                                            <p>No support</p>
                                        </td>
                                    </tr>
                                </tbody>
                                <tfoot>
                                    <tr>
                                        <td valign="top" class="o_mail_v_padding o_mail_col_mv bg-gray-lighter">
                                            <div class="separator"></div>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td valign="top" class="text-center o_mail_h_padding o_mail_col_mv bg-gray-lighter" style="padding-bottom: 20px;">
                                            <a role="button" href="#" class="btn btn-block btn-primary">More</a>
                                        </td>
                                    </tr>
                                </tfoot>
                            </table>
                        </td>
                        <td class="o_mail_h_padding o_mail_v_padding o_mail_col_mv">
                            <table cellspacing="0" cellpadding="0" class="o_mail_table_styles">
                                <thead>
                                    <tr>
                                        <th class="bg-o-color-2 o_mail_v_padding o_mail_col_mv">
                                            <h2 class="text-center o_mail_no_margin">PRO</h2>
                                        </th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr>
                                        <td class="text-center o_mail_col_mv bg-gray-lighter">
                                            <p class="o_mail_display_coupon text-center" style="margin-top:10px;">$18</p>
                                            <small>user / month (billed annually)</small>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td valign="top" class="o_mail_v_padding o_mail_col_mv bg-gray-lighter">
                                            <div class="separator"></div>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="o_mail_col_mv bg-gray-lighter">
                                            <p><strong>Advanced</strong> features</p>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="o_mail_col_mv bg-gray-lighter">
                                            <p><strong>Total</strong> management</p>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="o_mail_col_mv bg-gray-lighter">
                                            <p><strong>Fully customizable</strong></p>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td class="o_mail_col_mv bg-gray-lighter">
                                            <p><strong>24/7 Support</strong></p>
                                        </td>
                                    </tr>
                                </tbody>
                                <tfoot>
                                    <tr>
                                        <td valign="top" class="o_mail_v_padding o_mail_col_mv bg-gray-lighter">
                                            <div class="separator"></div>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td valign="top" class="text-center o_mail_h_padding o_mail_col_mv bg-gray-lighter" style="padding-bottom: 20px;">
                                            <a role="button" href="#" class="btn btn-block btn-primary">More</a>
                                        </td>
                                    </tr>
                                </tfoot>
                            </table>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_two_cols" name="Two Columns">
    <div class="o_mail_block_two_cols">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles o_mail_v_padding o_mail_col_table">
                <tbody>
                    <tr>
                        <td style="vertical-align:top;width:270px;" class="o_mail_col_mv o_mail_col_container">
                            <div>
                                <img src="/mass_mailing/static/src/img/theme_basic/s_default_image_block_two_cols_1.jpg" class="d-block mx-auto"/>
                                <h4 class="text-center">Column title</h4>
                                <p class="text-center o_mail_no_margin">
                                    Write one paragraph describing your product,
                                    services or a specific feature. To be successful
                                    your content needs to be useful to your readers.
                                </p>
                                <div class="text-center" style="margin-top:15px;">
                                    <a role="button" href="#" class="btn btn-link">Read More...</a>
                                </div>
                            </div>
                        </td>

                        <td style="vertical-align:top;width:270px;" class="o_mail_col_mv o_mail_col_container">
                            <div>
                                <img src="/mass_mailing/static/src/img/theme_basic/s_default_image_block_two_cols_2.jpg" class="d-block mx-auto"/>
                                <h4 class="text-center">Column title</h4>
                                <p class="text-center o_mail_no_margin">
                                    Write one paragraph describing your product,
                                    services or a specific feature. To be successful
                                    your content needs to be useful to your readers.
                                </p>
                                <div class="text-center" style="margin-top:15px;">
                                    <a role="button" href="#" class="btn btn-link">Read More...</a>
                                </div>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_three_cols" name="Three Columns">
    <div class="o_mail_block_three_cols">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles o_mail_v_padding o_mail_col_table">
                <tbody>
                    <tr>
                        <td style="vertical-align:top;width:180px;" class="o_mail_col_mv o_mail_col_container">
                            <div>
                                <img src="/mass_mailing/static/src/img/theme_basic/s_default_image_block_three_cols_1.jpg" class="d-block mx-auto"/>
                                <h4 class="text-center">Column Title</h4>
                                <p class="text-center o_mail_no_margin">
                                    A short description
                                </p>
                                <div class="text-center" style="margin-top:15px;">
                                    <a role="button" href="#" class="btn btn-link">Read More...</a>
                                </div>
                            </div>
                        </td>

                        <td style="vertical-align:top;width:180px;" class="o_mail_col_mv o_mail_col_container">
                            <div>
                                <img src="/mass_mailing/static/src/img/theme_basic/s_default_image_block_three_cols_2.jpg" class="d-block mx-auto"/>
                                <h4 class="text-center">Column Title</h4>
                                <p class="text-center o_mail_no_margin">
                                    A short description
                                </p>
                                <div class="text-center" style="margin-top:15px;">
                                    <a role="button" href="#" class="btn btn-link">Read More...</a>
                                </div>
                            </div>
                        </td>

                        <td style="vertical-align:top;width:180px;" class="o_mail_col_mv o_mail_col_container">
                            <div>
                                <img src="/mass_mailing/static/src/img/theme_basic/s_default_image_block_three_cols_3.jpg" class="d-block mx-auto"/>
                                <h4 class="text-center">Column Title</h4>
                                <p class="text-center o_mail_no_margin">
                                    A short description
                                </p>
                                <div class="text-center" style="margin-top:15px;">
                                    <a role="button" href="#" class="btn btn-link">Read More...</a>
                                </div>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_image_text" name="Image - Text">
    <div class="o_mail_block_image_text">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles">
                <tbody>
                    <tr>
                        <td style="width:270px;text-align:center;vertical-align:middle" class="o_mail_col_mv o_mail_img_container o_mail_h_padding o_mail_v_padding">
                            <img src="/mass_mailing/static/src/img/theme_basic/s_default_image_block_image_text.jpg" class="d-block mx-auto"/>
                        </td>
                        <td style="width:270px;vertical-align:middle;text-align:center;" class="o_mail_col_mv o_mail_h_padding o_mail_v_padding">
                            <h3>Omnichannel sales</h3>
                            <p class="o_mail_no_margin" style="text-align:justify;">Get your inside sales (CRM) fully integrated with online sales (eCommerce), in-store sales (Point of Sale) and marketplaces like eBay and Amazon.</p>
                            <div class="text-center" style="margin-top:15px;">
                                <a role="button" href="#" class="btn btn-link">Read More...</a>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_text_image" name="Text - Image">
    <div class="o_mail_block_text_image">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles">
                <tbody>
                    <tr>
                        <td style="width:270px;vertical-align:middle;text-align:center;" class="o_mail_col_mv o_mail_h_padding o_mail_v_padding">
                            <h3>A unique value</h3>
                            <p class="o_mail_no_margin" style="text-align:justify;">The open source model of Odoo has allowed us to leverage thousands of developers and business experts to build hundreds of apps in just a few years.</p>
                            <div class="text-center" style="margin-top:15px;">
                                <a role="button" href="#" class="btn btn-link">Read More...</a>
                            </div>
                        </td>
                        <td style="width:270px;text-align:center;vertical-align:middle" class="o_mail_col_mv o_mail_img_container o_mail_h_padding o_mail_v_padding">
                            <img src="/mass_mailing/static/src/img/theme_basic/s_default_image_block_text_image.jpg" class="d-block mx-auto"/>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_image" name="Image">
    <div class="o_mail_block_image">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles">
                <tbody>
                    <tr>
                        <td width="100%" align="center" style="text-align:center" class="o_mail_h_padding">
                            <img src="/mass_mailing/static/src/img/theme_basic/s_default_image_block_image.jpg" class="d-block mx-auto"/>
                        </td>
                    </tr>
                    <tr>
                        <td width="100%" align="center" style="text-align:center" class="o_mail_h_padding">
                            <table>
                                <td class="bg-o-color-2 o_mail_v_padding">
                                    <p class="text-center">With strong technical foundations, Odoo's framework is unique. It provides <strong>top notch usability that scales across all apps</strong>.</p>
                                </td>
                            </table>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_footer_separator" name="Separator">
    <div class="o_mail_block_footer_separator">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles o_mail_full_width_padding">
                <tbody>
                    <tr>
                        <td valign="top" style="width:100%;" class="o_mail_v_padding o_mail_no_colorpicker">
                            <div style="width:100%;" class="separator"></div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<template id="s_mail_block_footer_tag_line" name="Tag Line">
    <div class="o_mail_snippet_general o_mail_block_footer_tag_line s_footer_tag_line pt16 pb16 px-3 o_cc o_cc3">
        <h3 class="text-center">Apps That Help You Grow Your Business</h3>
        <br/>
        <div class="text-center">
            <a role="button" href="#" class="btn btn-primary">My Account</a>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_discount2" name="Promo Code">
    <div class="o_mail_block_discount2 mb32">
        <div class="o_mail_snippet_general">
            <div class="container o_mail_table_styles o_mail_h_padding">
                <div class="row">
                    <div class="col o_mail_v_padding">
                        <div class="text-center d-block mx-auto">
                            <p class="o_mail_display_coupon o_mail_no_margin text-center text-o-color-2" style="font-weight:800;">
                                $20
                            </p>
                            <h3 class="o_mail_no_margin">OFF YOUR NEXT ORDER!</h3>
                        </div>
                    </div>
                </div>
                <div class="row">
                    <div class="col text-center">
                        <p class="text-center" style="margin-top:10px;">
                            Use This Promo Code BEFORE 1st of August
                        </p>
                        <p class="o_mail_h_padding text-center">
                            <span style="line-height: 30px;"><small>CODE: </small></span><strong class="o_code h3 oe_unremovable">45A9E77DGW8455</strong>
                        </p>
                        <p class="text-center">
                            and save $20 on your next order!
                        </p>
                    </div>
                </div>
                <div class="row">
                    <div class="col mb16 mt16 text-center">
                        <a role="button" href="#" class="btn btn-primary">Use now</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_mail_block_discount1" name="Discount Offer">
    <div class="o_mail_snippet_general o_mail_block_discount1 s_discount1">
        <div class="container o_mail_table_styles">
            <div class="row">
                <div class="col-lg pt16 pb16 text-center">
                    <h4 class="text-o-color-2 text-center mt0 mb16" style="font-weight:800;">-20%</h4>
                    <p class="text-center">ON YOUR NEXT ORDER!</p>
                    <a role="button" href="#" class="btn btn-primary">Redeem Discount!</a>
                </div>
                <div class="col-lg pt16 pb16">
                    <p class="o_mail_no_margin">We are continuing to grow and we miss seeing you be a part of it! We've increased store hours and have lot's of new brands available. To welcome you back please accept this 20% discount on you next purchase by clicking the button.</p>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_mail_block_event" name="Event">
    <div class="o_mail_snippet_general o_mail_block_event s_event bg-300">
        <div class="container">
            <div class="row align-items-center">
                <div class="col-lg-3 px-0 text-center">
                    <h3 class="o_mail_no_margin">21 Jul</h3>
                    <p class="o_mail_no_margin">ALL DAY</p>
                </div>
                <div class="col-lg-2 px-0">
                    <img src="/mass_mailing/static/src/img/theme_default/s_default_image_block_event.jpg" class="img w-100"/>
                </div>
                <div class="col-lg-7">
                    <h4>Cybersecurity</h4>
                    <p>Cyber-threats continue to increase.
                        <br/>The discussion will examine how to develop new norms and integrate them into EU
                    </p>
                    <div class="text-left">
                        <a href="#" class="btn btn-primary">Register</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<!-- TODO: remove this template -->
<template id="s_mail_block_steps" name="Steps">
    <div class="o_mail_block_steps o_mail_no_colorpicker">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles o_mail_full_width_padding">
                <tbody>
                    <tr>
                        <td class="o_mail_col_mv o_mail_h_padding o_mail_v_padding">
                            <table align="center">
                                <tr>
                                    <td rowspan="2" class="text-right">
                                        &amp;nbsp;
                                        <span class="fa fa-compass fa-2x text-o-color-4" role="img" aria-label="Choose" title="Choose"></span>
                                        &amp;nbsp;
                                    </td>
                                    <td style="padding-left:10px;">
                                        <p class="o_mail_no_margin"><small>Step 1:</small></p>
                                        <h4 class="o_mail_no_margin">Choose</h4>
                                    </td>
                                </tr>
                            </table>
                        </td>
                        <td class="o_mail_col_mv o_mail_h_padding o_mail_v_padding">
                            <table align="center">
                                <tr>
                                    <td rowspan="2" class="text-right">
                                        &amp;nbsp;
                                        <span class="fa fa-credit-card fa-2x text-o-color-4" role="img" aria-label="Order" title="Order"></span>
                                        &amp;nbsp;
                                    </td>
                                    <td style="padding-left:10px;">
                                        <p class="o_mail_no_margin"><small>Step 2:</small></p>
                                        <h4 class="o_mail_no_margin ">Order</h4>
                                    </td>
                                </tr>
                            </table>
                        </td>
                        <td class="o_mail_col_mv o_mail_h_padding o_mail_v_padding">
                            <table align="center">
                                <tr>
                                    <td rowspan="2" class="text-right">
                                        &amp;nbsp;
                                        <span class="fa fa-smile-o fa-2x text-o-color-4" role="img" aria-label="Enjoy" title="Enjoy"></span>
                                        &amp;nbsp;
                                    </td>
                                    <td style="padding-left:10px;">
                                        <p class="o_mail_no_margin"><small>Step 3:</small></p>
                                        <h4 class="o_mail_no_margin ">Enjoy!</h4>
                                    </td>
                                </tr>
                            </table>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<template id="s_mail_block_footer_social" name="Footer Center">
    <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_center o_mail_snippet_general bg-200">
        <div class="container o_mail_table_styles">
            <div class="row">
                <div class="col-lg o_mail_footer_social">
                    <t t-call="mass_mailing.social_links"/>
                </div>
            </div>
            <div class="row">
                <div class="col-lg o_mail_footer_links">
                    <a role="button" href="/unsubscribe_from_list" class="btn btn-link">Unsubscribe</a>
                </div>
            </div>
            <div class="row">
                <div class="col-lg">
                    <p class="o_mail_no_margin o_mail_footer_copy">
                        <span class="fa fa-copyright" role="img" aria-label="Copyright" title="Copyright"/>
                        <t t-esc="datetime.datetime.now().year"/> All Rights Reserved
                    </p>
                </div>
            </div>
        </div>
    </div>
</template>

<template id="s_mail_block_footer_social_left" name="Footer Left">
    <div class="s_footer_social o_mail_block_footer_social o_mail_footer_social_left o_mail_snippet_general">
        <div class="container o_mail_table_styles">
            <div class="row">
                <div class="col-lg o_mail_footer_description">
                    <p t-if="res_company" class="o_mail_no_margin">
                        <strong><t t-esc="res_company.partner_id.name"/></strong>
                    </p>
                    <div class="o_mail_footer_links">
                        <a role="button" href="/unsubscribe_from_list" class="btn btn-link">Unsubscribe</a>
                    </div>
                </div>
                <div class="col-lg" align="right">
                    <div class="o_mail_footer_social pb16"><t t-call="mass_mailing.social_links"/></div>
                    <p class="o_mail_footer_copy"><span class="fa fa-copyright" role="img" aria-label="Copyright" title="Copyright"/> <t t-esc="datetime.datetime.now().year"/> All Rights Reserved</p>
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

<!-- Snippet themes Options -->
<template id="snippet_options">
    <t t-call="web_editor.snippet_options"/>
    <t t-out="0"/>

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

    <!-- H-ALIGN -->
    <div id="so_text_align" data-selector=".s_mail_text_highlight">
        <we-button-group string="Alignment">
            <we-button class="fa fa-fw fa-align-left" title="Left" data-select-class="text-left"/>
            <we-button class="fa fa-fw fa-align-center" title="Center" data-select-class="text-center"/>
            <we-button class="fa fa-fw fa-align-right" title="Right" data-select-class="text-right"/>
        </we-button-group>
    </div>

    <div id="so_width" data-selector=".s_mail_alert, .s_mail_blockquote, .s_mail_text_highlight">
        <we-select string="Width">
            <we-button data-select-class="w-25">25%</we-button>
            <we-button data-select-class="w-50">50%</we-button>
            <we-button data-select-class="w-75">75%</we-button>
            <we-button data-select-class="w-100" data-name="so_width_100">100%</we-button>
        </we-select>
    </div>

    <div id="so_block_align" data-selector=".s_mail_alert, .s_mail_blockquote, .s_mail_text_highlight">
        <we-button-group string="Alignment" data-dependencies="!so_width_100">
            <we-button class="fa fa-fw fa-align-left" title="Left" data-select-class="mr-auto"/>
            <we-button class="fa fa-fw fa-align-center" title="Center" data-select-class="mx-auto"/>
            <we-button class="fa fa-fw fa-align-right" title="Right" data-select-class="ml-auto"/>
        </we-button-group>
    </div>

    <div data-js="BodyWidth" data-selector=".o_layout" data-target=".o_mail_wrapper" data-no-check="true">
        <we-button-group string="Body Width">
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

    <div data-selector=".o_layout, .note-editable > div:not(.o_layout),
        .note-editable .oe_structure > div:not(:has(> .o_mail_snippet_general)),
        .note-editable .oe_structure > div.o_mail_snippet_general,
        .note-editable .oe_structure > div.o_mail_snippet_general .o_cc,
        .note-editable .oe_structure > div.o_mail_snippet_general .btn:not(.btn-link),
        td, td.o_mail_no_colorpicker div:first-child, th"
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

    <!-- Allow changing background images in Masonry -->
    <t t-call="web_editor.snippet_options_background_options">
        <t t-set="selector" t-value="'.s_masonry_block .row > div'"/>
        <t t-set="with_images" t-value="True"/>
    </t>

    <!-- COLOR | .s_three_columns | .s_comparisons -->
    <div data-js="Box"
         data-selector=".s_three_columns .row > div, .s_comparisons .row > div"
         data-target=".card-body">
        <we-colorpicker string="Background Color"
            data-select-style="true"
            data-no-transparency="true"
            data-css-property="background-color"
            data-color-prefix="bg-"/>
    </div>
    <!-- BORDER | .s_three_columns | .s_comparisons -->
    <div data-js="Box"
         data-selector=".s_three_columns .row > div, .s_comparisons .row > div"
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
    <div data-option-name="vAlignment" id="row_valign_snippet_option" data-selector=".s_text_image, .s_image_text, .s_three_columns" data-target=".row">
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
        <t t-call="mass_mailing.s_mail_block_header_logo" />
        <t t-call="mass_mailing.s_mail_block_title_text" />
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
                    <button name="%(action_create_mass_mailings_from_campaign)d" type="action" class="oe_highlight" groups="mass_mailing.group_mass_mailing_campaign" string="Send new Mailing"/>
            </xpath>
            <xpath expr="//div[hasclass('oe_button_box')]" position="inside">
                <button name="%(action_view_mass_mailings_from_campaign)d"
                    type="action" class="oe_stat_button order-9" icon="fa-envelope-o"
                    attrs="{'invisible': [('mailing_mail_count', '=', 0)]}" groups="mass_mailing.group_mass_mailing_campaign">
                    <field name="mailing_mail_count" widget="statinfo" string="Mailings"/>
                </button>
            </xpath>
            <xpath expr="//notebook" position="inside">
                <page string="Mailings" name="mailings" attrs="{'invisible': [('mailing_mail_count', '=', 0)]}">
                    <field name="mailing_mail_ids" readonly="1" nolabel="1">
                        <tree>
                            <field name="subject"/>
                            <field name="sent_date"/>
                            <field name="state"/>
                            <field name="delivered"/>
                            <field name="opened"/>
                            <field name="replied"/>
                            <field name="bounced"/>
                            <button name="action_duplicate" type="object" string="Duplicate"/>
                        </tree>
                    </field>
                    <div class="o_utm_campaign_mass_mailing_substats d-flex justify-content-end align-items-center">
                        <div class="d-flex justify-content-end align-items-center flex-column">
                            <label for="received_ratio" string="Delivered" class="m-0"/>
                            <div class="m-0">
                                <span class="text-right">
                                    <field name="received_ratio"/>
                                    <span>%</span>
                                </span>
                            </div>
                        </div>
                        <div class="d-flex justify-content-end align-items-center flex-column">
                            <label for="opened_ratio" string="Opened" class="m-0"/>
                            <div class="m-0">
                                <span class="text-right">
                                    <field name="opened_ratio"/>
                                    <span>%</span>
                                </span>
                            </div>
                        </div>
                        <div class="d-flex justify-content-end align-items-center flex-column">
                            <label for="replied_ratio" string="Replied" class="m-0"/>
                            <div class="m-0">
                                <span class="text-right">
                                    <field name="replied_ratio"/>
                                    <span>%</span>
                                </span>
                            </div>
                        </div>
                        <div class="d-flex justify-content-end align-items-center flex-column">
                            <label for="bounced_ratio" string="Bounced" class="m-0"/>
                            <div class="m-0">
                                <span class="text-right">
                                    <field name="bounced_ratio"/>
                                    <span>%</span>
                                </span>
                            </div>
                        </div>
                    </div>
                </page>
            </xpath>
        </field>
    </record>

    <record model="ir.ui.view" id="utm_campaign_view_kanban">
        <field name="name">utm.campaign.view.kanban</field>
        <field name="model">utm.campaign</field>
        <field name="inherit_id" ref="utm.utm_campaign_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='user_id']" position="after">
                <field name="mailing_mail_ids"/>
            </xpath>
            <xpath expr="//ul[@id='o_utm_actions']">
                <a name="%(action_view_mass_mailings_from_campaign)d" type="action"
                    t-attf-class="oe_mailings #{record.mailing_mail_ids.raw_value.length === 0 ? 'text-muted' : ''}"
                    groups="mass_mailing.group_mass_mailing_campaign">
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

    <menuitem name="Campaigns" id="menu_email_campaigns"
        parent="mass_mailing_menu_root" sequence="5"
        action="action_view_utm_campaigns"
        groups="mass_mailing.group_mass_mailing_campaign"/>

    <menuitem name="Campaign Stages" id="menu_view_mass_mailing_stages"
        parent="mass_mailing_configuration" sequence="1"
        groups="mass_mailing.group_mass_mailing_campaign"
        action="utm.action_view_utm_stage"/>

    <menuitem id="mass_mailing_tag_menu"
        parent="mass_mailing_configuration"
        action="utm.action_view_utm_tag"
        sequence="2"
        groups="mass_mailing.group_mass_mailing_campaign"/>
</odoo>

```

## File: views\snippets\s_alert.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<template id="s_alert" name="Alert">
    <div class="s_mail_alert s_alert_md alert-info w-100 clearfix o_mail_snippet_general pt16 pb16 px-3" data-snippet="s_alert">
        <div class="s_alert_icon" valign="top">
            <i class="fa fa-2x fa-info-circle"/>
        </div>
        <div class="s_alert_content">
            <p><b>Explain the benefits you offer</b>
            <br/>Don't write about products or services here, write about solutions.</p>
        </div>
    </div>
</template>

<template id="s_alert_options" inherit_id="mass_mailing.snippet_options">
    <xpath expr="//div[@id='so_width']" position="before">
        <div data-selector=".s_mail_alert" data-js="Alert">
            <we-select string="Type" data-apply-to=".fa.s_alert_icon" data-trigger="alert_colorpicker_opt">
                <we-button data-select-class="fa-user-circle" data-trigger-value="primary">Primary</we-button>
                <we-button data-select-class="fa-user-circle-o" data-trigger-value="secondary">Secondary</we-button>
                <we-button data-select-class="fa-info-circle" data-trigger-value="info">Info</we-button>
                <we-button data-select-class="fa-check-circle" data-trigger-value="success">Success</we-button>
                <we-button data-select-class="fa-exclamation-triangle" data-trigger-value="warning">Warning</we-button>
                <we-button data-select-class="fa-exclamation-circle" data-trigger-value="danger">Danger</we-button>
            </we-select>
        </div>
    </xpath>
    <!-- Keep those options in separate xpath for options order -->
    <xpath expr="//div[@id='so_width']" position="after">
        <div data-selector=".s_mail_alert">
            <we-select string="Size">
                <we-button data-select-class="s_alert_sm">Small</we-button>
                <we-button data-select-class="s_alert_md">Medium</we-button>
                <we-button data-select-class="s_alert_lg">Large</we-button>
            </we-select>
            <we-colorpicker string="Color" data-name="alert_colorpicker_opt"
                data-select-style="true"
                data-css-property="background-color"
                data-color-prefix="alert-"/>
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
    <div class="s_mail_blockquote o_cc o_cc2 pt16 pb16 o_mail_snippet_general s_blockquote_classic blockquote mb-0" data-snippet="s_blockquote">
        <div class="container">
            <div class="row">
                <div class="col-lg-1 d-flex align-items-start o_mail_no_resize" valign="top" align="right">
                    <i class="s_blockquote_icon fa fa-1x fa-quote-left"/>
                </div>
                <div class="col-lg-10 px-0">
                    <div class="s_blockquote_content">
                        <p><i>Write a quote here from one of your customers. Quotes are a great way to build confidence in your products or services.</i></p>
                        <div>
                            <img src="/web/image/mass_mailing.s_blockquote_default_image" class="s_blockquote_avatar img rounded-circle mr-2" style="width: 40px" alt=""/>
                            <span class="s_blockquote_author"><b>John DOE</b> &#8226; CEO of MyCompany</span>
                        </div>
                    </div>
                </div>
                <div class="col-lg-1 d-flex align-items-end o_mail_no_resize" valign="bottom" align="left">
                    <i class="s_blockquote_icon fa fa-1x fa-quote-right"/>
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
    <div class="s_call_to_action o_cc o_cc3 pt48 pb24 o_mail_snippet_general" data-snippet="s_call_to_action">
        <div class="container">
            <div class="row">
                <div class="col-lg-9 pb16">
                    <h3><b>50,000+ companies</b> run Odoo.</h3>
                    <p>Join us and make your company a better place.</p>
                </div>
                <div class="col-lg-3 pt8">
                    <a href="#" class="btn btn-primary btn-lg btn-block">Contact us</a>
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
        <div class="s_mail_color_blocks_2 o_mail_snippet_general" data-snippet="s_color_blocks_2">
            <div class="container">
                <div class="row">
                    <div class="col-lg-6 o_cc o_cc3 text-center pt32 pb32">
                        <i class="fa fa-shield fa-5x m-3"/>
                        <h2>A color block</h2>
                        <p>Color blocks are a simple and effective way to <b>present and highlight your content</b>. Choose an image or a color for the background. You can even resize and duplicate the blocks to create your own layout. Add images or icons to customize the blocks.</p>
                        <a href="#" class="btn btn-primary">More Details</a>
                    </div>
                    <div class="col-lg-6 o_cc o_cc5 text-center pt32 pb32">
                        <i class="fa fa-cube fa-5x m-3"/>
                        <h2>Another color block</h2>
                        <p>Color blocks are a simple and effective way to <b>present and highlight your content</b>. Choose an image or a color for the background. You can even resize and duplicate the blocks to create your own layout. Add images or icons to customize the blocks.</p>
                        <a href="#" class="btn btn-primary">More Details</a>
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
    <div class="s_company_team pt48 pb48 o_mail_snippet_general" data-snippet="s_company_team">
        <div class="container">
            <div class="row s_nb_column_fixed">
                <div class="col-lg-6 pt24 pb24">
                    <div class="container">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <div class="col-lg-3 pb16 px-0 o_not_editable" contenteditable="false" valign="top">
                                <img alt="" src="/web/image/mass_mailing.s_company_team_image_1" class="img-fluid rounded-circle mx-auto" contenteditable="true"/>
                            </div>
                            <div class="col-lg-9">
                                <h4>Tony Fred, CEO</h4>
                                <p>
                                    Founder and chief visionary, Tony is the driving force behind the company. He loves
                                    to keep his hands full by participating in the development of the software,
                                    marketing, and customer experience strategies.
                                </p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 pt24 pb24">
                    <div class="container">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <div class="col-lg-3 px-0 pb16 o_not_editable" contenteditable="false" valign="top">
                                <img alt="" src="/web/image/mass_mailing.s_company_team_image_2" class="img-fluid rounded-circle mx-auto" contenteditable="true"/>
                            </div>
                            <div class="col-lg-9">
                                <h4>Mich Stark, COO</h4>
                                <p>Mich loves taking on challenges. With his multi-year experience as Commercial Director in the software industry, Mich has helped the company to get where it is today. Mich is among the best minds.</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="row s_nb_column_fixed">
                <div class="col-lg-6 pt24 pb24">
                    <div class="container">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <div class="col-lg-3 px-0 pb16 o_not_editable" contenteditable="false" valign="top">
                                <img alt="" src="/web/image/mass_mailing.s_company_team_image_3" class="img-fluid rounded-circle mx-auto" contenteditable="true"/>
                            </div>
                            <div class="col-lg-9">
                                <h4>Aline Turner, CTO</h4>
                                <p>Aline is one of the iconic people in life who can say they love what they do. She mentors 100+ in-house developers and looks after the community of thousands of developers.</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 pt24 pb24">
                    <div class="container">
                        <div class="row s_col_no_resize s_col_no_bgcolor">
                            <div class="col-lg-3 px-0 pb16 o_not_editable" contenteditable="false" valign="top">
                                <img alt="" src="/web/image/mass_mailing.s_company_team_image_4" class="img-fluid rounded-circle mx-auto" contenteditable="true"/>
                            </div>
                            <div class="col-lg-9">
                                <h4>Iris Joe, CFO</h4>
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
    <div class="s_comparisons pt32 pb32 o_mail_snippet_general" data-snippet="s_comparisons">
        <div class="container">
            <div class="row">
                <div class="s_col_no_bgcolor text-center pt32 pb16 col-lg-6">
                    <div class="card o_cc o_cc2">
                        <div class="card-header"><span class="o_default_snippet_text" style="font-size:18px; font-weight: 500;">DEFAULT</span></div>
                        <div class="card-body text-center">
                            <h2 class="card-title o_mail_display_coupon text-center o_default_snippet_text">$8</h2>
                            <small class="o_default_snippet_text">user / month (billed annually)</small>

                        </div>
                        <ul class="list-group list-group-flush">
                            <li class="list-group-item o_default_snippet_text">Basic features</li>
                            <li class="list-group-item o_default_snippet_text">Basic management</li>
                            <li class="list-group-item o_default_snippet_text">No customization</li>
                            <li class="list-group-item o_default_snippet_text">No support</li>
                        </ul>
                        <div class="card-footer">
                            <a href="#" class="btn btn-block btn-primary o_default_snippet_text">More</a>
                        </div>
                    </div>
                </div>
                <div class="s_col_no_bgcolor text-center pt32 pb16 col-lg-6">
                    <div class="card o_cc o_cc3">
                        <div class="card-header"><span class="o_default_snippet_text" style="font-size:18px; font-weight: 500;">PRO</span></div>
                        <div class="card-body text-center">
                            <h2 class="card-title o_mail_display_coupon text-center o_default_snippet_text">$18</h2>
                            <small class="o_default_snippet_text">user / month (billed annually)</small>

                        </div>
                        <ul class="list-group list-group-flush">
                            <li class="list-group-item o_default_snippet_text">
                                <strong class="o_default_snippet_text">Advanced</strong>
                                features
                            </li>
                            <li class="list-group-item o_default_snippet_text">
                                <strong class="o_default_snippet_text">Total</strong>
                                management
                            </li>
                            <li class="list-group-item">
                                <strong class="o_default_snippet_text">Fully customizable</strong>
                            </li>
                            <li class="list-group-item">
                                <strong class="o_default_snippet_text">24/7 Support</strong>
                            </li>
                        </ul>
                        <div class="card-footer">
                            <a href="#" class="btn btn-block btn-primary o_default_snippet_text">More</a>
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
        <div class="o_mail_snippet_general o_mail_block_discount2 s_discount2 px-3 pt32 pb32">
            <h2 class="text-center"><b>GET $20 OFF</b></h2>
            <p class="text-center">
                Here's your coupon code - but hurry! Ends 9/28
            </p>
            <table border="0" cellpadding="0" cellspacing="0" align="center" class="border" style="border-collapse:collapse; mso-table-lspace:0pt; mso-table-rspace:0pt;">
                <tr>
                    <td width="50" height="50" align="center" class="o_mail_no_resize bg-300 text-center" style="width:50px!important; min-width: 50px; max-width:5.6rem;"><i class="fa fa-2x fa-ticket"></i></td>
                    <td width="200" height="50" align="center" class="text-center" style="font-size: 15px; line-height: 22px; font-weight: 700; min-width: 150px; width: 200px;">ENDOFSUMMER20</td>
                </tr>
            </table>
            <div class="text-center pt24">
                <a role="button" href="#" class="btn btn-primary">Use now</a>
            </div>
        </div>
    </template>
</odoo>

```

## File: views\snippets\s_cover.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="s_cover" name="Cover">
        <div class="s_cover o_mail_snippet_general" data-snippet="s_cover">
            <img src="/web/image/mass_mailing.s_default_image_block_banner" alt="Cover image" class="img-fluid w-100"/>
        </div>
    </template>
</odoo>

```

## File: views\snippets\s_features.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_features" name="Features">
    <div class="s_features pt32 pb32 o_mail_snippet_general" data-snippet="s_features">
        <div class="container">
            <div class="row">
                <div class="col-lg-4 text-center">
                    <i class="fa fa-3x fa-gear rounded bg-primary m-3"></i>
                    <h3>First Feature</h3>
                    <p>Tell what's the value for the <br/>customer for this feature.</p>
                </div>
                <div class="col-lg-4 text-center">
                    <i class="fa fa-3x fa-photo rounded bg-o-color-5 m-3"></i>
                    <h3>Second Feature</h3>
                    <p>Write what the customer would like to know, <br/>not what you want to show.</p>
                </div>
                <div class="col-lg-4 text-center">
                    <i class="fa fa-3x fa-leaf rounded bg-secondary m-3"></i>
                    <h3>Third Feature</h3>
                    <p>A small explanation of this great <br/>feature, in clear words.</p>
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
    <div class="s_mail_features_grid pt48 pb24 o_mail_snippet_general" data-snippet="s_features_grid">
        <div class="container">
            <div class="row">
                <div class="col-lg-6 s_col_no_bgcolor pb24">
                    <div class="container">
                        <div class="row no-gutters">
                            <div class="col-lg-12 pb24">
                                <h2>First list of Features</h2>
                                <h5>Add a great slogan.</h5>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0 text-right s_mail_features_grid_icon">
                                <i class="fa fa-2x fa-font-awesome rounded-circle bg-primary"></i>
                            </div>
                            <div class="col-lg-10 s_mail_features_grid_content pb16">
                                <h4>Change Icons</h4>
                                <p>Double click an icon to replace it with one of your choice.</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0 text-right s_mail_features_grid_icon">
                                <i class="fa fa-2x fa-files-o rounded-circle bg-primary"></i>
                            </div>
                            <div class="col-lg-10 s_mail_features_grid_content pb16">
                                <h4>Duplicate</h4>
                                <p>Duplicate blocks and columns to add more features.</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0 text-right s_mail_features_grid_icon">
                                <i class="fa fa-2x fa-trash rounded-circle bg-primary"></i>
                            </div>
                            <div class="col-lg-10 s_mail_features_grid_content pb16">
                                <h4>Delete Blocks</h4>
                                <p>Select and delete blocks to remove features.</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-lg-6 s_col_no_bgcolor pb24">
                    <div class="container">
                        <div class="row no-gutters">
                            <div class="col-lg-12 pb24">
                                <h2>Second list of Features</h2>
                                <h5>Add a great slogan.</h5>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0 text-right s_mail_features_grid_icon">
                                <i class="fa fa-2x fa-magic rounded bg-secondary"></i>
                            </div>
                            <div class="col-lg-10 s_mail_features_grid_content pb16">
                                <h4>Great Value</h4>
                                <p>Turn every feature into a benefit for your reader.</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0 text-right s_mail_features_grid_icon">
                                <i class="fa fa-2x fa-eyedropper rounded bg-secondary"></i>
                            </div>
                            <div class="col-lg-10 s_mail_features_grid_content pb16">
                                <h4>Edit Styles</h4>
                                <p>You can edit colors and backgrounds to highlight features.</p>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-lg-2 px-0 text-right s_mail_features_grid_icon">
                                <i class="fa fa-2x fa-picture-o rounded bg-secondary"></i>
                            </div>
                            <div class="col-lg-10 s_mail_features_grid_content pb16">
                                <h4>Sample Icons</h4>
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
                <we-button class="fa fa-fw fa-align-left" title="Left" data-select-class="mr-auto"/>
                <we-button class="fa fa-fw fa-align-center" title="Center" data-select-class="mx-auto"/>
                <we-button class="fa fa-fw fa-align-right" title="Right" data-select-class="ml-auto"/>
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
    <div class="s_text_image pt32 pb32 o_mail_snippet_general" data-snippet="s_image_text">
        <div class="container">
            <div class="row align-items-center">
                <div class="col-lg-6 px-0">
                    <img src="/mass_mailing/static/src/img/theme_default/s_default_image_block_image_text.jpg" class="img w-100" />
                </div>
                <div class="col-lg-6 pt16 pb16">
                    <h3>Omnichannel sales</h3>
                    <p class="text-justify">Get your inside sales (CRM) fully integrated with online sales (eCommerce), in-store sales (Point of Sale) and marketplaces like eBay and Amazon.</p>
                    <div class="text-left">
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
    <div class="s_masonry_block o_mail_snippet_general" data-vcss="001" data-snippet="s_masonry_block">
        <div class="container">
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
    <div class="s_media_list pt32 pb32 o_cc o_cc2 o_mail_snippet_general" data-vcss="001" data-snippet="s_media_list">
        <div class="container">
            <div class="row s_nb_column_fixed s_col_no_bgcolor">
                <div class="col-lg-12 s_media_list_item pt16 pb16" data-name="Media item">
                    <div class="row s_col_no_resize s_col_no_bgcolor no-gutters align-items-center o_cc o_cc1">
                        <div class="col-lg-4 align-self-stretch s_media_list_img_wrapper">
                            <img src="/web/image/mass_mailing.s_media_list_default_image_1" class="s_media_list_img h-100 w-100" alt=""/>
                        </div>
                        <div class="col-lg-8 s_media_list_body">
                            <h3>Media heading</h3>
                            <p>Use this snippet to build various types of components that feature a left- or right-aligned image alongside textual content. Duplicate the element to create a list that fits your needs.</p>
                            <a href="#" class="btn btn-primary mb-2">Discover</a>
                        </div>
                    </div>
                </div>
                <div class="col-lg-12 s_media_list_item pt16 pb16" data-name="Media item">
                    <div class="row s_col_no_resize s_col_no_bgcolor no-gutters align-items-center o_cc o_cc1">
                        <div class="col-lg-4 align-self-stretch s_media_list_img_wrapper">
                            <img src="/web/image/mass_mailing.s_media_list_default_image_2" class="s_media_list_img h-100 w-100" alt=""/>
                        </div>
                        <div class="col-lg-8 s_media_list_body">
                            <h3>Event heading</h3>
                            <p>Speakers from all over the world will join our experts to give inspiring talks on various topics. Stay on top of the latest business management trends &amp; technologies</p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-12 s_media_list_item pt16 pb16" data-name="Media item">
                    <div class="row s_col_no_resize s_col_no_bgcolor no-gutters align-items-center o_cc o_cc1">
                        <div class="col-lg-4 align-self-stretch s_media_list_img_wrapper">
                            <img src="/web/image/mass_mailing.s_media_list_default_image_3" class="s_media_list_img h-100 w-100" alt=""/>
                        </div>
                        <div class="col-lg-8 s_media_list_body">
                            <h3>Post heading</h3>
                            <p>Use this component for creating a list of featured elements to which you want to bring attention.</p>
                            <a href="#">Continue reading <i class="fa fa-long-arrow-right align-middle ml-1"/></a>
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
        <div class="s_numbers o_cc o_cc2 pt24 pb24 o_mail_snippet_general" data-snippet="s_numbers">
            <div class="container">
                <div class="row">
                    <div class="text-center pt24 pb24 col-lg-4" valign="top">
                        <span class="s_number display-4 o_default_snippet_text">12</span><br/>
                        <h6 class="o_default_snippet_text">Useful options</h6>
                    </div>
                    <div class="text-center pt24 pb24 col-lg-4" valign="top">
                        <span class="s_number display-4 o_default_snippet_text">45</span><br/>
                        <h6 class="">Beautiful snippets</h6>
                    </div>
                    <div class="text-center pt24 pb24 col-lg-4" valign="top">
                        <span class="s_number display-4 o_default_snippet_text">8</span><br/>
                        <h6 class="o_default_snippet_text">Amazing pages</h6>
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
    <div class="s_picture pt48 pb24 px-3 o_cc o_cc2 o_mail_snippet_general" data-snippet="s_picture">
        <h2 class="text-center">
            <font style="font-size: 48px;" class="o_default_snippet_text">A punchy Headline</font>
        </h2>
        <p class="text-center">With strong technical foundations, Odoo's framework is unique. It provides <strong>top notch usability that scales across all apps</strong>.</p>
        <img src="/mass_mailing/static/src/img/theme_default/s_default_image_block_image.jpg" class="img-thumbnail padding-large mx-auto d-block" width="500" alt=""/>
        <p class="figure-caption text-center py-3">Add a caption to enhance the meaning of this image.</p>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_product_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_product_list" name="Items">
    <div class="s_mail_product_list pt24 pb24 o_mail_snippet_general" data-snippet="s_product_list">
        <div class="container">
            <div class="row">
                <div class="col-lg-4">
                    <a href="">
                        <img src="/web/image/mass_mailing.s_product_list_default_image_1" alt="" class="img img-fluid"/>
                    </a>
                    <div class="pt16 text-center">
                        <p>Check out all our furniture</p>
                        <a class="btn btn-primary o_default_snippet_text" href="">Furniture</a>
                    </div>
                </div>
                <div class="col-lg-4">
                    <a href="">
                        <img src="/web/image/mass_mailing.s_product_list_default_image_2" alt="" class="img img-fluid"/>
                    </a>
                    <div class="pt16 text-center">
                        <p>Check out all our clothes</p>
                        <a class="btn btn-primary o_default_snippet_text" href="">Clothes</a>
                    </div>
                </div>
                <div class="col-lg-4">
                    <a href="">
                        <img src="/web/image/mass_mailing.s_product_list_default_image_3" alt="" class="img img-fluid"/>
                    </a>
                    <div class="pt16 text-center" data-original-title="" title="" aria-describedby="tooltip540733">
                        <p>Check out all our books</p>
                        <a class="btn btn-primary o_default_snippet_text" href="">Books</a>
                    </div>
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
    <div class="s_rating pt16 pb16 px-3 o_mail_snippet_general" data-vcss="001" data-icon="fa-star" data-snippet="s_rating">
        <h4 class="s_rating_title">Quality</h4>  
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
                    <i class="fa fa-fw fa-refresh mr-1"/> Replace Icon
                </we-button>
            </we-row>
            <we-row string="&#8985; Inactive">
                <we-colorpicker data-select-style="" data-apply-to=".s_rating_inactive_icons" data-css-property="color" data-color-prefix="text-"/>
                <we-button data-custom-icon="true" data-custom-active-icon="false" data-no-preview="true">
                    <i class="fa fa-fw fa-refresh mr-1"/> Replace Icon
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
    <div class="s_references pt32 pb32 o_mail_snippet_general" data-snippet="s_references">
        <div class="container">
            <h2 class="text-center">Our References</h2>
            <p class="text-center">We are in good company.</p>
            <div class="row">
                <div class="col-lg-3 pt16 pb16">
                    <img src="/web/image/mass_mailing.s_reference_demo_image_1" class="img img-fluid mx-auto" alt=""/>
                </div>
                <div class="col-lg-3 pt16 pb16">
                    <img src="/web/image/mass_mailing.s_reference_demo_image_2" class="img img-fluid mx-auto" alt=""/>
                </div>
                <div class="col-lg-3 pt16 pb16">
                    <img src="/web/image/mass_mailing.s_reference_demo_image_3" class="img img-fluid mx-auto" alt=""/>
                </div>
                <div class="col-lg-3 pt16 pb16">
                    <img src="/web/image/mass_mailing.s_reference_demo_image_4" class="img img-fluid mx-auto" alt=""/>
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
    <div class="s_showcase pt48 pb48 o_mail_snippet_general" data-snippet="s_showcase">
        <!-- TODO: (below) issue with height: `fit-content` is not supported, can we calculate it in px in translation ? 
        empty div height is 0 unless table has defined height (div.col-1 > div.w-50.h100.border-right)-->
        <div class="container" style="height: fit-content;">
            <div class="row no-gutters s_col_no_resize s_col_no_bgcolor s_nb_column_fixed">
                <div class="col-lg-5 text-right pb24" align="right">
                    <div class="mb-2">
                        <h3 class="d-inline-block">First feature</h3>
                        <i class="fa fa-2x fa-desktop text-secondary ml-3"/>
                    </div>
                    <p>A short description of this great feature.</p>
                </div>
                <div class="col-lg-2 o_mail_no_resize" align="left">
                    <div class="w-50 h-100 border-right"/>
                </div>
                <div class="col-lg-5 pb24" align="left">
                    <div class="mb-2">
                        <i class="fa fa-2x fa-heart text-secondary mr-3"/>
                        <h3 class="d-inline-block">Another feature</h3>
                    </div>
                    <p>A short description of this great feature.</p>
                </div>
            </div>
            <div class="row no-gutters s_col_no_resize s_col_no_bgcolor s_nb_column_fixed">
                <div class="col-lg-5 text-right" align="right">
                    <div class="mb-2">
                        <h3 class="d-inline-block">Second feature</h3>
                        <i class="fa fa-2x fa-paint-brush text-secondary ml-3"/>
                    </div>
                    <p>A short description of this great feature.</p>
                </div>
                <div class="col-lg-2 o_mail_no_resize" align="left">
                    <div class="w-50 h-100 border-right"/>
                </div>
                <div class="col-lg-5" align="left">
                    <div class="mb-2">
                        <i class="fa fa-2x fa-gift text-secondary mr-3"/>
                        <h3 class="d-inline-block">Last Feature</h3>
                    </div>
                    <p>A short description of this great feature.</p>
                </div>
            </div>
        </div>
        <div class="container text-center pt32" align="center">
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
        <div class="s_text_block pt40 pb40 px-3 o_mail_snippet_general" data-snippet="s_text_block">
            <div class="container s_allow_columns">
                <p class="o_default_snippet_text"> The open source model of Odoo has allowed us to leverage thousands of developers and
                    business experts to build hundreds of apps in just a few years.</p>
                <p class="o_default_snippet_text"> With strong technical foundations, Odoo's framework is unique.
                    It provides top notch usability that scales across all apps.</p>
                <p class="o_default_snippet_text"> Usability improvements made on Odoo will automatically apply to all
                    of our fully integrated apps.</p>
                <p class="o_default_snippet_text"> That way, Odoo evolves much faster than any other solution.</p>
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
    <div class="s_mail_text_highlight o_cc o_cc3 pt32 pb32 w-100 o_mail_snippet_general" data-snippet="s_text_highlight">
        <h3 class=" text-center">Text Highlight</h3>
        <p class="o_mail_no_margin text-center">Put the focus on what you have to say!</p>
    </div>
</template>

</odoo>

```

## File: views\snippets\s_text_image.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_text_image" name="Text - Image">
    <div class="s_text_image o_mail_snippet_general" data-snippet="s_text_image">
        <div class="container">
            <div class="row align-items-center">
                <div class="col-lg-6 pt16 pb16">
                    <h3>A unique value</h3>
                    <p>The open source model of Odoo has allowed us to leverage thousands of developers and business experts to build hundreds of apps in just a few years.</p>
                    <div class="text-left">
                        <a href="#" class="btn btn-link">Read More</a>
                    </div>
                </div>
                <div class="col-lg-6 px-0">
                    <img src="/mass_mailing/static/src/img/theme_default/s_default_image_block_text_image.jpg" class="img w-100"/>
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
    <div class="s_three_columns o_cc o_cc2 pt32 pb32 o_mail_snippet_general" data-snippet="s_three_columns">
        <div class="container">
            <div class="row d-flex align-items-stretch">
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16">
                    <div class="card bg-white h-100">
                        <img class="card-img-top" src="/mass_mailing/static/src/img/theme_default/s_default_image_block_three_cols_1.jpg" alt=""/>
                        <div class="card-body">
                            <h3 class="card-title">Feature One</h3>
                            <p class="card-text">Adapt these three columns to fit your design need. To duplicate, delete or move columns, select the column and use the top icons to perform your action.</p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16">
                    <div class="card bg-white h-100">
                        <img class="card-img-top" src="/mass_mailing/static/src/img/theme_default/s_default_image_block_three_cols_2.jpg" alt=""/>
                        <div class="card-body">
                            <h3 class="card-title">Feature Two</h3>
                            <p class="card-text">To add a fourth column, reduce the size of these three columns using the right icon of each block. Then, duplicate one of the columns to create a new one as a copy.</p>
                        </div>
                    </div>
                </div>
                <div class="col-lg-4 s_col_no_bgcolor pt16 pb16">
                    <div class="card bg-white h-100">
                        <img class="card-img-top" src="/mass_mailing/static/src/img/theme_default/s_default_image_block_three_cols_3.jpg" alt=""/>
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
    <div class="s_title o_mail_snippet_general pt32 pb32" data-snippet="s_title">
        <div class="container s_allow_columns">
            <h1 class="text-center"><font style="font-size: 42px;">Your Title</font></h1>
        </div>
    </div>
</template>

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
        mass_mail_layout = self.env.ref('mass_mailing.mass_mailing_mail_layout')

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
                'body_html': mass_mail_layout._render({'body': full_body}, engine='ir.qweb', minimal_qcontext=True),
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
    campaign_id = fields.Many2one('utm.campaign', string='Mass Mailing Campaign')
    mass_mailing_name = fields.Char(string='Mass Mailing Name')
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
            mass_mail_layout = self.env.ref('mass_mailing.mass_mailing_mail_layout', raise_if_not_found=False)
            for res_id in res_ids:
                mail_values = res[res_id]
                if mail_values.get('body_html') and mass_mail_layout:
                    mail_values['body_html'] = mass_mail_layout._render({'body': mail_values['body_html']}, engine='ir.qweb', minimal_qcontext=True)

                trace_vals = {
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
            </field>
        </record>

</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import mail_compose_message
from . import mailing_contact_to_list
from . import mailing_list_merge
from . import mailing_mailing_test
from . import mailing_mailing_schedule_date

```

