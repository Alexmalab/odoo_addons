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
        'report/sale_report_views.xml',
        'views/assets.xml',
        'views/website_slides_menu_views.xml',
        'views/slide_channel_views.xml',
        'views/website_slides_templates.xml',
    ],
    'demo': [
        'data/product_demo.xml',
        'data/slide_demo.xml',
        'data/sale_order_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: controllers\slides.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.website_slides.controllers.main import WebsiteSlides
from odoo.http import request


class WebsiteSaleSlides(WebsiteSlides):

    def _prepare_additional_channel_values(self, values, **kwargs):
        values = super(WebsiteSaleSlides, self)._prepare_additional_channel_values(values, **kwargs)
        channel = values['channel']
        if channel.enroll == 'payment' and channel.product_id:
            pricelist = request.website.get_current_pricelist()
            values['product_info'] = channel.product_id.product_tmpl_id._get_combination_info(product_id=channel.product_id.id, pricelist=pricelist)
            values['product_info']['currency_id'] = request.website.currency_id
        return values

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import slides

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
        <field name="type">service</field>
        <field name="is_published" eval="True"/>
        <field name="image_1920" type="base64" file="website_sale_slides/static/img/channel_demo_tree_1.jpg"/>
    </record>

    <!-- CHANNEL 6: DIY Furniture -->
    <!-- ================================================== -->
    <record id="product_course_channel_6" model="product.product">
        <field name="name">DIY Furniture Course</field>
        <field name="standard_price">100.0</field>
        <field name="list_price">100.0</field>
        <field name="type">service</field>
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
        <field name="pricelist_id" ref="product.list0"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=30)"/>
        <field name="team_id" ref="sales_team.salesteam_website_sales"/>
        <field name="website_id" eval="1"/>
        <field name="state">sale</field>
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
        <field name="pricelist_id" ref="product.list0"/>
        <field name="date_order" eval="DateTime.now() - relativedelta(days=100)"/>
        <field name="team_id" ref="sales_team.salesteam_website_sales"/>
        <field name="website_id" eval="1"/>
        <field name="state">sale</field>
    </record>
    <record id="sale_order_course_2_line_1" model="sale.order.line">
        <field name="order_id" ref="sale_order_course_2"/>
        <field name="name" model="sale.order.line" eval="obj().env.ref('website_sale_slides.product_course_channel_6').get_product_multiline_description_sale()"/>
        <field name="product_id" ref="website_sale_slides.product_course_channel_6"/>
        <field name="product_uom_qty">1</field>
        <field name="product_uom" ref="uom.product_uom_unit"/>
        <field name="price_unit">100.0</field>
    </record>

    <!-- Confirm sales -->
    <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_course_1')]]"/>
    <function model="sale.order" name="action_confirm" eval="[[ref('sale_order_course_2')]]"/>
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

from odoo import fields, models


class Product(models.Model):
    _inherit = "product.product"

    channel_ids = fields.One2many('slide.channel', 'product_id', string='Courses')

```

## File: models\sale_order.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, api


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
            [('product_id', 'in', products.ids)]
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

```

## File: models\slide_channel.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class Channel(models.Model):
    _inherit = 'slide.channel'

    enroll = fields.Selection(selection_add=[('payment', 'On payment')])
    product_id = fields.Many2one('product.product', 'Product', index=True)
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
        rg_data = dict(
            (item['product_id'][0], item['price_total'])
            for item in self.env['sale.report'].read_group(domain, ['product_id', 'price_total'], ['product_id'])
        )
        for channel in self:
            channel.product_sale_revenues = rg_data.get(channel.product_id.id, 0)

    @api.model
    def create(self, vals):
        channel = super(Channel, self).create(vals)
        if channel.enroll == 'payment':
            channel._synchronize_product_publish()
        return channel

    def write(self, vals):
        res = super(Channel, self).write(vals)
        if 'is_published' in vals:
            self.filtered(lambda channel: channel.enroll == 'payment')._synchronize_product_publish()
        return res

    def _synchronize_product_publish(self):
        self.filtered(lambda channel: channel.is_published and not channel.product_id.is_published).sudo().product_id.write({'is_published': True})
        self.filtered(lambda channel: not channel.is_published and channel.product_id.is_published).sudo().product_id.write({'is_published': False})

    def action_view_sales(self):
        action = self.env.ref('website_sale_slides.sale_report_action_slides').read()[0]
        action['domain'] = [('product_id', 'in', self.product_id.ids)]
        return action

    def _filter_add_members(self, target_partners, **member_values):
        """ Overridden to add 'payment' channels to the filtered channels. People
        that can write on payment-based channels can add members. """
        result = super(Channel, self)._filter_add_members(target_partners, **member_values)
        on_payment = self.filtered(lambda channel: channel.enroll == 'payment')
        if on_payment:
            try:
                on_payment.check_access_rights('write')
                on_payment.check_access_rule('write')
            except:
                pass
            else:
                result |= on_payment
        return result

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-

from . import product_product
from . import slide_channel
from . import sale_order

```

## File: report\sale_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sale_report_view_graph_slides" model="ir.ui.view">
         <field name="name">sale.report.view.graph.slides</field>
         <field name="model">sale.report</field>
         <field name="arch" type="xml">
             <graph string="eLearning Sales Analysis" type="line">
                 <field name="date" type="row" interval="day"/>
                 <field name="price_subtotal" type="measure"/>
             </graph>
         </field>
    </record>

	<record id="sale_report_action_slides" model="ir.actions.act_window">
        <field name="name">eLearning Revenues</field>
        <field name="res_model">sale.report</field>
        <field name="type">ir.actions.act_window</field>
        <field name="view_mode">graph,pivot</field>
        <field name="domain">[("product_id.channel_ids", "!=", False)]</field>
        <field name="context">{'group_by': ['date', 'product_id']}</field>
        <field name="view_id" ref="sale_report_view_graph_slides"/>
    </record>
</odoo>

```

## File: static\src\js\slides_course_quiz.js

```javascript
odoo.define('website_sale_slides.quiz', function (require) {
"use strict";

var sAnimations = require('website.content.snippets.animation');
var Quiz = require('website_slides.quiz');

sAnimations.registry.websiteSlidesQuizNoFullscreen.include({
    _extractChannelData: function (slideData){
        return _.extend({}, this._super.apply(this, arguments), {
            productId: slideData.productId,
            enroll: slideData.enroll,
            currencyName: slideData.currencyName,
            currencySymbol: slideData.currencySymbol,
            price: slideData.price,
            hasDiscountedPrice: slideData.hasDiscountedPrice
        });
    }
});

Quiz.include({
    xmlDependencies: (Quiz.prototype.xmlDependencies || []).concat(
        ["/website_sale_slides/static/src/xml/website_sale_slides_quiz.xml"]
    )
});
});

```

## File: static\src\js\slides_course_unsubscribe.js

```javascript
odoo.define('website_sale_slides.unsubscribe_modal', function (require) {
"use strict";

var SlidesUnsubscribe = require('website_slides.unsubscribe_modal');

SlidesUnsubscribe.websiteSlidesUnsubscribe.include({
    xmlDependencies: (SlidesUnsubscribe.websiteSlidesUnsubscribe.prototype.xmlDependencies || []).concat(
        ["/website_sale_slides/static/src/xml/website_slides_unsubscribe.xml"]
    ),
});

});

```

## File: static\src\xml\website_sale_slides_quiz.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-extend="slide.slide.quiz.validation">
        <t t-jquery="#validation" t-operation="append">
            <div t-if="widget.readonly &amp;&amp; widget.publicUser &amp;&amp; widget.channel.channelEnroll == 'payment'" class="alert alert-info d-flex align-items-center justify-content-between">
                <div>
                    <b class="h5 mb-0">Sign in and buy the course to join it and take the quiz!</b>
                    <span class="my-0 h4" style="line-height: 1">
                        <span title="Succeed and gain karma" aria-label="Succeed and gain karma" class="badge badge-pill badge-warning text-white font-weight-bold ml-3 px-2 py-1">
                            + <t t-esc="widget.quiz.quizKarmaGain"/> XP
                        </span>
                    </span>
                </div>
                <div>
                    <a t-att-href="'/web/login?redirect=' + widget.redirectURL" class="btn btn-primary font-weight-bold text-uppercase">Sign in</a>
                    <span t-if="widget.channel.signupAllowed" class="d-block mt-2">Don't have an account ? <a class="font-weight-bold" t-att-href="'/web/signup?redirect=' + widget.url">Sign Up !</a></span>
                </div>
            </div>
            <div t-if="widget.readonly &amp;&amp; !widget.publicUser &amp;&amp; widget.channel.channelEnroll == 'payment'" class="card col-md-3">
                <div class="card-body d-flex flex-column align-items-center">
                    <a role="button"
                        class="btn btn-primary btn-block o_wslides_js_course_join_link d-block mb-2"
                        title="Start Course" aria-label="Start Course Channel"
                        t-attf-href="/shop/cart/update?product_id=#{widget.channel.productId}&amp;express=1"
                        t-att-data-channel-id="widget.slide.channelId">
                        <span class="text-white text-uppercase font-weight-bold">
                            Buy the course
                        </span>
                    </a>
                    <div class="text-center">
                        <del t-if="widget.channel.hasDiscountedPrice"
                            class="text-600 text-nowrap oe_default_price d-inline"
                            t-esc="widget.channel.listPrice"/>
                        <span class="oe_price font-weight-bold text-nowrap h3 my-2"
                            t-esc="widget.channel.price"/>
                        <span t-if="widget.channel.currencySymbol" class="text-nowrap font-weight-bold h3 my-2" itemprop="priceCurrency" t-esc="widget.channel.currencySymbol"/>
                        <span t-else="" class="text-nowrap ml-1" itemprop="priceCurrency" t-esc="widget.channel.currencyName"/>
                    </div>
                </div>
            </div>
        </t>
    </t>
</templates>

```

## File: static\src\xml\website_slides_unsubscribe.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">

    <t t-extend="slides.course.unsubscribe.modal.leave">
        <t t-jquery="p" t-operation="after">
            <t t-if="widget.enroll === 'payment'">
                <p class="alert alert-warning">
                    <i class="fa fa-exclamation-triangle fa-3x float-left mr-3"></i>
                    This course is paid.<br/>
                    Leaving the course and re-enrolling afterwards means that you'll be charged again.
                </p>
            </t>
        </t>
    </t>

</templates>

```

## File: views\assets.xml

```xml
<?xml version="1.0" ?>
<odoo>
    <data>
        <template id="assets_frontend" inherit_id="website.assets_frontend" name="Buy course on quiz">
            <xpath expr="//script[last()]" position="after">
                <script type="text/javascript" src="/website_sale_slides/static/src/js/slides_course_quiz.js"/>
                <script type="text/javascript" src="/website_sale_slides/static/src/js/slides_course_unsubscribe.js"/>
            </xpath>
        </template>
    </data>
</odoo>

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
                    attrs="{'invisible': [('enroll', '!=', 'payment')], 'required': [('enroll', '=', 'payment')]}" />
            </xpath>
            <xpath expr="//button[@name='action_redirect_to_members']" position="after">
                <button name="action_view_sales"
                    type="object"
                    icon="fa-signal"
                    class="oe_stat_button"
                    attrs="{'invisible': [('enroll', '!=', 'payment')]}"
                    groups="sales_team.group_sale_salesman">
                    <field name="product_sale_revenues" string="Sales" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>

    <record id="slide_channel_view_kanban" model="ir.ui.view">
        <field name="name">slide.channel.view.kanban.inherit.sale</field>
        <field name="model">slide.channel</field>
        <field name="inherit_id" ref="website_slides.slide_channel_view_kanban"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='website_published']" position="before">
                <field name="enroll"/>
            </xpath>
            <xpath expr="//div[@name='info_total_time']" position="after">
                <div class="d-flex" attrs="{'invisible': [('enroll', '!=', 'payment')]}">
                    <span class="mr-auto"><label for="product_sale_revenues" class="mb0">Sales</label></span>
                    <field name="product_sale_revenues" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                    <field name="currency_id" attrs="{'invisible': True}"/>
                </div>
            </xpath>
            <xpath expr="//a[@name='action_channel_invite']" position="attributes">
                <attribute name="attrs">{'invisible': [('enroll', '=', 'payment')]}</attribute>
            </xpath>
        </field>
    </record>
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

<template id="lesson_content_quiz" inherit_id="website_slides.lesson_content_quiz">
    <xpath expr="//div[hasclass('o_wslides_js_lesson_quiz')]" position="attributes">
        <attribute name="t-att-data-price">product_info['price'] if product_info else None</attribute>
        <attribute name="t-att-data-currency-name">product_info['currency_id'].name if product_info else None</attribute>
        <attribute name="t-att-data-currency-symbol">product_info['currency_id'].symbol if product_info else None</attribute>
        <attribute name="t-att-data-has-discounted-price">product_info['has_discounted_price'] if product_info else None</attribute>
        <attribute name="t-att-data-product-id">slide.channel_id.product_id.id if slide.channel_id.product_id else None</attribute>
        <attribute name="t-att-data-list-price">product_info['list_price'] if product_info else None</attribute>
    </xpath>
</template>

<template id="slide_fullscreen" inherit_id="website_slides.slide_fullscreen">
    <xpath expr="//div[hasclass('o_wslides_fs_main')]" position="attributes">
        <attribute name="t-att-data-price">product_info['price'] if product_info else None</attribute>
        <attribute name="t-att-data-currency-name">product_info['currency_id'].name if product_info else None</attribute>
        <attribute name="t-att-data-currency-symbol">product_info['currency_id'].symbol if product_info else None</attribute>
        <attribute name="t-att-data-has-discounted-price">product_info['has_discounted_price'] if product_info else None</attribute>
        <attribute name="t-att-data-product-id">slide.channel_id.product_id.id if product_info else None</attribute>
        <attribute name="t-att-data-list-price">product_info['list_price'] if product_info else None</attribute>
    </xpath>
</template>

<template name="Join Channel: Product Button" id='course_sidebar' inherit_id="website_slides.course_sidebar">
    <!-- Channel main template: override button to join channel -->
    <xpath expr="//div[hasclass('o_wslides_js_course_join')]" position="inside">
        <t t-if="(not channel.is_member or channel.can_publish) and channel.enroll == 'payment'">
            <t t-if="channel.product_id.website_published">
                <div t-attf-class="text-center d-flex align-items-center text-center pb-1 #{'justify-content-between' if product_info['has_discounted_price'] else 'justify-content-around'}">
                    <div class="css_editable_mode_hidden">
                        <!-- real price -->
                        <div class="oe_price font-weight-bold text-nowrap h2 my-2"
                             t-esc="product_info['price']"
                             t-options="{'widget': 'monetary', 'display_currency': product_info['currency_id']}"/>
                        <span itemprop="price" style="display:none;" t-esc="product_info['price']"/>
                        <span itemprop="priceCurrency" style="display:none;" t-esc="product_info['currency_id'].name"/>
                        <!-- original discounted price, if any -->
                        <del t-att-class="'text-600 text-nowrap oe_default_price %s' % ('' if product_info['has_discounted_price'] else 'd-none')"
                             t-esc="product_info['list_price']"
                             t-options="{'widget': 'monetary', 'display_currency': product_info['currency_id']}"/>
                    </div>
                    <div class="css_non_editable_mode_hidden decimal_precision oe_price font-weight-bold text-nowrap h2 my-2"
                         t-att-data-precision="str(product_info['currency_id'].decimal_places)">
                        <span t-field="channel.product_id.list_price" t-options="{'widget': 'monetary', 'display_currency': product_info['currency_id']}"/>
                    </div>
                </div>
                <div class="oe_website_sale">
                    <div class="add_to_cart_button">
                        <form action="/shop/cart/update" method="POST">
                            <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()" />
                            <input type="hidden" class="product_id" name="product_id" t-att-value="channel.product_id.id"/>
                            <input type="hidden" class="product_template_id" name="product_template_id" t-att-value="channel.product_id.id"/>
                            <a id="add_to_cart" role="button" class="btn btn-primary btn-block js_check_product o_js_add_to_cart a-submit" href="#">
                                <i class="fa fa-shopping-cart"></i> Add to Cart
                            </a>
                            <div id="product_option_block"/>
                        </form>
                    </div>
                    <div class="buy_now_button"/>
                </div>
            </t>
            <t t-else="">
                <div class="alert my-0 bg-200 text-center">
                    Course Not Buyable
                </div>
            </t>
        </t>
    </xpath>
</template>

<template name="Buy Now Button" id="course_sidebar_buy_now_course" inherit_id="website_sale_slides.course_sidebar" active="False" customize_show="True">
    <xpath expr="//div[hasclass('buy_now_button')]" position="inside">
        <div style="margin-top:5px;">
            <a role="button" class="btn btn-outline-primary btn-block" t-att-href="'/shop/cart/update?product_id=%s&amp;express=1' % channel.product_id.id">
                <i class="fa fa-bolt"></i> Buy Now
            </a>
        </div>
    </xpath>
</template>
</data></odoo>

```

