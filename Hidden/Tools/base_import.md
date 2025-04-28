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
        'web.assets_backend': [
            'base_import/static/src/**/*.scss',
            'base_import/static/src/**/*.js',
            'base_import/static/src/**/*.xml',
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
    # pylint: disable=redefined-builtin
    def set_file(self, id):
        file = request.httprequest.files.getlist('ufile')[0]

        written = request.env['base_import.import'].browse(int(id)).write({
            'file': file.read(),
            'file_name': file.filename,
            'file_type': file.content_type,
        })

        return json.dumps({'result': written})

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
import codecs
import collections
import csv
import contextlib
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

from collections import defaultdict
from odoo import api, fields, models
from odoo.exceptions import UserError
from odoo.tools.translate import _
from odoo.tools.mimetypes import guess_mimetype
from odoo.tools import config, DEFAULT_SERVER_DATE_FORMAT, DEFAULT_SERVER_DATETIME_FORMAT, parse_version

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

if xlsx:
    from lxml import etree
    # xlrd.xlsx supports defusedxml, defusedxml's etree interface is broken
    # (missing ElementTree and thus ElementTree.iter) which causes a fallback to
    # Element.getiterator(), triggering a warning before 3.9 and an error from 3.9.
    #
    # We have defusedxml installed because zeep has a hard dep on defused and
    # doesn't want to drop it (mvantellingen/python-zeep#1014).
    #
    # Ignore the check and set the relevant flags directly using lxml as we have a
    # hard dependency on it.
    xlsx.ET = etree
    xlsx.ET_has_iterparse = True
    xlsx.Element_has_iter = True

try:
    from . import odf_ods_reader
except ImportError:
    odf_ods_reader = None

try:
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
            'model_name': model,
        }]
        if not depth:
            return importable_fields

        model_fields = Model.fields_get(attributes=[
            'string', 'required', 'type', 'readonly', 'relation',
            'definition_record', 'definition_record_field',
        ])
        blacklist = models.MAGIC_COLUMNS

        for name, field in dict(model_fields).items():
            if field['type'] not in 'properties':
                continue
            definition_record = field['definition_record']
            definition_record_field = field['definition_record_field']

            target_model = Model.env[Model._fields[definition_record].comodel_name]
            if not target_model.has_access('read'):  # ignore if you cannot read target_model at all
                continue
            # Do not take into account the definition of archived parents,
            # we do not import archived records most of the time.
            definition_records = target_model.search_fetch(
                [(definition_record_field, '!=', False)],
                [definition_record_field, 'display_name'],
                order='id',  # Avoid complex order
            )

            for record in definition_records:
                for definition in record[definition_record_field]:
                    definition_type = definition['type']
                    if (
                        definition_type == 'separator' or
                        (
                            definition_type in ('many2one', 'many2many')
                            and definition.get('comodel') not in Model.env
                        )
                    ):
                        continue
                    id_field = f"{name}.{definition['name']}"
                    model_fields[id_field] = {
                        'type': definition_type,
                        'string': _(
                            "%(property_string)s (%(parent_name)s)",
                            property_string=definition['string'], parent_name=record.display_name,
                        ),
                    }
                    if definition_type in ('many2one', 'many2many'):
                        model_fields[id_field]['relation'] = definition['comodel']

        for name, field in model_fields.items():
            if name in blacklist:
                continue
            if field.get('readonly'):
                continue
            field_value = {
                'id': name,
                'name': name,
                'string': field['string'],
                # Y U NO ALWAYS HAS REQUIRED
                'required': bool(field.get('required')),
                'fields': [],
                'type': field['type'],
                'model_name': model,
            }

            if field['type'] in ('many2many', 'many2one'):
                field_value['fields'] = [
                    dict(field_value, model_name=field['relation'], name='id', string=_("External ID"), type='id'),
                    dict(field_value, model_name=field['relation'], name='.id', string=_("Database ID"), type='id'),
                ]
                field_value['comodel_name'] = field['relation']
            elif field['type'] == 'one2many':
                field_value['fields'] = self.get_fields_tree(field['relation'], depth=depth-1)
                if self.env.user.has_group('base.group_no_one'):
                    field_value['fields'].append(
                        dict(field_value, model_name=field['relation'], fields=[], name='.id', string=_("Database ID"), type='id')
                    )
                field_value['comodel_name'] = field['relation']

            importable_fields.append(field_value)

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
        e = None
        # guess mimetype from file content
        mimetype = guess_mimetype(self.file or b'')
        (file_extension, handler, req) = FILE_TYPE_DICT.get(mimetype, (None, None, None))
        if handler:
            try:
                return getattr(self, '_read_' + file_extension)(options)
            except (ImportValidationError, ValueError):
                raise
            except Exception as exc:  # noqa: BLE001
                e = read_file_failed(exc, f"Unable to read file {self.file_name or '<unknown>'!r} as {file_extension!r} (guessed using mimetype {mimetype!r}).")

        # try reading with user-provided mimetype
        (file_extension, handler2, req2) = FILE_TYPE_DICT.get(self.file_type, (None, None, None))
        if handler2 and handler2 != handler:
            try:
                return getattr(self, '_read_' + file_extension)(options)
            except (ImportValidationError, ValueError):
                raise
            except Exception as exc:  # noqa: BLE001
                e = read_file_failed(exc, f"Unable to read file {self.file_name or '<unknown>'!r} as {file_extension!r} (decided from user-provided mimetype {self.file_type!r}).")

        # fallback on file extensions as mime types can be unreliable (e.g.
        # software setting incorrect mime types, or non-installed software
        # leading to browser not sending mime types)
        if self.file_name:
            _stem, ext = os.path.splitext(self.file_name)
            if (h := EXTENSIONS.get(ext)) and h != handler and h != handler2:
                try:
                    return getattr(self, '_read_' + ext[1:])(options)
                except (ImportValidationError, ValueError):
                    raise
                except Exception as exc:  # noqa: BLE001
                    e = read_file_failed(exc, f"Unable to read file {self.file_name!r} as {file_extension!r} (decided from file extension {ext!r}).")

        if e is not None:
            raise e

        if req2 or req:
            raise UserError(_("Unable to load \"{extension}\" file: requires Python module \"{modname}\"").format(extension=file_extension, modname=req2 or req))
        raise UserError(_("Unsupported file format \"{}\", import only supports CSV, ODS, XLS and XLSX").format(self.file_type))

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

        csv_text = csv_data.decode(encoding)

        separator = options.get('separator')
        if not separator:
            # default for unspecified separator so user gets a message about
            # having to specify it
            separator = ','
            for candidate in (',', ';', '\t', ' ', '|', unicodedata.lookup('unit separator')):
                # pass through the CSV and check if all rows are the same
                # length & at least 2-wide assume it's the correct one
                it = csv.reader(io.StringIO(csv_text), quotechar=options['quoting'], delimiter=candidate)
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

        csv_iterator = csv.reader(
            io.StringIO(csv_text),
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
        return ['text', 'char', 'binary', 'selection', 'html', 'tags']

    def _try_match_date_time(self, preview_values, options):
        # Or a date/datetime if it matches the pattern
        date_patterns = [options['date_format']] if options.get(
            'date_format') else []
        user_date_format = self.env['res.lang']._get_data(code=self.env.user.lang).date_format
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
            IrModelFieldsUs = self.with_context(lang='en_US').env['ir.model.fields']
            for field in fields_tree:
                fname = field['name']
                # exact match found based on the field technical name
                if header.casefold() == fname.casefold():
                    break
                # match found using either user translation, either model defined field label
                if header.casefold() == field['string'].casefold():
                    break
                field_strings_en = IrModelFieldsUs.get_field_string(field['model_name'])
                if fname in field_strings_en and header.casefold() == field_strings_en[fname].casefold():
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
                fname = field['name']
                # use string distance for fuzzy match only on most likely field types
                distances = [
                    self._get_distance(header.casefold(), fname.casefold()),
                    self._get_distance(header.casefold(), field['string'].casefold()),
                ]

                if field_string_en := IrModelFieldsUs.get_field_string(field['model_name']).get(fname):
                    distances.append(
                        self._get_distance(header.casefold(), field_string_en.casefold()),
                    )

                # Keep only the closest mapping suggestion. Note that in case of multiple mapping on the same field,
                # a mapping suggestion could be canceled by another one that has a smaller distance on the same field.
                # See 'deduplicate_mapping_suggestions' method for more info.
                current_field_dist = min(distances)
                if current_field_dist < min_dist:
                    min_dist_field = fname
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
                'debug': self.env.user.has_group('base.group_no_one'),
                'batch': batch,
                'file_length': len(rows),
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
                _(
                    "Error while importing records: all rows should be of the same size, but the title row has %(title_row_entries)d entries while the first row has %(first_row_entries)d. You may need to change the separator character.",
                    title_row_entries=len(fields),
                    first_row_entries=len(rows_to_import[0]),
                ),
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
                raise ImportValidationError(_("Column %(column)s contains incorrect values (value: %(value)s)", column=name, value=old_value), field=name)

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
                        elif '.' in line[index]:
                            # Detect if it's a filename
                            pass
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
                    _("Column %(column)s contains incorrect values. Error in line %(line)d: %(error)s", column=name, line=num + 1, error=e),
                    field=name, field_type=field_type
                )
            except Exception as e:
                raise ImportValidationError(
                    _("Error Parsing Date [%(field)s:L%(line)d]: %(error)s", field=name, line=num + 1, error=e),
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
        sp = self.env.cr.savepoint(flush=False)

        try:
            input_file_data, import_fields = self._convert_import_data(fields, options)
            # Parse date and float field
            input_file_data = self._parse_import_data(input_file_data, import_fields, options)
        except ImportValidationError as error:
            return {'messages': [error.__dict__]}

        _logger.info('importing %d rows...', len(input_file_data))

        binary_filenames = self._extract_binary_filenames(import_fields, input_file_data)

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
        with contextlib.suppress(psycopg2.InternalError):
            sp.close(rollback=dryrun)
        if dryrun:
            # cancel all changes done to the registry/ormcache
            # we need to clear the cache in case any created id was added to an ormcache and would be missing afterward
            self.pool.clear_all_caches()
            # don't propagate to other workers since it was rollbacked
            self.pool.reset_changes()

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
        if binary_filenames:
            import_result['binary_filenames'] = binary_filenames

        return import_result

    def _extract_binary_filenames(self, import_fields, data, model=False, prefix='', binary_filenames=False):
        model = model or self.res_model
        binary_filenames = binary_filenames or defaultdict(list)
        for name, field in self.env[model]._fields.items():
            name = prefix + name
            if any(name + '/' in import_field and name == import_field.split('/')[prefix.count('/')] for import_field in import_fields):
                # Recursive call with the relational as new model and add the field name to the prefix
                binary_filenames = self._extract_binary_filenames(import_fields, data, field.comodel_name, name + '/', binary_filenames)
            elif field.type == 'binary' and field.attachment and any(f in name for f in IMAGE_FIELDS) and name in import_fields:
                index = import_fields.index(name)
                for line in data:
                    filename = None
                    value = line[index]
                    if isinstance(value, str):
                        if re.match(config.get("import_image_regex", DEFAULT_IMAGE_REGEX), value):
                            pass
                        elif '.' in value:
                            # Detect if it's a filename
                            filename = value
                            line[index] = ''
                        # else base64 nothing to do
                    binary_filenames[name].append(filename)
        return binary_filenames

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
                    # if not on the last hierarchy level, retarget the model.
                    # Also check if the field exists to silently ignore properties field and
                    # since we don't have the definition here anyway.
                    if field != target_field and field in self.env[target_model]:
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
                    elif fallback_values[field]['field_type'] == "selection" and value.lower() not in fallback_values[field]["selection_values"]:
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


def read_file_failed(exc: Exception, message: str) -> UserError:
    _logger.warning(message, exc_info=True)
    e = UserError(message)
    e.__cause__ = exc
    return e

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

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import base_import

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_base_import_mapping,base.import.mapping,model_base_import_mapping,base.group_user,1,1,1,1
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

## File: static\src\binary_file_manager.js

```javascript
import { Mutex } from "@web/core/utils/concurrency";
import { checkFileSize } from "@web/core/utils/files";

export class BinaryFileManager {
    constructor(resModel, fields, parameters, context, orm, notificationService) {
        this.resModel = resModel;
        this.fields = [".id", ...fields];
        this.parameters = parameters;

        this.context = context;
        this.orm = orm;
        this.notificationService = notificationService;

        this.maxBatchSize = this.parameters.maxBatchSize * 0.95; // 0.95 for not calculated payload overhead
        this.delayAfterEachBatch = this.parameters.delayAfterEachBatch * 1000;

        this.dataToSend = {};

        this.mutex = new Mutex();
    }

    async addFile(id, field, file) {
        let data = await this._readFile(file);
        if (typeof data === "string" && data.startsWith("data:")) {
            // Remove data:image/*;base64,
            data = data.split(",")[1];
        }
        const dataSize = data.length;
        if (!checkFileSize(dataSize, this.notificationService)) {
            return;
        }

        if (this.getCurrentSize() + dataSize >= this.maxBatchSize) {
            await this.mutex.exec(async () => await this._send());
        }
        if (!(id in this.dataToSend)) {
            this.dataToSend[id] = Array(this.fields.length);
            this.dataToSend[id][0] = id;
        }
        const indexOfField = this.fields.indexOf(field, 1);
        this.dataToSend[id][indexOfField] = data;
    }

    async sendLastPayload() {
        if (Object.keys(this.dataToSend).length > 0) {
            await this.mutex.exec(async () => await this._send());
        }
    }

    async _send() {
        await new Promise((resolve) => {
            setTimeout(resolve, this.delayAfterEachBatch);
        });
        const data = Object.values(this.dataToSend);
        this.dataToSend = {};
        const context = {
            ...this.context,
            import_file: true,
            tracking_disable: this.parameters.tracking_disable,
            name_create_enabled_fields: this.parameters.name_create_enabled_fields || {},
            import_set_empty_fields: this.parameters.import_set_empty_fields || [],
            import_skip_records: this.parameters.import_skip_records || [],
        };
        let res;
        try {
            res = await this.orm.call(this.resModel, "load", [], {
                fields: this.fields,
                data,
                context,
            });
        } catch (error) {
            console.error(error);
            return { error };
        }
        return res;
    }

    _readFile(file) {
        return new Promise((resolve, reject) => {
            const reader = new FileReader();
            reader.onerror = (event) => reject(event);
            reader.onabort = (event) => reject(event);
            reader.onload = (event) => resolve(event.target.result);
            reader.readAsDataURL(file);
        });
    }

    getCurrentSize() {
        return JSON.stringify(this.dataToSend).length;
    }
}

```

## File: static\src\import_block_ui.js

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";

export class ImportBlockUI extends Component {
    static props = {
        message: { type: String, optional: true },
        blockComponent: { type: Object, optional: true },
    };
    static template = "base_import.BlockUI";
}

```

## File: static\src\import_block_ui.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <!-- TODO should re-use the blockUI template of web -->
    <t t-name="base_import.BlockUI">
        <div class="o_blockUI fixed-top d-flex justify-content-center align-items-center flex-column vh-100 bg-black-50">
            <div class="o_spinner mb-4">
                <img src="/web/static/img/spin.svg" alt="Loading..."/>
            </div>
            <div t-if="props.message or props.blockComponent">
                <div class="o_message text-center px-4" t-esc="props.message" />
                <t t-if="props.blockComponent" t-component="props.blockComponent.class" t-props="props.blockComponent.props"/>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\import_model.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { registry } from "@web/core/registry";
import { checkFileSize, DEFAULT_MAX_FILE_SIZE } from "@web/core/utils/files";
import { useService } from "@web/core/utils/hooks";
import { pick } from "@web/core/utils/objects";
import { groupBy, sortBy } from "@web/core/utils/arrays";
import { memoize } from "@web/core/utils/functions";
import { session } from "@web/session";
import { useState } from "@odoo/owl";
import { ImportBlockUI } from "./import_block_ui";
import { BinaryFileManager } from "./binary_file_manager";

const mainComponentRegistry = registry.category("main_components");

const strftimeFormatTable = {
    d: "w",
    DD: "d",
    ddd: "a",
    dddd: "A",
    DDDD: "j",
    ww: "U",
    WW: "W",
    mm: "M",
    MM: "m",
    MMM: "b",
    MMMM: "B",
    YYYY: "Y",
    YY: "y",
    ss: "S",
    hh: "h",
    HH: "H",
    A: "p",
};

/**
 * Convert a human readable format to Python strftime format. In case
 * no corresponding format is supported, a similar fallback is given
 * from the list of other supported formatting value.
 *
 * @param {string} value original Luxon format
 * @returns {string} valid strftime format
 */
const humanToStrftimeFormat = memoize(function humanToStrftimeFormat(value) {
    const regex = /(dddd|ddd|dd|d|mmmm|mmm|mm|ww|yyyy|yy|hh|ss|a)/gi;
    return value.replace(regex, (value) => {
        if (strftimeFormatTable[value]) {
            return "%" + strftimeFormatTable[value];
        }
        return (
            "%" +
            (strftimeFormatTable[value.toLowerCase()] || strftimeFormatTable[value.toUpperCase()])
        );
    });
});

const strftimeToHumanFormat = memoize(function strftimeToHumanFormat(value) {
    Object.entries(strftimeFormatTable).forEach(([k, v]) => {
        value = value.replace(`%${v}`, k);
    });
    return value;
});

/**
 * -------------------------------------------------------------------------
 * Base Import Business Logic
 * -------------------------------------------------------------------------
 *
 * Handles mapping and updating the preview data of the csv/excel files to be
 * used in the different base_import components.
 *
 * When uploading a file some "preview data" is returned by the backend, this
 * data consist of the different columns of the file and the odoo fields which
 * these columns can be mapped to.
 *
 * Only a small selection of the lines are returned so the user can get an idea
 * of how to correctly map the columns. *(this is why it is refered as "preview
 * data")*
 *
 */
export class BaseImportModel {
    constructor({ env, resModel, context, orm }) {
        this.id = 1;
        this.env = env;
        this.orm = orm;
        this.handleInterruption = false;

        this.resModel = resModel;
        this.context = context || {};

        this.fields = [];
        this.columns = [];
        this.importMessages = [];
        this._importOptions = {};

        this.importTemplates = [];

        this.formattingOptionsValues = this._getCSVFormattingOptions();

        this.importOptionsValues = {
            ...this.formattingOptionsValues,
            advanced: {
                reloadParse: true,
                value: true,
            },
            has_headers: {
                reloadParse: true,
                value: true,
            },
            keep_matches: {
                value: false,
            },
            limit: {
                value: 2000,
            },
            sheets: {
                value: [],
            },
            sheet: {
                label: _t("Selected Sheet:"),
                reloadParse: true,
                value: "",
            },
            skip: {
                value: 0,
            },
            tracking_disable: {
                value: true,
            },
        };

        const maxUploadSize = session.max_file_upload_size || DEFAULT_MAX_FILE_SIZE;
        this.binaryFilesParams = {
            binaryFiles: {
                value: {},
            },
            maxSizePerBatch: {
                help: _t("Defines how many megabytes can be imported in each batch import"),
                value: 10,
                max: Math.round(maxUploadSize / 1024 / 1024),
                min: 0,
            },
            delayAfterEachBatch: {
                help: _t(
                    "After each batch import, this delay is applied to avoid unthrottled calls"
                ),
                value: 1,
                min: 1,
            },
        };

        this.fieldsToHandle = {};

        this.notificationService = useService("notification");
    }

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    get formattingOptions() {
        return pick(this.importOptionsValues, ...Object.keys(this.formattingOptionsValues));
    }

    /**
     * This getter returns the current values pf the options, formatted to match the
     * server API (date and datetime options should be Python strftime formatted)
     */
    get formattedImportOptions() {
        const options = this.importOptions;
        options.date_format = humanToStrftimeFormat(options.date_format);
        options.datetime_format = humanToStrftimeFormat(options.datetime_format);
        return options;
    }

    get importOptions() {
        const tempImportOptions = {
            import_skip_records: [],
            import_set_empty_fields: [],
            fallback_values: {},
            name_create_enabled_fields: {},
        };
        for (const [name, option] of Object.entries(this.importOptionsValues)) {
            tempImportOptions[name] = option.value;
        }

        for (const key in this.fieldsToHandle) {
            const value = this.fieldsToHandle[key];
            if (value) {
                if (value.optionName === "import_skip_records") {
                    tempImportOptions.import_skip_records.push(key);
                } else if (value.optionName === "import_set_empty_fields") {
                    tempImportOptions.import_set_empty_fields.push(key);
                } else if (value.optionName === "name_create_enabled_fields") {
                    tempImportOptions.name_create_enabled_fields[key] = true;
                } else if (value.optionName === "fallback_values") {
                    tempImportOptions.fallback_values[key] = value.value;
                }
            }
        }

        this._importOptions = tempImportOptions;
        return tempImportOptions;
    }

    set importOptions(options) {
        for (const key in options) {
            this.importOptionsValues[key].value = options[key];
        }
    }

    /**
     * A custom BlockUI is required to add the progress bar or text when blocking
     * the UI, without modifying the core ui service to handle a generic use case
     */
    block(message, blockComponent) {
        mainComponentRegistry.add(
            "ImportBlockUI",
            {
                Component: ImportBlockUI,
                props: {
                    blockComponent,
                    message,
                },
            },
            { force: true }
        );
    }

    unblock() {
        mainComponentRegistry.remove("ImportBlockUI");
    }

    async init() {
        [this.importTemplates, this.id] = await Promise.all([
            this.orm.call(this.resModel, "get_import_templates", [], {
                context: this.context,
            }),
            this.orm.call("base_import.import", "create", [{ res_model: this.resModel }]),
        ]);
    }

    async executeImport(isTest = false, totalSteps, importProgress) {
        this.handleInterruption = false;
        this._updateComments();
        this.importMessages = [];

        const startRow = this.importOptions.skip;
        const importRes = {
            ids: [],
            fields: this.columns.map((e) => Boolean(e.fieldInfo) && e.fieldInfo.fieldPath),
            columns: this.columns.map((e) => e.name.trim().toLowerCase()),
            hasError: false,
        };

        for (let i = 1; i <= totalSteps; i++) {
            if (this.handleInterruption) {
                if (importRes.hasError || isTest) {
                    importRes.nextrow = startRow;
                    this.setOption("skip", startRow);
                }
                break;
            }

            const error = await this._executeImportStep(isTest, importRes);
            if (error) {
                const errorData = error.data || {};
                const message = errorData.arguments && (errorData.arguments[1] || errorData.arguments[0])
                    || _t("An unknown issue occurred during import (possibly lost connection, data limit exceeded or memory limits exceeded). Please retry in case the issue is transient. If the issue still occurs, try to split the file rather than import it at once.");

                if (error.message) {
                    this._addMessage("danger", [error.message, message]);
                } else {
                    this._addMessage("danger", [message]);
                }

                importRes.hasError = true;
                break;
            }

            if (importProgress) {
                importProgress.step = i;
                importProgress.value = Math.round((100 * (i - 1)) / totalSteps);
            }
        }

        if (!importRes.hasError) {
            if (importRes.nextrow) {
                this._addMessage("warning", [
                    _t(
                        "Click 'Resume' to proceed with the import, resuming at line %s.",
                        importRes.nextrow + 1
                    ),
                    _t("You can test or reload your file before resuming the import."),
                ]);
            }
            if (isTest) {
                this._addMessage("info", [_t("Everything seems valid.")]);
            }
        } else {
            importRes.nextrow = startRow;
        }
        return { res: importRes };
    }

    /**
     * Ask the server for the parsing preview
     * and update the data accordingly.
     */
    async updateData(fileChanged = false) {
        if (fileChanged) {
            this.importOptionsValues.sheet.value = "";
        }
        this.importMessages = [];

        const res = await this.orm.call("base_import.import", "parse_preview", [
            this.id,
            this.formattedImportOptions,
        ]);

        if (!res.error) {
            res.options.date_format = strftimeToHumanFormat(res.options.date_format);
            res.options.datetime_format = strftimeToHumanFormat(res.options.datetime_format);
            this._onLoadSuccess(res);
        } else {
            this._onLoadError();
        }
        return { res, error: res.error };
    }

    async setOption(optionName, value, fieldName) {
        if (fieldName) {
            this.fieldsToHandle[fieldName] = {
                optionName,
                value,
            };
            return;
        }
        this.importOptionsValues[optionName].value = value;
        if (this.importOptionsValues[optionName].reloadParse) {
            return this.updateData();
        }
    }

    onBinaryFilesParamsChanged(parameterName, value) {
        if (parameterName === "binaryFiles") {
            const files = {};
            for (const file of value) {
                if (checkFileSize(file.size, this.notificationService)) {
                    files[file.name] = file;
                }
            }
            value = files;
        }
        this.binaryFilesParams[parameterName].value = value;
    }

    setColumnField(column, fieldInfo) {
        column.fieldInfo = fieldInfo;
        this._updateComments(column);
    }

    isColumnFieldSet(column) {
        return column.fieldInfo != null;
    }

    /*
     * We must wait the current iteration of execute_import to conclude and it
     * will stop at the start of the next batch with handleInterruption
     */
    stopImport() {
        this.handleInterruption = true;
    }

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _addMessage(type, lines) {
        const importMsgs = this.importMessages;
        importMsgs.push({
            type: type.replace("error", "danger"),
            lines,
        });
        this.importMessages = importMsgs;
    }

    async _executeImportStep(isTest, importRes) {
        const importArgs = [
            this.id,
            importRes.fields,
            importRes.columns,
            this.formattedImportOptions,
        ];
        const { ids, messages, nextrow, name, error, binary_filenames } = await this._callImport(
            isTest,
            importArgs
        );

        // Handle server errors
        if (error) {
            return error;
        }

        if (ids) {
            importRes.ids = importRes.ids.concat(ids);
        }

        // Handle import errors
        if (messages && messages.length) {
            importRes.hasError = true;
            this.stopImport();
            if (this._handleImportErrors(messages, name)) {
                return false;
            }
        }

        // Push local image to records
        await this._pushLocalImageToRecords(ids, binary_filenames, isTest);

        // Check if we should continue
        if (nextrow) {
            this.setOption("skip", nextrow);
            importRes.nextrow = nextrow;
        } else {
            // Falsy `nextrow` signals there's nothing left to import
            this.stopImport();
        }
        return false;
    }

    async _pushLocalImageToRecords(ids, binaryFilenames, isTest) {
        if (typeof binaryFilenames === "object") {
            const parameters = {
                tracking_disable: this.importOptions.tracking_disable,
                delayAfterEachBatch: this.binaryFilesParams.delayAfterEachBatch.value,
                maxBatchSize: this.binaryFilesParams.maxSizePerBatch.value * 1024 * 1024,
            };

            if (!this.binaryFilesParams.binaryFiles) {
                return;
            }
            const binaryFiles = this.binaryFilesParams.binaryFiles.value;
            const fields = Object.keys(binaryFilenames);
            const binaryFileManager = new BinaryFileManager(
                this.resModel,
                fields,
                parameters,
                this.context,
                this.orm,
                this.notificationService
            );
            for (let rowIndex = 0; rowIndex < ids.length; rowIndex++) {
                const id = ids[rowIndex];
                for (const field of fields) {
                    const fileName = binaryFilenames[field][rowIndex];
                    if (!fileName) {
                        continue;
                    }
                    if (fileName in binaryFiles) {
                        const file = binaryFiles[fileName];
                        if (!file || isTest) {
                            continue;
                        }
                        await binaryFileManager.addFile(id, field, file);
                    }
                }
            }
            if (!isTest) {
                await binaryFileManager.sendLastPayload();
            }
        }
    }

    async _callImport(dryrun, args) {
        try {
            const res = await this.orm.silent.call("base_import.import", "execute_import", args, {
                dryrun,
                context: {
                    ...this.context,
                    tracking_disable: this.importOptions.tracking_disable,
                },
            });
            return res;
        } catch (error) {
            // This pattern isn't optimal but it is need to have
            // similar behaviours as in legacy. That is, catching
            // all import errors and showing them inside the top
            // "messages" area.
            return { error };
        }
    }

    _handleImportErrors(messages, name) {
        if (messages[0].not_matching_error) {
            this._addMessage(messages[0].type, [messages[0].message]);
            return true;
        }

        const sortedMessages = this._groupErrorsByField(messages);
        if (sortedMessages[0]) {
            this._addMessage(sortedMessages[0].type, [sortedMessages[0].message]);
            delete sortedMessages[0];
        } else {
            this._addMessage("danger", [_t("The file contains blocking errors (see below)")]);
        }

        for (const [columnFieldId, errors] of Object.entries(sortedMessages)) {
            // Handle errors regarding specific colums.
            const column = this.columns.find(
                (e) => e.fieldInfo && e.fieldInfo.fieldPath === columnFieldId
            );
            if (column) {
                column.resultNames = name;
                column.errors = errors;
            } else {
                for (const error of errors) {
                    // Handle errors regarding specific records.
                    if (error.record !== undefined) {
                        this._addMessage("danger", [
                            error.rows.from === error.rows.to
                                ? _t('Error at row %(row)s: "%(error)s"', {
                                      row: error.record,
                                      error: error.message,
                                  })
                                : _t("%s at multiple rows", error.message),
                        ]);
                    }
                    // Handle global errors.
                    else {
                        this._addMessage("danger", [error.message]);
                    }
                }
            }
        }
    }

    _groupErrorsByField(messages) {
        const groupedErrors = {};
        const errorsByMessage = groupBy(this._sortErrors(messages), (f) => f.message || "0");
        for (const [message, errors] of Object.entries(errorsByMessage)) {
            if (!message.record) {
                const foundError = errors.find((e) => e.record === undefined);
                if (foundError) {
                    groupedErrors[0] = foundError;
                    continue;
                }
            }

            errors[0].rows.to = errors[errors.length - 1].rows.to;
            const fieldId = errors[0].field_path ? errors[0].field_path.join("/") : errors[0].field;
            if (groupedErrors[fieldId]) {
                groupedErrors[fieldId].push(errors[0]);
            } else {
                groupedErrors[fieldId] = [errors[0]];
            }
        }
        return groupedErrors;
    }

    _sortErrors(messages) {
        return sortBy(messages, (e) => ["error", "warning", "info"].indexOf(e.priority));
    }

    /**
     * On the preview data succesfuly loaded, update the
     * import options, columns and messages.
     * @param {*} res
     */
    _onLoadSuccess(res) {
        // Set options
        for (const key in res.options) {
            if (this.importOptionsValues[key]) {
                this.importOptionsValues[key].value = res.options[key];
            }
        }

        if (!res.fields.length) {
            this.importOptionsValues.advanced.value = res.advanced_mode;
        }

        this.fields = res.fields;
        this.columns = this._getColumns(res);

        // Set import messages
        if (res.headers.length === 1) {
            this._addMessage("warning", [
                _t(
                    "A single column was found in the file, this often means the file separator is incorrect."
                ),
            ]);
        }

        this._updateComments();
    }

    _onLoadError() {
        this.columns = [];
        this.importMessages = [];
    }

    _getColumns(res) {
        function getId(res, index) {
            return res.matches && index in res.matches && res.matches[index].length > 0
                ? res.matches[index].join("/")
                : undefined;
        }

        if (this.importOptions.has_headers && res.headers && res.preview.length > 0) {
            return res.headers.flatMap((header, index) => {
                return this._createColumn(
                    res,
                    getId(res, index),
                    header,
                    index,
                    res.preview[index],
                    res.preview[index][0]
                );
            });
        } else if (res.preview && res.preview.length >= 2) {
            return res.preview.flatMap((preview, index) =>
                this._createColumn(
                    res,
                    preview[0],
                    this.importOptions.has_headers ? preview[0] : preview.join(", "),
                    index,
                    preview,
                    preview[1]
                )
            );
        }
        return [];
    }

    _createColumn(res, id, name, index, previews, preview) {
        const fields = this._getFields(res, index);
        return {
            id,
            name,
            preview,
            previews,
            fields,
            fieldInfo: this._findField(fields, id),
            comments: [],
            errors: [],
        };
    }

    _findField(fields, id) {
        return Object.entries(fields)
            .flatMap((e) => e[1])
            .find((field) => field.fieldPath === id);
    }

    /**
     * Sort fields into their respective categories, namely:
     * - Basic => Only the ID field
     * - Suggested => Non-relational fields from the header"s types
     * - Additional => Non-relational fields of any other type
     * - Relational => Relational fields
     * @param {*} res
     */
    _getFields(res, index) {
        const advanced = this.importOptionsValues.advanced.value;
        const fields = {
            basic: [],
            suggested: [],
            additional: [],
            relational: [],
        };

        function isRegular(subfields) {
            return (
                !subfields ||
                subfields.length === 0 ||
                (subfields.length === 2 &&
                    subfields[0].name === "id" &&
                    subfields[1].name === ".id")
            );
        }

        function hasType(types, field) {
            return types && types.indexOf(field.type) !== -1;
        }

        const sortSingleField = (field, ancestors, collection, types) => {
            ancestors.push(field);
            field.fieldPath = ancestors.map((f) => f.name).join("/");
            field.label = ancestors.map((f) => f.string).join(" / ");

            // Get field respective category
            if (!collection) {
                if (field.name === "id") {
                    collection = fields.basic;
                } else if (isRegular(field.fields)) {
                    collection = hasType(types, field) ? fields.suggested : fields.additional;
                } else {
                    collection = fields.relational;
                }
            }

            // Add field to found category
            collection.push(field);

            if (advanced) {
                for (const subfield of field.fields) {
                    sortSingleField(subfield, [...ancestors], collection, types);
                }
            }
        };

        // Sort fields in their respective categories
        for (const field of this.fields) {
            if (!field.isRelation) {
                if (advanced) {
                    sortSingleField(field, [], undefined, ["all"]);
                } else {
                    const acceptedTypes = res.header_types[index];
                    sortSingleField(field, [], undefined, acceptedTypes);
                }
            }
        }

        return fields;
    }

    _updateComments(updatedColumn) {
        for (const column of this.columns) {
            column.comments = [];
            column.errors = [];
            column.resultNames = [];
            column.importOptions =
                column.fieldInfo && this.fieldsToHandle[column.fieldInfo.fieldPath];

            if (!column.fieldInfo) {
                continue;
            }

            // Fields of type "char", "text" or "many2many" can be specified multiple
            // times and they will be concatenated, fields of other types must be unique.
            if (["char", "text", "many2many"].includes(column.fieldInfo.type)) {
                if (column.fieldInfo.type === "many2many") {
                    column.comments.push({
                        type: "info",
                        content: _t("To import multiple values, separate them by a comma."),
                    });
                }

                // If multiple columns are mapped on the same field, inform
                // the user that they will be concatenated.
                const samefieldColumns = this.columns.filter(
                    (col) => col.fieldInfo && col.fieldInfo.fieldPath === column.fieldInfo.fieldPath
                );
                if (samefieldColumns.length >= 2) {
                    column.comments.push({
                        type: "info",
                        content: _t("This column will be concatenated in field"),
                        fieldName: column.fieldInfo.string,
                    });
                }
            } else if (updatedColumn && column.id !== updatedColumn.id && updatedColumn.fieldInfo) {
                // If column is mapped on an already mapped field, remove that field
                // from the old column to keep it unique.
                if (updatedColumn.fieldInfo.fieldPath === column.fieldInfo.fieldPath) {
                    column.fieldInfo = null;
                }
            }
        }
    }

    _getCSVFormattingOptions() {
        return {
            encoding: {
                label: _t("Encoding:"),
                type: "select",
                value: "",
                options: [
                    "utf-8",
                    "utf-16",
                    "windows-1252",
                    "latin1",
                    "latin2",
                    "big5",
                    "gb18030",
                    "shift_jis",
                    "windows-1251",
                    "koi8_r",
                ],
            },
            separator: {
                label: _t("Separator:"),
                type: "select",
                value: "",
                options: [
                    { value: ",", label: _t("Comma") },
                    { value: ";", label: _t("Semicolon") },
                    { value: "\t", label: _t("Tab") },
                    { value: " ", label: _t("Space") },
                ],
            },
            quoting: {
                label: _t("Text Delimiter:"),
                type: "input",
                value: '"',
            },
            date_format: {
                help: _t(
                    "Use YYYY to represent the year, MM for the month and DD for the day. Include separators such as a dot, forward slash or dash. You can use a custom format in addition to the suggestions provided. Leave empty to let Odoo guess the format (recommended)"
                ),
                label: _t("Date Format:"),
                type: "input",
                value: "",
                options: [
                    "YYYY-MM-DD",
                    "YYYY/MM/DD",
                    "DD/MM/YYYY",
                    "DDMMYYYY",
                    "MM/DD/YYYY",
                    "MMDDYYYY",
                ],
            },
            datetime_format: {
                help: _t(
                    "Use HH for hours in a 24h system, use II in conjonction with 'p' for a 12h system. You can use a custom format in addition to the suggestions provided. Leave empty to let Odoo guess the format (recommended)"
                ),
                label: _t("Datetime Format:"),
                type: "input",
                value: "",
                options: [
                    "YYYY-MM-DD HH:mm:SS",
                    "YYYY/MM/DD HH:mm:SS",
                    "DD/MM/YYYY HH:mm:SS",
                    "DDMMYYYY HH:mm:SS",
                    "MM/DD/YYYY II:mm:SS p",
                    "MMDDYYYY II:mm:SS p",
                ],
            },
            float_thousand_separator: {
                label: _t("Thousands Separator:"),
                type: "select",
                value: ",",
                options: [
                    { value: ",", label: _t("Comma") },
                    { value: ".", label: _t("Dot") },
                    { value: "", label: _t("No Separator") },
                ],
            },
            float_decimal_separator: {
                label: _t("Decimals Separator:"),
                type: "select",
                value: ".",
                options: [
                    { value: ",", label: _t("Comma") },
                    { value: ".", label: _t("Dot") },
                ],
            },
        };
    }
}

/**
 * @returns {BaseImportModel}columns
 */
export function useImportModel({ env, resModel, context, orm }) {
    return useState(new BaseImportModel({ env, resModel, context, orm }));
}

```

## File: static\src\import_action\import_action.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { Component, onWillStart, onMounted, useRef, useState } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { useFileUploader } from "@web/core/utils/files";
import { useService } from "@web/core/utils/hooks";
import { FileInput } from "@web/core/file_input/file_input";
import { useDropzone } from "@web/core/dropzone/dropzone_hook";
import { useImportModel } from "../import_model";
import { ImportDataContent } from "../import_data_content/import_data_content";
import { ImportDataProgress } from "../import_data_progress/import_data_progress";
import { ImportDataSidepanel } from "../import_data_sidepanel/import_data_sidepanel";
import { Layout } from "@web/search/layout";
import { router } from "@web/core/browser/router";
import { standardActionServiceProps } from "@web/webclient/actions/action_service";
import { DocumentationLink } from "@web/views/widgets/documentation_link/documentation_link";

export class ImportAction extends Component {
    static template = "ImportAction";
    static nextId = 1;
    static components = {
        FileInput,
        ImportDataContent,
        ImportDataSidepanel,
        Layout,
        DocumentationLink,
    };
    static props = { ...standardActionServiceProps };

    setup() {
        this.notification = useService("notification");
        this.orm = useService("orm");
        this.env.config.setDisplayName(this.props.action.name || _t("Import a File"));
        // this.props.action.params.model is there for retro-compatiblity issues
        this.resModel = this.props.action.params.model || this.props.action.params.active_model;
        if (this.resModel) {
            this.props.updateActionState({ active_model: this.resModel });
        }
        this.model = useImportModel({
            env: this.env,
            resModel: this.resModel,
            context: this.props.action.params.context || {},
            orm: this.orm,
        });

        this.state = useState({
            filename: undefined,
            fileLength: 0,
            importMessages: [],
            importProgress: {
                value: 0,
                step: 1,
            },
            isPaused: false,
            previewError: "",
        });

        this.uploadFiles = useFileUploader();
        useDropzone(useRef("root"), async event => {
            const { files } = event.dataTransfer;
            if (files.length === 0) {
                this.notification.add(_t("Please upload an Excel (.xls or .xlsx) or .csv file to import."), {
                    type: "danger",
                });
            } else if (files.length > 1) {
                this.notification.add(_t("Please upload a single file."), {
                    type: "danger",
                });
            } else {
                const file = files[0];
                const isValidFile = file.name.endsWith(".csv")
                                 || file.name.endsWith(".xls")
                                 || file.name.endsWith(".xlsx");
                if (!isValidFile) {
                    this.notification.add(_t("Please upload an Excel (.xls or .xlsx) or .csv file to import."), {
                        type: "danger",
                    });
                } else {
                    await this.uploadFiles(this.uploadFilesRoute, {
                        csrf_token: odoo.csrf_token,
                        ufile: [file],
                        model: this.resModel,
                        id: this.model.id,
                    });
                    this.handleFilesUpload([file]);
                }
            }
        });

        onWillStart(() => this.model.init());
        onMounted(() => this.enter());
    }

    enter() {
        const newState = { action: "import", model: this.resModel };
        router.pushState(newState, { replace: true });
    }

    exit(resIds) {
        this.env.config.historyBack();
    }

    get display() {
        return {
            controlPanel: {},
        };
    }

    get importTemplates() {
        return this.model.importTemplates;
    }

    get uploadFilesRoute () {
        return "/base_import/set_file";
    }

    //--------------------------------------------------------------------------
    // Options
    //--------------------------------------------------------------------------

    get formattingOptions() {
        return this.model.formattingOptions;
    }

    get totalToImport() {
        return this.state.fileLength - parseInt(this.importOptions.skip);
    }

    get totalSteps() {
        return this.isBatched ? Math.ceil(this.totalToImport / this.importOptions.limit) : 1;
    }

    get importOptions() {
        return this.model.importOptions;
    }

    get isPreviewing() {
        return this.state.filename !== undefined;
    }

    // Activate the batch configuration panel only if the file length > 100. (In order to let the user choose
    // the batch size even for medium size file. Could be useful to reduce the batch size for complex models).
    get isBatched() {
        return this.state.fileLength > 100;
    }

    async onOptionChanged(name, value, fieldName = null) {
        this.model.block();
        const result = await this.model.setOption(name, value, fieldName);
        if (result) {
            const { res, error } = result;
            if (!error && res.file_length) {
                this.state.fileLength = res.file_length;
            }
        }
        this.model.unblock();
    }

    async reload() {
        this.model.block();
        await this.model.updateData();
        this.model.unblock();
    }

    //--------------------------------------------------------------------------
    // File
    //--------------------------------------------------------------------------

    async handleFilesUpload(files) {
        if (!files || files.length <= 0) {
            return;
        }

        this.state.filename = files[0].name;
        this.state.importMessages = [];

        this.model.block(_t("Loading file..."));
        const { res, error } = await this.model.updateData(true);

        if (error) {
            this.state.previewError = error;
        } else {
            this.state.fileLength = res.file_length;
            this.state.previewError = undefined;
        }
        this.model.unblock();
    }

    async handleImport(isTest = true) {
        const message = isTest ? _t("Testing") : _t("Importing");

        let blockComponent;
        if (this.isBatched) {
            blockComponent = {
                class: ImportDataProgress,
                props: {
                    stopImport: () => this.stopImport(),
                    totalSteps: this.totalSteps,
                    importProgress: this.state.importProgress,
                },
            };
        }

        this.model.block(message, blockComponent);

        let res = { ids: [] };
        try {
            const data = await this.model.executeImport(
                isTest,
                this.totalSteps,
                this.state.importProgress
            );
            res = data.res;
        } finally {
            this.model.unblock();
        }

        if (!isTest && res.nextrow) {
            this.state.isPaused = true;
        }

        if (!isTest && res.ids.length) {
            this.notification.add(_t("%s records successfully imported", res.ids.length), {
                type: "success",
            });
            this.exit(res.ids);
        }
    }

    stopImport() {
        this.model.stopImport();
    }

    //--------------------------------------------------------------------------
    // Fields
    //--------------------------------------------------------------------------

    onFieldChanged(column, fieldInfo) {
        this.model.setColumnField(column, fieldInfo);
    }

    isFieldSet(column) {
        return column.fieldInfo != null;
    }

    get hasBinaryFields() {
        return this.model.columns.some((column) => column.fieldInfo?.type === "binary");
    }
}

registry.category("actions").add("import", ImportAction);

```

## File: static\src\import_action\import_action.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="ImportAction">
        <div t-ref="root" class="h-100 d-flex flex-column">
            <Layout className="'o_import_action d-flex h-100 overflow-auto'" display="display">
                <t t-set-slot="control-panel-create-button">
                    <t t-if="isPreviewing">
                        <button t-if="!state.isPaused" type="button" class="btn btn-primary" t-on-click="() => this.handleImport(false)">Import</button>
                        <button t-else="" type="button" class="btn btn-primary" t-on-click="() => this.handleImport(false)">Resume</button>
                        <button type="button" class="btn btn-secondary" t-on-click="() => this.handleImport(true)">Test</button>
                    </t>
                    <FileInput
                        acceptedFileExtensions="'.csv, .xls, .xlsx, .xlsm, .ods'"
                        onUpload.bind="(data, files) => this.handleFilesUpload(files)"
                        resId="model.id"
                        resModel="this.resModel"
                        route="this.uploadFilesRoute"
                    >
                        <button t-if="isPreviewing" type="button" class="btn btn-secondary">Load Data File</button>
                        <button t-else="" type="button" class="btn btn-primary o_import_file">Upload Data File</button>
                    </FileInput>
                    <button t-on-click="() => this.exit()" type="button" class="btn btn-secondary">Cancel</button>
                </t>
                <t t-if="isPreviewing">
                    <ImportDataSidepanel
                        filename="state.filename"
                        options="importOptions"
                        formattingOptions="formattingOptions"
                        importTemplates="importTemplates"
                        isBatched="isBatched"
                        onOptionChanged.bind="onOptionChanged"
                        onReload.bind="reload"
                        hasBinaryFields="hasBinaryFields"
                        binaryFilesParams="model.binaryFilesParams"
                        onBinaryFilesParamsChanged.bind="(name, value) => model.onBinaryFilesParamsChanged(name, value)"
                    />
                    <ImportDataContent
                        columns="model.columns"
                        onFieldChanged.bind="onFieldChanged"
                        onOptionChanged.bind="onOptionChanged"
                        options="importOptions"
                        isFieldSet.bind="isFieldSet"
                        previewError="state.previewError"
                        importMessages="model.importMessages"
                    />
                </t>
                <div t-else="" class="o_view_nocontent">
                    <div class="o_nocontent_help">
                        <p class="o_view_nocontent_smiling_face">
                            Drop or upload a file to import
                        </p>
                        <p>
                            Excel files are recommended as formatting is automatic.<br/>
                            But, you can also use .csv files
                        </p>
                        <div class="mt16 mb4">
                            Need Help? <DocumentationLink path="'/applications/general/export_import_data.html'" label.translate="Import FAQ"/>
                        </div>
                        <div t-foreach="importTemplates" t-as="template" t-key="template">
                            <a class="btn btn-outline-primary mb32 mt8" t-att-href="template.template" aria-label="Download" data-tooltip="Download">
                                <i class="fa fa-download"/> <span><t t-esc="template.label"/></span>
                            </a>
                        </div>
                    </div>
                </div>
            </Layout>
        </div>
    </t>
</templates>

```

## File: static\src\import_data_column_error\import_data_column_error.js

```javascript
/** @odoo-module **/

import { Component, useState } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";

export class ImportDataColumnError extends Component {
    static template = "ImportDataColumnError";
    static props = {
        errors: { type: Array },
        fieldInfo: { type: Object },
        resultNames: { type: Array },
    };

    setup() {
        this.action = useService("action");
        this.orm = useService("orm");
        this.state = useState({
            isExpanded: false,
            moreInfoContent: undefined,
        });
    }
    get moreInfo() {
        const moreInfoObjects = this.props.errors.map((error) => error.moreinfo);
        return moreInfoObjects.length && moreInfoObjects[0];
    }
    isErrorVisible(index) {
        return this.state.isExpanded || index < 3;
    }
    onMoreInfoClicked() {
        const moreInfo = this.moreInfo;
        if (this.state.moreInfoContent) {
            this.state.moreInfoContent = undefined;
        } else if (moreInfo instanceof Array) {
            this.state.moreInfoContent = moreInfo;
        } else {
            this.action.doAction(moreInfo);
        }
    }
}

```

## File: static\src\import_data_column_error\import_data_column_error.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="ImportDataColumnError">
        <div class="o_import_report alert alert-danger mb-2">
            <t t-if="props.errors.length > 1">
                <p>
                    <t t-if="props.errors[0].value">No matching records found for the following name </t>
                    <t t-else="">Multiple errors occurred </t>
                      in field <b t-esc="props.fieldInfo.label"/>:
                </p>
                <ul>
                    <t t-foreach="props.errors" t-as="error" t-key="error_index">
                        <a t-if="error_index === 3" href="#" class="o_import_report_count" t-on-click="() => this.state.isExpanded = !this.state.isExpanded">
                            <i t-attf-class="fa fa-chevron-{{state.isExpanded ? 'up' : 'down'}}"></i> <t t-esc="props.errors.length - error_index"/> more
                        </a>
                        <li t-if="isErrorVisible(error_index)" t-attf-class="o_import_report_more p-0 text-{{error.priority}} {{error_index > 2 ? 'list-unstyled' : ''}}">
                            <t t-if="error.rows.from === error.rows.to">
                                <b t-esc="error.value"/> at row <t t-esc="error.rows.from + 1"/>
                                <t t-if="props.resultNames and props.resultNames.length > error.rows.from and props.resultNames[error.rows.from] !== ''">
                                    (<t t-esc="props.resultNames[error.rows.from]"/>)
                                </t>
                            </t>
                            <t t-else=""><b t-esc="error.value || error.message"/> at multiple rows</t>
                        </li>
                    </t>
                </ul>
            </t>
            <p t-else="" t-attf-class="mb-0 p-2 alert-{{props.errors[0].type}}" t-esc="props.errors[0].message" />
            <t t-if="moreInfo">
                <t t-if="typeof moreInfo === 'string'" t-esc="moreInfo"/>
                <div t-else="" class="o_import_moreinfo" t-on-click="onMoreInfoClicked">
                    <a href="#" class="o_import_see_all">
                        <i class="oi oi-arrow-right"></i> See possible values
                    </a>
                </div>
                <ul t-if="state.moreInfoContent" class="o_import_report_more">
                    <li t-foreach="state.moreInfoContent" t-as="msg" t-key="msg_index"><t t-esc="msg"/></li>
                </ul>
            </t>
        </div>
    </t>
</templates>

```

## File: static\src\import_data_content\import_data_content.js

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";
import { SelectMenu } from "@web/core/select_menu/select_menu";
import { ImportDataColumnError } from "../import_data_column_error/import_data_column_error";
import { ImportDataOptions } from "../import_data_options/import_data_options";
import { _t } from "@web/core/l10n/translation";

export class ImportDataContent extends Component {
    static template = "ImportDataContent";
    static components = {
        ImportDataColumnError,
        ImportDataOptions,
        SelectMenu,
    };
    static props = {
        columns: { type: Array },
        isFieldSet: { type: Function },
        onOptionChanged: { type: Function },
        onFieldChanged: { type: Function },
        options: { type: Object },
        importMessages: { type: Object },
        previewError: { type: String, optional: true },
    };

    getGroups(column) {
        const groups = [
            { choices: this.makeChoices(column.fields.basic) },
            { choices: this.makeChoices(column.fields.suggested), label: _t("Suggested Fields") },
            {
                choices: this.makeChoices(column.fields.additional),
                label:
                    column.fields.suggested.length > 0
                        ? _t("Additional Fields")
                        : _t("Standard Fields"),
            },
            { choices: this.makeChoices(column.fields.relational), label: _t("Relation Fields") },
        ];
        return groups;
    }

    makeChoices(fields) {
        return fields.map((field) => ({
            label: field.label,
            value: field.fieldPath,
            iconClass: `o_import_field_icon_${field.type}`,
        }));
    }

    getTooltipDetails(field) {
        return JSON.stringify({
            resModel: field.model_name,
            debug: true,
            field: {
                name: field.name,
                label: field.string,
                type: field.type,
            },
        });
    }

    getTooltip(column) {
        const displayCount = 5;
        if (column.previews.length > displayCount) {
            return JSON.stringify({
                lines: [
                    ...column.previews.slice(0, displayCount - 1),
                    `(+${column.previews.length - displayCount + 1})`,
                ],
            });
        } else {
            return JSON.stringify({ lines: column.previews.slice(0, displayCount) });
        }
    }

    getErrorMessageClass(messages, type, index) {
        return `alert alert-${type} m-0 p-2 ${index === messages.length - 1 ? "" : "mb-2"}`;
    }

    getCommentClass(column, comment, index) {
        return `alert-${comment.type} ${index < column.comments.length - 1 ? "mb-2" : "mb-0"}`;
    }

    onFieldChanged(column, fieldPath) {
        const fields = [
            ...column.fields.basic,
            ...column.fields.suggested,
            ...column.fields.additional,
            ...column.fields.relational,
        ];
        const fieldInfo = fields.find((f) => f.fieldPath === fieldPath);
        this.props.onFieldChanged(column, fieldInfo);
    }
}

```

## File: static\src\import_data_content\import_data_content.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="ImportDataContent">
        <div class="o_import_data_content flex-grow-1 overflow-auto border-start">
            <div t-if="props.previewError" class="border-bottom p-2">
                <div class="alert alert-danger mb-0">
                    <p>
                        Import preview failed due to: "
                        <t t-esc="props.previewError" />
                        ".
                    </p>
                    <p>For CSV files, you may need to select the correct separator.</p>
                    <p t-if="props.columns.length > 0">Here is the start of the file we could not import:</p>
                </div>
            </div>
            <div t-if="!props.options.has_headers" class="border-bottom p-2">
                <p class="alert alert-info mb-0">
                    If the file contains
                    the column names, Odoo can try auto-detecting the
                    field corresponding to the column. This makes imports
                    simpler especially when the file has many columns.
                </p>
            </div>
            <div t-if="props.importMessages.length > 0" class="border-bottom p-2">
                <t t-foreach="props.importMessages" t-as="message" t-key="`${message.lines[0]}-${message_index}`">
                    <p t-att-class="getErrorMessageClass(props.importMessages, message.type, message_index)">
                        <t t-foreach="message.lines" t-as="line" t-key="`${line}-${line_index}`">
                            <b t-esc="line" /><br/>
                        </t>
                    </p>
                </t>
            </div>
            <table t-if="!props.previewError" class="table table-borderless w-100 bg-view">
                <thead>
                    <tr class="border-bottom">
                        <th scope="col">File Column</th>
                        <th scope="col">Odoo Field</th>
                        <th scope="col">Comments</th>
                    </tr>
                </thead>
                <tbody>
                    <t t-foreach="props.columns" t-as="column" t-key="`${column_index}-${column.id}`">
                        <tr class="border-bottom align-middle">
                            <td>
                                <div class="o_import_file_column_cell d-flex flex-column user-select-none">
                                    <span class="fw-bold text-truncate">
                                        <span class="text-truncate" t-if="column.name" t-esc="column.name" />
                                        <span t-else="">Untitled</span>
                                    </span>
                                    <span
                                        t-if="props.options.has_headers"
                                        class="o_import_preview fst-italic small pe-2"
                                        data-tooltip-template="ImportDataContent.tooltip"
                                        t-att-data-tooltip-info="getTooltip(column)"
                                        data-tooltip-position="right"
                                    >
                                        <t t-esc="column.preview" />
                                    </span>
                                </div>
                            </td>
                            <td>
                                <SelectMenu
                                    value="column.fieldInfo ? column.fieldInfo.fieldPath : undefined"
                                    groups="getGroups(column)"
                                    onSelect="(fieldPath) => this.onFieldChanged(column, fieldPath)"
                                    searchPlaceholder.translate="Search a field..."
                                    togglerClass="column.fieldInfo ? 'ps-1' : ''"
                                >
                                    <t t-if="column.fieldInfo">
                                        <i
                                            class="o_import_field_icon border-end"
                                            t-att-class="'position-absolute d-inline-block o_import_field_icon_' + column.fieldInfo.type"
                                            data-tooltip-template="web.FieldTooltip"
                                            t-att-data-tooltip-info="getTooltipDetails(column.fieldInfo)"
                                        ></i>
                                        <span class="ps-5">
                                            <t t-esc="column.fieldInfo.label" />
                                        </span>
                                    </t>
                                    <span t-else="" class="text-warning">To import, select a field...</span>

                                    <t t-set-slot="choice" t-slot-scope="choice">
                                        <div t-att-class="`${choice.data.value and choice.data.value.required ? 'fw-bolder text-decoration-underline' : ''}`">
                                            <i class="o_import_field_choice o_import_field_icon position-relative d-inline-block align-middle border-end me-2"
                                                t-att-class="choice.data.iconClass"/>
                                            <t t-esc="choice.data.label" />
                                        </div>
                                    </t>
                                </SelectMenu>
                            </td>
                            <td>
                                <t t-foreach="column.comments" t-as="comment" t-key="comment_index">
                                    <p class="m-0 p-2 alert" t-att-class="getCommentClass(column, comment, comment_index)"><t t-out="comment.content" /> <b t-if="comment.fieldName" t-out="comment.fieldName"/></p>
                                </t>
                                <ImportDataColumnError t-if="column.errors.length" errors="column.errors" resultNames="column.resultNames" fieldInfo="column.fieldInfo" />
                                <ImportDataOptions t-if="column.errors.length or column.importOptions" importOptions="column.importOptions" fieldInfo="column.fieldInfo" onOptionChanged="props.onOptionChanged" />
                            </td>
                        </tr>
                    </t>
                </tbody>
            </table>
        </div>
    </t>

    <t t-name="ImportDataContent.tooltip">
        <h5 class="text-reset p-2 m-0">Preview</h5>
        <hr class="m-0"/>
        <div class="p-2 pe-3">
            <p
                t-foreach="lines"
                t-as="line"
                t-key="line_index"
                t-out="line"
                class="mb-0 fs-6"
            ></p>
        </div>
    </t>
</templates>

```

## File: static\src\import_data_options\import_data_options.js

```javascript
/** @odoo-module **/

import { Component, useState, onWillStart } from "@odoo/owl";
import { _t } from "@web/core/l10n/translation";
import { useService } from "@web/core/utils/hooks";

export class ImportDataOptions extends Component {
    static template = "ImportDataOptions";
    static props = {
        importOptions: { type: Object, optional: true },
        fieldInfo: { type: Object },
        onOptionChanged: { type: Function },
    };

    setup() {
        this.orm = useService("orm");
        this.state = useState({
            options: [],
        });
        this.currentModel = this.props.fieldInfo.comodel_name || this.props.fieldInfo.model_name;
        onWillStart(async () => {
            this.state.options = await this.loadOptions();
        });
    }
    get isVisible() {
        return ["many2one", "many2many", "selection", "boolean"].includes(
            this.props.fieldInfo.type
        );
    }
    async loadOptions() {
        const options = [["prevent", _t("Prevent import")]];
        if (this.props.fieldInfo.type === "boolean") {
            options.push(["false", _t("Set to: False")]);
            options.push(["true", _t("Set to: True")]);
            !this.props.fieldInfo.required &&
                options.push(["import_skip_records", _t("Skip record")]);
        }
        if (["many2one", "many2many", "selection"].includes(this.props.fieldInfo.type)) {
            if (!this.props.fieldInfo.required) {
                options.push(["import_set_empty_fields", _t("Set value as empty")]);
                options.push(["import_skip_records", _t("Skip record")]);
            }
            if (this.props.fieldInfo.type === "selection") {
                const fields = await this.orm.call(this.currentModel, "fields_get");
                const selection = fields[this.props.fieldInfo.name].selection.map((opt) => [
                    opt[0],
                    _t("Set to: %s", opt[1]),
                ]);
                options.push(...selection);
            } else {
                options.push(["name_create_enabled_fields", _t("Create new values")]);
            }
        }
        return options;
    }
    onSelectionChanged(ev) {
        if (
            [
                "name_create_enabled_fields",
                "import_set_empty_fields",
                "import_skip_records",
            ].includes(ev.target.value)
        ) {
            this.props.onOptionChanged(
                ev.target.value,
                ev.target.value,
                this.props.fieldInfo.fieldPath
            );
        } else {
            const value = {
                fallback_value: ev.target.value,
                field_model: this.currentModel,
                field_type: this.props.fieldInfo.type,
            };
            this.props.onOptionChanged("fallback_values", value, this.props.fieldInfo.fieldPath);
        }
    }
}

```

## File: static\src\import_data_options\import_data_options.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="ImportDataOptions">
        <div t-if="isVisible and state.options.length > 1" t-attf-class="o_import_dropdown o_import_field_#{props.fieldInfo.type} alert alert-danger mb-2">
            <b>When a value cannot be matched:</b>
            <select class="o_import_create_option form-select w-auto bg-light" t-att-type="props.fieldInfo.type" t-on-change="onSelectionChanged">
                <option t-foreach="state.options" t-as="option" t-key="option" t-att-value="option[0]">
                    <t t-esc="option[1]"/>
                </option>
            </select>
        </div>
    </t>
</templates>

```

## File: static\src\import_data_progress\import_data_progress.js

```javascript
/** @odoo-module **/

import { Component, useEffect, useState } from "@odoo/owl";

export class ImportDataProgress extends Component {
    static template = "ImportDataProgress";
    static props = {
        importProgress: { type: Object },
        stopImport: { type: Function },
        totalSteps: { type: Number },
    };

    setup() {
        this.timer = undefined;
        this.timeStart = Date.now();
        this.state = useState({
            isInterrupted: false,
            timeLeft: null,
        });

        useEffect(
            () => {
                this.updateTimer();
                return () => {
                    clearInterval(this.timer);
                };
            },
            () => []
        );
    }

    get minutesLeft() {
        return this.state.timeLeft.toFixed(2);
    }

    get secondsLeft() {
        return Math.round(this.state.timeLeft * 60);
    }

    interrupt() {
        this.state.isInterrupted = true;
        this.props.stopImport();
    }

    updateTimer() {
        if (this.timer) {
            clearInterval(this.timer);
        }
        this.state.timeLeft =
            ((Date.now() - this.timeStart) *
                ((100 - this.props.importProgress.value) / this.props.importProgress.value)) /
            60000;
        this.timer = setInterval(() => this.updateTimer(), 1000);
    }
}

```

## File: static\src\import_data_progress\import_data_progress.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="ImportDataProgress">
        <div t-if="props.importProgress.step > 1" class="o_import_data_progress d-flex align-items-center flex-column">
            <div class="o_import_progress_dialog_batch text-center">
                Batch <span class="o_import_progress_dialog_batch_count"><t t-esc="props.importProgress.step"/></span> out of <span t-esc="props.totalSteps"/>...
                <div t-if="props.importProgress.value and state.timeLeft and !state.isInterrupted" class="o_import_progress_dialog_time_left">
                    <span>Estimated time left:</span>
                    <t t-if="state.timeLeft >= 1">
                        <span class="o_import_progress_dialog_time_left_text mx-1" t-esc="minutesLeft"/><span>minutes</span>
                    </t>
                    <t t-else="">
                        <span class="o_import_progress_dialog_time_left_text mx-1" t-esc="secondsLeft"/><span>seconds</span>
                    </t>
                </div>
            </div>
            <span t-if="state.isInterrupted" class="o_import_progress_dialog_stop">
                Finalizing current batch before interrupting...
            </span>
            <div t-else="" class="d-flex align-items-center mt-2">
                <div class="progress flex-grow-1 rounded-3">
                    <div class="progress-bar progress-bar-striped" role="progressbar"
                         aria-valuenow="props.importProgress" aria-valuemin="0" aria-valuemax="100" aria-label="Progress bar"
                         t-att-style="'width: ' + props.importProgress.value + '%'">
                        <span class="fs-4"><t t-esc="props.importProgress.value" />%</span>
                    </div>
                </div>
                <a class="o_progress_stop_import ms-2" role="button" t-on-click="interrupt"><i class="fa fa-close fs-2 text-danger" aria-label="Stop Import" title="Stop Import"/></a>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\import_data_sidepanel\import_data_sidepanel.js

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";
import { CheckBox } from "@web/core/checkbox/checkbox";
import { _t } from "@web/core/l10n/translation";
import { DocumentationLink } from "@web/views/widgets/documentation_link/documentation_link";

export class ImportDataSidepanel extends Component {
    static template = "ImportDataSidepanel";
    static components = { CheckBox, DocumentationLink };
    static props = {
        filename: { type: String },
        formattingOptions: { type: Object, optional: true },
        options: { type: Object },
        importTemplates: { type: Array, optional: true },
        isBatched: { type: Boolean, optional: true },
        onOptionChanged: { type: Function },
        onReload: { type: Function },
        hasBinaryFields: { type: Boolean },
        binaryFilesParams: { type: Object },
        onBinaryFilesParamsChanged: { type: Function },
    };

    get fileName() {
        return this.props.filename.split(".")[0];
    }

    get fileExtension() {
        return "." + this.props.filename.split(".").pop();
    }

    getOptionValue(name) {
        if (name === "skip") {
            return (this.props.options.skip + 1).toString();
        }
        return this.props.options[name].toString();
    }

    setOptionValue(name, value) {
        this.props.onOptionChanged(name, isNaN(parseFloat(value)) ? value : Number(value));
    }

    // Start at row 1 = skip 0 lines
    onLimitChange(ev) {
        this.props.onOptionChanged("skip", ev.target.value ? ev.target.value - 1 : 0);
    }

    get binaryFilesLabel() {
        const files = this.props.binaryFilesParams.binaryFiles.value;
        const number = Object.keys(files).length;
        if (number > 0) {
            return _t("%(number)s file(s) selected", { number });
        }
        return _t("No file selected");
    }
}

```

## File: static\src\import_data_sidepanel\import_data_sidepanel.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="ImportDataSidepanel">
        <div class="o_import_data_sidepanel p-3 bg-view">
            <div class="pb-4">
                <h4>Data to import</h4>
                <div class="mb-2 d-flex align-items-center">
                    <i class="fa fa-file white me-2"></i>
                    <div class="fst-italic truncate"><t t-esc="fileName"/></div>
                    <span class="fst-italic"><t t-esc="fileExtension"/></span>
                </div>

                <div t-if="props.options.sheets.length > 0" class="flex-row my-2">
                    <label for="o_import_sheet">Sheet:</label>
                    <select
                        class="o_select o_import_sheet w-100 mb-3 form-select"
                        name="o_import_sheet"
                        t-on-change="ev => this.props.onOptionChanged('sheet', ev.target.value)"
                        t-att-value="props.options.sheet"
                    >
                        <option t-foreach="props.options.sheets" t-as="sheet" t-key="sheet" t-att-value="sheet" t-att-selected="sheet === props.options.sheet">
                            <t t-esc="sheet" />
                        </option>
                    </select>
                </div>
                <CheckBox value="props.options.has_headers" onChange.bind="isChecked => this.props.onOptionChanged('has_headers', isChecked)">
                    Use first row as header
                </CheckBox>
            </div>
            <div t-if="fileExtension === '.csv'" class="o_import_formatting pt-3 pb-4 border-top">
                <h4>Formatting</h4>
                <div t-foreach="Object.entries(props.formattingOptions)" t-as="option" t-key="`${option[0]}-${option_index}`" class="o_import_options">
                    <label t-att-for="`${option[0]}-${option_index}`">
                        <t t-esc="option[1].label" />
                        <sup t-if="option[1].help" class="text-info p-1" t-att-data-tooltip="option[1].help" t-esc="'?'" />
                    </label>
                    <input
                        t-if="option[1].type === 'input'"
                        t-att-id="`${option[0]}-${option_index}`"
                        t-att-list="option[1].options ? `list-${ option_index }` : null"
                        t-attf-class="o_import_#{option[0]} o_import_dropdown w-100 mb-3"
                        t-att-value="option[1].value"
                        t-on-change="(ev) => this.setOptionValue(option[0], ev.target.value)"
                    />
                    <datalist t-if="option[1].type === 'input' and option[1].options" t-att-id="`list-${ option_index }`">
                        <option t-foreach="option[1].options" t-as="opt" t-key="opt_index" t-att-value="opt.value or opt" t-att-selected="opt.value ? opt.value === option[1].value : opt === option[1].value">
                            <t t-esc="opt.label ? opt.label : opt" />
                        </option>
                    </datalist>
                    <select
                        t-elif="option[1].type === 'select'"
                        t-att-class="`o_select form-select w-100 mb-3 o_import_${option[0]} o_import_dropdown`"
                        t-att-name="`${option[0]}-${option_index}`"
                        t-on-change="(ev) => this.setOptionValue(option[0], ev.target.value)"
                    >
                        <option t-foreach="option[1].options" t-as="opt" t-key="opt_index" t-att-value="opt.value or opt.value === '' ? opt.value : opt" t-att-selected="opt.value or opt.value === '' ? opt.value === option[1].value : opt === option[1].value">
                            <t t-esc="opt.label ? opt.label : opt" />
                        </option>
                    </select>
                </div>
                <button class="btn btn-primary w-100" t-on-click="() => this.props.onReload()">Reimport</button>
            </div>
            <div t-if="props.isBatched" class="pt-3 pb-4 border-top" >
                <h4>Batch Import</h4>
                <div class="o_import_batch_alert fw-bold pb-2">
                    The file will be imported by batches
                </div>
                <div class="d-flex">
                    <div class="o_import_batch_limit w-50 pe-1">
                        <label class="mb-1" for="o_import_batch_limit">Batch limit</label>
                        <input class="w-100" id="o_import_batch_limit" t-att-value="getOptionValue('limit')" t-on-change="(ev) => this.setOptionValue('limit', ev.target.value || 1)" />
                    </div>
                    <div class="w-50 ps-1" data-tooltip="Warning: ignores the labels line, empty lines and lines composed only of empty cells">
                        <label class="mb-1" for="o_import_row_start">Start at line</label>
                        <input class="w-100" id="o_import_row_start" t-att-value="getOptionValue('skip')" t-on-change="onLimitChange" />
                    </div>
                </div>
            </div>
            <div t-if="props.hasBinaryFields" class="pt-3 pb-4 border-top o_import_file">
                <h4>Files to import</h4>
                <div class="o_import_grid gap-1">
                    <!-- First row-->
                    <div class="o_import_grid_col">
                        <label>
                            <span class="btn btn-sm btn-primary">Upload your files</span>
                            <input type="file" multiple="multiple" hidden="hidden"
                                   t-on-change="(event) => this.props.onBinaryFilesParamsChanged('binaryFiles', event.target.files)"/>
                        </label>
                    </div>
                    <div class="o_import_grid_col o_grid_span_2">
                        <t t-esc="binaryFilesLabel"/>
                    </div>

                    <!-- Second row-->
                    <t t-if="env.debug">
                        <div class="o_import_grid_col">
                            <label for="o_input_maxSizePerBatch">Max size per batch</label>
                            <sup t-if="props.binaryFilesParams.maxSizePerBatch.help" class="text-info p-1 cursor-default" t-att-data-tooltip="props.binaryFilesParams.maxSizePerBatch.help" t-esc="'?'"/>
                        </div>
                        <div class="o_import_grid_col">
                                <input type="number"
                                       id="o_input_maxSizePerBatch"
                                       t-att-value="props.binaryFilesParams.maxSizePerBatch.value"
                                       t-att-max="props.binaryFilesParams.maxSizePerBatch.max"
                                       t-att-min="props.binaryFilesParams.maxSizePerBatch.min"
                                       t-on-change="(event) => this.props.onBinaryFilesParamsChanged('maxSizePerBatch', event.target.value)"/>
                        </div>
                        <div class="o_import_grid_col">
                            <label for="o_input_maxSizePerBatch">Mb</label>
                        </div>

                        <!-- Third row-->
                        <div class="o_import_grid_col">
                            <label for="o_input_delayAfterEachBatch">Delay after each batch</label>
                            <sup t-if="props.binaryFilesParams.delayAfterEachBatch.help" class="text-info p-1 cursor-default" t-att-data-tooltip="props.binaryFilesParams.delayAfterEachBatch.help" t-esc="'?'"/>
                        </div>
                        <div class="o_import_grid_col">
                                <input type="number"
                                       id="o_input_delayAfterEachBatch"
                                       t-att-value="props.binaryFilesParams.delayAfterEachBatch.value"
                                       t-att-min="props.binaryFilesParams.delayAfterEachBatch.min"
                                       t-on-change="(event) => this.props.onBinaryFilesParamsChanged('delayAfterEachBatch', event.target.value)"/>
                        </div>
                        <div class="o_import_grid_col">
                            <label for="o_input_delayAfterEachBatch">seconds</label>
                        </div>
                    </t>
                </div>

            </div>
            <div t-if="env.debug" class="o_import_debug_options pt-3 pb-4 border-top">
                <h4>Advanced</h4>
                <div data-tooltip="If the model uses openchatter, history tracking will set up subscriptions and send notifications during the import, but lead to a slower import.">
                    <CheckBox value="!props.options.tracking_disable" onChange.bind="(isChecked) => this.props.onOptionChanged('tracking_disable', !isChecked)">
                        Track history during import
                    </CheckBox>
                </div>
                <CheckBox value="props.options.advanced" onChange.bind="(isChecked) => this.props.onOptionChanged('advanced', isChecked)">
                    Allow matching with subfields
                </CheckBox>
            </div>
            <div class="pt-3 pb-4 border-top">
                <h4>Help</h4>
                <t t-foreach="props.importTemplates" t-as="template" t-key="template">
                    <a class="d-block my-2" t-att-href="template.template" aria-label="Download" data-tooltip="Download">
                        <i class="fa fa-download"/> <span><t t-esc="template.label"/></span>
                    </a>
                </t>
                <DocumentationLink path="'/applications/general/export_import_data.html'" icon="'fa-external-link'" label.translate="Go to Import FAQ"/>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\import_records\import_records.js

```javascript
/** @odoo-module **/

import { Component } from "@odoo/owl";
import { DropdownItem } from "@web/core/dropdown/dropdown_item";
import { registry } from "@web/core/registry";
import { useService } from "@web/core/utils/hooks";
import { exprToBoolean } from "@web/core/utils/strings";
import { STATIC_ACTIONS_GROUP_NUMBER } from "@web/search/action_menus/action_menus";

const cogMenuRegistry = registry.category("cogMenu");

/**
 * 'Import records' menu
 *
 * This component is used to import the records for particular model.
 * @extends Component
 */
export class ImportRecords extends Component {
    static template = "base_import.ImportRecords";
    static components = { DropdownItem };
    static props = {};

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
            params: { active_model: resModel, context },
        });
    }
}

export const importRecordsItem = {
    Component: ImportRecords,
    groupNumber: STATIC_ACTIONS_GROUP_NUMBER,
    isDisplayed: ({ config, isSmall }) =>
        !isSmall &&
        config.actionType === "ir.actions.act_window" &&
        ["kanban", "list"].includes(config.viewType) &&
        exprToBoolean(config.viewArch.getAttribute("import"), true) &&
        exprToBoolean(config.viewArch.getAttribute("create"), true),
};

cogMenuRegistry.add("import-menu", importRecordsItem, { sequence: 1 });

```

## File: static\src\import_records\import_records.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-name="base_import.ImportRecords">
        <DropdownItem class="'o_import_menu'" onSelected.bind="importRecords">
            <i class="fa fa-fw fa-download me-1"/>Import records
        </DropdownItem>
    </t>

</templates>

```

