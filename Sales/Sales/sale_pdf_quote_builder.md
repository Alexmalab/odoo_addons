# Odoo Module: sale_pdf_quote_builder

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: utils.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import io

from odoo import _
from odoo.exceptions import ValidationError
from odoo.tools import pdf


def _ensure_document_not_encrypted(document):
    if pdf.PdfFileReader(io.BytesIO(document), strict=False).isEncrypted:
        raise ValidationError(_(
            "It seems that we're not able to process this pdf inside a quotation. It is either "
            "encrypted, or encoded in a format we do not support."
        ))

```

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizards

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
        'report/ir_actions_report.xml',
        'views/sale_order_template_views.xml',
        'wizards/res_config_settings_views.xml',
    ],
    'demo': [
        'data/sale_pdf_quote_builder_demo.xml',
    ],
    'auto_install': True,
    'license': 'LGPL-3',
}

```

## File: data\sale_pdf_quote_builder_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <record id="base.main_company" model="res.company">
        <field name="sale_header"
               type="base64"
               file="sale_pdf_quote_builder/static/quote_builder_pdf/quote_header_pages.pdf"/>
        <field name="sale_header_name">Header Example.pdf</field>
        <field name="sale_footer"
               type="base64"
               file="sale_pdf_quote_builder/static/quote_builder_pdf/quote_footer_pages.pdf"/>
        <field name="sale_footer_name">Footer Example.pdf</field>
    </record>

    <record id="product.product_product_4_product_template_document" model="product.document">
        <field name="attached_on">inside</field>
    </record>

    <record id="product.product_product_25_document" model="product.document">
        <field name="attached_on">inside</field>
    </record>

    <record id="product.consu_delivery_02_document" model="product.document">
        <field name="attached_on">inside</field>
    </record>

</odoo>

```

## File: models\ir_actions_report.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import io

from odoo import models
from odoo.tools import format_amount, format_date, format_datetime, pdf
from odoo.tools.pdf import PdfFileWriter, PdfFileReader, NameObject, createStringObject


class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    def _render_qweb_pdf_prepare_streams(self, report_ref, data, res_ids=None):
        result = super()._render_qweb_pdf_prepare_streams(report_ref, data, res_ids=res_ids)
        if self._get_report(report_ref).report_name != 'sale.report_saleorder':
            return result

        orders = self.env['sale.order'].browse(res_ids)

        for order in orders:
            initial_stream = result[order.id]['stream']
            if initial_stream:
                order_template = order.sale_order_template_id
                header_record = order_template if order_template.sale_header else order.company_id
                footer_record = order_template if order_template.sale_footer else order.company_id
                has_header = bool(header_record.sale_header)
                has_footer = bool(footer_record.sale_footer)
                included_product_docs = self.env['product.document']
                doc_line_id_mapping = {}
                for line in order.order_line:
                    product_product_docs = line.product_id.product_document_ids
                    product_template_docs = line.product_template_id.product_document_ids
                    doc_to_include = (
                        product_product_docs.filtered(lambda d: d.attached_on == 'inside')
                        or product_template_docs.filtered(lambda d: d.attached_on == 'inside')
                    )
                    included_product_docs = included_product_docs | doc_to_include
                    doc_line_id_mapping.update({doc.id: line.id for doc in doc_to_include})

                if (not has_header and not included_product_docs and not has_footer):
                    continue

                writer = PdfFileWriter()
                if has_header:
                    self._add_pages_to_writer(writer, base64.b64decode(header_record.sale_header))
                if included_product_docs:
                    for doc in included_product_docs:
                        self._add_pages_to_writer(
                            writer, base64.b64decode(doc.datas), doc_line_id_mapping[doc.id]
                        )
                self._add_pages_to_writer(writer, initial_stream.getvalue())
                if has_footer:
                    self._add_pages_to_writer(writer, base64.b64decode(footer_record.sale_footer))

                form_fields = self._get_form_fields_mapping(order, doc_line_id_mapping)
                pdf.fill_form_fields_pdf(writer, form_fields=form_fields)
                with io.BytesIO() as _buffer:
                    writer.write(_buffer)
                    stream = io.BytesIO(_buffer.getvalue())
                result[order.id].update({'stream': stream})

        return result

    def _add_pages_to_writer(self, writer, document, sol_id=None):
        prefix = f'{sol_id}_' if sol_id else ''
        reader = PdfFileReader(io.BytesIO(document), strict=False)
        sol_field_names = self._get_sol_form_fields_names()
        for page_id in range(0, reader.getNumPages()):
            page = reader.getPage(page_id)
            if sol_id and page.get('/Annots'):
                # Prefix all form fields in the document with the sale order line id.
                # This is necessary to avoid conflicts between fields with the same name.
                for j in range(0, len(page['/Annots'])):
                    reader_annot = page['/Annots'][j].getObject()
                    if reader_annot.get('/T') in sol_field_names:
                        reader_annot.update({
                            NameObject("/T"): createStringObject(prefix + reader_annot.get('/T'))
                        })
            writer.addPage(page)

    def _get_sol_form_fields_names(self):
        """ List of specific pdf fields name for an order line that needs to be renamed in the pdf.
        Override this method to add new fields to the list.
        """
        return ['description', 'quantity', 'uom', 'price_unit', 'discount', 'product_sale_price',
                'taxes', 'tax_excl_price', 'tax_incl_price']

    def _get_form_fields_mapping(self, order, doc_line_id_mapping=None):
        """ Dictionary mapping specific pdf fields name to Odoo fields data for a sale order.
        Override this method to add new fields to the mapping.

        :param recordset order: sale.order record
        :rtype: dict
        :return: mapping of fields name to Odoo fields data

        Note: order.ensure_one()
        """
        order.ensure_one()
        env = self.with_context(use_babel=True).env
        tz = order.partner_id.tz or self.env.user.tz or 'UTC'
        lang_code = order.partner_id.lang or self.env.user.lang
        form_fields_mapping = {
            'name': order.name,
            'partner_id__name': order.partner_id.name,
            'user_id__name': order.user_id.name,
            'amount_untaxed': format_amount(env, order.amount_untaxed, order.currency_id),
            'amount_total': format_amount(env, order.amount_total, order.currency_id),
            'delivery_date': format_datetime(env, order.commitment_date, tz=tz),
            'validity_date': format_date(env, order.validity_date, lang_code=lang_code),
            'client_order_ref': order.client_order_ref or '',
        }

        # Adding fields from each line, prefixed by the line_id to avoid conflicts
        lines_with_doc_ids = set(doc_line_id_mapping.values())
        for line in order.order_line.filtered(lambda sol: sol.id in lines_with_doc_ids):
            form_fields_mapping.update(self._get_sol_form_fields_mapping(line))

        return form_fields_mapping

    def _get_sol_form_fields_mapping(self, line):
        """ Dictionary mapping specific pdf fields name to Odoo fields data for a sale order line.

        Fields name are prefixed by the line id to avoid conflict between files.

        Override this method to add new fields to the mapping.

        :param recordset line: sale.order.line record
        :rtype: dict
        :return: mapping of prefixed fields name to Odoo fields data

        Note: line.ensure_one()
        """
        line.ensure_one()
        env = self.with_context(use_babel=True).env
        return {
            f'{line.id}_description': line.name,
            f'{line.id}_quantity': line.product_uom_qty,
            f'{line.id}_uom': line.product_uom.name,
            f'{line.id}_price_unit': format_amount(env, line.price_unit, line.currency_id),
            f'{line.id}_discount': line.discount,
            f'{line.id}_product_sale_price': format_amount(
                env, line.product_id.lst_price, line.product_id.currency_id
            ),
            f'{line.id}_taxes': ', '.join(tax.name for tax in line.tax_id),
            f'{line.id}_tax_excl_price': format_amount(env, line.price_subtotal, line.currency_id),
            f'{line.id}_tax_incl_price': format_amount(env, line.price_total, line.currency_id),
        }

```

## File: models\product_document.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError

from odoo.addons.sale_pdf_quote_builder import utils


class ProductDocument(models.Model):
    _inherit = 'product.document'

    attached_on = fields.Selection(
        selection_add=[('inside', "Inside quote")],
        help="Allows you to share the document with your customers within a sale.\n"
             "Leave it empty if you don't want to share this document with sales customer.\n"
             "Quotation: the document will be sent to and accessible by customers at any time.\n"
             "e.g. this option can be useful to share Product description files.\n"
             "Confirmed order: the document will be sent to and accessible by customers.\n"
             "e.g. this option can be useful to share User Manual or digital content bought"
             " on ecommerce. \n"
             "Inside quote: The document will be included in the pdf of the quotation \n"
             "and sale order between the header pages and the quote table. ",
    )

    @api.constrains('attached_on', 'datas', 'type')
    def _check_attached_on_and_datas_compatibility(self):
        for doc in self.filtered(lambda doc: doc.attached_on == 'inside'):
            if doc.type != 'binary':
                raise ValidationError(_(
                    "When attached inside a quote, the document must be a file, not a URL."
                ))
            if doc.datas and not doc.mimetype.endswith('pdf'):
                raise ValidationError(_("Only PDF documents can be attached inside a quote."))
            utils._ensure_document_not_encrypted(base64.b64decode(doc.datas))

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import api, fields, models

from odoo.addons.sale_pdf_quote_builder import utils


class ResCompany(models.Model):
    _inherit = 'res.company'

    sale_header = fields.Binary(string="Header pages")
    sale_header_name = fields.Char()
    sale_footer = fields.Binary(string="Footer pages")
    sale_footer_name = fields.Char()

    @api.constrains('sale_header')
    def _ensure_header_not_encrypted(self):
        for company in self:
            if company.sale_header:
                utils._ensure_document_not_encrypted(base64.b64decode(company.sale_header))

    @api.constrains('sale_footer')
    def _ensure_footer_not_encrypted(self):
        for company in self:
            if company.sale_footer:
                utils._ensure_document_not_encrypted(base64.b64decode(company.sale_footer))

```

## File: models\sale_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    def _filter_product_documents(self, documents):
        return (
            super()._filter_product_documents(documents)
            | documents.filtered(lambda document: document.attached_on == 'inside')
        )

```

## File: models\sale_order_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import api, fields, models

from odoo.addons.sale_pdf_quote_builder import utils


class SaleOrderTemplate(models.Model):
    _inherit = 'sale.order.template'

    sale_header = fields.Binary(
        string="Header pages", default=lambda self: self.env.company.sale_header)
    sale_header_name = fields.Char(default=lambda self: self.env.company.sale_header_name)
    sale_footer = fields.Binary(
        string="Footer pages", default=lambda self: self.env.company.sale_footer)
    sale_footer_name = fields.Char(default=lambda self: self.env.company.sale_footer_name)

    @api.constrains('sale_header')
    def _ensure_header_encryption(self):
        for template in self:
            if template.sale_header:
                utils._ensure_document_not_encrypted(base64.b64decode(template.sale_header))

    @api.constrains('sale_footer')
    def _ensure_footer_encryption(self):
        for template in self:
            if template.sale_footer:
                utils._ensure_document_not_encrypted(base64.b64decode(template.sale_footer))

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import ir_actions_report
from . import product_document
from . import res_company
from . import sale_order
from . import sale_order_template

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
                <page name="pdf_quote" string="PDF Quote Builder">
                    <group>
                        <p class="text-muted" colspan="2">
                            Provide header pages and footer pages to compose an attractive quotation
                            with more information about your company, your products and your services.
                            The pdf of your quotes will be built by putting together header pages,
                            product descriptions, details of the quote and then the footer pages.
                            If empty, it will use those define in the company settings.<br/>
                            <a href="https://www.odoo.com/documentation/17.0/_downloads/5f0840ed187116c425fdac2ab4b592e1/pdfquotebuilderexamples.zip">
                                <i class="fa fa-arrow-right"/> Download examples
                            </a>
                        </p>
                        <group>
                            <field name="sale_header_name" invisible="1"/>
                            <field name="sale_header" filename="sale_header_name" options="{'accepted_file_extensions': '.pdf'}"/>
                            <field name="sale_footer_name" invisible="1"/>
                            <field name="sale_footer" filename="sale_footer_name" options="{'accepted_file_extensions': '.pdf'}"/>
                        </group>
                        <p class="text-muted" colspan="2">
                            Products descriptions are pdf documents you can add directly on products.
                            To do so, go on a product, find the "product documents" button, then add a
                            new pdf document with a visibility set as "Inside Quotes". For each product
                            in the quote, if the product has an "inside quotes" document, this document
                            will be added after header pages and before the quotation details.
                        </p>
                        <p class="text-muted" colspan="2">
                            Some information specific to the quote (customer name, quotation reference, ... )
                            can be injected in these documents using pdf forms.
                            Refer to the documentation to know more about this feature.
                            <a href="https://www.odoo.com/documentation/17.0/applications/sales/sales/send_quotations/pdf_quote_builder.html">
                                Learn more from the documentation.
                            </a>
                        </p>
                    </group>
                </page>
            </notebook>
        </field>
    </record>

</odoo>

```

## File: wizards\res_config_settings.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    sale_header = fields.Binary(related='company_id.sale_header', readonly=False)
    sale_header_name = fields.Char(related='company_id.sale_header_name', readonly=False)
    sale_footer = fields.Binary(related='company_id.sale_footer', readonly=False)
    sale_footer_name = fields.Char(related='company_id.sale_footer_name', readonly=False)

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
            <setting id="sale_pdf_quote_builder" position="replace">
                <setting id="sale_pdf_quote_builder"
                         string="PDF Quote builder"
                         help="Make your quote attractive by adding header pages, product descriptions and footer pages to your quote."
                         documentation="/applications/sales/sales/send_quotations/pdf_quote_builder.html"
                         company_dependent="1">
                    <div class="mt16">
                        <field name="sale_header_name" invisible="1"/>
                        <label for="sale_header" class="me-2"/>
                        <field name="sale_header" filename="sale_header_name" options="{'accepted_file_extensions': '.pdf'}"/>
                    </div>
                    <div class="mt16">
                        <field name="sale_footer_name" invisible="1"/>
                        <label for="sale_footer" class="me-2"/>
                        <field name="sale_footer" filename="sale_footer_name" options="{'accepted_file_extensions': '.pdf'}"/>
                    </div>
                    <a href="https://www.odoo.com/documentation/17.0/_downloads/5f0840ed187116c425fdac2ab4b592e1/pdfquotebuilderexamples.zip">
                        <i class="fa fa-arrow-right"/> Download examples
                    </a>
                </setting>
            </setting>
        </field>
    </record>

</odoo>

```

## File: wizards\__init__.py

```python
from . import res_config_settings

```

