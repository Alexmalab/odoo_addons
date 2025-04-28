# Odoo Module: sale_pdf_quote_builder

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import io

from odoo import _
from odoo.exceptions import ValidationError
from odoo.tools import pdf


def _ensure_document_not_encrypted(document):
    if pdf.PdfFileReader(io.BytesIO(document), strict=False).isEncrypted:
        raise ValidationError(_(
            "It seems that we're not able to process this pdf inside a quotation. It is either"
            " encrypted, or encoded in a format we do not support."
        ))


def _get_form_fields_from_pdf(pdf_data):
    """Get the form text fields present in the pdf file.

    :param binary pdf_data: the pdf from where we should extract the new form fields that might
                            need to be mapped.
    :return: set of form fields that are in the pdf.
    :rtype: set
    """
    reader = pdf.PdfFileReader(io.BytesIO(base64.b64decode(pdf_data)), strict=False)

    return set(reader.getFormTextFields() or {})

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Sales PDF Quotation Builder",
    'category': 'Sales/Sales',
    'description': "Build nice quotations",
    'depends': ['sale_management'],
    'data': [
        'data/ir_cron.xml',
        'data/sale_pdf_form_field.xml',

        'report/ir_actions_report.xml',

        'security/ir.model.access.csv',
        'security/ir_rules.xml',

        'views/product_document_views.xml',
        'views/quotation_document_views.xml',
        'views/sale_order_template_views.xml',
        'views/sale_order_views.xml',
        'views/sale_pdf_form_field_views.xml',
        'views/sale_pdf_quote_builder_menus.xml',

        'wizards/res_config_settings_views.xml',
    ],
    'demo': [
        'data/sale_pdf_quote_builder_demo.xml',
    ],
    'auto_install': True,
    'assets': {
        'web.assets_backend': [
            'sale_pdf_quote_builder/static/src/js/**/*',
        ],
        'web.assets_tests': [
            'sale_pdf_quote_builder/static/tests/tours/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\quotation_document.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import json
import logging

from odoo import _
from odoo.http import Controller, request, route

from odoo.addons.sale_pdf_quote_builder import utils

logger = logging.getLogger(__name__)


class QuotationDocumentController(Controller):

    @route(
        '/sale_pdf_quote_builder/quotation_document/upload',
        type='http',
        methods=['POST'],
        auth='user',
    )
    def upload_document(self, ufile, sale_order_template_id=False):
        # TODO: add `allowed_company_ids` as method param in master
        if allowed_company_ids := request.params.get('allowed_company_ids'):
            request.update_context(allowed_company_ids=json.loads(allowed_company_ids))
        sale_order_template = request.env['sale.order.template'].browse(
            int(sale_order_template_id)
        )
        company = sale_order_template.company_id if sale_order_template else request.env.company
        files = request.httprequest.files.getlist('ufile')
        result = {'success': _("All files uploaded")}
        for ufile in files:
            try:
                mimetype = ufile.content_type
                doc = request.env['quotation.document'].create({
                    'name': ufile.filename,
                    'mimetype': mimetype,
                    'raw': ufile.read(),
                    'quotation_template_ids': sale_order_template.ids,
                    'company_id': company.id,
                })
                # pypdf will also catch malformed document
                utils._ensure_document_not_encrypted(base64.b64decode(doc.datas))
            except Exception as e:
                logger.exception("Failed to upload document %s", ufile.filename)
                result = {'error': str(e)}

        return json.dumps(result)

```

## File: controllers\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import quotation_document

```

## File: data\ir_config_parameters.xml

```xml

```

## File: data\ir_cron.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.cron" id="cron_post_upgrade_assign_missing_form_fields">
        <field name="name">Sale Pdf Quote Builder: assign form fields to documents post upgrade</field>
        <field name="model_id" ref="sale_pdf_quote_builder.model_sale_pdf_form_field" />
        <field name="state">code</field>
        <field name="code">model._cron_post_upgrade_assign_missing_form_fields()</field>
        <field name="user_id" ref="base.user_root" />
        <field name="interval_number">9999</field>
        <field name="interval_type">months</field>
    </record>

</odoo>

```

## File: data\sale_pdf_form_field.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">

    <function model="sale.pdf.form.field" name="_add_basic_mapped_form_fields"/>

</odoo>

```

## File: data\sale_pdf_quote_builder_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="sale_pdf_header_demo_page" model="quotation.document">
        <field name="name">Header</field>
        <field name="datas" type="base64" file="sale_pdf_quote_builder/static/quote_builder_pdf/quote_header.pdf"/>
        <field name="mimetype">application/pdf</field>
        <field name="document_type">header</field>
        <field name="sequence">15</field>
    </record>

    <record id="sale_pdf_header_about_us_demo_page" model="quotation.document">
        <field name="name">About Us</field>
        <field name="datas" type="base64" file="sale_pdf_quote_builder/static/quote_builder_pdf/quote_header_about_us.pdf"/>
        <field name="mimetype">application/pdf</field>
        <field name="document_type">header</field>
        <field name="sequence">25</field>
    </record>

    <record id="sale_pdf_header_project_description_demo_page" model="quotation.document">
        <field name="name">Project Description</field>
        <field name="datas" type="base64" file="sale_pdf_quote_builder/static/quote_builder_pdf/quote_header_project_description.pdf"/>
        <field name="mimetype">application/pdf</field>
        <field name="document_type">header</field>
        <field name="sequence">30</field>
    </record>

    <record id="sale_pdf_footer_testimonials_demo_page" model="quotation.document">
        <field name="name">Testimonials</field>
        <field name="datas" type="base64" file="sale_pdf_quote_builder/static/quote_builder_pdf/quote_footer_testimonials.pdf"/>
        <field name="mimetype">application/pdf</field>
        <field name="document_type">footer</field>
        <field name="sequence">35</field>
    </record>

    <record id="sale_pdf_footer_terms_demo_page" model="quotation.document">
        <field name="name">Terms and Conditions</field>
        <field name="datas" type="base64" file="sale_pdf_quote_builder/static/quote_builder_pdf/quote_footer_terms.pdf"/>
        <field name="mimetype">application/pdf</field>
        <field name="document_type">footer</field>
        <field name="sequence">40</field>
    </record>

    <record id="consu_delivery_02_document" model="product.document">
        <field name="name">Large Meeting Table Document</field>
        <field name="datas" type="base64" file="sale_pdf_quote_builder/static/quote_builder_pdf/large_meeting_table_with_form_fields.pdf"/>
        <field name="mimetype">application/pdf</field>
        <field name="res_model">product.template</field>
        <field name="attached_on_sale">inside</field>
        <field name="res_id" ref="product.consu_delivery_02_product_template"/>
    </record>

    <record id="product.product_product_4_product_template_document" model="product.document">
        <field name="attached_on_sale">inside</field>
    </record>

    <record id="product.product_product_25_document" model="product.document">
        <field name="attached_on_sale">inside</field>
    </record>

    <record id="sale_management.sale_order_template_1" model="sale.order.template">
        <field
            name="quotation_document_ids"
            eval="[
                Command.link(ref('sale_pdf_header_demo_page')),
                Command.link(ref('sale_pdf_header_project_description_demo_page')),
                Command.link(ref('sale_pdf_footer_testimonials_demo_page')),
                Command.link(ref('sale_pdf_footer_terms_demo_page')),
            ]"
        />
    </record>

</odoo>

```

## File: models\ir_actions_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import io
import json

from odoo import _, api, models
from odoo.tools import format_amount, format_date, format_datetime, pdf
from odoo.tools.pdf import PdfFileWriter, PdfFileReader, NameObject, NumberObject, createStringObject


class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    def _render_qweb_pdf_prepare_streams(self, report_ref, data, res_ids=None):
        """Override to add and fill headers, footers and product documents to the sale quotation."""
        result = super()._render_qweb_pdf_prepare_streams(report_ref, data, res_ids=res_ids)
        if self._get_report(report_ref).report_name != 'sale.report_saleorder':
            return result

        orders = self.env['sale.order'].browse(res_ids)

        for order in orders:
            initial_stream = result[order.id]['stream']
            if initial_stream:
                quotation_documents = order.quotation_document_ids
                headers = quotation_documents.filtered(lambda doc: doc.document_type == 'header')
                footers = quotation_documents - headers
                has_product_document = any(line.product_document_ids for line in order.order_line)

                if not headers and not has_product_document and not footers:
                    continue

                form_fields_values_mapping = {}
                writer = PdfFileWriter()

                self_with_order_context = self.with_context(
                    use_babel=True, lang=order._get_lang() or self.env.user.lang
                )

                if headers:
                    for header in headers:
                        prefix = f'quotation_document_id_{header.id}__'
                        self_with_order_context._update_mapping_and_add_pages_to_writer(
                            writer, header, form_fields_values_mapping, prefix, order
                        )
                if has_product_document:
                    for line in order.order_line:
                        for doc in line.product_document_ids:
                            # Use both the id of the line and the doc as variants could use the same
                            # document.
                            prefix = f'sol_id_{line.id}_product_document_id_{doc.id}__'
                            self_with_order_context._update_mapping_and_add_pages_to_writer(
                                writer, doc, form_fields_values_mapping, prefix, order, line
                            )
                self._add_pages_to_writer(writer, initial_stream.getvalue())
                if footers:
                    for footer in footers:
                        prefix = f'quotation_document_id_{footer.id}__'
                        self_with_order_context._update_mapping_and_add_pages_to_writer(
                            writer, footer, form_fields_values_mapping, prefix, order
                        )
                pdf.fill_form_fields_pdf(writer, form_fields=form_fields_values_mapping)
                with io.BytesIO() as _buffer:
                    writer.write(_buffer)
                    stream = io.BytesIO(_buffer.getvalue())
                result[order.id].update({'stream': stream})

        return result

    @api.model
    def _update_mapping_and_add_pages_to_writer(
        self, writer, document, form_fields_values_mapping, prefix, order, order_line=None
    ):
        """ Update the mapping with the field-value of the document, and add the doc to the writer.

        Note: document.ensure_one(), order.ensure_one(), order_line and order_line.ensure_one()

        :param PdfFileWriter writer: the writer to which pages needs to be added
        :param recordset document: the document that needs to be added to the writer and get its
                                   form fields mapped. Either a quotation.document or a
                                   product.document.
        :param dict form_fields_values_mapping: the existing prefixed form field names - values that
                                                will be updated to add those of the current document
        :param str prefix: the prefix needed to update existing form field name, to be able to add
                           the correct values in fields with the same name but on different
                           documents, either customizable fields or dynamic fields of different sale
                           order lines.
        :param recordset order: the sale order from where to take the values
        :param recordset order_line: the sale order line from where to take the values (optional)
        return: None
        """
        document.ensure_one()
        order.ensure_one()
        order_line and order_line.ensure_one()

        for form_field in document.form_field_ids:
            if form_field.path:  # Dynamic field
                field_value = self._get_value_from_path(form_field, order, order_line)
            else:  # Customizable field
                field_value = self._get_custom_value_from_order(
                    document, form_field.name, order, order_line
                )
            form_fields_values_mapping[prefix + form_field.name] = field_value

        # Avoid useless update of the pdf when no form field and just add the pdf
        prefix = prefix if document.form_field_ids else None
        decoded_document = base64.b64decode(document.datas)
        self._add_pages_to_writer(writer, decoded_document, prefix)

    @api.model
    def _get_value_from_path(self, form_field, order, order_line=None):
        """ Get the string value by following the path indicated in the record form_field.

        :param recordset form_field: sale.pdf.form.field that has a valid path.
        :param recordset order: sale.order from where the values and timezone need to be taken
        :param recordset order_line: sale.order.line from where the values need to be taken
                                     (optional, only for product.document)
        :return: value that need to be shown in the final pdf. Multiple values are joined by ', '
        :rtype: str
        """
        tz = order.partner_id.tz or order.env.user.tz or 'UTC'
        base_record = order_line or order
        path = form_field.path

        # If path = 'order_id.order_line.product_id.name'
        path = path.split('.')  # ['order_id', 'order_line', 'product_id', 'name']
        # Sudo to be able to follow the path set by the admin
        records = base_record.sudo().mapped('.'.join(path[:-1]))  # product.product(id1, id2, ...)
        field_name = path[-1]  # 'name'

        def _get_formatted_value(self):
            # self must be named so to be considered in the translation logic
            field_ = records._fields[field_name]
            field_type_ = field_.type
            for record_ in records:
                value_ = record_[field_name]
                if field_type_ == 'boolean':
                    formatted_value_ = _("Yes") if value_ else _("No")
                elif field_type_ == 'monetary':
                    currency_id_ = record_[field_.get_currency_field(record_)]
                    formatted_value_ = format_amount(
                        self.env, value_, currency_id_ or order.currency_id
                    )
                elif not value_:
                    formatted_value_ = ''
                elif field_type_ == 'date':
                    formatted_value_ = format_date(self.env, value_)
                elif field_type_ == 'datetime':
                    formatted_value_ = format_datetime(self.env, value_, tz=tz, dt_format=False)
                elif field_type_ == 'selection' and value_:
                    formatted_value_ = dict(field_._description_selection(self.env))[value_]
                elif field_type_ in {'one2many', 'many2one', 'many2many'}:
                    formatted_value_ = ', '.join([v.display_name for v in value_])
                else:
                    formatted_value_ = str(value_)

                yield formatted_value_

        return ', '.join(_get_formatted_value(self))

    @api.model
    def _get_custom_value_from_order(self, document, form_field_name, order, order_line):
        """ Get the custom value of a form field directly from the order.

        :param recordset document: the document that needs to be added to the writer and get its
                                   form fields mapped. Either a quotation.document or a
                                   product.document.
        :param str form_field_name: the name of the form field as present in the PDF.
        :param recordset order: the sale order from where to take the existing mapping.
        :param recordset order_line: the sale order line linked to the document (optional)
        :return: value that need to be shown in the final pdf.
        :rtype: str
        """
        existing_mapping = json.loads(order.customizable_pdf_form_fields)
        if order_line:
            base_values = existing_mapping.get('line', {}).get(str(order_line.id), {})
        elif document.document_type == 'header':
            base_values = existing_mapping.get('header', {})
        else:
            base_values = existing_mapping.get('footer', {})
        custom_form_fields = base_values.get(str(document.id), {}).get('custom_form_fields')
        return custom_form_fields.get(form_field_name, "")

    @api.model
    def _add_pages_to_writer(self, writer, document, prefix=None):
        """Add a PDF doc to the writer and fill the form text fields present in the pages if needed.

        :param PdfFileWriter writer: the writer to which pages needs to be added
        :param bytes document: the document to add in the final pdf
        :param str prefix: the prefix needed to update existing form field name, if any, to be able
                           to add the correct values in fields with the same name but on different
                           documents, either customizable fields or dynamic fields of different sale
                           order lines. (optional)
        :return: None
        """
        reader = PdfFileReader(io.BytesIO(document), strict=False)

        field_names = set()
        if prefix:
            field_names = reader.getFormTextFields()

        for page_id in range(reader.getNumPages()):
            page = reader.getPage(page_id)
            if prefix and page.get('/Annots'):
                # Modifying the annots that hold every information about the form fields
                for j in range(len(page['/Annots'])):
                    reader_annot = page['/Annots'][j].getObject()
                    if reader_annot.get('/T') in field_names:
                        # Prefix all form fields in the document with the document identifier.
                        # This is necessary to know which value needs to be taken when filling the forms.
                        form_key = reader_annot.get('/T')
                        new_key = prefix + form_key

                        # Modifying the form flags to force some characteristics
                        # 1. make all text fields read-only
                        # 2. make all text fields support multiline
                        form_flags = reader_annot.get('/Ff', 0)
                        readonly_flag = 1  # 1st bit sets readonly
                        multiline_flag = 1 << 12  # 13th bit sets multiline text
                        new_flags = form_flags | readonly_flag | multiline_flag

                        reader_annot.update({
                            NameObject("/T"): createStringObject(new_key),
                            NameObject("/Ff"): NumberObject(new_flags),
                        })
            writer.addPage(page)

```

## File: models\product_document.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import Command, _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.sale_pdf_quote_builder import utils


class ProductDocument(models.Model):
    _inherit = 'product.document'

    attached_on_sale = fields.Selection(
        selection_add=[('inside', "Inside quote pdf")],
        help="Allows you to share the document with your customers within a sale.\n"
             "Leave it empty if you don't want to share this document with sales customer.\n"
             "On quote: the document will be sent to and accessible by customers at any time.\n"
             "e.g. this option can be useful to share Product description files.\n"
             "On order confirmation: the document will be sent to and accessible by customers.\n"
             "e.g. this option can be useful to share User Manual or digital content bought on"
             " ecommerce. \n"
             "Inside quote: The document will be included in the pdf of the quotation and sale"
             " order between the header pages and the quote table. ",
        ondelete={'inside': 'set default'},
    )
    form_field_ids = fields.Many2many(
        string="Form Fields Included",
        comodel_name='sale.pdf.form.field',
        domain=[('document_type', '=', 'product_document')],
        compute='_compute_form_field_ids',
        store=True,
    )

    # === CONSTRAINT METHODS ===#

    @api.constrains('attached_on_sale', 'datas', 'type')
    def _check_attached_on_and_datas_compatibility(self):
        for doc in self.filtered(lambda doc: doc.attached_on_sale == 'inside'):
            if doc.type != 'binary':
                raise ValidationError(_(
                    "When attached inside a quote, the document must be a file, not a URL."
                ))
            if doc.datas and not doc.mimetype.endswith('pdf'):
                raise ValidationError(_("Only PDF documents can be attached inside a quote."))
            utils._ensure_document_not_encrypted(base64.b64decode(doc.datas))

    # === COMPUTE METHODS === #

    @api.depends('datas', 'attached_on_sale')
    def _compute_form_field_ids(self):
        # Empty the linked form fields as we want all and only those from the current datas
        self.form_field_ids = [Command.clear()]
        document_to_parse = self.filtered(
            lambda doc: doc.attached_on_sale == 'inside' and doc.datas
        )
        if document_to_parse:
            doc_type = 'product_document'
            self.env['sale.pdf.form.field']._create_or_update_form_fields_on_pdf_records(
                document_to_parse, doc_type
            )

    # === ACTION METHODS ===#

    def action_open_pdf_form_fields(self):
        self.ensure_one()
        return {
            'name': _('Form Fields'),
            'type': 'ir.actions.act_window',
            'res_model': 'sale.pdf.form.field',
            'view_mode': 'list',
            'context': {
                'default_document_type': 'product_document',
                'default_product_document_ids': self.id,
                'default_quotation_document_ids': False,
                'search_default_context_document': True,
            },
            'target': 'current',
        }

```

## File: models\quotation_document.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import Command, _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.sale_pdf_quote_builder import utils


class QuotationDocument(models.Model):
    _name = 'quotation.document'
    _description = "Quotation's Headers & Footers"
    _inherits = {
        'ir.attachment': 'ir_attachment_id',
    }
    _order = 'document_type desc, sequence, name'
    _check_company_auto = True

    ir_attachment_id = fields.Many2one(
        string="Related attachment",
        comodel_name='ir.attachment',
        ondelete='cascade',
        required=True,
    )
    document_type = fields.Selection(
        string="Document Type",
        selection=[('header', "Header"), ('footer', "Footer")],
        required=True,
        default='header',
    )
    active = fields.Boolean(
        help="If unchecked, it will allow you to hide the header or footer without removing it.",
        default=True,
    )
    sequence = fields.Integer(default=10)
    quotation_template_ids = fields.Many2many(
        string="Quotation Templates",
        comodel_name='sale.order.template',
        relation='header_footer_quotation_template_rel',
        check_company=True,
    )
    form_field_ids = fields.Many2many(
        string="Form Fields Included",
        comodel_name='sale.pdf.form.field',
        domain=[('document_type', '=', 'quotation_document')],
        compute='_compute_form_field_ids',
        store=True,
    )

    # === CONSTRAINT METHODS ===#

    @api.constrains('datas')
    def _check_pdf_validity(self):
        for doc in self:
            if doc.datas and not doc.mimetype.endswith('pdf'):
                raise ValidationError(_("Only PDF documents can be used as header or footer."))
            utils._ensure_document_not_encrypted(base64.b64decode(doc.datas))

    # === COMPUTE METHODS === #

    @api.depends('datas')
    def _compute_form_field_ids(self):
        # Empty the linked form fields as we want all and only those from the current datas
        self.form_field_ids = [Command.clear()]
        document_to_parse = self.filtered(lambda doc: doc.datas)
        if document_to_parse:
            doc_type = 'quotation_document'
            self.env['sale.pdf.form.field']._create_or_update_form_fields_on_pdf_records(
                document_to_parse, doc_type
            )

    # === ACTION METHODS ===#

    def action_open_pdf_form_fields(self):
        self.ensure_one()
        return {
            'name': _('Form Fields'),
            'type': 'ir.actions.act_window',
            'res_model': 'sale.pdf.form.field',
            'view_mode': 'list',
            'context': {
                'default_document_type': 'quotation_document',
                'default_product_document_ids': False,
                'default_quotation_document_ids': self.id,
                'search_default_context_document': True,
            },
            'target': 'current',
        }

    # === CRUD METHODS ===#

    @api.model_create_multi
    def create(self, vals_list):
        docs = super().create(vals_list)
        for doc in docs:
            doc.write({'res_model': 'quotation.document', 'res_id': doc.id})
        return docs

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import json

from odoo import _, api, fields, models


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    available_product_document_ids = fields.Many2many(
        string="Available Product Documents",
        comodel_name='quotation.document',
        compute='_compute_available_product_document_ids',
    )
    is_pdf_quote_builder_available = fields.Boolean(
        compute='_compute_is_pdf_quote_builder_available',
    )
    quotation_document_ids = fields.Many2many(
        string="Headers/Footers",
        comodel_name='quotation.document',
        readonly=False,
        check_company=True,
    )
    customizable_pdf_form_fields = fields.Json(
        string="Customizable PDF Form Fields",
        readonly=False,
    )

    # === COMPUTE METHODS === #

    @api.depends('sale_order_template_id')
    def _compute_available_product_document_ids(self):
        for order in self:
            order.available_product_document_ids = self.env['quotation.document'].search(
                self.env['quotation.document']._check_company_domain(order.company_id),
                order='sequence',
            ).filtered(lambda doc:
                order.sale_order_template_id in doc.quotation_template_ids
                or not doc.quotation_template_ids
            )

    @api.depends('available_product_document_ids', 'order_line', 'order_line.available_product_document_ids')
    def _compute_is_pdf_quote_builder_available(self):
        for order in self:
            order.is_pdf_quote_builder_available = bool(
                order.available_product_document_ids
                or order.order_line.available_product_document_ids
            )

    # === ACTION METHODS === #

    def get_update_included_pdf_params(self):
        if not self:
            return {
                'headers': {},
                'files': {},
                'footers': {},
            }
        self.ensure_one()
        existing_mapping = (
            self.customizable_pdf_form_fields
            and json.loads(self.customizable_pdf_form_fields)
        ) or {}

        headers_available = self.available_product_document_ids.filtered(
            lambda doc: doc.document_type == 'header'
        )
        footers_available = self.available_product_document_ids.filtered(
            lambda doc: doc.document_type == 'footer'
        )
        selected_documents = self.quotation_document_ids
        selected_headers = selected_documents.filtered(lambda doc: doc.document_type == 'header')
        selected_footers = selected_documents - selected_headers
        lines_params = []
        for line in self.order_line:
            if line.available_product_document_ids:
                lines_params.append({
                    'name': _("Product") + " > " + line.name.splitlines()[0],
                    'id': line.id,
                    'files': [{
                        'name': doc.name.rstrip('.pdf'),
                        'id': doc.id,
                        'is_selected': doc in line.sudo().product_document_ids, # User should be
                        # able to access all product documents even without sales access
                        'custom_form_fields': [{
                            'name': custom_form_field.name,
                            'value': existing_mapping.get('line', {}).get(str(line.id), {}).get(
                                str(doc.id), {}
                            ).get('custom_form_fields', {}).get(custom_form_field.name, ""),
                        } for custom_form_field in doc.form_field_ids.filtered(
                            lambda ff: not ff.path
                        )],
                    } for doc in line.available_product_document_ids]
                })
        dialog_params = {
            'headers': {'name': _("Header"), 'files': [{
                'id': header.id,
                'name': header.name,
                'is_selected': header in selected_headers,
                'custom_form_fields': [{
                    'name': custom_form_field.name,
                    'value': existing_mapping.get('header', {}).get(str(header.id), {}).get(
                        'custom_form_fields', {}
                    ).get(custom_form_field.name, ""),
                } for custom_form_field in header.form_field_ids.filtered(lambda ff: not ff.path)],
            } for header in headers_available]},
            'lines': lines_params,
            'footers': {'name': _("Footer"), 'files': [{
                'id': footer.id,
                'name': footer.name,
                'is_selected': footer in selected_footers,
                'custom_form_fields': [{
                    'name': custom_form_field.name,
                    'value': existing_mapping.get('footer', {}).get(str(footer.id), {}).get(
                        'custom_form_fields', {}
                    ).get(custom_form_field.name, ""),
                } for custom_form_field in footer.form_field_ids.filtered(lambda ff: not ff.path)],
            } for footer in footers_available]},
        }
        return dialog_params

    # === BUSINESS METHODS === #

    # FIXME EDM dead code below ?
    def save_included_pdf(self, selected_pdf):
        """ Configure the PDF that should be included in the PDF quote builder for a given quote

        Note: self.ensure_one()

        :param dic selected_pdf: Dictionary of all the sections linked to their header_footer or
                                 product_document ids, in the format: {
                                    'header': [doc_id],
                                    'lines': [{line_id: [doc_id]}],
                                    'footer': [doc_id]
                                }
        :return: None
        """
        self.ensure_one()
        quotation_doc = self.env['quotation.document']
        selected_headers = quotation_doc.browse(selected_pdf['header'])
        selected_footers = quotation_doc.browse(selected_pdf['footer'])
        self.quotation_document_ids = selected_headers.ids + selected_footers.ids
        for line in self.order_line:
            selected_lines = self.env['product.document'].browse(
                selected_pdf['lines'].get(str(line.id))
            )
            line.product_document_ids = selected_lines.ids

    def save_new_custom_content(self, document_type, form_field, content):
        """ Modify the content link to a form field in the custom content mapping of an order.

        Note: self.ensure_one()

        :param str document_type: The document type where the for field is. Either 'header_footer'
                                  or 'product_document'.
        :param str form_field: The form field in the custom content mapping.
        :param str content: The content of the form field in the custom content mapping.
        :return: None
        """
        self.ensure_one()
        mapping = json.loads(self.customizable_pdf_form_fields)
        mapping[document_type][form_field] = content
        self.customizable_pdf_form_fields = json.dumps(mapping)

```

## File: models\sale_order_line.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class SaleOrderLine(models.Model):
    _inherit = 'sale.order.line'

    available_product_document_ids = fields.Many2many(
        string="Available Product Documents",
        comodel_name='product.document',
        relation='available_sale_order_line_product_document_rel',
        compute='_compute_available_product_document_ids',
        compute_sudo=True, # To access attached_on_sale
    )
    product_document_ids = fields.Many2many(
        string="Product Documents",
        help="The product documents for this order line that will be merged in the PDF quote.",
        comodel_name='product.document',
        relation='sale_order_line_product_document_rel',
        domain="[('id', 'in', available_product_document_ids)]",
        readonly=False,
    )

    # === ONCHANGE METHODS === #

    @api.onchange('product_id', 'product_template_id')
    def _onchange_product(self):
        for line in self:
            # Ensure selected documents are still in the available documents
            line.product_document_ids &= line.available_product_document_ids

    # === COMPUTE METHODS === #

    @api.depends('product_id', 'product_template_id')
    def _compute_available_product_document_ids(self):
        for line in self:
            line.available_product_document_ids = self.env['product.document'].search([
                '|',
                    '&',
                        ('res_model', '=', 'product.product'),
                        ('res_id', '=', line.product_id.id),
                    '&',
                        ('res_model', '=', 'product.template'),
                        ('res_id', '=', line.product_template_id.id),
                ('attached_on_sale', '=', 'inside')
            ], order='res_model, sequence').ids

```

## File: models\sale_order_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class SaleOrderTemplate(models.Model):
    _inherit = 'sale.order.template'
    _check_company_auto = True

    quotation_document_ids = fields.Many2many(
        string="Headers and footers",
        comodel_name='quotation.document',
        relation='header_footer_quotation_template_rel',
        check_company=True,
    )

```

## File: models\sale_pdf_form_field.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import Command, _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.sale_pdf_quote_builder import utils


class SalePdfFormField(models.Model):
    _name = 'sale.pdf.form.field'
    _description = "Form fields of inside quotation documents."
    _order = 'name'

    name = fields.Char(
        string="Form Field Name",
        help="The form field name as written in the PDF.",
        readonly=True,
        required=True,
    )
    document_type = fields.Selection(
        string="Document Type",
        selection=[
            ('quotation_document', "Header/Footer"),
            ('product_document', "Product Document"),
        ],
        readonly=True,
        required=True,
    )
    path = fields.Char(
        string="Path",
        help="The path to follow to dynamically fill the form field. \n"
             "Leave empty to be able to customized it in the quotation form."
    )
    product_document_ids = fields.Many2many(
        string="Product Documents", comodel_name='product.document'
    )
    quotation_document_ids = fields.Many2many(
        string="Quotation Documents", comodel_name='quotation.document'
    )

    _sql_constraints = [(
        'unique_name_per_doc_type',
        'UNIQUE(name, document_type)',
        "Form field name must be unique for a given document type."
    )]

    # === CONSTRAINT METHODS ===#

    @api.constrains('name')
    def _check_form_field_name_follows_pattern(self):
        """ Ensure the names only contains alphanumerics, hyphens and underscores.

        :return: None
        :raises: ValidationError if the names aren't alphanumerics, hyphens and underscores.
        """
        name_pattern = re.compile(r'^(\w|-)+$')
        for form_field in self:
            if not re.match(name_pattern, form_field.name):
                raise ValidationError(_(
                    "Invalid form field name %(field_name)s. It should only contain alphanumerics,"
                    " hyphens or underscores.",
                    field_name=form_field.name,
                ))
            if form_field.name.startswith('sol_id_'):
                raise ValidationError(_(
                    "Invalid form field name %(field_name)s. A form field name in a header or a"
                    " footer can not start with \"sol_id_\".",
                    field_name=form_field.name,
                ))

    @api.constrains('path')
    def _check_valid_and_existing_paths(self):
        """ Verify that the paths exist and are valid.

        :return: None
        :raises: ValidationError if at least one of the paths isn't valid.
        """
        name_pattern = re.compile(r'^(\w|-|\.)+$')
        for form_field in self.filtered('path'):
            if not re.match(name_pattern, form_field.path):
                raise ValidationError(_(
                    "Invalid path %(path)s. It should only contain alphanumerics, hyphens,"
                    " underscores or points.",
                    path=form_field.path,
                ))

            path = form_field.path.split('.')
            is_header_footer = form_field.document_type == 'quotation_document'
            Model = self.env['sale.order'] if is_header_footer else self.env['sale.order.line']
            for i in range(len(path)):
                field_name = path[i]
                if Model == []:
                    raise ValidationError(_(
                        "Please use only relational fields until the last value of your path."
                    ))
                if field_name not in Model._fields:
                    raise ValidationError(_(
                        "The field %(field_name)s doesn't exist on model %(model_name)s",
                        field_name=field_name,
                        model_name=Model._name
                    ))
                if i != len(path) - 1:
                    Model = Model.mapped(field_name)

    @api.constrains('document_type', 'product_document_ids', 'quotation_document_ids')
    def _check_document_type_and_document_linked_compatibility(self):
        for form_field in self:
            doc_type = form_field.document_type
            if doc_type == 'quotation_document' and form_field.product_document_ids:
                raise ValidationError(_(
                    "A form field set as used in product documents can't be linked to a quotation"
                    " document."
                ))
            elif doc_type == 'product_document' and form_field.quotation_document_ids:
                raise ValidationError(_(
                    "A form field set as used in quotation documents can't be linked to a product"
                    " document."
                ))

    # === BUSINESS METHODS ===#

    @api.model
    def _add_basic_mapped_form_fields(self):
        mapped_form_fields = {
            'quotation_document': {
                "amount_total": "amount_total",
                "amount_untaxed": "amount_untaxed",
                "client_order_ref": "client_order_ref",
                "delivery_date": "commitment_date",
                "order_date": "date_order",
                "name": "name",
                "partner_id__name": "partner_id.name",
                "user_id__email": "user_id.login",
                "user_id__name": "user_id.name",
                "validity_date": "validity_date",
            },
            'product_document': {
                "amount_total": "order_id.amount_total",
                "amount_untaxed": "order_id.amount_untaxed",
                "client_order_ref": "order_id.client_order_ref",
                "delivery_date": "order_id.commitment_date",
                "description": "name",
                "discount": "discount",
                "name": "order_id.name",
                "partner_id__name": "order_partner_id.name",
                "price_unit": "price_unit",
                "product_sale_price": "product_id.lst_price",
                "quantity": "product_uom_qty",
                "tax_excl_price": "price_subtotal",
                "tax_incl_price": "price_total",
                "taxes": "tax_id",
                "uom": "product_uom.name",
                "user_id__name": "salesman_id.name",
                "validity_date": "order_id.validity_date",
            },
        }
        quote_doc = list(mapped_form_fields['quotation_document'])
        product_doc = list(mapped_form_fields['product_document'])
        existing_mapping = self.env['sale.pdf.form.field'].search([
            '|',
            '&', ('document_type', '=', 'quotation_document'), ('name', 'in', quote_doc),
            '&', ('document_type', '=', 'product_document'), ('name', 'in', product_doc)
        ])
        if existing_mapping:
            form_fields_to_add = {
                doc_type: {
                    name: path for name, path in mapped_form_fields[doc_type].items()
                    if not existing_mapping.filtered(
                        lambda ff: ff.document_type == doc_type and ff.name == name
                    )
                } for doc_type, mapping in mapped_form_fields.items()
            }
        else:
            form_fields_to_add = mapped_form_fields
        self.env['sale.pdf.form.field'].create([
            {'name': name, 'document_type': doc_type, 'path': path}
            for doc_type, mapping in form_fields_to_add.items()
            for name, path in mapping.items()
        ])

    @api.model
    def _cron_post_upgrade_assign_missing_form_fields(self):
        # Called post-upgrade as we can't access the files during the upgrade process
        product_documents = self.env['product.document'].search(
            [('attached_on_sale', '=', 'inside')]
        )
        quote_documents = self.env['quotation.document'].search([])
        self._create_or_update_form_fields_on_pdf_records(product_documents, 'product_document')
        self._create_or_update_form_fields_on_pdf_records(quote_documents, 'quotation_document')

    @api.model
    def _create_or_update_form_fields_on_pdf_records(self, records, doc_type):
        existing_form_fields = self.env['sale.pdf.form.field'].search(
            [('document_type', '=', doc_type)]
        )
        existing_form_fields_name = existing_form_fields.mapped('name')
        return_bin_size = self.env.context.get('bin_size')
        if return_bin_size:
            # guarantees that bin_size is always set to False
            records = records.with_context(bin_size=False)

        for document in records:
            if document.datas:
                form_fields = utils._get_form_fields_from_pdf(document.datas)
                for field in form_fields:
                    if field not in existing_form_fields_name:
                        document.form_field_ids = [
                            Command.create({
                                'name': field, 'document_type': doc_type
                            })
                        ]
                        existing_form_fields_name.append(field)
                        existing_form_fields += document.form_field_ids[-1]
                    else:
                        document.form_field_ids = [Command.link(existing_form_fields.filtered(
                            lambda form_field: form_field.name == field
                        ).id)]

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_actions_report
from . import product_document
from . import quotation_document
from . import sale_order
from . import sale_order_line
from . import sale_order_template
from . import sale_pdf_form_field

```

## File: report\ir_actions_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="action_report_saleorder_raw" model="ir.actions.report">
        <field name="name">Quotation / Order</field>
        <field name="model">sale.order</field>
        <field name="report_type">qweb-pdf</field>
        <field name="report_name">sale.report_saleorder_raw</field>
        <field name="report_file">sale.report_saleorder_raw</field>
        <field name="print_report_name">(object.state in ('draft', 'sent') and 'Quotation - %s' % (object.name)) or 'Order - %s' % (object.name)</field>
        <field name="binding_model_id" ref="sale.model_sale_order"/>
        <field name="binding_type">report</field>
    </record>

    <record id="sale.action_report_saleorder" model="ir.actions.report">
        <field name="name">PDF Quote</field>
    </record>

</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_quotation_document_sales_manager,access_quotation_document_sales_manager,model_quotation_document,sales_team.group_sale_manager,1,1,1,1
access_quotation_document_user,access_quotation_document_user,model_quotation_document,base.group_user,1,0,0,0
access_sale_pdf_form_field_system,access_sale_pdf_form_field_user,model_sale_pdf_form_field,base.group_system,1,1,1,1
access_sale_pdf_form_field_user,access_sale_pdf_form_field_user,model_sale_pdf_form_field,base.group_user,1,0,0,0

```

## File: security\ir_rules.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- Multi-company rules -->
    <record id="quotation_document_comp_rule" model="ir.rule">
        <field name="name">Quotation document multi-company rule</field>
        <field name="model_id" ref="model_quotation_document"/>
        <field name="domain_force">
            ['|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]
        </field>
    </record>

</odoo>

```

## File: static\src\js\custom_content_kanban_like_widget\custom_content_kanban_like_widget.js

```javascript
import { Component, useEffect, useState } from "@odoo/owl";
import {
    CustomFieldCard
} from "@sale_pdf_quote_builder/js/custom_content_kanban_like_widget/custom_field_card/custom_field_card";
import { x2ManyCommands } from "@web/core/orm_service";
import { registry } from '@web/core/registry';
import { useService } from "@web/core/utils/hooks";
import { standardWidgetProps } from "@web/views/widgets/standard_widget_props";

export class CustomContentKanbanLikeWidget extends Component {
    static components = { CustomFieldCard };
    static template = "sale_pdf_quote_builder.CustomContentKanbanLike";
    static props = {
        ...standardWidgetProps,
    };

    setup() {
        this.orm = useService("orm");
        this.state = useState({
            headers: {},
            lines: {},
            footers: {},
        });

        // Initialize the state and update available documents when updating the quotation template.
        useEffect((saleOrderTemplate) => {
            this.updateState();
        }, () => [this.props.record.data.sale_order_template_id]);
    }

    async updateState() {
        const saved = await this.props.record.save();  // To display documents of potentially unsaved SOL.
        if (saved) {  // do not fetch wrong form data if record was not saved.
            const { headers, lines, footers } = await this.orm.call(
                'sale.order', 'get_update_included_pdf_params', [this.props.record.resId]
            )
            this.state.headers = headers;
            this.state.lines = lines;
            this.state.footers = footers;
        }
    }

    updateJson() {
        const selectedHeaders = this.state.headers.files.filter(f => f.is_selected);
        const selectedFooters = this.state.footers.files.filter(f => f.is_selected);
        const value = JSON.stringify({
            'header': Object.assign({}, ...selectedHeaders.map(header => {
                return {
                    [header.id]: {
                        document_name: header.name,
                        custom_form_fields: Object.assign({}, ...header.custom_form_fields.map(
                            formField => ({[formField.name]: formField.value})
                        )),
                    }
            }})),
            'line': Object.assign({}, ...this.state.lines.map(line => {
                return {
                    [line.id]: Object.assign({}, ...line.files.filter(f => f.is_selected).map(doc => {
                        return {
                            [doc.id]: {
                                document_name: doc.name,
                                custom_form_fields: Object.assign({}, ...doc.custom_form_fields.map(
                                    formField => ({[formField.name]: formField.value})
                                )),
                            }
                    }})),
            }})),
            'footer': Object.assign({}, ...selectedFooters.map(footer => {
                return {
                    [footer.id]: {
                        document_name: footer.name,
                        custom_form_fields: Object.assign({}, ...footer.custom_form_fields.map(
                            formField => ({[formField.name]: formField.value})
                        )),
                    }
            }})),
        })
        this.props.record.update({ ['customizable_pdf_form_fields']: value });
    }

    async saveProductDocument(lineId, docId, isSelected) {
        const sol = this.props.record.data.order_line.records.find(
            sol => sol.resId === lineId
        );
        sol._noUpdateParent = true; // Ensure that no rpc will be made to save the changes
        if (isSelected) {
            // save is needed to ensure that no onChange call will be made
            await sol.update({product_document_ids: [x2ManyCommands.link(docId)]}, { save: true });
        } else {
            // save is needed to ensure that no onChange call will be made
            await sol.update({product_document_ids: [x2ManyCommands.unlink(docId)]}, { save: true });
        }
        await this.props.record.data.order_line._onUpdate({withoutOnchange: true});
        this.updateJson();
    };

    async saveQuotationDocument(docId, isSelected) {
        if (isSelected) {
            await this.props.record.update({
                quotation_document_ids: [
                    x2ManyCommands.link(docId),
                ],
            });
        } else {
            await this.props.record.update({
                quotation_document_ids: [
                    x2ManyCommands.unlink(docId),
                ],
            });
        }
        this.updateJson();
    };
}

export const customContentKanbanLikeWidget = {
    component: CustomContentKanbanLikeWidget,
};

registry.category("view_widgets").add(
    "customContentKanbanLikeWidget", customContentKanbanLikeWidget
);

```

## File: static\src\js\custom_content_kanban_like_widget\custom_content_kanban_like_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t t-name="sale_pdf_quote_builder.CustomContentKanbanLike">
        <t t-if="state.headers.files?.length">
            <t t-call="sale_pdf_quote_builder.section">
                <t t-set="section" t-value="state.headers"/>
                <t
                    t-set="save"
                    t-value="(lineId, docId, isSelected) => {
                        this.saveQuotationDocument(docId, isSelected);
                    }"
                />
            </t>
        </t>
        <t t-if="state.lines?.length">
            <t t-foreach="state.lines" t-as="section" t-key="section.id">
                <t t-call="sale_pdf_quote_builder.section">
                    <t
                        t-set="save"
                        t-value="(lineId, docId, isSelected) => {
                            this.saveProductDocument(lineId, docId, isSelected);
                        }"
                    />
                </t>
            </t>
        </t>
        <t t-if="state.footers.files?.length">
            <t t-call="sale_pdf_quote_builder.section">
                <t t-set="section" t-value="state.footers"/>
                <t
                    t-set="save"
                    t-value="(lineId, docId, isSelected) => {
                        this.saveQuotationDocument(docId, isSelected);
                    }"
                />
            </t>
        </t>
    </t>

    <t t-name="sale_pdf_quote_builder.section">
        <!-- Context:
            - section: The Object describing the section, its documents, and custom form fields.
        -->
        <t t-set="id" t-value="section.id or section.name"/>
        <div class="mb-4">
            <div class="mb-2">
                <h3 t-out="section.name"/>
                <t t-foreach="section.files" t-as="doc" t-key="id+'-button-'+doc.id">
                    <input
                        type="checkbox"
                        class="btn-check"
                        t-att-id="id+'-'+doc.id"
                        t-att-checked="doc.is_selected"
                        t-on-click="() => {
                            doc.is_selected = !doc.is_selected;
                            save(id, doc.id, doc.is_selected);
                        }"
                    />
                    <label
                        class="btn btn-secondary m-1"
                        t-att-for="id+'-'+doc.id"
                        t-out="doc.name"
                    />
                </t>
            </div>
            <t
                t-foreach="section.files.filter(doc => doc.is_selected)"
                t-as="doc"
                t-key="id+'-'+doc.id+'-custom_form_fields'"
            >
                <div
                    t-if="doc.custom_form_fields.length"
                    t-att-class="{'pb-1 mb-3 o_horizontal_separator': !doc_last}"
                >
                    <t
                        t-foreach="doc.custom_form_fields"
                        t-as="formField"

                        t-key="id+'-'+doc.id+'-custom_form_fields'+formField_index"
                    >
                        <CustomFieldCard
                            name="formField.name"
                            value="formField.value"
                            onChange="(value) => { formField.value = value; this.updateJson(); }"
                        />
                    </t>
                </div>
            </t>
        </div>
    </t>
</templates>

```

## File: static\src\js\custom_content_kanban_like_widget\custom_field_card\custom_field_card.js

```javascript
/** @odoo-module **/

import { Component, useRef } from "@odoo/owl";
import { _t } from "@web/core/l10n/translation";
import { useAutoresize } from "@web/core/utils/autoresize";

export class CustomFieldCard extends Component {
    static template = "sale_pdf_quote_builder.customFieldCard";
    static props = {
        name: String,
        value: String,
        onChange: Function,
    };

    setup() {
        this.customFormFieldTextAreaRef = useRef('customFieldCardTextArea');
        this.placeholder = _t("Click to write content for the PDF quote...");
        useAutoresize(this.customFormFieldTextAreaRef);
    }

    expandTextArea(ev) {
        const textarea = ev.target;
        textarea.style.height = textarea.scrollHeight+'px';
    }
}

```

## File: static\src\js\custom_content_kanban_like_widget\custom_field_card\custom_field_card.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>

    <t t-name="sale_pdf_quote_builder.customFieldCard">
        <div>
            <h5 t-out="this.props.name" class="text-primary"/>
            <div class="mb-3">
                <span t-attf-class="card-text #{this.props.value ? '' : 'text-muted'}">
                    <textarea
                        t-ref="customFieldCardTextArea"
                        t-model.lazy="this.props.value"
                        t-on-change="(ev) => { this.props.onChange(this.props.value) }"
                        t-att-placeholder="this.placeholder"
                        class="customFieldCardTextArea form-control overflow-y-hidden"
                    />
                </span>
            </div>
        </div>
    </t>

</templates>

```

## File: static\src\js\quotation_document_kanban\quotation_document_kanban_controller.js

```javascript
/** @odoo-module **/

import { onWillRender } from "@odoo/owl";
import { UploadButton } from '@product/js/product_document_kanban/upload_button/upload_button';
import { KanbanController } from '@web/views/kanban/kanban_controller';

export class QuotationDocumentKanbanController extends KanbanController {
    static components = { ...KanbanController.components, UploadButton };

    setup() {
        super.setup();
        this.uploadRoute = '/sale_pdf_quote_builder/quotation_document/upload';
        this.allowedMIMETypes='application/pdf';
        onWillRender(() => {
            this.formData = {
                'allowed_company_ids': JSON.stringify(this.props.context.allowed_company_ids),
            };
        });
    }
}

```

## File: static\src\js\quotation_document_kanban\quotation_document_kanban_view.js

```javascript
import {
    productDocumentKanbanView
} from '@product/js/product_document_kanban/product_document_kanban_view';
import {
    QuotationDocumentKanbanController
} from '@sale_pdf_quote_builder/js/quotation_document_kanban/quotation_document_kanban_controller';
import { registry } from '@web/core/registry';

export const quotationDocumentKanbanView = {
    ...productDocumentKanbanView,
    Controller: QuotationDocumentKanbanController,
};

registry.category('views').add('quotation_document_kanban', quotationDocumentKanbanView);

```

## File: static\src\js\quotation_document_kanban\quotation_document_kanban_widget.js

```javascript
/** @odoo-module **/

import {
    ProductDocumentKanbanRenderer
} from "@product/js/product_document_kanban/product_document_kanban_renderer";
import { UploadButton } from '@product/js/product_document_kanban/upload_button/upload_button';
import { registry } from '@web/core/registry';
import { X2ManyField, x2ManyField } from '@web/views/fields/x2many/x2many_field';
import { onWillRender } from "@odoo/owl";

export class QuotationDocumentX2ManyField extends X2ManyField {
    static template = 'sale_pdf_quote_builder.QuotationDocumentX2ManyField';
    static components = {
        ...X2ManyField.components,
        UploadButton,
        KanbanRenderer: ProductDocumentKanbanRenderer,
    };

    setup() {
        super.setup();
        this.uploadRoute = '/sale_pdf_quote_builder/quotation_document/upload';
        this.allowedMIMETypes='application/pdf';
        onWillRender(() => {
            this.formData = {
                'sale_order_template_id': this.props.record.resId,
            };
        });
    }
}

export const quotationDocumentX2ManyField = {
    ...x2ManyField,
    component: QuotationDocumentX2ManyField,
};

registry.category('fields').add('quotation_document_many2many', quotationDocumentX2ManyField);

```

## File: static\src\js\quotation_document_kanban\quotation_document_kanban_widget.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <t
        t-name="sale_pdf_quote_builder.QuotationDocumentX2ManyField"
        t-inherit-mode="primary"
        t-inherit="web.X2ManyField"
    >
        <xpath expr="//div[hasclass('o_cp_buttons')]" position="inside">
            <UploadButton
                t-if="formData.sale_order_template_id"
                formData="formData"
                allowedMIMETypes="allowedMIMETypes"
                load.bind="() => this.props.record.load()"
                uploadRoute="uploadRoute"
            />
        </xpath>
    </t>
</templates>

```

## File: views\product_document_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="product_document_form" model="ir.ui.view">
        <field name="name">product.document.form.sale</field>
        <field name="model">product.document</field>
        <field name="inherit_id" ref="product.product_document_form"/>
        <field name="arch" type="xml">
            <field name="datas" position="after">
                <button
                    name="action_open_pdf_form_fields"
                    type="object"
                    string="Configure dynamic fields"
                    help="Configure the path to fill the form fields."
                    class="btn btn-link"
                    icon="fa-pencil"
                    groups="base.group_system"
                />
            </field>
            <field name="company_id" position="before">
                <field name="form_field_ids" widget="many2many_tags"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\quotation_document_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="quotation_document_form" model="ir.ui.view">
        <field name="name">quotation.document.form</field>
        <field name="model">quotation.document</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <h1>
                        <field name="name" readonly="not datas"/>
                    </h1>
                    <group>
                        <group>
                            <field name="document_type"/>
                            <label for="datas"/>
                            <div class="o_row">
                                <field
                                    name="datas"
                                    filename="name"
                                    options="{'accepted_file_extensions': '.pdf'}"
                                    class="oe_inline"
                                />
                                <button
                                    name="action_open_pdf_form_fields"
                                    type="object"
                                    string="Configure dynamic fields"
                                    help="Mark fields as safe to fill in the quote."
                                    class="btn btn-link"
                                    icon="fa-pencil"
                                    colspan="2"
                                    groups="base.group_system,base.group_no_one"
                                />
                            </div>
                            <field
                                name="quotation_template_ids"
                                options="{'no_create': True}"
                                widget="many2many_tags"
                            />
                        </group>
                    </group>
                    <group string="Attached To" groups="base.group_multi_company,base.group_no_one">
                        <field
                            name="company_id"
                            placeholder="Visible to all"
                            groups="base.group_multi_company"
                            options="{'no_create': True}"
                            class="oe_inline"
                        />
                        <field
                            name="form_field_ids"
                            groups="base.group_no_one"
                            widget="many2many_tags"
                        />
                    </group>
                    <group string="History" groups="base.group_no_one" invisible="not create_date">
                        <label for="create_uid" string="Creation"/>
                        <div name="creation_div">
                            <field name="create_uid" readonly="1" class="oe_inline"/> on
                            <field name="create_date" readonly="1" class="oe_inline"/>
                        </div>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="quotation_document_kanban" model="ir.ui.view">
        <field name="name">quotation.document.kanban</field>
        <field name="model">quotation.document</field>
        <field name="arch" type="xml">
            <kanban js_class="quotation_document_kanban">
                <field name="sequence" widget="handle"/>
                <field name="ir_attachment_id"/>
                <field name="mimetype"/>
                <field name="document_type"/>
                <field name="name"/>
                <field name="active"/>
                <templates>
                    <t t-name="menu">
                        <a t-if="widget.editable" type="open" class="dropdown-item">Edit</a>
                        <a t-if="widget.deletable" type="delete" class="dropdown-item">Delete</a>
                        <a
                            t-attf-href="/web/content/#{record.ir_attachment_id.raw_value}?download=true"
                            download=""
                            class="dropdown-item"
                        >Download</a>
                    </t>
                    <t t-name="card" class="o_kanban_attachment flex-row">
                        <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                        <aside>
                            <div class="o_image o_kanban_previewer w-100 h-100" t-att-data-mimetype="record.mimetype.value"/>
                        </aside>
                        <main class="ms-2">
                            <field name="name" class="fw-bolder text-truncate"/>
                            <div class="d-flex mt-2">
                                <span>Document type:</span>
                                <field name="document_type" class="ms-2" widget="selection"/>
                            </div>
                            <div t-if="!!record.quotation_template_ids.raw_value.length" class="mt-2">
                                <span class="pe-2">Templates:</span>
                                <field name="quotation_template_ids" class="d-inline-block" widget="many2many_tags"/>
                            </div>
                        </main>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="quotation_document_list" model="ir.ui.view">
        <field name="name">quotation.document.list</field>
        <field name="model">quotation.document</field>
        <field name="arch" type="xml">
            <list editable="top" multi_edit="true">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="document_type"/>
                <field name="company_id" groups="base.group_multi_company" optional="show"/>
            </list>
        </field>
    </record>

    <record id="quotation_document_search_view" model="ir.ui.view">
        <field name="name">quotation.document.search</field>
        <field name="model">quotation.document</field>
        <field name="arch" type="xml">
            <search string="Quotation Document">
                <field name="name"/>
                <separator/>
                <group string="Group By">
                    <filter
                        string="Document type"
                        name="doc_type"
                        context="{'group_by': 'document_type'}"
                    />
                    <filter
                        string="Quotation Template"
                        name="quotation_template"
                        context="{'group_by': 'quotation_template_ids'}"
                    />
                    <filter name="all" string="All"/>
                    <filter name="archived" string="Archived" domain="[('active', '=', False)]"/>
                </group>
            </search>
        </field>
    </record>

    <record id="quotation_document_action" model="ir.actions.act_window">
        <field name="name">Headers/Footers</field>
        <field name="res_model">quotation.document</field>
        <field name="view_mode">kanban,list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Upload quotation headers and footers
            </p>
            <p>
                Personalize your quotes with catchy header and footer pages
                <br/>
                to boost your sales.
            </p>
        </field>
    </record>

</odoo>

```

## File: views\sale_order_template_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="sale_order_template_form" model="ir.ui.view">
        <field name="name">sale.order.template.form</field>
        <field name="model">sale.order.template</field>
        <field name="inherit_id" ref="sale_management.sale_order_template_view_form"/>
        <field name="arch" type="xml">
            <notebook position="inside">
                <page name="pdf_quote" string="Quote Builder">
                    <field
                        name="quotation_document_ids"
                        mode="kanban"
                        widget="quotation_document_many2many"
                        class="w-100"
                        nolabel="1"
                        options="{'create': False}"
                        context="{
                            'kanban_view_ref': 'sale_pdf_quote_builder.quotation_document_kanban',
                        }"
                    />
                    <p class="text-muted">
                        Provide header pages and footer pages to compose an attractive quotation with more information
                        about your company, your products and you services. <br/>
                        The pdf of your quotes will be built by putting together header pages, product descriptions,
                        details of the quote and then the footer pages. <br/>
                        If empty, it will use those define in the company settings. <br/>
                        <widget
                            name="documentation_link"
                            path="/_downloads/c2c6ce32294dfddffcfefcf2775f7a09/pdfquotebuilderexamples.zip"
                            icon="fa-arrow-right"
                            label=" Download examples"
                        />
                    </p>
                </page>
            </notebook>
        </field>
    </record>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="sale_order_form_inherit_sale_pdf_quote_builder" model="ir.ui.view">
        <field name="name">sale.order.form.pdf.quote.builder</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale_management.sale_order_form_quote"/>
        <field name="arch" type="xml">
            <!-- Needed by customContentKanbanLikeWidget to save selected documents on the product. -->
            <!-- Desktop view -->
            <xpath expr="//field[@name='order_line']/list" position="inside">
                <field name="product_document_ids" column_invisible="1"/>
            </xpath>
            <!-- Mobile view -->
            <xpath expr="//field[@name='order_line']/kanban" position="inside">
                <field name="product_document_ids" column_invisible="1"/>
            </xpath>
            <page name="optional_products" position="after">
                <page
                    name="pdf_quote_builder"
                    string="Quote Builder"
                    invisible="not (partner_id and is_pdf_quote_builder_available)"
                >
                    <!-- Needed by customContentKanbanLikeWidget to save selected documents. -->
                    <field name="quotation_document_ids" invisible="1"/>
                    <field name="customizable_pdf_form_fields" invisible="1"/>
                    <widget
                        name="customContentKanbanLikeWidget"
                        class="d-inline"
                    />
                </page>
            </page>
        </field>
    </record>

</odoo>

```

## File: views\sale_pdf_form_field_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="sale_pdf_form_field_list" model="ir.ui.view">
        <field name="name">sale.pdf.form.field.list</field>
        <field name="model">sale.pdf.form.field</field>
        <field name="arch" type="xml">
            <list editable="top" multi_edit="true" create="false">
                <field name="name"/>
                <field name="path"/>
                <field name="document_type"/>
                <field name="product_document_ids" widget="many2many_tags" optional="hide"/>
                <field name="quotation_document_ids" widget="many2many_tags" optional="hide"/>
            </list>
        </field>
    </record>

    <record id="quotation_document_search" model="ir.ui.view">
        <field name="name">sale.pdf.form.field.search</field>
        <field name="model">sale.pdf.form.field</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter name="customizable" string="Customizable" domain="[('path', '=', False)]"/>
                <filter
                    string="This document"
                    name="context_document"
                    domain="[
                        ('document_type', '=', context.get('default_document_type')),
                        ('product_document_ids', '=', context.get('default_product_document_ids')),
                        ('quotation_document_ids', '=', context.get('default_quotation_document_ids'))
                    ]"
                />
            </search>
        </field>
    </record>

</odoo>

```

## File: views\sale_pdf_quote_builder_menus.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <menuitem id="sale_menu_quotation_document_action"
        name="Headers/Footers"
        action="quotation_document_action"
        parent="sale.menu_sales_config"
        sequence="2"/>

</odoo>

```

## File: wizards\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.pdf.quote.builder</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale_management.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div name="sale_pdf_module_settings" position="inside">
                <button
                    name="%(sale_pdf_quote_builder.quotation_document_action)d"
                    icon="oi-arrow-right"
                    type="action"
                    string="Headers/Footers"
                    class="btn-link"
                />
            </div>
        </field>
    </record>

</odoo>

```

