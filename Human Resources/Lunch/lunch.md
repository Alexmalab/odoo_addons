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
from . import populate

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

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
    'certificate': '001292377792581874189',
    'assets': {
        'web.assets_backend': [
            'lunch/static/src/scss/lunch_view.scss',
            'lunch/static/src/scss/lunch_kanban.scss',
            'lunch/static/src/scss/lunch_list.scss',
            'lunch/static/src/js/lunch_controller_common.js',
            'lunch/static/src/js/lunch_widget.js',
            'lunch/static/src/js/lunch_mobile.js',
            'lunch/static/src/js/lunch_payment_dialog.js',
            'lunch/static/src/js/lunch_kanban_view.js',
            'lunch/static/src/js/lunch_kanban_controller.js',
            'lunch/static/src/js/lunch_kanban_renderer.js',
            'lunch/static/src/js/lunch_kanban_record.js',
            'lunch/static/src/js/lunch_model_extension.js',
            'lunch/static/src/js/lunch_list_view.js',
            'lunch/static/src/js/lunch_list_controller.js',
            'lunch/static/src/js/lunch_list_renderer.js',
        ],
        'web.qunit_suite_tests': [
            'lunch/static/tests/lunch_test_utils.js',
            'lunch/static/tests/lunch_kanban_tests.js',
            'lunch/static/tests/lunch_list_tests.js',
        ],
        'web.qunit_mobile_suite_tests': [
            'lunch/static/tests/lunch_test_utils.js',
            'lunch/static/tests/lunch_kanban_mobile_tests.js',
        ],
        'web.assets_qweb': [
            'lunch/static/src/xml/lunch_templates.xml',
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
    def infos(self, user_id=None):
        self._check_user_impersonification(user_id)
        user = request.env['res.users'].browse(user_id) if user_id else request.env.user

        infos = self._make_infos(user, order=False)

        lines = self._get_current_lines(user)
        if lines:
            lines = [{'id': line.id,
                      'product': (line.product_id.id, line.product_id.name, float_repr(float_round(line.price, 2), 2)),
                      'toppings': [(topping.name, float_repr(float_round(topping.price, 2), 2))
                                   for topping in line.topping_ids_1 | line.topping_ids_2 | line.topping_ids_3],
                      'quantity': line.quantity,
                      'price': line.price,
                      'state': line.state, # Only used for _get_state
                      'note': line.note} for line in lines]
            raw_state, state = self._get_state(lines)
            infos.update({
                'total': float_repr(float_round(sum(line['price'] for line in lines), 2), 2),
                'raw_state': raw_state,
                'state': state,
                'lines': lines,
            })
        return infos

    @http.route('/lunch/trash', type='json', auth='user')
    def trash(self, user_id=None):
        self._check_user_impersonification(user_id)
        user = request.env['res.users'].browse(user_id) if user_id else request.env.user

        lines = self._get_current_lines(user)
        lines.action_cancel()
        lines.unlink()

    @http.route('/lunch/pay', type='json', auth='user')
    def pay(self, user_id=None):
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
    def set_user_location(self, location_id=None, user_id=None):
        self._check_user_impersonification(user_id)
        user = request.env['res.users'].browse(user_id) if user_id else request.env.user

        user.sudo().last_lunch_location_id = request.env['lunch.location'].browse(location_id)
        return True

    @http.route('/lunch/user_location_get', type='json', auth='user')
    def get_user_location(self, user_id=None):
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
        has_multi_company_access = not user_location.company_id or user_location.company_id.id in request._context.get('allowed_company_ids', request.env.company.ids)

        if not user_location or not has_multi_company_access:
            user.last_lunch_location_id = user_location = request.env['lunch.location'].search([], limit=1)

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
            [('user_id', '=', user.id), ('date', '=', fields.Date.context_today(user)), ('state', '!=', 'cancelled')]
            )

    def _get_state(self, lines):
        """
            This method returns the lowest state of the list of lines

            eg: [confirmed, confirmed, new] will return ('new', 'To Order')
        """
        states_to_int = {'new': 0, 'ordered': 1, 'confirmed': 2, 'cancelled': 3}
        int_to_states = ['new', 'ordered', 'confirmed', 'cancelled']
        translated_states = dict(request.env['lunch.order']._fields['state']._description_selection(request.env))

        state = int_to_states[min(states_to_int[line['state']] for line in lines)]

        return (state, translated_states[state])

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
            <field name="email">cynthiasanchez@gmail.com</field>
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
            <field name="groups_id" eval="[(4, ref('lunch.group_lunch_user'))]"/>
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
        </record>

        <record id="partner_pizza_inn" model="res.partner">
            <field name="name">Pizza Inn</field>
            <field name="city">New Delhi TN</field>
            <field name="country_id" ref="base.us"/>
            <field name="street">#8, 1 st Floor,iscore complex</field>
            <field name="street2">Gandhi Gramam,Gandhi Nagar</field>
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
        </record>

        <record model="res.partner" id="partner_sushi_shop">
            <field name="name">Sushi Shop</field>
            <field name="city">Paris</field>
            <field name="country_id" ref="base.fr"/>
            <field name="street">Boulevard Saint-Germain</field>
            <field name="zip">486624</field>
            <field name="email">order@sushi.com</field>
            <field name="phone">+32498859912</field>
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
        <field name="name">Lunch: Send by email</field>
        <field name="model_id" ref="lunch.model_lunch_supplier"/>
        <field name="email_from">{{ ctx['order']['email_from'] }}</field>
        <field name="partner_to">{{ ctx['order']['supplier_id'] }}</field>
        <field name="subject">Orders for {{ ctx['order']['company_name'] }}</field>
        <field name="lang">{{ ctx.get('default_lang') }}</field>
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
                </td><td valign="middle" align="right">
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

from odoo import api, fields, models
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
                'numbercall': -1,
                'doall': False,
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
        super().write(values)
        if not CRON_DEPENDS.isdisjoint(values):
            self._sync_cron()

    def unlink(self):
        crons = self.cron_id.sudo()
        server_actions = crons.ir_actions_server_id
        super().unlink()
        crons.unlink()
        server_actions.unlink()

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
            self.env['mail.thread'].message_notify(body=self.message, partner_ids=partners.ids)

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

    def name_get(self):
        return [(cashmove.id, '%s %s' % (_('Lunch Cashmove'), '#%d' % cashmove.id)) for cashmove in self]

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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, UserError


class LunchOrder(models.Model):
    _name = 'lunch.order'
    _description = 'Lunch Order'
    _order = 'id desc'
    _display_name = 'product_id'

    name = fields.Char(related='product_id.name', string="Product Name", store=True, readonly=True)
    topping_ids_1 = fields.Many2many('lunch.topping', 'lunch_order_topping', 'order_id', 'topping_id', string='Extras 1', domain=[('topping_category', '=', 1)])
    topping_ids_2 = fields.Many2many('lunch.topping', 'lunch_order_topping', 'order_id', 'topping_id', string='Extras 2', domain=[('topping_category', '=', 2)])
    topping_ids_3 = fields.Many2many('lunch.topping', 'lunch_order_topping', 'order_id', 'topping_id', string='Extras 3', domain=[('topping_category', '=', 3)])
    product_id = fields.Many2one('lunch.product', string="Product", required=True)
    category_id = fields.Many2one(
        string='Product Category', related='product_id.category_id', store=True)
    date = fields.Date('Order Date', required=True, readonly=True,
                       states={'new': [('readonly', False)]},
                       default=fields.Date.context_today)
    supplier_id = fields.Many2one(
        string='Vendor', related='product_id.supplier_id', store=True, index=True)
    user_id = fields.Many2one('res.users', 'User', readonly=True,
                              states={'new': [('readonly', False)]},
                              default=lambda self: self.env.uid)
    lunch_location_id = fields.Many2one('lunch.location', default=lambda self: self.env.user.last_lunch_location_id)
    note = fields.Text('Notes')
    price = fields.Monetary('Total Price', compute='_compute_total_price', readonly=True, store=True)
    active = fields.Boolean('Active', default=True)
    state = fields.Selection([('new', 'To Order'),
                              ('ordered', 'Ordered'),
                              ('confirmed', 'Received'),
                              ('cancelled', 'Cancelled')],
                             'Status', readonly=True, index=True, default='new')
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

    @api.depends_context('show_reorder_button')
    @api.depends('state')
    def _compute_display_reorder_button(self):
        show_button = self.env.context.get('show_reorder_button')
        for order in self:
            order.display_reorder_button = show_button and order.state == 'confirmed'

    def init(self):
        self._cr.execute("""CREATE INDEX IF NOT EXISTS lunch_order_user_product_date ON %s (user_id, product_id, date)"""
            % self._table)

    def _extract_toppings(self, values):
        """
            If called in api.multi then it will pop topping_ids_1,2,3 from values
        """
        if self.ids:
            # TODO This is not taking into account all the toppings for each individual order, this is usually not a problem
            # since in the interface you usually don't update more than one order at a time but this is a bug nonetheless
            topping_1 = values.pop('topping_ids_1')[0][2] if 'topping_ids_1' in values else self[:1].topping_ids_1.ids
            topping_2 = values.pop('topping_ids_2')[0][2] if 'topping_ids_2' in values else self[:1].topping_ids_2.ids
            topping_3 = values.pop('topping_ids_3')[0][2] if 'topping_ids_3' in values else self[:1].topping_ids_3.ids
        else:
            topping_1 = values['topping_ids_1'][0][2] if 'topping_ids_1' in values else []
            topping_2 = values['topping_ids_2'][0][2] if 'topping_ids_2' in values else []
            topping_3 = values['topping_ids_3'][0][2] if 'topping_ids_3' in values else []

        return topping_1 + topping_2 + topping_3

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

    @api.model
    def create(self, values):
        lines = self._find_matching_lines({
            **values,
            'toppings': self._extract_toppings(values),
        })
        if lines:
            # YTI FIXME This will update multiple lines in the case there are multiple
            # matching lines which should not happen through the interface
            lines.update_quantity(1)
            return lines[:1]
        return super().create(values)

    def write(self, values):
        merge_needed = 'note' in values or 'topping_ids_1' in values or 'topping_ids_2' in values or 'topping_ids_3' in values
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
                self.invalidate_cache(['topping_ids_2', 'topping_ids_3'])
                values['topping_ids_1'] = [(6, 0, toppings)]
                matching_lines = self._find_matching_lines({
                    'user_id': values.get('user_id', line.user_id.id),
                    'product_id': values.get('product_id', line.product_id.id),
                    'note': values.get('note', line.note or False),
                    'toppings': toppings,
                    'lunch_location_id': values.get('lunch_location_id', default_location_id),
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
            ('date', '=', fields.Date.today()),
            ('note', '=', values.get('note', False)),
            ('lunch_location_id', '=', values.get('lunch_location_id', default_location_id)),
        ]
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
        for line in self.filtered(lambda line: line.state != 'confirmed'):
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
        self.flush()
        for line in self:
            if self.env['lunch.cashmove'].get_wallet_balance(line.user_id) < 0:
                raise ValidationError(_('Your wallet does not contain enough money to order that. To add some money to your wallet, please contact your lunch manager.'))

    def action_order(self):
        for order in self:
            if not order.supplier_id.available_today:
                raise UserError(_('The vendor related to this order is not available today.'))
        if self.filtered(lambda line: not line.product_id.active):
            raise ValidationError(_('Product is no longer available.'))
        self.write({
            'state': 'ordered',
        })
        for order in self:
            order.lunch_location_id = order.user_id.last_lunch_location_id
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
        pass

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

from odoo.modules.module import get_module_resource


class LunchProductCategory(models.Model):
    """ Category of the product such as pizza, sandwich, pasta, chinese, burger... """
    _name = 'lunch.product.category'
    _inherit = 'image.mixin'
    _description = 'Lunch Product Category'

    @api.model
    def _default_image(self):
        image_path = get_module_resource('lunch', 'static/img', 'lunch.png')
        return base64.b64encode(open(image_path, 'rb').read())

    name = fields.Char('Product Category', required=True, translate=True)
    company_id = fields.Many2one('res.company')
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id')
    product_count = fields.Integer(compute='_compute_product_count', help="The number of products related to this category")
    active = fields.Boolean(string='Active', default=True)
    image_1920 = fields.Image(default=_default_image)

    def _compute_product_count(self):
        product_data = self.env['lunch.product'].read_group([('category_id', 'in', self.ids)], ['category_id'], ['category_id'])
        data = {product['category_id'][0]: product['category_id_count'] for product in product_data}
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

from datetime import datetime, time, timedelta
from textwrap import dedent

from odoo import _, api, fields, models
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

    _sql_constraints = [
        ('automatic_email_time_range',
         'CHECK(automatic_email_time >= 0 AND automatic_email_time <= 12)',
         'Automatic Email Sending Time should be between 0 and 12'),
    ]

    def name_get(self):
        res = []
        for supplier in self:
            if supplier.phone:
                res.append((supplier.id, '%s %s' % (supplier.name, supplier.phone)))
            else:
                res.append((supplier.id, supplier.name))
        return res

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
                'numbercall': -1,
                'doall': False,
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
            topping_values = topping[2]
            if topping_values:
                topping_values.update({'topping_category': 2})
        for topping in values.get('topping_ids_3', []):
            topping_values = topping[2]
            if topping_values:
                topping_values.update({'topping_category': 3})
        if values.get('company_id'):
            self.env['lunch.order'].search([('supplier_id', 'in', self.ids)]).write({'company_id': values['company_id']})
        super().write(values)
        if not CRON_DEPENDS.isdisjoint(values):
            # flush automatic_email_time field to call _sql_constraints
            self.flush(['automatic_email_time'])
            self._sync_cron()

    def unlink(self):
        crons = self.cron_id.sudo()
        server_actions = crons.ir_actions_server_id
        super().unlink()
        crons.unlink()
        server_actions.unlink()

    def toggle_active(self):
        """ Archiving related lunch product """
        res = super().toggle_active()
        active_suppliers = self.filtered(lambda s: s.active)
        inactive_suppliers = self - active_suppliers
        Product = self.env['lunch.product'].with_context(active_test=False)
        Product.search([('supplier_id', 'in', active_suppliers.ids)]).write({'active': True})
        Product.search([('supplier_id', 'in', inactive_suppliers.ids)]).write({'active': False})
        return res

    def _send_auto_email(self):
        """ Send an email to the supplier with the order of the day """
        # Called daily by cron
        self.ensure_one()

        if not self.available_today:
            return

        if self.send_by != 'mail':
            raise UserError(_("Cannot send an email to this supplier!"))

        orders = self.env['lunch.order'].search([
            ('supplier_id', '=', self.id),
            ('state', '=', 'ordered'),
            ('date', '=', fields.Date.context_today(self.with_context(tz=self.tz))),
        ], order="user_id, name")
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

        orders.action_confirm()

    @api.depends('recurrency_end_date', 'mon', 'tue', 'wed', 'thu', 'fri', 'sat', 'sun')
    def _compute_available_today(self):
        now = fields.Datetime.now().replace(tzinfo=pytz.UTC)

        for supplier in self:
            now = now.astimezone(pytz.timezone(supplier.tz))

            if supplier.recurrency_end_date and now.date() >= supplier.recurrency_end_date:
                supplier.available_today = False
            else:
                fieldname = WEEKDAY_TO_NAME[now.weekday()]
                supplier.available_today = supplier[fieldname]

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

```

## File: models\lunch_topping.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

from odoo.tools import formatLang


class LunchTopping(models.Model):
    _name = 'lunch.topping'
    _description = 'Lunch Extras'

    name = fields.Char('Name', required=True)
    company_id = fields.Many2one('res.company', default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', related='company_id.currency_id')
    price = fields.Monetary('Price', required=True)
    supplier_id = fields.Many2one('lunch.supplier', ondelete='cascade')
    topping_category = fields.Integer('Topping Category', help="This field is a technical field", required=True, default=1)

    def name_get(self):
        currency_id = self.env.company.currency_id
        res = dict(super(LunchTopping, self).name_get())
        for topping in self:
            price = formatLang(self.env, topping.price, currency_obj=currency_id)
            res[topping.id] = '%s %s' % (topping.name, price)
        return list(res.items())

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class Company(models.Model):
    _inherit = 'res.company'

    lunch_minimum_threshold = fields.Float()

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

## File: populate\lunch.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import logging
from dateutil.relativedelta import relativedelta
from itertools import groupby

from odoo import models
from odoo.tools import populate

_logger = logging.getLogger(__name__)


class LunchProductCategory(models.Model):
    _inherit = 'lunch.product.category'
    _populate_sizes = {'small': 5, 'medium': 150, 'large': 400}
    _populate_dependencies = ['res.company']

    def _populate_factories(self):
        # TODO topping_ids_{1,2,3}, toppping_label_{1,2,3}, topping_quantity{1,2,3}
        company_ids = self.env.registry.populated_models['res.company']

        return [
            ('name', populate.constant('lunch_product_category_{counter}')),
            ('company_id', populate.iterate(
                [False, self.env.ref('base.main_company').id] + company_ids,
                [1, 1] + [2/(len(company_ids) or 1)]*len(company_ids))),
        ]


class LunchProduct(models.Model):
    _inherit = 'lunch.product'
    _populate_sizes = {'small': 10, 'medium': 150, 'large': 10000}
    _populate_dependencies = ['lunch.product.category', 'lunch.supplier']

    def _populate_factories(self):
        category_ids = self.env.registry.populated_models['lunch.product.category']
        category_records = self.env['lunch.product.category'].browse(category_ids)
        category_by_company = {k: list(v) for k, v in groupby(category_records, key=lambda rec: rec['company_id'].id)}

        supplier_ids = self.env.registry.populated_models['lunch.supplier']
        company_by_supplier = {rec.id: rec.company_id.id for rec in self.env['lunch.supplier'].browse(supplier_ids)}

        def get_category(random=None, values=None, **kwargs):
            company_id = company_by_supplier[values['supplier_id']]
            return random.choice(category_by_company[company_id]).id

        return [
            ('active', populate.iterate([True, False], [0.9, 0.1])),
            ('name', populate.constant('lunch_product_{counter}')),
            ('price', populate.randfloat(0.1, 50)),
            ('supplier_id', populate.randomize(supplier_ids)),
            ('category_id', populate.compute(get_category)),
        ]


class LunchLocation(models.Model):
    _inherit = 'lunch.location'

    _populate_sizes = {'small': 3, 'medium': 50, 'large': 500}
    _populate_dependencies = ['res.company']

    def _populate_factories(self):
        company_ids = self.env.registry.populated_models['res.company']

        return [
            ('name', populate.constant('lunch_location_{counter}')),
            ('address', populate.constant('lunch_address_location_{counter}')),
            ('company_id', populate.randomize(company_ids))
        ]


class LunchSupplier(models.Model):
    _inherit = 'lunch.supplier'

    _populate_sizes = {'small': 3, 'medium': 50, 'large': 1500}

    _populate_dependencies = ['lunch.location', 'res.partner', 'res.users']

    def _populate_factories(self):

        location_ids = self.env.registry.populated_models['lunch.location']
        partner_ids = self.env.registry.populated_models['res.partner']
        user_ids = self.env.registry.populated_models['res.users']

        def get_location_ids(random=None, **kwargs):
            nb_locations = random.randint(0, len(location_ids))
            return [(6, 0, random.choices(location_ids, k=nb_locations))]

        return [

            ('active', populate.cartesian([True, False])),
            ('send_by', populate.cartesian(['phone', 'mail'])),
            ('delivery', populate.cartesian(['delivery', 'no_delivery'])),
            ('mon', populate.iterate([True, False], [0.9, 0.1])),
            ('tue', populate.iterate([True, False], [0.9, 0.1])),
            ('wed', populate.iterate([True, False], [0.9, 0.1])),
            ('thu', populate.iterate([True, False], [0.9, 0.1])),
            ('fri', populate.iterate([True, False], [0.9, 0.1])),
            ('sat', populate.iterate([False, True], [0.9, 0.1])),
            ('sun', populate.iterate([False, True], [0.9, 0.1])),
            ('available_location_ids', populate.iterate(
                [[], [(6, 0, location_ids)]],
                then=populate.compute(get_location_ids))),
            ('partner_id', populate.randomize(partner_ids)),
            ('responsible_id', populate.randomize(user_ids)),
            ('moment', populate.iterate(['am', 'pm'])),
            ('automatic_email_time', populate.randfloat(0, 12)),
        ]


class LunchOrder(models.Model):
    _inherit = 'lunch.order'
    _populate_sizes = {'small': 20, 'medium': 3000, 'large': 15000}
    _populate_dependencies = ['lunch.product', 'res.users', 'res.company']

    def _populate_factories(self):
        # TODO topping_ids_{1,2,3}, topping_label_{1,3}, topping_quantity_{1,3}
        user_ids = self.env.registry.populated_models['res.users']
        product_ids = self.env.registry.populated_models['lunch.product']
        company_ids = self.env.registry.populated_models['res.company']

        return [
            ('active', populate.cartesian([True, False])),
            ('state', populate.cartesian(['new', 'confirmed', 'ordered', 'cancelled'])),
            ('product_id', populate.randomize(product_ids)),
            ('user_id', populate.randomize(user_ids)),
            ('note', populate.constant('lunch_note_{counter}')),
            ('company_id', populate.randomize(company_ids)),
            ('quantity', populate.randint(0, 10)),
        ]


class LunchAlert(models.Model):
    _inherit = 'lunch.alert'
    _populate_sizes = {'small': 10, 'medium': 40, 'large': 150}

    _populate_dependencies = ['lunch.location']

    def _populate_factories(self):

        location_ids = self.env.registry.populated_models['lunch.location']

        def get_location_ids(random=None, **kwargs):
            nb_max = len(location_ids)
            start = random.randint(0, nb_max)
            end = random.randint(start, nb_max)
            return location_ids[start:end]

        return [
            ('active', populate.cartesian([True, False])),
            ('recipients', populate.cartesian(['everyone', 'last_week', 'last_month', 'last_year'])),
            ('mode', populate.iterate(['alert', 'chat'])),
            ('mon', populate.iterate([True, False], [0.9, 0.1])),
            ('tue', populate.iterate([True, False], [0.9, 0.1])),
            ('wed', populate.iterate([True, False], [0.9, 0.1])),
            ('thu', populate.iterate([True, False], [0.9, 0.1])),
            ('fri', populate.iterate([True, False], [0.9, 0.1])),
            ('sat', populate.iterate([False, True], [0.9, 0.1])),
            ('sun', populate.iterate([False, True], [0.9, 0.1])),
            ('name', populate.constant('alert_{counter}')),
            ('message', populate.constant('<strong>alert message {counter}</strong>')),
            ('notification_time', populate.randfloat(0, 12)),
            ('notification_moment', populate.iterate(['am', 'pm'])),
            ('until', populate.randdatetime(relative_before=relativedelta(years=-2), relative_after=relativedelta(years=2))),
            ('location_ids', populate.compute(get_location_ids))
        ]

```

## File: populate\__init__.py

```python
from . import lunch

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

    def name_get(self):
        return [(cashmove.id, '%s %s' % (_('Lunch Cashmove'), '#%d' % cashmove.id)) for cashmove in self]

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
                    format('Order: %%s x %%s %%s', lol.quantity::text, lp.name, lol.display_toppings) as description
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
        <field name="name">lunch.cashmove.report.tree</field>
        <field name="model">lunch.cashmove.report</field>
        <field name="arch" type="xml">
            <tree string="cashmove tree">
                <field name="currency_id" invisible="1"/>
                <field name="date"/>
                <field name="user_id"/>
                <field name="description"/>
                <field name="amount" sum="Total" widget="monetary"/>
            </tree>
        </field>
    </record>

    <record id="lunch_cashmove_report_view_tree_2" model="ir.ui.view">
        <field name="name">lunch.cashmove.report.tree</field>
        <field name="model">lunch.cashmove.report</field>
        <field name="arch" type="xml">
            <tree string="cashmove tree" create='false'>
                <field name="currency_id" invisible="1"/>
                <field name="date"/>
                <field name="description"/>
                <field name="amount" sum="Total" widget="monetary"/>
            </tree>
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

    <record id="view_lunch_cashmove_report_kanban" model="ir.ui.view">
        <field name="name">lunch.cashmove.report.kanban</field>
        <field name="model">lunch.cashmove.report</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <field name="date"/>
                <field name="user_id"/>
                <field name="description"/>
                <field name="amount"/>
                <field name="currency_id" invisible="1"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div class="row mb4">
                                <div class="col-8">
                                    <span>
                                        <strong class="o_kanban_record_title"><t t-esc="record.description.value"/></strong>
                                    </span>
                                </div>
                                <div class="col-4 text-right">
                                    <span class="badge badge-pill">
                                        <strong><i class="fa fa-money" role="img" aria-label="Amount" title="Amount"/> <field name="amount" widget="monetary"/></strong>
                                    </span>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-6">
                                    <i class="fa fa-clock-o" role="img" aria-label="Date" title="Date"/>
                                    <t t-esc="record.date.value"/>
                                </div>
                                <div class="col-6">
                                    <div class="float-right">
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                    </div>
                                </div>
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
        <field name="view_mode">tree</field>
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
        <field name="view_mode">tree,kanban,form</field>
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
lunch_alert_user,access_lunch_alert_lunch_user,model_lunch_alert,group_lunch_user,1,1,1,1
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
            <field name="name">User</field>
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
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="70" height="70" viewBox="0 0 70 70">
    <defs>
        <path id="icon-a" d="M4,5.35309892e-14 C36.4160122,9.87060235e-15 58.0836068,-3.97961823e-14 65,5.07020818e-14 C69,6.733808e-14 70,1 70,5 C70,43.0488877 70,62.4235458 70,65 C70,69 69,70 65,70 C61,70 9,70 4,70 C1,70 7.10542736e-15,69 7.10542736e-15,65 C7.25721566e-15,62.4676575 3.83358709e-14,41.8005206 3.60818146e-14,5 C-1.13686838e-13,1 1,5.75716207e-14 4,5.35309892e-14 Z"/>
        <linearGradient id="icon-c" x1="100%" x2="0%" y1="0%" y2="100%">
            <stop offset="0%" stop-color="#CDC484"/>
            <stop offset="100%" stop-color="#B5AA59"/>
        </linearGradient>
        <path id="icon-d" d="M34.9928486,19.5391573 C35.0500601,19.8604454 36.1442308,25.9991002 36.1442308,28.3438202 C36.1442308,31.9190055 34.1561298,34.4688031 31.216887,35.4941909 L32.1394231,51.7705126 C32.1894832,52.7070335 31.409976,53.5 30.4230769,53.5 L25.8461538,53.5 C24.8664063,53.5 24.0797476,52.7138694 24.1298077,51.7705126 L25.0523437,35.4941909 C22.1059495,34.4688031 20.125,31.9121696 20.125,28.3438202 C20.125,25.9922643 21.2191707,19.8604454 21.2763822,19.5391573 C21.5052284,18.1514658 24.5159856,18.1309581 24.7019231,19.6143524 L24.7019231,29.2666692 C24.7948918,29.4990904 25.7817909,29.4854186 25.8461538,29.2666692 C25.946274,27.5371818 26.4111178,19.7510707 26.4182692,19.5733369 C26.6542668,18.1514658 29.6149639,18.1514658 29.8438101,19.5733369 C29.858113,19.7579067 30.3158053,27.5371818 30.4159255,29.2666692 C30.4802885,29.4854186 31.4743389,29.4990904 31.5601563,29.2666692 L31.5601563,19.6143524 C31.7460938,18.137794 34.7640024,18.1514658 34.9928486,19.5391573 Z M43.5173678,39.0693762 L42.4446514,51.7226612 C42.3588341,52.6796898 43.1526442,53.5 44.1538462,53.5 L48.1586538,53.5 C49.1097957,53.5 49.875,52.7685567 49.875,51.8593796 L49.875,20.1407181 C49.875,19.2383769 49.1097957,18.5000977 48.1586538,18.5000977 C42.2587139,18.5000977 32.3253606,30.7022121 43.5173678,39.0693762 Z"/>
        <path id="icon-e" d="M34.9928486,17.5391573 C35.0500601,17.8604454 36.1442308,23.9991002 36.1442308,26.3438202 C36.1442308,29.9190055 34.1561298,32.4688031 31.216887,33.4941909 L32.1394231,49.7705126 C32.1894832,50.7070335 31.409976,51.5 30.4230769,51.5 L25.8461538,51.5 C24.8664063,51.5 24.0797476,50.7138694 24.1298077,49.7705126 L25.0523437,33.4941909 C22.1059495,32.4688031 20.125,29.9121696 20.125,26.3438202 C20.125,23.9922643 21.2191707,17.8604454 21.2763822,17.5391573 C21.5052284,16.1514658 24.5159856,16.1309581 24.7019231,17.6143524 L24.7019231,27.2666692 C24.7948918,27.4990904 25.7817909,27.4854186 25.8461538,27.2666692 C25.946274,25.5371818 26.4111178,17.7510707 26.4182692,17.5733369 C26.6542668,16.1514658 29.6149639,16.1514658 29.8438101,17.5733369 C29.858113,17.7579067 30.3158053,25.5371818 30.4159255,27.2666692 C30.4802885,27.4854186 31.4743389,27.4990904 31.5601563,27.2666692 L31.5601563,17.6143524 C31.7460938,16.137794 34.7640024,16.1514658 34.9928486,17.5391573 Z M43.5173678,37.0693762 L42.4446514,49.7226612 C42.3588341,50.6796898 43.1526442,51.5 44.1538462,51.5 L48.1586538,51.5 C49.1097957,51.5 49.875,50.7685567 49.875,49.8593796 L49.875,18.1407181 C49.875,17.2383769 49.1097957,16.5000977 48.1586538,16.5000977 C42.2587139,16.5000977 32.3253606,28.7022121 43.5173678,37.0693762 Z"/>
    </defs>
    <g fill="none" fill-rule="evenodd">
        <mask id="icon-b" fill="#fff">
            <use xlink:href="#icon-a"/>
        </mask>
        <g mask="url(#icon-b)">
            <rect width="70" height="70" fill="url(#icon-c)"/>
            <path fill="#FFF" fill-opacity=".383" d="M4,1.8 L65,1.8 C67.6666667,1.8 69.3333333,1.13333333 70,-0.2 C70,2.46666667 70,3.46666667 70,2.8 L1.10547097e-14,2.8 C-1.65952376e-14,3.46666667 -2.9161925e-14,2.46666667 -2.66453526e-14,-0.2 C0.666666667,1.13333333 2,1.8 4,1.8 Z" transform="matrix(1 0 0 -1 0 2.8)"/>
            <path fill="#393939" d="M37.1559742,53 L4,52 C2,52 -7.10542736e-15,51.8543417 0,47.9215686 L2.11942169e-16,24.8800004 L21.6437579,0 L21,12.2352941 L26.7780762,0 L29.5940312,3.47275252 L33,0 L35,12.2352941 L41.1357671,3.93586958 L49,0 L49.7896212,33.2154878 L37.1559742,53 Z" opacity=".324" transform="translate(0 17)"/>
            <path fill="#000" fill-opacity=".383" d="M4,4 L65,4 C67.6666667,4 69.3333333,3 70,1 C70,3.66666667 70,5 70,5 L1.77635684e-15,5 C1.77635684e-15,5 1.77635684e-15,3.66666667 1.77635684e-15,1 C0.666666667,3 2,4 4,4 Z" transform="translate(0 65)"/>
            <use fill="#000" fill-rule="nonzero" opacity=".3" xlink:href="#icon-d"/>
            <use fill="#FFF" fill-rule="nonzero" xlink:href="#icon-e"/>
        </g>
    </g>
</svg>

```

## File: static\src\js\lunch_controller_common.js

```javascript
odoo.define('lunch.LunchControllerCommon', function (require) {
"use strict";

/**
 * This file defines the common events and functions used by Controllers for the Lunch view.
 */

var session = require('web.session');
var core = require('web.core');
const {Markup} = require('web.utils');
var LunchWidget = require('lunch.LunchWidget');
var LunchPaymentDialog = require('lunch.LunchPaymentDialog');

var _t = core._t;

var LunchControllerCommon = {
    custom_events: {
        add_product: '_onAddProduct',
        change_location: '_onLocationChanged',
        change_user: '_onUserChanged',
        open_wizard: '_onOpenWizard',
        order_now: '_onOrderNow',
        remove_product: '_onRemoveProduct',
        unlink_order: '_onUnlinkOrder',
    },
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.editMode = false;
        this.updated = false;
        this.widgetData = null;
        this.context = session.user_context;
        this.archiveEnabled = false;
    },
    /**
     * @override
     */
    start: function () {
        // create a div inside o_content that will be used to wrap the lunch
        // banner and renderer (this is required to get the desired
        // layout with the searchPanel to the left)
        var self = this;
        this.$('.o_content').append($('<div>').addClass('o_lunch_content'));
        return this._super.apply(this, arguments).then(function () {
            self.$('.o_lunch_content').append(self.$('.o_lunch_view'));
        });
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _fetchPaymentInfo: function () {
        return this._rpc({
            route: '/lunch/payment_message',
            params: {
                context: this.context,
            },
        });
    },
    async _fetchWidgetData() {
        const widgetData = await this._rpc({
            route: '/lunch/infos',
            params: {
                user_id: this.searchModel.get('userId'),
                context: this.context,
            },
        });
        widgetData.wallet = parseFloat(widgetData.wallet).toFixed(2);
        (widgetData.alerts || []).forEach(alert => { alert.message = Markup(alert.message); });
        this.widgetData = widgetData;
    },
    /**
     * Renders and appends the lunch banner widget.
     *
     * @private
     */
    _renderLunchWidget: function () {
        var oldWidget = this.widget;
        this.widget = new LunchWidget(this, Object.assign(this.widgetData, {edit: this.editMode}));
        return this.widget.appendTo(document.createDocumentFragment()).then(() => {
            this.$('.o_lunch_content').prepend(this.widget.$el);
            if (oldWidget) {
                oldWidget.destroy();
            }
        });
    },
    _showPaymentDialog: function (title) {
        var self = this;

        title = title || '';

        this._fetchPaymentInfo().then(function (data) {
            var paymentDialog = new LunchPaymentDialog(self, _.extend(data, {title: title}));
            paymentDialog.open();
        });
    },
    /**
     * Override to fetch and display the lunch data. Because of the presence of
     * the searchPanel, also wrap the lunch widget and the renderer into
     * a div, to get the desired layout.
     *
     * @override
     * @private
     */
    _update: function () {
        var def = this._fetchWidgetData().then(this._renderLunchWidget.bind(this));
        return Promise.all([def, this._super.apply(this, arguments)]);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onAddProduct: function (ev) {
        var self = this;
        ev.stopPropagation();

        this._rpc({
            model: 'lunch.order',
            method: 'update_quantity',
            args: [[ev.data.lineId], 1],
        }).then(function () {
            self.reload();
        });
    },
    _onLocationChanged: function (ev) {
        ev.stopPropagation();
        this.searchModel.dispatch('setLocationId', ev.data.locationId);
    },
    _onOpenWizard: function (ev) {
        var self = this;
        ev.stopPropagation();

        var ctx = this.searchModel.get('userId') ? {default_user_id: this.searchModel.get('userId')} : {};

        var options = {
            on_close: function () {
                self.reload();
            },
        };

        var action = {
            res_model: 'lunch.order',
            name: _t('Configure Your Order'),
            type: 'ir.actions.act_window',
            views: [[false, 'form']],
            target: 'new',
            context: _.extend(ctx, {default_product_id: ev.data.productId}),
        };

        if (ev.data.lineId) {
            action = _.extend(action, {
                res_id: ev.data.lineId,
                context: _.extend(action.context, {
                    active_id: ev.data.lineId,
                }),
            });
        }

        this.do_action(action, options);
    },
    _onOrderNow: function (ev) {
        var self = this;
        ev.stopPropagation();

        this._rpc({
            route: '/lunch/pay',
            params: {
                user_id: this.searchModel.get('userId'),
                context: this.context,
            },
        }).then(function (isPaid) {
            if (isPaid) {
                // TODO: feedback?
                self.reload();
            } else {
                self._showPaymentDialog(_t("Not enough money in your wallet"));
                self.reload();
            }
        });
    },
    _onRemoveProduct: function (ev) {
        var self = this;
        ev.stopPropagation();

        this._rpc({
            model: 'lunch.order',
            method: 'update_quantity',
            args: [[ev.data.lineId], -1],
        }).then(function () {
            self.reload();
        });
    },
    _onUserChanged: function (ev) {
        ev.stopPropagation();
        this.searchModel.dispatch('updateUserId', ev.data.userId);
    },
    _onUnlinkOrder: function (ev) {
        var self = this;
        ev.stopPropagation();

        this._rpc({
            route: '/lunch/trash',
            params: {
                user_id: this.searchModel.get('userId'),
                context: this.context,
            },
        }).then(function () {
            self.reload();
        });
    },
};

return LunchControllerCommon;

});

```

## File: static\src\js\lunch_kanban_controller.js

```javascript
odoo.define('lunch.LunchKanbanController', function (require) {
"use strict";

/**
 * This file defines the Controller for the Lunch Kanban view, which is an
 * override of the KanbanController.
 */

var KanbanController = require('web.KanbanController');
var LunchControllerCommon = require('lunch.LunchControllerCommon');

var LunchKanbanController = KanbanController.extend(LunchControllerCommon , {
    custom_events: _.extend({}, KanbanController.prototype.custom_events, LunchControllerCommon.custom_events),
});

return LunchKanbanController;

});

```

## File: static\src\js\lunch_kanban_record.js

```javascript
odoo.define('lunch.LunchKanbanRecord', function (require) {
    "use strict";

    /**
     * This file defines the KanbanRecord for the Lunch Kanban view.
     */

    var KanbanRecord = require('web.KanbanRecord');

    var LunchKanbanRecord = KanbanRecord.extend({
        events: _.extend({}, KanbanRecord.prototype.events, {
            'click': '_onSelectRecord',
        }),

        //--------------------------------------------------------------------------
        // Handlers
        //--------------------------------------------------------------------------

        /**
         * Open the add product wizard
         *
         * @private
         * @param {MouseEvent} ev Click event
         */
        _onSelectRecord: function (ev) {
            ev.preventDefault();
            // ignore clicks on oe_kanban_action elements
            if (!$(ev.target).hasClass('oe_kanban_action')) {
                this.trigger_up('open_wizard', {productId: this.recordData.product_id ? this.recordData.product_id.res_id: this.recordData.id});
            }
        },

        _openRecord() {},
    });

    return LunchKanbanRecord;

    });

```

## File: static\src\js\lunch_kanban_renderer.js

```javascript
odoo.define('lunch.LunchKanbanRenderer', function (require) {
"use strict";

/**
 * This file defines the Renderer for the Lunch Kanban view, which is an
 * override of the KanbanRenderer.
 */

var LunchKanbanRecord = require('lunch.LunchKanbanRecord');

var KanbanRenderer = require('web.KanbanRenderer');

var LunchKanbanRenderer = KanbanRenderer.extend({
    config: _.extend({}, KanbanRenderer.prototype.config, {
        KanbanRecord: LunchKanbanRecord,
    }),

    /**
     * @override
     */
    start: function () {
        this.$el.addClass('o_lunch_view o_lunch_kanban_view position-relative align-content-start flex-grow-1 flex-shrink-1');
        return this._super.apply(this, arguments);
    },
});

return LunchKanbanRenderer;

});

```

## File: static\src\js\lunch_kanban_view.js

```javascript
odoo.define('lunch.LunchKanbanView', function (require) {
"use strict";

var LunchKanbanController = require('lunch.LunchKanbanController');
var LunchKanbanRenderer = require('lunch.LunchKanbanRenderer');

var core = require('web.core');
var KanbanView = require('web.KanbanView');
var view_registry = require('web.view_registry');

var _lt = core._lt;

var LunchKanbanView = KanbanView.extend({
    config: _.extend({}, KanbanView.prototype.config, {
        Controller: LunchKanbanController,
        Renderer: LunchKanbanRenderer,
    }),
    display_name: _lt('Lunch Kanban'),

    /**
     * @override
     */
    _createSearchModel(params, extraExtensions = {}) {
        Object.assign(extraExtensions, { Lunch: {} });
        return this._super(params, extraExtensions);
    },
});

view_registry.add('lunch_kanban', LunchKanbanView);

return LunchKanbanView;

});

```

## File: static\src\js\lunch_list_controller.js

```javascript
odoo.define('lunch.LunchListController', function (require) {
"use strict";

/**
 * This file defines the Controller for the Lunch List view, which is an
 * override of the ListController.
 */

var ListController = require('web.ListController');
var LunchControllerCommon = require('lunch.LunchControllerCommon');

var LunchListController = ListController.extend(LunchControllerCommon, {
    custom_events: _.extend({}, ListController.prototype.custom_events, LunchControllerCommon.custom_events),
});

return LunchListController;

});

```

## File: static\src\js\lunch_list_renderer.js

```javascript
odoo.define('lunch.LunchListRenderer', function (require) {
"use strict";

/**
 * This file defines the Renderer for the Lunch List view, which is an
 * override of the ListRenderer.
 */

var ListRenderer = require('web.ListRenderer');

var LunchListRenderer = ListRenderer.extend({
    events: _.extend({}, ListRenderer.prototype.events, {
        'click .o_data_row': '_onClickListRow',
    }),

    /**
     * @override
     */
    start: function () {
        this.$el.addClass('o_lunch_view o_lunch_list_view');
        return this._super.apply(this, arguments);
    },
    /**
     * Override to add id of product_id in dataset.
     *
     * @override
     */
    _renderRow: function (record) {
        var tr = this._super.apply(this, arguments);
        tr.attr('data-product-id', record.data.id);
        return tr;
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * Open the add product wizard
     *
     * @private
     * @param {MouseEvent} ev Click event
     */
    _onClickListRow: function (ev) {
        ev.preventDefault();
        var productId = ev.currentTarget.dataset && ev.currentTarget.dataset.productId ? parseInt(ev.currentTarget.dataset.productId) : null;

        if (productId) {
            this.trigger_up('open_wizard', {productId: productId});
        }
    },
});

return LunchListRenderer;

});

```

## File: static\src\js\lunch_list_view.js

```javascript
odoo.define('lunch.LunchListView', function (require) {
"use strict";

var LunchListController = require('lunch.LunchListController');
var LunchListRenderer = require('lunch.LunchListRenderer');

var core = require('web.core');
var ListView = require('web.ListView');
var view_registry = require('web.view_registry');

var _lt = core._lt;

var LunchListView = ListView.extend({
    config: _.extend({}, ListView.prototype.config, {
        Controller: LunchListController,
        Renderer: LunchListRenderer,
    }),
    display_name: _lt('Lunch List'),

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * @override
     */
    _createSearchModel(params, extraExtensions = {}) {
        Object.assign(extraExtensions, { Lunch: {} });
        return this._super(params, extraExtensions);
    },

});

view_registry.add('lunch_list', LunchListView);

return LunchListView;

});

```

## File: static\src\js\lunch_mobile.js

```javascript
odoo.define('lunch.LunchMobile', function (require) {
"use strict";

var config = require('web.config');
var LunchWidget = require('lunch.LunchWidget');
var LunchKanbanController = require('lunch.LunchKanbanController');
var LunchListController = require('lunch.LunchListController');

if (!config.device.isMobile) {
    return;
}

LunchWidget.include({
    template: "LunchWidgetMobile",

    /**
     * Override to set the toggle state allowing initially open it.
     *
     * @override
     */
    init: function (parent, params) {
        this._super.apply(this, arguments);
        this.keepOpen = params.keepOpen || undefined;
    },
});

var mobileFunctions = {
    /**
     * @override
     */
    init: function () {
        this._super.apply(this, arguments);
        this.openWidget = false;
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    /**
     * Override to add the widget's toggle state to its data.
     *
     * @override
     * @private
     */
    _renderLunchWidget: function () {
        this.widgetData.keepOpen = this.openWidget;
        this.openWidget = false;
        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    /**
     * @override
     * @private
     */
    _onAddProduct: function () {
        this.openWidget = true;
        this._super.apply(this, arguments);
    },

    /**
     * @override
     * @private
     */
    _onRemoveProduct: function () {
        this.openWidget = true;
        this._super.apply(this, arguments);
    },
};

LunchKanbanController.include(mobileFunctions);
LunchListController.include(mobileFunctions);

});

```

## File: static\src\js\lunch_model.js

```javascript
odoo.define('lunch.LunchModel', function (require) {
"use strict";

/**
 * This file defines the Model for the Lunch Kanban view, which is an
 * override of the KanbanModel.
 */

var session = require('web.session');
var BasicModel = require('web.BasicModel');

var LunchModel = BasicModel.extend({
    init: function () {
        this.locationId = false;
        this.userId = false;
        this._promInitLocation = null;

        this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Public
    //--------------------------------------------------------------------------

    /**
     * @return {Promise} resolved with the location domain
     */
    getLocationDomain: function () {
        var self = this;
        return this._initUserLocation().then(function () {
            return self._buildLocationDomainLeaf() ? [self._buildLocationDomainLeaf()]: [];
        });
    },
    __load: function () {
        var self = this;
        var args = arguments;
        var _super = this._super;

        return this._initUserLocation().then(function () {
            var params = args[0];
            self._addOrUpdate(params.domain, self._buildLocationDomainLeaf());

            return _super.apply(self, args);
        });
    },
    __reload: function (id, options) {
        var domain = options && options.domain || this.localData[id].domain;

        this._addOrUpdate(domain, this._buildLocationDomainLeaf());
        options = _.extend(options, {domain: domain});

        return this._super.apply(this, arguments);
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _addOrUpdate: function (domain, subDomain) {
        if (subDomain && subDomain.length) {
            var key = subDomain[0];
            var index = _.findIndex(domain, function (val) {
                return val[0] === key;
            });

            if (index < 0) {
                domain.push(subDomain);
            } else {
                domain[index] = subDomain;
            }

            return domain;
        }

        return domain;
    },
    /**
     * Builds the domain leaf corresponding to the current user's location
     *
     * @private
     * @return {(Array[])|undefined}
     */
    _buildLocationDomainLeaf: function () {
        if (this.locationId) {
            return ['is_available_at', 'in', [this.locationId]];
        }
    },
    _getUserLocation: function () {
        return this._rpc({
            route: '/lunch/user_location_get',
            params: {
                context: session.user_context,
                user_id: this.userId,
            },
        });
    },
    /**
     * Gets the user location once.
     * Can be triggered from anywhere
     * Useful to inject the location domain in the search panel
     *
     * @private
     * @return {Promise}
     */
    _initUserLocation: function () {
        var self = this;
        if (!this._promInitLocation) {
            this._promInitLocation = new Promise(function (resolve) {
                self._getUserLocation().then(function (locationId) {
                    self.locationId = locationId;
                    resolve();
                });
            });
        }
        return this._promInitLocation;
    },
    _updateLocation: function (locationId) {
        this.locationId = locationId;
        return Promise.resolve();
    },
    _updateUser: function (userId) {
        this.userId = userId;
        this._promInitLocation = null;
        return this._initUserLocation();
    }
});

return LunchModel;

});

```

## File: static\src\js\lunch_model_extension.js

```javascript
odoo.define("lunch/static/src/js/lunch_model_extension.js", function (require) {
    "use strict";

    const ActionModel = require("web.ActionModel");

    class LunchModelExtension extends ActionModel.Extension {

        //---------------------------------------------------------------------
        // Public
        //---------------------------------------------------------------------

        /**
         * @override
         * @returns {any}
         */
        get(property) {
            switch (property) {
                case "domain": return this.getDomain();
                case "userId": return this.state.userId;
            }
        }

        /**
         * @override
         */
        async load() {
            await this._updateLocationId();
        }

        /**
         * @override
         */
        prepareState() {
            Object.assign(this.state, {
                locationId: null,
                userId: null,
            });
        }

        //---------------------------------------------------------------------
        // Actions / Getters
        //---------------------------------------------------------------------

        /**
         * @returns {Array[] | null}
         */
        getDomain() {
            if (this.state.locationId) {
                return [["is_available_at", "in", [this.state.locationId]]];
            }
            return null;
        }

        /**
         * @param {number} locationId
         * @returns {Promise}
         */
        setLocationId(locationId) {
            this.state.locationId = locationId;
            this.env.services.rpc({
                route: "/lunch/user_location_set",
                params: {
                    context: this.env.session.user_context,
                    location_id: this.state.locationId,
                    user_id: this.state.userId,
                },
            });
        }

        /**
         * @param {number} userId
         * @returns {Promise}
         */
        updateUserId(userId) {
            this.state.userId = userId;
            this.shouldLoad = true;
        }

        //---------------------------------------------------------------------
        // Private
        //---------------------------------------------------------------------

        /**
         * @returns {Promise}
         */
        async _updateLocationId() {
            this.state.locationId = await this.env.services.rpc({
                route: "/lunch/user_location_get",
                params: {
                    context: this.env.session.user_context,
                    user_id: this.state.userId,
                },
            });
        }
    }

    ActionModel.registry.add("Lunch", LunchModelExtension, 20);

    return LunchModelExtension;
});

```

## File: static\src\js\lunch_payment_dialog.js

```javascript
odoo.define('lunch.LunchPaymentDialog', function (require) {
"use strict";

var Dialog = require('web.Dialog');

var LunchPaymentDialog = Dialog.extend({
    template: 'lunch.LunchPaymentDialog',

    init: function (parent, options) {
        this._super.apply(this, arguments);

        options = options || {};

        this.message = options.message || '';
    },
});

return LunchPaymentDialog;

});

```

## File: static\src\js\lunch_widget.js

```javascript
odoo.define('lunch.LunchWidget', function (require) {
"use strict";

var core = require('web.core');
var relationalFields = require('web.relational_fields');
var session = require('web.session');
var Widget = require('web.Widget');

var _t = core._t;
var FieldMany2One = relationalFields.FieldMany2One;


var LunchMany2One = FieldMany2One.extend({
    start: function () {
        this.$el.addClass('w-100');
        return this._super.apply(this, arguments);
    }
});

var LunchWidget = Widget.extend({
    template: 'LunchWidget',
    custom_events: {
        field_changed: '_onFieldChanged',
    },
    events: {
        'click .o_add_product': '_onAddProduct',
        'click .o_lunch_widget_order_button': '_onOrderNow',
        'click .o_remove_product': '_onRemoveProduct',
        'click .o_lunch_widget_unlink': '_onUnlinkOrder',
        'click .o_lunch_open_wizard': '_onLunchOpenWizard',
    },

    init: function (parent, params) {
        this._super.apply(this, arguments);

        this.is_manager = params.is_manager || false;
        this.group_portal_id = params.group_portal_id || false;
        this.userimage = params.userimage || '';
        this.username = params.username || '';

        this.lunchUserField = null;

        this.locations = params.locations || [];
        this.userLocation = params.user_location[1] || '';

        const company_ids = [false].concat(session.user_context.allowed_company_ids || []);
        this.lunchLocationField = this._createMany2One('locations', 'lunch.location', this.userLocation, () => [
            ['company_id', 'in', company_ids]
        ]);

        this.wallet = params.wallet || 0;
        this.raw_state = params.raw_state || 'new';
        this.state = params.state || _t('To Order');
        this.lines = params.lines || [];
        this.total = params.total || 0;

        this.alerts = params.alerts || [];

        this.currency = params.currency || session.get_currency(session.company_currency_id);
    },
    willStart: function () {
        var superDef = this._super.apply(this, arguments);

        if (this.is_manager) {
            this.lunchUserField = this._createMany2One('users', 'res.users', this.username,
                () => [['groups_id', 'not in', [this.group_portal_id]]]
            );
        }
        return superDef;
    },
    renderElement: function () {
        this._super.apply(this, arguments);
        if (this.lunchUserField) {
            this.lunchUserField.appendTo(this.$('.o_lunch_user_field'));
        } else {
            this.$('.o_lunch_user_field').text(this.username);
        }

        if (this.userLocation) {
            this.lunchLocationField.appendTo(this.$('.o_lunch_location_field'));
        }
    },

    //--------------------------------------------------------------------------
    // Private
    //--------------------------------------------------------------------------

    _createMany2One: function (name, model, value, domain, context) {
        var fields = {};
        fields[name] = {type: 'many2one', relation: model, string: name};
        var data = {};
        data[name] = {data: {display_name: value}};

        var record = {
            id: name,
            res_id: 1,
            model: 'dummy',
            fields: fields,
            fieldsInfo: {
                default: fields,
            },
            data: data,
            getDomain: domain || function () { return []; },
            getContext: context || function () { return {}; },
        };
        var options = {
            mode: 'edit',
            noOpen: true,
            attrs: {
                can_create: false,
                can_write: false,
            }
        };
        return new LunchMany2One(this, name, record, options);
    },

    //--------------------------------------------------------------------------
    // Handlers
    //--------------------------------------------------------------------------

    _onAddProduct: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();
        this.trigger_up('add_product', {lineId: $(ev.currentTarget).data('id')});
    },
    _onOrderNow: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();

        this.trigger_up('order_now', {});
    },
    _onLunchOpenWizard: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();

        var target = $(ev.currentTarget);
        this.trigger_up('open_wizard', {productId: target.data('product-id'), lineId: target.data('id')});
    },
    _onFieldChanged: function (ev) {
        ev.stopPropagation();

        if (ev.data.dataPointID === 'users') {
            this.trigger_up('change_user', {userId: ev.data.changes.users.id});
        } else if (ev.data.dataPointID === 'locations') {
            this.trigger_up('change_location', {locationId: ev.data.changes.locations.id});
        }
    },
    _onRemoveProduct: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();

        this.trigger_up('remove_product', {lineId: $(ev.currentTarget).data('id')});
    },
    _onUnlinkOrder: function (ev) {
        ev.preventDefault();
        ev.stopPropagation();

        this.trigger_up('unlink_order', {});
    },
});

return LunchWidget;

});

```

## File: static\src\xml\lunch_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates>
    <span t-name="LunchWidget">
        <t t-foreach="widget.alerts" t-as="alert">
            <div class="alert alert-warning mb-0" role="alert">
                <t t-out="alert.message"/>
            </div>
        </t>
        <div class="o_lunch_banner container-fluid">
            <div class="o_lunch_widget row py-3 py-md-0">
                <div class="o_lunch_widget_info col-12 col-md-4 card border-0">
                    <div class="card-body row no-gutters align-items-center">
                        <div class="col-3 col-md-6 col-lg-3">
                            <img class="o_image_64_cover rounded-circle" t-attf-src="{{ widget.userimage }}"/>
                        </div>
                        <div class="col-9 col-md-6 col-lg-9">
                            <div class="pl-3">
                                <div class="o_lunch_user_field py-1"/>
                                <div class="o_lunch_location_field py-1"/>
                                <div class="d-flex flex-row py-1">
                                    <span class="flex-grow-1">Your Account</span>
                                    <t t-call="currency_field">
                                        <t t-set="value" t-value="widget.wallet"/>
                                        <t t-set="currency" t-value="widget.currency"/>
                                    </t>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="o_lunch_widget_info col-12 col-md-4 card border-0">
                    <t t-if="!_.isEmpty(widget.lines)">
                        <t t-if="widget.raw_state == 'ordered'">
                            <t t-set="state_class" t-value="'badge-warning o_lunch_ordered'"/>
                        </t>
                        <t t-else="widget.raw_state == 'confirmed'">
                            <t t-set="state_class" t-value="'badge-success o_lunch_confirmed'"/>
                        </t>
                        <div class="card-body">
                            <h4 class="card-title">
                                Your order
                                <button t-if="widget.raw_state != 'confirmed'" class="btn btn-sm btn-icon btn-link fa fa-trash o_lunch_widget_unlink"/>
                                <span t-if="widget.raw_state != 'new'" t-esc="widget.state" t-attf-class="badge badge-pill {{ state_class }}"/>
                            </h4>
                            <ul class="list-unstyled o_lunch_widget_lines">
                                <li t-foreach="widget.lines" t-as="line">
                                    <div class="d-flex align-items-center">
                                        <div class="flex-grow-0 flex-shrink-0 o_lunch_product_quantity">
                                            <button class="btn btn-sm btn-icon btn-link fa fa-minus-circle o_remove_product" t-if="widget.raw_state != 'confirmed'" t-attf-data-id="{{ line.id }}"/>
                                            <span t-esc="line.quantity"/>
                                            <button class="btn btn-sm btn-icon btn-link fa fa-plus-circle o_add_product" t-if="widget.raw_state != 'confirmed'" t-attf-data-id="{{ line.id }}"/>
                                        </div>
                                        <div class="flex-grow-1 pl-2">
                                            <button t-esc="line.product[1]" class="btn btn-link o_lunch_open_wizard" t-attf-data-product-id="{{ line.product[0] }}" t-attf-data-id="{{ line.id }}"/>
                                        </div>
                                        <div class="flex-grow-0">
                                            <t t-call="currency_field">
                                                <t t-set="value" t-value="line.product[2]"/>
                                                <t t-set="currency" t-value="widget.currency"/>
                                            </t>
                                        </div>
                                    </div>
                                    <div t-foreach="line.toppings" t-as="topping" class="d-flex flex-row">
                                        <div class="flex-grow-1 pl-5">
                                            <span>+ <t t-esc="topping[0]"/></span>
                                        </div>
                                        <div class="flex-grow-0">
                                            <t t-call="currency_field">
                                                <t t-set="value" t-value="topping[1]"/>
                                                <t t-set="currency" t-value="widget.currency"/>
                                            </t>
                                        </div>
                                    </div>
                                    <span t-if="line.note" t-esc="line.note" class="text-muted pl-5"/>
                                </li>
                            </ul>
                        </div>
                    </t>
                </div>
                <div class="o_lunch_widget_info col-12 col-md-4 card border-0">
                    <t t-if="!_.isEmpty(widget.lines) &amp;&amp; widget.raw_state == 'new'">
                        <div class="card-body d-flex flex-column justify-content-between">
                            <h4 class="card-title d-flex py-1">
                                <span class="flex-grow-1">Total</span>
                                <t t-call="currency_field">
                                    <t t-set="value" t-value="widget.total"/>
                                    <t t-set="currency" t-value="widget.currency"/>
                                </t>
                            </h4>
                            <button t-if="widget.raw_state == 'new'" class="btn btn-primary w-100 o_lunch_widget_order_button">Order now</button>
                        </div>
                    </t>
                </div>
            </div>
        </div>
    </span>

    <span t-name="currency_field" class="o_field_monetary o_field_number o_field_widget">
        <t t-js="ctx">
            ctx.value = _.str.sprintf('%.2f', parseFloat(ctx.value));
        </t>
        <t t-if="currency">
            <t t-if="currency.position == 'after'">
                <t t-esc="value"/><t t-esc="currency.symbol"/>
            </t>
            <t t-else="">
                <t t-esc="currency.symbol"/><t t-esc="value"/>
            </t>
        </t>
        <t t-else="">
            <t t-esc="value"/>
        </t>
    </span>

    <div t-name="lunch.LunchPaymentDialog">
        <span t-esc="widget.message"/>
    </div>

    <t t-name="LunchWidgetMobile">
        <details class="fixed-bottom" t-attf-open="#{widget.keepOpen}">
            <summary class="o_lunch_toggle_cart btn btn-primary w-100">
                <i class="fa fa-fw fa-shopping-cart"/>
                Your cart
                (<t t-call="currency_field">
                    <t t-set="value" t-value="widget.total"/>
                    <t t-set="currency" t-value="widget.currency"/>
                </t>)
            </summary>
            <t t-call="LunchWidget"/>
        </details>
    </t>
</templates>

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
        <field name="name">lunch.alert.tree</field>
        <field name="model">lunch.alert</field>
        <field name="arch" type="xml">
            <tree>
                <field name="name"/>
                <field name="mode"/>
                <field name="message" invisible="1"/>
                <field name="available_today"/>
                <field name="active" widget="boolean_toggle"/>
            </tree>
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
                            <field name="recipients" attrs="{'invisible': [['mode', '!=', 'chat']]}" widget="radio"/>
                            <field name="location_ids" widget="many2many_tags" required="1"/>
                            <field name="until"/>
                            <field name="active" widget="boolean_toggle"/>
                        </group>
                        <group>
                            <label for="notification_time" attrs="{'invisible': [['mode', '!=', 'chat']]}"/>
                            <div class="o_col">
                                <widget name="week_days"/>
                                <div class="o_row" attrs="{'invisible': [['mode', '!=', 'chat']]}">
                                    <field name="notification_time" attrs="{'required': [('mode', '=', 'chat')]}" widget="float_time"/>
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
                    <t t-name="kanban-box">
                        <div class="oe_kanban_card oe_kanban_global_click">
                            <div class="oe_kanban_content">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title"><field name="name"/></strong>
                                        <span><br/><field name="mode"/></span>
                                        <span attrs="{'invisible':[['mode', '!=', 'chat']]}">
                                            to <field name="recipients"/>
                                            on <field name="notification_time"/>
                                            <field name="notification_moment"/>
                                        </span>
                                    </div>
                                </div>
                                <div class="o_kanban_record_body">
                                    <field name="location_ids" widget="many2many_tags"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_alert_action" model="ir.actions.act_window">
        <field name="name">Lunch Alerts</field>
        <field name="res_model">lunch.alert</field>
        <field name="search_view_id" ref="lunch_alert_view_search"/>
        <field name="view_mode">tree,form,kanban</field>
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
        <field name="name">lunch.cashmove.tree</field>
        <field name="model">lunch.cashmove</field>
        <field name="arch" type="xml">
            <tree string="cashmove tree">
                <field name="currency_id" invisible="1"/>
                <field name="date"/>
                <field name="user_id"/>
                <field name="description"/>
                <field name="amount" sum="Total" widget="monetary"/>
            </tree>
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
                <field name="date"/>
                <field name="user_id"/>
                <field name="description"/>
                <field name="amount"/>
                <field name="currency_id" invisible="1"/>
                <templates>
                    <t t-name="kanban-box">
                        <div t-attf-class="oe_kanban_global_click">
                            <div class="row mb4">
                                <div class="col-8">
                                    <span>
                                        <strong class="o_kanban_record_title"><t t-esc="record.description.value"/></strong>
                                    </span>
                                </div>
                                <div class="col-4 text-right">
                                    <span class="badge badge-pill">
                                        <strong><i class="fa fa-money" role="img" aria-label="Amount" title="Amount"/> <field name="amount" widget="monetary"/></strong>
                                    </span>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-6">
                                    <i class="fa fa-clock-o" role="img" aria-label="Date" title="Date"/>
                                    <t t-esc="record.date.value"/>
                                </div>
                                <div class="col-6">
                                    <div class="float-right">
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                    </div>
                                </div>
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
        <field name="view_mode">tree,kanban,form</field>
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
            <tree editable="bottom">
                <field name="name"/>
                <field name="address"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record id="lunch_location_kanban_view" model="ir.ui.view">
        <field name="name">lunch.location.view.kanban</field>
        <field name="model">lunch.location</field>
        <field name="arch" type="xml">
            <kanban class="o_kanban_mobile">
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_card oe_kanban_global_click">
                            <div class="oe_kanban_content">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title"><field name="name"/></strong>
                                    </div>
                                </div>
                                <div class="o_kanban_record_body">
                                    <field name="company_id" groups="base.group_multi_company"/><br/>
                                    <field name="address"/>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_location_action" model="ir.actions.act_window">
        <field name="name">Lunch Locations</field>
        <field name="res_model">lunch.location</field>
        <field name="view_mode">tree,form,kanban</field>
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
        <field name="name">lunch.order.tree</field>
        <field name="model">lunch.order</field>
        <field name="arch" type="xml">
            <tree string="Order lines Tree" create="false" edit="false" decoration-muted="state == 'cancelled'" class="o_lunch_list">
                <header>
                    <button name="action_confirm" type="object" string="Receive"/>
                </header>
                <field name='date'/>
                <field name='supplier_id'/>
                <field name='product_id'/>
                <field name="display_toppings" class="o_text_overflow"/>
                <field name='note' class="o_text_overflow"/>
                <field name='user_id'/>
                <field name="lunch_location_id"/>
                <field name="currency_id" invisible="1"/>
                <field name='price' sum="Total" string="Price" widget="monetary"/>
                <field name='state'/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="display_reorder_button" invisible="1"/>
                <button name="action_reorder" string="Re-order" type="object" icon="fa-history" attrs="{'invisible': [('display_reorder_button', '=', False)]}" groups="lunch.group_lunch_user"/>
                <button name="action_confirm" string="Confirm" type="object" icon="fa-check" attrs="{'invisible': [('state', '!=', 'ordered')]}" groups="lunch.group_lunch_manager"/>
                <button name="action_cancel" string="Cancel" type="object" icon="fa-times" attrs="{'invisible': [('state', 'in', ['cancelled', 'confirmed'])]}" groups="lunch.group_lunch_manager"/>
                <button name="action_reset" string="Reset" type="object" icon="fa-undo" attrs="{'invisible': [('state', '!=', 'cancelled')]}" groups="lunch.group_lunch_manager"/>
            </tree>
        </field>
    </record>

    <record id='lunch_order_view_kanban' model='ir.ui.view'>
        <field name="name">lunch.order.kanban</field>
        <field name="model">lunch.order</field>
        <field name="arch" type="xml">
            <kanban create="false" edit="false">
                <field name="product_id"/>
                <field name="note"/>
                <field name="state"/>
                <field name="user_id"/>
                <field name="date"/>
                <field name="company_id"/>
                <field name="currency_id"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click">
                            <div class="o_kanban_record_top">
                                <div class="o_kanban_record_headings">
                                    <strong class="o_kanban_record_title"><field name="product_id"/></strong>
                                </div>
                                <field name="state" widget="label_selection" options="{'classes': {'new': 'default', 'confirmed': 'success', 'cancelled':'danger'}}"/>
                            </div>
                            <div>
                                <field name="note"/>
                            </div>
                            <div class="row">
                                <div class="col-6">
                                    <i class="fa fa-money" role="img" aria-label="Money" title="Money"/> <field name="price"/>
                                </div>
                                <div class="col-6 text-right">
                                    <i class="fa fa-clock-o" role="img" aria-label="Date" title="Date"/> <field name="date"/>
                                </div>
                            </div>
                            <div class="row mt4">
                                <div class="col-6">
                                    <a class="btn btn-sm btn-success" role="button" name="action_order" string="Order" type="object" attrs="{'invisible': ['|',('state','=','confirmed'),('state','=','ordered')]}" groups="lunch.group_lunch_manager">
                                        <i class="fa fa-phone" role="img" aria-label="Order button" title="Order button"/>
                                    </a>
                                    <a class="btn btn-sm btn-info" role="button" name="action_confirm" string="Receive" type="object" attrs="{'invisible': [('state','!=','ordered')]}" groups="lunch.group_lunch_manager">
                                        <i class="fa fa-check" role="img" aria-label="Receive button" title="Receive button"/>
                                    </a>
                                    <a class="btn btn-sm btn-danger" role="button" name="action_cancel" string="Cancel" type="object" attrs="{'invisible': [('state','=','cancelled')]}" groups="lunch.group_lunch_manager">
                                        <i class="fa fa-times" role="img" aria-label="Cancel button" title="Cancel button"/>
                                    </a>
                                </div>
                                <div class="col-6">
                                    <span class="float-right">
                                        <field name="user_id" widget="many2one_avatar_user"/>
                                    </span>
                                </div>
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
        <field name="view_mode">tree,kanban,pivot</field>
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
        <field name="view_mode">tree,kanban</field>
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
        <field name="view_mode">tree,kanban,pivot</field>
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
            <form>
                <field name="company_id" invisible="1"/>
                <field name="currency_id" invisible="1"/>
                <field name="quantity" invisible="1"/>
                <field name="product_id" invisible="1"/>
                <field name="category_id" invisible="1"/>
                <field name="available_toppings_1" invisible="1"/>
                <field name="available_toppings_2" invisible="1"/>
                <field name="available_toppings_3" invisible="1"/>
                <field name="supplier_id" invisible="1"/>
                <div class="d-flex">
                    <div class="flex-grow-0 pr-5">
                        <field name="image_1920" widget="image" class="o_lunch_image" options="{'image_preview': 'image_128'}"/>
                    </div>
                    <div class="flex-grow-1 pr-5">
                        <h2><field name="name"/></h2>
                        <h3 class="pt-3"><field name="price"/></h3>
                    </div>
                </div>
                <div class="o_lunch_wizard">
                    <div class="row">
                        <div class="col-2">
                            <field name="topping_label_1" nolabel="1" attrs="{'invisible': [('available_toppings_1', '=', False)]}" class="font-weight-bold"/>
                        </div>
                        <div class="col-10">
                            <field name="topping_ids_1" attrs="{'invisible': [('available_toppings_1', '=', False)]}" widget="many2many_checkboxes" nolabel="1" domain="[('topping_category', '=', 1), ('supplier_id', '=', supplier_id)]"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-2">
                            <field name="topping_label_2" nolabel="1" attrs="{'invisible': [('available_toppings_2', '=', False)]}" class="font-weight-bold"/>
                        </div>
                        <div class="col-10">
                            <field name="topping_ids_2" attrs="{'invisible': [('available_toppings_2', '=', False)]}" widget="many2many_checkboxes" nolabel="1" domain="[('topping_category', '=', 2), ('supplier_id', '=', supplier_id)]"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-2">
                            <field name="topping_label_3" nolabel="1" attrs="{'invisible': [('available_toppings_3', '=', False)]}" class="font-weight-bold"/>
                        </div>
                        <div class="col-10">
                            <field name="topping_ids_3" attrs="{'invisible': [('available_toppings_3', '=', False)]}" widget="many2many_checkboxes" nolabel="1" domain="[('topping_category', '=', 3), ('supplier_id', '=', supplier_id)]"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-2">
                            <label for="product_description" class="font-weight-bold"/>
                        </div>
                        <div class="col-10">
                            <field name="product_description" nolabel="1"/>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-2">
                            <label for="note" class="font-weight-bold" />
                        </div>
                        <div class="col-10">
                            <field name="note" nolabel="1" placeholder="Information, allergens, ..." />
                        </div>
                    </div>
                </div>
                <footer>
                    <button string="Save" special="save" data-hotkey="v" class="oe_highlight" invisible="not context.get('active_id', False)"/>
                    <button string="Add To Cart" name="add_to_cart" type="object" class="oe_highlight" invisible="context.get('active_id', False)" data-hotkey="w"/>
                    <button string="Discard" special="cancel" data-hotkey="z"/>
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
        <field name="name">lunch.product.tree</field>
        <field name="model">lunch.product</field>
        <field name="arch" type="xml">
            <tree string="Products Tree">
                <field name="currency_id" invisible="1"/>
                <field name="name"/>
                <field name="category_id"/>
                <field name="supplier_id"/>
                <field name="company_id" groups="base.group_multi_company"/>
                <field name="description"/>
                <field name="price" widget="monetary"/>
            </tree>
        </field>
    </record>

    <record id="lunch_product_view_tree_order" model="ir.ui.view">
        <field name="name">lunch.product.tree.order</field>
        <field name="inherit_id" ref="lunch_product_view_tree"/>
        <field name="model">lunch.product</field>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//tree" position="attributes">
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
                <field name="currency_id" invisible="1"/>
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
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
                <field name="id"/>
                <field name="name"/>
                <field name="category_id"/>
                <field name="supplier_id"/>
                <field name="description"/>
                <field name="currency_id"/>
                <field name="company_id"/>
                <field name="is_new"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click o_kanban_record_has_image_fill o_kanban_record">
                            <field name="image_128" class="o_lunch_image o_kanban_image_fill_left d-none d-md-block" options="{'placeholder': '/lunch/static/img/lunch.png', 'size': [94, 94]}" widget="image"/>
                            <div class="oe_kanban_details ml8">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title">
                                            <div>
                                                <span class="pr-1"><field name="is_favorite" widget="boolean_favorite" nolabel="1"/></span>
                                                <div class="float-right o_lunch_purple">
                                                    <field name="price" widget="monetary"/>
                                                </div>
                                                <div t-if="record.is_new.raw_value" class="float-right o_lunch_new_product mr-1 badge-pill badge-success">
                                                    New
                                                </div>
                                                <strong><span t-esc="record.name.value"/></strong>
                                            </div>
                                        </strong>
                                        <span class="o_kanban_record_subtitle"><span t-esc="record.supplier_id.value"/></span>
                                    </div>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <ul>
                                        <li t-out="record.description.value" class="text-muted"/>
                                    </ul>
                                </div>
                            </div>
                        </div>
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
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click o_kanban_record_has_image_fill o_kanban_record">
                            <div class="o_kanban_image_fill_left d-none d-md-block"
                                t-attf-style="background-image:url('#{kanban_image('lunch.product', 'image_128', record.id.raw_value)}')"/>
                            <div class="oe_kanban_details">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <strong class="o_kanban_record_title">
                                            <div>
                                                <div class="float-right">
                                                    <field name="price" widget="monetary"/>
                                                </div>
                                                <strong><span t-esc="record.name.value"/></strong>
                                            </div>
                                        </strong>
                                        <span class="o_kanban_record_subtitle"><span t-esc="record.supplier_id.value"/></span>
                                    </div>
                                </div>
                                <div class="o_kanban_record_bottom">
                                    <ul>
                                        <li t-out="record.description.value" class="text-muted"/>
                                    </ul>
                                </div>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>

    <record id="lunch_product_action_statbutton" model="ir.actions.act_window">
        <field name="name">Products</field>
        <field name="res_model">lunch.product</field>
        <field name="view_mode">kanban,tree,form</field>
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
        <field name="name">Product category Tree</field>
        <field name="model">lunch.product.category</field>
        <field name="arch" type="xml">
            <tree string="Products List">
                <field name='name' string="Product Category"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
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
                            context="{'search_default_category_id': active_id,'default_category_id': active_id}"
                            attrs="{'invisible': [('product_count', '=', 0)]}"
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
                <field name="id"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click o_kanban_record_has_image_fill o_kanban_record">
                            <div class="o_kanban_image_fill_left d-none d-md-block"
                                t-attf-style="background-image:url('#{kanban_image('lunch.product.category', 'image_128', record.id.raw_value)}')"/>
                            <div class="oe_kanban_details">
                                <div class="o_kanban_record_top">
                                    <div class="o_kanban_record_headings">
                                        <div class="float-right">
                                            <button class="badge badge-primary" type="action"
                                                name="%(lunch.lunch_product_action_statbutton)d"
                                                context="{'search_default_category_id': active_id,'default_category_id': active_id}"
                                                attrs="{'invisible': [('product_count', '=', 0)]}">
                                                <field string="Products" name="product_count" widget="statinfo"/>
                                            </button>
                                        </div>
                                    </div>
                                </div>
                                <div class="o_kanban_record_body">
                                    <strong><field name="name"/></strong><br/>
                                    <field name="company_id" groups="base.group_multi_company"/>
                                </div>
                            </div>
                        </div>
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
        <field name="view_mode">tree,kanban,form</field>
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
        <field name="res_model">lunch.product</field>
        <field name="view_mode">kanban,tree</field>
        <field name="view_ids" eval="[
            (5, 0, 0),
            (0, 0, {'view_mode': 'kanban', 'view_id': ref('view_lunch_product_kanban_order')}),
            (0, 0, {'view_mode': 'tree', 'view_id': ref('lunch_product_view_tree_order')})
        ]"/>
        <field name="search_view_id" ref="lunch_product_view_search"/>
        <field name="context">{'search_default_available_today': 1}</field>
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
        <field name="view_mode">tree,form,kanban</field>
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
        <field name="name">lunch.supplier.view.tree</field>
        <field name="model">lunch.supplier</field>
        <field name="arch" type="xml">
            <tree>
                <field name="name"/>
                <field name="phone" class="o_force_ltr"/>
                <field name="email"/>
            </tree>
        </field>
    </record>

    <record id="lunch_supplier_view_form" model="ir.ui.view">
        <field name="name">lunch.supplier.view.form</field>
        <field name="model">lunch.supplier</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
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
                            <field name="email" attrs="{'required': [('send_by', '=', 'mail')]}"/>
                            <field name="phone" class="o_force_ltr" attrs="{'required': [('send_by', '=', 'phone')]}"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="responsible_id" attrs="{'required': [('send_by', '=', 'mail')]}" groups="base.group_no_one" domain="[('share', '=', False)]"/>
                        </group>
                    </group>
                    <group>
                        <group string="Availability">
                            <label for="tz" groups="base.group_no_one"/>
                            <div class="o_col">
                                <div class="o_row">
                                    <field name="tz" groups="base.group_no_one"/>
                                </div>
                                <widget name="week_days"/>
                            </div>
                            <field name="recurrency_end_date" groups="base.group_no_one"/>
                        </group>
                        <group string="Orders">
                            <field name="delivery"/>
                            <field name="available_location_ids" widget="many2many_tags"/>
                            <field name="send_by" widget="radio"/>
                            <label for="automatic_email_time" attrs="{'invisible': [('send_by', '!=', 'mail')]}"/>
                            <div class="o_row" attrs="{'invisible': [('send_by', '!=', 'mail')]}"><field name="automatic_email_time" widget="float_time"/> <field name="moment"/></div>
                        </group>
                    </group>
                    <group>
                        <group>
                            <field name="active" invisible="1"/>
                            <field name="topping_label_1"/>
                            <field name="topping_quantity_1"/>
                        </group>
                        <div>
                            <field name="topping_ids_1" nolabel="1">
                                <tree editable="bottom">
                                    <field name="name"/>
                                    <field name="company_id" invisible="1"/>
                                    <field name="currency_id" invisible="1"/>
                                    <field name="price" widget="monetary"/>
                                </tree>
                            </field>
                        </div>
                        <group>
                            <field name="topping_label_2"/>
                            <field name="topping_quantity_2"/>
                        </group>
                        <div>
                            <field name="topping_ids_2" nolabel="1">
                                <tree editable="bottom">
                                    <field name="name"/>
                                    <field name="company_id" invisible="1"/>
                                    <field name="currency_id" invisible="1"/>
                                    <field name="price" widget="monetary"/>
                                </tree>
                            </field>
                        </div>
                        <group>
                            <field name="topping_label_3"/>
                            <field name="topping_quantity_3"/>
                        </group>
                        <div>
                            <field name="topping_ids_3" nolabel="1">
                                <tree editable="bottom">
                                    <field name="name"/>
                                    <field name="company_id" invisible="1"/>
                                    <field name="currency_id" invisible="1"/>
                                    <field name="price" widget="monetary"/>
                                </tree>
                            </field>
                        </div>
                    </group>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_follower_ids" groups="base.group_user"/>
                    <field name="activity_ids"/>
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="lunch_supplier_view_kanban" model="ir.ui.view">
        <field name="name">lunch.supplier.view.kanban</field>
        <field name="model">lunch.supplier</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="partner_id"/>
                <field name="city"/>
                <field name="country_id"/>
                <field name="email"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click o_kanban_record_has_image_fill o_res_partner_kanban">
                            <div class="oe_kanban_details">
                                <strong class="o_kanban_record_title oe_partner_heading"><field name="display_name"/></strong>
                                <ul>
                                    <li t-if="record.city.raw_value and !record.country_id.raw_value"><field name="city"/></li>
                                    <li t-if="!record.city.raw_value and record.country_id.raw_value"><field name="country_id"/></li>
                                    <li t-if="record.city.raw_value and record.country_id.raw_value"><field name="city"/>, <field name="country_id"/></li>
                                    <li t-if="record.email.raw_value" class="o_text_overflow"><field name="email"/></li>
                                </ul>
                            </div>
                        </div>
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
        <field name="view_mode">kanban,tree,form</field>
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
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="Lunch" string="Lunch" data-key="lunch" groups="lunch.group_lunch_manager">
                    <field name="currency_id" invisible="1"/>
                    <h2>Lunch</h2>
                    <div class="row mt16 o_settings_container" name="lunch_overdraft_setting_container">
                        <div class="col-12 col-lg-6 o_setting_box" id="lunch_minimum_threshold"
                            title="None">
                            <div class="o_setting_right_pane">
                                <span class="o_form_label">Lunch Overdraft</span>
                                <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                                <div class="text-muted">
                                    Maximum overdraft that your employees can reach
                                </div>
                                <div class="content-group">
                                    <div class="mt16 row">
                                        <label for="company_lunch_minimum_threshold" string="Overdraft" class="col-3 col-lg-3 o_light_label"/>
                                        <field name="company_lunch_minimum_threshold" widget="monetary"/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>

    <record id="lunch_config_settings_action" model="ir.actions.act_window">
        <field name="name">Settings</field>
        <field name="type">ir.actions.act_window</field>
        <field name="res_model">res.config.settings</field>
        <field name="view_id" ref="res_config_settings_view_form"/>
        <field name="view_mode">form</field>
        <field name="target">inline</field>
        <field name="context">{'module' : 'lunch', 'bin_size': False}</field>
    </record>
</odoo>

```

