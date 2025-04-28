# Odoo Module: website_sale_slides

Category: Hidden

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
    'name': "Sell Courses",
    'summary': 'Sell your courses online',
    'description': """Sell your courses using the e-commerce features of the website.""",
    'category': 'Hidden',
    'version': '1.0',

    'depends': ['website_slides', 'website_sale'],
    'installable': True,
    'data': [
        'data/product_data.xml',
        'report/sale_report_views.xml',
        'views/website_slides_menu_views.xml',
        'views/slide_channel_views.xml',
        'views/website_sale_templates.xml',
        'views/website_slides_templates.xml',
        'views/snippets.xml',
    ],
    'demo': [
        'data/product_demo.xml',
        'data/slide_demo.xml',
        'data/sale_order_demo.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'website_sale_slides/static/src/js/**/*',
            'website_sale_slides/static/src/xml/**/*',
        ],
        'web.assets_tests': [
            'website_sale_slides/static/tests/tours/*.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\sale.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.http import request
from odoo.addons.website_sale.controllers.main import WebsiteSale


class WebsiteSaleSlides(WebsiteSale):

    def _prepare_shop_payment_confirmation_values(self, order):
        values = super()._prepare_shop_payment_confirmation_values(order)
        if order.order_line.product_id.channel_ids:
            channel_partners = request.env['slide.channel.partner'].sudo().search([
                ('partner_id', '=', order.partner_id.id),
                ('channel_id', 'in', order.order_line.product_id.channel_ids.ids),
            ])
            values['course_memberships'] = {
                channel_partner.channel_id: channel_partner
                for channel_partner in channel_partners
            }
        return values

```

## File: controllers\slides.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_slides.controllers.main import WebsiteSlides
from odoo.http import request, route
from odoo.tools import format_amount


class WebsiteSaleSlides(WebsiteSlides):

    @route('/slides/get_course_products', type='json', auth='user')
    def get_course_products(self):
        """Return a list of the course products values with formatted price."""
        products = request.env['product.product'].search([('detailed_type', '=', 'course')])

        return [{
            'id': product.id,
            'name': f'{product.name} ({format_amount(request.env, product.list_price, product.currency_id)})',
        } for product in products]

    def _prepare_additional_channel_values(self, values, **kwargs):
        values = super(WebsiteSaleSlides, self)._prepare_additional_channel_values(values, **kwargs)
        channel = values['channel']
        if channel.enroll == 'payment':
            # search the product to apply ACLs, notably on published status, to avoid access errors
            product = request.env['product.product'].search([('id', '=', channel.product_id.id)]) if channel.product_id else request.env['product.product']
            if product:
                values['product_info'] = product._get_combination_info_variant()
            else:
                values['product_info'] = False
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import sale
from . import slides

```

## File: data\product_data.xml

```xml
<odoo><data>
    <record id="product_category_courses" model="product.category">
        <field name="parent_id" ref="product.product_category_1"/>
        <field name="name">Courses</field>
    </record>

    <record id="default_product_course" model="product.product">
        <field name="name">Course Access</field>
        <field name="standard_price">99.99</field>
        <field name="list_price">99.99</field>
        <field name="detailed_type">course</field>
        <field name="invoice_policy">order</field>
        <field name="is_published" eval="True"/>
        <field name="image_1920" type="base64" file="website_sale_slides/static/img/default_course_product.svg"/>
        <field name="categ_id" ref="product_category_courses"/>
    </record>
</data></odoo>

```

## File: data\product_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- CHANNEL 1: Taking care of Trees -->
    <!-- ================================================== -->
    <record id="product_course_channel_1" model="product.product">
        <field name="name">Taking care of Trees Course</field>
        <field name="standard_price">150.0</field>
        <field name="list_price">150.0</field>
        <field name="detailed_type">course</field>
        <field name="invoice_policy">order</field>
        <field name="is_published" eval="True"/>
        <field name="image_1920" type="base64" file="website_sale_slides/static/img/channel_demo_tree_1.jpg"/>
    </record>

    <!-- CHANNEL 5: Basics of Furniture Creation -->
    <!-- ================================================== -->
    <record id="product_course_channel_5" model="product.product">
        <field name="name">Basics of Furniture Creation</field>
        <field name="standard_price">200.0</field>
        <field name="list_price">200.0</field>
        <field name="detailed_type">course</field>
        <field name="invoice_policy">order</field>
        <field name="is_published" eval="True"/>
        <field name="image_1920" type="base64" file="website_sale_slides/static/img/channel_demo_furniture_2.jpg"/>
    </record>

    <!-- CHANNEL 6: DIY Furniture -->
    <!-- ================================================== -->
    <record id="product_course_channel_6" model="product.product">
        <field name="name">DIY Furniture Course</field>
        <field name="standard_price">100.0</field>
        <field name="list_price">100.0</field>
        <field name="detailed_type">course</field>
        <field name="invoice_policy">order</field>
        <field name="is_published" eval="True"/>
        <field name="image_1920" type="base64" file="website_sale_slides/static/img/product_course.png"/>
    </record>
</data></odoo>

```

## File: data\sale_order_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">

    <!-- CHANNEL 6: DIY Furniture -->
    <!-- ================================================== -->

    <record id="sale_order_course_1" model="sale.order">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=31)"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="partner_invoice_id" ref="base.partner_demo"/>
        <field name="partner_shipping_id" ref="base.partner_demo"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=30)"/>
        <field name="team_id" ref="sales_team.salesteam_website_sales"/>
        <field name="website_id" eval="1"/>
        <field name="tag_ids" eval="[(4, ref('sales_team.categ_oppor2')), (4, ref('sales_team.categ_oppor5'))]"/>
    </record>
    <record id="sale_order_course_1_line_1" model="sale.order.line">
        <field name="order_id" ref="sale_order_course_1"/>
        <field name="name" model="sale.order.line" eval="obj().env.ref('website_sale_slides.product_course_channel_6').get_product_multiline_description_sale()"/>
        <field name="product_id" ref="website_sale_slides.product_course_channel_6"/>
        <field name="product_uom_qty">1</field>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="price_unit">100.0</field>
    </record>

    <record id="sale_order_course_2" model="sale.order">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=100)"/>
        <field name="partner_id" ref="base.partner_demo_portal"/>
        <field name="partner_invoice_id" ref="base.partner_demo_portal"/>
        <field name="partner_shipping_id" ref="base.partner_demo_portal"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=100)"/>
        <field name="team_id" ref="sales_team.salesteam_website_sales"/>
        <field name="website_id" eval="1"/>
    </record>
    <record id="sale_order_course_2_line_1" model="sale.order.line">
        <field name="order_id" ref="sale_order_course_2"/>
        <field name="name" model="sale.order.line" eval="obj().env.ref('website_sale_slides.product_course_channel_1').get_product_multiline_description_sale()"/>
        <field name="product_id" ref="website_sale_slides.product_course_channel_1"/>
        <field name="product_uom_qty">1</field>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="price_unit">150.0</field>
    </record>
    <record id="sale_order_course_2_line_2" model="sale.order.line">
        <field name="order_id" ref="sale_order_course_2"/>
        <field name="name" model="sale.order.line" eval="obj().env.ref('website_sale_slides.product_course_channel_6').get_product_multiline_description_sale()"/>
        <field name="product_id" ref="website_sale_slides.product_course_channel_6"/>
        <field name="product_uom_qty">1</field>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="price_unit">100.0</field>
    </record>

    <!-- CHANNEL 5: Basics of Furniture Creation -->
    <!-- ================================================== -->

    <record id="sale_order_course_3" model="sale.order">
        <field name="create_date" eval="DateTime.now() - relativedelta(days=31)"/>
        <field name="partner_id" ref="base.partner_demo"/>
        <field name="partner_invoice_id" ref="base.partner_demo"/>
        <field name="partner_shipping_id" ref="base.partner_demo"/>
        <field name="user_id" ref="base.user_admin"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=30)"/>
        <field name="team_id" ref="sales_team.salesteam_website_sales"/>
        <field name="website_id" eval="1"/>
    </record>
    <record id="sale_order_course_3_line_1" model="sale.order.line">
        <field name="order_id" ref="sale_order_course_3"/>
        <field name="name" model="sale.order.line" eval="obj().env.ref('website_sale_slides.product_course_channel_5').get_product_multiline_description_sale()"/>
        <field name="product_id" ref="website_sale_slides.product_course_channel_5"/>
        <field name="product_uom_qty">1</field>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="price_unit">200.0</field>
    </record>

    <record id="website_sale_slides_activity_1" model="mail.activity">
        <field name="res_id" ref="website_sale_slides.sale_order_course_1"/>
        <field name="res_model_id" ref="sale.model_sale_order"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
        <field name="date_deadline" eval="DateTime.today().strftime('%Y-%m-%d %H:%M')"/>
        <field name="summary">Suggest optional products</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>
    <record id="website_sale_slides_activity_2" model="mail.activity">
        <field name="res_id" ref="website_sale_slides.sale_order_course_2"/>
        <field name="res_model_id" ref="sale.model_sale_order"/>
        <field name="activity_type_id" ref="mail.mail_activity_data_call"/>
        <field name="date_deadline" eval="(DateTime.today() - relativedelta(days=5)).strftime('%Y-%m-%d %H:%M')"/>
        <field name="summary">Discuss discount</field>
        <field name="create_uid" ref="base.user_admin"/>
        <field name="user_id" ref="base.user_admin"/>
    </record>

    <!-- Confirm sales -->
    <function model="sale.order" name="action_confirm" eval="[[
        ref('sale_order_course_1'),
        ref('sale_order_course_2'),
        ref('sale_order_course_3'),
    ]]"/>

</data></odoo>

```

## File: data\slide_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data noupdate="1">
    <!-- CHANNEL 1: Taking care of Trees -->
    <!-- ================================================== -->
    <record id="website_slides.slide_channel_demo_1_gard1" model="slide.channel">
        <field name="enroll">payment</field>
        <field name="product_id" ref="product_course_channel_1"/>
    </record>

    <!-- CHANNEL 5: Basics of Furniture Creation -->
    <!-- ================================================== -->
    <record id="website_slides.slide_channel_demo_5_furn2" model="slide.channel">
        <field name="enroll">payment</field>
        <field name="product_id" ref="product_course_channel_5"/>
    </record>

    <!-- CHANNEL 6: DIY Furniture -->
    <!-- ================================================== -->
    <record id="website_slides.slide_channel_demo_6_furn3" model="slide.channel">
        <field name="enroll">payment</field>
        <field name="product_id" ref="product_course_channel_6"/>
    </record>
</data></odoo>

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models


class Product(models.Model):
    _inherit = "product.product"

    channel_ids = fields.One2many('slide.channel', 'product_id', string='Courses')

    def get_product_multiline_description_sale(self):
        payment_channels = self.channel_ids.filtered(lambda course: course.enroll == 'payment')

        if not payment_channels:
            return super(Product, self).get_product_multiline_description_sale()

        new_line = '' if len(payment_channels) == 1 else '\n'
        return _('Access to: %s%s', new_line, '\n'.join(payment_channels.mapped('name')))

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    detailed_type = fields.Selection(selection_add=[
        ('course', 'Course'),
    ], ondelete={'course': 'set service'})

    def _detailed_type_mapping(self):
        type_mapping = super(ProductTemplate, self)._detailed_type_mapping()
        type_mapping['course'] = 'service'
        return type_mapping

    @api.model
    def _get_product_types_allow_zero_price(self):
        return super()._get_product_types_allow_zero_price() + ["course"]

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, _


class SaleOrder(models.Model):
    _inherit = "sale.order"

    def _action_confirm(self):
        """ If the product of an order line is a 'course', we add the client of the sale_order
        as a member of the channel(s) on which this product is configured (see slide.channel.product_id). """
        result = super(SaleOrder, self)._action_confirm()

        so_lines = self.env['sale.order.line'].search(
            [('order_id', 'in', self.ids)]
        )
        products = so_lines.mapped('product_id')
        related_channels = self.env['slide.channel'].search(
            [('product_id', 'in', products.ids), ('enroll', '=', 'payment')],
        )
        channel_products = related_channels.mapped('product_id')

        channels_per_so = {sale_order: self.env['slide.channel'] for sale_order in self}
        for so_line in so_lines:
            if so_line.product_id in channel_products:
                for related_channel in related_channels:
                    if related_channel.product_id == so_line.product_id:
                        channels_per_so[so_line.order_id] = channels_per_so[so_line.order_id] | related_channel

        for sale_order, channels in channels_per_so.items():
            channels.sudo()._action_add_members(sale_order.partner_id)

        return result

    def _verify_updated_quantity(self, order_line, product_id, new_qty, **kwargs):
        """Forbid quantity updates on courses lines."""
        product = self.env['product.product'].browse(product_id)
        if product.detailed_type == 'course' and new_qty > 1:
            return 1, _('You can only add a course once in your cart.')
        return super()._verify_updated_quantity(order_line, product_id, new_qty, **kwargs)

```

## File: models\slide_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import AccessError


class Channel(models.Model):
    _inherit = 'slide.channel'

    def _get_default_product_id(self):
        product_courses = self.env['product.product'].search(
            [('detailed_type', '=', 'course')], limit=2)
        return product_courses.id if len(product_courses) == 1 else False

    enroll = fields.Selection(selection_add=[
        ('payment', 'On payment')
    ], ondelete={'payment': lambda recs: recs.write({'enroll': 'invite'})})
    product_id = fields.Many2one('product.product', 'Product', domain=[('detailed_type', '=', 'course')],
                                 default=_get_default_product_id)
    product_sale_revenues = fields.Monetary(
        string='Total revenues', compute='_compute_product_sale_revenues',
        groups="sales_team.group_sale_salesman")
    currency_id = fields.Many2one(related='product_id.currency_id')

    _sql_constraints = [
        ('product_id_check', "CHECK( enroll!='payment' OR product_id IS NOT NULL )", "Product is required for on payment channels.")
    ]

    @api.depends('product_id')
    def _compute_product_sale_revenues(self):
        domain = [
            ('state', 'in', self.env['sale.report']._get_done_states()),
            ('product_id', 'in', self.product_id.ids),
        ]
        rg_data = {
            product.id: price_total
            for product, price_total in self.env['sale.report']._read_group(domain, ['product_id'], ['price_total:sum'])
        }
        for channel in self:
            channel.product_sale_revenues = rg_data.get(channel.product_id.id, 0)

    @api.model_create_multi
    def create(self, vals_list):
        channels = super(Channel, self).create(vals_list)
        channels.filtered(lambda channel: channel.enroll == 'payment')._synchronize_product_publish()
        return channels

    def write(self, vals):
        res = super(Channel, self).write(vals)
        if 'is_published' in vals:
            self.filtered(lambda channel: channel.enroll == 'payment')._synchronize_product_publish()
        return res

    def _synchronize_product_publish(self):
        """
        Ensure that when publishing a course that its linked product is also published
        If all courses linked to a product are unpublished, we also unpublished the product
        """
        if not self:
            return
        self.filtered(lambda channel: channel.is_published and not channel.product_id.is_published).sudo().product_id.write({'is_published': True})

        unpublished_channel_products = self.filtered(lambda channel: not channel.is_published).product_id
        group_data = self._read_group(
            [('is_published', '=', True), ('product_id', 'in', unpublished_channel_products.ids)],
            ['product_id'],
        )
        used_product_ids = {product.id for [product] in group_data}
        product_to_unpublish = unpublished_channel_products.filtered(lambda product: product.id not in used_product_ids)
        if product_to_unpublish:
            product_to_unpublish.sudo().write({'is_published': False})

    def action_view_sales(self):
        action = self.env["ir.actions.actions"]._for_xml_id("website_sale_slides.sale_report_action_slides")
        action['domain'] = [('product_id', 'in', self.product_id.ids)]
        return action

    def _filter_add_members(self, target_partners, raise_on_access=False):
        """ Overridden to add 'payment' channels to the filtered channels. People
        that can write on payment-based channels can add members. """
        result = super(Channel, self)._filter_add_members(target_partners, raise_on_access=raise_on_access)
        on_payment = self.filtered(lambda channel: channel.enroll == 'payment')
        if on_payment:
            try:
                on_payment.check_access_rights('write')
                on_payment.check_access_rule('write')
            except AccessError:
                if raise_on_access:
                    raise AccessError(_('You are not allowed to add members to this course. Please contact the course responsible or an administrator.'))
            else:
                result |= on_payment
        return result

```

## File: models\website.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from odoo.osv import expression


class Website(models.Model):
    _inherit = 'website'

    def sale_product_domain(self):
        return expression.AND([
            super(Website, self).sale_product_domain(),
            [('detailed_type', '!=', 'course')],
        ])

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import product_product
from . import product_template
from . import sale_order
from . import slide_channel
from . import website

```

## File: report\sale_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_report_view_graph_slides" model="ir.ui.view">
         <field name="name">sale.report.view.graph.slides</field>
         <field name="model">sale.report</field>
         <field name="arch" type="xml">
             <graph string="eLearning Sales Analysis" type="line" sample="1">
                 <field name="date" interval="day"/>
                 <field name="price_total" type="measure"/>
             </graph>
         </field>
    </record>

	<record id="sale_report_action_slides" model="ir.actions.act_window">
        <field name="name">eLearning Revenues</field>
        <field name="res_model">sale.report</field>
        <field name="view_mode">graph,pivot</field>
        <field name="domain">[("product_id.channel_ids", "!=", False)]</field>
        <field name="context">{'group_by': ['date', 'product_id'], 'pivot_measures': ['price_total']}</field>
        <field name="view_id" ref="sale_report_view_graph_slides"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                No sales data yet!
            </p>
            <p>Come back once your courses starts selling to report on your revenues.</p>
        </field>
    </record>
</odoo>

```

## File: static\img\default_course_product.svg

```svg
<?xml version="1.0" ?><svg id="solid" viewBox="0 0 512 512" xmlns="http://www.w3.org/2000/svg"><title/><path d="M407.951,343.116a7.2,7.2,0,0,1,.048.733c.121,13.4-15.891,29.5-41.725,41.981C345.344,395.944,309.222,408,256,408s-89.344-12.056-110.274-22.17C119.892,373.348,103.88,357.246,104,343.849a7.2,7.2,0,0,1,.048-.733L115.1,243.624a7,7,0,0,1,9.82-5.615L242.911,290.9a32.043,32.043,0,0,0,26.179,0l117.986-52.89a7,7,0,0,1,9.82,5.615ZM474.988,153.4l-205.9-92.3a32.037,32.037,0,0,0-26.179,0l-205.9,92.3a16,16,0,0,0,0,29.2l205.9,92.3a32.043,32.043,0,0,0,26.179,0L440,198.284v59.874A15.98,15.98,0,0,0,432,272V376h32V272a15.98,15.98,0,0,0-8-13.842V191.111l18.987-8.511a16,16,0,0,0,0-29.2Z" style="fill:#3b3b3b"/></svg>
```

## File: static\src\js\slides_course_join.js

```javascript
/** @odoo-module **/

import CourseJoin from "@website_slides/js/slides_course_join";
import wUtils from "@website/js/utils";

const CourseJoinWidget = CourseJoin.courseJoinWidget;

CourseJoinWidget.include({
    init: function (parent, options) {
        this._super.apply(this, arguments);
        this.productId = options.channel.productId || false;
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * When the user joins the course, if it's set as "on payment" and the user is logged in,
     * we redirect to the shop page for this course.
     *
     * @param {MouseEvent} ev
     * @override
     * @private
     */
    _onClickJoin: function (ev) {
        ev.preventDefault();

        if (this.channel.channelEnroll === 'payment' && !this.publicUser) {
            const self = this;
            this.beforeJoin().then(function () {
                wUtils.sendRequest('/shop/cart/update', {
                    product_id: self.productId,
                    express: 1,
                });
            });
        } else {
            this._super.apply(this, arguments);
        }
    },
});

export default CourseJoinWidget;

```

## File: static\src\js\slides_course_quiz.js

```javascript
/** @odoo-module **/

import {websiteSlidesQuizNoFullscreen} from "@website_slides/js/slides_course_quiz";

websiteSlidesQuizNoFullscreen.include({
    _extractChannelData: function (slideData) {
        return Object.assign({}, this._super.apply(this, arguments), {
            productId: slideData.productId,
            enroll: slideData.enroll,
            currencyName: slideData.currencyName,
            currencySymbol: slideData.currencySymbol,
            price: slideData.price,
            hasDiscountedPrice: slideData.hasDiscountedPrice
        });
    }
});

```

## File: static\src\xml\slide_course_join.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="website_slides.slide.course.join" t-inherit-mode="extension">
        <xpath expr="//*[hasclass('o_wslides_js_course_join_link')]" position="inside">
            <t t-if="widget.channel.channelEnroll == 'payment'">
                <t t-if="widget.publicUser">
                    Sign in
                </t>
                <t t-else="">
                    Buy this Course
                </t>
            </t>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\website_sale_slides_quiz.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-inherit="website_slides.slide.slide.quiz.validation" t-inherit-mode="extension">
        <xpath expr="//*[hasclass('o_wslides_quiz_join_course_message')]" position="inside">
            <span t-if="widget.channel.channelEnroll == 'payment'">
                <t t-if="widget.publicUser">
                    Sign in and buy the course to take the quiz
                </t>
                <t t-else="">
                    Buy the course to validate your answers!
                </t>
            </span>
        </xpath>
    </t>
</templates>

```

## File: static\src\xml\website_slides_unsubscribe.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-inherit="website_slides.SlideUnsubscribeDialog" t-inherit-mode="extension">
        <xpath expr="//t[@t-if=&quot;state.mode === 'leave'&quot;] //p[last()]" position="after">
            <t t-if="props.enroll === 'payment'">
                <p class="alert alert-warning">
                    <i class="fa fa-exclamation-triangle fa-3x float-start me-3"></i>
                    This course is paid.<br/>
                    Leaving the course and re-enrolling afterwards means that you'll be charged again.
                </p>
            </t>
        </xpath>
    </t>

</templates>

```

## File: views\slide_channel_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo><data>
    <record id="slide_channel_view_form" model="ir.ui.view">
        <field name="name">slide.channel.view.form.inherit.sale</field>
        <field name="model">slide.channel</field>
        <field name="inherit_id" ref="website_slides.view_slide_channel_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='enroll']" position="after">
                <field name="product_id"
                    invisible="enroll != 'payment'"
                    required="enroll == 'payment'"
                    context="{'default_detailed_type': 'course', 'default_invoice_policy': 'order', 'default_purchase_ok': False, 'default_sale_ok': True, 'default_website_published': True}"/>
            </xpath>
            <xpath expr="//button[@name='action_redirect_to_members']" position="after">
                <button name="action_view_sales"
                    type="object"
                    icon="fa-usd"
                    class="oe_stat_button"
                    invisible="enroll != 'payment'"
                    groups="sales_team.group_sale_salesman">
                    <field name="product_sale_revenues" string="Sales" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="slide_channel_view_tree_report" model="ir.ui.view">
        <field name="name">slide.channel.view.tree.report.inherit.sale_slides</field>
        <field name="model">slide.channel</field>
        <field name="inherit_id" ref="website_slides.slide_channel_view_tree_report"/>
        <field name="arch" type="xml">
            <field name="members_completed_count" position="after">
                <field name="currency_id" column_invisible="True"/>
                <field name="product_sale_revenues" string="Total Revenues" sum="Total Revenues" widget="monetary"/>
            </field>
        </field>
    </record>

    <record id="slide_channel_view_kanban" model="ir.ui.view">
        <field name="name">slide.channel.view.kanban.inherit.sale</field>
        <field name="model">slide.channel</field>
        <field name="inherit_id" ref="website_slides.slide_channel_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='info_avg_rating']" position="after">
                <div class="d-flex" invisible="enroll != 'payment'">
                    <span class="me-auto"><label for="product_sale_revenues" class="mb0">Sales</label></span>
                    <field name="product_sale_revenues" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                    <field name="currency_id" invisible="True"/>
                </div>
            </xpath>
        </field>
    </record>

    <record id="slide_channel_view_form_add_inherit_sale_slides" model="ir.ui.view">
        <field name="name">slide.channel.view.form.add.inherit.sale.slides</field>
        <field name="model">slide.channel</field>
        <field name="inherit_id" ref="website_slides.slide_channel_view_form_add"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='allow_comment']" position="after">
                <field name="enroll" widget="radio" options="{'horizontal': true}" string="Enroll Policy"/>
                <field name="product_id" invisible="enroll != 'payment'" required="enroll == 'payment'"
                context="{'default_detailed_type': 'course', 'default_invoice_policy': 'order', 'default_purchase_ok': False, 'default_sale_ok': True, 'default_website_published': True}"/>
    		</xpath>
        </field>
    </record>
</data></odoo>

```

## File: views\snippets.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

<template id="snippet_options" inherit_id="website.snippet_options" name="Slides Snippet Options">
    <xpath expr="." position="inside">
        <div data-selector="main:has(.o_wslides_course_header)" data-page-options="true" groups="website.group_website_designer" data-no-check="true" string="Course Page">
            <we-checkbox string="Buy Now Button"
                         data-customize-website-views="website_sale_slides.course_option_buy_course_now"
                         data-no-preview="true"
                         data-reload="/"/>
        </div>
    </xpath>
</template>

</odoo>

```

## File: views\website_sale_templates.xml

```xml
<?xml version="1.0" ?>
<odoo><data>

<template id="website_sale_confirmation_slide" inherit_id="website_sale.confirmation">
    <xpath expr="//div[@id='oe_structure_website_sale_confirmation_2']" position="after">
        <t t-if="any(product.detailed_type == 'course' for product in order.order_line.product_id)" t-call="website_sale_slides.course_purchased_confirmation_message"/>
    </xpath>
</template>

<template id="course_purchased_confirmation_message">
    <div>
        <h4>
            <t t-if="order.state == 'sale'">You have gained access to the following course(s):</t>
            <t t-else="">Once your order is paid &amp; confirmed, you will gain access to:</t>
        </h4>
    </div>
    <div class="mt-2">
        <t t-foreach="order.order_line" t-as="line">
            <div t-foreach="line.product_id.channel_ids" t-as="course" class="row mx-0 my-2 border">
                <div class="col-5 d-flex justify-content-center my-auto">
                    <span t-if="course.image_1920" t-field="course.image_1920" t-options="{'widget': 'image', 'class': 'my-2'}"/>
                    <img t-else="" class="img img-fluid my-2" src="/website_slides/static/src/img/channel-training-default.jpg"/>
                </div>
                <t t-set="invitation_link" t-value="course_memberships[course].invitation_link if course in course_memberships else ''"/>
                <div class="col-7">
                    <a t-att-href="invitation_link"><h3 t-out="course.name" class="m-2"/></a>
                    <div t-out="course.description_short" class="fw-light o_wslides_desc_truncate_2 ms-2"/>
                    <div class="fw-light ms-2 mt-2">
                        <t t-out="course.total_time" t-options="{'widget': 'duration', 'unit': 'hour', 'round': 'minute'}"/>
                        <t t-if="course.total_slides">
                           <t t-if="course.total_time"> - </t><t t-out="course.total_slides"/> step(s)
                        </t>
                    </div>
                    <a role="button" class="btn btn-primary ms-2 my-2" t-attf-class="btn btn-primary ms-2 my-2 #{'disabled' if not invitation_link else ''}" t-att-href="invitation_link">
                        Start Learning
                    </a>
                </div>
            </div>
        </t>
    </div>
</template>

<template id="cart_summary_inherit_website_sale_slides"
          inherit_id="website_sale.checkout_layout"
          name="Course Cart right column">
    <xpath expr="//td[@name='website_sale_cart_summary_product_name']/h6" position="after">
        <div t-if="line.product_id.channel_ids"
             t-foreach="line.product_id.channel_ids.filtered(lambda course: course.enroll == 'payment')"
             t-as="course"
             t-esc="course.name"/>
    </xpath>
</template>

</data></odoo>

```

## File: views\website_slides_menu_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem name="Revenues"
        id="website_slides_menu_report_revenues"
        parent="website_slides.website_slides_menu_report"
        sequence="3"
        action="sale_report_action_slides"/>
</odoo>

```

## File: views\website_slides_templates.xml

```xml
<?xml version="1.0" ?>
<odoo><data>

<!-- LESSON -->
<template id="lesson_content_quiz" inherit_id="website_slides.lesson_content_quiz">
    <xpath expr="//div[hasclass('o_wslides_js_lesson_quiz')]" position="attributes">
        <attribute name="t-att-data-price">product_info['price'] if product_info else None</attribute>
        <attribute name="t-att-data-currency-name">product_info['currency'].name if product_info else None</attribute>
        <attribute name="t-att-data-currency-symbol">product_info['currency'].symbol if product_info else None</attribute>
        <attribute name="t-att-data-has-discounted-price">product_info['has_discounted_price'] if product_info else None</attribute>
        <attribute name="t-att-data-product-id">slide.channel_id.product_id.id if slide.channel_id.product_id else None</attribute>
        <attribute name="t-att-data-list-price">product_info['list_price'] if product_info else None</attribute>
    </xpath>
</template>

<template name="Buy Course To Download Resource" id="slide_aside_training_category_buy_course" inherit_id="website_slides.slide_aside_training_category">
    <xpath expr="//div[hasclass('o_wslides_js_course_join') and hasclass('o_wslides_no_access')]" position="inside">
        <li t-elif="aside_slide.channel_id.enroll == 'payment'" class="text-decoration-none small">
            <i class="fa fa-download me-1"/>
            <t t-call="website_sale_slides.course_buy_course_link">
                <t t-set="for_resources" t-value="1"/>
                <t t-set="slide" t-value="aside_slide"/>
            </t>
        </li>
    </xpath>
</template>

<template name="Buy Course To Access Resources or Interact Slide Detail" id="slide_content_detailed_buy_course" inherit_id="website_slides.slide_content_detailed">
    <xpath expr="//div[hasclass('o_wslides_js_course_join') and hasclass('o_wslides_no_access')] //div[@t-else='']" position="before">
        <span t-elif="slide.channel_id.enroll == 'payment'" class="text-muted me-auto border-start ps-3">
            <t t-call="website_sale_slides.course_buy_course_link">
                <t t-set="for_resources" t-value="1"/>
            </t>
        </span>
    </xpath>
    <xpath expr="//div[hasclass('o_wslides_js_course_join') and hasclass('o_wslides_no_access_comments')]" position="inside">
        <span t-elif="slide.channel_id.enroll == 'payment'">
            <t t-call="website_sale_slides.course_buy_course_link"/>
        </span>
    </xpath>
</template>

<!-- Tweak "preview" badge: display Free Preview if payment-based course -->
<template id="course_slides_list_slide"
    name="Slide template for a training channel (Sale)"
    inherit_id="website_slides.course_slides_list_slide">
    <xpath expr="//a[@t-if='channel.can_upload']/span/span" position="replace">
        <span t-if="channel.enroll == 'payment'">Free Preview</span>
        <span t-else="">Preview</span>
    </xpath>
    <xpath expr="//t[@t-elif='slide.is_preview and not channel.is_member']/span/span" position="replace">
        <span t-if="channel.enroll == 'payment'">Free Preview</span>
        <span t-else="">Preview</span>
    </xpath>
</template>

<!-- FULLSCREEN -->
<template id="slide_fullscreen" inherit_id="website_slides.slide_fullscreen">
    <xpath expr="//div[hasclass('o_wslides_fs_main')]" position="attributes">
        <attribute name="t-att-data-price">product_info['price'] if product_info else None</attribute>
        <attribute name="t-att-data-currency-name">product_info['currency'].name if product_info else None</attribute>
        <attribute name="t-att-data-currency-symbol">product_info['currency'].symbol if product_info else None</attribute>
        <attribute name="t-att-data-has-discounted-price">product_info['has_discounted_price'] if product_info else None</attribute>
        <attribute name="t-att-data-product-id">slide.channel_id.product_id.id if product_info else None</attribute>
        <attribute name="t-att-data-list-price">product_info['list_price'] if product_info else None</attribute>
    </xpath>
</template>

<template name="Buy Course To Download Resource Fullscreen" id="slide_fullscreen_sidebar_category_buy_course" inherit_id="website_slides.slide_fullscreen_sidebar_category">
    <xpath expr="//div[hasclass('o_wslides_js_course_join') and hasclass('o_wslides_no_access')]" position="inside">
        <li t-elif="slide.channel_id.enroll == 'payment'" class="o_wslides_fs_slide_link mb-1">
            <i class="fa fa-download me-1"/>
            <t t-call="website_sale_slides.course_buy_course_link">
                <t t-set="for_resources" t-value="1"/>
            </t>
        </li>
    </xpath>
</template>

<!-- COURSE -->
<template name="Course Main" id="course_main" inherit_id="website_slides.course_main">
    <xpath expr="//div[@id='home']" position="before">
        <div t-if="channel.enroll == 'payment' and not channel.product_id.is_published"
            class="alert alert-info" role="alert" groups="website_slides.group_website_slides_officer">
            This course cannot be bought because its linked product
            <a t-attf-href="/web#id=#{channel.product_id.product_tmpl_id.id}&amp;view_type=form&amp;model=#{channel.product_id.product_tmpl_id._name}&amp;action=website_sale.product_template_action_website"
                class="alert-link" t-out="channel.product_id.name"/>
            is not published.
        </div>
    </xpath>
</template>

<template name="Course Sidebar (infos, CTA)" id="course_join" inherit_id="website_slides.course_join">
    <!-- Channel main template: override button to join channel -->
    <xpath expr="//div[hasclass('o_wslides_js_course_join')]" position="inside">
        <t t-if="(not channel.is_member or channel.can_publish) and channel.enroll == 'payment'">
            <t t-call="website_sale_slides.course_buy_course_button" />
        </t>
    </xpath>
</template>

<!-- TOOLS -->
<template name="Buy Course Link" id="course_buy_course_link">
    <a class="post_link" t-att-href="'/shop/cart/update?product_id=%s' % slide.channel_id.product_id.id">
        Join this Course</a><t t-if="for_resources"> to access resources</t>
</template>

<template name="Buy Course Button" id="course_buy_course_button">
    <t t-if="product_info and channel.product_id.website_published and not channel.is_member">
        <div t-attf-class="text-center d-flex align-items-center text-center pb-1 #{'justify-content-between' if product_info['has_discounted_price'] else 'justify-content-around'}">
            <div class="css_editable_mode_hidden">
                <!-- real price -->
                <div t-attf-class="oe_price fw-bold text-nowrap my-2 #{'h4' if len(str(product_info['price'])) > 10 else 'h2'}"
                     t-esc="product_info['price']"
                     t-options="{'widget': 'monetary', 'display_currency': product_info['currency']}"/>
                <span itemprop="price" style="display:none;" t-esc="product_info['price']"/>
                <span itemprop="priceCurrency" style="display:none;" t-esc="product_info['currency'].name"/>
                <!-- original discounted price, if any -->
                <del t-att-class="'text-600 text-nowrap oe_default_price %s' % ('' if product_info['has_discounted_price'] else 'd-none')"
                     t-esc="product_info['list_price']"
                     t-options="{'widget': 'monetary', 'display_currency': product_info['currency']}"/>
            </div>
            <div class="css_non_editable_mode_hidden decimal_precision oe_price fw-bold text-nowrap h2 my-2"
                 t-att-data-precision="str(product_info['currency'].decimal_places)">
                <span t-field="channel.product_id.list_price" t-options="{'widget': 'monetary', 'display_currency': product_info['currency']}"/>
            </div>
        </div>
        <t t-if="not invite_preview and channel.prerequisite_channel_ids and not channel.prerequisite_user_has_completed">
            <small t-if="len(channel.prerequisite_channel_ids) == 1" class="text-center mb-2">
                Prerequisite:
                <a t-attf-href="/slides/{{channel.prerequisite_channel_ids[0].id}}"
                   t-out="channel.prerequisite_channel_ids[0].name"/>
            </small>
            <small t-else="" class="text-center mb-2">
                There are some
                <a href="#" class="o_wslides_js_prerequisite_course"
                   t-att-data-channels="json.dumps([
                      {'course_id': course.id, 'course_name': course.name}
                      for course in channel.prerequisite_channel_ids]
                   )">
                    prerequisite courses.
                </a>
            </small>
        </t>
        <t t-if="invite_preview">
            <a type="button" class="btn btn-primary text-uppercase ms-2"
                t-att-aria-label="'Sign up' if is_partner_without_user else 'Log in'"
                t-attf-href="/slides/#{channel.id}/identify?#{keep_query('invite_partner_id', 'invite_hash')}">
                <t t-if="is_partner_without_user">Sign up</t>
                <t t-else="">Log in</t>
            </a>
        </t>
        <div t-else="" class="oe_website_sale">
            <div class="add_to_cart_button">
                <form action="/shop/cart/update" method="POST">
                    <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" />
                    <input type="hidden" class="product_id" name="product_id" t-att-value="channel.product_id.id"/>
                    <a id="add_to_cart" role="button" href="#"
                       class="btn btn-primary d-block js_check_product o_js_add_to_cart a-submit"
                       data-animation-selector=".o_wslides_course_pict">
                        <i class="fa fa-shopping-cart"></i> Add to Cart
                    </a>
                    <div id="product_option_block"/>
                </form>
            </div>
        </div>
    </t>
    <t t-elif="not channel.is_member">
        <div class="alert my-0 bg-200 text-center">
            Course Unavailable
        </div>
    </t>
</template>

<template name="Display 'Buy Now'" id="course_option_buy_course_now" inherit_id="website_sale_slides.course_buy_course_button" active="False">
    <xpath expr="//div[hasclass('add_to_cart_button')]" position="before">
        <div class="mb-1">
            <a role="button" class="post_link btn btn-primary d-block" t-attf-href="/shop/cart/update?product_id={{channel.product_id.id}}&amp;express=1">
                <i class="fa fa-bolt"></i> Buy Now
            </a>
        </div>
    </xpath>
    <xpath expr="//div[hasclass('add_to_cart_button')]//a[@id='add_to_cart']" position="attributes">
        <attribute name="class">btn btn-outline-primary d-block js_check_product o_js_add_to_cart a-submit</attribute>
    </xpath>
</template>

</data></odoo>

```

