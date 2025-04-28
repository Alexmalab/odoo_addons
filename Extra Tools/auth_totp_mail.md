# Odoo Module: auth_totp_mail

Category: Extra Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
{
    'name': '2FA Invite mail',
    'description': """
2FA Invite mail
===============
Allow the users to invite another user to use Two-Factor authentication
by sending an email to the target user. This email redirects them to:
- the users security settings if the user is internal.
- the portal security settings page if the user is not internal.
    """,
    'depends': ['auth_totp', 'mail'],
    'category': 'Extra Tools',
    'auto_install': True,
    'data': [
        'data/ir_action_data.xml',
        'data/mail_template_data.xml',
        'data/security_notifications_template.xml',
        'views/res_users_views.xml',
    ],
    'assets': {
        'web.assets_tests': [
            'auth_totp_mail/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\ir_action_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Invite to user 2FA -->
    <record model="ir.actions.server" id="action_invite_totp">
        <field name="name">Invite to use two-factor authentication</field>
        <field name="model_id" ref="base.model_res_users"/>
        <field name="binding_model_id" ref="base.model_res_users"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">
            action = records.action_totp_invite()
        </field>
        <field name="groups_id" eval="[(4, ref('base.group_erp_manager'))]"/>
    </record>

    <!--Action called when using the link in "Invite to use 2FA" mail-->
    <record model="ir.actions.server" id="action_activate_two_factor_authentication">
        <field name="name">Open two-factor authentication configuration</field>
        <field name="model_id" ref="base.model_res_users"/>
        <field name="state">code</field>
        <field name="code">
user = env.user
action = user.action_open_my_account_settings()
        </field>
        <field name="groups_id" eval="[(4, ref('base.group_user'))]"/>
    </record>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">
    <record id="mail_template_totp_invite" model="mail.template">
        <field name="name">Settings: 2Fa Invitation</field>
        <field name="model_id" ref="base.model_res_users" />
        <field name="email_from">{{ (object.company_id.email_formatted or user.email_formatted) }}</field>
        <field name="subject">Invitation to activate two-factor authentication on your Odoo account</field>
        <field name="partner_to">{{ object.partner_id.id }}</field>
        <field name="lang">{{ object.partner_id.lang }}</field>
        <field name="auto_delete" eval="True"/>
        <field name="body_html" type="html">
<div style="margin: 0px; padding: 0px; font-size: 13px;">
    <p style="margin: 0px; padding: 0px; font-size: 13px;">
        Dear <t t-out="object.partner_id.name or ''"></t><br/><br/>
        <t t-out="user.name  or ''"></t> requested you activate two-factor authentication to protect your account.<br/><br/>
        Two-factor Authentication ("2FA") is a system of double authentication.
        The first one is done with your password and the second one with a code you get from a dedicated mobile app.
        Popular ones include Authy, Google Authenticator or the Microsoft Authenticator.

        <p style="margin: 16px 0px 16px 0px; text-align: center;">
            <a t-att-href="object.get_totp_invite_url()"
                style="background-color:#875A7B; padding: 8px 16px 8px 16px; text-decoration: none; color: #fff; border-radius: 5px;">
                Activate my two-factor Authentication
            </a>
        </p>
    </p>
</div>
        </field>
    </record>
</data>
</odoo>

```

## File: data\security_notifications_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Extend the security update template to include 2fa suggestion -->
        <template id="account_security_setting_update" inherit_id="mail.account_security_setting_update">
            <xpath expr="//ul[hasclass('o_mail_account_security_suggestions')]/li[1]" position="after">
                <li t-if="suggest_2fa">
                    <span>Consider</span>
                    <a href="https://www.odoo.com/documentation/17.0/applications/general/auth/2fa.html">
                        activating Two-factor Authentication
                    </a>
                </li>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: models\auth_totp_device.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from collections import defaultdict


class AuthTotpDevice(models.Model):
    _inherit = "auth_totp.device"

    def unlink(self):
        """ Notify users when trusted devices are removed from their account. """
        removed_devices_by_user = self._classify_by_user()
        for user, removed_devices in removed_devices_by_user.items():
            user._notify_security_setting_update(
                _("Security Update: Device Removed"),
                _(
                    "A trusted device has just been removed from your account: %(device_names)s",
                    device_names=', '.join([device.name for device in removed_devices])
                ),
            )

        return super().unlink()

    def _generate(self, scope, name):
        """ Notify users when trusted devices are added onto their account.
        We override this method instead of 'create' as those records are inserted directly into the
        database using raw SQL. """

        res = super()._generate(scope, name)

        self.env.user._notify_security_setting_update(
            _("Security Update: Device Added"),
            _(
                "A trusted device has just been added to your account: %(device_name)s",
                device_name=name
            ),
        )

        return res

    def _classify_by_user(self):
        devices_by_user = defaultdict(lambda: self.env['auth_totp.device'])
        for device in self:
            devices_by_user[device.user_id] |= device

        return devices_by_user

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models


class Users(models.Model):
    _inherit = 'res.users'

    def write(self, vals):
        res = super().write(vals)

        if 'totp_secret' in vals:
            if vals.get('totp_secret'):
                self._notify_security_setting_update(
                    _("Security Update: 2FA Activated"),
                    _("Two-factor authentication has been activated on your account"),
                    suggest_2fa=False,
                )
            else:
                self._notify_security_setting_update(
                    _("Security Update: 2FA Deactivated"),
                    _("Two-factor authentication has been deactivated on your account"),
                    suggest_2fa=False,
                )

        return res

    def _notify_security_setting_update_prepare_values(self, content, suggest_2fa=True, **kwargs):
        """" Prepare rendering values for the 'mail.account_security_setting_update' qweb template

          :param bool suggest_2fa:
            Whether or not to suggest the end-user to turn on 2FA authentication in the email sent.
            It will only suggest to turn on 2FA if not already turned on on the user's account. """

        values = super()._notify_security_setting_update_prepare_values(content, **kwargs)
        values['suggest_2fa'] = suggest_2fa and not self.totp_enabled
        return values

    def action_open_my_account_settings(self):
        action = {
            "name": _("Account Security"),
            "type": "ir.actions.act_window",
            "res_model": "res.users",
            "views": [[self.env.ref('auth_totp_mail.res_users_view_form').id, "form"]],
            "res_id": self.id,
        }
        return action

    def get_totp_invite_url(self):
        return '/web#action=auth_totp_mail.action_activate_two_factor_authentication'

    def action_totp_invite(self):
        invite_template = self.env.ref('auth_totp_mail.mail_template_totp_invite')
        users_to_invite = self.sudo().filtered(lambda user: not user.totp_secret)
        for user in users_to_invite:
            email_values = {
                'email_from': self.env.user.email_formatted,
                'author_id': self.env.user.partner_id.id,
            }
            invite_template.send_mail(user.id, force_send=True, email_values=email_values,
                                      email_layout_xmlid='mail.mail_notification_light')

        # Display a confirmation toaster
        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'info',
                'sticky': False,
                'message': _("Invitation to use two-factor authentication sent for the following user(s): %s",
                             ', '.join(users_to_invite.mapped('name'))),
            }
        }

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import auth_totp_device
from . import res_users

```

## File: views\res_users_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="ir.ui.view" id="view_users_form">
        <field name="name">res.users.view.form.inherit.auth.totp.mail</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="auth_totp.view_totp_form"/>
        <field name="arch" type="xml">
            <xpath expr="//button[@name='action_totp_enable_wizard']" position="after">
                <button groups="base.group_erp_manager" invisible="id == uid"
                        name="action_totp_invite" string="Invite to use 2FA" type="object" class="btn btn-secondary"/>
            </xpath>
        </field>
    </record>

    <!-- View used when coming from "invite to use 2FA" mail -->
    <record model="ir.ui.view" id="auth_totp_mail.res_users_view_form">
        <field name="name">res.users.view.form.auth.totp.mail</field>
        <field name="model">res.users</field>
        <field name="inherit_id" ref="base.view_users_form_simple_modif"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <form position="attributes">
                <attribute name='create'>0</attribute>
                <attribute name='edit'>0</attribute>
                <attribute name='delete'>0</attribute>
            </form>
            <h1 position="replace"/>
            <xpath expr="//field[@name='image_1920']" position="replace"/>
            <notebook position="replace">
                <header>
                </header>
                <sheet>$0</sheet>
            </notebook>
            <notebook position="before">
                <field name="image_1920" widget="image" class="oe_avatar" options="{'zoom': true, 'preview_image':'image_128'}"/>
                    <div class="oe_title">
                        <h1>
                            <field name="name" placeholder="Name" required="True" readonly="context.get('from_my_profile', False)"/>
                        </h1>
                    </div>
            </notebook>
            <page name="preferences_page" position="replace"></page>
            <footer position="replace"/>
        </field>
    </record>
</odoo>

```

