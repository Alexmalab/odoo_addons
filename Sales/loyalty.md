# Odoo Module: loyalty

Category: Sales

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import controllers
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
    'depends': ['product', 'portal', 'account'],
    'data': [
        'security/ir.model.access.csv',
        'security/loyalty_security.xml',
        'report/loyalty_report_templates.xml',
        'report/loyalty_report.xml',
        'data/mail_template_data.xml',
        'data/loyalty_data.xml',
        'wizard/loyalty_card_update_balance_views.xml',
        'wizard/loyalty_generate_wizard_views.xml',
        'views/loyalty_card_views.xml',
        'views/loyalty_history_views.xml',
        'views/loyalty_mail_views.xml',
        'views/loyalty_program_views.xml',
        'views/loyalty_reward_views.xml',
        'views/loyalty_rule_views.xml',
        'views/portal_templates.xml',
        'views/res_partner_views.xml',
    ],
    'demo': [
        'data/loyalty_demo.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'loyalty/static/src/js/**/*.js',
            'loyalty/static/src/scss/*.scss',
            'loyalty/static/src/xml/*.xml',

            ('remove', 'loyalty/static/src/js/portal/**/*'),
        ],
        'web.assets_frontend': [
            'loyalty/static/src/js/portal/**/*',
        ],
    },
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: controllers\portal.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields
from odoo.http import request, route

from odoo.addons.portal.controllers.portal import CustomerPortal
from odoo.addons.portal.controllers.portal import pager as portal_pager


class CustomerPortalLoyalty(CustomerPortal):

    def _prepare_home_portal_values(self, counters):
        values = super()._prepare_home_portal_values(counters)
        if not counters:
            # we want those data to be added to the /my/home page only, and always computed
            values['cards_per_programs'] = dict(request.env['loyalty.card'].sudo()._read_group(
                domain=[
                    ('partner_id', '=', request.env.user.partner_id.id),
                    ('program_id.active', '=', True),
                    ('program_id.program_type', 'in', ['loyalty', 'ewallet']),
                    '|',
                        ('expiration_date', '>=', fields.Date().today()),
                        ('expiration_date', '=', False),
                ],
                groupby=['program_id'],
                aggregates=['id:recordset'],
            ))

        return values

    def _get_loyalty_searchbar_sortings(self):
        return {
            'date': {'label': _("Date"), 'order': 'create_date desc'},
            'used': {'label': _("Used"), 'order': 'used desc'},
            'description': {'label': _("Description"), 'order': 'description desc'},
            'issued': {'label': _("Issued"), 'order': 'issued desc'},
        }

    @route(
        [
            '/my/loyalty_card/<int:card_id>/history',
            '/my/loyalty_card/<int:card_id>/history/page/<int:page>',
        ],
        type='http',
        auth='user',
        website=True,
    )
    def portal_my_loyalty_card_history(self, card_id, page=1, sortby='date', **kw):
        card_sudo = request.env['loyalty.card'].sudo().search([
            ('id', '=', int(card_id)),
            ('partner_id', '=', request.env.user.partner_id.id),
        ])
        if not card_sudo:
            return request.redirect('/my')

        LoyaltyHistorySudo = request.env['loyalty.history'].sudo()
        searchbar_sortings = self._get_loyalty_searchbar_sortings()
        order = searchbar_sortings[sortby]['order']
        lines_count = LoyaltyHistorySudo.search_count([('card_id', '=', card_id)])
        pager = portal_pager(
            url='/my/loyalty_card/<int:card_id>/history',
            url_args={'sortby': sortby, 'card_id': card_id},
            total=lines_count,
            page=page,
            step=self._items_per_page,
        )
        history_lines = LoyaltyHistorySudo.search(
            domain=[
                ('card_id', '=', card_id),
                ('card_id.partner_id', '=', request.env.user.partner_id.id)
            ],
            order=order,
            limit=self._items_per_page,
            offset=pager['offset'],
        )
        values = {
            'pager': pager,
            'searchbar_sortings': searchbar_sortings,
            'page_name': 'loyalty_history',
            'sortby': sortby,
            'history_lines': history_lines,
        }

        return request.render('loyalty.loyalty_card_history_template', values)

    @route('/my/loyalty_card/<int:card_id>/values', type='json', auth='user')
    def portal_get_card_history_values(self, card_id):
        card_sudo = request.env['loyalty.card'].sudo().search([
            ('id', '=', int(card_id)),
            ('partner_id', '=', request.env.user.partner_id.id)
        ])
        if not card_sudo:
            return {}

        program_type = card_sudo.program_id.program_type
        rewards = request.env['loyalty.reward'].sudo().search(
            [
                ('program_id', '=', card_sudo.program_id.id),
                ('required_points', '<=', card_sudo.points)
            ],
            order='required_points desc',
            limit=3,
        )
        return {
            'card': {
                'id': card_sudo.id,
                'points_display': card_sudo.points_display,
                'expiration_date': card_sudo.expiration_date,
                'code': card_sudo.code,
            },
            'program': {
                'program_name': card_sudo.program_id.name,
                'program_type': program_type,
            },
            'history_lines': [{
                'order_id': line.order_id,
                'description': line.description,
                'order_portal_url': line._get_order_portal_url(),
                'points': f'{"-" if line.issued < line.used else "+"}'
                          f'{card_sudo._format_points(abs(line.issued - line.used))}',
            } for line in card_sudo.history_ids[:5]],
            'rewards': [{
                'description': reward.description,
                'points': card_sudo._format_points(reward.required_points),
            } for reward in rewards],
            'img_path': f'/loyalty/static/src/img/{program_type}.svg',
        }

```

## File: controllers\__init__.py

```python
from . import portal

```

## File: data\loyalty_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Basic product for gift card program -->
    <record id="gift_card_product_50" model="product.product">
        <field name="name">Gift Card</field>
        <field name="list_price">50</field>
        <field name="type">service</field>
        <field name="purchase_ok" eval="False"/>
        <field name="image_1920" type="base64" file="loyalty/static/img/gift_card.png"/>
    </record>
    <!-- Basic product for eWallet programs -->
    <record id="ewallet_product_50" model="product.product">
        <field name="name">Top-up eWallet</field>
        <field name="list_price">50</field>
        <field name="type">service</field>
        <field name="purchase_ok" eval="False"/>
    </record>

    <data noupdate="1">
        <record forcecreate="0" id="config_online_sync_proxy_mode" model="ir.config_parameter">
            <field name="key">loyalty.compute_all_discount_product_ids</field>
            <field name="value">False</field>
        </record>
    </data>

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
    <record id="gift_card_program_reward" model="loyalty.reward">
        <field name="reward_type">discount</field>
        <field name="discount_mode">per_point</field>
        <field name="discount">1</field>
        <field name="discount_applicability">order</field>
        <field name="required_points">1</field>
        <field name="program_id" ref="loyalty.gift_card_program"/>
    </record>
    <record id="gift_card_program_rule" model="loyalty.rule">
        <field name="reward_point_amount">1</field>
        <field name="reward_point_mode">money</field>
        <field name="reward_point_split">True</field>
        <field name="product_ids" eval="[(4, ref('loyalty.gift_card_product_50'))]"/>
        <field name="program_id" ref="loyalty.gift_card_program"/>
    </record>
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
        <field name="report_template_ids" eval="[(4, ref('loyalty.report_gift_card'))]"/>
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
        <field name="report_template_ids" eval="[(4, ref('loyalty.report_loyalty_card'))]"/>
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

    @api.depends('program_id', 'code')
    def _compute_display_name(self):
        for card in self:
            card.display_name = f'{card.program_id.name}: {card.code}'

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
    active = fields.Boolean(default=True)
    history_ids = fields.One2many(
        comodel_name='loyalty.history',
        inverse_name='card_id',
        readonly=True,
    )

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
            default_res_ids=self.ids,
            default_template_id=default_template and default_template.id,
            default_composition_mode='comment',
            default_email_layout_xmlid='mail.mail_notification_light',
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

    def action_loyalty_update_balance(self):
        return {
            'name': _("Update Balance"),
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'loyalty.card.update.balance',
            'target': 'new',
            'context': {
                'default_card_id': self.id,
            },
        }

```

## File: models\loyalty_history.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class LoyaltyHistory(models.Model):
    _name = 'loyalty.history'
    _description = "History for Loyalty cards and Ewallets"
    _order = 'id desc'

    card_id = fields.Many2one(comodel_name='loyalty.card', required=True, ondelete='cascade')
    company_id = fields.Many2one(related='card_id.company_id')

    description = fields.Text(required=True)

    issued = fields.Float()
    used = fields.Float()

    order_model = fields.Char(readonly=True)
    order_id = fields.Many2oneReference(model_field='order_model', readonly=True)

    def _get_order_portal_url(self):
        self.ensure_one()
        return False

    def _get_order_description(self):
        self.ensure_one()
        return self.env[self.order_model].browse(self.order_id).display_name

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
    mail_template_id = fields.Many2one('mail.template', string="Email Template", required=True, domain=[('model', '=', 'loyalty.card')], ondelete='cascade')

```

## File: models\loyalty_program.py

```python
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

    @api.model
    def default_get(self, fields_list):
        defaults = super().default_get(fields_list)
        program_type = defaults.get('program_type')
        if program_type:
            program_default_values = self._program_type_default_values()
            if program_type in program_default_values:
                default_values = program_default_values[program_type]
                defaults.update({k: v for k, v in default_values.items() if k in fields_list})
        return defaults

    name = fields.Char('Program Name', required=True, translate=True)
    active = fields.Boolean(default=True)
    sequence = fields.Integer(copy=False)
    company_id = fields.Many2one('res.company', 'Company', default=lambda self: self.env.company)
    currency_id = fields.Many2one('res.currency', 'Currency', compute='_compute_currency_id',
        readonly=False, required=True, store=True, precompute=True)
    currency_symbol = fields.Char(related='currency_id.symbol')
    pricelist_ids = fields.Many2many(
        string="Pricelist",
        help="This program is specific to this pricelist set.",
        comodel_name='product.pricelist',
        domain="[('currency_id', '=', currency_id)]",
    )

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
    date_from = fields.Date(
        string="Start Date",
        help="The start date is included in the validity period of this program",
    )
    date_to = fields.Date(
        string="End date",
        help="The end date is included in the validity period of this program",
    )
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

    @api.constrains('currency_id', 'pricelist_ids')
    def _check_pricelist_currency(self):
        if any(
            pricelist.currency_id != program.currency_id
            for program in self
            for pricelist in program.pricelist_ids
        ):
            raise UserError(_(
                "The loyalty program's currency must be the same as all it's pricelists ones."
            ))

    @api.constrains('date_from', 'date_to')
    def _check_date_from_date_to(self):
        if any(p.date_to and p.date_from and p.date_from > p.date_to for p in self):
            raise UserError(_(
                "The validity period's start date must be anterior or equal to its end date."
            ))

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
        read_group_data = self.env['loyalty.card']._read_group([('program_id', 'in', self.ids)], ['program_id'], ['__count'])
        count_per_program = {program.id: count for program, count in read_group_data}
        for program in self:
            program.coupon_count = count_per_program.get(program.id, 0)

    @api.depends('program_type', 'applies_on')
    def _compute_is_nominative(self):
        for program in self:
            program.is_nominative = program.applies_on == 'both' or\
                (program.program_type in ('ewallet', 'loyalty') and program.applies_on == 'future')

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
        first_sale_product = self.env['product.product'].search([('company_id', 'in', [False, self.env.company.id]), ('sale_ok', '=', True)], limit=1)
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
                    'reward_point_split': False,
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
                    'description': _("Sell Gift Cards, that allows to purchase products"),
                    'icon': 'gift_card',
                },
                'ewallet': {
                    'title': _("eWallet"),
                    'description': _("Fill in your eWallet, to pay future orders"),
                    'icon': 'ewallet',
                },
            }
        return {
            'promotion': {
                'title': _("Promotional Program"),
                'description': _("Automatic promo: 10% off on orders higher than $50"),
                'icon': 'promotional_program',
            },
            'promo_code': {
                'title': _("Promo Code"),
                'description': _("Get 10% off on some products, with a code"),
                'icon': 'promo_code',
            },
            'buy_x_get_y': {
                'title': _("Buy X Get Y"),
                'description': _("Buy 2 products and get a third one for free"),
                'icon': '2_plus_1',
            },
            'next_order_coupons': {
                'title': _("Next Order Coupon"),
                'description': _("Send a coupon after an order, valid for next purchase"),
                'icon': 'coupons',
            },
            'loyalty': {
                'title': _("Loyalty Card"),
                'description': _("Win points with each purchase, and claim gifts"),
                'icon': 'loyalty_cards',
            },
            'coupons': {
                'title': _("Coupon"),
                'description': _("Generate and share unique coupons with your customers"),
                'icon': 'coupons',
            },
            'fidelity': {
                'title': _("Fidelity Card"),
                'description': _("Buy 10 products to get 10$ off on the 11th one"),
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import ast
import json

from odoo import _, api, fields, models
from odoo.exceptions import ValidationError
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
            ('per_order', symbol),
            ('per_point', _('%s per point', symbol)),
        ]

    @api.depends('program_id', 'description')
    def _compute_display_name(self):
        for reward in self:
            reward.display_name = f'{reward.program_id.name} - {reward.description}'

    active = fields.Boolean(default=True)
    program_id = fields.Many2one('loyalty.program', required=True, ondelete='cascade')
    program_type = fields.Selection(related="program_id.program_type")
    # Stored for security rules
    company_id = fields.Many2one(related='program_id.company_id', store=True)
    currency_id = fields.Many2one(related='program_id.currency_id')

    description = fields.Char(
        translate=True,
        compute='_compute_description',
        precompute=True,
        store=True,
        readonly=False,
        required=True,
    )

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
    tax_ids = fields.Many2many(
        string="Taxes",
        help="Taxes to add on the discount line.",
        comodel_name='account.tax',
        domain="[('type_tax_use', '=', 'sale'), ('company_id', '=', company_id)]",
    )

    # Product rewards
    reward_product_id = fields.Many2one(
        'product.product', string='Product', domain=[('type', '!=', 'combo')]
    )
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
        constrains = []
        if self.discount_product_ids:
            constrains.append([('id', 'in', self.discount_product_ids.ids)])
        if self.discount_product_category_id:
            product_category_ids = self._find_all_category_children(self.discount_product_category_id, [])
            product_category_ids.append(self.discount_product_category_id.id)
            constrains.append([('categ_id', 'in', product_category_ids)])
        if self.discount_product_tag_id:
            constrains.append([('all_product_tag_ids', 'in', self.discount_product_tag_id.id)])
        domain = expression.OR(constrains) if constrains else []
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
            products = reward.reward_product_id + reward.reward_product_tag_id.product_ids.filtered(
                lambda product: product.type != 'combo'
            )
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
                    reward_string = _('Free Product - [%s]', ', '.join(products.with_context(display_default_code=False).mapped('display_name')))
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
                    reward_string = _('%s on ', formatted_amount)
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
            reward.is_global_discount = (
                reward.reward_type == 'discount'
                and reward.discount_applicability == 'order'
                and reward.discount_mode in ['per_order', 'percent']
            )

    @api.depends_context('uid')
    @api.depends("reward_type")
    def _compute_user_has_debug(self):
        self.user_has_debug = self.env.user.has_group('base.group_no_one')

    @api.constrains('reward_product_id')
    def _check_reward_product_id_no_combo(self):
        if any(reward.reward_product_id.type == 'combo' for reward in self):
            raise ValidationError(_("A reward product can't be of type \"combo\"."))

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
    minimum_amount_tax_mode = fields.Selection(
        selection=[
            ('incl', "tax included"),
            ('excl', "tax excluded"),
        ],
        default='incl',
        required=True,
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
        self.user_has_debug = self.env.user.has_group('base.group_no_one')

    def _get_valid_product_domain(self):
        self.ensure_one()
        constrains = []
        if self.product_ids:
            constrains.append([('id', 'in', self.product_ids.ids)])
        if self.product_category_id:
            constrains.append([('categ_id', 'child_of', self.product_category_id.id)])
        if self.product_tag_id:
            constrains.append([('all_product_tag_ids', 'in', self.product_tag_id.id)])
        domain = expression.OR(constrains) if constrains else []
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

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class ResPartner(models.Model):
    _inherit = 'res.partner'

    loyalty_card_count = fields.Integer(
        string="Active loyalty cards",
        compute='_compute_count_active_cards',
        compute_sudo=True,
        groups='base.group_user')

    def _compute_count_active_cards(self):
        loyalty_groups = self.env['loyalty.card']._read_group(
            domain=[
                '|', ('company_id', '=', False), ('company_id', 'in', self.env.companies.ids),
                ('partner_id', 'in', self.with_context(active_test=False)._search([('id', 'child_of', self.ids)])),
                ('points', '>', '0'),
                ('program_id.active', '=', True),
                '|',
                    ('expiration_date', '>=', fields.Date().context_today(self)),
                    ('expiration_date', '=', False),
            ],
            groupby=['partner_id'],
            aggregates=['__count'],
        )
        self.loyalty_card_count = 0
        for partner, count in loyalty_groups:
            while partner:
                if partner in self:
                    partner.loyalty_card_count += count
                partner = partner.parent_id

    def action_view_loyalty_cards(self):
        action = self.env['ir.actions.act_window']._for_xml_id('loyalty.loyalty_card_action')
        all_child = self.with_context(active_test=False).search([('id', 'child_of', self.ids)])
        action['domain'] = [('partner_id', 'in', all_child.ids)]
        action['context'] = {'search_default_active' : True, 'create': False}
        return action

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import loyalty_card
from . import loyalty_history
from . import loyalty_mail
from . import loyalty_reward
from . import loyalty_rule
from . import loyalty_program
from . import product_product
from . import product_template
from . import res_partner

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
                        <div class="oe_structure"></div>
                        <div class="row text-center">
                            <div class="o_offer col-lg-12">
                                <h4 t-if="o._get_mail_partner().name">
                                    Congratulations
                                    <span t-out="o._get_mail_partner().name">John Doe</span>,
                                </h4>
                                <t t-set="text">on your next order</t>
                                <h4>Here is your reward from <span t-out="o.program_id.company_id.name">Odoo</span>.</h4>
                                <div class="oe_structure"></div>
                                <t t-foreach="range(len(o.program_id.reward_ids))" t-as="reward_idx">
                                    <t t-set="reward" t-value="o.program_id.reward_ids[reward_idx]"/>
                                    <strong><span t-out="reward.description">loyalty Reward</span></strong>
                                    <br/>
                                    <t t-if="reward_idx &lt; (len(o.program_id.reward_ids) - 1)">
                                        <span class="text-center">OR</span>
                                        <br/>
                                    </t>
                                </t>
                                <h1 class="fw-bold" style="font-size: 34px"><span t-out="text">DEMO_TEXT</span></h1>
                                <br/>
                                <h4 t-if="o.expiration_date">
                                    Use this promo code before
                                    <span t-field="o.expiration_date" t-options='{"format": "yyyy-MM-d"}'>2023-08-20</span>
                                </h4>
                                <h2 class="mt-4">
                                    <strong class="bg-light"><span t-out="o.code">DEMO_CODE</span></strong>
                                </h2>
                                <t t-set="rule" t-value="o.program_id.rule_ids[:1]"/>
                                <h4 t-if="rule.minimum_qty > 1">
                                    <span>Minimum purchase of</span>
                                    <strong t-out="rule.minimum_qty">5</strong> <span>products</span>
                                </h4>
                                <h4 t-if="rule.minimum_amount">
                                    <span>Valid for purchase above</span>
                                    <strong t-out="rule.minimum_amount" t-options="{'widget': 'monetary', 'display_currency': rule.currency_id}">$100</strong>
                                </h4>
                                <div class="oe_structure"></div>
                                <br/>
                                <span t-field="o.code" t-options="{'widget': 'barcode', 'width': 600, 'height': 100}">ABCDE12345</span>
                                <div class="oe_structure"></div>
                                <br/><br/>
                                <h4>Thank you,</h4>
                                <div class="oe_structure"></div>
                                <br/>
                                <div class="mt32">
                                    <div class="text-center">
                                        <img alt="Logo" t-att-src="'/logo?company=%d' % (o.program_id.company_id)" t-att-alt="'%s' % (o.program_id.company_id.name)" style="border:0 solid transparent;" height="50"/>
                                    </div>
                                </div>
                                <div>
                                    <div class="text-center d-inline-block">
                                        <span t-field="o.program_id.company_id.partner_id"
                                            t-options='{"widget": "contact", "fields": ["address", "email"], "no_marker": True}'>John Doe</span>
                                    </div>
                                </div>
                                <div class="oe_structure"></div>
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
                <div class="oe_structure"></div>
                <div style="margin:0px; font-size:24px; font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:36px; color:#333333; text-align: center">
                    Here is your gift card!
                </div>
                <div class="oe_structure"></div>
                <div style="padding-top:20px; padding-bottom:20px">
                    <img src="/loyalty/static/img/gift_card.png" style="display:block; border:0; outline:none; text-decoration:none; margin:auto;" width="300"/>
                </div>
                <div class="oe_structure"></div>
                <div style="padding:0; margin:0px; padding-top:35px; padding-bottom:35px; text-align:center;">
                    <h3 style="margin:0px; line-height:48px; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:40px; font-style:normal; font-weight:normal; color:#333333; text-align:center">
                        <strong><span t-out="o.points" t-options="{'widget': 'monetary', 'display_currency': o.currency_id}">1000</span></strong>
                    </h3>
                </div>
                <div class="oe_structure"></div>
                <div style="padding:0; margin:0px; padding-top:35px; padding-bottom:35px; background-color:#efefef; text-align:center;">
                    <p style="margin:0px; font-size:14px;font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:21px; color:#333333">
                        <strong>Gift Card Code</strong>
                    </p>
                    <p style="margin:0px; font-size:25px;font-family:arial, 'helvetica neue', helvetica, sans-serif; line-height:38px; color:#A9A9A9">
                        <span t-field="o.code">ABCDE12345</span>
                    </p>
                </div>
                <div class="oe_structure"></div>
                <div t-if="o.expiration_date" style="padding:0; margin:0px; padding-top:10px; padding-bottom:10px; text-align:center;">
                    <h3 style="margin:0px; line-height:17px; font-family:arial, 'helvetica neue', helvetica, sans-serif; font-size:14px; font-style:normal; font-weight:normal; color:#A9A9A9; text-align:center">
                        Card expires <span t-field="o.expiration_date">2023-12-31</span>
                    </h3>
                </div>
                <div class="oe_structure"></div>
                <div style="padding:0; margin:0px; padding-top:10px; padding-bottom:10px; text-align:center;">
                    <img t-att-src="'/report/barcode/Code128/'+o.code" style="width:400px;height:75px" alt="Barcode"/>
                </div>
                <div class="oe_structure"></div>  
            </t>
        </t>
    </template>

    <template id="gift_card_report_i18n">
        <t t-call="web.html_container">
            <div class="oe_structure"></div>
            <t t-foreach="docs" t-as="o">
                <t t-set="o" t-value="o.with_context(lang=o._get_mail_partner().lang or o.env.lang)"/>
                <t t-call="loyalty.gift_card_report" t-lang="o._get_mail_partner().lang or o.env.lang"/>
            </t>
            <div class="oe_structure"></div>
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
access_loyalty_history,access_loyalty_history,model_loyalty_history,base.group_user,0,0,0,0
access_loyalty_card_update_balance_user,access_loyalty_card_update_balance,model_loyalty_card_update_balance,base.group_user,0,0,0,0

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

        <record id="loyalty_history_company_rule" model="ir.rule">
            <field name="name">Loyalty history multi company rule</field>
            <field name="model_id" ref="model_loyalty_history"/>
            <field name="domain_force">[('company_id', 'in', company_ids + [False])]</field>
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

## File: static\img\portal-loyalty.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M46.1574 46.4595L42.8079 44.513L36.2025 37.3103L39.552 39.2568L46.1574 46.4595Z" fill="#FBDBD0"/>
<path d="M46.1575 46.6881C46.0945 46.6881 46.0329 46.662 45.989 46.614L39.3836 39.4113C39.3031 39.3234 39.3037 39.1883 39.385 39.1011C42.4569 35.804 45.0001 31.7863 46.7403 27.4821C46.7696 27.4093 46.8343 27.3569 46.9115 27.3428C46.9249 27.3404 46.9385 27.3392 46.9521 27.3392C47.0155 27.3392 47.0769 27.3656 47.1207 27.4133L54.3577 35.3033C54.4093 35.3596 54.4294 35.4379 54.4113 35.512C54.3932 35.5861 54.3392 35.6464 54.2673 35.6725L47.6041 38.0993L46.3836 46.4924C46.3709 46.5799 46.3091 46.6519 46.2249 46.6781C46.2026 46.6848 46.18 46.6881 46.1575 46.6881Z" fill="white"/>
<path d="M46.9521 27.5679L54.1892 35.4577L47.3975 37.9313L46.1574 46.4595L39.552 39.2568C42.6392 35.9434 45.2241 31.8423 46.9521 27.5679ZM46.9521 27.1107C46.9251 27.1107 46.8978 27.1131 46.8707 27.118C46.7163 27.146 46.5871 27.2511 46.5283 27.3965C44.8234 31.6137 42.227 35.7151 39.2175 38.9452C39.0549 39.1197 39.0538 39.39 39.2151 39.5658L45.8205 46.7685C45.9084 46.8644 46.0313 46.9166 46.1574 46.9166C46.2023 46.9166 46.2477 46.9101 46.292 46.8964C46.4607 46.8444 46.5843 46.7 46.6098 46.5253L47.8106 38.2673L54.3457 35.8872C54.4891 35.835 54.5971 35.7146 54.6333 35.5662C54.6695 35.4179 54.6293 35.2613 54.5261 35.1487L47.289 27.2588C47.2016 27.1636 47.079 27.1107 46.9521 27.1107Z" fill="#374874"/>
<path d="M21.228 60.8524L17.8786 58.9059L16.5942 51.8352L19.9437 53.7817L21.228 60.8524Z" fill="#FBDBD0"/>
<path d="M13.1603 59.1457L9.81079 57.1993L16.476 42.1309L19.8254 44.0774L13.1603 59.1457Z" fill="#FBDBD0"/>
<path d="M21.2009 61.0794C21.1011 61.0676 21.021 60.9919 21.0029 60.8932L19.7869 54.197L13.3018 59.3249C13.2603 59.3578 13.2103 59.3743 13.1601 59.3743C13.1107 59.3743 13.0614 59.3584 13.0201 59.3265C12.9371 59.262 12.9085 59.1493 12.9511 59.0533L19.6163 43.985C19.6473 43.9144 19.7121 43.8645 19.7882 43.8519C19.8005 43.8497 19.813 43.8488 19.8253 43.8488C19.8886 43.8488 19.95 43.8751 19.9936 43.9227C21.502 45.5658 23.5525 46.4341 25.9235 46.4341C26.4424 46.4341 26.9815 46.3923 27.5257 46.3098C27.5373 46.308 27.5487 46.3071 27.5601 46.3071C27.6306 46.3071 27.698 46.3399 27.7418 46.3968C27.7925 46.4631 27.803 46.5517 27.769 46.6281L21.4369 60.9448C21.4 61.0283 21.3177 61.081 21.2279 61.081C21.219 61.081 21.2101 61.0805 21.2009 61.0794Z" fill="white"/>
<path d="M19.8253 44.0774C21.3733 45.7635 23.4719 46.6626 25.9234 46.6626C26.4533 46.6626 26.9992 46.6207 27.5601 46.5356L21.228 60.8525L19.9437 53.7818L13.1602 59.1458L19.8253 44.0774ZM19.8253 43.6203C19.8007 43.6203 19.7758 43.6223 19.7511 43.6263C19.5989 43.6514 19.4696 43.7515 19.4072 43.8925L12.7421 58.9608C12.6571 59.153 12.7141 59.3785 12.8802 59.5071C12.9626 59.571 13.0614 59.6029 13.1602 59.6029C13.2604 59.6029 13.3606 59.57 13.4437 59.5043L19.6299 54.6126L20.7782 60.9341C20.8141 61.1317 20.9746 61.2827 21.174 61.3064C21.1921 61.3085 21.2103 61.3096 21.2282 61.3096C21.4074 61.3096 21.5722 61.2042 21.6461 61.0374L27.9781 46.7204C28.0456 46.5678 28.0246 46.3905 27.9233 46.258C27.836 46.1438 27.7012 46.0784 27.5601 46.0784C27.5373 46.0784 27.5144 46.0801 27.4915 46.0836C26.9584 46.1644 26.4309 46.2055 25.9234 46.2055C23.6182 46.2055 21.626 45.3626 20.1621 43.7682C20.0747 43.6732 19.9522 43.6203 19.8253 43.6203Z" fill="#374874"/>
<path d="M41.2172 4.23825C38.279 2.53078 34.2101 2.77128 29.7196 5.36388C20.7944 10.5168 13.5843 23.0069 13.6134 33.2609C13.6244 37.1606 14.6804 40.1751 16.476 42.1309C17.0262 42.7302 17.6459 43.2301 18.3264 43.6255L21.6758 45.572C20.9954 45.1766 20.3757 44.6767 19.8254 44.0774C18.0298 42.1216 16.9739 39.1071 16.9628 35.2074C16.9338 24.9533 24.1438 12.4633 33.069 7.31037C37.5596 4.71773 41.6285 4.4772 44.5667 6.18474L41.2172 4.23825Z" fill="#FBDBD0"/>
<path d="M25.9231 46.8912C23.4208 46.891 21.2541 45.9716 19.657 44.2321C17.7556 42.1609 16.7449 39.0406 16.7342 35.208C16.705 24.8966 23.9815 12.293 32.9547 7.11242C35.5371 5.62156 38.0148 4.86554 40.319 4.86554C43.032 4.86554 45.3269 5.93406 46.9554 7.95528C48.6159 10.0164 49.499 12.9876 49.509 16.5481C49.5188 19.9946 48.7079 23.8345 47.1639 27.6535C45.4032 32.0089 42.8289 36.0751 39.7193 39.4126C37.688 41.593 35.5244 43.3531 33.2885 44.6439C31.3338 45.7725 29.4179 46.4847 27.5945 46.7613C27.0273 46.8475 26.4648 46.8912 25.9231 46.8912Z" fill="white"/>
<path d="M40.3189 5.09411C45.6134 5.09411 49.2598 9.28819 49.2804 16.5487C49.2905 20.0866 48.4384 23.8912 46.9521 27.5678C45.2241 31.8422 42.6391 35.9434 39.552 39.2568C37.5971 41.355 35.4406 43.1374 33.1742 44.4458C31.2011 45.585 29.31 46.27 27.5601 46.5355C26.9994 46.6206 26.4531 46.6626 25.9234 46.6626C23.4718 46.6626 21.3735 45.7635 19.8253 44.0773C18.0297 42.1216 16.9738 39.1071 16.9627 35.2073C16.9337 24.9533 24.1438 12.4633 33.0689 7.31033C35.674 5.80627 38.1361 5.09411 40.3189 5.09411ZM40.3189 4.63696C37.974 4.63696 35.4579 5.4032 32.8403 6.91447C28.4555 9.44606 24.3436 13.7798 21.2621 19.1174C18.1805 24.455 16.4913 30.1697 16.5056 35.2086C16.5166 39.0994 17.5481 42.273 19.4886 44.3865C21.1302 46.1746 23.3554 47.1197 25.9234 47.1197C26.4767 47.1197 27.0504 47.0752 27.6286 46.9875C29.4807 46.7065 31.4234 45.9846 33.4028 44.8418C35.658 43.5397 37.8394 41.7655 39.8864 39.5684C43.0149 36.2106 45.6048 32.1201 47.3759 27.7392C48.9308 23.893 49.7474 20.0229 49.7375 16.5475C49.7273 12.9345 48.8268 9.91376 47.1333 7.81193C45.4599 5.73486 43.1035 4.63696 40.3189 4.63696Z" fill="#374874"/>
<path d="M33.0825 12.0887C39.7092 8.26271 45.1004 11.335 45.122 18.9496C45.1436 26.5661 39.7874 35.8415 33.1607 39.6675C26.5324 43.4943 21.1428 40.423 21.1212 32.8065C21.0996 25.1919 26.4542 15.9156 33.0825 12.0887ZM39.8434 32.5422L38.5402 24.7193L43.9376 15.5282L36.4516 18.6L33.0843 12.7407L29.7597 22.4636L22.2823 28.031L27.7125 30.9706L26.4594 40.2694L33.14 32.3631L39.8434 32.5422Z" fill="#C1DBF6"/>
<path d="M36.9695 3.14758C38.5496 3.14763 39.9833 3.52119 41.2172 4.23821L44.5666 6.18475C44.558 6.17972 44.5492 6.17527 44.5405 6.17031C47.4563 7.84982 49.266 11.4397 49.2804 16.5488C49.2905 20.0867 48.4384 23.8913 46.9521 27.5679L54.1892 35.4576L47.3975 37.9312L46.1574 46.4595L42.8079 44.5131L38.7492 40.0873C37.0124 41.8262 35.1348 43.3139 33.1742 44.4459C31.2011 45.585 29.31 46.27 27.5601 46.5355L21.2281 60.8524L17.8786 58.9059L17.3241 55.8532L13.1602 59.1457L9.81075 57.1992L16.4759 42.1308C14.6803 40.1751 13.6244 37.1606 13.6133 33.2608C13.5843 23.0068 20.7943 10.5168 29.7195 5.36386C32.3244 3.85991 34.7868 3.14742 36.9695 3.14758ZM36.9696 2.23328C34.5436 2.23312 31.9506 3.02 29.2624 4.57207C24.8091 7.14315 20.6378 11.5364 17.5168 16.9423C14.3955 22.3487 12.6846 28.145 12.6991 33.2634C12.7097 36.9994 13.6435 40.1037 15.4063 42.2889L8.97464 56.8294C8.78609 57.2557 8.94842 57.7555 9.35141 57.9898L12.7009 59.9363C12.8436 60.0192 13.0022 60.06 13.1601 60.06C13.3617 60.06 13.5622 59.9935 13.7273 59.8629L16.6967 57.5149L16.9791 59.0693C17.0269 59.3327 17.1878 59.5619 17.4193 59.6965L20.7687 61.6429C20.9097 61.7249 21.0684 61.7667 21.2281 61.7667C21.323 61.7667 21.4183 61.7519 21.5105 61.722C21.7577 61.6417 21.9591 61.46 22.0643 61.2222L28.1981 47.3536C29.9541 47.0174 31.7793 46.3069 33.6314 45.2376C35.3715 44.233 37.077 42.9441 38.7118 41.3992L42.1341 45.131C42.1965 45.199 42.2688 45.2572 42.3486 45.3036L45.698 47.2501C45.8397 47.3324 45.9984 47.3738 46.1574 47.3738C46.2932 47.3738 46.4292 47.3436 46.555 47.2828C46.8285 47.1508 47.0185 46.8916 47.0622 46.5911L48.2237 38.6034L54.5021 36.3167C54.7891 36.2122 55.0049 35.9713 55.0774 35.6747C55.1499 35.3779 55.0694 35.0647 54.863 34.8396L48.0128 27.3717C49.4511 23.6489 50.2043 19.9177 50.1948 16.5462C50.1798 11.2815 48.3446 7.32297 45.0258 5.39474L45.026 5.39424L41.6766 3.4477C40.2901 2.64201 38.7064 2.23334 36.9696 2.23328Z" fill="#374874"/>
</svg>

```

## File: static\img\promotional_program.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"/><g id="b"/><g id="c"/><g id="d"/><g id="e"><g><g><path d="M58.32,64.91c-.18,0-.36-.02-.53-.07l-27.73-7.62c-.87-.24-1.47-1.03-1.47-1.93v-19c0-.9,.6-1.69,1.47-1.93l27.73-7.62c.6-.16,1.25-.04,1.74,.34,.5,.38,.79,.97,.79,1.59V62.91c0,.62-.29,1.21-.79,1.59-.35,.27-.78,.41-1.21,.41Zm-25.73-11.15l23.73,6.52V31.29l-23.73,6.52v15.95Z" style="fill:#7c6576;"/><g><path d="M24.32,72.61c.25,.94,.85,1.72,1.7,2.21,.84,.49,1.82,.61,2.76,.36h0c.94-.25,1.72-.85,2.21-1.69,.48-.84,.61-1.82,.36-2.76l-3.64-13.53h-7.53l4.14,15.42Z" style="fill:none;"/><path d="M35.21,69.69l-3.46-12.87c.51-.36,.84-.95,.84-1.63v-18.82c0-1.1-.9-2-2-2h-11.19c-4.46,0-8.09,3.63-8.09,8.09v6.63c0,3.17,1.84,5.92,4.51,7.24,0,.02,0,.05,.02,.07l4.63,17.23c.53,1.97,1.79,3.62,3.56,4.63,1.17,.68,2.48,1.02,3.8,1.02,.67,0,1.33-.09,1.99-.26,1.97-.53,3.61-1.79,4.63-3.56s1.29-3.82,.76-5.79ZM15.3,42.47c0-2.26,1.84-4.09,4.09-4.09h9.19v14.82h-9.19c-1.33,0-2.51-.65-3.26-1.64-.52-.69-.83-1.53-.83-2.45v-6.63Zm15.68,31.01c-.49,.84-1.27,1.44-2.21,1.69h0c-.94,.25-1.92,.12-2.76-.36-.84-.49-1.44-1.27-1.7-2.21l-4.14-15.42h7.53l3.64,13.53c.25,.94,.12,1.92-.36,2.76Z" style="fill:#7c6576;"/></g><g><path d="M69.27,45.79c0-1.52-.97-2.81-2.33-3.31v6.62c1.35-.5,2.33-1.79,2.33-3.31Z" style="fill:none;"/><path d="M66.94,38.36v-12.33c0-2.94-2.39-5.33-5.33-5.33s-5.33,2.39-5.33,5.33v39.53c0,2.94,2.39,5.33,5.33,5.33s5.33-2.39,5.33-5.33v-12.33c3.58-.58,6.33-3.69,6.33-7.43s-2.75-6.85-6.33-7.43Zm-4,.44v26.76c0,.73-.6,1.33-1.33,1.33s-1.33-.6-1.33-1.33V26.02c0-.73,.6-1.33,1.33-1.33s1.33,.6,1.33,1.33v12.77Zm4,10.31v-6.62c1.35,.5,2.33,1.79,2.33,3.31s-.97,2.81-2.33,3.31Z" style="fill:#7c6576;"/></g></g><g><path d="M86.69,47.79h-6.51c-1.1,0-2-.9-2-2s.9-2,2-2h6.51c1.1,0,2,.9,2,2s-.9,2-2,2Z" style="fill:#7c6576;"/><path d="M84.2,64.37c-.51,0-1.02-.2-1.41-.59l-4.42-4.42c-.78-.78-.78-2.05,0-2.83s2.05-.78,2.83,0l4.42,4.42c.78,.78,.78,2.05,0,2.83-.39,.39-.9,.59-1.41,.59Z" style="fill:#7c6576;"/><path d="M79.78,35.63c-.51,0-1.02-.2-1.41-.59-.78-.78-.78-2.05,0-2.83l4.43-4.43c.78-.78,2.05-.78,2.83,0s.78,2.05,0,2.83l-4.43,4.43c-.39,.39-.9,.59-1.41,.59Z" style="fill:#7c6576;"/></g></g></g><g id="f"/><g id="g"/><g id="h"/></svg>
```

## File: static\img\promo_code.svg

```svg
<?xml version="1.0" encoding="UTF-8"?><svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><g id="a"><g><path d="M53.33,8.53L8.53,53.33c-2.82,2.82-2.82,7.4,0,10.23l4.85,4.85c5.04-4.89,13.09-4.85,18.08,.13s5.02,13.03,.13,18.08l4.85,4.85c2.82,2.82,7.4,2.82,10.23,0l44.8-44.8c2.82-2.82,2.82-7.4,0-10.23L63.56,8.53c-2.82-2.82-7.4-2.82-10.23,0Z" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><path d="M31.59,86.62c4.89-5.04,4.85-13.09-.13-18.08s-13.03-5.02-18.08-.13" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><g><line x1="19.65" y1="42.53" x2="22.48" y2="45.36" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><line x1="28.86" y1="51.74" x2="51.19" y2="74.07" style="fill:none; stroke:#7c6576; stroke-dasharray:0 0 9.02 9.02; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><line x1="54.38" y1="77.26" x2="57.21" y2="80.09" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/></g><line x1="57.32" y1="59.75" x2="57.32" y2="25.61" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><circle cx="69.58" cy="42.13" r="4.93" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/><circle cx="44.81" cy="42.92" r="4.93" style="fill:none; stroke:#7c6576; stroke-linecap:round; stroke-linejoin:round; stroke-width:4px;"/></g></g><g id="b"/><g id="c"/><g id="d"/><g id="e"/><g id="f"/><g id="g"/><g id="h"/></svg>
```

## File: static\src\img\ewallet.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M13.2138 24.3524C12.8908 24.5338 12.7282 24.7757 12.7275 25.0184L12.6284 59.9615C12.6291 59.7187 12.7918 59.4768 13.1148 59.2955L48.2384 39.5764L48.3374 4.6333L13.2138 24.3524Z" fill="#C1DBF6"/>
<path d="M52.7867 7.29175L52.6876 42.2348L17.5641 61.9539L17.6632 27.0109L52.7867 7.29175Z" fill="white"/>
<path d="M9.47156 59.9539L9.57063 25.0109C9.56866 25.707 10.0166 26.4049 10.9184 26.9437C12.7517 28.0391 15.778 28.0692 17.6631 27.0109L17.564 61.954C15.679 63.0123 12.6526 62.9821 10.8194 61.8868C9.91758 61.348 9.46958 60.65 9.47156 59.9539Z" fill="#FBDBD0"/>
<path d="M46.136 3.31799L48.3373 4.63326L13.2137 24.3524C12.5776 24.7095 12.5634 25.3016 13.1834 25.6721C13.802 26.0417 14.8242 26.0519 15.4603 25.6947L50.5839 5.97559L52.7866 7.29168L17.663 27.0108C15.778 28.0692 12.7516 28.039 10.9184 26.9437C9.08515 25.8483 9.12732 24.0955 11.0124 23.0371L46.136 3.31799Z" fill="white"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M48.53 4.52518C48.5898 4.63159 48.5519 4.76628 48.4455 4.82602L13.322 24.5452C13.0441 24.7011 12.9516 24.8789 12.9485 25.0125C12.9454 25.1446 13.028 25.3218 13.2968 25.4825C13.5643 25.6423 13.9335 25.7313 14.3176 25.7351C14.7015 25.7389 15.0757 25.6574 15.3523 25.5021L50.4758 5.78301C50.5822 5.72327 50.7169 5.7611 50.7766 5.86751C50.8364 5.97392 50.7985 6.10862 50.6921 6.16836L15.5686 25.8875C15.209 26.0894 14.7541 26.1814 14.3132 26.177C13.8725 26.1726 13.4213 26.0716 13.0701 25.8618C12.7189 25.652 12.4986 25.3479 12.5067 25.0022C12.5147 24.6578 12.7473 24.361 13.1056 24.1598L48.2292 4.44068C48.3356 4.38094 48.4703 4.41877 48.53 4.52518Z" fill="#374874"/>
<path d="M15.4604 25.6947L37.9874 2.06631C38.5993 1.4544 39.5026 1.16302 40.3768 1.30871C41.1635 1.42526 41.7754 2.03717 41.9211 2.79477L43.4938 9.93472C43.4938 9.93472 15.2127 25.9424 15.4604 25.6947Z" fill="white"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M38.1455 2.22077L16.5735 24.8475C17.3183 24.4357 18.3694 23.8492 19.6281 23.144C22.2443 21.6782 25.756 19.7006 29.2756 17.7152C32.7952 15.7297 36.3225 13.7366 38.97 12.2397C40.2937 11.4913 41.3975 10.867 42.1703 10.4298L43.2429 9.82284L41.7054 2.84231C41.705 2.84038 41.7046 2.83844 41.7042 2.83651C41.5757 2.16831 41.0356 1.62968 40.3444 1.5273L40.3405 1.52667C39.537 1.39276 38.7065 1.66094 38.1455 2.22077ZM43.4938 9.93473C43.6026 10.127 43.6025 10.1271 43.6023 10.1272L43.2851 10.3067C43.078 10.4239 42.7743 10.5958 42.3879 10.8144C41.615 11.2516 40.5112 11.876 39.1875 12.6244C36.54 14.1213 33.0125 16.1145 29.4928 18.1001C25.9731 20.0856 22.4609 22.0635 19.8441 23.5295C18.5358 24.2625 17.4509 24.8678 16.7007 25.2821C16.3257 25.4892 16.0336 25.649 15.8387 25.7534C15.7417 25.8053 15.6669 25.8445 15.6175 25.8691C15.5945 25.8806 15.5707 25.892 15.5516 25.8997C15.5465 25.9018 15.5385 25.9048 15.5291 25.9078L15.5286 25.9079C15.5237 25.9095 15.4976 25.9178 15.4647 25.9185C15.4409 25.9178 15.3738 25.9014 15.3337 25.8795C15.282 25.8299 15.2381 25.7093 15.2426 25.6511C15.2511 25.6213 15.2721 25.5786 15.2817 25.5645C15.2868 25.558 15.2949 25.5485 15.3005 25.5424L37.8275 1.91385L37.8312 1.91008C38.4935 1.24777 39.4678 0.933872 40.4112 1.09044C41.2915 1.22173 41.9738 1.90506 42.1376 2.75006L43.7096 9.8872C43.7304 9.98195 43.6871 10.0792 43.6026 10.127L43.4938 9.93473Z" fill="#374874"/>
<path d="M15.4604 25.6948L40.9597 4.4407C41.5716 3.82879 42.4748 3.53741 43.349 3.6831C44.1358 3.79965 45.1265 4.26587 45.2722 5.05261L46.3212 8.34527L15.4604 25.6948Z" fill="white"/>
<path fill-rule="evenodd" clip-rule="evenodd" d="M43.3126 3.90106C42.5083 3.76701 41.6768 4.0359 41.1158 4.59695C41.1111 4.60167 41.1061 4.60617 41.101 4.61044L17.4547 24.3201L46.0559 8.24087L45.0615 5.11969C45.0587 5.1109 45.0565 5.10193 45.0548 5.09285C44.9975 4.78354 44.7694 4.51538 44.4331 4.3035C44.0989 4.09291 43.6846 3.95621 43.3165 3.90169L43.3126 3.90106ZM43.3833 3.46483C42.4433 3.30884 41.4727 3.61989 40.8105 4.27726L15.3189 25.5251C15.2311 25.5982 15.2136 25.7261 15.2784 25.8202C15.3432 25.9142 15.4691 25.9434 15.5686 25.8874L46.4293 8.53789C46.5207 8.48653 46.5634 8.37807 46.5316 8.27821L45.4866 4.99817C45.3942 4.52784 45.0569 4.17417 44.6687 3.92959C44.2751 3.68163 43.8014 3.52704 43.3833 3.46483Z" fill="#374874"/>
<path d="M54.5285 28.6393L52.7351 29.6808L52.7205 17.0964L54.4556 18.062L54.5285 28.6393Z" fill="#374874"/>
<path d="M54.5285 28.6393L50.2888 31.1015C48.2054 32.3107 45.5538 31.3346 45.5538 28.4062C45.5538 25.4777 47.1565 22.5202 49.7935 20.9904L54.4265 18.3242L54.5285 28.6393Z" fill="#FBDBD0"/>
<path d="M49.3653 27.7718C49.4524 26.8245 48.9912 26.0076 48.3351 25.9472C47.679 25.8869 47.0765 26.6059 46.9894 27.5532C46.9022 28.5006 47.3634 29.3175 48.0195 29.3778C48.6756 29.4382 49.2781 28.7192 49.3653 27.7718Z" fill="white"/>
<path d="M39.9286 1.2717C40.0778 1.2717 40.2278 1.28384 40.3768 1.30866C41.1636 1.42523 41.7755 2.03712 41.9212 2.79474L42.1333 3.75797C42.3848 3.68569 42.6469 3.64706 42.9114 3.64706C43.057 3.64706 43.2033 3.65879 43.3489 3.68302C43.7696 3.74535 44.2477 3.90866 44.6246 4.16649L46.136 3.31797L48.3373 4.63326L48.3299 7.24101L50.584 5.97557L52.7866 7.29166L52.7588 17.1177L54.4556 18.0619L54.5285 28.6392L52.7231 29.6876L52.6876 42.2347L17.564 61.9539C16.6428 62.4711 15.449 62.7283 14.255 62.7283C13.0059 62.7283 11.7567 62.4467 10.8194 61.8867C9.91755 61.3479 9.46958 60.6499 9.47154 59.9538L9.57061 25.0107C9.57049 25.0516 9.57717 25.0924 9.58016 25.1332C9.52469 24.3738 10.0003 23.6053 11.0124 23.0371L26.0349 14.6032L37.9874 2.06622C38.495 1.55863 39.203 1.27166 39.9286 1.2717ZM39.9287 0.387817C38.9674 0.387772 38.032 0.771726 37.3625 1.44126C37.3575 1.44622 37.3526 1.45125 37.3477 1.45634L25.4866 13.8973L10.5797 22.2664C9.41483 22.9204 8.74219 23.8649 8.69451 24.893C8.68949 24.9307 8.68683 24.9692 8.68672 25.0083L8.58766 59.9513C8.58471 61.0017 9.21625 61.9585 10.366 62.6455C11.4094 63.2689 12.7905 63.6122 14.255 63.6122C15.6484 63.6122 16.9772 63.297 17.9967 62.7246L53.1202 43.0055C53.3981 42.8495 53.5705 42.556 53.5714 42.2373L53.6055 30.1973L54.9723 29.4035C55.2466 29.2443 55.4144 28.9503 55.4123 28.6332L55.3394 18.0559C55.3372 17.7373 55.1637 17.4445 54.8854 17.2896L53.6441 16.5989L53.6705 7.29422C53.6714 6.98229 53.5077 6.69296 53.24 6.53298L51.0373 5.21689C50.8978 5.13358 50.7409 5.09177 50.5839 5.09177C50.4349 5.09177 50.2858 5.12942 50.1513 5.20491L49.2181 5.72882L49.2211 4.63579C49.222 4.32386 49.0584 4.03456 48.7906 3.87458L46.5893 2.55927C46.4498 2.47595 46.293 2.43415 46.1359 2.43415C45.9869 2.43415 45.8378 2.47179 45.7033 2.54729L44.623 3.15378C44.2695 2.98871 43.8758 2.86843 43.4863 2.8099C43.2976 2.77893 43.1042 2.76322 42.9114 2.76322C42.8807 2.76322 42.8501 2.76361 42.8195 2.76439L42.7866 2.61518C42.5631 1.48194 41.6522 0.607795 40.5146 0.435635C40.3219 0.403932 40.1249 0.387817 39.9287 0.387817Z" fill="#374874"/>
</svg>

```

## File: static\src\img\loyalty.svg

```svg
<svg width="64" height="64" viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M46.1574 46.4595L42.8079 44.513L36.2025 37.3103L39.552 39.2568L46.1574 46.4595Z" fill="#FBDBD0"/>
<path d="M46.1575 46.6881C46.0945 46.6881 46.0329 46.662 45.989 46.614L39.3836 39.4113C39.3031 39.3234 39.3037 39.1883 39.385 39.1011C42.4569 35.804 45.0001 31.7863 46.7403 27.4821C46.7696 27.4093 46.8343 27.3569 46.9115 27.3428C46.9249 27.3404 46.9385 27.3392 46.9521 27.3392C47.0155 27.3392 47.0769 27.3656 47.1207 27.4133L54.3577 35.3033C54.4093 35.3596 54.4294 35.4379 54.4113 35.512C54.3932 35.5861 54.3392 35.6464 54.2673 35.6725L47.6041 38.0993L46.3836 46.4924C46.3709 46.5799 46.3091 46.6519 46.2249 46.6781C46.2026 46.6848 46.18 46.6881 46.1575 46.6881Z" fill="white"/>
<path d="M46.9521 27.5679L54.1892 35.4577L47.3975 37.9313L46.1574 46.4595L39.552 39.2568C42.6392 35.9434 45.2241 31.8423 46.9521 27.5679ZM46.9521 27.1107C46.9251 27.1107 46.8978 27.1131 46.8707 27.118C46.7163 27.146 46.5871 27.2511 46.5283 27.3965C44.8234 31.6137 42.227 35.7151 39.2175 38.9452C39.0549 39.1197 39.0538 39.39 39.2151 39.5658L45.8205 46.7685C45.9084 46.8644 46.0313 46.9166 46.1574 46.9166C46.2023 46.9166 46.2477 46.9101 46.292 46.8964C46.4607 46.8444 46.5843 46.7 46.6098 46.5253L47.8106 38.2673L54.3457 35.8872C54.4891 35.835 54.5971 35.7146 54.6333 35.5662C54.6695 35.4179 54.6293 35.2613 54.5261 35.1487L47.289 27.2588C47.2016 27.1636 47.079 27.1107 46.9521 27.1107Z" fill="#374874"/>
<path d="M21.228 60.8524L17.8786 58.9059L16.5942 51.8352L19.9437 53.7817L21.228 60.8524Z" fill="#FBDBD0"/>
<path d="M13.1603 59.1457L9.81079 57.1993L16.476 42.1309L19.8254 44.0774L13.1603 59.1457Z" fill="#FBDBD0"/>
<path d="M21.2009 61.0794C21.1011 61.0676 21.021 60.9919 21.0029 60.8932L19.7869 54.197L13.3018 59.3249C13.2603 59.3578 13.2103 59.3743 13.1601 59.3743C13.1107 59.3743 13.0614 59.3584 13.0201 59.3265C12.9371 59.262 12.9085 59.1493 12.9511 59.0533L19.6163 43.985C19.6473 43.9144 19.7121 43.8645 19.7882 43.8519C19.8005 43.8497 19.813 43.8488 19.8253 43.8488C19.8886 43.8488 19.95 43.8751 19.9936 43.9227C21.502 45.5658 23.5525 46.4341 25.9235 46.4341C26.4424 46.4341 26.9815 46.3923 27.5257 46.3098C27.5373 46.308 27.5487 46.3071 27.5601 46.3071C27.6306 46.3071 27.698 46.3399 27.7418 46.3968C27.7925 46.4631 27.803 46.5517 27.769 46.6281L21.4369 60.9448C21.4 61.0283 21.3177 61.081 21.2279 61.081C21.219 61.081 21.2101 61.0805 21.2009 61.0794Z" fill="white"/>
<path d="M19.8253 44.0774C21.3733 45.7635 23.4719 46.6626 25.9234 46.6626C26.4533 46.6626 26.9992 46.6207 27.5601 46.5356L21.228 60.8525L19.9437 53.7818L13.1602 59.1458L19.8253 44.0774ZM19.8253 43.6203C19.8007 43.6203 19.7758 43.6223 19.7511 43.6263C19.5989 43.6514 19.4696 43.7515 19.4072 43.8925L12.7421 58.9608C12.6571 59.153 12.7141 59.3785 12.8802 59.5071C12.9626 59.571 13.0614 59.6029 13.1602 59.6029C13.2604 59.6029 13.3606 59.57 13.4437 59.5043L19.6299 54.6126L20.7782 60.9341C20.8141 61.1317 20.9746 61.2827 21.174 61.3064C21.1921 61.3085 21.2103 61.3096 21.2282 61.3096C21.4074 61.3096 21.5722 61.2042 21.6461 61.0374L27.9781 46.7204C28.0456 46.5678 28.0246 46.3905 27.9233 46.258C27.836 46.1438 27.7012 46.0784 27.5601 46.0784C27.5373 46.0784 27.5144 46.0801 27.4915 46.0836C26.9584 46.1644 26.4309 46.2055 25.9234 46.2055C23.6182 46.2055 21.626 45.3626 20.1621 43.7682C20.0747 43.6732 19.9522 43.6203 19.8253 43.6203Z" fill="#374874"/>
<path d="M41.2172 4.23825C38.279 2.53078 34.2101 2.77128 29.7196 5.36388C20.7944 10.5168 13.5843 23.0069 13.6134 33.2609C13.6244 37.1606 14.6804 40.1751 16.476 42.1309C17.0262 42.7302 17.6459 43.2301 18.3264 43.6255L21.6758 45.572C20.9954 45.1766 20.3757 44.6767 19.8254 44.0774C18.0298 42.1216 16.9739 39.1071 16.9628 35.2074C16.9338 24.9533 24.1438 12.4633 33.069 7.31037C37.5596 4.71773 41.6285 4.4772 44.5667 6.18474L41.2172 4.23825Z" fill="#FBDBD0"/>
<path d="M25.9231 46.8912C23.4208 46.891 21.2541 45.9716 19.657 44.2321C17.7556 42.1609 16.7449 39.0406 16.7342 35.208C16.705 24.8966 23.9815 12.293 32.9547 7.11242C35.5371 5.62156 38.0148 4.86554 40.319 4.86554C43.032 4.86554 45.3269 5.93406 46.9554 7.95528C48.6159 10.0164 49.499 12.9876 49.509 16.5481C49.5188 19.9946 48.7079 23.8345 47.1639 27.6535C45.4032 32.0089 42.8289 36.0751 39.7193 39.4126C37.688 41.593 35.5244 43.3531 33.2885 44.6439C31.3338 45.7725 29.4179 46.4847 27.5945 46.7613C27.0273 46.8475 26.4648 46.8912 25.9231 46.8912Z" fill="white"/>
<path d="M40.3189 5.09411C45.6134 5.09411 49.2598 9.28819 49.2804 16.5487C49.2905 20.0866 48.4384 23.8912 46.9521 27.5678C45.2241 31.8422 42.6391 35.9434 39.552 39.2568C37.5971 41.355 35.4406 43.1374 33.1742 44.4458C31.2011 45.585 29.31 46.27 27.5601 46.5355C26.9994 46.6206 26.4531 46.6626 25.9234 46.6626C23.4718 46.6626 21.3735 45.7635 19.8253 44.0773C18.0297 42.1216 16.9738 39.1071 16.9627 35.2073C16.9337 24.9533 24.1438 12.4633 33.0689 7.31033C35.674 5.80627 38.1361 5.09411 40.3189 5.09411ZM40.3189 4.63696C37.974 4.63696 35.4579 5.4032 32.8403 6.91447C28.4555 9.44606 24.3436 13.7798 21.2621 19.1174C18.1805 24.455 16.4913 30.1697 16.5056 35.2086C16.5166 39.0994 17.5481 42.273 19.4886 44.3865C21.1302 46.1746 23.3554 47.1197 25.9234 47.1197C26.4767 47.1197 27.0504 47.0752 27.6286 46.9875C29.4807 46.7065 31.4234 45.9846 33.4028 44.8418C35.658 43.5397 37.8394 41.7655 39.8864 39.5684C43.0149 36.2106 45.6048 32.1201 47.3759 27.7392C48.9308 23.893 49.7474 20.0229 49.7375 16.5475C49.7273 12.9345 48.8268 9.91376 47.1333 7.81193C45.4599 5.73486 43.1035 4.63696 40.3189 4.63696Z" fill="#374874"/>
<path d="M33.0825 12.0887C39.7092 8.26271 45.1004 11.335 45.122 18.9496C45.1436 26.5661 39.7874 35.8415 33.1607 39.6675C26.5324 43.4943 21.1428 40.423 21.1212 32.8065C21.0996 25.1919 26.4542 15.9156 33.0825 12.0887ZM39.8434 32.5422L38.5402 24.7193L43.9376 15.5282L36.4516 18.6L33.0843 12.7407L29.7597 22.4636L22.2823 28.031L27.7125 30.9706L26.4594 40.2694L33.14 32.3631L39.8434 32.5422Z" fill="#C1DBF6"/>
<path d="M36.9695 3.14758C38.5496 3.14763 39.9833 3.52119 41.2172 4.23821L44.5666 6.18475C44.558 6.17972 44.5492 6.17527 44.5405 6.17031C47.4563 7.84982 49.266 11.4397 49.2804 16.5488C49.2905 20.0867 48.4384 23.8913 46.9521 27.5679L54.1892 35.4576L47.3975 37.9312L46.1574 46.4595L42.8079 44.5131L38.7492 40.0873C37.0124 41.8262 35.1348 43.3139 33.1742 44.4459C31.2011 45.585 29.31 46.27 27.5601 46.5355L21.2281 60.8524L17.8786 58.9059L17.3241 55.8532L13.1602 59.1457L9.81075 57.1992L16.4759 42.1308C14.6803 40.1751 13.6244 37.1606 13.6133 33.2608C13.5843 23.0068 20.7943 10.5168 29.7195 5.36386C32.3244 3.85991 34.7868 3.14742 36.9695 3.14758ZM36.9696 2.23328C34.5436 2.23312 31.9506 3.02 29.2624 4.57207C24.8091 7.14315 20.6378 11.5364 17.5168 16.9423C14.3955 22.3487 12.6846 28.145 12.6991 33.2634C12.7097 36.9994 13.6435 40.1037 15.4063 42.2889L8.97464 56.8294C8.78609 57.2557 8.94842 57.7555 9.35141 57.9898L12.7009 59.9363C12.8436 60.0192 13.0022 60.06 13.1601 60.06C13.3617 60.06 13.5622 59.9935 13.7273 59.8629L16.6967 57.5149L16.9791 59.0693C17.0269 59.3327 17.1878 59.5619 17.4193 59.6965L20.7687 61.6429C20.9097 61.7249 21.0684 61.7667 21.2281 61.7667C21.323 61.7667 21.4183 61.7519 21.5105 61.722C21.7577 61.6417 21.9591 61.46 22.0643 61.2222L28.1981 47.3536C29.9541 47.0174 31.7793 46.3069 33.6314 45.2376C35.3715 44.233 37.077 42.9441 38.7118 41.3992L42.1341 45.131C42.1965 45.199 42.2688 45.2572 42.3486 45.3036L45.698 47.2501C45.8397 47.3324 45.9984 47.3738 46.1574 47.3738C46.2932 47.3738 46.4292 47.3436 46.555 47.2828C46.8285 47.1508 47.0185 46.8916 47.0622 46.5911L48.2237 38.6034L54.5021 36.3167C54.7891 36.2122 55.0049 35.9713 55.0774 35.6747C55.1499 35.3779 55.0694 35.0647 54.863 34.8396L48.0128 27.3717C49.4511 23.6489 50.2043 19.9177 50.1948 16.5462C50.1798 11.2815 48.3446 7.32297 45.0258 5.39474L45.026 5.39424L41.6766 3.4477C40.2901 2.64201 38.7064 2.23334 36.9696 2.23328Z" fill="#374874"/>
</svg>

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
import { X2ManyField, x2ManyField } from "@web/views/fields/x2many/x2many_field";

export class LoyaltyX2ManyField extends X2ManyField {
    static template = "loyalty.LoyaltyX2ManyField";
}

export const loyaltyX2ManyField = {
    ...x2ManyField,
    component: LoyaltyX2ManyField,
};

registry.category("fields").add("loyalty_one2many", loyaltyX2ManyField);

```

## File: static\src\js\loyalty_list_view.js

```javascript
/** @odoo-module **/

import { registry } from "@web/core/registry";
import { listView } from "@web/views/list/list_view";
import { ListRenderer } from "@web/views/list/list_renderer";
import { useService } from "@web/core/utils/hooks";
import { Component, onWillStart } from "@odoo/owl";

export class LoyaltyActionHelper extends Component {
    static template = "loyalty.LoyaltyActionHelper";
    static props = ["noContentHelp"];
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

export class LoyaltyListRenderer extends ListRenderer {
    static template = "loyalty.LoyaltyListRenderer";
    static components = {
        ...LoyaltyListRenderer.components,
        LoyaltyActionHelper,
    };
};

export const LoyaltyListView = {
    ...listView,
    Renderer: LoyaltyListRenderer,
};

registry.category("views").add("loyalty_program_list_view", LoyaltyListView);

```

## File: static\src\js\portal\loyalty_card.js

```javascript
import publicWidget from '@web/legacy/js/public/public_widget';
import { rpc } from '@web/core/network/rpc';

import { PortalLoyaltyCardDialog } from './loyalty_card_dialog/loyalty_card_dialog'

publicWidget.registry.PortalLoyaltyWidget = publicWidget.Widget.extend({
    selector: '.o_loyalty_container',
    events: {
        'click .o_loyalty_card': '_onClickLoyaltyCard',
    },

    async _onClickLoyaltyCard(ev) {
        const card_id = ev.currentTarget.dataset.card_id;
        let data = await rpc(`/my/loyalty_card/${card_id}/values`);
        this.call("dialog", "add", PortalLoyaltyCardDialog, data);
    },

});

```

## File: static\src\js\portal\loyalty_card_dialog\loyalty_card_dialog.js

```javascript
import { Component } from '@odoo/owl';
import { Dialog } from '@web/core/dialog/dialog';

export class PortalLoyaltyCardDialog extends Component {
    static components = { Dialog };
    static template = 'loyalty.portal_loyalty_card_dialog';
    static props = ['*'];

    setup() {
        this.csrf_token = odoo.csrf_token;
    }
}

```

## File: static\src\js\portal\loyalty_card_dialog\loyalty_card_dialog.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="loyalty.portal_loyalty_card_dialog">
        <Dialog size="'md'" header="false" footer="false" t-on-click="props.close">
            <div class="d-flex align-items-center" t-on-click.stop="">
                <div class="d-flex justify-content-between w-100 align-items-center">
                    <div class="d-flex align-items-center gap-2">
                        <img
                            class="img-fluid"
                            t-att-src="props.img_path"
                            width="40"
                        />
                        <div class="fs-5"><t t-out="props.program.program_name"/></div>
                    </div>
                    <div class="d-flex flex-column align-items-end">
                        <span class="fs-5" t-out="props.card.points_display"/>
                        <i class="text-secondary fs-6" t-if="props.card.expiration_date">
                            Valid until <t t-out="props.card.expiration_date"/>
                        </i>
                    </div>
                </div>
                <div
                    type="button"
                    class="d-sm-block btn-close p-0 p-md-2"
                    t-on-click="props.close"
                />
            </div>
            <hr/>
            <div class="mt-0 pt-0" t-on-click.stop="">
                <div name="history_lines">
                    <t t-if="props.history_lines.length">
                        <strong>Last Transactions</strong>
                        <div
                            t-foreach="props.history_lines"
                            t-as="line"
                            t-key="line_index"
                            class="row px-2"
                        >
                            <div class="col-7">
                                <t t-if="line.order_id and line.order_portal_url">
                                    <a t-att-href="line.order_portal_url">
                                        <t t-out="line.description"/>
                                    </a>
                                </t>
                                <t t-else="">
                                    <t t-out="line.description"/>
                                </t>
                            </div>
                            <div t-out="line.points" class="text-end col-5"/>
                        </div>
                        <div class="mt-1 mb-1">
                            <a t-attf-href="/my/loyalty_card/{{props.card.id}}/history/">
                                -> View History
                            </a>
                        </div>
                    </t>
                    <t t-else="">
                        <p class="alert alert-warning">
                            There are currently no transaction lines for this card.
                        </p>
                    </t>
                </div>
            </div>
        </Dialog>
    </t>
</templates>

```

## File: static\src\xml\loyalty_templates.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <t t-name="loyalty.LoyaltyListRenderer" t-inherit="web.ListRenderer" t-inherit-mode="primary">
        <t t-call="web.ActionHelper" position="replace">
            <t t-if="showNoContentHelper">
                <LoyaltyActionHelper noContentHelp="props.noContentHelp"/>
            </t>
        </t>
    </t>

    <t t-name="loyalty.LoyaltyActionHelper">
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

    <t t-name="loyalty.LoyaltyX2ManyField" t-inherit-mode="primary" t-inherit="web.X2ManyField">
        <t t-if="displayControlPanelButtons" position="replace">
            <h4 t-esc="field.string or ''"/>
            <t t-if="displayControlPanelButtons">
                <div class="o_cp_buttons me-0 ms-auto" role="toolbar" aria-label="Control panel buttons" t-ref="buttons">
                    <div>
                        <button type="button" class="btn btn-secondary o-kanban-button-new" title="Create record" accesskey="c" t-on-click="() => this.onAdd()">
                            Add
                        </button>
                    </div>
                </div>
            </t>
        </t>
    </t>

    <t t-name="loyalty.LoyaltyCardListView.buttons" t-inherit-mode="primary" t-inherit="web.ListView.Buttons">
        <xpath expr="//div[hasclass('o_list_buttons')]" position="inside">
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
                            <label string="Balance" for="points_display"/>
                            <button
                                name="action_loyalty_update_balance"
                                class="p-0 text-info fw-normal"
                                type="object"
                            >
                                <field name="points_display" nolabel="1"/>
                            </button>
                        </group>
                    </group>
                    <notebook invisible="not id">
                        <page string="History Lines">
                            <field name="history_ids">
                                <list>
                                    <field name="description"/>
                                    <field name="order_id"/>
                                    <field name="create_date" string="Date"/>
                                    <field name="issued"/>
                                    <field name="used"/>
                                </list>
                            </field>
                        </page>
                    </notebook>
                </sheet>
                <chatter/>
            </form>
        </field>
    </record>

    <record id="loyalty_card_view_tree" model="ir.ui.view">
        <field name="name">loyalty.card.view.list</field>
        <field name="model">loyalty.card</field>
        <field name="arch" type="xml">
            <list string="Coupons" edit="false" delete="false" js_class="loyalty_card_list_view">
                <field name="code" readonly="1"/>
                <field name="create_date" optional="hide"/>
                <field name="points_display" string="Balance"/>
                <field name="expiration_date"/>
                <field name="program_id"/>
                <field name="partner_id"/>
                <button name="action_coupon_send" string="Send" type="object" icon="fa-paper-plane-o"/>
            </list>
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
                <filter name="active" string="Active" domain="['&amp;', ('program_id.active', '=', True), '&amp;', ('points', '>', 0), '|', ('expiration_date', '>=', context_today().strftime('%Y-%m-%d 00:00:00')), ('expiration_date', '=', False)]"/>
                <filter name="inactive" string="Inactive" domain="['|', ('program_id.active', '=', False), '|', ('points', '&lt;=', 0), ('expiration_date', '&lt;', context_today().strftime('%Y-%m-%d 23:59:59'))]"/>
            </search>
        </field>
    </record>

    <record id="loyalty_card_action" model="ir.actions.act_window">
        <field name="name">Coupons</field>
        <field name="res_model">loyalty.card</field>
        <field name="view_mode">list,form</field>
        <field name="domain">[('program_id', '=', active_id)]</field>
        <field name="context">{'create': False}</field>
        <field name="help" type="html">
            <h1>No Coupons Found.</h1>
            <p>There haven't been any coupons generated yet.</p>
        </field>
    </record>
</odoo>

```

## File: views\loyalty_history_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="loyalty_history_form" model="ir.ui.view">
        <field name="name">loyalty.history.view.form</field>
        <field name="model">loyalty.history</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <group>
                            <field name="card_id"/>
                            <field name="description"/>
                            <field name="order_id" invisible="not order_id"/>
                            <field name="order_model" invisible="True"/>
                        </group>
                        <group>
                            <field name="issued"/>
                            <field name="used"/>
                        </group>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

</odoo>

```

## File: views\loyalty_mail_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="loyalty_mail_view_tree" model="ir.ui.view">
        <field name="name">loyalty.mail.view.list</field>
        <field name="model">loyalty.mail</field>
        <field name="arch" type="xml">
            <list editable="bottom">
                <field name="trigger"/>
                <field name="points" string="Limit"
                    invisible="trigger != 'points_reach'"
                    required="trigger == 'points_reach'"/>
                <field name="mail_template_id"/>
            </list>
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
                        invisible="program_type != 'coupons'"/>
                    <button name="%(loyalty_generate_wizard_action)d" string="Generate Gift Cards" class="btn-primary" type="action"
                        invisible="program_type != 'gift_card'"/>
                    <button name="%(loyalty_generate_wizard_action)d" string="Generate eWallet" class="btn-primary" type="action"
                        invisible="program_type != 'ewallet'" context="{'default_mode': 'selected'}"/>
                </header>
                <sheet>
                    <widget name="web_ribbon" title="Archived" bg_color="text-bg-danger" invisible="active"/>
                    <div class="oe_button_box" name="button_box">
                        <button class="oe_stat_button" type="object" name="action_open_loyalty_cards" icon="fa-tags">
                            <div class="o_form_field o_stat_info">
                                <span class="o_stat_value">
                                    <field name="coupon_count"/>
                                </span>
                                <span class="o_stat_text" invisible="program_type not in ('coupons', 'next_order_coupons')">Coupons</span>
                                <span class="o_stat_text" invisible="program_type != 'loyalty'">Loyalty Cards</span>
                                <span class="o_stat_text" invisible="program_type not in ('promotion', 'buy_x_get_y')">Promos</span>
                                <span class="o_stat_text" invisible="program_type != 'promo_code'">Discount</span>
                                <span class="o_stat_text" invisible="program_type != 'gift_card'">Gift Cards</span>
                                <span class="o_stat_text" invisible="program_type != 'ewallet'">eWallets</span>
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
                                <field name="program_type" widget="filterable_selection" readonly="coupon_count != 0" options="{'blacklisted_values': ['gift_card', 'ewallet']}"/>
                                <p class="text-muted" invisible="program_type != 'coupons'" colspan="2">
                                    Generate &amp; share coupon codes manually. It can be used in eCommerce, Point of Sale or regular orders to claim the Reward. You can define constraints on its usage through conditional rule.
                                    <div groups="base.group_no_one">
                                        When generating coupon, you can define a specific points value that can be exchanged for rewards.
                                    </div>
                                </p>
                                <p class="text-muted" invisible="program_type != 'loyalty'" colspan="2">
                                    When customers make an order, they accumulate points they can exchange for rewards on the current order or on a future one.
                                </p>
                                <p class="text-muted" invisible="program_type != 'promotion'" colspan="2">
                                    Set up conditional rules on the order that will give access to rewards for customers
                                    <div groups="base.group_no_one">
                                        Each rule can grant points to the customer he will be able to exchange against rewards
                                    </div>
                                </p>
                                <p class="text-muted" invisible="program_type != 'promo_code'" colspan="2">
                                    Define Discount codes on conditional rules then share it with your customers for rewards.
                                </p>
                                <p class="text-muted" invisible="program_type != 'buy_x_get_y'" colspan="2">
                                    Grant 1 credit for each item bought then reward the customer with Y items in exchange of X credits.
                                </p>
                                <p class="text-muted" invisible="program_type != 'next_order_coupons'" colspan="2">
                                    Drive repeat purchases by sending a unique, single-use coupon code for the next purchase when a customer buys something in your store.
                                </p>
                                <p class="text-muted" invisible="program_type != 'gift_card'" colspan="2">
                                    Gift Cards are created manually or automatically sent by email when the customer orders a gift card product.
                                    <br/>
                                    Then, Gift Cards can be used to pay orders.
                                </p>
                                <p class="text-muted" invisible="program_type != 'ewallet'" colspan="2">
                                    eWallets are created manually or automatically when the customer orders a eWallet product.
                                    <br/>
                                    Then, eWallets are proposed during the checkout, to pay orders.
                                </p>
                            </div>
                            <field name="trigger_product_ids" string="Gift Card Products" widget="many2many_tags" invisible="program_type != 'gift_card'"/>
                            <field name="trigger_product_ids" string="eWallet Products" widget="many2many_tags" invisible="program_type != 'ewallet'"/>
                            <field name="payment_program_discount_product_id" groups="base.group_no_one" invisible="program_type not in ('gift_card', 'ewallet')"/>
                            <field name="mail_template_id" invisible="program_type not in ('gift_card', 'ewallet')"/>
                            <field name="currency_id"/>
                            <field name="currency_symbol" invisible="1"/>
                            <field name="pricelist_ids"
                                   widget="many2many_tags"
                                   invisible="program_type in ('gift_card', 'ewallet')"
                                   groups="product.group_product_pricelist"/>
                            <field name="portal_point_name" invisible="program_type in ('loyalty', 'gift_card', 'ewallet')" string="Points Unit" groups="base.group_no_one"/>
                            <field name="portal_point_name" invisible="program_type in ('gift_card', 'ewallet') or program_type != 'loyalty'" string="Points Unit"/>
                            <field name="portal_visible" invisible="1"/>
                            <field name="portal_visible" groups="base.group_no_one" string="Show points Unit" invisible="program_type in ('gift_card', 'ewallet')"/>
                            <field name="trigger" invisible="1"/>
                            <field name="trigger" string="Program trigger" groups="base.group_no_one" widget="selection" readonly="1" force_save="1"/>
                            <field name="applies_on" invisible="1"/>
                            <field name="applies_on" string="Use points on" groups="base.group_no_one" widget="radio" force_save="1" readonly="program_type != 'loyalty'"/>
                        </group>
                        <group>
                            <field name="date_from" invisible="program_type in ('gift_card', 'ewallet')"/>
                            <field name="date_to" invisible="program_type in ('gift_card', 'ewallet')"/>
                            <label for="limit_usage" invisible="program_type in ('gift_card', 'ewallet')"/>
                            <span invisible="program_type in ('gift_card', 'ewallet')">
                                <field name="limit_usage" class="oe_inline"/>
                                <span invisible="not limit_usage"> to <field name="max_usage" class="oe_inline"/> usages</span>
                            </span>
                            <field name="company_id" invisible="1"/>
                            <field name="company_id" groups="base.group_multi_company"/>
                            <field name="available_on" invisible="1"/>
                            <label class="o_form_label" for="available_on" string="Available On" invisible="1"/>
                            <div id="o_loyalty_program_availabilities" invisible="1"/>
                            <field name="portal_point_name" invisible="program_type not in ('gift_card', 'ewallet')" string="Displayed as" groups="base.group_no_one"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Rules &amp; Rewards" name="rules_rewards" invisible="program_type in ('gift_card', 'ewallet')">
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
                        <page string="Rewards" name="rewards" groups="base.group_no_one" invisible="program_type not in ('gift_card', 'ewallet')">
                            <group>
                                <group groups="base.group_no_one">
                                    <field name="reward_ids" colspan="2" mode="kanban" nolabel="1" add-label="Add a reward"
                                      class="o_loyalty_kanban_inline" widget="loyalty_one2many" context="{'currency_symbol': currency_symbol, 'program_type': program_type}"/>
                                </group>
                            </group>
                        </page>
                        <page string="Communications" name="communications" invisible="applies_on == 'current' or program_type in ('gift_card', 'ewallet')">
                            <field name="communication_plan_ids" mode="list"/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="loyalty_program_view_tree" model="ir.ui.view">
        <field name="name">loyalty.program.view.list</field>
        <field name="model">loyalty.program</field>
        <field name="arch" type="xml">
            <list js_class="loyalty_program_list_view">
                <field name="sequence" widget="handle"/>
                <field name="name"/>
                <field name="program_type"/>
                <field name="coupon_count_display" string="Items"/>
                <field name="company_id" groups="base.group_multi_company"/>
            </list>
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
        <field name="view_mode">list,form</field>
        <field name="domain">[('program_type', 'not in', ('gift_card', 'ewallet'))]</field>
        <field name="help" type="html">
            <div class="o_loyalty_not_found container mt64">
                <h1>No program found.</h1>
                <p class="lead">Create one from scratch, or use a templates below:</p>
            </div>
        </field>
    </record>

    <record id="action_loyalty_program_tree_discount_loyalty" model="ir.actions.act_window.view">
        <field name="view_mode">list</field>
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
        <field name="view_mode">list,form</field>
        <field name="context">{'menu_type': 'gift_ewallet', 'default_program_type': 'gift_card'}</field>
        <field name="domain">[('program_type', 'in', ('gift_card', 'ewallet'))]</field>
        <field name="help" type="html">
            <div class="o_loyalty_not_found container mt64">
                <h1>No loyalty program found.</h1>
                <p class="lead">Create a new one from scratch, or use one of the templates below.</p>
            </div>
        </field>
    </record>

    <record id="action_loyalty_program_tree_gift_card_ewallet" model="ir.actions.act_window.view">
        <field name="view_mode">list</field>
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
                        <group string="Reward" name="reward_type_group" invisible="program_type in ('ewallet','gift_card')">
                            <field name="reward_type" widget="selection" force_save="1" readonly="program_type == 'buy_x_get_y'"/>
                            <label for="discount" invisible="reward_type != 'discount'"/>
                            <div class="d-flex flex-row" invisible="reward_type != 'discount'">
                                <field
                                    name="discount"
                                    class="oe_inline me-1 text-end"
                                    style="max-width: 4rem;"
                                />
                                <field name="discount_mode" no_label="1" class="w-auto me-1"/>
                                <span>on</span>
                            </div>
                            <label for="discount_applicability" string="" invisible="reward_type != 'discount'"/>
                            <field name="discount_applicability" nolabel="1" widget="radio" invisible="reward_type != 'discount'"/>
                        </group>

                        <group string="Among" invisible="reward_type != 'product'">
                            <field name="reward_product_qty" string="Quantity rewarded"/>
                            <field name="reward_product_id" required="reward_type == 'product' and not reward_product_ids"/>
                            <field name="reward_product_tag_id" required="reward_type == 'product' and not reward_product_ids"/>
                        </group>

                        <group string="Discount" invisible="reward_type != 'discount' or program_type in ('gift_card','ewallet')">
                            <field name="discount_max_amount"/>
                            <field name="tax_ids" invisible="1"/>
                            <field name="discount_product_domain" groups="base.group_no_one" widget="domain" options="{'model': 'product.product', 'in_dialog': true}" invisible="discount_applicability != 'specific'"/>
                            <field name="discount_product_ids" widget="many2many_tags" invisible="discount_applicability != 'specific'"/>
                            <field name="discount_product_category_id" invisible="discount_applicability != 'specific'"/>
                            <field name="discount_product_tag_id" invisible="discount_applicability != 'specific'"/>
                        </group>
                    </group>
                    <group string="Points" invisible="not user_has_debug and program_type not in ('loyalty', 'buy_x_get_y')">
                        <group>
                            <label for="required_points" string="In exchange of"/>
                            <div class="o_row">
                                <field
                                    name="required_points"
                                    class="oe_edit_only col-2 oe_inline text-end pe-2"
                                    style="max-width: 4rem;"
                                />
                                <field name="point_name" no_label="1"/>
                                <span invisible="not clear_wallet"> (or more)</span>
                            </div>
                            <label for="clear_wallet" string="Clear all promo point(s)" invisible="(not user_has_debug and program_type in ('loyalty', 'buy_x_get_y')) or program_type in ('gift_card', 'ewallet')"/>
                            <div class="o_row" invisible="(not user_has_debug and program_type in ('loyalty', 'buy_x_get_y')) or program_type in ('gift_card', 'ewallet')">
                                <field name="clear_wallet"/>
                            </div>
                        </group>
                    </group>
                    <group>
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
                <field name="company_id" invisible="1"/>
                <field name="currency_id"/>
                <field name="reward_type"/>
                <field name="discount_applicability"/>
                <field name="clear_wallet"/>
                <field name="program_type"/>
                <field name="user_has_debug"/>
                <templates>
                    <t t-name="card" class="flex-row">
                        <div class="mw-75 flex-grow-1" name="reward_info">
                            <t t-if="record.reward_type.raw_value === 'discount'">

                                <t t-if="record.discount">
                                    <field name="discount"/><field name="discount_mode"/> discount <t t-if="record.discount_max_amount.raw_value > 0">( Max <field name="discount_max_amount"/> )</t>
                                </t>

                                <t t-if="record.discount_applicability.raw_value === 'specific'">

                                    <div class="fw-bold text-decoration-underline mt-3">Applied to:</div>

                                    <div t-if="record.discount_product_ids.raw_value.length > 0" class="d-flex">
                                        <i class="fa fa-cube fa-fw" title="Products"/> <field name="discount_product_ids" widget="many2many_tags"/>
                                    </div>
                                    <div t-if="record.discount_product_category_id.raw_value" class="d-flex">
                                        <i class="fa fa-cubes fa-fw" title="Product Categories"/> <field name="discount_product_category_id"/>
                                    </div>
                                    <div t-if="record.discount_product_tag_id.raw_value" class="d-flex">
                                        <i class="fa fa-tags fa-fw" title="Product Tags"/> <field name="discount_product_tag_id"/>
                                    </div>
                                    <div t-if="record.discount_product_domain.raw_value &amp;&amp; record.discount_product_domain.raw_value !== '[]'" groups="base.group_no_one" class="d-flex">
                                        <i class="fa fa-search fa-fw" title="Product Domain"/> <field name="discount_product_domain"/>
                                    </div>
                                </t>

                                <t t-elif="record.discount_applicability.raw_value === 'cheapest'">
                                    on the cheapest product
                                </t>

                                <t t-elif="record.discount_applicability.raw_value === 'order'">
                                    on your order
                                </t>
                            </t>

                            <t t-elif="record.reward_type.raw_value === 'product'">
                                <div class="mb-3">Free product</div>
                                <div t-if="record.reward_product_id.raw_value" class="d-flex">
                                    <i class="fa fa-cube fa-fw" title="Product Domain"/><field name="reward_product_id" class="me-1"/> <t t-if="record.reward_product_qty.raw_value > 1"> x <field name="reward_product_qty" class="ms-1"/></t>
                                </div>
                                <div t-if="record.reward_product_tag_id.raw_value" class="d-flex">
                                    <i class="fa fa-tags fa-fw" title="Product Tags"/> <field name="reward_product_tag_id"/>
                                </div>
                            </t>
                        </div>

                        <div class="o_loyalty_kanban_card_right float-end text-muted" invisible="not user_has_debug and program_type not in ('loyalty', 'buy_x_get_y')">
                            <div class="fw-bold text-decoration-underline">In exchange of</div>
                            <t t-if="record.clear_wallet.raw_value">
                                all <field name="point_name"/> (if at least <field name="required_points"/> <field name="point_name"/>)
                            </t>
                            <t t-else="">
                                <field name="required_points"/> <field name="point_name"/>
                            </t>
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
                    <group invisible="program_type != 'promo_code'">
                        <group>
                            <field name="code" required="program_type == 'promo_code'"/>
                        </group>
                    </group>
                    <group>
                        <group>
                            <separator string="Conditions" colspan="2"/>
                            <field name="minimum_qty"/>
                            <label for="minimum_amount" invisible="program_type == 'buy_x_get_y'"/>
                            <div class="d-flex flex-row">
                                <field name="minimum_amount" class="oe_inline me-1"/>
                                <field name="minimum_amount_tax_mode" class="ms-1"/>
                            </div>
                            <separator string="Among" colspan="2"/>
                            <field name="product_domain" groups="base.group_no_one" widget="domain" options="{'model': 'product.product', 'in_dialog': true}"/>
                            <field name="product_ids" widget="many2many_tags"/>
                            <field name="product_category_id"/>
                            <field name="product_tag_id"/>
                        </group>
                        <group invisible="not user_has_debug and program_type not in ('loyalty', 'buy_x_get_y')">
                            <separator string="Point(s)" colspan="2"/>
                            <span colspan="2" invisible="program_type != 'coupons'">Grant the amount of coupon points defined as the coupon value</span>
                            <label for="reward_point_amount" string="Grant" invisible="program_type not in ('promotion', 'promo_code', 'next_order_coupons', 'loyalty', 'buy_x_get_y')"/>
                            <div class="d-flex flex-row" invisible="program_type not in ('promotion', 'promo_code', 'next_order_coupons', 'loyalty', 'buy_x_get_y')">
                                <field name="reward_point_amount" class="oe_inline me-1"/>
                                <field name="reward_point_name" class="w-auto"/>
                            </div>
                            <label
                                for="reward_point_mode"
                                string=""
                                invisible="program_type not in ('promotion', 'promo_code', 'next_order_coupons', 'loyalty', 'buy_x_get_y')"
                            />
                            <field
                                name="reward_point_mode"
                                widget="radio"
                                nolabel="1"
                                invisible="program_type not in ('promotion', 'promo_code', 'next_order_coupons', 'loyalty', 'buy_x_get_y')"
                            />
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
                <field name="minimum_amount_tax_mode"/>
                <field name="program_type"/>
                <field name="user_has_debug"/>
                <templates>
                    <t t-name="card" class="flex-row">
                        <div class="mw-75 flex-grow-1">
                            <div t-if="record.code.raw_value">Discount code <field name="code"/></div>
                            <div t-if="record.minimum_qty.raw_value > 0">If minimum <field name="minimum_qty"/> item(s) bought</div>
                            <div t-if="record.minimum_amount.raw_value > 0">If minimum <field name="minimum_amount"/> spent<t t-if="record.minimum_amount_tax_mode.raw_value === 'excl'"> (tax excluded)</t></div>

                            <t t-if="record.product_ids.raw_value.length != 0 || record.product_category_id.raw_value || record.product_tag_id.raw_value">
                                <div class="fw-bold text-decoration-underline mt-3">Among:</div>

                                <div t-if="record.product_ids.raw_value.length > 0" class="d-flex">
                                    <i class="fa fa-cube fa-fw" title="Products"/> <field name="product_ids" widget="many2many_tags"/>
                                </div>
                                <div t-if="record.product_category_id.raw_value" class="d-flex">
                                    <i class="fa fa-cubes fa-fw" title="Product Categories"/> <field name="product_category_id"/>
                                </div>
                                <div t-if="record.product_tag_id.raw_value" class="d-flex">
                                    <i class="fa fa-tags fa-fw" title="Product Tags"/> <field name="product_tag_id"/>
                                </div>
                                <div t-if="record.product_ids.raw_value.length === 0 and !record.product_category_id.raw_value &amp;&amp; !record.product_tag_id.raw_value" class="d-flex">
                                    <i class="fa fa-cube fa-fw" title="Products"/><span>All Products</span>
                                </div>
                                <div t-if="record.product_domain.raw_value and record.product_domain.raw_value !== '[]'" groups="base.group_no_one" class="d-flex">
                                    <i class="fa fa-search fa-fw" title="Product Domain"/> <field name="product_domain"/>
                                </div>
                            </t>
                        </div>

                        <div class="o_loyalty_kanban_card_right float-end text-muted" invisible="not user_has_debug and program_type not in ('loyalty', 'buy_x_get_y')">
                            <p invisible="program_type != 'coupons'">
                                <div class="fw-bold text-decoration-underline">Grant</div>
                                the value of the coupon
                            </p>
                            <p invisible="program_type not in ('promotion', 'promo_code', 'next_order_coupons', 'loyalty', 'buy_x_get_y')">
                                <div class="fw-bold text-decoration-underline">Grant</div>
                                <field name="reward_point_amount"/>
                                <field name="reward_point_name" class="ms-1"/>
                                <field name="reward_point_mode" class="ms-1"/>
                            </p>
                        </div>
                    </t>
                </templates>
            </kanban>
        </field>
    </record>
</odoo>

```

## File: views\portal_templates.xml

```xml
<odoo>
    <template id="loyalty_buttons" inherit_id="portal.side_content">
        <div name="portal_contact" position="after">
            <div class='o_loyalty_container'>
                <div
                    t-foreach="cards_per_programs"
                    t-as="program"
                    class="portal-loyalty-buttons cursor-pointer"
                >
                    <div
                        t-att-data-card_id="card.id"
                        t-foreach="cards_per_programs[program]"
                        t-as="card"
                        class="o_loyalty_card d-flex my-2"
                    >
                        <span
                            t-attf-class="o_notification_bar rounded-start"
                            style="width: 0.5rem; background-color: var(--cyan)"
                        />
                        <div class="w-100 py-3 ps-3 pe-5 border border-start-0 rounded-end d-flex flex-row gap-3">
                            <img
                                class="img-fluid"
                                t-attf-src="/loyalty/static/src/img/{{ program.program_type }}.svg"
                                width="40"
                            />
                            <div class="d-flex flex-column gap-1">
                                <div>
                                    <strong t-out="program.display_name"/>
                                </div>
                                <p id="card_points" class="m-0 text-600">
                                    <t t-out="card.points_display"/>
                                </p>
                                <t t-if="
                                    program.program_type == 'loyalty'
                                    and any(
                                        reward.required_points &lt;= card.points for reward in program.reward_ids
                                    )
                                ">
                                    <p class="m-0 text-info">A reward is waiting for you</p>
                                </t>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <template id="loyalty_card_history_template" name="My Loyalty History">
        <t t-call="portal.portal_layout">
            <t t-set="breadcrumbs_searchbar" t-value="True"/>
            <t t-call="portal.portal_searchbar">
                <t t-set="title">Loyalty Transaction</t>
            </t>
            <t t-if="not history_lines">
                <p class="alert alert-warning">
                    There are currently no transaction lines for this card.
                </p>
            </t>
            <t t-else="" t-call="portal.portal_table">
                <thead>
                    <tr class="active">
                        <th>Description</th>
                        <th>Order</th>
                        <th>Date</th>
                        <th>Issued</th>
                        <th>Used</th>
                    </tr>
                </thead>
                <tbody>
                    <tr t-foreach="history_lines" t-as="line">
                        <td><span t-out="line.description"/></td>
                        <td t-if="line.order_id">
                            <t t-set="order_portal_url" t-value="line._get_order_portal_url()"/>
                            <t t-set="order_description" t-value="line._get_order_description()"/>
                            <a t-if="order_portal_url" t-att-href="order_portal_url">
                                <t t-out="order_description"/>
                            </a>
                            <t t-else="">
                                <t t-out="order_description"/>
                            </t>
                        </td>
                        <td t-else=""/>
                        <td><span t-out="line.create_date"/></td>
                        <td><span t-out="line.card_id._format_points(line.issued)"/></td>
                        <td><span t-out="line.card_id._format_points(line.used)"/></td>
                    </tr>
                </tbody>
            </t>
        </t>
    </template>

    <template
        id="portal_loyalty_history_breadcrumbs"
        name="Portal layout : Loyalty History Lines"
        inherit_id="portal.portal_breadcrumbs"
    >
        <xpath expr="//ol[hasclass('o_portal_submenu')]" position="inside">
            <li class="breadcrumb-item" t-if="page_name == 'loyalty_history'">
                <span>History</span>
            </li>
        </xpath>
    </template>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="res_partner_form" model="ir.ui.view">
        <field name="name">res.partner.view.buttons</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="priority" eval="11"/>
        <field name="arch" type="xml">
            <div name="button_box" position="inside">
                <button name="action_view_loyalty_cards"
                    type="object"
                    class="oe_stat_button"
                    icon="fa-money"
                    groups="base.group_system"
                    invisible="loyalty_card_count == 0">
                    <field string="Loyalty Cards" name="loyalty_card_count" widget="statinfo"/>
                </button>
            </div>
        </field>
    </record>

</odoo>

```

## File: wizard\base_partner_merge.py

```python
from odoo import models


class MergePartnerAutomatic(models.TransientModel):
    _inherit = 'base.partner.merge.automatic.wizard'

    def _update_foreign_keys(self, src_partners, dst_partner):
        """ Override of base to merge corresponding nominative loyalty cards."""
        self._merge_loyalty_cards(src_partners, dst_partner)
        super()._update_foreign_keys(src_partners, dst_partner)

    def _merge_loyalty_cards(self, src_partners, dst_partner):
        """ Merge nominative loyalty cards.

        :param src_partners: recordset of source res.partner records to merge
        :param dst_partner: destination res.partner record
        """
        LoyaltyCard = self.env['loyalty.card'].sudo()
        cards_per_program = dict(
            LoyaltyCard._read_group(
                domain=[
                    ('partner_id', 'in', src_partners.ids),
                    '|',
                        ('program_id.applies_on', '=', 'both'),
                        '&',
                            ('program_id.program_type', 'in', ('ewallet', 'loyalty')),
                            ('program_id.applies_on', '=', 'future'),
                ],
                groupby=['program_id'],
                aggregates=['id:recordset'],
            )
        )
        for program, cards in cards_per_program.items():
            total_points = sum(card.points for card in cards)
            dst_card = LoyaltyCard.search([
                ('partner_id', '=', dst_partner.id),
                ('program_id', '=', program.id),
            ], limit=1)
            if dst_card:
                final_card = dst_card
                total_points += dst_card.points
            else:
                final_card = cards[0]
            final_card.sudo().write({'partner_id': dst_partner.id, 'points': total_points})
            (cards - final_card).sudo().write({'points': 0, 'active': False})

```

## File: wizard\loyalty_card_update_balance.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, fields, models
from odoo.exceptions import ValidationError


class LoyaltyCardUpdateBalance(models.TransientModel):
    _name = 'loyalty.card.update.balance'
    _description = "Update Loyalty Card Points"

    card_id = fields.Many2one(
        comodel_name='loyalty.card',
        required=True,
        readonly=True,
    )
    old_balance = fields.Float(related='card_id.points')
    new_balance = fields.Float()
    description = fields.Char(required=True)

    def action_update_card_point(self):
        if self.old_balance == self.new_balance or self.new_balance < 0:
            raise ValidationError(
                _("New Balance should be positive and different then old balance.")
            )
        difference = self.new_balance - self.old_balance
        used = 0
        issued = 0
        if difference > 0:
            issued = difference
        else:
            used = abs(difference)

        self.env['loyalty.history'].create({
            'card_id': self.card_id.id,
            'description': self.description or _("Gift for customer"),
            'used': used,
            'issued': issued,
        })
        self.card_id.points = self.new_balance

```

## File: wizard\loyalty_card_update_balance_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="loyalty_card_update_balance_form" model="ir.ui.view">
        <field name="name">loyalty.card.update.balance.view.form</field>
        <field name="model">loyalty.card.update.balance</field>
        <field name="arch" type="xml">
            <form string="Update Balance">
                <sheet>
                    <h3>You are about to change the balance of the card</h3>
                    <group>
                        <field name="card_id" invisible="1"/>
                        <field name="old_balance"/>
                        <field name="new_balance"/>
                        <field name="description" placeholder="Example: Gift for customer"/>
                    </group>
                </sheet>
                <footer>
                    <button
                        name="action_update_card_point"
                        type="object"
                        class="btn-primary"
                        data-hotkey="s"
                    >
                        Confirm
                    </button>
                    <button special="cancel" data-hotkey="x">Cancel</button>
                </footer>
            </form>
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
    confirmation_message = fields.Char(compute='_compute_confirmation_message')
    description = fields.Text(string="Description")

    def _get_partners(self):
        self.ensure_one()
        if self.mode != 'selected':
            return self.env['res.partner']
        domains = []
        if self.customer_ids:
            domains.append([('id', 'in', self.customer_ids.ids)])
        if self.customer_tag_ids:
            domains.append([('category_id', 'in', self.customer_tag_ids.ids)])
        return self.env['res.partner'].search(expression.OR(domains) if domains else [])

    @api.depends('program_type', 'points_granted', 'coupon_qty')
    def _compute_confirmation_message(self):
        self.confirmation_message = False
        for wizard in self:
            program_desc = dict(wizard._fields['program_type']._description_selection(wizard.env))
            wizard.confirmation_message = _("You're about to generate %(program_type)s with a value of %(value)s for %(customer_number)i customers",
                program_type=program_desc[wizard.program_type],
                value=wizard.points_granted,
                customer_number=wizard.coupon_qty,
            )

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
        coupons = self.env['loyalty.card'].create(coupon_create_vals)
        self.env['loyalty.history'].create([
            {
                'description': self.description or _("Gift For Customer"),
                'card_id': coupon.id,
                'issued': self.points_granted,
            } for coupon in coupons
        ])
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
                            <field name="mode" widget="radio" invisible="program_type == 'ewallet'"/>
                            <field name="customer_ids" widget="many2many_tags" placeholder="For all customers" invisible="mode == 'anonymous'"/>
                            <field name="customer_tag_ids" widget="many2many_tags" invisible="mode == 'anonymous'" options="{'color_field': 'color'}"/>
                            <field name="coupon_qty" string="Quantity to generate" readonly="mode == 'selected'" required="mode == 'anonymous'"/>
                            <label string="Coupon value" for="points_granted" groups="base.group_no_one" invisible="program_type != 'coupons'"/>
                            <span class="d-inline-block" groups="base.group_no_one" invisible="program_type != 'coupons'">
                                <field name="points_granted" class="oe_inline"/>
                                <field name="points_name" class="oe_inline"/>
                            </span>
                            <label string="Gift Card value" for="points_granted" invisible="program_type != 'gift_card'"/>
                            <span class="d-inline-block" invisible="program_type != 'gift_card'">
                                <field name="points_granted" class="w-auto oe_inline me-1"/>
                                <field name="points_name" class="d-inline" no_label="1"/>
                            </span>
                            <label string="eWallet value" for="points_granted" invisible="program_type != 'ewallet'"/>
                            <span class="d-inline-block" invisible="program_type != 'ewallet'">
                                <field name="points_granted" class="w-auto oe_inline me-1"/>
                                <field name="points_name" class="d-inline" no_label="1"/>
                            </span>
                            <field name="valid_until"/>
                        </group>
                        <group class="d-flex flex-column">
                            <div class="border-bottom w-100 fw-bold">Description</div>
                            <field name="description" nolabel="1" placeholder="Example: Gift for customer" required="True"/>
                        </group>
                    </group>
                    <div class="alert alert-warning text-center" invisible="customer_ids or mode == 'anonymous'" role="alert">
                        <field name="confirmation_message"/>
                    </div>
                </sheet>
                <footer>
                    <button name="generate_coupons" type="object" class="btn-primary" data-hotkey="q">
                        <span invisible="not will_send_mail">
                            Generate and Send 
                        </span>
                        <span invisible="will_send_mail">
                            Generate 
                        </span>
                        <field name="program_type" nolabel="1"/>
                    </button>
                    <button special="cancel" data-hotkey="x" string="Cancel"/>
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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import base_partner_merge
from . import loyalty_card_update_balance
from . import loyalty_generate_wizard

```

