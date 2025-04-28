# Odoo Module: auth_ldap

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: README.rst

```rst
Adds support for authentication by LDAP server.
===============================================
This module allows users to login with their LDAP username and password, and
will automatically create Odoo users for them on the fly.

**Note:** This module only work on servers that have Python's ``python-ldap`` module installed.

Configuration:
--------------
After installing this module, you need to configure the LDAP parameters in the
General Settings menu. Different companies may have different
LDAP servers, as long as they have unique usernames (usernames need to be unique
in Odoo, even across multiple companies).

Anonymous LDAP binding is also supported (for LDAP servers that allow it), by
simply keeping the LDAP user and password empty in the LDAP configuration.
This does not allow anonymous authentication for users, it is only for the master
LDAP account that is used to verify if a user exists before attempting to
authenticate it.

Securing the connection with STARTTLS is available for LDAP servers supporting
it, by enabling the TLS option in the LDAP configuration.

For further options configuring the LDAP settings, refer to the ldap.conf
manpage: manpage:`ldap.conf(5)`.

Security Considerations:
------------------------
Users' LDAP passwords are never stored in the Odoo database, the LDAP server
is queried whenever a user needs to be authenticated. No duplication of the
password occurs, and passwords are managed in one place only.

Odoo does not manage password changes in the LDAP, so any change of password
should be conducted by other means in the LDAP directory directly (for LDAP users).

It is also possible to have local Odoo users in the database along with
LDAP-authenticated users (the Administrator account is one obvious example).

Here is how it works:
---------------------
    * The system first attempts to authenticate users against the local Odoo
      database;
    * if this authentication fails (for example because the user has no local
      password), the system then attempts to authenticate against LDAP;

As LDAP users have blank passwords by default in the local Odoo database
(which means no access), the first step always fails and the LDAP server is
queried to do the authentication.

Enabling STARTTLS ensures that the authentication query to the LDAP server is
encrypted.

User Template:
--------------
In the LDAP configuration on the General Settings, it is possible to select a *User
Template*. If set, this user will be used as template to create the local users
whenever someone authenticates for the first time via LDAP authentication. This
allows pre-setting the default groups and menus of the first-time users.

**Warning:** if you set a password for the user template, this password will be
         assigned as local password for each new LDAP user, effectively setting
         a *master password* for these users (until manually changed). You
         usually do not want this. One easy way to setup a template user is to
         login once with a valid LDAP user, let Odoo create a blank local
         user with the same login (and a blank password), then rename this new
         user to a username that does not exist in LDAP, and setup its groups

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Authentication via LDAP',
    'depends': ['base', 'base_setup'],
    #'description': < auto-loaded from README file
    'category': 'Hidden/Tools',
    'data': [
        'views/ldap_installer_views.xml',
        'security/ir.model.access.csv',
        'views/res_config_settings_views.xml',
    ],
    'external_dependencies': {
        'python': ['ldap'],
    },
    'license': 'LGPL-3',
}

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = "res.company"

    ldaps = fields.One2many('res.company.ldap', 'company', string='LDAP Parameters',
                               copy=True, groups="base.group_system")

```

## File: models\res_company_ldap.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ldap
import logging
from ldap.filter import filter_format

from odoo import _, api, fields, models, tools
from odoo.exceptions import AccessDenied
from odoo.tools.misc import str2bool
from odoo.tools.pycompat import to_text

_logger = logging.getLogger(__name__)


class LDAPWrapper:
    def __init__(self, obj):
        self.__obj__ = obj

    def passwd_s(self, *args, **kwargs):
        self.__obj__.passwd_s(*args, **kwargs)

    def search_st(self, *args, **kwargs):
        return self.__obj__.search_st(*args, **kwargs)

    def simple_bind_s(self, *args, **kwargs):
        self.__obj__.simple_bind_s(*args, **kwargs)

    def unbind(self, *args, **kwargs):
        self.__obj__.unbind(*args, **kwargs)


class CompanyLDAP(models.Model):
    _name = 'res.company.ldap'
    _description = 'Company LDAP configuration'
    _order = 'sequence'
    _rec_name = 'ldap_server'

    sequence = fields.Integer(default=10)
    company = fields.Many2one('res.company', string='Company', required=True, ondelete='cascade')
    ldap_server = fields.Char(string='LDAP Server address', required=True, default='127.0.0.1')
    ldap_server_port = fields.Integer(string='LDAP Server port', required=True, default=389)
    ldap_binddn = fields.Char('LDAP binddn',
        help="The user account on the LDAP server that is used to query the directory. "
             "Leave empty to connect anonymously.")
    ldap_password = fields.Char(string='LDAP password',
        help="The password of the user account on the LDAP server that is used to query the directory.")
    ldap_filter = fields.Char(string='LDAP filter', required=True, help="""\
    Filter used to look up user accounts in the LDAP database. It is an\
    arbitrary LDAP filter in string representation. Any `%s` placeholder\
    will be replaced by the login (identifier) provided by the user, the filter\
    should contain at least one such placeholder.

    The filter must result in exactly one (1) result, otherwise the login will\
    be considered invalid.

    Example (actual attributes depend on LDAP server and setup):

        (&(objectCategory=person)(objectClass=user)(sAMAccountName=%s))

    or

        (|(mail=%s)(uid=%s))
    """)
    ldap_base = fields.Char(string='LDAP base', required=True, help="DN of the user search scope: all descendants of this base will be searched for users.")
    user = fields.Many2one('res.users', string='Template User',
        help="User to copy when creating new users")
    create_user = fields.Boolean(default=True,
        help="Automatically create local user accounts for new users authenticating via LDAP")
    ldap_tls = fields.Boolean(string='Use TLS',
        help="Request secure TLS/SSL encryption when connecting to the LDAP server. "
             "This option requires a server with STARTTLS enabled, "
             "otherwise all authentication attempts will fail.")

    def _get_ldap_dicts(self):
        """
        Retrieve res_company_ldap resources from the database in dictionary
        format.
        :return: ldap configurations
        :rtype: list of dictionaries
        """

        ldaps = self.sudo().search([('ldap_server', '!=', False)], order='sequence')
        res = ldaps.read([
            'id',
            'company',
            'ldap_server',
            'ldap_server_port',
            'ldap_binddn',
            'ldap_password',
            'ldap_filter',
            'ldap_base',
            'user',
            'create_user',
            'ldap_tls'
        ])
        return res

    def _connect(self, conf):
        """
        Connect to an LDAP server specified by an ldap
        configuration dictionary.

        :param dict conf: LDAP configuration
        :return: an LDAP object
        """

        uri = 'ldap://%s:%d' % (conf['ldap_server'], conf['ldap_server_port'])

        connection = ldap.initialize(uri)
        ldap_chase_ref_disabled = self.env['ir.config_parameter'].sudo().get_param('auth_ldap.disable_chase_ref')
        if str2bool(ldap_chase_ref_disabled):
            connection.set_option(ldap.OPT_REFERRALS, ldap.OPT_OFF)
        if conf['ldap_tls']:
            connection.start_tls_s()
        return LDAPWrapper(connection)

    def _get_entry(self, conf, login):
        filter_tmpl = conf['ldap_filter']
        placeholders = filter_tmpl.count('%s')
        if not placeholders:
            _logger.warning("LDAP filter %r contains no placeholder ('%%s').", filter_tmpl)

        formatted_filter = filter_format(filter_tmpl, [login] * placeholders)
        results = self._query(conf, formatted_filter)

        # Get rid of results (dn, attrs) without a dn
        results = [entry for entry in results if entry[0]]

        dn, entry = False, False
        if len(results) == 1:
            dn, _ = entry = results[0]
        return dn, entry

    def _authenticate(self, conf, login, password):
        """
        Authenticate a user against the specified LDAP server.

        In order to prevent an unintended 'unauthenticated authentication',
        which is an anonymous bind with a valid dn and a blank password,
        check for empty passwords explicitely (:rfc:`4513#section-6.3.1`)
        :param dict conf: LDAP configuration
        :param login: username
        :param password: Password for the LDAP user
        :return: LDAP entry of authenticated user or False
        :rtype: dictionary of attributes
        """

        if not password:
            return False

        dn, entry = self._get_entry(conf, login)
        if not dn:
            return False
        try:
            conn = self._connect(conf)
            conn.simple_bind_s(dn, to_text(password))
            conn.unbind()
        except ldap.INVALID_CREDENTIALS:
            return False
        except ldap.LDAPError as e:
            _logger.error('An LDAP exception occurred: %s', e)
            return False
        return entry

    def _query(self, conf, filter, retrieve_attributes=None):
        """
        Query an LDAP server with the filter argument and scope subtree.

        Allow for all authentication methods of the simple authentication
        method:

        - authenticated bind (non-empty binddn + valid password)
        - anonymous bind (empty binddn + empty password)
        - unauthenticated authentication (non-empty binddn + empty password)

        .. seealso::
           :rfc:`4513#section-5.1` - LDAP: Simple Authentication Method.

        :param dict conf: LDAP configuration
        :param filter: valid LDAP filter
        :param list retrieve_attributes: LDAP attributes to be retrieved. \
        If not specified, return all attributes.
        :return: ldap entries
        :rtype: list of tuples (dn, attrs)

        """

        results = []
        try:
            conn = self._connect(conf)
            ldap_password = conf['ldap_password'] or ''
            ldap_binddn = conf['ldap_binddn'] or ''
            conn.simple_bind_s(to_text(ldap_binddn), to_text(ldap_password))
            results = conn.search_st(to_text(conf['ldap_base']), ldap.SCOPE_SUBTREE, filter, retrieve_attributes, timeout=60)
            conn.unbind()
        except ldap.INVALID_CREDENTIALS:
            _logger.error('LDAP bind failed.')
        except ldap.LDAPError as e:
            _logger.error('An LDAP exception occurred: %s', e)
        return results

    def _map_ldap_attributes(self, conf, login, ldap_entry):
        """
        Compose values for a new resource of model res_users,
        based upon the retrieved ldap entry and the LDAP settings.
        :param dict conf: LDAP configuration
        :param login: the new user's login
        :param tuple ldap_entry: single LDAP result (dn, attrs)
        :return: parameters for a new resource of model res_users
        :rtype: dict
        """
        data = {
            'name': tools.ustr(ldap_entry[1]['cn'][0]),
            'login': login,
            'company_id': conf['company'][0]
        }
        if tools.single_email_re.match(login):
            data['email'] = login
        return data

    def _get_or_create_user(self, conf, login, ldap_entry):
        """
        Retrieve an active resource of model res_users with the specified
        login. Create the user if it is not initially found.

        :param dict conf: LDAP configuration
        :param login: the user's login
        :param tuple ldap_entry: single LDAP result (dn, attrs)
        :return: res_users id
        :rtype: int
        """
        login = tools.ustr(login.lower().strip())
        self.env.cr.execute("SELECT id, active FROM res_users WHERE lower(login)=%s", (login,))
        res = self.env.cr.fetchone()
        if res:
            if res[1]:
                return res[0]
        elif conf['create_user']:
            _logger.debug("Creating new Odoo user \"%s\" from LDAP" % login)
            values = self._map_ldap_attributes(conf, login, ldap_entry)
            SudoUser = self.env['res.users'].sudo().with_context(no_reset_password=True)
            if conf['user']:
                values['active'] = True
                return SudoUser.browse(conf['user'][0]).copy(default=values).id
            else:
                return SudoUser.create(values).id

        raise AccessDenied(_("No local user found for LDAP login and not configured to create one"))

    def _change_password(self, conf, login, old_passwd, new_passwd):
        changed = False
        dn, entry = self._get_entry(conf, login)
        if not dn:
            return False
        try:
            conn = self._connect(conf)
            conn.simple_bind_s(dn, to_text(old_passwd))
            conn.passwd_s(dn, old_passwd, new_passwd)
            changed = True
            conn.unbind()
        except ldap.INVALID_CREDENTIALS:
            pass
        except ldap.LDAPError as e:
            _logger.error('An LDAP exception occurred: %s', e)
        return changed

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    ldaps = fields.One2many(related='company_id.ldaps', string="LDAP Parameters", readonly=False)

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.exceptions import AccessDenied

from odoo import api, models, registry, SUPERUSER_ID


class Users(models.Model):
    _inherit = "res.users"

    @classmethod
    def _login(cls, db, login, password, user_agent_env):
        try:
            return super(Users, cls)._login(db, login, password, user_agent_env=user_agent_env)
        except AccessDenied as e:
            with registry(db).cursor() as cr:
                cr.execute("SELECT id FROM res_users WHERE lower(login)=%s", (login,))
                res = cr.fetchone()
                if res:
                    raise e

                env = api.Environment(cr, SUPERUSER_ID, {})
                Ldap = env['res.company.ldap']
                for conf in Ldap._get_ldap_dicts():
                    entry = Ldap._authenticate(conf, login, password)
                    if entry:
                        return Ldap._get_or_create_user(conf, login, entry)
                raise e

    def _check_credentials(self, password, env):
        try:
            return super(Users, self)._check_credentials(password, env)
        except AccessDenied:
            passwd_allowed = env['interactive'] or not self.env.user._rpc_api_keys_only()
            if passwd_allowed and self.env.user.active:
                Ldap = self.env['res.company.ldap']
                for conf in Ldap._get_ldap_dicts():
                    if Ldap._authenticate(conf, self.env.user.login, password):
                        return
            raise

    @api.model
    def change_password(self, old_passwd, new_passwd):
        if new_passwd:
            Ldap = self.env['res.company.ldap']
            for conf in Ldap._get_ldap_dicts():
                changed = Ldap._change_password(conf, self.env.user.login, old_passwd, new_passwd)
                if changed:
                    self.env.user._set_empty_password()
                    return True
        return super(Users, self).change_password(old_passwd, new_passwd)

    def _set_empty_password(self):
        self.flush_recordset(['password'])
        self.env.cr.execute(
            'UPDATE res_users SET password=NULL WHERE id=%s',
            (self.id,)
        )
        self.invalidate_recordset(['password'])

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_company
from . import res_company_ldap
from . import res_users
from . import res_config_settings
```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_res_company_ldap,res_company_ldap,model_res_company_ldap,base.group_system,1,1,1,1

```

## File: views\ldap_installer_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- ldap installer Form View -->
        <record id="view_ldap_installer_form" model="ir.ui.view">
            <field name="name">res.company.ldap.form</field>
            <field name="model">res.company.ldap</field>
            <field name="arch" type="xml">
                <form string="LDAP Configuration">
                    <group>
                        <field name="company"/>
                        <newline/>
                        <group string="Server Information">
                              <field name="ldap_server"/>
                              <field name="ldap_server_port"/>
                              <field name="ldap_tls"/>
                        </group>
                        <group string="Login Information">
                              <field name="ldap_binddn"/>
                              <field name="ldap_password" password="True"/>
                        </group>
                        <group string="Process Parameter">
                             <field name="ldap_base"/>
                             <field name="ldap_filter"/>
                             <field name="sequence"/>
                        </group>
                        <group string="User Information">
                             <field name="create_user"/>
                             <field name="user"/>
                        </group>
                    </group>
                </form>
            </field>
        </record>

        <record id="res_company_ldap_view_tree" model="ir.ui.view">
            <field name="name">res.company.ldap.tree</field>
            <field name="model">res.company.ldap</field>
            <field name="arch" type="xml">
                <tree string="LDAP Configuration">
                    <field name="company"/>
                    <field name="ldap_server"/>
                    <field name="ldap_server_port"/>
                </tree>
            </field>
        </record>

        <!-- ldap installer  action -->
        <record id="action_ldap_installer" model="ir.actions.act_window">
             <field name="name">Setup your LDAP Server</field>
             <field name="type">ir.actions.act_window</field>
             <field name="res_model">res.company.ldap</field>
             <field name="view_mode">tree,form</field>
        </record>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.auth.ldap</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div name='auth_ldap_right_pane' position="inside">
                <div class="mt8" attrs="{'invisible': [('module_auth_ldap','=',False)]}">
                   <button type="action" name="%(auth_ldap.action_ldap_installer)d" string="LDAP Server" icon="fa-arrow-right" class="btn-link"/>
                </div>
            </div>
            <div id="auth_ldap_warning" position="replace"/>
        </field>
    </record>
</odoo>

```

