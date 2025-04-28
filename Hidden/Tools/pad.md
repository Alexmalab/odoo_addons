# Odoo Module: pad

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
    'name': 'Collaborative Pads',
    'version': '2.0',
    'category': 'Hidden/Tools',
    'description': """
Adds enhanced support for (Ether)Pad attachments in the web client.
===================================================================

Lets the company customize which Pad installation should be used to link to new
pads (by default, http://etherpad.com/).
    """,
    'depends': ['web', 'base_setup'],
    'data': [
        'views/pad.xml',
        'views/res_config_settings_views.xml',
    ],
    'demo': ['data/pad_demo.xml'],
    'web': True,
    'qweb': ['static/src/xml/pad.xml'],
    'license': 'LGPL-3',
}

```

## File: data\pad_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="base.main_company" model="res.company">
        <field name="pad_server">https://pad.odoo.com</field>
    </record>

    <record id="base.main_company" model="res.company">
        <field name="pad_key">4DxmsNIbnQUVQMW9S9tx2oLOSjFdrx1l</field>
    </record>

</odoo>

```

## File: models\pad.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging
import random
import re
import string

import requests

from odoo import api, models, _
from odoo.exceptions import UserError

from ..py_etherpad import EtherpadLiteClient

_logger = logging.getLogger(__name__)


class PadCommon(models.AbstractModel):
    _name = 'pad.common'
    _description = 'Pad Common'

    def _valid_field_parameter(self, field, name):
        return name == 'pad_content_field' or super()._valid_field_parameter(field, name)

    @api.model
    def pad_is_configured(self):
        return bool(self.env.company.pad_server)

    @api.model
    def pad_generate_url(self):
        company = self.env.company.sudo()

        pad = {
            "server": company.pad_server,
            "key": company.pad_key,
        }

        # make sure pad server in the form of http://hostname
        if not pad["server"]:
            return pad
        if not pad["server"].startswith('http'):
            pad["server"] = 'http://' + pad["server"]
        pad["server"] = pad["server"].rstrip('/')
        # generate a salt
        s = string.ascii_uppercase + string.digits
        salt = ''.join([s[random.SystemRandom().randint(0, len(s) - 1)] for i in range(10)])
        # path
        # etherpad hardcodes pad id length limit to 50
        path = '-%s-%s' % (self._name, salt)
        path = '%s%s' % (self.env.cr.dbname.replace('_', '-')[0:50 - len(path)], path)
        # contruct the url
        url = '%s/p/%s' % (pad["server"], path)

        # if create with content
        if self.env.context.get('field_name') and self.env.context.get('model'):
            myPad = EtherpadLiteClient(pad["key"], pad["server"] + '/api')
            try:
                myPad.createPad(path)
            except IOError:
                raise UserError(_("Pad creation failed, either there is a problem with your pad server URL or with your connection."))

            # get attr on the field model
            model = self.env[self.env.context["model"]]
            field = model._fields[self.env.context['field_name']]
            real_field = field.pad_content_field

            res_id = self.env.context.get("object_id")
            record = model.browse(res_id)
            # get content of the real field
            real_field_value = record[real_field] or self.env.context.get('record', {}).get(real_field, '')
            if real_field_value:
                myPad.setHtmlFallbackText(path, real_field_value)

        return {
            "server": pad["server"],
            "path": path,
            "url": url,
        }

    @api.model
    def pad_get_content(self, url):
        company = self.env.company.sudo()
        myPad = EtherpadLiteClient(company.pad_key, (company.pad_server or '') + '/api')
        content = ''
        if url:
            split_url = url.split('/p/')
            path = len(split_url) == 2 and split_url[1]
            try:
                content = myPad.getHtml(path).get('html', '')
            except IOError:
                _logger.warning('Http Error: the credentials might be absent for url: "%s". Falling back.' % url)
                try:
                    r = requests.get('%s/export/html' % url)
                    r.raise_for_status()
                except Exception:
                    _logger.warning("No pad found with url '%s'.", url)
                else:
                    mo = re.search('<body>(.*)</body>', r.content.decode(), re.DOTALL)
                    if mo:
                        content = mo.group(1)

        return content

    # TODO
    # reverse engineer protocol to be setHtml without using the api key

    def write(self, vals):
        self._set_field_to_pad(vals)
        self._set_pad_to_field(vals)
        return super(PadCommon, self).write(vals)

    @api.model
    def create(self, vals):
        # Case of a regular creation: we receive the pad url, so we need to update the
        # corresponding field
        self._set_pad_to_field(vals)
        pad = super(PadCommon, self).create(vals)

        # Case of a programmatical creation (e.g. copy): we receive the field content, so we need
        # to create the corresponding pad
        if self.env.context.get('pad_no_create', False):
            return pad
        for k, field in self._fields.items():
            if hasattr(field, 'pad_content_field') and k not in vals:
                ctx = {
                    'model': self._name,
                    'field_name': k,
                    'object_id': pad.id,
                }
                pad_info = self.with_context(**ctx).pad_generate_url()
                pad[k] = pad_info.get('url')
        return pad

    def _set_field_to_pad(self, vals):
        # Update the pad if the `pad_content_field` is modified
        for k, field in self._fields.items():
            if hasattr(field, 'pad_content_field') and vals.get(field.pad_content_field) and self[k]:
                company = self.env.user.sudo().company_id
                myPad = EtherpadLiteClient(company.pad_key, (company.pad_server or '') + '/api')
                path = self[k].split('/p/')[1]
                myPad.setHtmlFallbackText(path, vals[field.pad_content_field])

    def _set_pad_to_field(self, vals):
        # Update the `pad_content_field` if the pad is modified
        for k, v in list(vals.items()):
            field = self._fields.get(k)
            if hasattr(field, 'pad_content_field'):
                vals[field.pad_content_field] = self.pad_get_content(v)

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    pad_server = fields.Char(help="Etherpad lite server. Example: beta.primarypad.com")
    pad_key = fields.Char('Pad Api Key', help="Etherpad lite api key.", groups="base.group_system")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    pad_server = fields.Char(related='company_id.pad_server', string="Pad Server", readonly=False)
    pad_key = fields.Char(related='company_id.pad_key', string="Pad API Key", readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings
from . import pad
from . import res_company

```

## File: py_etherpad\__init__.py

```python
"""Module to talk to EtherpadLite API."""
import requests
import logging

from odoo.tools import html2plaintext

_logger = logging.getLogger(__name__)


class EtherpadLiteClient:
    """Client to talk to EtherpadLite API."""
    API_VERSION = 1  # TODO probably 1.1 sometime soon

    CODE_OK = 0
    CODE_INVALID_PARAMETERS = 1
    CODE_INTERNAL_ERROR = 2
    CODE_INVALID_FUNCTION = 3
    CODE_INVALID_API_KEY = 4
    TIMEOUT = 20

    apiKey = ""
    baseUrl = "http://localhost:9001/api"

    def __init__(self, apiKey=None, baseUrl=None):
        if apiKey:
            self.apiKey = apiKey

        if baseUrl:
            self.baseUrl = baseUrl

    def call(self, function, arguments=None):
        """Create a dictionary of all parameters"""
        url = '%s/%d/%s' % (self.baseUrl, self.API_VERSION, function)

        params = arguments or {}
        params['apikey'] = self.apiKey

        r = requests.post(url, data=params, timeout=self.TIMEOUT)
        r.raise_for_status()
        return self.handleResult(r.json())

    def handleResult(self, result):
        """Handle API call result"""
        if 'code' not in result:
            raise Exception("API response has no code")
        if 'message' not in result:
            raise Exception("API response has no message")

        if 'data' not in result:
            result['data'] = None

        if result['code'] == self.CODE_OK:
            return result['data']
        elif result['code'] == self.CODE_INVALID_PARAMETERS or result['code'] == self.CODE_INVALID_API_KEY:
            raise ValueError(result['message'])
        elif result['code'] == self.CODE_INTERNAL_ERROR:
            raise Exception(result['message'])
        elif result['code'] == self.CODE_INVALID_FUNCTION:
            raise Exception(result['message'])
        else:
            raise Exception("An unexpected error occurred whilst handling the response")

    # GROUPS
    # Pads can belong to a group. There will always be public pads that do not belong to a group (or we give this group the id 0)

    def createGroup(self):
        """creates a new group"""
        return self.call("createGroup")

    def createGroupIfNotExistsFor(self, groupMapper):
        """this functions helps you to map your application group ids to etherpad lite group ids"""
        return self.call("createGroupIfNotExistsFor", {
            "groupMapper": groupMapper
        })

    def deleteGroup(self, groupID):
        """deletes a group"""
        return self.call("deleteGroup", {
            "groupID": groupID
        })

    def listPads(self, groupID):
        """returns all pads of this group"""
        return self.call("listPads", {
            "groupID": groupID
        })

    def createGroupPad(self, groupID, padName, text=''):
        """creates a new pad in this group"""
        params = {
            "groupID": groupID,
            "padName": padName,
        }
        if text:
            params['text'] = text
        return self.call("createGroupPad", params)

    # AUTHORS
    # Theses authors are bind to the attributes the users choose (color and name).

    def createAuthor(self, name=''):
        """creates a new author"""
        params = {}
        if name:
            params['name'] = name
        return self.call("createAuthor", params)

    def createAuthorIfNotExistsFor(self, authorMapper, name=''):
        """this functions helps you to map your application author ids to etherpad lite author ids"""
        params = {
            'authorMapper': authorMapper
        }
        if name:
            params['name'] = name
        return self.call("createAuthorIfNotExistsFor", params)

    # SESSIONS
    # Sessions can be created between a group and a author. This allows
    # an author to access more than one group. The sessionID will be set as
    # a cookie to the client and is valid until a certain date.

    def createSession(self, groupID, authorID, validUntil):
        """creates a new session"""
        return self.call("createSession", {
            "groupID": groupID,
            "authorID": authorID,
            "validUntil": validUntil
        })

    def deleteSession(self, sessionID):
        """deletes a session"""
        return self.call("deleteSession", {
            "sessionID": sessionID
        })

    def getSessionInfo(self, sessionID):
        """returns informations about a session"""
        return self.call("getSessionInfo", {
            "sessionID": sessionID
        })

    def listSessionsOfGroup(self, groupID):
        """returns all sessions of a group"""
        return self.call("listSessionsOfGroup", {
            "groupID": groupID
        })

    def listSessionsOfAuthor(self, authorID):
        """returns all sessions of an author"""
        return self.call("listSessionsOfAuthor", {
            "authorID": authorID
        })

    # PAD CONTENT
    # Pad content can be updated and retrieved through the API

    def getText(self, padID, rev=None):
        """returns the text of a pad"""
        params = {"padID": padID}
        if rev is not None:
            params['rev'] = rev
        return self.call("getText", params)

    # introduced with pull request merge
    def getHtml(self, padID, rev=None):
        """returns the html of a pad"""
        params = {"padID": padID}
        if rev is not None:
            params['rev'] = rev
        return self.call("getHTML", params)

    def setText(self, padID, text):
        """sets the text of a pad"""
        return self.call("setText", {
            "padID": padID,
            "text": text
        })

    def setHtmlFallbackText(self, padID, html):
        try:
            # Prevents malformed HTML errors
            html_wellformed = '<html><body>' + html + '</body></html>'
            return self.setHtml(padID, html_wellformed)
        except Exception:
            _logger.exception('Falling back to setText. SetHtml failed with message:')
            return self.setText(padID, html2plaintext(html).encode('UTF-8'))

    def setHtml(self, padID, html):
        """sets the text of a pad from html"""
        return self.call("setHTML", {
            "padID": padID,
            "html": html
        })

    # PAD
    # Group pads are normal pads, but with the name schema
    # GROUPID$PADNAME. A security manager controls access of them and its
    # forbidden for normal pads to include a  in the name.

    def createPad(self, padID, text=''):
        """creates a new pad"""
        params = {
            "padID": padID,
        }
        if text:
            params['text'] = text
        return self.call("createPad", params)

    def getRevisionsCount(self, padID):
        """returns the number of revisions of this pad"""
        return self.call("getRevisionsCount", {
            "padID": padID
        })

    def deletePad(self, padID):
        """deletes a pad"""
        return self.call("deletePad", {
            "padID": padID
        })

    def getReadOnlyID(self, padID):
        """returns the read only link of a pad"""
        return self.call("getReadOnlyID", {
            "padID": padID
        })

    def setPublicStatus(self, padID, publicStatus):
        """sets a boolean for the public status of a pad"""
        return self.call("setPublicStatus", {
            "padID": padID,
            "publicStatus": publicStatus
        })

    def getPublicStatus(self, padID):
        """return true of false"""
        return self.call("getPublicStatus", {
            "padID": padID
        })

    def setPassword(self, padID, password):
        """returns ok or a error message"""
        return self.call("setPassword", {
            "padID": padID,
            "password": password
        })

    def isPasswordProtected(self, padID):
        """returns true or false"""
        return self.call("isPasswordProtected", {
            "padID": padID
        })

```

## File: static\plugin\ep_disable_init_focus\ep.json

```json
{
    "parts":[
        {
            "name":"ep_disable_init_focus",
            "client_hooks":{
                "aceEditEvent":"ep_disable_init_focus/static/js/disable_init_focus"
            }
        }
    ]
}

```

## File: static\plugin\ep_disable_init_focus\package.json

```json
{
    "name":"ep_disable_init_focus",
    "version":"0.0.1",
    "description":"Disables init focus in etherpad-lite.",
    "dependencies":{

    },
    "engines":{
        "node":"*"
    },
    "author":{
        "name":"Odoo S.A. - Hitesh Trivedi",
        "email":"thiteshm155@gmail.com"
    }
}

```

## File: static\plugin\ep_disable_init_focus\static\js\disable_init_focus.js

```javascript
exports.aceEditEvent = function(hook, call, editorInfo, rep, documentAttributeManager){

    call.editorInfo.ace_focus = focus;
    function focus(){
        // Simple hook to disable the focus on the pad
    }

};

```

## File: static\src\js\pad.js

```javascript
odoo.define('pad.pad', function (require) {
"use strict";

var AbstractField = require('web.AbstractField');
var core = require('web.core');
var fieldRegistry = require('web.field_registry');

var _t = core._t;

var FieldPad = AbstractField.extend({
    template: 'FieldPad',
    content: "",
    events: {
        'click .oe_pad_switch': '_onToggleFullScreen',
    },

    /**
     * @override
     */
    willStart: function () {
        if (this.isPadConfigured === undefined) {
            return this._rpc({
                method: 'pad_is_configured',
                model: this.model,
            }).then(function (result) {
                // we write on the prototype to share the information between
                // all pad widgets instances, across all actions
                FieldPad.prototype.isPadConfigured = result;
            });
        }
        return this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    start: function () {
        if (!this.isPadConfigured) {
            this.$(".oe_unconfigured").removeClass('d-none');
            this.$(".oe_configured").addClass('d-none');
            return Promise.resolve();
        }
        if (this.mode === 'edit' && typeof(this.value) === 'object') {
            this.value = this.value.toJSON();
        }
        if (this.mode === 'edit' && _.str.startsWith(this.value, 'http')) {
            this.url = this.value;
            // please close your eyes and look elsewhere...
            // Since the pad value (the url) will not change during the edition
            // process, we have a problem: the description field will not be
            // properly updated.  We need to explicitely write the value each
            // time someone edit the record in order to force the server to read
            // the updated value of the pad and put it in the description field.
            //
            // However, the basic model optimizes away the changes if they are
            // not really different from the current value. So, we need to
            // either add special configuration options to the basic model, or
            // to trick him into accepting the same value as being different...
            // Guess what we decided...
            var url = {};
            url.toJSON = _.constant(this.url);
            this._setValue(url, {doNotSetDirty: true});
        }

        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * If we had to generate an url, we wait for the generation to be completed,
     * so the current record will be associated with the correct pad url.
     *
     * @override
     */
    commitChanges: function () {
        return this.urlDef;
    },
    /**
     * @override
     */
    isSet: function () {
        return true;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Note that this method has some serious side effects: performing rpcs and
     * setting the value of this field.  This is not conventional and should not
     * be copied in other code, unless really necessary.
     *
     * @override
     * @private
     */
    _renderEdit: function () {
        if (this.url) {
            // here, we have a valid url, so we can simply display an iframe
            // with the correct src attribute
            var userName = encodeURIComponent(this.getSession().name);
            var url = this.url + '?showChat=false&userName=' + userName;
            var content = '<iframe width="100%" height="100%" frameborder="0" src="' + url + '"></iframe>';
            this.$('.oe_pad_content').html(content);
        } else if (this.value) {
            // it looks like the field does not contain a valid url, so we just
            // display it (it cannot be edited in that case)
            this.$('.oe_pad_content').text(this.value);
        } else {
            // It is totally discouraged to have a render method that does
            // non-rendering work, especially since the work in question
            // involves doing RPCs and changing the value of the field.
            // However, this is kind of necessary in this case, because the
            // value of the field is actually only the url of the pad. The
            // actual content will be loaded in an iframe.  We could do this
            // work in the basic model, but the basic model does not know that
            // this widget is in edit or readonly, and we really do not want to
            // create a pad url everytime a task without a pad is viewed.
            var self = this;
            this.urlDef = this._rpc({
                method: 'pad_generate_url',
                model: this.model,
                context: {
                    model: this.model,
                    field_name: this.name,
                    object_id: this.res_id,
                    record: this.recordData,
                },
            }, {
                shadow: true
            }).then(function (result) {
                // We need to write the url of the pad to trigger
                // the write function which updates the actual value
                // of the field to the value of the pad content
                self.url = result.url;
                self._setValue(result.url, {doNotSetDirty: true});
            });
        }
    },
    /**
     * @override
     * @private
     */
    _renderReadonly: function () {
        if (_.str.startsWith(this.value, 'http')) {
            var self = this;
            this.$('.oe_pad_content')
                .addClass('oe_pad_loading')
                .text(_t("Loading"));
            this._rpc({
                method: 'pad_get_content',
                model: this.model,
                args: [this.value]
            }, {
                shadow: true
            }).then(function (data) {
                self.$('.oe_pad_content')
                    .removeClass('oe_pad_loading')
                    .html('<div class="oe_pad_readonly"><div>');
                self.$('.oe_pad_readonly').html(data);
            }).guardedCatch(function () {
                self.$('.oe_pad_content').text(_t('Unable to load pad'));
            });
        } else {
            this.$('.oe_pad_content')
                .addClass('oe_pad_loading')
                .show()
                .text(_t("This pad will be initialized on first edit"));
        }
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _onToggleFullScreen: function () {
        this.$el.toggleClass('oe_pad_fullscreen mb0');
        this.$('.oe_pad_switch').toggleClass('fa-expand fa-compress');
        this.$el.parents('.o_touch_device').toggleClass('o_scroll_hidden');
    },
});

fieldRegistry.add('pad', FieldPad);

return FieldPad;

});

```

## File: static\src\xml\pad.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="FieldPad">
        <div class="oe_form_field_text oe_pad">
            <p class="oe_unconfigured d-none">
                Please, enter your Etherpad credentials through the Settings.
            </p>
            <t t-if="widget.mode === 'readonly'">
                <div class="oe_pad_content etherpad_readonly oe_configured" />
            </t>
            <t t-if="widget.mode === 'edit'">
                <div class="oe_pad_switch_positioner oe_configured">
                    <span class="fa fa-expand oe_pad_switch" role="img" aria-label="Switch pad" title="Switch pad"/>
                </div>
                <div class="oe_pad_content oe_editing oe_configured" />
            </t>
        </div>
    </t>

</templates>

```

## File: views\pad.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="assets_backend" name="pad assets" inherit_id="web.assets_backend">
        <xpath expr="." position="inside">
            <link rel="stylesheet" href="/pad/static/src/css/etherpad.css"/>
            <script type="text/javascript" src="/pad/static/src/js/pad.js" />
        </xpath>
    </template>

    <template id="qunit_suite" name="pad tests" inherit_id="web.qunit_suite_tests">
        <xpath expr="//script[last()]" position="after">
            <script type="text/javascript" src="/pad/static/tests/pad_tests.js"></script>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>     
<odoo>        
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.pad</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="msg_module_pad" position="replace">
                <div class="content-group" id="pad_configuration_settings" attrs="{'invisible': [('module_pad', '=', False)]}">
                    <div class="mt16 row">
                        <label for="pad_server" string="Server" class="col-3 col-lg-3 o_light_label"/>
                        <field name="pad_server" placeholder="e.g. beta.primarypad.com" attrs="{'required': [('module_pad', '!=', False)]}"/>
                        <label for="pad_key" string="API Key" class="col-3 col-lg-3 o_light_label"/>
                        <field name="pad_key" attrs="{'required': [('module_pad', '!=', False)]}"/>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

