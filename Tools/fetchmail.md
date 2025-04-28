# Odoo Module: fetchmail

Category: Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
#-*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Email Gateway',
    'version': '1.0',
    'depends': ['mail'],
    'category': 'Tools',
    'description': """
Retrieve incoming email on POP/IMAP servers.
============================================

Enter the parameters of your POP/IMAP account(s), and any incoming emails on
these accounts will be automatically downloaded into your Odoo system. All
POP3/IMAP-compatible servers are supported, included those that require an
encrypted SSL/TLS connection.

This can be used to easily create email-based workflows for many email-enabled Odoo documents, such as:
----------------------------------------------------------------------------------------------------------
    * CRM Leads/Opportunities
    * CRM Claims
    * Project Issues
    * Project Tasks
    * Human Resource Recruitments (Applicants)

Just install the relevant application, and you can assign any of these document
types (Leads, Project Issues) to your incoming email accounts. New emails will
automatically spawn new documents of the chosen type, so it's a snap to create a
mailbox-to-Odoo integration. Even better: these documents directly act as mini
conversations synchronized by email. You can reply from within Odoo, and the
answers will automatically be collected when they come back, and attached to the
same *conversation* document.

For more specific needs, you may also assign custom-defined actions
(technically: Server Actions) to be triggered for each incoming mail.
    """,
    'data': [
        'data/fetchmail_data.xml',
        'security/ir.model.access.csv',
        'views/fetchmail_views.xml',
        'views/mail_mail_views.xml',
        'views/res_config_settings_views.xml',
    ],
    'demo': [],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\fetchmail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="ir_cron_mail_gateway_action" model="ir.cron">
            <field name="name">Mail: Fetchmail Service</field>
            <field name="model_id" ref="model_fetchmail_server"/>
            <field name="state">code</field>
            <field name="code">model._fetch_mails()</field>
            <field name="interval_number">5</field>
            <field name="interval_type">minutes</field>
            <field name="numbercall">-1</field>
            <field name="doall" eval="False"/>
            <!-- Active flag is set on fetchmail_server.create/write -->
            <field name="active" eval="False"/>
        </record>
    </data>
</odoo>

```

## File: models\fetchmail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import poplib
import socket
from imaplib import IMAP4, IMAP4_SSL
from poplib import POP3, POP3_SSL

from odoo import api, fields, models, tools, _
from odoo.exceptions import UserError


_logger = logging.getLogger(__name__)
MAX_POP_MESSAGES = 50
MAIL_TIMEOUT = 60

# Workaround for Python 2.7.8 bug https://bugs.python.org/issue23906
poplib._MAXLINE = 65536

# Add timeout to IMAP connections
# HACK https://bugs.python.org/issue38615
# TODO: clean in Python 3.9
IMAP4._create_socket = lambda self, timeout=MAIL_TIMEOUT: socket.create_connection((self.host or None, self.port), timeout)


class FetchmailServer(models.Model):
    """Incoming POP/IMAP mail server account"""

    _name = 'fetchmail.server'
    _description = 'Incoming Mail Server'
    _order = 'priority'

    name = fields.Char('Name', required=True)
    active = fields.Boolean('Active', default=True)
    state = fields.Selection([
        ('draft', 'Not Confirmed'),
        ('done', 'Confirmed'),
    ], string='Status', index=True, readonly=True, copy=False, default='draft')
    server = fields.Char(string='Server Name', readonly=True, help="Hostname or IP of the mail server", states={'draft': [('readonly', False)]})
    port = fields.Integer(readonly=True, states={'draft': [('readonly', False)]})
    server_type = fields.Selection([
        ('pop', 'POP Server'),
        ('imap', 'IMAP Server'),
        ('local', 'Local Server'),
    ], string='Server Type', index=True, required=True, default='pop')
    is_ssl = fields.Boolean('SSL/TLS', help="Connections are encrypted with SSL/TLS through a dedicated port (default: IMAPS=993, POP3S=995)")
    attach = fields.Boolean('Keep Attachments', help="Whether attachments should be downloaded. "
                                                     "If not enabled, incoming emails will be stripped of any attachments before being processed", default=True)
    original = fields.Boolean('Keep Original', help="Whether a full original copy of each email should be kept for reference "
                                                    "and attached to each processed message. This will usually double the size of your message database.")
    date = fields.Datetime(string='Last Fetch Date', readonly=True)
    user = fields.Char(string='Username', readonly=True, states={'draft': [('readonly', False)]})
    password = fields.Char(readonly=True, states={'draft': [('readonly', False)]})
    object_id = fields.Many2one('ir.model', string="Create a New Record", help="Process each incoming mail as part of a conversation "
                                                                                "corresponding to this document type. This will create "
                                                                                "new documents for new conversations, or attach follow-up "
                                                                                "emails to the existing conversations (documents).")
    priority = fields.Integer(string='Server Priority', readonly=True, states={'draft': [('readonly', False)]}, help="Defines the order of processing, lower values mean higher priority", default=5)
    message_ids = fields.One2many('mail.mail', 'fetchmail_server_id', string='Messages', readonly=True)
    configuration = fields.Text('Configuration', readonly=True)
    script = fields.Char(readonly=True, default='/mail/static/scripts/odoo-mailgate.py')

    @api.onchange('server_type', 'is_ssl', 'object_id')
    def onchange_server_type(self):
        self.port = 0
        if self.server_type == 'pop':
            self.port = self.is_ssl and 995 or 110
        elif self.server_type == 'imap':
            self.port = self.is_ssl and 993 or 143

        conf = {
            'dbname': self.env.cr.dbname,
            'uid': self.env.uid,
            'model': self.object_id.model if self.object_id else 'MODELNAME'
        }
        self.configuration = """Use the below script with the following command line options with your Mail Transport Agent (MTA)
odoo-mailgate.py --host=HOSTNAME --port=PORT -u %(uid)d -p PASSWORD -d %(dbname)s
Example configuration for the postfix mta running locally:
/etc/postfix/virtual_aliases: @youdomain odoo_mailgate@localhost
/etc/aliases:
odoo_mailgate: "|/path/to/odoo-mailgate.py --host=localhost -u %(uid)d -p PASSWORD -d %(dbname)s"
        """ % conf

    @api.model
    def create(self, values):
        res = super(FetchmailServer, self).create(values)
        self._update_cron()
        return res

    def write(self, values):
        res = super(FetchmailServer, self).write(values)
        self._update_cron()
        return res

    def unlink(self):
        res = super(FetchmailServer, self).unlink()
        self._update_cron()
        return res

    def set_draft(self):
        self.write({'state': 'draft'})
        return True

    def connect(self):
        self.ensure_one()
        if self.server_type == 'imap':
            if self.is_ssl:
                connection = IMAP4_SSL(self.server, int(self.port))
            else:
                connection = IMAP4(self.server, int(self.port))
            self._imap_login(connection)
        elif self.server_type == 'pop':
            if self.is_ssl:
                connection = POP3_SSL(self.server, int(self.port), timeout=MAIL_TIMEOUT)
            else:
                connection = POP3(self.server, int(self.port), timeout=MAIL_TIMEOUT)
            #TODO: use this to remove only unread messages
            #connection.user("recent:"+server.user)
            connection.user(self.user)
            connection.pass_(self.password)
        return connection

    def _imap_login(self, connection):
        """Authenticate the IMAP connection.

        Can be overridden in other module for different authentication methods.

        :param connection: The IMAP connection to authenticate
        """
        self.ensure_one()
        connection.login(self.user, self.password)

    def button_confirm_login(self):
        for server in self:
            try:
                connection = server.connect()
                server.write({'state': 'done'})
            except Exception as err:
                _logger.info("Failed to connect to %s server %s.", server.server_type, server.name, exc_info=True)
                raise UserError(_("Connection test failed: %s") % tools.ustr(err))
            finally:
                try:
                    if connection:
                        if server.server_type == 'imap':
                            connection.close()
                        elif server.server_type == 'pop':
                            connection.quit()
                except Exception:
                    # ignored, just a consequence of the previous exception
                    pass
        return True

    @api.model
    def _fetch_mails(self):
        """ Method called by cron to fetch mails from servers """
        return self.search([('state', '=', 'done'), ('server_type', 'in', ['pop', 'imap'])]).fetch_mail()

    def fetch_mail(self):
        """ WARNING: meant for cron usage only - will commit() after each email! """
        additionnal_context = {
            'fetchmail_cron_running': True
        }
        MailThread = self.env['mail.thread']
        for server in self:
            _logger.info('start checking for new emails on %s server %s', server.server_type, server.name)
            additionnal_context['default_fetchmail_server_id'] = server.id
            additionnal_context['server_type'] = server.server_type
            count, failed = 0, 0
            imap_server = None
            pop_server = None
            if server.server_type == 'imap':
                try:
                    imap_server = server.connect()
                    imap_server.select()
                    result, data = imap_server.search(None, '(UNSEEN)')
                    for num in data[0].split():
                        res_id = None
                        result, data = imap_server.fetch(num, '(RFC822)')
                        imap_server.store(num, '-FLAGS', '\\Seen')
                        try:
                            res_id = MailThread.with_context(**additionnal_context).message_process(server.object_id.model, data[0][1], save_original=server.original, strip_attachments=(not server.attach))
                        except Exception:
                            _logger.info('Failed to process mail from %s server %s.', server.server_type, server.name, exc_info=True)
                            failed += 1
                        imap_server.store(num, '+FLAGS', '\\Seen')
                        self._cr.commit()
                        count += 1
                    _logger.info("Fetched %d email(s) on %s server %s; %d succeeded, %d failed.", count, server.server_type, server.name, (count - failed), failed)
                except Exception:
                    _logger.info("General failure when trying to fetch mail from %s server %s.", server.server_type, server.name, exc_info=True)
                finally:
                    if imap_server:
                        imap_server.close()
                        imap_server.logout()
            elif server.server_type == 'pop':
                try:
                    while True:
                        failed_in_loop = 0
                        num = 0
                        pop_server = server.connect()
                        (num_messages, total_size) = pop_server.stat()
                        pop_server.list()
                        for num in range(1, min(MAX_POP_MESSAGES, num_messages) + 1):
                            (header, messages, octets) = pop_server.retr(num)
                            message = (b'\n').join(messages)
                            res_id = None
                            try:
                                res_id = MailThread.with_context(**additionnal_context).message_process(server.object_id.model, message, save_original=server.original, strip_attachments=(not server.attach))
                                pop_server.dele(num)
                            except Exception:
                                _logger.info('Failed to process mail from %s server %s.', server.server_type, server.name, exc_info=True)
                                failed += 1
                                failed_in_loop += 1
                            self.env.cr.commit()
                        _logger.info("Fetched %d email(s) on %s server %s; %d succeeded, %d failed.", num, server.server_type, server.name, (num - failed_in_loop), failed_in_loop)
                        # Stop if (1) no more message left or (2) all messages have failed
                        if num_messages < MAX_POP_MESSAGES or failed_in_loop == num:
                            break
                        pop_server.quit()
                except Exception:
                    _logger.info("General failure when trying to fetch mail from %s server %s.", server.server_type, server.name, exc_info=True)
                finally:
                    if pop_server:
                        pop_server.quit()
            server.write({'date': fields.Datetime.now()})
        return True

    @api.model
    def _update_cron(self):
        if self.env.context.get('fetchmail_cron_running'):
            return
        try:
            # Enabled/Disable cron based on the number of 'done' server of type pop or imap
            cron = self.env.ref('fetchmail.ir_cron_mail_gateway_action')
            cron.toggle(model=self._name, domain=[('state', '=', 'done'), ('server_type', 'in', ['pop', 'imap'])])
        except ValueError:
            pass

```

## File: models\mail_mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class MailMail(models.Model):
    _inherit = 'mail.mail'

    fetchmail_server_id = fields.Many2one('fetchmail.server', "Inbound Mail Server", readonly=True, index=True)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import fetchmail
from . import mail_mail

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_fetchmail_server,fetchmail.server,model_fetchmail_server,base.group_system,1,1,1,1

```

## File: views\fetchmail_views.xml

```xml
<?xml version="1.0"?>
<odoo>

        <record id="view_email_server_tree" model="ir.ui.view">
            <field name="name">fetchmail.server.list</field>
            <field name="model">fetchmail.server</field>
            <field name="arch" type="xml">
                <tree decoration-info="state == 'draft'" string="POP/IMAP Servers">
                    <field name="name"/>
                    <field name="server_type"/>
                    <field name="is_ssl"/>
                    <field name="object_id"/>
                    <field name="date"/>
                    <field name="message_ids" string="Email Count"/>
                    <field name="state"/>
                </tree>
            </field>
        </record>

        <record id="view_email_server_form" model="ir.ui.view">
            <field name="name">fetchmail.server.form</field>
            <field name="model">fetchmail.server</field>
            <field name="arch" type="xml">
                <form string="Incoming Mail Server">
                    <header attrs="{'invisible' : [('server_type', '=', 'local')]}">
                        <button string="Test &amp; Confirm" type="object" name="button_confirm_login" states="draft"/>
                        <button string="Fetch Now" type="object" name="fetch_mail" states="done"/>
                        <button string="Reset Confirmation" type="object" name="set_draft" states="done"/>
                        <field name="state" widget="statusbar"/>
                    </header>
                    <sheet>
                     <group col="4">
                        <field name="name"/>
                        <field name="server_type"/>
                        <field name="date"/>
                     </group>
                     <notebook>
                        <page string="Server &amp; Login">
                            <group>
                                <group attrs="{'invisible' : [('server_type', '=', 'local')]}" string="Server Information">
                                    <field name="server" colspan="2" attrs="{'required' : [('server_type', '!=', 'local')]}" />
                                    <field name="port" required="1" attrs="{'required' : [('server_type', '!=', 'local')]}" />
                                    <field name="is_ssl"/>
                                </group>
                                <group attrs="{'invisible' : [('server_type', '=', 'local')]}" string="Login Information">
                                    <field name="user" attrs="{'required' : [('server_type', '!=', 'local')]}"/>
                                    <field name="password" password="True" attrs="{'required' : [('server_type', '!=', 'local')]}"/>
                                </group>
                                <group string="Actions to Perform on Incoming Mails">
                                    <field name="object_id"/>
                                </group>
                                <group attrs="{'invisible' : [('server_type', '!=', 'local')]}" string="Configuration">
                                    <field name="configuration"/>
                                    <field name="script" widget="url"/>
                                </group>
                            </group>
                        </page>
                        <page string="Advanced" groups="base.group_no_one">
                            <group string="Advanced Options" col="4">
                                <field name="priority"/>
                                <field name="attach"/>
                                <field name="original"/>
                                <field name="active"/>
                            </group>
                        </page>
                    </notebook>
                  </sheet>
                </form>
            </field>
        </record>

        <record id="view_email_server_search" model="ir.ui.view">
            <field name="name">fetchmail.server.search</field>
            <field name="model">fetchmail.server</field>
            <field name="arch" type="xml">
                <search string="Search Incoming Mail Servers">
                    <field name="name" string="Incoming Mail Server"/>
                    <filter string="IMAP" name="imap" domain="[('server_type', '=', 'imap')]" help="Server type IMAP."/>
                    <filter string="POP" name="pop" domain="[('server_type', '=', 'pop')]" help="Server type POP."/>
                    <separator/>
                    <filter string="SSL" name="ssl" domain="[('is_ssl', '=', True)]" help="If SSL required."/>
                    <separator/>
                    <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                </search>
            </field>
        </record>

        <record id="action_email_server_tree" model="ir.actions.act_window">
            <field name="name">Incoming Mail Servers</field>
            <field name="res_model">fetchmail.server</field>
            <field name="view_mode">tree,form</field>
            <field name="view_id" ref="view_email_server_tree"/>
            <field name="search_view_id" ref="view_email_server_search"/>
        </record>

        <menuitem
            parent="base.menu_email"
            id="menu_action_fetchmail_server_tree"
            action="action_email_server_tree"
            name="Incoming Mail Servers"
            sequence="14"
            groups="base.group_no_one"
        />

</odoo>

```

## File: views\mail_mail_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record  id="email_message_tree_view" model="ir.ui.view">
        <field name="name">mail.mail.form.fetchmail</field>
        <field name="model">mail.mail</field>
        <field name="inherit_id" ref="mail.view_mail_form"/>
        <field name="arch" type="xml">
            <field name="references" position="after">
                <field name="fetchmail_server_id"/>
            </field>
        </field>
    </record>

    <act_window
        id="act_server_history"
        name="Messages"
        res_model="mail.mail"
        binding_model="fetchmail.server"
        binding_views="form"
        domain="[('email_from', '!=', False), ('fetchmail_server_id', '=', active_id)]"
        context="{'search_default_server_id': active_id, 'default_fetchmail_server_id': active_id}"/>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.fetchmail</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="mail.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="mail_alias_domain" position="after">
                <div class="mt8">
                    <button type="action"
                    name="%(action_email_server_tree)d"
                    string="Incoming Email Servers" icon="fa-arrow-right" class="btn-link"/>
                </div>
                <div class="mt8">
                    <button type="action"
                    name="%(base.action_ir_mail_server_list)d"
                    string="Outgoing Email Servers" icon="fa-arrow-right" class="btn-link"/>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

