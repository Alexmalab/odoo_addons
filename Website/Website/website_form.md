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
    'depends': ['website', 'mail'],
    'data': [
        'data/mail_mail_data.xml',
        'views/assets.xml',
        'views/ir_model_views.xml',
        'views/res_config_view.xml',
        'views/snippets.xml',
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
from odoo.tools.translate import _
from odoo.exceptions import ValidationError
from odoo.addons.base.models.ir_qweb_fields import nl2br


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

        model_record = request.env['ir.model'].sudo().search([('model', '=', model_name), ('website_form_access', '=', True)])
        if not model_record:
            return json.dumps(False)

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

    _meta_label = "%s\n________\n\n" % _("Metadata")  # Title for meta data

    # Dict of dynamically called filters following type of field to be fault tolerent

    def identity(self, field_label, field_input):
        return field_input

    def integer(self, field_label, field_input):
        return int(field_input)

    def floating(self, field_label, field_input):
        return float(field_input)

    def boolean(self, field_label, field_input):
        return bool(field_input)

    def date(self, field_label, field_input):
        try:
            lang = request.env['ir.qweb.field'].user_lang()
            dt = datetime.strptime(field_input, lang.date_format)
        except ValueError:
            dt = datetime.strptime(field_input, DEFAULT_SERVER_DATE_FORMAT)
        return dt.strftime(DEFAULT_SERVER_DATE_FORMAT)

    def datetime(self, field_label, field_input):
        lang = request.env['ir.qweb.field'].user_lang()
        strftime_format = (u"%s %s" % (lang.date_format, lang.time_format))
        user_tz = pytz.timezone(request.context.get('tz') or request.env.user.tz or 'UTC')
        try:
            dt = user_tz.localize(datetime.strptime(field_input, strftime_format)).astimezone(pytz.utc)
        except ValueError:
            dt = datetime.strptime(field_input, DEFAULT_SERVER_DATETIME_FORMAT)
        return dt.strftime(DEFAULT_SERVER_DATETIME_FORMAT)

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
        'date': date,
        'datetime': datetime,
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
            elif field_name != 'context':
                custom_fields.append((field_name, field_value))

        data['custom'] = "\n".join([u"%s : %s" % v for v in custom_fields])

        # Add metadata if enabled
        environ = request.httprequest.headers.environ
        if(request.website.website_form_enable_metadata):
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
            values.update({'reply_to': values.get('email_from')})
        record = request.env[model_name].with_user(SUPERUSER_ID).with_context(mail_create_nosubscribe=True).create(values)

        if custom or meta:
            _custom_label = "%s\n___________\n\n" % _("Other Information:")  # Title for custom fields
            if model_name == 'mail.mail':
                _custom_label = "%s\n___________\n\n" % _("This message has been posted on your website!")
            default_field = model.website_form_default_field_id
            default_field_data = values.get(default_field.name, '')
            custom_content = (default_field_data + "\n\n" if default_field_data else '') \
                           + (_custom_label + custom + "\n\n" if custom else '') \
                           + (self._meta_label + meta if meta else '')

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
                record.sudo()[file.field_name] = [(4, attachment_id.id)]
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

## File: models\models.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, SUPERUSER_ID
from odoo.http import request


class website_form_config(models.Model):
    _inherit = 'website'

    website_form_enable_metadata = fields.Boolean('Technical data on contact form', help="You can choose to log technical data like IP, User Agent ,...")

    def _website_form_last_record(self):
        if request and request.session.form_builder_model_model:
            return request.env[request.session.form_builder_model_model].browse(request.session.form_builder_id)
        return False


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'
    website_form_enable_metadata = fields.Boolean(related="website_id.website_form_enable_metadata", readonly=False)


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
            if fields_get[field].get('readonly') or field in MAGIC_FIELDS:
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

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70"><defs><path id="a" d="M4 0h61c4 0 5 1 5 5v60c0 4-1 5-5 5H4c-3 0-4-1-4-5V5c0-4 1-5 4-5z"/><linearGradient id="c" x1="100%" x2="0%" y1="0%" y2="100%"><stop offset="0%" stop-color="#B06161"/><stop offset="45.785%" stop-color="#984E4E"/><stop offset="100%" stop-color="#7C3838"/></linearGradient></defs><g fill="none" fill-rule="evenodd"><mask id="b" fill="#fff"><use xlink:href="#a"/></mask><g mask="url(#b)"><path fill="url(#c)" d="M0 0H70V70H0z"/><path fill="#FFF" fill-opacity=".383" d="M4 1h61c2.667 0 4.333.667 5 2V0H0v3c.667-1.333 2-2 4-2z"/><path fill="#393939" d="M41.118 69H4c-2 0-4-1-4-4V28.375L15 12h41v42L41.118 69z" opacity=".324"/><path fill="#000" fill-opacity=".383" d="M4 69h61c2.667 0 4.333-1 5-3v4H0v-4c.667 2 2 3 4 3z"/><path fill="#FFF" d="M34 17v-2h2v2h2v2h-2v2h-2v-2h-2v-2h2zM15 42h41v12H15V42zm19.51 8.985a.608.608 0 0 0 .73 0l5.942-4.792c.202-.162.202-.426 0-.589l-.73-.59a.608.608 0 0 0-.731 0l-4.846 3.909-2.262-1.825a.608.608 0 0 0-.731 0l-.73.59c-.202.162-.202.426 0 .589l3.358 2.708zM15 12h41v12H15V12zm0 15h41v12H15V27zm2-13v8h37v-8H17zm0 15v8h37v-8H17zm15 3h6v2h-6v-2z"/></g></g></svg>
```

## File: static\src\js\website_form.js

```javascript
odoo.define('website_form.animation', function (require) {
'use strict';

    var core = require('web.core');
    var time = require('web.time');
    var ajax = require('web.ajax');
    var publicWidget = require('web.public.widget');

    var _t = core._t;
    var qweb = core.qweb;

    publicWidget.registry.form_builder_send = publicWidget.Widget.extend({
        selector: '.s_website_form',

        willStart: function () {
            var prom;
            if (!$.fn.datetimepicker) {
                prom = ajax.loadJS("/web/static/lib/tempusdominus/tempusdominus.js");
            }
            return Promise.all([this._super.apply(this, arguments), prom]);
        },

        start: function (editable_mode) {
            if (editable_mode) {
                this.stop();
                return;
            }
            var self = this;
            this.templates_loaded = ajax.loadXML('/website_form/static/src/xml/website_form.xml', qweb);
            this.$target.find('.o_website_form_send').on('click',function (e) {self.send(e);});

            // Initialize datetimepickers
            var l10n = _t.database.parameters;
            var datepickers_options = {
                minDate: moment({ y: 1900 }),
                maxDate: moment({ y: 9999, M: 11, d: 31 }),
                calendarWeeks: true,
                icons : {
                    time: 'fa fa-clock-o',
                    date: 'fa fa-calendar',
                    next: 'fa fa-chevron-right',
                    previous: 'fa fa-chevron-left',
                    up: 'fa fa-chevron-up',
                    down: 'fa fa-chevron-down',
                   },
                locale : moment.locale(),
                format : time.getLangDatetimeFormat(),
            };
            this.$target.find('.o_website_form_datetime').datetimepicker(datepickers_options);

            // Adapt options to date-only pickers
            datepickers_options.format = time.getLangDateFormat();
            this.$target.find('.o_website_form_date').datetimepicker(datepickers_options);

            // Display form values from tag having data-for attribute
            // It's necessary to handle field values generated on server-side
            // Because, using t-att- inside form make it non-editable
            var $values = $('[data-for=' + this.$target.attr('id') + ']');
            if ($values.length) {
                var values = JSON.parse($values.data('values').replace('False', '""').replace('None', '""').replace(/'/g, '"'));
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

            return this._super.apply(this, arguments);
        },

        destroy: function () {
            this._super.apply(this, arguments);
            this.$target.find('button').off('click');
        },

        send: function (e) {
            e.preventDefault();  // Prevent the default submit behavior
            this.$target.find('.o_website_form_send')
                .off('click')
                .addClass('disabled')
                .attr('disabled', 'disabled');  // Prevent users from crazy clicking

            var self = this;

            self.$target.find('#o_website_form_result').empty();
            if (!self.check_error_fields({})) {
                self.update_status('invalid');
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

            // force server format if usage of textual month that would not be understood server-side
            if (time.getLangDatetimeFormat().indexOf('MMM') !== 1) {
                this.$target.find('.form-field:not(.o_website_form_custom)')
                .find('.o_website_form_date, .o_website_form_datetime').each(function () {
                    var date = $(this).datetimepicker('viewDate').clone().locale('en');
                    var format = 'YYYY-MM-DD';
                    if ($(this).hasClass('o_website_form_datetime')) {
                        date = date.utc();
                        format = 'YYYY-MM-DD HH:mm:ss';
                    }
                    form_values[$(this).find('input').attr('name')] = date.format(format);
                });
            }

            // Post form and handle result
            self.post_form(form_values)
        },

        post_form: function(form_values) {
            var self = this;
            ajax.post(this.$target.attr('action') + (this.$target.data('force_action')||this.$target.data('model_name')), form_values)
            .then(function (result_data) {
                result_data = JSON.parse(result_data);
                if (!result_data.id) {
                    // Failure, the server didn't return the created record ID
                    self.update_status('error');
                    if (result_data.error_fields) {
                        // If the server return a list of bad fields, show these fields for users
                        self.check_error_fields(result_data.error_fields);
                    }
                } else {
                    // Success, redirect or update status
                    var success_page = self.$target.attr('data-success_page');
                    if (success_page) {
                        $(window.location).attr('href', success_page);
                    }
                    else {
                        self.update_status('success');
                    }

                    // Reset the form
                    self.$target[0].reset();
                }
            })
            .guardedCatch(function (){
                self.update_status('error');
            });
        },

        check_error_fields: function (error_fields) {
            var self = this;
            var form_valid = true;
            // Loop on all fields
            this.$target.find('.form-field').each(function (k, field){
                var $field = $(field);
                var field_name = $field.find('.col-form-label').attr('for');

                // Validate inputs for this field
                var inputs = $field.find('.o_website_form_input:not(#editable_select)');
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
                        var checkboxes = _.filter(inputs, function (input){
                            return input.required && input.type === 'checkbox';
                        });
                        return !_.any(checkboxes, function (checkbox) { return checkbox.checked; });

                    // Special cases for dates and datetimes
                    } else if ($(input).hasClass('o_website_form_date')) {
                        if (!self.is_datetime_valid(input.value, 'date')) {
                            return true;
                        }
                    } else if ($(input).hasClass('o_website_form_datetime')) {
                        if (!self.is_datetime_valid(input.value, 'datetime')) {
                            return true;
                        }
                    }
                    return !input.checkValidity();
                });

                // Update field color if invalid or erroneous
                $field.removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
                if (invalid_inputs.length || error_fields[field_name]){
                    $field.addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid')
                    if (_.isString(error_fields[field_name])){
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
            var date_pattern_wo_zero = date_pattern.replace('MM','M').replace('DD','D'),
                time_pattern_wo_zero = time_pattern.replace('HH','H').replace('mm','m').replace('ss','s');
            switch (type_of_date) {
                case 'datetime':
                    var datetime = moment(value, [date_pattern + ' ' + time_pattern, date_pattern_wo_zero + ' ' + time_pattern_wo_zero], true);
                    if (datetime.isValid())
                        return time.datetime_to_str(datetime.toDate());
                    throw new Error(_.str.sprintf(_t("'%s' is not a correct datetime"), value));
                case 'date':
                    var date = moment(value, [date_pattern, date_pattern_wo_zero], true);
                    if (date.isValid())
                        return time.date_to_str(date.toDate());
                    throw new Error(_.str.sprintf(_t("'%s' is not a correct date"), value));
            }
            return value;
        },

        update_status: function (status) {
            var self = this;
            if (status !== 'success') {  // Restore send button behavior if result is an error
                this.$target.find('.o_website_form_send')
                    .removeClass('disabled')
                    .removeAttr('disabled')
                    .on('click', function (e) {
                        self.send(e);
                    });
            }
            var $result = this.$('#o_website_form_result');
            this.templates_loaded.then(function () {
                $result.replaceWith(qweb.render("website_form.status_" + status));
            });
        },
    });

    return publicWidget.registry.form_builder_send
});

```

## File: static\src\js\website_form_editor.js

```javascript
odoo.define('website_form_editor', function (require) {
    'use strict';

    /**
     * @todo this should be entirely refactored
     */

    var ajax = require('web.ajax');
    var core = require('web.core');
    var Dialog = require('web.Dialog');
    var FormEditorRegistry = require('website_form.form_editor_registry');
    var options = require('web_editor.snippets.options');
    var wUtils = require('website.utils');
    var Wysiwyg = require('web_editor.wysiwyg');

    var qweb = core.qweb;
    var _t = core._t;

    var FormEditorDialog = Dialog.extend({
        /**
         * @constructor
         */
        init: function (parent, options) {
            this._super(parent, _.extend({
                buttons: [{
                    text: _t('Save'),
                    classes: 'btn-primary',
                    close: true,
                    click: this._onSaveModal.bind(this),
                }, {
                    text: _t('Cancel'),
                    close: true
                }],
            }, options));
        },

        //----------------------------------------------------------------------
        // Handlers
        //----------------------------------------------------------------------

        /**
         * @private
         */
        _onSaveModal: function () {
            if (this.$el[0].checkValidity()) {
                this.trigger_up('save');
            } else {
                _.each(this.$el.find('.o_website_form_input'), function (input) {
                    var $field = $(input).closest('.form-field');
                    $field.removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');
                    if (!input.checkValidity()) {
                        $field.addClass('o_has_error').find('.form-control, .custom-select').addClass('is-invalid');
                    }
                });
            }
        },
    });

    options.registry['website_form_editor'] = options.Class.extend({
        xmlDependencies: ['/website_form/static/src/xml/website_form_editor.xml'],

        start: function () {
            this.$target.addClass('o_fake_not_editable').attr('contentEditable', false);
            this.$target.find('label:not(:has(span)), label span, .o_form_heading').addClass('o_fake_editable').attr('contentEditable', true);
            this.$target.find('.o_website_form_send').attr('contentEditable', true);
            return this._super.apply(this, arguments);
        },

        // Return the fields promise if we already issued a model
        // fields fetch request, or issue said request.
        fields: function () {
            return this.fields_promise || this.fetch_model_fields();
        },

        fetch_model_fields: function () {
            return this._rpc({
                model: "ir.model",
                method: "get_authorized_fields",
                args: [this.$target.closest('form').attr('data-model_name')],
            }).then(function (fields) {
                // The get_fields function doesn't return the name
                // in the field dict since it uses it has the key
                _.each(fields, function (field, field_name) {
                    field.name = field_name;
                });
                return fields;
            });
        },

        // Choose a model modal
        website_form_model_modal: function (previewMode, value, $li) {
            var self = this;
            this._rpc({
                model: "ir.model",
                method: "search_read",
                args: [
                    [['website_form_access', '=', true]],
                    ['id', 'model', 'name', 'website_form_label', 'website_form_key']
                ],
            }).then(function (models) {
                self.models = models;
                var selectedModel = self.$target.attr('data-model_name') || 'mail.mail';
                // Models selection input
                var modelSelection = qweb.render("website_form.field_many2one", {
                    field: {
                        name: 'model_selection',
                        string: 'Action',
                        required: true,
                        records: _.map(models, function (m) {
                            return {
                                id: m.id,
                                display_name: m.website_form_label || m.name,
                                selected: (m.model === selectedModel) ? 1 : null,
                            };
                        }),
                    }
                });

                // Success page input
                var successPage = qweb.render("website_form.field_char", {
                    field: {
                        name: 'success_page',
                        string: 'Thank You Page',
                        value: self.$target.attr('data-success_page')
                    }
                });

                var save = function () {
                    var successPage = this.$el.find("[name='success_page']").val();
                    self.init_form();
                    self.$target.attr('data-success_page', successPage);

                    this.$el.find('.o_form_parameter_custom').each(function () {
                        var $field = $(this).find('.o_website_form_input');
                        var value = $field.val();
                        var fieldName = $field.attr('name');
                        self.$target.find('.form-group:has("[name=' + fieldName + ']")').remove();
                        if (value) {
                            var $hiddenField = $(qweb.render('website_form.field_char', {
                                field: {
                                    name: fieldName,
                                    value: value,
                                }
                            })).addClass('d-none');
                            self.$target.find('.form-group:has(".o_website_form_send")').before($hiddenField);
                        }
                    });
                };

                var cancel = function () {
                    if (!self.$target.attr('data-model_name')) {
                        self.$target.remove();
                    }
                };

                var $content = $('<form role="form">' + modelSelection + successPage + '</form>');
                var dialog = new FormEditorDialog(self, {
                    title: 'Form Parameters',
                    size: 'medium',
                    $content: $content,
                }).open();
                dialog.on('closed', this, cancel);
                dialog.on('save', this, ev => {
                    ev.stopPropagation();
                    save.call(dialog);
                });

                wUtils.autocompleteWithPages(self, $content.find("input[name='success_page']"));
                self.originSuccessPage = $content.find("input[name='success_page']").val();
                self.originFormID = $content.find("[name='model_selection']").val();
                self._renderParameterFields($content);

                $content.find("[name='model_selection']").on('change', function () {
                    self._renderParameterFields($content);
                });
            });
        },

        //--------------------------------------------------------------------------
        // Private
        //--------------------------------------------------------------------------

        /**
         * @private
         * @returns {Promise}
         */
        _renderParameterFields: function ($modal) {
            var self = this;
            var $successPage = $modal.find("[name='success_page']");
            $modal.find('.o_form_parameter_custom').remove();
            var id = $modal.find("[name='model_selection']").val();
            this.activeForm = _.findWhere(this.models, {id: parseInt(id)});
            var formKey = this.activeForm.website_form_key;
            if (!formKey) {
                return Promise.resolve();
            }
            var proms = [];
            var formInfo = FormEditorRegistry.get(formKey);

            if (this.originFormID === id) {
                $successPage.val(this.originSuccessPage || formInfo.successPage || '/contactus-thank-you');
            } else {
                $successPage.val(formInfo.successPage || '/contactus-thank-you');
            }

            if (formInfo.fields && formInfo.fields.length) {
                _.each(formInfo.fields, function (field) {
                    var value = self.$target.find('[name="' + field.name + '"]').val();
                    proms.push(self.render_field(field).then(function ($field) {
                        $field.addClass('o_form_parameter_custom');
                        // Remove content editable (Added by render_field)
                        $field.find('label').removeAttr('contenteditable');
                        // Set tooltip on label
                        $field.find('label').attr('title', field.title);
                        // Set value
                        $field.find('.o_website_form_input').val(value);
                        $modal.append($field);
                    }));
                });
            }
            return Promise.all(proms);
        },

        // Choose a field modal
        website_form_field_modal: function (previewMode, value, $li) {
            var self = this;

            this.fields().then(function (fields) {
                // Make a nice array to render the select input
                var fields_array = _.map(fields, function (v, k) { return {id: k, name: v.name, display_name: v.string}; });
                // Filter the fields to remove the ones already in the form
                var fields_in_form = _.map(self.$target.find('.col-form-label'), function (label) { return label.getAttribute('for'); });
                var available_fields = _.filter(fields_array, function (field) { return !_.contains(fields_in_form, field.name); });
                // Render the select input
                var fieldSelection = qweb.render("website_form.field_many2one", {
                    field: {
                        name: 'field_selection',
                        string: 'Field',
                        records: _.sortBy(available_fields, 'display_name')
                    }
                });

                var save = function () {
                    var selectedFieldName = this.$el.find("[name='field_selection']").val();
                    var selectedField = fields[selectedFieldName];
                    self.append_field(selectedField);
                };

                var dialog = new FormEditorDialog(self, {
                    title: 'Field Parameters',
                    size: 'medium',
                    $content: '<form role="form">' + fieldSelection + '</form>',
                }).open();
                dialog.on('save', this, ev => {
                    ev.stopPropagation();
                    save.call(dialog);
                });
            });
        },

        // Create a custom field
        website_form_custom_field: function (previewMode, value, $li) {
            var default_field_name = 'Custom ' + $li.text();
            this.append_field({
                name: default_field_name,
                string: default_field_name,
                custom: true,
                type: value,
                // Default values for x2many fields
                records: [
                    {
                        id: 'Option 1',
                        display_name: _t('Option 1')
                    },
                    {
                        id: 'Option 2',
                        display_name: _t('Option 2')
                    },
                    {
                        id: 'Option 3',
                        display_name: _t('Option 3')
                    }
                ],
                // Default values for selection fields
                selection: [
                    [
                        'Option 1',
                        _t('Option 1')
                    ],
                    [
                        'Option 2',
                        _t('Option 2')
                    ],
                    [
                        'Option 3',
                        _t('Option 3')
                    ],
                ]
            });
        },

        // Re-render the field and replace the current one
        // website_form_editor_field_reset: function(previewMode, value, $li) {
        //     var self = this;
        //     var target_field_name = this.$target.find('.col-form-label').attr('for');
        //     this.fields().then(function(fields){
        //         self.render_field(fields[target_field_name]).done(function(field){
        //             self.$target.replaceWith(field);
        //         })
        //     });
        // },

        append_field: function (field) {
            var self = this;
            this.render_field(field).then(function (field){
                self.$target.find(".form-group:has('.o_website_form_send')").before(field);
            });
        },

        render_field: function (field) {
            // Convert the required boolean to a value directly usable
            // in qweb js to avoid duplicating this in the templates
            field.required = field.required ? 1 : null;

            // Fetch possible values for relation fields
            var fieldRelationProm;
            if (field.relation && field.relation !== 'ir.attachment') {
                fieldRelationProm = this._rpc({
                    model: field.relation,
                    method: 'search_read',
                    args: [
                        field.domain || [],
                        ['display_name']
                    ],
                }).then(function (records) {
                    field.records = records;
                });
            }

            return Promise.resolve(fieldRelationProm).then(function () {
                var $content = $(qweb.render("website_form.field_" + field.type, {field: field}));
                $content.find('label:not(:has(span)), label span').addClass('o_fake_editable').attr('contentEditable', true);
                return $content;
            });
        },

        onBuilt: function () {
            // Open the parameters modal on snippet drop
            this.website_form_model_modal('click', null, null);
        },

        /**
         * Hide change form parameters option for forms
         * e.g. User should not be enable to change existing job application form to opportunity form in 'Apply job' page.
         *
         * @override
         */
        onFocus: function () {
            this.$el.filter('[data-website_form_model_modal]').toggleClass('d-none', this.$target.attr('hide-change-model') !== undefined);
        },

        init_form: function () {
            var self = this;
            var modelName = this.activeForm.model;
            var formKey = this.activeForm.website_form_key;
            if (modelName !== this.$target.attr('data-model_name')) {
                this.$target.attr('data-model_name', modelName);
                this.$target.find(".form-field:not(:has('.o_website_form_send')), .o_form_heading").remove();

                if (formKey) {
                    var formInfo = FormEditorRegistry.get(formKey);
                    ajax.loadXML(formInfo.defaultTemplatePath, qweb).then(function () {
                        // Append form title
                        $('<h1>', {
                            class: 'o_form_heading',
                            text: self.activeForm.website_form_label,
                            contentEditable: true,
                        }).prependTo(self.$target.find('.container'));
                        self.$target.find('.form-group:has(".o_website_form_send")').before($(qweb.render(formInfo.defaultTemplateName)));
                    });
                } else {
                    // Force fetch the fields of the new model
                    // and render all model required fields
                    this.fetch_model_fields().then(function (fields) {
                        _.each(fields, function (field, field_name){
                            if (field.required) {
                                self.append_field(field);
                            }
                        });
                    });
                }
            }
        },

        cleanForSave: function () {
            var model = this.$target.data('model_name');
            // because apparently this can be called on the wrong widget and
            // we may not have a model, or fields...
            if (model) {
                // we may be re-whitelisting already whitelisted fields. Doesn't
                // really matter.
                var fields = this.$target.find('input.form-field[name=email_to], .form-field:not(.o_website_form_custom) :input').map(function (_, node) {
                    return node.getAttribute('name');
                }).get();
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

            // Prevent saving of the error colors  // TODO: would be better on Edit
            this.$target.find('.o_has_error').removeClass('o_has_error').find('.form-control, .custom-select').removeClass('is-invalid');

            // Prevent saving of the status message  // TODO: would be better on Edit
            this.$target.find('#o_website_form_result').empty();

            // Prevent saving disabled state of send button  // TODO: would be better on Edit
            this.$target.find('.o_website_form_send').removeClass('disabled').removeAttr('disabled')

            // Update values of custom inputs to mirror their labels
            var custom_inputs = this.$target.find('.o_website_form_custom .o_website_form_input');
            _.each(custom_inputs, function (input, index) {
                // Change the custom field name according to their label
                var field_label = $(input).closest('.form-field').find('label:first');
                input.name = field_label.text().trim();
                field_label.attr('for', input.name);

                // Change the custom radio or checkboxes values according to their label
                if (input.type === 'radio' || input.type === 'checkbox') {
                    var checkbox_label = $(input).closest('label').text().trim();
                    if (checkbox_label) {
                        input.value = checkbox_label;
                    }
                }
            });

            // Clear default values coming from data-for/data-values attributes
            this.$target.find('input[name],textarea[name]').each(function () {
                var original = $(this).data('website_form_original_default_value');
                if (original !== undefined && $(this).val() === original) {
                    $(this).val('').removeAttr('value');
                }
            });
        }
    });

    // Generic custom field options
    options.registry['website_form_editor_field'] = options.Class.extend({
        xmlDependencies: ['/website_form/static/src/xml/website_form_editor.xml'],

        // Option to toggle inputs required attribute
        website_form_field_require: function (previewMode, value, $li) {
            this.$target.find('.o_website_form_input').each(function (index, input) {
                input.required = !input.required;
            });
        }
    });

    // Dirty hack to transform select fields into an editable construct
    options.registry['website_form_editor_field_select'] = options.Class.extend({
        xmlDependencies: ['/website_form/static/src/xml/website_form_editor.xml'],

        start: function () {
            if (!this.$target.find('#editable_select').length) {
                var self = this;
                var select = this.$target.find('select');
                select.hide();
                this.editable_select = $('<div id="editable_select" class="form-control o_website_form_input" contenteditable="true"/>');
                _.each(select.children(), function (option) {
                    self.editable_select.append(
                        $('<div id="' + $(option).attr('value') + '" class="o_website_form_select_item">' + $(option).text().trim() + '</div>')
                    );
                });
                select.after(this.editable_select);
            }
            return this._super.apply(this, arguments);
        },

        cleanForSave: function () {
            if (this.$target.find('#editable_select').length) {
                var self = this;
                // Reconstruct the field from the select tag
                var select = this.$target.find('select');
                var field = {
                    name: select.attr('name'),
                    string: this.$target.find('.col-form-label').text().trim(),
                    required: self.$target.hasClass('o_website_form_required'),
                    custom: self.$target.hasClass('o_website_form_custom'),
                };

                // Build the new records list from the editable select field
                var records = [];
                var editable_options = this.$target.find('#editable_select .o_website_form_select_item');
                _.each(editable_options, function (option) {
                    records.push({
                        id: self.$target.hasClass('o_website_form_custom') ? $(option).text().trim() : $(option).attr('id'),
                        display_name: $(option).text().trim()
                    });
                });
                field.records = records;

                // Replace this field by the new one
                var $new_select = $(qweb.render("website_form.field_many2one", {field: field}));
                // Reapply the custom style classes
                if (this.$target.hasClass('o_website_form_required_custom')) {
                    $new_select.addClass('o_website_form_required_custom');
                }
                if (this.$target.hasClass('o_website_form_field_hidden')) {
                    $new_select.addClass('o_website_form_field_hidden');
                }
                var labelClasses = this.$target.find('> div:first').attr('class');
                var inputClasses = this.$target.find('> div:last').attr('class');
                $new_select.find('> div:first').attr('class', labelClasses);
                $new_select.find('> div:last').attr('class', inputClasses);
                this.$target.replaceWith($new_select);
            }
        }
    });

    // allow breaking of form select items, to create new ones
    Wysiwyg.include({
        /**
         * @override
         */
        _editorOptions: function () {
            var options = this._super.apply(this, arguments);
            var isUnbreakableNode = options.isUnbreakableNode;
            options.isUnbreakableNode = function (node) {
                var isSelItem = $(node).hasClass('o_website_form_select_item');
                return isUnbreakableNode(node) && !isSelItem;
            };
            return options;
        },
    });

    // Superclass for options that need to disable a button from the snippet overlay
    var disable_overlay_button_option = options.Class.extend({
        xmlDependencies: ['/website_form/static/src/xml/website_form_editor.xml'],

        // Disable a button of the snippet overlay
        disable_button: function (button_name, message) {
            // TODO refactor in master
            var className = 'oe_snippet_' + button_name;
            this.$overlay.add(this.$overlay.data('$optionsSection')).on('click', '.' + className, this.prevent_button);
            var $button = this.$overlay.add(this.$overlay.data('$optionsSection')).find('.' + className);
            $button.attr('title', message).tooltip({delay: 0});
            $button.removeClass(className); // Disable the functionnality
        },

        prevent_button: function (event) {
            // Snippet options bind their functions before the editor, so we
            // can't cleanly unbind the editor onRemove function from here
            event.preventDefault();
            event.stopImmediatePropagation();
        }
    });

    // Disable duplicate button for model fields
    options.registry['website_form_editor_field_model'] = disable_overlay_button_option.extend({
        start: function () {
            this.disable_button('clone', _t('You can\'t duplicate a model field.'));
            return this._super.apply(this, arguments);
        }
    });

    // Disable delete button for model required fields
    options.registry['website_form_editor_field_required'] = disable_overlay_button_option.extend({
        start: function () {
            this.disable_button('remove', _t('You can\'t remove a field that is required by the model itself.'));
            return this._super.apply(this, arguments);
        }
    });

    // Disable duplicate button for non-custom checkboxes and radio buttons
    options.registry['website_form_editor_field_x2many'] =disable_overlay_button_option.extend({
        start: function () {
            this.disable_button('clone', _t('You can\'t duplicate an item which refers to an actual record.'));
            return this._super.apply(this, arguments);
        }
    });
});

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
    defaultTemplateName: 'website_form.default_contactus_form',
    defaultTemplatePath: '/website_form/static/src/xml/website_form.xml',
    fields: [{
        name: 'email_to',
        type: 'char',
        required: true,
        string: _t('Recipient Email'),
    }],
});

});

```

## File: static\src\xml\website_form.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <!-- Success status -->
    <t t-name="website_form.status_success">
        <span id="o_website_form_result" class="text-success ml8">
            <i class="fa fa-check mr4" role="img" aria-label="Success" title="Success"/>The form has been sent successfully.
        </span>
    </t>

    <!-- Missing field status -->
    <t t-name="website_form.status_invalid">
        <span id="o_website_form_result" class="text-danger ml8">
            <i class="fa fa-close mr4" role="img" aria-label="Error" title="Error"/>Please fill in the form correctly.
        </span>
    </t>

    <!-- Error status -->
    <t t-name="website_form.status_error">
        <span id="o_website_form_result" class="text-danger ml8">
            <i class="fa fa-close mr4" role="img" aria-label="Error" title="Error"/>An error has occured, the form has not been sent.
        </span>
    </t>

    <t t-name="website_form.default_contactus_form">
        <div class="form-group row form-field o_website_form_custom o_website_form_required_custom">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="Name">Your Name</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <input type="text" class="form-control o_website_form_input" name="Name" required=""/>
            </div>
        </div>
        <div class="form-group row form-field o_website_form_custom">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="Phone">Phone Number</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <input type="text" class="form-control o_website_form_input" name="Phone"/>
            </div>
        </div>
        <div class="form-group row form-field o_website_form_required_custom">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="email_from">Email</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <input type="email" class="form-control o_website_form_input" name="email_from" required=""/>
            </div>
        </div>
        <div class="form-group row form-field o_website_form_custom">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="Partner Name">Your Company</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <input type="text" class="form-control o_website_form_input" name="Partner Name"/>
            </div>
        </div>
        <div class="form-group row form-field o_website_form_required_custom">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="subject">Subject</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <input type="text" class="form-control o_website_form_input" name="subject" required=""/>
            </div>
        </div>
        <div class="form-group row form-field o_website_form_custom o_website_form_required_custom">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" for="Description">Your Question</label>
            </div>
            <div class="col-lg-7 col-md-8">
                <textarea class="form-control o_website_form_input" name="Description" required=""></textarea>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\xml\website_form_editor.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <!-- Generic Field Layout -->
    <t t-name="website_form.field">
        <div t-att-class="'row form-group form-field' + (field.custom ? ' o_website_form_custom' : '') + (field.required ? ' o_website_form_required' : '')">
            <div class="col-lg-3 col-md-4">
                <label class="col-form-label" t-att-for="field.name">
                    <t t-esc="field.string"/>
                </label>
            </div>
            <div class="col-lg-7 col-md-8">
                <t t-raw="0"/>
            </div>
        </div>
    </t>

    <!-- Char Field -->
    <t t-name="website_form.field_char">
        <t t-call="website_form.field">
            <input
                type="text"
                class="form-control o_website_form_input"
                t-att-name="field.name"
                t-att-required="field.required"
                t-att-value="field.value"
            />
        </t>
    </t>

    <!-- Text Field -->
    <t t-name="website_form.field_text">
        <t t-call="website_form.field">
            <textarea
                class="form-control o_website_form_input"
                t-att-name="field.name"
                t-att-required="field.required"
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
                class="form-control o_website_form_input"
                t-att-name="field.name"
                step="1"
                t-att-required="field.required"
            />
        </t>
    </t>

    <!-- Float Field -->
    <t t-name="website_form.field_float">
        <t t-call="website_form.field">
            <input
                type="number"
                class="form-control o_website_form_input"
                t-att-name="field.name"
                step="any"
                t-att-required="field.required"
            />
        </t>
    </t>

    <!-- Date Field -->
    <t t-name="website_form.field_date">
        <t t-call="website_form.field">
            <t t-set="datepickerID" t-value="'datepicker' + Math.random().toString().substring(2)"/>
            <div class="o_website_form_date input-group date" t-att-id="datepickerID" data-target-input="nearest">
                <input
                        type="text"
                        class="form-control datetimepicker-input o_website_form_input"
                        t-attf-data-target="##{datepickerID}"
                        t-att-name="field.name"
                        t-att-required="field.required"
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
            <div class="o_website_form_datetime input-group date" t-att-id="datetimepickerID" data-target-input="nearest">
                <input
                        type="text"
                        class="form-control datetimepicker-input o_website_form_input"
                        t-attf-data-target="##{datetimepickerID}"
                        t-att-name="field.name"
                        t-att-required="field.required"
                />
                <div class="input-group-append" t-attf-data-target="##{datetimepickerID}" data-toggle="datetimepicker">
                    <div class="input-group-text"><i class="fa fa-calendar"></i></div>
                </div>
            </div>
        </t>
    </t>

    <!-- Boolean Field -->
    <t t-name="website_form.field_boolean">
        <t t-call="website_form.field">
            <input
                type="checkbox"
                value="Yes"
                class="o_website_form_input"
                t-att-name="field.name"
                t-att-required="field.required"
            />
        </t>
    </t>

    <!-- Selection Field -->
    <t t-name="website_form.field_selection">
        <t t-call="website_form.field">
            <div class="o_website_form_flex">
                <t t-foreach="field.selection" t-as="option">
                    <div class="radio o_website_form_flex_item">
                        <label>
                            <input
                                type="radio"
                                class="o_website_form_input"
                                t-att-name="field.name"
                                t-att-value="option[0]"
                                t-att-required="field.required"
                            />
                            <span>
                                <t t-esc="option[1]"/>
                            </span>
                        </label>
                    </div>
                </t>
            </div>
        </t>
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
                <select class="form-control o_website_form_input" t-att-name="field.name" t-att-required="field.required">
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
            <t t-call="website_form.field">
                <div class="o_website_form_flex">
                    <t t-if="!field.records">
                        <input
                            class="o_website_form_input"
                            t-att-name="field.name"
                            t-att-value="record.id"
                            t-att-required="field.required"
                            placeholder="No matching record !"
                        />
                    </t>
                    <t t-foreach="field.records" t-as="record">
                        <div class="checkbox o_website_form_flex_item">
                            <label>
                                <input
                                    type="checkbox"
                                    class="o_website_form_input"
                                    t-att-name="field.name"
                                    t-att-value="record.id"
                                    t-att-required="field.required"
                                />
                                <span>
                                    <t t-esc="record.display_name"/>
                                </span>
                            </label>
                        </div>
                    </t>
                </div>
            </t>
        </t>
    </t>

    <!-- Many2Many Field -->
    <t t-name="website_form.field_many2many">
        <t t-call="website_form.field_one2many"/>
    </t>

    <!-- Binary Field -->
    <t t-name="website_form.field_binary">
        <t t-call="website_form.field">
            <input
                type="file"
                class="form-control o_website_form_input"
                t-att-name="field.name"
                t-att-required="field.required"
                t-att-multiple="multiple"
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

        <template id="assets_frontend" name="Website Form Assets Frontend" inherit_id="website.assets_frontend">
            <xpath expr="." position="inside">
                <link rel="stylesheet" type="text/scss" href="/website_form/static/src/scss/website_form.scss"/>
                <script type="text/javascript" src="/website_form/static/src/js/website_form.js"/>
            </xpath>
        </template>

        <template id="assets_editor" name="Website Form Editor Assets Editor" inherit_id="website.assets_editor">
            <xpath expr="." position="inside">
                <script type="text/javascript" src="/website_form/static/src/js/website_form_editor.js"/>
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
                <page string="Website Forms">
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

## File: views\res_config_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="website.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='google_analytics_dashboard_setting']" position="before">
                <div class="col-12 col-lg-6 o_setting_box" groups="base.group_no_one">
                    <div class="o_setting_left_pane">
                        <field name="website_form_enable_metadata"/>
                    </div>
                    <div class="o_setting_right_pane">
                        <label for="website_form_enable_metadata"/>
                        <span class="fa fa-lg fa-globe" title="Values set here are website-specific." groups="website.group_multi_website"/>
                        <div class="text-muted">
                            Track metadata (IP, User Agent, ...) on your Website Forms
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <template id="s_website_form" name="Form Builder">
            <form action="/website_form/" method="post" class="s_website_form container-fluid mt32" enctype="multipart/form-data">
                <div class="container">
                    <div class="form-group row">
                        <div class="offset-lg-3 offset-md-4 col-md-8 col-lg-7">
                            <button class="btn btn-primary btn-lg o_website_form_send">Send</button>
                            <span id="o_website_form_result"></span>
                        </div>
                    </div>
                </div>
            </form>
        </template>

        <template id="remove_external_snippets" inherit_id="website.external_snippets">
            <xpath expr="//t[@t-install='website_form']" position="replace"/>
        </template>

        <template id="register_s_website_form" inherit_id="website.snippets" name="Snippet Form Builder">
            <xpath expr="//div[@id='snippet_feature']//t[@t-snippet][last()]" position="after">
                <!-- This snippet cannot be used in sanitized fields -->
                <!-- because it contains inputs that would be removed -->
                <t t-snippet="website_form.s_website_form" t-thumbnail="/website/static/src/img/s_website_form.png" t-forbid-sanitize="true"/>
            </xpath>
        </template>

        <template id="snippet_options" inherit_id="website.snippet_options">
            <xpath expr="//*[@id='so_snippet_addition']" position="attributes">
                <attribute name="data-selector" add=".s_website_form" separator=","/>
            </xpath>
            <xpath expr="//div" position="after">
              <!-- Form -->
              <div data-js="website_form_editor" data-selector=".s_website_form">
                  <we-button data-website_form_model_modal="" data-no-preview="true">Change Form Parameters</we-button>
                  <we-button data-website_form_field_modal="" data-no-preview="true">Add an existing field</we-button>
                  <we-collapse-area>
                      <we-toggler>Add a custom field</we-toggler>
                      <we-collapse data-no-preview="true">
                          <we-button data-website_form_custom_field="char">Text</we-button>
                          <we-button data-website_form_custom_field="text">Long Text</we-button>
                          <we-button data-website_form_custom_field="integer">Number</we-button>
                          <we-button data-website_form_custom_field="float">Decimal Number</we-button>
                          <we-button data-website_form_custom_field="boolean">Checkbox</we-button>
                          <we-button data-website_form_custom_field="selection">Radio Buttons</we-button>
                          <we-button data-website_form_custom_field="many2one">Selection</we-button>
                          <we-button data-website_form_custom_field="one2many">Multiple Checkboxes</we-button>
                          <we-button data-website_form_custom_field="date">Date</we-button>
                          <we-button data-website_form_custom_field="datetime">Date &amp; Time</we-button>
                          <we-button data-website_form_custom_field="binary">File Upload</we-button>
                      </we-collapse>
                  </we-collapse-area>
              </div>

              <!-- Field -->
              <div data-js='website_form_editor' data-selector=".form-field" data-drop-near=".form-field">
                    <we-button data-toggle-class="o_website_form_field_hidden" data-no-preview="true">Hidden</we-button>
              </div>

              <!-- Add move, duplicate and remove controllers to checkboxes and radio buttons -->
              <div data-selector=".o_website_form_flex_item" data-drop-near=".o_website_form_flex_item"/>

              <!-- Add move and remove controllers to select items -->
              <div data-selector=".s_website_form .form-field.o_website_form_custom .o_website_form_select_item" data-drop-near=".o_website_form_select_item"/>

              <!-- Required option for fields that are not required fields of the model -->
              <div data-js='website_form_editor_field' data-selector=".form-field:not(.o_website_form_required)">
                    <we-button data-website_form_field_require="" data-toggle-class="o_website_form_required_custom" data-no-preview="true">Required</we-button>
              </div>

              <!-- Remove the duplicate options of model fields -->
              <div data-js="website_form_editor_field_model" data-selector=".s_website_form .form-field:not(.o_website_form_custom)"/>

              <!-- Remove the delete options of model required fields -->
              <div data-js="website_form_editor_field_required" data-selector=".s_website_form .o_website_form_required"/>

              <!-- Remove the duplicate options of radio and checkboxes of model fields -->
              <div data-js="website_form_editor_field_x2many" data-selector=".s_website_form .form-field:not(.o_website_form_custom) .o_website_form_flex_item"/>

              <!-- Transform the select inputs into editable constructs -->
              <div data-js="website_form_editor_field_select" data-selector=".s_website_form .form-field:has(select)"/>

              <!-- Remove the duplicate option of model select items -->
              <div data-js="website_form_editor_field_x2many" data-selector=".s_website_form .form-field:not(.o_website_form_custom) .o_website_form_select_item" data-drop-near=".o_website_form_select_item"/>

              <!-- Remove the delete options of the Submit button -->
              <div data-js="website_form_editor_field_required" data-selector=".s_website_form .o_website_form_send"/>
            </xpath>
        </template>
</odoo>

```

## File: views\website_form_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="contactus_form" name="Contact Form" inherit_id="website.contactus" customize_show="True">
        <xpath expr="//div[@name='mail_button']" position="replace">
            <form id="contactus_form" t-ignore="true" action="/website_form/" method="post" data-model_name="mail.mail" class="s_website_form container-fluid mt32" enctype="multipart/form-data" data-success_page="/contactus-thank-you">
                <div class="form-group row form-field o_website_form_custom o_website_form_required_custom">
                    <div class="col-lg-3 col-md-4">
                        <label class="col-form-label" for="Name">Your Name</label>
                    </div>
                    <div class="col-lg-7 col-md-8">
                        <input type="text" class="form-control o_website_form_input" name="Name" required=""/>
                    </div>
                </div>
                <div class="form-group row form-field o_website_form_custom">
                    <div class="col-lg-3 col-md-4">
                        <label class="col-form-label" for="Phone">Phone Number</label>
                    </div>
                    <div class="col-lg-7 col-md-8">
                        <input type="text" class="form-control o_website_form_input" name="Phone"/>
                    </div>
                </div>
                <div class="form-group row form-field o_website_form_required_custom">
                    <div class="col-lg-3 col-md-4">
                        <label class="col-form-label" for="email_from">Email</label>
                    </div>
                    <div class="col-lg-7 col-md-8">
                        <input type="email" class="form-control o_website_form_input" name="email_from" required=""/>
                    </div>
                </div>
                <div class="form-group row form-field o_website_form_custom">
                    <div class="col-lg-3 col-md-4">
                        <label class="col-form-label" for="Partner Name">Your Company</label>
                    </div>
                    <div class="col-lg-7 col-md-8">
                        <input type="text" class="form-control o_website_form_input" name="Partner Name"/>
                    </div>
                </div>
                <div class="form-group row form-field o_website_form_required_custom">
                    <div class="col-lg-3 col-md-4">
                        <label class="col-form-label" for="subject">Subject</label>
                    </div>
                    <div class="col-lg-7 col-md-8">
                        <input type="text" class="form-control o_website_form_input" name="subject" required=""/>
                    </div>
                </div>
                <div class="form-group row form-field o_website_form_custom o_website_form_required_custom">
                    <div class="col-lg-3 col-md-4">
                        <label class="col-form-label" for="Description">Your Question</label>
                    </div>
                    <div class="col-lg-7 col-md-8">
                        <textarea class="form-control o_website_form_input" name="Description" required=""></textarea>
                    </div>
                </div>
                <div class="form-group row form-field d-none">
                    <div class="col-lg-3 col-md-4">
                        <label class="col-form-label" for="email_to">Email To</label>
                    </div>
                    <div class="col-lg-7 col-md-8">
                        <input type="hidden" class="form-control o_website_form_input" name="email_to" t-att-value="res_company.email"/>
                    </div>
                </div>
                <div class="form-group row">
                    <div class="offset-lg-3 offset-md-4 col-md-8 col-lg-7">
                        <a href="#" class="btn btn-primary btn-lg o_website_form_send">Send</a>
                        <span id="o_website_form_result"></span>
                    </div>
                </div>
            </form>
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
                    <div class="container pt-3">
                        <h1>Thanks!</h1>
                        <div class="row">
                            <div class="col-lg-8">
                                <div class="alert alert-success" role="status">
                                    Your message has been sent successfully.
                                    <button type="button" class="close" data-dismiss="alert">&amp;times;</button>
                                </div>
                                <p>
                                    We will get back to you shortly.
                                </p>
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

