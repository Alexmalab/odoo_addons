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
    'version': '2.2',
    'sequence': 60,
    'website': 'https://www.odoo.com/page/mailing',
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
        'wizard/mail_compose_message_views.xml',
        'wizard/mailing_list_merge_views.xml',
        'wizard/mailing_mailing_schedule_date_views.xml',
        'wizard/mailing_mailing_test_views.xml',
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
    ],
    'demo': [
        'data/mass_mailing_demo.xml',
    ],
    'qweb': [
        'static/src/xml/*.xml',
    ],
    'application': True,
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
from odoo.http import request
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

        request.env['mailing.trace'].sudo().set_opened(mail_mail_ids=[mail_id])
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
            base_url = request.env['ir.config_parameter'].sudo().get_param('web.base.url').rstrip('/')
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
                res[mailing_id] = mailing._render_template(res[mailing_id], mailing.mailing_model_real, [res_id], post_process=True)[res_id]

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
        return werkzeug.utils.redirect(request.env['link.tracker'].get_url_from_code(code), 301)

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

## File: data\mailing_data_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">
    <template id="mass_mailing.mass_mailing_kpi_link_trackers" name="Email Marketing: mailing link trackers statistic">
        <table t-if="link_trackers" cellspacing="0" cellpadding="0" align="center" border="0" bgcolor="#eeeeee" style="width:100%; font-family: Arial,Helvetica,Verdana,sans-serif;">
            <tr>
                <td align="center" valign="top">
                    <table bgcolor="#ffffff" cellspacing="0" cellpadding="0" width="650" align="center" border="0" style="border: 1px solid #eeeeee; border-bottom: none;border-top: none; width: 100%; max-width: 650px; padding:0 30px 30px 30px">
                        <tr>
                            <td style="width: 100%;">
                                <table cellspacing="0" cellpadding="0" border="0" width="580" align="center" style="width:100%; max-width:580px;">
                                    <tr>
                                        <td align="left" style="border-bottom: 1px solid #eeeeee;">
                                            <span style="color:#282f33; font-size: 15px; font-weight: bold; line-height: 30px">
                                                <t t-esc="'Click Rate Report on %i Emails Sent' % object.sent"/>
                                            </span>
                                        </td>
                                    </tr>
                                </table>
                            </td>
                        </tr>
                        <tr>
                            <td style="margin: 0; padding:0;">
                                <table cellspacing="0" cellpadding="0" border="0" width="580" align="center" style="width:100%; max-width:580px;">
                                    <tr style="color: #875a7b; font-size: 16px; font-weight: 500; border-bottom: 1px solid #e7e7e7;">
                                        <td style="width: 70%;padding: 10px 0; text-align: left;">Label</td>
                                        <td style="width: 30%;padding: 10px 0; text-align: right;">%Click (Total)</td>
                                    </tr>
                                    <tr t-foreach="link_trackers" t-as="link_tracker" style="color: #888888; font-size: 15px; font-weight: 300;">
                                        <td style="width: 70%;padding: 10px 0; text-align: left;">
                                            <a t-att-href="link_tracker.absolute_url" target="_blank" style="color: #56b3b5; text-decoration: none;" t-esc="link_tracker.label or link_tracker.url"/>
                                        </td>
                                        <td style="width: 30%;padding: 10px 0; text-align: right;">
                                            <t t-esc="int(link_tracker.count * 100 / object.sent) if object.sent else 0"/>% (<t t-esc="link_tracker.count"/>)
                                        </td>
                                    </tr>
                                </table>
                            </td>
                        </tr>
                    </table>
                </td>
            </tr>
        </table>
    </template>
</data>
</odoo>

```

## File: data\mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- Reference: https://litmus.com/community/learning/24-how-to-code-a-responsive-email-from-scratch -->
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
                <t t-raw="body"/>
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
        <!-- Cron that process the mass mailing queue -->
        <record id="ir_cron_mass_mailing_queue" model="ir.cron">
            <field name="name">Email Marketing: Process queue</field>
            <field name="model_id" ref="model_mailing_mailing"/>
            <field name="state">code</field>
            <field name="code">model._process_mass_mailing_queue()</field>
            <field name="user_id" ref="base.user_root" />
            <field name="interval_number">60</field>
            <field name="interval_type">minutes</field>
            <field name="numbercall">-1</field>
            <field eval="False" name="doall" />
        </record>
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

## File: data\mass_mailing_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="mass_mail_attach_1" model="ir.attachment">
            <field name="datas">bWlncmF0aW9uIHRlc3Q=</field>
            <field name="name">SampleDoc.doc</field>
        </record>

        <!-- Create Contacts -->
        <record id="mass_mail_contact_1" model="mailing.contact">
            <field name="name">Aristide Antario</field>
            <field name="email">aa@example.com</field>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_data')])]"/>
        </record>
        <record id="mass_mail_contact_2" model="mailing.contact">
            <field name="name">Beverly Bridge</field>
            <field name="email">bb@example.com</field>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_data')])]"/>
        </record>
        <record id="mass_mail_contact_3" model="mailing.contact">
            <field name="name">Carol Cartridge</field>
            <field name="email">cc@example.com</field>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_data')])]"/>
        </record>
        <record id="mass_mail_contact_4" model="mailing.contact">
            <field name="name">David Dawson</field>
            <field name="email">dd@example.com</field>
        </record>
        <record id="mass_mail_contact_5" model="mailing.contact">
            <field name="name">Elsa Ericson</field>
            <field name="email">ee@example.com</field>
            <field name="list_ids" eval="[(6,0,[ref('mass_mailing.mailing_list_data')])]"/>
        </record>

        <!-- Create Opt-out Records -->
        <record id="mass_mail_contact_list_rel_1" model="mailing.contact.subscription">
            <field name="list_id" ref="mass_mailing.mailing_list_data"/>
            <field name="contact_id" ref="mass_mailing.mass_mail_contact_4"/>
            <field name="opt_out">True</field>
        </record>

        <!-- Create Blacklist Records -->
        <record id="blacklist_1" model="mail.blacklist">
            <field name="email">ee@example.com</field>
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
            <field name="reply_to_mode">email</field>
            <field name="reply_to">Info &lt;info@yourcompany.example.com&gt;</field>
            <field name="body_html" type="html">
<div class="o_layout o_default_theme">
    <table class="o_mail_wrapper" style="border-collapse:collapse;">
        <tbody>
            <tr>
                <td class="o_mail_no_resize o_not_editable" style="text-align:left;"> </td>
                <td class="o_mail_no_options o_mail_wrapper_td oe_structure" style="text-align:left;width:100%;">
                    <div class="o_mail_block_header_logo" data-snippet="s_mail_block_header_logo">
                        <div class="o_mail_snippet_general" style="margin:0px auto 0px auto;background-color:rgb(255, 255, 255);max-width:600px;width:100%;">
                            <table align="center" cellspacing="0" cellpadding="0" border="0" class="o_mail_table_styles o_mail_h_padding" style="padding:0 20px 0 20px;width:100%;border-collapse:separate;">
                                <tbody>
                                    <tr>
                                        <td valign="center" width="30%" class="text-center o_mail_v_padding pb0" style="padding:20px 0 0px 0;vertical-align:middle;text-align:center;">
                                            <a href="http://www.example.com" style="text-decoration:none;font-weight:bold;background-color:transparent;color:rgb(100, 89, 116);">
                                                <img border="0" src="/mass_mailing/static/src/img/theme_default/s_default_image_logo.png" style="border-style:none;height:auto;vertical-align:middle;max-width:400px;width:auto"/> ​
                                            </a>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                    <div class="o_mail_block_footer_separator" data-snippet="s_mail_block_footer_separator" style="margin:0 20px 0 20px;">
                        <div class="o_mail_snippet_general" style="margin:0px auto 0px auto;background-color:rgb(255, 255, 255);max-width:600px;width:100%;">
                            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles o_mail_full_width_padding" style="width:100%;border-collapse:separate;">
                                <tbody>
                                    <tr>
                                        <td valign="top" style="padding:20px 0 20px 0;text-align:left;vertical-align:top;width:100%;" class="o_mail_v_padding o_mail_no_colorpicker">
                                            <div style="background-color:rgb(245, 245, 245);height:2px;width:100%;" class="separator"></div>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                    <div class="o_mail_block_paragraph" data-snippet="s_mail_block_paragraph">
                        <div class="o_mail_snippet_general" style="margin:0px auto 0px auto;background-color:rgb(255, 255, 255);max-width:600px;width:100%;">
                            <table align="center" cellspacing="0" cellpadding="0" border="0" class="o_mail_table_styles" style="width:100%;border-collapse:separate;">
                                <tbody>
                                    <tr>
                                        <td width="100%" class="o_mail_h_padding o_mail_v_padding o_mail_no_colorpicker" style="padding:20px;text-align:left;vertical-align:top;">
                                            <p style="margin:0px 0 1rem 0;font-size:14px;">
                                                Great stories have personality. Consider telling a great story that provides personality. Writing a story with personality for potential clients will assist with making a relationship connection. This shows up in small quirks like word choices or phrases. Write from your point of view, not from someone else's experience.
                                                <br/>Great stories are for everyone even when only written for just one person. If you try to write with a wide general audience in mind, your story will ring false and be bland. No one will be interested. Write for one person. If it’s genuine for the one, it’s genuine for the rest.
                                            </p>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                    <div class="o_mail_block_footer_social o_mail_footer_social_center" data-snippet="s_mail_block_footer_social">
                        <div class="o_mail_snippet_general" style="margin:0px auto 0px auto;background-color:rgb(255, 255, 255);max-width:600px;width:100%;">
                            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles o_mail_full_width_padding" style="border-style:solid none none none;padding:20px 0 20px 0;border-top-color:rgb(245, 245, 245);border-top-width:2px;width:100%;border-collapse:separate;">
                                <tbody>
                                    <tr>
                                        <td class="o_mail_footer_links o_default_snippet_text" style="padding:10px 0 10px 0;text-align:center;vertical-align:middle;">
                                            <a href="/unsubscribe_from_list" class="btn btn-link o_default_snippet_text" style="text-decoration:none;border-radius:0.25rem;border-style:solid;padding:0px;cursor:pointer;line-height:1.5;font-size:12px;border-left-color:transparent;border-bottom-color:transparent;border-right-color:transparent;border-top-color:transparent;border-left-width:1px;border-bottom-width:1px;border-right-width:1px;border-top-width:1px;user-select:none;vertical-align:middle;white-space:nowrap;text-align:center;font-weight:bold;display:inline-block;background-color:transparent;color:rgb(100, 89, 116);">Unsubscribe</a> |

                                            <a href="/contactus" class="btn btn-link o_default_snippet_text" style="text-decoration:none;border-radius:0.25rem;border-style:solid;padding:0px;cursor:pointer;line-height:1.5;font-size:12px;border-left-color:transparent;border-bottom-color:transparent;border-right-color:transparent;border-top-color:transparent;border-left-width:1px;border-bottom-width:1px;border-right-width:1px;border-top-width:1px;user-select:none;vertical-align:middle;white-space:nowrap;text-align:center;font-weight:bold;display:inline-block;background-color:transparent;color:rgb(100, 89, 116);">Contact</a>
                                        </td>
                                    </tr>
                                    <tr>
                                        <td style="text-align:left;vertical-align:middle;">
                                            <p class="o_mail_footer_copy" style="margin:0px 0 1rem 0;text-align:center;font-weight:bold;color:rgb(147, 146, 146);font-size:9px;">
                                                <img src="/web_editor/font_to_img/61945/rgb(147,146,146)/9" data-class="fa fa-copyright" style="border-style:none;max-width:100%;width:100%;vertical-align:middle;height: auto; width: auto;"/>2018 All Rights Reserved
                                            </p>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </td>
            </tr>
        </tbody>
    </table>
    <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles o_mail_full_width_padding" style="width:100%;border-collapse:separate;">
        <tbody>
            <tr>
                <td align="center" style="padding:16px 0 16px 0;" class="pt16 pb16">
                  Powered by <a target="_blank" href="https://www.odoo.com" style="text-decoration:none;background-color:transparent;color:#875A7B;">Odoo</a>
                </td>
            </tr>
        </tbody>
    </table>
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
            <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="opened" eval="(DateTime.today() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="replied" eval="(DateTime.today() - relativedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_1" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111001@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_13"/>
            <field name="email">kim.snyder96@example.com</field>
            <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="opened" eval="(DateTime.today() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="replied" eval="(DateTime.today() - relativedelta(days=0)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_2" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111002@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_14"/>
            <field name="email">edith.sanchez68@example.com</field>
            <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="opened" eval="(DateTime.today() - relativedelta(days=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_3" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111003@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_24"/>
            <field name="email">theodore.gardner36@example.com</field>
            <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="opened" eval="(DateTime.today() - relativedelta(days=1)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_4" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111004@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_32"/>
            <field name="email">sandra.neal80@example.com</field>
            <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_5" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111005@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_33"/>
            <field name="email">julie.richards84@example.com</field>
            <field name="sent" eval="False"/>
            <field name="exception" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_6" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111006@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_34"/>
            <field name="email">travis.mendoza24@example.com</field>
            <field name="sent" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
            <field name="bounced" eval="(DateTime.today() - relativedelta(days=3)).strftime('%Y-%m-%d %H:%M:%S')"/>
        </record>
        <record id="mass_mail_1_stat_7" model="mailing.trace">
            <field name="mass_mailing_id" ref="mass_mail_1"/>
            <field name="message_id">1111007@odoo.com</field>
            <field name="model">res.partner</field>
            <field name="res_id" ref="base.res_partner_address_34"/>
            <field name="email">travis.mendoza24@example.com</field>
            <field name="sent" eval="False"/>
            <field name="ignored" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M:%S')"/>
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
from datetime import datetime
from dateutil.relativedelta import relativedelta
from werkzeug.urls import url_join

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError
from odoo.osv import expression

_logger = logging.getLogger(__name__)

MASS_MAILING_BUSINESS_MODELS = [
    'crm.lead',
    'event.registration',
    'hr.applicant',
    'res.partner',
    'event.track',
    'sale.order',
    'mailing.list',
    'mailing.contact'
]

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
    def default_get(self, fields):
        vals = super(MassMailing, self).default_get(fields)
        if 'contact_list_ids' in fields and not vals.get('contact_list_ids') and vals.get('mailing_model_id'):
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
    subject = fields.Char('Subject', help='Subject of your Mailing', required=True, translate=True)
    preview = fields.Char(
        'Preview', translate=True,
        help='Catchy preview sentence that encourages recipients to open this email.\n'
             'In most inboxes, this is displayed next to the subject.\n'
             'Keep it empty if you prefer the first characters of your email content to appear instead.')
    email_from = fields.Char(string='Send From', required=True,
        default=lambda self: self.env.user.email_formatted)
    sent_date = fields.Datetime(string='Sent Date', copy=False)
    schedule_date = fields.Datetime(string='Scheduled for', tracking=True)
    # don't translate 'body_arch', the translations are only on 'body_html'
    body_arch = fields.Html(string='Body', translate=False)
    body_html = fields.Html(string='Body converted to be sent by mail', sanitize_attributes=False)
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
    reply_to_mode = fields.Selection([
        ('thread', 'Recipient Followers'), ('email', 'Specified Email Address')],
        string='Reply-To Mode', compute='_compute_reply_to_mode',
        readonly=False, store=True,
        help='Thread: replies go to target document. Email: replies are routed to a given email.')
    reply_to = fields.Char(
        string='Reply To', compute='_compute_reply_to', readonly=False, store=True,
        help='Preferred Reply-To Address')
    # recipients
    mailing_model_real = fields.Char(string='Recipients Real Model', compute='_compute_model')
    mailing_model_id = fields.Many2one(
        'ir.model', string='Recipients Model', ondelete='cascade', required=True,
        domain=[('model', 'in', MASS_MAILING_BUSINESS_MODELS)],
        default=lambda self: self.env.ref('mass_mailing.model_mailing_list').id)
    mailing_model_name = fields.Char(
        string='Recipients Model Name', related='mailing_model_id.model',
        readonly=True, related_sudo=True)
    mailing_domain = fields.Char(
        string='Domain', compute='_compute_mailing_domain',
        readonly=False, store=True)
    mail_server_id = fields.Many2one('ir.mail_server', string='Mail Server',
        default=_get_default_mail_server_id,
        help="Use a specific mail server in priority. Otherwise Odoo relies on the first outgoing mail server available (based on their sequencing) as it does for normal mails.")
    contact_list_ids = fields.Many2many('mailing.list', 'mail_mass_mailing_list_rel', string='Mailing Lists')
    contact_ab_pc = fields.Integer(string='A/B Testing percentage',
        help='Percentage of the contacts that will be mailed. Recipients will be taken randomly.', default=100)
    unique_ab_testing = fields.Boolean(string='Allow A/B Testing', default=False,
        help='If checked, recipients will be mailed only once for the whole campaign. '
             'This lets you send different mailings to randomly selected recipients and test '
             'the effectiveness of the mailings, without causing duplicate messages.')
    kpi_mail_required = fields.Boolean('KPI mail required', copy=False)
    # statistics data
    mailing_trace_ids = fields.One2many('mailing.trace', 'mass_mailing_id', string='Emails Statistics')
    total = fields.Integer(compute="_compute_total")
    scheduled = fields.Integer(compute="_compute_statistics")
    expected = fields.Integer(compute="_compute_statistics")
    ignored = fields.Integer(compute="_compute_statistics")
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

    def _compute_total(self):
        for mass_mailing in self:
            total = self.env[mass_mailing.mailing_model_real].search_count(mass_mailing._parse_mailing_domain())
            if mass_mailing.contact_ab_pc < 100:
                total = int(total / 100.0 * mass_mailing.contact_ab_pc)
            mass_mailing.total = total

    def _compute_clicks_ratio(self):
        self.env.cr.execute("""
            SELECT COUNT(DISTINCT(stats.id)) AS nb_mails, COUNT(DISTINCT(clicks.mailing_trace_id)) AS nb_clicks, stats.mass_mailing_id AS id
            FROM mailing_trace AS stats
            LEFT OUTER JOIN link_tracker_click AS clicks ON clicks.mailing_trace_id = stats.id
            WHERE stats.mass_mailing_id IN %s
            GROUP BY stats.mass_mailing_id
        """, [tuple(self.ids) or (None,)])
        mass_mailing_data = self.env.cr.dictfetchall()
        mapped_data = dict([(m['id'], 100 * m['nb_clicks'] / m['nb_mails']) for m in mass_mailing_data])
        for mass_mailing in self:
            mass_mailing.clicks_ratio = mapped_data.get(mass_mailing.id, 0)

    def _compute_statistics(self):
        """ Compute statistics of the mass mailing """
        for key in (
            'scheduled', 'expected', 'ignored', 'sent', 'delivered', 'opened',
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
                COUNT(CASE WHEN s.sent is not null THEN 1 ELSE null END) AS sent,
                COUNT(CASE WHEN s.scheduled is not null AND s.sent is null AND s.exception is null AND s.ignored is null AND s.bounced is null THEN 1 ELSE null END) AS scheduled,
                COUNT(CASE WHEN s.scheduled is not null AND s.sent is null AND s.exception is null AND s.ignored is not null THEN 1 ELSE null END) AS ignored,
                COUNT(CASE WHEN s.sent is not null AND s.exception is null AND s.bounced is null THEN 1 ELSE null END) AS delivered,
                COUNT(CASE WHEN s.opened is not null THEN 1 ELSE null END) AS opened,
                COUNT(CASE WHEN s.clicked is not null THEN 1 ELSE null END) AS clicked,
                COUNT(CASE WHEN s.replied is not null THEN 1 ELSE null END) AS replied,
                COUNT(CASE WHEN s.bounced is not null THEN 1 ELSE null END) AS bounced,
                COUNT(CASE WHEN s.exception is not null THEN 1 ELSE null END) AS failed
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
            total = (row['expected'] - row['ignored']) or 1
            row['received_ratio'] = 100.0 * row['delivered'] / total
            row['opened_ratio'] = 100.0 * row['opened'] / total
            row['replied_ratio'] = 100.0 * row['replied'] / total
            row['bounced_ratio'] = 100.0 * row['bounced'] / total
            self.browse(row.pop('mailing_id')).update(row)

    def _compute_next_departure(self):
        cron_next_call = self.env.ref('mass_mailing.ir_cron_mass_mailing_queue').sudo().nextcall
        str2dt = fields.Datetime.from_string
        cron_time = str2dt(cron_next_call)
        for mass_mailing in self:
            if mass_mailing.schedule_date:
                schedule_date = str2dt(mass_mailing.schedule_date)
                mass_mailing.next_departure = max(schedule_date, cron_time)
            else:
                mass_mailing.next_departure = cron_time

    @api.depends('mailing_type')
    def _compute_medium_id(self):
        for mailing in self:
            if mailing.mailing_type == 'mail' and not mailing.medium_id:
                mailing.medium_id = self.env.ref('utm.utm_medium_email').id

    @api.depends('mailing_model_id')
    def _compute_model(self):
        for record in self:
            record.mailing_model_real = (record.mailing_model_id.model != 'mailing.list') and record.mailing_model_id.model or 'mailing.contact'

    @api.depends('mailing_model_id')
    def _compute_reply_to_mode(self):
        """ For main models not really using chatter to gather answers (contacts
        and mailing contacts), set reply-to as email-based. Otherwise answers
        by default go on the original discussion thread (business document). Note
        that mailing_model being mailing.list means contacting mailing.contact
        (see mailing_model_name versus mailing_model_real). """
        for mailing in self:
            if mailing.mailing_model_id.model in ['res.partner', 'mailing.list', 'mailing.contact']:
                mailing.reply_to_mode = 'email'
            else:
                mailing.reply_to_mode = 'thread'

    @api.depends('reply_to_mode')
    def _compute_reply_to(self):
        for mailing in self:
            if mailing.reply_to_mode == 'email' and not mailing.reply_to:
                mailing.reply_to = self.env.user.email_formatted
            elif mailing.reply_to_mode == 'thread':
                mailing.reply_to = False

    @api.depends('mailing_model_id', 'contact_list_ids', 'mailing_type')
    def _compute_mailing_domain(self):
        for mailing in self:
            if not mailing.mailing_model_id:
                mailing.mailing_domain = ''
            else:
                mailing.mailing_domain = repr(mailing._get_default_mailing_domain())

    # ------------------------------------------------------
    # ORM
    # ------------------------------------------------------

    @api.model
    def create(self, values):
        if values.get('subject') and not values.get('name'):
            values['name'] = "%s %s" % (values['subject'], datetime.strftime(fields.datetime.now(), tools.DEFAULT_SERVER_DATETIME_FORMAT))
        if values.get('body_html'):
            values['body_html'] = self._convert_inline_images_to_urls(values['body_html'])
        return super().create(values)\
            ._fix_attachment_ownership()

    def write(self, values):
        if values.get('body_html'):
            values['body_html'] = self._convert_inline_images_to_urls(values['body_html'])
        super().write(values)
        self._fix_attachment_ownership()
        return True

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
            return {
                'type': 'ir.actions.act_window',
                'view_mode': 'form',
                'res_model': 'mailing.mailing',
                'res_id': mass_mailing_copy.id,
                'context': context,
            }
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

    def action_schedule(self):
        self.ensure_one()
        action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing.mailing_mailing_schedule_date_action")
        action['context'] = dict(self.env.context, default_mass_mailing_id=self.id)
        return action

    def action_put_in_queue(self):
        self.write({'state': 'in_queue'})

    def action_cancel(self):
        self.write({'state': 'draft', 'schedule_date': False, 'next_departure': False})

    def action_retry_failed(self):
        failed_mails = self.env['mail.mail'].sudo().search([
            ('mailing_id', 'in', self.ids),
            ('state', '=', 'exception')
        ])
        failed_mails.mapped('mailing_trace_ids').unlink()
        failed_mails.unlink()
        self.write({'state': 'in_queue'})

    def action_view_traces_scheduled(self):
        return self._action_view_traces_filtered('scheduled')

    def action_view_traces_ignored(self):
        return self._action_view_traces_filtered('ignored')

    def action_view_traces_failed(self):
        return self._action_view_traces_filtered('failed')

    def action_view_traces_sent(self):
        return self._action_view_traces_filtered('sent')

    def _action_view_traces_filtered(self, view_filter):
        action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing.mailing_trace_action")
        action['name'] = _('%s Traces') % (self.name)
        action['context'] = {'search_default_mass_mailing_id': self.id,}
        filter_key = 'search_default_filter_%s' % (view_filter)
        action['context'][filter_key] = True
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
        return self._action_view_documents_filtered('opened')

    def action_view_replied(self):
        return self._action_view_documents_filtered('replied')

    def action_view_bounced(self):
        return self._action_view_documents_filtered('bounced')

    def action_view_delivered(self):
        return self._action_view_documents_filtered('delivered')

    def _action_view_documents_filtered(self, view_filter):
        if view_filter in ('opened', 'replied', 'bounced'):
            opened_stats = self.mailing_trace_ids.filtered(lambda stat: stat[view_filter])
        elif view_filter == ('delivered'):
            opened_stats = self.mailing_trace_ids.filtered(lambda stat: stat.sent and not stat.bounced)
        else:
            opened_stats = self.env['mailing.trace']
        res_ids = opened_stats.mapped('res_id')
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
    # Email Sending
    # ------------------------------------------------------

    def _get_opt_out_list(self):
        """Returns a set of emails opted-out in target model"""
        self.ensure_one()
        opt_out = {}
        target = self.env[self.mailing_model_real]
        if self.mailing_model_real == "mailing.contact":
            # if user is opt_out on One list but not on another
            # or if two user with same email address, one opted in and the other one opted out, send the mail anyway
            # TODO DBE Fixme : Optimise the following to get real opt_out and opt_in
            target_list_contacts = self.env['mailing.contact.subscription'].search(
                [('list_id', 'in', self.contact_list_ids.ids)])
            opt_out_contacts = target_list_contacts.filtered(lambda rel: rel.opt_out).mapped('contact_id.email_normalized')
            opt_in_contacts = target_list_contacts.filtered(lambda rel: not rel.opt_out).mapped('contact_id.email_normalized')
            opt_out = set(c for c in opt_out_contacts if c not in opt_in_contacts)

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
             WHERE s.email IS NOT NULL
        """

        if self.unique_ab_testing:
            query += """
               AND s.campaign_id = %%(mailing_campaign_id)s;
            """
        else:
            query += """
               AND s.mass_mailing_id = %%(mailing_id)s
               AND s.model = %%(target_model)s;
            """
        query = query % {'target': target._table}
        params = {'mailing_campaign_id': self.campaign_id.id, 'mailing_id': self.id, 'target_model': self.mailing_model_real}
        self._cr.execute(query, params)
        seen_list = set(m[0] for m in self._cr.fetchall())
        _logger.info(
            "Mass-mailing %s has already reached %s %s emails", self, len(seen_list), target._name)
        return seen_list

    def _get_mass_mailing_context(self):
        """Returns extra context items with pre-filled blacklist and seen list for massmailing"""
        return {
            'mass_mailing_opt_out_list': self._get_opt_out_list(),
            'mass_mailing_seen_list': self._get_seen_list(),
            'post_convert_links': self._get_link_tracker_values(),
        }

    def _get_recipients(self):
        mailing_domain = self._parse_mailing_domain()
        res_ids = self.env[self.mailing_model_real].search(mailing_domain).ids

        # randomly choose a fragment
        if self.contact_ab_pc < 100:
            contact_nbr = self.env[self.mailing_model_real].search_count(mailing_domain)
            topick = int(contact_nbr / 100.0 * self.contact_ab_pc)
            if self.campaign_id and self.unique_ab_testing:
                already_mailed = self.campaign_id._get_mailing_recipients()[self.campaign_id.id]
            else:
                already_mailed = set([])
            remaining = set(res_ids).difference(already_mailed)
            if topick > len(remaining):
                topick = len(remaining)
            res_ids = random.sample(remaining, topick)
        return res_ids

    def _get_remaining_recipients(self):
        res_ids = self._get_recipients()
        already_mailed = self.env['mailing.trace'].search_read([
            ('model', '=', self.mailing_model_real),
            ('res_id', 'in', res_ids),
            ('mass_mailing_id', '=', self.id)], ['res_id'])
        done_res_ids = {record['res_id'] for record in already_mailed}
        return [rid for rid in res_ids if rid not in done_res_ids]

    def _get_unsubscribe_url(self, email_to, res_id):
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        url = werkzeug.urls.url_join(
            base_url, 'mail/mailing/%(mailing_id)s/unsubscribe?%(params)s' % {
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
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        url = werkzeug.urls.url_join(
            base_url, 'mailing/%(mailing_id)s/view?%(params)s' % {
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
                'no_auto_thread': mailing.reply_to_mode != 'thread',
                'template_id': None,
                'mail_server_id': mailing.mail_server_id.id,
            }
            if mailing.reply_to_mode == 'email':
                composer_values['reply_to'] = mailing.reply_to

            composer = self.env['mail.compose.message'].with_context(active_ids=res_ids).create(composer_values)
            extra_context = mailing._get_mass_mailing_context()
            composer = composer.with_context(active_ids=res_ids, **extra_context)
            # auto-commit except in testing mode
            auto_commit = not getattr(threading.currentThread(), 'testing', False)
            composer.send_mail(auto_commit=auto_commit)
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

        for mailing in self:
            user = mailing.user_id
            mailing = mailing.with_context(lang=user.lang or self._context.get('lang'))

            link_trackers = self.env['link.tracker'].search(
                [('mass_mailing_id', '=', mailing.id)]
            ).sorted('count', reverse=True)
            link_trackers_body = self.env['ir.qweb']._render(
                'mass_mailing.mass_mailing_kpi_link_trackers',
                {'object': mailing, 'link_trackers': link_trackers},
            )

            rendered_body = self.env['ir.qweb']._render(
                'digest.digest_mail_main',
                {
                    'body': tools.html_sanitize(link_trackers_body),
                    'company': user.company_id,
                    'user': user,
                    'display_mobile_banner': True,
                    ** mailing._prepare_statistics_email_values()
                },
            )

            full_mail = self.env['mail.render.mixin']._render_encapsulate(
                'digest.digest_mail_layout',
                rendered_body,
            )

            mail_values = {
                'subject': _('24H Stats of mailing "%s"') % mailing.subject,
                'email_from': user.email_formatted,
                'email_to': user.email_formatted,
                'body_html': full_mail,
                'auto_delete': True,
            }
            mail = self.env['mail.mail'].sudo().create(mail_values)
            mail.send(raise_exception=False)

    def _prepare_statistics_email_values(self):
        """Return some statistics that will be displayed in the mailing statistics email.

        Each item in the returned list will be displayed as a table, with a title and
        1, 2 or 3 columns.
        """
        self.ensure_one()

        random_tip = self.env['digest.tip'].search(
            [('group_id.category_id', '=', self.env.ref('base.module_category_marketing_email_marketing').id)]
        )
        if random_tip:
            random_tip = random.choice(random_tip).tip_description

        formatted_date = tools.format_datetime(
            self.env, self.sent_date, self.user_id.tz, 'MMM dd, YYYY',  self.user_id.lang
        ) if self.sent_date else False

        web_base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')

        return {
            'title': _('24H Stats of mailing'),
            'sub_title': '"%s"' % self.subject,
            'top_button_label': _('More Info'),
            'top_button_url': url_join(web_base_url, f'/web#id={self.id}&model=mailing.mailing&view_type=form'),
            'kpi_data': [
                {
                    'kpi_fullname': _('Engagement on %i Emails Sent') % self.sent,
                    'kpi_action': None,
                    'kpi_col1': {
                        'value': f'{self.received_ratio}%',
                        'col_subtitle': '%s (%i)' % (_('RECEIVED'), self.delivered),
                    },
                    'kpi_col2': {
                        'value': f'{self.opened_ratio}%',
                        'col_subtitle': '%s (%i)' % (_('OPENED'), self.opened),
                    },
                    'kpi_col3': {
                        'value': f'{self.replied_ratio}%',
                        'col_subtitle': '%s (%i)' % (_('REPLIED'), self.replied),
                    },
                }, {
                    'kpi_fullname': _('Business Benefits on %i Emails Sent') % self.sent,
                    'kpi_action': None,
                    'kpi_col1': {},
                    'kpi_col2': {},
                    'kpi_col3': {},
                },
            ],
            'tips': [random_tip] if random_tip else False,
            'formatted_date': formatted_date,
        }

    # ------------------------------------------------------
    # TOOLS
    # ------------------------------------------------------

    def _get_default_mailing_domain(self):
        mailing_domain = []
        if self.mailing_model_name == 'mailing.list' and self.contact_list_ids:
            mailing_domain = [('list_ids', 'in', self.contact_list_ids.ids)]

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

        def _image_to_url(b64image: bytes):
            """Store an image in an attachement and returns an url"""
            attachment = self.env['ir.attachment'].create({
                'datas': b64image,
                'name': "cropped_image_mailing_{}".format(self.id),
                'type': 'binary',})

            attachment.generate_access_token()

            return '/web/image/%s?access_token=%s' % (
                attachment.id, attachment.access_token)

        modified = False
        root = lxml.html.fromstring(body_html)
        for node in root.iter('img'):
            match = image_re.match(node.attrib.get('src', ''))
            if match:
                mime = match.group(1)  # unsed
                image = match.group(2).encode()  # base64 image as bytes

                node.attrib['src'] = _image_to_url(image)
                modified = True

        if modified:
            return lxml.html.tostring(root)

        return body_html

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

    @api.model
    def create(self, vals):
        if 'opt_out' in vals:
            vals['unsubscription_date'] = vals['opt_out'] and fields.Datetime.now()
        return super(MassMailingContactListRel, self).create(vals)

    def write(self, vals):
        if 'opt_out' in vals:
            vals['unsubscription_date'] = vals['opt_out'] and fields.Datetime.now()
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

    name = fields.Char(string='Mailing List', required=True)
    active = fields.Boolean(default=True)
    contact_nbr = fields.Integer(compute="_compute_contact_nbr", string='Number of Contacts')
    contact_ids = fields.Many2many(
        'mailing.contact', 'mailing_contact_list_rel', 'list_id', 'contact_id',
        string='Mailing Lists')
    subscription_ids = fields.One2many(
        'mailing.contact.subscription', 'list_id', string='Subscription Information',
        depends=['contact_ids'])
    is_public = fields.Boolean(default=True, help="The mailing list can be accessible by recipient in the unsubscription"
                                                  " page to allows him to update his subscription preferences.")

    # Compute number of contacts non opt-out, non blacklisted and valid email recipient for a mailing list
    def _compute_contact_nbr(self):
        if self.ids:
            self.env.cr.execute('''
                select
                    list_id, count(*)
                from
                    mailing_contact_list_rel r
                    left join mailing_contact c on (r.contact_id=c.id)
                    left join mail_blacklist bl on c.email_normalized = bl.email and bl.active
                where
                    list_id in %s
                    AND COALESCE(r.opt_out,FALSE) = FALSE
                    AND c.email_normalized IS NOT NULL
                    AND bl.id IS NULL
                group by
                    list_id
            ''', (tuple(self.ids), ))
            data = dict(self.env.cr.fetchall())
            for mailing_list in self:
                mailing_list.contact_nbr = data.get(mailing_list._origin.id, 0)
        else:
            self.contact_nbr = 0

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
        return [(list.id, "%s (%s)" % (list.name, list.contact_nbr)) for list in self]

    def action_view_contacts(self):
        action = self.env["ir.actions.actions"]._for_xml_id("mass_mailing.action_view_mass_mailing_contacts")
        action['domain'] = [('list_ids', 'in', self.ids)]
        context = dict(self.env.context, search_default_filter_valid_email_recipient=1, default_list_ids=self.ids)
        action['context'] = context
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
    without loosing the statistics about them. """
    _name = 'mailing.trace'
    _description = 'Mailing Statistics'
    _rec_name = 'id'
    _order = 'scheduled DESC'

    trace_type = fields.Selection([('mail', 'Mail')], string='Type', default='mail', required=True)
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
    message_id = fields.Char(string='Message-ID')
    # document
    model = fields.Char(string='Document model')
    res_id = fields.Integer(string='Document ID')
    # campaign / wave data
    mass_mailing_id = fields.Many2one('mailing.mailing', string='Mailing', index=True, ondelete='cascade')
    campaign_id = fields.Many2one(
        related='mass_mailing_id.campaign_id',
        string='Campaign',
        store=True, readonly=True, index=True)
    # Bounce and tracking
    ignored = fields.Datetime(help='Date when the email has been invalidated. '
                                   'Invalid emails are blacklisted, opted-out or invalid email format')
    scheduled = fields.Datetime(help='Date when the email has been created', default=fields.Datetime.now)
    sent = fields.Datetime(help='Date when the email has been sent')
    exception = fields.Datetime(help='Date of technical error leading to the email not being sent')
    opened = fields.Datetime(help='Date when the email has been opened the first time')
    replied = fields.Datetime(help='Date when this email has been replied for the first time.')
    bounced = fields.Datetime(help='Date when this email has bounced.')
    # Link tracking
    links_click_ids = fields.One2many('link.tracker.click', 'mailing_trace_id', string='Links click')
    clicked = fields.Datetime(help='Date when customer clicked on at least one tracked link')
    # Status
    state = fields.Selection(compute="_compute_state",
                             selection=[('outgoing', 'Outgoing'),
                                        ('exception', 'Exception'),
                                        ('sent', 'Sent'),
                                        ('opened', 'Opened'),
                                        ('replied', 'Replied'),
                                        ('bounced', 'Bounced'),
                                        ('ignored', 'Ignored')], store=True)
    failure_type = fields.Selection(selection=[
        ("SMTP", "Connection failed (outgoing mail server problem)"),
        ("RECIPIENT", "Invalid email address"),
        ("BOUNCE", "Email address rejected by destination"),
        ("UNKNOWN", "Unknown error"),
    ], string='Failure type')
    state_update = fields.Datetime(compute="_compute_state", string='State Update',
                                   help='Last state update of the mail',
                                   store=True)

    @api.depends('trace_type', 'mass_mailing_id')
    def _compute_display_name(self):
        for trace in self:
            trace.display_name = '%s: %s (%s)' % (trace.trace_type, trace.mass_mailing_id.name, trace.id)

    @api.depends('sent', 'opened', 'clicked', 'replied', 'bounced', 'exception', 'ignored')
    def _compute_state(self):
        self.update({'state_update': fields.Datetime.now()})
        for stat in self:
            if stat.ignored:
                stat.state = 'ignored'
            elif stat.exception:
                stat.state = 'exception'
            elif stat.replied:
                stat.state = 'replied'
            elif stat.opened or stat.clicked:
                stat.state = 'opened'
            elif stat.bounced:
                stat.state = 'bounced'
            elif stat.sent:
                stat.state = 'sent'
            else:
                stat.state = 'outgoing'

    @api.model_create_multi
    def create(self, values_list):
        for values in values_list:
            if 'mail_mail_id' in values:
                values['mail_mail_id_int'] = values['mail_mail_id']
        return super(MailingTrace, self).create(values_list)

    def _get_records(self, mail_mail_ids=None, mail_message_ids=None, domain=None):
        if not self.ids and mail_mail_ids:
            base_domain = [('mail_mail_id_int', 'in', mail_mail_ids)]
        elif not self.ids and mail_message_ids:
            base_domain = [('message_id', 'in', mail_message_ids)]
        else:
            base_domain = [('id', 'in', self.ids)]
        if domain:
            base_domain = ['&'] + domain + base_domain
        return self.search(base_domain)

    def set_opened(self, mail_mail_ids=None, mail_message_ids=None):
        traces = self._get_records(mail_mail_ids, mail_message_ids, [('opened', '=', False)])
        traces.write({'opened': fields.Datetime.now(), 'bounced': False})
        return traces

    def set_clicked(self, mail_mail_ids=None, mail_message_ids=None):
        traces = self._get_records(mail_mail_ids, mail_message_ids, [('clicked', '=', False)])
        traces.write({'clicked': fields.Datetime.now()})
        return traces

    def set_replied(self, mail_mail_ids=None, mail_message_ids=None):
        traces = self._get_records(mail_mail_ids, mail_message_ids, [('replied', '=', False)])
        traces.write({'replied': fields.Datetime.now()})
        return traces

    def set_bounced(self, mail_mail_ids=None, mail_message_ids=None):
        traces = self._get_records(mail_mail_ids, mail_message_ids, [('bounced', '=', False), ('opened', '=', False)])
        traces.write({'bounced': fields.Datetime.now()})
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
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        token = tools.hmac(self.env(su=True), 'mass_mailing-mail_mail-open', self.id)
        return werkzeug.urls.url_join(base_url, 'mail/track/%s/%s/blank.gif' % (self.id, token))

    def _send_prepare_body(self):
        """ Override to add the tracking URL to the body and to add
        trace ID in shortened urls """
        # TDE: temporary addition (mail was parameter) due to semi-new-API
        self.ensure_one()
        body = super(MailMail, self)._send_prepare_body()

        if self.mailing_id and body and self.mailing_trace_ids:
            for match in re.findall(tools.URL_REGEX, self.body_html):
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
        base_url = self.env['ir.config_parameter'].sudo().get_param('web.base.url').rstrip('/')
        if self.mailing_id and res.get('body') and res.get('email_to'):
            emails = tools.email_split(res.get('email_to')[0])
            email_to = emails and emails[0] or False

            urls_to_replace = [
               (base_url + '/unsubscribe_from_list', self.mailing_id._get_unsubscribe_url(email_to, self.res_id)),
               (base_url + '/view', self.mailing_id._get_view_url(email_to, self.res_id))
            ]

            for url_to_replace, new_url in urls_to_replace:
                if url_to_replace in res['body']:
                    res['body'] = res['body'].replace(url_to_replace, new_url if new_url else '#')
        return res

    def _postprocess_sent_message(self, success_pids, failure_reason=False, failure_type=None):
        mail_sent = not failure_type  # we consider that a recipient error is a failure with mass mailling and show them as failed
        for mail in self:
            if mail.mailing_id:
                if mail_sent is True and mail.mailing_trace_ids:
                    mail.mailing_trace_ids.write({'sent': fields.Datetime.now(), 'exception': False})
                elif mail_sent is False and mail.mailing_trace_ids:
                    mail.mailing_trace_ids.write({'exception': fields.Datetime.now(), 'failure_type': failure_type})
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
                self.env['mailing.trace'].set_opened(mail_message_ids=msg_references)
                self.env['mailing.trace'].set_replied(mail_message_ids=msg_references)
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
            self.env['mailing.trace'].set_bounced(mail_message_ids=bounced_msg_id)
        if bounced_email:
            three_months_ago = fields.Datetime.to_string(datetime.datetime.now() - datetime.timedelta(weeks=13))
            stats = self.env['mailing.trace'].search(['&', ('bounced', '>', three_months_ago), ('email', '=ilike', bounced_email)]).mapped('bounced')
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

from odoo import api, fields, models


class UtmCampaign(models.Model):
    _inherit = 'utm.campaign'

    mailing_mail_ids = fields.One2many(
        'mailing.mailing', 'campaign_id',
        domain=[('mailing_type', '=', 'mail')],
        string='Mass Mailings')
    mailing_mail_count = fields.Integer('Number of Mass Mailing', compute="_compute_mailing_mail_count")
    # stat fields
    received_ratio = fields.Integer(compute="_compute_statistics", string='Received Ratio')
    opened_ratio = fields.Integer(compute="_compute_statistics", string='Opened Ratio')
    replied_ratio = fields.Integer(compute="_compute_statistics", string='Replied Ratio')
    bounced_ratio = fields.Integer(compute="_compute_statistics", string='Bounced Ratio')

    @api.depends('mailing_mail_ids')
    def _compute_mailing_mail_count(self):
        if self.ids:
            mailing_data = self.env['mailing.mailing'].read_group(
                [('campaign_id', 'in', self.ids), ('mailing_type', '=', 'mail')],
                ['campaign_id'],
                ['campaign_id']
            )
            mapped_data = {m['campaign_id'][0]: m['campaign_id_count'] for m in mailing_data}
        else:
            mapped_data = dict()
        for campaign in self:
            campaign.mailing_mail_count = mapped_data.get(campaign.id, 0)

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
                COUNT(CASE WHEN s.sent is not null THEN 1 ELSE null END) AS sent,
                COUNT(CASE WHEN s.scheduled is not null AND s.sent is null AND s.exception is null AND s.ignored is not null THEN 1 ELSE null END) AS ignored,
                COUNT(CASE WHEN s.id is not null AND s.bounced is null THEN 1 ELSE null END) AS delivered,
                COUNT(CASE WHEN s.opened is not null THEN 1 ELSE null END) AS opened,
                COUNT(CASE WHEN s.replied is not null THEN 1 ELSE null END) AS replied,
                COUNT(CASE WHEN s.bounced is not null THEN 1 ELSE null END) AS bounced
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
                total = (stats['expected'] - stats['ignored']) or 1
                delivered = stats['sent'] - stats['bounced']
                vals = {
                    'received_ratio': 100.0 * delivered / total,
                    'opened_ratio': 100.0 * stats['opened'] / total,
                    'replied_ratio': 100.0 * stats['replied'] / total,
                    'bounced_ratio': 100.0 * stats['bounced'] / total
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

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import link_tracker
from . import mailing_contact
from . import mailing_list
from . import mailing_trace
from . import mailing
from . import mail_mail
from . import mail_render_mixin
from . import mail_thread
from . import res_config_settings
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
    sent = fields.Integer(readonly=True)
    delivered = fields.Integer(readonly=True)
    opened = fields.Integer(readonly=True)
    replied = fields.Integer(readonly=True)
    clicked = fields.Integer(readonly=True)
    bounced = fields.Integer(readonly=True)

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
            'trace.scheduled as scheduled_date',
            'mailing.state',
            'mailing.email_from',
            'count(trace.sent) as sent',
            '(count(trace.sent) - count(trace.bounced)) as delivered',
            'count(trace.opened) as opened',
            'count(trace.replied) as replied',
            'count(trace.clicked) as clicked',
            'count(trace.bounced) as bounced'
        ]

    def _report_get_request_from_items(self):
        return [
            'mailing_trace as trace',
            'left join mailing_mailing as mailing ON (trace.mass_mailing_id=mailing.id)',
            'left join utm_campaign as utm_campaign ON (mailing.campaign_id = utm_campaign.id)',
            'left join utm_source as utm_source ON (mailing.source_id = utm_source.id)'
        ]

    def _report_get_request_where_items(self):
        return []

    def _report_get_request_group_by_items(self):
        return [
            'trace.scheduled',
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
        <record id="mailing_trace_report_view_pivot" model="ir.ui.view">
            <field name="name">mailing.trace.report.view.pivot</field>
            <field name="model">mailing.trace.report</field>
            <field name="arch" type="xml">
                <pivot string="Mass Mailing Statistics" disable_linking="True" sample="1">
                    <field name="name" type="row"/>
                    <field name="sent" type="measure"/>
                    <field name="delivered" type="measure"/>
                    <field name="opened" type="measure"/>
                    <field name="bounced" type="measure"/>
                    <field name="replied" type="measure" invisible="0"/>
                    <field name="clicked" type="measure"/>
                </pivot>
            </field>
        </record>

        <record id="mailing_trace_report_view_graph" model="ir.ui.view">
            <field name="name">mailing.trace.report.view.graph</field>
            <field name="model">mailing.trace.report</field>
            <field name="arch" type="xml">
                <graph string="Mass Mailing Statistics" sample="1" disable_linking="1">
                    <field name="name"/>
                    <field name="sent" type="measure"/>
                    <field name="replied" invisible="0"/>
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
           <field name="view_mode">graph,pivot</field>
           <field name="help" type="html"><p>Mass Mailing Statistics allows you to check different mailing related information like number of bounced mails, opened mails, replied mails. You can sort out your analysis by different groups to get accurate grained analysis.</p></field>
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
access_mailing_trace_user,mailing.trace.user,model_mailing_trace,base.group_user,1,1,1,1
access_mailing_trace_mm_user,access.mailing.trace.mm.user,model_mailing_trace,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_trace_report_mm_user,access.mailing.trace.report.mm.user,model_mailing_trace_report,mass_mailing.group_mass_mailing_user,1,1,1,1
access_utm_source,access_utm_source,utm.model_utm_source,mass_mailing.group_mass_mailing_user,1,1,1,0
access_ir_mail_server,access_ir_mail_server,base.model_ir_mail_server,mass_mailing.group_mass_mailing_user,1,0,0,0
access_mail_blacklist_mass_mailing_user,access.mail.blacklist.mass_mailing_user,mail.model_mail_blacklist,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mail_blacklist_remove_mass_mailing_user,acesss.mail.blacklist.remove.mass_mailing_user,mail.model_mail_blacklist_remove,mass_mailing.group_mass_mailing_user,1,1,1,1
access_link_tracker_mailing,access.link.tracker.mailing,link_tracker.model_link_tracker,mass_mailing.group_mass_mailing_user,1,1,1,1
access_mailing_list_merge,access.mailing.list.merge,model_mailing_list_merge,mass_mailing.group_mass_mailing_user,1,1,1,0
access_mailing_mailing_schedule_date,access.mailing.mailing.schedule.date,model_mailing_mailing_schedule_date,mass_mailing.group_mass_mailing_user,1,1,1,0
access_mailing_mailing_test,access.mailing.mailing.test,model_mailing_mailing_test,mass_mailing.group_mass_mailing_user,1,1,1,0

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
    </data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="98.616%"><stop offset="0%" stop-color="#797C79"/><stop offset="100%" stop-color="#545554"/></linearGradient><path id="d" d="M16.995 35.708c-2.743 1.538-2.48 5.47.455 6.638L26.564 46c.611 7.72 1.423 11.72 2.436 12 .948.262 3.602-2.35 7.962-7.835l8.923 3.57c.47.185.965.278 1.46.278.653 0 1.299-.163 1.881-.48a3.736 3.736 0 0 0 1.906-2.665C54.969 28.136 56.548 16.513 55.868 16c-.77-.582-13.728 5.987-38.873 19.708zm13.397 16.813v-4.99l2.918 1.166-2.918 3.824zm16.952-2.217L35.08 45.398l11.18-15.631c.853-1.198-.758-2.589-1.89-1.638l-16.865 14.24-8.596-3.446 33.172-18.544-4.737 29.925z"/><path id="e" d="M16.995 33.708c-2.743 1.538-2.48 5.47.455 6.638L26.564 44c.611 7.72 1.423 11.72 2.436 12 .948.262 3.602-2.35 7.962-7.835l8.923 3.57c.47.185.965.278 1.46.278.653 0 1.299-.163 1.881-.48a3.736 3.736 0 0 0 1.906-2.665C54.969 26.136 56.548 14.513 55.868 14c-.77-.582-13.728 5.987-38.873 19.708zm13.397 16.813v-4.99l2.918 1.166-2.918 3.824zm16.952-2.217L35.08 43.398l11.18-15.631c.853-1.198-.758-2.589-1.89-1.638l-16.865 14.24-8.596-3.446 33.172-18.544-4.737 29.925z"/></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M40.1 69H4c-2 0-4-.146-4-4.078V47.679L15.8 35 54 17l-3.062 32.732L40.1 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#d"/><use fill="#FFF" fill-rule="nonzero" xlink:href="#e"/></g></g></svg>
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

## File: static\src\js\mass_mailing_list_kanban_record.js

```javascript
odoo.define('mass_mailing.ListKanbanRecord', function (require) {
"use strict";

var KanbanRecord = require('web.KanbanRecord');

var MassMailingListKanbanRecord = KanbanRecord.extend({
    /**
     * @override
     * @private
     */
    _openRecord: function () {
        this.$('.o_mailing_list_kanban_boxes a').first().click();
    }
});

return MassMailingListKanbanRecord;

});

```

## File: static\src\js\mass_mailing_list_kanban_renderer.js

```javascript
odoo.define('mass_mailing.ListKanbanRenderer', function (require) {
"use strict";

var MassMailingListKanbanRecord = require('mass_mailing.ListKanbanRecord');

var KanbanRenderer = require('web.KanbanRenderer');

var MassMailingListKanbanRenderer = KanbanRenderer.extend({
    config: _.extend({}, KanbanRenderer.prototype.config, {
        KanbanRecord: MassMailingListKanbanRecord,
    })
});

return MassMailingListKanbanRenderer;

});

```

## File: static\src\js\mass_mailing_list_kanban_view.js

```javascript
odoo.define('mass_mailing.ListKanbanView', function (require) {
"use strict";

var MassMailingListKanbanRenderer = require('mass_mailing.ListKanbanRenderer');

var KanbanView = require('web.KanbanView');
var view_registry = require('web.view_registry');

var MassMailingListKanbanView = KanbanView.extend({
    config: _.extend({}, KanbanView.prototype.config, {
        Renderer: MassMailingListKanbanRenderer,
    }),
});

view_registry.add('mass_mailing_list_kanban', MassMailingListKanbanView);

return MassMailingListKanbanView;

});

```

## File: static\src\js\mass_mailing_snippets.js

```javascript
odoo.define('mass_mailing.snippets.options', function (require) {
"use strict";

var options = require('web_editor.snippets.options');

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
                    self.change_width(event, self.$target, target_width, offset, true);
                }
            });
            $body.one("mouseup", function () {
                $body.off('.mass_mailing_width_x');
            });
        });

        return def;
    },
    change_width: function (event, target, target_width, offset, grow) {
        target.css("width", grow ? (event.pageX - offset) : (offset + target_width - event.pageX));
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

options.registry.mass_mailing_table_item = options.Class.extend({
    onClone: function (options) {
        this._super.apply(this, arguments);

        // If we cloned a td or th element...
        if (options.isCurrent && this.$target.is("td, th")) {
            // ... and that the td or th element was alone on its row ...
            if (this.$target.siblings().length === 1) {
                var $tr = this.$target.parent();
                $tr.clone().empty().insertAfter($tr).append(this.$target); // ... move the clone in a new row instead
                return;
            }

            // ... if not, if the clone neighbor is an empty cell, remove this empty cell (like if the clone content had been put in that cell)
            var $next = this.$target.next();
            if ($next.length && $next.text().trim() === "") {
                $next.remove();
                return;
            }

            // ... if not, insert an empty col in each other row, at the index of the clone
            var width = this.$target.width();
            var $trs = this.$target.closest("table").children("thead, tbody, tfoot").addBack().children("tr").not(this.$target.parent());
            _.each($trs.children(":nth-child(" + this.$target.index() + ")"), function (col) {
                $(col).after($("<td/>", {style: "width: " + width + "px;"}));
            });
        }
    },
    onRemove: function () {
        this._super.apply(this, arguments);

        // If we are removing a td or th element which was not alone on its row ...
        if (this.$target.is("td, th") && this.$target.siblings().length > 0) {
            var $trs = this.$target.closest("table").children("thead, tbody, tfoot").addBack().children("tr").not(this.$target.parent());
            if ($trs.length) { // ... if there are other rows in the table ...
                var $last_tds = $trs.children(":last-child");
                if (_.reduce($last_tds, function (memo, td) { return memo + (td.innerHTML || ""); }, "").trim() === "") {
                    $last_tds.remove(); // ... remove the potential full empty column in the table
                } else {
                    this.$target.parent().append("<td/>"); // ... else, if there is no full empty column, append an empty col in the current row
                }
            }
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

// TODO remove in master when removing the XML div. The option has been disabled
// in 14.0 because of tricky problems to resolve that require refactoring:
// the ability to clean snippet without saving and reloading the page.
options.registry.SnippetSave.include({

    async saveSnippet(previewMode, widgetValue, params) {},

    async _computeVisibility() {
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
    ],

    custom_events: _.extend({}, FieldHtml.prototype.custom_events, {
        snippets_loaded: '_onSnippetsLoaded',
    }),

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        if (!this.nodeOptions.snippets) {
            this.nodeOptions.snippets = 'mass_mailing.email_designer_snippets';
        }

        // All the code related to this __extraAssetsForIframe variable is an
        // ugly hack to restore mass mailing options in stable versions. The
        // whole logic has to be refactored as soon as possible...
        this.__extraAssetsForIframe = [{
            jsLibs: ['/mass_mailing/static/src/js/mass_mailing_snippets.js'],
        }];
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
    commitChanges: function () {
        var self = this;
        if (config.isDebug() && this.mode === 'edit') {
            var layoutInfo = $.summernote.core.dom.makeLayoutInfo(this.wysiwyg.$editor);
            $.summernote.pluginEvents.codeview(undefined, undefined, layoutInfo, false);
        }
        if (this.mode === 'readonly' || !this.isRendered) {
            return this._super();
        }
        var fieldName = this.nodeOptions['inline-field'];

        if (this.$content.find('.o_basic_theme').length) {
            this.$content.find('*').css('font-family', '');
        }

        var $editable = this.wysiwyg.getEditable();

        return this.wysiwyg.saveModifiedImages(this.$content).then(function () {
            return self.wysiwyg.save().then(function (result) {
                self._isDirty = result.isDirty;

                convertInline.attachmentThumbnailToLinkImg($editable);
                convertInline.fontToImg($editable);
                convertInline.classToStyle($editable);

                // fix outlook image rendering bug
                _.each(['width', 'height'], function(attribute) {
                    $editable.find('img[style*="width"], img[style*="height"]').attr(attribute, function(){
                        return $(this)[attribute]();
                    }).css(attribute, function(){
                        return $(this).get(0).style[attribute] || 'auto';
                    });
                });

                self.trigger_up('field_changed', {
                    dataPointID: self.dataPointID,
                    changes: _.object([fieldName], [self._unWrap($editable.html())])
                });
                self.wysiwyg.setValue(result.html);

                if (self._isDirty && self.mode === 'edit') {
                    return self._doAction();
                }
            });
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
     * Returns true if must force the user to choose a theme.
     *
     * @private
     * @returns {Boolean}
     */
    _checkIfMustForceThemeChoice: function () {
        var firstChoice = this._editableAreaIsEmpty();
        this.$content.closest('body').toggleClass("o_force_mail_theme_choice", firstChoice);
        return firstChoice;
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
        this._isFromInline = !!this.value;
        if (!this.value) {
            this.value = this.recordData[this.nodeOptions['inline-field']];
        }
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    _renderReadonly: function () {
        this.value = this.recordData[this.nodeOptions['inline-field']];
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
                    'class': 'o_field_translate fa fa-globe btn btn-link',
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
    },
    /**
     * Switch themes or import first theme.
     *
     * @private
     * @param {Boolean} firstChoice true if this is the first chosen theme (going from no theme to a theme)
     * @param {Object} themeParams
     */
    _switchThemes: function (firstChoice, themeParams) {
        if (!themeParams || this.switchThemeLast === themeParams) {
            return;
        }
        this.switchThemeLast = themeParams;

        this.$content.closest('body').removeClass(this._allClasses).addClass(themeParams.className);

        var $old_layout = this.$content.find('.o_layout');

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
            $new_wrapper = $('<table/>', {
                class: 'o_mail_wrapper'
            });
            $newWrapperContent = $('<td/>', {
                class: 'o_mail_no_options o_mail_wrapper_td oe_structure'
            });
            $new_wrapper.append($('<tr/>').append(
                $('<td/>', {
                    class: 'o_mail_no_resize o_not_editable',
                    contenteditable: 'false'
                }),
                $newWrapperContent,
                $('<td/>', {
                    class: 'o_mail_no_resize o_not_editable',
                    contenteditable: 'false'
                })
            ));
        }
        var $newLayout = $('<div/>', {
            class: 'o_layout ' + themeParams.className
        }).append($new_wrapper);

        var $contents;
        if (firstChoice) {
            $contents = themeParams.template;
        } else if ($old_layout.length) {
            $contents = ($old_layout.hasClass('oe_structure') ? $old_layout : $old_layout.find('.oe_structure').first()).contents();
        } else {
            $contents = this.$content.find('.o_editable').contents();
        }

        $newWrapperContent.append($contents);
        this._switchImages(themeParams, $newWrapperContent);
        this.$content.find('.o_editable').empty().append($newLayout);
        $old_layout.remove();

        if (firstChoice) {
            $newWrapperContent.find('*').addBack()
                .contents()
                .filter(function () {
                    return this.nodeType === 3 && this.textContent.match(/\S/);
                }).parent().addClass('o_default_snippet_text');

            if (themeParams.name == 'basic') {
                this.$content.focusIn();
            }
        }
        this.wysiwyg.trigger('reload_snippet_dropzones');
    },

    //--------------------------------------------------------------------------
    // Handler
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _onLoadWysiwyg: function () {
        if (this._isFromInline) {
            this._fromInline();
        }
        if (this.snippetsLoaded) {
            this._onSnippetsLoaded(this.snippetsLoaded);
        }
        this._super();
    },
    /**
     * @private
     * @param {OdooEvent} ev
     */
    _onSnippetsLoaded: function (ev) {
        var self = this;
        if (!this.$content) {
            this.snippetsLoaded = ev;
            return;
        }
        var $snippetsSideBar = ev.data;
        var $themes = $snippetsSideBar.find("#email_designer_themes").children();
        var $snippets = $snippetsSideBar.find(".oe_snippet");
        var $snippets_menu = $snippetsSideBar.find("#snippets_menu");

        for (const button of $snippets_menu.get(0).children) {
            if (!button.hasAttribute('tabindex') && !button.hasAttribute('accesskey')) {
                button.style.display = 'none';
            }
        }

        if (config.device.isMobile) {
            $snippetsSideBar.hide();
            this.$content.attr('style', 'padding-left: 0px !important');
        }

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
        var $dropdown = $(core.qweb.render("mass_mailing.theme_selector", {
            themes: themesParams
        })).dropdown();

        var firstChoice = this._checkIfMustForceThemeChoice();

        /**
         * Add proposition to install enterprise themes if not installed.
         */
        var $mail_themes_upgrade = $dropdown.find(".o_mass_mailing_themes_upgrade");
        $mail_themes_upgrade.on("click", function (e) {
            e.stopImmediatePropagation();
            e.preventDefault();
            self.do_action("mass_mailing.action_mass_mailing_configuration");
        });

        /**
         * Switch theme when a theme button is hovered. Confirm change if the theme button
         * is pressed.
         */
        var selectedTheme = false;
        $dropdown.on("mouseenter", ".dropdown-item", function (e) {
            if (firstChoice) {
                return;
            }
            e.preventDefault();
            var themeParams = themesParams[$(e.currentTarget).index()];
            self._switchThemes(firstChoice, themeParams);
        });
        $dropdown.on("mouseleave", ".dropdown-item", function (e) {
            self._switchThemes(false, selectedTheme);
        });
        $dropdown.on("click", '[data-toggle="dropdown"]', function (e) {
            var $menu = $dropdown.find('.dropdown-menu');
            var isVisible = $menu.hasClass('show');
            if (isVisible) {
                e.preventDefault();
                e.stopImmediatePropagation();
                $menu.removeClass('show');
            }
        });

        $dropdown.on("click", ".dropdown-item", function (e) {
            e.preventDefault();
            e.stopImmediatePropagation();
            var themeParams = themesParams[$(e.currentTarget).index()];
            if (firstChoice) {
                self._switchThemes(firstChoice, themeParams);
                self.$content.closest('body').removeClass("o_force_mail_theme_choice");
                firstChoice = false;

                if ($mail_themes_upgrade.length) {
                    $dropdown.remove();
                    $snippets_menu.empty();
                }
            }

            self._switchImages(themeParams, $snippets);

            selectedTheme = themeParams;

            // Notify form view
            self.wysiwyg.getEditable().trigger('change');
            $dropdown.find('.dropdown-menu').removeClass('show');
            $dropdown.find('.dropdown-item.selected').removeClass('selected');
            $dropdown.find('.dropdown-item:eq(' + themesParams.indexOf(selectedTheme) + ')').addClass('selected');
        });

        // Prevent expansion of drop-down while clicking on empty area during theme selection
        $dropdown.on("click", ".dropdown-menu", function (ev) {
            ev.preventDefault();
            ev.stopPropagation();
        });

        /**
         * If the user opens the theme selection screen, indicates which one is active and
         * saves the information...
         * ... then when the user closes check if the user confirmed its choice and restore
         * previous state if this is not the case.
         */
        $dropdown.on("shown.bs.dropdown", function () {
            selectedTheme = self._getSelectedTheme(themesParams);
            $dropdown.find(".dropdown-item").removeClass("selected").filter(function () {
                return ($(this).has(".o_thumb[style=\"" + "background-image: url(" + (selectedTheme && selectedTheme.img) + "_small.png)" + "\"]").length > 0);
            }).addClass("selected");
        });
        $dropdown.on("hidden.bs.dropdown", function () {
            self._switchThemes(firstChoice, selectedTheme);
        });

        /**
         * On page load, check the selected theme and force switching to it (body needs the
         * theme style for its edition toolbar).
         */
        selectedTheme = this._getSelectedTheme(themesParams);
        if (selectedTheme) {
            this.$content.closest('body').addClass(selectedTheme.className);
            $dropdown.find('.dropdown-item:eq(' + themesParams.indexOf(selectedTheme) + ')').addClass('selected');
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

        $dropdown.insertAfter($snippets_menu);
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
        return Promise.reject("DOM doesn't contain '.o_unsubscribe_form'");
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
                        $('#subscription_info').html(_t('An error occured. Please try again later or contact us.'));
                        $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-warning').addClass('alert-error');
                    }
                })
                .guardedCatch(function () {
                    $('#subscription_info').html(_t('An error occured. Please try again later or contact us.'));
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
                    $('#subscription_info').html(_t('You are not authorized to do this!'));
                    $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-error').addClass('alert-warning');
                }
                else if (result == true) {
                    $('#subscription_info').html(_t('Your changes have been saved.'));
                    $('#info_state').removeClass('alert-info').addClass('alert-success');
                }
                else {
                    $('#subscription_info').html(_t('An error occurred. Your changes have not been saved, try again later.'));
                    $('#info_state').removeClass('alert-info').addClass('alert-warning');
                }
            })
            .guardedCatch(function () {
                $('#subscription_info').html(_t('An error occurred. Your changes have not been saved, try again later.'));
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
                    $('#subscription_info').html(_t('You are not authorized to do this!'));
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
                        $('#subscription_info').html(_t('An error occured. Please try again later or contact us.'));
                        $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-warning').addClass('alert-error');
                    }
                    $('#button_add_blacklist').hide();
                    $('#button_remove_blacklist').show();
                    $('#unsubscribed_info').hide();
                }
            })
            .guardedCatch(function () {
                $('#subscription_info').html(_t('An error occured. Please try again later or contact us.'));
                $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-warning').addClass('alert-error');
            });
    });

    $('#button_remove_blacklist').click(function (e) {
        e.preventDefault();

        ajax.jsonRpc('/mailing/blacklist/remove', 'call', {'email': email, 'mailing_id': mailing_id, 'res_id': res_id, 'token': token})
            .then(function (result) {
                if (result == 'unauthorized'){
                    $('#subscription_info').html(_t('You are not authorized to do this!'));
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
                        $('#subscription_info').html(_t('An error occured. Please try again later or contact us.'));
                        $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-warning').addClass('alert-error');
                    }
                    $('#button_add_blacklist').show();
                    $('#button_remove_blacklist').hide();
                    $('#unsubscribed_info').hide();
                }
            })
            .guardedCatch(function () {
                $('#subscription_info').html(_t('An error occured. Please try again later or contact us.'));
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
                    $('#subscription_info').html(_t('You are not authorized to do this!'));
                    $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-error').addClass('alert-warning');
                }
                else if (result == true){
                    $('#subscription_info').html(_t('Thank you! Your feedback has been sent successfully!'));
                    $('#info_state').removeClass('alert-warning').removeClass('alert-info').removeClass('alert-error').addClass('alert-success');
                    $("#div_feedback").hide();
                }
                else {
                    $('#subscription_info').html(_t('An error occured. Please try again later or contact us.'));
                    $('#info_state').removeClass('alert-success').removeClass('alert-info').removeClass('alert-error').addClass('alert-warning');
                }
            })
            .guardedCatch(function () {
                $('#subscription_info').html(_t('An error occured. Please try again later or contact us.'));
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

## File: static\src\js\tours\mass_mailing_tour.js

```javascript
odoo.define('mass_mailing.mass_mailing_tour', function (require) {
    "use strict";

    var core = require('web.core');
    var _t = core._t;
    var tour = require('web_tour.tour');
    var now = moment();

    tour.register('mass_mailing_tour', {
        url: '/web',
        rainbowManMessage: _t('Congratulations, I love your first mailing. :)'),
        sequence: 200,
    }, [tour.stepUtils.showAppsMenuItem(), {
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
        content: _t("Let's try the Email Marketing app."),
        width: 210,
        position: 'bottom',
        edition: 'enterprise',
    }, {
        trigger: '.o_app[data-menu-xmlid="mass_mailing.mass_mailing_menu_root"]',
        content: _t("Let's try the Email Marketing app."),
        edition: 'community',
    }, {
        trigger: '.o-kanban-button-new',
        extra_trigger: '.oe_kanban_mass_mailing',
        content: _t("Start by creating your first <b>Mailing</b>."),
        position: 'bottom',
    }, {
        trigger: 'input[name="subject"]',
        content: _t('Pick the <b>email subject</b>.'),
        position: 'right',
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
        content: _t('Choose this <b>theme</b>.'),
        position: 'left',
        edition: 'enterprise',
        run: 'click',
    }, {
        trigger: 'div[name="body_arch"] iframe #default',
        content: _t('Choose this <b>theme</b>.'),
        position: 'right',
        edition: 'community',
        run: 'click',
    }, {
        trigger: 'div[name="body_arch"] iframe div.o_mail_block_paragraph',
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
        trigger: 'button[name="action_put_in_queue"]',
        content: _t("Ready for take-off!"),
        position: 'bottom',
    }, {
        trigger: '.btn-primary:contains("Ok")',
        content: _t("Don't worry, the mailing contact we created is an internal user."),
        position: 'bottom',
        run: "click",
    }, {
        trigger: '.o_back_button',
        content: _t("By using the <b>Breadcrumb</b>, you can navigate back to the overview."),
        position: 'bottom',
        run: 'click',
    }]
    );
});

```

## File: static\src\xml\mass_mailing.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <div t-name="mass_mailing.theme_selector" class="o_mail_theme_selector">
        <a role="button" href="#" class="btn btn-sm dropdown-toggle" data-toggle="dropdown">
            <i class="fa fa-paint-brush"/> Change Style
        </a>
        <div class="dropdown-menu" role="menu">
            <t t-foreach="themes" t-as="theme">
                <a t-att-id="theme.name" role="menuitem" href="#" class="dropdown-item">
                    <div class="o_thumb small"  t-attf-style="background-image: url(#{theme.img}_small.png)"/>
                    <div class="o_thumb large" t-attf-style="background-image: url(#{theme.img}_large.png)"/>
                    <div class="o_thumb logo" t-attf-style="background-image: url(#{theme.img}_logo.png)"/>
                </a>
            </t>
            <t t-if="themes.length === 1">
                <a role="menuitem" href="#" class="dropdown-item o_mass_mailing_themes_upgrade">
                    <div class="o_thumb"><i class="fa fa-plus" role="img" aria-label="Upgrade theme" title="Upgrade theme"/></div>
                </a>
            </t>
        </div>
    </div>
</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="mass_mailing assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/mass_mailing/static/src/scss/mass_mailing.scss"/>
            <link rel="stylesheet" type="text/scss" href="/mass_mailing/static/src/scss/mass_mailing_mobile.scss"/>
            <link rel="stylesheet" href="/mass_mailing/static/src/css/email_template.css"/>
            <link rel="stylesheet" type="text/scss" href="/mass_mailing/static/src/scss/mass_mailing.ui.jw.scss"/>

            <script type="text/javascript" src="/mass_mailing/static/src/js/mass_mailing.js"></script>
            <script type="text/javascript" src="/mass_mailing/static/src/js/mass_mailing_widget.js"></script>
        </xpath>
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/mass_mailing/static/src/js/mass_mailing_list_kanban_record.js"></script>
            <script type="text/javascript" src="/mass_mailing/static/src/js/mass_mailing_list_kanban_renderer.js"></script>
            <script type="text/javascript" src="/mass_mailing/static/src/js/mass_mailing_list_kanban_view.js"></script>
            <script type="text/javascript" src="/mass_mailing/static/src/js/unsubscribe.js"></script>
        </xpath>
    </template>

    <template id="assets_mail_themes">
        <link rel="stylesheet" type="text/scss" href="/mass_mailing/static/src/scss/themes/theme_basic.scss"/>
        <link rel="stylesheet" type="text/scss" href="/mass_mailing/static/src/scss/themes/theme_default.scss"/>
        <t t-call="mass_mailing.mass_mailing_mail_style"/>
    </template>

    <template id="assets_mail_themes_edition"> <!-- maybe to remove and convert into a field dumy with attr invisible if the template is not selected -->
        <t t-call="web._assets_helpers"/>
        <link rel="stylesheet" type="text/scss" href="/mass_mailing/static/src/scss/mass_mailing.ui.scss"/>
        <link rel="stylesheet" type="text/scss" href="/mass_mailing/static/src/scss/mass_mailing.ui.shadow.scss"/>
        <link rel="stylesheet" type="text/scss" href="/web/static/src/scss/webclient.scss"/>
    </template>

    <template id="iframe_css_assets_edit" groups="base.group_user">
        <t t-call-assets="web.assets_common" t-js="false"/>
        <t t-call-assets="web_editor.assets_wysiwyg" t-js="false"/>
        <t t-call-assets="mass_mailing.assets_mail_themes" t-js="false"/>
        <t t-call-assets="mass_mailing.assets_mail_themes_edition" t-js="false"/>
    </template>

    <template id="iframe_css_assets_readonly" groups="base.group_user">
        <link rel="stylesheet" type="text/scss" href="/mass_mailing/static/src/css/basic_theme_readonly.css"/>
    </template>

    <template id="assets_common" name="Mass Mailing Assets Common" inherit_id="web.assets_common">
        <xpath expr="//script[last()]" position="after">
            <script type="text/javscript" src="/mass_mailing/static/src/js/tours/mass_mailing_tour.js"/>
        </xpath>
    </template>

    <template id="qunit_suite" inherit_id="web.qunit_suite_tests">
        <xpath expr="." position="inside">
            <script type="text/javascript">
                odoo.define('mass_mailing.FieldHtml.test', function (require) {
                    'use strict';
                    var MassMailingFieldHtml = require('mass_mailing.FieldHtml');
                    MassMailingFieldHtml.include({jsLibs: []});
                });
            </script>
            <script type="text/javascript" src="/mass_mailing/static/src/js/mass_mailing_snippets.js"/>
            <script type="text/javascript" src="/mass_mailing/static/tests/mass_mailing_html_tests.js"/>
        </xpath>
    </template>

    <template id="mass_mailing_mail_style">
        <style>
            @media screen and (max-width: 768px) {
                .o_mail_col_mv {
                    display: block !important;
                    width: auto !important;
                }
                .o_mail_table_styles {
                    width: 100% !important;
                }
                .o_mail_col_container {
                    margin: 0px 0px 10px 0px !important;
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
                <sheet>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name" placeholder="e.g. John Smith"/>
                        </h1>
                        <label for="tag_ids" class="oe_edit_only"/>
                        <div>
                            <field name="tag_ids" widget="many2many_tags" style="width: 100%%"/>
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
                            <field name="country_id"/>
                        </group>
                        <group>
                            <field name="create_date" readonly="1"/>
                            <field name="message_bounce"/>
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
            <pivot string="Mailing List Contacts" stacked="True" sample="1">
                <field name="create_date" type="row"/>
            </pivot>
        </field>
    </record>

    <record id="mailing_contact_view_graph" model="ir.ui.view">
        <field name="name">mailing.contact.view.graph</field>
        <field name="model">mailing.contact</field>
        <field name="priority">10</field>
        <field name="arch" type="xml">
            <graph string="Mailing List Contacts" stacked="True" sample="1">
                <field name="create_date" type="row"/>
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
                <field name="create_date"/>
                <field name="is_public"/>
                <field name="contact_nbr"/>
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
                            <field name="contact_nbr" string="Recipients" widget="statinfo"/>
                        </button>
                    </div>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_title">
                        <label for="name" class="oe_edit_only"/>
                        <h1>
                            <field name="name" placeholder="e.g. Consumer Newsletter"/>
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
        <field name="arch" type="xml">
            <form string="Contact List">
                <group>
                    <group>
                        <div class="oe_title">
                            <label for="name" class="oe_edit_only"/>
                            <h1>
                                <field name="name" placeholder="e.g. Consumer Newsletter"/>
                            </h1>
                        </div>
                    </group>
                </group>
                <group>
                    <field name="is_public"/>
                </group>
                <footer>
                    <button string="Create" name="close_dialog" type="object" class="btn-primary"/>
                    <button string="Discard" class="btn-secondary" special="cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="open_create_mass_mailing_list" model="ir.actions.act_window">
        <field name="name">Create a Mailing List</field>
        <field name="res_model">mailing.list</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="mailing_list_view_form_simplified"/>
        <field name="target">new</field>
    </record>

    <record id="mailing_list_view_kanban" model="ir.ui.view">
        <field name="name">mailing.list.view.kanban</field>
        <field name="model">mailing.list</field>
        <field name="arch" type="xml">
            <kanban js_class="mass_mailing_list_kanban" class="o_kanban_mobile" on_create="mass_mailing.open_create_mass_mailing_list" sample="1">
                <field name="name"/>
                <field name="contact_nbr"/>
                <field name="active"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div class="o_mass_mailing_kanban_main">
                                <div class="o_kanban_card_content">
                                    <div class="o_kanban_primary_left">
                                        <div class="o_primary">
                                            <span><t t-esc="record.name.value"/></span>
                                        </div>
                                    </div>
                                </div>
                                <div class="o_kanban_card_manage_pane">
                                    <div class="o_kanban_card_manage_section o_dropdown_kanban dropdown">
                                        <a role="button" class="dropdown-toggle o-no-caret btn" data-toggle="dropdown" href="#" data-display="static" aria-label="Dropdown menu" title="Dropdown menu">
                                            <span class="fa fa-ellipsis-v"/>
                                        </a>
                                        <div class="dropdown-menu" role="menu">
                                            <a t-if="widget.editable" role="menuitem" type="edit" class="dropdown-item">Edit</a>
                                            <a t-if="widget.deletable" role="menuitem" type="delete" class="dropdown-item">Delete</a>
                                            <a role="menuitem" class="dropdown-item o_kanban_mailing_active" name="toggle_active" type="object">
                                                <t t-if="record.active.raw_value">Archive</t>
                                                <t t-if="!record.active.raw_value">Restore</t>
                                            </a>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <div class="o_mailing_list_kanban_boxes">
                                <a name="action_view_contacts" type="object">
                                    <div>
                                        <span class="badge badge-pill">
                                            <i class="fa fa-user" role="img" aria-label="Contacts" title="Contacts"/>
                                            <t t-esc="record.contact_nbr.value"/>
                                        </span>
                                    </div>
                                </a>
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
            Create a new mailing list
          </p><p>
            You don't need to import your mailing lists, you can easily
            send emails<br/> to any contact saved in other Odoo apps.
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
                <tree string="Mailings" sample="1">
                    <field name="subject" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                    <field name="mailing_type" invisible="1"/>
                    <field name="mailing_model_id" string="Recipients"/>
                    <field name="user_id" widget="many2one_avatar_user"/>
                    <field name="schedule_date" string="Scheduled" widget="remaining_days"/>
                    <field name="sent_date" widget="date"/>
                    <field name="state" decoration-info="state == 'draft' or state == 'in_queue'" decoration-success="state == 'sending' or state == 'done'" widget="badge"/>
                    <field name="campaign_id" string="Campaign"
                        groups="mass_mailing.group_mass_mailing_campaign"/>
                    <field name="sent"/>
                    <field name="bounced_ratio" string="Bounced (%)"/>
                    <field name="received_ratio" string="Delivered (%)"/>
                    <field name="opened_ratio" string="Opened (%)"/>
                    <field name="clicks_ratio" string="Clicked (%)"/>
                    <field name="replied_ratio" string="Replied (%)"/>
                </tree>
            </field>
        </record>

        <record model="ir.ui.view" id="view_mail_mass_mailing_form">
            <field name="name">mailing.mailing.form</field>
            <field name="model">mailing.mailing</field>
            <field name="arch" type="xml">
                <form string="Mailing">
                    <header style="min-height:31px;">
                        <button name="action_put_in_queue" type="object" attrs="{'invisible': [('state', 'in', ('in_queue', 'sending', 'done'))]}" class="oe_highlight" string="Send"
                            confirm="This will send the email to all recipients. Do you still want to proceed ?"/>
                        <button name="action_schedule" type="object" attrs="{'invisible': [('state', 'in', ('in_queue', 'sending', 'done'))]}" class="btn-secondary" string="Schedule"/>
                        <button name="action_test" type="object" class="btn-secondary" string="Test"/>
                        <button name="action_cancel" type="object" attrs="{'invisible': [('state', '!=', 'in_queue')]}" class="btn-secondary" string="Cancel"/>
                        <button name="action_retry_failed" type="object" attrs="{'invisible': ['|', ('state', '!=', 'done'), ('failed', '=', 0)]}" class="oe_highlight" string="Retry"/>

                        <field name="state" readonly="1" widget="statusbar"/>
                    </header>
                    <div class="alert alert-info text-center" role="alert" attrs="{'invisible': ['&amp;','&amp;','&amp;','&amp;',('state', '!=', 'in_queue'),('sent', '=', 0),('ignored', '=', 0),('scheduled', '=', 0),('failed', '=', 0)]}">
                        <div attrs="{'invisible': [('ignored', '=', 0)]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_ignored"
                                    type="object">
                                <strong>
                                    <field name="ignored" class="oe_inline mr-2"/>
                                    <span name="ignored_text">emails have been ignored and will not be sent.</span>
                                </strong>
                            </button>
                        </div>
                        <div attrs="{'invisible': [('scheduled', '=', 0)]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_scheduled"
                                    type="object">
                                <strong>
                                    <field name="scheduled" class="oe_inline mr-2"/>
                                    <span name="scheduled_text">emails are in queue and will be sent soon.</span>
                                </strong>
                            </button>
                        </div>
                        <div attrs="{'invisible': ['&amp;', ('sent', '=', 0), ('state', 'in', ('draft', 'test', 'in_queue'))]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_sent"
                                    type="object">
                                <strong>
                                    <field name="sent" class="oe_inline mr-2"/>
                                    <span name="sent">emails have been sent.</span>
                                </strong>
                            </button>
                        </div>
                        <div attrs="{'invisible': ['|', ('state', '!=', 'done'), ('failed', '=', 0)]}">
                            <button class="btn-link py-0"
                                    name="action_view_traces_failed"
                                    type="object">
                                <strong>
                                    <field name="failed" class="oe_inline mr-2"/>
                                    <span name="failed_text">emails could not be sent.</span>
                                </strong>
                            </button>
                        </div>
                        <div attrs="{'invisible': [('state', '!=', 'in_queue')]}">
                            <strong>
                                <span name="next_departure_text">This mailing is scheduled for </span>
                                <field name="next_departure" class="oe_inline"/>.
                            </strong>
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
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="mailing_type" widget="radio" options="{'horizontal': true}" invisible="1"
                                attrs="{'readonly': [('state', '!=', 'draft')]}" force_save="1"/>
                            <field name="subject" string="Subject" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}" widget="char_emojis" placeholder="e.g. New Sale on all T-shirts"/>
                            <field name="preview" string="Preview Text" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}" widget="char_emojis" placeholder="e.g. Check it out before it's too late!"/>
                            <label for="mailing_model_id" string="Recipients"/>
                            <div name="mailing_model_id_container">
                                <div class="row">
                                    <div class="col-xs-12 col-md-3" >
                                        <field name="mailing_model_id" widget="selection"
                                            attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                    </div>
                                    <div attrs="{'invisible': [('mailing_model_name', '!=', 'mailing.list')]}" class="col-xs-12 col-md-9 pt-1">
                                        <label for="contact_list_ids" string="Select mailing lists:" class="oe_edit_only"/>
                                        <field name="contact_list_ids" widget="many2many_tags"
                                            placeholder="Select mailing lists..." class="oe_inline"
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
                                <field name="body_html" class="oe_read_only" widget="html"
                                    options="{'cssReadonly': 'mass_mailing.iframe_css_assets_readonly'}"/>
                                <field name="body_arch" class="o_mail_body oe_edit_only" widget="mass_mailing_html"
                                    options="{
                                        'snippets': 'mass_mailing.email_designer_snippets',
                                        'cssEdit': 'mass_mailing.iframe_css_assets_edit',
                                        'inline-field': 'body_html'
                                }" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
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
                            <page string="Settings" name="settings">
                                <group>
                                    <group>
                                        <field name="id" invisible="1"/>
                                        <field name="name" required="False" groups="base.group_no_one" string="Name"/>
                                        <field name="user_id" domain="[('share', '=', False)]"/>
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
                                                    'required': [('reply_to_mode', '=', 'email')],
                                                    'invisible': [('reply_to_mode', '=', 'thread')],
                                                    'readonly': [('state', 'in', ('sending', 'done'))]
                                            }"/>
                                            <div style="margin-top:-5px">
                                                <small class="oe_edit_only text-muted mb-2"
                                                    style="font-size:74%"
                                                    attrs="{'invisible': ['|', ('reply_to_mode', '=', 'thread'), ('mailing_model_name', 'in', ['mailing.contact', 'res.partner', 'mailing.list'])],}">
                                                    To track replies, this address must belong to this database.
                                                </small>
                                            </div>
                                        </div>
                                        <label for="attachment_ids"/>
                                        <div name="attachment_ids_details">
                                            <field name="attachment_ids"  widget="many2many_binary" string="Attach a file" class="oe_inline"
                                                attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                        </div>
                                        <field name="mail_server_id" groups="base.group_no_one" options="{'no_create': True, 'no_open': True}"/>
                                        <field name="keep_archives" groups="base.group_no_one" attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                    </group>
                                    <group string="Marketing" groups="base.group_no_one,mass_mailing.group_mass_mailing_campaign">
                                        <field name="campaign_id"
                                            string="Mailing Campaign"
                                            groups="mass_mailing.group_mass_mailing_campaign"
                                            attrs="{'readonly': [('state', 'in', ('sending', 'done'))],
                                                    'required': [('unique_ab_testing', '=', True)]}"/>
                                        <field name="source_id"
                                            string="Source"
                                            readonly="1"
                                            required="False"
                                            groups="base.group_no_one"
                                            attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                        <field name="medium_id"
                                             string="Medium"
                                             required="True"
                                             groups="base.group_no_one"
                                             attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                        <field name="unique_ab_testing" 
                                            groups="mass_mailing.group_mass_mailing_campaign"
                                            attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/>
                                        <label for="contact_ab_pc" groups="mass_mailing.group_mass_mailing_campaign"/>
                                        <div groups="mass_mailing.group_mass_mailing_campaign">
                                            <field name="contact_ab_pc"
                                                class="oe_inline"
                                                attrs="{'readonly': [('state', 'in', ('sending', 'done'))]}"/> %
                                        </div>
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
                                            <h3 class="my-1 o_text_overflow"  attrs="{'invisible': [('sent_date', '!=', False)]}">
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

        <record id="view_mail_mass_mailing_graph" model="ir.ui.view">
            <field name="name">mailing.mailing.graph</field>
            <field name="model">mailing.mailing</field>
            <field name="arch" type="xml">
                <graph string="Mailing" type="bar" sample="1">
                    <field name="state" type="row"/>
                </graph>
            </field>
        </record>

        <record id="mailing_mailing_action_mail" model="ir.actions.act_window">
            <field name="name">Mailings</field>
            <field name="res_model">mailing.mailing</field>
            <field name="view_mode">kanban,tree,form,graph</field>
            <field name="domain">[('mailing_type', '=', 'mail')]</field>
            <field name="context">{
                    'search_default_assigned_to_me': 1,
                    'default_user_id': uid,
                    'default_mailing_type': 'mail',
            }</field>
            <field name="help" type="html">
              <p class="o_view_nocontent_smiling_face">
                Create a new mailing
              </p><p>
                You don't need to import your mailing lists, you can easily
                send emails<br/> to any contact saved in other Odoo apps.
              </p>
            </field>
        </record>

        <record id="action_view_mass_mailings_from_campaign" model="ir.actions.act_window">
            <field name="name">Mailings</field>
            <field name="res_model">mailing.mailing</field>
            <field name="view_mode">kanban,tree,form</field>
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
    <menuitem name="Email Marketing" id="mass_mailing_menu_root" sequence="60" web_icon="mass_mailing,static/description/icon.png"/>
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
        <field name="name">mailing.trace.search</field>
        <field name="model">mailing.trace</field>
        <field name="arch" type="xml">
           <search string="Mail Statistics">
                <field name="mail_mail_id_int"/>
                <field name="message_id"/>
                <field name="email"/>
                <field name="mass_mailing_id"/>
                <filter string="Scheduled" name="filter_scheduled" domain="[('scheduled', '!=', False), ('sent', '=', False), ('exception', '=', False), ('ignored', '=', False), ('bounced', '=', False)]"/>
                <filter string="Ignored" name="filter_ignored" domain="[('scheduled', '!=', False), ('sent', '=', False), ('exception', '=', False), ('ignored', '!=', False)]"/>
                <filter string="Sent" name="filter_sent" domain="[('sent', '!=', False)]"/>
                <filter string="Delivered" name="filter_delivered" domain="[('sent', '!=', False), ('exception', '=', False), ('bounced', '=', False)]"/>
                <separator/>
                <filter string="Opened" name="filter_opened" domain="[('opened', '!=', False)]"/>
                <filter string="Clicked" name="filter_clicked" domain="[('clicked', '!=', False)]"/>
                <filter string="Replied" name="filter_replied" domain="[('replied', '!=', False)]"/>
                <filter string="Bounced" name="filter_bounced" domain="[('bounced', '!=', False)]"/>
                <filter string="Failed" name="filter_failed" domain="[('exception', '!=', False)]"/>
                <group expand="0" string="Group By">
                    <filter string="State" name="state" domain="[]" context="{'group_by':'state'}"/>
                    <filter string="Open Date" name="group_open_date" context="{'group_by': 'opened:day'}"/>
                    <filter string="Reply Date" name="group_reply_date" context="{'group_by': 'replied:day'}"/>
                    <filter string="Last State Update" name="state_update" domain="[]" context="{'group_by':'state_update'}"/>
                    <filter string="Mass Mailing"  name="mass_mailing" domain="[]" context="{'group_by':'mass_mailing_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record model="ir.ui.view" id="mailing_trace_view_tree">
        <field name="name">mailing.trace.tree</field>
        <field name="model">mailing.trace</field>
        <field name="arch" type="xml">
            <tree string="Mail Statistics" create="0">
                <field name="mass_mailing_id"/>
                <field name="email"/>
                <field name="state"/>
                <field name="message_id"/>
                <field name="scheduled"/>
                <field name="sent"/>
                <field name="exception"/>
                <field name="opened"/>
                <field name="clicked"/>
                <field name="replied"/>
                <field name="bounced"/>
                <field name="ignored"/>
            </tree>
        </field>
    </record>

    <record model="ir.ui.view" id="mailing_trace_view_form">
        <field name="name">mailing.trace.form</field>
        <field name="model">mailing.trace</field>
        <field name="arch" type="xml">
            <form string="Mail Statistics" create="0">
                <header>
                    <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <div class="alert alert-info text-center" role="alert" name="alert_mail_exception"
                        attrs="{'invisible': [('exception', '=', False), ('bounced', '=', False)]}">
                        <p>
                            <strong><span name="trace_type_name_mail">This email</span>
                            <span attrs="{'invisible': [('exception', '=', False)]}"> could not be sent</span>
                            <span attrs="{'invisible': [('bounced', '=', False)]}"> appears to be invalid</span>
                            </strong>
                        </p>
                    </div>
                    <group>
                        <group string="Recipient">
                            <field name="trace_type" invisible="1"/>
                            <field name="email"/>
                            <field name="mail_mail_id_int"/>
                            <field name="message_id"/>
                        </group>
                        <group string="Document">
                            <field name="model"/>
                            <field name="res_id"/>
                            <field name="state_update"/>
                        </group>
                    </group>
                    <group string="Marketing">
                        <group>
                            <field name="mass_mailing_id"/>
                            <field name="campaign_id"/>
                            <field name="sent"/>
                            <field name="opened"/>
                            <field name="clicked"/>
                            <field name="replied"/>
                        </group>
                        <group>
                            <field name="exception"/>
                            <field name="ignored"/>
                            <field name="bounced"/>
                            <field name="failure_type"/>
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
            <graph string="Mail Statistics" type="bar" stacked="True" interval="day" sample="1">
                <field name="state_update" type="row" interval="day"/>
                <field name="state" type="row"/>
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
        <t t-raw="body"/>
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
        <t t-call="web.layout">
            <t t-set="head">
                <t t-call-assets="web.assets_common"/>
                <t t-call-assets="mass_mailing.assets_backend"/>
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
                    <t t-raw="0"/>
                </main>
            </body>
            <xpath expr="//footer" position="replace">
                <div class="container mt16 mb8">
                    <div class="pull-right" t-ignore="true" t-if="not editable">
                        Create a <a target="_blank" href="https://www.odoo.com/page/website-builder">free website</a> with
                        <a target="_blank" class="label label-danger" href="https://www.odoo.com/page/website-builder">Odoo</a>
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
    <xpath expr="//div[@id='snippets_menu']" position="inside">
        <button type="button" disabled="disabled">Select a template</button>
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
                    <t t-snippet="mass_mailing.s_mail_block_banner" t-thumbnail="/mass_mailing/static/src/img/blocks/block_banner.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_header_view" t-thumbnail="/mass_mailing/static/src/img/blocks/block_header_browser.png"/>
                </div>
            </div>
            <div id="email_designer_default_body" class="o_panel">
                <div class="o_panel_header">Body</div>
                <div class="o_panel_body" id="email_designer_body_elements">
                    <t t-snippet="mass_mailing.s_mail_block_title_text" t-thumbnail="/mass_mailing/static/src/img/blocks/block_title_text.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_title_sub" t-thumbnail="/mass_mailing/static/src/img/blocks/block_title_sub.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_paragraph" t-thumbnail="/mass_mailing/static/src/img/blocks/block_paragraph.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_comparison_table" t-thumbnail="/mass_mailing/static/src/img/blocks/block_comparison_table.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_two_cols" t-thumbnail="/mass_mailing/static/src/img/blocks/block_two_cols.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_three_cols" t-thumbnail="/mass_mailing/static/src/img/blocks/block_three_cols.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_image_text" t-thumbnail="/mass_mailing/static/src/img/blocks/block_image_text.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_text_image" t-thumbnail="/mass_mailing/static/src/img/blocks/block_text_image.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_image" t-thumbnail="/mass_mailing/static/src/img/blocks/block_image.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_footer_separator" t-thumbnail="/mass_mailing/static/src/img/blocks/block_footer_separator.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_footer_tag_line" t-thumbnail="/mass_mailing/static/src/img/blocks/block_footer_tag_line.png"/>
                </div>
            </div>
            <div id="email_designer_default_extra" class="o_panel">
                <div class="o_panel_header">Marketing Content</div>
                <div class="o_panel_body" id="email_designer_marketing_elements">
                    <t t-snippet="mass_mailing.s_mail_block_discount2" t-thumbnail="/mass_mailing/static/src/img/blocks/block_discount2.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_discount1" t-thumbnail="/mass_mailing/static/src/img/blocks/block_discount1.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_event" t-thumbnail="/mass_mailing/static/src/img/blocks/block_event.png"/>
                    <t t-snippet="mass_mailing.s_mail_block_steps" t-thumbnail="/mass_mailing/static/src/img/blocks/block_steps.png"/>
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
    <div class="o_mail_block_header_social">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" border="0" class="o_mail_table_styles">
                <tr>
                    <td width="70%"  class="o_mail_logo_container o_mail_h_padding o_mail_v_padding">
                        &amp;nbsp;
                        <a t-att-href="(company_id.website) or '#'" style="text-decoration:none;float:none;">
                             <img border="0" src="/mass_mailing/static/src/img/theme_basic/s_default_image_logo.png" style="height:auto;max-width:400px;" alt="Your Logo" />
                        </a>
                        &amp;nbsp;
                    </td>
                    <td width="30%" class="text-right o_mail_no_resize">
                        <div class="o_mail_header_social">
                            <t t-call="mass_mailing.social_links"/>
                        </div>
                    </td>
                </tr>
            </table>
        </div>
    </div>
</template>

<template id="s_mail_block_header_text_social" name="Left Text">
    <div class="o_mail_block_header_text_social">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" border="0" class="o_mail_table_styles">
                <tr>
                    <td width="70%" class="o_mail_h_padding o_mail_v_padding">
                        &amp;nbsp;
                        <h3>
                            <a t-att-href="(company_id.website) or '#'">
                                My Company
                            </a>
                        </h3>
                        &amp;nbsp;
                    </td>
                    <td width="30%" class="text-right o_mail_no_resize">
                        <div class="o_mail_header_social">
                            <t t-call="mass_mailing.social_links"/>
                        </div>
                    </td>
                </tr>
            </table>
        </div>
    </div>
</template>

<template id="s_mail_block_header_logo" name="Centered Logo">
    <div class="o_mail_block_header_logo">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" border="0" class="o_mail_table_styles o_mail_h_padding">
                <tr>
                    <td width="35%"/>
                    <td valign="center" width="30%" class="text-center o_mail_v_padding">
                        <a t-att-href="(company_id.website) or '#'" style="text-decoration:none;">
                            <img border="0" src="/mass_mailing/static/src/img/theme_basic/s_default_image_logo.png" style="height:auto;max-width:400px;width:auto"/>
                        </a>
                    </td>
                    <td width="35%" style="text-align:right"/>
                </tr>
            </table>
        </div>
    </div>
</template>

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

<template id="s_mail_block_header_view" name="View Online">
    <div class="o_snippet_view_in_browser">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" border="0" class="o_mail_table_styles">
                <tbody>
                    <tr>
                        <td width="100%" class="o_mail_h_padding o_mail_v_padding o_mail_no_colorpicker text-center">
                            <a href="/view">
                                View Online
                            </a>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<template id="s_mail_block_title_text" name="Title Content">
    <div class="o_mail_block_title_text">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" border="0" class="o_mail_table_styles">
                <tbody>
                    <tr>
                        <td width="100%" class="o_mail_h_padding o_mail_v_padding o_mail_no_colorpicker">
                            <h2 class="mt0">Thank you for joining us!</h2>
                            <p>We want to take this opportunity to welcome you to our ever-growing community!<br/></p>
                            <p>Your platform is ready for work. It will help you reduce the costs of digital signage, attract new customers and increase sales.</p>
                            <p>Enjoy,</p>
                            <img src="/mass_mailing/static/src/img/theme_default//demo/signature.png" style="width:125px; margin-top:8px;margin-bottom:-25px;" alt="Demo Signature"/>
                            <p>
                                <small>
                                    <strong>Michael Fletcher</strong><br/>
                                    <small>Community Manager</small>
                                </small>
                            </p>
                            <div class="o_mail_v_padding text-center">
                                <a role="button" href="#" class="btn btn-primary">LOGIN</a>
                            </div>

                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

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
    <div class="o_mail_block_footer_tag_line o_mail_no_colorpicker">
        <div class="o_mail_snippet_general">
            <table class="o_mail_table_styles o_mail_full_width_padding" align="center" cellpadding="0" cellspacing="0">
                <tbody>
                    <tr>
                        <td class="text-center o_mail_h_padding o_mail_v_padding bg-o-color-2">
                            <h3 style="margin:10px 0;">Apps That Help You Grow Your Business</h3>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-center o_mail_h_padding o_mail_v_padding bg-o-color-2" style="padding:10px;">
                            &amp;nbsp;
                            <a role="button" href="#" class="btn btn-primary">My Account</a>
                            &amp;nbsp;
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <div class="bg-o-color-2">
                                &amp;nbsp;
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<template id="s_mail_block_discount2" name="Promo Code">
    <div class="o_mail_block_discount2 mb32">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles o_mail_h_padding">
                <tbody>
                    <tr>
                        <td class="o_mail_v_padding">
                            <div class="text-center d-block mx-auto">
                                <p class="o_mail_display_coupon o_mail_no_margin text-center text-o-color-2" style="font-weight:800;">
                                    $20
                                </p>
                                <h3 class="o_mail_no_margin">OFF YOUR NEXT ORDER!</h3>
                            </div>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-center">
                            <p class="text-center" style="margin-top:10px;">
                                Use This Promo Code BEFORE 1st of August
                            </p>
                            <p class="o_mail_h_padding text-center">
                                <span style="line-height: 30px;"><small>CODE: </small></span><strong class="o_code h3">45A9E77DGW8455</strong>
                            </p>
                            <p class="text-center">
                                and save $20 on your next order!
                            </p>
                        </td>
                    </tr>
                    <tr>
                        <td class="mb16 mt16 text-center">
                            <a role="button" href="#" class="btn btn-primary">Use now</a>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<template id="s_mail_block_discount1" name="Discount Offer">
    <div class="o_mail_block_discount1">
        <div class="o_mail_snippet_general">
            <table align="center" cellpadding="0" cellspacing="0" class="o_mail_table_styles">
                <tbody>
                    <tr>
                        <td class="o_mail_h_padding o_mail_v_padding text-center o_mail_col_mv" width="250">
                            <table class="o_mail_table_styles">
                                <tr>
                                    <td>
                                        <p class="o_mail_display_coupon text-o-color-2 text-center o_mail_no_margin" style="font-weight:800;">20%</p>
                                        <h4 class="text-o-color-2 text-center mt0 mb16" style="font-weight:800;">OFF</h4>
                                    </td>
                                </tr>
                                <tr>
                                    <td>
                                        <p class="text-center">FROM YOUR NEXT ORDER!</p>
                                    </td>
                                </tr>
                                <tr>
                                    <td class="text-center">
                                        <a role="button" href="#" class="btn btn-primary">Redeem Discount!</a>
                                        &amp;nbsp;
                                    </td>
                                </tr>
                            </table>
                        </td>
                        <td class="o_mail_h_padding o_mail_v_padding o_mail_col_mv" width="350">
                            <p class="o_mail_no_margin">We are continuing to grow and we miss seeing you be a part of it! We've increased store hours and have lot's of new brands available. To welcome you back please accept this 20% discount on you next purchase by clicking the button.</p>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<template id="s_mail_block_event" name="Event">
    <div class="o_mail_block_event o_mail_no_colorpicker">
        <div class="o_mail_snippet_general">
            <table align="center" cellpadding="0" cellspacing="0" class="o_mail_table_styles">
                <tr>
                    <td style="width:25%;min-height: 150px;vertical-align:middle;" class="bg-o-color-2 text-center o_mail_col_mv">
                        <div class="o_mail_h_padding o_mail_v_padding">
                            <h3 class="o_mail_no_margin">21 Jul</h3>
                            <p class="o_mail_no_margin">ALL DAY</p>
                        </div>
                    </td>
                    <td style="width:25%;min-height: 150px;" class="text-center bg-o-color-4 o_mail_col_mv">
                        <img src="/mass_mailing/static/src/img/theme_basic/s_default_image_block_event.jpg" class="d-block mx-auto"/>
                    </td>
                    <td class="o_mail_h_padding o_mail_v_padding o_mail_col_mv" style="min-height: 150px;">
                        <div style="min-height: 111px">
                            <h4>Cybersecurity</h4>
                            <p class="o_mail_no_margin">
                                Cyber-threats continue to increase.<br/>
                                The discussion will examine how to develop new norms and integrate them into EU
                            </p>
                        </div>
                        <div class="small mt16">
                            <a role="button" href="#" class="btn btn-primary">Registration</a>
                            <a role="button" href="#" class="btn btn-link" style="margin-left:10px">More Info</a>
                        </div>
                    </td>
                </tr>
            </table>
        </div>
    </div>
</template>

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
    <div class="o_mail_block_footer_social o_mail_footer_social_center">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles o_mail_full_width_padding">
                <tbody>
                    <tr>
                        <td class="o_mail_footer_social">
                            <t t-call="mass_mailing.social_links"/>
                        </td>
                    </tr>
                    <tr>
                        <td class="o_mail_footer_links">
                            <a role="button" href="/unsubscribe_from_list" class="btn btn-link">Unsubscribe</a>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <p class="o_mail_footer_copy"><span class="fa fa-copyright" role="img" aria-label="Copyright" title="Copyright"/> <t t-esc="datetime.datetime.now().year"/> All Rights Reserved</p>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<template id="s_mail_block_footer_social_left" name="Footer Left">
    <div class="o_mail_block_footer_social o_mail_footer_social_left">
        <div class="o_mail_snippet_general">
            <table align="center" cellspacing="0" cellpadding="0" class="o_mail_table_styles o_mail_full_width_padding">
                <tbody>
                    <tr>
                        <td class="o_mail_footer_description">
                            <p t-if="res_company" class="o_mail_no_margin">
                                <strong><t t-esc="res_company.partner_id.name"/></strong>
                            </p>
                            <div class="o_mail_footer_links">
                                <a role="button" href="/unsubscribe_from_list" class="btn btn-link">Unsubscribe</a>
                            </div>
                            <div>
                                <p class="o_mail_footer_copy"><span class="fa fa-copyright" role="img" aria-label="Copyright" title="Copyright"/> <t t-esc="datetime.datetime.now().year"/> All Rights Reserved</p>
                            </div>
                        </td>
                        <td class="o_mail_footer_social">
                            <t t-call="mass_mailing.social_links"/>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<template id="social_links">

    <t t-set="social_links" t-value="company_id._get_social_media_links()"/>

    <a t-if="social_links.get('social_facebook')" t-att-href="social_links.get('social_facebook')" aria-label="Facebook" title="Facebook">
        <span class="fa fa-facebook"></span>
    </a>
    <a t-if="social_links.get('social_linkedin')" t-att-href="social_links.get('social_linkedin')" style="margin-left:10px" aria-label="LinkedIn" title="LinkedIn">
        <span class="fa fa-linkedin"></span>
    </a>
    <a t-if="social_links.get('social_twitter')" t-att-href="social_links.get('social_twitter')" style="margin-left:10px" aria-label="Twitter" title="Twitter">
        <span class="fa fa-twitter"></span>
    </a>
    <a t-if="social_links.get('social_instagram')" t-att-href="social_links.get('social_instagram')" style="margin-left:10px" aria-label="Instagram" title="Instagram">
        <span class="fa fa-instagram"></span>
    </a>
</template>

<!-- Snippet themes Options -->
<template id="snippet_options">
    <t t-call="web_editor.snippet_options"/>
    <t t-raw="0"/>

    <div data-js="mass_mailing_sizing_x"
        data-selector="img, .mv, .col_mv, td, th"
        data-exclude=".o_mail_no_resize, .o_mail_no_options"/>

    <div data-js="mass_mailing_table_item"
        data-selector="td, th"
        data-exclude=".o_mail_no_options"/>

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

    <!-- TODO remove in master, the option has been disabled in 14.0 because -->
    <!-- of tricky problems to resolve that require refactoring -->
    <div t-if="False"
         data-js="SnippetSave"
         t-att-data-selector="mailing_content_selector">
        <we-button class="fa fa-fw fa-save"
                   title="Save the block to use it elsewhere"
                   data-save-snippet=""
                   data-no-preview="true"/>
    </div>

    <div data-js="sizing_y"
        data-selector=".note-editable > div:not(.o_layout), .note-editable .oe_structure > div, td, th"
        data-exclude=".o_mail_no_resize, .o_mail_no_options"/>

    <div data-selector=".note-editable > div:not(.o_layout)[style*=&quot;background-color&quot;], .note-editable .oe_structure > div[style*=&quot;background-color&quot;], table, td, th"
         data-exclude=".o_mail_no_colorpicker, .o_mail_no_options">
        <we-colorpicker string="Background Color"
            data-select-style="true"
            data-css-property="background-color"
            data-color-prefix="bg-"/>
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

    <!-- Default Theme -->
    <template id="theme_default_template">
        <div class="o_mail_block_header_logo mb16" data-snippet="s_mail_block_header_logo">
            <div class="o_mail_snippet_general">
                <table align="center" cellspacing="0" cellpadding="0" border="0" class="o_mail_table_styles o_mail_h_padding">
                    <tr>
                        <td width="20%"/>
                        <td valign="center" width="60%" class="o_mail_logo_container text-center o_mail_v_padding">
                            <a t-att-href="(company_id.website) or '#'" style="text-decoration:none;">
                                <img border="0" src="/mass_mailing/static/src/img/theme_default/s_default_image_logo.png" style="height:auto;max-width:400px;"/>
                            </a>
                        </td>
                        <td width="20%" style="text-align:right"/>
                    </tr>
                </table>
            </div>
        </div>
        <t t-snippet-call="mass_mailing.s_mail_block_title_text" />
        <t t-snippet-call="mass_mailing.s_mail_block_footer_social_left"/>
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
                    t-attf-class="oe_mailings #{record.mailing_mail_ids.raw_value.length === 0 ? 'text-muted' : ''}">
                    <t t-raw="record.mailing_mail_ids.raw_value.length"/> Mailings
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
        <field name="domain">[('is_website', '=', False)]</field>
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
                        <field name="contact_nbr" string="Number of Recipients"/>
                    </tree>
                </field>
                <footer>
                    <button name="action_mailing_lists_merge" type="object" string="Merge" class="btn-primary"/>
                    <button string="Cancel" class="btn-secondary" special="cancel"/>
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

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class MailingMailingScheduleDate(models.TransientModel):
    _name = 'mailing.mailing.schedule.date'
    _description = 'Mass Mailing Scheduling'

    schedule_date = fields.Datetime(string='Scheduled for')
    mass_mailing_id = fields.Many2one('mailing.mailing', required=True, ondelete='cascade')

    @api.constrains('schedule_date')
    def _check_schedule_date(self):
        for scheduler in self:
            if scheduler.schedule_date < fields.Datetime.now():
                raise ValidationError(_('Please select a date equal/or greater than the current date.'))

    def set_schedule_date(self):
        self.mass_mailing_id.write({'schedule_date': self.schedule_date, 'state': 'in_queue'})

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
                    <button string="Schedule" name="set_schedule_date" type="object" class="btn-primary"/>
                    <button string="Discard " class="btn-secondary" special="cancel" />
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

from odoo import fields, models, tools


class TestMassMailing(models.TransientModel):
    _name = 'mailing.mailing.test'
    _description = 'Sample Mail Wizard'

    email_to = fields.Char(string='Recipients', required=True,
                           help='Comma-separated list of email addresses.', default=lambda self: self.env.user.email_formatted)
    mass_mailing_id = fields.Many2one('mailing.mailing', string='Mailing', required=True, ondelete='cascade')

    def send_mail_test(self):
        self.ensure_one()
        ctx = dict(self.env.context)
        ctx.pop('default_state', None)
        self = self.with_context(ctx)

        mails_sudo = self.env['mail.mail'].sudo()
        mailing = self.mass_mailing_id
        test_emails = tools.email_split(self.email_to)
        mass_mail_layout = self.env.ref('mass_mailing.mass_mailing_mail_layout')

        record = self.env[mailing.mailing_model_real].search([], limit=1)
        body = mailing._prepend_preview(mailing.body_html, mailing.preview)
        subject = mailing.subject

        # If there is atleast 1 record for the model used in this mailing, then we use this one to render the template
        # Downside: Jinja syntax is only tested when there is atleast one record of the mailing's model
        if record:
            # Returns a proper error if there is a syntax error with jinja
            body = self.env['mail.render.mixin']._render_template(body, mailing.mailing_model_real, record.ids, post_process=True)[record.id]
            subject = self.env['mail.render.mixin']._render_template(subject, mailing.mailing_model_real, record.ids)[record.id]

        # Convert links in absolute URLs before the application of the shortener
        body = self.env['mail.render.mixin']._replace_local_links(body)
        body = tools.html_sanitize(body, sanitize_attributes=True, sanitize_style=True)

        for test_mail in test_emails:
            mail_values = {
                'email_from': mailing.email_from,
                'reply_to': mailing.reply_to,
                'email_to': test_mail,
                'subject': subject,
                'body_html': mass_mail_layout._render({'body': body}, engine='ir.qweb', minimal_qcontext=True),
                'notification': True,
                'mailing_id': mailing.id,
                'attachment_ids': [(4, attachment.id) for attachment in mailing.attachment_ids],
                'auto_delete': True,
                'mail_server_id': mailing.mail_server_id.id,
            }
            mail = self.env['mail.mail'].sudo().create(mail_values)
            mails_sudo |= mail
        mails_sudo.send()
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
                        Send a sample email for testing purpose to the address below.
                    </p>
                    <group>
                        <field name="email_to"/>
                    </group>
                    <footer>
                        <button string="Send Sample Mail" name="send_mail_test" type="object" class="btn-primary"/>
                        <button string="Cancel" class="btn-secondary" special="cancel" />
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

from odoo import api, fields, models, tools
from odoo.tools import email_re


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
        self.ensure_one()
        res = super(MailComposeMessage, self).get_mail_values(res_ids)
        # use only for allowed models in mass mailing
        if self.composition_mode == 'mass_mail' and \
                (self.mass_mailing_name or self.mass_mailing_id) and \
                self.env['ir.model'].sudo().search([('model', '=', self.model), ('is_mail_thread', '=', True)], limit=1):
            mass_mailing = self.mass_mailing_id
            if not mass_mailing:
                reply_to_mode = 'email' if self.no_auto_thread else 'thread'
                reply_to = self.reply_to if self.no_auto_thread else False
                mass_mailing = self.env['mailing.mailing'].create({
                        'campaign_id': self.campaign_id.id,
                        'name': self.mass_mailing_name,
                        'subject': self.subject,
                        'state': 'done',
                        'reply_to_mode': reply_to_mode,
                        'reply_to': reply_to,
                        'sent_date': fields.Datetime.now(),
                        'body_html': self.body,
                        'mailing_model_id': self.env['ir.model']._get(self.model).id,
                        'mailing_domain': self.active_domain,
                })

            # Preprocess res.partners to batch-fetch from db
            # if recipient_ids is present, it means they are partners
            # (the only object to fill get_default_recipient this way)
            recipient_partners_ids = []
            read_partners = {}
            for res_id in res_ids:
                mail_values = res[res_id]
                if mail_values.get('recipient_ids'):
                    # recipient_ids is a list of x2m command tuples at this point
                    recipient_partners_ids.append(mail_values.get('recipient_ids')[0][1])
            read_partners = self.env['res.partner'].browse(recipient_partners_ids)

            partners_email = {p.id: p.email for p in read_partners}

            opt_out_list = self._context.get('mass_mailing_opt_out_list')
            seen_list = self._context.get('mass_mailing_seen_list')
            mass_mail_layout = self.env.ref('mass_mailing.mass_mailing_mail_layout', raise_if_not_found=False)
            for res_id in res_ids:
                mail_values = res[res_id]
                if mail_values.get('email_to'):
                    mail_to = tools.email_normalize(mail_values['email_to'], force_single=False)
                else:
                    partner_id = (mail_values.get('recipient_ids') or [(False, '')])[0][1]
                    mail_to = tools.email_normalize(partners_email.get(partner_id), force_single=False)
                if (opt_out_list and mail_to in opt_out_list) or (seen_list and mail_to in seen_list) \
                        or not mail_to:
                    # prevent sending to blocked addresses that were included by mistake
                    mail_values['state'] = 'cancel'
                elif seen_list is not None:
                    seen_list.add(mail_to)
                trace_vals = {
                    'model': self.model,
                    'res_id': res_id,
                    'mass_mailing_id': mass_mailing.id,
                    # if mail_to is void, keep falsy values to allow searching / debugging traces
                    'email': mail_to or mail_values.get('email_to'),
                }
                if mail_values.get('body_html') and mass_mail_layout:
                    mail_values['body_html'] = mass_mail_layout._render({'body': mail_values['body_html']}, engine='ir.qweb', minimal_qcontext=True)
                # propagate ignored state to trace when still-born
                if mail_values.get('state') == 'cancel':
                    trace_vals['ignored'] = fields.Datetime.now()
                mail_values.update({
                    'mailing_id': mass_mailing.id,
                    'mailing_trace_ids': [(0, 0, trace_vals)],
                    # email-mode: keep original message for routing
                    'notification': mass_mailing.reply_to_mode == 'thread',
                    'auto_delete': not mass_mailing.keep_archives,
                })
        return res

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
from . import mailing_list_merge
from . import mailing_mailing_schedule_date
from . import mailing_mailing_test

```

