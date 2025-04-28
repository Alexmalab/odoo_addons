# Odoo Module: base_setup

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models, controllers

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Initial Setup Tools',
    'version': '1.0',
    'category': 'Hidden',
    'description': """
This module helps to configure the system at the installation of a new database.
================================================================================

Shows you a list of applications features to install from.

    """,
    'depends': ['base', 'web'],
    'data': [
        'data/base_setup_data.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
        ],
    'auto_install': True,
    'installable': True,

    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, http
from odoo.exceptions import AccessError
from odoo.http import request


class BaseSetup(http.Controller):
    @http.route('/base_setup/data', type='json', auth='user')
    def base_setup_data(self, **kw):
        if not request.env.user.has_group('base.group_erp_manager'):
            raise AccessError(_("Access Denied"))

        cr = request.cr
        cr.execute("""
            SELECT count(*)
              FROM res_users
             WHERE active=true AND
                   share=false
        """)
        active_count = cr.dictfetchall()[0].get('count')

        cr.execute("""
            SELECT count(u.*)
            FROM res_users u
            WHERE active=true AND
                  share=false AND
                  NOT exists(SELECT 1 FROM res_users_log WHERE create_uid=u.id)
        """)
        pending_count = cr.dictfetchall()[0].get('count')

        cr.execute("""
           SELECT id, login
             FROM res_users u
            WHERE active=true AND
                  share=false AND
                  NOT exists(SELECT 1 FROM res_users_log WHERE create_uid=u.id)
         ORDER BY id desc
            LIMIT 10
        """)
        pending_users = cr.fetchall()
        action_pending_users = request.env['res.users'].browse(
            [uid for (uid, login) in pending_users])._action_show()

        return {
            'active_users': active_count,
            'pending_count': pending_count,
            'pending_users': pending_users,
            'action_pending_users': action_pending_users,
        }

    @http.route('/base_setup/demo_active', type='json', auth='user')
    def base_setup_is_demo(self, **kwargs):
        # We assume that if there's at least one module with demo data active, then the db was
        # initialized with demo=True or it has been force-activated by the `Load demo data` button
        # in the settings dashboard.
        demo_active = bool(request.env['ir.module.module'].search_count([('demo', '=', True)]))
        return demo_active

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
from . import main

```

## File: data\base_setup_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record model="ir.config_parameter" id="show_effect" forcecreate="False">
            <field name="key">base_setup.show_effect</field>
            <field name="value">True</field>
        </record>
    </data>
</odoo>

```

## File: models\ir_http.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        result = super(IrHttp, self).session_info()
        if self.env.user._is_internal():
            result['show_effect'] = bool(self.env['ir.config_parameter'].sudo().get_param('base_setup.show_effect'))
        return result

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError


class ResConfigSettings(models.TransientModel):

    _inherit = 'res.config.settings'

    company_id = fields.Many2one('res.company', string='Company', required=True,
        default=lambda self: self.env.company)
    is_root_company = fields.Boolean(compute='_compute_is_root_company')
    user_default_rights = fields.Boolean(
        "Default Access Rights",
        config_parameter='base_setup.default_user_rights')
    module_base_import = fields.Boolean("Allow users to import data from CSV/XLS/XLSX/ODS files")
    module_google_calendar = fields.Boolean(
        string='Allow the users to synchronize their calendar  with Google Calendar')
    module_microsoft_calendar = fields.Boolean(
        string='Allow the users to synchronize their calendar with Outlook Calendar')
    module_mail_plugin = fields.Boolean(
        string='Allow integration with the mail plugins'
    )
    module_auth_oauth = fields.Boolean("Use external authentication providers (OAuth)")
    module_auth_ldap = fields.Boolean("LDAP Authentication")
    module_account_inter_company_rules = fields.Boolean("Manage Inter Company")
    module_voip = fields.Boolean("Asterisk (VoIP)")
    module_web_unsplash = fields.Boolean("Unsplash Image Library")
    module_partner_autocomplete = fields.Boolean("Partner Autocomplete")
    module_base_geolocalize = fields.Boolean("GeoLocalize")
    module_google_recaptcha = fields.Boolean("reCAPTCHA")
    module_website_cf_turnstile = fields.Boolean("Cloudflare Turnstile")
    report_footer = fields.Html(related="company_id.report_footer", string='Custom Report Footer', help="Footer text displayed at the bottom of all reports.", readonly=False)
    group_multi_currency = fields.Boolean(string='Multi-Currencies',
            implied_group='base.group_multi_currency',
            help="Allows to work in a multi currency environment")
    external_report_layout_id = fields.Many2one(related="company_id.external_report_layout_id")
    show_effect = fields.Boolean(string="Show Effect", config_parameter='base_setup.show_effect')
    company_count = fields.Integer('Number of Companies', compute="_compute_company_count")
    active_user_count = fields.Integer('Number of Active Users', compute="_compute_active_user_count")
    language_count = fields.Integer('Number of Languages', compute="_compute_language_count")
    company_name = fields.Char(related="company_id.display_name", string="Company Name")
    company_informations = fields.Text(compute="_compute_company_informations")
    company_country_code = fields.Char(related="company_id.country_id.code", string="Company Country Code", readonly=True)
    profiling_enabled_until = fields.Datetime("Profiling enabled until", config_parameter='base.profiling_enabled_until')
    module_product_images = fields.Boolean("Get product pictures using barcode")

    def open_company(self):
        return {
            'type': 'ir.actions.act_window',
            'name': 'My Company',
            'view_mode': 'form',
            'res_model': 'res.company',
            'res_id': self.env.company.id,
            'target': 'current',
        }

    def open_default_user(self):
        action = self.env["ir.actions.actions"]._for_xml_id("base.action_res_users")
        if self.env.ref('base.default_user', raise_if_not_found=False):
            action['res_id'] = self.env.ref('base.default_user').id
        else:
            raise UserError(_("Default User Template not found."))
        action['views'] = [[self.env.ref('base.view_users_form').id, 'form']]
        return action

    @api.model
    def _prepare_report_view_action(self, template):
        template_id = self.env.ref(template)
        return {
            'type': 'ir.actions.act_window',
            'res_model': 'ir.ui.view',
            'view_mode': 'form',
            'res_id': template_id.id,
        }

    def edit_external_header(self):
        if not self.external_report_layout_id:
            return False
        return self._prepare_report_view_action(self.external_report_layout_id.key)

    # NOTE: These fields depend on the context, if we want them to be computed
    # we have to make them depend on a field. This is because we are on a TransientModel.
    @api.depends('company_id')
    def _compute_company_count(self):
        company_count = self.env['res.company'].sudo().search_count([])
        for record in self:
            record.company_count = company_count

    @api.depends('company_id')
    def _compute_active_user_count(self):
        active_user_count = self.env['res.users'].sudo().search_count([('share', '=', False)])
        for record in self:
            record.active_user_count = active_user_count

    @api.depends('company_id')
    def _compute_language_count(self):
        language_count = len(self.env['res.lang'].get_installed())
        for record in self:
            record.language_count = language_count

    @api.depends('company_id')
    def _compute_company_informations(self):
        informations = '%s\n' % self.company_id.street if self.company_id.street else ''
        informations += '%s\n' % self.company_id.street2 if self.company_id.street2 else ''
        informations += '%s' % self.company_id.zip if self.company_id.zip else ''
        informations += '\n' if self.company_id.zip and not self.company_id.city else ''
        informations += ' - ' if self.company_id.zip and self.company_id.city else ''
        informations += '%s\n' % self.company_id.city if self.company_id.city else ''
        informations += '%s\n' % self.company_id.state_id.display_name if self.company_id.state_id else ''
        informations += '%s' % self.company_id.country_id.display_name if self.company_id.country_id else ''
        vat_display = self.company_id.country_id.vat_label or _('VAT')
        vat_display = '\n' + vat_display + ': '
        informations += '%s %s' % (vat_display, self.company_id.vat) if self.company_id.vat else ''

        for record in self:
            record.company_informations = informations

    @api.depends('company_id')
    def _compute_is_root_company(self):
        for record in self:
            record.is_root_company = not record.company_id.parent_id

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api, tools
from odoo.tools.misc import str2bool


class ResUsers(models.Model):
    _inherit = 'res.users'

    @api.model
    def web_create_users(self, emails):
        emails_normalized = [tools.mail.parse_contact_from_email(email)[1] for email in emails]

        # Reactivate already existing users if needed
        deactivated_users = self.with_context(active_test=False).search([
            ('active', '=', False),
            '|', ('login', 'in', emails + emails_normalized), ('email_normalized', 'in', emails_normalized)])
        for user in deactivated_users:
            user.active = True
        done = deactivated_users.mapped('email_normalized')

        new_emails = set(emails) - set(deactivated_users.mapped('email'))

        # Process new email addresses : create new users
        for email in new_emails:
            name, email_normalized = tools.mail.parse_contact_from_email(email)
            if email_normalized in done:
                continue
            default_values = {'login': email_normalized, 'name': name or email_normalized, 'email': email_normalized, 'active': True}
            user = self.with_context(signup_valid=True).create(default_values)

        return True

    def _default_groups(self):
        """Default groups for employees

        If base_setup.default_user_minimal is set, only the "Employee" group is used
        """
        if str2bool(self.env['ir.config_parameter'].sudo().get_param("base_setup.default_user_rights_minimal"), default=False):
            employee_group = self.env.ref("base.group_user")
            # force the trans_implied_ids during default for consistency in the interface
            return employee_group | employee_group.trans_implied_ids
        return super()._default_groups()

    def _apply_groups_to_existing_employees(self):
        """
        If base_setup.default_user_rights_minimal is set, do not apply any new groups to existing employees
        """
        if str2bool(self.env['ir.config_parameter'].sudo().get_param("base_setup.default_user_rights_minimal"), default=False):
            return False
        return super()._apply_groups_to_existing_employees()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import res_config_settings
from . import res_users

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.base.setup</field>
            <field name="model">res.config.settings</field>
            <field name="priority" eval="0"/>
            <field name="inherit_id" ref="base.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//form" position="inside">
                    <field name="is_root_company" invisible="1"/>
                    <app data-string="General Settings" string="General Settings" name="general_settings" logo="/base/static/description/settings.png">

                        <div id="invite_users">
                            <block title="Users" name="users_setting_container">
                                <setting id="invite_users_setting">
                                    <widget name='res_config_invite_users'/>
                                </setting>
                                <setting id="active_user_setting">
                                    <span class="fa fa-lg fa-users" aria-label="Number of active users"/>
                                    <field name='active_user_count' class="w-auto ps-3 fw-bold"/>
                                    <span class='o_form_label' invisible="active_user_count &gt; 1">
                                        Active User
                                    </span>
                                    <span class='o_form_label' invisible="active_user_count &lt;= 1">
                                        Active Users
                                    </span>
                                    <a href="https://www.odoo.com/documentation/17.0/applications/general/users.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                    <br/>
                                    <button name="%(base.action_res_users)d" icon="oi-arrow-right" type="action" string="Manage Users" class="btn-link o_web_settings_access_rights"/>
                                </setting>
                            </block>
                        </div>

                        <div id="languages">
                            <block title="Languages" name="languages_setting_container">
                                <setting id="languages_setting">
                                    <!-- TODO This is not an ideal solution but it looks ok on the interface -->
                                    <div class="w-50">
                                        <field name="language_count" class="w-auto ps-1 fw-bold"/>
                                        <span class='o_form_label' invisible="language_count &gt; 1">
                                            Language
                                        </span>
                                        <span class='o_form_label' invisible="language_count &lt;= 1">
                                            Languages
                                        </span>
                                    </div>
                                    <div class="mt8">
                                        <button name="%(base.action_view_base_language_install)d" icon="oi-arrow-right" type="action" string="Add Languages" class="btn-link"/>
                                    </div>
                                    <div class="mt8" groups="base.group_no_one">
                                        <button name="%(base.res_lang_act_window)d" icon="oi-arrow-right" type="action" string="Manage Languages" class="btn-link"/>
                                    </div>
                                </setting>
                            </block>
                        </div>

                        <div id="companies">
                            <block title="Companies" name="companies_setting_container">
                                <field name="company_id" invisible="1"/>
                                <setting id="company_details_settings">
                                    <field name="company_name" nolabel="1" class="fw-bold"/>
                                    <br/>
                                    <field name="company_informations" class="text-muted" style="width: 90%;"/>
                                    <br/>
                                    <button name="open_company" icon="oi-arrow-right" type="object" string="Update Info" class="btn-link"/>
                                </setting>
                                <setting id="companies_setting">
                                    <field name='company_count' nolabel="1" class="w-auto ps-1 fw-bold"/>
                                    <span class='o_form_label' invisible="company_count &gt; 1">
                                        Company
                                    </span>
                                    <span class='o_form_label' invisible="company_count &lt;= 1">
                                        Companies
                                    </span>
                                    <br/>
                                    <div class="mt8">
                                        <button name="%(base.action_res_company_form)d" icon="oi-arrow-right" type="action" string="Manage Companies" class="btn-link"/>
                                    </div>
                                </setting>
                                <setting id="document_layout_setting" string="Document Layout" help="Choose the layout of your documents" company_dependent="1">
                                    <div class="content-group">
                                        <div class="mt16" groups="base.group_no_one">
                                            <label for="external_report_layout_id" string="Layout" class="col-3 col-lg-3 o_light_label"/>
                                            <field name="external_report_layout_id" domain="[('type','=', 'qweb')]" class="oe_inline"/>
                                        </div>
                                            <button name="%(web.action_base_document_layout_configurator)d" string="Configure Document Layout" type="action" class="oe_link" icon="oi-arrow-right"/>
                                            <br groups="base.group_no_one"/>
                                            <button name="edit_external_header" string="Edit Layout" type="object" class="oe_link" groups="base.group_no_one" icon="oi-arrow-right"/>
                                            <br groups="base.group_no_one"/>
                                            <button name="%(web.action_report_externalpreview)d" string="Preview Document" type="action" class="oe_link" groups="base.group_no_one" icon="oi-arrow-right"/>
                                    </div>
                                </setting>
                                <field name="company_id" invisible="1"/>
                                <setting id="inter_company" string="Inter-Company Transactions" company_dependent="1" help="Automatically generate counterpart documents for orders/invoices between companies" groups="base.group_multi_company" title="Configure company rules to automatically create SO/PO when one of your company sells/buys to another of your company.">
                                    <field name="module_account_inter_company_rules" widget="upgrade_boolean"/>
                                    <div class="content-group" invisible="not module_account_inter_company_rules" id="inter_companies_rules">
                                        <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                    </div>
                                </setting>
                            </block>
                        </div>
                        <div id="emails"/>

                        <div id="contacts_settings">
                            <block title="Contacts" name="contacts_setting_container">
                                <setting id="sms" string="Send SMS" documentation="/applications/marketing/sms_marketing/pricing/pricing_and_faq.html" help="Send texts to your contacts" />
                                <setting help="Automatically enrich your contact base with company data" title="When populating your address book, Odoo provides a list of matching companies. When selecting one item, the company data and logo are auto-filled." id="partner_autocomplete">
                                    <field name="module_partner_autocomplete"/>
                                </setting>
                        </block>
                    </div>

                    <block title="Permissions" id="user_default_rights">
                        <setting string="Default Access Rights" help="Set custom access rights for new users" title="By default, new users get highest access rights for all installed apps." id="access_rights">
                            <field name="user_default_rights"/>
                            <div class="content-group" invisible="not user_default_rights">
                                <div class="mt8">
                                    <button type="object" name="open_default_user" string="Default Access Rights" icon="oi-arrow-right" class="btn-link"/>
                                </div>
                            </div>
                        </setting>
                        <setting string="API Keys" help="API Keys allow your users to access Odoo with external tools when multi-factor authentication is enabled." groups="base.group_system">
                            <button type="action" name="%(base.action_apikeys_admin)d" string="Manage API Keys" icon="oi-arrow-right" class="btn-link"/>
                        </setting>
                        <setting string="Import &amp; Export" help="Allow users to import data from CSV/XLS/XLSX/ODS files" documentation="/applications/general/export_import_data.html" groups="base.group_no_one" id="allow_import">
                            <field name="module_base_import" />
                        </setting>
                        <setting id="feedback_motivate_setting" help="Add fun feedback and motivate your employees" groups="base.group_no_one">
                            <field name="show_effect"/>
                        </setting>
                    </block>

                    <block title="Progressive Web App" id="pwa_settings" groups="base.group_no_one">
                        <setting help="This name will be used for the application when Odoo is installed through the browser.">
                            <field name="web_app_name" placeholder="Odoo"/>
                        </setting>
                    </block>

                        <block title="Integrations" name="integration">
                            <setting string="Mail Plugin" documentation="/applications/productivity/mail_plugins.html" help="Integrate with mail client plugins" id="mail_pluggin_setting">
                                <field name="module_mail_plugin" />
                            </setting>
                            <div id="product_get_pic_setting"/>
                            <setting string="OAuth Authentication" help="Use external accounts to log in (Google, Facebook, etc.)" id="module_auth_oauth">
                                <field name="module_auth_oauth" />
                                <div class="content-group mt16" invisible="not module_auth_oauth" id="msg_module_auth_oauth">
                                    <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                </div>
                            </setting>
                            <setting string="LDAP Authentication" help="Use LDAP credentials to log in" documentation="/applications/general/auth/ldap.html" id="module_auth_ldap">
                                <field name="module_auth_ldap"/>
                                <div class="content-group" invisible="not module_auth_ldap" id="auth_ldap_warning">
                                    <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                </div>
                            </setting>
                            <setting documentation="/applications/websites/website/optimize/unsplash.html" help="Find free high-resolution images from Unsplash" id="unsplash">
                                <field name="module_web_unsplash"/>
                                <div class="content-group" invisible="not module_web_unsplash" id="web_unsplash_warning">
                                    <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                </div>
                            </setting>
                            <setting string="Geo Localization" help="GeoLocalize your partners" id="base_geolocalize">
                                <field name="module_base_geolocalize"/>
                                <div class="content-group" invisible="not module_base_geolocalize" name="base_geolocalize_warning">
                                    <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to choose your Geo Provider.</div>
                                </div>
                            </setting>
                            <setting help="Protect your forms from spam and abuse." id="recaptcha">
                                <field name="module_google_recaptcha"/>
                                <div class="content-group" invisible="not module_google_recaptcha" id="recaptcha_warning">
                                    <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up reCaptcha.</div>
                                </div>
                            </setting>
                            <setting help="Protect your forms with CF Turnstile." id="cf-turnstile">
                                <field name="module_website_cf_turnstile"/>
                                <div class="content-group" invisible="not module_website_cf_turnstile" id="turnstile_warning">
                                    <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up Cloudflare turnstile.</div>
                                </div>
                            </setting>

                        </block>

                        <block title="Performance" groups="base.group_no_one" name="performance">
                            <setting id="profiling_enabled_until" help="Enable the profiling tool. Profiling may impact performance while being active.">
                                <field name="profiling_enabled_until"/>
                            </setting>
                        </block>

                        <widget name='res_config_dev_tool'/>
                        <div id='about'>
                            <block title="About" name="about_setting_container">
                                <setting id='appstore'>
                                    <div class="d-flex">
                                        <div>
                                            <!-- FIXME Those links are defined directly in the template which means that we will have to
                                            update the template code is the link ever changes -->
                                            <a class="d-block mx-auto" href="https://play.google.com/store/apps/details?id=com.odoo.mobile" target="blank">
                                                <img alt="On Google Play" class="d-block mx-auto img img-fluid" src="/base_setup/static/src/img/google_play.png"/>
                                            </a>
                                        </div>
                                        <div>
                                            <a class='d-block mx-auto' href="https://itunes.apple.com/us/app/odoo/id1272543640" target="blank">
                                                <img alt="On Apple Store" class="d-block mx-auto img img-fluid" src="/base_setup/static/src/img/app_store.png"/>
                                            </a>
                                        </div>
                                    </div>
                                </setting>
                                <widget name='res_config_edition'/>
                            </block>
                        </div>
                    </app>
                </xpath>
            </field>
        </record>

        <record id="action_general_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="res_model">res.config.settings</field>
            <field name="view_mode">form</field>
            <field name="target">inline</field>
            <field name="context">{'module' : 'general_settings', 'bin_size': False}</field>
        </record>

        <menuitem
            id="menu_config"
            name="General Settings"
            parent="base.menu_administration"
            sequence="0"
            action="action_general_configuration"
            groups="base.group_system"/>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Add partner categories in partner kanban view -->
        <record id="res_partner_kanban_view" model="ir.ui.view">
            <field name="name">res.partner.kanban.inherit</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.res_partner_kanban_view"/>
            <field name="arch" type="xml">
                <xpath expr="//div[hasclass('oe_kanban_partner_categories')]" position="inside">
                    <span class="oe_kanban_list_many2many">
                        <field name="category_id" widget="many2many_tags" options="{'color_field': 'color'}"/>
                    </span>
                </xpath>
            </field>
        </record>
</odoo>

```

