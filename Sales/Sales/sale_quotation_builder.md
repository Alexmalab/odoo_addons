# Odoo Module: sale_quotation_builder

Category: Sales/Sales

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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Quotation Builder',
    'category': 'Sales/Sales',
    'summary': 'Build great quotation templates',
    'website': 'https://www.odoo.com/page/quote-builder',
    'version': '1.0',
    'description': "Design great quotation templates with building blocks to significantly boost your success rate.",
    'depends': ['website', 'sale_management', 'website_mail'],
    'data': [
        'data/sale_order_template_data.xml',
        'views/sale_portal_templates.xml',
        'views/sale_order_template_views.xml',
        'views/res_config_settings_views.xml',
        'views/sale_order_views.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import http
from odoo.http import request
from odoo.addons.http_routing.models.ir_http import unslug
from odoo.addons.portal.controllers.portal import CustomerPortal


class CustomerPortal(CustomerPortal):

    @http.route(["/sale_quotation_builder/template/<string:template_id>"], type='http', auth="user", website=True)
    def sale_quotation_builder_template_view(self, template_id, **post):
        template_id = unslug(template_id)[-1]
        template = request.env['sale.order.template'].browse(template_id).with_context(
            allowed_company_ids=request.env.user.company_ids.ids,
        )
        values = {'template': template}
        return request.render('sale_quotation_builder.so_template', values)

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import portal

```

## File: data\sale_order_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- no update so users can freely customize/delete the template -->
    <data noupdate="1">
        <record id="sale_order_template_default" model="sale.order.template">
            <field name="name">Default Template</field>
            <field name="number_of_days">30</field>

            <field name="website_description" type="xml">
                <section data-snippet-id="title" class="mt32">
                    <h2 class="o_page_header">About us</h2>
                </section>
                <section data-snippet-id="text-block">
                    <div class="row">
                        <div class="col-lg-12">
                            <p>
                                This is a <strong>sample quotation template</strong>. You should
                                customize it to fit your own needs from the <i>Sales</i>
                                application, using the menu: Configuration /
                                Quotation Templates.
                            </p><p>
                                Great quotation templates will significantly
                                <strong>boost your success rate</strong>. The
                                first section is usually about your company,
                                your references, your methodology or
                                guarantees, your team, SLA, terms and conditions, etc.
                            </p>
                        </div>
                    </div>
                </section>
                <section data-snippet-id="quality">
                    <div class="card-deck">
                        <div class="card">
                            <div class="card-header">Our Quality</div>
                            <div class="card-body">
                                Product quality is the foundation we
                                stand on; we build it with a relentless
                                focus on fabric, performance and craftsmanship.
                            </div>
                        </div>
                        <div class="card">
                            <div class="card-header">Our Service</div>
                            <div class="card-body">
                                As a leading professional services firm,
                                we know that success is all about the
                                commitment we put on strong services.
                            </div>
                        </div>
                        <div class="card">
                            <div class="card-header">Price</div>
                            <div class="card-body">
                                We always ensure that our products are
                                set at a fair price so that you will be
                                happy to buy them.
                            </div>
                        </div>
                    </div>
                </section>
                <section data-snippet-id="title" class="mt32">
                    <h2 class="o_page_header">Our Offer</h2>
                </section>
                <section data-snippet-id="text-block">
                    <p>
                        You can <strong>set a description per product</strong>. Odoo will
                        automatically create a quotation using the descriptions
                        of all products in the proposal. The table of content
                        on the left is generated automatically using the styles you
                        used in your description (heading 1, heading 2, ...)
                    </p><p>
                        If you edit a quotation from the 'Preview' of a quotation, you will
                        update that quotation only. If you edit the quotation
                        template (from the Configuration menu), all future quotations will
                        use this modified template.
                    </p>
                </section>
            </field>
        </record>

        <function model="res.company" name="_set_default_sale_order_template_id_if_empty"/>
    </data>
</odoo>

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api
from odoo.tools.translate import html_translate


class ProductTemplate(models.Model):
    _inherit = "product.template"

    quotation_only_description = fields.Html('Quotation Only Description', sanitize_attributes=False,
        translate=html_translate, help="The quotation description (not used on eCommerce)")

    quotation_description = fields.Html('Quotation Description', compute='_compute_quotation_description',
        sanitize_attributes=False, help="This field uses the Quotation Only Description if it is defined, otherwise it will try to read the eCommerce Description.")

    def _compute_quotation_description(self):
        for record in self:
            if record.quotation_only_description:
                record.quotation_description = record.quotation_only_description
            elif hasattr(record, 'website_description') and record.website_description:
                record.quotation_description = record.website_description
            else:
                record.quotation_description = ''

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


class ResCompany(models.Model):
    _inherit = 'res.company'

    @api.model
    def _set_default_sale_order_template_id_if_empty(self):
        template = self.env.ref('sale_quotation_builder.sale_order_template_default', raise_if_not_found=False)
        if not template:
            return
        companies = self.sudo().search([])
        for company in companies:
            company.sale_order_template_id = company.sale_order_template_id or template

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.translate import html_translate


class SaleOrder(models.Model):
    _inherit = 'sale.order'

    website_description = fields.Html('Website Description', sanitize_attributes=False, translate=html_translate, sanitize_form=False)

    @api.onchange('partner_id')
    def onchange_update_description_lang(self):
        if not self.sale_order_template_id:
            return
        else:
            template = self.sale_order_template_id.with_context(lang=self.partner_id.lang)
            self.website_description = template.website_description

    def _compute_line_data_for_template_change(self, line):
        vals = super(SaleOrder, self)._compute_line_data_for_template_change(line)
        vals.update(website_description=line.website_description)
        return vals

    def _compute_option_data_for_template_change(self, option):
        vals = super(SaleOrder, self)._compute_option_data_for_template_change(option)
        vals.update(website_description=option.website_description)
        return vals

    @api.onchange('sale_order_template_id')
    def onchange_sale_order_template_id(self):
        ret = super(SaleOrder, self).onchange_sale_order_template_id()
        if self.sale_order_template_id:
            template = self.sale_order_template_id.with_context(lang=self.partner_id.lang)
            self.website_description = template.website_description
        return ret


class SaleOrderLine(models.Model):
    _inherit = "sale.order.line"

    website_description = fields.Html('Website Description', sanitize=False, translate=html_translate, sanitize_form=False)

    @api.model
    def create(self, values):
        values = self._inject_quotation_description(values)
        return super(SaleOrderLine, self).create(values)

    def write(self, values):
        values = self._inject_quotation_description(values)
        return super(SaleOrderLine, self).write(values)

    def _inject_quotation_description(self, values):
        values = dict(values or {})
        if not values.get('website_description') and values.get('product_id'):
            product = self.env['product.product'].browse(values['product_id'])
            values.update(website_description=product.quotation_description)
        return values


class SaleOrderOption(models.Model):
    _inherit = "sale.order.option"

    website_description = fields.Html('Website Description', sanitize_attributes=False, translate=html_translate)

    @api.onchange('product_id', 'uom_id')
    def _onchange_product_id(self):
        ret = super(SaleOrderOption, self)._onchange_product_id()
        if self.product_id:
            product = self.product_id.with_context(lang=self.order_id.partner_id.lang)
            self.website_description = product.quotation_description
        return ret

    def _get_values_to_add_to_order(self):
        values = super(SaleOrderOption, self)._get_values_to_add_to_order()
        values.update(website_description=self.website_description)
        return values

```

## File: models\sale_order_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
from odoo.tools.translate import html_translate


class SaleOrderTemplate(models.Model):
    _inherit = "sale.order.template"

    website_description = fields.Html('Website Description', translate=html_translate, sanitize_attributes=False, sanitize_form=False)

    def open_template(self):
        self.ensure_one()
        return {
            'type': 'ir.actions.act_url',
            'target': 'self',
            'url': '/sale_quotation_builder/template/%d' % self.id
        }


class SaleOrderTemplateLine(models.Model):
    _inherit = "sale.order.template.line"

    website_description = fields.Html('Website Description', related='product_id.product_tmpl_id.quotation_only_description', translate=html_translate, readonly=False, sanitize_form=False)

    @api.onchange('product_id')
    def _onchange_product_id(self):
        ret = super(SaleOrderTemplateLine, self)._onchange_product_id()
        if self.product_id:
            self.website_description = self.product_id.quotation_description
        return ret

    @api.model
    def create(self, values):
        values = self._inject_quotation_description(values)
        return super(SaleOrderTemplateLine, self).create(values)

    def write(self, values):
        values = self._inject_quotation_description(values)
        return super(SaleOrderTemplateLine, self).write(values)

    def _inject_quotation_description(self, values):
        values = dict(values or {})
        if not values.get('website_description') and values.get('product_id'):
            product = self.env['product.product'].browse(values['product_id'])
            values['website_description'] = product.quotation_description
        return values


class SaleOrderTemplateOption(models.Model):
    _inherit = "sale.order.template.option"

    website_description = fields.Html('Website Description', translate=html_translate, sanitize_attributes=False)

    @api.onchange('product_id')
    def _onchange_product_id(self):
        ret = super(SaleOrderTemplateOption, self)._onchange_product_id()
        if self.product_id:
            self.website_description = self.product_id.quotation_description
        return ret

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product_template
from . import res_company
from . import sale_order
from . import sale_order_template

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_quotation_builder.res_config_settings_view_form_inherit" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.sale.management.inherit.sale.quotation.builder</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="sale_management.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//label[@for='module_sale_quotation_builder']/following::div/em" position="replace"/>
        </field>
    </record>
</odoo>

```

## File: views\sale_order_template_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="sale_order_template_view_form_inherit_sale_quotation_builder" model="ir.ui.view">
        <field name="name">sale.order.template.form.inherit.sale_quotation_builder</field>
        <field name="inherit_id" ref="sale_management.sale_order_template_view_form"/>
        <field name="model">sale.order.template</field>
        <field name="type">form</field>
        <field name="arch" type="xml">
            <xpath expr="//sheet" position='before'>
                <header>
                    <button name="open_template" class="oe_highlight" string="Design Template" type="object"/>
                </header>
            </xpath>

            <xpath expr="//notebook[@name='description']" position="inside">
                <page string="Website Description" name="website_description">
                    <field name="website_description" />
                </page>
            </xpath>

            <xpath expr="//tree/field[@name='product_uom_id']" position="after">
                <field name="website_description" invisible="1"/>
            </xpath>

            <xpath expr="//notebook[@name='main_book']" position="inside">
                <field name="website_description" invisible="1"/>
            </xpath>

        </field>
    </record>

</odoo>

```

## File: views\sale_order_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="sale_order_form_quote_design" model="ir.ui.view">
        <field name="name">sale.order.form.sale_quotation_builder</field>
        <field name="model">sale.order</field>
        <field name="inherit_id" ref="sale_management.sale_order_form_quote"/>
        <field name="arch" type="xml">

            <xpath expr="//page/field[@name='order_line']/tree/field[@name='name']" position="after">
                <field name="website_description" invisible="1"/>
            </xpath>
            <xpath expr="//page/field[@name='order_line']/form/field[@name='name']" position="after">
                <field name="website_description" invisible="1"/>
            </xpath>

            <xpath expr="//page/field[@name='sale_order_option_ids']/kanban/field[@name='product_id']" position="after">
                <field name="website_description" invisible="1" readonly="1" force_save="1"/>
            </xpath>
            <xpath expr="//page/field[@name='sale_order_option_ids']/form//field[@name='name']" position="after">
                <field name="website_description" invisible="1" readonly="1" force_save="1"/>
            </xpath>
            <xpath expr="//page/field[@name='sale_order_option_ids']/tree/field[@name='name']" position="after">
                <field name="website_description" invisible="1" readonly="1" force_save="1"/>
            </xpath>

            <xpath expr="//button[@name='button_add_to_order']" position="after">
            	<field name="website_description" invisible="1"/>
            </xpath>

            <xpath expr="//field[@name='require_payment']" position="after">
                <field name="website_description" invisible="1"/>
            </xpath>

        </field>
    </record>

</odoo>

```

## File: views\sale_portal_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="sale_order_portal_content_inherit_sale_quotation_builder" name="Order Design" inherit_id="sale.sale_order_portal_content">
        <xpath expr="//div[@id='informations']" position="after">
            <div t-field="sale_order.website_description" class="oe_no_empty"/>
            <t t-set="product_tmpl_ids" t-value="[]"/>
            <t t-foreach="sale_order.order_line" t-as="line">
                <t t-if="line.product_id.product_tmpl_id.id not in product_tmpl_ids">
                    <t t-set="product_tmpl_ids" t-value="product_tmpl_ids + [line.product_id.product_tmpl_id.id]"/>
                    <a t-att-id="line.id"/>
                    <div class="alert alert-info alert-dismissable mt16 css_non_editable_mode_hidden o_not_editable" t-ignore="True" role="status">
                        <button type="button" class="close" data-dismiss="alert" aria-label="Close">×</button>
                        Product: <strong t-esc="line.product_id.name"/>:
                        this content will appear on the quotation only if this
                        product is not removed.
                    </div>
                    <div t-att-class="'oe_no_empty' if line.website_description else 'oe_no_empty d-print-none'" t-field="line.website_description"/>
                </t>
            </t>
        </xpath>
    </template>

    <!-- Template to edit the quotation template with the website editor -->
    <template id="so_template" name="SO Template">
        <t t-call="website.layout">
            <body>
                <t t-set="o_portal_fullwidth_alert">
                    <t t-call="portal.portal_back_in_edit_mode">
                        <t t-set="backend_url" t-value="'/web#model=%s&amp;id=%s&amp;action=%s&amp;view_type=form' % (template._name, template.id, request.env.ref('sale_management.sale_order_template_action').id)"/>
                        <t t-set="custom_html">This is a preview of the sale order template.</t>
                    </t>
                </t>
                <div class="container o_sale_order">
                    <div class="row mt16">
                        <div class="col-lg-9 ml-auto">
                            <div class="alert alert-info" t-ignore="True" role="status">
                                <p>
                                    <strong>Template Header:</strong> this content
                                    will appear on all quotations using this
                                    template.
                                </p>
                                <p class="text-muted">
                                    Titles with style <i>Heading 2</i> and
                                    <i>Heading 3</i> will be used to generate the
                                    table of content automatically.
                                </p>
                            </div>
                            <div id="template_introduction" t-field="template.website_description" class="oe_no_empty"/>
                            <t t-set="product_tmpl_ids" t-value="[]"/>
                            <t t-foreach="template.sale_order_template_line_ids" t-as="line">
                                <t t-if="line.product_id.product_tmpl_id.id not in product_tmpl_ids">
                                    <t t-set="product_tmpl_ids" t-value="product_tmpl_ids + [line.product_id.product_tmpl_id.id]"/>
                                    <div class="alert alert-info mt16" t-ignore="True" role="status">
                                        Product: <strong t-esc="line.product_id.name"/>:
                                        this content will appear on the quotation only if this
                                        product is put on the quote.
                                    </div>
                                    <div t-field="line.website_description" class="oe_no_empty"/>
                                </t>
                            </t>
                            <t t-set="product_tmpl_ids" t-value="[]"/>
                            <t t-foreach="template.sale_order_template_option_ids" t-as="option_line">
                                <t t-if="option_line.product_id.product_tmpl_id.id not in product_tmpl_ids">
                                    <t t-set="product_tmpl_ids" t-value="product_tmpl_ids + [option_line.product_id.product_tmpl_id.id]"/>
                                    <div class="alert alert-info mt16" t-ignore="True" role="status">
                                        Optional Product: <strong t-esc="option_line.product_id.name"/>:
                                        this content will appear on the quotation only if this
                                        product is used in the quote.
                                    </div>
                                    <div t-field="option_line.website_description" class="oe_no_empty"/>
                                </t>
                            </t>
                            <section id="terms" class="container" t-if="template.note">
                                <h1 t-ignore="True">Terms &amp; Conditions</h1>
                                <p t-field="template.note"/>
                            </section>
                        </div>
                    </div>
                </div>
            </body>
        </t>
    </template>

    <template id="brand_promotion" inherit_id="website.brand_promotion">
        <xpath expr="//t[@t-call='web.brand_promotion_message']" position="replace">
            <t t-call="web.brand_promotion_message">
                <t t-set="_message">
                    An awesome <a target="_blank" href="https://www.odoo.com/page/crm?utm_source=db&amp;utm_medium=portal">Open Source CRM</a>
                </t>
                <t t-set="_utm_medium" t-valuef="portal"/>
            </t>
        </xpath>
    </template>

</odoo>

```

