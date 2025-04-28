# Odoo Module: loyalty

Category: Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Coupons & Loyalty',
    'summary': "Use discounts, gift card, eWallets and loyalty programs in different sales channels",
    'category': 'Sales',
    'version': '1.0',
    'depends': ['product'],
    'data': [
        'security/ir.model.access.csv',
        'security/loyalty_security.xml',
        'report/loyalty_report_templates.xml',
        'report/loyalty_report.xml',
        'data/loyalty_data.xml',
        'data/mail_template_data.xml',
        'wizard/loyalty_generate_wizard_views.xml',
        'views/loyalty_card_views.xml',
        'views/loyalty_mail_views.xml',
        'views/loyalty_program_views.xml',
        'views/loyalty_reward_views.xml',
        'views/loyalty_rule_views.xml',
    ],
    'demo': [
        'data/loyalty_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'loyalty/static/src/js/loyalty_card_list_view.js',
            'loyalty/static/src/js/loyalty_control_panel_widget.js',
            'loyalty/static/src/js/loyalty_list_view.js',
            'loyalty/static/src/scss/loyalty.scss',
            'loyalty/static/src/js/filterable_selection_field/filterable_selection_field.js',
            'loyalty/static/src/xml/loyalty_templates.xml',
        ],
        'web.qunit_suite_tests': [
            'loyalty/static/tests/**/*.js',
        ],
    },
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\loyalty_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Basic product for gift card program -->
    <record id="gift_card_product_50" model="product.product">
        <field name="name">Gift Card</field>
        <field name="list_price">50</field>
        <field name="detailed_type">service</field>
        <field name="purchase_ok" eval="False"/>
        <field name="image_1920" type="base64" file="loyalty/static/img/gift_card.png"/>
    </record>
    <!-- Basic product for eWallet programs -->
    <record id="ewallet_product_50" model="product.product">
        <field name="name">Top-up eWallet</field>
        <field name="list_price">50</field>
        <field name="detailed_type">service</field>
        <field name="purchase_ok" eval="False"/>
    </record>

    <data noupdate="1">
        <record forcecreate="0" id="config_online_sync_proxy_mode" model="ir.config_parameter">
            <field name="key">loyalty.compute_all_discount_product_ids</field>
            <field name="value">False</field>
        </record>
    </data>
</odoo>

```

## File: data\loyalty_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- 10 percent with code -->
    <record id="10_percent_with_code" model="loyalty.program">
        <field name="name">Code for 10% on orders</field>
        <field name="program_type">promo_code</field>
        <field name="trigger">with_code</field>
        <field name="portal_visible">False</field>
        <field name="portal_point_name">Discount point(s)</field>
    </record>

    <record id="10_percent_with_code_rule" model="loyalty.rule">
        <field name="mode">with_code</field>
        <field name="code">10pc</field>
        <field name="program_id" ref="loyalty.10_percent_with_code"/>
    </record>

    <record id="10_percent_with_code_reward" model="loyalty.reward">
        <field name="reward_type">discount</field>
        <field name="discount">10</field>
        <field name="discount_mode">percent</field>
        <field name="discount_applicability">order</field>
        <field name="program_id" ref="loyalty.10_percent_with_code"/>
    </record>
    <!-- 3 cabinet + 1 free -->
    <record id="3_cabinets_plus_1_free" model="loyalty.program">
        <field name="name">Buy 3 large cabinets, get one for free</field>
        <field name="program_type">buy_x_get_y</field>
        <field name="applies_on">current</field>
        <field name="trigger">auto</field>
        <field name="portal_visible">False</field>
        <field name="portal_point_name">Credit(s)</field>
    </record>

    <record id="3_cabinets_plus_1_free_rule" model="loyalty.rule">
        <field name="minimum_qty">3</field>
        <field name="reward_point_mode">unit</field>
        <field name="reward_point_amount">1</field>
        <field name="product_ids" eval="[(4, ref('product.product_product_6'))]"/>
        <field name="program_id" ref="loyalty.3_cabinets_plus_1_free"/>
    </record>

    <record id="3_cabinets_plus_1_free_reward" model="loyalty.reward">
        <field name="reward_type">product</field>
        <field name="reward_product_id" ref="product.product_product_6"/>
        <field name="required_points">3</field>
        <field name="program_id" ref="loyalty.3_cabinets_plus_1_free"/>
    </record>    
    <!-- 10 percent coupons -->
    <record id="10_percent_coupon" model="loyalty.program">
        <field name="name">10% Discount Coupons</field>
        <field name="program_type">coupons</field>
        <field name="applies_on">current</field>
        <field name="trigger">with_code</field>
        <field name="portal_point_name">Coupon points</field>
    </record>

    <record id="10_percent_coupon_rule" model="loyalty.rule">
        <field name="program_id" ref="loyalty.10_percent_coupon"/>
    </record>

    <record id="10_percent_coupon_reward" model="loyalty.reward">
        <field name="reward_type">discount</field>
        <field name="discount">10</field>
        <field name="discount_mode">percent</field>
        <field name="discount_applicability">order</field>
        <field name="program_id" ref="loyalty.10_percent_coupon"/>
    </record>

    <record id="10_percent_coupon_communication" model="loyalty.mail">
        <field name="trigger">create</field>
        <field name="mail_template_id" ref="loyalty.mail_template_loyalty_card"/>
        <field name="program_id" ref="loyalty.10_percent_coupon"/>
    </record>
    <!-- Gift Cards -->
    <record id="gift_card_program" model="loyalty.program">
        <field name="name">Gift Cards</field>
        <field name="program_type">gift_card</field>
        <field name="applies_on">future</field>
        <field name="trigger">auto</field>
        <field name="portal_visible">True</field>
        <field name="portal_point_name">$</field>
        <field name="mail_template_id" ref="loyalty.mail_template_gift_card"/>
    </record>

    <record id="gift_card_program_rule" model="loyalty.rule">
        <field name="reward_point_amount">1</field>
        <field name="reward_point_mode">money</field>
        <field name="reward_point_split">True</field>
        <field name="product_ids" eval="[(4, ref('loyalty.gift_card_product_50'))]"/>
        <field name="program_id" ref="loyalty.gift_card_program"/>
    </record>

    <record id="gift_card_program_reward" model="loyalty.reward">
        <field name="reward_type">discount</field>
        <field name="discount_mode">per_point</field>
        <field name="discount">1</field>
        <field name="discount_applicability">order</field>
        <field name="required_points">1</field>
        <field name="program_id" ref="loyalty.gift_card_program"/>
    </record>
</odoo>

```

## File: data\mail_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="mail_template_gift_card" model="mail.template">
        <field name="name">Gift Card: Gift Card Information</field>
        <field name="model_id" ref="model_loyalty_card"/>
        <field name="subject">Your Gift Card at {{ object.company_id.name }}</field>
        <field name="partner_to">{{ object._get_mail_partner().id }}</field>
        <field name="lang">{{ object._get_mail_partner().lang }}</field>
        <field name="description">Sent to customer who purchased a gift card</field>
        <field name="body_html" type="html">
            <div style="background: #ffffff">
                <div style="margin:0px; font-size:24px; font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:36px; color:#333333; text-align: center">
                    Here is your gift card!
                </div>
                <div style="padding-top:20px; padding-bottom:20px">
                    <img src="/loyalty/static/img/gift_card.png" style="display:block; border:0; outline:none; text-decoration:none; margin:auto;" width="300"/>
                </div>
                <div style="padding:0; margin:0px; padding-top:35px; padding-bottom:35px; text-align:center;">
                    <h3 style="margin:0px; line-height:48px; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:40px; font-style:normal; font-weight:normal; color:#333333; text-align:center">
                        <strong t-out="format_amount(object.points, object.currency_id) or ''">$ 150.00</strong></h3>
                </div>
                <div style="padding:0; margin:0px; padding-top:35px; padding-bottom:35px; background-color:#efefef; text-align:center;">
                    <p style="margin:0px; font-size:14px;font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:21px; color:#333333">
                        <strong>Gift Card Code</strong>
                    </p>
                    <p style="margin:0px; font-size:25px;font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:38px; color:#A9A9A9" t-out="object.code or ''">4f10-15d6-41b7-b04c-7b3e</p>
                </div>
                <div t-if="object.expiration_date" style="padding:0; margin:0px; padding-top:10px; padding-bottom:10px; text-align:center;">
                    <h3 style="margin:0px; line-height:17px; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:14px; font-style:normal; font-weight:normal; color:#A9A9A9; text-align:center">Card expires <t t-out="format_date(object.expiration_date) or ''">05/05/2021</t></h3>
                </div>
                <div style="padding:20px; margin:0px; text-align:center;">
                    <span style="background-color:#999999; display:inline-block; width:auto; border-radius:5px;">
                        <a t-attf-href="{{ object.get_base_url() }}/shop" target="_blank" style="text-decoration:none; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:22px; color:#FFFFFF; border-style:solid; border-color:#999999; border-width:20px 30px; display:inline-block; background-color:#999999; border-radius:5px; font-weight:bold; font-style:normal; line-height:26px; width:auto; text-align:center">Use it right now!</a>
                    </span>
                </div>
            </div>
        </field>
        <field name="report_template" ref="loyalty.report_gift_card"/>
        <field name="report_name">Your Gift Card</field>
        <field name="auto_delete" eval="True"/>
    </record>

    <record id="mail_template_loyalty_card" model="mail.template">
        <field name="name">Coupon: Coupon Information</field>
        <field name="model_id" ref="loyalty.model_loyalty_card"/>
        <field name="subject">Your reward coupon from {{ object.program_id.company_id.name }} </field>
        <field name="email_from">{{ object.program_id.company_id.email }}</field>
        <field name="partner_to">{{ object._get_mail_partner().id }}</field>
        <field name="lang">{{ object._get_mail_partner().lang }}</field>
        <field name="description">Sent to customer with coupon information</field>
        <field name="body_html" type="html">
<table border="0" cellpadding="0" cellspacing="0" style="width:100%; margin:0px auto; background:#ffffff; color:#333333;"><tbody>
<tr>
    <td valign="top" style="text-align: center; font-size: 14px;">
        <t t-if="object._get_mail_partner().name">
            Congratulations <t t-out="object._get_mail_partner().name or ''">Brandon Freeman</t>,<br />
        </t>

        Here is your reward from <t t-out="object.program_id.company_id.name or ''">YourCompany</t>.<br />

        <t t-foreach="object.program_id.reward_ids" t-as="reward">
            <t t-if="reward.required_points &lt;= object.points">
                <span style="font-size: 50px; color: #875A7B; font-weight: bold;" t-esc="reward.description">Reward Description</span>
                <br/>
            </t>
        </t>
    </td>
</tr>
<tr style="margin-top: 16px">
    <td valign="top" style="text-align: center; font-size: 14px;">
        Use this promo code
        <t t-if="object.expiration_date">
            before <t t-out="object.expiration_date or ''">2021-06-16</t>
        </t>
        <p style="margin-top: 16px;">
            <strong style="padding: 16px 8px 16px 8px; border-radius: 3px; background-color: #F1F1F1;" t-out="object.code or ''">15637502648479132902</strong>
        </p>
        <t t-foreach="object.program_id.rule_ids" t-as="rule">
            <t t-if="rule.minimum_qty not in [0, 1]">
                <span style="font-size: 14px;">
                    Minimum purchase of <t t-out="rule.minimum_qty or ''">10</t> products
                </span><br />
            </t>
            <t t-if="rule.minimum_amount != 0.00">
                <span style="font-size: 14px;">
                    Valid for purchase above <t t-out="rule.company_id.currency_id.symbol or ''">$</t><t t-out="'%0.2f' % float(rule.minimum_amount) or ''">10.00</t>
                </span><br />
            </t>
        </t>
        <br/>
        Thank you,
        <t t-if="object._get_signature()">
            <br />
            <t t-out="object._get_signature() or ''">--<br/>Mitchell Admin</t>
        </t>
    </td>
</tr>
</tbody></table>
        </field>
        <field name="report_template" ref="loyalty.report_loyalty_card"/>
        <field name="report_name">Your Coupon Code</field>
        <field name="auto_delete" eval="True"/>
    </record>
</odoo>

```

## File: models\loyalty_card.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from uuid import uuid4

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.tools import format_amount


class LoyaltyCard(models.Model):
    _name = 'loyalty.card'
    _inherit = ['mail.thread']
    _description = 'Loyalty Coupon'
    _rec_name = 'code'

    @api.model
    def _generate_code(self):
        """
        Barcode identifiable codes.
        """
        return '044' + str(uuid4())[7:-18]

    def name_get(self):
        return [(card.id, f'{card.program_id.name}: {card.code}') for card in self]

    program_id = fields.Many2one('loyalty.program', ondelete='restrict', default=lambda self: self.env.context.get('active_id', None))
    program_type = fields.Selection(related='program_id.program_type')
    company_id = fields.Many2one(related='program_id.company_id', store=True)
    currency_id = fields.Many2one(related='program_id.currency_id')
    # Reserved for this partner if non-empty
    partner_id = fields.Many2one('res.partner', index=True)
    points = fields.Float(tracking=True)
    point_name = fields.Char(related='program_id.portal_point_name', readonly=True)
    points_display = fields.Char(compute='_compute_points_display')

    code = fields.Char(default=lambda self: self._generate_code(), required=True)
    expiration_date = fields.Date()

    use_count = fields.Integer(compute='_compute_use_count')

    _sql_constraints = [
        ('card_code_unique', 'UNIQUE(code)', 'A coupon/loyalty card must have a unique code.')
    ]

    @api.constrains('code')
    def _contrains_code(self):
        # Prevent a coupon from having the same code a program
        if self.env['loyalty.rule'].search_count([('mode', '=', 'with_code'), ('code', 'in', self.mapped('code'))]):
            raise ValidationError(_('A trigger with the same code as one of your coupon already exists.'))

    @api.depends('points', 'point_name')
    def _compute_points_display(self):
        for card in self:
            card.points_display = card._format_points(card.points)

    @api.onchange('expiration_date')
    def _restrict_expiration_on_loyalty(self):
        for card in self:
            if card.program_type == 'loyalty' and card.expiration_date:
                raise ValidationError(_("Expiration date cannot be set on a loyalty card."))

    def _format_points(self, points):
        self.ensure_one()
        if self.point_name == self.program_id.currency_id.symbol:
            return format_amount(self.env, points, self.program_id.currency_id)
        if points == int(points):
            return f"{int(points)} {self.point_name or ''}"
        return f"{points:.2f} {self.point_name or ''}"

    # Meant to be overriden
    def _compute_use_count(self):
        self.use_count = 0

    def _get_default_template(self):
        self.ensure_one()
        return self.program_id.communication_plan_ids.filtered(lambda m: m.trigger == 'create').mail_template_id[:1]

    def _get_mail_partner(self):
        self.ensure_one()
        return self.partner_id

    def _get_mail_author(self):
        self.ensure_one()
        return (
            self.env.user._is_internal() and self.env.user or self.company_id or self.env.company
        ).partner_id

    def _get_signature(self):
        """To be overriden"""
        self.ensure_one()
        return None

    def _has_source_order(self):
        return False

    def action_coupon_send(self):
        """ Open a window to compose an email, with the default template returned by `_get_default_template`
            message loaded by default
        """
        self.ensure_one()
        default_template = self._get_default_template()
        compose_form = self.env.ref('mail.email_compose_message_wizard_form', False)
        ctx = dict(
            default_model='loyalty.card',
            default_res_id=self.id,
            default_use_template=bool(default_template),
            default_template_id=default_template and default_template.id,
            default_composition_mode='comment',
            default_email_layout_xmlid='mail.mail_notification_light',
            mark_coupon_as_sent=True,
            force_email=True,
        )
        return {
            'name': _('Compose Email'),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'mail.compose.message',
            'views': [(compose_form.id, 'form')],
            'view_id': compose_form.id,
            'target': 'new',
            'context': ctx,
        }

    def _send_creation_communication(self, force_send=False):
        """
        Sends the 'At Creation' communication plan if it exist for the given coupons.
        """
        if self.env.context.get('loyalty_no_mail', False) or self.env.context.get('action_no_send_mail', False):
            return
        # Ideally one per program, but multiple is supported
        create_comm_per_program = dict()
        for program in self.program_id:
            create_comm_per_program[program] = program.communication_plan_ids.filtered(lambda c: c.trigger == 'create')
        for coupon in self:
            if not create_comm_per_program[coupon.program_id] or not coupon._get_mail_partner():
                continue
            for comm in create_comm_per_program[coupon.program_id]:
                mail_template = comm.mail_template_id
                email_values = {}
                if not mail_template.email_from:
                    # provide author_id & email_from values to ensure the email gets sent
                    author = coupon._get_mail_author()
                    email_values.update(author_id=author.id, email_from=author.email_formatted)
                mail_template.send_mail(
                    res_id=coupon.id,
                    force_send=force_send,
                    email_layout_xmlid='mail.mail_notification_light',
                    email_values=email_values,
                )

    def _send_points_reach_communication(self, points_changes):
        """
        Send the 'When Reaching' communicaton plans for the given coupons.

        If a coupons passes multiple milestones we will only send the one with the highest target.
        """
        if self.env.context.get('loyalty_no_mail', False):
            return
        milestones_per_program = dict()
        for program in self.program_id:
            milestones_per_program[program] = program.communication_plan_ids\
                .filtered(lambda c: c.trigger == 'points_reach')\
                .sorted('points', reverse=True)
        for coupon in self:
            if not coupon._get_mail_partner():
                continue
            coupon_change = points_changes[coupon]
            # Do nothing if coupon lost points or did not change
            if not milestones_per_program[coupon.program_id] or\
                not coupon.partner_id or\
                coupon_change['old'] >= coupon_change['new']:
                continue
            this_milestone = False
            for milestone in milestones_per_program[coupon.program_id]:
                if coupon_change['old'] < milestone.points and milestone.points <= coupon_change['new']:
                    this_milestone = milestone
                    break
            if not this_milestone:
                continue
            this_milestone.mail_template_id.send_mail(res_id=coupon.id, email_layout_xmlid='mail.mail_notification_light')


    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        res._send_creation_communication()
        return res

    def write(self, vals):
        if not self.env.context.get('loyalty_no_mail', False) and 'points' in vals:
            points_before = {coupon: coupon.points for coupon in self}
        res = super().write(vals)
        if not self.env.context.get('loyalty_no_mail', False) and 'points' in vals:
            points_changes = {coupon: {'old': points_before[coupon], 'new': coupon.points} for coupon in self}
            self._send_points_reach_communication(points_changes)
        return res

```

## File: models\loyalty_mail.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

# Allow promo programs to send mails upon certain triggers
# Like : 'At creation' and 'When reaching X points'

class LoyaltyMail(models.Model):
    _name = 'loyalty.mail'
    _description = 'Loyalty Communication'

    active = fields.Boolean(default=True)
    program_id = fields.Many2one('loyalty.program', required=True, ondelete='cascade')
    trigger = fields.Selection([
        ('create', 'At Creation'),
        ('points_reach', 'When Reaching')], string='When', required=True
    )
    points = fields.Float()
    mail_template_id = fields.Many2one('mail.template', string="Email Template", required=True, domain=[('model', '=', 'loyalty.card')])

```

## File: models\loyalty_program.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from collections import defaultdict

from odoo import _, api, fields, models
from odoo.exceptions import UserError, ValidationError

from uuid import uuid4

class LoyaltyProgram(models.Model):
    _name = 'loyalty.program'
    _description = 'Loyalty Program'
    _order = 'sequence'
    _rec_name = 'name'

    name = fields.Char('Program Name', required=True, translate=True)
    active = fields.Boolean(default=True)
    sequence = fields.Integer(copy=False)
    company_id = fields.Many2one('res.company', 'Company', default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', 'Currency', compute='_compute_currency_id',
        readonly=False, required=True, store=True, precompute=True)
    currency_symbol = fields.Char(related='currency_id.symbol')

    total_order_count = fields.Integer("Total Order Count", compute="_compute_total_order_count")

    rule_ids = fields.One2many('loyalty.rule', 'program_id', 'Conditional rules', copy=True,
         compute='_compute_from_program_type', readonly=False, store=True)
    reward_ids = fields.One2many('loyalty.reward', 'program_id', 'Rewards', copy=True,
         compute='_compute_from_program_type', readonly=False, store=True)
    communication_plan_ids = fields.One2many('loyalty.mail', 'program_id', copy=True,
         compute='_compute_from_program_type', readonly=False, store=True)

    # These fields are used for the simplified view of gift_card and ewallet
    mail_template_id = fields.Many2one('mail.template', compute='_compute_mail_template_id', inverse='_inverse_mail_template_id', string="Email template", readonly=False)
    trigger_product_ids = fields.Many2many(related='rule_ids.product_ids', readonly=False)

    coupon_ids = fields.One2many('loyalty.card', 'program_id')
    coupon_count = fields.Integer(compute='_compute_coupon_count')
    coupon_count_display = fields.Char(compute='_compute_coupon_count_display', string="Items")

    program_type = fields.Selection([
        ('coupons', 'Coupons'),
        ('gift_card', 'Gift Card'),
        ('loyalty', 'Loyalty Cards'),
        ('promotion', 'Promotions'),
        ('ewallet', 'eWallet'),
        ('promo_code', 'Discount Code'),
        ('buy_x_get_y', 'Buy X Get Y'),
        ('next_order_coupons', 'Next Order Coupons')],
        default='promotion', required=True,
    )
    date_to = fields.Date(string='Validity')
    limit_usage = fields.Boolean(string='Limit Usage')
    max_usage = fields.Integer()
    # Dictates when the points can be used:
    # current: if the order gives enough points on that order, the reward may directly be claimed, points lost otherwise
    # future: if the order gives enough points on that order, a coupon is generated for a next order
    # both: points are accumulated on the coupon to claim rewards, the reward may directly be claimed
    applies_on = fields.Selection([
        ('current', 'Current order'),
        ('future', 'Future orders'),
        ('both', 'Current & Future orders')], default='current', required=True,
         compute='_compute_from_program_type', readonly=False, store=True,
    )
    trigger = fields.Selection([
        ('auto', 'Automatic'),
        ('with_code', 'Use a code')],
        compute='_compute_from_program_type', readonly=False, store=True,
        help="""
        Automatic: Customers will be eligible for a reward automatically in their cart.
        Use a code: Customers will be eligible for a reward if they enter a code.
        """
    )
    portal_visible = fields.Boolean(default=False,
        help="""
        Show in web portal, PoS customer ticket, eCommerce checkout, the number of points available and used by reward.
        """)
    portal_point_name = fields.Char(default='Points', translate=True,
         compute='_compute_portal_point_name', readonly=False, store=True)
    is_nominative = fields.Boolean(compute='_compute_is_nominative')
    is_payment_program = fields.Boolean(compute='_compute_is_payment_program')

    payment_program_discount_product_id = fields.Many2one(
        'product.product',
        string='Discount Product',
        compute='_compute_payment_program_discount_product_id',
        readonly=True,
        help="Product used in the sales order to apply the discount."
    )

    # Technical field used for a label
    available_on = fields.Boolean("Available On", store=False,
        help="""
        Manage where your program should be available for use.
        """
    )

    _sql_constraints = [
        ('check_max_usage', 'CHECK (limit_usage = False OR max_usage > 0)',
            'Max usage must be strictly positive if a limit is used.'),
    ]

    @api.constrains('reward_ids')
    def _constrains_reward_ids(self):
        if self.env.context.get('loyalty_skip_reward_check'):
            return
        if any(not program.reward_ids for program in self):
            raise ValidationError(_('A program must have at least one reward.'))

    def _compute_total_order_count(self):
        self.total_order_count = 0

    @api.depends('coupon_count', 'program_type')
    def _compute_coupon_count_display(self):
        program_items_name = self._program_items_name()
        for program in self:
            program.coupon_count_display = "%i %s" % (program.coupon_count or 0, program_items_name[program.program_type] or '')

    @api.depends("communication_plan_ids.mail_template_id")
    def _compute_mail_template_id(self):
        for program in self:
            program.mail_template_id = program.communication_plan_ids.mail_template_id[:1]

    def _inverse_mail_template_id(self):
        for program in self:
            if program.program_type not in ("gift_card", "ewallet"):
                continue
            if not program.mail_template_id:
                program.communication_plan_ids = [(5, 0, 0)]
            elif not program.communication_plan_ids:
                program.communication_plan_ids = self.env['loyalty.mail'].create({
                    'program_id': program.id,
                    'trigger': 'create',
                    'mail_template_id': program.mail_template_id.id,
                })
            else:
                program.communication_plan_ids.write({
                    'trigger': 'create',
                    'mail_template_id': program.mail_template_id.id,
                })

    @api.depends('company_id')
    def _compute_currency_id(self):
        for program in self:
            program.currency_id = program.company_id.currency_id or program.currency_id

    @api.depends('coupon_ids')
    def _compute_coupon_count(self):
        read_group_data = self.env['loyalty.card']._read_group([('program_id', 'in', self.ids)], ['program_id'], ['program_id'])
        count_per_program = {r['program_id'][0]: r['program_id_count'] for r in read_group_data}
        for program in self:
            program.coupon_count = count_per_program.get(program.id, 0)

    @api.depends('program_type', 'applies_on')
    def _compute_is_nominative(self):
        for program in self:
            program.is_nominative = program.applies_on == 'both' or\
                (program.program_type == 'ewallet' and program.applies_on == 'future')

    @api.depends('program_type')
    def _compute_is_payment_program(self):
        for program in self:
            program.is_payment_program = program.program_type in ('gift_card', 'ewallet')

    @api.depends('reward_ids.discount_line_product_id')
    def _compute_payment_program_discount_product_id(self):
        for program in self:
            if program.is_payment_program:
                program.payment_program_discount_product_id = program.reward_ids[:1].discount_line_product_id
            else:
                program.payment_program_discount_product_id = False

    @api.model
    def _program_items_name(self):
        return {
            'coupons': _('Coupons'),
            'promotion': _('Promos'),
            'gift_card': _('Gift Cards'),
            'loyalty': _('Loyalty Cards'),
            'ewallet': _('eWallets'),
            'promo_code': _('Discounts'),
            'buy_x_get_y': _('Promos'),
            'next_order_coupons': _('Coupons'),
        }

    @api.model
    def _program_type_default_values(self):
        # All values to change when program_type changes
        # NOTE: any field used in `rule_ids`, `reward_ids` and `communication_plan_ids` MUST be present in the kanban view for it to work properly.
        first_sale_product = self.env['product.product'].search([
            '|', ('company_id', '=', False), ('company_id', '=', self.company_id.id),
            ('sale_ok', '=', True)
        ], limit=1)
        return {
            'coupons': {
                'applies_on': 'current',
                'trigger': 'with_code',
                'portal_visible': False,
                'portal_point_name': _('Coupon point(s)'),
                'rule_ids': [(5, 0, 0)],
                'reward_ids': [(5, 0, 0), (0, 0, {
                    'required_points': 1,
                    'discount': 10,
                })],
                'communication_plan_ids': [(5, 0, 0), (0, 0, {
                    'trigger': 'create',
                    'mail_template_id': (self.env.ref('loyalty.mail_template_loyalty_card', raise_if_not_found=False) or self.env['mail.template']).id,
                })],
            },
            'promotion': {
                'applies_on': 'current',
                'trigger': 'auto',
                'portal_visible': False,
                'portal_point_name': _('Promo point(s)'),
                'rule_ids': [(5, 0, 0), (0, 0, {
                    'reward_point_amount': 1,
                    'reward_point_mode': 'order',
                    'minimum_amount': 50,
                    'minimum_qty': 0,
                })],
                'reward_ids': [(5, 0, 0), (0, 0, {
                    'required_points': 1,
                    'discount': 10,
                })],
                'communication_plan_ids': [(5, 0, 0)],
            },
            'gift_card': {
                'applies_on': 'future',
                'trigger': 'auto',
                'portal_visible': True,
                'portal_point_name': self.env.company.currency_id.symbol,
                'rule_ids': [(5, 0, 0), (0, 0, {
                    'reward_point_amount': 1,
                    'reward_point_mode': 'money',
                    'reward_point_split': True,
                    'product_ids': self.env.ref('loyalty.gift_card_product_50', raise_if_not_found=False),
                    'minimum_qty': 0,
                })],
                'reward_ids': [(5, 0, 0), (0, 0, {
                    'reward_type': 'discount',
                    'discount_mode': 'per_point',
                    'discount': 1,
                    'discount_applicability': 'order',
                    'required_points': 1,
                    'description': _('Gift Card'),
                })],
                'communication_plan_ids': [(5, 0, 0), (0, 0, {
                    'trigger': 'create',
                    'mail_template_id': (self.env.ref('loyalty.mail_template_gift_card', raise_if_not_found=False) or self.env['mail.template']).id,
                })],
            },
            'loyalty': {
                'applies_on': 'both',
                'trigger': 'auto',
                'portal_visible': True,
                'portal_point_name': _('Loyalty point(s)'),
                'rule_ids': [(5, 0, 0), (0, 0, {
                    'reward_point_mode': 'money',
                })],
                'reward_ids': [(5, 0, 0), (0, 0, {
                    'discount': 5,
                    'required_points': 200,
                })],
                'communication_plan_ids': [(5, 0, 0)],
            },
            'ewallet': {
                'trigger': 'auto',
                'applies_on': 'future',
                'portal_visible': True,
                'portal_point_name': self.env.company.currency_id.symbol,
                'rule_ids': [(5, 0, 0), (0, 0, {
                    'reward_point_amount': '1',
                    'reward_point_mode': 'money',
                    'product_ids': self.env.ref('loyalty.ewallet_product_50', raise_if_not_found=False),
                })],
                'reward_ids': [(5, 0, 0), (0, 0, {
                    'reward_type': 'discount',
                    'discount_mode': 'per_point',
                    'discount': 1,
                    'discount_applicability': 'order',
                    'required_points': 1,
                    'description': _('eWallet'),
                })],
                'communication_plan_ids': [(5, 0, 0)],
            },
            'promo_code': {
                'applies_on': 'current',
                'trigger': 'with_code',
                'portal_visible': False,
                'portal_point_name': _('Discount point(s)'),
                'rule_ids': [(5, 0, 0), (0, 0, {
                    'mode': 'with_code',
                    'code': 'PROMO_CODE_' + str(uuid4())[:4], # We should try not to trigger any unicity constraint
                    'minimum_qty': 0,
                })],
                'reward_ids': [(5, 0, 0), (0, 0, {
                    'discount_applicability': 'specific',
                    'discount_product_ids': first_sale_product,
                    'discount_mode': 'percent',
                    'discount': 10,
                })],
                'communication_plan_ids': [(5, 0, 0)],
            },
            'buy_x_get_y': {
                'applies_on': 'current',
                'trigger': 'auto',
                'portal_visible': False,
                'portal_point_name': _('Credit(s)'),
                'rule_ids': [(5, 0, 0), (0, 0, {
                    'reward_point_mode': 'unit',
                    'product_ids': first_sale_product,
                    'minimum_qty': 2,
                })],
                'reward_ids': [(5, 0, 0), (0, 0, {
                    'reward_type': 'product',
                    'reward_product_id': first_sale_product.id,
                    'required_points': 2,
                })],
                'communication_plan_ids': [(5, 0, 0)],
            },
            'next_order_coupons': {
                'applies_on': 'future',
                'trigger': 'auto',
                'portal_visible': True,
                'portal_point_name': _('Coupon point(s)'),
                'rule_ids': [(5, 0, 0), (0, 0, {
                    'minimum_amount': 100,
                    'minimum_qty': 0,
                })],
                'reward_ids': [(5, 0, 0), (0, 0, {
                    'reward_type': 'discount',
                    'discount_mode': 'percent',
                    'discount': 15,
                    'discount_applicability': 'order',
                })],
                'communication_plan_ids': [(5, 0, 0), (0, 0, {
                    'trigger': 'create',
                    'mail_template_id': (
                        self.env.ref('loyalty.mail_template_loyalty_card', raise_if_not_found=False)
                        or self.env['mail.template']
                    ).id,
                })],
            },
        }

    @api.depends('program_type')
    def _compute_from_program_type(self):
        program_type_defaults = self._program_type_default_values()
        grouped_programs = defaultdict(lambda: self.env['loyalty.program'])
        for program in self:
            grouped_programs[program.program_type] |= program
        for program_type, programs in grouped_programs.items():
            if program_type in program_type_defaults:
                programs.write(program_type_defaults[program_type])

    @api.depends("currency_id", "program_type")
    def _compute_portal_point_name(self):
        for program in self:
            if program.program_type not in ('ewallet', 'gift_card'):
                continue
            program.portal_point_name = program.currency_id.symbol or ''

    def _get_valid_products(self, products):
        '''
        Returns a dict containing the products that match per rule of the program
        '''
        rule_products = dict()
        for rule in self.rule_ids:
            domain = rule._get_valid_product_domain()
            if domain:
                rule_products[rule] = products.filtered_domain(domain)
            elif not domain and rule.program_type != "gift_card":
                rule_products[rule] = products
            else:
                continue
        return rule_products

    def action_open_loyalty_cards(self):
        self.ensure_one()
        action = self.env['ir.actions.act_window']._for_xml_id("loyalty.loyalty_card_action")
        action['name'] = self._program_items_name()[self.program_type]
        action['display_name'] = action['name']
        action['context'] = {
            'program_type': self.program_type,
            'program_item_name': self._program_items_name()[self.program_type],
            'default_program_id': self.id,
            # For the wizard
            'default_mode': self.program_type == 'ewallet' and 'selected' or 'anonymous',
        }
        return action

    @api.ondelete(at_uninstall=False)
    def _unlink_except_active(self):
        if any(program.active for program in self):
            raise UserError(_('You can not delete a program in an active state'))

    def toggle_active(self):
        res = super().toggle_active()
        # Propagate active state to children
        for program in self.with_context(active_test=False):
            program.rule_ids.active = program.active
            program.reward_ids.active = program.active
            program.communication_plan_ids.active = program.active
            program.reward_ids.with_context(active_test=True).discount_line_product_id.active = program.active
        return res

    def write(self, vals):
        # There is an issue when we change the program type, since we clear the rewards and create new ones.
        # The orm actually does it in this order upon writing, triggering the constraint before creating the new rewards.
        # However we can check that the result of reward_ids would actually be empty or not, and if not, skip the constraint.
        if 'reward_ids' in vals and self._fields['reward_ids'].convert_to_cache(vals['reward_ids'], self):
            self = self.with_context(loyalty_skip_reward_check=True)
            # We need add the program type to the context to avoid getting the default value
            # ('discount') for reward type when calling the `default_get` method of
            #`loyalty.reward`.
            if 'program_type' in vals:
                self = self.with_context(program_type=vals['program_type'])
                return super().write(vals)
            else:
                for program in self:
                    program = program.with_context(program_type=program.program_type)
                    super(LoyaltyProgram, program).write(vals)
                return True
        else:
            return super().write(vals)

    @api.model
    def get_program_templates(self):
        '''
        Returns the templates to be used for promotional programs.
        '''
        ctx_menu_type = self.env.context.get('menu_type')
        if ctx_menu_type == 'gift_ewallet':
            return {
                'gift_card': {
                    'title': _("Gift Card"),
                    'description': _("Sell Gift Cards, that can be used to purchase products."),
                    'icon': 'gift_card',
                },
                'ewallet': {
                    'title': _("eWallet"),
                    'description': _("Fill in your eWallet, and use it to pay future orders."),
                    'icon': 'ewallet',
                },
            }
        return {
            'promotion': {
                'title': _("Promotion Program"),
                'description': _(
                    "Define promotions to apply automatically on your customers' orders."
                ),
                'icon': 'promotional_program',
            },
            'promo_code': {
                'title': _("Discount Code"),
                'description': _(
                    "Share a discount code with your customers to create a purchase incentive."
                ),
                'icon': 'promo_code',
            },
            'buy_x_get_y': {
                'title': _("Buy X Get Y"),
                'description': _(
                    "Offer Y to your customers if they are buying X; for example, 2+1 free."
                ),
                'icon': '2_plus_1',
            },
            'next_order_coupons': {
                'title': _("Next Order Coupons"),
                'description': _(
                    "Reward your customers for a purchase with a coupon to use on their next order."
                ),
                'icon': 'coupons',
            },
            'loyalty': {
                'title': _("Loyalty Cards"),
                'description': _("Win points with each purchase, and use points to get gifts."),
                'icon': 'loyalty_cards',
            },
            'coupons': {
                'title': _("Coupons"),
                'description': _("Generate and share unique coupons with your customers."),
                'icon': 'coupons',
            },
            'fidelity': {
                'title': _("Fidelity Cards"),
                'description': _("Buy 10 products, and get 10$ discount on the 11th one."),
                'icon': 'fidelity_cards',
            },
        }

    @api.model
    def create_from_template(self, template_id):
        '''
        Creates the program from the template id defined in `get_program_templates`.

        Returns an action leading to that new record.
        '''
        template_values = self._get_template_values()
        if template_id not in template_values:
            return False
        program = self.create(template_values[template_id])
        action = {}
        if self.env.context.get('menu_type') == 'gift_ewallet':
            action = self.env['ir.actions.act_window']._for_xml_id('loyalty.loyalty_program_gift_ewallet_action')
            action['views'] = [[False, 'form']]
        else:
            action = self.env['ir.actions.act_window']._for_xml_id('loyalty.loyalty_program_discount_loyalty_action')
            view_id = self.env.ref('loyalty.loyalty_program_view_form').id
            action['views'] = [[view_id, 'form']]
        action['view_mode'] = 'form'
        action['res_id'] = program.id
        return action

    @api.model
    def _get_template_values(self):
        '''
        Returns the values to create a program using the template keys defined above.
        '''
        program_type_defaults = self._program_type_default_values()
        # For programs that require a product get the first sellable.
        product = self.env['product.product'].search([('sale_ok', '=', True)], limit=1)
        return {
            'gift_card': {
                'name': _('Gift Card'),
                'program_type': 'gift_card',
                **program_type_defaults['gift_card']
            },
            'ewallet': {
                'name': _('eWallet'),
                'program_type': 'ewallet',
                **program_type_defaults['ewallet'],
            },
            'loyalty': {
                'name': _('Loyalty Cards'),
                'program_type': 'loyalty',
                **program_type_defaults['loyalty'],
            },
            'coupons': {
                'name': _('Coupons'),
                'program_type': 'coupons',
                **program_type_defaults['coupons'],
            },
            'promotion': {
                'name': _('Promotional Program'),
                'program_type': 'promotion',
                **program_type_defaults['promotion'],
            },
            'promo_code': {
                'name': _('Discount code'),
                'program_type': 'promo_code',
                **program_type_defaults['promo_code'],
            },
            'buy_x_get_y': {
                'name': _('2+1 Free'),
                'program_type': 'buy_x_get_y',
                **program_type_defaults['buy_x_get_y'],
            },
            'next_order_coupons': {
                'name': _('Next Order Coupons'),
                'program_type': 'next_order_coupons',
                **program_type_defaults['next_order_coupons'],
            },
            'fidelity': {
                'name': _('Fidelity Cards'),
                'program_type': 'loyalty',
                'applies_on': 'both',
                'trigger': 'auto',
                'rule_ids': [(0, 0, {
                    'reward_point_mode': 'unit',
                    'product_ids': product,
                })],
                'reward_ids': [(0, 0, {
                    'discount_mode': 'per_order',
                    'required_points': 11,
                    'discount_applicability': 'specific',
                    'discount_product_ids': product,
                    'discount': 10,
                })]
            },
        }

    @api.model_create_multi
    def create(self, vals_list):
        """
        trigger_product_ids will overwrite product ids defined in a loyalty rule in certain instances. Thus, it should
        be explicitly removed from an incoming vals dict unless, of course, it was actually a visible field.
        """
        for vals in vals_list:
            if 'trigger_product_ids' in vals and vals.get('program_type') not in ['gift_card', 'ewallet']:
                del vals['trigger_product_ids']

        return super().create(vals_list)

```

## File: models\loyalty_reward.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import json

from odoo import _, api, fields, models
from odoo.exceptions import UserError
from odoo.osv import expression

class LoyaltyReward(models.Model):
    _name = 'loyalty.reward'
    _description = 'Loyalty Reward'
    _rec_name = 'description'
    _order = 'required_points asc'

    @api.model
    def default_get(self, fields_list):
        # Try to copy the values of the program types default's
        result = super().default_get(fields_list)
        if 'program_type' in self.env.context:
            program_type = self.env.context['program_type']
            program_default_values = self.env['loyalty.program']._program_type_default_values()
            if program_type in program_default_values and\
                len(program_default_values[program_type]['reward_ids']) == 2 and\
                isinstance(program_default_values[program_type]['reward_ids'][1][2], dict):
                result.update({
                    k: v for k, v in program_default_values[program_type]['reward_ids'][1][2].items() if k in fields_list
                })
        return result

    def _get_discount_mode_select(self):
        # The value is provided in the loyalty program's view since we may not have a program_id yet
        #  and makes sure to display the currency related to the program instead of the company's.
        symbol = self.env.context.get('currency_symbol', self.env.company.currency_id.symbol)
        return [
            ('percent', '%'),
            ('per_point', _('%s per point', symbol)),
            ('per_order', _('%s per order', symbol))
        ]

    def name_get(self):
        return [(reward.id, '%s - %s' % (reward.program_id.name, reward.description)) for reward in self]

    active = fields.Boolean(default=True)
    program_id = fields.Many2one('loyalty.program', required=True, ondelete='cascade')
    program_type = fields.Selection(related="program_id.program_type")
    # Stored for security rules
    company_id = fields.Many2one(related='program_id.company_id', store=True)
    currency_id = fields.Many2one(related='program_id.currency_id')

    description = fields.Char(compute='_compute_description', readonly=False, store=True, translate=True)

    reward_type = fields.Selection([
        ('product', 'Free Product'),
        ('discount', 'Discount')],
        default='discount', required=True,
    )
    user_has_debug = fields.Boolean(compute='_compute_user_has_debug')

    # Discount rewards
    discount = fields.Float('Discount', default=10)
    discount_mode = fields.Selection(selection=_get_discount_mode_select, required=True, default='percent')
    discount_applicability = fields.Selection([
        ('order', 'Order'),
        ('cheapest', 'Cheapest Product'),
        ('specific', 'Specific Products')], default='order',
    )
    discount_product_domain = fields.Char(default="[]")
    discount_product_ids = fields.Many2many('product.product', string="Discounted Products")
    discount_product_category_id = fields.Many2one('product.category', string="Discounted Prod. Categories")
    discount_product_tag_id = fields.Many2one('product.tag', string="Discounted Prod. Tag")
    all_discount_product_ids = fields.Many2many('product.product', compute='_compute_all_discount_product_ids')
    reward_product_domain = fields.Char(compute='_compute_reward_product_domain', store=False)
    discount_max_amount = fields.Monetary('Max Discount', 'currency_id',
        help="This is the max amount this reward may discount, leave to 0 for no limit.")
    discount_line_product_id = fields.Many2one('product.product', copy=False, ondelete='restrict',
        help="Product used in the sales order to apply the discount. Each reward has its own product for reporting purpose")
    is_global_discount = fields.Boolean(compute='_compute_is_global_discount')

    # Product rewards
    reward_product_id = fields.Many2one('product.product', string='Product')
    reward_product_tag_id = fields.Many2one('product.tag', string='Product Tag')
    multi_product = fields.Boolean(compute='_compute_multi_product')
    reward_product_ids = fields.Many2many(
        'product.product', string="Reward Products", compute='_compute_multi_product',
        search='_search_reward_product_ids',
        help="These are the products that can be claimed with this rule.")
    reward_product_qty = fields.Integer(default=1)
    reward_product_uom_id = fields.Many2one('uom.uom', compute='_compute_reward_product_uom_id')

    required_points = fields.Float('Points needed', default=1)
    point_name = fields.Char(related='program_id.portal_point_name', readonly=True)
    clear_wallet = fields.Boolean(default=False)

    _sql_constraints = [
        ('required_points_positive', 'CHECK (required_points > 0)',
            'The required points for a reward must be strictly positive.'),
        ('product_qty_positive', "CHECK (reward_type != 'product' OR reward_product_qty > 0)",
            'The reward product quantity must be strictly positive.'),
        ('discount_positive', "CHECK (reward_type != 'discount' OR discount > 0)",
            'The discount must be strictly positive.'),
    ]

    @api.depends('reward_product_id.product_tmpl_id.uom_id', 'reward_product_tag_id')
    def _compute_reward_product_uom_id(self):
        for reward in self:
            reward.reward_product_uom_id = reward.reward_product_ids.product_tmpl_id.uom_id[:1]

    def _find_all_category_children(self, category_id, child_ids):
        if len(category_id.child_id) > 0:
            for child_id in category_id.child_id:
                child_ids.append(child_id.id)
                self._find_all_category_children(child_id, child_ids)
        return child_ids

    def _get_discount_product_domain(self):
        self.ensure_one()
        domain = []
        if self.discount_product_ids:
            domain = [('id', 'in', self.discount_product_ids.ids)]
        if self.discount_product_category_id:
            product_category_ids = self._find_all_category_children(self.discount_product_category_id, [])
            product_category_ids.append(self.discount_product_category_id.id)
            domain = expression.OR([domain, [('categ_id', 'in', product_category_ids)]])
        if self.discount_product_tag_id:
            domain = expression.OR([domain, [('all_product_tag_ids', 'in', self.discount_product_tag_id.id)]])
        if self.discount_product_domain and self.discount_product_domain != '[]':
            domain = expression.AND([domain, ast.literal_eval(self.discount_product_domain)])
        return domain

    @api.model
    def _get_active_products_domain(self):
        return [
            '|',
                ('reward_type', '!=', 'product'),
                '&',
                    ('reward_type', '=', 'product'),
                    '|',
                        '&',
                            ('reward_product_tag_id', '=', False),
                            ('reward_product_id.active', '=', True),
                        '&',
                            ('reward_product_tag_id', '!=', False),
                            ('reward_product_ids.active', '=', True)
        ]

    @api.depends('discount_product_domain')
    def _compute_reward_product_domain(self):
        compute_all_discount_product = self.env['ir.config_parameter'].sudo().get_param('loyalty.compute_all_discount_product_ids', 'enabled')
        for reward in self:
            if compute_all_discount_product == 'enabled':
                reward.reward_product_domain = "null"
            else:
                reward.reward_product_domain = json.dumps(reward._get_discount_product_domain())

    @api.depends('discount_product_ids', 'discount_product_category_id', 'discount_product_tag_id', 'discount_product_domain')
    def _compute_all_discount_product_ids(self):
        compute_all_discount_product = self.env['ir.config_parameter'].sudo().get_param('loyalty.compute_all_discount_product_ids', 'enabled')
        for reward in self:
            if compute_all_discount_product == 'enabled':
                reward.all_discount_product_ids = self.env['product.product'].search(reward._get_discount_product_domain())
            else:
                reward.all_discount_product_ids = self.env['product.product']

    @api.depends('reward_product_id', 'reward_product_tag_id', 'reward_type')
    def _compute_multi_product(self):
        for reward in self:
            products = reward.reward_product_id + reward.reward_product_tag_id.product_ids
            reward.multi_product = reward.reward_type == 'product' and len(products) > 1
            reward.reward_product_ids = reward.reward_type == 'product' and products or self.env['product.product']

    def _search_reward_product_ids(self, operator, value):
        if operator not in ('=', '!=', 'in'):
            raise NotImplementedError("Unsupported search operator")
        return [
            '&', ('reward_type', '=', 'product'),
            '|', ('reward_product_id', operator, value),
            ('reward_product_tag_id.product_ids', operator, value)
        ]

    @api.depends('reward_type', 'reward_product_id', 'discount_mode', 'reward_product_tag_id',
                 'discount', 'currency_id', 'discount_applicability', 'all_discount_product_ids')
    def _compute_description(self):
        for reward in self:
            reward_string = ""
            if reward.program_type == 'gift_card':
                reward_string = _("Gift Card")
            elif reward.program_type == 'ewallet':
                reward_string = _("eWallet")
            elif reward.reward_type == 'product':
                products = reward.reward_product_ids
                if len(products) == 0:
                    reward_string = _('Free Product')
                elif len(products) == 1:
                    reward_string = _('Free Product - %s', reward.reward_product_id.with_context(display_default_code=False).display_name)
                else:
                    reward_string = _('Free Product - [%s]', ', '.join(products._origin.with_context(display_default_code=False).mapped('display_name')))
            elif reward.reward_type == 'discount':
                format_string = '%(amount)g %(symbol)s'
                if reward.currency_id.position == 'before':
                    format_string = '%(symbol)s %(amount)g'
                formatted_amount = format_string % {'amount': reward.discount, 'symbol': reward.currency_id.symbol}
                if reward.discount_mode == 'percent':
                    reward_string = _('%g%% on ', reward.discount)
                elif reward.discount_mode == 'per_point':
                    reward_string = _('%s per point on ', formatted_amount)
                elif reward.discount_mode == 'per_order':
                    reward_string = _('%s per order on ', formatted_amount)
                if reward.discount_applicability == 'order':
                    reward_string += _('your order')
                elif reward.discount_applicability == 'cheapest':
                    reward_string += _('the cheapest product')
                elif reward.discount_applicability == 'specific':
                    product_available = self.env['product.product'].search(reward._get_discount_product_domain(), limit=2)
                    if len(product_available) == 1:
                        reward_string += product_available.with_context(display_default_code=False).display_name
                    else:
                        reward_string += _('specific products')
                if reward.discount_max_amount:
                    format_string = '%(amount)g %(symbol)s'
                    if reward.currency_id.position == 'before':
                        format_string = '%(symbol)s %(amount)g'
                    formatted_amount = format_string % {'amount': reward.discount_max_amount, 'symbol': reward.currency_id.symbol}
                    reward_string += _(' (Max %s)', formatted_amount)
            reward.description = reward_string

    @api.depends('reward_type', 'discount_applicability', 'discount_mode')
    def _compute_is_global_discount(self):
        for reward in self:
            reward.is_global_discount = reward.reward_type == 'discount' and\
                                        reward.discount_applicability == 'order' and\
                                        reward.discount_mode == 'percent'

    @api.depends_context('uid')
    @api.depends("reward_type")
    def _compute_user_has_debug(self):
        self.user_has_debug = self.user_has_groups('base.group_no_one')

    @api.onchange('description')
    def _ensure_reward_has_description(self):
        for reward in self:
            if not reward.description:
                raise UserError(_("The reward description field cannot be empty."))

    def _create_missing_discount_line_products(self):
        # Make sure we create the product that will be used for our discounts
        rewards = self.filtered(lambda r: not r.discount_line_product_id)
        products = self.env['product.product'].create(rewards._get_discount_product_values())
        for reward, product in zip(rewards, products):
            reward.discount_line_product_id = product

    @api.model_create_multi
    def create(self, vals_list):
        res = super().create(vals_list)
        res._create_missing_discount_line_products()
        return res

    def write(self, vals):
        res = super().write(vals)
        if 'description' in vals:
            self._create_missing_discount_line_products()
            # Keep the name of our discount product up to date
            for reward in self:
                reward.discount_line_product_id.write({'name': reward.description})
        if 'active' in vals:
            if vals['active']:
                self.discount_line_product_id.action_unarchive()
            else:
                self.discount_line_product_id.action_archive()
        return res

    def unlink(self):
        programs = self.program_id
        res = super().unlink()
        # Not guaranteed to trigger the constraint
        programs._constrains_reward_ids()
        return res

    def _get_discount_product_values(self):
        return [{
            'name': reward.description,
            'type': 'service',
            'sale_ok': False,
            'purchase_ok': False,
            'lst_price': 0,
        } for reward in self]

```

## File: models\loyalty_rule.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.osv import expression

class LoyaltyRule(models.Model):
    _name = 'loyalty.rule'
    _description = 'Loyalty Rule'

    @api.model
    def default_get(self, fields_list):
        # Try to copy the values of the program types default's
        result = super().default_get(fields_list)
        if 'program_type' in self.env.context:
            program_type = self.env.context['program_type']
            program_default_values = self.env['loyalty.program']._program_type_default_values()
            if program_type in program_default_values and\
                len(program_default_values[program_type]['rule_ids']) == 2 and\
                isinstance(program_default_values[program_type]['rule_ids'][1][2], dict):
                result.update({
                    k: v for k, v in program_default_values[program_type]['rule_ids'][1][2].items() if k in fields_list
                })
        return result

    def _get_reward_point_mode_selection(self):
        # The value is provided in the loyalty program's view since we may not have a program_id yet
        #  and makes sure to display the currency related to the program instead of the company's.
        symbol = self.env.context.get('currency_symbol', self.env.company.currency_id.symbol)
        return [
            ('order', _('per order')),
            ('money', _('per %s spent', symbol)),
            ('unit', _('per unit paid')),
        ]

    active = fields.Boolean(default=True)
    program_id = fields.Many2one('loyalty.program', required=True, ondelete='cascade')
    program_type = fields.Selection(related="program_id.program_type")
    # Stored for security rules
    company_id = fields.Many2one(related='program_id.company_id', store=True)
    currency_id = fields.Many2one(related='program_id.currency_id')

    # Only for dev mode
    user_has_debug = fields.Boolean(compute='_compute_user_has_debug')
    product_domain = fields.Char(default="[]")

    product_ids = fields.Many2many('product.product', string='Products')
    product_category_id = fields.Many2one('product.category', string='Categories')
    product_tag_id = fields.Many2one('product.tag', string='Product Tag')

    reward_point_amount = fields.Float(default=1, string="Reward")
    # Only used for program_id.applies_on == 'future'
    reward_point_split = fields.Boolean(string='Split per unit', default=False,
        help="Whether to separate reward coupons per matched unit, only applies to 'future' programs and trigger mode per money spent or unit paid..")
    reward_point_name = fields.Char(related='program_id.portal_point_name', readonly=True)
    reward_point_mode = fields.Selection(selection=_get_reward_point_mode_selection, required=True, default='order')

    minimum_qty = fields.Integer('Minimum Quantity', default=1)
    minimum_amount = fields.Monetary('Minimum Purchase', 'currency_id')
    minimum_amount_tax_mode = fields.Selection([
        ('incl', 'Included'),
        ('excl', 'Excluded')], default='incl', required=True,
    )

    mode = fields.Selection([
        ('auto', 'Automatic'),
        ('with_code', 'With a promotion code'),
    ], string="Application", compute='_compute_mode', store=True, readonly=False)
    code = fields.Char(string='Discount code', compute='_compute_code', store=True, readonly=False)

    _sql_constraints = [
        ('reward_point_amount_positive', 'CHECK (reward_point_amount > 0)', 'Rule points reward must be strictly positive.'),
    ]

    @api.constrains('reward_point_split')
    def _constraint_trigger_multi(self):
        # Prevent setting trigger multi in case of nominative programs, it does not make sense to allow this
        for rule in self:
            if rule.reward_point_split and (rule.program_id.applies_on == 'both' or rule.program_id.program_type == 'ewallet'):
                raise ValidationError(_('Split per unit is not allowed for Loyalty and eWallet programs.'))

    @api.constrains('code')
    def _constrains_code(self):
        mapped_codes = self.filtered('code').mapped('code')
        # Program code must be unique
        if len(mapped_codes) != len(set(mapped_codes)) or\
            self.env['loyalty.rule'].search_count(
                [('mode', '=', 'with_code'), ('code', 'in', mapped_codes), ('id', 'not in', self.ids)]):
            raise ValidationError(_('The promo code must be unique.'))
        # Prevent coupons and programs from sharing a code
        if self.env['loyalty.card'].search_count([('code', 'in', mapped_codes)]):
            raise ValidationError(_('A coupon with the same code was found.'))

    @api.depends('mode')
    def _compute_code(self):
        # Reset code when mode is set to auto
        for rule in self:
            if rule.mode == 'auto':
                rule.code = False

    @api.depends('code')
    def _compute_mode(self):
        for rule in self:
            if rule.code:
                rule.mode = 'with_code'
            else:
                rule.mode = 'auto'

    @api.depends_context('uid')
    @api.depends("mode")
    def _compute_user_has_debug(self):
        self.user_has_debug = self.user_has_groups('base.group_no_one')

    def _get_valid_product_domain(self):
        self.ensure_one()
        domain = []
        if self.product_ids:
            domain = [('id', 'in', self.product_ids.ids)]
        if self.product_category_id:
            domain = expression.OR([domain, [('categ_id', 'child_of', self.product_category_id.id)]])
        if self.product_tag_id:
            domain = expression.OR([domain, [('all_product_tag_ids', 'in', self.product_tag_id.id)]])
        if self.product_domain and self.product_domain != '[]':
            domain = expression.AND([domain, ast.literal_eval(self.product_domain)])
        return domain

    def _get_valid_products(self):
        self.ensure_one()
        return self.env['product.product'].search(self._get_valid_product_domain())

    def _compute_amount(self, currency_to):
        self.ensure_one()
        return self.currency_id._convert(
            self.minimum_amount,
            currency_to,
            self.company_id or self.env.company,
            fields.Date.today()
        )

```

## File: models\product_product.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models
from odoo.exceptions import UserError, ValidationError


class ProductProduct(models.Model):
    _inherit = 'product.product'

    def write(self, vals):
        if not vals.get('active', True) and any(product.active for product in self):
            # Prevent archiving products used for giving rewards
            rewards = self.env['loyalty.reward'].sudo().search([
                ('active', '=', True),
                '|',
                ('discount_line_product_id', 'in', self.ids),
                ('discount_product_ids', 'in', self.ids),
            ], limit=1)
            if rewards:
                raise ValidationError(_("This product may not be archived. It is being used for an active promotion program."))
        return super().write(vals)

    @api.ondelete(at_uninstall=False)
    def _unlink_except_loyalty_products(self):
        product_data = [
            self.env.ref('loyalty.gift_card_product_50', False),
            self.env.ref('loyalty.ewallet_product_50', False),
        ]
        for product in self.filtered(lambda p: p in product_data):
            raise UserError(_(
                "You cannot delete %(name)s as it is used in 'Coupons & Loyalty'."
                " Please archive it instead.",
                name=product.with_context(display_default_code=False).display_name
            ))

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, models
from odoo.exceptions import UserError


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    @api.ondelete(at_uninstall=False)
    def _unlink_except_loyalty_products(self):
        product_data = [
            self.env.ref('loyalty.gift_card_product_50', False),
            self.env.ref('loyalty.ewallet_product_50', False),
        ]
        for product in self.filtered(lambda p: p.product_variant_id in product_data):
            raise UserError(_(
                "You cannot delete %(name)s as it is used in 'Coupons & Loyalty'."
                " Please archive it instead.",
                name=product.with_context(display_default_code=False).display_name
            ))

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import loyalty_card
from . import loyalty_mail
from . import loyalty_reward
from . import loyalty_rule
from . import loyalty_program
from . import product_product
from . import product_template

```

## File: report\loyalty_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="report_loyalty_card" model="ir.actions.report">
            <field name="name">Coupon Code</field>
            <field name="model">loyalty.card</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">loyalty.loyalty_report_i18n</field>
            <field name="report_file">loyalty.loyalty_report_i18n</field>
            <field name="binding_model_id" ref="model_loyalty_card"/>
            <field name="binding_type">report</field>
        </record>

        <record id="report_gift_card" model="ir.actions.report">
            <field name="name">Gift Card</field>
            <field name="model">loyalty.card</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">loyalty.gift_card_report_i18n</field>
            <field name="report_file">loyalty.gift_card_report_i18n</field>
            <field name="binding_model_id" ref="model_loyalty_card"/>
            <field name="binding_type">report</field>
        </record>
    </data>
</odoo>

```

## File: report\loyalty_report_templates.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="loyalty_report">
        <t t-call="web.internal_layout">
            <div class="card">
                <div class="card-body">
                    <div class="page">
                        <div class="row text-center">
                            <div class="o_offer col-lg-12">
                                <h4 t-if="o._get_mail_partner().name">
                                    Congratulations
                                    <t t-esc="o._get_mail_partner().name"/>,
                                </h4>
                                <t t-set="text">on your next order</t>
                                <h4>Here is your reward from <t t-esc="o.program_id.company_id.name"/>.</h4>
                                <t t-foreach="range(len(o.program_id.reward_ids))" t-as="reward_idx">
                                    <t t-set="reward" t-value="o.program_id.reward_ids[reward_idx]"/>
                                    <strong><t t-esc="reward.description"/></strong>
                                    <br/>
                                    <t t-if="reward_idx &lt; (len(o.program_id.reward_ids) - 1)">
                                        <span class="text-center">OR</span>
                                        <br/>
                                    </t>
                                </t>
                                <h1 class="fw-bold" style="font-size: 34px" t-esc="text"/>
                                <br/>
                                <h4 t-if="o.expiration_date">
                                    Use this promo code before
                                    <span t-field="o.expiration_date" t-options='{"format": "yyyy-MM-d"}'/>
                                </h4>
                                <h2 class="mt-4">
                                    <strong class="bg-light" t-esc="o.code"></strong>
                                </h2>
                                <t t-set="rule" t-value="o.program_id.rule_ids[:1]"/>
                                <h4 t-if="rule.minimum_qty > 1">
                                    <span>Minimum purchase of</span>
                                    <strong t-esc="rule.minimum_qty"/> <span>products</span>
                                </h4>
                                <h4 t-if="rule.minimum_amount">
                                    <span>Valid for purchase above</span>
                                    <strong t-esc="rule.minimum_amount" t-options="{'widget': 'monetary', 'display_currency': rule.currency_id}"/>
                                </h4>
                                <br/>
                                <div t-field="o.code" t-options="{'widget': 'barcode', 'width': 600, 'height': 100}"/>
                                <br/><br/>
                                <h4>Thank you,</h4>
                                <br/>
                                <div class="mt32">
                                    <div class="text-center">
                                        <img alt="Logo" t-att-src="'/logo?company=%d' % (o.program_id.company_id)" t-att-alt="'%s' % (o.program_id.company_id.name)" style="border:0 solid transparent;" height="50"/>
                                    </div>
                                </div>
                                <div>
                                    <div class="text-center d-inline-block">
                                        <span t-field="o.program_id.company_id.partner_id"
                                            t-options='{"widget": "contact", "fields": ["address", "email"], "no_marker": True}'/>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </t>
    </template>

    <template id="loyalty_report_i18n">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-set="o" t-value="o.with_context(lang=o._get_mail_partner().lang or o.env.lang)"/>
                <t t-call="loyalty.loyalty_report" t-lang="o._get_mail_partner().lang or o.env.lang"/>
            </t>
        </t>
    </template>

    <template id="gift_card_report">
        <t t-call="web.html_container">
            <t t-call="web.external_layout">
                <div style="margin:0px; font-size:24px; font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:36px; color:#333333; text-align: center">
                    Here is your gift card!
                </div>
                <div style="padding-top:20px; padding-bottom:20px">
                    <img src="/loyalty/static/img/gift_card.png" style="display:block; border:0; outline:none; text-decoration:none; margin:auto;" width="300"/>
                </div>
                <div style="padding:0; margin:0px; padding-top:35px; padding-bottom:35px; text-align:center;">
                    <h3 style="margin:0px; line-height:48px; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:40px; font-style:normal; font-weight:normal; color:#333333; text-align:center">
                        <strong><span t-esc="o.points" t-options="{'widget': 'monetary', 'display_currency': o.currency_id}"/></strong>
                    </h3>
                </div>
                <div style="padding:0; margin:0px; padding-top:35px; padding-bottom:35px; background-color:#efefef; text-align:center;">
                    <p style="margin:0px; font-size:14px;font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:21px; color:#333333">
                        <strong>Gift Card Code</strong>
                    </p>
                    <p style="margin:0px; font-size:25px;font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:38px; color:#A9A9A9">
                        <span t-field="o.code"/>
                    </p>
                </div>
                <div t-if="o.expiration_date" style="padding:0; margin:0px; padding-top:10px; padding-bottom:10px; text-align:center;">
                    <h3 style="margin:0px; line-height:17px; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:14px; font-style:normal; font-weight:normal; color:#A9A9A9; text-align:center">
                        Card expires <span t-field="o.expiration_date"/>
                    </h3>
                </div>
                <div style="padding:0; margin:0px; padding-top:10px; padding-bottom:10px; text-align:center;">
                    <img t-att-src="'/report/barcode/Code128/'+o.code" style="width:400px;height:75px" alt="Barcode"/>
                </div>
            </t>
        </t>
    </template>

    <template id="gift_card_report_i18n">
        <t t-call="web.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-set="o" t-value="o.with_context(lang=o._get_mail_partner().lang or o.env.lang)"/>
                <t t-call="loyalty.gift_card_report" t-lang="o._get_mail_partner().lang or o.env.lang"/>
            </t>
        </t>
    </template>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_loyalty_card,access_loyalty_card,model_loyalty_card,base.group_user,0,0,0,0
access_loyalty_mail,access_loyalty_mail,model_loyalty_mail,base.group_user,0,0,0,0
access_loyalty_program,access_loyalty_program,model_loyalty_program,base.group_user,0,0,0,0
access_loyalty_reward,access_loyalty_reward,model_loyalty_reward,base.group_user,0,0,0,0
access_loyalty_rule,access_loyalty_rule,model_loyalty_rule,base.group_user,0,0,0,0
access_loyalty_generate_wizard,access_loyalty_generate_wizard,model_loyalty_generate_wizard,base.group_user,0,0,0,0

```

## File: security\loyalty_security.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <record id="sale_loyalty_program_company_rule" model="ir.rule">
            <field name="name">Loyalty program multi company rule</field>
            <field name="model_id" ref="model_loyalty_program"/>
            <field name="domain_force">['|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]</field>
        </record>

        <record id="sale_loyalty_card_company_rule" model="ir.rule">
            <field name="name">Loyalty card multi company rule</field>
            <field name="model_id" ref="model_loyalty_card"/>
            <field name="domain_force">['|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]</field>
        </record>

        <record id="sale_loyalty_rule_company_rule" model="ir.rule">
            <field name="name">Loyalty rule multi company rule</field>
            <field name="model_id" ref="model_loyalty_rule"/>
            <field name="domain_force">['|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]</field>
        </record>

        <record id="sale_loyalty_reward_company_rule" model="ir.rule">
            <field name="name">Loyalty reward multi company rule</field>
            <field name="model_id" ref="model_loyalty_reward"/>
            <field name="domain_force">['|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]</field>
        </record>
    </data>
</odoo>

```

## File: static\img\2_plus_1.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"/><g id="b"/><g id="c"/><g id="d"/><g id="e"/><g id="f"/><g id="g"><g><polyline points="32.77 50.17 32.77 30.11 49.81 20.27 50.29 19.74 32.72 9.6 15.15 19.74 15.21 20.13 32.31 30.01 41.24 25.08 24.14 15.2 15.15 19.74 15.15 40.03 32.72 50.17 50.29 40.03 50.29 19.74" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><polyline points="67.34 50.36 67.34 30.29 84.38 20.46 84.85 19.93 67.28 9.78 49.71 19.93 49.77 20.32 66.87 30.19 75.8 25.26 58.7 15.39 49.71 19.93 49.71 40.21 67.28 50.36 84.85 40.21 84.85 19.93" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><polyline points="50.28 90.4 50.28 70.34 67.32 60.5 67.79 59.97 50.23 49.83 32.66 59.97 32.71 60.37 49.82 70.24 58.75 65.31 41.64 55.44 32.66 59.97 32.66 80.26 50.23 90.4 67.79 80.26 67.79 59.97" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/></g></g><g id="h"/></svg>
```

## File: static\img\coupons.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"/><g id="b"/><g id="c"/><g id="d"/><g id="e"/><g id="f"><g><g><line x1="19.65" y1="42.53" x2="22.48" y2="45.36" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><line x1="28.86" y1="51.74" x2="51.19" y2="74.07" style="fill:none; stroke:#7c6576; stroke-dasharray:0 0 9.02 9.02; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><line x1="54.38" y1="77.26" x2="57.21" y2="80.09" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/></g><path d="M68.28,13.26c-4.89,5.04-4.85,13.09,.13,18.08s13.03,5.02,18.08,.13h.01l4.98,4.97c2.82,2.82,2.82,7.4,0,10.23l-44.8,44.8c-2.82,2.82-7.4,2.82-10.23,0l-4.85-4.85c4.89-5.04,4.85-13.09-.13-18.08s-13.03-5.02-18.08-.13l-4.85-4.85c-2.82-2.82-2.82-7.4,0-10.23L53.33,8.53c2.82-2.82,7.4-2.82,10.23,0l4.72,4.72" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/></g></g><g id="g"/><g id="h"/></svg>
```

## File: static\img\ewallet.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"/><g id="b"/><g id="c"/><g id="d"/><g id="e"/><g id="f"/><g id="g"/><g id="h"><g><g><path d="M94.82,38.97l-3.66-3.65h0l-2.05-2.06-2.83,2.83,2.05,2.05,3.66,3.66c.72,.72,1.12,1.62,1.24,2.55,.17,1.32-.23,2.69-1.24,3.7l-43.4,43.4c-1.01,1.01-2.39,1.41-3.7,1.24-.93-.12-1.84-.52-2.55-1.24l-3.66-3.66-21.51-21.51c-.83-.83-1.29-1.95-1.29-3.12s.46-2.29,1.29-3.13L60.57,16.63c.83-.83,1.95-1.29,3.13-1.29s2.29,.46,3.12,1.29l5.71,5.71,2.83-2.83-12.55-12.55c-1.65-1.65-3.85-2.56-6.19-2.56s-4.54,.91-6.19,2.56L7.49,49.9c-3.41,3.41-3.41,8.96,0,12.38l28.35,28.35h.01l3.65,3.66c1.64,1.64,3.8,2.46,5.95,2.46s4.31-.82,5.95-2.46l43.4-43.4c3.28-3.28,3.28-8.62,0-11.91ZM12.1,61.22l-1.78-1.78c-1.85-1.85-1.85-4.87,0-6.72L53.26,9.79c.93-.93,2.14-1.39,3.36-1.39s2.43,.46,3.36,1.39l1.78,1.78c-1.51,.35-2.89,1.11-4.02,2.23L14.34,57.21c-1.12,1.12-1.88,2.51-2.23,4.02Z" style="fill:#7c6576;"/><path d="M63.42,37.56c-.2,.48-.29,1-.29,1.51s.1,1.03,.29,1.51c.2,.48,.49,.93,.88,1.33l2.42,2.42c.3,.3,.64,.52,1,.71,.58,.3,1.2,.47,1.84,.47,.77,0,1.54-.22,2.2-.66,.22-.15,.43-.32,.63-.51l11.06-11.06,2.83-2.83,.77-.77c.39-.39,.68-.84,.88-1.33,.39-.96,.39-2.05,0-3.02-.2-.48-.49-.93-.88-1.33l-2.42-2.42c-.2-.2-.41-.37-.63-.51-.17-.11-.34-.21-.52-.29-.53-.25-1.11-.37-1.69-.37-1.03,0-2.05,.39-2.83,1.17l-.77,.77-2.83,2.83-11.06,11.06c-.39,.39-.68,.84-.88,1.33Zm4.15-.13c1-1,2.63-1,3.63,0,.48,.48,.75,1.13,.75,1.81,0,.69-.27,1.33-.75,1.82-.5,.5-1.16,.75-1.81,.75s-1.31-.25-1.81-.75c-1-1.01-1-2.63,0-3.63Z" style="fill:#7c6576;"/></g><g><path d="M50.19,83.55c-.51,0-1.02-.2-1.41-.59-1.48-1.49-3.91-1.49-5.39,0-.78,.78-2.05,.78-2.83,0s-.78-2.05,0-2.83c3.05-3.05,8-3.05,11.05,0,.78,.78,.78,2.05,0,2.83-.39,.39-.9,.59-1.41,.59Z" style="fill:#7c6576;"/><path d="M56.51,77.56c-.51,0-1.02-.2-1.41-.59-2.36-2.36-5.5-3.67-8.85-3.67s-6.48,1.3-8.85,3.67c-.78,.78-2.05,.78-2.83,0-.78-.78-.78-2.05,0-2.83,3.12-3.12,7.26-4.84,11.67-4.84s8.56,1.72,11.68,4.84c.78,.78,.78,2.05,0,2.83-.39,.39-.9,.59-1.41,.59Z" style="fill:#7c6576;"/><path d="M62.41,71.53c-.51,0-1.02-.2-1.41-.59-3.96-3.96-9.23-6.14-14.83-6.14s-10.87,2.18-14.83,6.14c-.78,.78-2.05,.78-2.83,0s-.78-2.05,0-2.83c4.72-4.72,10.99-7.31,17.66-7.31s12.94,2.6,17.66,7.31c.78,.78,.78,2.05,0,2.83-.39,.39-.9,.59-1.41,.59Z" style="fill:#7c6576;"/></g></g></g></svg>
```

## File: static\img\fidelity_cards.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"/><g id="b"/><g id="c"/><g id="d"><g><rect x="11.09" y="23.03" width="77.83" height="53.94" rx="7.23" ry="7.23" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><g><path d="M31,34.36l1.17,2.37c.12,.24,.35,.41,.61,.45l2.62,.38c.67,.1,.93,.92,.45,1.39l-1.89,1.85c-.19,.19-.28,.46-.23,.72l.45,2.61c.11,.67-.58,1.17-1.18,.86l-2.34-1.23c-.24-.12-.52-.12-.76,0l-2.34,1.23c-.6,.31-1.3-.19-1.18-.86l.45-2.61c.05-.26-.04-.53-.23-.72l-1.89-1.85c-.48-.47-.22-1.29,.45-1.39l2.62-.38c.27-.04,.49-.21,.61-.45l1.17-2.37c.3-.61,1.16-.61,1.46,0Z" style="fill:#7c6576; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><path d="M51.01,34.36l1.17,2.37c.12,.24,.35,.41,.61,.45l2.62,.38c.67,.1,.93,.92,.45,1.39l-1.89,1.85c-.19,.19-.28,.46-.23,.72l.45,2.61c.11,.67-.58,1.17-1.18,.86l-2.34-1.23c-.24-.12-.52-.12-.76,0l-2.34,1.23c-.6,.31-1.3-.19-1.18-.86l.45-2.61c.05-.26-.04-.53-.23-.72l-1.89-1.85c-.48-.47-.22-1.29,.45-1.39l2.62-.38c.27-.04,.49-.21,.61-.45l1.17-2.37c.3-.61,1.16-.61,1.46,0Z" style="fill:#7c6576; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><path d="M31,54.37l1.17,2.37c.12,.24,.35,.41,.61,.45l2.62,.38c.67,.1,.93,.92,.45,1.39l-1.89,1.85c-.19,.19-.28,.46-.23,.72l.45,2.61c.11,.67-.58,1.17-1.18,.86l-2.34-1.23c-.24-.12-.52-.12-.76,0l-2.34,1.23c-.6,.31-1.3-.19-1.18-.86l.45-2.61c.05-.26-.04-.53-.23-.72l-1.89-1.85c-.48-.47-.22-1.29,.45-1.39l2.62-.38c.27-.04,.49-.21,.61-.45l1.17-2.37c.3-.61,1.16-.61,1.46,0Z" style="fill:#7c6576; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><path d="M51.01,54.37l1.17,2.37c.12,.24,.35,.41,.61,.45l2.62,.38c.67,.1,.93,.92,.45,1.39l-1.89,1.85c-.19,.19-.28,.46-.23,.72l.45,2.61c.11,.67-.58,1.17-1.18,.86l-2.34-1.23c-.24-.12-.52-.12-.76,0l-2.34,1.23c-.6,.31-1.3-.19-1.18-.86l.45-2.61c.05-.26-.04-.53-.23-.72l-1.89-1.85c-.48-.47-.22-1.29,.45-1.39l2.62-.38c.27-.04,.49-.21,.61-.45l1.17-2.37c.3-.61,1.16-.61,1.46,0Z" style="fill:#7c6576; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><path d="M71.03,34.36l1.17,2.37c.12,.24,.35,.41,.61,.45l2.62,.38c.67,.1,.93,.92,.45,1.39l-1.89,1.85c-.19,.19-.28,.46-.23,.72l.45,2.61c.11,.67-.58,1.17-1.18,.86l-2.34-1.23c-.24-.12-.52-.12-.76,0l-2.34,1.23c-.6,.31-1.3-.19-1.18-.86l.45-2.61c.05-.26-.04-.53-.23-.72l-1.89-1.85c-.48-.47-.22-1.29,.45-1.39l2.62-.38c.27-.04,.49-.21,.61-.45l1.17-2.37c.3-.61,1.16-.61,1.46,0Z" style="fill:#7c6576; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/></g></g></g><g id="e"/><g id="f"/><g id="g"/><g id="h"/></svg>
```

## File: static\img\gift_card.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"/><g id="b"><path d="M81.68,21.03H18.32c-5.09,0-9.23,4.14-9.23,9.23v39.47c0,5.09,4.14,9.23,9.23,9.23h63.36c5.09,0,9.23-4.14,9.23-9.23V30.26c0-5.09-4.14-9.23-9.23-9.23Zm-35.04,53.94l40.28-29.08v5.41l-32.78,23.67h-7.49Zm-2.96-30.58l-.46-.16-3.37-1.14-.61-.21c.92-1.19,1.35-2.63,1.28-4.06-.02-.35-.06-.7-.13-1.05-.19-.85-.57-1.66-1.13-2.38l4.86-1.36c.07,.25,.19,.49,.36,.7,.01,.02,.03,.04,.04,.06,.76,.97,1.58,2.29,1.78,3.65,.05,.32,.07,.65,.04,.97v.02c-.12,1.08-.75,2.04-1.92,2.94-.62,.48-.89,1.27-.73,2.01Zm.5-14.53l-.53,.15,6.67-4.82c.07-.05,.12-.11,.18-.16h7.53l-9.93,7.17c-.1-.43-.28-.85-.56-1.22-.79-1.03-2.11-1.47-3.36-1.12Zm-25.86-4.83h25.39l-10.32,7.45-7.5-2.54c-1.45-.49-3.04-.06-4.05,1.09-.63,.71-.95,1.62-.94,2.52-1.36,1.32-2.18,2.83-2.44,4.51-.05,.16-.07,.33-.08,.51,0,.06,0,.12,0,.17-.08,1.3,.17,2.66,.76,4.03l-6.06,4.37V30.26c0-2.89,2.35-5.23,5.23-5.23Zm-5.23,27.04l7.67-5.53c.11,.26,.24,.52,.42,.75,.61,.79,1.53,1.24,2.5,1.24,.29,0,.58-.04,.86-.12l.61-.17v.54l-12.06,8.7v-5.41Zm11.52-7.84c-.04-.13-.08-.25-.15-.37-.06-.12-.13-.23-.21-.33-.67-.85-1.41-2-1.72-3.21-.12-.48-.18-.96-.14-1.44,0-.02,0-.04,0-.05,.12-1.08,.75-2.04,1.92-2.94,.62-.48,.89-1.27,.73-2.01l4.37,1.48,.07,.02c-.79,1.02-1.22,2.22-1.28,3.45-.04,.72,.06,1.44,.27,2.13,.2,.66,.52,1.29,.95,1.87,.01,.01,.02,.03,.03,.04l-.32,.09-4,1.12-.54,.15Zm4.54,6.59v-3.71l5.52-1.55,1.2,.41,3.35,1.13v15.19l-3.71-3.44c-.38-.36-.87-.53-1.36-.53s-.98,.18-1.37,.54l-3.64,3.41v-11.44Zm3.06-11.99c.04-.25,.12-.5,.24-.73,.1-.18,.21-.36,.37-.51,.33-.33,.74-.53,1.17-.6,.12-.02,.25-.04,.37-.04,.56,0,1.12,.21,1.55,.64,.35,.35,.55,.79,.61,1.24,.09,.66-.11,1.35-.61,1.85-.85,.85-2.24,.85-3.09,0-.5-.5-.7-1.19-.61-1.85Zm-19.12,30.91v-7.32l12.06-8.7v13.16c0,.8,.47,1.52,1.2,1.83,.26,.11,.53,.17,.8,.17,.5,0,.99-.19,1.37-.54l5.65-5.28,5.7,5.29c.58,.54,1.43,.68,2.16,.37,.73-.32,1.2-1.04,1.2-1.83v-18.45c1.34,.3,2.74-.14,3.66-1.19,.63-.71,.95-1.62,.94-2.53,1.36-1.32,2.18-2.83,2.44-4.51,.05-.16,.07-.33,.08-.51,0-.06,0-.11,0-.17,.07-1.16-.13-2.36-.58-3.58l15.03-10.85s.05-.05,.08-.07h16.83c2.89,0,5.23,2.35,5.23,5.23v10.69l-46.53,33.59c-.17,.12-.3,.26-.42,.42H18.32c-2.88,0-5.23-2.35-5.23-5.23Zm68.6,5.23h-20.72l25.95-18.73v13.5c0,2.89-2.35,5.23-5.23,5.23Z" style="fill:#7c6576;"/></g><g id="c"/><g id="d"/><g id="e"/><g id="f"/><g id="g"/><g id="h"/></svg>
```

## File: static\img\loyalty_cards.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"/><g id="b"/><g id="c"><g><path d="M69.02,26.08L21.21,73.89c-.78,.78-2.05,.78-2.83,0s-.78-2.05,0-2.83L66.19,23.25c.78-.78,2.05-.78,2.83,0s.78,2.05,0,2.83Z" style="fill:#7c6576;"/><path d="M63.69,20.76L15.89,68.56c-.78,.78-2.05,.78-2.83,0s-.78-2.05,0-2.83L60.86,17.93c.78-.78,2.05-.78,2.83,0s.78,2.05,0,2.83Z" style="fill:#7c6576;"/><path d="M47.77,75.37l-3.64,3.64c-3.03,3.03-7.95,3.03-10.98,0s-3.03-7.95,0-10.98l3.64-3.64c3.03-3.03,7.95-3.03,10.98,0s3.03,7.95,0,10.98Zm-11.79-4.51c-1.47,1.47-1.47,3.86,0,5.32s3.86,1.47,5.32,0l3.64-3.64c1.47-1.47,1.47-3.86,0-5.32s-3.86-1.47-5.32,0l-3.64,3.64Z" style="fill:#7c6576;"/><g><path d="M43.56,88.76c-1.76,1.76-4.62,1.76-6.38,0L12.07,63.64c-1.76-1.76-1.76-4.62,0-6.38L52.39,16.94c1.76-1.76,4.62-1.76,6.38,0l15.72,15.72,1.61,.55,2.63-1.96L61.6,14.11c-3.32-3.32-8.72-3.32-12.03,0L9.24,54.43c-3.32,3.32-3.32,8.72,0,12.03l25.12,25.12c3.32,3.32,8.72,3.32,12.03,0l28.37-28.37-2.34-3.31-28.85,28.85Z" style="fill:#7c6576;"/><path d="M60.44,56.01c-.22-.42-.37-.86-.48-1.31l-3.85,3.85c-.78,.78-.78,2.05,0,2.83s2.05,.78,2.83,0l3.26-3.26c-.72-.55-1.33-1.26-1.76-2.1Z" style="fill:#7c6576;"/><path d="M63.22,40.78l-12.44,12.44c-.78,.78-.78,2.05,0,2.83s2.05,.78,2.83,0l9.92-9.92,1.03-1.38-1.34-3.96Z" style="fill:#7c6576;"/><path d="M85.55,32.64c-.17-.09-.35-.15-.54-.19-.56-.12-1.18-.02-1.72,.38l-1.7,1.27-3.24,2.42-.64,.48c-.28,.21-.61,.35-.95,.4-.34,.05-.69,.03-1.02-.09l-6.59-2.24c-.42-.14-.84-.15-1.21-.05-.38,.1-.72,.3-.99,.57-.14,.14-.25,.29-.35,.45-.19,.33-.3,.72-.28,1.13,0,.14,.05,.29,.08,.44,.02,.06,.01,.12,.03,.18l1.38,4.06,.86,2.53c.06,.17,.09,.34,.11,.51,.04,.52-.1,1.04-.42,1.46l-.04,.05-4.13,5.53c-.88,1.18-.31,2.76,.93,3.26,.25,.1,.52,.17,.82,.17l6.96-.09c.53,0,1.04,.18,1.43,.52,.13,.11,.25,.24,.35,.39l.61,.86,2.34,3.31,1.07,1.51c.12,.17,.25,.3,.4,.42,.04,.04,.09,.07,.14,.1,.11,.07,.22,.14,.33,.19,.06,.02,.11,.05,.17,.07,.12,.04,.24,.07,.37,.09,.05,0,.1,.02,.15,.02,.17,.02,.35,.01,.52-.01,.02,0,.05-.01,.07-.02,.15-.03,.29-.07,.43-.13,.05-.02,.1-.05,.15-.07,.11-.06,.21-.12,.31-.19,.05-.04,.1-.07,.14-.11,.03-.03,.06-.05,.09-.07,.06-.06,.11-.14,.16-.21,.04-.05,.08-.09,.11-.14,.1-.16,.19-.33,.25-.53l2.07-6.65c.1-.34,.29-.64,.53-.88,.18-.18,.4-.32,.64-.42,.08-.04,.15-.08,.24-.11l6.65-2.07c.2-.06,.37-.15,.53-.25,.05-.03,.1-.07,.14-.11,.07-.05,.15-.1,.21-.16,.03-.03,.05-.06,.07-.09,.04-.05,.08-.09,.11-.14,.07-.1,.14-.2,.19-.31,.03-.05,.05-.1,.07-.15,.06-.14,.1-.28,.13-.43,0-.02,.01-.05,.02-.07,.03-.17,.03-.35,.01-.52,0-.05-.02-.1-.03-.15-.02-.12-.05-.25-.09-.37-.02-.06-.04-.11-.07-.17-.05-.11-.12-.22-.19-.33-.03-.05-.06-.1-.1-.14-.12-.14-.26-.28-.42-.4l-4.05-2.87-1.63-1.15c-.14-.1-.27-.22-.39-.35-.34-.39-.53-.9-.52-1.43l.03-2.37,.06-4.6c0-.44-.12-.84-.33-1.17s-.5-.59-.84-.77Z" style="fill:#7c6576;"/></g></g></g><g id="d"/><g id="e"/><g id="f"/><g id="g"/><g id="h"/></svg>
```

## File: static\img\loyalty_cards_2.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"/><g id="b"/><g id="c"><g><rect x="9.79" y="23.51" width="77.83" height="53.94" rx="7.23" ry="7.23" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><line x1="86.26" y1="43.76" x2="11.14" y2="43.76" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><line x1="11.14" y1="35.4" x2="86.26" y2="35.4" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><g><line x1="69.26" y1="63.57" x2="50.61" y2="63.57" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><line x1="50.61" y1="55.2" x2="77.03" y2="55.2" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/></g><rect x="18.71" y="52.98" width="18.53" height="12.81" rx="6.41" ry="6.41" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><g><path d="M86.23,86.49c-.71,0-1.4-.17-2.04-.51l-6.85-3.6c-.06-.03-.12-.04-.18-.04s-.12,.01-.18,.04l-6.85,3.6c-.64,.34-1.33,.51-2.04,.51-1.29,0-2.52-.57-3.36-1.57-.83-.98-1.18-2.28-.96-3.56l1.31-7.62c.02-.12-.02-.25-.11-.34l-5.54-5.4c-1.2-1.17-1.63-2.89-1.11-4.49,.52-1.6,1.88-2.74,3.54-2.98l7.66-1.11c.12-.02,.23-.1,.29-.21l3.42-6.94c.74-1.51,2.25-2.44,3.93-2.44h0c1.68,0,3.19,.94,3.93,2.44l3.42,6.94c.06,.11,.16,.19,.29,.21l7.65,1.11c1.66,.24,3.02,1.38,3.54,2.98,.52,1.6,.09,3.32-1.11,4.49l-5.54,5.4c-.09,.09-.13,.21-.11,.34l1.31,7.62c.22,1.28-.13,2.57-.96,3.56-.84,1-2.07,1.57-3.36,1.57Z" style="fill:#7c6576;"/><path d="M77.17,51.83c.85,0,1.7,.44,2.14,1.33l3.42,6.94c.35,.7,1.02,1.19,1.79,1.3l7.66,1.11c1.95,.28,2.73,2.68,1.32,4.06l-5.54,5.4c-.56,.55-.82,1.34-.69,2.11l1.31,7.62c.26,1.54-.96,2.79-2.35,2.79-.37,0-.74-.09-1.11-.28l-6.85-3.6c-.35-.18-.73-.27-1.11-.27s-.76,.09-1.11,.27l-6.85,3.6c-.36,.19-.74,.28-1.11,.28-1.39,0-2.61-1.25-2.35-2.79l1.31-7.62c.13-.77-.12-1.56-.69-2.11l-5.54-5.4c-1.41-1.38-.63-3.78,1.32-4.06l7.66-1.11c.78-.11,1.45-.6,1.79-1.3l3.42-6.94c.44-.89,1.29-1.33,2.14-1.33m0-4c-2.45,0-4.64,1.36-5.72,3.56l-3.05,6.17-6.81,.99c-2.42,.35-4.4,2.02-5.15,4.34-.76,2.33-.14,4.83,1.61,6.54l4.93,4.81-1.16,6.79c-.32,1.86,.19,3.75,1.4,5.18,1.22,1.45,3,2.28,4.89,2.28,1.02,0,2.05-.26,2.97-.74l6.09-3.2,6.09,3.2c.92,.48,1.95,.74,2.97,.74,1.88,0,3.67-.83,4.89-2.28,1.21-1.43,1.72-3.32,1.4-5.18l-1.16-6.79,4.93-4.81c1.75-1.71,2.37-4.21,1.61-6.54-.76-2.33-2.73-3.99-5.15-4.34l-6.81-.99-3.05-6.17c-1.08-2.19-3.28-3.56-5.72-3.56h0Z" style="fill:#fff;"/></g></g></g><g id="d"/><g id="e"/><g id="f"/><g id="g"/><g id="h"/></svg>
```

## File: static\img\promotional_program.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"/><g id="b"/><g id="c"/><g id="d"/><g id="e"><g><g><path d="M58.32,64.91c-.18,0-.36-.02-.53-.07l-27.73-7.62c-.87-.24-1.47-1.03-1.47-1.93v-19c0-.9,.6-1.69,1.47-1.93l27.73-7.62c.6-.16,1.25-.04,1.74,.34,.5,.38,.79,.97,.79,1.59V62.91c0,.62-.29,1.21-.79,1.59-.35,.27-.78,.41-1.21,.41Zm-25.73-11.15l23.73,6.52V31.29l-23.73,6.52v15.95Z" style="fill:#7c6576;"/><g><path d="M24.32,72.61c.25,.94,.85,1.72,1.7,2.21,.84,.49,1.82,.61,2.76,.36h0c.94-.25,1.72-.85,2.21-1.69,.48-.84,.61-1.82,.36-2.76l-3.64-13.53h-7.53l4.14,15.42Z" style="fill:none;"/><path d="M35.21,69.69l-3.46-12.87c.51-.36,.84-.95,.84-1.63v-18.82c0-1.1-.9-2-2-2h-11.19c-4.46,0-8.09,3.63-8.09,8.09v6.63c0,3.17,1.84,5.92,4.51,7.24,0,.02,0,.05,.02,.07l4.63,17.23c.53,1.97,1.79,3.62,3.56,4.63,1.17,.68,2.48,1.02,3.8,1.02,.67,0,1.33-.09,1.99-.26,1.97-.53,3.61-1.79,4.63-3.56s1.29-3.82,.76-5.79ZM15.3,42.47c0-2.26,1.84-4.09,4.09-4.09h9.19v14.82h-9.19c-1.33,0-2.51-.65-3.26-1.64-.52-.69-.83-1.53-.83-2.45v-6.63Zm15.68,31.01c-.49,.84-1.27,1.44-2.21,1.69h0c-.94,.25-1.92,.12-2.76-.36-.84-.49-1.44-1.27-1.7-2.21l-4.14-15.42h7.53l3.64,13.53c.25,.94,.12,1.92-.36,2.76Z" style="fill:#7c6576;"/></g><g><path d="M69.27,45.79c0-1.52-.97-2.81-2.33-3.31v6.62c1.35-.5,2.33-1.79,2.33-3.31Z" style="fill:none;"/><path d="M66.94,38.36v-12.33c0-2.94-2.39-5.33-5.33-5.33s-5.33,2.39-5.33,5.33v39.53c0,2.94,2.39,5.33,5.33,5.33s5.33-2.39,5.33-5.33v-12.33c3.58-.58,6.33-3.69,6.33-7.43s-2.75-6.85-6.33-7.43Zm-4,.44v26.76c0,.73-.6,1.33-1.33,1.33s-1.33-.6-1.33-1.33V26.02c0-.73,.6-1.33,1.33-1.33s1.33,.6,1.33,1.33v12.77Zm4,10.31v-6.62c1.35,.5,2.33,1.79,2.33,3.31s-.97,2.81-2.33,3.31Z" style="fill:#7c6576;"/></g></g><g><path d="M86.69,47.79h-6.51c-1.1,0-2-.9-2-2s.9-2,2-2h6.51c1.1,0,2,.9,2,2s-.9,2-2,2Z" style="fill:#7c6576;"/><path d="M84.2,64.37c-.51,0-1.02-.2-1.41-.59l-4.42-4.42c-.78-.78-.78-2.05,0-2.83s2.05-.78,2.83,0l4.42,4.42c.78,.78,.78,2.05,0,2.83-.39,.39-.9,.59-1.41,.59Z" style="fill:#7c6576;"/><path d="M79.78,35.63c-.51,0-1.02-.2-1.41-.59-.78-.78-.78-2.05,0-2.83l4.43-4.43c.78-.78,2.05-.78,2.83,0s.78,2.05,0,2.83l-4.43,4.43c-.39,.39-.9,.59-1.41,.59Z" style="fill:#7c6576;"/></g></g></g><g id="f"/><g id="g"/><g id="h"/></svg>
```

## File: static\img\promo_code.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"><g><path d="M53.33,8.53L8.53,53.33c-2.82,2.82-2.82,7.4,0,10.23l4.85,4.85c5.04-4.89,13.09-4.85,18.08,.13s5.02,13.03,.13,18.08l4.85,4.85c2.82,2.82,7.4,2.82,10.23,0l44.8-44.8c2.82-2.82,2.82-7.4,0-10.23L63.56,8.53c-2.82-2.82-7.4-2.82-10.23,0Z" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><path d="M31.59,86.62c4.89-5.04,4.85-13.09-.13-18.08s-13.03-5.02-18.08-.13" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><g><line x1="19.65" y1="42.53" x2="22.48" y2="45.36" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><line x1="28.86" y1="51.74" x2="51.19" y2="74.07" style="fill:none; stroke:#7c6576; stroke-dasharray:0 0 9.02 9.02; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><line x1="54.38" y1="77.26" x2="57.21" y2="80.09" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/></g><line x1="57.32" y1="59.75" x2="57.32" y2="25.61" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><circle cx="69.58" cy="42.13" r="4.93" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><circle cx="44.81" cy="42.92" r="4.93" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/></g></g><g id="b"/><g id="c"/><g id="d"/><g id="e"/><g id="f"/><g id="g"/><g id="h"/></svg>
```

## File: static\src\js\loyalty_card_list_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { listView } from "@web/views/list/list_view";

export const LoyaltyCardListView = {
    ...listView,
    buttonTemplate: "loyalty.LoyaltyCardListView.buttons",
};

registry.category("views").add("loyalty_card_list_view", LoyaltyCardListView);

```

## File: static\src\js\loyalty_control_panel_widget.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { X2ManyField } from "@web/views/fields/x2many/x2many_field";

export class LoyaltyX2ManyField extends X2ManyField {};
LoyaltyX2ManyField.template = "loyalty.LoyaltyX2ManyField";

registry.category("fields").add("loyalty_one2many", LoyaltyX2ManyField);

```

## File: static\src\js\loyalty_list_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { listView } from "@web/views/list/list_view";
import { ListRenderer } from "@web/views/list/list_renderer";
import { useService } from "@web/core/utils/hooks";

const { Component, onWillStart } = owl;

export class LoyaltyActionHelper extends Component {
    setup() {
        this.orm = useService("orm");
        this.action = useService("action");

        onWillStart(async () => {
            this.loyaltyTemplateData = await this.orm.call(
                "loyalty.program",
                "get_program_templates",
                [],
                {
                    context: this.env.model.root.context,
                },
            );
        });
    }

    async onTemplateClick(templateId) {
        const action = await this.orm.call(
            "loyalty.program",
            "create_from_template",
            [templateId],
            {context: this.env.model.root.context},
        );
        if (!action) {
            return;
        }
        this.action.doAction(action);
    }
};
LoyaltyActionHelper.template = "loyalty.LoyaltyActionHelper";

export class LoyaltyListRenderer extends ListRenderer {};
LoyaltyListRenderer.template = "loyalty.LoyaltyListRenderer";
LoyaltyListRenderer.components = {
    ...LoyaltyListRenderer.components,
    LoyaltyActionHelper,
};

export const LoyaltyListView = {
    ...listView,
    Renderer: LoyaltyListRenderer,
};

registry.category("views").add("loyalty_program_list_view", LoyaltyListView);

```

## File: static\src\js\filterable_selection_field\filterable_selection_field.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { SelectionField } from "@web/views/fields/selection/selection_field";

/**
 * The purpose of this field is to be able to define some values which should not be
 * displayed on our selection field, this way we can have multiple views for the same model
 * that uses different possible sets of values on the same selection field.
 */
export class FilterableSelectionField extends SelectionField {
    /**
     * @override
     */
    get options() {
        let options = super.options;
        if (this.props.whitelisted_values) {
            options = options.filter((option) => {
                return option[0] === this.props.value || this.props.whitelisted_values.includes(option[0])
            });
        } else if (this.props.blacklisted_values) {
            options = options.filter((option) => {
                return option[0] === this.props.value || !this.props.blacklisted_values.includes(option[0]);
            });
        }
        return options;
    }
};

FilterableSelectionField.props = {
    ...SelectionField.props,
    whitelisted_values: { type: Array, optional: true },
    blacklisted_values: { type: Array, optional: true },
};

FilterableSelectionField.extractProps = ({ attrs }) => {
    return {
        ...SelectionField.extractProps({ attrs }),
        whitelisted_values: attrs.options.whitelisted_values,
        blacklisted_values: attrs.options.blacklisted_values,
    };
};

registry.category("fields").add("filterable_selection", FilterableSelectionField);

```

## File: static\src\xml\loyalty_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="loyalty.LoyaltyListRenderer" t-inherit="web.ListRenderer" t-inherit-mode="primary" owl="1">
        <t t-call="web.ActionHelper" position="replace">
            <t t-if="showNoContentHelper">
                <LoyaltyActionHelper noContentHelp="props.noContentHelp"/>
            </t>
        </t>
    </t>

    <t t-name="loyalty.LoyaltyActionHelper" owl="1">
        <div class="o_view_nocontent flex-wrap pt-5">
            <div class="container">
                <div class="o_nocontent_help">
                    <t t-out="props.noContentHelp"/>
                </div>
                <div class="row justify-content-center loyalty-templates-container">
                    <t t-foreach="Object.entries(loyaltyTemplateData)" t-as="data" t-key="data[0]">
                        <t t-set="loyalty_el_icon" t-value="data[1].icon"/>
                        <t t-set="loyalty_el_title" t-value="data[1].title"/>
                        <div class="col-6 col-md-4 col-lg-3 py-4">
                            <div class="card rounded p-3 d-flex align-items-stretch h-100 loyalty-template" t-on-click.stop.prevent="() => this.onTemplateClick(data[0])">
                                <div class="row m-0 w-100 h-100">
                                    <div class="col-lg-4 p-0">
                                        <div class="d-flex w-100 h-100 align-items-start justify-content-center display-3 p-3 text-muted">
                                            <img t-attf-src="/loyalty/static/img/{{loyalty_el_icon}}.svg" t-attf-alt="{{loyalty_el_title}}"/>
                                        </div>
                                    </div>
                                    <div class="col-lg-8 p-0">
                                        <div class="card-body d-flex flex-column align-items-start justify-content-start h-100">
                                            <h3 class="card-title" t-out="loyalty_el_title"/>
                                            <p class="card-text" t-out="data[1].description"/>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </t>
                </div>
            </div>
        </div>
    </t>

    <t t-name="loyalty.LoyaltyX2ManyField" owl="1" t-inherit-mode="primary" t-inherit="web.X2ManyField">
        <t t-if="displayAddButton" position="replace">
            <h4 t-esc="field.string or ''"/>
            <t t-if="displayAddButton">
                <div class="o_cp_buttons me-0 ms-auto" role="toolbar" aria-label="Control panel buttons" t-ref="buttons">
                    <div>
                        <button type="button" class="btn btn-secondary o-kanban-button-new" title="Create record" accesskey="c" t-on-click="() => this.onAdd()">
                            Add
                        </button>
                    </div>
                </div>
            </t>
        </t>
        <div role="search" position="attributes">
            <attribute name="t-if">props.value.count > props.value.limit</attribute>
        </div>
    </t>

    <t t-name="loyalty.LoyaltyCardListView.buttons" owl="1" t-inherit-mode="primary" t-inherit="web.ListView.Buttons">
        <xpath expr="//button[hasclass('o_list_button_add')]" position="replace"/>
        <xpath expr="//t[contains(@t-if, 'isExportEnable')]" position="before">
            <t t-set="supportedProgramTypes" t-value="['coupons', 'gift_card', 'ewallet']"/>
            <button t-if="supportedProgramTypes.includes(props.context.program_type)" type="button" class="btn btn-primary o_loyalty_card_list_button_generate" t-attf-data-tooltip="Generate {{props.context.program_item_name}}"
                t-on-click.stop.prevent="() => this.actionService.doAction('loyalty.loyalty_generate_wizard_action', { additionalContext: this.props.context, onClose: () => {this.model.load()} })">
                Generate <t t-esc="props.context.program_item_name"/>
            </button>
        </xpath>
    </t>
</odoo>

```

## File: views\loyalty_card_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="loyalty_card_view_form" model="ir.ui.view">
        <field name="name">loyalty.card.view.form</field>
        <field name="model">loyalty.card</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <group>
                            <field name="code" readonly="1"/>
                            <field name="expiration_date"/>
                            <field name="partner_id"/>
                            <label string="Balance" for="points"/>
                            <span class="d-inline-block">
                                <field name="points" class="w-auto oe_inline me-1"/>
                                <field name="point_name" no_label="1" class="d-inline"/>
                            </span>
                        </group>
                    </group>
                </sheet>
                <div class="oe_chatter">
                    <field name="message_ids"/>
                </div>
            </form>
        </field>
    </record>

    <record id="loyalty_card_view_tree" model="ir.ui.view">
        <field name="name">loyalty.card.view.tree</field>
        <field name="model">loyalty.card</field>
        <field name="arch" type="xml">
            <tree string="Coupons" edit="false" delete="false" js_class="loyalty_card_list_view">
                <field name="code" readonly="1"/>
                <field name="create_date" optional="hide"/>
                <field name="points_display" string="Balance"/>
                <field name="expiration_date"/>
                <field name="program_id"/>
                <field name="partner_id"/>
                <button name="action_coupon_send" string="Send" type="object" icon="fa-paper-plane-o"/>
            </tree>
        </field>
    </record>

    <record id="loyalty_card_view_search" model="ir.ui.view">
        <field name="name">loyalty.card.view.search</field>
        <field name="model">loyalty.card</field>
        <field name="arch" type="xml">
            <search>
                <field name="code"/>
                <field name="partner_id"/>
                <field name="program_id"/>
                <separator/>
                <filter name="active" string="Active" domain="['&amp;', ('points', '>', 0), '|', ('expiration_date', '>=', context_today().strftime('%Y-%m-%d 00:00:00')), ('expiration_date', '=', False)]"/>
                <filter name="inactive" string="Inactive" domain="['|', ('points', '&lt;=', 0), ('expiration_date', '&lt;', context_today().strftime('%Y-%m-%d 23:59:59'))]"/>
            </search>
        </field>
    </record>

    <record id="loyalty_card_action" model="ir.actions.act_window">
        <field name="name">Coupons</field>
        <field name="res_model">loyalty.card</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('program_id', '=', active_id)]</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <h1>No Coupons Found.</h1>
            <p>There haven't been any coupons generated yet.</p>
        </field>
    </record>
</odoo>

```

## File: views\loyalty_mail_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="loyalty_mail_view_tree" model="ir.ui.view">
        <field name="name">loyalty.mail.view.tree</field>
        <field name="model">loyalty.mail</field>
        <field name="arch" type="xml">
            <tree editable="bottom">
                <field name="trigger"/>
                <field name="points" string="Limit"
                    attrs="{'required': [('trigger', '=', 'points_reach')], 'invisible': [('trigger', '!=', 'points_reach')]}"/>
                <field name="mail_template_id"/>
            </tree>
        </field>
    </record>
</odoo>

```

## File: views\loyalty_program_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- DISCOUNT & LOYALTY -->
    <record id="loyalty_program_view_form" model="ir.ui.view">
        <field name="name">loyalty.program.view.form</field>
        <field name="model">loyalty.program</field>
        <field name="arch" type="xml">
            <form string="Discount &amp; Loyalty">
                <header>
                    <button name="%(loyalty_generate_wizard_action)d" string="Generate Coupons" class="btn-primary" type="action"
                        attrs="{'invisible': [('program_type', '!=', 'coupons')]}"/>
                    <button name="%(loyalty_generate_wizard_action)d" string="Generate Gift Cards" class="btn-primary" type="action"
                        attrs="{'invisible': [('program_type', '!=', 'gift_card')]}"/>
                    <button name="%(loyalty_generate_wizard_action)d" string="Generate eWallet" class="btn-primary" type="action"
                        attrs="{'invisible': [('program_type', '!=', 'ewallet')]}" context="{'default_mode': 'selected'}"/>
                </header>
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button" type="object" name="action_open_loyalty_cards" icon="fa-tags">
                            <div class="o_form_field o_stat_info">
                                <span class="o_stat_value">
                                    <field name="coupon_count"/>
                                </span>
                                <span class="o_stat_text" attrs="{'invisible': [('program_type', 'not in', ('coupons', 'next_order_coupons'))]}">Coupons</span>
                                <span class="o_stat_text" attrs="{'invisible': [('program_type', '!=', 'loyalty')]}">Loyalty Cards</span>
                                <span class="o_stat_text" attrs="{'invisible': [('program_type', 'not in', ('promotion', 'buy_x_get_y'))]}">Promos</span>
                                <span class="o_stat_text" attrs="{'invisible': [('program_type', '!=', 'promo_code')]}">Discount</span>
                                <span class="o_stat_text" attrs="{'invisible': [('program_type', '!=', 'gift_card')]}">Gift Cards</span>
                                <span class="o_stat_text" attrs="{'invisible': [('program_type', '!=', 'ewallet')]}">eWallets</span>
                            </div>
                        </button>
                    </div>
                    <field name="active" invisible="1"/>
                    <field name="applies_on" invisible="1"/>
                    <div class="oe_title">
                        <label for="name" string="Program Name"/>
                        <h1>
                            <field name="name" placeholder="e.g. 10% discount on laptops"/>
                        </h1>
                    </div>
                    <group>
                        <group>
                            <label for="program_type"/>
                            <div>
                                <field name="program_type" widget="filterable_selection" attrs="{'readonly': [('coupon_count', '!=', 0)]}" options="{'blacklisted_values': ['gift_card', 'ewallet']}"/>
                                <p class="text-muted" attrs="{'invisible': [('program_type', '!=', 'coupons')]}" colspan="2">
                                    Generate &amp; share coupon codes manually. It can be used in eCommerce, Point of Sale or regular orders to claim the Reward. You can define constraints on its usage through conditional rule.
                                    <div groups="base.group_no_one">
                                        When generating coupon, you can define a specific points value that can be exchanged for rewards. 
                                    </div>
                                </p>
                                <p class="text-muted" attrs="{'invisible': [('program_type', '!=', 'loyalty')]}" colspan="2">
                                    When customers make an order, they accumulate points they can exchange for rewards on the current order or on a future one.
                                </p>
                                <p class="text-muted" attrs="{'invisible': [('program_type', '!=', 'promotion')]}" colspan="2">
                                    Set up conditional rules on the order that will give access to rewards for customers                                            
                                    <div groups="base.group_no_one">
                                        Each rule can grant points to the customer he will be able to exchange against rewards
                                    </div>
                                </p>
                                <p class="text-muted" attrs="{'invisible': [('program_type', '!=', 'promo_code')]}" colspan="2">
                                    Define Discount codes on conditional rules then share it with your customers for rewards.
                                </p>
                                <p class="text-muted" attrs="{'invisible': [('program_type', '!=', 'buy_x_get_y')]}" colspan="2">
                                    Grant 1 credit for each item bought then reward the customer with Y items in exchange of X credits.
                                </p>
                                <p class="text-muted" attrs="{'invisible': [('program_type', '!=', 'next_order_coupons')]}" colspan="2">
                                    Drive repeat purchases by sending a unique, single-use coupon code for the next purchase when a customer buys something in your store.
                                </p>
                                <p class="text-muted" attrs="{'invisible': [('program_type', '!=', 'gift_card')]}" colspan="2">
                                    Gift Cards are created manually or automatically sent by email when the customer orders a gift card product.
                                    <br/>
                                    Then, Gift Cards can be used to pay orders.
                                </p>
                                <p class="text-muted" attrs="{'invisible': [('program_type', '!=', 'ewallet')]}" colspan="2">
                                    eWallets are created manually or automatically when the customer orders a eWallet product.
                                    <br/>
                                    Then, eWallets are proposed during the checkout, to pay orders.
                                </p>
                            </div>
                            <field name="trigger_product_ids" string="Gift Card Products" widget="many2many_tags" attrs="{'invisible': [('program_type', '!=', 'gift_card')]}"/>
                            <field name="trigger_product_ids" string="eWallet Products" widget="many2many_tags" attrs="{'invisible': [('program_type', '!=', 'ewallet')]}"/>
                            <field name="payment_program_discount_product_id" groups="base.group_no_one" attrs="{'invisible': [('program_type', 'not in', ('gift_card', 'ewallet'))]}"/>
                            <field name="mail_template_id" attrs="{'invisible': [('program_type', 'not in', ('gift_card', 'ewallet'))]}"/>
                            <field name="currency_id"/>
                            <field name="currency_symbol" invisible="1"/>
                            <field name="portal_point_name" attrs="{'invisible': [('program_type', 'in', ('loyalty', 'gift_card', 'ewallet'))]}" string="Points Unit" groups="base.group_no_one"/>
                            <field name="portal_point_name" attrs="{'invisible': ['|', ('program_type', 'in', ('gift_card', 'ewallet')), ('program_type', '!=', 'loyalty')]}" string="Points Unit"/>
                            <field name="portal_visible" invisible="1"/>
                            <field name="portal_visible" groups="base.group_no_one" string="Show points Unit" attrs="{'invisible': [('program_type', 'in', ('gift_card', 'ewallet'))]}"/>
                            <field name="trigger" invisible="1"/>
                            <field name="trigger" string="Program trigger" groups="base.group_no_one" widget="selection" readonly="1" force_save="1"/>
                            <field name="applies_on" invisible="1"/>
                            <field name="applies_on" string="Use points on" groups="base.group_no_one" widget="radio" readonly="1" force_save="1"/>
                        </group>
                        <group>
                            <field name="date_to" attrs="{'invisible': [('program_type', 'in', ('gift_card', 'ewallet'))]}"/>
                            <label for="limit_usage" attrs="{'invisible': [('program_type', 'in', ('gift_card', 'ewallet'))]}"/>
                            <span attrs="{'invisible': [('program_type', 'in', ('gift_card', 'ewallet'))]}">
                                <field name="limit_usage" class="oe_inline"/>
                                <span attrs="{'invisible': [('limit_usage', '=', False)]}"> to <field name="max_usage" class="oe_inline"/> usages</span>
                            </span>
                            <field name="company_id" invisible="1"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="available_on" invisible="1"/>
                            <label class="o_form_label" for="available_on" string="Available On" invisible="1"/>
                            <div id="o_loyalty_program_availabilities" invisible="1"/>
                            <field name="portal_point_name" attrs="{'invisible': [('program_type', 'not in', ('gift_card', 'ewallet'))]}" string="Displayed as" groups="base.group_no_one"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Rules &amp; Rewards" name="rules_rewards" attrs="{'invisible': [('program_type', 'in', ('gift_card', 'ewallet'))]}">
                            <group>
                                <group>
                                    <field name="rule_ids" colspan="2" mode="kanban" nolabel="1" add-label="Add a rule"
                                        class="o_loyalty_kanban_inline" widget="loyalty_one2many" context="{'currency_symbol': currency_symbol, 'program_type': program_type}"/>
                                </group>
                                <group>
                                    <field name="reward_ids" colspan="2" mode="kanban" nolabel="1" add-label="Add a reward"
                                      class="o_loyalty_kanban_inline" widget="loyalty_one2many" context="{'currency_symbol': currency_symbol, 'program_type': program_type}"/>
                                </group>
                            </group>
                        </page>
                        <page string="Rewards" name="rewards" groups="base.group_no_one" attrs="{'invisible': [('program_type', 'not in', ('gift_card', 'ewallet'))]}">
                            <group>
                                <group groups="base.group_no_one">
                                    <field name="reward_ids" colspan="2" mode="kanban" nolabel="1" add-label="Add a reward"
                                      class="o_loyalty_kanban_inline" widget="loyalty_one2many" context="{'currency_symbol': currency_symbol, 'program_type': program_type}"/>
                                </group>
                            </group>
                        </page>
                        <page string="Communications" name="communications" attrs="{'invisible': ['|', ('applies_on', '=', 'current'), ('program_type', 'in', ('ewallet','gift_card'))]}">
                            <field name="communication_plan_ids" mode="tree"/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="loyalty_program_view_tree" model="ir.ui.view">
        <field name="name">loyalty.program.view.tree</field>
        <field name="model">loyalty.program</field>
        <field name="arch" type="xml">
            <tree js_class="loyalty_program_list_view" class="loyalty-program-list-view">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="program_type"/>
                <field name="coupon_count_display" string="Items"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </tree>
        </field>
    </record>

    <record id="loyalty_program_view_search" model="ir.ui.view">
        <field name="name">loyalty.program.view.search</field>
        <field name="model">loyalty.program</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <separator/>
                <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
            </search>
        </field>
    </record>

    <record id="loyalty_program_discount_loyalty_action" model="ir.actions.act_window">
        <field name="name">Discount &amp; Loyalty</field>
        <field name="res_model">loyalty.program</field>
        <field name="view_mode">tree,form</field>
        <field name="domain">[('program_type', 'not in', ('gift_card', 'ewallet'))]</field>
        <field name="help" type="html">
            <div class="o_loyalty_not_found container">
                <h1>No program found.</h1>
                <p class="lead">Create a new one from scratch, or use one of the templates below.</p>
            </div>
        </field>
    </record>

    <record id="action_loyalty_program_tree_discount_loyalty" model="ir.actions.act_window.view">
        <field name="view_mode">tree</field>
        <field name="sequence">1</field>
        <field name="view_id" ref="loyalty_program_view_tree"/>
        <field name="act_window_id" ref="loyalty_program_discount_loyalty_action"/>
    </record>

    <record id="action_loyalty_program_form_discount_loyalty" model="ir.actions.act_window.view">
        <field name="view_mode">form</field>
        <field name="sequence">2</field>
        <field name="view_id" ref="loyalty_program_view_form"/>
        <field name="act_window_id" ref="loyalty_program_discount_loyalty_action"/>
    </record>

    <!-- GIFT & EWALLET -->
    <record id="loyalty_program_gift_ewallet_view_form" model="ir.ui.view">
        <field name="name">loyalty.program.gift.ewallet.view.form</field>
        <field name="model">loyalty.program</field>
        <field name="inherit_id" ref="loyalty_program_view_form"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <form position="attributes">
                <attribute name="string">Gift &amp; Ewallet</attribute>
            </form>
            <field name="program_type" position="attributes">
                <attribute name="options">{'whitelisted_values': ['gift_card', 'ewallet']}</attribute>
            </field>
        </field>
    </record>

    <record id="loyalty_program_gift_ewallet_action" model="ir.actions.act_window">
        <field name="name">Gift cards &amp; eWallet</field>
        <field name="res_model">loyalty.program</field>
        <field name="view_mode">tree,form</field>
        <field name="context">{'menu_type': 'gift_ewallet', 'default_program_type': 'gift_card'}</field>
        <field name="domain">[('program_type', 'in', ('gift_card', 'ewallet'))]</field>
        <field name="help" type="html">
            <div class="o_loyalty_not_found container">
                <h1>No loyalty program found.</h1>
                <p class="lead">Create a new one from scratch, or use one of the templates below.</p>
            </div>
        </field>
    </record>

    <record id="action_loyalty_program_tree_gift_card_ewallet" model="ir.actions.act_window.view">
        <field name="view_mode">tree</field>
        <field name="sequence">1</field>
        <field name="view_id" ref="loyalty_program_view_tree"/>
        <field name="act_window_id" ref="loyalty_program_gift_ewallet_action"/>
    </record>

    <record id="action_loyalty_program_form_gift_card_ewallet" model="ir.actions.act_window.view">
        <field name="view_mode">form</field>
        <field name="sequence">2</field>
        <field name="view_id" ref="loyalty_program_gift_ewallet_view_form"/>
        <field name="act_window_id" ref="loyalty_program_gift_ewallet_action"/>
    </record>
</odoo>

```

## File: views\loyalty_reward_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="loyalty_reward_view_form" model="ir.ui.view">
        <field name="name">loyalty.reward.view.form</field>
        <field name="model">loyalty.reward</field>
        <field name="arch" type="xml">
            <form>
                <field name="program_type" invisible="1"/>
                <field name="user_has_debug" invisible="1"/>
                <field name="multi_product" invisible="1"/>
                <field name="reward_product_uom_id" invisible="1"/>
                <field name="reward_product_ids" invisible="1"/>
                <field name="all_discount_product_ids" invisible="1"/>
                <sheet>
                    <group>
                        <group string="Reward" id="reward_type_group" attrs="{'invisible': [('program_type', 'in', ('gift_card', 'ewallet'))]}">
                            <field name="reward_type" widget="selection" force_save="1" attrs="{'readonly' : [('program_type', '=', 'buy_x_get_y')]}"/>
                            <label for="discount" attrs="{'invisible': [('reward_type', '!=', 'discount')]}"/>
                            <div class="d-flex flex-row" attrs="{'invisible': [('reward_type', '!=', 'discount')]}">
                                <field name="discount" class="oe_inline me-1"/>
                                <field name="discount_mode" no_label="1" class="w-auto me-1"/>
                                <span>on</span>
                            </div>
                            <label for="discount_applicability" string="" attrs="{'invisible': [('reward_type', '!=', 'discount')]}"/>
                            <field name="discount_applicability" nolabel="1" widget="radio" attrs="{'invisible': [('reward_type', '!=', 'discount')]}"/>
                        </group>

                        <group string="Among" attrs="{'invisible': [('reward_type', '!=', 'product')]}">
                            <field name="reward_product_qty" string="Quantity rewarded"/>
                            <field name="reward_product_id" attrs="{'required': [('reward_type', '=', 'product'), ('reward_product_ids', '=', [])]}"/>
                            <field name="reward_product_tag_id" attrs="{'required': [('reward_type', '=', 'product'), ('reward_product_ids', '=', [])]}"/>
                        </group>

                        <group string="Discount" attrs="{'invisible': ['|', ('reward_type', '!=', 'discount'), ('program_type', 'in', ('gift_card', 'ewallet'))], }">
                            <field name="discount_max_amount"/>
                            <field name="discount_product_domain" groups="base.group_no_one" widget="domain" options="{'model': 'product.product', 'in_dialog': true}" attrs="{'invisible': [('discount_applicability', '!=', 'specific')]}"/>
                            <field name="discount_product_ids" widget="many2many_tags" attrs="{'invisible': [('discount_applicability', '!=', 'specific')]}"/>
                            <field name="discount_product_category_id" attrs="{'invisible': [('discount_applicability', '!=', 'specific')]}"/>
                            <field name="discount_product_tag_id" attrs="{'invisible': [('discount_applicability', '!=', 'specific')]}"/>
                        </group>
                    </group>
                    <group string="Points" attrs="{'invisible': [('user_has_debug', '=', False), ('program_type', 'not in', ('loyalty', 'buy_x_get_y'))]}">
                        <group>
                            <label for="required_points" string="In exchange of"/>
                            <div class="o_row">
                                <field name="required_points" class="oe_edit_only col-2 oe_inline text-center pe-2"/>
                                <field name="point_name" no_label="1"/>
                                <span attrs="{'invisible': [('clear_wallet', '!=', True)]}"> (or more)</span>
                            </div>
                            <label for="clear_wallet" string="Clear all promo point(s)" attrs="{'invisible': ['|','&amp;',('user_has_debug', '=', False), ('program_type', 'in', ('loyalty', 'buy_x_get_y')), ('program_type', 'in', ('ewallet','gift_card'))]}"/>
                            <div class="o_row" attrs="{'invisible': ['|','&amp;',('user_has_debug', '=', False), ('program_type', 'in', ('loyalty', 'buy_x_get_y')), ('program_type', 'in', ('ewallet','gift_card'))]}">
                                <field name="clear_wallet"/>
                            </div>
                        </group>
                    </group>
                    <group attrs="{'invisible': [('program_type', 'in', ('gift_card', 'ewallet'))]}">
                        <field name="description" string="Description on order"/>
                        <field name="discount_line_product_id" string="Discount product" groups="base.group_no_one"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="loyalty_reward_view_kanban" model="ir.ui.view">
        <field name="name">loyalty.reward.view.kanban</field>
        <field name="model">loyalty.reward</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="program_id"/>
                <field name="company_id"/>
                <field name="currency_id"/>
                <field name="description"/>
                <field name="reward_type"/>
                <field name="discount"/>
                <field name="discount_mode"/>
                <field name="discount_applicability"/>
                <field name="discount_product_domain"/>
                <field name="discount_product_category_id"/>
                <field name="discount_product_tag_id"/>
                <field name="discount_max_amount"/>
                <field name="discount_line_product_id"/>
                <field name="reward_product_id"/>
                <field name="reward_product_ids"/>
                <field name="all_discount_product_ids"/>
                <field name="reward_product_tag_id"/>
                <field name="multi_product"/>
                <field name="reward_product_qty"/>
                <field name="reward_product_uom_id"/>
                <field name="required_points"/>
                <field name="point_name"/>
                <field name="clear_wallet"/>
                <field name="program_type"/>
                <field name="user_has_debug"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click_edit mx-0 d-flex flex-row">
                            <div class="o_loyalty_kanban_card_left mw-75 flex-grow-1" id="reward_info">
                                <t t-if="record.reward_type.raw_value === 'discount'">

                                    <t t-if="record.discount">
                                        <a><field name="discount"/><field name="discount_mode"/> discount <t t-if="record.discount_max_amount.raw_value > 0">( Max <field name="discount_max_amount"/> )</t></a>
                                    </t> 

                                    <t t-if="record.discount_applicability.raw_value === 'specific'">
                                        <br/>
                                        <br/>

                                        <span class="fw-bold text-decoration-underline">Applied to:</span>

                                        <t t-if="record.discount_product_ids.raw_value.length > 0">
                                            <div class="d-flex"><i class="fa fa-cube fa-fw" title="Products"/> <field name="discount_product_ids" widget="many2many_tags" class="d-inline"/></div>
                                        </t>
                                        <t t-if="record.discount_product_category_id.raw_value">
                                            <div class="d-flex"><i class="fa fa-cubes fa-fw" title="Product Categories"/> <field name="discount_product_category_id" class="d-inline"/></div>
                                        </t>
                                        <t t-if="record.discount_product_tag_id.raw_value">
                                            <div class="d-flex"><i class="fa fa-tags fa-fw" title="Product Tags"/> <field name="discount_product_tag_id" class="d-inline"/></div>
                                        </t>  
                                        <t t-if="record.discount_product_domain.raw_value &amp;&amp; record.discount_product_domain.raw_value !== '[]'" groups="base.group_no_one">
                                            <div class="d-flex"><i class="fa fa-search fa-fw" title="Product Domain"/> <field name="discount_product_domain" class="d-inline"/></div>
                                        </t>
                                    </t>

                                    <t t-elif="record.discount_applicability.raw_value === 'cheapest'">
                                        on the cheapest product
                                        <br/>
                                    </t>

                                    <t t-elif="record.discount_applicability.raw_value === 'order'">
                                        on your order
                                        <br/>
                                    </t>
                                </t>

                                <t t-if="record.reward_type.raw_value === 'product'">
                                    <a>Free product</a>
                                    <br/>
                                    <br/>
                                    <t t-if="record.reward_product_id.raw_value">
                                        <div class="d-flex"><i class="fa fa-cube fa-fw" title="Product Domain"/><field name="reward_product_id"/> <t t-if="record.reward_product_qty.raw_value > 1"><span> x </span><field name="reward_product_qty"/></t></div>
                                    </t>
                                    <t t-if="record.reward_product_tag_id.raw_value">
                                        <div class="d-flex"><i class="fa fa-tags fa-fw" title="Product Tags"/> <field name="reward_product_tag_id" class="d-inline"/></div>
                                    </t> 
                                </t>
                            </div>

                            <div class="o_loyalty_kanban_card_right" attrs="{'invisible': [('user_has_debug', '=', False), ('program_type', 'not in', ('loyalty', 'buy_x_get_y'))]}">
                                <p class="text-muted">
                                    <span class="fw-bold text-decoration-underline">In exchange of</span>
                                    <br/>
                                    <t t-if="record.clear_wallet.raw_value">
                                        all <field name="point_name"/> (if at least <field name="required_points"/> <field name="point_name"/>)
                                    </t>
                                    <t t-else="">
                                        <field name="required_points"/> <field name="point_name"/>
                                    </t>
                                </p>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>
</odoo>

```

## File: views\loyalty_rule_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="loyalty_rule_view_form" model="ir.ui.view">
        <field name="name">loyalty.rule.view.form</field>
        <field name="model">loyalty.rule</field>
        <field name="arch" type="xml">
            <form class="loyalty-rule-form">
                <field name="program_type" invisible="1"/> 
                <field name="user_has_debug" invisible="1"/>
                <sheet>
                    <group attrs="{'invisible': [('program_type', '!=', 'promo_code')]}">
                        <group>
                            <field name="code" attrs="{'required': [('program_type', '=', 'promo_code')]}"/>
                        </group>
                    </group>
                    <group>
                        <group>
                            <separator string="Conditions" colspan="2"/>
                            <field name="minimum_qty"/>
                            <label for="minimum_amount" attrs="{'invisible': [('program_type', '=', 'buy_x_get_y')]}"/>
                            <div class="d-flex flex-row">
                                <field name="minimum_amount" class="oe_inline me-1"/>
                                <span>tax</span>
                                <field name="minimum_amount_tax_mode" class="ms-1"/>
                            </div>
                            <separator string="Among" colspan="2"/>
                            <field name="product_domain" groups="base.group_no_one" widget="domain" options="{'model': 'product.product', 'in_dialog': true}"/>
                            <field name="product_ids" widget="many2many_tags"/>
                            <field name="product_category_id"/>
                            <field name="product_tag_id"/>
                        </group>
                        <group attrs="{'invisible': [('user_has_debug', '=', False), ('program_type', 'not in', ('loyalty', 'buy_x_get_y'))]}">
                            <separator string="Point(s)" colspan="2"/>
                            <span colspan="2" attrs="{'invisible': [('program_type', '!=', 'coupons')]}">Grant the amount of coupon points defined as the coupon value</span>
                            <label for="reward_point_amount" string="Grant" attrs="{'invisible': [('program_type', 'not in', ('promotion', 'promo_code', 'next_order_coupons', 'loyalty', 'buy_x_get_y'))]}"/>
                            <div class="d-flex flex-row" attrs="{'invisible': [('program_type', 'not in', ('promotion', 'promo_code', 'next_order_coupons', 'loyalty', 'buy_x_get_y'))]}">
                                <field name="reward_point_amount" class="oe_inline me-1"/>
                                <field name="reward_point_name" class="w-auto"/>
                            </div>
                            <label for="reward_point_mode" string=""/>
                            <field name="reward_point_mode" widget="radio" nolabel="1" attrs="{'invisible': [('program_type', 'not in', ('promotion', 'promo_code', 'next_order_coupons', 'loyalty', 'buy_x_get_y'))]}"/>     
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="loyalty_rule_view_kanban" model="ir.ui.view">
        <field name="name">loyalty.rule.view.kanban</field>
        <field name="model">loyalty.rule</field>
        <field name="arch" type="xml">
            <kanban>
                <field name="program_id"/>
                <field name="company_id"/>
                <field name="currency_id"/>
                <field name="product_domain"/>
                <field name="product_ids"/>
                <field name="product_category_id"/>
                <field name="product_tag_id"/>
                <field name="reward_point_amount"/>
                <field name="reward_point_split"/>
                <field name="reward_point_name"/>
                <field name="reward_point_mode"/>
                <field name="minimum_qty"/>
                <field name="minimum_amount"/>
                <field name="minimum_amount_tax_mode"/>
                <field name="mode"/>
                <field name="code"/>
                <field name="program_type"/>
                <field name="user_has_debug"/>
                <templates>
                    <t t-name="kanban-box">
                        <div class="oe_kanban_global_click_edit mx-0 d-flex flex-row">
                            <div class="o_loyalty_kanban_card_left mw-75 flex-grow-1">
                                <t t-if="record.code.raw_value"><span>Discount code <field name="code"/></span><br/></t>
                                <t t-if="record.minimum_qty.raw_value > 0"><span>If minimum <field name="minimum_qty"/> item(s) bought</span><br/></t>
                                <t t-if="record.minimum_amount.raw_value > 0"><span>If minimum <field name="minimum_amount"/> spent<t t-if="record.minimum_amount_tax_mode.raw_value === 'excl'"> (tax excluded)</t></span><br/></t>
                                <br/>

                                <t t-if="record.product_ids.raw_value.length != 0 || record.product_category_id.raw_value || record.product_tag_id.raw_value">
                                    <span class="fw-bold text-decoration-underline">Among:</span>
                                    <br/>

                                    <t t-if="record.product_ids.raw_value.length > 0">
                                        <div class="d-flex"><i class="fa fa-cube fa-fw" title="Products"/> <field name="product_ids" widget="many2many_tags" class="d-inline"/></div>
                                    </t>
                                    <t t-if="record.product_category_id.raw_value">
                                        <div class="d-flex"><i class="fa fa-cubes fa-fw" title="Product Categories"/> <field name="product_category_id" class="d-inline"/></div>
                                    </t>
                                    <t t-if="record.product_tag_id.raw_value">
                                        <div class="d-flex"><i class="fa fa-tags fa-fw" title="Product Tags"/> <field name="product_tag_id" class="d-inline"/></div>
                                    </t>
                                    <t t-if="record.product_ids.raw_value.length === 0 &amp;&amp; !record.product_category_id.raw_value &amp;&amp; !record.product_tag_id.raw_value">
                                        <div class="d-flex"><i class="fa fa-cube fa-fw" title="Products"/><span>All Products</span></div>
                                    </t>
                                    <t t-if="record.product_domain.raw_value &amp;&amp; record.product_domain.raw_value !== '[]'" groups="base.group_no_one">
                                        <div class="d-flex"><i class="fa fa-search fa-fw" title="Product Domain"/> <field name="product_domain" class="d-inline"/></div>
                                    </t>
                                </t>
                            </div>

                            <div class="o_loyalty_kanban_card_right" attrs="{'invisible': [('user_has_debug', '=', False), ('program_type', 'not in', ('loyalty', 'buy_x_get_y'))]}">
                                <p class="text-muted" attrs="{'invisible': [('program_type', '!=', 'coupons')]}">
                                    <span class="fw-bold text-decoration-underline">Grant</span>
                                    <br/>
                                    the value of the coupon
                                </p>
                                <p class="text-muted" attrs="{'invisible': [('program_type', 'not in', ('promotion', 'promo_code', 'next_order_coupons', 'loyalty', 'buy_x_get_y'))]}">
                                    <span class="fw-bold text-decoration-underline">Grant</span>
                                    <br/>
                                    <field name="reward_point_amount"/>
                                    <span> </span>
                                    <field name="reward_point_name"/>
                                    <span> </span>
                                    <field name="reward_point_mode"/>
                                </p>
                            </div>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>
</odoo>

```

## File: wizard\loyalty_generate_wizard.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
from odoo.osv import expression

class LoyaltyGenerateWizard(models.TransientModel):
    _name = 'loyalty.generate.wizard'
    _description = 'Generate Coupons'

    program_id = fields.Many2one('loyalty.program', required=True, default=lambda self: self.env.context.get('active_id', False) or self.env.context.get('default_program_id', False))
    program_type = fields.Selection(related='program_id.program_type')

    mode = fields.Selection([
        ('anonymous', 'Anonymous Customers'),
        ('selected', 'Selected Customers')],
        string='For', required=True, default='anonymous'
    )

    customer_ids = fields.Many2many('res.partner', string='Customers')
    customer_tag_ids = fields.Many2many('res.partner.category', string='Customer Tags')

    coupon_qty = fields.Integer('Quantity',
        compute='_compute_coupon_qty', readonly=False, store=True)
    points_granted = fields.Float('Grant', required=True, default=1)
    points_name = fields.Char(related='program_id.portal_point_name', readonly=True)
    valid_until = fields.Date()
    will_send_mail = fields.Boolean(compute='_compute_will_send_mail')

    def _get_partners(self):
        self.ensure_one()
        if self.mode != 'selected':
            return self.env['res.partner']
        domain = []
        if self.customer_ids:
            domain = [('id', 'in', self.customer_ids.ids)]
        if self.customer_tag_ids:
            domain = expression.OR([domain, [('category_id', 'in', self.customer_tag_ids.ids)]])
        return self.env['res.partner'].search(domain)

    @api.depends('customer_ids', 'customer_tag_ids', 'mode')
    def _compute_coupon_qty(self):
        for wizard in self:
            if wizard.mode == 'selected':
                wizard.coupon_qty = len(wizard._get_partners())
            else:
                wizard.coupon_qty = wizard.coupon_qty or 0

    @api.depends("mode", "program_id")
    def _compute_will_send_mail(self):
        for wizard in self:
            wizard.will_send_mail = wizard.mode == 'selected' and 'create' in wizard.program_id.mapped('communication_plan_ids.trigger')

    def _get_coupon_values(self, partner):
        self.ensure_one()
        return {
            'program_id': self.program_id.id,
            'points': self.points_granted,
            'expiration_date': self.valid_until,
            'partner_id': partner.id if self.mode == 'selected' else False,
        }

    def generate_coupons(self):
        if any(not wizard.program_id for wizard in self):
            raise ValidationError(_("Can not generate coupon, no program is set."))
        if any(wizard.coupon_qty <= 0 for wizard in self):
            raise ValidationError(_("Invalid quantity."))
        coupon_create_vals = []
        for wizard in self:
            customers = wizard._get_partners() or range(wizard.coupon_qty)
            for partner in customers:
                coupon_create_vals.append(wizard._get_coupon_values(partner))
        self.env['loyalty.card'].create(coupon_create_vals)
        return True

```

## File: wizard\loyalty_generate_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="loyalty_generate_wizard_view_form" model="ir.ui.view">
        <field name="name">loyalty.generate.wizard.view.form</field>
        <field name="model">loyalty.generate.wizard</field>
        <field name="arch" type="xml">
            <form string="Generate">
                <sheet>
                    <group>
                        <group>
                            <field name="program_id" invisible="1"/>
                            <field name="will_send_mail" invisible="1"/>
                            <field name="program_type" invisible="1"/>
                            <field name="mode" widget="radio" attrs="{'invisible': [('program_type', '=', 'ewallet')]}"/>
                            <field name="customer_ids" widget="many2many_tags" attrs="{'invisible': [('mode', '=', 'anonymous')]}"/>
                            <field name="customer_tag_ids" widget="many2many_tags" attrs="{'invisible': [('mode', '=', 'anonymous')]}" options="{'color_field': 'color'}"/>
                            <field name="coupon_qty" string="Quantity to generate" attrs="{'readonly': [('mode', '=', 'selected')], 'required': [('mode', '=', 'anonymous')]}"/>
                            <label string="Coupon value" for="points_granted" groups="base.group_no_one" attrs="{'invisible': [('program_type', '!=', 'coupons')]}"/>
                            <span class="d-inline-block" groups="base.group_no_one" attrs="{'invisible': [('program_type', '!=', 'coupons')]}">
                                <field name="points_granted" class="oe_inline"/>
                                <field name="points_name" class="oe_inline"/>
                            </span>
                            <label string="Gift Card value" for="points_granted" attrs="{'invisible': [('program_type', '!=', 'gift_card')]}"/>
                            <span class="d-inline-block" attrs="{'invisible': [('program_type', '!=', 'gift_card')]}">
                                <field name="points_granted" class="w-auto oe_inline me-1"/>
                                <field name="points_name" class="d-inline" no_label="1"/>
                            </span>
                            <label string="eWallet value" for="points_granted" attrs="{'invisible': [('program_type', '!=', 'ewallet')]}"/>
                            <span class="d-inline-block" attrs="{'invisible': [('program_type', '!=', 'ewallet')]}">
                                <field name="points_granted" class="w-auto oe_inline me-1"/>
                                <field name="points_name" class="d-inline" no_label="1"/>
                            </span>
                            <field name="valid_until"/>
                        </group>
                    </group>
                </sheet>
                <footer>
                    <button name="generate_coupons" type="object" class="btn-primary" data-hotkey="q">
                        <span attrs="{'invisible': [('will_send_mail', '=', False)]}">
                            Generate and Send 
                        </span>
                        <span attrs="{'invisible': [('will_send_mail', '=', True)]}">
                            Generate 
                        </span>
                        <field name="program_type" nolabel="1"/>
                    </button>
                    <button special="cancel" data-hotkey="z" string="Cancel"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="loyalty_generate_wizard_action" model="ir.actions.act_window">
        <field name="name">Generate</field>
        <field name="res_model">loyalty.generate.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import loyalty_generate_wizard

```

