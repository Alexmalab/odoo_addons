# Odoo Module: lunch

Category: Human Resources/Lunch

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
from . import models
from . import report

```

## File: __manifest__.py

```python
{
    'name': 'Lunch',
    'sequence': 300,
    'version': '1.0',
    'depends': ['mail'],
    'category': 'Human Resources/Lunch',
    'summary': 'Handle lunch orders of your employees',
    'description': """
The base module to manage lunch.
================================

Many companies order sandwiches, pizzas and other, from usual vendors, for their employees to offer them more facilities.

However lunches management within the company requires proper administration especially when the number of employees or vendors is important.

The “Lunch Order” module has been developed to make this management easier but also to offer employees more tools and usability.

In addition to a full meal and vendor management, this module offers the possibility to display warning and provides quick order selection based on employee’s preferences.

If you want to save your employees' time and avoid them to always have coins in their pockets, this module is essential.
    """,
    'data': [
        'security/lunch_security.xml',
        'security/ir.model.access.csv',
        'report/lunch_cashmove_report_views.xml',
        'views/lunch_templates.xml',
        'views/lunch_alert_views.xml',
        'views/lunch_cashmove_views.xml',
        'views/lunch_location_views.xml',
        'views/lunch_orders_views.xml',
        'views/lunch_product_views.xml',
        'views/lunch_supplier_views.xml',
        'views/res_config_settings.xml',
        'views/lunch_views.xml',
        'data/mail_template_data.xml',
        'data/lunch_data.xml',
    ],
    'demo': ['data/lunch_demo.xml'],
    'installable': True,
    'application': True,
    'assets': {
        'web.assets_backend': [
            'lunch/static/src/components/*',
            'lunch/static/src/mixins/*.js',
            'lunch/static/src/views/*',
            'lunch/static/src/scss/lunch_view.scss',
            'lunch/static/src/scss/lunch_kanban.scss',
        ],
        'web.assets_tests': [
            'lunch/static/tests/tours/*.js',
        ],
        'web.assets_unit_tests': [
            'lunch/static/tests/**/*.test.js',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: controllers\main.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, http, fields
from odoo.exceptions import AccessError
from odoo.http import request
from odoo.osv import expression
from odoo.tools import float_round, float_repr


class LunchController(http.Controller):
    @http.route('/lunch/infos', type='json', auth='user')
    def infos(self, user_id=None, context=None):
        if context:
            request.update_context(**context)
        self._check_user_impersonification(user_id)
        user = request.env['res.users'].browse(user_id) if user_id else request.env.user

        infos = self._make_infos(user, order=False)

        lines = self._get_current_lines(user)
        if lines:
            translated_states = dict(request.env['lunch.order']._fields['state']._description_selection(request.env))
            lines = [{
                'id': line.id,
                'product': (line.product_id.id, line.product_id.name, float_repr(
                    float_round(line.product_id.price, 2) * line.quantity, 2),
                float_round(line.product_id.price, 2)),
                'toppings': [(topping.name, float_repr(float_round(topping.price, 2) * line.quantity, 2),
                float_round(topping.price, 2))
                    for topping in line.topping_ids_1 | line.topping_ids_2 | line.topping_ids_3],
                'quantity': line.quantity,
                'price': line.price,
                'raw_state': line.state,
                'state': translated_states[line.state],
                'date': line.date,
                'location': line.lunch_location_id.name,
                'note': line.note
                } for line in lines.sorted('date')]
            total = float_round(sum(line['price'] for line in lines), 2)
            paid_subtotal = float_round(sum(line['price'] for line in lines if line['raw_state'] != 'new'), 2)
            unpaid_subtotal = total - paid_subtotal
            infos.update({
                'total': float_repr(total, 2),
                'paid_subtotal': float_repr(paid_subtotal, 2),
                'unpaid_subtotal': float_repr(unpaid_subtotal, 2),
                'raw_state': self._get_state(lines),
                'lines': lines,
            })
        return infos

    @http.route('/lunch/trash', type='json', auth='user')
    def trash(self, user_id=None, context=None):
        if context:
            request.update_context(**context)
        self._check_user_impersonification(user_id)
        user = request.env['res.users'].browse(user_id) if user_id else request.env.user

        lines = self._get_current_lines(user)
        lines = lines.filtered_domain([('state', 'not in', ['sent', 'confirmed'])])
        lines.action_cancel()
        lines.unlink()

    @http.route('/lunch/pay', type='json', auth='user')
    def pay(self, user_id=None, context=None):
        if context:
            request.update_context(**context)
        self._check_user_impersonification(user_id)
        user = request.env['res.users'].browse(user_id) if user_id else request.env.user

        lines = self._get_current_lines(user)
        if lines:
            lines = lines.filtered(lambda line: line.state == 'new')
            lines.action_order()
            return True

        return False

    @http.route('/lunch/payment_message', type='json', auth='user')
    def payment_message(self):
        return {'message': request.env['ir.qweb']._render('lunch.lunch_payment_dialog', {})}

    @http.route('/lunch/user_location_set', type='json', auth='user')
    def set_user_location(self, location_id=None, user_id=None, context=None):
        if context:
            request.update_context(**context)
        self._check_user_impersonification(user_id)
        user = request.env['res.users'].browse(user_id) if user_id else request.env.user

        user.sudo().last_lunch_location_id = request.env['lunch.location'].browse(location_id)
        return True

    @http.route('/lunch/user_location_get', type='json', auth='user')
    def get_user_location(self, user_id=None, context=None):
        if context:
            request.update_context(**context)
        self._check_user_impersonification(user_id)
        user = request.env['res.users'].browse(user_id) if user_id else request.env.user

        company_ids = request.env.context.get('allowed_company_ids', request.env.company.ids)
        user_location = user.last_lunch_location_id
        has_multi_company_access = not user_location.company_id or user_location.company_id.id in company_ids

        if not user_location or not has_multi_company_access:
            return request.env['lunch.location'].search([('company_id', 'in', [False] + company_ids)], limit=1).id
        return user_location.id

    def _make_infos(self, user, **kwargs):
        res = dict(kwargs)

        is_manager = request.env.user.has_group('lunch.group_lunch_manager')

        currency = user.company_id.currency_id

        res.update({
            'username': user.sudo().name,
            'userimage': '/web/image?model=res.users&id=%s&field=avatar_128' % user.id,
            'wallet': request.env['lunch.cashmove'].get_wallet_balance(user, False),
            'is_manager': is_manager,
            'group_portal_id': request.env.ref('base.group_portal').id,
            'locations': request.env['lunch.location'].search_read([], ['name']),
            'currency': {'symbol': currency.symbol, 'position': currency.position},
        })

        user_location = user.last_lunch_location_id
        has_multi_company_access = not user_location.company_id or user_location.company_id.id in request.env.context.get('allowed_company_ids', request.env.company.ids)

        if not user_location or not has_multi_company_access:
            user.last_lunch_location_id = user_location = request.env['lunch.location'].search([], limit=1) or user_location

        alert_domain = expression.AND([
            [('available_today', '=', True)],
            [('location_ids', 'in', user_location.id)],
            [('mode', '=', 'alert')],
        ])

        res.update({
            'user_location': (user_location.id, user_location.name),
            'alerts': request.env['lunch.alert'].search_read(alert_domain, ['message']),
        })

        return res

    def _check_user_impersonification(self, user_id=None):
        if (user_id and request.env.uid != user_id and not request.env.user.has_group('lunch.group_lunch_manager')):
            raise AccessError(_('You are trying to impersonate another user, but this can only be done by a lunch manager'))

    def _get_current_lines(self, user):
        return request.env['lunch.order'].search(
            [('user_id', '=', user.id), ('date', '>=', fields.Date.context_today(user)), ('state', '!=', 'cancelled')]
            )

    def _get_state(self, lines):
        """
            This method returns the lowest state of the list of lines

            eg: [confirmed, confirmed, new] will return ('new', 'To Order')
        """
        states_to_int = {'new': 0, 'ordered': 1, 'sent': 2, 'confirmed': 3, 'cancelled': 4}
        int_to_states = ['new', 'ordered', 'sent', 'confirmed', 'cancelled']

        return int_to_states[min(states_to_int[line['raw_state']] for line in lines)]

```

## File: controllers\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import main

```

## File: data\lunch_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="1">
    <record id="lunch_location_main" model="lunch.location" forcecreate="0">
        <field name="name">HQ Office</field>
    </record>

    <record model="lunch.product.category" id="categ_sandwich" forcecreate="0">
        <field name="name">Sandwich</field>
    </record>

    <record id="categ_pizza" model="lunch.product.category" forcecreate="0">
        <field name="name">Pizza</field>
        <field name="image_1920" type="base64" file="lunch/static/img/pizza.png"/>
    </record>

    <record id="categ_burger" model="lunch.product.category" forcecreate="0">
        <field name="name">Burger</field>
        <field name="image_1920" type="base64" file="lunch/static/img/burger.png"/>
    </record>

    <record id="categ_drinks" model="lunch.product.category" forcecreate="0">
        <field name="name">Drinks</field>
        <field name="image_1920" type="base64" file="lunch/static/img/drink.png"/>
    </record>

    <record id="partner_hungry_dog" model="res.partner" forcecreate="0">
        <field name="name">Lunch Supplier</field>
    </record>

    <record id="supplier_hungry_dog" model="lunch.supplier" forcecreate="0">
        <field name="partner_id" ref="partner_hungry_dog"/>
        <field name="available_location_ids" eval="[
            (6, 0, [ref('lunch_location_main')]),
            ]"/>
    </record>
</data>
<data>
    <record id="lunch_order_action_confirm" model="ir.actions.server">
        <field name="name">Lunch: Receive meals</field>
        <field name="model_id" ref="model_lunch_order"/>
        <field name="binding_model_id" ref="model_lunch_order"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">records.action_confirm()</field>
    </record>

    <record id="lunch_order_action_cancel" model="ir.actions.server">
        <field name="name">Lunch: Cancel meals</field>
        <field name="model_id" ref="model_lunch_order"/>
        <field name="binding_model_id" ref="model_lunch_order"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">records.action_cancel()</field>
    </record>

    <record id="lunch_order_action_notify" model="ir.actions.server">
        <field name="name">Lunch: Send notifications</field>
        <field name="model_id" ref="model_lunch_order"/>
        <field name="binding_model_id" ref="model_lunch_order"/>
        <field name="binding_view_types">list</field>
        <field name="state">code</field>
        <field name="code">records.action_notify()</field>
    </record>
</data>

</odoo>

```

## File: data\lunch_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <record id="lunch_location_main" model="lunch.location">
            <field name="address">87323 Francis Corner Oscarhaven, OK 12782</field>
        </record>

        <record id="partner_hungry_dog" model="res.partner">
            <field name="name">Hungry Dog</field>
            <field name="city">Russeltown</field>
            <field name="country_id" ref="base.us"/>
            <field name="street">975 Bullock Orchard</field>
            <field name="zip">02155</field>
            <field name="email">hungry_dog@yourcompany.example.com</field>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record id="product_bacon_0" model="lunch.product">
            <field name="name">Bacon</field>
            <field name="category_id" ref="categ_burger"/>
            <field name="price">7.2</field>
            <field name="supplier_id" ref="supplier_hungry_dog"/>
            <field name="description">Beef, Bacon, Salad, Cheddar, Fried Onion, BBQ Sauce</field>
            <field name="image_1920" type="base64" file="lunch/static/img/bacon_burger.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record id="product_cheese_burger_0" model="lunch.product">
            <field name="name">Cheese Burger</field>
            <field name="category_id" ref="categ_burger"/>
            <field name="price">6.8</field>
            <field name="supplier_id" ref="supplier_hungry_dog"/>
            <field name="description">Beef, Cheddar, Salad, Fried Onions, BBQ Sauce</field>
            <field name="image_1920" type="base64" file="lunch/static/img/cheeseburger.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record id="product_club_0" model="lunch.product">
            <field name="name">Club</field>
            <field name="category_id" ref="categ_sandwich"/>
            <field name="price">3.4</field>
            <field name="supplier_id" ref="supplier_hungry_dog"/>
            <field name="description">Ham, Cheese, Vegetables</field>
            <field name="image_1920" type="base64" file="lunch/static/img/club.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record id="product_coke_0" model="lunch.product">
            <field name="name">Coca Cola</field>
            <field name="category_id" ref="categ_drinks"/>
            <field name="price">2.9</field>
            <field name="description"></field>
            <field name="supplier_id" ref="supplier_hungry_dog"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record id="product_pizza_0" model="lunch.product">
            <field name="name">Pizza Margherita</field>
            <field name="category_id" ref="categ_pizza"/>
            <field name="price">6.90</field>
            <field name="supplier_id" ref="supplier_hungry_dog"/>
            <field name="description">Tomatoes, Mozzarella</field>
            <field name="company_id" ref="base.main_company"/>
        </record>


        <record id="location_office_1" model="lunch.location">
            <field name="name">Office 1</field>
        </record>

        <record id="location_office_2" model="lunch.location">
            <field name="name">Office 2</field>
        </record>

        <record id="location_office_3" model="lunch.location">
            <field name="name">Office 3</field>
        </record>

        <record id="alert_office_3" model="lunch.alert">
            <field name="name">Alert for Office 3</field>
            <field name="message">Please order</field>
            <field name="location_ids" eval="[(4, ref('location_office_3'))]" />
            <field name="mode">chat</field>
        </record>

        <record id="base.user_admin" model="res.users">
            <field name="last_lunch_location_id" ref="location_office_2"/>
        </record>

        <record id="base.user_demo" model="res.users">
            <field name="last_lunch_location_id" ref="location_office_3"/>
            <field name="groups_id" eval="[(3, ref('lunch.group_lunch_manager'))]"/>
        </record>

        <record model="lunch.product.category" id="categ_pasta">
            <field name="name">Pasta</field>
        </record>

        <record model="lunch.product.category" id="categ_sushi">
            <field name="name">Sushi</field>
        </record>

        <record model="lunch.product.category" id="categ_temaki">
            <field name="name">Temaki</field>
        </record>

        <record model="lunch.product.category" id="categ_chirashi">
            <field name="name">Chirashi</field>
        </record>

        <record id="partner_coin_gourmand" model="res.partner">
            <field name="name">Coin gourmand</field>
            <field name="city">Tirana</field>
            <field name="country_id" ref="base.al"/>
            <field name="street">Rr. e Durrësit, Pall. M.C. Inerte</field>
            <field name="street2">Kati.1, Laprakë, Tirana, Shqipëri</field>
            <field name="email">coin.gourmand@yourcompany.example.com</field>
            <field name="phone">+32485562388</field>
            <field name="company_type">company</field>
        </record>

        <record id="partner_pizza_inn" model="res.partner">
            <field name="name">Pizza Inn</field>
            <field name="city">Gandhi Nagar</field>
            <field name="country_id" ref="base.in"/>
            <field name="company_type">company</field>
            <field name="image_1920" type="base64" file="lunch/static/img/pizza.jpeg"/>
            <field name="street">#8, 1 st Floor,iscore complex</field>
            <field name="street2">Gandhi Gramam</field>
            <field name="zip">607308</field>
            <field name="email">pizza.inn@yourcompany.example.com</field>
            <field name="phone">+32456325289</field>
        </record>

        <record model="res.partner" id="partner_corner">
            <field name="name">The Corner</field>
            <field name="city">Atlanta</field>
            <field name="country_id" ref="base.us"/>
            <field name="street">Genessee Ave SW</field>
            <field name="zip">607409</field>
            <field name="email">info@corner.com</field>
            <field name="phone">+32654321515</field>
            <field name="company_type">company</field>
        </record>

        <record model="res.partner" id="partner_sushi_shop">
            <field name="name">Sushi Shop</field>
            <field name="city">Paris</field>
            <field name="country_id" ref="base.fr"/>
            <field name="street">Boulevard Saint-Germain</field>
            <field name="zip">486624</field>
            <field name="email">order@sushi.com</field>
            <field name="phone">+32498859912</field>
            <field name="company_type">company</field>
        </record>

        <record model="lunch.supplier" id="supplier_coin_gourmand">
            <field name="partner_id" ref="partner_coin_gourmand"/>
            <field name="available_location_ids" eval="[
                (6, 0, [ref('location_office_1'), ref('location_office_2')]),
                ]"/>
        </record>

        <record model="lunch.supplier" id="supplier_pizza_inn">
            <field name="partner_id" ref="partner_pizza_inn"/>
            <field name="send_by">mail</field>
            <field name="automatic_email_time">11</field>
            <field name="available_location_ids" eval="[
                (6, 0, [ref('location_office_1'), ref('location_office_2')]),
                ]"/>
        </record>

        <record model="lunch.supplier" id="supplier_corner">
            <field name="partner_id" ref="partner_corner"/>
            <field name="available_location_ids" eval="[
                (6, 0, [ref('location_office_3')]),
                ]"/>
        </record>

        <record model="lunch.supplier" id="supplier_sushi_shop">
            <field name="partner_id" ref="partner_sushi_shop"/>
            <field name="available_location_ids" eval="[
                (6, 0, [ref('location_office_1'), ref('location_office_2')]),
                ]"/>
        </record>

        <record model="lunch.product" id="product_bacon">
            <field name="name">Bacon</field>
            <field name="category_id" ref="categ_burger"/>
            <field name="price">7.5</field>
            <field name="supplier_id" ref="supplier_corner"/>
            <field name="description">Beef, Bacon, Salad, Cheddar, Fried Onion, BBQ Sauce</field>
            <field name="image_1920" type="base64" file="lunch/static/img/bacon_burger.png"/>
            <field name="new_until" eval="datetime.today() + relativedelta(weeks=1)"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_cheeseburger">
            <field name="name">Cheese Burger</field>
            <field name="category_id" ref="categ_burger"/>
            <field name="price">7.0</field>
            <field name="supplier_id" ref="supplier_corner"/>
            <field name="description">Beef, Cheddar, Salad, Fried Onions, BBQ Sauce</field>
            <field name="image_1920" type="base64" file="lunch/static/img/cheeseburger.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_chicken_curry">
            <field name="name">Chicken Curry</field>
            <field name="category_id" ref="categ_sandwich"/>
            <field name="price">3.0</field>
            <field name="supplier_id" ref="supplier_corner"/>
            <field name="image_1920" type="base64" file="lunch/static/img/chicken_curry.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_spicy_tuna">
            <field name="name">Spicy Tuna</field>
            <field name="category_id" ref="categ_sandwich"/>
            <field name="price">3.0</field>
            <field name="supplier_id" ref="supplier_corner"/>
            <field name="description"></field>
            <field name="image_1920" type="base64" file="lunch/static/img/chicken_curry.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_mozzarella">
            <field name="name">Mozzarella</field>
            <field name="category_id" ref="categ_sandwich"/>
            <field name="price">3.9</field>
            <field name="supplier_id" ref="supplier_corner"/>
            <field name="description">Mozzarella, Pesto, Tomatoes</field>
            <field name="image_1920" type="base64" file="lunch/static/img/mozza.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_club">
            <field name="name">Club</field>
            <field name="category_id" ref="categ_sandwich"/>
            <field name="price">3.4</field>
            <field name="supplier_id" ref="supplier_corner"/>
            <field name="description">Ham, Cheese, Vegetables</field>
            <field name="image_1920" type="base64" file="lunch/static/img/club.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_maki">
            <field name="name">Lunch Maki 18pc</field>
            <field name="category_id" ref="categ_sushi"/>
            <field name="price">12.0</field>
            <field name="supplier_id" ref="supplier_sushi_shop"/>
            <field name="description">6 Maki Salmon - 6 Maki Tuna - 6 Maki Shrimp/Avocado</field>
            <field name="image_1920" type="base64" file="lunch/static/img/maki.png"/>
            <field name="new_until" eval="datetime.today() + relativedelta(weeks=1)"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_salmon">
            <field name="name">Lunch Salmon 20pc</field>
            <field name="category_id" ref="categ_sushi"/>
            <field name="price">13.80</field>
            <field name="supplier_id" ref="supplier_sushi_shop"/>
            <field name="description">4 Sushi Salmon - 6 Maki Salmon - 4 Sashimi Salmon </field>
            <field name="image_1920" type="base64" file="lunch/static/img/salmon_sushi.png"/>
            <field name="new_until" eval="datetime.today() + relativedelta(weeks=1)"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_temaki">
            <field name="name">Lunch Temaki mix 3pc</field>
            <field name="category_id" ref="categ_temaki"/>
            <field name="price">14.0</field>
            <field name="supplier_id" ref="supplier_sushi_shop"/>
            <field name="description">1 Avocado - 1 Salmon - 1 Eggs - 1 Tuna</field>
            <field name="image_1920" type="base64" file="lunch/static/img/temaki.png"/>
            <field name="new_until" eval="datetime.today() + relativedelta(weeks=1)"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_chirashi">
            <field name="name">Salmon and Avocado</field>
            <field name="category_id" ref="categ_chirashi"/>
            <field name="price">9.25</field>
            <field name="supplier_id" ref="supplier_sushi_shop"/>
            <field name="description">2 Tempuras, Cabbages, Onions, Sesame Sauce</field>
            <field name="image_1920" type="base64" file="lunch/static/img/chirashi.png"/>
            <field name="new_until" eval="datetime.today() + relativedelta(weeks=1)"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_cheese_ham">
            <field name="name">Cheese And Ham</field>
            <field name="category_id" ref="categ_sandwich"/>
            <field name="price">3.30</field>
            <field name="supplier_id" ref="supplier_coin_gourmand"/>
            <field name="description">Cheese, Ham, Salad, Tomatoes, cucumbers, eggs</field>
            <field name="image_1920" type="base64" file="lunch/static/img/club.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_country">
            <field name="name">The Country</field>
            <field name="category_id" ref="categ_sandwich"/>
            <field name="price">3.30</field>
            <field name="supplier_id" ref="supplier_coin_gourmand"/>
            <field name="description">Brie, Honey, Walnut Kernels</field>
            <field name="image_1920" type="base64" file="lunch/static/img/brie.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_tuna">
            <field name="name">Tuna</field>
            <field name="category_id" ref="categ_sandwich"/>
            <field name="price">2.50</field>
            <field name="supplier_id" ref="supplier_coin_gourmand"/>
            <field name="description">Tuna, Mayonnaise</field>
            <field name="image_1920" type="base64" file="lunch/static/img/tuna_sandwich.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_gouda">
            <field name="name">Gouda Cheese</field>
            <field name="category_id" ref="categ_sandwich"/>
            <field name="price">2.50</field>
            <field name="supplier_id" ref="supplier_coin_gourmand"/>
            <field name="description"></field>
            <field name="image_1920" type="base64" file="lunch/static/img/gouda.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_chicken_curry">
            <field name="name">Chicken Curry</field>
            <field name="category_id" ref="categ_sandwich"/>
            <field name="price">2.60</field>
            <field name="supplier_id" ref="supplier_coin_gourmand"/>
            <field name="description"></field>
            <field name="image_1920" type="base64" file="lunch/static/img/chicken_curry.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_margherita">
            <field name="name">Pizza Margherita</field>
            <field name="category_id" ref="categ_pizza"/>
            <field name="price">6.90</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
            <field name="description">Tomatoes, Mozzarella</field>
            <field name="image_1920" type="base64" file="lunch/static/img/pizza_margherita.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_funghi">
            <field name="name">Pizza Funghi</field>
            <field name="category_id" ref="categ_pizza"/>
            <field name="price">7.00</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
            <field name="description">Tomatoes, Mushrooms, Mozzarella</field>
            <field name="image_1920" type="base64" file="lunch/static/img/pizza_funghi.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_vege">
            <field name="name">Pizza Vegetarian</field>
            <field name="category_id" ref="categ_pizza"/>
            <field name="price">7.00</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
            <field name="description">Tomatoes, Mozzarella, Mushrooms, Peppers, Olives</field>
            <field name="image_1920" type="base64" file="lunch/static/img/pizza_veggie.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_italiana">
            <field name="name">Pizza Italiana</field>
            <field name="category_id" ref="categ_pizza"/>
            <field name="price">7.40</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
            <field name="description">Fresh Tomatoes, Basil, Mozzarella</field>
            <field name="image_1920" type="base64" file="lunch/static/img/italiana.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.product" id="product_Bolognese">
            <field name="name">Bolognese Pasta</field>
            <field name="category_id" ref="categ_pasta"/>
            <field name="price">7.70</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
            <field name="description"></field>
            <field name="image_1920" type="base64" file="lunch/static/img/pasta_bolognese.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

         <record model="lunch.product" id="product_Napoli">
            <field name="name">Napoli Pasta</field>
            <field name="category_id" ref="categ_pasta"/>
            <field name="price">7.70</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
            <field name="description">Tomatoes, Basil</field>
            <field name="image_1920" type="base64" file="lunch/static/img/napoli.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.topping" id="product_olives">
            <field name="name">Olives</field>
            <field name="price">0.30</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
        </record>

         <record model="lunch.product" id="product_4formaggi">
            <field name="name">4 Formaggi</field>
            <field name="category_id" ref="categ_pasta"/>
            <field name="price">5.50</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
            <field name="description">Tomato sauce, Olive oil, Fresh Tomatoes, Onions, Vegetables, Parmesan</field>
            <field name="image_1920" type="base64" file="lunch/static/img/4formaggio.png"/>
            <field name="company_id" ref="base.main_company"/>
        </record>

        <record model="lunch.cashmove" id="cashmove_1">
            <field name="user_id" ref="base.user_demo"/>
            <field name="date" eval="(DateTime.now() + timedelta(days=3)).strftime('%Y-%m-%d')"/>
            <field name="description">Payment: 5 lunch tickets (6€)</field>
            <field name="amount">30</field>
        </record>

        <record model="lunch.cashmove" id="cashmove_2">
            <field name="user_id" ref="base.user_admin"/>
            <field name="date" eval="(DateTime.now() - timedelta(days=3)).strftime('%Y-%m-%d')"/>
            <field name="description">Payment: 7 lunch tickets (6€)</field>
            <field name="amount">42</field>
        </record>

        <record model="lunch.order" id="order_line_1">
            <field name="user_id" ref="base.user_demo"/>
            <field name="product_id" ref="product_Bolognese"/>
            <field name="price">7.70</field>
            <field name="date" eval="(DateTime.now() + timedelta(days=2)).strftime('%Y-%m-%d')"/>
            <field name="state">new</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
            <field name="quantity">1</field>
            <field name="lunch_location_id" ref="location_office_3"/>
        </record>

        <record model="lunch.order" id="order_line_2">
            <field name="user_id" ref="base.user_demo"/>
            <field name="product_id" ref="product_italiana"/>
            <field name="price">7.40</field>
            <field name="date" eval="(DateTime.now() + timedelta(days=1)).strftime('%Y-%m-%d')"/>
            <field name="state">confirmed</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
            <field name="quantity">1</field>
            <field name="lunch_location_id" ref="location_office_3"/>
        </record>

        <record model="lunch.order" id="order_line_3">
            <field name="user_id" ref="base.user_demo"/>
            <field name="product_id" ref="product_gouda"/>
            <field name="price">2.50</field>
            <field name="date" eval="(DateTime.now() + timedelta(days=3)).strftime('%Y-%m-%d')"/>
            <field name="state">cancelled</field>
            <field name="supplier_id" ref="supplier_coin_gourmand"/>
            <field name="quantity">1</field>
            <field name="lunch_location_id" ref="location_office_3"/>
        </record>

        <record model="lunch.order" id="order_line_4">
            <field name="user_id" ref="base.user_demo"/>
            <field name="product_id" ref="product_chicken_curry"/>
            <field name="price">2.60</field>
            <field name="date" eval="(DateTime.now() + timedelta(days=3)).strftime('%Y-%m-%d')"/>
            <field name="state">confirmed</field>
            <field name="supplier_id" ref="supplier_coin_gourmand"/>
            <field name="quantity">1</field>
            <field name="lunch_location_id" ref="location_office_3"/>
        </record>

        <record model="lunch.order" id="order_line_5">
            <field name="user_id" ref="base.user_admin"/>
            <field name="product_id" ref="product_4formaggi"/>
            <field name="price">5.50</field>
            <field name="date" eval="(DateTime.now() + timedelta(days=3)).strftime('%Y-%m-%d')"/>
            <field name="state">confirmed</field>
            <field name="supplier_id" ref="supplier_pizza_inn"/>
            <field name="lunch_location_id" ref="location_office_2"/>
        </record>
    </data>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data noupdate="0">
    <record id="lunch_order_mail_supplier" model="mail.template">
        <field name="name">Lunch: Supplier Order</field>
        <field name="model_id" ref="lunch.model_lunch_supplier"/>
        <field name="email_from">{{ ctx['order']['email_from'] }}</field>
        <field name="partner_to">{{ ctx['order']['supplier_id'] }}</field>
        <field name="subject">Orders for {{ ctx['order']['company_name'] }}</field>
        <field name="lang">{{ ctx.get('default_lang') }}</field>
        <field name="description">Sent to vendor with the order of the day</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="padding-top: 16px; background-color: #F1F1F1; font-family:Verdana, Arial,sans-serif; color: #454748; width: 100%; border-collapse:separate;"><tr><td align="center">
<table border="0" cellpadding="0" cellspacing="0" width="590" style="padding: 16px; background-color: white; color: #454748; border-collapse:separate;">
<tbody>
    <!-- HEADER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle">
                    <span style="font-size: 10px;">Lunch Order</span><br/>
                </td><td valign="middle" align="right" t-if="not user.company_id.uses_default_logo">
                    <img t-attf-src="/logo.png?company={{ user.company_id.id }}" style="padding: 0px; margin: 0px; height: auto; width: 80px;" t-att-alt="user.company_id.name"/>
                </td></tr>
                <tr><td colspan="2" style="text-align:center;">
                  <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin:16px 0px 16px 0px;"/>
                </td></tr>
            </table>
        </td>
    </tr>
    <!-- CONTENT -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr>
                    <td valign="top" style="font-size: 13px;">
    <div>
        <t t-set="lines" t-value="ctx.get('lines', [])"/>
        <t t-set="order" t-value="ctx.get('order')"/>
        <t t-set="currency" t-value="user.env['res.currency'].browse(order.get('currency_id'))"/>
        <p>
        Dear <t t-out="order.get('supplier_name', '')">Laurie Poiret</t>,
        </p><p>
        Here is, today orders for <t t-out="order.get('company_name', '')">LunchCompany</t>:
        </p>

        <t t-if="sites">
            <br/>
            <p>Location</p>
            <t t-foreach="site" t-as="site">
                <p><t t-out="site['name'] or ''"></t> : <t t-out="site['address'] or ''"></t></p>
            </t>
            <br/>
        </t>

        <table>
            <thead>
                <tr style="background-color:rgb(233,232,233);">
                    <th style="width: 100%; min-width: 96px; font-size: 13px;"><strong>Product</strong></th>
                    <th style="width: 100%; min-width: 96px; font-size: 13px;"><strong>Comments</strong></th>
                    <th style="width: 100%; min-width: 96px; font-size: 13px;"><strong>Person</strong></th>
                    <th style="width: 100%; min-width: 96px; font-size: 13px;"><strong>Site</strong></th>
                    <th style="width: 100%; min-width: 96px; font-size: 13px;" align="center"><strong>Qty</strong></th>
                    <th style="width: 100%; min-width: 96px; font-size: 13px;" align="center"><strong>Price</strong></th>
                </tr>
            </thead>
            <tbody>
                <tr t-foreach="lines" t-as="line">
                    <td style="width: 100%; font-size: 13px;" valign="top" t-out="line['product'] or ''">Sushi salmon</td>
                    <td style="width: 100%; font-size: 13px;" valign="top">
                    <t t-if="line['toppings']">
                        <t t-out="line['toppings'] or ''">Soy sauce</t>
                    </t>
                    <t t-if="line['note']">
                        <div style="color: rgb(173,181,189);" t-out="line['note'] or ''">With wasabi.</div>
                    </t>
                    </td>
                    <td style="width: 100%; font-size: 13px;" valign="top" t-out="line['username'] or ''">lap</td>
                    <td style="width: 100%; font-size: 13px;" valign="top" t-out="line['site'] or ''">Office 1</td>
                    <td style="width: 100%; font-size: 13px;" valign="top" align="right" t-out="line['quantity'] or ''">10</td>
                    <td style="width: 100%; font-size: 13px;" valign="top" align="right" t-out="format_amount(line['price'], currency) or ''">$ 1.00</td>
                </tr>
                <tr>
                    <td></td>
                    <td></td>
                    <td></td>
                    <td></td>
                    <td style="width: 100%; font-size: 13px; border-top: 1px solid black;"><strong>Total</strong></td>
                    <td style="width: 100%; font-size: 13px; border-top: 1px solid black;" align="right"><strong t-out="format_amount(order['amount_total'], currency) or ''">$ 10.00</strong></td>
                </tr>
            </tbody>
        </table>

        <p>Do not hesitate to contact us if you have any questions.</p>
    </div>
                    </td>
                </tr>
                <tr>
                    <td style="text-align:center;">
                        <hr width="100%" style="background-color:rgb(204,204,204);border:medium none;clear:both;display:block;font-size:0px;min-height:1px;line-height:0; margin: 16px 0px 16px 0px;"/>
                    </td>
                </tr>
            </table>
        </td>
    </tr>
    <!-- FOOTER -->
    <tr>
        <td align="center" style="min-width: 590px;">
            <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: white; font-size: 11px; padding: 0px 8px 0px 8px; border-collapse:separate;">
                <tr><td valign="middle" align="left">
                    <t t-out="user.company_id.name or ''">YourCompany</t>
                </td></tr>
                <tr><td valign="middle" align="left" style="opacity: 0.7;">
                    <t t-out="user.company_id.phone or ''">+1 650-123-4567</t>
                    <t t-if="user.company_id.phone and (user.company_id.email or user.company_id.website)">|</t>
                    <t t-if="user.company_id.email">
                        <a t-attf-href="'mailto:%s' % {{ user.company_id.email }}" style="text-decoration:none; color: #454748;" t-out="user.company_id.email or ''">info@yourcompany.com</a>
                    </t>
                    <t t-if="user.company_id.email and user.company_id.website">|</t>
                    <t t-if="user.company_id.website">
                        <a t-attf-href="'%s' % {{ user.company_id.website }}" style="text-decoration:none; color: #454748;" t-out="user.company_id.website or ''">http://www.example.com</a>
                    </t>
                </td></tr>
            </table>
        </td>
    </tr>
</tbody>
</table>
</td></tr>
<!-- POWERED BY -->
<tr><td align="center" style="min-width: 590px;">
    <table border="0" cellpadding="0" cellspacing="0" width="590" style="min-width: 590px; background-color: #F1F1F1; color: #454748; padding: 8px; border-collapse:separate;">
      <tr><td style="text-align: center; font-size: 13px;">
        Powered by <a target="_blank" href="https://www.odoo.com" style="color: #875A7B;">Odoo</a>
      </td></tr>
    </table>
</td></tr>
</table>
        </field>
    </record>

</data></odoo>

```

## File: models\lunch_alert.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import pytz
import logging

from odoo import api, fields, models, _
from odoo.osv import expression

from .lunch_supplier import float_to_time
from datetime import datetime, timedelta
from textwrap import dedent

from odoo.addons.base.models.res_partner import _tz_get

_logger = logging.getLogger(__name__)
WEEKDAY_TO_NAME = ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun']
CRON_DEPENDS = {'name', 'active', 'mode', 'until', 'notification_time', 'notification_moment', 'tz'}


class LunchAlert(models.Model):
    """ Alerts to display during a lunch order. An alert can be specific to a
    given day, weekly or daily. The alert is displayed from start to end hour. """
    _name = 'lunch.alert'
    _description = 'Lunch Alert'
    _order = 'write_date desc, id'

    name = fields.Char('Alert Name', required=True, translate=True)
    message = fields.Html('Message', required=True, translate=True)

    mode = fields.Selection([
        ('alert', 'Alert in app'),
        ('chat', 'Chat notification')], string='Display', default='alert')
    recipients = fields.Selection([
        ('everyone', 'Everyone'),
        ('last_week', 'Employee who ordered last week'),
        ('last_month', 'Employee who ordered last month'),
        ('last_year', 'Employee who ordered last year')], string='Recipients', default='everyone')
    notification_time = fields.Float(default=10.0, string='Notification Time')
    notification_moment = fields.Selection([
        ('am', 'AM'),
        ('pm', 'PM')], default='am', required=True)
    tz = fields.Selection(_tz_get, string='Timezone', required=True, default=lambda self: self.env.user.tz or 'UTC')
    cron_id = fields.Many2one('ir.cron', ondelete='cascade', required=True, readonly=True)

    until = fields.Date('Show Until')
    mon = fields.Boolean(default=True)
    tue = fields.Boolean(default=True)
    wed = fields.Boolean(default=True)
    thu = fields.Boolean(default=True)
    fri = fields.Boolean(default=True)
    sat = fields.Boolean(default=True)
    sun = fields.Boolean(default=True)

    available_today = fields.Boolean('Is Displayed Today',
                                     compute='_compute_available_today', search='_search_available_today')

    active = fields.Boolean('Active', default=True)

    location_ids = fields.Many2many('lunch.location', string='Location')

    _sql_constraints = [
        ('notification_time_range',
            'CHECK(notification_time >= 0 and notification_time <= 12)',
            'Notification time must be between 0 and 12')
    ]

    @api.depends('mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun')
    def _compute_available_today(self):
        today = fields.Date.context_today(self)
        fieldname = WEEKDAY_TO_NAME[today.weekday()]

        for alert in self:
            alert.available_today = alert.until > today if alert.until else True and alert[fieldname]

    def _search_available_today(self, operator, value):
        if (not operator in ['=', '!=']) or (not value in [True, False]):
            return []

        searching_for_true = (operator == '=' and value) or (operator == '!=' and not value)
        today = fields.Date.context_today(self)
        fieldname = WEEKDAY_TO_NAME[today.weekday()]

        return expression.AND([
            [(fieldname, operator, value)],
            expression.OR([
                [('until', '=', False)],
                [('until', '>' if searching_for_true else '<', today)],
            ])
        ])

    def _sync_cron(self):
        """ Synchronise the related cron fields to reflect this alert """
        for alert in self:
            alert = alert.with_context(tz=alert.tz)

            cron_required = (
                alert.active
                and alert.mode == 'chat'
                and (not alert.until or fields.Date.context_today(alert) <= alert.until)
            )

            sendat_tz = pytz.timezone(alert.tz).localize(datetime.combine(
                fields.Date.context_today(alert, fields.Datetime.now()),
                float_to_time(alert.notification_time, alert.notification_moment)))
            cron = alert.cron_id.sudo()
            lc = cron.lastcall
            if ((
                lc and sendat_tz.date() <= fields.Datetime.context_timestamp(alert, lc).date()
            ) or (
                not lc and sendat_tz <= fields.Datetime.context_timestamp(alert, fields.Datetime.now())
            )):
                sendat_tz += timedelta(days=1)
            sendat_utc = sendat_tz.astimezone(pytz.UTC).replace(tzinfo=None)

            cron.name = f"Lunch: alert chat notification ({alert.name})"
            cron.active = cron_required
            cron.nextcall = sendat_utc
            cron.code = dedent(f"""\
                # This cron is dynamically controlled by {self._description}.
                # Do NOT modify this cron, modify the related record instead.
                env['{self._name}'].browse([{alert.id}])._notify_chat()""")

    @api.model_create_multi
    def create(self, vals_list):
        crons = self.env['ir.cron'].sudo().create([
            {
                'user_id': self.env.ref('base.user_root').id,
                'active': False,
                'interval_type': 'days',
                'interval_number': 1,
                'name': "Lunch: alert chat notification",
                'model_id': self.env['ir.model']._get_id(self._name),
                'state': 'code',
                'code': "",
            }
            for _ in range(len(vals_list))
        ])
        self.env['ir.model.data'].sudo().create([{
            'name': f'lunch_alert_cron_sa_{cron.ir_actions_server_id.id}',
            'module': 'lunch',
            'res_id': cron.ir_actions_server_id.id,
            'model': 'ir.actions.server',
            # noupdate is set to true to avoid to delete record at module update
            'noupdate': True,
        } for cron in crons])
        for vals, cron in zip(vals_list, crons):
            vals['cron_id'] = cron.id

        alerts = super().create(vals_list)
        alerts._sync_cron()
        return alerts

    def write(self, values):
        res = super().write(values)
        if not CRON_DEPENDS.isdisjoint(values):
            self._sync_cron()
        return res

    def unlink(self):
        crons = self.cron_id.sudo()
        server_actions = crons.ir_actions_server_id
        res = super().unlink()
        crons.unlink()
        server_actions.unlink()
        return res

    def _notify_chat(self):
        # Called daily by cron
        self.ensure_one()

        if not self.available_today:
            _logger.warning("cancelled, not available today")
            if self.cron_id and self.until and fields.Date.context_today(self) > self.until:
                self.cron_id.unlink()
                self.cron_id = False
            return

        if not self.active or self.mode != 'chat':
            raise ValueError("Cannot send a chat notification in the current state")

        order_domain = [('state', '!=', 'cancelled')]

        if self.location_ids.ids:
            order_domain = expression.AND([order_domain, [('user_id.last_lunch_location_id', 'in', self.location_ids.ids)]])

        if self.recipients != 'everyone':
            weeksago = fields.Date.today() - timedelta(weeks=(
                1 if self.recipients == 'last_week' else
                4 if self.recipients == 'last_month' else
                52  # if self.recipients == 'last_year'
            ))
            order_domain = expression.AND([order_domain, [('date', '>=', weeksago)]])

        partners = self.env['lunch.order'].search(order_domain).user_id.partner_id
        if partners:
            self.env['mail.thread'].message_notify(
                model=self._name,
                res_id=self.id,
                body=self.message,
                partner_ids=partners.ids,
                subject=_('Your Lunch Order'),
            )

```

## File: models\lunch_cashmove.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.tools import float_round


class LunchCashMove(models.Model):
    """ Two types of cashmoves: payment (credit) or order (debit) """
    _name = 'lunch.cashmove'
    _description = 'Lunch Cashmove'
    _order = 'date desc'

    currency_id = fields.Many2one('res.currency', default=lambda self: self.env.company.currency_id, required=True)
    user_id = fields.Many2one('res.users', 'User',
                              default=lambda self: self.env.uid)
    date = fields.Date('Date', required=True, default=fields.Date.context_today)
    amount = fields.Float('Amount', required=True)
    description = fields.Text('Description')

    def _compute_display_name(self):
        for cashmove in self:
            cashmove.display_name = '{} {}'.format(_('Lunch Cashmove'), '#%d' % cashmove.id)

    @api.model
    def get_wallet_balance(self, user, include_config=True):
        result = float_round(sum(move['amount'] for move in self.env['lunch.cashmove.report'].search_read(
            [('user_id', '=', user.id)], ['amount'])), precision_digits=2)
        if include_config:
            result += user.company_id.lunch_minimum_threshold
        return result

```

## File: models\lunch_location.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class LunchLocation(models.Model):
    _name = 'lunch.location'
    _description = 'Lunch Locations'

    name = fields.Char('Location Name', required=True)
    address = fields.Text('Address')
    company_id = fields.Many2one('res.company', default=lambda self: self.env.company)

```

## File: models\lunch_order.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _

from odoo.exceptions import ValidationError, UserError
from odoo.osv.expression import AND


class LunchOrder(models.Model):
    _name = 'lunch.order'
    _description = 'Lunch Order'
    _order = 'id desc'
    _display_name = 'product_id'

    name = fields.Char(related='product_id.name', string="Product Name", readonly=True)
    topping_ids_1 = fields.Many2many('lunch.topping', 'lunch_order_topping', 'order_id', 'topping_id', string='Extras 1', domain=[('topping_category', '=', 1)])
    topping_ids_2 = fields.Many2many('lunch.topping', 'lunch_order_topping', 'order_id', 'topping_id', string='Extras 2', domain=[('topping_category', '=', 2)])
    topping_ids_3 = fields.Many2many('lunch.topping', 'lunch_order_topping', 'order_id', 'topping_id', string='Extras 3', domain=[('topping_category', '=', 3)])
    product_id = fields.Many2one('lunch.product', string="Product", required=True)
    category_id = fields.Many2one(
        string='Product Category', related='product_id.category_id', store=True)
    date = fields.Date('Order Date', required=True, readonly=False,
                       default=fields.Date.context_today)
    supplier_id = fields.Many2one(
        string='Vendor', related='product_id.supplier_id', store=True, index=True)
    available_today = fields.Boolean(related='supplier_id.available_today')

    available_on_date = fields.Boolean(compute='_compute_available_on_date')
    order_deadline_passed = fields.Boolean(compute='_compute_order_deadline_passed')
    user_id = fields.Many2one('res.users', 'User', default=lambda self: self.env.uid)
    lunch_location_id = fields.Many2one('lunch.location', default=lambda self: self.env.user.last_lunch_location_id)
    note = fields.Text('Notes')
    price = fields.Monetary('Total Price', compute='_compute_total_price', readonly=True, store=True)
    active = fields.Boolean('Active', default=True)
    state = fields.Selection([('new', 'To Order'),
                              ('ordered', 'Ordered'),       # "Internally" ordered
                              ('sent', 'Sent'),             # Order sent to the supplier
                              ('confirmed', 'Received'),    # Order received
                              ('cancelled', 'Cancelled')],
                             'Status', readonly=True, index=True, default='new')
    notified = fields.Boolean(default=False)
    company_id = fields.Many2one('res.company', default=lambda self: self.env.company.id)
    currency_id = fields.Many2one(related='company_id.currency_id', store=True)
    quantity = fields.Float('Quantity', required=True, default=1)

    display_toppings = fields.Text('Extras', compute='_compute_display_toppings', store=True)

    product_description = fields.Html('Description', related='product_id.description')
    topping_label_1 = fields.Char(related='product_id.supplier_id.topping_label_1')
    topping_label_2 = fields.Char(related='product_id.supplier_id.topping_label_2')
    topping_label_3 = fields.Char(related='product_id.supplier_id.topping_label_3')
    topping_quantity_1 = fields.Selection(related='product_id.supplier_id.topping_quantity_1')
    topping_quantity_2 = fields.Selection(related='product_id.supplier_id.topping_quantity_2')
    topping_quantity_3 = fields.Selection(related='product_id.supplier_id.topping_quantity_3')
    image_1920 = fields.Image(compute='_compute_product_images')
    image_128 = fields.Image(compute='_compute_product_images')

    available_toppings_1 = fields.Boolean(help='Are extras available for this product', compute='_compute_available_toppings')
    available_toppings_2 = fields.Boolean(help='Are extras available for this product', compute='_compute_available_toppings')
    available_toppings_3 = fields.Boolean(help='Are extras available for this product', compute='_compute_available_toppings')
    display_reorder_button = fields.Boolean(compute='_compute_display_reorder_button')
    display_add_button = fields.Boolean(compute='_compute_display_add_button')

    @api.depends('product_id')
    def _compute_product_images(self):
        for line in self:
            line.image_1920 = line.product_id.image_1920 or line.category_id.image_1920
            line.image_128 = line.product_id.image_128 or line.category_id.image_128

    @api.depends('category_id')
    def _compute_available_toppings(self):
        for order in self:
            order.available_toppings_1 = bool(order.env['lunch.topping'].search_count([('supplier_id', '=', order.supplier_id.id), ('topping_category', '=', 1)]))
            order.available_toppings_2 = bool(order.env['lunch.topping'].search_count([('supplier_id', '=', order.supplier_id.id), ('topping_category', '=', 2)]))
            order.available_toppings_3 = bool(order.env['lunch.topping'].search_count([('supplier_id', '=', order.supplier_id.id), ('topping_category', '=', 3)]))

    @api.depends('name')
    def _compute_display_add_button(self):
        new_orders = dict(self.env["lunch.order"]._read_group([
            ("date", "in", self.mapped("date")),
            ("user_id", "in", self.user_id.ids),
            ("state", "=", "new"),
        ], ['user_id'], ['id:recordset']))
        for order in self:
            user_new_orders = new_orders.get(order.user_id)
            price = 0
            if user_new_orders:
                user_new_orders = user_new_orders.filtered(lambda lunch_order: lunch_order.date == order.date)
                price = sum(order.price for order in user_new_orders)
            wallet_amount = self.env['lunch.cashmove'].get_wallet_balance(order.user_id, False) - price
            order.display_add_button = wallet_amount >= order.price

    @api.depends_context('show_reorder_button')
    @api.depends('state')
    def _compute_display_reorder_button(self):
        show_button = self.env.context.get('show_reorder_button')
        for order in self:
            order.display_reorder_button = show_button and order.state == 'confirmed' and order.supplier_id.available_today

    @api.depends('date', 'supplier_id')
    def _compute_available_on_date(self):
        for order in self:
            order.available_on_date = order.supplier_id._available_on_date(order.date)

    @api.depends('supplier_id', 'date')
    def _compute_order_deadline_passed(self):
        today = fields.Date.context_today(self)
        for order in self:
            if order.date < today:
                order.order_deadline_passed = True
            elif order.date == today:
                order.order_deadline_passed = order.supplier_id.order_deadline_passed
            else:
                order.order_deadline_passed = False

    def init(self):
        self._cr.execute("""CREATE INDEX IF NOT EXISTS lunch_order_user_product_date ON %s (user_id, product_id, date)"""
            % self._table)

    def _get_topping_ids(self, field, values):
        return list(self._fields[field].convert_to_cache(values, self))

    def _extract_toppings(self, values):
        """
            If called in api.multi then it will pop topping_ids_1,2,3 from values
        """
        topping_ids = []

        for i in range(1, 4):
            topping_field = f'topping_ids_{i}'
            topping_values = values.get(topping_field, False)

            if self.ids:
                # TODO This is not taking into account all the toppings for each individual order, this is usually not a problem
                # since in the interface you usually don't update more than one order at a time but this is a bug nonetheless
                topping_ids += self._get_topping_ids(topping_field, values.pop(topping_field)) \
                    if topping_values else self[:1][topping_field].ids
            else:
                topping_ids += self._get_topping_ids(topping_field, topping_values) if topping_values else []

        return topping_ids

    @api.constrains('topping_ids_1', 'topping_ids_2', 'topping_ids_3')
    def _check_topping_quantity(self):
        errors = {
            '1_more': _('You should order at least one %s'),
            '1': _('You have to order one and only one %s'),
        }
        for line in self:
            for index in range(1, 4):
                availability = line['available_toppings_%s' % index]
                quantity = line['topping_quantity_%s' % index]
                toppings = line['topping_ids_%s' % index].filtered(lambda x: x.topping_category == index)
                label = line['topping_label_%s' % index]

                if availability and quantity != '0_more':
                    check = bool(len(toppings) == 1 if quantity == '1' else toppings)
                    if not check:
                        raise ValidationError(errors[quantity] % label)

    @api.model_create_multi
    def create(self, vals_list):
        orders = self.env['lunch.order']
        for vals in vals_list:
            lines = self._find_matching_lines({
                **vals,
                'toppings': self._extract_toppings(vals),
            })
            if lines.filtered(lambda l: l.state == 'new'):
                # YTI FIXME This will update multiple lines in the case there are multiple
                # matching lines which should not happen through the interface
                lines.update_quantity(1)
                orders |= lines[:1]
            else:
                orders |= super().create(vals)
        return orders

    def write(self, values):
        change_topping = 'topping_ids_1' in values or 'topping_ids_2' in values or 'topping_ids_3' in values
        merge_needed = 'note' in values or change_topping or 'state' in values
        default_location_id = self.env.user.last_lunch_location_id and self.env.user.last_lunch_location_id.id or False

        if merge_needed:
            lines_to_deactivate = self.env['lunch.order']
            for line in self:
                # Only write on topping_ids_1 because they all share the same table
                # and we don't want to remove all the records
                # _extract_toppings will pop topping_ids_1, topping_ids_2 and topping_ids_3 from values
                # This also forces us to invalidate the cache for topping_ids_2 and topping_ids_3 that
                # could have changed through topping_ids_1 without the cache knowing about it
                toppings = self._extract_toppings(values)
                if change_topping:
                    self.invalidate_model(['topping_ids_2', 'topping_ids_3'])
                    values['topping_ids_1'] = [(6, 0, toppings)]
                matching_lines = self._find_matching_lines({
                    'user_id': values.get('user_id', line.user_id.id),
                    'product_id': values.get('product_id', line.product_id.id),
                    'note': values.get('note', line.note or False),
                    'toppings': toppings,
                    'lunch_location_id': values.get('lunch_location_id', default_location_id),
                    'state': values.get('state'),
                })
                if matching_lines:
                    lines_to_deactivate |= line
                    matching_lines.update_quantity(line.quantity)
            lines_to_deactivate.write({'active': False})
            return super(LunchOrder, self - lines_to_deactivate).write(values)
        return super().write(values)

    @api.model
    def _find_matching_lines(self, values):
        default_location_id = self.env.user.last_lunch_location_id and self.env.user.last_lunch_location_id.id or False
        domain = [
            ('user_id', '=', values.get('user_id', self.default_get(['user_id'])['user_id'])),
            ('product_id', '=', values.get('product_id', False)),
            ('date', '=', values.get('date', fields.Date.today())),
            ('note', '=', values.get('note', False)),
            ('lunch_location_id', '=', values.get('lunch_location_id', default_location_id)),
        ]
        if values.get('state'):
            domain = AND([domain, [('state', '=', values['state'])]])
        toppings = values.get('toppings', [])
        return self.search(domain).filtered(lambda line: (line.topping_ids_1 | line.topping_ids_2 | line.topping_ids_3).ids == toppings)

    @api.depends('topping_ids_1', 'topping_ids_2', 'topping_ids_3', 'product_id', 'quantity')
    def _compute_total_price(self):
        for line in self:
            line.price = line.quantity * (line.product_id.price + sum((line.topping_ids_1 | line.topping_ids_2 | line.topping_ids_3).mapped('price')))

    @api.depends('topping_ids_1', 'topping_ids_2', 'topping_ids_3')
    def _compute_display_toppings(self):
        for line in self:
            toppings = line.topping_ids_1 | line.topping_ids_2 | line.topping_ids_3
            line.display_toppings = ' + '.join(toppings.mapped('name'))

    def update_quantity(self, increment):
        for line in self.filtered(lambda line: line.state not in ['sent', 'confirmed']):
            if line.quantity <= -increment:
                # TODO: maybe unlink the order?
                line.active = False
            else:
                line.quantity += increment
        self._check_wallet()

    def add_to_cart(self):
        """
            This method currently does nothing, we currently need it in order to
            be able to reuse this model in place of a wizard
        """
        # YTI FIXME: Find a way to drop this.
        return True

    def _check_wallet(self):
        self.env.flush_all()
        for line in self:
            if self.env['lunch.cashmove'].get_wallet_balance(line.user_id) < 0:
                raise ValidationError(_('Your wallet does not contain enough money to order that. To add some money to your wallet, please contact your lunch manager.'))

    def action_order(self):
        for order in self:
            if not order.available_on_date:
                raise UserError(_('The vendor related to this order is not available at the selected date.'))
        if self.filtered(lambda line: not line.product_id.active):
            raise ValidationError(_('Product is no longer available.'))
        self.write({
            'state': 'ordered',
        })
        self._check_wallet()

    def action_reorder(self):
        self.ensure_one()
        if not self.supplier_id.available_today:
            raise UserError(_('The vendor related to this order is not available today.'))
        self.copy({
            'date': fields.Date.context_today(self),
            'state': 'ordered',
        })
        action = self.env['ir.actions.act_window']._for_xml_id('lunch.lunch_order_action')
        return action

    def action_confirm(self):
        self.write({'state': 'confirmed'})

    def action_cancel(self):
        self.write({'state': 'cancelled'})

    def action_reset(self):
        self.write({'state': 'ordered'})

    def action_send(self):
        self.state = 'sent'

    def action_notify(self):
        self -= self.filtered('notified')
        if not self:
            return
        notified_users = set()
        # (company, lang): (subject, body)
        translate_cache = dict()
        for order in self:
            user = order.user_id
            if user in notified_users:
                continue
            _key = (order.company_id, user.lang)
            if _key not in translate_cache:
                context = {'lang': user.lang}
                translate_cache[_key] = (_('Lunch notification'), order.company_id.with_context(lang=user.lang).lunch_notify_message)
                del context
            subject, body = translate_cache[_key]
            user.partner_id.message_notify(
                subject=subject,
                body=body,
                partner_ids=user.partner_id.ids,
                email_layout_xmlid='mail.mail_notification_light',
            )
            notified_users.add(user)
        self.write({'notified': True})

```

## File: models\lunch_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv import expression


class LunchProduct(models.Model):
    """ Products available to order. A product is linked to a specific vendor. """
    _name = 'lunch.product'
    _description = 'Lunch Product'
    _inherit = 'image.mixin'
    _order = 'name'
    _check_company_auto = True

    name = fields.Char('Product Name', required=True, translate=True)
    category_id = fields.Many2one('lunch.product.category', 'Product Category', check_company=True, required=True)
    description = fields.Html('Description', translate=True)
    price = fields.Float('Price', digits='Account', required=True)
    supplier_id = fields.Many2one('lunch.supplier', 'Vendor', check_company=True, required=True)
    active = fields.Boolean(default=True)

    company_id = fields.Many2one('res.company', related='supplier_id.company_id', readonly=False, store=True)
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id')

    new_until = fields.Date('New Until')
    is_new = fields.Boolean(compute='_compute_is_new')

    favorite_user_ids = fields.Many2many('res.users', 'lunch_product_favorite_user_rel', 'product_id', 'user_id', check_company=True)
    is_favorite = fields.Boolean(compute='_compute_is_favorite', inverse='_inverse_is_favorite')

    last_order_date = fields.Date(compute='_compute_last_order_date')

    product_image = fields.Image(compute='_compute_product_image')
    # This field is used only for searching
    is_available_at = fields.Many2one('lunch.location', 'Product Availability', compute='_compute_is_available_at', search='_search_is_available_at')

    @api.depends('image_128', 'category_id.image_128')
    def _compute_product_image(self):
        for product in self:
            product.product_image = product.image_128 or product.category_id.image_128

    @api.depends('new_until')
    def _compute_is_new(self):
        today = fields.Date.context_today(self)
        for product in self:
            if product.new_until:
                product.is_new = today <= product.new_until
            else:
                product.is_new = False

    @api.depends_context('uid')
    @api.depends('favorite_user_ids')
    def _compute_is_favorite(self):
        for product in self:
            product.is_favorite = self.env.user in product.favorite_user_ids

    @api.depends_context('uid')
    def _compute_last_order_date(self):
        all_orders = self.env['lunch.order'].search([
            ('user_id', '=', self.env.user.id),
            ('product_id', 'in', self.ids),
        ])
        mapped_orders = defaultdict(lambda: self.env['lunch.order'])
        for order in all_orders:
            mapped_orders[order.product_id] |= order
        for product in self:
            if not mapped_orders[product]:
                product.last_order_date = False
            else:
                product.last_order_date = max(mapped_orders[product].mapped('date'))

    def _compute_is_available_at(self):
        """
            Is available_at is always false when browsing it
            this field is there only to search (see _search_is_available_at)
        """
        for product in self:
            product.is_available_at = False

    def _search_is_available_at(self, operator, value):
        supported_operators = ['in', 'not in', '=', '!=']

        if not operator in supported_operators:
            return expression.TRUE_DOMAIN

        if isinstance(value, int):
            value = [value]

        if operator in expression.NEGATIVE_TERM_OPERATORS:
            return expression.AND([[('supplier_id.available_location_ids', 'not in', value)], [('supplier_id.available_location_ids', '!=', False)]])

        return expression.OR([[('supplier_id.available_location_ids', 'in', value)], [('supplier_id.available_location_ids', '=', False)]])

    def _sync_active_from_related(self):
        """ Archive/unarchive product after related field is archived/unarchived """
        return self.filtered(lambda p: (p.category_id.active and p.supplier_id.active) != p.active).toggle_active()

    def toggle_active(self):
        invalid_products = self.filtered(lambda product: not product.active and not product.category_id.active)
        if invalid_products:
            raise UserError(_("The following product categories are archived. You should either unarchive the categories or change the category of the product.\n%s", '\n'.join(invalid_products.category_id.mapped('name'))))
        invalid_products = self.filtered(lambda product: not product.active and not product.supplier_id.active)
        if invalid_products:
            raise UserError(_("The following suppliers are archived. You should either unarchive the suppliers or change the supplier of the product.\n%s", '\n'.join(invalid_products.supplier_id.mapped('name'))))
        return super().toggle_active()

    def _inverse_is_favorite(self):
        """ Handled in the write() """
        return

    def write(self, vals):
        if 'is_favorite' in vals:
            if vals.pop('is_favorite'):
                commands = [(4, product.id) for product in self]
            else:
                commands = [(3, product.id) for product in self]
            self.env.user.write({
                'favorite_lunch_product_ids': commands,
            })

        if not vals:
            return True
        return super().write(vals)

```

## File: models\lunch_product_category.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import api, fields, models

from odoo.tools.misc import file_open


class LunchProductCategory(models.Model):
    """ Category of the product such as pizza, sandwich, pasta, chinese, burger... """
    _name = 'lunch.product.category'
    _inherit = 'image.mixin'
    _description = 'Lunch Product Category'

    @api.model
    def _default_image(self):
        return base64.b64encode(file_open('lunch/static/img/lunch.png', 'rb').read())

    name = fields.Char('Product Category', required=True, translate=True)
    company_id = fields.Many2one('res.company')
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id')
    product_count = fields.Integer(compute='_compute_product_count', help="The number of products related to this category")
    active = fields.Boolean(string='Active', default=True)
    image_1920 = fields.Image(default=_default_image)

    def _compute_product_count(self):
        product_data = self.env['lunch.product']._read_group([('category_id', 'in', self.ids)], ['category_id'], ['__count'])
        data = {category.id: count for category, count in product_data}
        for category in self:
            category.product_count = data.get(category.id, 0)

    def toggle_active(self):
        """ Archiving related lunch product """
        res = super().toggle_active()
        Product = self.env['lunch.product'].with_context(active_test=False)
        all_products = Product.search([('category_id', 'in', self.ids)])
        all_products._sync_active_from_related()
        return res

```

## File: models\lunch_supplier.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import math
import pytz

from collections import defaultdict
from datetime import datetime, time, timedelta
from textwrap import dedent

from odoo import api, fields, models, _
from odoo.exceptions import UserError
from odoo.osv import expression
from odoo.tools import float_round

from odoo.addons.base.models.res_partner import _tz_get


WEEKDAY_TO_NAME = ['mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun']
CRON_DEPENDS = {'name', 'active', 'send_by', 'automatic_email_time', 'moment', 'tz'}

def float_to_time(hours, moment='am'):
    """ Convert a number of hours into a time object. """
    if hours == 12.0 and moment == 'pm':
        return time.max
    fractional, integral = math.modf(hours)
    if moment == 'pm':
        integral += 12
    return time(int(integral), int(float_round(60 * fractional, precision_digits=0)), 0)

def time_to_float(t):
    return float_round(t.hour + t.minute/60 + t.second/3600, precision_digits=2)

class LunchSupplier(models.Model):
    _name = 'lunch.supplier'
    _description = 'Lunch Supplier'
    _inherit = ['mail.thread', 'mail.activity.mixin']

    partner_id = fields.Many2one('res.partner', string='Vendor', required=True)

    name = fields.Char('Name', related='partner_id.name', readonly=False)

    email = fields.Char(related='partner_id.email', readonly=False)
    email_formatted = fields.Char(related='partner_id.email_formatted', readonly=True)
    phone = fields.Char(related='partner_id.phone', readonly=False)
    street = fields.Char(related='partner_id.street', readonly=False)
    street2 = fields.Char(related='partner_id.street2', readonly=False)
    zip_code = fields.Char(related='partner_id.zip', readonly=False)
    city = fields.Char(related='partner_id.city', readonly=False)
    state_id = fields.Many2one("res.country.state", related='partner_id.state_id', readonly=False)
    country_id = fields.Many2one('res.country', related='partner_id.country_id', readonly=False)
    company_id = fields.Many2one('res.company', related='partner_id.company_id', readonly=False, store=True)

    responsible_id = fields.Many2one('res.users', string="Responsible", domain=lambda self: [('groups_id', 'in', self.env.ref('lunch.group_lunch_manager').id)],
                                     default=lambda self: self.env.user,
                                     help="The responsible is the person that will order lunch for everyone. It will be used as the 'from' when sending the automatic email.")

    send_by = fields.Selection([
        ('phone', 'Phone'),
        ('mail', 'Email'),
    ], 'Send Order By', default='phone')
    automatic_email_time = fields.Float('Order Time', default=12.0, required=True)
    cron_id = fields.Many2one('ir.cron', ondelete='cascade', required=True, readonly=True)

    mon = fields.Boolean(default=True)
    tue = fields.Boolean(default=True)
    wed = fields.Boolean(default=True)
    thu = fields.Boolean(default=True)
    fri = fields.Boolean(default=True)
    sat = fields.Boolean()
    sun = fields.Boolean()

    recurrency_end_date = fields.Date('Until', help="This field is used in order to ")

    available_location_ids = fields.Many2many('lunch.location', string='Location')
    available_today = fields.Boolean('This is True when if the supplier is available today',
                                     compute='_compute_available_today', search='_search_available_today')
    order_deadline_passed = fields.Boolean(compute='_compute_order_deadline_passed')

    tz = fields.Selection(_tz_get, string='Timezone', required=True, default=lambda self: self.env.user.tz or 'UTC')

    active = fields.Boolean(default=True)

    moment = fields.Selection([
        ('am', 'AM'),
        ('pm', 'PM'),
    ], default='am', required=True)

    delivery = fields.Selection([
        ('delivery', 'Delivery'),
        ('no_delivery', 'No Delivery')
    ], default='no_delivery')

    topping_label_1 = fields.Char('Extra 1 Label', required=True, default='Extras')
    topping_label_2 = fields.Char('Extra 2 Label', required=True, default='Beverages')
    topping_label_3 = fields.Char('Extra 3 Label', required=True, default='Extra Label 3')
    topping_ids_1 = fields.One2many('lunch.topping', 'supplier_id', domain=[('topping_category', '=', 1)])
    topping_ids_2 = fields.One2many('lunch.topping', 'supplier_id', domain=[('topping_category', '=', 2)])
    topping_ids_3 = fields.One2many('lunch.topping', 'supplier_id', domain=[('topping_category', '=', 3)])
    topping_quantity_1 = fields.Selection([
        ('0_more', 'None or More'),
        ('1_more', 'One or More'),
        ('1', 'Only One')], 'Extra 1 Quantity', default='0_more', required=True)
    topping_quantity_2 = fields.Selection([
        ('0_more', 'None or More'),
        ('1_more', 'One or More'),
        ('1', 'Only One')], 'Extra 2 Quantity', default='0_more', required=True)
    topping_quantity_3 = fields.Selection([
        ('0_more', 'None or More'),
        ('1_more', 'One or More'),
        ('1', 'Only One')], 'Extra 3 Quantity', default='0_more', required=True)

    show_order_button = fields.Boolean(compute='_compute_buttons')
    show_confirm_button = fields.Boolean(compute='_compute_buttons')

    _sql_constraints = [
        ('automatic_email_time_range',
         'CHECK(automatic_email_time >= 0 AND automatic_email_time <= 12)',
         'Automatic Email Sending Time should be between 0 and 12'),
    ]

    @api.depends('phone')
    def _compute_display_name(self):
        for supplier in self:
            if supplier.phone:
                supplier.display_name = f'{supplier.name} {supplier.phone}'
            else:
                supplier.display_name = supplier.name

    def _sync_cron(self):
        for supplier in self:
            supplier = supplier.with_context(tz=supplier.tz)

            sendat_tz = pytz.timezone(supplier.tz).localize(datetime.combine(
                fields.Date.context_today(supplier),
                float_to_time(supplier.automatic_email_time, supplier.moment)))
            cron = supplier.cron_id.sudo()
            lc = cron.lastcall
            if ((
                lc and sendat_tz.date() <= fields.Datetime.context_timestamp(supplier, lc).date()
            ) or (
                not lc and sendat_tz <= fields.Datetime.context_timestamp(supplier, fields.Datetime.now())
            )):
                sendat_tz += timedelta(days=1)
            sendat_utc = sendat_tz.astimezone(pytz.UTC).replace(tzinfo=None)

            cron.active = supplier.active and supplier.send_by == 'mail'
            cron.name = f"Lunch: send automatic email to {supplier.name}"
            cron.nextcall = sendat_utc
            cron.code = dedent(f"""\
                # This cron is dynamically controlled by {self._description}.
                # Do NOT modify this cron, modify the related record instead.
                env['{self._name}'].browse([{supplier.id}])._send_auto_email()""")

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            for topping in vals.get('topping_ids_2', []):
                topping[2].update({'topping_category': 2})
            for topping in vals.get('topping_ids_3', []):
                topping[2].update({'topping_category': 3})
        crons = self.env['ir.cron'].sudo().create([
            {
                'user_id': self.env.ref('base.user_root').id,
                'active': False,
                'interval_type': 'days',
                'interval_number': 1,
                'name': "Lunch: send automatic email",
                'model_id': self.env['ir.model']._get_id(self._name),
                'state': 'code',
                'code': "",
            }
            for _ in range(len(vals_list))
        ])
        self.env['ir.model.data'].sudo().create([{
            'name': f'lunch_supplier_cron_sa_{cron.ir_actions_server_id.id}',
            'module': 'lunch',
            'res_id': cron.ir_actions_server_id.id,
            'model': 'ir.actions.server',
            # noupdate is set to true to avoid to delete record at module update
            'noupdate': True,
        } for cron in crons])
        for vals, cron in zip(vals_list, crons):
            vals['cron_id'] = cron.id

        suppliers = super().create(vals_list)
        suppliers._sync_cron()
        return suppliers

    def write(self, values):
        for topping in values.get('topping_ids_2', []):
            topping_values = topping[2] if len(topping) > 2 else False
            if topping_values:
                topping_values.update({'topping_category': 2})
        for topping in values.get('topping_ids_3', []):
            topping_values = topping[2] if len(topping) > 2 else False
            if topping_values:
                topping_values.update({'topping_category': 3})
        if values.get('company_id'):
            self.env['lunch.order'].search([('supplier_id', 'in', self.ids)]).write({'company_id': values['company_id']})
        res = super().write(values)
        if not CRON_DEPENDS.isdisjoint(values):
            # flush automatic_email_time field to call _sql_constraints
            if 'automatic_email_time' in values:
                self.flush_model(['automatic_email_time'])
            self._sync_cron()
        days_removed = [val for val in values if val in ('mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun') and not values[val]]
        if days_removed:
            self._cancel_future_days(days_removed)
        return res

    def unlink(self):
        crons = self.cron_id.sudo()
        server_actions = crons.ir_actions_server_id
        res = super().unlink()
        crons.unlink()
        server_actions.unlink()
        return res

    def toggle_active(self):
        """ Archiving related lunch product """
        res = super().toggle_active()
        active_suppliers = self.filtered(lambda s: s.active)
        inactive_suppliers = self - active_suppliers
        Product = self.env['lunch.product'].with_context(active_test=False)
        Product.search([('supplier_id', 'in', active_suppliers.ids)]).write({'active': True})
        Product.search([('supplier_id', 'in', inactive_suppliers.ids)]).write({'active': False})
        return res

    def _cancel_future_days(self, weekdays):
        weekdays_n = [WEEKDAY_TO_NAME.index(wd) for wd in weekdays]
        self.env['lunch.order'].search([
            ('supplier_id', 'in', self.ids),
            ('state', 'in', ('new', 'ordered')),
            ('date', '>=', fields.Date.context_today(self.with_context(tz=self.tz))),
        ]).filtered(lambda lo: lo.date.weekday() in weekdays_n).write({'state': 'cancelled'})

    def _get_current_orders(self, state='ordered'):
        """ Returns today's orders """
        available_today = self.filtered('available_today')
        if not available_today:
            return self.env['lunch.order']

        orders = self.env['lunch.order'].search([
            ('supplier_id', 'in', available_today.ids),
            ('state', '=', state),
            ('date', '=', fields.Date.context_today(self.with_context(tz=self.tz))),
        ], order="user_id, product_id")
        return orders

    def _send_auto_email(self):
        """ Send an email to the supplier with the order of the day """
        # Called daily by cron
        self.ensure_one()

        if not self.available_today:
            return

        if self.send_by != 'mail':
            raise UserError(_("Cannot send an email to this supplier!"))

        orders = self._get_current_orders()
        if not orders:
            return

        order = {
            'company_name': orders[0].company_id.name,
            'currency_id': orders[0].currency_id.id,
            'supplier_id': self.partner_id.id,
            'supplier_name': self.name,
            'email_from': self.responsible_id.email_formatted,
            'amount_total': sum(order.price for order in orders),
        }

        sites = orders.mapped('user_id.last_lunch_location_id').sorted(lambda x: x.name)
        orders_per_site = orders.sorted(lambda x: x.user_id.last_lunch_location_id.id)

        email_orders = [{
            'product': order.product_id.name,
            'note': order.note,
            'quantity': order.quantity,
            'price': order.price,
            'toppings': order.display_toppings,
            'username': order.user_id.name,
            'site': order.user_id.last_lunch_location_id.name,
        } for order in orders_per_site]

        email_sites = [{
            'name': site.name,
            'address': site.address,
        } for site in sites]

        self.env.ref('lunch.lunch_order_mail_supplier').with_context(
            order=order, lines=email_orders, sites=email_sites
        ).send_mail(self.id)

        orders.action_send()

    @api.depends('recurrency_end_date', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun')
    def _compute_available_today(self):
        now = fields.Datetime.now().replace(tzinfo=pytz.UTC)

        for supplier in self:
            supplier_date = now.astimezone(pytz.timezone(supplier.tz))
            supplier.available_today = supplier._available_on_date(supplier_date)

    def _available_on_date(self, date):
        self.ensure_one()

        fieldname = WEEKDAY_TO_NAME[date.weekday()]
        return not (self.recurrency_end_date and date.date() >= self.recurrency_end_date) and self[fieldname]

    @api.depends('available_today', 'automatic_email_time', 'send_by')
    def _compute_order_deadline_passed(self):
        now = fields.Datetime.now().replace(tzinfo=pytz.UTC)

        for supplier in self:
            if supplier.send_by == 'mail':
                now = now.astimezone(pytz.timezone(supplier.tz))
                email_time = pytz.timezone(supplier.tz).localize(datetime.combine(
                    fields.Date.context_today(supplier),
                    float_to_time(supplier.automatic_email_time, supplier.moment)))
                supplier.order_deadline_passed = supplier.available_today and now > email_time
            else:
                supplier.order_deadline_passed = not supplier.available_today

    def _search_available_today(self, operator, value):
        if (not operator in ['=', '!=']) or (not value in [True, False]):
            return []

        searching_for_true = (operator == '=' and value) or (operator == '!=' and not value)

        now = fields.Datetime.now().replace(tzinfo=pytz.UTC).astimezone(pytz.timezone(self.env.user.tz or 'UTC'))
        fieldname = WEEKDAY_TO_NAME[now.weekday()]

        recurrency_domain = expression.OR([
            [('recurrency_end_date', '=', False)],
            [('recurrency_end_date', '>' if searching_for_true else '<', now)]
        ])

        return expression.AND([
            recurrency_domain,
            [(fieldname, operator, value)]
        ])

    def _compute_buttons(self):
        self.env.cr.execute("""
            SELECT supplier_id, state, COUNT(*)
              FROM lunch_order
             WHERE supplier_id IN %s
               AND state in ('ordered', 'sent')
               AND date = %s
               AND active
          GROUP BY supplier_id, state
        """, (tuple(self.ids), fields.Date.context_today(self)))
        supplier_orders = defaultdict(dict)
        for order in self.env.cr.fetchall():
            supplier_orders[order[0]][order[1]] = order[2]
        for supplier in self:
            supplier.show_order_button = supplier_orders[supplier.id].get('ordered', False)
            supplier.show_confirm_button = supplier_orders[supplier.id].get('sent', False)

    def action_send_orders(self):
        no_auto_mail = self.filtered(lambda s: s.send_by != 'mail')

        for supplier in self - no_auto_mail:
            supplier._send_auto_email()
        orders = no_auto_mail._get_current_orders()
        orders.action_send()

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'message': _('The orders have been sent!'),
                'next': {'type': 'ir.actions.act_window_close'},
            }
        }

    def action_confirm_orders(self):
        orders = self._get_current_orders(state='sent')
        orders.action_confirm()

        return {
            'type': 'ir.actions.client',
            'tag': 'display_notification',
            'params': {
                'type': 'success',
                'message': _('The orders have been confirmed!'),
                'next': {'type': 'ir.actions.act_window_close'},
            }
        }

```

## File: models\lunch_topping.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models

from odoo.tools import formatLang


class LunchTopping(models.Model):
    _name = 'lunch.topping'
    _description = 'Lunch Extras'

    name = fields.Char('Name', required=True)
    company_id = fields.Many2one('res.company', default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id')
    price = fields.Monetary('Price', required=True)
    supplier_id = fields.Many2one('lunch.supplier', ondelete='cascade')
    topping_category = fields.Integer('Topping Category', required=True, default=1)

    @api.depends('price')
    @api.depends_context('company')
    def _compute_display_name(self):
        currency_id = self.env.company.currency_id
        for topping in self:
            price = formatLang(self.env, topping.price, currency_obj=currency_id)
            topping.display_name = f'{topping.name} {price}'

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class Company(models.Model):
    _inherit = 'res.company'

    lunch_minimum_threshold = fields.Float()
    lunch_notify_message = fields.Html(
        default="""Your lunch has been delivered.
Enjoy your meal!""", translate=True)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    currency_id = fields.Many2one('res.currency', related='company_id.currency_id')
    company_lunch_minimum_threshold = fields.Float(string="Maximum Allowed Overdraft", readonly=False, related='company_id.lunch_minimum_threshold')
    company_lunch_notify_message = fields.Html(string="Lunch notification message", readonly=False, related="company_id.lunch_notify_message")

```

## File: models\res_users.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResUsers(models.Model):
    _inherit = 'res.users'

    last_lunch_location_id = fields.Many2one('lunch.location')
    favorite_lunch_product_ids = fields.Many2many('lunch.product', 'lunch_product_favorite_user_rel', 'user_id', 'product_id')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import lunch_alert
from . import lunch_cashmove
from . import lunch_location
from . import lunch_order
from . import lunch_product
from . import lunch_product_category
from . import lunch_topping
from . import lunch_supplier
from . import res_company
from . import res_config_settings
from . import res_users

```

## File: report\lunch_cashmove_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools, _


class CashmoveReport(models.Model):
    _name = "lunch.cashmove.report"
    _description = 'Cashmoves report'
    _auto = False
    _order = "date desc"

    id = fields.Integer('ID')
    amount = fields.Float('Amount')
    date = fields.Date('Date')
    currency_id = fields.Many2one('res.currency', string='Currency')
    user_id = fields.Many2one('res.users', string='User')
    description = fields.Text('Description')

    def _compute_display_name(self):
        for cashmove in self:
            cashmove.display_name = '{} {}'.format(_('Lunch Cashmove'), '#%d' % cashmove.id)

    def init(self):
        tools.drop_view_if_exists(self._cr, self._table)

        self._cr.execute("""
            CREATE or REPLACE view %s as (
                SELECT
                    lc.id as id,
                    lc.amount as amount,
                    lc.date as date,
                    lc.currency_id as currency_id,
                    lc.user_id as user_id,
                    lc.description as description
                FROM lunch_cashmove lc
                UNION ALL
                SELECT
                    -lol.id as id,
                    -lol.price as amount,
                    lol.date as date,
                    lol.currency_id as currency_id,
                    lol.user_id as user_id,
                    format('Order: %%s x %%s %%s', lol.quantity::text, lp.name->>'en_US', lol.display_toppings) as description
                FROM lunch_order lol
                JOIN lunch_product lp ON lp.id = lol.product_id
                WHERE
                    lol.state in ('ordered', 'confirmed')
                    AND lol.active = True
            );
        """ % self._table)

```

## File: report\lunch_cashmove_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="lunch_cashmove_report_view_search" model="ir.ui.view">
        <field name='name'>lunch.cashmove.report.search</field>
        <field name='model'>lunch.cashmove.report</field>
        <field name='arch' type='xml'>
            <search string="lunch employee payment">
                <field name="description"/>
                <field name="user_id"/>
                <filter name='is_payment' string="Payment" domain="[('amount', '>', 0)]"/>
                <separator/>
                <filter name='is_mine_group' string="My Account grouped" domain="[('user_id','=',uid)]" context="{'group_by':'user_id'}"/>
                <filter name="group_by_user" string="By User" context="{'group_by':'user_id'}"/>
            </search>
        </field>
    </record>

    <record id="lunch_cashmove_report_view_search_2" model="ir.ui.view">
        <field name='name'>lunch.cashmove.report.search</field>
        <field name='model'>lunch.cashmove.report</field>
        <field name='arch' type='xml'>
            <search string="lunch cashmove">
                <field name="description"/>
                <field name="user_id"/>
                <group expand="0" string="Group By">
                    <filter name='group_by_user' string="By Employee" context="{'group_by':'user_id'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="lunch_cashmove_report_view_tree" model="ir.ui.view">
        <field name="name">lunch.cashmove.report.list</field>
        <field name="model">lunch.cashmove.report</field>
        <field name="arch" type="xml">
            <list string="cashmove list">
                <field name="currency_id" column_invisible="True"/>
                <field name="date"/>
                <field name="user_id" widget="many2one_avatar_user"/>
                <field name="description"/>
                <field name="amount" sum="Total" widget="monetary"/>
            </list>
        </field>
    </record>

    <record id="lunch_cashmove_report_view_tree_2" model="ir.ui.view">
        <field name="name">lunch.cashmove.report.list</field>
        <field name="model">lunch.cashmove.report</field>
        <field name="arch" type="xml">
            <list string="cashmove list" create='false'>
                <field name="currency_id" column_invisible="True"/>
                <field name="date"/>
                <field name="description"/>
                <field name="amount" sum="Total" widget="monetary"/>
            </list>
        </field>
    </record>

    <record id="lunch_cashmove_report_view_form" model="ir.ui.view">
        <field name="name">lunch.cashmove.report.form</field>
        <field name="model">lunch.cashmove.report</field>
        <field name="arch" type="xml">
            <form string="cashmove form">
                <sheet>
                    <group>
                        <field name="currency_id" invisible="1"/>
                        <field name="user_id" required="1" widget="many2one_avatar"/>
                        <field name="date"/>
                        <field name="amount" widget="monetary"/>
                    </group>
                    <label for='description'/>
                    <field name="description"/>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_lunch_cashmove_report_kanban" model="ir.ui.view">
        <field name="name">lunch.cashmove.report.kanban</field>
        <field name="model">lunch.cashmove.report</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <field name="currency_id"/>
                <templates>
                    <t t-name="card">
                        <div class="row mb4">
                            <div class="col-8 fw-bold fs-5">
                                <field name="description" />
                            </div>
                            <div class="col-4 text-end badge rounded-pill fw-bolder pe-3 pt-1">
                                <i class="fa fa-money" role="img" aria-label="Amount" title="Amount"/> <field name="amount" widget="monetary"/>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-6">
                                <i class="fa fa-clock-o" role="img" aria-label="Date" title="Date"/> <field name="date"/>
                            </div>
                            <div class="col-6">
                                <field name="user_id" widget="many2one_avatar_user" class="float-end"/>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_cashmove_report_action_account" model="ir.actions.act_window">
        <field name="name">My Account</field>
        <field name="res_model">lunch.cashmove.report</field>
        <field name="view_mode">list</field>
        <field name="search_view_id" ref="lunch_cashmove_report_view_search"/>
        <field name="domain">[('user_id','=',uid)]</field>
        <field name="view_id" ref="lunch_cashmove_report_view_tree_2"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
            No cash move yet
          </p><p>
            Here you can see your cash moves.<br/>A cash move can either be an expense or a payment.
            An expense is automatically created when an order is received while a payment is a reimbursement to the company encoded by the manager.
          </p>
        </field>
    </record>

    <record id="lunch_cashmove_report_action_control_accounts" model="ir.actions.act_window">
        <field name="name">Control Accounts</field>
        <field name="res_model">lunch.cashmove.report</field>
        <field name="view_mode">list,kanban,form</field>
        <field name="search_view_id" ref="lunch_cashmove_report_view_search_2"/>
        <field name="context">{"search_default_group_by_user":1}</field>
        <field name="view_id" ref="lunch_cashmove_report_view_tree"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Create a new payment
          </p><p>
            A cashmove can either be an expense or a payment.<br/>
            An expense is automatically created at the order receipt.<br/>
            A payment represents the employee reimbursement to the company.
          </p>
        </field>
    </record>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import lunch_cashmove_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
cashmove_user,"Cashmove user",model_lunch_cashmove,group_lunch_user,1,0,0,0
cashmove_manager,"Cashmove user",model_lunch_cashmove,group_lunch_manager,1,1,1,1
order_user,"Order Line user",model_lunch_order,group_lunch_user,1,1,1,1
order_manager,"Order Line user",model_lunch_order,group_lunch_manager,1,1,1,1
product_user,"Product user",model_lunch_product,group_lunch_user,1,0,0,0
product_manager,"Product user",model_lunch_product,group_lunch_manager,1,1,1,1
product_category_user,"Product category user",model_lunch_product_category,group_lunch_user,1,0,0,0
product_category_manager,"Product category user",model_lunch_product_category,group_lunch_manager,1,1,1,1
lunch_alert_access,access_lunch_alert_user,model_lunch_alert,base.group_user,1,0,0,0
lunch_alert_manager,access_lunch_alert_manager,model_lunch_alert,group_lunch_manager,1,1,1,1
lunch_cashmove_report_manager,access_lunch_cashmove_report,model_lunch_cashmove_report,base.group_user,1,0,0,0
lunch_location_user,access_lunch_location,model_lunch_location,group_lunch_user,1,1,0,0
lunch_location_manager,access_lunch_location,model_lunch_location,group_lunch_manager,1,1,1,1
lunch_topping_user,access_lunch_topping,model_lunch_topping,group_lunch_user,1,0,0,0
lunch_topping_manager,access_lunch_topping,model_lunch_topping,group_lunch_manager,1,1,1,1
lunch_supplier_user,"Lunch Supplier User Rights",model_lunch_supplier,group_lunch_user,1,0,0,0
lunch_supplier_manager,"Lunch Supplier Manager Rights",model_lunch_supplier,group_lunch_manager,1,1,1,1

```

## File: security\lunch_security.xml

```xml
<?xml version="1.0" ?>
<odoo>
        <record model="ir.module.category" id="module_lunch_category">
            <field name="name">Lunch</field>
            <field name="description">Helps you handle your lunch needs, if you are a manager you will be able to create new products, cashmoves and to confirm or cancel orders.</field>
            <field name="sequence">16</field>
        </record>
        <record id="group_lunch_user" model="res.groups">
            <field name="name">User : Order your meal</field>
            <field name="category_id" ref="base.module_category_human_resources_lunch"/>
        </record>
        <record id="group_lunch_manager" model="res.groups">
            <field name="name">Administrator</field>
            <field name="implied_ids" eval="[(4, ref('group_lunch_user'))]"/>
            <field name="category_id" ref="base.module_category_human_resources_lunch"/>
            <field name="users" eval="[(4, ref('base.user_root')), (4, ref('base.user_admin'))]"/>
        </record>

    <data noupdate="1">

        <record id="lunch_mind_your_own_food_money" model="ir.rule">
            <field name="name">lunch.cashmove: do not see other people's cashmove</field>
            <field name="model_id" ref="model_lunch_cashmove"/>
            <field name="groups" eval="[(4, ref('group_lunch_user'))]"/>
            <field name="domain_force">[('user_id', '=', user.id)]</field>
        </record>
        <record id="lunch_mind_other_food_money" model="ir.rule">
            <field name="name">lunch.cashmove: do see other people's cashmove</field>
            <field name="model_id" ref="model_lunch_cashmove"/>
            <field name="groups" eval="[(4, ref('group_lunch_manager'))]"/>
            <field name="domain_force">[(1, '=', 1)]</field>
        </record>
        <record id="lunch_order_rule_delete" model="ir.rule">
            <field name="name">lunch.order: Only new and cancelled order lines deleted.</field>
            <field name="model_id" ref="lunch.model_lunch_order"/>
            <field name="domain_force">[('state', 'in', ('new', 'cancelled'))]</field>
            <field name="perm_read" eval="0"/>
            <field name="perm_write" eval="0"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_unlink" eval="1" />
            <field name="groups" eval="[(4,ref('lunch.group_lunch_user'))]"/>
        </record>

        <record id="lunch_order_rule_write" model="ir.rule">
            <field name="name">lunch.order: Don't change confirmed order</field>
            <field name="model_id" ref="lunch.model_lunch_order"/>
            <field name="domain_force">[('state', '!=', 'confirmed'), ('user_id', '=', user.id)]</field>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_unlink" eval="0"/>
            <field name="groups" eval="[(4, ref('base.group_user'))]"/>
        </record>

        <record id="lunch_order_rule_write_manager" model="ir.rule">
            <field name="name">manager can do whatever</field>
            <field name="model_id" ref="lunch.model_lunch_order"/>
            <field name="domain_force">[(1, '=', 1)]</field>
            <field name="perm_read" eval="0"/>
            <field name="perm_create" eval="0"/>
            <field name="perm_unlink" eval="0"/>
            <field name="groups" eval="[(4, ref('lunch.group_lunch_manager'))]"/>
        </record>

        <record id="base.default_user" model="res.users">
            <field name="groups_id" eval="[(4,ref('lunch.group_lunch_manager'))]"/>
        </record>

        <record id="ir_rule_lunch_supplier_multi_company" model="ir.rule">
            <field name="name">Lunch supplier: Multi Company</field>
            <field name="model_id" ref="model_lunch_supplier"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>

        <record id="ir_rule_lunch_order_multi_company" model="ir.rule">
            <field name="name">Lunch order: Multi Company</field>
            <field name="model_id" ref="model_lunch_order"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>

        <record id="ir_rule_lunch_product_multi_company" model="ir.rule">
            <field name="name">Lunch product: Multi Company</field>
            <field name="model_id" ref="model_lunch_product"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>

        <record id="ir_rule_lunch_product_category_multi_company" model="ir.rule">
            <field name="name">Lunch product category: Multi Company</field>
            <field name="model_id" ref="model_lunch_product_category"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>

        <record id="ir_rule_lunch_location_multi_company" model="ir.rule">
            <field name="name">Lunch location: Multi Company</field>
            <field name="model_id" ref="model_lunch_location"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
        </record>
    </data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg width="50" height="50" viewBox="0 0 50 50" xmlns="http://www.w3.org/2000/svg"><path d="M42 25c0 9.389-7.611 17-17 17S8 34.389 8 25 15.611 8 25 8s17 7.611 17 17Z" fill="#F78613"/><path d="M19.008 40.914C12.576 38.49 8 32.28 8 25c0-9.389 7.611-17 17-17 7.28 0 13.492 4.576 15.914 11.01-2.887 10.638-11.267 19.017-21.906 21.904Z" fill="#FBB945"/><path d="M19.678 15.36c0-1.808-1.52-3.33-3.19-2.639a8.338 8.338 0 0 0-4.513 4.513c-.692 1.67.831 3.19 2.64 3.19h1.063a4 4 0 0 0 4-4v-1.063Z" fill="#fff"/><path d="M8.258 42.696c-.115-.31-.012-.748.232-.976L45.304 7.136c.243-.229.533-.162.649.149.115.311.011.748-.232.976L8.907 42.846c-.244.228-.534.161-.65-.15Z" fill="#1AD3BB"/><path d="M4.036 42.658a.892.892 0 0 1 .332-.961l38.438-25.991c.293-.198.62-.088.729.244a.892.892 0 0 1-.332.961L4.764 42.902c-.293.198-.619.089-.728-.244Z" fill="#03AF89"/></svg>

```

## File: static\src\components\lunch_dashboard.js

```javascript
/** @odoo-module */

import { rpc } from "@web/core/network/rpc";
import { user } from "@web/core/user";
import { useBus, useService } from "@web/core/utils/hooks";
import { Many2XAutocomplete } from "@web/views/fields/relational_utils";
import { DateTimeInput } from '@web/core/datetime/datetime_input';
import { Component, useState, onWillStart, markup, xml } from "@odoo/owl";

export class LunchCurrency extends Component {
    static template = "lunch.LunchCurrency";
    static props = ["currency", "amount"];

    get amount() {
        return parseFloat(this.props.amount).toFixed(2);
    }
}

export class LunchOrderLine extends Component {
    static template = "lunch.LunchOrderLine";
    static props = ["line", "currency", "onUpdateQuantity", "openOrderLine", "infos"];
    static components = {
        LunchCurrency,
    };

    setup() {
        super.setup();
        this.orm = useService('orm');
        this.state = useState({ mobileOpen: false });
    }

    get line() {
        return this.props.line;
    }

    get canAdd(){
        let price = this.line.product[3]
        this.line.toppings.forEach((line) => price += line[3])
        const unpaid = parseFloat(this.props.infos.unpaid_subtotal)
        return this.canEdit && (this.props.infos.wallet - unpaid) > price;
    }

    get canEdit() {
        return !['sent', 'confirmed'].includes(this.line.raw_state);
    }

    get badgeClass() {
        const mapping = {'new': 'warning', 'confirmed': 'success', 'sent': 'info', 'ordered': 'danger'};
        return mapping[this.line.raw_state];
    }

    get hasToppings() {
        return this.line.toppings.length !== 0;
    }

    async updateQuantity(increment) {
        await this.orm.call('lunch.order', 'update_quantity', [
            this.props.line.id,
            increment
        ]);

        await this.props.onUpdateQuantity();
    }
}

export class LunchAlert extends Component {
    static props = ["message"];
    static template = xml`<t t-out="message"/>`;
    get message() {
        return markup(this.props.message);
    }
}

export class LunchAlerts extends Component {
    static components = {
        LunchAlert,
    };
    static props = ["alerts"];
    static template = "lunch.LunchAlerts";
}

export class LunchUser extends Component {
    static components = {
        Many2XAutocomplete,
    };
    static props = ["username", "isManager", "onUpdateUser"];
    static template = "lunch.LunchUser";
    getDomain() {
        return [['share', '=', false]];
    }
}

export class LunchLocation extends Component {
    static components = {
        Many2XAutocomplete,
    };
    static props = ["location", "onUpdateLunchLocation"];
    static template = "lunch.LunchLocation";
    getDomain() {
        return [];
    }
}

export class LunchDashboard extends Component {
    static components = {
        LunchAlerts,
        LunchCurrency,
        LunchLocation,
        LunchOrderLine,
        LunchUser,
        Many2XAutocomplete,
    };
    static props = ["openOrderLine"];
    static template = "lunch.LunchDashboard";
    setup() {
        super.setup();
        this.state = useState({
            infos: {},
            date: new Date(),
        });

        useBus(this.env.bus, 'lunch_update_dashboard', () => this._fetchLunchInfos());
        onWillStart(async () => {
            await this._fetchLunchInfos()
            this.env.searchModel.updateLocationId(this.state.infos.user_location[0]);
        });
    }

    async lunchRpc(route, args = {}) {
        return await rpc(route, {
            ...args,
            context: user.context,
            user_id: this.env.searchModel.lunchState.userId,
        })
    }

    async _fetchLunchInfos() {
        this.state.infos = await this.lunchRpc('/lunch/infos');
    }

    async emptyCart() {
        await this.lunchRpc('/lunch/trash');
        await this._fetchLunchInfos();
    }

    get hasLines() {
        return this.state.infos.lines && this.state.infos.lines.length !== 0;
    }

    get canOrder() {
        return this.state.infos.raw_state === 'new';
    }

    get location() {
        return this.state.infos.user_location && this.state.infos.user_location[1];
    }

    async orderNow() {
        if (!this.canOrder) {
            return;
        }

        await this.lunchRpc('/lunch/pay');
        await this._fetchLunchInfos();
    }

    async onUpdateQuantity() {
        await this._fetchLunchInfos();
    }

    async onUpdateUser(value) {
        if (!value) {
            return;
        }
        this.env.searchModel.updateUserId(value[0].id);
        await this._fetchLunchInfos();
    }

    async onUpdateLunchLocation(value) {
        if (!value) {
            return;
        }

        await this.lunchRpc('/lunch/user_location_set', {
            location_id: value[0].id,
        });
        await this._fetchLunchInfos();
        this.env.searchModel.updateLocationId(value[0].id);
    }

    async onUpdateLunchTime(value) {
        if (value) {
            // Set time at 12:00
            this.state.date.setTime(value + 12 * 60 * 60 * 1000);
        } else {
            this.state.date.setTime(new Date());
        }
        this.env.searchModel.updateDate(this.state.date);
    }
}

LunchDashboard.components = {
    LunchAlerts,
    LunchCurrency,
    LunchLocation,
    LunchOrderLine,
    LunchUser,
    Many2XAutocomplete,
    DateTimeInput,
};
LunchDashboard.props = ["openOrderLine"];
LunchDashboard.template = 'lunch.LunchDashboard';

```

## File: static\src\components\lunch_dashboard.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="lunch.LunchCurrency">
        <div class="text-end d-flex" >
            <span class="col-4" t-if="props.currency.position == 'before'" t-esc="props.currency.symbol"/>
            <span class="col-8" t-esc="amount"/>
            <span class="col-4" t-if="props.currency.position == 'after'" t-esc="props.currency.symbol"/>
        </div>
    </t>

    <t t-name="lunch.LunchOrderLine">
        <tr>
            <td class="text-center">
                <span
                    t-if="canEdit"
                    type="button"
                    class="btn btn-sm btn-icon btn-link fa fa-minus-circle px-0"
                    t-on-click="() => this.updateQuantity(-1)"/>
                <span
                    t-else=""
                    type="button"
                    class="btn btn-sm btn-icon btn-link disabled fa fa-minus-circle px-0"/>
            </td>
            <td t-esc="line.quantity" class="text-center"/>
            <td class="text-center">
                <span
                    t-if="canAdd"
                    type="button"
                    class="btn btn-sm btn-icon btn-link fa fa-plus-circle px-0"
                    t-on-click="() => this.updateQuantity(1)"/>
                <span
                    t-else=""
                    type="button"
                    class="btn btn-sm btn-icon btn-link disabled fa fa-plus-circle px-0"/>
            </td>
            <td
                t-if="canEdit"
                t-esc="line.product[1]"
                t-on-click="() => props.openOrderLine(line.product[0], line.id)"
                role="button"
                title="Edit order"/>
            <td t-else="" t-esc="line.product[1]"/>
            <td>
                <span t-esc="line.state" t-attf-class="badge rounded-pill text-bg-#{badgeClass} border-#{badgeClass}" style="vertical-align: bottom;"/>
            </td>
            <td t-esc="line.location"/>
            <td t-esc="line.date" class="mx-4"/>
            <td>
                <LunchCurrency currency="props.currency" amount="line.product[2]"/>
            </td>
            </tr>
            <t t-if="hasToppings" t-foreach="line.toppings" t-as="topping" t-key="topping">
                <tr>
                    <td/>
                    <td/>
                    <td/>
                    <td class="lunch_topping" t-esc="topping[0]"/>
                    <td/>
                    <td/>
                    <td/>
                    <td>
                        <LunchCurrency currency="props.currency" amount="topping[1]"/>
                    </td>
                </tr>
            </t>
            <tr t-if="line.note">
                <td/>
                <td/>
                <td/>
                <td t-esc="line.note" class="text-muted"/>
            </tr>
    </t>

    <t t-name="lunch.LunchAlerts">
        <div class="alert alert-warning mb-0" t-if="props.alerts.length !== 0" role="alert">
            <t t-foreach="props.alerts" t-as="alert" t-key="alert.id">
                <LunchAlert message="alert.message" />
            </t>
        </div>
    </t>

    <t t-name="lunch.LunchUser">
        <div class="lunch_user pb-1">
            <span t-if="!props.isManager" t-esc="props.username"/>
            <Many2XAutocomplete
                t-else=""
                value="props.username"
                resModel="'res.users'"
                getDomain="getDomain"
                fieldString="props.username"
                activeActions="{}"
                update.bind="props.onUpdateUser"
            />
        </div>
    </t>

    <t t-name="lunch.LunchLocation">
        <div class="lunch_location pb-1">
            <t t-if="props.location">
                <Many2XAutocomplete
                    value="props.location"
                    resModel="'lunch.location'"
                    fieldString="props.location"
                    getDomain="getDomain"
                    activeActions="{}"
                    update.bind="props.onUpdateLunchLocation"
                />
            </t>
            <t t-else="">
                <p>No lunch location available.</p>
            </t>
        </div>
    </t>

    <t t-name="lunch.LunchDashboardOrder">
        <LunchAlerts alerts="state.infos.alerts"/>

        <div class="o_lunch_banner container-fluid mt-3 mt-md-0 py-md-4 bg-view">
            <div class="row row-gap-3 h-100">
                <div class="col-12 col-lg-3">
                    <div class="d-flex gap-2 align-content-center h-100">
                        <div>
                            <img class="o_image_64_cover rounded" t-att-src="state.infos.userimage"/>
                        </div>
                        <div class="w-100">
                            <LunchUser
                                isManager="state.infos.is_manager"
                                username="state.infos.username"
                                onUpdateUser.bind="onUpdateUser"/>

                            <LunchLocation
                                location="location"
                                onUpdateLunchLocation.bind="onUpdateLunchLocation"/>

                            <div id="lunch_order_date">
                                <DateTimeInput
                                    type="'date'"
                                    placeholder.translate="Today"
                                    onChange.bind="onUpdateLunchTime"/>
                            </div>

                            <div class="d-flex pb-1">
                                <span class="flex-grow-1">Your Account</span>
                                <span>
                                    <LunchCurrency currency="currency" amount="state.infos.wallet"/>
                                </span>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="col-12 col-lg-6" t-if="hasLines">
                    <h4 class="">
                        Your Order
                        <button
                            t-if="(['new', 'ordered'].includes(state.infos.raw_state))"
                            class="btn btn-sm btn-icon btn-link fa fa-trash"
                            t-on-click.prevent="emptyCart"/>
                    </h4>
                    <table class="o_lunch_widget_lines w-100">
                        <t t-foreach="state.infos.lines" t-as="line" t-key="line.id">
                            <LunchOrderLine line="line" currency="currency" onUpdateQuantity.bind="onUpdateQuantity" openOrderLine.bind="props.openOrderLine" infos="state.infos"/>
                        </t>
                    </table>
                </div>
                <div class="col-12 col-lg-2 offset-lg-1 d-flex flex-column row-gap-1" t-if="hasLines">
                    <span class="d-flex flex-row text-muted">
                        <span class="flex-grow-1 column-gap-1">
                            Total
                        </span>
                        <span class="col-5">
                            <LunchCurrency currency="currency" amount="state.infos.total"/>
                        </span>
                    </span>
                    <span class="d-flex column-gap-1 text-muted">
                        <span class="flex-grow-1">
                            Already Paid
                        </span>
                        <span class="col-5">
                            <LunchCurrency currency="currency" amount="state.infos.paid_subtotal"/>
                        </span>
                    </span>
                    <h4 class="d-flex column-gap-1 mt-auto">
                        <span class="flex-grow-1">
                            To Pay
                        </span>
                        <span class="col-5">
                            <LunchCurrency currency="currency" amount="state.infos.unpaid_subtotal"/>
                        </span>
                    </h4>
                    <button class="btn btn-primary" t-if="canOrder" t-on-click="orderNow">Order Now</button>
                </div>
            </div>
        </div>
    </t>

    <t t-name="lunch.LunchDashboard">
        <t t-set="currency" t-value="state.infos.currency"/>
        <t t-if="!env.isSmall">
            <t t-call="lunch.LunchDashboardOrder"/>
        </t>
        <t t-else="">
            <details class="fixed-bottom mh-100 bg-view p-2 overflow-y-auto" t-att-open="state.mobileOpen">
                <summary class="btn btn-primary w-100" t-on-click="() => state.mobileOpen = !state.mobileOpen">
                    <i class="fa fa-fw fa-shopping-cart"/>
                    Your Cart (<LunchCurrency currency="currency" amount="state.infos.total || 0"/>)
                </summary>

                <t t-call="lunch.LunchDashboardOrder"/>
            </details>
        </t>
    </t>
</templates>

```

## File: static\src\components\lunch_is_favorite_field.js

```javascript
import { registry } from "@web/core/registry";
import { booleanFavoriteField } from "@web/views/fields/boolean_favorite/boolean_favorite_field";

export const lunchIsFavoriteField = {
    ...booleanFavoriteField,
    extractProps: (fieldsInfo, dynamicInfo) => {
        return {
            ...booleanFavoriteField.extractProps(fieldsInfo, dynamicInfo),
            readonly: Boolean(fieldsInfo.attrs.readonly),
        };
    },
};

registry.category("fields").add("lunch_is_favorite", lunchIsFavoriteField);

```

## File: static\src\mixins\lunch_renderer_mixin.js

```javascript
/** @odoo-module **/

import { _t } from "@web/core/l10n/translation";
import { useBus, useService } from "@web/core/utils/hooks";

export const LunchRendererMixin = (T) => class LunchRendererMixin extends T {
    setup() {
        super.setup(...arguments);

        this.action = useService("action");
        useBus(this.env.bus, 'lunch_open_order', (ev) => this.openOrderLine(ev.detail.productId));
    }

    openOrderLine(productId, orderId) {
        let context = {};

        if (this.env.searchModel.lunchState.userId) {
            context['default_user_id'] = this.env.searchModel.lunchState.userId;
        }
        if (this.env.searchModel.lunchState.date) {
            context['default_date'] = this.env.searchModel.lunchState.date;
        }
        if (this.env.searchModel.lunchState.locationId) {
            context['default_lunch_location_id'] = this.env.searchModel.lunchState.locationId;
        }

        let action = {
            res_model: 'lunch.order',
            name: _t('Configure Your Order'),
            type: 'ir.actions.act_window',
            views: [[false, 'form']],
            target: 'new',
            context: {
                ...context,
                default_product_id: productId,
            },
        };

        if (orderId) {
            action['res_id'] = orderId;
        }

        this.action.doAction(action, {
            onClose: () => this.env.bus.trigger('lunch_update_dashboard')
        });
    }
};

```

## File: static\src\views\kanban.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';

import { kanbanView } from '@web/views/kanban/kanban_view';
import { KanbanRecord } from '@web/views/kanban/kanban_record';
import { KanbanRenderer } from '@web/views/kanban/kanban_renderer';

import { LunchDashboard } from '../components/lunch_dashboard';
import { LunchRendererMixin } from '../mixins/lunch_renderer_mixin';

import { LunchSearchModel } from './search_model';


export class LunchKanbanRecord extends KanbanRecord {
    onGlobalClick(ev) {
        this.env.bus.trigger('lunch_open_order', {productId: this.props.record.resId});
    }
}

export class LunchKanbanRenderer extends LunchRendererMixin(KanbanRenderer) {
    static template = "lunch.KanbanRenderer";
    static components = {
        ...LunchKanbanRenderer.components,
        LunchDashboard,
        KanbanRecord: LunchKanbanRecord,
    };

    getGroupsOrRecords() {
        const { locationId } = this.env.searchModel.lunchState;
        if (!locationId) {
            return [];
        } else {
            return super.getGroupsOrRecords(...arguments);
        }
    }
}

registry.category('views').add('lunch_kanban', {
    ...kanbanView,
    Renderer: LunchKanbanRenderer,
    SearchModel: LunchSearchModel,
});

```

## File: static\src\views\kanban.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="lunch.KanbanRenderer">
        <div class="o_lunch_content d-flex flex-column h-100">
            <LunchDashboard openOrderLine.bind="openOrderLine"/>

            <div class="overflow-auto border-top flex-grow-1">
                <t t-call="lunch.WebKanbanRenderer"/>
            </div>
        </div>
    </t>

    <t t-name="lunch.WebKanbanRenderer" t-inherit="web.KanbanRenderer" t-inherit-mode="primary" owl="1">
        <t t-if="showNoContentHelper" position="after">
            <t t-call="lunch.NoContentHelper"/>
        </t>
    </t>
</templates>

```

## File: static\src\views\list.js

```javascript
/** @odoo-module */

import { registry } from '@web/core/registry';

import { listView } from '@web/views/list/list_view';
import { ListRenderer } from '@web/views/list/list_renderer';

import { LunchDashboard } from '../components/lunch_dashboard';
import { LunchRendererMixin } from '../mixins/lunch_renderer_mixin';

import { LunchSearchModel } from './search_model';


export class LunchListRenderer extends LunchRendererMixin(ListRenderer) {
    static template = "lunch.ListRenderer";
    static components = {
        ...LunchListRenderer.components,
        LunchDashboard,
    };

    setup() {
        super.setup();
        const { locationId } = this.env.searchModel.lunchState;
        if (!locationId) {
            this.props.list.records = [];
        }
    }

    onCellClicked(record, column) {
        this.openOrderLine(record.resId);
    }
}


registry.category('views').add('lunch_list', {
    ...listView,
    Renderer: LunchListRenderer,
    SearchModel: LunchSearchModel,
});

```

## File: static\src\views\list.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">
    <t t-name="lunch.ListRenderer">
        <div class="o_lunch_content d-flex flex-column h-100">
            <LunchDashboard openOrderLine.bind="openOrderLine"/>

            <div class="overflow-auto flex-grow-1">
                <t t-call="lunch.WebListRenderer"/>
            </div>
        </div>
    </t>

    <t t-name="lunch.WebListRenderer" t-inherit="web.ListRenderer" t-inherit-mode="primary" owl="1">
        <t t-call="web.ActionHelper" position="after">
            <t t-call="lunch.NoContentHelper"/>
        </t>
    </t>
</templates>

```

## File: static\src\views\no_content_helper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates id="template" xml:space="preserve">

    <t t-name="lunch.NoContentHelper" owl="1">
        <t t-set="showNoLocationHelper" t-value="!this.env.searchModel.lunchState.locationId"/>

        <div t-if="showNoLocationHelper and !showNoContentHelper" class="o_view_nocontent" style="pointer-events: all">
            <div class="o_nocontent_help">
                <p class="o_view_nocontent_smiling_face">
                    No location found
                </p>
                <p>
                    Please create a location to start ordering.
                </p>
            </div>
        </div>
    </t>
</templates>

```

## File: static\src\views\search_model.js

```javascript
/** @odoo-module */


import { Domain } from '@web/core/domain';
import { rpc } from "@web/core/network/rpc";
import { SearchModel } from '@web/search/search_model';
import { useState, onWillStart } from "@odoo/owl";

export class LunchSearchModel extends SearchModel {
    setup() {
        super.setup(...arguments);

        this.lunchState = useState({
            locationId: false,
            userId: false,
            date: new Date(),
        });

        onWillStart(async () => {
            const locationId = await rpc('/lunch/user_location_get', {});
            this.updateLocationId(locationId);
        });
    }

    exportState() {
        const state = super.exportState();
        state.locationId = this.lunchState.locationId;
        state.userId = this.lunchState.userId;
        return state;
    }

    _importState(state) {
        super._importState(...arguments);

        if (state.locationId) {
            this.lunchState.locationId = state.locationId;
        }
        if (state.userId) {
            this.lunchState.userId = state.userId;
        }
    }

    updateUserId(userId) {
        this.lunchState.userId = userId;
        this._notify();
    }

    updateLocationId(locationId) {
        this.lunchState.locationId = locationId;
        this._notify();
    }

    updateDate(date) {
        this.lunchState.date.setTime(date);
        const domain_key = ['available_on_sun', 'available_on_mon', 'available_on_tue', 'available_on_wed',
        'available_on_thu', 'available_on_fri', 'available_on_sat'][this.lunchState.date.getDay()];
        const filter = Object.values(this.searchItems).find(o => o['name'] === domain_key);
        this.deactivateGroup(filter.groupId)
        this.toggleSearchItem(filter.id);
        this._notify();
    }

    _getDomain(params = {}) {
        const domain = super._getDomain(params);

        if (!this.lunchState.locationId) {
            return domain;
        }
        const result = Domain.and([
            domain,
            [['is_available_at', '=', this.lunchState.locationId]]
        ]);
        return params.raw ? result : result.toList();
    }
}

```

## File: views\lunch_alert_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="lunch_alert_view_search" model="ir.ui.view">
        <field name="name">lunch.alert.search</field>
        <field name="model">lunch.alert</field>
        <field name="arch" type="xml">
            <search string="Search">
                <field name="message"/>
                <filter name="inactive_today" string="Currently inactive" domain="[('available_today', '=', False)]"/>
                <separator/>
                <filter name="active" string="Active" domain="[('active', '=', True)]"/>
                <filter name="inactive" string="Archived" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="lunch_alert_view_tree" model="ir.ui.view">
        <field name="name">lunch.alert.list</field>
        <field name="model">lunch.alert</field>
        <field name="arch" type="xml">
            <list>
                <field name="name"/>
                <field name="mode"/>
                <field name="message" column_invisible="True"/>
                <field name="available_today"/>
                <field name="active" widget="boolean_toggle"/>
            </list>
        </field>
    </record>

    <record id="lunch_alert_view_form" model="ir.ui.view">
        <field name="name">lunch.alert.form</field>
        <field name="model">lunch.alert</field>
        <field name="arch" type="xml">
            <form string="alert form">
                <sheet>
                    <div class="oe_title" name="title">
                        <label for="name"/>
                        <h1>
                            <field name="name" placeholder="e.g. Order before 11am"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="mode" widget="radio"/>
                            <field name="recipients" invisible="mode != 'chat'" widget="radio"/>
                            <field name="location_ids" widget="many2many_tags" required="1"/>
                            <field name="until"/>
                            <field name="active" widget="boolean_toggle"/>
                        </group>
                        <group>
                            <div class="o_td_label">
                                <label for="notification_time" invisible="mode != 'chat'"/>
                            </div>
                            <div class="o_col">
                                <widget name="week_days"/>
                                <div class="o_row" invisible="mode != 'chat'">
                                    <field name="notification_time" required="mode == 'chat'" widget="float_time"/>
                                    <field name="notification_moment"/>
                                </div>
                            </div>
                            <field name="tz" groups="base.group_no_one"/>
                        </group>
                        <group>
                            <field name="message"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="lunch_alert_view_kanban" model="ir.ui.view">
        <field name="name">lunch.alert.kanban</field>
        <field name="model">lunch.alert</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <templates>
                    <t t-name="card">
                        <field class="fw-bold fs-5" name="name"/>
                        <div>
                            <field name="mode"/>
                            <span invisible="mode != 'chat'">
                                to <field name="recipients"/>
                                on <field name="notification_time"/>
                                <field name="notification_moment"/>
                            </span>
                        </div>
                        <field name="location_ids" widget="many2many_tags"/>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_alert_action" model="ir.actions.act_window">
        <field name="name">Lunch Alerts</field>
        <field name="res_model">lunch.alert</field>
        <field name="search_view_id" ref="lunch_alert_view_search"/>
        <field name="view_mode">list,form,kanban</field>
        <field name="domain">['|', ('active', '=', True), ('active', '=', False)]</field>
        <field name="context">{}</field>
        <field name="view_id" ref="lunch_alert_view_tree"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create new lunch alerts
            </p>
        </field>
    </record>
</odoo>

```

## File: views\lunch_cashmove_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="lunch_cashmove_view_search" model="ir.ui.view">
        <field name='name'>lunch.cashmove.search</field>
        <field name='model'>lunch.cashmove</field>
        <field name='arch' type='xml'>
            <search string="lunch employee payment">
                <field name="description"/>
                <field name="user_id"/>
                <filter name='is_mine_group' string="My Account grouped" domain="[('user_id','=',uid)]" context="{'group_by':'user_id'}"/>
                <filter name="group_by_user" string="By User" context="{'group_by':'user_id'}"/>
            </search>
        </field>
    </record>

    <record id="lunch_cashmove_view_tree" model="ir.ui.view">
        <field name="name">lunch.cashmove.list</field>
        <field name="model">lunch.cashmove</field>
        <field name="arch" type="xml">
            <list string="cashmove list">
                <field name="currency_id" column_invisible="True"/>
                <field name="date"/>
                <field name="user_id"/>
                <field name="description"/>
                <field name="amount" sum="Total" widget="monetary"/>
            </list>
        </field>
    </record>

    <record id="lunch_cashmove_view_form" model="ir.ui.view">
        <field name="name">lunch.cashmove.form</field>
        <field name="model">lunch.cashmove</field>
        <field name="arch" type="xml">
            <form string="cashmove form">
                <sheet>
                    <group>
                        <field name="currency_id" invisible="1"/>
                        <field name="user_id" required="1"/>
                        <field name="date"/>
                        <field name="amount" widget="monetary"/>
                    </group>
                    <label for='description'/>
                    <field name="description"/>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_lunch_cashmove_kanban" model="ir.ui.view">
        <field name="name">lunch.cashmove.kanban</field>
        <field name="model">lunch.cashmove</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <field name="currency_id"/>
                <templates>
                    <t t-name="card">
                        <div class="row mb4">
                            <div class="col-8 fw-bold fs-5">
                                <field name="description" />
                            </div>
                            <div class="col-4 text-end badge rounded-pill fw-bolder pe-3 pt-1">
                                <i class="fa fa-money" role="img" aria-label="Amount" title="Amount"/> <field name="amount" widget="monetary"/>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-6">
                                <i class="fa fa-clock-o" role="img" aria-label="Date" title="Date"/> <field name="date" />
                            </div>
                            <div class="col-6">
                                <field name="user_id" widget="many2one_avatar_user" class="float-end"/>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_cashmove_action_payment" model="ir.actions.act_window">
        <field name="name">Cash Moves</field>
        <field name="res_model">lunch.cashmove</field>
        <field name="view_mode">list,kanban,form</field>
        <field name="search_view_id" ref="lunch_cashmove_view_search"/>
        <field name="view_id" ref="lunch_cashmove_view_tree"/>
        <field name="help" type="html">
          <p class="o_view_nocontent_smiling_face">
            Register a payment
          </p><p>
            Payments are used to register liquidity movements. You can process those payments by your own means or by using installed facilities.
          </p>
        </field>
    </record>
</odoo>

```

## File: views\lunch_location_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="lunch_location_view_search" model="ir.ui.view">
        <field name="name">lunch.location.view.search</field>
        <field name="model">lunch.location</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <field name="address"/>
            </search>
        </field>
    </record>

    <record id="lunch_location_form_view" model="ir.ui.view">
        <field name="name">lunch.location.view.form</field>
        <field name="model">lunch.location</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="address"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="lunch_location_tree_view" model="ir.ui.view">
        <field name="name">lunch.location.view.form</field>
        <field name="model">lunch.location</field>
        <field name="arch" type="xml">
            <list editable="bottom">
                <field name="name"/>
                <field name="address"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </list>
        </field>
    </record>

    <record id="lunch_location_kanban_view" model="ir.ui.view">
        <field name="name">lunch.location.view.kanban</field>
        <field name="model">lunch.location</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <templates>
                    <t t-name="card">
                        <field name="name" class="fw-bold fs-5"/>
                        <field name="company_id" groups="base.group_multi_company"/>
                        <field name="address"/>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_location_action" model="ir.actions.act_window">
        <field name="name">Lunch Locations</field>
        <field name="res_model">lunch.location</field>
        <field name="view_mode">list,form,kanban</field>
        <field name="search_view_id" ref="lunch_location_view_search"/>
        <field name="help" type="html">
            <!-- TODO: better help message -->
            <p class="o_view_nocontent_smiling_face">
                To see some locations, create one using the create button
            </p>
        </field>
    </record>
</odoo>

```

## File: views\lunch_orders_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="lunch_order_view_search" model="ir.ui.view">
        <field name="name">lunch.order.search</field>
        <field name="model">lunch.order</field>
        <field name="arch" type="xml">
            <search string="Search">
                <field name="name" string="Product" filter_domain="['|', ('name', 'ilike', self), ('note', 'ilike', self)]"/>
                <field name="user_id"/>
                <filter name='is_mine' string="My Orders" domain="[('user_id', '=', uid)]"/>
                <separator/>
                <filter name="not_confirmed" string="Not Received" domain="[('state', '!=', ('confirmed'))]"/>
                <filter name="confirmed" string="Received" domain="[('state', '=', 'confirmed')]"/>
                <filter name="cancelled" string="Cancelled" domain="[('state', '=', 'cancelled')]"/>
                <separator/>
                <filter name="date_filter" string="Today" domain="[('date', '=', context_today().strftime('%Y-%m-%d'))]" />
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
                <group expand="0" string="Group By">
                    <filter name="group_by_user" string="User" context="{'group_by': 'user_id'}"/>
                    <filter name="group_by_supplier" string="Vendor" context="{'group_by': 'supplier_id'}"/>
                    <filter name="group_by_date" string="Order Date" context="{'group_by': 'date:day'}" help="Vendor Orders by Date"/>
                </group>
            </search>
        </field>
    </record>

    <record id="lunch_order_view_tree" model="ir.ui.view">
        <field name="name">lunch.order.list</field>
        <field name="model">lunch.order</field>
        <field name="arch" type="xml">
            <list string="Order lines List" create="false" edit="false" decoration-muted="state == 'cancelled'" expand="1">
                <header>
                    <button name="action_confirm" type="object" string="Receive"/>
                </header>
                <field name='date' readonly="state != 'new'"/>
                <field name='supplier_id'/>
                <field name='product_id'/>
                <field name="display_toppings" class="o_text_overflow"/>
                <field name='note' class="o_text_overflow"/>
                <field name='user_id' widget='many2one_avatar_user' readonly="state != 'new'"/>
                <field name="lunch_location_id"/>
                <field name="currency_id" column_invisible="True"/>
                <field name='price' sum="Total" string="Price" widget="monetary"/>
                <field name='state' widget="badge" decoration-warning="state == 'new'" decoration-success="state == 'confirmed'" decoration-info="state == 'sent'" decoration-danger="state == 'ordered'"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="display_reorder_button" column_invisible="True"/>
                <field name="notified" column_invisible="True"/>
                <button name="action_reorder" string="Re-order" type="object" icon="fa-history" invisible="not display_reorder_button or not display_add_button" groups="lunch.group_lunch_user"/>
                <button name="action_confirm" string="Confirm" type="object" icon="fa-check" invisible="state != 'sent'" groups="lunch.group_lunch_manager"/>
                <button name="action_cancel" string="Cancel" type="object" icon="fa-times" invisible="state in ['cancelled', 'confirmed']" groups="lunch.group_lunch_manager"/>
                <button name="action_reset" string="Reset" type="object" icon="fa-undo" invisible="state != 'cancelled'" groups="lunch.group_lunch_manager"/>
                <button name="action_notify" string="Send Notification" type="object" icon="fa-envelope" invisible="state != 'confirmed' or notified" groups="lunch.group_lunch_manager"/>
                <groupby name="supplier_id">
                    <field name="show_order_button" invisible="1" />
                    <field name="show_confirm_button" invisible="1" />
                    <button string="Send Orders" type="object" name="action_send_orders" invisible="not show_order_button"/>
                    <button string="Confirm Orders" type="object" name="action_confirm_orders" invisible="not show_confirm_button"/>
                </groupby>
            </list>
        </field>
    </record>

    <record id='lunch_order_view_kanban' model='ir.ui.view'>
        <field name="name">lunch.order.kanban</field>
        <field name="model">lunch.order</field>
        <field name="arch" type="xml">
            <kanban create="false" edit="false">
                <field name="currency_id"/>
                <field name="notified"/>
                <templates>
                    <t t-name="card">
                        <div class="d-flex">
                            <field name="product_id" class="fw-bold fs-5"/>
                            <field name="state" widget="label_selection" options="{'classes': {'new': 'default', 'confirmed': 'success', 'cancelled':'danger'}}" class="ms-auto"/>
                        </div>
                        <field name="note"/>
                        <div class="row">
                            <div class="col-6">
                                <i class="fa fa-money" role="img" aria-label="Money" title="Money"/> <field name="price"/>
                            </div>
                            <div class="col-6 text-end">
                                <i class="fa fa-clock-o" role="img" aria-label="Date" title="Date"/> <field name="date" readonly="state != 'new'"/>
                            </div>
                        </div>
                        <div class="row mt4">
                            <div class="col-6">
                                <a class="btn btn-sm btn-success" role="button" name="action_order" string="Order" type="object" invisible="state in ['sent', 'ordered', 'confirmed']" groups="lunch.group_lunch_manager">
                                    <i class="fa fa-phone" role="img" aria-label="Order button" title="Order button"/>
                                </a>
                                <a class="btn btn-sm btn-primary" role="button" name="action_send" string="Send" type="object" invisible="state != 'ordered'" groups="lunch.group_lunch_manager">
                                    <i class="fa fa-paper-plane" role="img" aria-label="Send button" title="Send button"/>
                                </a>
                                <a class="btn btn-sm btn-info" role="button" name="action_confirm" string="Receive" type="object" invisible="state != 'sent'" groups="lunch.group_lunch_manager">
                                    <i class="fa fa-check" role="img" aria-label="Receive button" title="Receive button"/>
                                </a>
                                <a class="btn btn-sm btn-danger" role="button" name="action_cancel" string="Cancel" type="object" invisible="state in ['cancelled', 'confirmed']" groups="lunch.group_lunch_manager">
                                    <i class="fa fa-times" role="img" aria-label="Cancel button" title="Cancel button"/>
                                </a>
                                <a class="btn btn-sm btn-info" role="button" name="action_notify" string="Send Notification" type="object" invisible="state != 'confirmed' or notified" groups="lunch.group_lunch_manager">
                                    <i class="fa fa-envelope" role="img" aria-label="Send notification" title="Send notification"/>
                                </a>
                            </div>
                            <div class="col-6">
                                <field name="user_id" widget="many2one_avatar_user" readonly="state != 'new'" class="float-end"/>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_order_view_pivot" model="ir.ui.view">
        <field name="name">lunch.order.pivot</field>
        <field name="model">lunch.order</field>
        <field name="arch" type="xml">
            <pivot sample="1">
                <field name="date" type="col"/>
                <field name="supplier_id" type="row"/>
            </pivot>
        </field>
    </record>

    <record id="lunch_order_view_graph" model="ir.ui.view">
        <field name="name">lunch.order.graph</field>
        <field name="model">lunch.order</field>
        <field name="arch" type="xml">
            <graph sample="1">
                <field name="product_id"/>
            </graph>
        </field>
    </record>

    <record id="lunch_order_action" model="ir.actions.act_window">
        <field name="name">My Orders</field>
        <field name="res_model">lunch.order</field>
        <field name="view_mode">list,kanban,pivot</field>
        <field name="search_view_id" ref="lunch_order_view_search"/>
        <field name="context">{"search_default_is_mine":1, "search_default_group_by_date": 1, 'show_reorder_button': True}</field>
        <field name="help" type="html">
        <p class="o_view_nocontent_empty_folder">
            No previous order found
        </p><p>
            There is no previous order recorded. Click on "My Lunch" and then create a new lunch order.
        </p>
        </field>
    </record>

    <record id="lunch_order_action_by_supplier" model="ir.actions.act_window">
        <field name="name">Today's Orders</field>
        <field name="res_model">lunch.order</field>
        <field name="view_mode">list,kanban</field>
        <field name="search_view_id" ref="lunch_order_view_search"/>
        <field name="context">{"search_default_group_by_supplier":1, "search_default_date_filter":1}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
            Nothing to order today
          </p><p>
            Here you can see today's orders grouped by vendors.
          </p>
        </field>
    </record>

    <record id="lunch_order_action_control_suppliers" model="ir.actions.act_window">
        <field name="name">Control Vendors</field>
        <field name="res_model">lunch.order</field>
        <field name="view_mode">list,kanban,pivot</field>
        <field name="search_view_id" ref="lunch_order_view_search"/>
        <field name="context">{"search_default_group_by_supplier":1}</field>
        <field name="help" type="html">
          <p class="o_view_nocontent_empty_folder">
            No lunch order yet
          </p><p>
            Summary of all lunch orders, grouped by vendor and by date.
          </p><p>
            Click on the <span class="fa fa-phone text-success" role="img" aria-label="Order button" title="Order button"/> to announce that the order is ordered.<br/>
            Click on the <span class="fa fa-check text-success" role="img" aria-label="Receive button" title="Receive button"/> to announce that the order is received.<br/>
            Click on the <span class="fa fa-times-circle text-danger" role="img" aria-label="Cancel button" title="Cancel button"/> red X to announce that the order isn't available.
          </p>
        </field>
    </record>

    <record id="lunch_order_view_form" model="ir.ui.view">
        <field name="name">lunch.order.view.form</field>
        <field name="model">lunch.order</field>
        <field name="arch" type="xml">
            <form class="flex-column">
                <field name="company_id" invisible="1"/>
                <field name="date" invisible="1" readonly="state != 'new'"/>
                <field name="currency_id" invisible="1"/>
                <field name="quantity" invisible="1"/>
                <field name="product_id" invisible="1"/>
                <field name="state" invisible="1"/>
                <field name="category_id" invisible="1"/>
                <field name="available_toppings_1" invisible="1"/>
                <field name="available_toppings_2" invisible="1"/>
                <field name="available_toppings_3" invisible="1"/>
                <field name="supplier_id" invisible="1"/>
                <field name="order_deadline_passed" invisible="1"/>
                <field name="available_today" invisible="1"/>
                <div class="d-flex">
                    <div class="flex-grow-0 pe-5">
                        <field name="image_1920" widget="image" class="o_lunch_image" options="{'image_preview': 'image_128'}"/>
                    </div>
                    <div class="flex-grow-1 pe-5">
                        <h2><field name="name"/></h2>
                        <h3 class="pt-3"><field name="price"/></h3>
                    </div>
                </div>
                <div class="o_lunch_wizard">
                    <div class="row py-3 py-md-0">
                        <div class="o_td_label col-3 col-md-2">
                            <field name="topping_label_1" nolabel="1" invisible="not available_toppings_1" class="o_form_label"/>
                        </div>
                        <div class="col-9 col-md-10">
                            <field name="topping_ids_1" invisible="not available_toppings_1" widget="many2many_checkboxes" nolabel="1" domain="[('topping_category', '=', 1), ('supplier_id', '=', supplier_id)]" class="o_field_widget o_quick_editable"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="o_td_label col-3 col-md-2">
                            <field name="topping_label_2" nolabel="1" invisible="not available_toppings_2" class="o_form_label"/>
                        </div>
                        <div class="col-9 col-md-10">
                            <field name="topping_ids_2" invisible="not available_toppings_2" widget="many2many_checkboxes" nolabel="1" domain="[('topping_category', '=', 2), ('supplier_id', '=', supplier_id)]" class="o_field_widget o_quick_editable"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="o_td_label col-3 col-md-2">
                            <field name="topping_label_3" nolabel="1" invisible="not available_toppings_3" class="o_form_label"/>
                        </div>
                        <div class="col-9 col-md-10">
                            <field name="topping_ids_3" invisible="not available_toppings_3" widget="many2many_checkboxes" nolabel="1" domain="[('topping_category', '=', 3), ('supplier_id', '=', supplier_id)]" class="o_field_widget o_quick_editable"/>
                        </div>
                    </div>
                    <div class="row pb-2">
                        <div class="o_td_label col-3 col-md-2">
                            <label for="product_description" class="o_form_label"/>
                        </div>
                        <div class="col-9 col-md-10">
                            <field name="product_description" nolabel="1" class="o_field_widget o_quick_editable"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="o_td_label col-3 col-md-2">
                            <label for="note" class="o_form_label" />
                        </div>
                        <div class="col-9 col-md-10">
                            <field name="note" nolabel="1" placeholder="Information, allergens, ..." class="o_field_widget o_quick_editable"/>
                        </div>
                    </div>

                    <div class="row" invisible="not order_deadline_passed">
                        <div class="col-12">
                            <div class="alert alert-warning" role="alert">
                                The orders for this vendor have already been sent.
                            </div>
                        </div>
                    </div>
                    <div class="row" invisible="display_add_button">
                        <div class="col-12">
                            <div class="alert alert-warning" role="alert">
                                Your wallet does not contain enough money to order that. To add some money to your wallet, please contact your lunch manager.
                            </div>
                        </div>
                    </div>
                </div>
                <footer>
                    <button string="Add To Cart" name="add_to_cart" type="object" class="oe_highlight" invisible="order_deadline_passed or not display_add_button" data-hotkey="w"/>
                    <button string="Discard" special="cancel" data-hotkey="x"/>
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: views\lunch_product_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="lunch_product_view_search" model="ir.ui.view">
        <field name="name">lunch.product.search</field>
        <field name="model">lunch.product</field>
        <field name="arch" type="xml">
            <search string="Product Search">
                <field name="name" string="Product"/>
                <field name="category_id" string="Category"/>
                <field name="supplier_id"/>
                <field name="description"/>
                <separator/>
                <filter name="available_today" string="Available Today" domain="[('supplier_id.available_today', '=', True)]"/>
                <separator/>
                <filter name="available_on_mon" string="Monday" domain="[('supplier_id.mon', '=', True)]"/>
                <filter name="available_on_tue" string="Tuesday" domain="[('supplier_id.tue', '=', True)]"/>
                <filter name="available_on_wed" string="Wednesday" domain="[('supplier_id.wed', '=', True)]"/>
                <filter name="available_on_thu" string="Thursday" domain="[('supplier_id.thu', '=', True)]"/>
                <filter name="available_on_fri" string="Friday" domain="[('supplier_id.fri', '=', True)]"/>
                <filter name="available_on_sat" string="Saturday" domain="[('supplier_id.sat', '=', True)]"/>
                <filter name="available_on_sun" string="Sunday" domain="[('supplier_id.sun', '=', True)]"/>
                <separator/>
                <filter name="inactive" string="Archived" domain="[('active', '=', False)]"/>
                <group expand="0" string="Group By">
                    <filter name="group_by_supplier" string="Vendor" context="{'group_by': 'supplier_id'}"/>
                    <filter name="group_by_category" string="Category" context="{'group_by': 'category_id'}"/>
                </group>
                <searchpanel>
                    <field name="category_id" select="multi" string="Categories" icon="fa-cutlery" color="#875A7B" enable_counters="1"/>
                    <field name="supplier_id" select="multi" string="Vendors" icon="fa-truck" enable_counters="1"/>
                </searchpanel>
            </search>
        </field>
    </record>

    <record id="lunch_product_view_tree" model="ir.ui.view">
        <field name="name">lunch.product.list</field>
        <field name="model">lunch.product</field>
        <field name="arch" type="xml">
            <list string="Products List">
                <field name="currency_id" column_invisible="True"/>
                <field name="name"/>
                <field name="category_id"/>
                <field name="supplier_id"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="description"/>
                <field name="price" widget="monetary"/>
            </list>
        </field>
    </record>

    <record id="lunch_product_view_tree_order" model="ir.ui.view">
        <field name="name">lunch.product.list.order</field>
        <field name="inherit_id" ref="lunch_product_view_tree"/>
        <field name="model">lunch.product</field>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//list" position="attributes">
                <attribute name="js_class">lunch_list</attribute>
                <attribute name="create">0</attribute>
            </xpath>
        </field>
    </record>

    <record id="lunch_product_view_form" model="ir.ui.view">
        <field name="name">lunch.product.form</field>
        <field name="model">lunch.product</field>
        <field name="arch" type="xml">
            <form string="Products Form">
                <field name="company_id" invisible="1"/>
                <field name="currency_id" invisible="1"/>
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                    <div class="oe_title">
                        <label for="name" class="oe-edit-only"/>
                        <h1>
                            <field name='name'/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name='category_id'/>
                            <field name='supplier_id'/>
                            <field name='price' widget="monetary"/>
                        </group>
                        <group>
                            <field name="new_until"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </group>
                        <label for="description"/>
                        <field name='description'/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_lunch_product_kanban_order" model="ir.ui.view">
        <field name="name">lunch.product.kanban</field>
        <field name="model">lunch.product</field>
        <field name="priority">999</field>
        <field name="arch" type="xml">
            <kanban js_class="lunch_kanban" create="0" edit="0" group_create="0" class="o_kanban_mobile">
                <field name="currency_id"/>
                <field name="is_new"/>
                <templates>
                    <t t-name="card" class="row g-0">
                        <aside class="col-4">
                            <field name="image_128" options="{'placeholder': '/lunch/static/img/lunch.png', 'size': [94, 94], 'img_class': 'w-100 h-100'}" widget="image"/>
                        </aside>
                        <main class="col">
                            <div class="d-flex">
                                <div class="d-flex">
                                    <field class="pe-1 pt-1" name="is_favorite" widget="lunch_is_favorite" nolabel="1"/>
                                    <field name="name" class="fw-bolder fs-5" />
                                </div>
                                <div class="text-primary ms-auto">
                                    <div t-if="record.is_new.raw_value" class="o_lunch_new_product me-1 py-1 fs-6 badge rounded-pill text-bg-success">
                                        New
                                    </div>
                                    <field name="price" widget="monetary" class="fw-bold "/>
                                </div>
                            </div>
                            <field name="supplier_id" />
                            <footer class="pt-0 mt-0">
                                <field name="description" class="text-muted"/>
                            </footer>
                        </main>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="view_lunch_product_kanban" model="ir.ui.view">
        <field name="name">lunch.product.kanban</field>
        <field name="model">lunch.product</field>
        <field name="priority">5</field>
        <field name="arch" type="xml">
            <kanban create="1" edit="0" class="o_kanban_mobile">
                <field name="id"/>
                <field name="name"/>
                <field name="category_id"/>
                <field name="supplier_id"/>
                <field name="description"/>
                <field name="currency_id"/>
                <templates>
                    <t t-name="card" class="flex-row">
                        <aside class="o_kanban_aside_full d-none d-md-block">
                            <field name="image_128" widget="image" alt="Product Image"/>
                        </aside>
                        <main>
                            <div class="d-flex">
                                <field name="name" class="fw-bold fs-5"/>
                                <field name="price" widget="monetary" class="fw-bold ms-auto"/>
                            </div>
                            <field name="supplier_id" />
                            <footer class="pt-0 mt-0">
                                <field name="description" class="text-muted"/>
                            </footer>
                        </main>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_product_action_statbutton" model="ir.actions.act_window">
        <field name="name">Products</field>
        <field name="res_model">lunch.product</field>
        <field name="view_mode">kanban,list,form</field>
        <field name="context">{'search_default_group_by_supplier': 1}</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            Create a new product for lunch
            </p><p>
            A product is defined by its name, category, price and vendor.
            </p>
        </field>
    </record>

    <record id="lunch_product_category_view_tree" model="ir.ui.view">
        <field name="name">Product category List</field>
        <field name="model">lunch.product.category</field>
        <field name="arch" type="xml">
            <list string="Products List">
                <field name='name' string="Product Category"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </list>
        </field>
    </record>

    <record id="lunch_product_category_view_form" model="ir.ui.view">
        <field name="name">Product category Form</field>
        <field name="model">lunch.product.category</field>
        <field name="arch" type="xml">
            <form string="Product Categories Form">
                <sheet>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button" type="action" name="%(lunch.lunch_product_action_statbutton)d"
                            context="{'search_default_category_id': id,'default_category_id': id}"
                            invisible="product_count == 0"
                            icon="fa-cutlery">
                            <field string="Products" name="product_count" widget="statinfo"/>
                        </button>
                    </div>
                    <field name="image_1920" widget="image" class="oe_avatar" options="{'preview_image': 'image_128'}"/>
                    <div class="oe_title">
                        <label for="name" class="oe-edit-only"/>
                        <h1>
                            <field name="name"/>
                        </h1>
                    </div>
                    <label for="company_id" groups="base.group_multi_company"/>
                    <field name="company_id" groups="base.group_multi_company"/>
                </sheet>
            </form>
        </field>
    </record>

     <record id="lunch_product_category_view_kanban" model="ir.ui.view">
        <field name="name">Product category Kanban</field>
        <field name="model">lunch.product.category</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <templates>
                    <t t-name="card">
                        <aside class="d-none d-md-block">
                            <field name="image_128" widget="image"/>
                        </aside>
                        <main>
                            <button class="badge text-bg-primary ms-auto" type="action"
                                name="%(lunch.lunch_product_action_statbutton)d"
                                context="{'search_default_category_id': id,'default_category_id': id}"
                                invisible="product_count == 0">
                                <field string="Products" name="product_count" widget="statinfo"/>
                            </button>
                            <field name="name" class="fw-bold"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                        </main>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_product_category_view_search" model="ir.ui.view">
        <field name="name">lunch.product.category.search</field>
        <field name="model">lunch.product.category</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="lunch_product_action" model="ir.actions.act_window">
        <field name="name">Products</field>
        <field name="res_model">lunch.product</field>
        <field name="view_mode">list,kanban,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            Create a new product for lunch
            </p><p>
            A product is defined by its name, category, price and vendor.
            </p>
        </field>
    </record>

    <record id="lunch_product_action_order" model="ir.actions.act_window">
        <field name="name">Order Your Lunch</field>
        <field name="path">lunch</field>
        <field name="res_model">lunch.product</field>
        <field name="view_mode">kanban,list</field>
        <field name="view_ids" eval="[
            (5, 0, 0),
            (0, 0, {'view_mode': 'kanban', 'view_id': ref('view_lunch_product_kanban_order')}),
            (0, 0, {'view_mode': 'list', 'view_id': ref('lunch_product_view_tree_order')})
        ]"/>
        <field name="search_view_id" ref="lunch_product_view_search"/>
        <field name="domain">[]</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            There is no product available today
            </p><p>
            To see some products, check if your vendors are available today and that you have configured some products
            </p>
        </field>
    </record>

    <record id="lunch_product_category_action" model="ir.actions.act_window">
        <field name="name">Product Categories</field>
        <field name="res_model">lunch.product.category</field>
        <field name="view_mode">list,form,kanban</field>
        <field name="search_view_id" ref="lunch_product_category_view_search"/>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
            Create a new product category
            </p><p>
            Here you can access all categories for the lunch products.
            </p>
        </field>
    </record>
</odoo>

```

## File: views\lunch_supplier_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="lunch_supplier_view_tree" model="ir.ui.view">
        <field name="name">lunch.supplier.view.list</field>
        <field name="model">lunch.supplier</field>
        <field name="arch" type="xml">
            <list>
                <field name="name"/>
                <field name="phone" class="o_force_ltr"/>
                <field name="email"/>
            </list>
        </field>
    </record>

    <record id="lunch_supplier_view_form" model="ir.ui.view">
        <field name="name">lunch.supplier.view.form</field>
        <field name="model">lunch.supplier</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_title">
                        <label for="name" string="Vendor"/>
                        <h1><field name="name" required="1" placeholder="e.g. The Pizzeria Inn"/></h1>
                    </div>
                    <group>
                        <group>
                            <field name="partner_id" context="{'default_is_company': True, 'default_city': city, 'default_name': name, 'default_street': street, 'default_street2': street2, 'default_state_id': state_id, 'default_zip': zip_code, 'default_country_id': country_id, 'default_phone': phone}"/>
                            <label for="street" string="Address"/>
                            <div class="o_address_format">
                                <field name="street" placeholder="Street..." class="o_address_street"/>
                                <field name="street2" placeholder="Street 2..." class="o_address_street"/>
                                <field name="city" placeholder="City" class="o_address_city"/>
                                <field name="state_id" class="o_address_state" placeholder="State" options="{'no_open': True}" context="{'country_id': country_id, 'zip': zip_code}"/>
                                <field name="zip_code" placeholder="ZIP" class="o_address_zip"/>
                                <field name="country_id" placeholder="Country" class="o_address_country" options="{'no_open': True, 'no_create': True}"/>
                            </div>
                        </group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="email" required="send_by == 'mail'"/>
                            <field name="phone" class="o_force_ltr" required="send_by == 'phone'"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="responsible_id" required="send_by == 'mail'" groups="base.group_no_one" domain="[('share', '=', False)]"/>
                        </group>
                    </group>
                    <group>
                        <group string="Availability">
                            <field name="tz" groups="base.group_no_one"/>
                            <label for="sun" class="d-none"/>
                            <field name="sun" invisible="1"/>
                            <widget name="week_days"/>
                            <field name="recurrency_end_date" groups="base.group_no_one"/>
                        </group>
                        <group string="Orders">
                            <field name="delivery"/>
                            <field name="available_location_ids" widget="many2many_tags"/>
                            <field name="send_by" widget="radio"/>
                            <label for="automatic_email_time" invisible="send_by != 'mail'"/>
                            <div class="o_row" invisible="send_by != 'mail'"><field name="automatic_email_time" widget="float_time"/> <field name="moment"/></div>
                        </group>
                    </group>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="topping_label_1"/>
                            <field name="topping_quantity_1" class="w-50"/>
                        </group>
                        <div>
                            <field name="topping_ids_1" nolabel="1">
                                <list editable="bottom">
                                    <field name="name"/>
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="currency_id" column_invisible="True"/>
                                    <field name="price" widget="monetary"/>
                                </list>
                            </field>
                        </div>
                        <group>
                            <field name="topping_label_2"/>
                            <field name="topping_quantity_2" class="w-50"/>
                        </group>
                        <div>
                            <field name="topping_ids_2" nolabel="1">
                                <list editable="bottom">
                                    <field name="name"/>
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="currency_id" column_invisible="True"/>
                                    <field name="price" widget="monetary"/>
                                </list>
                            </field>
                        </div>
                        <group>
                            <field name="topping_label_3"/>
                            <field name="topping_quantity_3" class="w-50"/>
                        </group>
                        <div>
                            <field name="topping_ids_3" nolabel="1">
                                <list editable="bottom">
                                    <field name="name"/>
                                    <field name="company_id" column_invisible="True"/>
                                    <field name="currency_id" column_invisible="True"/>
                                    <field name="price" widget="monetary"/>
                                </list>
                            </field>
                        </div>
                    </group>
                </sheet>
                <chatter/>
            </form>
        </field>
    </record>

    <record id="lunch_supplier_view_kanban" model="ir.ui.view">
        <field name="name">lunch.supplier.view.kanban</field>
        <field name="model">lunch.supplier</field>
        <field name="arch" type="xml">
            <kanban>
                <templates>
                    <t t-name="card">
                        <main>
                            <field name="display_name" class="fw-bold fs-5"/>
                            <field t-if="record.city.raw_value and !record.country_id.raw_value" name="city"/>
                            <field t-if="!record.city.raw_value and record.country_id.raw_value" name="country_id"/>
                            <div t-if="record.city.raw_value and record.country_id.raw_value">
                                <field name="city"/>, <field name="country_id"/>
                            </div>
                            <field t-if="record.email.raw_value" class="o_text_overflow" name="email"/>
                        </main>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_supplier_view_search" model="ir.ui.view">
        <field name="name">lunch.supplier.view.search</field>
        <field name="model">lunch.supplier</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="lunch_vendors_action" model="ir.actions.act_window">
        <field name="name">Vendors</field>
        <field name="res_model">lunch.supplier</field>
        <field name="view_mode">kanban,list,form</field>
        <field name="search_view_id" ref="lunch_supplier_view_search"/>
    </record>
</odoo>

```

## File: views\lunch_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="lunch_payment_dialog" name="Lunch Payment Dialog">
        To add some money to your wallet, please contact your lunch manager.
    </template>
</odoo>

```

## File: views\lunch_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Top menu item -->
    <menuitem id='menu_lunch' name='Lunch' sequence="235" groups="group_lunch_user" web_icon="lunch,static/description/icon.png">
        <menuitem name="My Lunch" id="menu_lunch_title" sequence="50">
            <menuitem name="New Order" id="lunch_order_menu_form" action="lunch.lunch_product_action_order" sequence="1"/>
            <menuitem name="My Order History" id="lunch_order_menu_tree" action="lunch_order_action" sequence="2"/>
            <menuitem name="My Account History" id="lunch_cashmove_report_menu_form" action="lunch_cashmove_report_action_account" sequence="3"/>
        </menuitem>
        <menuitem name="Manager" id="menu_lunch_admin" sequence="51" groups="group_lunch_manager">
            <menuitem name="Today's Orders" id="lunch_order_menu_by_supplier" action="lunch_order_action_by_supplier" />
            <menuitem name="Control Vendors" id="lunch_order_menu_control_suppliers" action="lunch_order_action_control_suppliers" />
            <menuitem name="Control Accounts" id="lunch_cashmove_report_menu_control_accounts" action="lunch_cashmove_report_action_control_accounts"/>
            <menuitem name="Cash Moves" id="lunch_cashmove_report_menu_payment" action="lunch_cashmove_action_payment"/>
        </menuitem>
        <menuitem name="Configuration" id="menu_lunch_config" sequence="53" groups="group_lunch_manager">
            <menuitem name="Settings" id="lunch_settings_menu" action="lunch_config_settings_action" sequence="1"/>
            <menuitem name="Vendors" id="lunch_vendors_menu" action="lunch_vendors_action" sequence="2"/>
            <menuitem name="Locations" id="lunch_location_menu" action="lunch_location_action" sequence="3"/>
            <menuitem name="Products" id="lunch_product_menu" action="lunch_product_action" sequence="4"/>
            <menuitem name="Product Categories" id="lunch_product_category_menu" action="lunch_product_category_action" sequence="5"/>
            <menuitem name="Alerts" id="lunch_alert_menu" action="lunch_alert_action" sequence="6"/>
        </menuitem>
    </menuitem>
</odoo>

```

## File: views\res_config_settings.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.lunch</field>
        <field name="model">res.config.settings</field>
        <field name="priority" eval="90"/>
        <field name="inherit_id" ref="base.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//form" position="inside">
                <app data-string="Lunch" string="Lunch" name="lunch" groups="lunch.group_lunch_manager">
                    <field name="currency_id" invisible="1"/>
                    <block title="Lunch" name="lunch_overdraft_setting_container">
                        <setting id="lunch_minimum_threshold" string="Lunch Overdraft" help="Maximum overdraft that your employees can reach" company_dependent="1">
                            <div class="content-group">
                                <div class="mt16 row">
                                    <label for="company_lunch_minimum_threshold" string="Overdraft" class="col-3 col-lg-3 o_light_label"/>
                                    <field name="company_lunch_minimum_threshold" widget="monetary"/>
                                </div>
                            </div>
                        </setting>
                    </block>
                    <block name="lunch_notification_setting_container">
                        <setting string="Reception notification" company_dependent="1" help="Send this message to your users when their order has been delivered." id="lunch_notification">
                            <field name="company_lunch_notify_message" widget="html" class="col col-lg w-100"/>
                        </setting>
                    </block>
                </app>
            </xpath>
        </field>
    </record>

    <record id="lunch_config_settings_action" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_id" ref="res_config_settings_view_form"/>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'lunch', 'bin_size': False}</field>
    </record>
</odoo>

```

