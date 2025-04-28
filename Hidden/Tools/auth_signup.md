# Odoo Module: auth_signup

Category: Hidden/Tools

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
    'name': 'Signup',
    'description': """
Allow users to sign up and reset their password
===============================================
    """,
    'version': '1.0',
    'category': 'Hidden/Tools',
    'auto_install': True,
    'depends': [
        'base_setup',
        'mail',
        'web',
    ],
    'data': [
        'data/ir_config_parameter_data.xml',
        'data/ir_cron_data.xml',
        'data/mail_template_data.xml',
        'views/res_config_settings_views.xml',
        'views/res_users_views.xml',
        'views/auth_signup_login_templates.xml',
        'views/auth_signup_templates_email.xml',
        'views/webclient_templates.xml',
        ],
    'bootstrap': True,
    'assets': {
        'web.assets_frontend': [
            'auth_signup/static/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
import werkzeug
from werkzeug.urls import url_encode

from odoo import http, tools, _
from odoo.addons.auth_signup.models.res_users import SignupError
from odoo.addons.web.controllers.home import ensure_db, Home, SIGN_UP_REQUEST_PARAMS, LOGIN_SUCCESSFUL_PARAMS
from odoo.addons.base_setup.controllers.main import BaseSetup
from odoo.exceptions import UserError
from odoo.http import request
from markupsafe import Markup

_logger = logging.getLogger(__name__)

LOGIN_SUCCESSFUL_PARAMS.add('account_created')


class AuthSignupHome(Home):

    @http.route()
    def web_login(self, *args, **kw):
        ensure_db()
        response = super().web_login(*args, **kw)
        response.qcontext.update(self.get_auth_signup_config())
        if request.session.uid:
            if request.httprequest.method == 'GET' and request.params.get('redirect'):
                # Redirect if already logged in and redirect param is present
                return request.redirect(request.params.get('redirect'))
            # Add message for non-internal user account without redirect if account was just created
            if response.location == '/web/login_successful' and kw.get('confirm_password'):
                return request.redirect_query('/web/login_successful', query={'account_created': True})
        return response

    @http.route('/web/signup', type='http', auth='public', website=True, sitemap=False)
    def web_auth_signup(self, *args, **kw):
        qcontext = self.get_auth_signup_qcontext()

        if not qcontext.get('token') and not qcontext.get('signup_enabled'):
            raise werkzeug.exceptions.NotFound()

        if 'error' not in qcontext and request.httprequest.method == 'POST':
            try:
                if not request.env['ir.http']._verify_request_recaptcha_token('signup'):
                    raise UserError(_("Suspicious activity detected by Google reCaptcha."))

                self.do_signup(qcontext)

                # Set user to public if they were not signed in by do_signup
                # (mfa enabled)
                if request.session.uid is None:
                    public_user = request.env.ref('base.public_user')
                    request.update_env(user=public_user)

                # Send an account creation confirmation email
                User = request.env['res.users']
                user_sudo = User.sudo().search(
                    User._get_login_domain(qcontext.get('login')), order=User._get_login_order(), limit=1
                )
                template = request.env.ref('auth_signup.mail_template_user_signup_account_created', raise_if_not_found=False)
                if user_sudo and template:
                    template.sudo().send_mail(user_sudo.id, force_send=True)
                return self.web_login(*args, **kw)
            except UserError as e:
                qcontext['error'] = e.args[0]
            except (SignupError, AssertionError) as e:
                if request.env["res.users"].sudo().search_count([("login", "=", qcontext.get("login"))], limit=1):
                    qcontext["error"] = _("Another user is already registered using this email address.")
                else:
                    _logger.warning("%s", e)
                    qcontext['error'] = _("Could not create a new account.") + Markup('<br/>') + str(e)

        elif 'signup_email' in qcontext:
            user = request.env['res.users'].sudo().search([('email', '=', qcontext.get('signup_email')), ('state', '!=', 'new')], limit=1)
            if user:
                return request.redirect('/web/login?%s' % url_encode({'login': user.login, 'redirect': '/web'}))

        response = request.render('auth_signup.signup', qcontext)
        response.headers['X-Frame-Options'] = 'SAMEORIGIN'
        response.headers['Content-Security-Policy'] = "frame-ancestors 'self'"
        return response

    @http.route('/web/reset_password', type='http', auth='public', website=True, sitemap=False)
    def web_auth_reset_password(self, *args, **kw):
        qcontext = self.get_auth_signup_qcontext()

        if not qcontext.get('token') and not qcontext.get('reset_password_enabled'):
            raise werkzeug.exceptions.NotFound()

        if 'error' not in qcontext and request.httprequest.method == 'POST':
            try:
                if not request.env['ir.http']._verify_request_recaptcha_token('password_reset'):
                    raise UserError(_("Suspicious activity detected by Google reCaptcha."))
                if qcontext.get('token'):
                    self.do_signup(qcontext)
                    return self.web_login(*args, **kw)
                else:
                    login = qcontext.get('login')
                    assert login, _("No login provided.")
                    _logger.info(
                        "Password reset attempt for <%s> by user <%s> from %s",
                        login, request.env.user.login, request.httprequest.remote_addr)
                    request.env['res.users'].sudo().reset_password(login)
                    qcontext['message'] = _("Password reset instructions sent to your email")
            except UserError as e:
                qcontext['error'] = e.args[0]
            except SignupError:
                qcontext['error'] = _("Could not reset your password")
                _logger.exception('error when resetting password')
            except Exception as e:
                qcontext['error'] = str(e)

        elif 'signup_email' in qcontext:
            user = request.env['res.users'].sudo().search([('email', '=', qcontext.get('signup_email')), ('state', '!=', 'new')], limit=1)
            if user:
                return request.redirect('/web/login?%s' % url_encode({'login': user.login, 'redirect': '/web'}))

        response = request.render('auth_signup.reset_password', qcontext)
        response.headers['X-Frame-Options'] = 'SAMEORIGIN'
        response.headers['Content-Security-Policy'] = "frame-ancestors 'self'"
        return response

    def get_auth_signup_config(self):
        """retrieve the module config (which features are enabled) for the login page"""

        get_param = request.env['ir.config_parameter'].sudo().get_param
        return {
            'disable_database_manager': not tools.config['list_db'],
            'signup_enabled': request.env['res.users']._get_signup_invitation_scope() == 'b2c',
            'reset_password_enabled': get_param('auth_signup.reset_password') == 'True',
        }

    def get_auth_signup_qcontext(self):
        """ Shared helper returning the rendering context for signup and reset password """
        qcontext = {k: v for (k, v) in request.params.items() if k in SIGN_UP_REQUEST_PARAMS}
        qcontext.update(self.get_auth_signup_config())
        if not qcontext.get('token') and request.session.get('auth_signup_token'):
            qcontext['token'] = request.session.get('auth_signup_token')
        if qcontext.get('token'):
            try:
                # retrieve the user info (name, login or email) corresponding to a signup token
                token_infos = request.env['res.partner'].sudo()._signup_retrieve_info(qcontext.get('token'))
                for k, v in token_infos.items():
                    qcontext.setdefault(k, v)
            except:
                qcontext['error'] = _("Invalid signup token")
                qcontext['invalid_token'] = True
        return qcontext

    def _prepare_signup_values(self, qcontext):
        values = { key: qcontext.get(key) for key in ('login', 'name', 'password') }
        if not values:
            raise UserError(_("The form was not properly filled in."))
        if values.get('password') != qcontext.get('confirm_password'):
            raise UserError(_("Passwords do not match; please retype them."))
        supported_lang_codes = [code for code, _ in request.env['res.lang'].get_installed()]
        lang = request.context.get('lang', '')
        if lang in supported_lang_codes:
            values['lang'] = lang
        return values

    def do_signup(self, qcontext):
        """ Shared helper that creates a res.partner out of a token """
        values = self._prepare_signup_values(qcontext)
        self._signup_with_values(qcontext.get('token'), values)
        request.env.cr.commit()

    def _signup_with_values(self, token, values):
        login, password = request.env['res.users'].sudo().signup(values, token)
        request.env.cr.commit()     # as authenticate will use its own cursor we need to commit the current transaction
        credential = {'login': login, 'password': password, 'type': 'password'}
        request.session.authenticate(request.db, credential)

class AuthBaseSetup(BaseSetup):
    @http.route()
    def base_setup_data(self, **kwargs):
        res = super().base_setup_data(**kwargs)
        res.update({'resend_invitation': True})
        return res

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-

from . import main

```

## File: data\ir_config_parameter_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- activate B2C portal by default -->
        <function model="ir.config_parameter" name="set_param" eval="('auth_signup.invitation_scope', 'b2c')"/>
        <!-- activate Reset password by default -->
        <function model="ir.config_parameter" name="set_param" eval="('auth_signup.reset_password', 'True')"/>
    </data>
</odoo>

```

## File: data\ir_cron_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ir_cron_auth_signup_send_pending_user_reminder" model="ir.cron">
        <field name="name">Users: Notify About Unregistered Users</field>
        <field name="model_id" ref="model_res_users"/>
        <field name="state">code</field>
        <field name="code">model.send_unregistered_user_reminder(batch_size=100)</field>
        <field name="user_id" ref="base.user_root" />
        <field name="interval_number">1</field>
        <field name="interval_type">days</field>
        <field name="priority">6</field>
    </record>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Email template for new users -->
        <record id="set_password_email" model="mail.template">
            <field name="name">Settings: New User Invite</field>
            <field name="model_id" ref="base.model_res_users"/>
            <field name="subject">{{ object.create_uid.name }} from {{ object.company_id.name }} invites you to connect to Odoo</field>
            <field name="email_from">{{ (object.company_id.email_formatted or user.email_formatted) }}</field>
            <field name="email_to">{{ object.email_formatted }}</field>
            <field name="description">Sent to new user after you invited them</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #FFFFFF; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: #FFFFFF; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Welcome to Odoo</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">
                        <t t-out="object.name or ''">Marc Demo</t>
                    </span>
                </td><td valign="middle" align="right" t-if="not object.company_id.uses_default_logo">
                    <img t-attf-src="/logo.png?company={{ object.company_id.id }}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" t-att-alt="object.company_id.name"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 13px;">
                    <div>
                        Dear <t t-out="object.name or ''">Marc Demo</t>,<br /><br />
                        You have been invited by <t t-out="object.create_uid.name or ''">OdooBot</t> of <t t-out="object.company_id.name or ''">YourCompany</t> to connect on Odoo.
                        <div style="margin: 16px 0px 16px 0px;">
                            <a t-att-href="object.partner_id._get_signup_url()"
                                t-attf-style="background-color: {{object.company_id.email_secondary_color or '#875A7B'}}; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                                Accept invitation
                            </a>
                        </div>
                        <b>  This link will remain valid during <t t-out="int(int(object.env['ir.config_parameter'].sudo().get_param('auth_signup.signup.validity.hours',144))/24)"></t> days </b> <br/>
                        <t t-set="website_url" t-value="object.get_base_url()"></t>
                        Your Odoo domain is: <b><a t-att-href='website_url' t-out="website_url or ''">http://yourcompany.odoo.com</a></b><br />
                        Your sign in email is: <b><a t-attf-href="/web/login?login={{ object.email }}" target="_blank" t-out="object.email or ''">mark.brown23@example.com</a></b><br /><br />
                        Never heard of Odoo? It’s an all-in-one business software loved by 7+ million users. It will considerably improve your experience at work and increase your productivity.
                        <br /><br />
                        Have a look at the <a href="https://www.odoo.com/page/tour?utm_source=db&amp;utm_medium=auth" style="color: #875A7B;">Odoo Tour</a> to discover the tool.
                        <br /><br />
                        Enjoy Odoo!<br />
                        --<br/>The <t t-out="object.company_id.name or ''">YourCompany</t> Team
                    </div>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; font-size: 11px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle" align="left">
                    <t t-out="object.company_id.name or ''">YourCompany</t>
                </td></tr>
                <tr><td valign="middle" align="left" style="opacity: 0.7;">
                    <t t-out="object.company_id.phone or ''">+1 650-123-4567</t>
                    <t t-if="object.company_id.email">
                        | <a t-att-href="'mailto:%s' % object.company_id.email" style="text-decoration:none; color: #454748;" t-out="object.company_id.email or ''">info@yourcompany.com</a>
                    </t>
                    <t t-if="object.company_id.website">
                        | <a t-att-href="'%s' % object.company_id.website" style="text-decoration:none; color: #454748;" t-out="object.company_id.website or ''">http://www.example.com</a>
                    </t>
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 13px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=auth" style="color: #875A7B;">Odoo</a>
      </td></tr>
    </table>
</td></tr>
</table></field>
            <field name="lang">{{ object.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- Email template for reminder of unregistered users -->
        <record id="mail_template_data_unregistered_users" model="mail.template">
            <field name="name">Settings: Unregistered User Reminder</field>
            <field name="model_id" ref="base.model_res_users"/>
            <field name="subject">Reminder for unregistered users</field>
            <field name="email_from">{{ (object.company_id.email_formatted or user.email_formatted) }}</field>
            <field name="email_to">{{ object.email_formatted }}</field>
            <field name="description">Sent automatically to admin if new user haven't responded to the invitation</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="background-color: #FFFFFF; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: #FFFFFF; color: #454748; border-collapse:separate;">
<tbody>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <t t-set="invited_users" t-value="ctx.get('invited_users', [])" />
                <td style="text-align : left">
                    <span style="font-size: 20px; font-weight: bold;">
                        Pending Invitations
                    </span><br/><br/>
                </td>
                <tr><td valign="top" style="font-size: 13px;">
                    <div>
                        Dear <t t-out="object.name or ''">Mitchell Admin</t>,<br/> <br/>
                        You added the following user(s) to your database but they haven't registered yet:
                        <ul>
                            <t t-foreach="invited_users" t-as="invited_user">
                                <li t-out="invited_user or ''">demo@example.com</li>
                            </t>
                        </ul>
                        Follow up with them so they can access your database and start working with you.
                        <br /><br/>
                        Have a nice day!<br />
                        --<br/>The <t t-out="object.company_id.name or ''">YourCompany</t> Team
                    </div>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
</table>
            </field>
            <field name="lang">{{ object.partner_id.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

        <!-- Email template for new users that used a signup token -->
        <record id="mail_template_user_signup_account_created" model="mail.template">
            <field name="name">Settings: New Portal Sign Up</field>
            <field name="model_id" ref="base.model_res_users"/>
            <field name="subject">Welcome to {{ object.company_id.name }}!</field>
            <field name="email_from">{{ (object.company_id.email_formatted or user.email_formatted) }}</field>
            <field name="email_to">{{ object.email_formatted }}</field>
            <field name="description">Sent to portal user who registered themselves</field>
            <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #FFFFFF; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: #FFFFFF; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Your Account</span><br/>
                    <span style="font-size: 20px; font-weight: bold;">
                        <t t-out="object.name or ''">Marc Demo</t>
                    </span>
                </td><td valign="middle" align="right" t-if="not object.company_id.uses_default_logo">
                    <img t-attf-src="/logo.png?company={{ object.company_id.id }}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" t-att-alt="object.company_id.name"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="top" style="font-size: 13px;">
                    <div>
                        Dear <t t-out="object.name or ''">Marc Demo</t>,<br/><br/>
                        Your account has been successfully created!<br/>
                        Your login is <strong><t t-out="object.email or ''">mark.brown23@example.com</t></strong><br/>
                        To gain access to your account, you can use the following link:
                        <div style="margin: 16px 0px 16px 0px;">
                            <a t-attf-href="/web/login?auth_login={{object.email}}"
                                style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                                Go to My Account
                            </a>
                        </div>
                        Thanks,<br/>
                        <t t-if="user.signature">
                            <br/>
                            <t t-out="user.signature or ''">--<br/>Mitchell Admin</t>
                        </t>
                    </div>
                </td></tr>
                <tr><td style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; font-size: 11px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle" align="left">
                    <t t-out="object.company_id.name or ''">YourCompany</t>
                </td></tr>
                <tr><td valign="middle" align="left" style="opacity: 0.7;">
                    <t t-out="object.company_id.phone or ''">+1 650-123-4567</t>
                    <t t-if="object.company_id.email">
                        | <a t-attf-href="'mailto:%s' % {{ object.company_id.email }}" style="text-decoration:none; color: #454748;"><t t-out="object.company_id.email or ''">info@yourcompany.com</t></a>
                    </t>
                    <t t-if="object.company_id.website">
                        | <a t-attf-href="'%s' % {{ object.company_id.website }}" style="text-decoration:none; color: #454748;">
                            <t t-out="object.company_id.website or ''">http://www.example.com</t>
                        </a>
                    </t>
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 13px;">
        Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=auth" style="color: #875A7B;">Odoo</a>
      </td></tr>
    </table>
</td></tr>
</table></field>
            <field name="lang">{{ object.lang }}</field>
            <field name="auto_delete" eval="True"/>
        </record>

    </data>
</odoo>

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.http import request


class Http(models.AbstractModel):
    _inherit = 'ir.http'

    @classmethod
    def _pre_dispatch(cls, rule, args):
        super()._pre_dispatch(rule, args)

        # add signup token or login to the session if given
        for key in ('auth_signup_token', 'auth_login'):
            val = request.httprequest.args.get(key)
            if val is not None:
                request.session[key] = val

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    auth_signup_reset_password = fields.Boolean(
        string='Enable password reset from Login page',
        config_parameter='auth_signup.reset_password')
    auth_signup_uninvited = fields.Selection(
        selection=[
            ('b2b', 'On invitation'),
            ('b2c', 'Free sign up'),
        ],
        string='Customer Account',
        default='b2c',
        config_parameter='auth_signup.invitation_scope')
    auth_signup_template_user_id = fields.Many2one(
        'res.users',
        string='Template user for new users created through signup',
        config_parameter='base.template_portal_user_id')

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import random
import werkzeug.urls

from collections import defaultdict
from datetime import datetime, timedelta

from odoo import api, exceptions, fields, models, tools, _

class SignupError(Exception):
    pass

def random_token():
    # the token has an entropy of about 120 bits (6 bits/char * 20 chars)
    chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
    return ''.join(random.SystemRandom().choice(chars) for _ in range(20))

def now(**kwargs):
    return datetime.now() + timedelta(**kwargs)


class ResPartner(models.Model):
    _inherit = 'res.partner'

    signup_type = fields.Char(string='Signup Token Type', copy=False, groups="base.group_erp_manager")

    def _get_signup_url(self):
        self.ensure_one()
        result = self.sudo()._get_signup_url_for_action()
        if any(u._is_internal() for u in self.user_ids if u != self.env.user):
            self.env['res.users'].check_access('write')
        if any(u._is_portal() for u in self.user_ids if u != self.env.user):
            self.env['res.partner'].check_access('write')
        return result.get(self.id, False)

    def _get_signup_url_for_action(self, url=None, action=None, view_type=None, menu_id=None, res_id=None, model=None):
        """ generate a signup url for the given partner ids and action, possibly overriding
            the url state components (menu_id, id, view_type) """

        res = dict.fromkeys(self.ids, False)
        for partner in self:
            base_url = partner.get_base_url()
            # when required, make sure the partner has a valid signup token
            if self.env.context.get('signup_valid') and not partner.user_ids:
                partner.sudo().signup_prepare()

            route = 'login'
            # the parameters to encode for the query
            query = {'db': self.env.cr.dbname}
            if self.env.context.get('create_user'):
                query['signup_email'] = partner.email

            signup_type = self.env.context.get('signup_force_type_in_url', partner.sudo().signup_type or '')
            if signup_type:
                route = 'reset_password' if signup_type == 'reset' else signup_type

            query['token'] = partner.sudo()._generate_signup_token()

            if url:
                query['redirect'] = url
            else:
                fragment = dict()
                base = '/odoo/'
                if action == '/mail/view':
                    base = '/mail/view?'
                elif action:
                    fragment['action'] = action
                if view_type:
                    fragment['view_type'] = view_type
                if menu_id:
                    fragment['menu_id'] = menu_id
                if model:
                    fragment['model'] = model
                if res_id:
                    fragment['res_id'] = res_id

                if fragment:
                    query['redirect'] = base + werkzeug.urls.url_encode(fragment)

            signup_url = "/web/%s?%s" % (route, werkzeug.urls.url_encode(query))
            if not self.env.context.get('relative_url'):
                signup_url = werkzeug.urls.url_join(base_url, signup_url)
            res[partner.id] = signup_url
        return res

    def action_signup_prepare(self):
        return self.signup_prepare()

    def signup_get_auth_param(self):
        """ Get a signup token related to the partner if signup is enabled.
            If the partner already has a user, get the login parameter.
        """
        if not self.env.user._is_internal() and not self.env.is_admin():
            raise exceptions.AccessDenied()

        res = defaultdict(dict)

        allow_signup = self.env['res.users']._get_signup_invitation_scope() == 'b2c'
        for partner in self:
            partner = partner.sudo()
            if allow_signup and not partner.user_ids:
                partner.signup_prepare()
                res[partner.id]['auth_signup_token'] = partner._generate_signup_token()
            elif partner.user_ids:
                res[partner.id]['auth_login'] = partner.user_ids[0].login
        return res

    def signup_cancel(self):
        return self.write({'signup_type': None})

    def signup_prepare(self, signup_type="signup"):
        """ generate a new token for the partners with the given validity, if necessary
            :param expiration: the expiration datetime of the token (string, optional)
        """
        self.write({'signup_type': signup_type})
        return True

    @api.model
    def _signup_retrieve_partner(self, token, check_validity=False, raise_exception=False):
        """ find the partner corresponding to a token, and possibly check its validity
            :param token: the token to resolve
            :param check_validity: if True, also check validity
            :param raise_exception: if True, raise exception instead of returning False
            :return: partner (browse record) or False (if raise_exception is False)
        """
        partner = self._get_partner_from_token(token)
        if not partner:
            raise exceptions.UserError(_("Signup token '%s' is not valid or expired", token))
        return partner

    @api.model
    def _signup_retrieve_info(self, token):
        """ retrieve the user info about the token
            :return: a dictionary with the user information if the token is valid, None otherwise:
                - 'db': the name of the database
                - 'token': the token, if token is valid
                - 'name': the name of the partner, if token is valid
                - 'login': the user login, if the user already exists
                - 'email': the partner email, if the user does not exist
        """
        partner = self._get_partner_from_token(token)
        if not partner:
            return None
        res = {'db': self.env.cr.dbname}
        res['token'] = token
        res['name'] = partner.name
        if partner.user_ids:
            res['login'] = partner.user_ids[0].login
        else:
            res['email'] = res['login'] = partner.email or ''
        return res

    def _get_login_date(self):
        self.ensure_one()
        users_login_dates = self.user_ids.mapped('login_date')
        users_login_dates = list(filter(None, users_login_dates))  # remove falsy values
        if any(users_login_dates):
            return int(max(map(datetime.timestamp, users_login_dates)))
        return None

    def _generate_signup_token(self, expiration=None):
        """ This function generate the signup token for the partner in self.
            pre-condition: self.signup_type must be either 'signup' or 'reset'
            :return: the signed payload/token that can be used to reset the password/signup.
                - 'expiration': the time in hours before the expiration of the token
        Since the last_login_date is part of the payload, this token is invalidated as soon as the user logs in
        """
        self.ensure_one()
        if not expiration:
            if self.signup_type == 'reset':
                expiration = int(self.env['ir.config_parameter'].get_param("auth_signup.reset_password.validity.hours", 4))
            else:
                expiration = int(self.env['ir.config_parameter'].get_param("auth_signup.signup.validity.hours", 144))
        plist = [self.id, self.user_ids.ids, self._get_login_date(), self.signup_type]
        payload = tools.hash_sign(self.sudo().env, 'signup', plist, expiration_hours=expiration)
        return payload

    @api.model
    def _get_partner_from_token(self, token):
        if payload := tools.verify_hash_signed(self.sudo().env, 'signup', token):
            partner_id, user_ids, login_date, signup_type = payload
            # login_date can be either an int or "None" as a string for signup
            partner = self.browse(partner_id)
            if login_date == partner._get_login_date() and partner.user_ids.ids == user_ids and signup_type == partner.browse(partner_id).signup_type:
                return partner
        return None

```

## File: models\res_users.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import contextlib
import logging

from ast import literal_eval
from dateutil.relativedelta import relativedelta

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv import expression
from odoo.http import request

from odoo.addons.base.models.ir_mail_server import MailDeliveryException
from odoo.addons.auth_signup.models.res_partner import SignupError

_logger = logging.getLogger(__name__)

class ResUsers(models.Model):
    _inherit = 'res.users'

    state = fields.Selection(compute='_compute_state', search='_search_state', string='Status',
                 selection=[('new', 'Never Connected'), ('active', 'Confirmed')])

    def _search_state(self, operator, value):
        negative = operator in expression.NEGATIVE_TERM_OPERATORS

        # In case we have no value
        if not value:
            return expression.TRUE_DOMAIN if negative else expression.FALSE_DOMAIN

        if operator in ['in', 'not in']:
            if len(value) > 1:
                return expression.FALSE_DOMAIN if negative else expression.TRUE_DOMAIN
            if value[0] == 'new':
                comp = '!=' if negative else '='
            if value[0] == 'active':
                comp = '=' if negative else '!='
            return [('log_ids', comp, False)]

        if operator in ['=', '!=']:
            # In case we search against anything else than new, we have to invert the operator
            if value != 'new':
                operator = expression.TERM_OPERATORS_NEGATION[operator]

            return [('log_ids', operator, False)]

        return expression.TRUE_DOMAIN

    def _compute_state(self):
        for user in self:
            user.state = 'active' if user.login_date else 'new'

    @api.model
    def signup(self, values, token=None):
        """ signup a user, to either:
            - create a new user (no token), or
            - create a user for a partner (with token, but no user for partner), or
            - change the password of a user (with token, and existing user).
            :param values: a dictionary with field values that are written on user
            :param token: signup token (optional)
            :return: (dbname, login, password) for the signed up user
        """
        if token:
            # signup with a token: find the corresponding partner id
            partner = self.env['res.partner']._signup_retrieve_partner(token, check_validity=True, raise_exception=True)
            # invalidate signup token
            partner.write({'signup_type': False})
            partner_user = partner.user_ids and partner.user_ids[0] or False

            # avoid overwriting existing (presumably correct) values with geolocation data
            if partner.country_id or partner.zip or partner.city:
                values.pop('city', None)
                values.pop('country_id', None)
            if partner.lang:
                values.pop('lang', None)

            if partner_user:
                # user exists, modify it according to values
                values.pop('login', None)
                values.pop('name', None)
                partner_user.write(values)
                if not partner_user.login_date:
                    partner_user._notify_inviter()
                return (partner_user.login, values.get('password'))
            else:
                # user does not exist: sign up invited user
                values.update({
                    'name': partner.name,
                    'partner_id': partner.id,
                    'email': values.get('email') or values.get('login'),
                })
                if partner.company_id:
                    values['company_id'] = partner.company_id.id
                    values['company_ids'] = [(6, 0, [partner.company_id.id])]
                partner_user = self._signup_create_user(values)
                partner_user._notify_inviter()
        else:
            # no token, sign up an external user
            values['email'] = values.get('email') or values.get('login')
            self._signup_create_user(values)

        return (values.get('login'), values.get('password'))

    @api.model
    def _get_signup_invitation_scope(self):
        return self.env['ir.config_parameter'].sudo().get_param('auth_signup.invitation_scope', 'b2b')

    @api.model
    def _signup_create_user(self, values):
        """ signup a new user using the template user """

        # check that uninvited users may sign up
        if 'partner_id' not in values:
            if self._get_signup_invitation_scope() != 'b2c':
                raise SignupError(_('Signup is not allowed for uninvited users'))
        return self._create_user_from_template(values)

    @classmethod
    def authenticate(cls, db, credential, user_agent_env):
        auth_info = super().authenticate(db, credential, user_agent_env)
        try:
            with cls.pool.cursor() as cr:
                env = api.Environment(cr, auth_info['uid'], {})
                if env.user._should_alert_new_device():
                    env.user._alert_new_device()
        except MailDeliveryException:
            pass
        return auth_info

    def _notify_inviter(self):
        for user in self:
            # notify invite user that new user is connected
            user.create_uid._bus_send(
                "res.users/connection", {"username": user.name, "partnerId": user.partner_id.id}
            )

    def _create_user_from_template(self, values):
        template_user_id = literal_eval(self.env['ir.config_parameter'].sudo().get_param('base.template_portal_user_id', 'False'))
        template_user = self.browse(template_user_id)
        if not template_user.exists():
            raise ValueError(_('Signup: invalid template user'))

        if not values.get('login'):
            raise ValueError(_('Signup: no login given for new user'))
        if not values.get('partner_id') and not values.get('name'):
            raise ValueError(_('Signup: no name or partner given for new user'))

        # create a copy of the template user (attached to a specific partner_id if given)
        values['active'] = True
        try:
            with self.env.cr.savepoint():
                return template_user.with_context(no_reset_password=True).copy(values)
        except Exception as e:
            # copy may failed if asked login is not available.
            raise SignupError(str(e))

    def reset_password(self, login):
        """ retrieve the user corresponding to login (login or email),
            and reset their password
        """
        users = self.search(self._get_login_domain(login))
        if not users:
            users = self.search(self._get_email_domain(login))
        if not users:
            raise Exception(_('No account found for this login'))
        if len(users) > 1:
            raise Exception(_('Multiple accounts found for this login'))
        return users.action_reset_password()

    def action_reset_password(self):
        try:
            if self.env.context.get('create_user') == 1:
                return self._action_reset_password(signup_type="signup")
            else:
                return self._action_reset_password(signup_type="reset")
        except MailDeliveryException as mde:
            if len(mde.args) == 2 and isinstance(mde.args[1], ConnectionRefusedError):
                raise UserError(_("Could not contact the mail server, please check your outgoing email server configuration")) from mde
            else:
                raise UserError(_("There was an error when trying to deliver your Email, please check your configuration")) from mde

    def _action_reset_password(self, signup_type="reset"):
        """ create signup token for each user, and send their signup url by email """
        if self.env.context.get('install_mode') or self.env.context.get('import_file'):
            return
        if self.filtered(lambda user: not user.active):
            raise UserError(_("You cannot perform this action on an archived user."))
        # prepare reset password signup
        create_mode = bool(self.env.context.get('create_user'))

        self.mapped('partner_id').signup_prepare(signup_type=signup_type)

        # send email to users with their signup url
        account_created_template = None
        if create_mode:
            account_created_template = self.env.ref('auth_signup.set_password_email', raise_if_not_found=False)
            if account_created_template and account_created_template._name != 'mail.template':
                _logger.error("Wrong set password template %r", account_created_template)
                return

        email_values = {
            'email_cc': False,
            'auto_delete': True,
            'message_type': 'user_notification',
            'recipient_ids': [],
            'partner_ids': [],
            'scheduled_date': False,
        }

        for user in self:
            if not user.email:
                raise UserError(_("Cannot send email: user %s has no email address.", user.name))
            email_values['email_to'] = user.email
            with contextlib.closing(self.env.cr.savepoint()):
                if account_created_template:
                    account_created_template.send_mail(
                        user.id, force_send=True,
                        raise_exception=True, email_values=email_values)
                else:
                    user_lang = user.lang or self.env.lang or 'en_US'
                    body = self.env['mail.render.mixin'].with_context(lang=user_lang)._render_template(
                        self.env.ref('auth_signup.reset_password_email'),
                        model='res.users', res_ids=user.ids,
                        engine='qweb_view', options={'post_process': True})[user.id]
                    mail = self.env['mail.mail'].sudo().create({
                        'subject': self.with_context(lang=user_lang).env._('Password reset'),
                        'email_from': user.company_id.email_formatted or user.email_formatted,
                        'body_html': body,
                        **email_values,
                    })
                    mail.send()
            if signup_type == 'reset':
                _logger.info("Password reset email sent for user <%s> to <%s>", user.login, user.email)
                message = _('A reset password link was send by email')
            else:
                _logger.info("Signup email sent for user <%s> to <%s>", user.login, user.email)
                message = _('A signup link was send by email')
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'title': 'Notification',
                'message': message,
                'sticky': False
            }
        }

    def send_unregistered_user_reminder(self, *, after_days=5, batch_size=100):
        email_template = self.env.ref('auth_signup.mail_template_data_unregistered_users', raise_if_not_found=False)
        if not email_template:
            _logger.warning("Template 'auth_signup.mail_template_data_unregistered_users' was not found. Cannot send reminder notifications.")
            self.env['ir.cron']._commit_progress(deactivate=True)
            return
        datetime_min = fields.Datetime.today() - relativedelta(days=after_days)
        datetime_max = datetime_min + relativedelta(days=1)

        invited_by_users = self.search_fetch([
            ('share', '=', False),
            ('create_uid.email', '!=', False),
            ('create_date', '>=', datetime_min),
            ('create_date', '<', datetime_max),
            ('log_ids', '=', False),
        ], ['name', 'login', 'create_uid']).grouped('create_uid')

        # Do not use progress since we have no way of knowing to whom we have
        # already sent e-mails.

        done = 0
        for user, invited_users in invited_by_users.items():
            invited_user_emails = [f"{u.name} ({u.login})" for u in invited_users]
            template = email_template.with_context(dbname=self.env.cr.dbname, invited_users=invited_user_emails)
            template.send_mail(user.id, email_layout_xmlid='mail.mail_notification_light', force_send=False)
            done += len(invited_users)
            # do not set remaining and the search will return always the same users!
            self.env['ir.cron']._notify_progress(done=done, remaining=0)
            self.env.cr.commit()

    def _alert_new_device(self):
        self.ensure_one()
        if self.email:
            email_values = {
                'email_cc': False,
                'auto_delete': True,
                'message_type': 'user_notification',
                'recipient_ids': [],
                'partner_ids': [],
                'scheduled_date': False,
                'email_to': self.email
            }

            body = self.env['mail.render.mixin']._render_template(
                    'auth_signup.alert_login_new_device',
                    model='res.users', res_ids=self.ids,
                    engine='qweb_view', options={'post_process': True},
                    add_context=self._prepare_new_device_notice_values())[self.id]
            mail = self.env['mail.mail'].sudo().create({
                'subject': _('New Connection to your Account'),
                'email_from': self.company_id.email_formatted or self.email_formatted,
                'body_html': body,
                **email_values,
            })
            mail.send()
            _logger.info("New device alert email sent for user <%s> to <%s>", self.login, self.email)

    def _prepare_new_device_notice_values(self):
        values = {
            'login_date': fields.Datetime.now(),
            'location_address': False,
            'ip_address': False,
            'browser': False,
            'useros': False,
        }

        if not request:
            return values

        city = request.geoip.get('city') or False
        region = request.geoip.get('region_name') or False
        country = request.geoip.get('country') or False
        if country:
            if region and city:
                values['location_address'] = _("Near %(city)s, %(region)s, %(country)s", city=city, region=region, country=country)
            elif region:
                values['location_address'] = _("Near %(region)s, %(country)s", region=region, country=country)
            else:
                values['location_address'] = _("In %(country)s", country=country)
        else:
            values['location_address'] = False
        values['ip_address'] = request.httprequest.environ['REMOTE_ADDR']
        if request.httprequest.user_agent:
            if request.httprequest.user_agent.browser:
                values['browser'] = request.httprequest.user_agent.browser.capitalize()
            if request.httprequest.user_agent.platform:
                values['useros'] = request.httprequest.user_agent.platform.capitalize()
        return values

    @api.model
    def web_create_users(self, emails):
        inactive_users = self.search([('state', '=', 'new'), '|', ('login', 'in', emails), ('email', 'in', emails)])
        new_emails = set(emails) - set(inactive_users.mapped('email'))
        res = super(ResUsers, self).web_create_users(list(new_emails))
        if inactive_users:
            inactive_users.with_context(create_user=True).action_reset_password()
        return res

    @api.model_create_multi
    def create(self, vals_list):
        # overridden to automatically invite user to sign up
        users = super(ResUsers, self).create(vals_list)
        if not self.env.context.get('no_reset_password'):
            users_with_email = users.filtered('email')
            if users_with_email:
                try:
                    users_with_email.with_context(create_user=True)._action_reset_password(signup_type='signup')
                except MailDeliveryException:
                    users_with_email.partner_id.with_context(create_user=True).signup_cancel()
        return users

    def copy(self, default=None):
        if not default or not default.get('email'):
            # avoid sending email to the user we are duplicating
            self = self.with_context(no_reset_password=True)
        return super().copy(default=default)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import res_config_settings
from . import ir_http
from . import res_partner
from . import res_users

```

## File: static\src\js\reset_password.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.ResetPasswordForm = publicWidget.Widget.extend({
    selector: '.oe_reset_password_form',
    events: {
        'submit': '_onSubmit',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onSubmit: function () {
        var $btn = this.$('.oe_login_buttons > button[type="submit"]');
        if ($btn.prop("disabled")) {
            return;
        }
        $btn.attr('disabled', 'disabled');
        $btn.prepend('<i class="fa fa-refresh fa-spin"/> ');
    },
});

```

## File: static\src\js\signup.js

```javascript
/** @odoo-module **/

import publicWidget from "@web/legacy/js/public/public_widget";

publicWidget.registry.SignUpForm = publicWidget.Widget.extend({
    selector: '.oe_signup_form',
    events: {
        'submit': '_onSubmit',
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onSubmit: function () {
        var $btn = this.$('.oe_login_buttons > button[type="submit"]');
        if ($btn.prop("disabled")) {
            return;
        }
        $btn.attr('disabled', 'disabled');
        $btn.prepend('<i class="fa fa-refresh fa-spin"/> ');
    },
});

```

## File: views\auth_signup_login_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <template id="auth_signup.login" inherit_id="web.login" name="Sign up - Reset Password">
            <xpath expr="//div[hasclass('oe_login_buttons')]" position="after">
                <p class="mt-2 text-center">
                    <a t-if="signup_enabled" class="btn btn-link btn-sm" t-attf-href="/web/signup?{{ keep_query() }}">Don't have an account?</a>
                </p>
            </xpath>
            <xpath expr="//label[@for='password']" position="inside">
                <a t-if="reset_password_enabled" class="btn btn-link btn-sm" tabindex="1" t-attf-href="/web/reset_password?{{ keep_query() }}">Reset Password</a>
            </xpath>
        </template>

        <template id="auth_signup.fields" name="Auth Signup/ResetPassword form fields">

            <div class="mb-3 field-login">
                <label for="login">Your Email</label>
                <input type="text" name="login" t-att-value="login" id="login" class="form-control form-control-sm" autofocus="autofocus"
                    autocapitalize="off" required="required" t-att-readonly="'readonly' if only_passwords else None"/>
            </div>

            <div class="mb-3 field-name">
                <label for="name">Your Name</label>
                <input type="text" name="name" t-att-value="name" id="name" class="form-control form-control-sm" placeholder="e.g. John Doe"
                    required="required" t-att-readonly="'readonly' if only_passwords else None"
                    t-att-autofocus="'autofocus' if login and not only_passwords else None" />
            </div>

            <div class="mb-3 field-password pt-2">
                <label for="password">Password</label>
                <input type="password" name="password" id="password" class="form-control form-control-sm"
                    required="required" t-att-autofocus="'autofocus' if only_passwords else None"/>
            </div>

            <div class="mb-3 field-confirm_password">
                <label for="confirm_password">Confirm Password</label>
                <input type="password" name="confirm_password" id="confirm_password" class="form-control form-control-sm" required="required"/>
            </div>
        </template>

        <template id="auth_signup.signup" name="Sign up login">
            <t t-call="web.login_layout">
                <form class="oe_signup_form" role="form" method="post" t-if="not message">
                  <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>

                    <t t-call="auth_signup.fields">
                        <t t-set="only_passwords" t-value="bool(token and not invalid_token)"/>
                    </t>

                    <p class="alert alert-danger" t-if="error" role="alert">
                        <t t-esc="error"/>
                    </p>
                    <input type="hidden" name="redirect" t-att-value="redirect"/>
                    <input type="hidden" name="token" t-att-value="token"/>
                    <div class="text-center oe_login_buttons d-grid pt-3">
                        <button type="submit" class="btn btn-primary"> Sign up</button>
                        <a t-attf-href="/web/login?{{ keep_query() }}" class="btn btn-link btn-sm" role="button">Already have an account?</a>
                        <div class="o_login_auth"/>
                    </div>
                </form>
            </t>
        </template>

        <template id="auth_signup.reset_password" name="Reset password">
            <t t-call="web.login_layout">
                <div t-if="message" class="oe_login_form clearfix">
                    <p class="alert alert-success" t-if="message" role="status">
                        <t t-esc="message"/>
                    </p>
                    <a href="/web/login" class="btn btn-link btn-sm float-start" role="button">Back to Login</a>
                </div>

                <form class="oe_reset_password_form" role="form" method="post" t-if="not message">
                  <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>

                    <t t-if="token and not invalid_token">
                        <t t-call="auth_signup.fields">
                            <t t-set="only_passwords" t-value="1"/>
                        </t>
                    </t>

                    <t t-if="not token">
                        <div class="mb-3 field-login">
                            <label for="login" class="col-form-label">Your Email</label>
                            <input type="text" name="login" t-att-value="login" id="login" class="form-control"
                                autofocus="autofocus" required="required" autocapitalize="off"/>
                        </div>
                    </t>

                    <p class="alert alert-danger" t-if="error" role="alert">
                        <t t-esc="error"/>
                    </p>
                    <input type="hidden" name="redirect" t-att-value="redirect"/>
                    <input type="hidden" name="token" t-att-value="token"/>
                    <div class="clearfix oe_login_buttons d-grid mt-3">
                        <button type="submit" class="btn btn-primary">Reset Password</button>
                        <div class="d-flex justify-content-between align-items-center small mt-2">
                            <a t-if="not token" t-attf-href="/web/login?{{ keep_query() }}">Back to Login</a>
                            <a t-if="invalid_token" href="/web/login">Back to Login</a>
                        </div>
                        <div class="o_login_auth"/>
                    </div>

                </form>

            </t>
        </template>
</odoo>

```

## File: views\auth_signup_templates_email.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="reset_password_email" name="User Reset Password">
        <table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #FFFFFF; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
        <table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: #FFFFFF; color: #454748; border-collapse:separate;">
        <tbody>
            <!-- HEADER -->
            <tr>
                <td align="center" style="min-width: 590px;">
                    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                        <tr><td valign="middle">
                            <span style="font-size: 10px;">Your Account</span><br/>
                            <span style="font-size: 20px; font-weight: bold;">
                                <t t-out="object.name or ''">Marc Demo</t>
                            </span>
                        </td><td valign="middle" align="right" t-if="not object.company_id.uses_default_logo">
                            <img t-attf-src="/logo.png?company={{ object.company_id.id }}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" t-att-alt="object.company_id.name"/>
                        </td></tr>
                        <tr><td colspan="2" style="text-align:center;">
                          <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                        </td></tr>
                    </table>
                </td>
            </tr>
            <!-- CONTENT -->
            <tr>
                <td align="center" style="min-width: 590px;">
                    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                        <tr><td valign="top" style="font-size: 13px;">
                            <div>
                                Dear <t t-out="object.name or ''">Marc Demo</t>,<br/><br/>
                                A password reset was requested for the Odoo account linked to this email.
                                You may change your password by following this link which will remain valid during <t t-out="user.env['ir.config_parameter'].sudo().get_param('auth_signup.reset_password.validity.hours',4)"></t> hours:<br/>
                                <div style="margin: 16px 0px 16px 0px;">
                                    <a t-att-href="object.partner_id._get_signup_url()"
                                        style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                                        Change password
                                    </a>
                                </div>
                                If you do not expect this, you can safely ignore this email.<br/><br/>
                                Thanks,
                                <t t-if="user.signature">
                                    <br/>
                                    <t t-out="user.signature">--<br/>Mitchell Admin</t>
                                </t>
                            </div>
                        </td></tr>
                        <tr><td style="text-align:center;">
                          <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                        </td></tr>
                    </table>
                </td>
            </tr>
            <!-- FOOTER -->
            <tr>
                <td align="center" style="min-width: 590px;">
                    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; font-size: 11px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                        <tr><td valign="middle" align="left">
                            <t t-out="object.company_id.name or ''">YourCompany</t>
                        </td></tr>
                        <tr><td valign="middle" align="left" style="opacity: 0.7;">
                            <t t-out="object.company_id.phone or ''">+1 650-123-4567</t>

                            <t t-if="object.company_id.email">
                                | <a t-att-href="'mailto:%s' % object.company_id.email" style="text-decoration:none; color: #454748;" t-out="object.company_id.email">info@yourcompany.com</a>
                            </t>
                            <t t-if="object.company_id.website">
                                | <a t-att-href="'%s' % object.company_id.website" style="text-decoration:none; color: #454748;" t-out="object.company_id.website">http://www.example.com</a>
                            </t>
                        </td></tr>
                    </table>
                </td>
            </tr>
        </tbody>
        </table>
        </td></tr>
        <!-- POWERED BY -->
        <tr><td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
              <tr><td style="text-align: center; font-size: 13px;">
                Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=auth" style="color: #875A7B;">Odoo</a>
              </td></tr>
            </table>
        </td></tr>
        </table>
    </template>

    <template id="alert_login_new_device" name="Alert Login with new Device">
        <table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #FFFFFF; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
        <table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: #FFFFFF; color: #454748; border-collapse:separate;">
        <tbody>
            <!-- HEADER -->
            <tr>
                <td align="center" style="min-width: 590px;">
                    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                    <tr><td valign="top" style="font-size: 13px;">
                    <div>
                        Dear <t t-out="object.name or ''">Marc Demo</t>,<br/><br/>
                        A new device was used to sign in to your account. <br/><br/>
                        Here are some details about the connection:<br/>
                        <ul>
                            <li><span style="font-weight: bold;">
                                Date:</span> <t t-out="format_datetime(login_date, dt_format='long')">day, month dd, yyyy - hh:mm:ss (GMT)</t></li>
                            <t t-if="location_address">
                                <li><span style="font-weight: bold;">
                                    Location:</span> <t t-out="location_address">City, Region, Country</t></li>
                            </t>
                            <t t-if="useros">
                                <li><span style="font-weight: bold;">
                                    Platform:</span> <t t-out="useros">OS</t></li>
                            </t>
                            <t t-if="browser">
                                <li><span style="font-weight: bold;">
                                    Browser:</span> <t t-out="browser">Browser</t></li>
                            </t>
                            <li><span style="font-weight: bold;">
                                IP Address:</span> <t t-out="ip_address">111.222.333.444</t></li>
                        </ul>
                        If you don't recognize it, you should change your password immediately via this link:<br/>
                        <div style="margin: 16px 0px 16px 0px;">
                            <a t-attf-href="{{ object.get_base_url() }}/web/reset_password"
                                style="background-color: #875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px; font-size:13px;">
                                Reset Password
                            </a>
                        </div>
                        Otherwise, you can safely ignore this email.

                    </div>
                    </td></tr>
                    <tr><td style="text-align:center;">
                      <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    </td></tr>
                    </table>
                </td>
            </tr>
        </tbody>
        </table>
        </td></tr>
        <!-- POWERED BY -->
        <tr><td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
              <tr><td style="text-align: center; font-size: 13px;">
                Powered by <a target="_blank" href="https://www.odoo.com?utm_source=db&amp;utm_medium=auth" style="color: #875A7B;">Odoo</a>
              </td></tr>
            </table>
        </td></tr>
        </table>
    </template>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.auth.signup</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//setting[@id='access_rights']" position="before">
                    <setting id="login_documents" title="To send invitations in B2B mode, open a contact or select several ones in list view and click on 'Portal Access Management' option in the dropdown menu *Action*." help="Let your customers log in to see their documents">
                        <field name="auth_signup_uninvited" class="o_light_label" widget="radio" options="{'horizontal': true}" required="True"/>
                        <div class="content-group" invisible="auth_signup_uninvited == 'b2b'">
                            <div class="mt8">
                                <button type="object" name="action_open_template_user" string="Default Access Rights" icon="oi-arrow-right" class="btn-link"/>
                            </div>
                        </div>
                    </setting>
                    <setting string="Password Reset" help="Enable password reset from Login page" id="enable_password_reset">
                        <field name="auth_signup_reset_password"/>
                    </setting>
                </xpath>
            </field>
        </record>

</odoo>

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="res_users_view_form" model="ir.ui.view">
            <field name="name">res.users.form.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_form"/>
            <field name="arch" type="xml">
                <!-- add state field in header -->
                <xpath expr="//header" position="inside">
                    <button string="Send Password Reset Instructions"
                                type="object" name="action_reset_password"
                                invisible="state != 'active'"/>
                    <button string="Send an Invitation Email"
                                type="object" name="action_reset_password" context="{'create_user': 1}"
                                invisible="state != 'new'"/>
                    <field name="state" widget="statusbar"/>
                </xpath>
            </field>
        </record>

        <record id="view_users_state_tree" model="ir.ui.view">
            <field name="name">res.users.list.inherit</field>
            <field name="model">res.users</field>
            <field name="inherit_id" ref="base.view_users_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='company_id']" position="after">
                    <field name="state" widget="badge"
                        decoration-info="state == 'new'" decoration-success="state == 'active'"/>
                </xpath>
            </field>
        </record>

        <record id="action_send_password_reset_instructions" model="ir.actions.server">
            <field name="name">Send Password Reset Instructions</field>
            <field name="model_id" ref="base.model_res_users"/>
            <field name="groups_id" eval="[(4, ref('base.group_erp_manager'))]"/>
            <field name="binding_model_id" ref="base.model_res_users" />
            <field name="state">code</field>
            <field name="code">records.action_reset_password()</field>
        </record>

</odoo>

```

## File: views\webclient_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="login_successful" inherit_id="web.login_successful">
        <xpath expr="//div[hasclass('oe_login_form')]/p" position="before">
            <p class="alert alert-success" t-if="account_created" role="status">
                Registration successful.
            </p>
            <!-- Remove parameter from URL, do not show "Account created" if page is refreshed -->
            <script defer="defer" type="text/javascript">
                window.history.replaceState({}, null, '/web/login_successful');
            </script>
        </xpath>
    </template>

</odoo>

```

