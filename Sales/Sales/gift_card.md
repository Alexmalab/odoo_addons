# Odoo Module: gift_card

Category: Sales/Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Gift Card",
    'summary': "Use gift card",
    'description': """Integrate gift card mechanism""",
    'category': 'Sales/Sales',
    'version': '1.0',
    'depends': ['product'],
    'data': [
        'data/gift_card_data.xml',
        'views/views.xml',
        'security/ir.model.access.csv',
    ],
    'license': 'LGPL-3',
}

```

## File: data\gift_card_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- the product used to deduce on sale order -->
        <record id="pay_with_gift_card_product" model="product.product">
            <field name="name">Gift Card</field>
            <field name="list_price">0</field>
            <field name="detailed_type">service</field>
            <field name="purchase_ok" eval="False"/>
            <field name="sale_ok" eval="False"/>
            <field name="image_1920" type="base64" file="gift_card/static/img/gift_card.png"/>
        </record>

        <!-- the product to sell to generate gift cards automatically -->
        <record id="gift_card_product_50" model="product.product">
            <field name="name">Gift Card</field>
            <field name="list_price">50</field>
            <field name="detailed_type">gift</field>
            <field name="purchase_ok" eval="False"/>
            <field name="image_1920" type="base64" file="gift_card/static/img/gift_card.png"/>
        </record>
    </data>
</odoo>

```

## File: models\gift_card.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from uuid import uuid4


class GiftCard(models.Model):
    _name = "gift.card"
    _description = "Gift Card"
    _order = 'id desc'
    _check_company_auto = True

    @api.model
    def _generate_code(self):
        return '044' + str(uuid4())[4:-8][3:]

    name = fields.Char(compute='_compute_name')
    code = fields.Char(default=lambda x: x._generate_code(), required=True, readonly=True, copy=False)
    partner_id = fields.Many2one('res.partner', help="If empty, all users can use it")
    company_id = fields.Many2one('res.company', default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', readonly=True, related='company_id.currency_id')
    initial_amount = fields.Monetary(required=True, currency_field='currency_id')
    balance = fields.Monetary(compute="_compute_balance")  # in company currency
    expired_date = fields.Date(default=lambda self: fields.Date.add(fields.Date.today(), years=1))
    state = fields.Selection(
        selection=[('valid', 'Valid'), ('expired', 'Expired')],
        default='valid',
        copy=False
    )

    _sql_constraints = [
        ('unique_gift_card_code', 'UNIQUE(code)', 'The gift card code must be unique.'),
        ('check_amount', 'CHECK(initial_amount >= 0)', 'The initial amount must be positive.')
    ]

    def _compute_name(self):
        for record in self:
            record.name = _("Gift #%s", record.id)

    @api.autovacuum
    def _gc_mark_expired_gift_card(self):
        self.env['gift.card'].search([
            '&', ('state', '=', 'valid'), ('expired_date', '<', fields.Date.today())
        ]).write({'state': 'expired'})

    def balance_converted(self, currency_id=False):
        # helper to convert the current balance in the currency provided
        return self.currency_id._convert(self.balance, currency_id, self.env.company, fields.Date.today())

    def can_be_used(self):
        # expired state are computed once a day, so can be not synchro
        return self.state == 'valid' and self.balance > 0 and (not self.expired_date or self.expired_date >= fields.Date.today())

    @api.depends("initial_amount")
    def _compute_balance(self):
        for record in self:
            record.balance = record.initial_amount

```

## File: models\product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import UserError


class ProductTemplate(models.Model):
    _inherit = "product.template"

    detailed_type = fields.Selection(selection_add=[
        ('gift', 'Gift Card'),
    ], ondelete={'gift': 'set service'})

    def _detailed_type_mapping(self):
        type_mapping = super()._detailed_type_mapping()
        type_mapping['gift'] = 'service'
        return type_mapping

    @api.ondelete(at_uninstall=False)
    def _unlink_gift_card_product(self):
        if self.env.ref('gift_card.pay_with_gift_card_product').product_tmpl_id in self:
            raise UserError(_('Deleting the Gift Card Pay product is not allowed.'))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import gift_card
from . import product

```

## File: security\ir.model.access.csv

```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_gift_card_all,Gift Card Program All,model_gift_card,,0,0,0,0

```

## File: views\views.xml

```xml
<?xml version="1.0"?>
<odoo>
    <!-- Actions -->
    <record id="gift_card_action" model="ir.actions.act_window">
        <field name="name">Gift Cards</field>
        <field name="res_model">gift.card</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'search_default_valid': True}</field>
    </record>

    <!-- view mode -->
    <record id="gift_card_view_tree" model="ir.ui.view">
        <field name="name">gift.card.tree</field>
        <field name="model">gift.card</field>
        <field name="arch" type="xml">
            <tree string="Gift Card">
                <field name="code"/>
                <field name="balance"/>
                <field name="state"/>
            </tree>
        </field>
    </record>

    <record id="gift_card_view_form" model="ir.ui.view">
        <field name="name">gift.card.form</field>
        <field name="model">gift.card</field>
        <field name="arch" type="xml">
            <form>
                <header>
                     <field name="state" widget="statusbar"/>
                </header>
                <sheet>
                    <group>
                        <group>
                            <field name="create_date"/>
                            <field name="expired_date"/>
                            <field name="code"/>
                        </group>
                        <group name="gift_card">
                            <field name="currency_id" attrs="{'invisible': True}"/>
                            <field name="initial_amount"/>
                            <field name="balance"/>
                            <field name="partner_id"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <!-- Searching -->
    <record id="gift_card_view_search" model="ir.ui.view">
        <field name="name">gift.card.search</field>
        <field name="model">gift.card</field>
        <field name="arch" type="xml">
            <search string="Gift Card">
                <field name="code"/>
                <filter name="valid" string="Valid" domain="[('state', '=', 'valid')]" />
            </search>
        </field>
    </record>
</odoo>

```

