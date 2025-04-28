# Odoo Module: digest

Category: Marketing

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
    'name': 'KPI Digests',
    'category': 'Marketing',
    'description': """
Send KPI Digests periodically
=============================
""",
    'version': '1.1',
    'depends': [
        'mail',
        'portal',
        'resource',
    ],
    'data': [
        'security/ir.model.access.csv',
        'data/digest_data.xml',
        'data/digest_tips_data.xml',
        'data/ir_cron_data.xml',
        'data/res_config_settings_data.xml',
        'views/digest_views.xml',
        'views/digest_templates.xml',
        'views/res_config_settings_views.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from werkzeug.exceptions import Forbidden, NotFound
from werkzeug.urls import url_encode

from odoo import _
from odoo.http import Controller, request, Response, route
from odoo.tools import consteq


class DigestController(Controller):

    # csrf is disabled here because it will be called by the MUA with unpredictable session at that time
    @route('/digest/<int:digest_id>/unsubscribe_oneclik', type='http', website=True, auth='public',
           methods=['POST'], csrf=False)
    def digest_unsubscribe_oneclick(self, digest_id, token=None, user_id=None):
        """ Propose a one click button to the user to unsubscribe as defined in
        Only POST method is allowed preventing the risk that anti-spam trigger unwanted
        unsubscribe (scenario explained in the same rfc). Note: this method
        must support encoding method 'multipart/form-data' and 'application/x-www-form-urlencoded'.
        """
        self.digest_unsubscribe(digest_id, token=token, user_id=user_id)
        return Response(status=200)

    @route('/digest/<int:digest_id>/unsubscribe', type='http', website=True, auth='public', methods=['GET', 'POST'])
    def digest_unsubscribe(self, digest_id, token=None, user_id=None, one_click=None):
        """ Unsubscribe a given user from a given digest

        :param int digest_id: id of digest to unsubscribe from
        :param str token: token preventing URL forgery
        :param user_id: id of user to unsubscribe

        :param int one_click: set it to 1 when using the URL in the header of
          the email to allow mail user agent to propose a one click button to the
          user to unsubscribe as defined in rfc8058. When set to True, only POST
          method is allowed preventing the risk that anti-spam trigger unwanted
          unsubscribe (scenario explained in the same rfc). Note: this method
          must support encoding method 'multipart/form-data' and 'application/x-www-form-urlencoded'.
          NOTE: DEPRECATED PARAMETER
        """
        if one_click and int(one_click) and request.httprequest.method != "POST":
            raise Forbidden()

        digest_sudo = request.env['digest.digest'].sudo().browse(digest_id).exists()

        # new route parameters
        if digest_sudo and token and user_id:
            correct_token = digest_sudo._get_unsubscribe_token(int(user_id))
            if not consteq(correct_token, token):
                raise NotFound()
            digest_sudo._action_unsubscribe_users(request.env['res.users'].sudo().browse(int(user_id)))
        # old route was given without any token or user_id but only for auth users
        elif digest_sudo and not token and not user_id and not request.env.user.share:
            digest_sudo.action_unsubscribe()
        else:
            raise NotFound()

        return request.render('digest.portal_digest_unsubscribed', {
            'digest': digest_sudo,
        })

    @route('/digest/<int:digest_id>/set_periodicity', type='http', website=True, auth='user')
    def digest_set_periodicity(self, digest_id, periodicity='weekly'):
        if not request.env.user.has_group('base.group_erp_manager'):
            raise Forbidden()
        if periodicity not in ('daily', 'weekly', 'monthly', 'quarterly'):
            raise ValueError(_('Invalid periodicity set on digest'))

        digest = request.env['digest.digest'].browse(digest_id).exists()
        digest.action_set_periodicity(periodicity)

        url_params = {
            'model': digest._name,
            'id': digest.id,
            'active_id': digest.id,
        }
        return request.redirect('/web?#%s' % url_encode(url_params))

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\digest_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <!-- Email layout: encapsulation when sending (not used in backend display) -->
    <template id="digest_mail_layout">
&lt;!DOCTYPE html&gt;
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
        <meta name="format-detection" content="telephone=no"/>
        <meta name="viewport" content="width=device-width; initial-scale=1.0; maximum-scale=1.0; user-scalable=no;"/>
        <meta http-equiv="X-UA-Compatible" content="IE=9; IE=8; IE=7; IE=EDGE" />
        <style type="text/css">
            <t t-set="color_company" t-value="company.email_secondary_color or '#714B67'"/>
            /* Remove space around the email design. */
            html,
            body {
                margin: 0 auto !important;
                padding: 0 !important;
                height: 100% !important;
                width: 100% !important;
                font-family: Arial, Helvetica, Verdana, sans-serif;
            }
            /* Prevent Windows 10 Mail from underlining links. Styles for underlined links should be inline. */
            a {
                text-decoration: none;
            }
            #header_background {
                padding-top:20px;
            }
            #header {
                border-top: 1px solid #d8dadd;
            }
            .global_layout {
                width: 588px;
                margin: 0 auto;
                background-color: #ffffff;
                border-left: 1px solid #d8dadd;
                border-right: 1px solid #d8dadd;
            }
            .company_name {
                display: inline;
                vertical-align: middle;
                color: #878d97;
                font-weight: bold;
                font-size: 24px;
            }
            .header_title {
                color: #374151;
                font-size: 18px;
                word-break: break-all;
            }
            .td_button {
                border-radius: 3px;
                white-space: nowrap;
            }
            .td_button_connect {
                background-color: <t t-out="color_company"/>;
            }
            #button_connect {
                color: #ffffff;
                font-size: 16px;
            }
            #button_open_report {
                color: #007e84;
                font-size: 14px;
            }
            .header_date {
                color: #878d97;
                font-size: 14px;
            }
            .tip_title {
                margin-top: 0;
                font-weight: bold;
                font-size: 20px;
            }
            .tip_content {
                margin: 0 auto;
                color: #374151;
                text-align: justify;
                text-justify: inter-word;
                margin: 15px auto 0 auto;
                font-size: 16px;
                line-height: 25px;
            }
            .tip_button {
                background-color: <t t-out="color_company"/>;
                border-radius: 3px;
                padding: 10px;
                text-decoration: none;
            }
            .tip_button_text {
                color: #ffffff;
            }
            .illustration_border {
                width: 100%;
                border: 1px solid #d8dadd;
                margin-top: 20px;
            }
            .kpi_row_footer {
                padding-bottom: 20px;
            }
            .kpi_header {
                font-size: 14px;
                font-weight: bold;
                color: #374151;
            }
            .kpi_cell {
                width: 33%;
                text-align: center;
                padding-top: 10px;
                padding: 0;
            }
            .kpi_value {
                color: #374151;
                font-weight: bold;
                text-decoration: none;
                font-size: 28px;
            }
            .kpi_border_col {
                color: #374151;
            }
            .kpi_value_label {
                display: inline-block;
                margin-bottom: 10px;
                color: #878d97;
                font-size: 14px;
            }
            .kpi_margin_margin {
                margin-bottom: 10px;
            }
            .download_app {
                margin-bottom: 5px;
                display: inline-block;
            }
            .preference {
                margin-bottom: 15px;
                color: #374151;
                font-size: 14px;
            }
            .by_odoo {
                color: #878d97;
                font-size: 12px;
            }
            .odoo_link_text {
                font-weight: bold;
                color: <t t-out="color_company"/>;
            }
            .run_business {
                color: #374151;
                margin: 15px auto;
                font-size: 18px;
            }
            #footer {
                background-color: #F9FAFB;
                color: #878d97;
                text-align: center;
                font-size: 20px;
                border: 1px solid #F9FAFB;
                border-top: 0;
            }
            #stock_tip {
                overflow: auto;
                margin-top: 20px;
            }
            #stock_div_img {
                text-align: center;
            }
            #stock_img {
                width: 70%;
            }
            @media only screen and (max-width: 650px) {
                .global_layout {
                    width: 100% !important;
                }
                .d-block {
                    display: block !important;
                }
                #header_background {
                    padding-top: 0px;
                }
                #header {
                    padding: 15px 20px;
                    border: 1px solid #F9FAFB;
                }
                .company_name {
                    font-size: 15px;
                }
                .header_title {
                    margin: 5px auto;
                }
                .td_button_connect {
                    padding: 0px 8px !important;
                    height: 22px !important;
                    font-size: 12px;
                }
                .td_button_open_report {
                    padding: 0px 10px !important;
                    font-size: 12px;
                }
                #button_connect {
                    font-size: 12px;
                }
                .header_date {
                    margin-top: 10px;
                    font-size: 10px;
                }
                .tip_title {
                    font-size: 14px;
                }
                .tip_content {
                    margin: 10px auto 0 auto;
                    font-size: 12px;
                    line-height: 20px;
                }
                .illustration_border {
                    margin-top: 15px;
                }
                .kpi_value {
                    font-size: 20px;
                }
                .kpi_margin {
                    font-size: 10px !important;
                }
                .kpi_margin_margin {
                    margin-bottom: 5px;
                }
                .preference {
                    font-size: 12px;
                }
                .by_odoo {
                    font-size: 10px;
                }
                .run_business {
                    margin: 20px auto;
                    font-size: 12px;
                }
                #footer {
                    font-size: 15px;
                }
                #powered {
                    margin-top: 15px;
                }
                .tip_twocol {
                    overflow: auto;
                    margin-top: 15px;
                }
                .tip_twocol_left {
                    text-align: left;
                }
                .tip_twocol_img {
                    width: 90%;
                }
                .global_td {
                    padding: 20px;
                }
                .p0 {
                    padding: 0 !important;
                }
            }
        </style>
    </head>
    <body>
        <t t-out="body"/>
    </body>
</html>
    </template>

    <!-- DIGEST MAIN TEMPLATE -->
    <template id="digest_mail_main">
<table cellspacing="0" cellpadding="0" style="width: 100%;background-color: #F9FAFB;" align="center">
    <tbody>
        <tr>
            <td id="header_background" align="center">
                <table cellspacing="0" cellpadding="0" border="0" id="header" class="global_layout">
                    <tbody>
                        <tr>
                            <td style="padding: 20px 20px 5px 20px;" class="p0"><p t-field="company.name" class="company_name" /></td>
                            <td align="right" style="padding: 20px 20px 5px 0px;" class="p0">
                                <table>
                                    <tbody>
                                        <tr>
                                            <td class="td_button td_button_connect" style="height: 29px;padding: 3px 10px;">
                                                <a t-att-href="top_button_url" target="_blank">
                                                    <span t-esc="top_button_label" class="button" id="button_connect" />
                                                </a>
                                            </td>
                                        </tr>
                                    </tbody>
                                </table>
                            </td>
                        </tr>
                        <tr>
                            <td style="padding-left: 20px" class="p0" colspan="2">
                                <div class="header_title">
                                    <p t-esc="title"/>
                                    <p t-if="sub_title" t-esc="sub_title"/>
                                </div>
                            </td>
                        </tr>
                        <tr>
                            <td style="padding: 10px 0px 20px 20px;" class="p0" colspan="2">
                                <span t-esc="formatted_date" class="header_date"/>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </td>
        </tr>
    </tbody>
</table>
<table cellspacing="0" cellpadding="0" border="0" style="width: 100%;background-color: #F9FAFB;">
    <tbody>
        <tr>
            <td align="center">
                <table cellspacing="0" cellpadding="0" border="0" class="global_layout">
                    <tbody>
                        <tr t-if="tips" t-foreach="tips" t-as="tip">
                            <td colspan="3" style="width: 100%;padding: 20px;border: 1px solid #F9FAFB;">
                                <table>
                                    <tbody>
                                        <tr>
                                            <td t-out="tip"></td>
                                        </tr>
                                    </tbody>
                                </table>
                            </td>
                        </tr>
                        <tr t-if="kpi_data">
                            <td style="padding: 20px 20px 0px 20px;" class='global_td'>
                                <table t-foreach="kpi_data" t-as="kpi_info" style="width: 100%;" cellspacing="0" cellpadding="0">
                                    <tr>
                                        <td style="padding-bottom: 20px;">
                                            <table t-if="kpi_info.get('kpi_col1') or kpi_info.get('kpi_col2') or kpi_info.get('kpi_col3')"
                                                    t-att-data-field="kpi_info['kpi_name']" cellspacing="0" cellpadding="10" style="width: 100%;margin-bottom: 5px;">
                                                <tr class="kpi_header">
                                                    <td colspan="2" style="padding: 0px 0px 5px 0px;">
                                                        <span style="text-transform: uppercase;" t-esc="kpi_info['kpi_fullname']"  />
                                                    </td>
                                                    <td t-if="kpi_info['kpi_action']" align="right" style="padding: 0px 0px 5px 0px;">
                                                        <table>
                                                            <tbody>
                                                                <tr>
                                                                    <td class="td_button td_button_open_report" style="padding: 1px 5px;height: 24px;">
                                                                        <a t-att-href="'/web#action=%s' % kpi_info['kpi_action']">
                                                                            <span class="button" id="button_open_report">➔ Open Report</span>
                                                                        </a>
                                                                    </td>
                                                                </tr>
                                                            </tbody>
                                                        </table>
                                                    </td>
                                                </tr>
                                                <tr style="vertical-align: top;">
                                                    <td t-if="kpi_info.get('kpi_col1')" class="kpi_cell" style="padding-top: 10px; border-top: 1px solid #e6e6e6;">
                                                        <div t-call="digest.digest_tool_kpi">
                                                            <t t-set="kpi_value" t-value="kpi_info['kpi_col1']['value']" />
                                                            <t t-set="kpi_margin" t-value="kpi_info['kpi_col1'].get('margin')" />
                                                            <t t-set="kpi_subtitle" t-value="kpi_info['kpi_col1']['col_subtitle']" />
                                                        </div>
                                                    </td>
                                                    <td t-if="kpi_info.get('kpi_col2')" class="kpi_cell" style="padding-top: 10px; border-top: 1px solid #e6e6e6;">
                                                        <div t-call="digest.digest_tool_kpi">
                                                            <t t-set="kpi_value" t-value="kpi_info['kpi_col2']['value']" />
                                                            <t t-set="kpi_margin" t-value="kpi_info['kpi_col2'].get('margin')" />
                                                            <t t-set="kpi_subtitle" t-value="kpi_info['kpi_col2']['col_subtitle']" />
                                                        </div>
                                                    </td>
                                                    <td t-if="kpi_info.get('kpi_col3')" class="kpi_cell" style="padding-top: 10px; border-top: 1px solid #e6e6e6;">
                                                        <div t-call="digest.digest_tool_kpi">
                                                            <t t-set="kpi_value" t-value="kpi_info['kpi_col3']['value']" />
                                                            <t t-set="kpi_margin" t-value="kpi_info['kpi_col3'].get('margin')" />
                                                            <t t-set="kpi_subtitle" t-value="kpi_info['kpi_col3']['col_subtitle']" />
                                                        </div>
                                                    </td>
                                                </tr>
                                            </table>
                                        </td>
                                    </tr>
                                </table>
                            </td>
                        </tr>
                        <tr t-if="body" >
                            <td style="padding: 20px 20px 0px 20px;" class='global_td'><t t-out="body" /></td>
                        </tr>
                    </tbody>
                    <tfoot>
                        <tr>
                            <td style="padding: 20px; border-bottom: 1px solid #d8dadd;">
                                <table border="0" width="100%">
                                    <tbody>
                                        <tr style="background-color: #F9FAFB;">
                                            <td align="center" colspan="3" valign="center" style="padding: 15px;">
                                                <div t-if="preferences" t-foreach="preferences" t-as="preference" class="preference">
                                                    <t t-out="preference" />
                                                </div>
                                                <div class="by_odoo" style="margin-bottom: 15px;">
                                                    Sent by <a href="https://www.odoo.com" target="_blank">
                                                    <span class="odoo_link_text">Odoo</span></a>
                                                    <t t-if="unsubscribe_token">
                                                        –
                                                        <a t-attf-href="/digest/#{object.id}/unsubscribe?token=#{unsubscribe_token}&amp;user_id=#{user.id}" target="_blank"
                                                            style="text-decoration: none;">
                                                            <span style="color: #878d97;">Unsubscribe</span>
                                                        </a>
                                                    </t>
                                                    <t t-elif="object and object._name == 'digest.digest'">
                                                        –
                                                        <a t-att-href="'/web#view_type=form&amp;model=digest.digest&amp;id=%s' % object.id" target="_blank"
                                                            style="text-decoration: none;">
                                                            <span style="color: #878d97;">Unsubscribe</span>
                                                        </a>
                                                    </t>
                                                </div>
                                            </td>
                                        </tr>
                                    </tbody>
                                </table>
                            </td>
                        </tr>
                        <tr t-if="display_mobile_banner" t-call="digest.digest_section_mobile" />
                    </tfoot>
                </table>
            </td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td align="center" style="padding: 20px 0px 0px 0px;">
                <table align="center">
                    <tbody>
                        <tr>
                            <div id="footer">
                                <p style="font-weight: bold;" t-esc="company.name" />
                                <p class="by_odoo" id="powered">
                                    Powered by <a href="https://www.odoo.com" target="_blank" class="odoo_link">
                                        <span class="odoo_link_text">Odoo</span></a>
                                </p>
                            </div>
                        </tr>
                    </tbody>
                </table>
            </td>
        </tr>
    </tfoot>
</table>
    </template>

    <!--                     DIGEST PARTS                    -->

    <!-- MOBILE BANNER -->
    <template id="digest_section_mobile">
<td colspan="3" style="padding: 20px;width:100%; border-bottom: 1px solid #d8dadd;">
    <table>
        <tbody>
            <tr>
                <td align="right" style="width: 33%;">
                    <img src="https://www.odoo.com/web/image/38874595-16ef5349/odoo-mobile.png" alt="Odoo Mobile" />
                </td>
                <td align="left" style="width: 66%;">
                    <table>
                        <tbody>
                            <tr>
                                <td>
                                    <p class="run_business">Run your business from anywhere with <b>Odoo Mobile</b>.</p>
                                </td>
                            </tr>
                            <tr>
                                <td>
                                    <a href="https://play.google.com/store/apps/details?id=com.odoo.mobile"
                                        target="_blank">
                                            <img class="download_app" height="40" width="135"
                                                    src="https://download.odoocdn.com/digests/digest/static/src/img/google_play.png" />
                                    </a>
                                </td>
                            </tr>
                            <tr>
                                <td>
                                    <a href="https://itunes.apple.com/us/app/odoo/id1272543640" target="_blank">
                                        <img class="download_app" height="40" width="135"
                                                src="https://download.odoocdn.com/digests/digest/static/src/img/app_store.png" />
                                    </a>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </td>
            </tr>
        </tbody>
    </table>
</td>
    </template>


    <!--                     DIGEST TOOLS                    -->

    <!-- KPI DISPLAY -->
    <template id="digest_tool_kpi">
<span t-esc="kpi_value" style="color: #374151;text-decoration: none;" class="kpi_value kpi_border_col" />
<br/>
<span t-esc="kpi_subtitle" class="kpi_value_label" />
<table t-if="kpi_margin" class="kpi_margin_margin" align="center" border="0" cellspacing="0" cellpadding="0">
    <tr>
        <td t-if="kpi_margin &gt; 0.0" class="kpi_margin positive_kpi_margin" style="padding: 3px 10px;font-size: 12px;text-decoration: none;border-radius: 50px;border: 1px solid #c4ecd7;border-radius: 5px; background-color: #c4ecd7;color: #17613a;">
            ⬆ <t t-esc="'%.2f' % kpi_margin" /> %
        </td>
        <td t-elif="kpi_margin &lt; 0.0" class="kpi_margin negative_kpi_margin" style="padding: 3px 10px;font-size: 12px;text-decoration: none;border-radius: 50px;border: 1px solid #f4cfce;background-color: #f7dddc;color: #712b29;">
            ⬇ <t t-esc="'%.2f' % kpi_margin" /> %
        </td>
    </tr>
</table>
    </template>

</data>
</odoo>

```

## File: data\digest_tips_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <data noupdate="1">
        <record id="digest_digest_default" model="digest.digest">
            <field name="name">Your Odoo Periodic Digest</field>
            <field name="periodicity">daily</field>
            <field name="user_ids" eval="[(4, ref('base.user_admin'))]"/>
            <field name="next_run_date" eval="DateTime.now().strftime('%Y-%m-%d')"/>
            <field name="kpi_res_users_connected">True</field>
            <field name="kpi_mail_message_total">True</field>
        </record>
    </data>
    <data noupdate="0">
        <record id="digest_tip_digest_0" model="digest.tip">
            <field name="name">Tip: Speed up your workflow with shortcuts</field>
            <field name="sequence">800</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Speed up your workflow with shortcuts</p>
    <p class="tip_content">Press ALT in any screen to highlight shortcuts for every button in the screen. It is useful to process multiple documents in batch.</p>
    <img src="https://download.odoocdn.com/digests/digest/static/src/img/milk-alt-shortcuts.gif" width="540" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_digest_1" model="digest.tip">
            <field name="name">Tip: Click on an avatar to chat with a user</field>
            <field name="sequence">2100</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Click on an avatar to chat with a user</p>
    <p class="tip_content">Have a question about a document? Click on the responsible user's picture to start a conversation. If his avatar has a green dot, he is online.</p>
    <img src="https://download.odoocdn.com/digests/digest/static/src/img/milk-avatar.gif" width="540" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_digest_2" model="digest.tip">
            <field name="name">Tip: A calculator in Odoo</field>
            <field name="sequence">2300</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: A calculator in Odoo</p>
    <p class="tip_content">When editing a number, you can use formulae by typing the `=` character. This is useful when computing a margin or a discount on a quotation, sale order or invoice.</p>
    <img src="https://download.odoocdn.com/digests/digest/static/src/img/milk-calculator.gif" width="540" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_digest_3" model="digest.tip">
            <field name="name">Tip: How to ping users in internal notes?</field>
            <field name="sequence">2400</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: How to ping users in internal notes?</p>
    <p class="tip_content">Type "@" to notify someone in a message, or "#" to link to a channel. Try to notify @OdooBot to test the feature.</p>
    <img src="https://download.odoocdn.com/digests/digest/static/src/img/milk-notifications.png" width="540" class="illustration_border" />
</div>
            </field>
        </record>
        <record id="digest_tip_digest_4" model="digest.tip">
            <field name="name">Tip: Knowledge is power</field>
            <field name="sequence">2500</field>
            <field name="group_id" ref="base.group_user" />
            <field name="tip_description" type="html">
<div>
    <p class="tip_title">Tip: Knowledge is power</p>
    <p class="tip_content">When following documents, use the pencil icon to fine-tune the information you want to receive.
Follow a project / sales team to keep track of this project's tasks / this team's opportunities.</p>
    <img src="https://download.odoocdn.com/digests/digest/static/src/img/milk-following.png" width="540" class="illustration_border" />
</div>
            </field>
        </record>
    </data>
</odoo>

```

## File: data\ir_cron_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <record forcecreate="True" id="ir_cron_digest_scheduler_action" model="ir.cron">
        <field name="name">Digest Emails</field>
        <field name="model_id" ref="model_digest_digest"/>
        <field name="state">code</field>
        <field name="code">model._cron_send_digest_email()</field>
        <field name="user_id" ref="base.user_root"/>
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="numbercall">-1</field>
        <field name="nextcall" eval="(DateTime.now() + timedelta(hours=2)).strftime('%Y-%m-%d %H:%M:%S')"/>
    </record>
</odoo>

```

## File: data\res_config_settings_data.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo noupdate="1">
    <record id="default_emails_digest" model="ir.config_parameter" forcecreate="0">
        <field name="key">digest.default_digest_emails</field>
        <field name="value">True</field>
    </record>
    <record id="default_digest" model="ir.config_parameter" forcecreate="0">
        <field name="key">digest.default_digest_id</field>
        <field name="value" ref="digest.digest_digest_default"/>
    </record>
</odoo>

```

## File: models\digest.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import pytz

from datetime import datetime, date
from dateutil.relativedelta import relativedelta
from markupsafe import Markup
from werkzeug.urls import url_encode, url_join

from odoo import api, fields, models, tools, _
from odoo.addons.base.models.ir_mail_server import MailDeliveryException
from odoo.exceptions import AccessError
from odoo.osv import expression
from odoo.tools.float_utils import float_round

_logger = logging.getLogger(__name__)


class Digest(models.Model):
    _name = 'digest.digest'
    _description = 'Digest'

    # Digest description
    name = fields.Char(string='Name', required=True, translate=True)
    user_ids = fields.Many2many('res.users', string='Recipients', domain="[('share', '=', False)]")
    periodicity = fields.Selection([('daily', 'Daily'),
                                    ('weekly', 'Weekly'),
                                    ('monthly', 'Monthly'),
                                    ('quarterly', 'Quarterly')],
                                   string='Periodicity', default='daily', required=True)
    next_run_date = fields.Date(string='Next Mailing Date')
    currency_id = fields.Many2one(related="company_id.currency_id", string='Currency', readonly=False)
    company_id = fields.Many2one('res.company', string='Company', default=lambda self: self.env.company.id)
    available_fields = fields.Char(compute='_compute_available_fields')
    is_subscribed = fields.Boolean('Is user subscribed', compute='_compute_is_subscribed')
    state = fields.Selection([('activated', 'Activated'), ('deactivated', 'Deactivated')], string='Status', readonly=True, default='activated')
    # First base-related KPIs
    kpi_res_users_connected = fields.Boolean('Connected Users')
    kpi_res_users_connected_value = fields.Integer(compute='_compute_kpi_res_users_connected_value')
    kpi_mail_message_total = fields.Boolean('Messages Sent')
    kpi_mail_message_total_value = fields.Integer(compute='_compute_kpi_mail_message_total_value')

    @api.depends('user_ids')
    def _compute_is_subscribed(self):
        for digest in self:
            digest.is_subscribed = self.env.user in digest.user_ids

    def _compute_available_fields(self):
        for digest in self:
            kpis_values_fields = []
            for field_name, field in digest._fields.items():
                if field.type == 'boolean' and field_name.startswith(('kpi_', 'x_kpi_', 'x_studio_kpi_')) and digest[field_name]:
                    kpis_values_fields += [field_name + '_value']
            digest.available_fields = ', '.join(kpis_values_fields)

    def _get_kpi_compute_parameters(self):
        """Get the parameters used to computed the KPI value."""
        companies = self.company_id
        if any(not digest.company_id for digest in self):
            # No company: we will use the current company to compute the KPIs
            companies |= self.env.company

        return (
            fields.Datetime.to_string(self.env.context.get('start_datetime')),
            fields.Datetime.to_string(self.env.context.get('end_datetime')),
            companies,
        )

    def _compute_kpi_res_users_connected_value(self):
        self._calculate_company_based_kpi(
            'res.users',
            'kpi_res_users_connected_value',
            date_field='login_date',
        )

    def _compute_kpi_mail_message_total_value(self):
        start, end, __ = self._get_kpi_compute_parameters()
        self.kpi_mail_message_total_value = self.env['mail.message'].search_count([
            ('create_date', '>=', start),
            ('create_date', '<', end),
            ('subtype_id', '=', self.env.ref('mail.mt_comment').id),
            ('message_type', 'in', ('comment', 'email', 'email_outgoing')),
        ])

    @api.onchange('periodicity')
    def _onchange_periodicity(self):
        self.next_run_date = self._get_next_run_date()

    @api.model_create_multi
    def create(self, vals_list):
        digests = super().create(vals_list)
        for digest in digests:
            if not digest.next_run_date:
                digest.next_run_date = digest._get_next_run_date()
        return digests

    # ------------------------------------------------------------
    # ACTIONS
    # ------------------------------------------------------------

    def action_subscribe(self):
        if self.env.user._is_internal() and self.env.user not in self.user_ids:
            self._action_subscribe_users(self.env.user)

    def _action_subscribe_users(self, users):
        """ Private method to manage subscriptions. Done as sudo() to speedup
        computation and avoid ACLs issues. """
        self.sudo().user_ids |= users

    def action_unsubscribe(self):
        if self.env.user._is_internal() and self.env.user in self.user_ids:
            self._action_unsubscribe_users(self.env.user)

    def _action_unsubscribe_users(self, users):
        """ Private method to manage subscriptions. Done as sudo() to speedup
        computation and avoid ACLs issues. """
        self.sudo().user_ids -= users

    def action_activate(self):
        self.state = 'activated'

    def action_deactivate(self):
        self.state = 'deactivated'

    def action_set_periodicity(self, periodicity):
        self.periodicity = periodicity

    def action_send(self):
        """ Send digests emails to all the registered users. """
        return self._action_send(update_periodicity=True)

    def action_send_manual(self):
        """ Manually send digests emails to all registered users. In that case
        do not update periodicity as this is not an automation rule that could
        be considered as unwanted spam. """
        return self._action_send(update_periodicity=False)

    def _action_send(self, update_periodicity=True):
        """ Send digests email to all the registered users.

        :param bool update_periodicity: if True, check user logs to update
          periodicity of digests. Purpose is to slow down digest whose users
          do not connect to avoid spam;
        """
        to_slowdown = self._check_daily_logs() if update_periodicity else self.env['digest.digest']

        for digest in self:
            for user in digest.user_ids:
                digest.with_context(
                    digest_slowdown=digest in to_slowdown,
                    lang=user.lang
                )._action_send_to_user(user, tips_count=1)
            if digest in to_slowdown:
                digest.periodicity = digest._get_next_periodicity()[0]
            digest.next_run_date = digest._get_next_run_date()

    def _action_send_to_user(self, user, tips_count=1, consume_tips=True):
        unsubscribe_token = self._get_unsubscribe_token(user.id)

        rendered_body = self.env['mail.render.mixin']._render_template(
            'digest.digest_mail_main',
            'digest.digest',
            self.ids,
            engine='qweb_view',
            add_context={
                'title': self.name,
                'top_button_label': _('Connect'),
                'top_button_url': self.get_base_url(),
                'company': user.company_id,
                'user': user,
                'unsubscribe_token': unsubscribe_token,
                'tips_count': tips_count,
                'formatted_date': datetime.today().strftime('%B %d, %Y'),
                'display_mobile_banner': True,
                'kpi_data': self._compute_kpis(user.company_id, user),
                'tips': self._compute_tips(user.company_id, user, tips_count=tips_count, consumed=consume_tips),
                'preferences': self._compute_preferences(user.company_id, user),
            },
            options={
                'preserve_comments': True,
                'post_process': True,
            },
        )[self.id]
        full_mail = self.env['mail.render.mixin']._render_encapsulate(
            'digest.digest_mail_layout',
            rendered_body,
            add_context={
                'company': user.company_id,
                'user': user,
            },
        )
        # create a mail_mail based on values, without attachments
        unsub_params = url_encode({
            "token": unsubscribe_token,
            "user_id": user.id,
        })
        unsub_url = url_join(
            self.get_base_url(),
            f'/digest/{self.id}/unsubscribe_oneclik?{unsub_params}'
        )
        mail_values = {
            'auto_delete': True,
            'author_id': self.env.user.partner_id.id,
            'body_html': full_mail,
            'email_from': (
                self.company_id.partner_id.email_formatted
                or self.env.user.email_formatted
                or self.env.ref('base.user_root').email_formatted
            ),
            'email_to': user.email_formatted,
            # Add headers that allow the MUA to offer a one click button to unsubscribe (requires DKIM to work)
            'headers': {
                'List-Unsubscribe': f'<{unsub_url}>',
                'List-Unsubscribe-Post': 'List-Unsubscribe=One-Click',
                'X-Auto-Response-Suppress': 'OOF',  # avoid out-of-office replies from MS Exchange
            },
            'state': 'outgoing',
            'subject': '%s: %s' % (user.company_id.name, self.name),
        }
        self.env['mail.mail'].sudo().create(mail_values)
        return True

    @api.model
    def _cron_send_digest_email(self):
        digests = self.search([('next_run_date', '<=', fields.Date.today()), ('state', '=', 'activated')])
        for digest in digests:
            try:
                digest.action_send()
            except MailDeliveryException as e:
                _logger.warning('MailDeliveryException while sending digest %d. Digest is now scheduled for next cron update.', digest.id)

    def _get_unsubscribe_token(self, user_id):
        """Generate a secure hash for this digest and user. It allows to
        unsubscribe from a digest while keeping some security in that process.

        :param int user_id: ID of the user to unsubscribe
        """
        return tools.hmac(self.env(su=True), 'digest-unsubscribe', (self.id, user_id))

    # ------------------------------------------------------------
    # KPIS
    # ------------------------------------------------------------

    def _compute_kpis(self, company, user):
        """ Compute KPIs to display in the digest template. It is expected to be
        a list of KPIs, each containing values for 3 columns display.

        :return list: result [{
            'kpi_name': 'kpi_mail_message',
            'kpi_fullname': 'Messages',  # translated
            'kpi_action': 'crm.crm_lead_action_pipeline',  # xml id of an action to execute
            'kpi_col1': {
                'value': '12.0',
                'margin': 32.36,
                'col_subtitle': 'Yesterday',  # translated
            },
            'kpi_col2': { ... },
            'kpi_col3':  { ... },
        }, { ... }] """
        self.ensure_one()
        digest_fields = self._get_kpi_fields()
        invalid_fields = []
        kpis = [
            dict(kpi_name=field_name,
                 kpi_fullname=self.env['ir.model.fields']._get(self._name, field_name).field_description,
                 kpi_action=False,
                 kpi_col1=dict(),
                 kpi_col2=dict(),
                 kpi_col3=dict(),
                 )
            for field_name in digest_fields
        ]
        kpis_actions = self._compute_kpis_actions(company, user)

        for col_index, (tf_name, tf) in enumerate(self._compute_timeframes(company)):
            digest = self.with_context(start_datetime=tf[0][0], end_datetime=tf[0][1]).with_user(user).with_company(company)
            previous_digest = self.with_context(start_datetime=tf[1][0], end_datetime=tf[1][1]).with_user(user).with_company(company)
            for index, field_name in enumerate(digest_fields):
                kpi_values = kpis[index]
                kpi_values['kpi_action'] = kpis_actions.get(field_name)
                try:
                    compute_value = digest[field_name + '_value']
                    # Context start and end date is different each time so invalidate to recompute.
                    digest.invalidate_model([field_name + '_value'])
                    previous_value = previous_digest[field_name + '_value']
                    # Context start and end date is different each time so invalidate to recompute.
                    previous_digest.invalidate_model([field_name + '_value'])
                except AccessError:  # no access rights -> just skip that digest details from that user's digest email
                    invalid_fields.append(field_name)
                    continue
                margin = self._get_margin_value(compute_value, previous_value)
                if self._fields['%s_value' % field_name].type == 'monetary':
                    converted_amount = tools.format_decimalized_amount(compute_value)
                    compute_value = self._format_currency_amount(converted_amount, company.currency_id)
                elif self._fields['%s_value' % field_name].type == 'float':
                    compute_value = "%.2f" % compute_value

                kpi_values['kpi_col%s' % (col_index + 1)].update({
                    'value': compute_value,
                    'margin': margin,
                    'col_subtitle': tf_name,
                })

        # filter failed KPIs
        return [kpi for kpi in kpis if kpi['kpi_name'] not in invalid_fields]

    def _compute_tips(self, company, user, tips_count=1, consumed=True):
        tips = self.env['digest.tip'].search([
            ('user_ids', '!=', user.id),
            '|', ('group_id', 'in', user.groups_id.ids), ('group_id', '=', False)
        ], limit=tips_count)
        tip_descriptions = [
            tools.html_sanitize(
                self.env['mail.render.mixin'].sudo()._render_template(
                    tip.tip_description,
                    'digest.tip',
                    tip.ids,
                    engine="qweb",
                    options={'post_process': True},
                )[tip.id]
            )
            for tip in tips
        ]
        if consumed:
            tips.user_ids += user
        return tip_descriptions

    def _compute_kpis_actions(self, company, user):
        """ Give an optional action to display in digest email linked to some KPIs.

        :return dict: key: kpi name (field name), value: an action that will be
          concatenated with /web#action={action}
        """
        return {}

    def _compute_preferences(self, company, user):
        """ Give an optional text for preferences, like a shortcut for configuration.

        :return string: html to put in template
        """
        preferences = []
        if self._context.get('digest_slowdown'):
            _dummy, new_perioridicy_str = self._get_next_periodicity()
            preferences.append(
                _("We have noticed you did not connect these last few days. We have automatically switched your preference to %(new_perioridicy_str)s Digests.",
                  new_perioridicy_str=new_perioridicy_str)
            )
        elif self.periodicity == 'daily' and user.has_group('base.group_erp_manager'):
            preferences.append(Markup('<p>%s<br /><a href="%s" target="_blank" style="color:#017e84; font-weight: bold;">%s</a></p>') % (
                _('Prefer a broader overview?'),
                f'/digest/{self.id:d}/set_periodicity?periodicity=weekly',
                _('Switch to weekly Digests')
            ))
        if user.has_group('base.group_erp_manager'):
            preferences.append(Markup('<p>%s<br /><a href="%s" target="_blank" style="color:#017e84; font-weight: bold;">%s</a></p>') % (
                _('Want to customize this email?'),
                f'/web#view_type=form&model={self._name}&id={self.id:d}',
                _('Choose the metrics you care about')
            ))

        return preferences

    def _get_next_run_date(self):
        self.ensure_one()
        if self.periodicity == 'daily':
            delta = relativedelta(days=1)
        elif self.periodicity == 'weekly':
            delta = relativedelta(weeks=1)
        elif self.periodicity == 'monthly':
            delta = relativedelta(months=1)
        else:
            delta = relativedelta(months=3)
        return date.today() + delta

    def _compute_timeframes(self, company):
        start_datetime = datetime.utcnow()
        tz_name = company.resource_calendar_id.tz
        if tz_name:
            start_datetime = pytz.timezone(tz_name).localize(start_datetime)
        return [
            (_('Last 24 hours'), (
                (start_datetime + relativedelta(days=-1), start_datetime),
                (start_datetime + relativedelta(days=-2), start_datetime + relativedelta(days=-1)))
            ), (_('Last 7 Days'), (
                (start_datetime + relativedelta(weeks=-1), start_datetime),
                (start_datetime + relativedelta(weeks=-2), start_datetime + relativedelta(weeks=-1)))
            ), (_('Last 30 Days'), (
                (start_datetime + relativedelta(months=-1), start_datetime),
                (start_datetime + relativedelta(months=-2), start_datetime + relativedelta(months=-1)))
            )
        ]

    # ------------------------------------------------------------
    # FORMATTING / TOOLS
    # ------------------------------------------------------------

    def _calculate_company_based_kpi(self, model, digest_kpi_field, date_field='create_date',
                                     additional_domain=None, sum_field=None):
        """Generic method that computes the KPI on a given model.

        :param model: Model on which we will compute the KPI
            This model must have a "company_id" field
        :param digest_kpi_field: Field name on which we will write the KPI
        :param date_field: Field used for the date range
        :param additional_domain: Additional domain
        :param sum_field: Field to sum to obtain the KPI,
            if None it will count the number of records
        """
        start, end, companies = self._get_kpi_compute_parameters()

        base_domain = [
            ('company_id', 'in', companies.ids),
            (date_field, '>=', start),
            (date_field, '<', end),
        ]

        if additional_domain:
            base_domain = expression.AND([base_domain, additional_domain])

        values = self.env[model]._read_group(
            domain=base_domain,
            groupby=['company_id'],
            aggregates=[f'{sum_field}:sum'] if sum_field else ['__count'],
        )

        values_per_company = {company.id: agg for company, agg in values}
        for digest in self:
            company = digest.company_id or self.env.company
            digest[digest_kpi_field] = values_per_company.get(company.id, 0)

    def _get_kpi_fields(self):
        return [field_name for field_name, field in self._fields.items()
                if field.type == 'boolean' and field_name.startswith(('kpi_', 'x_kpi_', 'x_studio_kpi_')) and self[field_name]
               ]

    def _get_margin_value(self, value, previous_value=0.0):
        margin = 0.0
        if (value != previous_value) and (value != 0.0 and previous_value != 0.0):
            margin = float_round((float(value-previous_value) / previous_value or 1) * 100, precision_digits=2)
        return margin

    def _check_daily_logs(self):
        """ Badly named method that checks user logs and slowdown the sending
        of digest emails based on recipients being away. """
        today = datetime.now().replace(microsecond=0)
        to_slowdown = self.env['digest.digest']
        for digest in self:
            if digest.periodicity == 'daily':  # 2 days ago
                limit_dt = today - relativedelta(days=2)
            elif digest.periodicity == 'weekly':  # 1 week ago
                limit_dt = today - relativedelta(days=7)
            elif digest.periodicity == 'monthly':  # 1 month ago
                limit_dt = today - relativedelta(months=1)
            elif digest.periodicity == 'quarterly':  # 3 month ago
                limit_dt = today - relativedelta(months=3)
            users_logs = self.env['res.users.log'].sudo().search_count([
                ('create_uid', 'in', digest.user_ids.ids),
                ('create_date', '>=', limit_dt)
            ])
            if not users_logs:
                to_slowdown += digest
        return to_slowdown

    def _get_next_periodicity(self):
        if self.periodicity == 'daily':
            return 'weekly', _('weekly')
        if self.periodicity == 'weekly':
            return 'monthly', _('monthly')
        return 'quarterly', _('quarterly')

    def _format_currency_amount(self, amount, currency_id):
        pre = currency_id.position == 'before'
        symbol = u'{symbol}'.format(symbol=currency_id.symbol or '')
        return u'{pre}{0}{post}'.format(amount, pre=symbol if pre else '', post=symbol if not pre else '')

```

## File: models\digest_tip.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models
from odoo.tools.translate import html_translate


class DigestTip(models.Model):
    _name = 'digest.tip'
    _description = 'Digest Tips'
    _order = 'sequence'

    sequence = fields.Integer(
        'Sequence', default=1,
        help='Used to display digest tip in email template base on order')
    name = fields.Char('Name', translate=True)
    user_ids = fields.Many2many(
        'res.users', string='Recipients',
        help='Users having already received this tip')
    tip_description = fields.Html('Tip description', translate=html_translate, sanitize=False)
    group_id = fields.Many2one(
        'res.groups', string='Authorized Group',
        default=lambda self: self.env.ref('base.group_user'))

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    digest_emails = fields.Boolean(string="Digest Emails", config_parameter='digest.default_digest_emails')
    digest_id = fields.Many2one('digest.digest', string='Digest Email', config_parameter='digest.default_digest_id')

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class ResUsers(models.Model):
    _inherit = "res.users"

    @api.model_create_multi
    def create(self, vals_list):
        """ Automatically subscribe employee users to default digest if activated """
        users = super(ResUsers, self).create(vals_list)
        default_digest_emails = self.env['ir.config_parameter'].sudo().get_param('digest.default_digest_emails')
        default_digest_id = self.env['ir.config_parameter'].sudo().get_param('digest.default_digest_id')
        users_to_subscribe = users.filtered_domain([('share', '=', False)])
        if default_digest_emails and default_digest_id and users_to_subscribe:
            digest = self.env['digest.digest'].sudo().browse(int(default_digest_id)).exists()
            digest.user_ids |= users_to_subscribe
        return users

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import digest
from . import digest_tip
from . import res_config_settings
from . import res_users

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_digest_digest_system,digest.digest.administration,model_digest_digest,base.group_erp_manager,1,1,1,1
access_digest_digest_user,digest.digest.user,model_digest_digest,base.group_user,1,0,0,0
access_digest_tip_system,digest.tip.administration,model_digest_tip,base.group_erp_manager,1,1,1,1
access_digest_tip_user,digest.tip.user,model_digest_tip,base.group_user,1,0,0,0

```

## File: views\digest_templates.xml

```xml
<odoo>
    <template id="portal_digest_unsubscribed" name="Unsubscription">
        <t t-call="portal.portal_layout">
            <div class="container mt8">
                <div class="row">
                    <div class="col-lg-6 offset-lg-3">
                        <h3>Digest Subscriptions</h3>
                        <div class="alert alert-success text-center" role="status">
                            <p>You have been successfully unsubscribed from:<br/>
                            <strong t-field="digest.name"/></p>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>
</odoo>
```

## File: views\digest_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="digest_digest_view_tree" model="ir.ui.view">
        <field name="name">digest.digest.view.tree</field>
        <field name="model">digest.digest</field>
        <field name="arch" type="xml">
            <tree string="KPI Digest">
                <field name="name" string="Title"/>
                <field name="periodicity"/>
                <field name="next_run_date" groups="base.group_no_one"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="state" groups="base.group_no_one" widget="badge" decoration-success="state == 'activated'"/>
            </tree>
        </field>
    </record>
    <record id="digest_digest_view_form" model="ir.ui.view">
        <field name="name">digest.digest.view.form</field>
        <field name="model">digest.digest</field>
        <field name="arch" type="xml">
            <form string="KPI Digest">
                <field name="is_subscribed" invisible="1"/>
                <header>
                    <button type="object" name="action_send_manual" string="Send Now"
                        class="oe_highlight"
                        invisible="state == 'deactivated'" groups="base.group_system"/>
                    <button type="object" name="action_deactivate" string="Deactivate"
                        invisible="state == 'deactivated'" groups="base.group_system"/>
                    <button type="object" name="action_activate" string="Activate"
                        class="oe_highlight"
                        invisible="state == 'activated'" groups="base.group_system"/>
                    <field name="state" widget="statusbar" statusbar_visible="0"/>
                </header>
                <sheet>
                    <div class="oe_title">
                        <label for="name" string="Digest Title"/>
                        <h1>
                            <field name="name" placeholder="e.g. Your Weekly Digest"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="periodicity" widget="radio" options="{'horizontal': true}"/>
                            <field name="company_id" options="{'no_create': True}" invisible="1"/>
                        </group>
                        <group>
                            <field name="next_run_date" groups="base.group_system"/>
                        </group>
                    </group>
                    <notebook>
                        <page name="kpis" string="KPIs">
                            <group name="kpis">
                                <group name="kpi_general" string="General" groups="base.group_system">
                                    <field name="kpi_res_users_connected"/>
                                    <field name="kpi_mail_message_total"/>
                                </group>
                                <group name="kpi_sales"/>
                                <group name="custom" string="Custom" groups="base.group_system">
                                    <div colspan="2">
                                        <p>Want to add your own KPIs?<br />
                                        <a href="https://www.odoo.com/documentation/17.0/applications/general/digest_emails.html#custom-digest-emails" target="_blank"><i class="oi oi-arrow-right"></i> Check our Documentation</a></p>
                                    </div>
                                </group>
                            </group>
                        </page>
                        <page name="recipients" string="Recipients" groups="base.group_system">
                            <field name="user_ids" options="{'no_create': True}">
                                <tree string="Recipients">
                                    <field name="name"/>
                                    <field name="email" string="Email Address" />
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>
    <record id="digest_digest_view_search" model="ir.ui.view">
        <field name="name">digest.digest.view.search</field>
        <field name="model">digest.digest</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <field name="user_ids"/>
                <filter name="filter_activated" string="Activated" domain="[('state', '=', 'activated')]"/>
                <filter name="filter_deactivated" string="Deactivated" domain="[('state', '=', 'deactivated')]"/>
                <group expand="1" string="Group by">
                    <filter string="Periodicity" name="periodicity" context="{'group_by': 'periodicity'}"/>
                </group>
            </search>
        </field>
    </record>
    <record id="digest_digest_action" model="ir.actions.act_window">
        <field name="name">Digest Emails</field>
        <field name="res_model">digest.digest</field>
        <field name="context">{'search_default_filter_activated': 1}</field>
        <field name="search_view_id" ref="digest_digest_view_search"/>
    </record>

    <menuitem id="digest_menu"
        action="digest_digest_action"
        parent="base.menu_email"
        groups="base.group_erp_manager"
        sequence="80"/>

    <!-- DIGEST.TIP -->
    <record id="digest_tip_view_tree" model="ir.ui.view">
        <field name="name">digest.tip.view.tree</field>
        <field name="model">digest.tip</field>
        <field name="arch" type="xml">
            <tree string="KPI Digest Tips">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="group_id"/>
            </tree>
        </field>
    </record>
    <record id="digest_tip_view_form" model="ir.ui.view">
        <field name="name">digest.tip.view.form</field>
        <field name="model">digest.tip</field>
        <field name="arch" type="xml">
            <form string="KPI Digest Tip">
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="tip_description"/>
                        <field name="group_id"/>
                        <field name="user_ids" widget="many2many_tags"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
    <record id="digest_tip_view_search" model="ir.ui.view">
        <field name="name">digest.tip.view.search</field>
        <field name="model">digest.tip</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <field name="tip_description"/>
                <field name="group_id"/>
            </search>
        </field>
    </record>
    <record id="digest_tip_action" model="ir.actions.act_window">
        <field name="name">Digest Tips</field>
        <field name="res_model">digest.tip</field>
        <field name="search_view_id" ref="digest_tip_view_search"/>
    </record>

    <menuitem id="digest_tip_menu"
        action="digest_tip_action"
        parent="base.menu_email"
        groups="base.group_erp_manager"
        sequence="81"/>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.digest</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='contacts_settings']" position="before">
                <div id="statistics" >
                    <block title="Statistics" id='statistics_div'>
                        <setting string="Digest Email" help="Add new users as recipient of a periodic email with key metrics" documentation="/applications/general/digest_emails.html" title="New users are automatically added as recipient of the following digest email." name="digest_email_setting_container">
                            <field name="digest_emails"/>
                            <div class="content-group" invisible="not digest_emails">
                                <div class="mt16">
                                    <label for="digest_id" class="o_light_label mr8"/>
                                    <field name="digest_id" class="oe_inline"/>
                                </div>
                                <div class="mt8">
                                    <button type="action" name="%(digest.digest_digest_action)d" string="Configure Digest Emails" icon="oi-arrow-right" class="btn-link"/>
                                </div>
                            </div>
                        </setting>
                    </block>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

