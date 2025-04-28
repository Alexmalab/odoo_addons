# Odoo Module: base_import

Category: Hidden/Tools

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Base import',
    'description': """
New extensible file import for Odoo
======================================

Re-implement Odoo's file import system:

* Server side, the previous system forces most of the logic into the
  client which duplicates the effort (between clients), makes the
  import system much harder to use without a client (direct RPC or
  other forms of automation) and makes knowledge about the
  import/export system much harder to gather as it is spread over
  3+ different projects.

* In a more extensible manner, so users and partners can build their
  own front-end to import from other file formats (e.g. OpenDocument
  files) which may be simpler to handle in their work flow or from
  their data production sources.

* In a module, so that administrators and users of Odoo who do not
  need or want an online import can avoid it being available to users.
""",
    'depends': ['web'],
    'version': '2.0',
    'category': 'Hidden/Tools',
    'installable': True,
    'auto_install': True,
    'data': [
        'security/ir.model.access.csv',
    ],
    'assets': {
        'web.assets_qweb': [
            'base_import/static/src/**/*.xml',
        ],
        'web.assets_backend': [
            'base_import/static/lib/javascript-state-machine/state-machine.js',
            'base_import/static/src/**/*.scss',
            'base_import/static/src/**/*.js',
        ],
        'web.qunit_suite_tests': [
            'base_import/static/tests/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import http
from odoo.http import request
from odoo.tools import misc


class ImportController(http.Controller):

    @http.route('/base_import/set_file', methods=['POST'])
    def set_file(self, file, import_id, jsonp='callback'):
        import_id = int(import_id)

        written = request.env['base_import.import'].browse(import_id).write({
            'file': file.read(),
            'file_name': file.filename,
            'file_type': file.content_type,
        })

        return 'window.top.%s(%s)' % (misc.html_escape(jsonp), json.dumps({'result': written}))

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: models\base_import.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import binascii
import codecs
import collections
import difflib
import unicodedata

import chardet
import datetime
import io
import itertools
import logging
import psycopg2
import operator
import os
import re
import requests

from PIL import Image

from odoo import api, fields, models
from odoo.tools.translate import _
from odoo.tools.mimetypes import guess_mimetype
from odoo.tools import config, DEFAULT_SERVER_DATE_FORMAT, DEFAULT_SERVER_DATETIME_FORMAT, pycompat, parse_version

FIELDS_RECURSION_LIMIT = 3
ERROR_PREVIEW_BYTES = 200
DEFAULT_IMAGE_TIMEOUT = 3
DEFAULT_IMAGE_MAXBYTES = 10 * 1024 * 1024
DEFAULT_IMAGE_REGEX = r"^(?:http|https)://"
DEFAULT_IMAGE_CHUNK_SIZE = 32768
IMAGE_FIELDS = ["icon", "image", "logo", "picture"]
_logger = logging.getLogger(__name__)
BOM_MAP = {
    'utf-16le': codecs.BOM_UTF16_LE,
    'utf-16be': codecs.BOM_UTF16_BE,
    'utf-32le': codecs.BOM_UTF32_LE,
    'utf-32be': codecs.BOM_UTF32_BE,
}

try:
    import xlrd
    try:
        from xlrd import xlsx
    except ImportError:
        xlsx = None
except ImportError:
    xlrd = xlsx = None

try:
    from . import odf_ods_reader
except ImportError:
    odf_ods_reader = None

try:
    os.environ['OPENPYXL_DEFUSEDXML'] = 'False'
    from openpyxl import load_workbook
except ImportError:
    load_workbook = None


FILE_TYPE_DICT = {
    'text/csv': ('csv', True, None),
    'application/vnd.ms-excel': ('xls', xlrd, 'xlrd'),
    'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet': (
        'xlsx',
        load_workbook or xlsx,
        # if xlrd 2.x then xlsx is not available, so don't suggest it
        'openpyxl' if xlrd and parse_version(xlrd.__VERSION__) >= parse_version("2.0") else 'openpyxl or xlrd >= 1.0.0 < 2.0',
    ),
    'application/vnd.oasis.opendocument.spreadsheet': ('ods', odf_ods_reader, 'odfpy')
}
EXTENSIONS = {
    '.' + ext: handler
    for mime, (ext, handler, req) in FILE_TYPE_DICT.items()
}


class ImportValidationError(Exception):
    """
    This class is made to correctly format all the different error types that
    can occur during the pre-validation of the import that is made before
    calling the data loading itself. The Error data structure is meant to copy
    the one of the errors raised during the data loading. It simplifies the
    error management at client side as all errors can be treated the same way.

    This exception is typically raised when there is an error during data
    parsing (image, int, dates, etc..) or if the user did not select at least
    one field to map with a column.
    """
    def __init__(self, message, **kwargs):
        super().__init__(message)
        self.type = kwargs.get('error_type', 'error')
        self.message = message
        self.record = False
        self.not_matching_error = True
        self.field_path = [kwargs['field']] if kwargs.get('field') else False
        self.field_type = kwargs.get('field_type')


class Base(models.AbstractModel):
    _inherit = 'base'

    @api.model
    def get_import_templates(self):
        """
        Get the import templates label and path.

        :return: a list(dict) containing label and template path
                 like ``[{'label': 'foo', 'template': 'path'}]``
        """
        return []

class ImportMapping(models.Model):
    """ mapping of previous column:field selections

    This is useful when repeatedly importing from a third-party
    system: column names generated by the external system may
    not match Odoo's field names or labels. This model is used
    to save the mapping between column names and fields so that
    next time a user imports from the same third-party systems
    we can automatically match the columns to the correct field
    without them having to re-enter the mapping every single
    time.
    """
    _name = 'base_import.mapping'
    _description = 'Base Import Mapping'

    res_model = fields.Char(index=True)
    column_name = fields.Char()
    field_name = fields.Char()


class ResUsers(models.Model):
    _inherit = 'res.users'

    def _can_import_remote_urls(self):
        """ Hook to decide whether the current user is allowed to import
        images via URL (as such an import can DOS a worker). By default,
        allows the administrator group.

        :rtype: bool
        """
        self.ensure_one()
        return self._is_admin()

class Import(models.TransientModel):
    """
    This model is used to prepare the loading of data coming from a user file.

    Here is the process that is followed:

    #. The user selects a file to import.
    #. File parsing and mapping suggestion (see "parse_preview" method)

       #. Extract the current model's importable fields tree (see :meth:`get_fields_tree`).
       #. Read the file (see :meth:`_read_file`) and extract header names and file
          length (used for batch import).
       #. Extract headers types from the data preview (10 first line of the file)
          (see :meth:`_extract_headers_types`).
       #. Try to find for each header a field to map with (see :meth:`_get_mapping_suggestions`)

          - First check the previously saved mappings between the header name
            and one of the model's fields.
          - If no mapping found, try an exact match comparison using fields
            technical names, labels and user language translated labels.
          - If nothing found, try a fuzzy match using word distance between
            header name and fields tachnical names, labels and user language
            translated labels. Keep only the closest match.

       #. Prepare examples for each columns using the first non null value from each column.
       #. Send the info back to the UI where the user can modify the suggested mapping.
    #. Execute the import: There are two import mode with uses the same process. (see :meth:`execute_import`)

       #. Test import: Try to import but rollback the transaction. This allows
          the check errors during the import process and allow the user to
          choose import options for the different encountered errors.
       #. Real import: Try to import the file using the configured mapping and
          the eventual "error mapping options". If import encounters blocking
          errors, the transaction is rollbacked and the user is allowed to
          choose import options for the different errors.

          - Get file data and fields to import into (see :meth:`_convert_import_data`).
          - Parse date, float and binary data (see :meth:`_parse_import_data`).
          - Handle multiple mapping -> concatenate char/text/many2many columns
            mapped on the same field (see :meth:`_handle_multi_mapping`).
          - Handle fallback values for boolean and selection fields, in case
            input data does not match any allowed values (see :meth:`_handle_fallback_values`).
          - Load data (see ir.model "load" method).
          - Rollback transaction if test mode or if encountered error.
          - Save mapping if any import is successful to ease later mapping suggestions.
          - Return import result to the UI (success or errors if any).
    """

    _name = 'base_import.import'
    _description = 'Base Import'

    # allow imports to survive for 12h in case user is slow
    _transient_max_hours = 12.0
    # we consider that if the difference is more than 0.2, then the two compared strings are "too different" to propose
    # any match between them. (see '_get_mapping_suggestion' for more details)
    FUZZY_MATCH_DISTANCE = 0.2

    res_model = fields.Char('Model')
    file = fields.Binary('File', help="File to check and/or import, raw binary (not base64)", attachment=False)
    file_name = fields.Char('File Name')
    file_type = fields.Char('File Type')

    @api.model
    def get_fields_tree(self, model, depth=FIELDS_RECURSION_LIMIT):
        """ Recursively get fields for the provided model (through
        fields_get) and filter them according to importability

        The output format is a list of :class:`Field`:

        .. class:: Field

            .. attribute:: id: str

                A non-unique identifier for the field, used to compute
                the span of the ``required`` attribute: if multiple
                ``required`` fields have the same id, only one of them
                is necessary.

            .. attribute:: name: str

                The field's logical (Odoo) name within the scope of
                its parent.

            .. attribute:: string: str

                The field's human-readable name (``@string``)

            .. attribute:: required: bool

                Whether the field is marked as required in the
                model. Clients must provide non-empty import values
                for all required fields or the import will error out.

            .. attribute:: fields: list[Field]

                The current field's subfields. The database and
                external identifiers for m2o and m2m fields; a
                filtered and transformed fields_get for o2m fields (to
                a variable depth defined by ``depth``).

                Fields with no sub-fields will have an empty list of
                sub-fields.

            .. attribute:: model_name: str

                Used in the Odoo Field Tooltip on the import view
                and to get the model of the field of the related field(s).
                Name of the current field's model.

            .. attribute:: comodel_name: str

                Used in the Odoo Field Tooltip on the import view
                and to get the model of the field of the related field(s).
                Name of the current field's comodel, i.e. if the field is a relation field.

        Structure example for 'crm.team' model for returned importable_fields::

            [
                {'name': 'message_ids', 'string': 'Messages', 'model_name': 'crm.team', 'comodel_name': 'mail.message', 'fields': [
                    {'name': 'moderation_status', 'string': 'Moderation Status', 'model_name': 'mail.message', 'fields': []},
                    {'name': 'body', 'string': 'Contents', 'model_name': 'mail.message', 'fields' : []}
                ]},
                {{'name': 'name', 'string': 'Sales Team', 'model_name': 'crm.team', 'fields' : []}
            ]

        :param str model: name of the model to get fields form
        :param int depth: depth of recursion into o2m fields
        """
        Model = self.env[model]
        importable_fields = [{
            'id': 'id',
            'name': 'id',
            'string': _("External ID"),
            'required': False,
            'fields': [],
            'type': 'id',
        }]
        if not depth:
            return importable_fields

        model_fields = Model.fields_get()
        blacklist = models.MAGIC_COLUMNS + [Model.CONCURRENCY_CHECK_FIELD]
        for name, field in model_fields.items():
            if name in blacklist:
                continue
            # an empty string means the field is deprecated, @deprecated must
            # be absent or False to mean not-deprecated
            if field.get('deprecated', False) is not False:
                continue
            if field.get('readonly'):
                states = field.get('states')
                if not states:
                    continue
                # states = {state: [(attr, value), (attr2, value2)], state2:...}
                if not any(attr == 'readonly' and value is False
                           for attr, value in itertools.chain.from_iterable(states.values())):
                    continue
            field_value = {
                'id': name,
                'name': name,
                'string': field['string'],
                # Y U NO ALWAYS HAS REQUIRED
                'required': bool(field.get('required')),
                'fields': [],
                'type': field['type'],
                'model_name': model
            }

            if field['type'] in ('many2many', 'many2one'):
                field_value['fields'] = [
                    dict(field_value, name='id', string=_("External ID"), type='id'),
                    dict(field_value, name='.id', string=_("Database ID"), type='id'),
                ]
                field_value['comodel_name'] = field['relation']
            elif field['type'] == 'one2many':
                field_value['fields'] = self.get_fields_tree(field['relation'], depth=depth-1)
                if self.user_has_groups('base.group_no_one'):
                    field_value['fields'].append({'id': '.id', 'name': '.id', 'string': _("Database ID"), 'required': False, 'fields': [], 'type': 'id'})
                field_value['comodel_name'] = field['relation']

            importable_fields.append(field_value)

        # TODO: cache on model?
        return importable_fields

    def _filter_fields_by_types(self, model_fields_tree, header_types):
        """ Remove from model_fields_tree param all the fields and subfields
        that do not match the types in header_types

        :param: list[dict] model_fields_tree: Contains recursively all the importable fields of the target model.
                                              Generated in "get_fields_tree" method.
        :param: list header_types: Contains the extracted fields types of the current header.
                                   Generated in :meth:`_extract_header_types`.
        """
        most_likely_fields_tree = []
        for field in model_fields_tree:
            subfields = field.get('fields')
            if subfields:
                filtered_field = dict(field)  # Avoid modifying fields.
                filtered_field['fields'] = self._filter_fields_by_types(subfields, header_types)
                most_likely_fields_tree.append(filtered_field)
            elif field.get('type') in header_types:
                most_likely_fields_tree.append(field)
        return most_likely_fields_tree

    def _read_file(self, options):
        """ Dispatch to specific method to read file content, according to its mimetype or file type

        :param dict options: reading options (quoting, separator, ...)
        """
        self.ensure_one()
        # guess mimetype from file content
        mimetype = guess_mimetype(self.file or b'')
        (file_extension, handler, req) = FILE_TYPE_DICT.get(mimetype, (None, None, None))
        if handler:
            try:
                return getattr(self, '_read_' + file_extension)(options)
            except ValueError as e:
                raise e
            except ImportValidationError as e:
                raise e
            except Exception:
                _logger.warning("Failed to read file '%s' (transient id %d) using guessed mimetype %s", self.file_name or '<unknown>', self.id, mimetype)

        # try reading with user-provided mimetype
        (file_extension, handler, req) = FILE_TYPE_DICT.get(self.file_type, (None, None, None))
        if handler:
            try:
                return getattr(self, '_read_' + file_extension)(options)
            except ValueError as e:
                raise e
            except ImportValidationError as e:
                raise e
            except Exception:
                _logger.warning("Failed to read file '%s' (transient id %d) using user-provided mimetype %s", self.file_name or '<unknown>', self.id, self.file_type)

        # fallback on file extensions as mime types can be unreliable (e.g.
        # software setting incorrect mime types, or non-installed software
        # leading to browser not sending mime types)
        if self.file_name:
            p, ext = os.path.splitext(self.file_name)
            if ext in EXTENSIONS:
                try:
                    return getattr(self, '_read_' + ext[1:])(options)
                except ValueError as e:
                    raise e
                except Exception:
                    _logger.warning("Failed to read file '%s' (transient id %s) using file extension", self.file_name, self.id)

        if req:
            raise ImportError(_("Unable to load \"{extension}\" file: requires Python module \"{modname}\"").format(extension=file_extension, modname=req))
        raise ValueError(_("Unsupported file format \"{}\", import only supports CSV, ODS, XLS and XLSX").format(self.file_type))

    def _read_xls(self, options):
        book = xlrd.open_workbook(file_contents=self.file or b'')
        sheets = options['sheets'] = book.sheet_names()
        sheet = options['sheet'] = options.get('sheet') or sheets[0]
        return self._read_xls_book(book, sheet)

    def _read_xls_book(self, book, sheet_name):
        sheet = book.sheet_by_name(sheet_name)
        rows = []
        # emulate Sheet.get_rows for pre-0.9.4
        for rowx, row in enumerate(map(sheet.row, range(sheet.nrows)), 1):
            values = []
            for colx, cell in enumerate(row, 1):
                if cell.ctype is xlrd.XL_CELL_NUMBER:
                    is_float = cell.value % 1 != 0.0
                    values.append(
                        str(cell.value)
                        if is_float
                        else str(int(cell.value))
                    )
                elif cell.ctype is xlrd.XL_CELL_DATE:
                    is_datetime = cell.value % 1 != 0.0
                    # emulate xldate_as_datetime for pre-0.9.3
                    dt = datetime.datetime(*xlrd.xldate.xldate_as_tuple(cell.value, book.datemode))
                    values.append(
                        dt.strftime(DEFAULT_SERVER_DATETIME_FORMAT)
                        if is_datetime
                        else dt.strftime(DEFAULT_SERVER_DATE_FORMAT)
                    )
                elif cell.ctype is xlrd.XL_CELL_BOOLEAN:
                    values.append(u'True' if cell.value else u'False')
                elif cell.ctype is xlrd.XL_CELL_ERROR:
                    raise ValueError(
                        _("Invalid cell value at row %(row)s, column %(col)s: %(cell_value)s") % {
                            'row': rowx,
                            'col': colx,
                            'cell_value': xlrd.error_text_from_code.get(cell.value, _("unknown error code %s", cell.value))
                        }
                    )
                else:
                    values.append(cell.value)
            if any(x for x in values if x.strip()):
                rows.append(values)

        # return the file length as first value
        return sheet.nrows, rows

    # use the same method for xlsx and xls files
    def _read_xlsx(self, options):
        if xlsx:
            return self._read_xls(options)

        import openpyxl.cell.cell as types
        import openpyxl.styles.numbers as styles  # noqa: PLC0415
        book = load_workbook(io.BytesIO(self.file or b''), data_only=True)
        sheets = options['sheets'] = book.sheetnames
        sheet_name = options['sheet'] = options.get('sheet') or sheets[0]
        sheet = book[sheet_name]
        rows = []
        for rowx, row in enumerate(sheet.rows, 1):
            values = []
            for colx, cell in enumerate(row, 1):
                if cell.data_type is types.TYPE_ERROR:
                    raise ValueError(
                        _("Invalid cell value at row %(row)s, column %(col)s: %(cell_value)s", row=rowx, col=colx, cell_value=cell.value)
                    )

                if cell.value is None:
                    values.append('')
                elif isinstance(cell.value, float):
                    if cell.value % 1 == 0:
                        values.append(str(int(cell.value)))
                    else:
                        values.append(str(cell.value))
                elif cell.is_date:
                    d_fmt = styles.is_datetime(cell.number_format)
                    if d_fmt == "datetime":
                        values.append(cell.value.strftime(DEFAULT_SERVER_DATETIME_FORMAT))
                    elif d_fmt == "date":
                        values.append(cell.value.strftime(DEFAULT_SERVER_DATE_FORMAT))
                    else:
                        raise ValueError(
                        _("Invalid cell format at row %(row)s, column %(col)s: %(cell_value)s, with format: %(cell_format)s, as (%(format_type)s) formats are not supported.", row=rowx, col=colx, cell_value=cell.value, cell_format=cell.number_format, format_type=d_fmt)
                        )
                else:
                    values.append(str(cell.value))

            if any(x.strip() for x in values):
                rows.append(values)
        return sheet.max_row, rows

    def _read_ods(self, options):
        doc = odf_ods_reader.ODSReader(file=io.BytesIO(self.file or b''))
        sheets = options['sheets'] = list(doc.SHEETS.keys())
        sheet = options['sheet'] = options.get('sheet') or sheets[0]

        content = [
            row
            for row in doc.getSheet(sheet)
            if any(x for x in row if x.strip())
        ]

        # return the file length as first value
        return len(content), content

    def _read_csv(self, options):
        """ Returns file length and a CSV-parsed list of all non-empty lines in the file.

        :raises csv.Error: if an error is detected during CSV parsing
        """
        csv_data = self.file or b''
        if not csv_data:
            return ()

        encoding = options.get('encoding')
        if not encoding:
            encoding = options['encoding'] = chardet.detect(csv_data)['encoding'].lower()
            # some versions of chardet (e.g. 2.3.0 but not 3.x) will return
            # utf-(16|32)(le|be), which for python means "ignore / don't strip
            # BOM". We don't want that, so rectify the encoding to non-marked
            # IFF the guessed encoding is LE/BE and csv_data starts with a BOM
            bom = BOM_MAP.get(encoding)
            if bom and csv_data.startswith(bom):
                encoding = options['encoding'] = encoding[:-2]

        if encoding != 'utf-8':
            csv_data = csv_data.decode(encoding).encode('utf-8')

        separator = options.get('separator')
        if not separator:
            # default for unspecified separator so user gets a message about
            # having to specify it
            separator = ','
            for candidate in (',', ';', '\t', ' ', '|', unicodedata.lookup('unit separator')):
                # pass through the CSV and check if all rows are the same
                # length & at least 2-wide assume it's the correct one
                it = pycompat.csv_reader(io.BytesIO(csv_data), quotechar=options['quoting'], delimiter=candidate)
                w = None
                for row in it:
                    width = len(row)
                    if w is None:
                        w = width
                    if width == 1 or width != w:
                        break # next candidate
                else: # nobreak
                    separator = options['separator'] = candidate
                    break

        if not len(options['quoting']) == 1:
            raise ImportValidationError(_("Error while importing records: Text Delimiter should be a single character."))

        csv_iterator = pycompat.csv_reader(
            io.BytesIO(csv_data),
            quotechar=options['quoting'],
            delimiter=separator)

        content = [
            row for row in csv_iterator
            if any(x for x in row if x.strip())
        ]

        # return the file length as first value
        return len(content), content

    @api.model
    def _extract_header_types(self, preview_values, options):
        """ Returns the potential field types, based on the preview values, using heuristics.

        This methods is only used for suggested mapping at 2 levels:

        1. for fuzzy mapping at file load -> Execute the fuzzy mapping only
           on "most likely field types"
        2. For "Suggested fields" section in the fields mapping dropdown list at UI side.

        The following heuristic is used: If all preview values

        - Start with ``__export__``: return id + relational field types
        - Can be cast into integer: return id + relational field types, integer, float and monetary
        - Can be cast into Boolean: return boolean
        - Can be cast into float: return float, monetary
        - Can be cast into date/datetime: return date / datetime
        - Cannot be cast into any of the previous types: return only text based fields

        :param preview_values: list of value for the column to determine
                               see :meth:`parse_preview` for more details.
        :param options: parsing options
        """
        values = set(preview_values)
        # If all values are empty in preview than can be any field
        if values == {''}:
            return ['all']

        # If all values starts with __export__ this is probably an id
        if all(v.startswith('__export__') for v in values):
            return ['id', 'many2many', 'many2one', 'one2many']

        # If all values can be cast to int type is either id, float or monetary
        # Exception: if we only have 1 and 0, it can also be a boolean
        if all(v.isdigit() for v in values if v):
            field_type = ['integer', 'float', 'monetary']
            if {'0', '1', ''}.issuperset(values):
                field_type.append('boolean')
            return field_type

        # If all values are either True or False, type is boolean
        if all(val.lower() in ('true', 'false', 't', 'f', '') for val in preview_values):
            return ['boolean']

        # If all values can be cast to float, type is either float or monetary
        try:
            thousand_separator = decimal_separator = False
            for val in preview_values:
                val = val.strip()
                if not val:
                    continue
                # value might have the currency symbol left or right from the value
                val = self._remove_currency_symbol(val)
                if val:
                    if options.get('float_thousand_separator') and options.get('float_decimal_separator'):
                        if options['float_decimal_separator'] == '.' and val.count('.') > 1:
                            # This is not a float so exit this try
                            float('a')
                        val = val.replace(options['float_thousand_separator'], '').replace(options['float_decimal_separator'], '.')
                    # We are now sure that this is a float, but we still need to find the
                    # thousand and decimal separator
                    else:
                        if val.count('.') > 1:
                            options['float_thousand_separator'] = '.'
                            options['float_decimal_separator'] = ','
                        elif val.count(',') > 1:
                            options['float_thousand_separator'] = ','
                            options['float_decimal_separator'] = '.'
                        elif val.find('.') > val.find(','):
                            thousand_separator = ','
                            decimal_separator = '.'
                        elif val.find(',') > val.find('.'):
                            thousand_separator = '.'
                            decimal_separator = ','
                else:
                    # This is not a float so exit this try
                    float('a')
            if thousand_separator and not options.get('float_decimal_separator'):
                options['float_thousand_separator'] = thousand_separator
                options['float_decimal_separator'] = decimal_separator
            return ['float', 'monetary']  # Allow float to be mapped on a text field.
        except ValueError:
            pass

        results = self._try_match_date_time(preview_values, options)
        if results:
            return results

        # If not boolean, date/datetime, float or integer, only suggest text based fields.
        return ['text', 'char', 'binary', 'selection', 'html']

    def _try_match_date_time(self, preview_values, options):
        # Or a date/datetime if it matches the pattern
        date_patterns = [options['date_format']] if options.get(
            'date_format') else []
        user_date_format = self.env['res.lang']._lang_get(self.env.user.lang).date_format
        if user_date_format:
            try:
                to_re(user_date_format)
                date_patterns.append(user_date_format)
            except KeyError:
                pass
        date_patterns.extend(DATE_PATTERNS)
        match = check_patterns(date_patterns, preview_values)
        if match:
            options['date_format'] = match
            return ['date', 'datetime']

        datetime_patterns = [options['datetime_format']] if options.get(
            'datetime_format') else []
        datetime_patterns.extend(
            "%s %s" % (d, t)
            for d in date_patterns
            for t in TIME_PATTERNS
        )
        match = check_patterns(datetime_patterns, preview_values)
        if match:
            options['datetime_format'] = match
            return ['datetime']

        return []

    @api.model
    def _extract_headers_types(self, headers, preview, options):
        """
        For each column, this method will extract the potential data types based on the preview values

        :param list headers: list of headers names. Used as part of key for
                             returned headers_types to ease understanding of its usage
        :param list preview: list of the first file records (see "parse_preview" for more detail) e.g.::

            [ ["lead_name1", "1", "partner_id1"], ["lead_name2", "2", "partner_id2"], ... ]

        :param options: parsing options
        :returns: dict headers_types:

            contains all the extracted header types for each header e.g.::

                {
                    (header_index, header_name): ["char", "text", ...],
                    ...
                }
        """
        headers_types = {}
        for column_index, header_name in enumerate(headers):
            preview_values = [record[column_index].strip() for record in preview]
            type_field = self._extract_header_types(preview_values, options)
            headers_types[(column_index, header_name)] = type_field
        return headers_types

    def _get_mapping_suggestion(self, header, fields_tree, header_types, mapping_fields):
        """ Attempts to match a given header to a field of the imported model.

            We can distinguish 2 types of header format:

            - simple header string that aim to directly match a field of the target model
              e.g.: "lead_id" or "Opportunities" or "description".
            - composed '/' joined header string that aim to match a field of a
              relation field of the target model (= subfield) e.g.:
              'lead_id/description' aim to match the field ``description`` of the field lead_id.

            When returning result, to ease further treatments, the result is
            returned as a list, where each element of the list is a field or
            a sub-field of the preceding field.

            - ``["lead_id"]`` for simple case = simple matching
            - ``["lead_id", "description"]`` for composed case = hierarchy matching

            Mapping suggestion is found using the following heuristic:

            - first we check if there was a saved mapping by the user
            - then try to make an exact match on the field technical name /
              english label / translated label
            - finally, try the "fuzzy match": word distance between the header
              title and the field technical name / english label / translated
              label, using the lowest result. The field used for the fuzzy match
              are based on the field types we extracted from the header data
              (see :meth:`_extract_header_types`).

            For subfields, use the same logic.

            Word distance is a score between 0 and 1 to express the distance
            between two char strings where ``0`` denotes an exact match and
            ``1`` indicates completely different strings

            In order to keep only one column matched per field, we return the
            distance. That distance will be used during the deduplicate process
            (see :meth:`_deduplicate_mapping_suggestions`) and only the
            mapping with the smallest distance will be kept in case of multiple
            mapping on the same field. Note that we don't need to return the
            distance in case of hierachy mapping as we consider that as an
            advanced behaviour. The deduplicate process will ignore hierarchy
            mapping. The user will have to manually select on which field he
            wants to map what in case of mapping duplicates for sub-fields.

            :param str header: header name from the file
            :param list fields_tree: list of all the field of the target model
                Coming from :meth:`get_fields_tree`
                e.g: ``[ { 'name': 'fieldName', 'string': 'fieldLabel', fields: [ { 'name': 'subfieldName', ...} ]} , ... ]``
            :param list header_types: Extracted field types for each column in the parsed file, based on its data content.
                Coming from :meth:`_extract_header_types`
                e.g.: ``['int', 'float', 'char', 'many2one', ...]``
            :param dict mapping_fields: contains the previously saved mapping between header and field for the current model.
                E.g.: ``{ header_name: field_name }``
            :returns: if the header couldn't be matched: an empty dict
                      else: a dict with the field path and the distance between header and the matched field.
            :rtype: ``dict(field_path + Word distance)``

                    In case of simple matching: ``{'field_path': [field_name], distance: word_distance}``
                                           e.g.: ``{'field_path': ['lead_id'], distance: 0.23254}``
                    In case of hierarchy matching: ``{'field_path': [parent_field_name, child_field_name, subchild_field_name]}``
                                              e.g.: ``{'field_path': ['lead_id', 'description']}``
        """
        if not fields_tree:
            return {}

        # First, check in saved mapped fields
        mapping_field_name = mapping_fields.get(header.lower())
        if mapping_field_name and mapping_field_name:
            return {
                'field_path': [name for name in mapping_field_name.split('/')],
                'distance': -1  # Trick to force to keep that match during mapping deduplication.
            }

        if '/' not in header:
            # Then, try exact match
            IrTranslation = self.env['ir.translation']
            translated_header = IrTranslation._get_source('ir.model.fields,field_description', 'model', self.env.lang, header).lower()
            for field in fields_tree:
                # exact match found based on the field technical name
                if header.casefold() == field['name'].casefold():
                    break

                field_string = field.get('string', '').casefold()
                # match found using either user translation, either model defined field label
                if translated_header == field_string or header.casefold() == field_string:
                    break
            else:
                field = None

            if field:  # found an exact match, no need to go further
                return {
                    'field_path': [field['name']],
                    'distance': 0
                }

            # If no match found, try fuzzy match on fields filtered based on extracted header types
            # Filter out fields with types that does not match corresponding header types.
            filtered_fields = self._filter_fields_by_types(fields_tree, header_types)
            if not filtered_fields:
                return {}

            min_dist = 1
            min_dist_field = False
            for field in filtered_fields:
                field_string = field.get('string', '').casefold()

                # use string distance for fuzzy match only on most likely field types
                name_field_dist = self._get_distance(header.casefold(), field['name'].casefold())
                string_field_dist = self._get_distance(header.casefold(), field_string)
                translated_string_field_dist = self._get_distance(translated_header.casefold(), field_string)

                # Keep only the closest mapping suggestion. Note that in case of multiple mapping on the same field,
                # a mapping suggestion could be canceled by another one that has a smaller distance on the same field.
                # See 'deduplicate_mapping_suggestions' method for more info.
                current_field_dist = min([name_field_dist, string_field_dist, translated_string_field_dist])
                if current_field_dist < min_dist:
                    min_dist_field = field['name']
                    min_dist = current_field_dist

            if min_dist < self.FUZZY_MATCH_DISTANCE:
                return {
                    'field_path': [min_dist_field],
                    'distance': min_dist
                }

            return {}

        # relational field path
        field_path = []
        subfields_tree = fields_tree
        # Iteratively dive into fields tree
        for sub_header in header.split('/'):
            # Strip sub_header in case spaces are added around '/' for
            # readability of paths
            # Skip Saved mapping (mapping_field = {})
            match = self._get_mapping_suggestion(sub_header.strip(), subfields_tree, header_types, {})
            # Any match failure, exit
            if not match:
                return {}
            # prep subfields for next iteration within match['name'][0]
            field_name = match['field_path'][0]
            subfields_tree = next(item['fields'] for item in subfields_tree if item['name'] == field_name)
            field_path.append(field_name)
        # No need to return distance for hierarchy mapping
        return {'field_path': field_path}

    def _get_distance(self, a, b):
        """ This method return an index that reflects the distance between the
        two given string a and b.

        This index is a score between 0 and 1 where ``0`` indicates an exact
        match and ``1`` indicates completely different strings.
        """
        return 1 - difflib.SequenceMatcher(None, a, b).ratio()

    def _get_mapping_suggestions(self, headers, header_types, fields_tree):
        """ Attempts to match the imported model's fields to the
            titles of the parsed CSV file, if the file is supposed to have
            headers.

            Returns a dict mapping cell indices to key paths in the ``fields`` tree.

            :param list headers: titles of the parsed file
            :param dict header_types:

                extracted types for each column in the parsed file e.g.::

                    {
                        (header_index, header_name): ['int', 'float', 'char', 'many2one',...],
                         ...
                    }

            :param list fields_tree:

                list of the target model's fields e.g.::

                    [
                        {
                            'name': 'fieldName',
                            'string': 'fieldLabel',
                            'fields': [{ 'name': 'subfieldName', ...}]
                        },
                        ...
                    ]

            :rtype: dict[(int, str), {'field_path': list[str], 'distance': int}]
            :returns: mapping_suggestions e.g.:

                .. code-block:: python

                    {
                        (header_index, header_name): {
                            'field_path': ['child_id','name'],
                            'distance': 0
                        },
                        ...
                    }
        """
        mapping_suggestions = {}
        mapping_records = self.env['base_import.mapping'].search_read([('res_model', '=', self.res_model)], ['column_name', 'field_name'])
        mapping_fields = {rec['column_name']: rec['field_name'] for rec in mapping_records}
        for index, header in enumerate(headers):
            match_field = self._get_mapping_suggestion(header, fields_tree, header_types[(index, header)], mapping_fields)
            mapping_suggestions[(index, header)] = match_field or None

        self._deduplicate_mapping_suggestions(mapping_suggestions)
        return mapping_suggestions

    def _deduplicate_mapping_suggestions(self, mapping_suggestions):
        """ This method is meant to avoid multiple columns to be matched on the same field.

        Taking ``mapping_suggestions`` as input, it will check if multiple
        columns are mapped to the same field and will only keep the mapping
        that has the smallest distance. The other columns that were matched
        to the same field are removed from the mapping suggestions.

        Hierarchy mapping is considered as advanced and is skipped during this
        deduplication process. We consider that multiple mapping on hierarchy
        mapping will not occur often and due to the fact that this won't lead
        to any particular issues when a non 'char/text' field is selected more
        than once in the UI, we keep only the last selected mapping. The
        objective is to lighten the mapping suggestion process as much as we can.

        :param dict mapping_suggestions: ``{ (column_index, header_name) : { 'field_path': [header_name], 'distance': word_distance }}``
        """
        min_dist_per_field = {}
        headers_to_keep = []
        for header, suggestion in mapping_suggestions.items():
            if suggestion is None or len(suggestion['field_path']) > 1:
                headers_to_keep.append(header)
                continue

            field_name = suggestion['field_path'][0]
            field_distance = suggestion['distance']

            best_distance, _best_header = min_dist_per_field.get(field_name, (1, None))
            if field_distance < best_distance:
                min_dist_per_field[field_name] = (field_distance, header)

        headers_to_keep = headers_to_keep + [value[1] for value in min_dist_per_field.values()]
        for header in mapping_suggestions.keys() - headers_to_keep:
            del mapping_suggestions[header]

    def parse_preview(self, options, count=10):
        """ Generates a preview of the uploaded files, and performs
            fields-matching between the import's file data and the model's
            columns.

            If the headers are not requested (not options.has_headers),
            returned ``matches`` and ``headers`` are both ``False``.

            :param int count: number of preview lines to generate
            :param options: format-specific options.
                            CSV: {quoting, separator, headers}
            :type options: {str, str, str, bool}
            :returns: ``{fields, matches, headers, preview} | {error, preview}``
            :rtype: {dict(str: dict(...)), dict(int, list(str)), list(str), list(list(str))} | {str, str}
        """
        self.ensure_one()
        fields_tree = self.get_fields_tree(self.res_model)
        try:
            file_length, rows = self._read_file(options)
            if file_length <= 0:
                raise ImportValidationError(_("Import file has no content or is corrupt"))

            preview = rows[:count]

            # Get file headers
            if options.get('has_headers') and preview:
                # We need the header types before matching columns to fields
                headers = preview.pop(0)
                header_types = self._extract_headers_types(headers, preview, options)
            else:
                header_types, headers = {}, []

            # Get matches: the ones already selected by the user or propose a new matching.
            matches = {}
            # If user checked to the advanced mode, we re-parse the file but we keep the mapping "as is".
            # No need to make another mapping proposal
            if options.get('keep_matches') and options.get('fields'):
                for index, match in enumerate(options.get('fields', [])):
                    if match:
                        matches[index] = match.split('/')
            elif options.get('has_headers'):
                matches = self._get_mapping_suggestions(headers, header_types, fields_tree)
                # remove header_name for matches keys as tuples are no supported in json.
                # and remove distance from suggestion (keep only the field path) as not used at client side.
                matches = {
                    header_key[0]: suggestion['field_path']
                    for header_key, suggestion in matches.items()
                    if suggestion
                }

            # compute if we should activate advanced mode or not:
            # if was already activated of if file contains "relational fields".
            if options.get('keep_matches'):
                advanced_mode = options.get('advanced')
            else:
                # Check is label contain relational field
                has_relational_header = any(len(models.fix_import_export_id_paths(col)) > 1 for col in headers)
                # Check is matches fields have relational field
                has_relational_match = any(len(match) > 1 for field, match in matches.items() if match)
                advanced_mode = has_relational_header or has_relational_match

            # Take first non null values for each column to show preview to users.
            # Initially first non null value is displayed to the user.
            # On hover preview consists in 5 values.
            column_example = []
            for column_index, _unused in enumerate(preview[0]):
                vals = []
                for record in preview:
                    if record[column_index]:
                        vals.append("%s%s" % (record[column_index][:50], "..." if len(record[column_index]) > 50 else ""))
                    if len(vals) == 5:
                        break
                column_example.append(
                    vals or
                    [""]  # blank value if no example have been found at all for the current column
                )

            # Batch management
            batch = False
            batch_cutoff = options.get('limit')
            if batch_cutoff:
                if count > batch_cutoff:
                    batch = len(preview) > batch_cutoff
                else:
                    batch = bool(next(
                        itertools.islice(rows, batch_cutoff - count, None),
                        None
                    ))

            return {
                'fields': fields_tree,
                'matches': matches or False,
                'headers': headers or False,
                'header_types': list(header_types.values()) or False,
                'preview': column_example,
                'options': options,
                'advanced_mode': advanced_mode,
                'debug': self.user_has_groups('base.group_no_one'),
                'batch': batch,
                'file_length': file_length
            }
        except Exception as error:
            # Due to lazy generators, UnicodeDecodeError (for
            # instance) may only be raised when serializing the
            # preview to a list in the return.
            _logger.debug("Error during parsing preview", exc_info=True)
            preview = None
            if self.file_type == 'text/csv' and self.file:
                preview = self.file[:ERROR_PREVIEW_BYTES].decode('iso-8859-1')
            return {
                'error': str(error),
                # iso-8859-1 ensures decoding will always succeed,
                # even if it yields non-printable characters. This is
                # in case of UnicodeDecodeError (or csv.Error
                # compounded with UnicodeDecodeError)
                'preview': preview,
            }

    @api.model
    def _convert_import_data(self, fields, options):
        """ Extracts the input BaseModel and fields list (with
            ``False``-y placeholders for fields to *not* import) into a
            format Model.import_data can use: a fields list without holes
            and the precisely matching data matrix

            :param list(str|bool): fields
            :returns: (data, fields)
            :rtype: (list(list(str)), list(str))
            :raises ValueError: in case the import data could not be converted
        """
        # Get indices for non-empty fields
        indices = [index for index, field in enumerate(fields) if field]
        if not indices:
            raise ImportValidationError(_("You must configure at least one field to import"))
        # If only one index, itemgetter will return an atom rather
        # than a 1-tuple
        if len(indices) == 1:
            mapper = lambda row: [row[indices[0]]]
        else:
            mapper = operator.itemgetter(*indices)
        # Get only list of actually imported fields
        import_fields = [f for f in fields if f]

        _file_length, rows_to_import = self._read_file(options)
        if len(rows_to_import[0]) != len(fields):
            raise ImportValidationError(
                _("Error while importing records: all rows should be of the same size, but the title row has %d entries while the first row has %d. You may need to change the separator character.", len(fields), len(rows_to_import[0]))
            )

        if options.get('has_headers'):
            rows_to_import = rows_to_import[1:]
        data = [
            list(row) for row in map(mapper, rows_to_import)
            # don't try inserting completely empty rows (e.g. from
            # filtering out o2m fields)
            if any(row)
        ]

        # slicing needs to happen after filtering out empty rows as the
        # data offsets from load are post-filtering
        return data[options.get('skip'):], import_fields

    @api.model
    def _remove_currency_symbol(self, value):
        value = value.strip()
        negative = False
        # Careful that some countries use () for negative so replace it by - sign
        if value.startswith('(') and value.endswith(')'):
            value = value[1:-1]
            negative = True
        float_regex = re.compile(r'([+-]?[0-9.,]+)')
        split_value = [g for g in float_regex.split(value) if g]
        if len(split_value) > 2:
            # This is probably not a float
            return False
        if len(split_value) == 1:
            if float_regex.search(split_value[0]) is not None:
                return split_value[0] if not negative else '-' + split_value[0]
            return False
        else:
            # String has been split in 2, locate which index contains the float and which does not
            currency_index = 0
            if float_regex.search(split_value[0]) is not None:
                currency_index = 1
            # Check that currency exists
            currency = self.env['res.currency'].search([('symbol', '=', split_value[currency_index].strip())])
            if len(currency):
                return split_value[(currency_index + 1) % 2] if not negative else '-' + split_value[(currency_index + 1) % 2]
            # Otherwise it is not a float with a currency symbol
            return False

    @api.model
    def _parse_float_from_data(self, data, index, name, options):
        for line in data:
            line[index] = line[index].strip()
            if not line[index]:
                continue
            thousand_separator, decimal_separator = self._infer_separators(line[index], options)

            if 'E' in line[index] or 'e' in line[index]:
                tmp_value = line[index].replace(thousand_separator, '.')
                try:
                    tmp_value = '{:f}'.format(float(tmp_value))
                    line[index] = tmp_value
                    thousand_separator = ' '
                except Exception:
                    pass

            line[index] = line[index].replace(thousand_separator, '').replace(decimal_separator, '.')
            old_value = line[index]
            line[index] = self._remove_currency_symbol(line[index])
            if line[index] is False:
                raise ImportValidationError(_("Column %s contains incorrect values (value: %s)", name, old_value), field=name)

    def _infer_separators(self, value, options):
        """ Try to infer the shape of the separators: if there are two
        different "non-numberic" characters in the number, the
        former/duplicated one would be grouping ("thousands" separator) and
        the latter would be the decimal separator. The decimal separator
        should furthermore be unique.
        """
        # can't use \p{Sc} using re so handroll it
        non_number = [
            # any character
            c for c in value
            # which is not a numeric decoration (() is used for negative
            # by accountants)
            if c not in '()-+'
            # which is not a digit or a currency symbol
            if unicodedata.category(c) not in ('Nd', 'Sc')
        ]

        counts = collections.Counter(non_number)
        # if we have two non-numbers *and* the last one has a count of 1,
        # we probably have grouping & decimal separators
        if len(counts) == 2 and counts[non_number[-1]] == 1:
            return [character for character, _count in counts.most_common()]

        # otherwise get whatever's in the options, or fallback to a default
        thousand_separator = options.get('float_thousand_separator', ' ')
        decimal_separator = options.get('float_decimal_separator', '.')
        return thousand_separator, decimal_separator

    def _parse_import_data(self, data, import_fields, options):
        """ Lauch first call to :meth:`_parse_import_data_recursive` with an
        empty prefix. :meth:`_parse_import_data_recursive` will be run
        recursively for each relational field.
        """
        return self._parse_import_data_recursive(self.res_model, '', data, import_fields, options)

    def _parse_import_data_recursive(self, model, prefix, data, import_fields, options):
        # Get fields of type date/datetime
        all_fields = self.env[model].fields_get()
        for name, field in all_fields.items():
            name = prefix + name
            if field['type'] in ('date', 'datetime') and name in import_fields:
                index = import_fields.index(name)
                self._parse_date_from_data(data, index, name, field['type'], options)
            # Check if the field is in import_field and is a relational (followed by /)
            # Also verify that the field name exactly match the import_field at the correct level.
            elif any(name + '/' in import_field and name == import_field.split('/')[prefix.count('/')] for import_field in import_fields):
                # Recursive call with the relational as new model and add the field name to the prefix
                self._parse_import_data_recursive(field['relation'], name + '/', data, import_fields, options)
            elif field['type'] in ('float', 'monetary') and name in import_fields:
                # Parse float, sometimes float values from file have currency symbol or () to denote a negative value
                # We should be able to manage both case
                index = import_fields.index(name)
                self._parse_float_from_data(data, index, name, options)
            elif field['type'] == 'binary' and field.get('attachment') and any(f in name for f in IMAGE_FIELDS) and name in import_fields:
                index = import_fields.index(name)

                with requests.Session() as session:
                    session.stream = True

                    for num, line in enumerate(data):
                        if re.match(config.get("import_image_regex", DEFAULT_IMAGE_REGEX), line[index]):
                            if not self.env.user._can_import_remote_urls():
                                raise ImportValidationError(
                                    _("You can not import images via URL, check with your administrator or support for the reason."),
                                    field=name, field_type=field['type']
                                )

                            line[index] = self._import_image_by_url(line[index], session, name, num)
                        else:
                            try:
                                base64.b64decode(line[index], validate=True)
                            except ValueError:
                                raise ImportValidationError(
                                    _("Found invalid image data, images should be imported as either URLs or base64-encoded data."),
                                    field=name, field_type=field['type']
                                )

        return data

    def _parse_date_from_data(self, data, index, name, field_type, options):
        dt = datetime.datetime
        fmt = fields.Date.to_string if field_type == 'date' else fields.Datetime.to_string
        d_fmt = options.get('date_format') or DEFAULT_SERVER_DATE_FORMAT
        dt_fmt = options.get('datetime_format') or DEFAULT_SERVER_DATETIME_FORMAT
        for num, line in enumerate(data):
            if not line[index]:
                continue

            v = line[index].strip()
            try:
                # first try parsing as a datetime if it's one
                if dt_fmt and field_type == 'datetime':
                    try:
                        line[index] = fmt(dt.strptime(v, dt_fmt))
                        continue
                    except ValueError:
                        pass
                # otherwise try parsing as a date whether it's a date
                # or datetime
                line[index] = fmt(dt.strptime(v, d_fmt))
            except ValueError as e:
                raise ImportValidationError(
                    _("Column %s contains incorrect values. Error in line %d: %s") % (name, num + 1, e),
                    field=name, field_type=field_type
                )
            except Exception as e:
                raise ImportValidationError(
                    _("Error Parsing Date [%s:L%d]: %s") % (name, num + 1, e),
                    field=name, field_type=field_type
                )

    def _import_image_by_url(self, url, session, field, line_number):
        """ Imports an image by URL

        :param str url: the original field value
        :param requests.Session session:
        :param str field: name of the field (for logging/debugging)
        :param int line_number: 0-indexed line number within the imported file (for logging/debugging)
        :return: the replacement value
        :rtype: bytes
        """
        maxsize = int(config.get("import_image_maxbytes", DEFAULT_IMAGE_MAXBYTES))
        _logger.debug("Trying to import image from URL: %s into field %s, at line %s" % (url, field, line_number))
        try:
            response = session.get(url, timeout=int(config.get("import_image_timeout", DEFAULT_IMAGE_TIMEOUT)))
            response.raise_for_status()

            if response.headers.get('Content-Length') and int(response.headers['Content-Length']) > maxsize:
                raise ImportValidationError(
                    _("File size exceeds configured maximum (%s bytes)", maxsize),
                    field=field
                )

            content = bytearray()
            for chunk in response.iter_content(DEFAULT_IMAGE_CHUNK_SIZE):
                content += chunk
                if len(content) > maxsize:
                    raise ImportValidationError(
                        _("File size exceeds configured maximum (%s bytes)", maxsize),
                        field=field
                    )

            image = Image.open(io.BytesIO(content))
            w, h = image.size
            if w * h > 42e6:  # Nokia Lumia 1020 photo resolution
                raise ImportValidationError(
                    _("Image size excessive, imported images must be smaller than 42 million pixel"),
                    field=field
                )

            return base64.b64encode(content)
        except Exception as e:
            _logger.warning(e, exc_info=True)
            raise ImportValidationError(_("Could not retrieve URL: %(url)s [%(field_name)s: L%(line_number)d]: %(error)s") % {
                'url': url,
                'field_name': field,
                'line_number': line_number + 1,
                'error': e
            })

    def execute_import(self, fields, columns, options, dryrun=False):
        """ Actual execution of the import

        :param fields: import mapping: maps each column to a field,
                       ``False`` for the columns to ignore
        :type fields: list(str|bool)
        :param columns: columns label
        :type columns: list(str|bool)
        :param dict options:
        :param bool dryrun: performs all import operations (and
                            validations) but rollbacks writes, allows
                            getting as much errors as possible without
                            the risk of clobbering the database.
        :returns: A list of errors. If the list is empty the import
                  executed fully and correctly. If the list is
                  non-empty it contains dicts with 3 keys:

                  ``type``
                    the type of error (``error|warning``)
                  ``message``
                    the error message associated with the error (a string)
                  ``record``
                    the data which failed to import (or ``false`` if that data
                    isn't available or provided)
        :rtype: dict(ids: list(int), messages: list({type, message, record}))
        """
        self.ensure_one()
        self._cr.execute('SAVEPOINT import')

        try:
            input_file_data, import_fields = self._convert_import_data(fields, options)
            # Parse date and float field
            input_file_data = self._parse_import_data(input_file_data, import_fields, options)
        except ImportValidationError as error:
            return {'messages': [error.__dict__]}

        _logger.info('importing %d rows...', len(input_file_data))

        import_fields, merged_data = self._handle_multi_mapping(import_fields, input_file_data)

        if options.get('fallback_values'):
            merged_data = self._handle_fallback_values(import_fields, merged_data, options['fallback_values'])

        name_create_enabled_fields = options.pop('name_create_enabled_fields', {})
        import_limit = options.pop('limit', None)
        model = self.env[self.res_model].with_context(
            import_file=True,
            name_create_enabled_fields=name_create_enabled_fields,
            import_set_empty_fields=options.get('import_set_empty_fields', []),
            import_skip_records=options.get('import_skip_records', []),
            _import_limit=import_limit)
        import_result = model.load(import_fields, merged_data)
        _logger.info('done')

        # If transaction aborted, RELEASE SAVEPOINT is going to raise
        # an InternalError (ROLLBACK should work, maybe). Ignore that.
        # TODO: to handle multiple errors, create savepoint around
        #       write and release it in case of write error (after
        #       adding error to errors array) => can keep on trying to
        #       import stuff, and rollback at the end if there is any
        #       error in the results.
        try:
            if dryrun:
                self._cr.execute('ROLLBACK TO SAVEPOINT import')
                # cancel all changes done to the registry/ormcache
                self.pool.clear_caches()
                self.pool.reset_changes()
            else:
                self._cr.execute('RELEASE SAVEPOINT import')
        except psycopg2.InternalError:
            pass

        # Insert/Update mapping columns when import complete successfully
        if import_result['ids'] and options.get('has_headers'):
            BaseImportMapping = self.env['base_import.mapping']
            for index, column_name in enumerate(columns):
                if column_name:
                    # Update to latest selected field
                    mapping_domain = [('res_model', '=', self.res_model), ('column_name', '=', column_name)]
                    column_mapping = BaseImportMapping.search(mapping_domain, limit=1)
                    if column_mapping:
                        if column_mapping.field_name != fields[index]:
                            column_mapping.field_name = fields[index]
                    else:
                        BaseImportMapping.create({
                            'res_model': self.res_model,
                            'column_name': column_name,
                            'field_name': fields[index]
                        })
        if 'name' in import_fields:
            index_of_name = import_fields.index('name')
            skipped = options.get('skip', 0)
            # pad front as data doesn't contain anythig for skipped lines
            r = import_result['name'] = [''] * skipped
            # only add names for the window being imported
            r.extend(x[index_of_name] for x in input_file_data[:import_limit])
            # pad back (though that's probably not useful)
            r.extend([''] * (len(input_file_data) - (import_limit or 0)))
        else:
            import_result['name'] = []

        skip = options.get('skip', 0)
        # convert load's internal nextrow to the imported file's
        if import_result['nextrow']: # don't update if nextrow = 0 (= no nextrow)
            import_result['nextrow'] += skip

        return import_result

    def _handle_multi_mapping(self, import_fields, input_file_data):
        """ This method handles multiple mapping on the same field.

        It will return the list of the mapped fields and the concatenated data for each field:

        - If two column are mapped on the same text or char field, they will end up
          in only one column, concatenated via space (char) or new line (text).
        - The same logic is used for many2many fields. Multiple values can be
          imported if they are separated by ``,``.

        Input/output Example:

        input data
            .. code-block:: python

                [
                    ["Value part 1", "1", "res.partner_id1", "Value part 2"],
                    ["I am", "1", "res.partner_id1", "Batman"],
                ]

        import_fields
            ``[desc, some_number, partner, desc]``

        output merged_data
            .. code-block:: python

                [
                    ["Value part 1 Value part 2", "1", "res.partner_id1"],
                    ["I am Batman", "1", "res.partner_id1"],
                ]
        fields
            ``[desc, some_number, partner]``
        """
        # Get fields and their occurrences indexes
        # Among the fields that have been mapped, we get their corresponding mapped column indexes
        # as multiple fields could have been mapped to multiple columns.
        mapped_field_indexes = {}
        for idx, field in enumerate(field for field in import_fields if field):
            mapped_field_indexes.setdefault(field, list()).append(idx)
        import_fields = list(mapped_field_indexes.keys())

        # recreate data and merge duplicates (applies only on text or char fields)
        # Also handles multi-mapping on "field of relation fields".
        merged_data = []
        for record in input_file_data:
            new_record = []
            for fields, indexes in mapped_field_indexes.items():
                split_fields = fields.split('/')
                target_field = split_fields[-1]

                # get target_field type (on target model)
                target_model = self.res_model
                for field in split_fields:
                    if field != target_field:  # if not on the last hierarchy level, retarget the model
                        target_model = self.env[target_model][field]._name
                field = self.env[target_model]._fields.get(target_field)
                field_type = field.type if field else ''

                # merge data if necessary
                if field_type == 'char':
                    new_record.append(' '.join(record[idx] for idx in indexes if record[idx]))
                elif field_type == 'text':
                    new_record.append('\n'.join(record[idx] for idx in indexes if record[idx]))
                elif field_type == 'many2many':
                    new_record.append(','.join(record[idx] for idx in indexes if record[idx]))
                else:
                    new_record.append(record[indexes[0]])

            merged_data.append(new_record)

        return import_fields, merged_data

    def _handle_fallback_values(self, import_field, input_file_data, fallback_values):
        """
        If there are fallback values, this method will replace the input file
        data value if it does not match the possible values for the given field.
        This is only valid for boolean and selection fields.

        .. note::

            We can consider that we need to retrieve the selection values for
            all the fields in fallback_values, as if they are present, it's because
            there was already a conflict during first import run and user had to
            select a fallback value for the field.

        :param: list import_field: ordered list of field that have been matched to import data
        :param: list input_file_data: ordered list of values (list) that need to be imported in the given import_fields
        :param: dict fallback_values:

            contains all the fields that have been tagged by the user to use a
            specific fallback value in case the value to import does not match
            values accepted by the field (selection or boolean) e.g.::

                {
                    'fieldName': {
                        'fallback_value': fallback_value,
                        'field_model': field_model,
                        'field_type': field_type
                    },
                    'state': {
                        'fallback_value': 'draft',
                        'field_model': field_model,
                        'field_type': 'selection'
                    },
                    'active': {
                        'fallback_value': 'true',
                        'field_model': field_model,
                        'field_type': 'boolean'
                    }
                }
        """
        # add possible selection values into our fallback dictionary for fields of type "selection"
        for field_string in fallback_values:
            if fallback_values[field_string]['field_type'] != "selection":
                continue
            field_path = field_string.split('/')
            target_field = field_path[-1]
            target_model = self.env[fallback_values[field_string]['field_model']]

            selection_values = [value.lower() for (key, value) in target_model.fields_get([target_field])[target_field]['selection']]
            fallback_values[field_string]['selection_values'] = selection_values

        # check fallback values
        for record_index, records in enumerate(input_file_data):
            for column_index, value in enumerate(records):
                field = import_field[column_index]

                if field in fallback_values:
                    fallback_value = fallback_values[field]['fallback_value']
                    # Boolean
                    if fallback_values[field]['field_type'] == "boolean":
                        value = value if value.lower() in ('0', '1', 'true', 'false') else fallback_value
                    # Selection
                    elif value.lower() not in fallback_values[field]["selection_values"]:
                        value = fallback_value if fallback_value != 'skip' else None  # don't set any value if we skip

                    input_file_data[record_index][column_index] = value

        return input_file_data

_SEPARATORS = [' ', '/', '-', '.', '']
_PATTERN_BASELINE = [
    ('%m', '%d', '%Y'),
    ('%d', '%m', '%Y'),
    ('%Y', '%m', '%d'),
    ('%Y', '%d', '%m'),
]
DATE_FORMATS = []
# take the baseline format and duplicate performing the following
# substitution: long year -> short year, numerical month -> short
# month, numerical month -> long month. Each substitution builds on
# the previous two
for ps in _PATTERN_BASELINE:
    patterns = {ps}
    for s, t in [('%Y', '%y')]:
        patterns.update([ # need listcomp: with genexpr "set changed size during iteration"
            tuple(t if it == s else it for it in f)
            for f in patterns
        ])
    DATE_FORMATS.extend(patterns)
DATE_PATTERNS = [
    sep.join(fmt)
    for sep in _SEPARATORS
    for fmt in DATE_FORMATS
]
TIME_PATTERNS = [
    '%H:%M:%S', '%H:%M', '%H', # 24h
    '%I:%M:%S %p', '%I:%M %p', '%I %p', # 12h
]

def check_patterns(patterns, values):
    for pattern in patterns:
        p = to_re(pattern)
        for val in values:
            if val and not p.match(val):
                break

        else:  # no break, all match
            return pattern

    return None

def to_re(pattern):
    """ cut down version of TimeRE converting strptime patterns to regex
    """
    pattern = re.sub(r'\s+', r'\\s+', pattern)
    pattern = re.sub('%([a-z])', _replacer, pattern, flags=re.IGNORECASE)
    pattern = '^' + pattern + '$'
    return re.compile(pattern, re.IGNORECASE)
def _replacer(m):
    return _P_TO_RE[m.group(1)]

_P_TO_RE = {
    'd': r"(3[0-1]|[1-2]\d|0[1-9]|[1-9]| [1-9])",
    'H': r"(2[0-3]|[0-1]\d|\d)",
    'I': r"(1[0-2]|0[1-9]|[1-9])",
    'm': r"(1[0-2]|0[1-9]|[1-9])",
    'M': r"([0-5]\d|\d)",
    'S': r"(6[0-1]|[0-5]\d|\d)",
    'y': r"(\d\d)",
    'Y': r"(\d\d\d\d)",

    'p': r"(am|pm)",

    '%': '%',
}

```

## File: models\odf_ods_reader.py

```python
# Copyright 2011 Marco Conti

# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# sourced from https://github.com/marcoconti83/read-ods-with-odfpy
# further altered locally

from odf import opendocument
from odf.table import Table, TableRow, TableCell
from odf.text import P


class ODSReader(object):

    # loads the file
    def __init__(self, file=None, content=None, clonespannedcolumns=None):
        if not content:
            self.clonespannedcolumns = clonespannedcolumns
            self.doc = opendocument.load(file)
        else:
            self.clonespannedcolumns = clonespannedcolumns
            self.doc = content
        self.SHEETS = {}
        for sheet in self.doc.spreadsheet.getElementsByType(Table):
            self.readSheet(sheet)

    # reads a sheet in the sheet dictionary, storing each sheet as an
    # array (rows) of arrays (columns)
    def readSheet(self, sheet):
        name = sheet.getAttribute("name")
        rows = sheet.getElementsByType(TableRow)
        arrRows = []

        # for each row
        for row in rows:
            arrCells = []
            cells = row.getElementsByType(TableCell)

            # for each cell
            for count, cell in enumerate(cells, start=1):
                # repeated value?
                repeat = 0
                if count != len(cells):
                    repeat = cell.getAttribute("numbercolumnsrepeated")
                if not repeat:
                    repeat = 1
                    spanned = int(cell.getAttribute('numbercolumnsspanned') or 0)
                    # clone spanned cells
                    if self.clonespannedcolumns is not None and spanned > 1:
                        repeat = spanned

                ps = cell.getElementsByType(P)
                textContent = u""

                # for each text/text:span node
                for p in ps:
                    for n in p.childNodes:
                        if n.nodeType == 1 and n.tagName == "text:span":
                            for c in n.childNodes:
                                if c.nodeType == 3:
                                    textContent = u'{}{}'.format(textContent, n.data)

                        if n.nodeType == 3:
                            textContent = u'{}{}'.format(textContent, n.data)

                if textContent:
                    if not textContent.startswith("#"):  # ignore comments cells
                        for rr in range(int(repeat)):  # repeated?
                            arrCells.append(textContent)
                else:
                    for rr in range(int(repeat)):
                        arrCells.append("")

            # if row contained something
            if arrCells:
                arrRows.append(arrCells)

            #else:
            #    print ("Empty or commented row (", row_comment, ")")

        self.SHEETS[name] = arrRows

    # returns a sheet as an array (rows) of arrays (columns)
    def getSheet(self, name):
        return self.SHEETS[name]

    def getFirstSheet(self):
        return next(iter(self.SHEETS.values()))

```

## File: models\test_models.py

```python
# -*- coding: utf-8 -*-
from odoo import fields, models


def model(suffix_name):
    return 'base_import.tests.models.%s' % suffix_name


class Char(models.Model):
    _name = model('char')
    _description = 'Tests : Base Import Model, Character'

    value = fields.Char()
class CharRequired(models.Model):
    _name = model('char.required')
    _description = 'Tests : Base Import Model, Character required'

    value = fields.Char(required=True)

class CharReadonly(models.Model):
    _name = model('char.readonly')
    _description = 'Tests : Base Import Model, Character readonly'

    value = fields.Char(readonly=True)

class CharStates(models.Model):
    _name = model('char.states')
    _description = 'Tests : Base Import Model, Character states'

    value = fields.Char(readonly=True, states={'draft': [('readonly', False)]})

class CharNoreadonly(models.Model):
    _name = model('char.noreadonly')
    _description = 'Tests : Base Import Model, Character No readonly'

    value = fields.Char(readonly=True, states={'draft': [('invisible', True)]})

class CharStillreadonly(models.Model):
    _name = model('char.stillreadonly')
    _description = 'Tests : Base Import Model, Character still readonly'

    value = fields.Char(readonly=True, states={'draft': [('readonly', True)]})

# TODO: complex field (m2m, o2m, m2o)
class M2o(models.Model):
    _name = model('m2o')
    _description = 'Tests : Base Import Model, Many to One'

    value = fields.Many2one(model('m2o.related'))

class M2oRelated(models.Model):
    _name = model('m2o.related')
    _description = 'Tests : Base Import Model, Many to One related'

    value = fields.Integer(default=42)

class M2oRequired(models.Model):
    _name = model('m2o.required')
    _description = 'Tests : Base Import Model, Many to One required'

    value = fields.Many2one(model('m2o.required.related'), required=True)

class M2oRequiredRelated(models.Model):
    _name = model('m2o.required.related')
    _description = 'Tests : Base Import Model, Many to One required related'

    value = fields.Integer(default=42)

class O2m(models.Model):
    _name = model('o2m')
    _description = 'Tests : Base Import Model, One to Many'

    name = fields.Char()
    value = fields.One2many(model('o2m.child'), 'parent_id')

class O2mChild(models.Model):
    _name = model('o2m.child')
    _description = 'Tests : Base Import Model, One to Many child'

    parent_id = fields.Many2one(model('o2m'))
    value = fields.Integer()

class PreviewModel(models.Model):
    _name = model('preview')
    _description = 'Tests : Base Import Model Preview'

    name = fields.Char('Name')
    somevalue = fields.Integer(string='Some Value', required=True)
    othervalue = fields.Integer(string='Other Variable')

class FloatModel(models.Model):
    _name = model('float')
    _description = 'Tests: Base Import Model Float'

    value = fields.Float()
    value2 = fields.Monetary()
    currency_id = fields.Many2one('res.currency')

class ComplexModel(models.Model):
    _name = model('complex')
    _description = 'Tests: Base Import Model Complex'

    f = fields.Float()
    m = fields.Monetary()
    c = fields.Char()
    currency_id = fields.Many2one('res.currency')
    d = fields.Date()
    dt = fields.Datetime()

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import base_import
from . import test_models

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_base_import_tests_models_char,base.import.tests.models.char,model_base_import_tests_models_char,base.group_user,1,1,1,1
access_base_import_tests_models_char_required,base.import.tests.models.char.required,model_base_import_tests_models_char_required,base.group_user,1,1,1,1
access_base_import_tests_models_char_readonly,base.import.tests.models.char.readonly,model_base_import_tests_models_char_readonly,base.group_user,1,1,1,1
access_base_import_tests_models_char_states,base.import.tests.models.char.states,model_base_import_tests_models_char_states,base.group_user,1,1,1,1
access_base_import_tests_models_char_noreadonly,base.import.tests.models.char.noreadonly,model_base_import_tests_models_char_noreadonly,base.group_user,1,1,1,1
access_base_import_tests_models_char_stillreadonly,base.import.tests.models.char.stillreadonly,model_base_import_tests_models_char_stillreadonly,base.group_user,1,1,1,1
access_base_import_tests_models_m2o,base.import.tests.models.m2o,model_base_import_tests_models_m2o,base.group_user,1,1,1,1
access_base_import_tests_models_m2o_related,base.import.tests.models.m2o.related,model_base_import_tests_models_m2o_related,base.group_user,1,1,1,1
access_base_import_tests_models_m2o_required,base.import.tests.models.m2o.required,model_base_import_tests_models_m2o_required,base.group_user,1,1,1,1
access_base_import_tests_models_m2o_required_related,base.import.tests.models.m2o.required.related,model_base_import_tests_models_m2o_required_related,base.group_user,1,1,1,1
access_base_import_tests_models_o2m,base.import.tests.models.o2m,model_base_import_tests_models_o2m,base.group_user,1,1,1,1
access_base_import_tests_models_o2m_child,base.import.tests.models.o2m.child,model_base_import_tests_models_o2m_child,base.group_user,1,1,1,1
access_base_import_tests_models_float,base.import.tests.models.float,model_base_import_tests_models_float,base.group_user,1,1,1,1
access_base_import_tests_models_preview,base.import.tests.models.preview,model_base_import_tests_models_preview,base.group_user,1,1,1,1
access_base_import_mapping,base.import.mapping,model_base_import_mapping,base.group_user,1,1,1,1
access_base_import_tests_models_complex,access_base_import_tests_models_complex,model_base_import_tests_models_complex,base.group_user,1,0,0,0
access_base_import_import,access.base_import.import,model_base_import_import,base.group_user,1,1,1,0

```

## File: static\csv\database_import_test.sql

```sql
--
-- PostgreSQL database dump
--

-- Dumped from database version 9.5.2
-- Dumped by pg_dump version 9.5.2

SET statement_timeout = 0;
SET lock_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SET check_function_bodies = false;
SET client_min_messages = warning;
SET row_security = off;

--
-- Name: plpgsql; Type: EXTENSION; Schema: -; Owner: -
--

CREATE EXTENSION IF NOT EXISTS plpgsql WITH SCHEMA pg_catalog;


--
-- Name: EXTENSION plpgsql; Type: COMMENT; Schema: -; Owner: -
--

COMMENT ON EXTENSION plpgsql IS 'PL/pgSQL procedural language';


SET search_path = public, pg_catalog;

SET default_tablespace = '';

SET default_with_oids = false;

--
-- Name: companies; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE companies (
    id integer NOT NULL,
    company_name character varying
);


--
-- Name: companies_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

CREATE SEQUENCE companies_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


--
-- Name: companies_id_seq; Type: SEQUENCE OWNED BY; Schema: public; Owner: -
--

ALTER SEQUENCE companies_id_seq OWNED BY companies.id;


--
-- Name: persons; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE persons (
    id integer NOT NULL,
    company_id integer,
    person_name character varying
);


--
-- Name: persons_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

CREATE SEQUENCE persons_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


--
-- Name: persons_id_seq; Type: SEQUENCE OWNED BY; Schema: public; Owner: -
--

ALTER SEQUENCE persons_id_seq OWNED BY persons.id;


--
-- Name: id; Type: DEFAULT; Schema: public; Owner: -
--

ALTER TABLE ONLY companies ALTER COLUMN id SET DEFAULT nextval('companies_id_seq'::regclass);


--
-- Name: id; Type: DEFAULT; Schema: public; Owner: -
--

ALTER TABLE ONLY persons ALTER COLUMN id SET DEFAULT nextval('persons_id_seq'::regclass);


--
-- Data for Name: companies; Type: TABLE DATA; Schema: public; Owner: -
--

COPY companies (id, company_name) FROM stdin;
1	Bigees
2	Organi
3	Boum
\.


--
-- Name: companies_id_seq; Type: SEQUENCE SET; Schema: public; Owner: -
--

SELECT pg_catalog.setval('companies_id_seq', 3, true);


--
-- Data for Name: persons; Type: TABLE DATA; Schema: public; Owner: -
--

COPY persons (id, company_id, person_name) FROM stdin;
1	1	Fabien
2	1	Laurence
3	2	Eric
4	3	Ramzy
\.


--
-- Name: persons_id_seq; Type: SEQUENCE SET; Schema: public; Owner: -
--

SELECT pg_catalog.setval('persons_id_seq', 4, true);


--
-- Name: companies_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY companies
    ADD CONSTRAINT companies_pkey PRIMARY KEY (id);


--
-- Name: persons_company_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY persons
    ADD CONSTRAINT persons_company_id_fkey FOREIGN KEY (company_id) REFERENCES companies(id);


--
-- Name: public; Type: ACL; Schema: -; Owner: -
--

REVOKE ALL ON SCHEMA public FROM PUBLIC;
REVOKE ALL ON SCHEMA public FROM postgres;
GRANT ALL ON SCHEMA public TO postgres;
GRANT ALL ON SCHEMA public TO PUBLIC;


--
-- PostgreSQL database dump complete
--


```

## File: static\csv\External_id_3rd_party_application_products.csv

```csv
External ID,Name,Internal Reference,Category/External ID,Can be Expensed,Can be Purchased,Can be Sold,Sale Price,Cost,Supply Method,Product Type,Procurement Method
a6,Aluminum Stool,ALS,a5,False,True,True,49.00,25.00,Buy,Stockable Product,Make to Stock
a7,Chair,CHR,a5,False,True,True,89.00,40.00,Buy,Stockable Product,Make to Stock
a8,Table,TBL,a4,False,True,True,169.00,100.00,Buy,Stockable Product,Make to Stock
a9,Software Book Tutorial,SBT,a2,False,True,False,19.00,8.00,Buy,Consumable,Make to Stock
a10,Fuel,FL,a1,True,False,False,0.30,0.25,Buy,Service,Make to Stock

```

## File: static\csv\External_id_3rd_party_application_product_categories.csv

```csv
External ID,Name,Parent Category/External ID
a1,Expenses,product.product_category_all
a2,Other Products,product.product_category_all
a3,Sellable Products,product.product_category_all
a4,Tables,a1
a5,Seating furniture,a2

```

## File: static\csv\m2m_customers_tags.csv

```csv
Name,Reference,Tags,Customer,Street,City,Country
Credit & Leasing,3,Services,True,Central Avenue 814,Johannesburg,South Africa
Services & Finance,5,"Consultancy Services,IT Services",True,Grove Road 5,London,United Kingdom
Hydra Supplies,6,"Manufacturer,Retailer",True,Palm Street 9,Los Angeles,United States
Bolts & Screws,8,"Wholesaler,Components Buyer",True,Rua Américo 1000,Campinas,Brazil
National Parts & Supplies,18,"Manufacturer,Wholesaler",True,Guangdong Way 20,Shenzen,China

```

## File: static\csv\Name_products.csv

```csv
Name,Internal Reference,Can be Expensed,Can be Purchased,Can be Sold,Sale Price,Cost,Supply Method,Product Type,Procurement Method
Aluminum Stool,ALS,False,True,True,49.00,25.00,Buy,Stockable Product,Make to Stock
Chair,CHR,False,True,True,89.00,40.00,Buy,Stockable Product,Make to Stock
Table,TBL,False,True,True,169.00,100.00,Buy,Stockable Product,Make to Stock
Software Book Tutorial,SBT,False,True,False,19.00,8.00,Buy,Consumable,Make to Stock
Fuel,FL,True,False,False,0.30,0.25,Buy,Service,Make to Stock

```

## File: static\csv\o2m_customers_contacts.csv

```csv
Name,Is a company,Related company,Address type,Street,ZIP,City,State,Country
Aurora Shelves,1,,,25 Pacific Road,95101,San José,California,United States
Roger Martins,0,Aurora Shelves,Invoice address,27 Pacific Road,95102,San José,California,United States
House Sales Direct,1,,,104 Saint Mary Avenue,94059,Redwood,California,United States
Yvan Holiday,0,House Sales Direct,Contact,104 Saint Mary Avenue,94060,Redwood,California,United States
Jack Unsworth,0,House Sales Direct,Invoice address,227 Jackson Road,94061,Redwood,California,United States
Michael Mason,0,,,16 5th Avenue,94104,San Francisco,California,United States
International Wood,1,,,748 White House Boulevard,20004,Washington,D.C.,United States
Sharon Pecker,0,International Wood,Invoice address,755 White House Boulevard,20005,Washington,D.C.,United States
```

## File: static\csv\o2m_purchase_order_lines.csv

```csv
"Order Date","Order Reference",Vendor,"Order Lines / Product","Order Lines / Description","Order Lines / Quantity","Order Lines / Product Unit of Measure","Order Lines / Unit Price","Order Lines / Scheduled Date"
2016-12-15,PO00008,ASUSTeK,ADPT,ADPT,20.0,"Units",13.0,2016-12-15
,,,CARD,CARD,30,"Units",876.0,2016-12-15
,,,C-Case,C-Case,40,"Units",20.0,2016-12-15
2016-12-15,PO00009,Camptocamp,CD,CD,5,"Units",18.4,2016-12-15
,,,CPUa8,CPUa8,15,"Units",1910.0,2016-12-15

```

## File: static\csv\purchase.order_functional_error_line_cant_adpat.csv

```csv
"Order Reference","Vendor"
"PO000020","ASUSTeK"
"PO000021","Camptocamp"

```

## File: static\lib\javascript-state-machine\index.html

```html
<!DOCTYPE html> 
<html>
<head>
  <title>Javascript Finite State Machine</title>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/> 
  <link href="demo/demo.css" media="screen, print" rel="stylesheet" type="text/css" /> 
</head> 
 
<body> 

  <div id="demo" class='green'>

    <h1> Finite State Machine </h1>

    <div id="controls">
      <button id="clear" onclick="Demo.clear();">clear</button>
      <button id="calm"  onclick="Demo.calm();">calm</button>
      <button id="warn"  onclick="Demo.warn();">warn</button>
      <button id="panic" onclick="Demo.panic();">panic!</button>
    </div>

    <div id="diagram">
    </div>

    <div id="notes">
      <i>dashed lines are asynchronous state transitions (3 seconds)</i>
    </div>

    <textarea id="output">
    </textarea>

  </div>


  <script src="state-machine.js"></script>
  <script src="demo/demo.js"></script>

</body> 
</html>

```

## File: static\lib\javascript-state-machine\LICENSE

```text
Copyright (c) 2012 Jake Gordon and contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.


```

## File: static\lib\javascript-state-machine\Rakefile

```text

desc "create minified version of state-machine.js"
task :minify do
  require File.expand_path(File.join(File.dirname(__FILE__), 'minifier/minifier'))
  Minifier.enabled = true
  Minifier.minify('state-machine.js')
end


```

## File: static\lib\javascript-state-machine\state-machine.js

```javascript
(function (window) {

  StateMachine = {

    //---------------------------------------------------------------------------

    VERSION: "2.1.0",

    //---------------------------------------------------------------------------

    Result: {
      SUCCEEDED:    1, // the event transitioned successfully from one state to another
      NOTRANSITION: 2, // the event was successfull but no state transition was necessary
      CANCELLED:    3, // the event was cancelled by the caller in a beforeEvent callback
      ASYNC:        4, // the event is asynchronous and the caller is in control of when the transition occurs
    },

    Error: {
      INVALID_TRANSITION: 100, // caller tried to fire an event that was innapropriate in the current state
      PENDING_TRANSITION: 200, // caller tried to fire an event while an async transition was still pending
      INVALID_CALLBACK:   300, // caller provided callback function threw an exception
    },

    WILDCARD: '*',
    ASYNC: 'async',

    //---------------------------------------------------------------------------

    create: function(cfg, target) {

      var initial   = (typeof cfg.initial == 'string') ? { state: cfg.initial } : cfg.initial; // allow for a simple string, or an object with { state: 'foo', event: 'setup', defer: true|false }
      var fsm       = target || cfg.target  || {};
      var events    = cfg.events || [];
      var callbacks = cfg.callbacks || {};
      var map       = {};

      var add = function(e) {
        var from = (e.from instanceof Array) ? e.from : (e.from ? [e.from] : [StateMachine.WILDCARD]); // allow 'wildcard' transition if 'from' is not specified
        map[e.name] = map[e.name] || {};
        for (var n = 0 ; n < from.length ; n++)
          map[e.name][from[n]] = e.to || from[n]; // allow no-op transition if 'to' is not specified
      };

      if (initial) {
        initial.event = initial.event || 'startup';
        add({ name: initial.event, from: 'none', to: initial.state });
      }

      for(var n = 0 ; n < events.length ; n++)
        add(events[n]);

      for(var name in map) {
        if (map.hasOwnProperty(name))
          fsm[name] = StateMachine.buildEvent(name, map[name]);
      }

      for(var name in callbacks) {
        if (callbacks.hasOwnProperty(name))
          fsm[name] = callbacks[name]
      }

      fsm.current = 'none';
      fsm.is      = function(state) { return this.current == state; };
      fsm.can     = function(event) { return !this.transition && (map[event].hasOwnProperty(this.current) || map[event].hasOwnProperty(StateMachine.WILDCARD)); }
      fsm.cannot  = function(event) { return !this.can(event); };
      fsm.error   = cfg.error || function(name, from, to, args, error, msg) { throw msg; }; // default behavior when something unexpected happens is to throw an exception, but caller can override this behavior if desired (see github issue #3)

      if (initial && !initial.defer)
        fsm[initial.event]();

      return fsm;

    },

    //===========================================================================

    doCallback: function(fsm, func, name, from, to, args) {
      if (func) {
        try {
          return func.apply(fsm, [name, from, to].concat(args));
        }
        catch(e) {
          return fsm.error(name, from, to, args, StateMachine.Error.INVALID_CALLBACK, "an exception occurred in a caller-provided callback function");
        }
      }
    },

    beforeEvent: function(fsm, name, from, to, args) { return StateMachine.doCallback(fsm, fsm['onbefore' + name],                     name, from, to, args); },
    afterEvent:  function(fsm, name, from, to, args) { return StateMachine.doCallback(fsm, fsm['onafter'  + name] || fsm['on' + name], name, from, to, args); },
    leaveState:  function(fsm, name, from, to, args) { return StateMachine.doCallback(fsm, fsm['onleave'  + from],                     name, from, to, args); },
    enterState:  function(fsm, name, from, to, args) { return StateMachine.doCallback(fsm, fsm['onenter'  + to]   || fsm['on' + to],   name, from, to, args); },
    changeState: function(fsm, name, from, to, args) { return StateMachine.doCallback(fsm, fsm['onchangestate'],                       name, from, to, args); },


    buildEvent: function(name, map) {
      return function() {

        var from  = this.current;
        var to    = map[from] || map[StateMachine.WILDCARD] || from;
        var args  = Array.prototype.slice.call(arguments); // turn arguments into pure array

        if (this.transition)
          return this.error(name, from, to, args, StateMachine.Error.PENDING_TRANSITION, "event " + name + " inappropriate because previous transition did not complete");

        if (this.cannot(name))
          return this.error(name, from, to, args, StateMachine.Error.INVALID_TRANSITION, "event " + name + " inappropriate in current state " + this.current);

        if (false === StateMachine.beforeEvent(this, name, from, to, args))
          return StateMachine.CANCELLED;

        if (from === to) {
          StateMachine.afterEvent(this, name, from, to, args);
          return StateMachine.NOTRANSITION;
        }

        // prepare a transition method for use EITHER lower down, or by caller if they want an async transition (indicated by an ASYNC return value from leaveState)
        var fsm = this;
        this.transition = function() {
          fsm.transition = null; // this method should only ever be called once
          fsm.current = to;
          StateMachine.enterState( fsm, name, from, to, args);
          StateMachine.changeState(fsm, name, from, to, args);
          StateMachine.afterEvent( fsm, name, from, to, args);
        };

        var leave = StateMachine.leaveState(this, name, from, to, args);
        if (false === leave) {
          this.transition = null;
          return StateMachine.CANCELLED;
        }
        else if ("async" === leave) {
          return StateMachine.ASYNC;
        }
        else {
          if (this.transition)
            this.transition(); // in case user manually called transition() but forgot to return ASYNC
          return StateMachine.SUCCEEDED;
        }

      };
    }

  }; // StateMachine

  //===========================================================================

  if ("function" === typeof define) {
    define("statemachine", [], function() { return StateMachine; });
  }
  else {
    window.StateMachine = StateMachine;
  }

}(this));


```

## File: static\src\import_records\import_records.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";

const { Component } = owl;
const favoriteMenuRegistry = registry.category("favoriteMenu");

/**
 * 'Import records' menu
 *
 * This component is used to import the records for particular model.
 * @extends Component
 */
export class ImportRecords extends Component {
    setup() {
        this.action = useService("action");
    }

    //---------------------------------------------------------------------
    // Protected
    //---------------------------------------------------------------------

    importRecords() {
        const { context, resModel } = this.env.searchModel;
        this.action.doAction({
            type: "ir.actions.client",
            tag: "import",
            params: { model: resModel, context }
        });
    }
}

ImportRecords.template = "base_import.ImportRecords";

const importRecordsItem = {
    Component: ImportRecords,
    groupNumber: 4,
    isDisplayed: ({ config, isSmall }) =>
        !isSmall &&
        config.actionType === "ir.actions.act_window" &&
        ["kanban", "list"].includes(config.viewType)
        // TODO: add arch info to searchModel?
        // !!JSON.parse(env.view.arch.attrs.import || "1") &&
        // !!JSON.parse(env.view.arch.attrs.create || "1"),
};

favoriteMenuRegistry.add("import-menu", importRecordsItem, { sequence: 1 });

```

## File: static\src\import_records\import_records.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="base_import.ImportRecords" owl="1">
        <DropdownItem class="o_import_menu dropdown-item" t-on-click="importRecords">
            Import records
        </DropdownItem>
    </t>

</templates>

```

## File: static\src\legacy\js\import_action.js

```javascript
odoo.define('base_import.import', function (require) {
"use strict";

var AbstractAction = require('web.AbstractAction');
var config = require('web.config');
var core = require('web.core');
var Dialog = require('web.Dialog');
var session = require('web.session');
var time = require('web.time');
var fieldUtils = require('web.field_utils');

var QWeb = core.qweb;
var _t = core._t;
var _lt = core._lt;
var StateMachine = window.StateMachine;

/**
 * Safari does not deal well at all with raw JSON data being
 * returned. As a result, we're going to cheat by using a
 * pseudo-jsonp: instead of getting JSON data in the iframe, we're
 * getting a ``script`` tag which consists of a function call and
 * the returned data (the json dump).
 *
 * The function is an auto-generated name bound to ``window``,
 * which calls back into the callback provided here.
 *
 * @param {Object} form the form element (DOM or jQuery) to use in the call
 * @param {Object} attributes jquery.form attributes object
 * @param {Function} callback function to call with the returned data
 */
function jsonp(form, attributes, callback) {
    attributes = attributes || {};
    var options = {jsonp: _.uniqueId('import_callback_')};
    window[options.jsonp] = function () {
        delete window[options.jsonp];
        callback.apply(null, arguments);
    };
    if ('data' in attributes) {
        _.extend(attributes.data, options);
    } else {
        _.extend(attributes, {data: options});
    }
    _.extend(attributes, {
        dataType: 'script',
    });
    $(form).ajaxSubmit(attributes);
}
function _make_option(term) { return {id: term, text: term }; }
function _from_data(data, term) {
    return _.findWhere(data, {id: term}) || _make_option(term);
}

/**
 * query returns a list of suggestion select2 objects, this function:
 *
 * * returns data exactly matching query by either id or text if those exist
 * * otherwise it returns a select2 option matching the term and any data
 *   option whose id or text matches (by substring)
 */
function dataFilteredQuery(q) {
    var suggestions = _.clone(this.data);
    if (q.term) {
        var exact = _.filter(suggestions, function (s) {
            return s.id === q.term || s.text === q.term;
        });
        if (exact.length) {
            suggestions = exact;
        } else {
            suggestions = [_make_option(q.term)].concat(_.filter(suggestions, function (s) {
                return s.id.indexOf(q.term) !== -1 || s.text.indexOf(q.term) !== -1
            }));
        }
    }
    q.callback({results: suggestions});
}

var DataImport = AbstractAction.extend({
    hasControlPanel: true,
    contentTemplate: 'ImportView',
    opts: [
        {name: 'encoding', label: _lt("Encoding:"), value: ''},
        {name: 'separator', label: _lt("Separator:"), value: ''},
        {name: 'quoting', label: _lt("Text Delimiter:"), value: '"'}
    ],
    spreadsheet_opts: [
        {name: 'sheet', label: _lt("Selected Sheet:"), value: ''},
    ],
    parse_opts_formats: [
        {name: 'date_format', label: _lt("Date Format:"), value: ''},
        {name: 'datetime_format', label: _lt("Datetime Format:"), value: ''},
    ],
    parse_opts_separators: [
        {name: 'float_thousand_separator', label: _lt("Thousands Separator:"), value: ','},
        {name: 'float_decimal_separator', label: _lt("Decimal Separator:"), value: '.'}
    ],
    events: {
        'change .oe_import_file': 'loaded_file',
        'change input.oe_import_has_header, .oe_import_sheet': 'settings_changed',
        'change input.oe_import_advanced_mode': function (e) {
            this.do_not_change_match = true;
            this['settings_changed']();
        },
        'click .oe_import_report a.oe_import_report_count': function (e) {
            e.preventDefault();
            $(e.target).parent().find('i.arrow').toggleClass('up down');
            $(e.target).parent().parent().toggleClass('oe_import_report_showmore');
        },
        'click .oe_import_report_see_possible_value': function (e) {
            e.preventDefault();
            $(e.target).parent().toggleClass('oe_import_report_showmore');
        },
        'click .oe_import_moreinfo_action a': function (e) {
            e.preventDefault();
            // #data will parse the attribute on its own, we don't like
            // that sort of things
            var action = JSON.parse($(e.target).attr('data-action'));
            // FIXME: when JS-side clean_action
            action.views = _(action.views).map(function (view) {
                var id = view[0], type = view[1];
                return [
                    id,
                    type !== 'tree' ? type
                      : action.view_type === 'form' ? 'list'
                      : 'tree'
                ];
            });
            this.do_action(_.extend(action, {
                target: 'new',
                flags: {
                    search_view: true,
                    display_title: true,
                    pager: true,
                    list: {selectable: false}
                }
            }));
        },
    },
    init: function (parent, action) {
        this._super.apply(this, arguments);
        this.action_manager = parent;
        this.res_model = action.params.model;
        this.parent_context = action.params.context || {};
        // import object id
        this.id = null;
        this.session = session;
        this._title = _t('Import a File'); // Displayed in the breadcrumbs
        this.do_not_change_match = false;
        this.sheets = [];
        this.selectionFields = {};  // Used to compute fallback values in backend.
    },
    /**
     * @override
     */
    willStart: function () {
        var self = this;
        var def = this._rpc({
            model: this.res_model,
            method: 'get_import_templates',
            context: this.parent_context,
        }).then(function (result) {
            self.importTemplates = result;
        });
        return Promise.all([this._super.apply(this, arguments), def]);
    },
    start: function () {
        var self = this;
        this.$form = this.$('form');
        this.setup_encoding_picker();
        this.setup_separator_picker();
        this.setup_float_format_picker();
        this.setup_date_format_picker();
        this.setup_sheets_picker();

        return this._super().then(function () {
            return self.create_model().then(function (id) {
                self.id = id;
                self.$('input[name=import_id]').val(id);

                self.renderButtons();
                var status = {
                    cp_content: {$buttons: self.$buttons},
                };
                self.updateControlPanel(status);
            });
        });
    },
    create_model: function() {
        return this._rpc({
                model: 'base_import.import',
                method: 'create',
                args: [{res_model: this.res_model}],
                kwargs: {context: session.user_context},
            });
    },
    renderButtons: function() {
        var self = this;
        this.$buttons = $(QWeb.render("ImportView.buttons", this));
        this.$buttons.filter('.o_import_validate').on('click', this.validate.bind(this));
        this.$buttons.filter('.o_import_import').on('click', this.import.bind(this));
        this.$buttons.filter('.oe_import_file').on('click', function () {
            self.$('.o_content .oe_import_file').click();
        });
        this.$buttons.filter('.o_import_cancel').on('click', function(e) {
            e.preventDefault();
            self.exit();
        });
    },
    setup_encoding_picker: function () {
        this.$('input.oe_import_encoding').select2({
            width: '50%',
            data: _.map(('utf-8 utf-16 windows-1252 latin1 latin2 big5 gb18030 shift_jis windows-1251 koi8_r').split(/\s+/), _make_option),
            query: dataFilteredQuery,
            minimumResultsForSearch: -1,
            initSelection: function ($e, c) {
                return c(_make_option($e.val()));
            }
        });
    },
    setup_separator_picker: function () {
        var data = [
            {id: ',', text: _t("Comma")},
            {id: ';', text: _t("Semicolon")},
            {id: '\t', text: _t("Tab")},
            {id: ' ', text: _t("Space")}
        ];
        this.$('input.oe_import_separator').select2({
            width: '50%',
            data: data,
            query: dataFilteredQuery,
            minimumResultsForSearch: -1,
            // this is not provided to initSelection so can't use this.data
            initSelection: function ($e, c) {
                c(_from_data(data, $e.val()) || _make_option($e.val()))
            }
        });
    },
    setup_float_format_picker: function () {
        var data_decimal = [
            {id: ',', text: _t("Comma")},
            {id: '.', text: _t("Dot")},
        ];
        var data_digits = data_decimal.concat([{id: '', text: _t("No Separator")}]);
        this.$('input.oe_import_float_thousand_separator').select2({
            width: '50%',
            data: data_digits,
            query: dataFilteredQuery,
            minimumResultsForSearch: -1,
            initSelection: function ($e, c) {
                c(_from_data(data_digits, $e.val()) || _make_option($e.val()))
            }
        });
        this.$('input.oe_import_float_decimal_separator').select2({
            width: '50%',
            data: data_decimal,
            query: dataFilteredQuery,
            minimumResultsForSearch: -1,
            initSelection: function ($e, c) {
                c(_from_data(data_decimal, $e.val()) || _make_option($e.val()))
            }
        });
    },
    setup_date_format_picker: function () {
        var data = _([
            'YYYY-MM-DD',
            'DD/MM/YY',
            'DD/MM/YYYY',
            'DD-MM-YYYY',
            'DD-MMM-YY',
            'DD-MMM-YYYY',
            'DD.MM.YY',
            'DD.MM.YYYY',
            'MM/DD/YY',
            'MM/DD/YYYY',
            'MM-DD-YY',
            'MM-DD-YYYY',
            'DDMMYY',
            'DDMMYYYY',
            'YYMMDD',
            'YYYYMMDD',
            'YY/MM/DD',
            'YYYY/MM/DD',
            'MMDDYY',
            'MMDDYYYY',
        ]).map(_make_option);
        this.$('input.oe_import_date_format').select2({
            width: '50%',
            data: data,
            query: dataFilteredQuery,
            minimumResultsForSearch: -1,
            initSelection: function ($e, c) {
                c(_from_data(data, $e.val()) || _make_option($e.val()));
            }
        })
    },
    setup_sheets_picker: function () {
        var data = this.sheets.map(_make_option);
        this.$('input.oe_import_sheet').select2({
            width: '100%',
            data: data,
            query: dataFilteredQuery,
            minimumResultsForSearch: -1,
            initSelection: function ($e, c) {
                c(_from_data(data, $e.val()) || _make_option($e.val()))
            },
        });
    },

    import_options: function () {
        var self = this;
        var options = {
            has_headers: this.$('input.oe_import_has_header').prop('checked'),
            advanced: this.$('input.oe_import_advanced_mode').prop('checked'),
            keep_matches: this.do_not_change_match,
            name_create_enabled_fields: {},
            import_set_empty_fields: [],
            import_skip_records: [],
            fallback_values: {},
            // start at row 1 = skip 0 lines
            skip: Number(this.$('#oe_import_row_start').val()) - 1 || 0,
            limit: Number(this.$('#oe_import_batch_limit').val()) || null,
        };
        _.each(_.union(this.opts, this.spreadsheet_opts), function (opt) {
            options[opt.name] =
                self.$('input.oe_import_' + opt.name).val();
        });
        _(this.parse_opts_formats).each(function (opt) {
            options[opt.name] = time.moment_to_strftime_format(self.$('input.oe_import_' + opt.name).val());
        });
        _(this.parse_opts_separators).each(function (opt) {
            options[opt.name] = self.$('input.oe_import_' + opt.name).val();
        });

        options['fields'] = [];
        if (this.do_not_change_match) {
            options['fields'] = this.$('input.oe_import_match_field').map(function (index, el) {
                return $(el).select2('val') || false;
            }).get();
        }
        this.do_not_change_match = false;
        this.$('select.o_import_create_option').each(function () {
            var field = this.getAttribute('field');
            var type = this.getAttribute('type');
            if (field) {
                if (['boolean', 'many2one', 'many2many', 'selection'].includes(type) && this.value === 'skip_record') {
                    options.import_skip_records.push(field);
                } else if (['many2one', "many2many", "selection"].includes(type)) {
                    if (this.value === 'set_empty') {
                        options.import_set_empty_fields.push(field);
                    } else {
                        options.name_create_enabled_fields[field] = this.value === 'create';
                    }
                // for selection, include also 'skip' that will be interpreted as 'None' in backend.
                } else if (['boolean','selection'].includes(type) && this.value !== 'prevent') {
                    options.fallback_values[field] = {
                        fallback_value: this.value,
                        field_model: this.getAttribute('model'),
                        field_type: type
                    };
                }
            }
        });
        return options;
    },

    //- File & settings change section
    onfile_loaded: function (event, from, to, arg) {
        // arg is null if reload -> don't reset partial import
        if (arg != null ) {
            var savedSkipLines = 0;
            var isPartialEnabled = this.$('.oe_import').hasClass('o_import_partial_mode');
            if (isPartialEnabled) {
                savedSkipLines = this.$('#oe_import_row_start').val();
            }
            this.toggle_partial(null);
            if (isPartialEnabled && savedSkipLines) {
                // If partial mode was already enabled, we want to keep the 'start at line' parameter
                // This can help the end user when he has partially imported a file and wants to make
                // some modifications into it before re-uploading and resuming the upload.
                this.$('#oe_import_row_start').val(savedSkipLines);
            }
        }

        this.$buttons.filter('.o_import_import, .o_import_validate').addClass('d-none');
        if (!this.$('input.oe_import_file').val()) { return this['settings_changed'](); }
        this.$('.oe_import_date_format').select2('val', '');
        this.$('.oe_import_datetime_format').val('');
        this.$('.oe_import_sheet').val('');

        this.$form.removeClass('oe_import_preview oe_import_error');
        var import_toggle = false;
        var file = this.$('input.oe_import_file')[0].files[0];
        // some platforms send text/csv, application/csv, or other things if Excel is prevent
        var isCSV = ((file.type && _.last(file.type.split('/')) === "csv") || ( _.last(file.name.split('.')) === "csv"))
        this.$form.find('.o_import_formatting').toggleClass('d-none', !isCSV);

        // get file name and extension separately, to apply ellipsis to the name and still keep the extension visible.
        // E.g. : superLongFileName.csv -> "superLongFi....csv" instead of "superLongFileNa..."
        var fileName = file.name.split('.');
        var fileExtension = fileName.pop();
        fileName = fileName.join('.');
        this.$('#oe_imported_file').text(fileName);
        this.$('#oe_imported_file_extension').text('.' + fileExtension);

        this.$form.find('.oe_import_box').toggle(true);
        jsonp(this.$form, {
            url: '/base_import/set_file'
        }, this.proxy('settings_changed'));
    },
    onpreviewing: function () {
        var self = this;
        this.$buttons.filter('.o_import_import, .o_import_validate').addClass('d-none');
        this.$form.addClass('oe_import_with_file');
        // TODO: test that write // succeeded?
        this.$form.removeClass('oe_import_preview_error oe_import_error');
        this.$form.toggleClass(
            'oe_import_noheaders',
            !this.$('input.oe_import_has_header').prop('checked'));

        // Clear the input value to allow onchange to be triggered
        // if the file is the same (for all browsers)
        this.$('input.oe_import_file').val('');
        this.$('.oe_import_options_cell,.oe_import_options_header').addClass('d-none');

        this._cleanComments();

        // Block UI during loading file.
        $.blockUI({message: QWeb.render('Throbber')});
        $(document.body).addClass('o_ui_blocked');
        $('.oe_throbber_message').text(_t("Loading file..."));

        this._rpc({
                model: 'base_import.import',
                method: 'parse_preview',
                args: [this.id, this.import_options()],
                kwargs: {context: session.user_context},
            }).then(function (result) {
                var signal = result.error ? 'preview_failed' : 'preview_succeeded';
                self[signal](result);
                $(document.body).removeClass('o_ui_blocked');
                $.unblockUI();
            });
    },
    onpreview_error: function (event, from, to, result) {
        this.$('.oe_import_options').show();
        this.$form.addClass('oe_import_preview_error oe_import_error');
        this.$form.find('.oe_import_box, .oe_import_with_file').removeClass('d-none');
        this.$form.find('.o_view_nocontent').addClass('d-none');
        this.$('.oe_import_error_report').html(
                QWeb.render('ImportView.preview.error', result));
    },
    onpreview_success: function (event, from, to, result) {
        var self = this;
        this.$buttons.filter('.oe_import_file')
            .text(_t('Load File'))
            .removeClass('btn-primary').addClass('btn-secondary')
            .blur();
        this.$buttons.filter('.o_import_import, .o_import_validate').removeClass('d-none');
        this.$form.find('.oe_import_box, .oe_import_with_file').removeClass('d-none');
        this.$form.find('.o_view_nocontent').addClass('d-none');
        this.$form.addClass('oe_import_preview');
        this.$('input.oe_import_advanced_mode').prop('checked', result.advanced_mode);
        this.$('.oe_import_grid').html(QWeb.render('ImportView.preview', result));
        this.$('.oe_import_grid .o_import_preview').each((index, element) => {
            $(element).popover({
                title: _t("Preview"),
                trigger: 'hover',
                html: true,
                content: QWeb.render('ImportView.preview_popover', { preview: result.preview[index] }),
            });
        });
        // Activate the batch configuration panel only of the file length > 100. (In order to let the user choose
        // the batch size even for medium size file. Could be useful to reduce the batch size for complex models).
        this.fileLength = result.file_length;
        this.$('.o_import_batch').toggleClass('d-none', !(this.fileLength > 100));
        this.$('.o_import_batch_alert').toggleClass('d-none', !result.batch);

        var messages = [];
        if (result.headers.length === 1) {
            messages.push({type: 'warning', message: _t("A single column was found in the file, this often means the file separator is incorrect")});
        }

        if (!_.isEmpty(messages)) {
            this.$('.oe_import_options').show();
            this.onresults(null, null, null, {'messages': messages});
        }

        if (!_.isEqual(this.sheets, result.options.sheets)) {
            this.sheets = result.options.sheets || [];
            this.setup_sheets_picker();
        }
        this.$('div.oe_import_has_multiple_sheets').toggle(
            this.sheets.length > 1
        );

        // merge option values back in case they were updated/guessed
        _.each(['encoding', 'separator', 'float_thousand_separator', 'float_decimal_separator', 'sheet'], function (id) {
            self.$('.oe_import_' + id).select2('val', result.options[id])
        });
        this.$('.oe_import_date_format').select2('val', time.strftime_to_moment_format(result.options.date_format));
        this.$('.oe_import_datetime_format').val(time.strftime_to_moment_format(result.options.datetime_format));
        // hide all "true debug" options when not in debug mode
        this.$('.oe_import_debug_option').toggleClass('d-none', !result.debug);

        var $fields = this.$('.oe_import_match_field');
        this.render_fields_matches(result, $fields);
        var data = this._generate_fields_completion(result);
        var item_finder = function (id, items) {
            items = items || data;
            for (var i=0; i < items.length; ++i) {
                var item = items[i];
                if (item.id === id) {
                    return item;
                }
                var val;
                if (item.children && (val = item_finder(id, item.children))) {
                    return val;
                }
            }
            return '';
        };
        $fields.each(function (k,v) {
            var $fieldInput = $(v);
            var filteredData = self._generate_fields_completion(result, k);

            var updateFieldInformation = function (fieldInfo) {
                // Get the comment cell on the same row in the table.
                var $commentCell = $fieldInput.closest('tr.oe_import_grid-row').find('.oe_import_comment_cell');
                var $optionsDiv = $commentCell.find('.oe_import_options_div');
                var isRelational = fieldInfo.type === 'many2many' || fieldInfo.type === 'many2one';
                var setEmpty = !fieldInfo.required && (isRelational || fieldInfo.type === 'selection');

                // if relational field is child of one2many then do not display skip record option
                var isO2MField = false;
                if (isRelational && fieldInfo.id.includes('/')) {
                    var fieldName = fieldInfo.id.split('/')[0];
                    var field = result.fields.find((field) => field.name === fieldName);
                    if (field && field.type === 'one2many') {
                        isO2MField = true;
                    }
                }
                var showSkipRecord = setEmpty && !isO2MField || (!fieldInfo.required && fieldInfo.type === 'boolean');

                // get options cell related to that field
                if (isRelational || fieldInfo.type === 'boolean') {
                    $optionsDiv.empty();
                    $optionsDiv.append(
                        QWeb.render('ImportView.create_record_option', {
                            data: fieldInfo,
                            is_relational: isRelational,
                            set_empty: setEmpty,
                            show_skip_record: showSkipRecord
                        })
                    );
                } else if (fieldInfo.type === 'selection') {
                    self._rpc({
                        model: fieldInfo.comodel_name || fieldInfo.model_name,
                        method: 'fields_get',
                    }).then(function (values) {
                        var selectionField = fieldInfo.id.split('/').pop();
                        var selectionLabels = values[selectionField]["selection"].map(value => value[1]);

                        $optionsDiv.empty();
                        $optionsDiv.append(
                            QWeb.render('ImportView.create_record_option', {
                                data: fieldInfo,
                                values: selectionLabels,
                                set_empty: setEmpty,
                                show_skip_record: showSkipRecord
                            })
                        );
                        self.selectionFields[data.id] = values;
                    });
                } else if ($optionsDiv.find('.o_import_create_option').length == 1) {
                    $optionsDiv.empty();
                }

                // re-attribute comment cell to the new selected field
                $commentCell.attr('field', fieldInfo.id || "");
                $commentCell.attr('model', fieldInfo.comodel_name || fieldInfo.model_name || "");

                // assign class for field icon
                var $fieldDropdown = $fieldInput.closest('.oe_import_match_cell').find('.select2-choice');
                var oldType = $fieldDropdown.getAttributes()['type'];
                $fieldDropdown.removeClass(`o_import_field_${oldType}`).addClass(`o_import_field_icon o_import_field_${fieldInfo.type}`);
                $fieldDropdown.attr('type', fieldInfo.type);
            };

            var default_value = $fieldInput.val();
            var fieldInfo = item_finder(default_value);
            if (!fieldInfo) {
                $fieldInput.val('');
            }

            $fieldInput.select2({
                allowClear: true,
                minimumInputLength: 0,
                data: filteredData,
                initSelection: function (element, callback) {
                    if (!default_value) {
                        callback('');
                        return;
                    }

                    updateFieldInformation(fieldInfo);
                    callback(fieldInfo);
                    self._handleMappingComments(v, fieldInfo);
                },
                // Format the tooltip.
                formatSelection: function (object, container) {
                    var fieldTooltipString = `%(label)s: %(labelValue)s
%(name)s: %(nameValue)s
%(type)s: %(typeValue)s
%(model)s: %(modelValue)s`;
                    var tooltip = _.str.sprintf(fieldTooltipString, {
                        'label': _t("Label"), 'labelValue': object.text,
                        'name': _t("Name"), 'nameValue': object.id,
                        'type': _t("Type"), 'typeValue': object.type,
                        'model': _t("Model"), 'modelValue': object.comodel_name || object.model_name,
                    });
                    $(container[0]).closest('a').attr('title', tooltip);
                    return object.text;
                },
                // For each choice, if field is required, make it bold in the list
                formatResultCssClass: function (object) {
                    if (object.required) { return "font-weight-bold text-decoration-underline"; }
                    return "";
                },
                placeholder: _t('To import, select a field...'),
                width: 'resolve',
                dropdownCssClass: 'oe_import_selector'
            }).on('change', function (event) {
                var changedField = event.currentTarget;
                var fieldRemovedId = event.removed ? event.removed.id : false;
                self._cleanFieldComments(changedField, fieldRemovedId);

                var fieldInfo = item_finder(changedField.value);
                updateFieldInformation(fieldInfo);
                self._handleMappingComments(changedField, fieldInfo);

                if (!event.val) {
                    var $matchingCell = $(changedField).closest('.oe_import_match_cell')
                    $matchingCell.find('.o_import_field_icon').removeClass('o_import_field_icon');
                    $matchingCell.find('a.select2-choice').removeAttr("title");
                }
            });

            $fieldInput.closest('.oe_import_match_cell').find('.select2-input').attr('placeholder', _t('Search for a field...'));
        });
    },

    /**
    * This method is called when changing the field to map with a file column, or when removing it.
    * It will clean the eventual comments, errors and the mapping options related to the previously selected field.
    *
    * @private
    */
    _cleanFieldComments: function (changedField, fieldRemovedId) {
        // Check that the column was not mapped to same field than another column
        if (fieldRemovedId) {
            var $sameMappedFields = this.$(`.oe_import_comment_cell[field=\"${fieldRemovedId}\"]`).find('.oe_import_same_mapped_field');
            if ($sameMappedFields.length == 2) {
                // remove all same mapped field comments
                $sameMappedFields.remove();
            }
        }
        // remove all comments for this header-field mapping row
        var $fieldRow = $(changedField).closest('tr.oe_import_grid-row');
        $fieldRow.find('.oe_import_comments_div').empty();
        $fieldRow.find('.oe_import_options_div').addClass("d-none");
    },

    /**
    * This method is called when selecting a sheet or a new file to import, but also when running to import or the
    * import test. It will remove all the general comments and/or field specific warnings and errors
    *
    * @private
    */
    _cleanComments: function () {
        this.$('.oe_import_error_report').empty();
        this.$('.oe_import_comments_div').find('.alert-error,.alert-warning').remove();
        this.$form.removeClass('oe_import_error');
    },

    /**
     * Called at the start of every batch to update the progress bar display.
     * It also computes an estimated time left based on: starting time of the whole process,
     * percentage done so far and remaining records.
     */
    _onBatchStart: function () {
        var recordsDone = this.batchSize * (this.currentBatchNumber - 1);
        var percentage = parseInt(recordsDone / this.totalToImport * 100);
        $('.o_import_progress_dialog')
            .find('.progress-bar')
            .text(percentage + "%")
            .attr('aria-valuenow', percentage)
            .css('width', percentage + '%');
        $('.o_import_progress_dialog')
            .find('.o_import_progress_dialog_batch_count')
            .text(this.currentBatchNumber);
        if (percentage !== 0) {
            // e.g: it took 1 seconds (1000 millis) to import 33%
            // -> there is (1000) * ((100 - 33) / 33) / 60000 minutes left
            // -> 1000 * (66 / 33) / 60000 -> 2000 / 60000 -> 0.03 minute left (2 seconds) left
            var estimatedTimeLeftMinutes = ((Date.now() - this.importStartTime) * ((100 - percentage) / percentage)) / 60000;
            $('.o_import_progress_dialog_time_left').removeClass('d-none');
            $('.o_import_progress_dialog_time_left_text')
                .text(fieldUtils.format.float_time(estimatedTimeLeftMinutes));
        }
    },

    /**
     * Called when the user manually interrupts the import during a batched import.
     * Sets 'stopImport' to true, which will stop the process when the current batch is done.
     *
     * @private
     * @param {MouseEvent} event
     */
    _onStopImport: function (event) {
        var $currentTarget = $(event.currentTarget);
        $currentTarget.enable(false);
        $currentTarget
            .closest('.o_import_progress_dialog')
            .find('.o_import_progress_dialog_stop, .o_import_progress_dialog_batch')
            .toggleClass('d-none');
        this.stopImport = true;
    },

   /**
    * This method is called when selecting a new mapping field, or when changing/removing an already selected mapping
    * field. It will check if multiple field columns are/were mapped on the same field.
    * 1. In case the field type is char or text:
    *   This method will display / remove the comments informing the user that two columns content will be
    *   concatenated on the field they have chosen.
    * 2. Else:
    *  This method will remove the field mapping from the old column, as each field, except for the case 1., can only
    *  be mapped once.
    *
    * @private
    */
    _handleMappingComments: function (changedField, fieldInfo) {
        // check if two columns are mapped on the same fields (for char/text fields)
        var commentsToAdd = [];
        var $sameMappedFields = this.$('.oe_import_comment_cell[field="' + fieldInfo.id + '"]');

        if (fieldInfo.type == 'many2many') {
            commentsToAdd.push(QWeb.render('ImportView.comment_m2m_field'));
        }
        if ($sameMappedFields.length >= 2) {
            if (['char', 'text', "many2many"].includes(fieldInfo.type)) {
                commentsToAdd.push(QWeb.render('ImportView.comment_same_mapped_field', {
                    field: fieldInfo.text,
                }));
            } else {  // if column is mapped on an already mapped field, remove that field from the old column.
                var $targetMappedFieldId = $(changedField).parent().find('div.oe_import_match_field').getAttributes()['id'];
                _.each($sameMappedFields, function(fieldComment) {
                    var $mappingCell = $(fieldComment).parent().find('div.oe_import_match_field');
                    if ($mappingCell.getAttributes()['id'] !== $targetMappedFieldId) {
                        $mappingCell.find('.select2-search-choice-close').trigger('mousedown').trigger('click');
                    }
                });
            }
        }

        var $commentDiv = $sameMappedFields.find(".oe_import_comments_div");
        $commentDiv.empty();
        _.each($commentDiv, function(fieldComment) {
            _.each(commentsToAdd, function(comment) {
                $(fieldComment).append(comment);
            });
        });
    },

    _generate_fields_completion: function (root, index) {
        var self = this;
        var basic = [];
        var regulars = [];
        var o2m = [];
        var suggested = [];
        var header_types = root.header_types;
        function traverse(field, ancestors, collection, type) {
            var subfields = field.fields;
            var advanced_mode = self.$('input.oe_import_advanced_mode').prop('checked');
            var field_path = ancestors.concat(field);
            var label = _(field_path).pluck('string').join(' / ');
            var id = _(field_path).pluck('name').join('/');
            // If non-relational, m2o or m2m, collection is either suggested if field type is in header_types, either regulars if type does not match
            if (!collection) {
                if (field.name === 'id') {
                    collection = basic;
                } else if (_.isEmpty(subfields)
                        || _.isEqual(_.pluck(subfields, 'name'), ['id', '.id'])) {
                    collection = (type && type.indexOf(field['type']) !== -1) ? suggested : regulars;
                } else {
                    collection = o2m;
                }
            }

            collection.push({
                id: id,
                text: label,
                required: field.required,
                type: field.type,
                default_value: field.default_value,
                comodel_name: field.comodel_name,
                model_name: field.model_name,
            });

            if (advanced_mode){
                for(var i=0, end=subfields.length; i<end; ++i) {
                    traverse(subfields[i], field_path, collection, type);
                }
            }
        }
        _(root.fields).each(function (field) {
            if (index === undefined) {
                traverse(field, []);
            }
            else {
                if (self.$('input.oe_import_advanced_mode').prop('checked')){
                    traverse(field, [], undefined, ['all']);
                }
                else {
                    traverse(field, [], undefined, header_types[index]);
                }
            }
        });

        var cmp = function (field1, field2) {
            return field1.text.localeCompare(field2.text);

        };
        suggested.sort(cmp);
        regulars.sort(cmp);
        o2m.sort(cmp);
        if (!_.isEmpty(regulars) && !_.isEmpty(o2m)){
            if (!_.isEmpty(suggested)) {
                basic = basic.concat({ text: _t("Suggested Fields"), children: suggested });
            }
            return basic.concat([
                { text: !_.isEmpty(suggested) ? _t("Additional Fields") : _t("Standard Fields"), children: regulars },
                { text: _t("Relation Fields"), children: o2m },
            ]);
        } else {
            return basic.concat(suggested, regulars, o2m);
        }
    },
    render_fields_matches: function (result, $fields) {
        if (_(result.matches).isEmpty()) { return; }
        $fields.each(function (index, input) {
            var match = result.matches[index];
            if (!match) { return; }

            var current_field = result;
            input.value = _(match).chain()
                .map(function (name) {
                    // WARNING: does both mapping and folding (over the
                    //          ``field`` iterator variable)
                    return current_field = _(current_field.fields).find(function (subfield) {
                        return subfield.name === name;
                    });
                })
                .pluck('name')
                .value()
                .join('/');
        });
    },

    //- import itself
    call_import: function (kwargs) {
        var fields = this.$('input.oe_import_match_field').map(function (index, el) {
            return $(el).select2('val') || false;
        }).get();
        var columns = this.$('.o_import_header_name').map(function () {
            return $(this).text().trim().toLowerCase() || false;
        }).get();

        var tracking_disable = 'tracking_disable' in kwargs ? kwargs.tracking_disable : !this.$('#oe_import_tracking').prop('checked');
        delete kwargs.tracking_disable;
        kwargs.context = _.extend(
            {}, this.parent_context,
            {tracking_disable: tracking_disable}
        );

        this.importStartTime = Date.now();
        this.stopImport = false;
        this.totalToImport = this.fileLength - parseInt(this.$('#oe_import_row_start').val());
        this.batchSize = parseInt(this.$('#oe_import_batch_limit').val() || 0);
        var isBatch = this.batchSize !== 0 && this.totalToImport > this.batchSize;
        var totalSteps = isBatch ? Math.floor(this.totalToImport / this.batchSize) + 1 : 1;
        this.currentBatchNumber = 1;

        $.blockUI({
            message: QWeb.render(
                'base_import.progressDialog', {
                    importMode: kwargs.dryrun ? _t('Testing') : _t('Importing'),
                    isBatch: isBatch,
                    totalSteps: totalSteps,
                }
            )
        });
        $(document.body).addClass('o_ui_blocked');

        $('.o_import_progress_dialog')
            .find('.o_progress_stop_import')
            .on('click', this._onStopImport.bind(this));

        var opts = this.import_options();

        return this._batchedImport(opts, [this.id, fields, columns], kwargs, {done: 0, prev: 0})
            .then(null, function (reason) {
                var error = reason.message;
                var event = reason.event;
                // In case of unexpected exception, convert
                // "JSON-RPC error" to an import failure, and
                // prevent default handling (warning dialog)
                if (event) { event.preventDefault(); }

                var errordata = error.data || {};
                const msg = errordata.arguments && (errordata.arguments[1] || errordata.arguments[0])
                    || error.message || _t("An unknown issue occurred during import (possibly lost connection, data limit exceeded or memory limits exceeded). Please retry in case the issue is transient. If the issue still occurs, try to split the file rather than import it at once.");

                return Promise.resolve({'messages': [{
                    type: 'error',
                    record: false,
                    message: msg,
                }]});
            }).finally(function () {
                $(document.body).removeClass('o_ui_blocked');
                $.unblockUI();
            });
    }, /**
     *
     * @param opts import options
     * @param args positional arguments to pass along (augmented with the options)
     * @param kwargs keyword arguments to pass along (directly)
     * @param {Object} rec recursion information record
     * @param {Number} rec.done how many records have been loaded so far
     * @param {Number} rec.prev nextrow of the previous call so we can know
     *                          how many rows the call we're here performing
     *                          will have consumed, and thus by how much we
     *                          need to offset the messages of the *next* call
     * @returns {Promise<{name, ids, messages}>}
     * @private
     */
    _batchedImport: function (opts, args, kwargs, rec) {
        var self = this;
        opts.callback && opts.callback(this);
        this._onBatchStart();
        this.currentBatchNumber += 1;

        if (this.stopImport) {
            $(document.body).removeClass('o_ui_blocked');
            $.unblockUI();
            return Promise.resolve({});
        }

        return this._rpc({
            model: 'base_import.import',
            method: 'execute_import',
            args: args.concat([opts]),
            kwargs: kwargs
        }, {
            shadow: true,
        }).then(function (results) {
            _.each(results.messages, offset_by(opts.skip));
            if (!kwargs.dryrun && !results.ids) {
                // update skip to failed batch
                self.$('#oe_import_row_start').val(opts.skip + 1);
                if (opts.skip) {
                    // there's been an error during a "proper" import, stop & warn
                    // about partial import maybe
                    results.messages.push({
                        type: 'info',
                        priority: true,
                        message: _.str.sprintf(_t("This file has been successfully imported up to line %d."), opts.skip)
                    });
                }
                return results;
            }
            if (!results.nextrow) {
                // we're done
                return results;
            }

            // do the next batch
            return self._batchedImport(
                // avoid modifying opts in-place
                _.defaults({skip: results.nextrow}, opts),
                args, kwargs, {
                    done: rec.done + (results.ids || []).length,
                    prev: results.nextrow
                }
            ).then(function (r2) {
                return {
                    name: _.zip(results.name, r2.name).map(function (names) {
                        return names[0] || names[1];
                    }),
                    ids: (results.ids || []).concat(r2.ids || []),
                    messages: r2.messages ? results.messages.concat(r2.messages) : results.messages,
                    skip: r2.skip || results.nextrow,
                    nextrow: r2.nextrow
                }
            });
        });
    },
    onvalidate: function () {
        var prom = this.call_import({ dryrun: true, tracking_disable: true });
        prom.then(this.proxy('validated'));
        return prom;
    },
    onimport: function () {
        var self = this;
        var prom = this.call_import({ dryrun: false });
        prom.then(function (results) {
            if (self.stopImport) {
                var recordsImported = results.ids ? results.ids.length : 0;
                if (recordsImported) {
                    self.$('#oe_import_row_start').val((results.skip || 0) + 1);
                    self.displayNotification({ message: _.str.sprintf(
                        _t("%d records successfully imported"),
                        recordsImported
                    )});
                }
                self['import_interrupted'](results);
            } else if (!_.any(results.messages, function (message) {
                    return message.type === 'error'; })) {
                self['import_succeeded'](results);
                return;
            }
            self['import_failed'](results);
        });
        return prom;
    },
    onimported: function (event, from, to, results) {
        this.displayNotification({ message: _.str.sprintf(
            _t("%d records successfully imported"),
            results.ids.length
        ) });
        this.exit();
    },
    exit: function () {
        this.trigger_up('history_back');
    },
    onresults: function (event, from, to, results) {
        var self = this;
        var fields = this.$('input.oe_import_match_field').map(function (index, el) {
            return $(el).select2('val') || false;
        }).get();

        var error_type = "warning";
        var errorMessages = results.messages;

        if (_.isEmpty(errorMessages) && event !== 'import_interrupted') {
            errorMessages.push({
                type: 'info',
                message: _t("Everything seems valid.")
            });
            error_type = false;
        } else if (event === 'import_interrupted' && results.ids) {
            this.toggle_partial(results);
            error_type = false;
        } else if (event === 'import_failed' && results.ids) {
            // both ids in a failed import -> partial import
            this.toggle_partial(results);
        }

        // row indexes come back 0-indexed, spreadsheets
        // display 1-indexed.
        var offset = 1;
        // offset more if header
        if (this.import_options().has_headers) { offset += 1; }

        // Sort errors by error gravity then by field
        var errorsSorted = _.sortBy(_(errorMessages).groupBy('message'), function (group) {
            if (group[0].priority){
                return -2;
            }

            // sort by gravity, then, order of field in list
            var order = 0;
            switch (group[0].type) {
            case 'error': order = 0; error_type = 'error'; break;
            case 'warning': order = fields.length + 1; break;
            case 'info': order = 2 * (fields.length + 1); break;
            default: order = 3 * (fields.length + 1); break;
            }
            return order + _.indexOf(fields, group[0].field);
        });

        // regroup errors by field
        var errorsByFields = this._regroupErrorsByFields(errorsSorted);

        // If no general comments (index "0"), add general warning that there are some errors/warnings
        if (!errorsByFields[0] && error_type) {
            var message = error_type === "warning" ?
                _t("The file contains non-blocking warnings (see below)") :
                _t("The file contains blocking errors (see below)");
            errorsByFields[0] = [{
                'type': error_type,
                'message': message,
            }];
        }

        // Clean old comments
        self._cleanComments();

        // Add new comments
        _.each(errorsByFields, function (errors, field) {
            var $errorCell = self.$('.oe_import_comment_cell[field="'+ field +'"]');
            var $errorDiv = $errorCell.find(".oe_import_comments_div");
            var errorTemplate = 'ImportView.fieldError';
            // If error not linked to a targeted Odoo field, show in global error div
            if ($errorDiv.length === 0) {
                $errorDiv = self.$('.oe_import_error_report');
                self.$form.addClass('oe_import_error');
                field = 0;
                errorTemplate = 'ImportView.error';
            }
            $errorDiv.append(
                QWeb.render(errorTemplate, {
                    errors: errors,
                    field: field,
                    result_names: results.name,
                    offset: offset,
                })
            );
            var mainError = errors[0];
            if (!mainError.not_matching_error && mainError.type === "error") {
                $errorCell.find(".oe_import_options_div").removeClass('d-none');
            }
        });
    },

    /**
    * This method regroups the errors by their 'field' key.
    * Errors of same nature encountered on multiple rows for the same field are regrouped into the same unique error
    * and rows_from and rows_to are adapted accordingly.
    * The goal is to ease the error attribution by looping through each mapped field and add their corresponding errors
    * to the comment cell of that mapped field.
    * Some errors are not linked to a mapped field and will be displayed in the general error container.
    * Those general errors will be identified with key "0".
    *
    * @private
    * @param {Array} errorsSorted: list of errors sorted by error gravity, then by field.
    *   Each item of the list is a list of error of the same nature and can be therefore composed of 1 or more errors.
    *   E.g.: [
    *       [
    *           {
    *               'type': 'error',
    *               'field_path': 'product_id/price',
    *               'rows': {'from': 1, 'to': 1}, // This error have been encountered between row 1 and row 1 (at row 1)
    *               'message': {"Missing Value"}
    *           },
    *           {
    *               'type': 'error',
    *               'field_path': 'product_id/price',
    *               'rows': {'from': 2, 'to': 2}, // This error have been encountered between row 2 and row 2 (at row 2)
    *               'message': {"Missing Value"}
    *           }
    *       ],
    *       [ // this one is a general error, not linked specifically to a mapped field.
    *           {
    *               'type': 'error',
    *               'message': {"Found more than 10 errors and more than one error per 10 records, interrupted to avoid showing too many errors."}
    *           }
    *       ],
    *       [
    *           {
    *               'type': 'warning',
    *               'field': 'quantity',
    *               'rows': {'from': 2, 'to': 4}, // This error have been encountered between row 2 and row 4 (at 3 different rows)
    *               'message': {}
    *           }
    *       ],
    *   ]
    * @returns {Dict} errorsByFields: {fieldName: [error1, error2, ...]}
    *   E.g.: {
    *       0: [
    *           {
    *               'type': 'error',
    *               'message': {"Found more than 10 errors and more than one error per 10 records, interrupted to avoid showing too many errors."}
    *           }
    *       ],
    *       'product_id/price': [
    *           {
    *               'type': 'error',
    *               'field_path': 'product_id/price',
    *               'rows': {'from': 1, 'to': 2}, // This error have been encountered between row 1 and row 2
    *               'message': {"Missing Value"}
    *           }
    *       ],
    *       'quantity': [
    *           {
    *               'type': 'warning',
    *               'field': 'quantity',
    *               'rows': {'from': 2, 'to': 4}, // This error have been encountered between row 2 and row 4
    *               'message': {}
    *           }
    *       ],
    *   }
    */
    _regroupErrorsByFields: function (errorsSorted) {
        var errorsByFields = {}
        _.each(errorsSorted, function (errors) {
            var mainError = errors[0];
            var field = mainError.field_path ? mainError.field_path.join('/') : mainError.field || 0;

            // regroup errors for same value at different rows in the same error.
            if (errors.length > 1 && mainError.rows) {
                var rowFrom = mainError.rows.from;
                var rowTo = errors[errors.length - 1].rows.to;
                errors = mainError;
                mainError.rows.from = rowFrom;
                mainError.rows.to = rowTo;
            } else {
                errors = mainError;
            }

            if (!errorsByFields[field]) {
                errorsByFields[field] = [errors];
            } else {
                errorsByFields[field].push(errors);
            }
        });

        return errorsByFields;
    },

    toggle_partial: function (result) {
        var $form = this.$('.oe_import');
        var $partial_warning = this.$('.o_import_partial_alert');
        var $partial_count = this.$('.o_import_partial_count');
        if (result == null) {
            $partial_warning.addClass('d-none');
            $form.add(this.$buttons).removeClass('o_import_partial_mode');
            var $skip = this.$('#oe_import_row_start');
            $skip.val($skip.attr('value'));
            $partial_count.text('');
            return;
        }

        this.$('.o_import_batch_alert').addClass('d-none');
        $partial_warning.removeClass('d-none');
        $form.add(this.$buttons).addClass('o_import_partial_mode');
        $partial_count.text((result.skip || 0) + 1);
    }
});
core.action_registry.add('import', DataImport);

// FSM-ize DataImport
StateMachine.create({
    target: DataImport.prototype,
    events: [
        { name: 'loaded_file',
          from: ['none', 'file_loaded', 'preview_error', 'preview_success', 'results', 'imported'],
          to: 'file_loaded' },
        { name: 'settings_changed',
          from: ['file_loaded', 'preview_error', 'preview_success', 'results'],
          to: 'previewing' },
        { name: 'preview_failed', from: 'previewing', to: 'preview_error' },
        { name: 'preview_succeeded', from: 'previewing', to: 'preview_success' },
        { name: 'validate', from: 'preview_success', to: 'validating' },
        { name: 'validate', from: 'results', to: 'validating' },
        { name: 'validated', from: 'validating', to: 'results' },
        { name: 'import', from: ['preview_success', 'results'], to: 'importing' },
        { name: 'import_succeeded', from: 'importing', to: 'imported'},
        { name: 'import_interrupted', from: 'importing', to: 'results' },
        { name: 'import_failed', from: 'importing', to: 'results' }
    ],
});

function offset_by(by) {
    return function offset_message(msg) {
        if (msg.rows) {
            msg.rows.from += by;
            msg.rows.to += by;
        }
    }
}

return {
    DataImport: DataImport,
};

});

```

## File: static\src\legacy\js\import_menu.js

```javascript
odoo.define('base_import.ImportMenu', function (require) {
    "use strict";

    const FavoriteMenu = require('web.FavoriteMenu');
    const { useModel } = require('web.Model');

    const { Component } = owl;

    /**
     * Import Records menu
     *
     * This component is used to import the records for particular model.
     */
    class ImportMenu extends Component {
        constructor() {
            super(...arguments);
            this.model = useModel('searchModel');
        }

        //---------------------------------------------------------------------
        // Handlers
        //---------------------------------------------------------------------

        /**
         * @private
         */
        importRecords() {
            const action = {
                type: 'ir.actions.client',
                tag: 'import',
                params: {
                    model: this.model.config.modelName,
                    context: this.model.config.context,
                }
            };
            this.trigger('do-action', {action: action});
        }

        //---------------------------------------------------------------------
        // Static
        //---------------------------------------------------------------------

        /**
         * @param {Object} env
         * @returns {boolean}
         */
        static shouldBeDisplayed(env) {
            return env.view &&
                ['kanban', 'list'].includes(env.view.type) &&
                env.action.type === 'ir.actions.act_window' &&
                !env.device.isMobile &&
                !!JSON.parse(env.view.arch.attrs.import || '1') &&
                !!JSON.parse(env.view.arch.attrs.create || '1');
        }
    }

    ImportMenu.props = {};
    ImportMenu.template = "base_import.ImportRecords";

    FavoriteMenu.registry.add('import-menu', ImportMenu, 1);

    return ImportMenu;
});

```

## File: static\src\legacy\xml\base_import.xml

```xml
<templates>
    <t t-name="ImportView">
        <t t-set="_id" t-value="_.uniqueId('export')"/>
        <form action="" method="post" enctype="multipart/form-data" class="oe_import">
            <input type="hidden" name="csrf_token" t-att-value="csrf_token"/>
            <input type="hidden" name="import_id"/>

            <div class="d-flex h-100">
                <t t-call="ImportView.side_panel"/>
                <t t-call="ImportView.data_matching"/>
            </div>

            <div class="o_view_nocontent">
                <div class="o_nocontent_help">
                    <p class="o_view_nocontent_smiling_face">
                        Upload an Excel or CSV file to import
                    </p>
                    <p>
                        Excel files are recommended as formatting is automatic.
                    </p>
                    <div class="mt16 mb4">Need Help?</div>
                    <div t-foreach="widget.importTemplates" t-as="template">
                        <a t-att-href="template.template" aria-label="Download" title="Download">
                            <i class="fa fa-download"/> <span><t t-esc="template.label"/></span>
                        </a>
                    </div>
                    <a href="https://www.odoo.com/documentation/15.0/applications/general/export_import_data.html" target="new">Import FAQ</a>
                </div>
            </div>
        </form>
    </t>

    <t t-name="ImportView.buttons">
        <button type="button" class="btn btn-primary o_import_import o_import_import_full d-none">Import</button>
        <button type="button" class="btn btn-primary o_import_import o_import_import_partial d-none">Resume</button>
        <button type="button" class="btn btn-secondary o_import_validate d-none">Test</button>
        <button type="button" class="btn btn-primary oe_import_file">Upload File</button>
        <button type="button" class="btn btn-secondary o_import_cancel">Cancel</button>
    </t>

    <t t-name="ImportView.create_record_option">
        <div t-attf-class="oe_import_dropdown o_import_field_#{data.type} position-relative alert alert-error">
            <b>When a value cannot be matched:</b>
            <select class="o_import_create_option" t-att-type="data.type" t-att-field="data.id"
                    t-att-model="data.comodel_name || data.model_name">
                <option value="prevent">Prevent import</option>
                <t t-if="data.type === 'boolean'">
                    <option value="false">Set to: False</option>
                    <option value="true" >Set to: True</option>
                </t>
                <option t-if="set_empty" value="set_empty">Set value as empty</option>
                <option t-if="show_skip_record" value="skip_record">Skip record</option>
                <option t-if="is_relational" value="create">Create new values</option>
                <t t-if="data.type === 'selection'">
                    <option t-foreach="values" t-as="value" t-att-value="value">Set to: <t t-esc="value"/></option>
                </t>
            </select>
        </div>
    </t>

    <t t-name="ImportView.preview">
        <thead>
            <tr class="oe_import_grid-header">
                <td class="oe_import_grid-cell border-0 font-weight-bold" style="width: 20%;">
                    <span class="o_single_line_text" title="File Column">File Column</span></td>
                <td class="oe_import_grid-cell border-0 font-weight-bold" style="width: 15%;" >
                    <span class="o_single_line_text" title="File Column">Odoo Field</span></td>
                <td class="oe_import_grid-cell border-0 font-weight-bold">
                    <span class="o_single_line_text" title="File Column">Comments</span></td>
            </tr>
        </thead>
        <tbody>
            <t t-set="columns" t-value="headers || preview"/>
            <tr t-if="columns" t-foreach="columns" t-as="column" class="oe_import_grid-row">
                <td class="oe_import_grid-cell oe_import_name_cell">
                    <span class="o_import_header_name o_single_line_text font-weight-bold" t-att-title="column">
                        <span t-if="column" t-esc="column"/>
                        <span t-else="">Untitled</span>
                    </span>
                    <span t-if="options['has_headers']" class="font-italic small o_import_preview">
                        <t t-esc="preview[column_index][0]"/>
                    </span>
                </td>
                <td class="oe_import_grid-cell oe_import_match_cell">
                    <input class="oe_import_match_field oe_import_dropdown"/>
                </td>
                <td class="oe_import_grid-cell oe_import_comment_cell">
                    <div class="oe_import_comments_div"/>
                    <div class="oe_import_options_div d-none"/>
                </td>
            </tr>
        </tbody>
    </t>

    <t t-name="ImportView.comment_same_mapped_field">
        <div class="oe_import_same_mapped_field alert alert-info">
            This column will be concatenated in field <b t-esc="field"/>
        </div>
    </t>

    <t t-name="ImportView.comment_m2m_field">
        <div class="oe_import_m2m_field alert alert-info">
            To import multiple values, separate them by a comma
        </div>
    </t>

    <t t-name="ImportView.preview_popover">
        <div class="o_import_preview_popover" style="position: relative">
            <div t-foreach="preview" t-as="preview_value" t-esc="preview_value"/>
        </div>
    </t>

    <t t-name="ImportView.preview.error">
        <div class="oe_import_report alert alert-danger">
            <p>Import preview failed due to: <t t-esc="error"/>.</p>
            <p>For CSV files, you may need to select the correct separator.</p>
            <p t-if="preview">Here is the start of the file we could not import:</p>
        </div>
        <pre t-if="preview"><t t-esc="preview"/></pre>
    </t>

    <t t-name="ImportView.error">
        <div t-foreach="errors" t-as="error" t-attf-class="oe_import_report alert alert-#{error.type}">
            <span class="oe_import_report_message">
                <b><t t-esc="error.message"/></b>
                <t t-if="error.rows" t-call="ImportView.error.at">
                    <t t-set="rows" t-value="error.rows"/>
                </t>
            </span>
            <t t-if="error.moreinfo" t-call="ImportView.error.more_info">
                <t t-set="messages" t-value="error.moreinfo"/>
            </t>
        </div>
    </t>

    <div t-name="ImportView.fieldError" t-attf-class="oe_import_report alert alert-#{errors[0].type}">
        <t t-if="errors.length &gt; 1" t-call="ImportView.fieldError.multi"></t>
        <t t-else="" t-call="ImportView.fieldError.single">
            <t t-set="error" t-value="errors[0]"/>
        </t>
    </div>
    <t t-name="ImportView.fieldError.single">
        <span class="oe_import_report_message">
            <t t-if="error.value &amp;&amp; !error.error_message">
                No matching records found for <t t-esc="error.field_type"/> <b>'<t t-esc="error.value"/>'</b> in field <b t-esc="error.field_name"/>
            </t>
            <b t-else="" t-esc="error.message"/>
        </span>
        <t t-if="error.rows" t-call="ImportView.error.at">
            <t t-set="rows" t-value="error.rows"/>
        </t>
        <t t-if="error.moreinfo" t-call="ImportView.error.more_info">
            <t t-set="messages" t-value="error.moreinfo"/>
        </t>
    </t>

    <t t-name="ImportView.fieldError.multi">
        <div class="oe_import_report_message">
            <t t-if="errors[0].value">
            No matching records found for the following <t t-esc="errors[0].field_type"/> </t>
            <t t-else="">Multiple errors occurred </t>
            in field <b t-esc="errors[0].field_name"/>:
            <ul>
                <t t-foreach="errors.length" t-as="index">
                    <li t-att-class="index &gt; 2 ? 'oe_import_report_more':''">
                        <b t-if="errors[index].error_message" t-esc="errors[index].message"/>
                        <b t-else="" t-esc="errors[index].value || errors[index].message"/>
                        <t t-if="errors[index].rows" t-call="ImportView.error.at">
                            <t t-set="rows" t-value="errors[index].rows"/>
                        </t>
                    </li>
                    <li t-if="errors.length &gt; 3 and index == 2" style="display: block;">
                        <a href="#" class="oe_import_report_count">
                            ( <i class="arrow down"></i> <t t-esc="errors.length - 3"/> more)
                        </a>
                    </li>
                </t>
            </ul>
        </div>

        <t t-if="errors[0].moreinfo" t-call="ImportView.error.more_info">
            <t t-set="messages" t-value="errors[0].moreinfo"/>
        </t>
    </t>

    <t t-name="ImportView.error.at">
        <t t-set="from" t-value="rows.from + offset"/>
        <t t-set="to" t-value="rows.to + offset"/>

        <t t-if="from === to">
            at row <t t-esc="from"/> <t t-if="result_names.length > rows.from &amp;&amp; result_names[rows.from] !== ''">(<t t-esc="result_names[rows.from]"/>)</t>
        </t>
        <t t-else="">
            at multiple rows
        </t>
    </t>

    <t t-name="ImportView.error.more_info">
        <div t-if="typeof messages === 'string'" class="oe_import_moreinfo oe_import_moreinfo_action">
            <t t-esc="messages"/>
        </div>
        <div t-elif="messages instanceof Array" class="oe_import_moreinfo oe_import_moreinfo_choices">
            <a href="#" class="oe_import_report_see_possible_value oe_import_see_all">
                <i class="fa fa-arrow-right"/> See possible values </a>
            <ul class="oe_import_report_more">
                <li t-foreach="messages" t-as="msg"><t t-esc="msg"/></li>
            </ul>
        </div>
        <div t-else="" class="oe_import_moreinfo oe_import_moreinfo_action">
            <a href="#" t-att-data-action="JSON.stringify(messages)" class="oe_import_see_all">
                <i class="fa fa-arrow-right"/> See possible values </a>
        </div>
    </t>

    <t t-name="ImportView.data_matching">
        <div class="oe_import_with_file d-none">
            <div class="oe_import_noheaders">
                <p class="alert alert-info">If the file contains
                    the column names, Odoo can try auto-detecting the
                    field corresponding to the column. This makes imports
                    simpler especially when the file has many columns.</p>
            </div>
            <div class="o_import_partial_alert alert alert-warning d-none mx-2 my-2 font-weight-bold">
                Click 'Resume' to proceed with the import, resuming at line
                <span class="o_import_partial_count">0</span>.<br/>
                You can test or reload your file before resuming the import.
            </div>
            <div class="oe_import_error_report px-3 py-2"></div>
            <div class="table-responsive">
                <table class="table-striped oe_import_grid w-100 overflow-hidden" />
            </div>
        </div>
    </t>

    <t t-name="ImportView.side_panel">
        <div class="oe_import_box d-none bg-white overflow-auto">
            <input accept=".csv, .xls, .xlsx, .xlsm, .ods" t-attf-id="file_#{_id}"
                  name="file" class="oe_import_file" type="file" style="display:none;"/>
            <div class="oe_import_with_file">
                <h4>Imported file</h4>
                <div class="mb-2 d-flex align-items-center">
                    <i class="fa fa-file white mr-2"></i>
                    <div id="oe_imported_file" class="font-italic truncate"></div>
                    <span id="oe_imported_file_extension" class="font-italic"></span>
                </div>

                <div class="oe_import_has_multiple_sheets flex-column mt-2">
                    <label for="oe_import_sheet">Sheet:</label>
                    <input class="oe_import_sheet oe_import_dropdown" id="oe_import_sheet"/>
                </div>

                <input type="checkbox" class="oe_import_has_header custom-control-input"
                       id="oe_import_has_header" checked="checked"/>
                <label class="custom-control-label" for="oe_import_has_header">Use first row as header</label>

                <div class="o_import_formatting">
                    <h4 class="mt-3">Formatting</h4>
                    <div t-foreach="[widget.opts, widget.parse_opts_formats, widget.parse_opts_separators]" t-as="options" class="oe_import_options">
                        <div t-attf-class="mb-1 oe_import_#{option.type}_box" t-foreach="options" t-as="option">
                            <!-- no @name, avoid submission when file_update called -->
                            <label t-attf-for="#{option.name}_#{_id}">
                                <t t-esc="option.label"/></label>
                            <input
                                   t-attf-id="#{option.name}_#{_id}"
                                   t-attf-class="oe_import_#{option.name} oe_import_dropdown w-100 mb-3"
                                   t-att-value="option.value"/>
                        </div>
                    </div>
                </div>

                <div class="o_import_batch">
                    <h4 class="mt-3">Batch Import</h4>
                    <div class="o_import_batch_alert font-weight-bold d-none pb-2">
                        The file will be imported by batches
                    </div>
                    <div class="d-flex">
                        <div class="oe_import_options oe_import_batch_limit w-50 pr-1">
                            <label class="mb-1" for="oe_import_batch_limit">Batch limit</label>
                            <input class="w-100" id="oe_import_batch_limit" value="2000"/>
                        </div>
                        <div class="oe_import_options w-50 pl-1" title="Warning: ignores the labels line, empty lines and
lines composed only of empty cells">
                            <label class="mb-1" for="oe_import_row_start">Start at line</label>
                            <input class="w-100" id="oe_import_row_start" value="1"/>
                        </div>
                    </div>
                </div>

                <h4 class="mt-3">Help</h4>
                <div t-foreach="widget.importTemplates" t-as="template">
                    <a t-att-href="template.template" aria-label="Download" title="Download">
                        <i class="fa fa-download"/> <span>Download Template</span>
                    </a>
                </div>
                <a href="https://www.odoo.com/documentation/15.0/applications/general/export_import_data.html" target="new">
                    <i class="fa fa-external-link"></i> Go to Import FAQ
                </a>

                <div class="oe_import_debug_option">
                    <h4 class="mt-3">Advanced</h4>
                    <div title="If the model uses openchatter, history tracking will set up subscriptions and send notifications during the import, but lead to a slower import.">
                        <input type="checkbox" id="oe_import_tracking" class="custom-control-input"/>
                        <label class="custom-control-label" for="oe_import_tracking">
                            Track history during import
                        </label>
                    </div>

                    <div>
                        <input type="checkbox" class="oe_import_advanced_mode custom-control-input" checked="checked"
                               id="oe_import_advanced_mode"/>
                        <label class="custom-control-label" for="oe_import_advanced_mode">Allow matching with subfields</label>
                    </div>
                </div>

            </div>
        </div>
    </t>

    <div t-name="base_import.progressDialog" class="o_import_progress_dialog text-white">
        <span class="fa fa-spin fa-circle-o-notch fa-2x mb-2"/>
        <div t-if="isBatch">
            <div class="o_import_progress_dialog_batch">
                <span t-esc="importMode"/> batch <span class="o_import_progress_dialog_batch_count">1</span> out of <span t-esc="totalSteps"/>...
                <div class="o_import_progress_dialog_time_left d-none">
                    <span>Estimated time left:</span>
                    <span class="o_import_progress_dialog_time_left_text"/>
                    <span>minutes</span>
                </div>
            </div>
            <span class="o_import_progress_dialog_stop d-none">
                Finalizing current batch before interrupting...
            </span>
            <div class="d-flex align-items-center mt-2">
                <div class="progress flex-grow-1">
                    <div class="progress-bar progress-bar-striped" role="progressbar"
                         aria-valuenow="0" aria-valuemin="0" aria-valuemax="100">
                        <span>0%</span>
                    </div>
                </div>
                <a class="o_progress_stop_import ml-2" role="button"><i class="fa fa-close" aria-label="Stop Import"/></a>
            </div>
        </div>
        <div t-else="">
            <span t-esc="task"/>...
        </div>
    </div>
</templates>

```

