# Odoo Module: fetchmail_outlook

Category: Hidden

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name": "Fetchmail Outlook",
    "version": "1.0",
    "category": "Hidden",
    "description": "OAuth authentication for incoming Outlook mail server",
    "depends": [
        "microsoft_outlook",
        "fetchmail",
    ],
    "data": [
        "views/fetchmail_server_views.xml",
    ],
    "auto_install": True,
}

```

## File: models\fetchmail_server.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models
from odoo.exceptions import UserError


class FetchmailServer(models.Model):
    """Add the Outlook OAuth authentication on the incoming mail servers."""

    _name = 'fetchmail.server'
    _inherit = ['fetchmail.server', 'microsoft.outlook.mixin']

    _OUTLOOK_SCOPE = 'https://outlook.office.com/IMAP.AccessAsUser.All'

    @api.constrains('use_microsoft_outlook_service', 'server_type', 'password', 'is_ssl')
    def _check_use_microsoft_outlook_service(self):
        for server in self:
            if not server.use_microsoft_outlook_service:
                continue

            if server.server_type != 'imap':
                raise UserError(_('Outlook mail server %r only supports IMAP server type.') % server.name)

            if server.password:
                raise UserError(_(
                    'Please leave the password field empty for Outlook mail server %r. '
                    'The OAuth process does not require it')
                    % server.name)

            if not server.is_ssl:
                raise UserError(_('SSL is required .') % server.name)

    @api.onchange('use_microsoft_outlook_service')
    def _onchange_use_microsoft_outlook_service(self):
        """Set the default configuration for a IMAP Outlook server."""
        if self.use_microsoft_outlook_service:
            self.server = 'imap.outlook.com'
            self.server_type = 'imap'
            self.is_ssl = True
            self.port = 993
        else:
            self.microsoft_outlook_refresh_token = False
            self.microsoft_outlook_access_token = False
            self.microsoft_outlook_access_token_expiration = False

    def _imap_login(self, connection):
        """Authenticate the IMAP connection.

        If the mail server is Outlook, we use the OAuth2 authentication protocol.
        """
        self.ensure_one()
        if self.use_microsoft_outlook_service:
            auth_string = self._generate_outlook_oauth2_string(self.user)
            connection.authenticate('XOAUTH2', lambda x: auth_string)
            connection.select('INBOX')
        else:
            super()._imap_login(connection)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import fetchmail_server

```

## File: views\fetchmail_server_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fetchmail_server_view_form" model="ir.ui.view">
        <field name="name">fetchmail.server.view.form.inherit.outlook</field>
        <field name="model">fetchmail.server</field>
        <field name="priority">1000</field>
        <field name="inherit_id" ref="fetchmail.view_email_server_form"/>
        <field name="arch" type="xml">
            <field name="server" position="before">
                <field name="use_microsoft_outlook_service" string="Outlook"
                    attrs="{'readonly': [('state', '=', 'done')]}"/>
            </field>
            <field name="user" position="after">
                <field name="is_microsoft_outlook_configured" invisible="1"/>
                <field name="microsoft_outlook_refresh_token" invisible="1"/>
                <field name="microsoft_outlook_access_token" invisible="1"/>
                <field name="microsoft_outlook_access_token_expiration" invisible="1"/>
                <div></div>
                <div attrs="{'invisible': [('use_microsoft_outlook_service', '=', False)]}">
                    <span attrs="{'invisible': ['|', ('use_microsoft_outlook_service', '=', False), ('microsoft_outlook_refresh_token', '=', False)]}"
                        class="badge badge-success">
                        Outlook Token Valid
                    </span>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0"
                        attrs="{'invisible': ['|', '|', '|', ('is_microsoft_outlook_configured', '=', False), ('use_microsoft_outlook_service', '=', False), ('microsoft_outlook_refresh_token', '!=', False)]}">
                        <i class="fa fa-arrow-right"/>
                        Connect your Outlook account
                    </button>
                    <button type="object"
                        name="open_microsoft_outlook_uri" class="btn-link px-0"
                        attrs="{'invisible': ['|', '|', '|', ('is_microsoft_outlook_configured', '=', False), ('use_microsoft_outlook_service', '=', False), ('microsoft_outlook_refresh_token', '=', False)]}">
                        <i class="fa fa-cog"/>
                        Edit Settings
                    </button>
                    <div class="alert alert-warning" role="alert"
                        attrs="{'invisible': ['|', ('is_microsoft_outlook_configured', '=', True), ('use_microsoft_outlook_service', '=', False)]}">
                        Setup your Outlook API credentials in the general settings to link a Outlook account.
                    </div>
                </div>
            </field>
            <field name="password" position="attributes">
                <attribute name="attrs">{}</attribute>
            </field>
        </field>
    </record>
</odoo>

```

