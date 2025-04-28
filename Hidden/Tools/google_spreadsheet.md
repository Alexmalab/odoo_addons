# Odoo Module: google_spreadsheet

Category: Hidden/Tools

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
    'name': 'Google Spreadsheet',
    'version': '1.0',
    'category': 'Hidden/Tools',
    'description': """
The module adds the possibility to display data from Odoo in Google Spreadsheets in real time.
=================================================================================================
""",
    'depends': ['google_drive'],
    'data': [
        'data/google_spreadsheet_data.xml',
        'views/google_spreadsheet_views.xml',
        'views/google_spreadsheet_templates.xml',
        'views/res_config_settings_views.xml',
    ],
    'qweb': ['static/src/xml/*.xml'],
    'demo': [],
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}

```

## File: data\google_spreadsheet_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="google_spreadsheet_template" model="google.drive.config">
            <field name="name">Base Spreadsheet Template</field>
            <field name="model_id" ref="base.model_res_partner"/>
            <field name="google_drive_template_url">https://docs.google.com/spreadsheet/ccc?key=1KUlSSzZMzkEEzyroyZ2NerRt1RVCspkG9VQIh2V6dnM</field>
            <field name="name_template">Reporting %(name)s</field>
            <field name="active" eval="0" />
        </record>

    </data>
</odoo>

```

## File: models\google_drive.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json
import logging

import requests
from lxml import etree
import re
import werkzeug.urls

from odoo import api, models
from odoo.tools import misc
from odoo.addons.google_account import TIMEOUT

_logger = logging.getLogger(__name__)


class GoogleDrive(models.Model):
    _inherit = 'google.drive.config'

    def get_google_scope(self):
        scope = super(GoogleDrive, self).get_google_scope()
        return '%s https://www.googleapis.com/auth/spreadsheets' % scope

    @api.model
    def write_config_formula(self, attachment_id, spreadsheet_key, model, domain, groupbys, view_id):
        access_token = self.get_access_token(scope='https://www.googleapis.com/auth/spreadsheets')

        formula = self._get_data_formula(model, domain, groupbys, view_id)

        url = self.env['ir.config_parameter'].sudo().get_param('web.base.url')
        dbname = self._cr.dbname
        user = self.env['res.users'].browse(self.env.user.id).read(['login', 'password'])[0]
        username = user['login']
        password = user['password']
        if not password:
            config_formula = '=oe_settings("%s";"%s")' % (url, dbname)
        else:
            config_formula = '=oe_settings("%s";"%s";"%s";"%s")' % (url, dbname, username, password)
        request = {
            "valueInputOption": "USER_ENTERED",
            "data": [
                {"range": "A1", "values": [[formula]]},
                {"range": "O60", "values": [[config_formula]]},
            ]
        }
        try:
            req = requests.post(
                'https://sheets.googleapis.com/v4/spreadsheets/%s/values:batchUpdate?%s' % (spreadsheet_key, werkzeug.urls.url_encode({'access_token': access_token})),
                data=json.dumps(request),
                headers={'content-type': 'application/json', 'If-Match': '*'},
                timeout=TIMEOUT,
            )
        except IOError:
            _logger.warning("An error occured while writing the formula on the Google Spreadsheet.")

        description = '''
        formula: %s
        ''' % formula
        if attachment_id:
            self.env['ir.attachment'].browse(attachment_id).write({'description': description})
        return True

    def _get_data_formula(self, model, domain, groupbys, view_id):
        fields = self.env[model].fields_view_get(view_id=view_id, view_type='tree')
        doc = etree.XML(fields.get('arch'))
        display_fields = []
        for node in doc.xpath("//field"):
            if node.get('modifiers'):
                modifiers = json.loads(node.get('modifiers'))
                if not modifiers.get('invisible') and not modifiers.get('column_invisible'):
                    display_fields.append(node.get('name'))
        fields = " ".join(display_fields)
        domain = domain.replace("'", r"\'").replace('"', "'").replace('True', 'true').replace('False', 'false')
        if groupbys:
            fields = "%s %s" % (groupbys, fields)
            formula = '=oe_read_group("%s";"%s";"%s";"%s")' % (model, fields, groupbys, domain)
        else:
            formula = '=oe_browse("%s";"%s";"%s")' % (model, fields, domain)
        return formula

    @api.model
    def set_spreadsheet(self, model, domain, groupbys, view_id):
        try:
            config_id = self.env['ir.model.data'].get_object_reference('google_spreadsheet', 'google_spreadsheet_template')[1]
        except ValueError:
            raise
        config = self.browse(config_id)

        if self._module_deprecated():
            return {
                'url': config.google_drive_template_url,
                'deprecated': True,
                'formula': self._get_data_formula(model, domain, groupbys, view_id),
            }

        title = 'Spreadsheet %s' % model
        res = self.copy_doc(False, config.google_drive_resource_id, title, model)

        mo = re.search("(key=|/d/)([A-Za-z0-9-_]+)", res['url'])
        if mo:
            key = mo.group(2)

        self.write_config_formula(res.get('id'), key, model, domain, groupbys, view_id)
        return res

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = "res.config.settings"

    google_drive_uri_copy = fields.Char(related='google_drive_uri', string='URI Copy', help="The URL to generate the authorization code from Google", readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings
from . import google_drive

```

## File: static\src\js\add_to_google_spreadsheet_menu.js

```javascript
odoo.define('board.AddToGoogleSpreadsheetMenu', function (require) {
    "use strict";

    const Domain = require('web.Domain');
    const DropdownMenuItem = require('web.DropdownMenuItem');
    const FavoriteMenu = require('web.FavoriteMenu');

    const Dialog = require('web.OwlDialog');
    const { useState } = owl.hooks;

    /**
     * 'Add to Google spreadsheet' menu
     *
     * Component consisting only of a button calling the server to add the current
     * view to the user's spreadsheet configuration.
     * This component is only available in actions of type 'ir.actions.act_window'.
     * @extends DropdownMenuItem
     */
    class AddToGoogleSpreadsheetMenu extends DropdownMenuItem {
        constructor() {
            super(...arguments);

            this.state = useState({
                showDialog: false,
                url: false,
                formula: false,
            });
        }

        //---------------------------------------------------------------------
        // Handlers
        //---------------------------------------------------------------------

        /**
         * @private
         */
        async _onAddToSpreadsheet() {
            const searchQuery = this.env.searchModel.get('query');
            const listView = this.env.action.views.find(view => view.type === 'list');
            const modelName = this.env.action.res_model;
            const domain = Domain.prototype.arrayToString(searchQuery.domain);
            const groupBys = searchQuery.groupBy.join(" ");
            const listViewId = listView ? listView.viewID : false;
            const result = await this.rpc({
                model: 'google.drive.config',
                method: 'set_spreadsheet',
                args: [modelName, domain, groupBys, listViewId],
            });
            if (result.deprecated) {
                this.state.url = result.url;
                this.state.formula = result.formula;
                this.state.showDialog = true;
                this.state.open = false;
                return;
            }
            if (result.url) {
                // According to MDN doc, one should not use _blank as title.
                // todo: find a good name for the new window
                window.open(result.url, '_blank');
            }
        }

        //---------------------------------------------------------------------
        // Static
        //---------------------------------------------------------------------

        /**
         * @param {Object} env
         * @returns {boolean}
         */
        static shouldBeDisplayed(env) {
            return env.action.type === 'ir.actions.act_window';
        }
    }

    AddToGoogleSpreadsheetMenu.components = { Dialog };
    AddToGoogleSpreadsheetMenu.props = {};
    AddToGoogleSpreadsheetMenu.template = 'AddToGoogleSpreadsheetMenu';

    FavoriteMenu.registry.add('add-to-google-spreadsheet-menu', AddToGoogleSpreadsheetMenu, 20);

    return AddToGoogleSpreadsheetMenu;
});

```

## File: static\src\xml\addtospreadsheet.xml

```xml
<template>
    <t t-name="AddToGoogleSpreadsheetMenu" owl="1">
        <li class="o_menu_item o_add_to_spreadsheet" role="menuitem">
            <a class="dropdown-item" href="#" t-on-click.prevent="_onAddToSpreadsheet">Add to Google Spreadsheet</a>
        </li>

        <Dialog
            title="'Google Spreadsheet'"
            t-if="state.showDialog"
            t-on-dialog-closed="state.showDialog = false">
            <div>
                <p>To insert this data inside of a Google Sheet:</p>
                <ul>
                    <li>Duplicate the <a t-att-href="state.url" target="_blank">Spreadsheet Template</a></li>
                    <li>Setup your Odoo credentials in the sheet via the <code>Odoo &gt; Settings</code> menu</li>
                    <li>Paste the following formula in your spreadsheet:<br/>
                        <code><t t-esc="state.formula"/></code>
                    </li>
                </ul>
            </div>
        </Dialog>
    </t>
</template>

```

## File: views\google_spreadsheet_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <template id="assets_backend" name="google_spreadsheet assets" inherit_id="web.assets_backend">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/google_spreadsheet/static/src/js/add_to_google_spreadsheet_menu.js"></script>
            </xpath>
        </template>

</odoo>

```

## File: views\google_spreadsheet_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <!-- add google drive config field in user form -->

        <record id="view_ir_attachment_google_spreadsheet_tree" model="ir.ui.view">
            <field name="name">ir.attachment.google.spreadsheet.tree</field>
            <field name="model">ir.attachment</field>
            <field name="priority">100</field>
            <field name="arch" type="xml">
                <tree string="Google Spreadsheets">
                    <field name="name" string="Name"/>
                    <field name="url" widget="url" />
                </tree>
            </field>
        </record>

        <record id="view_ir_attachment_google_spreadsheet_form" model="ir.ui.view">
            <field name="name">ir.attachment.google.spreadsheet.form</field>
            <field name="model">ir.attachment</field>
            <field name="priority">100</field>
            <field name="arch" type="xml">
                <form string="Google Spreadsheets">
                    <sheet>
                        <group>
                            <group>
                                <field name="name" string="Name"/>
                                <field name="url" widget="url"/>
                            </group>
                            <group colspan="2">
                                <label for="description" colspan="2"/>
                                <field name="description" nolabel="1" colspan="2"/>
                            </group>
                        </group>
                    </sheet>
                </form>
            </field>
        </record>

        <record id="action_ir_attachment_google_spreadsheet_tree" model="ir.actions.act_window">
            <field name="name">Google Spreadsheets</field>
            <field name="res_model">ir.attachment</field>
            <field name="view_mode">tree,form</field>
            <field name="context">{'default_res_model': 'res.users', 'default_res_id': uid}</field>
            <field name="domain">[('url', '=ilike', '%google%/spreadsheet%')]</field>
            <field name="help">Google Spreadsheets</field>
        </record>

        <record id="action_ir_attachment_google_spreadsheet_tree_view" model="ir.actions.act_window.view">
            <field eval="1" name="sequence"/>
            <field name="view_mode">tree</field>
            <field name="view_id" ref="view_ir_attachment_google_spreadsheet_tree"/>
            <field name="act_window_id" ref="action_ir_attachment_google_spreadsheet_tree"/>
        </record>

        <record id="action_ir_attachment_google_spreadsheet_form_view" model="ir.actions.act_window.view">
            <field eval="2" name="sequence"/>
            <field name="view_mode">form</field>
            <field name="view_id" ref="view_ir_attachment_google_spreadsheet_form"/>
            <field name="act_window_id" ref="action_ir_attachment_google_spreadsheet_tree"/>
        </record>

        <menuitem
            id="menu_reporting_dashboard_google_spreadsheets"
            parent="base.menu_board_root"
            action="action_ir_attachment_google_spreadsheet_tree"/>

</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.google.spreadsheet</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[@id='msg_module_google_spreadsheet']" position="replace"/>
        </field>
    </record>
</odoo>

```

