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
        'views/assets.xml',
    ],
    'demo': [],
    'installable': True,
    'auto_install': False,

    'qweb': [
        'static/src/xml/res_config_dev_tool.xml',
        'static/src/xml/res_config_edition.xml',
        'static/src/xml/res_config_invite_users.xml',
    ],

    
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

        return {
            'active_users': active_count,
            'pending_count': pending_count,
            'pending_users': pending_users,
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
from odoo.http import request


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        result = super(IrHttp, self).session_info()
        if request.env.user.has_group('base.group_user'):
            result['show_effect'] = request.env['ir.config_parameter'].sudo().get_param('base_setup.show_effect')
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
    user_default_rights = fields.Boolean(
        "Default Access Rights",
        config_parameter='base_setup.default_user_rights')
    external_email_server_default = fields.Boolean(
        "External Email Servers",
        config_parameter='base_setup.default_external_email_server')
    module_base_import = fields.Boolean("Allow users to import data from CSV/XLS/XLSX/ODS files")
    module_google_calendar = fields.Boolean(
        string='Allow the users to synchronize their calendar  with Google Calendar')
    module_microsoft_calendar = fields.Boolean(
        string='Allow the users to synchronize their calendar with Outlook Calendar')
    module_google_drive = fields.Boolean("Attach Google documents to any record")
    module_google_spreadsheet = fields.Boolean("Google Spreadsheet")
    module_auth_oauth = fields.Boolean("Use external authentication providers (OAuth)")
    module_auth_ldap = fields.Boolean("LDAP Authentication")
    # TODO: remove in master
    module_base_gengo = fields.Boolean("Translate Your Website with Gengo")
    module_account_inter_company_rules = fields.Boolean("Manage Inter Company")
    module_pad = fields.Boolean("Collaborative Pads")
    module_voip = fields.Boolean("Asterisk (VoIP)")
    module_web_unsplash = fields.Boolean("Unsplash Image Library")
    module_partner_autocomplete = fields.Boolean("Partner Autocomplete")
    module_base_geolocalize = fields.Boolean("GeoLocalize")
    module_google_recaptcha = fields.Boolean("reCAPTCHA: Easy on Humans, Hard on Bots")
    report_footer = fields.Text(related="company_id.report_footer", string='Custom Report Footer', help="Footer text displayed at the bottom of all reports.", readonly=False)
    group_multi_currency = fields.Boolean(string='Multi-Currencies',
            implied_group='base.group_multi_currency',
            help="Allows to work in a multi currency environment")
    paperformat_id = fields.Many2one(related="company_id.paperformat_id", string='Paper format', readonly=False)
    external_report_layout_id = fields.Many2one(related="company_id.external_report_layout_id", readonly=False)
    show_effect = fields.Boolean(string="Show Effect", config_parameter='base_setup.show_effect')
    company_count = fields.Integer('Number of Companies', compute="_compute_company_count")
    active_user_count = fields.Integer('Number of Active Users', compute="_compute_active_user_count")
    language_count = fields.Integer('Number of Languages', compute="_compute_language_count")
    company_name = fields.Char(related="company_id.display_name", string="Company Name")
    company_informations = fields.Text(compute="_compute_company_informations")

    def open_company(self):
        return {
            'type': 'ir.actions.act_window',
            'name': 'My Company',
            'view_mode': 'form',
            'res_model': 'res.company',
            'res_id': self.env.company.id,
            'target': 'current',
            'context': {
                'form_view_initial_mode': 'edit',
            },
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

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class ResUsers(models.Model):
    _inherit = 'res.users'

    @api.model
    def web_create_users(self, emails):

        # Reactivate already existing users if needed
        deactivated_users = self.with_context(active_test=False).search([('active', '=', False), '|', ('login', 'in', emails), ('email', 'in', emails)])
        for user in deactivated_users:
            user.active = True

        new_emails = set(emails) - set(deactivated_users.mapped('email'))

        # Process new email addresses : create new users
        for email in new_emails:
            default_values = {'login': email, 'name': email.split('@')[0], 'email': email, 'active': True}
            user = self.with_context(signup_valid=True).create(default_values)

        return True

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_http
from . import res_config_settings
from . import res_users

```

## File: static\src\js\res_config_dev_tool.js

```javascript
odoo.define('base_setup.ResConfigDevTool', function (require) {
    "use strict";

    var config = require('web.config');
    var Widget = require('web.Widget');
    var widget_registry = require('web.widget_registry');

    var ResConfigDevTool = Widget.extend({
        template: 'res_config_dev_tool',
        events: {
            'click .o_web_settings_force_demo': '_onClickForceDemo',
        },

        init: function () {
            this._super.apply(this, arguments);
            this.isDebug = config.isDebug();
            this.isAssets = config.isDebug("assets");
            this.isTests = config.isDebug("tests");
        },

        willStart: function () {
            var self = this;
            return this._super.apply(this, arguments).then(function () {
                return self._rpc({
                    route: '/base_setup/demo_active',
                }).then(function (demo_active) {
                    self.demo_active = demo_active;
                });
            });
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * Forces demo data to be installed in a database without demo data installed.
         *
         * @private
         * @param {MouseEvent} ev
         */
        _onClickForceDemo: function (ev) {
            ev.preventDefault();
            this.do_action('base.demo_force_install_action');
        },
    });

    widget_registry.add('res_config_dev_tool', ResConfigDevTool);
});

```

## File: static\src\js\res_config_edition.js

```javascript
odoo.define('base_setup.ResConfigEdition', function (require) {
    "use strict";

    var Widget = require('web.Widget');
    var widget_registry = require('web.widget_registry');
    var session = require ('web.session');

    var ResConfigEdition = Widget.extend({
        template: 'res_config_edition',

       /**
        * @override
        */
        init: function () {
            this._super.apply(this, arguments);
            this.server_version = session.server_version;
            this.expiration_date = session.expiration_date
                ? moment(session.expiration_date)
                : moment().add(30, 'd');
        },
   });

   widget_registry.add('res_config_edition', ResConfigEdition);

    return ResConfigEdition;
});

```

## File: static\src\js\res_config_invite_users.js

```javascript
odoo.define('base_setup.ResConfigInviteUsers', function (require) {
    "use strict";

    var Widget = require('web.Widget');
    var widget_registry = require('web.widget_registry');
    var core = require('web.core');

    var _t = core._t;

    var ResConfigInviteUsers = Widget.extend({
        template: 'res_config_invite_users',

        events: {
            'click .o_web_settings_invite': '_onClickInvite',
            'click .o_web_settings_user': '_onClickUser',
            'click .o_web_settings_more': '_onClickMore',
            'keydown .o_user_emails': '_onKeydownUserEmails',
        },

        /**
         * @override
         */
        init: function () {
            this._super.apply(this, arguments);
            this.emails = [];
        },

        willStart: function () {
            var self = this;

            return this._super.apply(this, arguments).then(function () {
                return self.load();
            });
        },

        load: function () {
            var self = this;

            return this._rpc({
                route: '/base_setup/data',
            }).then(function (data) {
                self.active_users = data.active_users;
                self.pending_users = data.pending_users;
                self.pending_count = data.pending_count;
                self.resend_invitation = data.resend_invitation || false;
            });
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * @private
         * @param {string} email
         * @returns {boolean} true if the given email address is valid
         */
        _validateEmail: function (email) {
            var re = /^([a-z0-9][-a-z0-9_\+\.]*)@((?:[\w-]+\.)*\w[\w-]{0,66})\.([a-z]{2,63}(?:\.[a-z]{2})?)$/i;
            return re.test(email);
        },

        /**
         * Send invitation for valid and unique email addresses
         *
         * @private
         */
        _invite: function () {
            var self = this;

            var $userEmails = this.$('.o_user_emails');
            $userEmails.prop('disabled', true);
            this.$('.o_web_settings_invite').prop('disabled', true);
            var value = $userEmails.val().trim();
            if (value) {
                // filter out duplicates
                var emails = _.uniq(value.split(/[ ,;\n]+/));

                // filter out invalid email addresses
                var invalidEmails = _.reject(emails, this._validateEmail);
                if (invalidEmails.length) {
                    this.do_warn(false, _.str.sprintf(
                        _t('Invalid email addresses: %s.'),
                        invalidEmails.join(', ')
                    ));
                }
                emails = _.difference(emails, invalidEmails);

                if (!this.resend_invitation) {
                    // filter out already processed or pending addresses
                    var pendingEmails = _.map(this.pending_users, function (info) {
                        return info[1];
                    });
                    var existingEmails = _.intersection(emails, this.emails.concat(pendingEmails));
                    if (existingEmails.length) {
                        this.do_warn(false, _.str.sprintf(
                            _t('Email addresses already existing: %s.'),
                            existingEmails.join(', ')
                        ));
                    }
                    emails = _.difference(emails, existingEmails);
                }

                if (emails.length) {
                    $userEmails.val('');
                    this._rpc({
                        model: 'res.users',
                        method: 'web_create_users',
                        args: [emails],
                    }).then(function () {
                        return self.load().then(function () {
                            self.renderElement();
                            self.$('.o_user_emails').focus();
                        });
                    });
                } else {
                    $userEmails.prop('disabled', false);
                    this.$('.o_web_settings_invite').prop('disabled', false);
                    self.$('.o_user_emails').focus();
                }
            }
        },

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * @private
         * @param {MouseEvent} ev
         */
        _onClickInvite: function (ev) {
            if (this.$('.o_user_emails').val().length) {
                var $button = $(ev.target);
                $button.button('loading');
                return this._invite();
            }
        },
        /**
         * @private
         * @param {MouseEvent} ev
         */
        _onClickMore: function (ev) {
            var self = this;
            ev.preventDefault();
            this._rpc({
                model: 'ir.model.data',
                method: 'xmlid_to_res_model_res_id',
                args: ["base.view_users_form"],
            })
            .then(function (data) {
                self.do_action({
                    name: _t('Users'),
                    type: 'ir.actions.act_window',
                    view_mode: 'tree,form',
                    res_model: 'res.users',
                    domain: [['log_ids', '=', false]],
                    context: {
                        search_default_no_share: true,
                    },
                    views: [[false, 'list'], [data[1], 'form']],
                });
            });
        },
        /**
         * @private
         * @param {MouseEvent} ev
         */
        _onClickUser: function (ev) {
            var self = this;
            ev.preventDefault();
            var user_id = $(ev.currentTarget).data('user-id');
            this._rpc({
                model: 'ir.model.data',
                method: 'xmlid_to_res_model_res_id',
                args: ["base.view_users_form"],
            })
            .then(function (data) {
                self.do_action({
                    type: 'ir.actions.act_window',
                    res_model: 'res.users',
                    view_mode: 'form',
                    res_id: user_id,
                    views: [[data[1], 'form']],
                });
            });
        },
        /**
         * @private
         * @param {KeyboardEvent} ev
         */
        _onKeydownUserEmails: function (ev) {
            var keyCodes = [$.ui.keyCode.TAB, $.ui.keyCode.COMMA, $.ui.keyCode.ENTER];
            if (_.contains(keyCodes, ev.which)) {
                ev.preventDefault();
                this._invite();
            }
        },
    });

   widget_registry.add('res_config_invite_users', ResConfigInviteUsers);

});

```

## File: static\src\xml\res_config_dev_tool.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<template>
    <div t-name='res_config_dev_tool'>
        <div id="developer_tool">
            <h2>Developer Tools</h2>
            <div class="row mt16 o_settings_container">
                <div class="col-12 col-lg-6 o_setting_box" id="devel_tool">
                    <div class="o_setting_right_pane">
                        <a t-if="!widget.isDebug" class="d-block" href="?debug=1">Activate the developer mode</a>
                        <a t-if="!widget.isAssets" class="d-block" href="?debug=assets">Activate the developer mode (with assets)</a>
                        <a t-if="!widget.isTests" class="d-block" href="?debug=assets,tests">Activate the developer mode (with tests assets)</a>
                        <a t-if="widget.isDebug" class="d-block" href="?debug=">Deactivate the developer mode</a>
                        <a t-if="widget.isDebug and !widget.demo_active" class="o_web_settings_force_demo" href="#">Load demo data</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

```

## File: static\src\xml\res_config_edition.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- TODO See if this has to be editable in some way, see if it is easy for someone to change this -->
<!-- TODO Also see what is done to override the (Community Edition) -->
<template>
    <div t-name='res_config_edition'>
        <div class="col-12 o_setting_box" id="edition">
            <div class="o_setting_right_pane">
                <div class="user-heading">
                    <h3>
                        Odoo <t t-esc="widget.server_version"/>
                        (Community Edition)
                    </h3>
                </div>
                <div>
                    <div class="tab-content">
                        <div role="tabpanel" id="settings" class="tab-pane active text-muted o_web_settings_compact_subtitle">
                            <small>Copyright © 2004 <a target="_blank" href="https://www.odoo.com" style="text-decoration: underline;">Odoo S.A.</a> <a id="license" target="_blank" href="http://www.gnu.org/licenses/lgpl.html" style="text-decoration: underline;">GNU LGPL Licensed</a></small>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

```

## File: static\src\xml\res_config_invite_users.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
    <div t-name='res_config_invite_users'>
        <p class="o_form_label">Invite New Users</p>
        <div class="d-flex">
            <input class="o_user_emails o_input mt8 text-truncate" type="text" placeholder="Enter e-mail address"/>
            <button class="btn btn-primary o_web_settings_invite" data-loading-text="Inviting..."><strong>Invite</strong></button>
        </div>

        <t t-if="widget.pending_users.length">
            <p class="o_form_label pt-3">Pending Invitations:</p>
            <span t-foreach="widget.pending_users" t-as="pending">
                <a href="#" class="badge badge-pill o_web_settings_user" t-att-data-user-id="pending[0]"> <t t-esc="pending[1]"/> </a>
            </span>
            <t t-if="widget.pending_users.length &lt; widget.pending_count">
                <br/>
                <a href="#" class="o_web_settings_more"><t t-esc="widget.pending_count - widget.pending_users.length"/> more </a>
            </t>
        </t>
    </div>
</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="dev tool assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" type="text/scss" href="/base_setup/static/src/scss/settings.scss"/>
            <script type="text/javascript" src="/base_setup/static/src/js/res_config_dev_tool.js"></script>
            <script type="text/javascript" src="/base_setup/static/src/js/res_config_edition.js"></script>
            <script type="text/javascript" src="/base_setup/static/src/js/res_config_invite_users.js"></script>
        </xpath>
    </template>
</odoo>

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
                <xpath expr="//div[hasclass('settings')]" position="inside">
                    <div class="app_settings_block" data-string="General Settings" string="General Settings" data-key="general_settings">

                        <div id="invite_users">
                            <h2>Users</h2>
                            <div class="row mt16 o_settings_container" name="users_setting_container">
                                <div class="col-12 col-lg-6 o_setting_box" id="active_user_setting">
                                    <div class="o_setting_right_pane">
                                        <span class="fa fa-lg fa-users" aria-label="Number of active users"/>
                                        <field name='active_user_count' class="w-auto pl-3 font-weight-bold"/>
                                        <span class='o_form_label' attrs="{'invisible':[('active_user_count', '&gt;', '1')]}">
                                            Active User
                                        </span>
                                        <span class='o_form_label' attrs="{'invisible':[('active_user_count', '&lt;=', '1')]}">
                                            Active Users
                                        </span>
                                        <a href="https://www.odoo.com/documentation/14.0/applications/general/odoo_basics/users.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                        <br/>
                                        <button name="%(base.action_res_users)d" icon="fa-arrow-right" type="action" string="Manage Users" class="btn-link o_web_settings_access_rights"/>

                                    </div>
                                </div>
                                <div class="col-12 col-lg-6 o_setting_box" id="invite_users_setting">
                                    <div class="o_setting_right_pane">
                                        <widget name='res_config_invite_users'/>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div id="companies">
                            <h2>Companies</h2>
                            <div class="row mt16 o_settings_container" name="companies_setting_container">
                                <div class="col-12 col-lg-6 o_setting_box" id="company_details_settings">
                                    <div class="o_setting_right_pane">
                                        <field name="company_name" class="font-weight-bold"/>
                                        <br/>
                                        <field name="company_informations" class="text-muted" style="width: 90%;"/>
                                        <br/>
                                        <button name="open_company" icon="fa-arrow-right" type="object" string="Update Info" class="btn-link"/>
                                    </div>
                                </div>
                                <div class="col-12 col-lg-6 o_setting_box" id="companies_setting">
                                    <div class="o_setting_right_pane">
                                        <field name='company_count' class="w-auto pl-1 font-weight-bold"/>
                                        <span class='o_form_label' attrs="{'invisible':[('company_count', '&gt;', '1')]}">
                                            Company
                                        </span>
                                        <span class='o_form_label' attrs="{'invisible':[('company_count', '&lt;=', '1')]}">
                                            Companies
                                        </span>
                                        <br/>
                                        <div class="mt8">
                                            <button name="%(base.action_res_company_form)d" icon="fa-arrow-right" type="action" string="Manage Companies" class="btn-link"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div id="languages">
                            <h2>Languages</h2>
                            <div class='row mt16 o_settings_container' name="languages_setting_container">
                                <div class='col-xs-12 col-md-6 o_setting_box' id="languages_setting">
                                    <div class='o_setting_right_pane'>
                                        <!-- TODO This is not an ideal solution but it looks ok on the interface -->
                                        <div class="w-50">
                                            <field name="language_count" class="w-auto pl-1 font-weight-bold"/>
                                            <span class='o_form_label' attrs="{'invisible':[('language_count', '&gt;', '1')]}">
                                                Language
                                            </span>
                                            <span class='o_form_label' attrs="{'invisible':[('language_count', '&lt;=', '1')]}">
                                                Languages
                                            </span>
                                        </div>
                                        <div class="mt8">
                                            <button name="%(base.action_view_base_language_install)d" icon="fa-arrow-right" type="action" string="Add Language" class="btn-link"/>
                                        </div>
                                        <div class="mt8" groups="base.group_no_one">
                                            <button name="%(base.res_lang_act_window)d" icon="fa-arrow-right" type="action" string="Manage Languages" class="btn-link"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div id="business_documents">
                            <h2>Business Documents</h2>
                            <div class="row mt16 o_settings_container" name="business_documents_setting_container">
                                <div class="col-12 col-lg-6 o_setting_box" id="paper_format_setting">
                                    <div class="o_setting_right_pane">
                                        <span class="o_form_label">Format</span>
                                        <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                                        <div class="text-muted">
                                            Set the paper format of printed documents
                                        </div>
                                        <div class="content-group">
                                            <div class="mt16 row">
                                                <label for="paperformat_id" string="Format" class="col-3 col-lg-3 o_light_label"/>
                                                <field name="paperformat_id" class="oe_inline" required="1"/>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-12 col-lg-6 o_setting_box" id="document_layout_setting">
                                    <div class="o_setting_right_pane">
                                        <span class="o_form_label">Document Layout</span>
                                        <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                                        <div class="text-muted">
                                            Choose the layout of your documents
                                        </div>
                                        <div class="content-group">
                                            <div class="mt16" groups="base.group_no_one">
                                                <label for="external_report_layout_id" string="Layout" class="col-3 col-lg-3 o_light_label"/>
                                                <field name="external_report_layout_id" domain="[('type','=', 'qweb')]" class="oe_inline"/>
                                            </div>
                                            <div class="mt8">
                                                <button name="%(web.action_base_document_layout_configurator)d" string="Configure Document Layout" type="action" class="oe_link" icon="fa-arrow-right"/>
                                                <button name="edit_external_header" string="Edit Layout" type="object" class="oe_link" groups="base.group_no_one"/>
                                                <button name="%(web.action_report_externalpreview)d" string="Preview Document" type="action" class="oe_link" groups="base.group_no_one"/>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div id="contacts_settings">
                            <h2>Contacts</h2>
                            <div class="row mt16 o_settings_container" name="contacts_setting_container">
                                <div class="col-xs-12 col-md-6 o_setting_box" id="sms">
                                        <div class="o_setting_right_pane" id="sms_settings">
                                            <div class="o_form_label">
                                            Send SMS
                                            <a href="https://www.odoo.com/documentation/14.0/applications/marketing/sms_marketing/pricing/pricing_and_faq.html" title="Documentation" class="ml-1 o_doc_link" target="_blank"></a>
                                            </div>
                                            <div class="text-muted">
                                                Send texts to your contacts
                                            </div>
                                        </div>
                                </div>
                                <div class="col-xs-12 col-md-6 o_setting_box" title="When populating your address book, Odoo provides a list of matching companies. When selecting one item, the company data and logo are auto-filled." id="partner_autocomplete">
                                    <div class="o_setting_left_pane">
                                        <field name="module_partner_autocomplete"/>
                                    </div>
                                    <div class="o_setting_right_pane" id="partner_autocomplete_settings">
                                        <label for="module_partner_autocomplete"/>
                                        <div class="text-muted">
                                            Automatically enrich your contact base with company data
                                        </div>
                                    </div>
                                </div>
                        </div>
                    </div>
                    <div id="emails"/>
                    <h2>Permissions</h2>
                    <div class="row mt16 o_settings_container" id="user_default_rights">
                        <div class="col-12 col-lg-6 o_setting_box"  title="By default, new users get highest access rights for all installed apps." id="access_rights">
                            <div class="o_setting_left_pane">
                                <field name="user_default_rights"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label string="Default Access Rights" for="user_default_rights"/>
                                <div class="text-muted">
                                    Set custom access rights for new users
                                </div>
                                <div class="content-group" attrs="{'invisible': [('user_default_rights','=',False)]}">
                                    <div class="mt8">
                                        <button type="object" name="open_default_user" string="Default Access Rights" icon="fa-arrow-right" class="btn-link"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box"
                             groups="base.group_system">
                            <div class="o_setting_left_pane"/>
                            <div class="o_setting_right_pane">
                                <button type="action" name="%(base.action_apikeys_admin)d" string="Manage API Keys" icon="fa-arrow-right" class="btn-link"/>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" groups="base.group_no_one" id="allow_import">
                            <div class="o_setting_left_pane">
                                <field name="module_base_import" />
                            </div>
                            <div class="o_setting_right_pane">
                                <label string="Import &amp; Export" for="module_base_import"/>
                                <a href="https://www.odoo.com/documentation/14.0/applications/general/base_import/import_faq.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                <div class="text-muted">
                                    Allow users to import data from CSV/XLS/XLSX/ODS files
                                </div>
                            </div>
                        </div>
                        <div class="col-12 col-lg-6 o_setting_box" id="feedback_motivate_setting" groups="base.group_no_one">
                            <div class="o_setting_left_pane">
                                <field name="show_effect"/>
                            </div>
                            <div class="o_setting_right_pane">
                                <label for="show_effect"/>
                                <div class="text-muted">
                                    Add fun feedback and motivate your employees
                                </div>
                            </div>
                        </div>
                        </div>
                        <div id="multi_company" groups="base.group_multi_company">
                            <field name="company_id" invisible="1"/>
                            <h2>Multi-Company</h2>
                            <div class="row mt16 o_settings_container" name="multicompany_setting_container">
                                <div class="col-12 col-lg-6 o_setting_box" title="Configure company rules to automatically create SO/PO when one of your company sells/buys to another of your company." id="inter_company">
                                    <div class="o_setting_left_pane">
                                        <field name="module_account_inter_company_rules" widget="upgrade_boolean"/>
                                    </div>
                                    <div class="o_setting_right_pane">
                                        <label string="Inter-Company Transactions" for="module_account_inter_company_rules"/>
                                        <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                                        <div class="text-muted">
                                            Automatically generate counterpart documents for orders/invoices between companies
                                        </div>
                                        <div class="content-group" attrs="{'invisible': [('module_account_inter_company_rules','=',False)]}" id="inter_companies_rules">
                                            <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <h2>Integrations</h2>
                        <div class="row mt16 o_settings_container" name="integration">
                            <div class="col-12 col-lg-6 o_setting_box" id="external_pads_setting">
                                <div class="o_setting_left_pane">
                                    <field name="module_pad"/>
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_pad"/>
                                    <div class="text-muted">
                                        Use external pads in Odoo Notes
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('module_pad','=',False)]}" id="msg_module_pad">
                                        <div class="text-warning mt16"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="sync_google_calendar_setting">
                                <div class="o_setting_left_pane">
                                    <field name="module_microsoft_calendar" />
                                </div>
                                <div class="o_setting_right_pane">
                                    <label string="Outlook Calendar" for="module_microsoft_calendar"/>
                                    <div class="text-muted">
                                        Synchronize your calendar with Outlook
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('module_microsoft_calendar', '=', False)]}" id="msg_module_microsoft_calendar">
                                        <div class="text-warning mt16"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="sync_google_calendar_setting">
                                <div class="o_setting_left_pane">
                                    <field name="module_google_calendar" />
                                </div>
                                <div class="o_setting_right_pane">
                                    <label string="Google Calendar" for="module_google_calendar"/>
                                    <a href="https://www.odoo.com/documentation/14.0/applications/general/calendars/google/google_calendar_credentials.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                    <div class="text-muted">
                                        Synchronize your calendar with Google Calendar
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('module_google_calendar','=',False)]}" id="msg_module_google_calendar">
                                        <div class="text-warning mt16"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="google_drive_documents_setting">
                                <div class="o_setting_left_pane">
                                    <field name="module_google_drive" />
                                </div>
                                <div class="o_setting_right_pane">
                                    <label string="Google Drive" for="module_google_drive"/>
                                    <div class="text-muted">
                                        Create and attach Google Drive documents to any record
                                    </div>
                                    <div class="content-group mt16" attrs="{'invisible': [('module_google_drive','=',False)]}" id="msg_module_google_drive">
                                        <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="google_spreadsheet_setting">
                                <div class="o_setting_left_pane">
                                    <field name="module_google_spreadsheet" />
                                </div>
                                <div class="o_setting_right_pane">
                                    <label for="module_google_spreadsheet"/>
                                    <div class="text-muted">
                                        Extract and analyze Odoo data from Google Spreadsheet
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('module_google_spreadsheet','=',False)]}" id="msg_module_google_spreadsheet">
                                        <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="module_auth_oauth">
                                <div class="o_setting_left_pane">
                                    <field name="module_auth_oauth" />
                                </div>
                                <div class="o_setting_right_pane">
                                    <label string="OAuth Authentication" for="module_auth_oauth"/>
                                    <div class="text-muted">
                                       Use external accounts to log in (Google, Facebook, etc.)
                                    </div>
                                    <div class="content-group mt16" attrs="{'invisible': [('module_auth_oauth','=',False)]}" id="msg_module_auth_oauth">
                                        <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="oauth">
                                <div class="o_setting_left_pane">
                                    <field name="module_auth_ldap"/>
                                </div>
                                <div class="o_setting_right_pane" name="auth_ldap_right_pane">
                                    <label string="LDAP Authentication" for="module_auth_ldap"/>
                                    <a href="https://www.odoo.com/documentation/14.0/applications/general/auth/ldap.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                    <div class="text-muted">
                                       Use LDAP credentials to log in
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('module_auth_ldap','=',False)]}" id="auth_ldap_warning">
                                        <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="unsplash">
                                <div class="o_setting_left_pane">
                                    <field name="module_web_unsplash"/>
                                </div>
                                <div class="o_setting_right_pane" id="web_unsplash_settings">
                                    <label for="module_web_unsplash"/>
                                    <a href="https://www.odoo.com/documentation/14.0/applications/general/unsplash/unsplash_access_key.html" title="Documentation" class="o_doc_link" target="_blank"></a>
                                    <div class="text-muted">
                                        Find free high-resolution images from Unsplash
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('module_web_unsplash', '=', False)]}" id="web_unsplash_warning">
                                        <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up the feature.</div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="base_geolocalize">
                                <div class="o_setting_left_pane">
                                    <field name="module_base_geolocalize"/>
                                </div>
                                <div class="o_setting_right_pane" id="web_geolocalize_settings">
                                    <label string="Geo Localization" for="module_base_geolocalize"/>
                                    <div class="text-muted">
                                       GeoLocalize your partners
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('module_base_geolocalize','=', False)]}" name="base_geolocalize_warning">
                                        <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to choose your Geo Provider.</div>
                                    </div>
                                </div>
                            </div>
                            <div class="col-12 col-lg-6 o_setting_box" id="recaptcha">
                                <div class="o_setting_left_pane">
                                    <field name="module_google_recaptcha"/>
                                </div>
                                <div class="o_setting_right_pane" id="website_recaptcha_settings">
                                    <label for="module_google_recaptcha"/>
                                    <div class="text-muted">
                                        Protect your forms from spam and abuse.
                                    </div>
                                    <div class="content-group" attrs="{'invisible': [('module_google_recaptcha', '=', False)]}" id="recaptcha_warning">
                                        <div class="mt16 text-warning"><strong>Save</strong> this page and come back here to set up reCaptcha.</div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <widget name='res_config_dev_tool'/>
                        <div id='about'>
                            <h2>About</h2>
                            <div class="row mt16 o_settings_container" name="about_setting_container">
                                <div class='col-12 col-lg-6 o_setting_box' id='appstore'>
                                    <div class="d-flex">
                                        <div class="o_setting_right_pane">
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
                                </div>
                                <widget name='res_config_edition'/>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>

        <record id="action_general_configuration" model="ir.actions.act_window">
            <field name="name">Settings</field>
            <field name="type">ir.actions.act_window</field>
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

