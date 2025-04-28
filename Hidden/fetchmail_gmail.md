# Odoo Module: fetchmail_gmail

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
    "name": "Fetchmail Gmail",
    "version": "1.0",
    "category": "Hidden",
    "description": "Google authentication for incoming mail server",
    "depends": [
        "google_gmail",
        "fetchmail",
    ],
    "data": ["views/fetchmail_server_views.xml"],
    "auto_install": True,
}

```

## File: models\fetchmail_server.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, _
from odoo.exceptions import UserError


class FetchmailServer(models.Model):
    _name = 'fetchmail.server'
    _inherit = ['fetchmail.server', 'google.gmail.mixin']

    @api.constrains('use_google_gmail_service', 'server_type')
    def _check_use_google_gmail_service(self):
        if any(server.use_google_gmail_service and server.server_type != 'imap' for server in self):
            raise UserError(_('Gmail authentication only supports IMAP server type.'))

    @api.onchange('use_google_gmail_service')
    def _onchange_use_google_gmail_service(self):
        """Set the default configuration for a IMAP Gmail server."""
        if self.use_google_gmail_service:
            self.server = 'imap.gmail.com'
            self.server_type = 'imap'
            self.is_ssl = True
            self.port = 993
        else:
            self.google_gmail_authorization_code = False
            self.google_gmail_refresh_token = False
            self.google_gmail_access_token = False
            self.google_gmail_access_token_expiration = False

    def _imap_login(self, connection):
        """Authenticate the IMAP connection.

        If the mail server is Gmail, we use the OAuth2 authentication protocol.
        """
        self.ensure_one()
        if self.use_google_gmail_service:
            auth_string = self._generate_oauth2_string(self.user, self.google_gmail_refresh_token)
            connection.authenticate('XOAUTH2', lambda x: auth_string)
            connection.select('INBOX')
        else:
            super(FetchmailServer, self)._imap_login(connection)

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
        <field name="name">fetchmail.server.view.form.inherit.gmail</field>
        <field name="model">fetchmail.server</field>
        <field name="priority">100</field>
        <field name="inherit_id" ref="fetchmail.view_email_server_form"/>
        <field name="arch" type="xml">
            <field name="server" position="before">
                <field name="use_google_gmail_service" string="Gmail" attrs="{'readonly': [('state', '=', 'done')]}"/>
            </field>
            <field name="user" position="after">
                <field name="google_gmail_uri" invisible="1"/>
                <field name="google_gmail_refresh_token" invisible="1"/>
                <div></div>
                <div attrs="{'invisible': [('use_google_gmail_service', '=', False)]}">
                    <span attrs="{'invisible': ['|', ('use_google_gmail_service', '=', False), ('google_gmail_refresh_token', '=', False)]}"
                        class="badge badge-success">
                        Gmail Token Valid
                    </span>
                    <button type="object"
                        name="open_google_gmail_uri" class="btn-link px-0"
                        attrs="{'invisible': ['|', '|', ('google_gmail_uri', '=', False), ('use_google_gmail_service', '=', False), ('google_gmail_refresh_token', '!=', False)]}">
                        <i class="fa fa-arrow-right"/>
                        Connect your Gmail account
                    </button>
                    <button type="object"
                        name="open_google_gmail_uri" class="btn-link px-0"
                        attrs="{'invisible': ['|', '|', ('google_gmail_uri', '=', False), ('use_google_gmail_service', '=', False), ('google_gmail_refresh_token', '=', False)]}">
                        <i class="fa fa-cog"/>
                        Edit Settings
                    </button>
                    <div class="alert alert-warning" role="alert"
                        attrs="{'invisible': ['|', ('use_google_gmail_service', '=', False), ('google_gmail_uri', '!=', False)]}">
                        Setup your Gmail API credentials in the general settings to link a Gmail account.
                    </div>
                </div>
            </field>
            <field name="password" position="attributes">
                <attribute name="attrs">{'required' : [('server_type', '!=', 'local'), ('use_google_gmail_service', '=', False), ('password', '!=', False)], 'invisible' : [('use_google_gmail_service', '=', True)]}</attribute>
            </field>
        </field>
    </record>
</odoo>

```

