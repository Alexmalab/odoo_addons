# Odoo Module: website_form

Category: Website/Website

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import controllers
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Website Form',
    'category': 'Website/Website',
    'summary': 'Build custom web forms',
    'description': """
        Customize and create your own web forms.
        This module adds a new building block in the website builder in order to build new forms from scratch in any website page.
    """,
    'version': '1.0',
    'depends': ['website', 'mail', 'google_recaptcha'],
    'data': [
        'data/mail_mail_data.xml',
        'views/assets.xml',
        'views/ir_model_views.xml',
        'views/snippets/snippets.xml',
        'views/snippets/s_website_form.xml',
        'views/website_form_templates.xml',
    ],
    'installable': True,
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import json
import pytz

from datetime import datetime
from psycopg2 import IntegrityError
from werkzeug.exceptions import BadRequest

from odoo import http, SUPERUSER_ID
from odoo.http import request
from odoo.tools import DEFAULT_SERVER_DATE_FORMAT, DEFAULT_SERVER_DATETIME_FORMAT
from odoo.tools.translate import _, _lt
from odoo.exceptions import AccessDenied, ValidationError, UserError
from odoo.addons.base.models.ir_qweb_fields import nl2br
from odoo.tools.misc import hmac, consteq


class WebsiteForm(http.Controller):

    @http.route('/website_form/', type='http', auth="public", methods=['POST'], multilang=False)
    def website_form_empty(self, **kwargs):
        # This is a workaround to don't add language prefix to <form action="/website_form/" ...>
        return ""

    # Check and insert values from the form on the model <model>
    @http.route('/website_form/<string:model_name>', type='http', auth="public", methods=['POST'], website=True, csrf=False)
    def website_form(self, model_name, **kwargs):
        # Partial CSRF check, only performed when session is authenticated, as there
        # is no real risk for unauthenticated sessions here. It's a common case for
        # embedded forms now: SameSite policy rejects the cookies, so the session
        # is lost, and the CSRF check fails, breaking the post for no good reason.
        csrf_token = request.params.pop('csrf_token', None)
        if request.session.uid and not request.validate_csrf(csrf_token):
            raise BadRequest('Session expired (invalid CSRF token)')

        try:
            # The except clause below should not let what has been done inside
            # here be committed. It should not either roll back everything in
            # this controller method. Instead, we use a savepoint to roll back
            # what has been done inside the try clause.
            with request.env.cr.savepoint():
                if request.env['ir.http']._verify_request_recaptcha_token('website_form'):
                    return self._handle_website_form(model_name, **kwargs)
            error = _("Suspicious activity detected by Google reCaptcha.")
        except (ValidationError, UserError) as e:
            error = e.args[0]
        return json.dumps({
            'error': error,
        })

    def _handle_website_form(self, model_name, **kwargs):
        model_record = request.env['ir.model'].sudo().search([('model', '=', model_name), ('website_form_access', '=', True)])
        if not model_record:
            return json.dumps({
                'error': _("The form's specified model does not exist")
            })

        try:
            data = self.extract_data(model_record, request.params)
        # If we encounter an issue while extracting data
        except ValidationError as e:
            # I couldn't find a cleaner way to pass data to an exception
            return json.dumps({'error_fields' : e.args[0]})

        try:
            id_record = self.insert_record(request, model_record, data['record'], data['custom'], data.get('meta'))
            if id_record:
                self.insert_attachment(model_record, id_record, data['attachments'])
                # in case of an email, we want to send it immediately instead of waiting
                # for the email queue to process
                if model_name == 'mail.mail':
                    form_has_email_cc = {'email_cc', 'email_bcc'} & kwargs.keys() or \
                        'email_cc' in kwargs["website_form_signature"]
                    # remove the email_cc information from the signature
                    if kwargs.get("email_to"):
                        kwargs["website_form_signature"] = kwargs["website_form_signature"].split(':')[0]
                        value = kwargs['email_to'] + (':email_cc' if form_has_email_cc else '')
                        hash_value = hmac(model_record.env, 'website_form_signature', value)
                        if not consteq(kwargs["website_form_signature"], hash_value):
                            raise AccessDenied('invalid website_form_signature')
                    request.env[model_name].sudo().browse(id_record).send()

        # Some fields have additional SQL constraints that we can't check generically
        # Ex: crm.lead.probability which is a float between 0 and 1
        # TODO: How to get the name of the erroneous field ?
        except IntegrityError:
            return json.dumps(False)

        request.session['form_builder_model_model'] = model_record.model
        request.session['form_builder_model'] = model_record.name
        request.session['form_builder_id'] = id_record

        return json.dumps({'id': id_record})

    # Constants string to make metadata readable on a text field

    _meta_label = _lt("Metadata")  # Title for meta data

    # Dict of dynamically called filters following type of field to be fault tolerent

    def identity(self, field_label, field_input):
        return field_input

    def integer(self, field_label, field_input):
        return int(field_input)

    def floating(self, field_label, field_input):
        return float(field_input)

    def boolean(self, field_label, field_input):
        return bool(field_input)

    def binary(self, field_label, field_input):
        return base64.b64encode(field_input.read())

    def one2many(self, field_label, field_input):
        return [int(i) for i in field_input.split(',')]

    def many2many(self, field_label, field_input, *args):
        return [(args[0] if args else (6,0)) + (self.one2many(field_label, field_input),)]

    _input_filters = {
        'char': identity,
        'text': identity,
        'html': identity,
        'date': identity,
        'datetime': identity,
        'many2one': integer,
        'one2many': one2many,
        'many2many':many2many,
        'selection': identity,
        'boolean': boolean,
        'integer': integer,
        'float': floating,
        'binary': binary,
        'monetary': floating,
    }


    # Extract all data sent by the form and sort its on several properties
    def extract_data(self, model, values):
        dest_model = request.env[model.sudo().model]

        data = {
            'record': {},        # Values to create record
            'attachments': [],  # Attached files
            'custom': '',        # Custom fields values
            'meta': '',         # Add metadata if enabled
        }

        authorized_fields = model.sudo()._get_form_writable_fields()
        error_fields = []
        custom_fields = []

        for field_name, field_value in values.items():
            # If the value of the field if a file
            if hasattr(field_value, 'filename'):
                # Undo file upload field name indexing
                field_name = field_name.split('[', 1)[0]

                # If it's an actual binary field, convert the input file
                # If it's not, we'll use attachments instead
                if field_name in authorized_fields and authorized_fields[field_name]['type'] == 'binary':
                    data['record'][field_name] = base64.b64encode(field_value.read())
                    field_value.stream.seek(0) # do not consume value forever
                    if authorized_fields[field_name]['manual'] and field_name + "_filename" in dest_model:
                        data['record'][field_name + "_filename"] = field_value.filename
                else:
                    field_value.field_name = field_name
                    data['attachments'].append(field_value)

            # If it's a known field
            elif field_name in authorized_fields:
                try:
                    input_filter = self._input_filters[authorized_fields[field_name]['type']]
                    data['record'][field_name] = input_filter(self, field_name, field_value)
                except ValueError:
                    error_fields.append(field_name)

            # If it's a custom field
            elif field_name not in ('context', 'website_form_signature'):
                custom_fields.append((field_name, field_value))

        data['custom'] = "\n".join([u"%s : %s" % v for v in custom_fields])

        # Add metadata if enabled  # ICP for retrocompatibility
        if request.env['ir.config_parameter'].sudo().get_param('website_form_enable_metadata'):
            environ = request.httprequest.headers.environ
            data['meta'] += "%s : %s\n%s : %s\n%s : %s\n%s : %s\n" % (
                "IP"                , environ.get("REMOTE_ADDR"),
                "USER_AGENT"        , environ.get("HTTP_USER_AGENT"),
                "ACCEPT_LANGUAGE"   , environ.get("HTTP_ACCEPT_LANGUAGE"),
                "REFERER"           , environ.get("HTTP_REFERER")
            )

        # This function can be defined on any model to provide
        # a model-specific filtering of the record values
        # Example:
        # def website_form_input_filter(self, values):
        #     values['name'] = '%s\'s Application' % values['partner_name']
        #     return values
        if hasattr(dest_model, "website_form_input_filter"):
            data['record'] = dest_model.website_form_input_filter(request, data['record'])

        missing_required_fields = [label for label, field in authorized_fields.items() if field['required'] and not label in data['record']]
        if any(error_fields):
            raise ValidationError(error_fields + missing_required_fields)

        return data

    def insert_record(self, request, model, values, custom, meta=None):
        model_name = model.sudo().model
        if model_name == 'mail.mail':
            email_from = _('"%s form submission" <%s>') % (request.env.company.name, request.env.company.email)
            values.update({'reply_to': values.get('email_from'), 'email_from': email_from})
        record = request.env[model_name].with_user(SUPERUSER_ID).with_context(
            mail_create_nosubscribe=True,
            commit_assetsbundle=False,
        ).create(values)

        if custom or meta:
            _custom_label = "%s\n___________\n\n" % _("Other Information:")  # Title for custom fields
            if model_name == 'mail.mail':
                _custom_label = "%s\n___________\n\n" % _("This message has been posted on your website!")
            default_field = model.website_form_default_field_id
            default_field_data = values.get(default_field.name, '')
            custom_content = (default_field_data + "\n\n" if default_field_data else '') \
                           + (_custom_label + custom + "\n\n" if custom else '') \
                           + (self._meta_label + "\n________\n\n" + meta if meta else '')

            # If there is a default field configured for this model, use it.
            # If there isn't, put the custom data in a message instead
            if default_field.name:
                if default_field.ttype == 'html' or model_name == 'mail.mail':
                    custom_content = nl2br(custom_content)
                record.update({default_field.name: custom_content})
            else:
                values = {
                    'body': nl2br(custom_content),
                    'model': model_name,
                    'message_type': 'comment',
                    'no_auto_thread': False,
                    'res_id': record.id,
                }
                mail_id = request.env['mail.message'].with_user(SUPERUSER_ID).create(values)

        return record.id

    # Link all files attached on the form
    def insert_attachment(self, model, id_record, files):
        orphan_attachment_ids = []
        model_name = model.sudo().model
        record = model.env[model_name].browse(id_record)
        authorized_fields = model.sudo()._get_form_writable_fields()
        for file in files:
            custom_field = file.field_name not in authorized_fields
            attachment_value = {
                'name': file.filename,
                'datas': base64.encodebytes(file.read()),
                'res_model': model_name,
                'res_id': record.id,
            }
            attachment_id = request.env['ir.attachment'].sudo().create(attachment_value)
            if attachment_id and not custom_field:
                record_sudo = record.sudo()
                value = [(4, attachment_id.id)]
                if record_sudo._fields[file.field_name].type == 'many2one':
                    value = attachment_id.id
                record_sudo[file.field_name] = value
            else:
                orphan_attachment_ids.append(attachment_id.id)

        if model_name != 'mail.mail':
            # If some attachments didn't match a field on the model,
            # we create a mail.message to link them to the record
            if orphan_attachment_ids:
                values = {
                    'body': _('<p>Attached files : </p>'),
                    'model': model_name,
                    'message_type': 'comment',
                    'no_auto_thread': False,
                    'res_id': id_record,
                    'attachment_ids': [(6, 0, orphan_attachment_ids)],
                    'subtype_id': request.env['ir.model.data'].xmlid_to_res_id('mail.mt_comment'),
                }
                mail_id = request.env['mail.message'].with_user(SUPERUSER_ID).create(values)
        else:
            # If the model is mail.mail then we have no other choice but to
            # attach the custom binary field files on the attachment_ids field.
            for attachment_id_id in orphan_attachment_ids:
                record.attachment_ids = [(4, attachment_id_id)]

```

## File: controllers\__init__.py

```python
from . import main

```

## File: data\mail_mail_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mail.model_mail_mail" model="ir.model">
      <field name="website_form_key">send_mail</field>
      <field name="website_form_default_field_id" ref="mail.field_mail_mail__body_html" />
      <field name="website_form_access">True</field>
      <field name="website_form_label">Send an E-mail</field>
    </record>
    <function model="ir.model.fields" name="formbuilder_whitelist">
      <value>mail.mail</value>
      <value eval="[
        'subject',
        'body_html',
        'email_to',
        'email_from',
        'record_name',
        'attachment_ids',
        ]"/>
    </function>
</odoo>

```

## File: models\ir_ui_view.py

```python
# -*- coding: utf-8 -*-
from lxml import etree

from odoo import models
from odoo.tools.misc import hmac


class View(models.Model):

    _inherit = "ir.ui.view"

    def read_combined(self, fields=None):
        root = super(View, self).read_combined(fields)
        if self.type != "qweb" or '/website_form/' not in root['arch']:  #Performance related check, reduce the amount of operation for unrelated views
            return root
        root_node = etree.fromstring(root['arch'])
        nodes = root_node.xpath('.//form[contains(@action, "/website_form/")]')
        for form in nodes:
            existing_hash_node = form.find('.//input[@type="hidden"][@name="website_form_signature"]')
            if existing_hash_node is not None:
                existing_hash_node.getparent().remove(existing_hash_node)
            input_nodes = form.xpath('.//input[contains(@name, "email_")]')
            form_values = {input_node.attrib['name']: input_node for input_node in input_nodes}
            # if this form does not send an email, ignore. But at this stage,
            # the value of email_to can still be None in case of default value
            if 'email_to' not in form_values.keys():
                continue
            elif not form_values['email_to'].attrib.get('value'):
                form_values['email_to'].attrib['value'] = self.env.company.email or ''
            has_cc = {'email_cc', 'email_bcc'} & form_values.keys()
            value = form_values['email_to'].attrib['value'] + (':email_cc' if has_cc else '')
            hash_value = hmac(self.sudo().env, 'website_form_signature', value)
            hash_node = '<input type="hidden" class="form-control s_website_form_input s_website_form_custom" name="website_form_signature" value=""/>'
            if has_cc:
                hash_value += ':email_cc'
            form_values['email_to'].addnext(etree.fromstring(hash_node))
            form_values['email_to'].getnext().attrib['value'] = hash_value
        root['arch'] = etree.tostring(root_node)
        return root

```

## File: models\models.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, SUPERUSER_ID
from odoo.http import request


class website_form_config(models.Model):
    _inherit = 'website'

    def _website_form_last_record(self):
        if request and request.session.form_builder_model_model:
            return request.env[request.session.form_builder_model_model].browse(request.session.form_builder_id)
        return False


class website_form_model(models.Model):
    _name = 'ir.model'
    _description = 'Models'
    _inherit = 'ir.model'

    website_form_access = fields.Boolean('Allowed to use in forms', help='Enable the form builder feature for this model.')
    website_form_default_field_id = fields.Many2one('ir.model.fields', 'Field for custom form data', domain="[('model', '=', model), ('ttype', '=', 'text')]", help="Specify the field which will contain meta and custom form fields datas.")
    website_form_label = fields.Char("Label for form action", help="Form action label. Ex: crm.lead could be 'Send an e-mail' and project.issue could be 'Create an Issue'.")
    website_form_key = fields.Char(help='Used in FormBuilder Registry')

    def _get_form_writable_fields(self):
        """
        Restriction of "authorized fields" (fields which can be used in the
        form builders) to fields which have actually been opted into form
        builders and are writable. By default no field is writable by the
        form builder.
        """
        if self.model == "mail.mail":
            included = {'email_from', 'email_to', 'email_cc', 'email_bcc', 'body', 'reply_to', 'subject'}
        else:
            included = {
                field.name
                for field in self.env['ir.model.fields'].sudo().search([
                    ('model_id', '=', self.id),
                    ('website_form_blacklisted', '=', False)
                ])
            }

        return {
            k: v for k, v in self.get_authorized_fields(self.model).items()
            if k in included
        }

    @api.model
    def get_authorized_fields(self, model_name):
        """ Return the fields of the given model name as a mapping like method `fields_get`. """
        model = self.env[model_name]
        fields_get = model.fields_get()

        for key, val in model._inherits.items():
            fields_get.pop(val, None)

        # Unrequire fields with default values
        default_values = model.with_user(SUPERUSER_ID).default_get(list(fields_get))
        for field in [f for f in fields_get if f in default_values]:
            fields_get[field]['required'] = False

        # Remove readonly and magic fields
        # Remove string domains which are supposed to be evaluated
        # (e.g. "[('product_id', '=', product_id)]")
        MAGIC_FIELDS = models.MAGIC_COLUMNS + [model.CONCURRENCY_CHECK_FIELD]
        for field in list(fields_get):
            if 'domain' in fields_get[field] and isinstance(fields_get[field]['domain'], str):
                del fields_get[field]['domain']
            if fields_get[field].get('readonly') or field in MAGIC_FIELDS or fields_get[field]['type'] == 'many2one_reference':
                del fields_get[field]

        return fields_get


class website_form_model_fields(models.Model):
    """ fields configuration for form builder """
    _name = 'ir.model.fields'
    _description = 'Fields'
    _inherit = 'ir.model.fields'

    def init(self):
        # set all existing unset website_form_blacklisted fields to ``true``
        #  (so that we can use it as a whitelist rather than a blacklist)
        self._cr.execute('UPDATE ir_model_fields'
                         ' SET website_form_blacklisted=true'
                         ' WHERE website_form_blacklisted IS NULL')
        # add an SQL-level default value on website_form_blacklisted to that
        # pure-SQL ir.model.field creations (e.g. in _reflect) generate
        # the right default value for a whitelist (aka fields should be
        # blacklisted by default)
        self._cr.execute('ALTER TABLE ir_model_fields '
                         ' ALTER COLUMN website_form_blacklisted SET DEFAULT true')

    @api.model
    def formbuilder_whitelist(self, model, fields):
        """
        :param str model: name of the model on which to whitelist fields
        :param list(str) fields: list of fields to whitelist on the model
        :return: nothing of import
        """
        # postgres does *not* like ``in [EMPTY TUPLE]`` queries
        if not fields: return False

        # only allow users who can change the website structure
        if not self.env['res.users'].has_group('website.group_website_designer'):
            return False

        # the ORM only allows writing on custom fields and will trigger a
        # registry reload once that's happened. We want to be able to
        # whitelist non-custom fields and the registry reload absolutely
        # isn't desirable, so go with a method and raw SQL
        self.env.cr.execute(
            "UPDATE ir_model_fields"
            " SET website_form_blacklisted=false"
            " WHERE model=%s AND name in %s", (model, tuple(fields)))
        return True

    website_form_blacklisted = fields.Boolean(
        'Blacklisted in web forms', default=True, index=True, # required=True,
        help='Blacklist this field for web forms'
    )

```

## File: models\__init__.py

```python
from . import models
from . import ir_ui_view

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#B06161"/><stop offset="45.785%" stop-color="#984E4E"/><stop offset="100%" stop-color="#7C3838"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M41.118 69H4c-2 0-4-1-4-4V28.375L15 12h41v42L41.118 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#FFF" d="M34 17v-2h2v2h2v2h-2v2h-2v-2h-2v-2h2zM15 42h41v12H15V42zm19.51 8.985a.608.608 0 0 0 .73 0l5.942-4.792c.202-.162.202-.426 0-.589l-.73-.59a.608.608 0 0 0-.731 0l-4.846 3.909-2.262-1.825a.608.608 0 0 0-.731 0l-.73.59c-.202.162-.202.426 0 .589l3.358 2.708zM15 12h41v12H15V12zm0 15h41v12H15V27zm2-13v8h37v-8H17zm0 15v8h37v-8H17zm15 3h6v2h-6v-2z"/></g></g></svg>
```

## File: static\src\js\website_form_editor_registry.js

```javascript
odoo.define('website_form.form_editor_registry', function (require) {
'use strict';

var Registry = require('web.Registry');

return new Registry();

});

odoo.define('website_form.send_mail_form', function (require) {
'use strict';

var core = require('web.core');
var FormEditorRegistry = require('website_form.form_editor_registry');

var _t = core._t;

FormEditorRegistry.add('send_mail', {
    formFields: [{
        type: 'char',
        custom: true,
        required: true,
        name: 'Your Name',
    }, {
        type: 'tel',
        custom: true,
        name: 'Phone Number',
    }, {
        type: 'email',
        modelRequired: true,
        name: 'email_from',
        string: 'Your Email',
    }, {
        type: 'char',
        custom: true,
        name: 'Your Company',
    }, {
        type: 'char',
        modelRequired: true,
        name: 'subject',
        string: 'Subject',
    }, {
        type: 'text',
        custom: true,
        required: true,
        name: 'Your Question',
    }],
    fields: [{
        name: 'email_to',
        type: 'char',
        required: true,
        string: _t('Recipient Email'),
        defaultValue: 'info@yourcompany.example.com',
    }],
});

});

```

## File: static\src\snippets\s_website_form\000.js

```javascript
odoo.define('website_form.s_website_form', function (require) {
    'use strict';

    var core = require('web.core');
    var time = require('web.time');
    const {ReCaptcha} = require('google_recaptcha.ReCaptchaV3');
    var ajax = require('web.ajax');
    var publicWidget = require('web.public.widget');
    const dom = require('web.dom');
    const session = require('web.session');

    var _t = core._t;
    var qweb = core.qweb;

    publicWidget.registry.s_website_form = publicWidget.Widget.extend({
        selector: '.s_website_form form, form.s_website_form', // !compatibility
        xmlDependencies: ['/website_form/static/src/xml/website_form.xml'],
        events: {
            'click .s_website_form_send, .o_website_form_send': 'send', // !compatibility
        },

        /**
         * @constructor
         */
        init: function () {
            this._super(...arguments);
            this._recaptcha = new ReCaptcha();
            this.__started = new Promise(resolve => this.__startResolve = resolve);
        },
        willStart: function () {
            const res = this._super(...arguments);
            if (!this.$target[0].classList.contains('s_website_form_no_recaptcha')) {
                this._recaptchaLoaded = true;
                this._recaptcha.loadLibs();
            }
            return res;
        },
        start: function () {
            var self = this;

            // Initialize datetimepickers
            var datepickers_options = {
                minDate: moment({ y: 1000 }),
                maxDate: moment({y: 9999, M: 11, d: 31}),
                calendarWeeks: true,
                icons: {
                    time: 'fa fa-clock-o',
                    date: 'fa fa-calendar',
                    next: 'fa fa-chevron-right',
                    previous: 'fa fa-chevron-left',
                    up: 'fa fa-chevron-up',
                    down: 'fa fa-chevron-down',
                    },
                locale: moment.locale(),
                format: time.getLangDatetimeFormat(),
            };
            this.$target.find('.s_website_form_datetime, .o_website_form_datetime').datetimepicker(datepickers_options); // !compatibility

            // Adapt options to date-only pickers
            datepickers_options.format = time.getLangDateFormat();
            this.$target.find('.s_website_form_date, .o_website_form_date').datetimepicker(datepickers_options); // !compatibility

            // Display form values from tag having data-for attribute
            // It's necessary to handle field values generated on server-side
            // Because, using t-att- inside form make it non-editable
            var $values = $('[data-for=' + this.$target.attr('id') + ']');
            if ($values.length) {
                const values = JSON.parse($values.data('values')
                    // replaces `True` by `true` if they are after `,` or `:` or `[`
                    .replace(/([,:\[]\s*)True/g, '$1true')
                    // replaces `False` and `None` by `""` if they are after `,` or `:` or `[`
                    .replace(/([,:\[]\s*)(False|None)/g, '$1""')
                    // replaces the `'` by `"` if they are before `,` or `:` or `]` or `}`
                    .replace(/'(\s*[,:\]}])/g, '"$1')
                    // replaces the `'` by `"` if they are after `{` or `[` or `,` or `:`
                    .replace(/([{\[:,]\s*)'/g, '$1"')
                );
                var fields = _.pluck(this.$target.serializeArray(), 'name');
                _.each(fields, function (field) {
                    if (_.has(values, field)) {
                        var $field = self.$target.find('input[name="' + field + '"], textarea[name="' + field + '"]');
                        if (!$field.val()) {
                            $field.val(values[field]);
                            $field.data('website_form_original_default_value', $field.val());
                        }
                    }
                });
            }

            return this._super(...arguments).then(() => this.__startResolve());
        },

        destroy: function () {
            this._super.apply(this, arguments);
            this.$target.find('button').off('click');

            // Empty imputs
            this.$target[0].reset();

            // Remove saving of the error colors
            this.$target.find('.o_has_error').removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');

            // Remove the status message
            this.$target.find('#s_website_form_result, #o_website_form_result').empty(); // !compatibility

            // Remove the success message and display the form
            this.$target.removeClass('d-none');
            this.$target.parent().find('.s_website_form_end_message').addClass('d-none');
        },

        send: async function (e) {
            e.preventDefault(); // Prevent the default submit behavior
             // Prevent users from crazy clicking
            this.$target.find('.s_website_form_send, .o_website_form_send')
                .addClass('disabled')    // !compatibility
                .attr('disabled', 'disabled');

            var self = this;

            self.$target.find('#s_website_form_result, #o_website_form_result').empty(); // !compatibility
            if (!self.check_error_fields({})) {
                self.update_status('error', _t("Please fill in the form correctly."));
                return false;
            }

            // Prepare form inputs
            this.form_fields = this.$target.serializeArray();
            $.each(this.$target.find('input[type=file]'), function (outer_index, input) {
                $.each($(input).prop('files'), function (index, file) {
                    // Index field name as ajax won't accept arrays of files
                    // when aggregating multiple files into a single field value
                    self.form_fields.push({
                        name: input.name + '[' + outer_index + '][' + index + ']',
                        value: file
                    });
                });
            });

            // Serialize form inputs into a single object
            // Aggregate multiple values into arrays
            var form_values = {};
            _.each(this.form_fields, function (input) {
                if (input.name in form_values) {
                    // If a value already exists for this field,
                    // we are facing a x2many field, so we store
                    // the values in an array.
                    if (Array.isArray(form_values[input.name])) {
                        form_values[input.name].push(input.value);
                    } else {
                        form_values[input.name] = [form_values[input.name], input.value];
                    }
                } else {
                    if (input.value !== '') {
                        form_values[input.name] = input.value;
                    }
                }
            });

            // force server date format usage for existing fields
            this.$target.find('.s_website_form_field:not(.s_website_form_custom)')
            .find('.s_website_form_date, .s_website_form_datetime').each(function () {
                const $input = $(this).find('input');

                // Datetimepicker('viewDate') will return `new Date()` if the
                // input is empty but we want to keep the empty value
                if (!$input.val()) {
                    return;
                }

                var date = $(this).datetimepicker('viewDate').clone().locale('en');
                var format = 'YYYY-MM-DD';
                if ($(this).hasClass('s_website_form_datetime')) {
                    date = date.utc();
                    format = 'YYYY-MM-DD HH:mm:ss';
                }
                form_values[$input.attr('name')] = date.format(format);
            });

            if (this._recaptchaLoaded) {
                const tokenObj = await this._recaptcha.getToken('website_form');
                if (tokenObj.token) {
                    form_values['recaptcha_token_response'] = tokenObj.token;
                } else if (tokenObj.error) {
                    self.update_status('error', tokenObj.error);
                    return false;
                }
            }

            // Post form and handle result
            ajax.post(this.$target.attr('action') + (this.$target.data('force_action') || this.$target.data('model_name')), form_values)
            .then(function (result_data) {
                // Restore send button behavior
                self.$target.find('.s_website_form_send, .o_website_form_send')
                    .removeAttr('disabled')
                    .removeClass('disabled'); // !compatibility
                result_data = JSON.parse(result_data);
                if (!result_data.id) {
                    // Failure, the server didn't return the created record ID
                    self.update_status('error', result_data.error ? result_data.error : false);
                    if (result_data.error_fields) {
                        // If the server return a list of bad fields, show these fields for users
                        self.check_error_fields(result_data.error_fields);
                    }
                } else {
                    // Success, redirect or update status
                    let successMode = self.$target[0].dataset.successMode;
                    let successPage = self.$target[0].dataset.successPage;
                    if (!successMode) {
                        successPage = self.$target.attr('data-success_page'); // Compatibility
                        successMode = successPage ? 'redirect' : 'nothing';
                    }
                    switch (successMode) {
                        case 'redirect': {
                            let hashIndex = successPage.indexOf("#");
                            if (hashIndex > 0) {
                                // URL containing an anchor detected: extract
                                // the anchor from the URL if the URL is the
                                // same as the current page URL so we can scroll
                                // directly to the element (if found) later
                                // instead of redirecting.
                                // Note that both currentUrlPath and successPage
                                // can exist with or without a trailing slash
                                // before the hash (e.g. "domain.com#footer" or
                                // "domain.com/#footer"). Therefore, if they are
                                // not present, we add them to be able to
                                // compare the two variables correctly.
                                let currentUrlPath = window.location.pathname;
                                if (!currentUrlPath.endsWith("/")) {
                                    currentUrlPath = currentUrlPath + "/";
                                }
                                if (!successPage.includes("/#")) {
                                    successPage = successPage.replace("#", "/#");
                                    hashIndex++;
                                }
                                if ([successPage, "/" + session.lang_url_code + successPage].some(link => link.startsWith(currentUrlPath + '#'))) {
                                    successPage = successPage.substring(hashIndex);
                                }
                            }
                            if (successPage.charAt(0) === "#") {
                                const successAnchorEl = document.getElementById(successPage.substring(1));
                                if (successAnchorEl) {
                                    dom.scrollTo(successAnchorEl, {
                                        duration: 500,
                                        extraOffset: 0,
                                    });
                                }
                            } else {
                                $(window.location).attr('href', successPage);
                            }
                            break;
                        }
                        case 'message':
                            self.$target[0].classList.add('d-none');
                            self.$target[0].parentElement.querySelector('.s_website_form_end_message').classList.remove('d-none');
                            break;
                        default:
                            self.update_status('success');
                            break;
                    }

                    // Reset the form
                    self.$target[0].reset();
                }
            })
            .guardedCatch(function () {
                self.update_status('error');
            });
        },

        check_error_fields: function (error_fields) {
            var self = this;
            var form_valid = true;
            // Loop on all fields
            this.$target.find('.form-field, .s_website_form_field').each(function (k, field) { // !compatibility
                var $field = $(field);
                var field_name = $field.find('.col-form-label').attr('for');

                // Validate inputs for this field
                var inputs = $field.find('.s_website_form_input, .o_website_form_input').not('#editable_select'); // !compatibility
                var invalid_inputs = inputs.toArray().filter(function (input, k, inputs) {
                    // Special check for multiple required checkbox for same
                    // field as it seems checkValidity forces every required
                    // checkbox to be checked, instead of looking at other
                    // checkboxes with the same name and only requiring one
                    // of them to be checked.
                    if (input.required && input.type === 'checkbox') {
                        // Considering we are currently processing a single
                        // field, we can assume that all checkboxes in the
                        // inputs variable have the same name
                        var checkboxes = _.filter(inputs, function (input) {
                            return input.required && input.type === 'checkbox';
                        });
                        return !_.any(checkboxes, checkbox => checkbox.checked);

                    // Special cases for dates and datetimes
                    } else if ($(input).hasClass('s_website_form_date') || $(input).hasClass('o_website_form_date')) { // !compatibility
                        if (!self.is_datetime_valid(input.value, 'date')) {
                            return true;
                        }
                    } else if ($(input).hasClass('s_website_form_datetime') || $(input).hasClass('o_website_form_datetime')) { // !compatibility
                        if (!self.is_datetime_valid(input.value, 'datetime')) {
                            return true;
                        }
                    }
                    return !input.checkValidity();
                });

                // Update field color if invalid or erroneous
                $field.removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
                if (invalid_inputs.length || error_fields[field_name]) {
                    $field.addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
                    if (_.isString(error_fields[field_name])) {
                        $field.popover({content: error_fields[field_name], trigger: 'hover', container: 'body', placement: 'top'});
                        // update error message and show it.
                        $field.data("bs.popover").config.content = error_fields[field_name];
                        $field.popover('show');
                    }
                    form_valid = false;
                }
            });
            return form_valid;
        },

        is_datetime_valid: function (value, type_of_date) {
            if (value === "") {
                return true;
            } else {
                try {
                    this.parse_date(value, type_of_date);
                    return true;
                } catch (e) {
                    return false;
                }
            }
        },

        // This is a stripped down version of format.js parse_value function
        parse_date: function (value, type_of_date, value_if_empty) {
            var date_pattern = time.getLangDateFormat(),
                time_pattern = time.getLangTimeFormat();
            var date_pattern_wo_zero = date_pattern.replace('MM', 'M').replace('DD', 'D'),
                time_pattern_wo_zero = time_pattern.replace('HH', 'H').replace('mm', 'm').replace('ss', 's');
            switch (type_of_date) {
                case 'datetime':
                    var datetime = moment(value, [date_pattern + ' ' + time_pattern, date_pattern_wo_zero + ' ' + time_pattern_wo_zero], true);
                    if (datetime.isValid()) {
                        return time.datetime_to_str(datetime.toDate());
                    }
                    throw new Error(_.str.sprintf(_t("'%s' is not a correct datetime"), value));
                case 'date':
                    var date = moment(value, [date_pattern, date_pattern_wo_zero], true);
                    if (date.isValid()) {
                        return time.date_to_str(date.toDate());
                    }
                    throw new Error(_.str.sprintf(_t("'%s' is not a correct date"), value));
            }
            return value;
        },

        update_status: function (status, message) {
            if (status !== 'success') { // Restore send button behavior if result is an error
                this.$target.find('.s_website_form_send, .o_website_form_send')
                    .removeAttr('disabled')
                    .removeClass('disabled'); // !compatibility
            }
            var $result = this.$('#s_website_form_result, #o_website_form_result'); // !compatibility

            if (status === 'error' && !message) {
                message = _t("An error has occured, the form has not been sent.");
            }

            // Note: we still need to wait that the widget is properly started
            // before any qweb rendering which depends on xmlDependencies
            // because the event handlers are binded before the call to
            // willStart for public widgets...
            this.__started.then(() => $result.replaceWith(qweb.render(`website_form.status_${status}`, {
                message: message,
            })));
        },
    });
});

```

## File: static\src\snippets\s_website_form\options.js

```javascript
odoo.define('website_form_editor', function (require) {
'use strict';

const core = require('web.core');
const FormEditorRegistry = require('website_form.form_editor_registry');
const options = require('web_editor.snippets.options');

const qweb = core.qweb;
const _t = core._t;

const FormEditor = options.Class.extend({
    xmlDependencies: [
        '/website_form/static/src/xml/website_form_editor.xml',
        '/google_recaptcha/static/src/xml/recaptcha.xml',
    ],

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * Returns a promise which is resolved once the records of the field
     * have been retrieved.
     *
     * @private
     * @param {Object} field
     * @returns {Promise<Object>}
     */
    _fetchFieldRecords: async function (field) {
        // Convert the required boolean to a value directly usable
        // in qweb js to avoid duplicating this in the templates
        field.required = field.required ? 1 : null;

        if (field.records) {
            return field.records;
        }
        // Set selection as records to avoid added conplexity
        if (field.type === 'selection') {
            field.records = field.selection.map(el => ({
                id: el[0],
                display_name: el[1],
            }));
        } else if (field.relation && field.relation !== 'ir.attachment') {
            field.records = await this._rpc({
                model: field.relation,
                method: 'search_read',
                args: [
                    field.domain,
                    ['display_name']
                ],
            });
        }
        return field.records;
    },
    /**
     * Generates a new ID.
     *
     * @private
     * @returns {string} The new ID
     */
    _generateUniqueID() {
        return `o${Math.random().toString(36).substring(2, 15)}`;
    },
    /**
     * Returns a field object
     *
     * @private
     * @param {string} type the type of the field
     * @param {string} name The name of the field used also as label
     * @returns {Object}
     */
    _getCustomField: function (type, name) {
        return {
            name: name,
            string: name,
            custom: true,
            type: type,
            // Default values for x2many fields and selection
            records: [{
                id: _t('Option 1'),
                display_name: _t('Option 1'),
            }, {
                id: _t('Option 2'),
                display_name: _t('Option 2'),
            }, {
                id: _t('Option 3'),
                display_name: _t('Option 3'),
            }],
        };
    },
    /**
     * Returns the default formatInfos of a field.
     *
     * @private
     * @returns {Object}
     */
    _getDefaultFormat: function () {
        return {
            labelWidth: this.$target[0].querySelector('.s_website_form_label').style.width,
            labelPosition: 'left',
            multiPosition: 'horizontal',
            requiredMark: this._isRequiredMark(),
            optionalMark: this._isOptionalMark(),
            mark: this._getMark(),
        };
    },
    /**
     * @private
     * @returns {string}
     */
    _getMark: function () {
        return this.$target[0].dataset.mark;
    },
    /**
     * @private
     * @returns {boolean}
     */
    _isOptionalMark: function () {
        return this.$target[0].classList.contains('o_mark_optional');
    },
    /**
     * @private
     * @returns {boolean}
     */
    _isRequiredMark: function () {
        return this.$target[0].classList.contains('o_mark_required');
    },
    /**
     * @private
     * @param {Object} field
     * @returns {HTMLElement}
     */
    _renderField: function (field) {
        field.id = this._generateUniqueID();
        const template = document.createElement('template');
        template.innerHTML = qweb.render("website_form.field_" + field.type, {field: field}).trim();
        return template.content.firstElementChild;
    },
});

const FieldEditor = FormEditor.extend({
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.formEl = this.$target[0].closest('form');
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Returns the target as a field Object
     *
     * @private
     * @returns {Object}
     */
    _getActiveField: function () {
        let field;
        const labelText = this.$target.find('.s_website_form_label_content').text();
        if (this._isFieldCustom()) {
            field = this._getCustomField(this.$target[0].dataset.type, labelText);
        } else {
            field = Object.assign({}, this.fields[this._getFieldName()]);
            field.string = labelText;
            field.type = this._getFieldType();
        }
        field.records = this._getListItems();
        this._setActiveProperties(field);
        return field;
    },
    /**
     * Returns the format object of a field containing
     * the position, labelWidth and bootstrap col class
     *
     * @private
     * @returns {Object}
     */
    _getFieldFormat: function () {
        let requiredMark, optionalMark;
        const mark = this.$target[0].querySelector('.s_website_form_mark');
        if (mark) {
            requiredMark = this._isFieldRequired();
            optionalMark = !requiredMark;
        }
        const multipleInput = this._getMultipleInputs();
        const format = {
            labelPosition: this._getLabelPosition(),
            labelWidth: this.$target[0].querySelector('.s_website_form_label').style.width,
            multiPosition: multipleInput && multipleInput.dataset.display || 'horizontal',
            col: [...this.$target[0].classList].filter(el => el.match(/^col-/g)).join(' '),
            requiredMark: requiredMark,
            optionalMark: optionalMark,
            mark: mark && mark.textContent,
        };
        return format;
    },
    /**
     * Returns the name of the field
     *
     * @private
     * @returns {string}
     */
    _getFieldName: function () {
        const multipleName = this.$target[0].querySelector('.s_website_form_multiple');
        return multipleName ? multipleName.dataset.name : this.$target[0].querySelector('.s_website_form_input').name;
    },
    /**
     * Returns the type of the  field, can be used for both custom and existing fields
     *
     * @private
     * @returns {string}
     */
    _getFieldType: function () {
        return this.$target[0].dataset.type;
    },
    /**
     * @private
     * @returns {string}
     */
    _getLabelPosition: function () {
        const label = this.$target[0].querySelector('.s_website_form_label');
        if (this.$target[0].querySelector('.row:not(.s_website_form_multiple)')) {
            return label.classList.contains('text-right') ? 'right' : 'left';
        } else {
            return label.classList.contains('d-none') ? 'none' : 'top';
        }
    },
    /**
     * Returns the multiple checkbox/radio element if it exist else null
     *
     * @private
     * @returns {HTMLElement}
     */
    _getMultipleInputs: function () {
        return this.$target[0].querySelector('.s_website_form_multiple');
    },
    /**
     * @private
     * @returns {string}
     */
    _getPlaceholder: function () {
        const input = this._getPlaceholderInput();
        return input ? input.placeholder : '';
    },
    /**
     * Returns the field's input if it is placeholder compatible, else null
     *
     * @private
     * @returns {HTMLElement}
     */
    _getPlaceholderInput: function () {
        return this.$target[0].querySelector('input[type="text"], input[type="email"], input[type="number"], input[type="tel"], input[type="url"], textarea');
    },
    /**
     * Returns true if the field is a custom field, false if it is an existing field
     *
     * @private
     * @returns {boolean}
     */
    _isFieldCustom: function () {
        return !!this.$target[0].classList.contains('s_website_form_custom');
    },
    /**
     * Returns true if the field is required by the model or by the user.
     *
     * @private
     * @returns {boolean}
     */
    _isFieldRequired: function () {
        const classList = this.$target[0].classList;
        return classList.contains('s_website_form_required') || classList.contains('s_website_form_model_required');
    },
    /**
     * Set the active field properties on the field Object
     *
     * @param {Object} field Field to complete with the active field info
     */
    _setActiveProperties(field) {
        const classList = this.$target[0].classList;
        const textarea = this.$target[0].querySelector('textarea');
        field.placeholder = this._getPlaceholder();
        field.rows = textarea && textarea.rows;
        field.required = classList.contains('s_website_form_required');
        field.modelRequired = classList.contains('s_website_form_model_required');
        field.hidden = classList.contains('s_website_form_field_hidden');
        field.formatInfo = this._getFieldFormat();
    },
    /**
     * Set the placeholder on the current field if the input allow it
     *
     * @private
     * @param {string} value
     */
    _setPlaceholder: function (value) {
        const input = this._getPlaceholderInput();
        if (input) {
            input.placeholder = value;
        }
    },
});

options.registry.WebsiteFormEditor = FormEditor.extend({
    events: _.extend({}, options.Class.prototype.events || {}, {
        'click .toggle-edit-message': '_onToggleEndMessageClick',
    }),

    /**
     * @override
     */
    willStart: async function () {
        const _super = this._super.bind(this);

        // Hide change form parameters option for forms
        // e.g. User should not be enable to change existing job application form
        // to opportunity form in 'Apply job' page.
        this.modelCantChange = this.$target.attr('hide-change-model') !== undefined;
        if (this.modelCantChange) {
            return _super(...arguments);
        }

        // Get list of website_form compatible models.
        this.models = await this._rpc({
            model: "ir.model",
            method: "search_read",
            args: [
                [['website_form_access', '=', true]],
                ['id', 'model', 'name', 'website_form_label', 'website_form_key']
            ],
        });

        const targetModelName = this.$target[0].dataset.model_name || 'mail.mail';
        this.activeForm = _.findWhere(this.models, {model: targetModelName});
        // Create the Form Action select
        this.selectActionEl = document.createElement('we-select');
        this.selectActionEl.setAttribute('string', 'Action');
        this.selectActionEl.dataset.noPreview = 'true';
        this.models.forEach(el => {
            const option = document.createElement('we-button');
            option.textContent = el.website_form_label;
            option.dataset.selectAction = el.id;
            this.selectActionEl.append(option);
        });

        return _super(...arguments);
    },
    /**
     * @override
     */
    start: function () {
        const proms = [this._super(...arguments)];
        // Disable text edition
        this.$target.attr('contentEditable', false);
        // Make button and recaptcha editable
        this.$target.find('.s_website_form_send, .s_website_form_recaptcha').attr('contentEditable', true);
        // Get potential message
        this.$message = this.$target.parent().find('.s_website_form_end_message');
        this.showEndMessage = false;
        // If the form has no model it means a new snippet has been dropped.
        // Apply the default model selected in willStart on it.
        if (!this.$target[0].dataset.model_name) {
            proms.push(this._applyFormModel());
        }
        return Promise.all(proms);
    },
    /**
     * @override
     */
    cleanForSave: function () {
        const model = this.$target[0].dataset.model_name;
        // because apparently this can be called on the wrong widget and
        // we may not have a model, or fields...
        if (model) {
            // we may be re-whitelisting already whitelisted fields. Doesn't
            // really matter.
            const fields = [...this.$target[0].querySelectorAll('.s_website_form_field:not(.s_website_form_custom) .s_website_form_input')].map(el => el.name);
            if (fields.length) {
                // ideally we'd only do this if saving the form
                // succeeds... but no idea how to do that
                this._rpc({
                    model: 'ir.model.fields',
                    method: 'formbuilder_whitelist',
                    args: [model, _.uniq(fields)],
                });
            }
        }
        if (this.$message.length) {
            this.$target.removeClass('d-none');
            this.$message.addClass("d-none");
        }

        // Clear default values coming from data-for/data-values attributes
        this.$target.find('input[name],textarea[name]').each(function () {
            var original = $(this).data('website_form_original_default_value');
            if (original !== undefined && $(this).val() === original) {
                $(this).val('').removeAttr('value');
            }
        });
    },
    /**
     * @override
     */
    updateUI: async function () {
        // If we want to rerender the xml we need to avoid the updateUI
        // as they are asynchronous and the ui might try to update while
        // we are building the UserValueWidgets.
        if (this.rerender) {
            this.rerender = false;
            await this._rerenderXML();
            return;
        }
        await this._super.apply(this, arguments);
        // End Message UI
        this.updateUIEndMessage();
    },
    /**
     * @see this.updateUI
     */
    updateUIEndMessage: function () {
        this.$target.toggleClass("d-none", this.showEndMessage);
        this.$message.toggleClass("d-none", !this.showEndMessage);
        this.$el.find(".toggle-edit-message").toggleClass('text-primary', this.showEndMessage);
    },
    /**
     * @override
     */
    notify: function (name, data) {
        this._super(...arguments);
        if (name === 'field_mark') {
            this._setLabelsMark();
        } else if (name === 'add_field') {
            const field = this._getCustomField('char', 'Custom Text');
            field.formatInfo = data.formatInfo;
            field.formatInfo.requiredMark = this._isRequiredMark();
            field.formatInfo.optionalMark = this._isOptionalMark();
            field.formatInfo.mark = this._getMark();
            const fieldEl = this._renderField(field);
            data.$target.after(fieldEl);
            this.trigger_up('activate_snippet', {
                $snippet: $(fieldEl),
            });
        }
    },

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Select the value of a field (hidden) that will be used on the model as a preset.
     * ie: The Job you apply for if the form is on that job's page.
     */
    addActionField: function (previewMode, value, params) {
        const fieldName = params.fieldName;
        if (params.isSelect === 'true') {
            value = parseInt(value);
        }
        this._addHiddenField(value, fieldName);
    },
    /**
     * Changes the onSuccess event.
     */
    onSuccess: function (previewMode, value, params) {
        this.$target[0].dataset.successMode = value;
        if (value === 'message') {
            if (!this.$message.length) {
                this.$message = $(qweb.render('website_form.s_website_form_end_message'));
            }
            this.$target.after(this.$message);
        } else {
            this.showEndMessage = false;
            this.$message.remove();
        }
    },
    /**
     * Select the model to create with the form.
     */
    selectAction: async function (previewMode, value, params) {
        if (this.modelCantChange) {
            return;
        }
        await this._applyFormModel(parseInt(value));
        this.rerender = true;
    },
    /**
     * @override
     */
    selectClass: function (previewMode, value, params) {
        this._super(...arguments);
        if (params.name === 'field_mark_select') {
            this._setLabelsMark();
        }
    },
    /**
     * Set the mark string on the form
     */
    setMark: function (previewMode, value, params) {
        this.$target[0].dataset.mark = value.trim();
        this._setLabelsMark();
    },
    /**
     * Toggle the recaptcha legal terms
     */
    toggleRecaptchaLegal: function (previewMode, value, params) {
        const recaptchaLegalEl = this.$target[0].querySelector('.s_website_form_recaptcha');
        if (recaptchaLegalEl) {
            recaptchaLegalEl.remove();
        } else {
            const template = document.createElement('template');
            const labelWidth = this.$target[0].querySelector('.s_website_form_label').style.width;
            template.innerHTML = qweb.render("webite_form.s_website_form_recaptcha_legal", {labelWidth: labelWidth});
            const legal = template.content.firstElementChild;
            legal.setAttribute('contentEditable', true);
            this.$target.find('.s_website_form_submit').before(legal);
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState: function (methodName, params) {
        switch (methodName) {
            case 'selectAction':
                return this.activeForm.id;
            case 'addActionField': {
                const value = this.$target.find(`.s_website_form_dnone input[name="${params.fieldName}"]`).val();
                if (value) {
                    return value;
                } else {
                    return params.isSelect ? '0' : '';
                }
            }
            case 'onSuccess':
                return this.$target[0].dataset.successMode;
            case 'setMark':
                return this._getMark();
            case 'toggleRecaptchaLegal':
                return !this.$target[0].querySelector('.s_website_form_recaptcha') || '';
        }
        return this._super(...arguments);
    },
    /**
     * @override
     */
    _renderCustomXML: function (uiFragment) {
        if (this.modelCantChange) {
            return;
        }
        // Add Action select
        const firstOption = uiFragment.childNodes[0];
        uiFragment.insertBefore(this.selectActionEl.cloneNode(true), firstOption);

        // Add Action related options
        const formKey = this.activeForm.website_form_key;
        const formInfo = FormEditorRegistry.get(formKey);
        if (!formInfo || !formInfo.fields) {
            return;
        }
        const proms = formInfo.fields.map(field => this._fetchFieldRecords(field));
        return Promise.all(proms).then(() => {
            formInfo.fields.forEach(field => {
                let option;
                switch (field.type) {
                    case 'many2one':
                        option = this._buildSelect(field);
                        break;
                    case 'char':
                        option = this._buildInput(field);
                        break;
                }
                if (field.required) {
                    // Try to retrieve hidden value in form, else,
                    // get default value or for many2one fields the first option.
                    const currentValue = this.$target.find(`.s_website_form_dnone input[name="${field.name}"]`).val();
                    const defaultValue = field.defaultValue || field.records[0].id;
                    this._addHiddenField(currentValue || defaultValue, field.name);
                }
                uiFragment.insertBefore(option, firstOption);
            });
        });
    },
    /**
     * Add a hidden field to the form
     *
     * @private
     * @param {string} value
     * @param {string} fieldName
     */
    _addHiddenField: function (value, fieldName) {
        this.$target.find(`.s_website_form_dnone:has(input[name="${fieldName}"])`).remove();
        if (value) {
            const hiddenField = qweb.render('website_form.field_hidden', {
                field: {
                    name: fieldName,
                    value: value,
                },
            });
            this.$target.find('.s_website_form_submit').before(hiddenField);
        }
    },
    /**
     * Returns a we-input element from the field
     *
     * @private
     * @param {Object} field
     * @returns {HTMLElement}
     */
    _buildInput: function (field) {
        const inputEl = document.createElement('we-input');
        inputEl.dataset.noPreview = 'true';
        inputEl.dataset.fieldName = field.name;
        inputEl.dataset.addActionField = '';
        inputEl.setAttribute('string', field.string);
        inputEl.classList.add('o_we_large_input');
        return inputEl;
    },
    /**
     * Returns a we-select element with field's records as it's options
     *
     * @private
     * @param {Object} field
     * @return {HTMLElement}
     */
    _buildSelect: function (field) {
        const selectEl = document.createElement('we-select');
        selectEl.dataset.noPreview = 'true';
        selectEl.dataset.fieldName = field.name;
        selectEl.dataset.isSelect = 'true';
        selectEl.setAttribute('string', field.string);
        if (!field.required) {
            const noneButton = document.createElement('we-button');
            noneButton.textContent = 'None';
            noneButton.dataset.addActionField = 0;
            selectEl.append(noneButton);
        }
        field.records.forEach(el => {
            const button = document.createElement('we-button');
            button.textContent = el.display_name;
            button.dataset.addActionField = el.id;
            selectEl.append(button);
        });
        return selectEl;
    },
    /**
     * Apply the model on the form changing it's fields
     *
     * @private
     * @param {Integer} modelId
     */
    _applyFormModel: async function (modelId) {
        let oldFormInfo;
        if (modelId) {
            const oldFormKey = this.activeForm.website_form_key;
            if (oldFormKey) {
                oldFormInfo = FormEditorRegistry.get(oldFormKey);
            }
            this.$target.find('.s_website_form_field').remove();
            this.activeForm = _.findWhere(this.models, {id: modelId});
        }
        const formKey = this.activeForm.website_form_key;
        const formInfo = FormEditorRegistry.get(formKey);
        // Success page
        if (!this.$target[0].dataset.successMode) {
            this.$target[0].dataset.successMode = 'redirect';
        }
        if (this.$target[0].dataset.successMode === 'redirect') {
            const currentSuccessPage = this.$target[0].dataset.successPage;
            if (formInfo && formInfo.successPage) {
                this.$target[0].dataset.successPage = formInfo.successPage;
            } else if (!oldFormInfo || (oldFormInfo !== formInfo && oldFormInfo.successPage && currentSuccessPage === oldFormInfo.successPage)) {
                this.$target[0].dataset.successPage = '/contactus-thank-you';
            }
        }
        // Model name
        this.$target[0].dataset.model_name = this.activeForm.model;
        // Load template
        if (formInfo) {
            const formatInfo = this._getDefaultFormat();
            await formInfo.formFields.forEach(async field => {
                field.formatInfo = formatInfo;
                await this._fetchFieldRecords(field);
                this.$target.find('.s_website_form_submit, .s_website_form_recaptcha').first().before(this._renderField(field));
            });
        }
    },
    /**
     * Set the correct mark on all fields.
     *
     * @private
     */
    _setLabelsMark: function () {
        this.$target[0].querySelectorAll('.s_website_form_mark').forEach(el => el.remove());
        const mark = this._getMark();
        if (!mark) {
            return;
        }
        let fieldsToMark = [];
        const requiredSelector = '.s_website_form_model_required, .s_website_form_required';
        const fields = Array.from(this.$target[0].querySelectorAll('.s_website_form_field'));
        if (this._isRequiredMark()) {
            fieldsToMark = fields.filter(el => el.matches(requiredSelector));
        } else if (this._isOptionalMark()) {
            fieldsToMark = fields.filter(el => !el.matches(requiredSelector));
        }
        fieldsToMark.forEach(field => {
            let span = document.createElement('span');
            span.classList.add('s_website_form_mark');
            span.textContent = ` ${mark}`;
            field.querySelector('.s_website_form_label').appendChild(span);
        });
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     */
    _onToggleEndMessageClick: function () {
        this.showEndMessage = !this.showEndMessage;
        this.updateUIEndMessage();
        this.trigger_up('activate_snippet', {
            $snippet: this.showEndMessage ? this.$message : this.$target,
        });
    },
});

const authorizedFieldsCache = {};

options.registry.WebsiteFieldEditor = FieldEditor.extend({
    events: _.extend({}, FieldEditor.prototype.events, {
        'click we-button.o_we_select_remove_option': '_onRemoveItemClick',
        'click we-button.o_we_list_add_optional': '_onAddCustomItemClick',
        'click we-button.o_we_list_add_existing': '_onAddExistingItemClick',
        'click we-list we-select': '_onAddItemSelectClick',
        'input we-list input': '_onListItemInput',
    }),

    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.rerender = true;
    },
    /**
     * @override
     */
    willStart: async function () {
        const _super = this._super.bind(this);
        // Get the authorized existing fields for the form model
        const model = this.formEl.dataset.model_name;
        let getFields;
        if (model in authorizedFieldsCache) {
            getFields = authorizedFieldsCache[model];
        } else {
            getFields = this._rpc({
                model: "ir.model",
                method: "get_authorized_fields",
                args: [model],
            });
            authorizedFieldsCache[model] = getFields;
        }

        this.existingFields = await getFields.then(fields => {
            this.fields = _.each(fields, function (field, fieldName) {
                field.name = fieldName;
                field.domain = field.domain || [];
            });
            // Create the buttons for the type we-select
            return Object.keys(fields).map(key => {
                const field = fields[key];
                const button = document.createElement('we-button');
                button.textContent = field.string;
                button.dataset.existingField = field.name;
                return button;
            }).sort((a, b) => (a.textContent > b.textContent) ? 1 : (a.textContent < b.textContent) ? -1 : 0);
        });
        return _super(...arguments);
    },
    /**
     * @override
     */
    cleanForSave: function () {
        this.$target[0].querySelectorAll('#editable_select').forEach(el => el.remove());
        const select = this._getSelect();
        if (select && this.listTable) {
            select.style.display = '';
            select.innerHTML = '';
            // Rebuild the select from the we-list
            this.listTable.querySelectorAll('input').forEach(el => {
                const option = document.createElement('option');
                option.textContent = el.value;
                option.value = this._isFieldCustom() ? el.value : el.name;
                select.appendChild(option);
            });
        }
    },
    /**
     * @override
     */
    updateUI: async function () {
        // See Form updateUI
        if (this.rerender) {
            const select = this._getSelect();
            if (select && !this.$target[0].querySelector('#editable_select')) {
                select.style.display = 'none';
                const editableSelect = document.createElement('div');
                editableSelect.id = 'editable_select';
                editableSelect.classList = 'form-control s_website_form_input';
                select.parentElement.appendChild(editableSelect);
            }
            this.rerender = false;
            await this._rerenderXML().then(() => this._renderList());
            return;
        }
        await this._super.apply(this, arguments);
    },
    /**
     * @override
     */
    onFocus: function () {
        // Other fields type might have change to an existing type.
        // We need to reload the existing type list.
        this.rerender = true;
    },
    /**
     * Rerenders the clone to avoid id duplicates.
     * Rendering the list before replacing the field is needed
     * for selects, in order to build this.listTable.
     *
     * @override
     */
    onClone() {
        this._renderList();
        const field = this._getActiveField();
        const fieldEl = this._renderField(field);
        this._replaceFieldElement(fieldEl);
    },

    //----------------------------------------------------------------------
    // Options
    //----------------------------------------------------------------------

    /**
     * Replace the current field with the custom field selected.
     */
    customField: async function (previewMode, value, params) {
        // Both custom Field and existingField are called when selecting an option
        // value is '' for the method that should not be called.
        if (!value) {
            return;
        }
        const name = this.el.querySelector(`[data-custom-field="${value}"]`).textContent;
        const field = this._getCustomField(value, `Custom ${name}`);
        this._setActiveProperties(field);
        await this._replaceField(field);
        this.rerender = true;
    },
    /**
     * Replace the current field with the existing field selected.
     */
    existingField: async function (previewMode, value, params) {
        // see customField
        if (!value) {
            return;
        }
        const field = Object.assign({}, this.fields[value]);
        this._setActiveProperties(field);
        await this._replaceField(field);
        this.rerender = true;
    },
    /**
     * Set the name of the field on the label
     */
    setLabelText: function (previewMode, value, params) {
        this.$target.find('.s_website_form_label_content').text(value);
        if (this._isFieldCustom()) {
            const multiple = this.$target[0].querySelector('.s_website_form_multiple');
            if (multiple) {
                multiple.dataset.name = value;
            }
            this.$target[0].querySelectorAll('.s_website_form_input').forEach(el => el.name = value);
        }
    },
    /*
    * Set the placeholder of the input
    */
    setPlaceholder: function (previewMode, value, params) {
        this._setPlaceholder(value);
    },
    /**
     * Replace the field with the same field having the label in a different position.
     */
    selectLabelPosition: async function (previewMode, value, params) {
        const field = this._getActiveField();
        field.formatInfo.labelPosition = value;
        await this._replaceField(field);
        this.rerender = true;
    },
    selectType: async function (previewMode, value, params) {
        const field = this._getActiveField();
        field.type = value;
        await this._replaceField(field);
    },
    /**
     * Select the display of the multicheckbox field (vertical & horizontal)
     */
    multiCheckboxDisplay: function (previewMode, value, params) {
        const target = this._getMultipleInputs();
        target.querySelectorAll('.checkbox, .radio').forEach(el => {
            if (value === 'horizontal') {
                el.classList.add('col-lg-4', 'col-md-6');
            } else {
                el.classList.remove('col-lg-4', 'col-md-6');
            }
        });
        target.dataset.display = value;
    },
    /**
     * Set the field as required or not
     */
    toggleRequired: function (previewMode, value, params) {
        const isRequired = this.$target[0].classList.contains(params.activeValue);
        this.$target[0].classList.toggle(params.activeValue, !isRequired);
        this.$target[0].querySelectorAll('input, select, textarea').forEach(el => el.toggleAttribute('required', !isRequired));
        this.trigger_up('option_update', {
            optionName: 'WebsiteFormEditor',
            name: 'field_mark',
        });
    },

    //----------------------------------------------------------------------
    // Private
    //----------------------------------------------------------------------

    /**
     * @override
     */
    _computeWidgetState: function (methodName, params) {
        switch (methodName) {
            case 'customField':
                return this._isFieldCustom() ? this._getFieldType() : '';
            case 'existingField':
                return this._isFieldCustom() ? '' : this._getFieldName();
            case 'setLabelText':
                return this.$target.find('.s_website_form_label_content').text();
            case 'setPlaceholder':
                return this._getPlaceholder();
            case 'selectLabelPosition':
                return this._getLabelPosition();
            case 'selectType':
                return this._getFieldType();
            case 'multiCheckboxDisplay': {
                const target = this._getMultipleInputs();
                return target ? target.dataset.display : '';
            }
            case 'toggleRequired':
                return this.$target[0].classList.contains(params.activeValue) ? params.activeValue : 'false';
        }
        return this._super(...arguments);
    },
    /**
     * @override
     */
    _computeWidgetVisibility: function (widgetName, params) {
        switch (widgetName) {
            case 'char_input_type_opt':
                return !this.$target[0].classList.contains('s_website_form_custom') && ['char', 'email', 'tel', 'url'].includes(this.$target[0].dataset.type);
            case 'multi_check_display_opt':
                return !!this._getMultipleInputs();
            case 'placeholder_opt':
                return !!this._getPlaceholderInput();
            case 'required_opt':
            case 'hidden_opt':
            case 'type_opt':
                return !this.$target[0].classList.contains('s_website_form_model_required');
        }
        return this._super(...arguments);
    },
    /**
     * @override
     */
    _renderCustomXML: function (uiFragment) {
        const selectEl = uiFragment.querySelector('we-select[data-name="type_opt"]');
        const currentFieldName = this._getFieldName();
        const fieldsInForm = Array.from(this.formEl.querySelectorAll('.s_website_form_field:not(.s_website_form_custom) .s_website_form_input')).map(el => el.name).filter(el => el !== currentFieldName);
        const availableFields = this.existingFields.filter(el => !fieldsInForm.includes(el.dataset.existingField));
        if (availableFields.length) {
            const title = document.createElement('we-title');
            title.textContent = 'Existing fields';
            availableFields.unshift(title);
            availableFields.forEach(option => selectEl.append(option.cloneNode(true)));
        }
    },
    /**
     * Replaces the target content with the field provided.
     *
     * @private
     * @param {Object} field
     * @returns {Promise}
     */
    _replaceField: async function (field) {
        await this._fetchFieldRecords(field);
        const fieldEl = this._renderField(field);
        this._replaceFieldElement(fieldEl);
    },
    /**
     * Replaces the target with provided field.
     *
     * @private
     * @param {HTMLElement} fieldEl
     */
    _replaceFieldElement(fieldEl) {
        [...this.$target[0].childNodes].forEach(node => node.remove());
        [...fieldEl.childNodes].forEach(node => this.$target[0].appendChild(node));
        [...fieldEl.attributes].forEach(el => this.$target[0].removeAttribute(el.nodeName));
        [...fieldEl.attributes].forEach(el => this.$target[0].setAttribute(el.nodeName, el.nodeValue));
    },

    /**
     * To do after rerenderXML to add the list to the options
     *
     * @private
     */
    _renderList: function () {
        let addItemButton, addItemTitle, listTitle;
        const select = this._getSelect();
        const multipleInputs = this._getMultipleInputs();
        this.listTable = document.createElement('table');
        const isCustomField = this._isFieldCustom();

        if (select) {
            listTitle = 'Options List';
            addItemTitle = 'Add new Option';
            select.querySelectorAll('option').forEach(opt => {
                this._addItemToTable(opt.value, opt.textContent.trim());
            });
            this._renderListItems();
        } else if (multipleInputs) {
            listTitle = multipleInputs.querySelector('.radio') ? 'Radio List' : 'Checkbox List';
            addItemTitle = 'Add new Checkbox';
            multipleInputs.querySelectorAll('.checkbox, .radio').forEach(opt => {
                this._addItemToTable(opt.querySelector('input').value, opt.querySelector('.s_website_form_check_label').textContent.trim());
            });
        } else {
            return;
        }

        if (isCustomField) {
            addItemButton = document.createElement('we-button');
            addItemButton.textContent = addItemTitle;
            addItemButton.classList.add('o_we_list_add_optional');
            addItemButton.dataset.noPreview = 'true';
        } else {
            addItemButton = document.createElement('we-select');
            addItemButton.classList.add('o_we_user_value_widget'); // Todo dont use user value widget class
            const togglerEl = document.createElement('we-toggler');
            togglerEl.textContent = addItemTitle;
            addItemButton.appendChild(togglerEl);
            const selectMenuEl = document.createElement('we-selection-items');
            addItemButton.appendChild(selectMenuEl);
            this._loadListDropdown(selectMenuEl);
        }
        const selectInputEl = document.createElement('we-list');
        const title = document.createElement('we-title');
        title.textContent = listTitle;
        selectInputEl.appendChild(title);
        const tableWrapper = document.createElement('div');
        tableWrapper.classList.add('oe_we_table_wraper');
        tableWrapper.appendChild(this.listTable);
        selectInputEl.appendChild(tableWrapper);
        selectInputEl.appendChild(addItemButton);
        this.el.insertBefore(selectInputEl, this.el.querySelector('[data-set-placeholder]'));
        this._makeListItemsSortable();
    },
    /**
     * Load the dropdown of the list with the records missing from the list.
     *
     * @private
     * @param {HTMLElement} selectMenu
     */
    _loadListDropdown: function (selectMenu) {
        selectMenu = selectMenu || this.el.querySelector('we-list we-selection-items');
        if (selectMenu) {
            selectMenu.innerHTML = '';
            const field = Object.assign({}, this.fields[this._getFieldName()]);
            this._fetchFieldRecords(field).then(() => {
                let buttonItems;
                const optionIds = Array.from(this.listTable.querySelectorAll('input')).map(opt => {
                    return field.type === 'selection' ? opt.name : parseInt(opt.name);
                });
                const availableRecords = (field.records || []).filter(el => !optionIds.includes(el.id));
                if (availableRecords.length) {
                    buttonItems = availableRecords.map(el => {
                        const option = document.createElement('we-button');
                        option.classList.add('o_we_list_add_existing');
                        option.dataset.addOption = el.id;
                        option.dataset.noPreview = 'true';
                        option.textContent = el.display_name;
                        return option;
                    });
                } else {
                    const title = document.createElement('we-title');
                    title.textContent = 'No more records';
                    buttonItems = [title];
                }
                buttonItems.forEach(button => selectMenu.appendChild(button));
            });
        }
    },
    /**
     * @private
     */
    _makeListItemsSortable: function () {
        $(this.listTable).sortable({
            axis: 'y',
            handle: '.o_we_drag_handle',
            items: 'tr',
            cursor: 'move',
            opacity: 0.6,
            stop: (event, ui) => {
                this._renderListItems();
            },
        });
    },
    /**
     * @private
     * @param {string} id
     * @param {string} text
     */
    _addItemToTable: function (id, text) {
        const isCustomField = this._isFieldCustom();
        const draggableEl = document.createElement('we-button');
        draggableEl.classList.add('o_we_drag_handle', 'o_we_link', 'fa', 'fa-fw', 'fa-arrows');
        draggableEl.dataset.noPreview = 'true';
        const inputEl = document.createElement('input');
        inputEl.type = 'text';
        if (text) {
            inputEl.value = text;
        }
        if (!isCustomField && id) {
            inputEl.name = id;
        }
        inputEl.disabled = !isCustomField;
        const trEl = document.createElement('tr');
        const buttonEl = document.createElement('we-button');
        buttonEl.classList.add('o_we_select_remove_option', 'o_we_link', 'o_we_text_danger', 'fa', 'fa-fw', 'fa-minus');
        buttonEl.dataset.removeOption = id;
        buttonEl.dataset.noPreview = 'true';
        const draggableTdEl = document.createElement('td');
        const inputTdEl = document.createElement('td');
        const buttonTdEl = document.createElement('td');
        draggableTdEl.appendChild(draggableEl);
        trEl.appendChild(draggableTdEl);
        inputTdEl.appendChild(inputEl);
        trEl.appendChild(inputTdEl);
        buttonTdEl.appendChild(buttonEl);
        trEl.appendChild(buttonTdEl);
        this.listTable.appendChild(trEl);
        if (isCustomField) {
            inputEl.focus();
        }
    },
    /**
     * Apply the we-list on the target and rebuild the input(s)
     *
     * @private
     */
    _renderListItems: function () {
        const multiInputsWrap = this._getMultipleInputs();
        const selectWrap = this.$target[0].querySelector('#editable_select');
        const isRequiredField = this._isFieldRequired();
        const name = this._getFieldName();
        if (multiInputsWrap) {
            const type = multiInputsWrap.querySelector('.radio') ? 'radio' : 'checkbox';
            multiInputsWrap.innerHTML = '';
            const params = {
                field: {
                    name: name,
                    id: this._generateUniqueID(),
                    required: isRequiredField,
                    formatInfo: {
                        multiPosition: multiInputsWrap.dataset.display,
                    }
                }
            };
            this._getListItems().forEach((record, idx) => {
                params.record_index = idx;
                params.record = record;
                const template = document.createElement('template');
                template.innerHTML = qweb.render(`website_form.${type}`, params);
                multiInputsWrap.appendChild(template.content.firstElementChild);
            });
        } else if (selectWrap) {
            selectWrap.innerHTML = '';
            this.listTable.querySelectorAll('input').forEach(el => {
                const option = document.createElement('div');
                option.id = (el.name || el.value);
                option.classList.add('s_website_form_select_item');
                option.textContent = el.value;
                selectWrap.appendChild(option);
            });
        }
    },
    /**
     * Returns an array based on the we-list containing the field's records
     *
     * @returns {Array}
     */
    _getListItems: function () {
        if (!this.listTable) {
            return null;
        }
        const isCustomField = this._isFieldCustom();
        const records = [];
        this.listTable.querySelectorAll('input').forEach(el => {
            const id = isCustomField ? el.value : el.name;
            records.push({
                id: id,
                display_name: el.value,
            });
        });
        return records;
    },
    /**
     * Returns the select element if it exist else null
     *
     * @private
     * @returns {HTMLElement}
     */
    _getSelect: function () {
        return this.$target[0].querySelector('select');
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @private
     * @param {Event} ev
     */
    _onRemoveItemClick: function (ev) {
        ev.target.closest('tr').remove();
        this._loadListDropdown();
        this._renderListItems();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onAddCustomItemClick: function (ev) {
        this._addItemToTable();
        this._makeListItemsSortable();
        this._renderListItems();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onAddExistingItemClick: function (ev) {
        const value = ev.currentTarget.dataset.addOption;
        this._addItemToTable(value, ev.currentTarget.textContent);
        this._makeListItemsSortable();
        this._loadListDropdown();
        this._renderListItems();
    },
    /**
     * @private
     * @param {Event} ev
     */
    _onAddItemSelectClick: function (ev) {
        ev.currentTarget.querySelector('we-toggler').classList.toggle('active');
    },
    /**
     * @private
     */
    _onListItemInput: function () {
        this._renderListItems();
    },
});

options.registry.AddFieldForm = FormEditor.extend({
    isTopOption: true,

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Add a char field at the end of the form.
     * New field is set as active
     */
    addField: async function (previewMode, value, params) {
        const field = this._getCustomField('char', 'Custom Text');
        field.formatInfo = this._getDefaultFormat();
        const fieldEl = this._renderField(field);
        this.$target.find('.s_website_form_submit, .s_website_form_recaptcha').first().before(fieldEl);
        this.trigger_up('activate_snippet', {
            $snippet: $(fieldEl),
        });
    },
});

options.registry.AddField = FieldEditor.extend({
    isTopOption: true,

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    /**
     * Add a char field with active field properties after the active field.
     * New field is set as active
     */
    addField: async function (previewMode, value, params) {
        this.trigger_up('option_update', {
            optionName: 'WebsiteFormEditor',
            name: 'add_field',
            data: {
                formatInfo: this._getFieldFormat(),
                $target: this.$target,
            },
        });
    },
});

// Superclass for options that need to disable a button from the snippet overlay
const DisableOverlayButtonOption = options.Class.extend({
    // Disable a button of the snippet overlay
    disableButton: function (buttonName, message) {
        // TODO refactor in master
        const className = 'oe_snippet_' + buttonName;
        this.$overlay.add(this.$overlay.data('$optionsSection')).on('click', '.' + className, this.preventButton);
        const $button = this.$overlay.add(this.$overlay.data('$optionsSection')).find('.' + className);
        $button.attr('title', message).tooltip({delay: 0});
        // TODO In master: add `o_disabled` but keep actual class.
        $button.removeClass(className); // Disable the functionnality
    },

    preventButton: function (event) {
        // Snippet options bind their functions before the editor, so we
        // can't cleanly unbind the editor onRemove function from here
        event.preventDefault();
        event.stopImmediatePropagation();
    }
});

// Disable duplicate button for model fields
options.registry.WebsiteFormFieldModel = DisableOverlayButtonOption.extend({
    start: function () {
        this.disableButton('clone', _t('You can\'t duplicate a model field.'));
        return this._super.apply(this, arguments);
    }
});

// Disable delete button for model required fields
options.registry.WebsiteFormFieldRequired = DisableOverlayButtonOption.extend({
    start: function () {
        this.disableButton('remove', _t('You can\'t remove a field that is required by the model itself.'));
        return this._super.apply(this, arguments);
    }
});

// Disable delete and duplicate button for submit
options.registry.WebsiteFormSubmitRequired = DisableOverlayButtonOption.extend({
    start: function () {
        this.disableButton('remove', _t('You can\'t remove the submit button of the form'));
        this.disableButton('clone', _t('You can\'t duplicate the submit button of the form.'));
        return this._super.apply(this, arguments);
    }
});
});

```

## File: static\src\xml\website_form.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <!-- Success status -->
    <t t-name="website_form.status_success">
        <span id="s_website_form_result" class="text-success ml8">
            <i class="fa fa-check mr4" role="img" aria-label="Success" title="Success"/>The form has been sent successfully.
        </span>
    </t>

    <!-- Error status -->
    <t t-name="website_form.status_error">
        <span id="s_website_form_result" class="text-danger ml8">
            <i class="fa fa-close mr4" role="img" aria-label="Error" title="Error"/>
            <t t-esc="message"/>
        </span>
    </t>
</templates>

```

## File: static\src\xml\website_form_editor.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <!-- End Message -->
    <t t-name="website_form.s_website_form_end_message">
        <div class="s_website_form_end_message d-none">
            <div class="oe_structure">
                <section class="s_text_block pt64 pb64 o_colored_level o_cc o_cc2" data-snippet="s_text_block">
                    <div class="container">
                        <h2 class="text-center">
                            <span class="fa fa-check-circle"/>
                                Thank You For Your Feedback
                        </h2>
                        <p class="text-center">
                            Our team will message you back as soon as possible.<br/>
                            In the meantime we invite you to visit our <a href="/">website</a>.<br/>
                        </p>
                    </div>
                </section>
            </div>
        </div>
    </t>

    <t t-name="webite_form.s_website_form_recaptcha_legal">
        <div class="col-12 s_website_form_recaptcha" data-name="Recaptcha Legal">
            <div t-attf-style="width: #{labelWidth or '200px'}" class="s_website_form_label"/>
            <div class="col-sm">
                <t t-call="google_recaptcha.recaptcha_legal_terms"/>
            </div>
        </div>
    </t>

    <!-- Generic Field Layout -->
    <!-- Changes made here needs to be reflected in the different Form view (Contact Us, Jobs, ...) -->
    <t t-name="website_form.field">
        <div t-attf-class="form-group s_website_form_field #{field.formatInfo.col or 'col-12'} #{field.custom and 's_website_form_custom' or ''} #{(field.required and 's_website_form_required' or '') or (field.modelRequired and 's_website_form_model_required' or '')} #{field.hidden and 's_website_form_field_hidden' or ''} #{field.dnone and 's_website_form_dnone' or ''}"
            t-att-data-type="field.type"
            data-name="Field">
            <div t-if="field.formatInfo.labelPosition != 'none' and field.formatInfo.labelPosition != 'top'" class="row s_col_no_resize s_col_no_bgcolor">
                <label t-attf-class="#{!field.isCheck and 'col-form-label' or ''} col-sm-auto s_website_form_label #{field.formatInfo.labelPosition == 'right' and 'text-right' or ''}" t-attf-style="width: #{field.formatInfo.labelWidth or '200px'}" t-att-for="field.id">
                     <t t-call="website_form.label_content"/>
                </label>
                <div class="col-sm">
                    <t t-raw="0"/>
                </div>
            </div>
            <t t-else="">
                <label t-attf-class="s_website_form_label #{field.formatInfo.labelPosition == 'none' and 'd-none' or ''}" t-attf-style="width: #{field.formatInfo.labelWidth or '200px'}" t-att-for="field.id">
                     <t t-call="website_form.label_content"/>
                </label>
                <t t-raw="0"/>
            </t>
        </div>
    </t>

    <t t-name="website_form.label_content">
        <t t-if="field.custom" t-set="field.string" t-value="field.name"/>
        <span class="s_website_form_label_content" t-esc="field.string"/>
        <t t-if="field.required or field.modelRequired">
            <span class="s_website_form_mark" t-if="field.formatInfo.requiredMark" t-esc="' ' + field.formatInfo.mark"/>
        </t>
        <t t-else="">
            <span class="s_website_form_mark" t-if="field.formatInfo.optionalMark" t-esc="' ' + field.formatInfo.mark"/>
        </t>
        <span t-if="['email_cc', 'email_to'].includes(field.name)" title="Separate email addresses with a comma.">
            <i class="fa fa-info-circle"/>
        </span>
    </t>

    <!-- Hidden Field -->
    <t t-name="website_form.field_hidden">
        <t t-set="field.dnone" t-value="true"/>
        <t t-set="field.formatInfo" t-value="{}"/>
        <t t-call="website_form.field">
            <input
                type="hidden"
                class="form-control s_website_form_input"
                t-att-name="field.name"
                t-att-value="field.value"
                t-att-id="field.id"
            />
        </t>
    </t>

    <!-- Char Field -->
    <t t-name="website_form.field_char">
        <t t-call="website_form.field">
            <input
                t-att-type="field.inputType || 'text'"
                class="form-control s_website_form_input"
                t-att-name="field.name"
                t-att-required="field.required || field.modelRequired || None"
                t-att-value="field.value"
                t-att-placeholder="field.placeholder"
                t-att-id="field.id"
            />
        </t>
    </t>

    <!-- Email Field -->
    <t t-name="website_form.field_email">
        <t t-set="field.inputType" t-value="'email'"/>
        <t t-call="website_form.field_char"/>
    </t>

    <!-- Telephone Field -->
    <t t-name="website_form.field_tel">
        <t t-set="field.inputType" t-value="'tel'"/>
        <t t-call="website_form.field_char"/>
    </t>

    <!-- Url Field -->
    <t t-name="website_form.field_url">
        <t t-set="field.inputType" t-value="'url'"/>
        <t t-call="website_form.field_char"/>
    </t>

    <!-- Text Field -->
    <t t-name="website_form.field_text">
        <t t-call="website_form.field">
            <textarea
                class="form-control s_website_form_input"
                t-att-name="field.name"
                t-att-required="field.required || field.modelRequired || None"
                t-att-placeholder="field.placeholder"
                t-att-id="field.id"
                t-att-rows="field.rows || 3"
            />
        </t>
    </t>

    <!-- HTML Field -->
    <t t-name="website_form.field_html">
        <!--
            Maybe use web_editor ? Not sure it actually makes
            sense to have random people editing html in a form...
        -->
        <t t-call="website_form.field_text"/>
    </t>

    <!-- Integer Field -->
    <t t-name="website_form.field_integer">
        <t t-call="website_form.field">
            <input
                type="number"
                class="form-control s_website_form_input"
                t-att-name="field.name"
                step="1"
                t-att-required="field.required || field.modelRequired || None"
                t-att-placeholder="field.placeholder"
                t-att-id="field.id"
            />
        </t>
    </t>

    <!-- Float Field -->
    <t t-name="website_form.field_float">
        <t t-call="website_form.field">
            <input
                type="number"
                class="form-control s_website_form_input"
                t-att-name="field.name"
                step="any"
                t-att-required="field.required || field.modelRequired || None"
                t-att-placeholder="field.placeholder"
                t-att-id="field.id"
            />
        </t>
    </t>

    <!-- Date Field -->
    <t t-name="website_form.field_date">
        <t t-call="website_form.field">
            <t t-set="datepickerID" t-value="'datepicker' + Math.random().toString().substring(2)"/>
            <div class="s_website_form_date input-group date" t-att-id="datepickerID" data-target-input="nearest">
                <input
                        type="text"
                        class="form-control datetimepicker-input s_website_form_input"
                        t-attf-data-target="##{datepickerID}"
                        t-att-name="field.name"
                        t-att-required="field.required || field.modelRequired || None"
                        t-att-placeholder="field.placeholder"
                        t-att-id="field.id"
                />
                <div class="input-group-append" t-attf-data-target="##{datepickerID}" data-toggle="datetimepicker">
                    <div class="input-group-text"><i class="fa fa-calendar"></i></div>
                </div>
            </div>
        </t>
    </t>

    <!-- Datetime Field -->
    <t t-name="website_form.field_datetime">
        <t t-call="website_form.field">
            <t t-set="datetimepickerID" t-value="'datetimepicker' + Math.random().toString().substring(2)"/>
            <div class="s_website_form_datetime input-group date" t-att-id="datetimepickerID" data-target-input="nearest">
                <input
                        type="text"
                        class="form-control datetimepicker-input s_website_form_input"
                        t-attf-data-target="##{datetimepickerID}"
                        t-att-name="field.name"
                        t-att-required="field.required || field.modelRequired || None"
                        t-att-placeholder="field.placeholder"
                        t-att-id="field.id"
                />
                <div class="input-group-append" t-attf-data-target="##{datetimepickerID}" data-toggle="datetimepicker">
                    <div class="input-group-text"><i class="fa fa-calendar"></i></div>
                </div>
            </div>
        </t>
    </t>

    <!-- Boolean Field -->
    <t t-name="website_form.field_boolean">
        <t t-set="field.isCheck" t-value="true"/>
        <t t-call="website_form.field">
            <input
                type="checkbox"
                value="Yes"
                class="s_website_form_input"
                t-att-name="field.name"
                t-att-required="field.required || field.modelRequired || None"
                t-att-id="field.id"
            />
        </t>
    </t>

    <!-- Selection Field -->
    <t t-name="website_form.field_selection">
        <t t-set="field.isCheck" t-value="true"/>
        <t t-call="website_form.field">
            <div class="row s_col_no_resize s_col_no_bgcolor s_website_form_multiple" t-att-data-name="field.name" t-att-data-display="field.formatInfo.multiPosition">
                <t t-if="!field.records">
                    <input
                        class="s_website_form_input"
                        t-att-name="field.name"
                        t-att-value="record.id"
                        t-att-required="field.required || field.modelRequired || None"
                        placeholder="No matching record !"
                    />
                </t>
                <t t-foreach="field.records" t-as="record">
                    <t t-call="website_form.radio"/>
                </t>
            </div>
        </t>
    </t>

    <!-- Radio -->
    <t t-name="website_form.radio">
        <t t-set="recordId" t-value="field.id + record_index"/>
        <div t-attf-class="radio col-12 #{field.formatInfo.multiPosition === 'horizontal' and 'col-lg-4 col-md-6' or ''}">
            <div class="form-check">
                <input
                    type="radio"
                    class="s_website_form_input form-check-input"
                    t-att-id="recordId"
                    t-att-name="field.name"
                    t-att-value="record.id"
                    t-att-required="field.required || field.modelRequired || None"
                />
                <label class="form-check-label s_website_form_check_label" t-att-for="recordId">
                    <t t-esc="record.display_name"/>
                </label>
            </div>
        </div>
    </t>

    <!-- Many2One Field -->
    <t t-name="website_form.field_many2one">
        <!-- Binary one2many -->
        <t t-if="field.relation == 'ir.attachment'">
            <t t-call="website_form.field_binary"/>
        </t>
        <!-- Generic one2many -->
        <t t-if="field.relation != 'ir.attachment'">
            <t t-call="website_form.field">
                <select class="form-control s_website_form_input" t-att-name="field.name" t-att-required="field.required || field.modelRequired || None" t-att-id="field.id">
                    <t t-foreach="field.records" t-as="record">
                        <option t-att-value="record.id" t-att-selected="record.selected">
                            <t t-esc="record.display_name"/>
                        </option>
                    </t>
                </select>
            </t>
        </t>
    </t>

    <!-- One2Many Field -->
    <t t-name="website_form.field_one2many">
        <!-- Binary one2many -->
        <t t-if="field.relation == 'ir.attachment'">
            <t t-call="website_form.field_binary">
                <t t-set="multiple" t-value="1"/>
            </t>
        </t>
        <!-- Generic one2many -->
        <t t-if="field.relation != 'ir.attachment'">
            <t t-set="field.isCheck" t-value="true"/>
            <t t-call="website_form.field">
                <div class="row s_col_no_resize s_col_no_bgcolor s_website_form_multiple" t-att-data-name="field.name" t-att-data-display="field.formatInfo.multiPosition">
                    <t t-if="!field.records">
                        <input
                            class="s_website_form_input"
                            t-att-name="field.name"
                            t-att-value="record.id"
                            t-att-required="field.required || field.modelRequired || None"
                            placeholder="No matching record !"
                        />
                    </t>
                    <t t-foreach="field.records" t-as="record">
                        <t t-call="website_form.checkbox"/>
                    </t>
                </div>
            </t>
        </t>
    </t>

    <!-- Checkbox -->
    <t t-name="website_form.checkbox">
        <t t-set="recordId" t-value="field.id + record_index"/>
        <div t-attf-class="checkbox col-12 #{field.formatInfo.multiPosition === 'horizontal' and 'col-lg-4 col-md-6' or ''}">
            <div class="form-check">
                <input
                    type="checkbox"
                    class="s_website_form_input form-check-input"
                    t-att-id="recordId"
                    t-att-name="field.name"
                    t-att-value="record.id"
                    t-att-required="field.required || field.modelRequired || None"
                />
                <label class="form-check-label s_website_form_check_label" t-att-for="recordId">
                    <t t-esc="record.display_name"/>
                </label>
            </div>
        </div>
    </t>

    <!-- Many2Many Field -->
    <t t-name="website_form.field_many2many">
        <t t-call="website_form.field_one2many"/>
    </t>

    <!-- Binary Field -->
    <t t-name="website_form.field_binary">
        <t t-set="field.isCheck" t-value="true"/>
        <t t-call="website_form.field">
            <input
                type="file"
                class="form-control-file s_website_form_input"
                t-att-name="field.name"
                t-att-required="field.required || field.modelRequired || None"
                t-att-multiple="multiple"
                t-att-id="field.id"
            />
        </t>
    </t>

    <!-- Monetary Field -->
    <t t-name="website_form.field_monetary">
        <t t-call="website_form.field_float" />
    </t>
</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <template id="assets_editor" name="Website Form Editor Assets Editor" inherit_id="website.assets_editor">
            <xpath expr="." position="inside">
                <link rel="stylesheet" type="text/scss" href="/website_form/static/src/scss/wysiwyg_snippets.scss"/>
                <script type="text/javascript" src="/website_form/static/src/snippets/s_website_form/options.js"/>
                <script type="text/javascript" src="/website_form/static/src/js/website_form_editor_registry.js"/>
            </xpath>
        </template>

        <template id="assets_tests" name="Website Form Assets Tests" inherit_id="web.assets_tests">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/website_form/static/tests/tours/website_form_editor.js"/>
            </xpath>
        </template>
    </data>
</odoo>

```

## File: views\ir_model_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="ir_model_view" model="ir.ui.view">
        <field name="name">website_form_editor.ir.model.view.form</field>
        <field name="model">ir.model</field>
        <field name="inherit_id" ref="base.view_model_form"/>
        <field name="arch" type="xml">
            <xpath expr="//notebook" position="inside">
                <page string="Website Forms" name="website_forms">
                    <group>
                        <field name="website_form_access"/>
                        <field name="website_form_label"/>
                        <field name="website_form_default_field_id"/>
                    </group>
                </page>
            </xpath>

            <xpath expr="//page[@name='base']/group/group/field[@name='translate']" position="after">
                <field name="website_form_blacklisted"/>
            </xpath>
        </field>
    </record>

    <record id="ir_model_fields_view" model="ir.ui.view">
        <field name="name">website_form_editor.ir.model.fields.view.form</field>
        <field name="model">ir.model.fields</field>
        <field name="inherit_id" ref="base.view_model_fields_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='translate']" position="after">
                <field name="website_form_blacklisted"/>
            </xpath>
        </field>
    </record>


</odoo>

```

## File: views\website_form_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="contactus_form" name="Contact Form" inherit_id="website.contactus" customize_show="True">
        <xpath expr="//div[@name='mail_button']" position="replace">
            <span class="hidden" data-for="contactus_form" t-att-data-values="{'email_to': res_company.email}" />
            <div id="contactus_section">
                <section class="s_website_form" data-vcss="001" data-snippet="s_website_form">
                    <div class="container">
                        <form id="contactus_form" action="/website_form/" method="post" enctype="multipart/form-data" class="o_mark_required" data-mark="*" data-model_name="mail.mail" data-success-mode="redirect" data-success-page="/contactus-thank-you">
                            <div class="s_website_form_rows row s_col_no_bgcolor">
                                <div class="form-group col-12 s_website_form_field s_website_form_custom s_website_form_required" data-type="char" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="contact1">
                                            <span class="s_website_form_label_content">Your Name</span>
                                            <span class="s_website_form_mark"> *</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="contact1" type="text" class="form-control s_website_form_input" name="Name" required=""/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_custom" data-type="char" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="contact2">
                                            <span class="s_website_form_label_content">Phone Number</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="contact2" type="tel" class="form-control s_website_form_input" name="Phone"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_required s_website_form_model_required" data-type="email" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="contact3">
                                            <span class="s_website_form_label_content">Email</span>
                                            <span class="s_website_form_mark"> *</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="contact3" type="email" class="form-control s_website_form_input" name="email_from" required=""/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_custom" data-type="char" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="contact4">
                                            <span class="s_website_form_label_content">Your Company</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="contact4" type="text" class="form-control s_website_form_input" name="Partner Name"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_required s_website_form_model_required" data-type="char" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="contact5">
                                            <span class="s_website_form_label_content">Subject</span>
                                            <span class="s_website_form_mark"> *</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="contact5" type="text" class="form-control s_website_form_input" name="subject" required=""/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_custom s_website_form_required" data-type="text" data-name="Field">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="contact6">
                                            <span class="s_website_form_label_content">Your Question</span>
                                        </label>
                                        <div class="col-sm">
                                            <textarea id="contact6" class="form-control s_website_form_input" name="Description" required=""></textarea>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_field s_website_form_dnone">
                                    <div class="row s_col_no_resize s_col_no_bgcolor">
                                        <label class="col-form-label col-sm-auto s_website_form_label" style="width: 200px" for="contact7">
                                            <span class="s_website_form_label_content">Email To</span>
                                        </label>
                                        <div class="col-sm">
                                            <input id="contact7" type="hidden" class="form-control s_website_form_input" name="email_to"/>
                                        </div>
                                    </div>
                                </div>
                                <div class="form-group col-12 s_website_form_submit" data-name="Submit Button">
                                    <div style="width: 200px;" class="s_website_form_label"/>
                                    <a href="#" role="button" class="btn btn-primary btn-lg s_website_form_send">Submit</a>
                                    <span id="s_website_form_result"></span>
                                </div>
                            </div>
                        </form>
                    </div>
                </section>
            </div>
        </xpath>
    </template>

    <record id="contactus_thanks" model="website.page">
        <field name="name">Thanks (Contact us)</field>
        <field name="type">qweb</field>
        <field name="url">/contactus-thank-you</field>
        <field name="website_indexed" eval="False"/>
        <field name="is_published">True</field>
        <field name="key">website_form.contactus_thanks</field>
        <field name="arch" type="xml">
            <t name="Thanks (Contact us)" t-name="website_form.contactus_thanks">
                <t t-call="website.layout">
                  <div id="wrap">
                    <div class="oe_structure" id="oe_structure_website_form_contact_us_thanks_1"/>
                    <div class="container mt-4">
                        <div class="row">
                            <div class="col-lg-7 col-xl-6 mr-lg-auto oe_structure">
                                <section class="pt40 s_text_block pb40 o_colored_level o_cc o_cc1" data-snippet="s_text_block" data-name="Text">
                                    <div class="container">
                                        <span class="d-block fa fa-4x fa-thumbs-up mx-auto rounded-circle bg-primary"/><br/>
                                        <h1 class="text-center">Thank You!</h1>
                                        <div class="pb16 pt16 s_hr" data-snippet="s_hr" data-name="Separator">
                                            <hr class="mx-auto border-top w-50 border-dark text-center"/>
                                        </div>
                                        <h5 class="text-center">
                                            <span class="fa fa-check-circle"/>
                                            <span>Your message has been sent <b>successfully</b></span>
                                        </h5>
                                        <p class="text-center">We will get back to you shortly.</p>
                                    </div>
                                </section>
                            </div>
                            <div class="col-lg-4">
                                <t t-call="website.company_description"/>
                            </div>
                        </div>
                    </div>
                    <div class="oe_structure" id="oe_structure_website_form_contact_us_thanks_2"/>
                  </div>
                </t>
            </t>
        </field>
    </record>

</odoo>

```

## File: views\snippets\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="remove_external_snippets" inherit_id="website.external_snippets">
    <xpath expr="//t[@t-install='website_form']" position="replace"/>
</template>

<template id="snippets" inherit_id="website.snippets" name="Snippet Form Builder">
    <xpath expr="//t[@id='form_form_hook']" position="replace">
        <!-- This snippet cannot be used in sanitized fields -->
        <!-- because it contains inputs that would be removed -->
        <t t-snippet="website_form.s_website_form" t-thumbnail="/website/static/src/img/snippets_thumbs/s_website_form.svg" t-forbid-sanitize="form"/>
    </xpath>
</template>

</odoo>

```

## File: views\snippets\s_website_form.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="s_website_form" name="Form">
    <section class="s_website_form pt16 pb16" data-vcss="001" data-snippet="s_website_form">
        <div class="container">
            <form action="/website_form/" method="post" enctype="multipart/form-data" class="o_mark_required" data-mark="*">
                <div class="s_website_form_rows row s_col_no_bgcolor">
                    <div class="form-group col-12 s_website_form_submit" data-name="Submit Button">
                        <div style="width: 200px;" class="s_website_form_label"/>
                        <a href="#" role="button" class="btn btn-primary btn-lg s_website_form_send">Submit</a>
                        <span id="s_website_form_result"></span>
                    </div>
                </div>
            </form>
        </div>
    </section>
</template>

<template id="s_website_form_options" inherit_id="website.snippet_options">
    <!-- Extend drop locations to columns -->
    <xpath expr="//t[@t-set='so_content_addition_selector']" position="inside">, .s_website_form</xpath>

    <xpath expr="//div" position="after">
        <!-- Form -->
        <div data-js="WebsiteFormEditor" data-selector=".s_website_form" data-target="form">
            <we-select string="Marked Fields" data-name="field_mark_select">
                <we-button data-select-class="">None</we-button>
                <we-button data-select-class="o_mark_required" data-name="form_required_opt">Required</we-button>
                <we-button data-select-class="o_mark_optional" data-name="form_optional_opt">Optional</we-button>
            </we-select>
            <we-input string="Mark Text" data-set-mark="" data-dependencies="form_required_opt, form_optional_opt"/>
            <we-input string="Labels Width"
                data-select-style="" data-css-property="width"
                data-unit="px" data-apply-to=".s_website_form_label"/>
            <we-row>
                <we-select string="On Success" data-no-preview="true">
                    <we-button data-on-success="nothing">Nothing</we-button>
                    <we-button data-on-success="redirect" data-name="show_redirect_opt">Redirect</we-button>
                    <we-button data-on-success="message" data-name="show_message_opt">Show Message</we-button>
                </we-select>
                <we-button class="fa fa-fw fa-eye align-self-end toggle-edit-message" title="Edit Message" data-name="message_opt" data-dependencies="show_message_opt"/>
            </we-row>
            <we-urlpicker string="URL" data-select-data-attribute="/contactus-thank-you" data-attribute-name="successPage" data-name="url_opt" data-dependencies="show_redirect_opt"/>
            <t t-set="recaptcha_public_key" t-value="request.env['ir.config_parameter'].sudo().get_param('recaptcha_public_key')"/>
            <we-checkbox t-if="recaptcha_public_key" string="Show reCaptcha Policy" data-toggle-recaptcha-legal="" data-no-preview="true"/>
        </div>

        <!-- Add Field Form -->
        <div data-js="AddFieldForm" data-selector=".s_website_form" data-target="form">
            <we-button class="o_we_text_success"
                title="Add a new field at the end"
                data-add-field=""
                data-no-preview="true">
                <i class="fa fa-fw fa-plus"/>
            </we-button>
        </div>

        <!-- Add Field -->
        <div data-js="AddField" data-selector=".s_website_form_field" data-exclude=".s_website_form_dnone">
            <we-button class="o_we_text_success"
                title="Add a new field after this one"
                data-add-field=""
                data-no-preview="true">
                <i class="fa fa-fw fa-plus"/>
            </we-button>
        </div>

        <!-- Field -->
        <div data-js='WebsiteFieldEditor' data-selector=".s_website_form_field"
             data-exclude=".s_website_form_dnone" data-drop-near=".s_website_form_field">
            <we-select data-name="type_opt" string="Type" data-no-preview="true">
                <we-title>Custom field</we-title>
                <we-button data-custom-field="char">Text</we-button>
                <we-button data-custom-field="text">Long Text</we-button>
                <we-button data-custom-field="email">Email</we-button>
                <we-button data-custom-field="tel">Telephone</we-button>
                <we-button data-custom-field="url">Url</we-button>
                <we-button data-custom-field="integer">Number</we-button>
                <we-button data-custom-field="float">Decimal Number</we-button>
                <we-button data-custom-field="boolean">Checkbox</we-button>
                <we-button data-custom-field="one2many">Multiple Checkboxes</we-button>
                <we-button data-custom-field="selection">Radio Buttons</we-button>
                <we-button data-custom-field="many2one">Selection</we-button>
                <we-button data-custom-field="date">Date</we-button>
                <we-button data-custom-field="datetime">Date &amp; Time</we-button>
                <we-button data-custom-field="binary">File Upload</we-button>
            </we-select>
            <we-select data-name="char_input_type_opt" string="Input Type" data-no-preview="true">
                <we-button data-select-type="char">Text</we-button>
                <we-button data-select-type="email">Email</we-button>
                <we-button data-select-type="tel">Telephone</we-button>
                <we-button data-select-type="url">Url</we-button>
            </we-select>
            <t t-set="unit_textarea_height">rows</t>
            <we-input string="Height" data-step="1" t-attf-data-select-attribute="3#{unit_textarea_height}" t-att-data-unit="unit_textarea_height"
                data-attribute-name="rows" data-apply-to="textarea"/>
            <we-select string="Display" data-name="multi_check_display_opt" data-no-preview="true">
                <we-button data-multi-checkbox-display="horizontal">Horizontal</we-button>
                <we-button data-multi-checkbox-display="vertical">Vertical</we-button>
            </we-select>
            <we-input string="Input Placeholder" class="o_we_large_input" data-name="placeholder_opt" data-set-placeholder=""/>
            <we-input string="Label Name" class="o_we_large_input" data-set-label-text=""/>
            <we-button-group string="Label Position">
                <we-button title="Hide"
                           data-select-label-position="none">
                    <i class="fa fa-eye-slash"/>
                </we-button>
                <we-button title="Top"
                           data-select-label-position="top"
                           data-img="/website/static/src/img/snippets_options/pos_top.svg"/>
                <we-button title="Left"
                           data-select-label-position="left"
                           data-img="/website/static/src/img/snippets_options/pos_left.svg"/>
                <we-button title="Right"
                           data-select-label-position="right"
                           data-img="/website/static/src/img/snippets_options/pos_right.svg"/>
            </we-button-group>
            <we-checkbox string="Required" data-name="required_opt" data-no-preview="true"
                data-toggle-required="s_website_form_required"/>
            <we-checkbox string="Hidden" data-name="hidden_opt" data-no-preview="true"
                data-select-class="s_website_form_field_hidden"/>
        </div>

        <div data-js="WebsiteFormSubmit" data-selector=".s_website_form_submit" data-exclude=".s_website_form_no_submit_options">
            <we-select string="Button Position">
                <we-button data-select-class="text-left s_website_form_no_submit_label">Left</we-button>
                <we-button data-select-class="text-center s_website_form_no_submit_label">Center</we-button>
                <we-button data-select-class="text-right s_website_form_no_submit_label">Right</we-button>
                <we-button data-select-class="">Input Aligned</we-button>
            </we-select>
        </div>

        <!-- Remove the duplicate option of model fields -->
        <div data-js="WebsiteFormFieldModel" data-selector=".s_website_form .s_website_form_field:not(.s_website_form_custom)"/>

        <!-- Remove the delete option of model required fields -->
        <div data-js="WebsiteFormFieldRequired" data-selector=".s_website_form .s_website_form_model_required"/>

        <!-- Remove the delete and duplicate option of the submit button -->
        <div data-js="WebsiteFormSubmitRequired" data-selector=".s_website_form .s_website_form_submit"/>
    </xpath>
</template>

<template id="assets_snippet_s_website_form_css_000" inherit_id="website.assets_frontend" active="False">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website_form/static/src/snippets/s_website_form/000.scss"/>
    </xpath>
</template>
<template id="assets_snippet_s_website_form_css_001" inherit_id="website.assets_frontend">
    <xpath expr="//link[last()]" position="after">
        <link rel="stylesheet" type="text/scss" href="/website_form/static/src/snippets/s_website_form/001.scss"/>
    </xpath>
</template>

<template id="assets_snippet_s_website_form_js_000" inherit_id="website.assets_frontend">
    <xpath expr="//script[last()]" position="after">
        <script type="text/javascript" src="/website_form/static/src/snippets/s_website_form/000.js"/>
    </xpath>
</template>

</odoo>

```

